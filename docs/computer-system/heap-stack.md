# 堆和栈的区别

（小端系统，intel）

栈的内存增长方向为**从高到低**，因此可以进行如下测试

```c++
// test.cc

int main(int argc, char* argv[]){
  int i = 0;
  int arr[3] = {0};
  for(; i<=3; i++){
    arr[i] = 0;
    printf("hello world\n");
  }
  return 0;
}
```



```bash
g++ test.cc -fno-stack-protector -o test
```

