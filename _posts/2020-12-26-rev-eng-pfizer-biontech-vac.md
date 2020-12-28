---
layout: post
title: 【翻译】逐行解释BioNTech/Pfizer COVID疫苗的源代码
category: posts
comments: true
visible: true
---

[原文链接](https://berthub.eu/articles/posts/reverse-engineering-source-code-of-the-biontech-pfizer-vaccine/)
[原作者联络方式](mailto:bert@hubertnet.nl)

# 逐行解释BioNTech/Pfizer COVID疫苗源代码

欢迎大家！在这篇文章中我们将逐字解译BioNTech/Pfizer SARS-CoV-2 mRNA疫苗的源代码。

大家听起来可能很奇怪：疫苗是一种注射到人们手臂中的液体，源代码又是从何谈起？

![BNT162b2 mRNA的前500个字符。资料来源：世界卫生组织](https://berthub.eu/articles/bnt162b2.png)

这是一个很好的问题。要回答这个问题，就让我们从BioNTech/Pfizer疫苗源代码的一部分说起，也就是[BNT162b2](https://en.wikipedia.org/wiki/Tozinameran)，也称为Tozinameran，[或是Comirnaty](https://twitter.com/PowerDNS_Bert/status/1342109138965422083)。
这段代码也构成了BNT162b mRNA疫苗的核心。这段代码的长度为4284个字符，和一组推文差不多长。真正疫苗生产的过程就是从上传这段代码到【DNA打印机】开始（打印机如图），然后打印机就会将这段代码打印成对应的DNA分子。

![Codex DNA BioXp 3200 DNA打印机](https://berthub.eu/articles/bioxp-3200.jpg)

这些打印机会先产生少量的DNA，经过大量的生物和化学处理后，这些DNA分子最终就变成了疫苗瓶中的RNA序列（疫苗瓶标注的30微克的剂量实际上就是指包含30微克的RNA分子）。除了RNA本身外，疫苗瓶中还有一些脂质来帮助这些mRNA导入人体细胞。

生物体和计算机对于“源代码”的处理方式有很多惊人的相似之处。DNA就像生物体存储在硬盘上的信息：DNA序列自身包含冗余信息并且非常可靠耐用，而RNA就像是DNA的“内存”版。就好像在执行一段程序时，计算机并不会直接从硬盘执行代码而是先把代码复制到更快，也更短期的内存中，生物体也会先把DNA序列转化成RNA序列，再开始用RNA序列生产蛋白。与此同时，RNA序列也具有内存上存储的信息容易损失的缺点，这也是为什么Pfizer/BioNTech mRNA疫苗必须存放在超低温冷柜中储藏才能保证其有效性。

每个RNA字符的重量约为0.53·10<sup>-21</sup>克，这意味着在30毫克的疫苗剂量中就有6·10<sup>16</sup>个字符。换算成字节大约一针就传了有25PB的信息，不过其实这一针上传的是2万亿份相同的4284个字符。疫苗中真正的信息量刚刚超过1KB，而[SARS-CoV-2病毒](https://www.ncbi.nlm.nih.gov/projects/sviewer/?id=NC_045512&tracks=[key:sequence_track,name:Sequence,display_name:Sequence,id:STD649220238,annots:Sequence,ShowLabel:false,ColorGaps:false,shown:true,order:1][key:gene_model_track,name:Genes,display_name:Genes,id:STD3194982005,annots:Unnamed,Options:ShowAllButGenes,CDSProductFeats:true,NtRuler:true,AaRuler:true,HighlightMode:2,ShowLabel:true,shown:true,order:9]&v=1:29903&c=null&select=null&slim=0)序列本身则有包含了大约7.5KB的信息。

## 一些背景知识

DNA可以说是生物体的”代码“。与使用0和1的计算机不同，生物体使用A，C，G和U / T（“核苷酸”，“核苷”/“碱基”）来存储关于自己的信息。

在计算机中，我们会用许多物理现象来表示0和1：电荷的存在与否，电流高低，电压高低，跃迁，信号的波形，反射率的改变都可以用来存储二进制信息。简而言之，0和1并不是某种抽象概念，而是以某种物理现象客观存在着。在自然界中，A，C，G和U/T也对应着一些分子结构，存储在DNA（或RNA）的链状分子序列中。

在计算机中，我们将8个二进制字符称为一个字节，一个字节也就构成了储存和处理数据的基础单位。自然界则将3个核苷酸分组为一个密码子，该密码子则是蛋白质解码和加工的基础单位。每个密码子包含6位信息（每个DNA字符2位，每个密码子有3个字符也就是2<sup>3</sup>=6位信息。这意味着一个密码子可以表示2<sup>4</sup>=64个不同的信息）。

这样看来，生物界和计算机对数字信息的储存和处理如出一辙。大家可以自己去看[世界卫生组织发布的疫苗序列](https://mednet-communities.net/inn/db/media/docs/11889.doc)就可以感受到疫苗序列和计算机中的二进制序列的惊人的相似之处。

> 若果你想了解更多相关信息，原作者也在[这里](https://berthub.eu/articles/posts/what-is-life/)还有[这里](https://berthub.eu/dna/)提供了更多的背景知识。

## Pfizer/BioNTech疫苗”代码“到底想做些什么？

要理解这段疫苗“代码”的作用我们就要首先了解疫苗的大致工作原理。疫苗的主要作用就是帮助我们的免疫系统产生对付病原体的抗体，而又确保我们在接种的过程中不被病毒感染。传统的疫苗是通过注入弱化或灭活的病毒，再加上“疫苗佐剂”来刺激（惊吓）我们的免疫系统产生抗体来完成的。与生物体中DNA/RNA的”数字化“信息存储相比，这完全是一种模拟技术：研发和生产的过程需要用到数十亿个生物卵（传统工艺会用到鸡蛋）或昆虫。这种传统疫苗的研制还有很多运气的成分并花费大量时间，有时还需要使用一些其他的病毒来帮助产生所需的灭活病原体蛋白。

mRNA疫苗则是用一种不同的方式达到相同的目的。这种新的方式像激光一样，可以准确地产生所需的病原体特征蛋白，整个过程非常精准但也非常强大。

简单说来，这种mRNA疫苗的工作原理就是将描述了SARS-CoV-2“穗”蛋白（Spike protein）的mRNA序列注射到人体，再通过一些化学手段使这种遗传物质进入我们的某些细胞。然后，这些分子开始按照这些mRNA序列开始接管这些细胞并大量生产SARS-CoV-2穗蛋白，从而刺激我们的免疫系统生产对应的抗体。

这就是95％有效疫苗的工作流程。

## 源代码！

世界卫生组织提供了这样一个概括图片：

![疫苗序列概括](https://berthub.eu/articles/vaccine-toc.png)

这张图片就像是一个目录，对应着疫苗序列每一部分的功能。我们就从这个"cap"开始讲起。

