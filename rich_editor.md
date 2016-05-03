## 富文本编辑器

### 添加ckeditor

下载ckeditor，地址：http://ckeditor.com/download 

下载插件：http://www.zen-cart.com/index.php?main_page=product_contrib_info&products_id=1626

将下载的ckeditor解压后，可以看到有两个目录admin和editors

将admin目录下的所有文件复制到管理后台目录，将editors目录下的ckeditor文件夹复制到editors目录下，当编辑器有更新时只需要更新editors/ckeditor目录下的文件即可。

进入后台configuration -> My store -> HTML Editor编辑选择新的编辑器，进行Category -> Categories/Products 下拉框TextEditor选择ckeditor。

进入产品编辑即可看到已经可以使用新的编辑器了。

includes/init_includes/overriders/init_html_editor.php文件可以不复制，直接修正zencart一处Bug, 在includes/init_includes/init_html_editor.php第一行：

```php
if (!defined('DIR_WS_EDITORS')) define('DIR_WS_EDITORS', 'editors');
```
改为

```php
if (!defined('DIR_WS_EDITORS')) define('DIR_WS_EDITORS', 'editors/');
```

### tinyMCE编辑器

下载tinyMCE，地址：http://www.tinymce.com/download/download.php 

下载插件：http://www.zen-cart.com/index.php?main_page=product_contrib_info&products_id=269

需下载jQuery版本源文件，解压至editors目录，即cheditor放置在editors/cheditor目录，tinyMCE放置在editors/tiny_mce目录下，zencart已经内置这些编辑器的支持代码，可以先测试使用像这样的网址：http://localhost/web/editors/ckeditor/_samples/index.html或者http://localhost/web/editors/tiny_mce/examples/index.html，

修改文件：includes/languages/<language>/extra_definitions/editors_list.php
添加：

```php
if (!defined('EDITOR_HTMLAREA')) define('EDITOR_HTMLAREA', 'HTMLArea');
if (!defined('EDITOR_CKEDITOR')) define('EDITOR_CKEDITOR', 'CKEditor');
if (!defined('EDITOR_TINYMCE')) define('EDITOR_TINYMCE', 'tinyMCE');
```
