---
title: Emacs配置（二）-基本配置
date: 2020-08-04 21:30:17
tags: [Emacs配置, Workflow]
categories: Workflow
---



## Emacs 基本配置

在上一篇中，完成了在```mac```系统下的```Emacs```安装。现在拥有了一个UI非常原始的```Emacs```。在本节中，将规划配置目录结构，以方便配置文件的备份和共享。 

同时将引入包管理工具，以方便快速的安装后续所支持的插件。

再最后，利用包管理工具，尝试变更主题，以及默认全屏打开、默认显示文件行号等基础配置。



### 目录结构规划

从```Emacs 23```版本开始的，在启动时首先会尝试加载```~/.emacs.d/init.el```文件或```~/.emacs.d/init.elc```文件。利用这个特性，我们可以将目录结构规划如下：

<img src="emacs_config_dir.jpg" width="100%" height="100%" title="Emacs目录结构" alt="Emacs目录结构"/>

- init.el

  该文件为```Emacs```启动时首先加载的文件，在该文件内定义需要使用的组件以及通用配置

- lisp

  该目录存放自定义的```emacs-lisp```代码文件，用于实现自定义的效果。如键盘映射变更等。

- site-lisp

  该目录存放第三方插件。存放包管理工具中获取的依赖包。在```emacs版本 >=24```时，使用该目录，否则使用 ```elpa``` 目录

- auto-save-list

  该目录存放为自动存稿文件



### 通用配置

打开```init.el```文件。

#### step1

加入 ```(package-initialize)``` 这个必备项，该项必须放在所有安装包的配置项之前。

#### step2

设定额外包加载路径。按我个人的习惯而言，有两个: ```~/.emacs.d/lisp``` 和 ```~/.emacs.d/elpa.``` 前者放置自定义的```emacs-lisp```文件，后者为包管理器```elpa```所下载包所依赖的路径

当前配置代码如下

```lisp
;;; 初始化配置的入口

;; 设定lisp文件路径

;; Added by Package.el.  This must come before configurations of
;; installed packages.  Don't delete this line.  If you don't want it,
;; just comment it out by adding a semicolon to the start of the line.
;; You may delete these explanatory comments.
(package-initialize)

;; 添加lisp路径到启动加载路径
(add-to-list 'load-path (expand-file-name "lisp" user-emacs-directory))
;; 添加elpa路径到启动加载路径，该路径为包管理器elpa所使用
(add-to-list 'load-path (expand-file-name "elpa" user-emacs-directory))
```



使用 ```M-x load-file```命令重新加载 ```init.el```文件，使上述配置生效。如下图所示

<img src="emacs_common_config.jpg" width="100%" height="100%" title="目录结构" alt="目录结构"/>

### 包管理工具配置

包管理工具规划在启动项配置中。首先在```lisp```目录中新建一个文件 ```init-elpa.el```

代码如下 

