---
title: "Cryptography"
subtitle: "密码学复习"
layout: post
author: "tracer"
hidden: true
header-img: "img/inPost/crypto/cryptography-bg.jpg"
tags:
    - Cryptography
    - Protocol Security
    - Web Security
---

# 现代密码学&网络与协议安全复习

<img src='https://github.com/ztracer/ztracer.github.io/blob/master/img/51.gif?raw=true' />

## 1. 对称密钥加密

#### 1.1 流密码 【Stream Ciphers】
- 1.1.1 流密码的安全性在于?
    - PRG：安全性关注且 **只** 关注 PRG
- 1.1.2 OTP：one time pad 【真正的安全.jpg】
    - attack
        - 2TP
        - NO integrity-malleable
- 1.1.3 伪随机数生成器PRG【 PseudoRandom Generators】
    - 使得OTP更加实用
    - G:{0,1}^s -> {0,1}^n 其中n >> s
    - weak PRG
        - 线性同余方程
    - Security
        - 定义：key与随机生成的r无法区分“indistinguishable”
            - “indistinguishable”定义
                - 统计测试要有*可忽略*的优势
                    - 优势Advantage的定义：
                        统计测试A输出1的可能性概率
                        相对于
                        PRG输出1的可能性概率的差
- 1.1.4 流密码的一些实际的例子
    - [RC4](https://en.wikipedia.org/wiki/RC4)
        - 面向软件设计的
    - [CSS](https://en.wikipedia.org/wiki/Content_Scramble_System) 【content scrambling system】
        - DVD中的一种加密算法【所以是面向硬件设计的一种加密算法】
        - 现状：badly broken
        - 用的PRG是LSFR
            - LSFR专题: [知乎](https://zhuanlan.zhihu.com/p/366067972)

#### 1.2 分组密码 【Block Ciphers】
- 基本模型
    - key k  -----扩展成------> 多次加密所需的k<sub>n</sub>
       m 经过 n 轮加密
- PRP & PRF
    - PR**Function** : K*X -> Y
        - Security: 与F[x,y]不可区分
    - PR**Permutation** : K*X -> X
        - 一一映射的
        - Security: 与E[x,x']不可区分
            - 注意E为一一映射函数
            - 分组密码的安全性保障
    - PRF ---构建---> PRGenerator
        - Security: PRG生成的K能使得PRF与真随机不可区分
- standard
    - DES【data encryption standard】
        - 构建 : feistel network
            - R/L两个nbit的块组合成2nbit
        - attack
        - 3DES
           n=64bits
           k=168bits
    - AES【Advanced Encryption Standard】
        - 构建 : 可逆的Subs-Perm Network
        - n=128bits
           k=128\192\256bits
- Using Block Ciphers
    - ECB【electronic code book】
        - 泄露轮廓信息
        - 不能加密大于>1个分组的信息
    - OTK【one time key】
        - security for *many time key*
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
            - 1.random *IV*
                - IV<sub>1</sub>=E(k,m[0]⊕IV)
                - ciphertext为
                   IV || c[0] || ···· || c[n]
            - 2.nonce-based
                - IV = E(k<sub>1</sub>,nonce)
        - 明文短了怎么办 - 》补齐到加密格的大小
            - Attack: 密文偷窃问题
    - CTR【counter mode】
        - 构建CBC
            - 1.rand ctr-mode
                - c[L]=m[L]⊕F(k,IV+L)
            - 2.nonce ctr-mode

## 2. 消息完整性【Message Integrity】

#### 2.1 基本定义和应用
- MAC【Message Authentication Codes】
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

## 4. 密钥与认证【与谁who-干什么what问题】*写得有点乱这个部分*

#### 4.0 有哪些密钥【网络与协议安全中重要的一个部分】
- 【**密钥是有生存周期的**】
- 基本密钥/会话密钥
- 公钥加密的：公私钥
- 用来签名的：公私钥

#### 4.1 认证和授权
- 4.1.1 认证 (Authentication):进程通过认证过程来验证它的通信对方是否是它所**期望的实体**而不是假冒者【关于who的问题】
- 4.1.2 认证过程
    - 1.需要建立共享密钥:
        ✔️方法1【**非对称加密**的D-H】: 
        - 1、初始化：A选取一个生成元g和一个大素数p，发送给B
        - 2、A随机选取0<x<p-1，计算X = g<sup>x</sup> mod p后，将X传给B
        - 3、B随机选取0<y<p-1，计算Y = g<sup>y</sup> mod p后，将Y传给A
        - 4、A计算Y<sup>x</sup> mod p得到会话密钥K1
        - 5、B计算X<sup>y</sup> mod p得到会话密钥K2
        - 6、由下可知 K1 == K2，此时双方便可利用会话密钥进行加密通信了
        - 这个方法依赖于离散对数问题的难度
        ⚠️但是D-H存在*中间人攻击*
        ![](/img/inPost/crypto/middle_attack.png)
        - **预防措施与新的问题**：
            - 可以用自己另外一对公私钥中的私钥签名【但是公钥的分发存在问题】
              - 公钥如果直接发送也存在*中间人攻击*，确保不了与网站之间生成了对话密钥
              - 这里需要第三方CA----Certificate Authority
              ![技术蛋老师](/img/inPost/crypto/CA.png)[Pic From BiliBili 技术蛋老师](https://www.bilibili.com/video/BV1mj421d7VE)
              核心在于CA的数字签名
              - BUT又双叒叕有一个问题——>CA网站发给你的时候可能也会有中间人攻击——>这就套娃了——>所以需要证书链&一个根证书【OS和浏览器内预先安装】
              - 结合一个实际例子来看看
              ![](/img/inPost/crypto/eg1_compare.png)
              - 中心化依赖着的是对于“中心”的信任，有没有“去中心化的监督呢”----新机制[CT](https://www.bilibili.com/video/BV1uY4y1D7Ng)-Certificacte Transparency【类似于区块链Merkle tree】
            - 使用公钥基础设施（PKI）：通过使用证书颁发机构（CA）颁发的公钥证书来验证通信双方的身份。在Diffie-Hellman交换之前，双方首先交换并验证对方的证书。这可以确保交换的密钥不会被第三方篡改。
        
        <br>✔️方法2【需要第三方KDC来分发】: Key Distribution Center
        - 但是⚠️存在*重放攻击*
        - 重放 Alice 的初始请求：攻击者可能截获 Alice 发给 KDC 的加密请求（该请求包含希望与 Bob 使用的会话密钥 KS），然后重新发送这个请求。如果系统没有适当的防护措施，如时间戳或一次性标记，KDC 可能会误认为是 Alice 再次请求与 Bob 通信，从而导致未授权的会话密钥复用。
        - 重放 KDC 对 Bob 的响应：攻击者也可能截获从 KDC 发往 Bob 的加密消息，这个消息包含会话密钥 KS 和 Alice 的身份。攻击者随后可以重新发送这个消息给 Bob，使 Bob 相信是收到了来自 Alice 通过 KDC 的新的会话请求。这可能导致 Bob 在不知情的情况下使用旧的或已泄露的会话密钥。
        - **预防措施**：
            - *时间戳*：包含在每个请求和响应中的时间戳可以帮助接收者验证消息的时效性，确保消息是在一个预期的时间窗口内发送的。
          - *序列号*：为每个消息分配唯一的序列号可以防止消息被重新发送。
          - *短期会话密钥*：限制会话密钥的有效期可以减少因重放攻击而造成的风险。
    <br>
    - 2.基于共享密钥的认证: 【有了共享密钥之后就是对称加密了】
        - 一方发送一个随机数给另一方，然后由另一方以特殊的方式对其进行转换并返回结果。这种协议称为挑战-响应协议。
        ![](/img/inPost/crypto/challen.png)
        - 容易受到*反射攻击*【用“共享的密钥”这该特性来“借刀杀人”】【关键在于***返回对方发过来的随机数***】
        ![](/img/inPost/crypto/reflection-attack.png)
        - 解决方法：使用HMAC
            - A --------------------R<sub>A</sub>------------------> B
            - A <---R<sub>B</sub>,HMAC[R<sub>A</sub>,R<sub>B</sub>,A,B,K<sub>AB</sub>]-- B
            - A ---------HMAC[R<sub>A</sub>,R<sub>B</sub>,K<sub>AB</sub>]-------> B 【A告诉B我能解出来】
        两个 HMAC 包含发送方选择的值，KAB攻击者未知，攻击者无法修改选择的值。
- 4.1.3 授权 (authentication):授权关注的是允许这个进程做什么样的事情

#### 4.3 现有的解决方案1：Kerberos 
>【基于可信第三方的认证，分布式环境下的认证服务】

![](/img/inPost/crypto/Kerberos.png)

#### 4.4

#### 4.5 Merkle Puzzles
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
           k<-H(x)作为中间密钥 
           输出(y,E(k,m))
- PKCS1
    - 预处理
        - 02||random pad(~~FF~~)||FF||msg
        - ATTACK

#### 5.5 基于Diffie-Hellman的公钥加密


