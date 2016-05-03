## 语言文件的加载

zencart中的语言文件不是自动加载的，必须由页面功能代码里手工添加以下代码：

```php
require(DIR_WS_MODULES . zen_get_module_directory('require_languages.php'));
```

通过require_languages.php语言控制文件来控制如何加载语言文件。

语言文件主要分成三种类型：语言总控制文件、全局语言定义文件、页面语言文件。我们介绍一下这三种语言文件的含义及其加载顺序。

1. 语言总控制文件，优先级1级，此语言文件最先被加载，一般为 `includes/languages/<language>.php` 文件。该文件主要是针对语言级的格式及规范汇总，如日期时间格式、语言默认币种、称谓、基础名词的翻译等等。
	语言总控制定义的详细内容请看“附录之语言总控制定义明细表”

2. 全局语言定义文件，优先级2级，全局语言定义文件不只是单个文件，而一组文件，它们是由 `includes/languages/<language>.php` 加载的。

	以下为几个基本全局语言定义文件，这些文件加载所检查的目录都是相同的，首先检查 `includes/languages/<language>/<template>/` 模板目录下是否含有文件，有则加载，否则加载 `includes/languages/<language>/` 目录下的对应文件。

		* header.php 用于页面头部的常用功能定义
		* email_extras.php 邮件相关的内容
		* button_names.php 定义按钮名称及文件名
		* icon_names.php 图标名称及文件名
		* other_images_names.php 其它图片名称及文件名
		* credit_cards.php 信用卡
		* whos_online.php 在线
		* meta_tags.php META和Tag相关

3. 页面语言文件，优先级3级，页面语言文件是最后被加载的。页面语言文件主要是针对当前页面定义相应语言常量。页面语言文件的加载分两种情况：

	1. 当`<template>`模板目录下有符合的页面语言文件时，先加载 `includes/languages/<language>/<template>/<page>***.php` 分模板语言文件，再加载 `includes/languages/<language>/<page>***.php` 页面语言文件，主语言会覆盖分模板语言中相同的定义。
	2. 当 `<template>` 目录下无符合的语言文件时，直接加载 `includes/languages/<language>/<page>***.php` 页面语言文件。

	大家应该已经注意到页面语言文件的文件名为 `<page>***.php` 格式，只要以页面名称开头，以php为后缀的都被应为是当前页面语言文件。笔者在此认为这应该是zencart的一个bug，是否应该加载所有以 `<page>` 开头的语言文件，值得商榷。
