@**staticmethod**

将方法转换为静态方法。 

静态方法不会接收隐式的第一个参数。要声明一个静态方法，请使用此语法

**class** **C**:

  **@****staticmethod**

  **def** f(arg1, arg2, ...): ...

 

@staticmethod 这样的形式称为函数的 [decorator](https://docs.python.org/zh-cn/3/glossary.html#term-decorator) -- 详情参阅 [函数定义](https://docs.python.org/zh-cn/3/reference/compound_stmts.html#function)。

静态方法的调用可以在类上进行 (例如 C.f()) 也可以在实例上进行 (例如 C().f())。 

Python中的静态方法与Java或C ++中的静态方法类似。另请参阅 [classmethod()](https://docs.python.org/zh-cn/3/library/functions.html?highlight=staticmethod#classmethod) ，用于创建备用类构造函数的变体。

像所有装饰器一样，也可以像常规函数一样调用 staticmethod ，并对其结果执行某些操作。比如某些情况下需要从类主体引用函数并且您希望避免自动转换为实例方法。对于这些情况，请使用此语法:

**class** **C**:

  builtin_open = staticmethod(open)

 

想了解更多有关静态方法的信息，请参阅 [标准类型层级结构](https://docs.python.org/zh-cn/3/reference/datamodel.html#types) 。

*class* **str**(*object=''*)

*class* **str**(*object=b''*, *encoding='utf-8'*, *errors='strict'*) 

返回一个 [str](https://docs.python.org/zh-cn/3/library/stdtypes.html#str) 版本的 *object* 。有关详细信息，请参阅 [str()](https://docs.python.org/zh-cn/3/library/stdtypes.html#str) 。

str 是内置字符串 [class](https://docs.python.org/zh-cn/3/glossary.html#term-class) 。更多关于字符串的信息查看 [文本序列类型 --- str](https://docs.python.org/zh-cn/3/library/stdtypes.html#textseq)。 