---
title: Emacs配置（三）-Markdown
date: 2020-08-08 21:20:44
tags: [Emacs配置, Workflow]
categories: Workflow
---

## Emacs Markdown支持

在上一篇中，完成了基本操作以及主题的支持。 目前工作上更常用的场景是使用```Markdown```编写一些技术预研方案和博客。因此，对于```markdown```的支持提上了第一优先级。

另外，由于编写博客采用的是```hexo```这个框架，而```Emacs```对于原生```shell```的支持并不是特别好，因此，也在本文中顺带解决。

第三个要解决的点是对于```git```和项目内文件快速搜索的支持。（类似于```mac os```下面很多IDE工具支持的 ```cmd+shift+o``` 快速检索项目内所有文件一样）

### Markdown的支持

对于markdown的支持，最基本的一点需要对```markdown```语法的支持。其次，需要支持实时预览。然后还需要一些基本语法的自动补全以及常用的```snippet```，尤其是表格。 ```markdown```本身对于表格的支持并不是特别好，因此需要编辑器作出支持。

另外，经常需要将markdown导出为```word```或者```pdf```等格式。此处也需要引进```Pandoc```这把格式转换的"瑞士军刀"。关于```Pandoc```的详细介绍可以参考[官网](https://pandoc.org/index.html)

- step1 - markdown语法支持

  在```.emacs.d/lisp```目录中新建文件```init-markdown.el```文件。依赖安装包```markdown-mode```。代码如下：

  ```lisp
  
  ;;-------------------------
  ;; 配置markdown语法支持
  ;;-------------------------
  (require-package 'markdown-mode) ;; 依赖markdown-mode包
  (add-to-list 'auto-mode-alist '("\\.md\\'" . markdown-mode))
  (add-to-list 'auto-mode-alist '("\\.markdown\\'" . markdown-mode))
  
  (provide 'init-markdown)
  ```
  
  完成上述代码后，执行命令```M-x load-file```重新加载```init.el```。即可发现，目前```Emacs```已经支持```Markdown```
  
  <img src="01.jpg" alt="markdown语法支持" title="markdown语法支持" />
  
- step2 - markdown实时预览支持
  
  仍旧在```init-markdown.el```文件中,依赖安装包```markdown-preview-mode```。代码如下：
  
  ```lisp
  (require-package 'markdown-preview-mode)
  ```
  
  Tips: ```markdown-preview-mode```依赖外部工具，需要在终端中执行命令```gem install redcarpet```进行安装，若遇到权限问题，需要使用```sudo```进行提权

  问题: 目前的方式只能在浏览器中预览。 需要实现的方式为在左右两边分屏实现。 昨天为```markdown```原文，右边为实时预览界面。需要再找其他解决方案。
	
- step3 - 自动补全&snippet支持
  
  常用的命令快捷键如下:
  
  ``` javascript
  C-c C-t n: 插入标题, # title # 类型其中 n 为 1 到 6, 对应于 HTML 对应的标题.
  C-c C-t t: 插入标题, underline 类型, =.
  C-c C-t s: 插入标题, underline 类型, -.
  C-c -: 插入水平线, 默认插入设定的每行字节数的减号.
  
  C-c c-a L: 插入引用链接, 或者 C-c C-a r, [text][label].
  C-c C-a l: 插入链接, [text](url).
  C-c C-a f: 插入注脚.

  C-c C-i i: 插入图像, 或者 C-c TAB i, ![text](url).
  C-c C-i I: 插入引用图像, 或者 C-c TAB I, ![text][label].
  C-c C-s b: 插入引用, >.
  
  C-c C-s c: 插入代码, 用撇号包围的.
  C-c C-s s: 粗体.
  C-c C-p e: 意大利体.
  
  C-c C-x d: 向下移动列表.
  C-c C-x l: 提升列表等级, 即减少空格.
  C-c C-x m: 插入列表项.
  C-c C-x r: 降低列表等级, 即增加空格.
  C-c C-x u: 向上移动列表.  
  ```

### Shell支持

```Emacs```原生支持```shell```，使用命令```M-x shell```即可打开原生的```shell```窗口。但是存在一些问题，比如，对于中文和```powerline```类型字体支持并不好。那么在这里，采用```eshell```来解决这些问题。

执行命令```eshell```即可进入到```eshell```模式。该```shell```对于中文支持没问题，不过对比```zsh```或者```oh-my-zsh```，还是差了不少功能。 

<img src="02.jpg" alt="eshell效果" title="eshell效果" width="100%" height="100%" />
