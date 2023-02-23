---
title: test3
categories:
  - - 分类1
  - - 分类2
tags:
  - 标签1
  - 标签2
date: 2023-02-22 17:49:50
---
# 我的需求：
+ 项目中遇到的`什么Bug`需要解决，遇到的`哪一类技术难点`需要攻克<font color="#C0C4CC">[这里写一些为啥要整理这部分笔记，整理博客的前因后果]</font>

# <font color="#F56C6C">我需要解决的问题：</font>
- 实现业务和流程的解耦等<font color="#C0C4CC">[这里写一我的需求bug需要解决什么问题]</font>
  - 问题一
  - 问题二

# 我是这样做的：
* `百度`寻找解决办法，向公司大佬`请教`。
* 在构建上基于某设计模式，利用了什么特性，结合了什么思想等。
* <font color="#C0C4CC">[这里大概描述一下我解决问题的步骤，方式]</font>

**你存在的意义，诠释了我仓促青春里的爱情水。
这里写一句鼓励自己的话,可以加一个颜色(可选)]

<!--more-->

***
<font color="#C0C4CC">博客内容</font>

# 一级标题
代码
``` java
public static MultipartFile getMultipartFile(InputStream inputStream, String fileName) {
    FileItem fileItem = createFileItem(inputStream, fileName);
    //CommonsMultipartFile是feign对multipartFile的封装，但是要FileItem类对象
    return new CommonsMultipartFile(fileItem);
}
```
## 二级标题
### 三级标题
#### 四级标题
##### <font color="#C0C4CC">五级标题</font>
###### 六级标题

`内容文字颜色设置`
<font color="#000000">具体的正文</font>
<font color="#303133">具体的正文</font>
<font color="#606266">具体的正文</font>
<font color="#909399">具体的正文</font>
<font color="#C0C4CC">具体的正文</font>

`重点标记性文字设置`
**<font color="#409EFF">具体的标记性正文</font>**
**<font color="#67C23A">具体的标记性正文</font>**
**<font color="#E6A23C">具体的标记性正文</font>**
**<font color="#F56C6C">具体的标记性正文</font>**
**<font color="#009688">具体的标记性正文</font>**
**<font color="##00C5CD">具体的标记性正文</font>**