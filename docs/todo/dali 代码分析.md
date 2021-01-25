1. 首先定义外部数据ExternalInputIterator迭代器，实现对外部数据的访问。

2. 定义外部数据的ExternalSourcePipeline，初始化声明全部op及参数，定义静态图确定逻辑关系，编写迭代器填充数据。

3.  定义pipeline的pytorch迭代器，用于train获取数据。

 

分析一个batch的处理过程：

For循环访问迭代器pii，首次执行build静态图，将所有op按顺序排列，并实例化（待补充）。

Build过程

调用build函数，首先_prepare_graph（define_graph），输入的是用户自定义的静态图。

在后端实现代码中找到pipeline的class，实例化这个类。

将参数batchsize，numthreads等参数传入，并构造pipeline类

设置pipeline允许cpu、gpu overlapped（此处应该就是提速或使用mixed模式），允许异步执行，cpu和gpu的batch深度不分开，均为默认的2

设置output=用户自定义的tuple格式

解析全部op，将参数传递给上述构造类，添加op，调用AddOperator()添加op，并传递到对应的device

其中mixed的op，例如imagedecoder需要分成两步骤，cpustage和gpustage，判断op执行device是否存在及op是否符合逻辑，然后设置存在一个list里。

将需要输出的数据和执行的device组成一个tuple，_names_and_devices

至此，_prepare_graph结束，执行Build，调用self._pipe.Build(self._names_and_devices)，将状态位置为true，代表build完成    self._built = True

Build过程：输入的是self._names_and_devices的list，

创建executor

根据之前传入执行模式参数，确定返回async_pipelined_executor执行器，

将pipeline的参数传入并初始化执行器，之后将全部的图，即op传入执行器。

build图后，关于用户的数据，img和label，分别实例化了一个ExternalSource类，各自初始化后形成一个空列表。



关于多线程

初始化执行器的时候，创建了线程池

```c++
template <typename WorkspacePolicy, typename QueuePolicy>
class DLL_PUBLIC Executor : public ExecutorBase, public WorkspacePolicy, public QueuePolicy {
 public:
  DLL_PUBLIC inline Executor(int batch_size, int num_thread, int device_id,
                             size_t bytes_per_sample_hint, bool set_affinity = false,
                             int max_num_stream = -1, int default_cuda_stream_priority = 0,
                             QueueSizes prefetch_queue_depth = QueueSizes{2, 2})
      : batch_size_(batch_size),
        device_id_(device_id),
        bytes_per_sample_hint_(bytes_per_sample_hint),
        callback_(nullptr),
        stream_pool_(max_num_stream, true, default_cuda_stream_priority),
        event_pool_(),
        thread_pool_(num_thread, device_id, set_affinity),
        exec_error_(false),
        queue_sizes_(prefetch_queue_depth),
        mixed_op_stream_(0),
        gpu_op_stream_(0) {
    DALI_ENFORCE(batch_size_ > 0, "Batch size must be greater than 0.");
    DALI_ENFORCE(device_id >= 0, "Device id must be non-negative.");

    stage_queue_depths_ = QueuePolicy::GetQueueSizes(prefetch_queue_depth);
  }
```

即，其中的thread pool，构造函数如下：

```c++
ThreadPool::ThreadPool(int num_thread, int device_id, bool set_affinity)
    : threads_(num_thread), running_(true), work_complete_(true), active_threads_(0) {
  DALI_ENFORCE(num_thread > 0, "Thread pool must have non-zero size");
#if NVML_ENABLED
  nvml::Init();
#endif
  // Start the threads in the main loop
  for (int i = 0; i < num_thread; ++i) {
    threads_[i] = std::thread(std::bind(&ThreadPool::ThreadMain, this, i, device_id, set_affinity));
  }
  tl_errors_.resize(num_thread);
}
```

 

 

4. 执行第一个batch，

首先执行p.schedule_run()，_prefetch，_run_once，_iter_setup，此时从user定义的代码获取数据，feed进jepgs和labels，迭代器计数+1，self._batches_to_consume += 1

```python
def _run_once(self):
    """Start running the whole pipeline once without waiting for its results.

    If the pipeline was created with `exec_async` option set to `True`,
    this function will return without waiting for the execution to end."""
    try:
        if not self._last_iter:
            self._iter_setup()
            self._batches_to_consume += 1
        # Special case to prevent a deadlock if user didn't release the only buffer
        if not self._exec_async and self._prefetch_queue_depth == 1:
            self.release_outputs()
        self._run_cpu()
        self._run_gpu()
    except StopIteration:
        self._last_iter = True
```

调用iter_setup读取用户数据，并feed给之前创建好的img和label的list，待处理_batches_to_consume计数+1，准备运算

```python
    def iter_setup(self):
        try:
            (images, labels) = self.iterator.next()
            self.feed_input(self.jpegs, images)
            self.feed_input(self.labels, labels)
        except StopIteration:
            self.iterator = iter(self.external_data)
            raise StopIteration
```

分别执行cpu和gpu上的op，之前graph已定义，并对各自待处理的batch计数进行增减。

