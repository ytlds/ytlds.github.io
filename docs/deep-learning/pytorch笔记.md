# pytorch 学习笔记

## 1. torch 模块
### tensor属性&方法

- 数值类型转换方法

  - tensor.double()， tensor.short()

  - tensor.to(torch.double)

  - 

  - tensor.to(dtype=torch.short)

- tensor.squeeze(dim=None, *, out=None) 未指定 dim 时： 降低维度，将张量维度为1的去掉并返回 指定 dim 时： 在指定维度上降维操作，若对应维度非1，则无操作

- tensor.unsqueeze(input, dim) 扩展维度，在输入的位置插入维度 1

- tensor.storage()  
  查看 tensor 底层存储结构

- tensor.storage_offset()  
  查看tensor在storage中的偏移

- tensor.stride()  
  每个维度+1时storage中的步长

- tensor.clone()  
  复制tensor返回

- tensor.is_contiguous()  
  判断tensor是否连续

- tensor.contiguous()  
  将tensor变为连续

- tensor.to(device=‘cuda:0’ / ’cpu’)  
  移动 tensor 至 GPU:0 / CPU

- tensor.view(*shape), shape:(torch.Size or int...)   
  返回一个数据相同 shape 不同的 tensor  
  只能用于 contiguous tensor

- tensor.permute()  
  改变维度排序，多维转置

### torch.transpose(input, dim0, dim1)  

  tensor.tranpose(dim0,dim1)  
  转置，只能两个维度

### tensor.numpy()  

  tensor转为numpy数组

### torch.from_numpy(npdarray)  

  numpy数组转为tensor

### torch.save()  

  序列化一个对象，存储

### torch.load()  

  加载由 torch.save 序列化的对象

