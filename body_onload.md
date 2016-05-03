# Body onload 代码

Body onload代码是在页面加载完成时自动执行的JS代码，通常用于初始化页面元素。onload代码的文件名格式为on_load_***.js，它可以放置在以下几个目录下：

* includes/modules/pages/<page>/
* includes/templates/<template>/jscript/on_load/
* includes/templates/template_default/jscript/on_load/

凡是在这些目录中的onload代码，会自动被合并，并在tpl_main_page.php里显示出来。

```php
<body id="<?php echo $body_id . 'Body'; ?>"<?php if($zv_onload !='') echo ' onload="'.$zv_onload.'"'; ?>>
```

其中$zv_onload保存的是onload代码。$body_id为当前页面名称。
