---
title: "Cryptography"
subtitle: "密码学复习"
layout: post
author: "tracer"
header-img: "img/inPost/crypto/cryptography-bg.jpg"
tags:
    - Cryptography
    - Protocol Security
    - Web Security
---

***计算机安全的三个关键因素（CIA）***
机密性(Confidentiality)、
完整性(Intergrity)、
可用性(Availability)

# 现代密码学复习

<img src='https://github.com/ztracer/ztracer.github.io/blob/master/img/inPost/EOC/155.gif?raw=true' />

## 1. 对称密钥加密

### 1.1 流密码 【Stream Ciphers】

#### 1.1.1 流密码的安全性在于?
- PRG：安全性关注且 **只** 关注 PRG

#### 1.1.2 OTP：one time pad 【真正的安全.jpg】
- attack
  - 2TP
  - NO integrity-malleable

#### 1.1.3 伪随机数生成器PRG【 PseudoRandom Generators】
- 使得OTP更加实用
- G:{0,1}^s -> {0,1}^n 其中n >> s
- weak PRG
    - 线性同余方程
- Security定义：key与随机生成的r无法区分“indistinguishable”
    - “indistinguishable”定义：统计测试要有*可忽略*的优势
      - 优势Advantage的定义：
          统计测试A输出1的可能性概率
          相对于
          PRG输出1的可能性概率的差