```python
def _run_cpu(self):
    """Run CPU portion of the pipeline."""
    if not self._built:
        raise RuntimeError("Pipeline must be built first.")
    if not self._last_iter:
        self._pipe.RunCPU()
        self._cpu_batches_to_consume += 1

def _run_gpu(self):
    """Run GPU portion of the pipeline."""
    if not self._built:
        raise RuntimeError("Pipeline must be built first.")
    if self._cpu_batches_to_consume > 0:
        self._pipe.RunGPU()
        self._cpu_batches_to_consume -= 1
        self._gpu_batches_to_consume += 1
```

以下是cpu的c++实现，mixed和gpu类似，此处注意mixed和gpu通过event控制是否需要执行计算，如有output则创建mixed和gpu的新队列，传给queue policy的handller，

```c++
void AsyncPipelinedExecutor::RunCPU() {
  CheckForErrors();
  std::unique_lock<std::mutex> lock(GetReadyMutex());
  ++cpu_work_counter_;
  lock.unlock();
  cpu_thread_.DoWork([this]() {
        // Run the cpu work. We know there is cpu
        // work so we do not have to wait to take
        // the work
        std::unique_lock<std::mutex> lock(GetReadyMutex());
        DALI_ENFORCE(cpu_work_counter_ > 0,
            "Internal error, thread has no cpu work.");
        --cpu_work_counter_;

        if (exec_error_ || IsStopSignaled()) {
          mixed_work_cv_.notify_all();#此处会通知，如果有错误
          return;
        }
        lock.unlock();

        PipelinedExecutor::RunCPU();

        // Mark that there is now mixed work to do
        // and signal to any threads that are waiting
        std::unique_lock<std::mutex> mixed_lock(GetReadyMutex());
        ++mixed_work_counter_;
        mixed_work_cv_.notify_one();#通知下一步
      });
}
```

运算完成后，调用share_ouput返回batch处理后的结果，重制计数器。



```python
def share_outputs(self):
    """Returns the outputs of the pipeline.

    Main difference to :meth:`outputs`
    is that share_outputs doesn't release returned buffers, release_outputs
    need to be called for that. If the pipeline is executed asynchronously,
    this function blocks until the results become available. It provides
    the user with better control about when he wants to run the pipeline, when he wants
    to obtain the resulting buffers and when they can be returned to DALI pool when the
    results have been consumed.
    Needs to be used together with :meth:`release_outputs`
    and :meth:`schedule_run`
    Should not be mixed with :meth:`run` in the same pipeline.

    :return:
        A list of `TensorList` objects for respective pipeline outputs
    """
    with self._check_api_type_scope(types.PipelineAPIType.SCHEDULED):
        if self._batches_to_consume == 0 or self._gpu_batches_to_consume == 0:
            raise StopIteration
        self._batches_to_consume -= 1
        self._gpu_batches_to_consume -= 1
        return self._pipe.ShareOutputs()
```

调用到excutor底层，在workspace中，根据pipeline_outputs_的tensor_id从graph中寻找数据，加入output队列，

```c++
for (size_t i = 0; i < pipeline_outputs_.size(); i++) {
  auto out_tensor_id = pipeline_outputs_[i];
  auto &out_tensor = graph_->Tensor(out_tensor_id);
  auto op_type = graph_->Node(out_tensor.producer.node).op_type;
  auto storage_dev = out_tensor.producer.storage_device;
  VALUE_SWITCH(storage_dev, storage_dev_static, (StorageDevice::GPU, StorageDevice::CPU),
  (
    VALUE_SWITCH(op_type, op_type_static, (OpType::MIXED, OpType::GPU),
    (
      auto &queue = get_queue<op_type_static, storage_dev_static>(
          tensor_to_store_queue_[out_tensor_id]);
      auto stage_output_idx = output_idx[op_type_static];
      ws->AddOutput(queue[stage_output_idx]);
    ), DALI_FAIL("Invalid op type"));  // NOLINT(whitespace/parens)
  ), DALI_FAIL("Invalid storage device"));  // NOLINT(whitespace/parens)
}
```

在addoutput中填充vector，outputs_.push_back({name, device});最后返回

```python
.def("ShareOutputs",
    [](Pipeline *p) {
      DeviceWorkspace ws;
      p->ShareOutputs(&ws);

      py::list list;
      for (int i = 0; i < ws.NumOutput(); ++i) {
        if (ws.OutputIsType<CPUBackend>(i)) {
          list.append(&ws.Output<CPUBackend>(i));
        } else {
          list.append(&ws.Output<GPUBackend>(i));
        }
      }
      return py::cast<py::tuple>(list);
    }, py::return_value_policy::take_ownership)
```

同理检查mixed和gpu的队列。

```c++
// We than need to wait for GPU outputs from Mixed & GPU stages that are computed asynchronously.
// If the output event list is not empty, it means that there are outputs on GPU that we
// have to wait for.
if (!mixed_output_events_.empty()) {
  auto queue_idx = output_idx[OpType::MIXED];
  CUDA_CALL(cudaEventSynchronize(mixed_output_events_.GetEvent(queue_idx)));
}
if (!gpu_output_events_.empty()) {
  auto queue_idx = output_idx[OpType::GPU];
  CUDA_CALL(cudaEventSynchronize(gpu_output_events_.GetEvent(queue_idx)));
}
```

 

-- -

28.5G

