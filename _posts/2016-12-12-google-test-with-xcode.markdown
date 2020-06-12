---
layout: post
title: "XCode下搭建GoogleTest框架测试环境"
data: 2016-12-12 21:25:00 +0800
---

一直想系统学习一下C++的东西，几本C++的书也落了很厚的灰。之前对C++的认识也停留在应用开发的层面，这次决心深入的去系统学习一下。Mac上首选就是XCode，我非常推崇测试驱动开发，所以决定用GoogleTest来做测试框架，来验证学习过程中的代码。

搭好GoogleTest之后发现还是有些地方需要记录一下，主要参考GoogleTest项目下的Doc目录。

环境 / 版本：MacOS，XCode version 8.1，googletest version 1.70


- Clone代码
```
	git clone https://github.com/google/googletest.git
```
- 打开`gtest.xcodeproj`编译`gtest.framework`
- 新建一个Command Line Tool的XCode项目,将`gtest.framework`加入到项目中
- 复制googletest下的include到测试项目下并加到头文件搜索路径。
- 添加环境变量`DYLD_FRAMEWORK_PATH`,值为`gtest.framework`所在的目录

```
Produce -> Scheme -> Edit Scheme -> Arguments -> Environment Variables
```
- 修改测试项目设置在`Build Settings` -> `C++ Standard Library`的值为libstdc++(GUN C++ Standard Library)
- 搭建完成


测试代码

```
//
//  main.cpp
//  CPP_Practice
//
//  Created by Sean on 12/12/2016.
//  Copyright © 2016 Sean. All rights reserved.
//

#include "gtest.h"

int Factorial(int n)
{
    if (1 == n)
    {
        return 1;
    }
    
    return n*Factorial(n-1);
}

// 测试用例
TEST(FactorialTest, HandlesPositiveInput) {
    EXPECT_EQ(1, Factorial(1));
    EXPECT_EQ(2, Factorial(2));
    EXPECT_EQ(6, Factorial(3));
    EXPECT_EQ(40320, Factorial(8));
}

int main(int argc, char** argv)
{
    testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```
