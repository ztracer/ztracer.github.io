---
title: "Cryptography"
subtitle: "密码学复习"
layout: post
author: "tracer"
hidden: true
catalog: false
published: false
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

#### 1.1 流密码 【Stream Ciphers】

##### 1.1.1 流密码的安全性在于?
- PRG：安全性关注且 **只** 关注 PRG

##### 1.1.2 OTP：one time pad 【真正的安全.jpg】
- attack
  - 2TP
  - NO integrity-malleable

##### 1.1.3 伪随机数生成器PRG【 PseudoRandom Generators】
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

##### 1.1.4 流密码的一些实际的例子
- [RC4](https://en.wikipedia.org/wiki/RC4)
    - 面向软件设计的
- [CSS](https://en.wikipedia.org/wiki/Content_Scramble_System) 【content scrambling system】
    - DVD中的一种加密算法【所以是面向硬件设计的一种加密算法】
    - 现状：badly broken
    - 用的PRG是LSFR
        - LSFR专题: [知乎](https://zhuanlan.zhihu.com/p/366067972)

#### 1.2 分组密码 【Block Ciphers】

##### 1.2.1 基本模型
key k  -----扩展成------> 多次加密所需的k<sub>n</sub>
    m 经过 n 轮加密

##### 1.2.2 PRP & PRF
- PR**Function** : K*X -> Y
    - Security: 与F[x,y]不可区分
- PR**Permutation** : K*X -> X
    - 一一映射的
    - Security: 与E[x,x']不可区分
        - 注意E为一一映射函数
        - 分组密码的安全性保障
- PRF ---构建---> PRGenerator
    - Security: PRG生成的K能使得PRF与真随机不可区分

##### 1.2.3 standard 分组密码标准
**DES【data encryption standard】**：第一个并且是最重要的现代分组密码算法（用 **56位密钥**来加密 **64位数据**的方法）
- 构建*feistel network* : 
    ![](/img/inPost/crypto/DES总体架构.png) 
    1. 黄色部分的：初始置换和初始逆置换![初始置换与初始逆置换是互逆的](/img/inPost/crypto/DES初始置换和初始逆置换.png)
    2. DES-16轮中的每轮迭代加密步骤：![](/img/inPost/crypto/DES每轮迭代加密.png)
    3. 其中的***f***函数是什么？![](/img/inPost/crypto/DES-f函数.png)
    ![](/img/inPost/crypto/DES流程图.png)
       - Expansion-Ebox是:
        ![](/img/inPost/crypto/DES-E盒扩展.png)
        将32bit扩展到48bit
       - 48bitEbox输出与48bit密钥逐一异或
       - Substitution-Sbox：分成8块【混乱】
       (Sbox是核心!是非线性变换的关键)
        ![](/img/inPost/crypto/DES-S盒.png)
       Sbox提供了密码算法所必须的混乱作用，将48bit变回32bit
       - Permutation-Pbox【扩散】
       进行很常见的"置换"后输出32bit数据
    4. 密钥扩展（64bit -> 扩展成每轮的48bit）
       1. PC-1置换选择1：置换 + 选择64bit其中的56bit【其中有8位是奇偶效验位】（置换后的前28bit放在C寄存器中，其余的后28bit放在D寄存器中）
       2. LS移位：两个28位密钥的寄存器循环左移（1，2，9，16轮循环左移1位，其他12轮循环左移2位 = 4\*1+12\*2 = 28 —— 16轮后刚刚好移位数等于28）
       3. PC-2置换选择2：置换 + 选择56bit其中的48bit
- attack
    1. 雪崩效应：在高质量的分组密码中，无论密钥或明文的任何细微变化都应当引起密文的剧烈改变。如果存在“弱密钥：即给定初始密钥k，各轮的子密钥都相同，即有 k1=k2= … =k16，就称给定密钥 k 为弱密钥 (Weak key)”，这种情况下DES在选择明文攻击下的搜索量减半。
    2. 穷举攻击：1999年时，已经缩短到22个小时
- [一个比较好的关于DES的讲解](https://blog.csdn.net/qq_44131896/article/details/117573452)
- 3DES
   - n=64bits
   - k=168bits
   - but： 软件实现速度慢，分组短

**AES【Advanced Encryption Standard】**：
- n=128bits；k=128\192\256bits
- 构建 : 可逆的Subs-Perm Network——in GF(2<sup>8</sup>)

##### 1.2.4 分组密码的工作模式 Using Block Ciphers
- ECB【electronic code book】
    - 泄露轮廓信息（相同明文加密产生相同密文）
    - 不能加密大于>1个分组的信息
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
- CBC【cipher block chaining】
  - 构建CBC
      1. random *IV*
          - IV<sub>1</sub>=E(k,m[0]⊕IV)
          - ciphertext为
             IV || c[0] || ···· || c[n]
      2. nonce-based
          - IV = E(k<sub>1</sub>,nonce)
  - 明文短了怎么办 - 》补齐到加密格的大小
      - Attack: 密文偷窃问题
- CTR【counter mode】
  - 构建CTR
      - 1.rand ctr-mode
          - c[L]=m[L]⊕F(k,IV+L)
      - 2.nonce ctr-mode

## 2. 消息完整性【Message Integrity】

#### 2.1 基本定义和应用
##### MAC【Message Authentication Codes】
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
        - m[pad] = k<sub>LAST</sub> || fpad 为补齐
- PMAC
    - (⊕)$\sum_{i=0}^{n-1}F(k_i,m[i]⊕P(k,i))$
       tag=F(k1,(⊕))

#### 2.2 抗碰撞【Collision resistance】
- Attack
    - birthday paradox
- 构建抗碰撞的HASH--Merkle-Damgard Paradigm
- HMAC(HASH+MAC)

## 3. 认证加密 【Authenticated Encryption】

#### 单纯的加解密算法（解决机密性问题）无法抵抗下述攻击：
• 内容修改
• 顺序修改
• 计时修改
• 发送方否认
• 接收方否认
这些属于“数据完整性”范畴

SO -> 我们需要加密与认证分离

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

#### 3.1 认证加密用来解决什么问题-》不被窃听+不被修改
- (E,D)一对算法的输出需要满足
   E:K\*M\*N -> C
   D:K\*C\*N -> M U {不存在}
    - 能解决问题1 : 是认证的加密,重放攻击是无效的
    - 能解决问题2 : 同时拥有CPA和CCA(针对oracle的安全)

#### 3.2 construction
- 1.MAC-then-Encrypt
- 2.Encrypt-then-MAC
- 3.Encrypt-and-MAC x

#### 3.3 安全的散列函数
为文件、消息或其他数据块产生“指纹”，浓缩任意长的消息M到一个固定长度的取值h=H(M)，对于Hash函数h=H(x)，称x是h的原像，称即把数据块x作为输入，称使用Hash函数H得到h。

- 如果满足**x≠y**且**H(x)=H(y)****，称则称出现碰撞 (collision)
  - 抗原像攻击（单向性）：*对任意给定的散列码h，找到满足H(x)=h的x*在计算上不可行，单向性。
  - 抗第二原像攻击（抗弱碰撞攻击）：*对任何给定分组x，找到满足y≠x且H(x)=H(y)的y*在计算上不可行，抗弱碰撞性。
  - 抗碰撞攻击（抗强碰撞攻击）：*找到任何满足H(x)=H(y)的偶对(x, y)*在计算上不可行，抗强碰撞性。

## 4. 密钥与认证【与谁-干什么；who-what问题】*写得有点乱这个部分*

### 4.0 有哪些密钥【网络与协议安全中重要的一个部分】
- 【**密钥是有生存周期的**】
- 基本密钥/会话密钥
- 公钥加密的：公私钥
- 用来签名的：公私钥

### 4.1 认证过程

小对比：
授权 (authentication):授权关注的是允许这个进程做什么样的事情
认证 (Authentication):进程通过认证过程来验证它的通信对方是否是它所**期望的实体**而不是假冒者【关于who的问题】

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
- 一方发送一个随机数给另一方，然后由另一方以特殊的方式对其进行转换并返回结果。这种协议称为挑战-响应协议。
![](/img/inPost/crypto/challen.png)
- 容易受到*反射攻击*【用“共享的密钥”这该特性来“借刀杀人”】【关键在于***返回对方发过来的随机数***】
![](/img/inPost/crypto/reflection-attack.png)
- 解决方法：使用HMAC
    - A --------------------R<sub>A</sub>------------------> B
    - A <---R<sub>B</sub>,HMAC[R<sub>A</sub>,R<sub>B</sub>,A,B,K<sub>AB</sub>]-- B
    - A ---------HMAC[R<sub>A</sub>,R<sub>B</sub>,K<sub>AB</sub>]-------> B 【A告诉B我能解出来】
两个 HMAC 包含发送方选择的值，KAB攻击者未知，攻击者无法修改选择的值。

### 4.2 现有的解决方案1：Kerberos 
>【基于可信第三方的认证，分布式环境下的认证服务】

![](/img/inPost/crypto/Kerberos.png)

### 4.3 Merkle Puzzles
- 密钥交换双方O(n),攻击者O(n<sup>2</sup>)

## 5.公钥加密

#### 5.1 数论的基本知识

#### 5.2 应用场景
- 非“互动”应用
- 会话建立【网络】

#### 5.3 公钥的安全定义
- 抗窃听
    - 攻击者产生两个信息m0和m1，长度相同，然后攻击者用公交加密得到c0或c1，他不能区分他获得的是m0的加密还是m1的加密。
    - （不用考虑多次加密的情况，因为本身就能多次加密）
- 对抗主动攻击
    - 选择密文安全

#### 5.4 基于陷门置换的公钥加密
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
    - TESTBOOK RSA【是一种错误的使用方法】
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

#### 5.5 基于Diffie-Hellman的公钥加密


# 网络与协议安全部分复习

## 网络与协议安全期末复习大纲

#### 1. 考试结构与分值分布
- **选择题**：10分
- **填空题**：20分
- **简答题**：40分
- **综合题**：30分【协议运用】

#### 2. 网络安全基础
- **核心概念**：机密性、完整性、可用性
- **OSI模型**：网络安全设计的基础框架
- **TCP/IP模型**：网络协议的基础划分

#### 3. 密码学基础
- **古典密码与对称密码**：包括置换和混淆
- **公钥密码体系**：基于困难数学问题，如RSA
- **密钥管理**：保密性依赖于密钥的机密性

#### 4. 对称加密算法
- **流密码**：每次加密时密钥只使用一次
- **分组密码**：如AES的不同变体（AES-128, AES-192, AES-256）

#### 5. 公钥密码体系
- **RSA算法**：理解其数学基础和加密/解密过程

#### 6. 安全协议
- **消息认证码（MAC）**：使用哈希函数进行消息认证
- **数字签名**：确保消息的完整性和来源验证

#### 7. 网络安全协议
- **IPsec**：传输模式和隧道模式
- **TLS**：握手过程和使用的密码套件
- **Kerberos**：认证服务和票据系统

#### 8. 无线网络安全
- **802.11i**：无线局域网的安全协议
- **密钥管理**：包括3A密钥和PKI

#### 9. 应用层安全
- **电子邮件安全**：PGP和S/MIME协议
- **DNS操作**：电子邮件传输过程中的DNS相关操作

#### 10. 网络攻击与防护
- **恶意软件**：依赖于宿主的和独立于宿主 or 不进行复制的和进行复制
- **入侵检测系统**：基于规则和异常的检测技术
- **蜜罐技术**：诱捕攻击者

#### 11. 口令安全
- **哈希表**：用于存储和验证用户口令
- **错误过滤器**：减少碰撞概率和提高安全性

#### 12. 防火墙
- **作用**：监控和控制进出网络流量
- **局限性**：不能完全保证网络安全

#### 复习策略
- **理解核心概念**：确保对基础概念有深刻理解。
- **掌握加密算法**：重点复习DES和AES算法的细节。
- **学习安全协议**：熟悉常用安全协议的工作原理和应用场景。
- **实践应用**：通过实例和案例学习加深对协议和算法的理解。
- **关注安全性**：了解常见的网络攻击手段和防护策略。
- **参与课堂讨论**：积极提问，解决理解上的疑惑。

## Web安全、Internet 安全途径

Web安全威胁：完整性、机密性、拒绝服务、认证

#### 1.网络接口层 * 链路层 —— 链路加密

#### 2.网络层 —— IPSec
- IP 提供的是**不可靠**、**无连接**的数据报传输服务
- IP 安全机制（目前为止仅有IPSec是提供安全保障的）在网络层提供认证、机密性和密钥管理服务

##### IPSec
- ※1：IPSec传输模式和隧道模式
    - 传输模式 Transport Mode: 用于两个主机之间的端到端通信
    - 隧道模式 Tunnel Mode: 适用于当 SA（安全关联）的一端或两端是安全网关
        - 安全关联SA是在发送者和为通信提供安全服务的接受者之间的一种单向关系。
        - 若需要双向的安全交换，就要用两个安全关联。
        - 提供给一个SA的安全服务用于AH（身份认证头）或ESP（封装安全载荷），但不能同时用于两者。
- ※2：IP 安全策略


#### 3.传输层 —— SSL(Secure Sockets Layer)/TLS(Transport Layer Security)

##### 3.1 SSL

SSL连接（connection）：暂时的，点对点，提供恰当类型服务的传输

SSL会话（session）：会话定义了一组可供多个连接共享的密码安全参数，为客户与服务器之间的关联。会话避免了为每一个连接提供新的安全参数所需昂贵的协商代价

SSL 记录协议为 SSL 连接提供两种服务
- 保密性/机密性：握手协议定义了共享的、可以对 SSL 有效载荷进行常规加密的密钥。
- 消息完整性：握手协议定义了共享的、可以用来形成报文鉴别码**MAC** 的密钥。


##### 3.2 TLS

![](/img/inPost/crypto/TLS.png)

##### 3.3 HTTPS

HTTPS：HTTP 和 SSL/TLS 结合，实现浏览器和服务器之
间的安全通信。

HTTPS被加密的内容：
- 请求文件的URL
- HTTP头部内容
- 浏览器表单的内容
- Cookie
- 文件内容

#### 4.应用层 —— S/MIME, PGP, PEM, SET, Kerberos, SHTTP, SSH ……

###### 安全电子邮件系统 PGP (Pretty Good Privacy)

邮件系统

非交互的发送---RSA公钥发送

会话密钥---传统加密（主要是太大了，公钥加密不太适合）

#### 5.无线网络安全

##### IEEE 802.11

##### IEEE 802.11i


## 恶意软件、入侵者、防火墙

- 恶意软件分类：依赖宿主的、独立于宿主的、进行复制的、不进行复制的

- 入侵检测：基于统计异常的入侵检测、基于规则的入侵检测

- 防火墙分类：包过滤路由器、应用级网关、电路级网关

- 分级安全关键策略：
    - 禁止向上读：指的是不允许用户或系统从较低的安全级别读取较高安全级别的数据。这是为了防止敏感信息被低安全级别的用户或系统访问。

    - 禁止向下写：指的是不允许用户或系统将数据写入到比当前安全级别更低的存储介质或系统中。这是为了防止敏感信息被存储在不安全的介质上，从而增加泄露的风险。

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


# Reference
[西电学长的期末复习](https://blog.csdn.net/qq_56406979/article/details/131565846)