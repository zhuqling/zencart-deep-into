## 自动加载机制原理

为了更好地了解zencart自动加载机制，我将以更简短的代码来展示这一功能zencart的程序流程。

1、前端文件，默认index.php

```php
$loaderPrefix = 'paypal_ipn';
require('includes/application_top.php');
```

> 是否设置$loadPrefix可选。

2、程序初始，includes/application_top.php

```php
$autoLoadConfig = array();
if (isset($loaderPrefix)) {
   $loaderPrefix = preg_replace('/[a-z_]^/', '', $loaderPrefix);
} else {
   $loaderPrefix = 'config';
}
$loader_file = $loaderPrefix . '.core.php';
require('includes/initsystem.php');
require('includes/autoload_func.php');
```

> 1. 初始化$autoLoadConfig配置数组
> 2. 当未设置$loadPrefix时，设置默认值config

设置核心配置文件为$loadPrefix.core.php。

3、加载所有相关配置文件，includes/initsystem.php

```php
$base_dir = DIR_WS_INCLUDES . 'auto_loaders/';
if (file_exists(DIR_WS_INCLUDES . 'auto_loaders/overrides/' . $loader_file)) {
  $base_dir = DIR_WS_INCLUDES . 'auto_loaders/overrides/';
}
include($base_dir . $loader_file);
if ($loader_dir = dir(DIR_WS_INCLUDES . 'auto_loaders')) {
  while ($loader_file = $loader_dir->read()) {
    $matchPattern = '/^' . $loaderPrefix . '\./';
    if ((preg_match($matchPattern, $loader_file) > 0) && (preg_match('/\.php$/', $loader_file) > 0)) {
      if ($loader_file != $loaderPrefix . '.core.php') {
        $base_dir = DIR_WS_INCLUDES . 'auto_loaders/';
        if (file_exists(DIR_WS_INCLUDES . 'auto_loaders/overrides/' . $loader_file)) {
          $base_dir = DIR_WS_INCLUDES . 'auto_loaders/overrides/';
        }
        include($base_dir . $loader_file);
      }
    }
  }
  $loader_dir->close();
}
```

> 1. 判断includes/auto_loaders/overriders是否重载了$loadPrefix.core.php核心配置文件，有则载入，否则加载includes/auto_loaders/下的核心配置文件。
> 2. 扫描includes/auto_loaders/下所有文件，过滤出符合文件名规则的文件，当overriders目录下有同名文件时，加载重载文件，否则加载auto_loaders目录下的配置文件。

4、按配置执行, includes/autoload_func.php

```php
reset($autoLoadConfig);
ksort($autoLoadConfig);
foreach ($autoLoadConfig as $actionPoint => $row) {
  $debugOutput = "";
  foreach($row as $entry) {
    $debugOutput = 'actionPoint=>'.$actionPoint . ' ';
    switch($entry['autoType']) {
      case 'include':
      break;
      case 'require':
      break;
      ......
    }
    if (DEBUG_AUTOLOAD === true) echo $debugOutput;
  }
}
```

> 1. 重置$autoLoadConfig配置数组排序。
> 2. 循环$autoLoadConfig数组，按指定autoType类型执行相应功能加载。 
> 3. 当DEBUG_AUTOLOAD = true时，输出加载调试信息。
