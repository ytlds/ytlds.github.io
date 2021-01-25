## 1 项目背景和目标

### **1.1 项目背景**

PyTorch 进行训练任务时，典型的 pipeline 如下图所示，使用 DataLoader 加载图片数据后进行增广操作，然后传给模型进行训练。

![img](https://miro.medium.com/max/2443/1*00PmRDHTyGWf6j8wKUa3-A.png)

在[2080Ti-PyTorch DataLoader 测试（2020-01-14）](https://confluence.momenta.works/pages/viewpage.action?pageId=62604988)中，可基本确认在数据预处理环节，resize 操作耗时较多，此时 CPU 资源占用也处于很高的情况。

在上述 pipeline 中，训练之前的工作都基于 CPU 进行，当训练数据量级较大，模型计算量较少时，数据的预处理环节就会成为训练过程的瓶颈。

为解决该问题，NVIDIA 发布了 Data Loading Library （下文简称DALI），DALI 利用 GPU 的计算资源，能够对从读取磁盘到准备进行训练以及推理的完整的 pipeline 加速。因此本方案选择使用 DALI 对 pipeline 进行改进，如下图所示：

![img](https://miro.medium.com/max/2448/1*t3O2Rpm-YYfdUW8sUpsPMA.png)


### **1.2 项目目标**

1. 储备 DALI 相关的开发资料和技能，对于典型模型（例如 resnet50）在典型数据集（例如 ImageNet）上的性能数据。
2. 与项目组合作，在真实的业务模型上试用改进后的 pipeline。

### **1.3 总体工期规划**

参考训练平台季度规划，本项目预计起止时间为 3.9 - 4.17，总工期 30 人日。

### **1.4 项目里程碑**

| **周**| **目标**  | **量化考核标准**           | **备注** |
| ----  | -------  | --------------------- | -------- |
| 1     | 完成 20% | 调研现有方案 DALI 使用方案，明确 pipeline 改进的技术细节 |          |
| 2     | 完成 50%  | 提供 resnet50 使用改进后的 pipeline 在 ImageNet 上的性能数据 |          |
| 3 	  | 完成 80% | 在项目组提供的业务模型上应用成功，提供性能数据 |          |
| 4 | 完成 100% | 提供使用文档 | |


## 2.项目范围

1. 在基于 PyTorch 的模型中增加改进后的 pipeline 代码
2. 使用文档

## 3. 项目进度

TODO


## 4.项目风险

暂无