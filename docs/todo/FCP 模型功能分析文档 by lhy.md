1. 模型运行环境配置

镜像版本：`registry.momenta.works/perception_hpc/torchland:release`

因镜像内无模型依赖包，所以进入容器内安装。执行命令 `slurm batch sbatch.sh -J [jobname]` 提交任务，sbatch.sh 内容如下:

```shell
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --gres=gpu:8
#SBATCH --partition=nv2080ti-test
#SBATCH --workdir=/home/liuhuiya/model/mmdetection
#SBATCH --dataset=deep3d-flc
#SBATCH --framework=registry.momenta.works/perception_hpc/torchland:release

sleep infinity
```

使用 `slurm exec [JOBID] bash` 进入节点进行调试，根据 FCP 同事邮件说明，首先安装依赖库：
>pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple mmcv==0.2.14
pip3 install imgaug==0.2.6 albumentations imagecorruptions terminaltables
进入mmdection目录，执行 python3 setup.py develop --user

2. 运行模型

进入 mmdetection 目录，执行以下命令

```shell
# 切换 PyTorch1.4.0 环境
source /opt/torch1.4/bin/activate

# 使用 8 个 GPU 开始训练
python3 -m torch.distributed.launch --nproc_per_node=8 tools/train.py configs/deep3d/t_ig_pc_rl_rg_lw_pa_rs_bak.py --launcher pytorch

# 或执行分布式训练脚本
./tools/dist_train.sh configs/deep3d/t_ig_pc_rl_rg_lw_pa_rs.py 8
```

模型开始运行，打印如下结果：

2020-03-05 09:20:25,198 - INFO - Epoch [32][4150/5999]  lr: 0.00010, eta: 11:25:44, time: 0.416, data_time: 0.231, memory: 1644, loss_cls: nan, loss_bbox: nan, loss_full_bbox: nan, loss_centerness: nan, loss_size: nan, loss_location: nan, loss_y: 0.0000, loss_regress: 0.0000, loss_direction: nan, loss_cos: nan, loss_alpha_constrain: nan, loss_relative: nan, loss: nan

如果报错请确认环境是否存在异常，如无法解决请联系 MTS。

3. 模型训练过程简析

<details>
<summary> 以下模型信息来自 FCP 组邮件，展开查看 </summary>
单帧3D的one stage模型：
该模型我们主要基于mmdetection和FCOS搭建。
其源码位于https://gitlab.momenta.works/1v1r/rd/mmdetection，配置文件选用configs/deep3d/t_ig_pc_rl_rg_lw_pa_rs.py，可参照tools/test.py构建模型。
模型的一个参数列表可通过https://gitlab.momenta.works/1v1r/rd/fcp3d/blob/gaoyu/benchmark/models/monocular3d/normal_theta_size.pth访问。m=torch.load(该文件)，再通过m['model'].state_dict()访问它的参数(下同)。(注意此时mmdetection需要已被安装或在PYTHONPATH内)
该模型使用方式是直接输入一张图片和相机参数即可，参照https://gitlab.momenta.works/1v1r/rd/fcp3d/blob/gaoyu/benchmark/test/struct_test.py内TestTorchModel。
运行mmdetection需要额外安装：
  pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple mmcv==0.2.14 && \
  pip3 install imgaug==0.2.6 albumentations imagecorruptions terminaltables运行   
</details>

在 `mmdetection/tools/train.py` 文件中，main() 函数首先导入配置文件，初始化如模型类型、工作路径、GPU 数量、launcher 等参数。

然后调用 `build_detector()`构建模型，代码如下：

```python
# mmdetection/mmdet/models/builder.py
def build_detector(cfg, train_cfg=None, test_cfg=None):
    return build(cfg, DETECTORS, dict(train_cfg=train_cfg, test_cfg=test_cfg))
    
def build(cfg, registry, default_args=None):
    if isinstance(cfg, list):
        modules = [
            build_from_cfg(cfg_, registry, default_args) for cfg_ in cfg
        ]
        return nn.Sequential(*modules)
    else:
        return build_from_cfg(cfg, registry, default_args)
```

DETECTORS 是 Registry 类的一个实例化对象，最终调用 build_from_cfg 构建模型。*Deep3DTheta*

```python
# mmdection/mmdet/utils/registry.py
def build_from_cfg(cfg, registry, default_args=None):
    """Build a module from config dict.

    Args:
        cfg (dict): Config dict. It should at least contain the key "type".
        registry (:obj:`Registry`): The registry to search the type from.
        default_args (dict, optional): Default initialization arguments.

    Returns:
        obj: The constructed object.
    """
    assert isinstance(cfg, dict) and 'type' in cfg
    assert isinstance(default_args, dict) or default_args is None
    args = cfg.copy()
    obj_type = args.pop('type')
    if mmcv.is_str(obj_type):
        obj_cls = registry.get(obj_type)
        if obj_cls is None:
            raise KeyError('{} is not in the {} registry'.format(
                obj_type, registry.name))
    elif inspect.isclass(obj_type):
        obj_cls = obj_type
    else:
        raise TypeError('type must be a str or valid type, but got {}'.format(
            type(obj_type)))
    if default_args is not None:
        for name, value in default_args.items():
            args.setdefault(name, value)
    return obj_cls(**args)
```