#### 1.1.4 流密码的一些实际的例子
- [RC4](https://en.wikipedia.org/wiki/RC4)
    - 面向软件设计的
- [CSS](https://en.wikipedia.org/wiki/Content_Scramble_System) 【content scrambling system】
    - DVD中的一种加密算法【所以是面向硬件设计的一种加密算法】
    - 现状：badly broken
    - 用的PRG是LSFR
        - LSFR专题: [知乎](https://zhuanlan.zhihu.com/p/366067972)

### 1.2 分组密码 【Block Ciphers】

#### 1.2.1 基本模型
key k  -----扩展成------> 多次加密所需的k<sub>n</sub>
    m 经过 n 轮加密

#### 1.2.2 PRP & PRF
- PR**F**unction : K*X -> Y
    - Security: 与F[x,y]不可区分
- PR**P**ermutation : K*X -> X
    - 一一映射的
    - Security: 与E[x,x']不可区分
        - 注意E为一一映射函数
        - 分组密码的安全性保障
- PRF ---构建---> PRGenerator
    - Security: PRG生成的K能使得PRF与真随机不可区分

#### 1.2.3 standard 分组密码标准
**DES【data encryption standard】**：第一个并且是最重要的现代分组密码算法（用 **56位密钥**来加密 **64位数据**的方法）
- 构建*feistel network*，总体架构如下: 
    ![](/img/inPost/crypto/DES总体架构.png) 
    1. 黄色部分的：初始置换和初始逆置换![初始置换与初始逆置换是互逆的](/img/inPost/crypto/DES初始置换和初始逆置换.png)
    2. DES-16轮中的每轮迭代加密步骤：![](/img/inPost/crypto/DES每轮迭代加密.png)
    3. 其中的***f***函数是什么？![](/img/inPost/crypto/DES-f函数.png)![](/img/inPost/crypto/DES流程图.png)
       1. Expansion-Ebox是:
        ![](/img/inPost/crypto/DES-E盒扩展.png)
        将32bit扩展到48bit
       2. 48bitEbox输出与48bit密钥逐一异或
       3. Substitution-Sbox【混乱】：48bit分成8块*6bit(Sbox是核心!是非线性变换的关键)![](/img/inPost/crypto/DES-S盒.png)Sbox提供了密码算法所必须的混乱作用，将48bit变回32bit
       4. Permutation-Pbox【扩散】：32bit进行很常见的"置换"后同样输出32bit数据。
    4. 密钥扩展（原始密钥64bit -> 扩展成最终每轮的48bit）
       1. **PC-1置换选择**：置换 + 选择64bit其中的56bit【其中有8位是奇偶效验位（8,16,24,32,40,48,56,64）不做置换处理】
       2. PC-1置换后的**前28bit放在C寄存器**中，**后28bit放在D寄存器中**。
       3. **LS移位**：两个28位密钥的寄存器循环左移（第1，2，9，16轮循环左移1位，其他12轮循环左移2位 = 4\*1+12\*2 = 28 —— 16轮后刚刚好移位数等于28）
       4. **PC-2置换选择**：置换 + 选择56bit其中的48bit
- attack
    1. 雪崩效应：在高质量的分组密码中，无论密钥或明文的任何细微变化都应当引起密文的剧烈改变。如果存在“弱密钥” —— 即给定初始密钥k，各轮的子密钥都相同（也就是 k1=k2= … =k16），就称给定密钥 k 为弱密钥Weak key，这种情况下DES在选择明文攻击下的搜索量减半。
    2. 穷举攻击：1999年时，已经缩短到22个小时
- [一个比较好的关于DES的讲解](https://blog.csdn.net/qq_44131896/article/details/117573452)
- 3DES
   - n=64bits
   - k=168bits
   - but：软件实现速度慢，分组短 -> 来个AES（硬件实现快）

**AES【Advanced Encryption Standard】**：
- 非feistel网络
- n=128bits；k=128\192\256bits

构建：可逆的Subs-Perm Network——in GF(2<sup>8</sup>)

除了最后一圈，其余各圈都包含：
- 字节代替变换 （唯一的非线性变换，非线性层）
  - S盒变换就是由一个byte的数据（8位二进制）转换为另一个byte的数据（8位二进制、2位16进制）
  - 查表/计算
- 行移位变换 （线性混合层）
  - ![](/img/inPost/crypto/行移位.png)
  - State 的第 1 行保持不变
  - State 的第 2 行循环左移 1 个字节
  - State 的第 3 行循环左移 2 个字节
  - State 的第 4 行循环左移 3 个字节
- 列混合 （线性混合层）
  - ![](/img/inPost/crypto/列混合.png)
  - ![](/img/inPost/crypto/列混合2.png)
- ⨁ K<sub>i</sub> （与密钥进行异或运算）

**※重要※**：最后一轮没列混合

#### 1.2.4 分组密码的工作模式 Using Block Ciphers
- **ECB**【electronic code book】
    - 泄露轮廓信息（相同明文加密产生相同密文）
    ![](/img/inPost/crypto/ECB-beenSeen.png)
    由于在此图像中，许多像素具有相同的颜色，因此这些相同的块加密为相同的密文块。
    - 不能加密大于 > 1个分组的信息
- OTK【one time key】：security for *many time key*
  - 目标 : 2倍相同的明文加密出来的**并非**2倍密文
  - solution
      - 1.随机加密（加密到一组 ---》 也就是一对多映射 ）这个"多"展示了随机性
      - 2.**Nonce** based encryption
          - 对于(k,n)仅能加密一个密文两个解决方案
             1.对每个明文选择新的随机密钥
             2.选择新的nonce(**unique**)
          - nonce的选择方法
             1.random
             2.counter(数据包的计数器)
- **CBC**【cipher block chaining】
  - 构建CBC：
    ![构建CBC](https://upload.wikimedia.org/wikipedia/commons/8/80/CBC_encryption.svg)
      1. random *IV*
          - IV<sub>1</sub>=E(k,m[0]⊕IV)
          - 密文ciphertext为：
             IV \|\| c[0] \|\| ···· \|\| c[n]
      2. nonce-based
          - IV = E(k<sub>1</sub>,nonce)

  - 明文短了怎么办 -> 补齐到加密格的大小
      - Attack: 存在密文偷窃问题
- **CTR**【counter mode】
  - 构建CTR
    ![](https://bernardoamc.com/4d9317e0640a84dba1dc08ad8adc3d8a/ctr_encryption.svg)
      - 1.rand ctr-mode
          - c[L]=m[L]⊕F(k,IV+L)
      - 2.nonce ctr-mode
- 

## 2. 消息完整性【Message Integrity】

### 2.1 MAC基本定义和应用

**Message Authentication Codes**

- 是一对 生成+验证 的算法
   tag <-- Sign(k,m)
   Vertify(k,m,tag) = 'yes'
- 安全性：尽管有能力生成标签,也无法生成新的独特的(m,t)
- ECBC-MAC
    - PRP:K\*X ---> X演变的
       PRF<sub>ECBC</sub> : K<sup>2</sup> \* X<sup><=L</sup> ---> X
    - F(K,m[0])⊕m[1])成为新的F输入
       F<sub>LAST</sub>输入到F(k<sub>1</sub>,F<sub>LAST</sub>)生成tag
    - padding问题
- NMAC
    - PRF:K\*X ---> K演变的
       PRF<sub>NMAC</sub> : K<sup>2</sup> \* X<sup><=L</sup> ---> K
    - k<sub>n</sub> = F(K<sub>n-1</sub>,m[n])
       输出k<sub>LAST</sub>补齐成分组大小
       tags = F(k1,m[pad])
        - m[pad] = k<sub>LAST</sub> \|\| fpad 为补齐
- PMAC
    - (⊕)$\sum_{i=0}^{n-1}F(k_i,m[i]⊕P(k,i))$
       tag=F(k1,(⊕))

### 2.2 抗碰撞【Collision resistance】
- Attack
    - birthday paradox
- 构建抗碰撞的HASH--Merkle-Damgard Paradigm
- HMAC(HASH+MAC)

## 3. 认证加密 【Authenticated Encryption】

单纯的加解密算法（解决机密性问题）无法抵抗下述攻击：
• 内容修改
• 顺序修改
• 计时修改
• 发送方否认
• 接收方否认
这些属于“数据完整性”范畴

SO -> 我们需要加密与认证分离

![](/img/inPost/crypto/加密与认证分离.png)


### 3.1 认证加密用来解决什么问题
- > 不被窃听+不被修改
- (E,D)一对算法的输出需要满足
   E:K\*M\*N -> C
   D:K\*C\*N -> M U {不存在}
    - 能解决问题1 : 是认证的加密,重放攻击是无效的
    - 能解决问题2 : 同时拥有CPA和CCA(针对oracle的安全)

### 3.2 Construction
- 1.MAC-then-Encrypt
- 2.Encrypt-then-MAC
- 3.Encrypt-and-MAC x

### 3.3 安全的散列函数
为文件、消息或其他数据块产生“指纹”，浓缩任意长的消息M到一个固定长度的取值h=H(M)，对于Hash函数h=H(x)，称x是h的原像，称即把数据块x作为输入，称使用Hash函数H得到h。

- 如果满足**x≠y**且**H(x)=H(y)****，称则称出现碰撞：
  - 抗原像攻击（单向性）：*对任意给定的散列码h，找到满足H(x)=h的x* 在计算上不可行，单向性。
  - 抗第二原像攻击（抗弱碰撞攻击）：*对任何给定分组x，找到满足y≠x且H(x)=H(y)的y* 在计算上不可行，抗弱碰撞性。
  - 抗碰撞攻击（抗强碰撞攻击）：*找到任何满足H(x)=H(y)的偶对(x, y)* 在计算上不可行，抗强碰撞性。

### 3.4 SHA系列算法

SHA系列的算法-输入多长、输出多长、密钥长度多少？
SHA-1:密钥、结果160-块512
SHA-256:密钥、结果256-块512

## 4. 密钥与认证【与谁-干什么；who-what问题】*写得有点乱这个部分*

### 4.0 有哪些密钥【网络与协议安全中重要的一个部分】
- 【**密钥是有生存周期的**】
- 基本密钥/会话密钥
- 公钥加密的：公私钥
- 用来签名的：公私钥

### 4.1 认证过程

小对比：

授权 (authentication)：授权关注的是允许这个进程做什么样的事情

认证 (Authentication)：进程通过认证过程来验证它的通信对方是否是它所**期望的实体**而不是假冒者【关于who的问题】

#### 4.1.1 需要建立共享密钥:

##### ✔️方法1【**非对称加密**的D-H】: (※这个方法依赖于离散对数问题的难度)
1. 初始化：A选取一个生成元g和一个大素数p，发送给B
2. A随机选取0<x<p-1，计算X = g<sup>x</sup> mod p后，将X传给B
3. B随机选取0<y<p-1，计算Y = g<sup>y</sup> mod p后，将Y传给A
4. A计算Y<sup>x</sup> mod p得到会话密钥K1
5. B计算X<sup>y</sup> mod p得到会话密钥K2
6. 由下可知 K1 == K2，此时双方便可利用会话密钥进行加密通信了

⚠️但是D-H存在*中间人攻击*
![](/img/inPost/crypto/middle_attack.png)
- **预防措施与新的问题**：
    1. 可以用自己另外一对公私钥中的私钥签名【但是公钥的分发存在问题】
        - 公钥如果直接发送也存在*中间人攻击*，确保不了与网站之间生成了对话密钥
        - 这里需要第三方CA----Certificate Authority
        ![技术蛋老师](/img/inPost/crypto/CA.png)[Pic From BiliBili 技术蛋老师](https://www.bilibili.com/video/BV1mj421d7VE)
        核心在于CA的数字签名
        - BUT又双叒叕有一个问题：CA网站发给你的时候可能也会有中间人攻击——>这就套娃了——>所以需要证书链&一个根证书【OS和浏览器内预先安装】
        - 结合一个实际例子来看看
        ![](/img/inPost/crypto/eg1_compare.png)
        - 中心化依赖着的是对于“中心”的信任，有没有“去中心化的监督呢”----新机制[CT](https://www.bilibili.com/video/BV1uY4y1D7Ng)-Certificacte Transparency【类似于区块链Merkle tree】
    2. 使用公钥基础设施（PKI）：通过使用证书颁发机构（CA）颁发的公钥证书来验证通信双方的身份。在Diffie-Hellman交换之前，双方首先交换并验证对方的证书。这可以确保交换的密钥不会被第三方篡改。
        
##### ✔️方法2【需要第三方KDC来分发】: Key Distribution Center
- 但是⚠️存在*重放攻击*
- 重放 Alice 的初始请求：攻击者可能截获 Alice 发给 KDC 的加密请求（该请求包含希望与 Bob 使用的会话密钥 KS），然后重新发送这个请求。如果系统没有适当的防护措施，如时间戳或一次性标记，KDC 可能会误认为是 Alice 再次请求与 Bob 通信，从而导致未授权的会话密钥复用。
- 重放 KDC 对 Bob 的响应：攻击者也可能截获从 KDC 发往 Bob 的加密消息，这个消息包含会话密钥 KS 和 Alice 的身份。攻击者随后可以重新发送这个消息给 Bob，使 Bob 相信是收到了来自 Alice 通过 KDC 的新的会话请求。这可能导致 Bob 在不知情的情况下使用旧的或已泄露的会话密钥。
- **预防措施**：
    - *时间戳*：包含在每个请求和响应中的时间戳可以帮助接收者验证消息的时效性，确保消息是在一个预期的时间窗口内发送的。
    - *序列号*：为每个消息分配唯一的序列号可以防止消息被重新发送。
    - *短期会话密钥*：限制会话密钥的有效期可以减少因重放攻击而造成的风险。

#### 4.1.2 基于共享密钥的认证: 【有了共享密钥之后就是对称加密了】
- 一方发送一个随机数给另一方，然后由另一方以特殊的方式对其进行转换并返回结果。这种协议称为"挑战-响应"协议。
![](/img/inPost/crypto/challen.png)
- 容易受到*反射攻击*【用“共享的密钥”这该特性来“借刀杀人”】【关键在于***返回对方发过来的随机数***】
![](/img/inPost/crypto/reflection-attack.png)
- 解决方法：使用HMAC
    - A --------------------R<sub>A</sub>------------------> B
    - A <---R<sub>B</sub>,HMAC[R<sub>A</sub>,R<sub>B</sub>,A,B,K<sub>AB</sub>]-- B
    - A ---------HMAC[R<sub>A</sub>,R<sub>B</sub>,K<sub>AB</sub>]-------> B 【A告诉B我能解出来】
两个 HMAC 包含发送方选择的值，K<sub>AB</sub>攻击者未知，攻击者无法修改选择的值。

### 4.2 现有的解决方案1：[Kerberos](https://cloud.tencent.com/developer/article/1496451) 

>【基于可信第三方的认证，分布式环境下的认证服务】

- 一个认证服务器（Authentication Server，简称 AS）：验证Client端的身份（确定你是身份证上的本人），验证通过就会给一张票证授予票证（Ticket Granting Ticket，简称 TGT）给 Client。
- 一个票据授权服务器（Ticket Granting Server，简称 TGS）：通过 TGT（AS 发送给 Client 的票）获取访问 Server 端的票（Server Ticket，简称 ST）。ST（Service Ticket）也有资料称为 TGS Ticket。

![](/img/inPost/crypto/Kerberos.png)


### 4.3 Merkle Puzzles
- 密钥交换双方O(n),攻击者O(n<sup>2</sup>)

### 4.4 [X.509证书结构](https://cloud.tencent.com/developer/article/1883540)
![](https://ask.qcloudimg.com/http-save/yehe-8818887/772e432e9f0dbbd5dbc585118e60b5ec.png)

## 5.公钥加密

### 5.1 数论的基本知识
tips:补充

### 5.2 应用场景
- 非“互动”应用
- 会话建立【网络】

### 5.3 公钥的安全定义
- 抗窃听
    - 攻击者产生两个信息m0和m1，长度相同，然后攻击者用公交加密得到c0或c1，他不能区分他获得的是m0的加密还是m1的加密。
    - （不用考虑多次加密的情况，因为本身就能多次加密）
- 对抗主动攻击
    - 选择密文安全

### 5.4 基于陷门置换的公钥加密

[手算RSA](https://www.cnblogs.com/moshk/p/13162447.html)

- 陷门函数TDF
    - 定义
        - *加密过程* E(pk,m):
           x <-R- X **[随机选取随机元素x]**
           y <---- F(pk,x) **[陷门]**
           k <---- H(x) **[用来对称加密的对称密钥]**
           c <---- E(k,m)
           输出(y,c)
        - *解密过程* E(sk,(y,c)):
           x <---- F<sup>-1</sup>(sk,y) **[陷门逆变换]**
           k <---- H(x) **[用解出来的x]**
           m<---- D(k,c)
           输出m
    - 错误的使用方法
        - 对明文m直接使用陷门函数（泄露轮廓信息）
- RSA陷门
    - TESTBOOK RSA【是一种[错误的](https://www.packetmania.net/2020/12/01/RSA-attack-defense/)使用方法】
        - p/q ≈ 1024bits
           N=pq
           ed=1(mod Φ(N)) e->加密指数 d->解密指数
           输出 pk = (N,e) sk = (N,d)
    - 基于RSA陷门的ISO标准
        - 用RSA(x)生成y
        - k<-H(x)作为中间密钥 
        - 输出(y,E(k,m))
- PKCS1
    - 预处理
        - 02\|\|random pad(~~FF~~)\|\|FF\|\|msg
        - ATTACK
- [修复方法](https://blog.csdn.net/weixin_54634208/article/details/131608780)

### 5.5 基于Diffie-Hellman的公钥加密


# 网络与协议安全部分复习

## 1.网络与协议安全期末复习大纲

### 1.1 考试结构与分值分布
- **选择题**：10分
- **填空题**：20分
- **简答题**：40分【有密码部分的计算题】
- **综合题**：30分=15*2【协议运用】

### 1.2 网络安全基础【第一章：绪论】
- **核心概念**：机密性（访问控制和加密技术）、完整性（MAC、数字签名和哈希函数）、可用性（可靠、及时访问）

- **被动&主动攻击**：
  - *主动*：主动攻击是指在未经授权的情况下，攻击者对目标系统进行更改或破坏的行为。这种攻击通常涉及篡改数据、注入恶意代码、拒绝服务攻击等手段。主动攻击的目的是破坏信息的完整性和可用性，或干扰系统的正常运行。
  - *被动*：被动攻击则是一种较为隐蔽的威胁。在被动攻击中，攻击者主要关注信息的窃取和监听，而不是直接修改或破坏系统资源。被动攻击通常包括窃听、监视和流量分析等手段，以收集敏感信息为目的。

- **OSI安全框架-7层**：网络安全设计的基础框架
- **TCP/IP-5层模型**：网络协议的基础划分

| 层级 | OSI七层模型 | TCP/IP五层模型 | 常用协议                                  |
|------|------------|----------------|-------------------------------------------|
| 7    | 应用层        | 应用层     | HTTPS、HTTP、Telnet、FTP、TFTP、DNS、SMTP |
| 6    | 表示层        | 应用层            | LPP、NBSSP |
| 5    | 会话层        | 应用层            | SSL、TLS、DAP、LDAP |
| 4    | 传输层        | 传输层            | TCP、UDP |
| 3    | 网络层        | 网络层            | IP、ICMP、RIP、IGMP、OSPF |
| 2    | 数据链路层    | 数据链路层         | 以太网、令牌环、PPP、PPTP、L2TP、ARP、ATMP   |
| 1    | 物理层        | 物理层            | 物理线路、光纤、无线电  |

### 1.3 密码学基础
- **古典密码与对称密码**：包括置换和混淆
- **公钥密码体系**：基于困难数学问题，如RSA（常见的数学问题掌握）
- **密钥管理**：保密性依赖于密钥的机密性（柯克霍夫曼原则——如下），而不依赖于算法的保密性
    1. 即使非数学上不可破解，系统也应在实质（实用）程度上无法破解。
    2. 系统内不应含任何机密物，即使落入敌人手中也不会造成困扰。
    3. 密匙必须易于沟通和记忆，而不须写下；且双方可以容易的改变密匙。
    4. 系统应可以用于电讯。
    5. 系统应可以携带，不应需要两个人或以上才能使用（应只要一个人就能使用）。
    6. 系统应容易使用，不致让用户的脑力过分操劳，也无需记得长串的规则。

### 1.10 网络攻击与防护
- **恶意软件**：依赖于宿主的和独立于宿主 or 不进行复制的和进行复制
- **入侵检测系统**：基于规则和异常的检测技术
- **蜜罐技术**：诱捕攻击者


## Web安全、Internet 安全途径

Web安全威胁：完整性、机密性、拒绝服务、认证

![TCPIP簇中的各类协议.png](/img/inPost/crypto/TCPIP簇中的各类协议.png)

### 1.网络接口层 、 链路层 —— 链路加密

### 2.网络层 —— IPSec

- IP 提供的是**不可靠**、**无连接**的数据报传输服务
- IP 安全机制（目前为止仅有IPSec是提供安全保障的）在网络层提供认证、机密性和密钥管理服务

#### IPSec

[h3c的官方讲解，讲的蛮好的，很详细](https://www.bilibili.com/video/BV1gS4y1A7PG)

- 提供的服务：数据机密性（加密）、数据完整性（HASH）、数据来源认证（数字签名）、抗重放（序列号）

- 协议框架
  - 认证头AH
  - 封装安全载荷ESP
  - 密钥交换协议IKE
  - 加密认证的算法
- SA(安全联盟)
  - 对于IPSec对等实体要素的约定。
  - 包括：
    - 安全协议：AH(来源/完整性/抗重放)、ESP(加密/来源/完整性/抗重放)
    - 算法：加密(DES/AES/3DES/SM4)、密钥协商算法(DH)、认证算法(MD5、SHA1、SM3)
    - **封装模式**：
      - 传输模式：不改变IP传输的报头，适用于安全传输的起点和终点为**实际的**起点和终点（用于两个主机之间的端到端通信）。
      - 隧道模式：头尾插入到IP报文之前，加新的IP头。保护安全网关的起点和终点，非数据包的实际起点和终点。
- 加密和认证流程![](/img/inPost/crypto/IPSec加密过程.png)

### 3.传输层 —— SSL(Secure Sockets Layer)/TLS(Transport Layer Security)

#### 3.1 SSL

SSL连接（connection）：暂时的，点对点，提供恰当类型服务的传输

SSL会话（session）：会话定义了一组可供多个连接共享的密码安全参数，为客户与服务器之间的关联。会话避免了为每一个连接提供新的安全参数所需昂贵的协商代价

SSL 记录协议为 SSL 连接提供两种服务
- 保密性/机密性：握手协议定义了共享的、可以对 SSL 有效载荷进行常规加密的密钥。
- 消息完整性：握手协议定义了共享的、可以用来形成报文鉴别码**MAC** 的密钥。

#### 3.2 TLS

![](/img/inPost/crypto/TLS.png)
[四个步骤、详细知识点](https://segmentfault.com/a/1190000002554673)
[原文](https://blog.cloudflare.com/keyless-ssl-the-nitty-gritty-technical-details/)

TLS连接是使用传输层安全协议在客户端和服务器之间建立的安全通信通道。它为通过互联网传输的数据提供隐私、完整性和身份验证。

会话状态是指在TLS握手过程中协商建立的参数和加密密钥。会话状态通常包括以下参数:
- 协议版本
- 密码套件(加密算法和密钥交换方法)
- 压缩方法
- 主秘密(用于派生用于加密数据的密钥)

TLS中常见的密码套件包括用于加密和身份验证的AES-256与SHA-256、用于较旧兼容性的AES-128与SHA，以及一些遗留套件，如带有MD5的RC4-128(由于漏洞不再推荐)。

在TLS中填充的目的是混淆被加密的明文数据的真实长度。这样做是为了防止某些类型的密码分析攻击，这些攻击可能仅根据密文长度推断出有关明文的信息。

TLS握手是在客户端和服务器之间建立安全会话的关键过程。它包括四个主要阶段:

- Hello:客户端发送一个带有支持的密码套件的ClientHello消息，服务器响应一个ServerHello消息，选择密码套件。
- 密钥交换:客户端和服务器使用RSA、Diffie-Hellman或椭圆曲线Diffie-Hellman等算法交换密钥。
- 身份验证:服务器发送自己的证书进行身份验证，如果需要相互认证，客户端也可以发送证书。
- Finished:双方验证密钥和密码套件是否正确，发送Finished消息以确认握手完成。


#### 3.3 HTTPS

[整体讲解的非常好的博客](https://segmentfault.com/a/1190000021494676)

HTTPS：HTTP + **S**SL/TLS=HTTP**S**实现浏览器和服务器之间的安全通信。

HTTPS被加密的内容：
- 请求文件的URL
- HTTP头部内容
- 浏览器表单的内容
- Cookie
- 文件内容

![](https://segmentfault.com/img/bVbClUl)

### 4.应用层 —— S/MIME, PGP, PEM, SET, Kerberos, SHTTP, SSH

#### 安全电子邮件系统 PGP (Pretty Good Privacy)

邮件系统

非交互的发送---RSA公钥发送

会话密钥---传统加密（主要是太大了，公钥加密不太适合）

### 5.无线网络安全

#### IEEE 802.11

#### IEEE 802.11i

[发展历程](https://blog.csdn.net/dolphin98629/article/details/39934373)

无线网络安全是确保无线网络数据传输安全的重要方面。802.11i是IEEE 802.11无线网络标准的一个重要修正案，它专注于提高无线网络的安全性。在802.11i标准中，引入了多种加密和认证机制，其中WAP（Wired Equivalent Privacy，有线等效保密）和CCMP（Counter Mode with Cipher Block Chaining Message Authentication Code Protocol，计数器模式密码块链接消息认证码协议）是两个关键的组成部分。

WEP（Wired Equivalent Privacy）是早期的无线网络加密标准，但由于其安全漏洞，逐渐被更安全的加密方法所取代。WEP使用的是RC4流密码算法，但因为其密钥管理和IV（初始化向量）的弱点，容易受到各种攻击，如FMS攻击（Fluhrer, Mantin, and Shamir）等。

CCMP是802.11i标准中定义的一种新的加密协议，用于取代WEP和TKIP（Temporal Key Integrity Protocol，临时密钥完整性协议）。CCMP使用AES（Advanced Encryption Standard，高级加密标准）算法作为其加密机制，提供更强的数据保护。CCMP结合了CTR（Counter Mode，计数器模式）来提供数据保密性，CBC-MAC（Cipher Block Chaining Message Authentication Code，密码块链接消息认证码）用于认证和完整性保护。

CCMP的加密过程包括以下几个步骤：
1. 使用AES算法和128位密钥对数据进行加密。
2. 每个加密的数据包都有一个128位的PN（Packet Number，数据包编号），用于防止重放攻击。
3. 一个104位的Nonce（一次性随机数），由PN、QoS（Quality of Service，服务质量）优先级字段和发送者地址组成。
4. AAD（Additional Authenticated Data，附加认证数据）由MPDU（MAC Protocol Data Unit，MAC协议数据单元）头部构建，确保MAC头部的完整性。

CCMP的这些特性使其成为无线网络安全中一个非常强大的加密协议，有效地提高了数据传输的安全性。然而，值得注意的是，CCMP对硬件的要求较高，因为它需要能够快速执行AES算法，这可能限制了它在一些旧设备上的应用。

总的来说，802.11i标准通过引入CCMP等加密机制，显著提高了无线网络的安全性，为用户在无线环境中的数据传输提供了强有力的保护。尽管WEP存在缺陷，但CCMP等后续技术的发展，确保了无线网络能够提供接近有线网络的安全水平。

## 恶意软件、入侵者、防火墙

- 恶意软件分类：依赖宿主的、独立于宿主的、进行复制的、不进行复制的

- 入侵检测：基于统计异常的入侵检测、基于规则的入侵检测

- 防火墙分类：包过滤路由器、应用级网关、电路级网关

- 分级安全关键策略：
    - 禁止向上读：指的是不允许用户或系统从较低的安全级别读取较高安全级别的数据。这是为了防止敏感信息被低安全级别的用户或系统访问。

    - 禁止向下写：指的是不允许用户或系统将数据写入到比当前安全级别更低的存储介质或系统中。这是为了防止敏感信息被存储在不安全的介质上，从而增加泄露的风险。

#### 蜜罐

蜜罐（Honeypot）是一种网络安全技术，用于诱捕攻击者。它通常是一个模拟的系统，看起来像是有价值的目标，但实际上是设计来检测、分析以及研究攻击者的恶意活动。蜜罐可以是软件，也可以是硬件，或者两者的组合，它们通常包含各种漏洞和陷阱，以吸引攻击者进行攻击。

蜜罐的主要特点和用途包括：

1. **诱捕攻击者**：蜜罐模拟易受攻击的系统，吸引攻击者尝试入侵。

2. **收集情报**：通过记录攻击者的行为，蜜罐可以帮助安全研究人员了解攻击者的策略、技术和程序（TTPs）。

3. **警报系统**：当攻击者与蜜罐交互时，可以触发警报，提醒网络安全团队有潜在的威胁。

4. **研究和教育**：蜜罐提供了一个受控的环境，用于研究攻击技术、测试防御措施，以及教育网络安全专业人员。

5. **隔离风险**：蜜罐可以放置在网络的非关键区域，以隔离和限制攻击者的活动范围，防止他们进一步渗透到敏感区域。

6. **法律和道德考量**：虽然蜜罐可以提供有关攻击者的信息，但它们的使用需要遵守法律和道德标准，避免侵犯隐私或进行非法监控。

蜜罐有多种类型，包括：

- **生产蜜罐**：部署在生产环境中，用于实时监控和分析攻击。
- **研究蜜罐**：用于学术研究或开发新的安全技术和策略。
- **分布式蜜罐**：分布在多个位置，以收集更广泛的攻击数据。
- **高交互蜜罐**：提供高度仿真的环境，可以运行操作系统和服务，以吸引更复杂的攻击。
- **低交互蜜罐**：只提供基本的仿真，通常用于检测扫描和简单的攻击尝试。

蜜罐是网络安全防御体系中的一个重要组成部分，它们帮助组织更好地了解和应对网络威胁。

#### bloom过滤器

Bloom过滤器是一种空间效率很高的数据结构，用于测试一个元素是否在一个集合中。它由Burton Howard Bloom在1970年提出，因此得名。Bloom过滤器可以回答两个问题：一个元素是否在集合中，或者一个元素一定不在集合中。然而，它也有一定的误判率，即可能会错误地报告一个元素在集合中，即使它实际上不在。

Bloom过滤器的主要特点包括：

1. **空间效率**：Bloom过滤器使用比存储元素本身少得多的内存空间来表示集合。

2. **时间效率**：检查一个元素是否存在于Bloom过滤器中的时间复杂度是O(k)，其中k是使用的哈希函数的数量。

3. **误判**：Bloom过滤器允许一定的误报率（false positives），但不允许误漏（false negatives）。这意味着它可能会告诉你一个元素在集合中，而实际上它不在，但它永远不会告诉你一个元素不在集合中，而实际上它在。

4. **不可逆**：Bloom过滤器是不可逆的，你不能从Bloom过滤器中恢复原始的元素集合。

Bloom过滤器的工作原理：

1. **初始化**：创建一个m位的位数组，所有位都初始化为0。

2. **添加元素**：对于要添加到集合的每个元素，使用k个不同的哈希函数计算出k个位索引，然后将这些位设置为1。

3. **查询元素**：对于要查询的元素，同样使用k个哈希函数计算出k个位索引，检查这些位是否都是1。如果是，那么元素可能在集合中（注意，这是可能，不是一定）；如果任何一个位是0，那么元素一定不在集合中。

4. **删除元素**：Bloom过滤器不支持删除操作，因为删除一个元素可能会影响其他元素的查询结果。

Bloom过滤器在以下场景中非常有用：

- 缓存系统，用于快速判断数据是否在缓存中。
- 数据库，用于快速过滤查询，减少不必要的磁盘I/O操作。
- 网络应用，用于快速识别重复的数据包或请求。

Bloom过滤器是一种非常实用的数据结构，尤其适用于需要快速查找和节省内存的场景。然而，由于其误判的特性，它通常不适用于那些对准确性要求极高的应用。

# 奇怪的草稿

显示有问题的一集
<!-- https://mermaid.nodejs.cn/syntax/flowchart.html -->

```mermaid
    graph TD
    R["R<32bit>"] -->|32 bit| Ebox
    Ebox -- 48bit --> +((+))
    密钥K -- 48bit --> +
    + -->|6 bit| S1
    + -->|6 bit| S2
    + -->|6 bit| S3
    + -->|6 bit| S4
    + -->|6 bit| S5
    + -->|6 bit| S6
    + -->|6 bit| S7
    + -->|6 bit| S8
    S1 -->|4 bit| Pbox
    S2 -->|4 bit| Pbox
    S3 -->|4 bit| Pbox
    S4 -->|4 bit| Pbox
    S5 -->|4 bit| Pbox
    S6 -->|4 bit| Pbox
    S7 -->|4 bit| Pbox
    S8 -->|4 bit| Pbox
    Pbox --> Result[32bit数据]
``` 

<span class='mermaid'>
graph LR
A[明文] -->|64 bit| B[初始置换]
B --> C((Round1))
C --> D((Roundi))
D --> E((Round16))
E --> F[初始逆置换]
F --> |64 bit| G[密文]
</span>

```math
好处=\left\{
\begin{array}{l}
 a.加密本身不能实现真实性认证功能 \\
 b.认证和加密的分离带来灵活性 \left\{
    \begin{array}{l}
        1.只要认证而不需保密\\
        2.不能加密的场合\\
        3.存档期间的保护
    \end{array}
    \right.
\end{array}
\right.
```

# Reference
[西电学长的期末复习](https://blog.csdn.net/qq_56406979/article/details/131565846)

[LMAR博主的AES、DES都蛮清晰](https://blog.csdn.net/qq_44131896/article/details/117626013)

[密码学部分的基础 —— Dan Boneh斯坦福密码学课程](https://www.coursera.org/learn/crypto)

[安全协议部分大全讲解](https://thomas-li-sjtu.github.io/2020/11/03/Internet%E5%AE%89%E5%85%A8%E5%8D%8F%E8%AE%AE%E4%B8%8E%E5%88%86%E6%9E%90_%E7%AC%94%E8%AE%B0%E6%95%B4%E7%90%86/#%E7%AC%AC%E4%B8%89%E7%AB%A0-Kerberos)

[山大的试卷1](https://blog.csdn.net/m0_52559974/article/details/129046455)

[山大的试卷2](https://blog.csdn.net/sduwaer/article/details/135412570?)