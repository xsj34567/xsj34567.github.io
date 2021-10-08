---
layout: post
title:  "开发工具之Markdown"
date:   2013-05-04 09:26:54
categories: Windows/Linux/Mac 工具
tags: Windows Linux Mac
updateTime: 2021-04-09
---

* content
{:toc}
系统常用工具之Markdown


## 文档工具

	熟练使用常用文档工具

### 一、Markdown 操作

	基本操作：标题、段落....

#### 1. 表格

	用途：便于比较各个组件差异、更明了...

```
|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |

```

设置表格的对齐方式：

-: 设置内容和标题栏居右对齐。
:- 设置内容和标题栏居左对齐。
:-: 设置内容和标题栏居中对齐。

```
| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |

-- 全部左对齐

| 左对齐 | 左对齐 | 左对齐 |
| :-----| :---- | :---- |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |

```

| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |

#### 2. 插入图片

````markdown
```
# 格式：相对路径（本地不可访问，线上可访问）
![2021-04-14_JVM数据运行时](\image\2021-04-14_JVM数据运行时.png)
​```

````



#### 3.  加粗、设置颜色

##### 3.1 **加粗**

```markdown
** 加粗 **
```

##### 3.2 <font color='red'>设置颜色</font>

```markdown
<font color='red'>设置颜色</font>
```



### 参考文档

[菜鸟教程Markdown](https://www.runoob.com/markdown/md-tutorial.html)

[Markdown高级技巧](https://www.runoob.com/markdown/md-advance.html)

[Markdown 中设置文本字体为红色（改变字体颜色）的方法](https://blog.csdn.net/COCO56/article/details/105155328/)

```haskell
成功=思维方式×热情×能力。
```

