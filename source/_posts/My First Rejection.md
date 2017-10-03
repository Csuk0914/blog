---
title: My First Rejection
date: 2017-9-18 22:00:00
categories: diary
---

今天吴老师通知我，LibHIR "has been rejected from publication on JMLR MLOSS Track"， 因为”unable to find any evidence of a significant community of external users“。虽然结果很让人沮丧，但平心而论，reviewer的理由简直让人没脾气。

回顾LibHIR的开发，CPP+STL的选型确实保证了跨平台，但与此同时，模型复杂性也导致暴露给使用者的细节过多。说句实在话，用起来连我自己都烦；为了兼容LibSVM和LibFM进行的妥协，也牺牲了模型为context feature提供的灵活性。但考虑到HIR是一个本身不复杂使用场景又有限的模型（提出HIR的师兄不要打我，motivation还是很好的），LibHIR未来大概只会用来做做对比实验所用，我选择了主动忽视这些。并且在写了不算诚心的doc，把项目放上GitHub之后([Repo of LibHIR](https://github.com/TagineerDai/LibHIR))，就完全没再迭代维护了。

Reviewer不谈易用性和可读性的硬伤，只很委婉的说”immature“，并给出原因”We have a policy not to publish projects that are only used by the developer themselves, because these often have issues that will only be ironed out after contact with real external users“，应该。。。还算口下留情？

Reviewer拒信里给了一个option，大概是先放上[mloss的官网](mloss.org)吸引人用，顺便迭代吧。老板也建议慢慢养肥再投，这个用户反馈/默默维护的路终究是绕不过去的。投稿就像求婚，被拒之后痛哭流涕没必要，但什么都不做也显得很不诚心不是嘛。

这之后大概会有几件事情：
+ CPP的版本保留，**给出几大开源框架中上打包好的版本**。前两天看李沐团队开入门课程，还不忘宣传Gluon，就觉得自己这种写完扔下的心态相当要不得，这不今天报应就来了。受他们两条路线教学的思路启发吧，至少Tensorflow啊pytorch啊mxnet这些框架（我可没说caffe和caffe2），应该给出可用的封装好的版本。
+ 定期有意识关注下Kaggle，**用LibHIR炼炼丹**。丰富sample，也放上论坛作为option。现在这种取易用性舍可读性的，复杂到狗都嫌的版本我自己都不想用，更不要说推荐给别人或直接用在应用里面了。
+ **改版文档**之后再把paper挂出来略做宣传。初步选型是[Sphinx](http://www.sphinx-doc.org/en/stable/) and [Read the Docs](https://readthedocs.org/),我搭网站做博客什么的总有点不顺手，上次就MarkDown写了份文档交差，看来绕来绕去还是绕不开。

我在做这个项目的时候，或多或少受了“哎呀这又不是我的idea随便做做”的想法的影响。然而，无论是因何而起的事情，做的不好，总是不能给人带来最终的快乐的。这几件事不为publication了，不着急，慢慢做，毕竟我还有漫长的申请季，博士生涯，博后生涯，博后生涯和博后生涯（开玩笑，希望不会有这么多）。

> 在一起　会有多美  
> 在一起　也会不美  
> 一个人　同偕到老  
> 不靠运气  
