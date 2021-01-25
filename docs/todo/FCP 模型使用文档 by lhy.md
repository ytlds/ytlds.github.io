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