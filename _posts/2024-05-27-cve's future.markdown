---
title: "CVE-CPE-NVD 未来何去何从?"
subtitle: "CVE-why-did-it-update-slowdown"
layout: post
author: "tracer"
hidden: true
header-img: "img/bg-about2.jpg"
tags:
    - CVE
    - CPE
    - SBOM
    - NVD
    - NIST
---
# NVD陷入更新停滞？疑似陷入动荡期

## Reference

NVD官网 https://www.nist.gov/itl/nvd

3.8 缘起 https://anchore.com/blog/national-vulnerability-database-opaque-changes-and-unanswered-questions/

3.15 基础信息与SBOM https://www.infosecurity-magazine.com/news/nist-vulnerability-database/

3.20 一些CVE的知识以及2月1日公告 https://heimdalsecurity.com/blog/nvd-stopped-updating-cves/

3.28 https://www.infosecurity-magazine.com/news/nist-unveils-new-nvd-consortium/

4.16 https://www.infosecurity-magazine.com/news/open-letter-nist-restore-nvd/

4.25 nvd https://nvd.nist.gov/general/news/nvd-program-transition-announcement

![alt text](/img/inPost/CVE/425NVD-en.png)
![alt text](/img/inPost/CVE/425NVD-zh.png)

5.14 https://www.infosecurity-magazine.com/news/nist-unveils-new-nvd-consortium/

CHINA CNVD? https://www.atlanticcouncil.org/in-depth-research-reports/report/sleight-of-hand-how-china-weaponizes-software-vulnerability/

5.23 https://www.infosecurity-magazine.com/news/nvd-exploited-vulnerabilities/

5.30 https://www.infosecurity-magazine.com/interviews/open-source-key-nvd-overreliance/

![alt text](/img/inPost/CVE/529NVD-en.png)
![alt text](/img/inPost/CVE/529NVD-zh.png)


## 安全人员视角->有人发现·····

自2024年2月12日起，NIST几乎完全停止扩充其国家漏洞数据库（NVD）中列出的软件漏洞，NVD是世界上使用最广泛的软件漏洞数据库。

### 没给出详细的分析

固件安全提供商NetRise的首席执行官汤姆·佩斯（TomPace）告诉Infosecurity，自该日期以来发布的2700个漏洞中，只有200个漏洞（称为常见漏洞和披露（CVE））得到了丰富。未能丰富CVE意味着已上传添加到数据库的2500多个漏洞，而没有关键的元数据信息。此信息包括对可能导致漏洞利用的漏洞和软件“弱点”的描述（称为常见弱点和暴露，或CWE）、受影响的软件产品名称、漏洞的严重性评分（CVSS）和漏洞的修补状态。

该问题最初是由软件安全提供商Anchore的安全副总裁[Josh Bressers](https://anchore.com/blog/national-vulnerability-database-opaque-changes-and-unanswered-questions/)发现的，他在3月8日发表了一篇博客文章，显示从2月12日左右开始，NVD上的丰富数据大幅下降。

![](img/inPost/CVE/NVD_CVE_2024.png)
绿色是NVD中的所有CVE-ID。红色是附加了CPE的ID

从2月12日开始，NVD发布了数千个CVE-ID，没有任何分析记录。自2024年初以来，共有**6171**个CVE-ID，其中只有**3625**个被NVD丰富。这留下了**2546**个缺口（42%！）

NVD 已成为组织和安全产品依赖其数据进行安全操作的行业标准，例如确定漏洞修复的优先级和保护基础设施。CVE ID 会不断添加和更新，但这些 ID 缺少 NVD 提供的关键分析数据。任何依赖 NVD 获取漏洞数据（如 CVSS 分数）的组织都不再接收 CVE 数据的更新。这意味着依赖这些数据的组织会因新的漏洞而蒙在鼓里，从而为其环境带来更大的风险和不受管理的攻击面。

### NVD与NIST公告与变动？预示着什么？

### NVD的前世今生-简短的历史课
成立于2005年，国家漏洞数据库NVD是美国国家标准与技术研究院（NIST）收集的漏洞数据。截至今天，许多公司依靠NVD数据进行安全运营和漏洞研究。

在过去的 2020 年，国家漏洞数据库向获得 CVE ID 的漏洞添加了以下信息：
- 通过通用漏洞评分系统 （CVSS） 的估计严重性评分
- 漏洞类型 – 常见弱点枚举 （CWE）
- 有关 CVE 影响的版本和产品的数据（CPE 条目）
- 有关漏洞功能的信息
- 有关黑客如何利用 CVE 的信息
- 公告链接

在NIST发表声明之前，关于正在发生的事情的猜测包括：

NIST内部的预算问题，因为立法者最近批准了NIST本财年14.6亿美元的预算，比上一年减少了近12%
与承包商（可能是亨廷顿英格尔斯工业公司）签订的终止合同，亨廷顿英格尔斯工业公司是一家造船承包商，在NVD上与NIST公开合作
内部讨论以取代 NVD 使用的一些漏洞标准，例如用作 IT 产品指纹的通用产品枚举器 （CPE），用于清楚地识别软件、硬件和系统
内部讨论开始采用软件包 URL （PURL），这是一种列出软件包通用地址的新标准


### 过于依赖NVD，我们需要思考一些什么？？
IM：你认为商业软件产品在漏洞管理方面过于依赖NVD吗？

高炉： 是的，我认为太多的工具依赖于 NVD 源来获取漏洞丰富的数据。一旦他们停止这样做，所有这些工具实际上都对新的漏洞视而不见。当然，被披露的漏洞数量和世界上软件的数量都在逐年增加。这意味着，作为一个软件开发人员，你真的有这个巨大的规模和广度问题。为了解决这个问题，由MITRE运营并由美国网络安全和基础设施安全局（CISA）赞助的CVE计划一直在努力通过引入CNA并允许不同的组织直接分配号码来建立联合模型。这取代了您必须请求号码并自己进行报告的系统。我认为这种联合模型在允许整个程序扩展方面非常有效。然而，NVD似乎没有采取同样的方法。他们仍在雇用合同研究人员来有效地研究一个大规模增长的空间。这些人自己添加扩充的数据，例如通用平台枚举器 （CPE） 和严重性评分 （CVSS），并在单个源中提供这些数据。这个系统从来没有完全适应开源安全，现在已经达到了极限。


IM：为什么你认为NVD不能很好地适应开源安全？

高炉：从本质上讲，CPE 系统不太适合开源组件。通过使用 CPE 识别漏洞，您可以只添加几个常见字段，例如项目、创建者和版本。如果您正在考虑像 Microsoft Word 版本 2010 这样的商业产品，这将非常有用。但是，这不适用于像 Apache Struts 或 Spring 这样的组件，其中有 80 到 100 个单独的组件。在如此复杂的开源项目中为漏洞分配 CPE 编号意味着您将不知道漏洞的确切位置——这可能会导致大量误报。它不够精确。