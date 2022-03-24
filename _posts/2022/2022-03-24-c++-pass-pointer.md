---
title: 一个指针的指针引起的bug
categories:
  - C++
tags:
  - C++
classes: wide
---
### 思考一个问题

我们需要在一个函数里修改函数外面的一个变量

那么很简单，假设我们有一个变量int c，对应的函数就是
```cpp
#include <iostream>
using namespace std;

// 参数类型是需要修改的变量的指针类型
void changeValue(int* p){
  *p = 100;
}

int main()
{
    int c = 1;
    changeValue(&c);
    cout << c << endl;
}
```
[运行结果: 100](https://coliru.stacked-crooked.com/a/29cca0f6c7f07023)

那么我们需要在一个函数里修改函数外面的一个指针变量，就是让这个指针指向另一个变量。

那么很简单，假设我们有一个变量int* cp，我们让它指向另一个int d，对应的函数就是
```cpp
#include <iostream>
using namespace std;

// 参数类型是需要修改的变量的指针类型
void changeValue2(int** pp){
  // 在heap中new一个int
  int* dp = new int(100);
  // 让pp指向int 100
  pp = &dp;
}

int main()
{
    int c = 1;
    // cp指针指向c
    int* cp = &c;
    // 让cp指针指向d
    changeValue2(&cp);
    cout << "c,cp is " << c << "," << *cp << endl;
}
```
[运行结果: 1,100](https://coliru.stacked-crooked.com/a/cf92d252b1467c09)

可以看到指针指向的值是100.

### 现在正式开始思考
上面的代码和结果对吗？
思考一分钟在往下面看。
![](https://raw.githubusercontent.com/vonway/picgo/main/202203242109246.jpg)

当然是有问题啦。运行的结果应该是“1,1”，cp并没有指向100。

```cpp
  // 让pp指向int 100
  pp = &dp;
  // 这里pp是指向了100，但是cp并没有指向100
```
原因是void changeValue2(int** pp){...}这个函数，参数是传值的。
`pp = &dp;`这个语句调用之前是这样的：
- cp --> c
- pp --> cp -->c
- dp --> 100

调用之后
- cp --> c
- pp --> dp --> 100

改变了pp指向的变量，不是cp指向的变量。

**所以这里如果参数类型是int\*，那么传入的值int\*，不论怎么改变，外面的cp的值永远不会被改变。参数类型必须是int\*\*。**

正确的语句应该是
`*pp = dp`

这个语句调用之前是这样的：
- cp --> c
- pp --> cp -->c
- dp --> 100

调用之后
- cp --> 100
- pp --> dp --> 100

此时cp和dp的值相同。
*pp就是cp，`*pp = dp`把dp的值赋值给cp，就是让cp指向dp指向的值。

看下正确的结果：
[运行结果: 1,100](https://coliru.stacked-crooked.com/a/0425563c0afb4e07)

就这样
