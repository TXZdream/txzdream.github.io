---
title: 系分项目个人总结
date: 2018-07-01 20:31:25
tags:
- SAAD
- 系统分析与设计
---
## 个人总结

1. 环境部署

  将产品容器化是一个非常好的选择，虽然开头可能会花时间去写文件，但是一劳永逸；CI同样如此，没有什么可以比只写代码更令人舒服的了

2. 后台架构

  后台一定要分层，个人认为最好的实践就是垂直去分层，每一层只能使用紧邻层的服务，而不要跨层调用，这会给维护带来意想不到的好处

3. 管理

  作为小组长，如果团队人数过多，千万不要想着把权利放自己手上，一定要分权，下层对上层负责，就像后台分层一样，但是上层要拥有绝对话语权

4. 开发

  开发大部分时候是枯燥的，没有想象中的和算法打交道绞尽脑汁那么愉悦，很多时候你只是在和数据库打交道，处理一堆莫名其妙的问题，所以一定不要想一天写完所有，那样会心态爆炸的，而是要分阶段执行，每天完成一部分任务，保证质量

总结：以上要说的，就在于一个字——**分**，各司其职，做好每一件最简单的事情才最不简单；懂得协作，一人打天下的时代过去了；注重理论，软件工程已经不是创造了，而是严谨的，按部就班的工程，你想要的快乐，也在这一要求下生存

## PSP 2.1

### Planing
  
  预计由4次迭代组成，每次迭代预计花费时间为2-3周，总时间为2月

### Development

  - Analysis
    2 iter, 2 days per iter
  - Design Spec
    2 iter, 3 days per iter
  - Design Review
    2 iter, 1 day per iter
  - Coding Standard
    3 days
  - Design
    each iter, 2 days per iter
  - Coding
    each iter, 3 days per iter
  - Code Review
    each iter, 1 days per iter
  - Test
    each iter, 2 days per iter

### Reporting

  - Test Report
    none
  - Size Measurement
    2 days
  - Postmortem & Process Improvement Plan
    2 days

## git统计报告

### Document

![](images/saad-all-1.png)

### Service-end

![](images/saad-all-1.png)

### Script

![](images/saad-all-1.png)

## 工作清单

1. 服务端整体架构，和组内成员完成docker-compose的构建和部署
1. 框架和语言的选择，后台语言和框架的选型
1. CI，即与github、docker之间的深度集成

## 博客清单

[travis与docker hub集成](https://txzdream.github.io/2018/06/07/travis-docker-hub/)

[系分作业第一次总结](https://txzdream.github.io/2018/04/15/SAAD-summary-iter1/)