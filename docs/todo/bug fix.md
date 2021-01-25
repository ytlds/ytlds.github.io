AttributeError: module 'google.protobuf.descriptor' has no attribute '_internal_create_key'

pip install --upgrade protobuf



**1. os.getcwd()**
[https://docs.python.org/2/library/os.html#os.getcwd](https://link.jianshu.com/?t=https%3A%2F%2Fdocs.python.org%2F2%2Flibrary%2Fos.html%23os.getcwd)
获取文件当前工作目录路径（绝对路径）

**2. sys.path[0]**
获取文件当前工作目录路径（绝对路径）
sys.argv[0]|获得模块所在的路径（由系统决定是否是全名）
若显示调用python指令，如python demo.py，会得到绝对路径;
若直接执行脚本，如./demo.py，会得到相对路径。

**3. __file__**
获得文件所在的路径（由系统决定是否是全名）
若显示执行Python，会得到绝对路径;
若按相对路径来直接执行脚本./pyws/path_demo.py，会得到相对路径。

**4. os.path.abspath(__file__)**
获得文件所在的路径（绝对路径），例如：

```python
print os.path.abspath("/usr/bin/python")



'/usr/bin/python'
```

**5. os.path.realpath(__file__)**
[https://docs.python.org/2/library/os.path.html#os.path.realpat](https://link.jianshu.com/?t=https%3A%2F%2Fdocs.python.org%2F2%2Flibrary%2Fos.path.html%23os.path.realpath)
获得文件所在的路径（绝对路径，而非软链接所在的路径）例如：

```python
print os.path.realpath("/usr/bin/python")



'/usr/bin/python2.7'
```

**6. os.path.split(os.path.realpath(__file__))**
[https://docs.python.org/2/library/os.path.html#os.path.split](https://link.jianshu.com/?t=https%3A%2F%2Fdocs.python.org%2F2%2Flibrary%2Fos.path.html%23os.path.split)
生成二元元组（文件目录，文件名）。