完成模型构建之后，调用 build_dataset() 构造训练数据集，代码如下：

```python
# mmdetection/mmdet/datasets/builder.py
def build_dataset(cfg, default_args=None):
    if isinstance(cfg, (list, tuple)):
        dataset = ConcatDataset([build_dataset(c, default_args) for c in cfg])
    elif cfg['type'] == 'RepeatDataset':
        dataset = RepeatDataset(
            build_dataset(cfg['dataset'], default_args), cfg['times'])
    elif isinstance(cfg['ann_file'], (list, tuple)):
        dataset = _concat_dataset(cfg, default_args)
    else:
        dataset = build_from_cfg(cfg, DATASETS, default_args)

    return dataset
```

模型和数据集准备完成后，调用 `train_detector()` 开始训练，代码如下：

```python
# mmdetection/mmdet/apis/train.py
def train_detector(model,
                   dataset,
                   cfg,
                   distributed=False,
                   validate=False,
                   logger=None):
    print(cfg.log_level)
    if logger is None:
        logger = get_root_logger(cfg.log_level)

    # start training
    if distributed:
        logger.info("distributed train")
        _dist_train(model, dataset, cfg, validate=validate)
    else:
        logger.info("no distributed train")
        _non_dist_train(model, dataset, cfg, validate=validate)
```

根据是否启动分布式执行不同的分支，这里我们执行分布式训练分支，代码如下：


```python
# mmdetection/mmdet/apis/train.py
def _dist_train(model, dataset, cfg, validate=False):
    # prepare data loaders
    dataset = dataset if isinstance(dataset, (list, tuple)) else [dataset]
    data_loaders = [
        build_dataloader(
            ds, cfg.data.imgs_per_gpu, cfg.data.workers_per_gpu, dist=True)
        for ds in dataset
    ]
    # put model on gpus
    model = MMDistributedDataParallel(model.cuda())
        #device_ids=[torch.cuda.current_device()],
        #broadcast_buffers=False)

    # build runner
    optimizer = build_optimizer(model, cfg.optimizer)
    runner = Runner(model, batch_processor, optimizer, cfg.work_dir,
                    cfg.log_level)


    # fp16 setting
    fp16_cfg = cfg.get('fp16', None)
    if fp16_cfg is not None:
        optimizer_config = Fp16OptimizerHook(**cfg.optimizer_config,
                                             **fp16_cfg)
    else:
        optimizer_config = DistOptimizerHook(**cfg.optimizer_config)

    # register hooks
    runner.register_training_hooks(cfg.lr_config, optimizer_config,
                                   cfg.checkpoint_config, cfg.log_config)
    runner.register_hook(DistSamplerSeedHook())

    ……

    if cfg.resume_from:
        runner.resume(cfg.resume_from)
    elif cfg.load_from:
        runner.load_checkpoint(cfg.load_from)

    runner.run(data_loaders, cfg.workflow, cfg.total_epochs)
```