```lisp
;; 包管理器设定

;; init elpa package
;; 根据版本不同，设定正确的package.el
(let ((package-el-site-lisp-dir
       (expand-file-name "site-lisp/package" user-emacs-directory)))
  (when (and (file-directory-p package-el-site-lisp-dir)
	     (> emacs-major-version 23))
    (message "Removing local package.el from load-path to avoid shadowing bundled version")
    (setq load-path (remove package-el-site-lisp-dir load-path))))

(require 'package)

;; 设定package源
(when (< emacs-major-version 24)
  (add-to-list 'package-archives '("marmalade" . "http://marmalade-repo.org/packages/")))

(add-to-list 'package-archives '("org" . "http://orgmode.org/elpa"))

(when (< emacs-major-version 24)
  (add-to-list 'package-archives '("gun" . "http://elpa.gnu.org/packages/")))

(add-to-list 'package-archives '("melpa" . "http://melpa.org/packages/"))

(add-to-list 'package-archives '("melpa-stable" . "http://stable.melpa.org/packages/"))

;; 设定gpg
(defun sanityinc/package-maybe-enable-signatures ()
  (setq package-check-signature (when (executable-find "gpg") 'allow-unsigned)))

(sanityinc/package-maybe-enable-signatures)
(after-load 'init-exec-path
	    (sanityinc/package-maybe-enable-signatures))

;; 请求第三方包，下载并安装
(defun require-package (package &optional min-version no-refresh)
  "Install given PACKAGE, optionally reqquiring MIN-VERSION.
if NO-REFRESH is non-nil, the available package lists will not b re-downloaded in order to locate PACKAGE."
  (if (package-installed-p package min-version)
      t
    (if (or (assoc package package-archive-contents) no-refresh)
	(package-install package)
      (progn
	(package-refresh-contents)
	(require-package package min-version t)))))

(defun maybe-require-package (package &optional min-version no-refresh)
  "Try to install PACKAGE, and return non-nil if successful.
In the event of failure, return nil and print a warning message.
Optionally require MIN-VERSION. if NO-REFRESH is non-nil, the available package lists will not be re-downloaded in order to locate PACKAGE."
  (condition-case err
      (require-package package min-version no-refresh)
    (error
     (message "Couldn't install package '%s': %S" package err)
     nil)))

(setq package-enable-startup nil)
(package-initialize)

;; 全屏
(require-package 'fullframe)
(fullframe list-packages quit-window)

;; 记载cl-lib
(require-package 'cl-lib)
(require 'cl-lib)

(defun sanityinc/set-tabulated-list-column-width (col-name width)
  "Set any column with name COL-NAME to the given WIDTH."
  (cl-loop for column across tabulated-list-format
	   when (string= col-name (car column))
	   do (setf (elt column 1) width)))

(defun sanityinc/maybe-widen-package-menu-columns ()
  "Widen some columns of the package menu table to avoid truncation."
  (when (boundp 'tabulated-list-format)
    (sanityinc/set-tabulated-list-column-width "Version" 13)
    (let ((longest-archive-name (apply 'max (mapcar 'length (mapcar 'car package-archives)))))
      (sanityinc/set-tabulated-list-column-width "Archive" longest-archive-name))))

;; 加载包名菜单
(add-hook 'package-menu-mode-hook 'sanityinc/maybe-widen-package-menu-columns)

(provide 'init-elpa)

```

在上述代码中，定义了不同版本兼容性的包下载和安装目录。以及配置了多个安装包下载源。

完成上述代码编写之后，只需要在 ```init.el```中引入该文件即可。如：```(require 'init-elpa)```

