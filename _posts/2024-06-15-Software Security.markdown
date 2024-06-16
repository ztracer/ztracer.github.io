---
title: "软件与系统安全复习"
subtitle: "RevIEw"
layout: post
author: "tracer"
hidden: true
header-img: "img/home-bg-o.jpg"
tags:
    - Software Security
    - Reverse
---
# 软件与系统安全课程复习

>**实验班分值:**
>*期末考试: 50%*
>上机作业(2次): 15%×2
>平时书面作业(1次): 10%
>线上学习成绩: 10%

>**题型:**
>客观题（10分）+主观题（90分）

>**一些参考链接:**
>[学长的一些复习笔记](https://blog.csdn.net/weixin_47102975/category_11969200.html)
>[学长的期末复习](https://blog.csdn.net/weixin_47102975/article/details/126817255)

※ 文章标记：**重点** *次重点* ※

# 复习重点

## 1.概述

## 2.技术基础

*威胁模型*, 可信计算基(Trusted computing base), *安全策略与策略执行*, 通用的安全设计原则, 攻击面

x86概念与**内存模型**, 寄存器与数据类型,**指令集**(栈操作和函数调用相关)与调用惯例, 静态反汇编, *ELF文件格式*

#### 2.1 详细知识

##### 2.1.1 信任
- 主体被起到表现出某种行为的程度
- 可信计算基TCB
- 可信根TPM/TCM
    - P: Platform
        - 可信平台模块,植于计算机内部为计算机提供可信根的芯片。该芯片的规格由可信计算组（Trusted Computing Group）来制定
    - C: Cryptography
        - 可信密码模块； 是可信计算平台的硬件模块，为可信计算平台提供密码运算功能，具有受保护的存储空间，是我国国内研究，与TPM对应

##### 2.1.2 威胁模型 
- 什么可信什么不可信
    攻击者的资源动机能力
    攻击造成的影响
    - 例子
        - 接收客户端请求的web服务器
            什么可信: web服务器
            什么不可信: 客户端
               -  客户端可能发送恶意输入
               -  客户端可能发起拒绝服务攻击
               -  客户端可能接管web服务器: 偷取敏感信息; 安装恶意软件
            实际上是一个“软件可信但输入不可信”的场景
        - 保护计算机系统免受恶意代码和恶意输入的威胁
            系统本身是可信的
        - 保护软件免受恶意篡改
            代码本身是可信的
            但代码运行在一个不可信的系统上
- 漏洞 
    - 可以被对缺陷具有利用能力的攻击者访问并
        利用的缺陷
    - 要素(对应上CIA三要素)
        - 缺陷 (flaw) - 机密
        - 攻击者能访问缺陷 - 完整(权限管理)
        - 攻击者有利用缺陷的能力 - 可用

##### 2.1.3 安全策略和策略执行
安全策略
- 谁被允许做什么
- CIA模型
    - 机密性（Confidentiality）
        - 信息仅能被授权者知道
            例: Bob买了1000股微软股票且希望此信息保密
    - 完整性（Integrity）
        - 系统中存储的信息是正确(未被篡改的), 攻击者无法修改被保护
            的信息
            例: Bill Gates卖了100万股微软股票; 此信息是公开的, 但没有
            人能将100万改为1000万
    - 可用性（Availability）
        - 当需要信息或服务时, 信息或服务可用, 攻击者无法阻碍计算过程
            例: Bob想从经纪人那里买1000股微软股票, 但经纪人不在
            (生病了)

##### 2.1.4 安全设计原则
- 开放设计(Open Design)原则
    - 安全不能依赖于对设计和实现的保密(通过“晦涩”得到的安全)
- 最小权限(Least Privilege)原则
    - 主体应仅被授予完成其任务所必需的权限
    - 权限在需要时添加, 在使用后收回
- 缺省失败安全(Fail-Safe Defaults)原则
    - 指定“合法”字符, 并丢弃所有非合法的字符; 而不是指定
        “非法”字符且仅丢弃它们(“白名单”而非“黑名单”)
- 安全机制经济性(Economy of Mechanism)原则
    - 每个工具完成单一任务
    - 定义完备的接口
- 完全仲裁(Complete Mediation)原则
    - 检查并授权对于对象的每一次访问
- 特权分离(Separation of Privilege)原则
    - 责任分裂
        - 填支票数额的人和签支票的人应为不同的人
- 最小通用机制(Least Common Mechanism)原则
    - 访问不同资源的机制应不是共享的
- 心理可接受性(Psychological Acceptability)原则
    - 安全机制不应增加访问资源的难度

##### 2.1.5 攻击面
- 攻击者的可访问性质:能够被攻击者访问到的入口点
- 如何识别
    - 识别出可能被攻击者控制的系统资源子集
        - 谁是程序攻击者?
            哪些资源可能被他们控制?
    - 识别出这些系统资源在何时会被程序使用(即程序入口点)
        - 程序访问外部源(如文件, 网络socket等)所使用的程序语句
            是入口点
    - ![[./reverse_pic/攻击面.png]]

##### 2.1.6软件逆向基础
- X86小端序
    - X86
        - IA-32
            - 内存模型：段地址
                逻辑地址=段选择器+偏移量
        - x86-64
- 寄存器与数据类型
    - 通用寄存器8个32位的
        - EAX:（操作数和结果数据的）累加器
            EBX:（在DataS段中数据的指针）基址寄存器
            ECX:（字符串和循环操作的）计数器
            EDX:（I/O指针）数据寄存器
            EDI: 变址寄存器, 字符串/内存操作的目的地址
            ESI: 变址寄存器, 字符串/内存操作的源地址
            EBP:（StackSeg段中的）栈内数据指针, 栈帧的基地址, 用于为函数调用创建栈帧
            ESP:（StackSeg段中的）栈指针, 栈区域的栈顶地址
- 二进制文件格式

## 3.软件漏洞利用与防护

*内存破坏漏洞*: **栈溢出**, *整数溢出*, **堆溢出**, Use after free,Double free, Type Confusion, 格式化字符串, 防御性编程

**高级防御与攻击: stack canaries, DEP, 代码注入, 代码重用,return-to-libc, ASLR, ROP**

## 4.模糊测试

*软件测试基础*(**指标定义**), **模糊测试原理**、**分类及指标**, AFL简介, 渗透测试概述

## 5.软件自我保护

**MATE攻击模型**, **代码混淆**, *软件防篡改*, *软件水印*, 软件胎记

#### 5.1 MATE(*M*an-*A*t-*T*he-*E*nd)攻击模型

- 攻击者： 位于终端，对终端计算资源有高控制权限
- 攻击对象：安装在受控终端上的软件程序
- 攻击(表象)目的： 获悉、篡改软件的内部逻辑

![MATE和防范措施](img/inPost/SoftwareSecurity/MATE.png)

#### 5.2 代码混淆Obfuscation
通过重构增加代码逆向分析难度的技术

混淆是一种程序变换



## 6.Web安全

**SQL注入**, XSS, CSRF

#### 6.1 SQL注入

SQL语言的核心结构：
- Sth. to do
- @ target
- Sth. to do (optional)
- under condition (optional)

##### 6.1.1 一些例子-1：

输入username：’; drop TABLE Accounts;/*

SELECT * FROM Accounts
WHERE Username = ’ ’

SELECT * FROM Accounts
WHERE Username = ’’; drop TABLES Accounts;
/*’AND  Password = ’geheim’;

##### 6.1.2 一些例子-2：

输入Username：’OR 1=1;/*

SELECT * FROM Accounts
WHERE Username = ’’
AND Password = ’’;

SELECT * FROM Accounts
WHERE Username = ’’ OR 1=1;（被终结在这）
/*’AND Password = ’geheim’;（这里注释掉了）

##### ※6.1.3 SQL注入原理：

SQL语言中的一些特殊符号
- 分号***;***，意味着指令结束，有可能开始下一个指令
- 单引号**’**，用于字符串常量，系统在输入命令中顺序匹配 ‘’
- **#或--**，意味着注释， 出现在其后的内容会被系统忽略掉

上述符号的存在，使得经过设计的SQL语句可以**篡改设计者事先定义的查询语义**

##### 6.1.4 数字型漏洞

http://xxx.xxx.xxx/abcd.php?id=XX
若XX为数字类型，例如页码、 ID等，存在注入时则为数字类型的注入

①给参数赋值为or 1=1，页面正常
http://xxx.xxx.xxx/abcd.php?id=XX or 1=1

②接着给参数赋值为and 1=2，页面报错
http://xxx.xxx.xxx/abcd.php?id=XX and 1=2

select * from <表名> where id=XX or 1=1
select * from <表名> where id=XX and 1=2

##### 6.1.5 字符型漏洞

http://xxx.xxx.xxx/abcd.php?id=XX
若XX为字符串，注入测试需使用单引号来闭合

①给参数赋值为or 1=1，页面正常
http://xxx.xxx.xxx/abcd.php?id=XX ’or ‘1’=‘1

②接着给参数赋值为and 1=2，页面报错
http://xxx.xxx.xxx/abcd.php?id=XX ’and ‘1’=‘2

select * from <表名> where id=‘XX’ or ‘1’=‘1’
select * from <表名> where id=‘XX’ and ‘1’=‘2

##### 6.1.6 Hack方法

SQL注入过程典型示例
- 寻找可能存在SQL注入漏洞的链接
- 测试该网站是否有SQL注入漏洞
- 猜管理员帐号表
- 猜测管理员表中的字段
- 猜测用户名和密码的长度
- 猜测用户名
- 猜测密码



#### 6.2 XSS - Cross Site Scripting
[蛋老师的讲解](https://www.bilibili.com/video/BV1rg411v7B8/)

>也称为HTML注入


#### 6.3 CSRF - Cross Site Scripting Forgery

[蛋老师跨站请求伪造](https://www.bilibili.com/video/BV1rg411v7B8/)

## 7.引用监控器

## 8.恶意代码的机理及其防护

**木马、病毒、蠕虫概念及区别**



**病毒的隐藏技术, 木马的隐藏技术**
