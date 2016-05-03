# Meta控制与显示

Meta相关文件

1. 语言文件，由includes/languages/<language>.php自动载入

- includes/languages/<language>/<template>/meta_tags.php
- includes/languages/<language>/meta_tags.php

2. 内容控制

includes/modules/meta_tags.php

3. 页面模板，由模板牵头调用（2）内容控制，再显示出来

- includes/templates/<template>/common/html_header.php
- includes/templates/common/html_header.php

```php
require(DIR_WS_MODULES . zen_get_module_directory('meta_tags.php')); 
?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" <?php echo HTML_PARAMS; ?>>
<head>
<title><?php echo META_TAG_TITLE; ?></title>
<meta http-equiv="Content-Type" content="text/html; charset=<?php echo CHARSET; ?>" />
<meta name="keywords" content="<?php echo META_TAG_KEYWORDS; ?>" />
<meta name="description" content="<?php echo META_TAG_DESCRIPTION; ?>" />
```