关于```elpa```包管理器的使用，可参考[文章](https://www.shellcodes.org/Emacs/使用ELPA管理Emacs扩展包.html)



#### Tips

TODO: ```init-elpa.el```依赖了 ```init-utils.el```中的某个自定义函数，若不在之前先加载```iint-utils.el``` 会导致加载失败，返回值为void。



### UI配置

当完成了上述配置后，可使用```color-theme```配置主题. 在```lisp```目录中新建一个文件 ```init-themes.el```

代码如下：

```lisp
;; 配置界面&UI

;; 老版本emacs兼容判断
(when (< emacs-major-version 24)
  (require-package 'color-theme))

(require-package 'color-theme-sanityinc-solarized)

;; 使用color-theme支持老版本的主题
(defcustom window-system-color-theme 'color-theme-sanityinc-solarized-dark
  "Color theme to use in window-system frames.
If Emacs' native theme support is available, this setting is ignored: use `custom-enabled-themes' instead."
  :type 'symbol)

(defcustom tty-color-theme 'color-theme-terminal
  "Color theme to use in TTY frames.
If Emacs' native theem support is available, this setting is ignored: use `custom-enabled-themes' instead."
  :type 'symbol)

(unless (boundp 'custom-enabled-themes)
  (defun color-theme-terminal ()
    (interactive)
    (color-theme-solarized-dark))
  (defun apply-best-color-theme-for-frame-type (frame)
    (with-selected-frame frame
      (funcall (if window-system
		   window-system-color-theme
		 tty-color-theme))))

  (defun reapply-color-themes ()
    (interactive)
    (mapcar 'apply-best-color-theme-for-frame-type (frame-list)))

  (set-variable 'color-theme-is-global nil)
  (add-hook 'after-make-frame-functions 'apply-best-color-theme-for-frame-type)
  (add-hook 'after-init-hook 'reapply-color-themes)
  (apply-best-color-theme-for-frame-type (selected-frame)))

;; 新版本主题支持
;; If you don't customize it, this is the theme you get
(setq-default custom-enabled-themes '(sanityinc-solarized-dark))

(defun reapply-themes ()
  "Forcibly load the themes listed in `custom-enabled-themes'."
  (dolist (theme custom-enabled-themes)
    (unless (custom-theme-p theme)
      (load-theme theme t)))
  (custom-set-variables `(custom-enabled-themes (quote ,custom-enabled-themes))))

(add-hook 'after-init-hook 'reapply-themes)

;; 切换深浅色主题
(defun light ()
  "Activate a light color theme."
  (interactive)
  (color-theme-sanityinc-solarized-light))

(defun dark ()
  "Activate a dark color theme."
  (interactive)
  (color-theme-sanityinc-solarized-dark))

(provide 'init-themes)
```

完成上述代码编写之后，只需要在 ```init.el```中引入该文件即可。如：```(require 'init-themes)```

接着，输入 ```M-x dark``` 可以看到效果如下：

<img src="emacs_theme_01.jpg" width="100%" height="100%" title="目录结构" alt="目录结构"/>



#### 字体配置

上述完成了主题的配置，并且自定义了两个快速切换亮色与暗色主题的函数 ```light() | dark() ```

不过可以看到，默认的字体和大小均有点太小了，尤其是对于我这种眼神不好的。接下来就改变他吧。 同时统一下文件的默认编码等。在```lisp```中新建文件```init-editing.el```文件

```lisp
;; 编辑属性设置

;; 字体大小设置
(set-default-font "-*-Menlo-normal-normal-normal-*-14-*-*-*-m-0-iso10646-1")

;; 全局开启行号显示
(global-linum-mode t) ;; 开启全局行号显示

;; 设置问答为简问答
(fset 'yes-or-no-p 'y-or-n-p)

;; 不生成备份文件
(setq make-backup-files nil)

;; 开启ido-mode
(setq ido-enable-flex-matching t)
(setq ido-everywhere t)
(ido-mode 1)

;; 设置默认的tab模式为空格，且为4格
(setq-default indent-tabs-mode t)
(setq default-tab-width 4)
(dolist (hook (list
	       'python-mode
	       'html-node
	       'js-mode
	       'erlang-mode
	       'go-mode
		   'c-mode
		   'c++-mode
	       ))
  (add-hook hook '(lambda () (setq indent-tabs-mode nil))))

;; 设置编码
(set-terminal-coding-system 'utf-8)
(set-keyboard-coding-system 'utf-8)
(set-language-environment "UTF-8")
(prefer-coding-system 'utf-8)

;; 删除选中模型
(delete-selection-mode)

;; 回车并缩进
(global-set-key (kbd "RET") 'newline-and-indent)

;; 设置kill-ring大小限制
(setq global-mark-ring-max 5000
	  mark-ring-max 5000
	  mode-require-final-newline t)
(setq kill-ring-max 5000
	  kill-whole-line t)

;; 在diff-mode中显示 whitespace
(add-hook 'diff-mode-hook (lambda ()
							(setq-local whitespace-style
										'(face
										  tabs
										  tab-mark
										  spaces
										  space-mark
										  trailing
										  indentation::space
										  indentation::tab
										  newline
										  newline-mark))
							(whitespace-mode 1)))

(provide 'init-editing)

```

完成上述代码编写之后，只需要在 ```init.el```中引入该文件即可。如：```(require 'init-editing)```

接着，输入 ```M-x load-file``` 可以看到效果如下：

<img src="emacs_theme_02.jpg" width="100%" height="100%" title="当前效果"/>



### 遇到的新问题以及解决方案

- 打开目录，提示"operation not permitted"错误

  在```mac os catalina```以上版本，对应用的权限访问做了更为严格的控制，需要手动开启。开启路径为：```系统偏好设置 -> 安全性与隐私 -> 完全磁盘访问权限``` 随后把应用添加进去即可。

  此处要注意的是，```mac```上的```Emacs```启动脚本为 ```/usr/bin/ruby```, 因此如果此处磁盘完全访问权限加入的是 ```emacs```这个App本身，仍旧还是会提示 "operation not permitted"错误。需要将 ```/usr/bin/ruby```加入进来才行。

  