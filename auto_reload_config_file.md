## 自动加载配置文件

前面我们说到配置文件必须放置在includes/auto_loaders/目录下，这是因为zencart会扫描这个目录的所有文件，这个目录是自动加载配置文件专用文件夹。

但配置文件的文件名是不是可以随意填写呢？

答案是否定的，首先文件名是由以下几部分构成的：

```
<loadPrefix>.<userNamed>.php
```

其中第一部分<loadPrefix>是标准前缀，此值由全局变量$loadPrefix指定，默认值为config（在includes/application_top.php指定）。

第二部分 `<userNamed>` 为用户自定名称。

当 `<userNamed>` 为core时，该配置文件为核心配置文件，zencart会第一时间加载此配置文件，然后才加载其它文件。

我们接着来说一下$loadPrefix的作用，当我们在处理一些页面或程序时，这些页面都使用相同配置，我们不需要为每个页面分别编写配置，而是通常情况下会将配置分组，划分这些页面使用配置A，那些页面使用配置B，zencart中的$loadPrefix就是起到配置分组的作用。

zencart是以index.php文件为前端文件，其它所有页面都可以使用相同的配置，所以zencart使用config为$loadPrefix的默认值，我们只需要编写名称为config.xxx.php的配置文件就可以在几乎所有页面中加载此配置。

当我们新增了一个前端文件，这个文件需要的功能与标准zencart服务不太一样或者可以精简，需要使用非标准配置文件时，我们将在加载application_top.php文件前修改$loadPrefix的值，这样便可以更改配置组。

```php
$loadPrefix = 'myconfig';
require('includes/application_top.php');
```

经过修改后，我们的配置文件名也必须按myconfig.xxx.php的格式编写了。