首先构建 data_loaders ，这里调用了 pytorch 内部的 DataLoader 方法创建(可参考 [PyTorch DataLoader 数据加载过程简析](https://confluence.momenta.works/pages/viewpage.action?pageId=62588452) ）。

接下来实例化 runner 对象（定义在 MMCV），通过 `runner.run()` 函数开始训练，根据 workflow (本模型中`workflow = [('train', 1)]`) 的参数选择调用 `train()` 还是 `val()` 。

```python
# mmcv/runner/runner.py
def run(self, data_loaders, workflow, max_epochs, **kwargs):

    ……

    while self.epoch < max_epochs:
        for i, flow in enumerate(workflow):
            mode, epochs = flow
            if isinstance(mode, str):  # self.train()
                if not hasattr(self, mode):
                    raise ValueError(
                        'runner has no method named "{}" to run an epoch'.
                        format(mode))
                epoch_runner = getattr(self, mode)
            elif callable(mode):  # custom train()
                epoch_runner = mode
            else:
                raise TypeError('mode in workflow must be a str or '
                                'callable function, not {}'.format(
                                    type(mode)))
            for _ in range(epochs):
                if mode == 'train' and self.epoch >= max_epochs:
                    return
                epoch_runner(data_loaders[i], **kwargs)

    time.sleep(1)  # wait for some hooks like loggers to finish
    self.call_hook('after_run')
```
在 `train()` 函数中描述了训练过程，如下：

```python
# mmcv/runner/runner.py
def train(self, data_loader, **kwargs):
    self.model.train()
    self.mode = 'train'
    self.data_loader = data_loader
    self._max_iters = self._max_epochs * len(data_loader)
    self.call_hook('before_train_epoch')
    for i, data_batch in enumerate(data_loader):
        self._inner_iter = i
        self.call_hook('before_train_iter')
        # 调用batch_processor这个函数处理一个batch，前向传播
        outputs = self.batch_processor(
            self.model, data_batch, train_mode=True, **kwargs)
        if not isinstance(outputs, dict):
            raise TypeError('batch_processor() must return a dict')
        if 'log_vars' in outputs:
            self.log_buffer.update(outputs['log_vars'],
                                   outputs['num_samples'])
        self.outputs = outputs
        self.call_hook('after_train_iter') # 后向传播
        self._iter += 1

    self.call_hook('after_train_epoch') # 后向传播
    self._epoch += 1
```

在训练过程中，调用 `batch_processor()` 完成前向传播：

```python
# mmdetection/mmdet/apis/train.py
def batch_processor(model, data, train_mode):
    # 模型计算数据得到 loss，即前向传播
    losses = model(**data)
    # 处理 loss
    loss, log_vars = parse_losses(losses)

    outputs = dict(
        loss=loss, log_vars=log_vars, num_samples=len(data['img'].data))

    return outputs
  
def parse_losses(losses):
    log_vars = OrderedDict()
    for loss_name, loss_value in losses.items():
        if isinstance(loss_value, torch.Tensor):
            log_vars[loss_name] = loss_value.mean()
        elif isinstance(loss_value, list):
            log_vars[loss_name] = sum(_loss.mean() for _loss in loss_value)
        else:
            raise TypeError(
                '{} is not a tensor or list of tensors'.format(loss_name))

    loss = sum(_value for _key, _value in log_vars.items() if 'loss' in _key)

    log_vars['loss'] = loss
    for name in log_vars:
        log_vars[name] = log_vars[name].item()

    return loss, log_vars
```

通过注册 hook 完成后向传播，调用 `call_hook('after_train_iter')`  和 `call_hook('after_train_epoch')` 分别在完成一个 batch 和一个 epoch 后的处理。注册 hook 的过程如下：

```python
# mmdetection/mmdet/apis/train.py
def _dist_train(model, dataset, cfg, validate=False):

		……
  
    # register hooks
    runner.register_training_hooks(cfg.lr_config, optimizer_config,
                                   cfg.checkpoint_config, cfg.log_config)
    runner.register_hook(DistSamplerSeedHook())
    
    ……

```

```python
# mmcv/runner/runner.py
def register_training_hooks(self,
                            lr_config,
                            optimizer_config=None,
                            checkpoint_config=None,
                            log_config=None):
    """Register default hooks for training.

    Default hooks include:

    - LrUpdaterHook
    - OptimizerStepperHook
    - CheckpointSaverHook
    - IterTimerHook
    - LoggerHook(s)
    """
    if optimizer_config is None:
        optimizer_config = {}
    if checkpoint_config is None:
        checkpoint_config = {}
    self.register_lr_hooks(lr_config)
    self.register_hook(self.build_hook(optimizer_config, OptimizerHook))
    self.register_hook(self.build_hook(checkpoint_config, CheckpointHook))
    self.register_hook(IterTimerHook())
    if log_config is not None:
        self.register_logger_hooks(log_config)
```

调用 `register_hook()` 注册 hook 构建 list，通过 `call_hook()` 寻找 list 中已经注册的 hook 执行对应的动作，即 OptimizerHook和CheckpointHook，分别对应于 `call_hook('after_train_iter')`  和 `call_hook('after_train_epoch')`。

其中 `call_hook('after_train_iter')`  过程即对应于 `zero_grad(), backward(), step()`。

`call_hook('after_train_epoch')` 在整个 epoch 结束后存储 checkpoint。

```python
# mmcv/runner/hooks/optimizer.py
class OptimizerHook(Hook):

    def __init__(self, grad_clip=None):
        self.grad_clip = grad_clip

    def clip_grads(self, params):
        clip_grad.clip_grad_norm_(
            filter(lambda p: p.requires_grad, params), **self.grad_clip)

    def after_train_iter(self, runner):
        runner.optimizer.zero_grad()
        runner.outputs['loss'].backward()
        if self.grad_clip is not None:
            self.clip_grads(runner.model.parameters())
        runner.optimizer.step()
```

`call_hook('after_train_epoch')` 在整个 epoch 结束后存储 checkpoint。

```python
# mmcv/runner/hooks/checkpoint.py
class CheckpointHook(Hook):

    def __init__(self,
                 interval=-1,
                 save_optimizer=True,
                 out_dir=None,
                 **kwargs):
        self.interval = interval
        self.save_optimizer = save_optimizer
        self.out_dir = out_dir
        self.args = kwargs

    @master_only
    def after_train_epoch(self, runner):
        if not self.every_n_epochs(runner, self.interval):
            return

        if not self.out_dir:
            self.out_dir = runner.work_dir
        runner.save_checkpoint(
            self.out_dir, save_optimizer=self.save_optimizer, **self.args)
```






