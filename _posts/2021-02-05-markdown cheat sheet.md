---
title: markdown cheat sheet
author: Zhang Ge
date: 2021-02-05 22:31:00 +0800
categories: [有趣/有用的资料, cheat sheets]
tags: [cheat sheets, 转载]
math: true
---

镇楼：没有什么比[Markdown官方教程](https://markdown.com.cn/)更懂Markdown.


# Markdown emoji

:point_right:See this github repo:  [ikatyang/emoji-cheat-sheet](https://github.com/ikatyang/emoji-cheat-sheet)

and this：[rxaviers/7360908](https://gist.github.com/rxaviers/7360908)

- 可以先到[这里](https://www.emojiall.com/zh-hans)搜关键词，找到图标，然后到[这里](https://gist.github.com/rxaviers/7360908)直接搜图标得到markdown代码。
- 有的emoji因为各种原因在目前的markdown中不支持，可以在[这里](https://emojipedia.org/)搜索，跳过markdown编码，直接复制表情。如这个:partying_face: 🥳



# 其他经验积累

## 为公式添加编号

在公式末尾加入`\tag{id}`，如` $$ x+y = z \tag{1} $$`可渲染为



$$
x + y = z \tag{1}
$$

## 显示github项目star数目

[参考](https://zhuanlan.zhihu.com/p/93132008)

类似于显示一张图片（或者本质就是），方法是：

`![GitHub stars](https://img.shields.io/github/stars/<usr_name>/<repo_name>.svg?style=flat&label=Star)`

比如[我的一个~~star少得可怜的~~github repo](https://github.com/ZhangGe6/sports_counting_by_pose_estimation)![GitHub stars](https://img.shields.io/github/stars/ZhangGe6/sports_counting_by_pose_estimation.svg?style=flat&label=Star)



