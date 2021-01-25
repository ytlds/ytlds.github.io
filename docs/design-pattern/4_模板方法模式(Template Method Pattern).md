# 4.模板方法模式(Template Method Pattern)

定义：

\- Define the skeleton of an algorithm in an operation, deferring some steps to subclasses.

\- Template Method lets subclasses redefine certain steps of an algorithm without letting them to change the algorithm's structure.

场景：

优点：

1. 封装不变部分、扩展可变部分
2. 提取公共部分代码、便于维护
3. 行为由父类控制，子类实现
4. 扩展性良好，继承，覆写

缺点：

1. 父类完成了全部定义和逻辑(多由client定义使用场景，即逻辑)，子类仅负责实现。

适用场景：

1. 多个子类公有方法，并且逻辑基本相同
2. 重要、负责的算法，可以设计为模板方法，细节由子类实现
3. 重构时经常使用，相同代码提取至父类通过钩子函数约束行为

代码示例：

/*abstract product class*/

class CAbstractPhoneCase

{

public:

  CAbstractPhoneCase();

  virtual ~CAbstractPhoneCase();

protected:

  virtual void paintColor() = 0;

  virtual void printWord() = 0;

 

public:

  void caseTest()

  {

​    paintColor();

​    printWord();

  }

};

 

 

class CiPhone7Case()

{

public:

  CiPhone7Case();

  virtual ~CiPhone7Case();

protected:

  virtual void paintColor()

  {

​    printf("case color: iPhone7\n");

  }

 

  virtual void printWord()

  {

​    printf("case word: iPhone7\n");

  }

};

 

 

class CiPhone7PlusCase()

{

public:

  CiPhone7PlusCase();

  virtual ~CiPhone7PlusCase();

protected:

  virtual void paintColor()

  {

​    printf("case color: iPhone7 Plus\n");

  }

 

  virtual void printWord()

  {

​    printf("case word: iPhone7 Plus\n");

  }

};

 

\-------------------------------------------------------------

int main()

{

  CAbstractPhoneCase * case0 = new CiPhone7Case();

  CAbstractPhoneCase * case1 = new CiPhone7CasePlus();

 

  case0.caseTest();

  case1.caseTest();

 

  return 0;

}

源码：

<<Template Method Pattern.cpp>>

 

car：engineStart，alarm，run，shake。

if（isAlarm（））alarm；



 

钩子方法：添加判断语句，hook method