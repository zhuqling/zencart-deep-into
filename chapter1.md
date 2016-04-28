# 自动加载

通常情况下，无论是处理页面内容显示，还是响应表单提交，除非功能相当简单，否则我们都需要在处理实际功能之前初始化/加载一些必要的文件，这些文件含有类、函数等等，通过将通用功能类化/函数化可以缩减代码、增加开发效率，这个方法对于比较大型的项目是非常有用的。

zencart作为标准的电子商务售卖网站框架，为了支持电子商务特有的功能，加入了很多基础的类和函数，这些类和函数保证了人们利用zencart建立电子商务网站的正常工作，也为我们开发电子商务相关功能提供了良好的基础。

代码：加载数据库类，并初始化全局变量

```php
include('includes/db/mysql.php');
$db = new queryFactory();
```

这是一段典型的先加载类文件再初始化类实例的代码，因为电子商务需要很多特有的功能，所以zencart中就需要很多像上面这样的代码来初始化基础功能，这些功能包括语种、币种、购物车、数据库、消息堆栈、面包屑、缓存等等。因为功能多，而且几乎所有页面都需要这些功能，这就需要开发人员编写大量相似的代码，为了简化代码，优化这一过程，zencart中引入了自动加载机制。

利用自动加载机制，如上的代码可以写成以下配置代码，并将放置在includes/auto_loaders/目录下，zencart会自动加载配置并执行。

代码：自动加载配置示例config.test.php

```php
$autoLoadConfig[0][] = array('autoType'     => 'class',
                             'loadFile'     => 'mysql.php',
                             'classPath'    => 'includes/db/'
                            );
$autoLoadConfig[0][] = array('autoType'     => 'classInstantiate',
                             'className'    => 'queryFactory',
                             'objectName'   => 'db'
                            );
```

在代码第一行，当autoType参数指定为class时，表示需要加载类，loadFile参数指定文件名，classPath指定文件所在路径，这段代码就相当于上面的include('includes/db/mysql.php')的效果。

代码第二行，指定了autoType为classInstantiate，代表需要初始化类实例，className参数代表类名，objectName参数代表变量名，这段代码相当于执行了$db = new queryFactory()。

通过上面的示例，可以看到zencart通过查询设计好的配置参数，即可以完成加载文件和初始化类实例的过程。

如果你拥有多年编程经验，你是不是会发觉通过这样的设计，虽然没有了重复编写相似代码的工作量，转而取代为编辑配置参数，但zencart内置的代码需要遍历配置信息并完成加载工作，这样不是反而增加了PHP需要执行的代码量了吗？这样的自动加载机制到底有什么好处呢？

是的，按最原始的编码方式应该是最高效了，但zencart的自动加载机制有以下特点，使得效能上的少量降低就不是那么重要了。

1. 允许对加载和初始化步骤进行调试，可以有效避免重复加载。
2. 支持自动加载类、初始化类实例、执行实例方法、include/require文件、载入初始文件等功能。
3. 简化编写加载的初始化的代码，易扩展，使代码更清晰可读，不易出错。

简而言之，zencart自动加载机制的最大好处是允许对执行步骤（优先级）进行调试以及自动化执行多种操作。

如何开启测试自动加载机制？
修改includes/application_top.php文件的第75行
```define('DEBUG_AUTOLOAD', false);```

将DEBUG_AUTOLOAD常量定义为true，即可开启自动加载机制的测试输出。
开启测试后，所有页面的头部都会显示类似以下的输出：

```
actionPoint=>0 include('e:/jason/zencart/includes/classes/class.base.php');
actionPoint=>0 include('e:/jason/zencart/includes/classes/class.notifier.php');
actionPoint=>0 $zco_notifier = new notifier();
actionPoint=>0 include('e:/jason/zencart/includes/classes/class.phpmailer.php');
actionPoint=>0 include('e:/jason/zencart/includes/classes/class.smtp.php');
```

其中，actionPoint代表执行顺序号，即配置参数$autoLoadConfig数组的下标，后面紧跟的是实际执行的代码。

zencart自动加载机制支持的功能

1、以include方式包含文件
参数：
autoType = include
loadFile = 文件路径
等价代码：
```php
include($loadFile);
```

2、以require方式加载文件
参数：
autoType = require
loadFile = 文件路径
等价代码：
```php
require($loadFile);
```

3、加载初始脚本
参数：
autoType = init_script
loadFile = 脚本文件名
初始脚本位于includes/init_includes目录下，如果在includes/init_includes/overrides目录下有相同文件名存在，则执行overrides目录下的脚本。
等价代码：

```php
require('includes/init_includes/'.$loadFile); 或 require('includes/init_includes/overrides/'.$loadFile);
```

4、加载类文件
参数：
autoType = class
loadFile = 类文件名
classPath = 文件所在路径（可选）
当classPath未填时，系统会使用默认的类文件夹includes/classes中搜索。
等价代码：
1) 当classPath有值时，include($classPath.$loadFile);
2) 未设置classPath时，include('includes/classes/'.$loadFile);

5、初始类实例
参数：
autoType = classInstantiate
objectName = 实例变量名
className = 类名
classSession = 是否将变量保存到SESSION会话使其成为全局变量（可选），checkInstantiated = 创建变量前是否检查SESSION会话已经存在此变量，存在则不创建（可选，仅当classSession=true时有效）
等价代码：
1) 当未设置classSession时，$objectName = new className();
2) 当设置classSession = true，未设置checkInstantiated时，$_SESSION[$objectName] = new className();
3) 当classSession = true，checkInstantiated = true时，if(!isset($_SESSION[$objectName])) $_SESSION[$objectName] = new className();

6、执行变量方法
参数：
autoType = objectMethod
objectName = 变量名
methodName = 方法名
等价代码：
if(is_object($_SESSION[$objectName]))
  $_SESSION[$objectName]->methodName();
else
	$objectName->methodName();

表：自动加载参数，参数加方括号代码可选
功能	autoType值	参数	参数名称
以include方式包含文件	include	loadFile	文件名
以require方式加载文件	require	loadFile	文件名
加载初始脚本	init_script	loadFile	文件名
加载类文件	class	loadFile	文件名
		[classPath]	所在目录
初始类实例	classInstantiate	objectName	实例名
		className	类名
		[classSession]	是否SESSION类
		[checkInstantiated]	是否检查重复
执行变量方法	objectMethod	objectName	实例名
		methodName	方法名

自动加载配置文件
前面我们说到配置文件必须放置在includes/auto_loaders/目录下，这是因为zencart会扫描这个目录的所有文件，这个目录是自动加载配置文件专用文件夹。

但配置文件的文件名是不是可以随意填写呢？
答案是否定的，首先文件名是由以下几部分构成的：
<loadPrefix>.<userNamed>.php
其中第一部分<loadPrefix>是标准前缀，此值由全局变量$loadPrefix指定，默认值为config（在includes/application_top.php指定）。
第二部分<userNamed>为用户自定名称。
当<userNamed>为core时，该配置文件为核心配置文件，zencart会第一时间加载此配置文件，然后才加载其它文件。

我们接着来说一下$loadPrefix的作用，当我们在处理一些页面或程序时，这些页面都使用相同配置，我们不需要为每个页面分别编写配置，而是通常情况下会将配置分组，划分这些页面使用配置A，那些页面使用配置B，zencart中的$loadPrefix就是起到配置分组的作用。

zencart是以index.php文件为前端文件，其它所有页面都可以使用相同的配置，所以zencart使用config为$loadPrefix的默认值，我们只需要编写名称为config.xxx.php的配置文件就可以在几乎所有页面中加载此配置。

当我们新增了一个前端文件，这个文件需要的功能与标准zencart服务不太一样或者可以精简，需要使用非标准配置文件时，我们将在加载application_top.php文件前修改$loadPrefix的值，这样便可以更改配置组。

```php
$loadPrefix = 'myconfig';
require('includes/application_top.php');
```

经过修改后，我们的配置文件名也必须按myconfig.xxx.php的格式编写了。

自动加载机制原理

为了更好地了解zencart自动加载机制，我将以更简短的代码来展示这一功能zencart的程序流程。

1、前端文件，默认index.php
是否设置$loadPrefix可选。

2、程序初始
includes/application_top.php
a) 初始化$autoLoadConfig配置数组
b) 当未设置$loadPrefix时，设置默认值config
设置核心配置文件为$loadPrefix.core.php。

3、加载所有相关配置文件
includes/initsystem.php
a) 判断includes/auto_loaders/overriders是否重载了$loadPrefix.core.php核心配置文件，有则载入，否则加载includes/auto_loaders/下的核心配置文件。
b) 扫描includes/auto_loaders/下所有文件，过滤出符合文件名规则的文件，当overriders目录下有同名文件时，加载重载文件，否则加载auto_loaders目录下的配置文件。

4、按配置执行
includes/autoload_func.php
a) 重置$autoLoadConfig配置数组
排序。
b) 循环$autoLoadConfig数组，
按指定autoType类型执行相应
功能加载。 
c) 当DEBUG_AUTOLOAD = true
时，输出加载调试信息。


关于初始化脚本
当autoType = init_script时，zencart会加载初始化脚本，那么究竟什么是初始化脚本呢？我们不是已经有了自动初始化机制了吗，为什么还有一个初始化脚本呢？
我们知道自动加载机制能够自动执行的功能毕竟是有限的，如果要实现复杂的初始化还是得回复原始的代码编写方式，如何协调好自动加载与手写初始化这两种截然不同的方式，使得代码更好管理，zencart的作法是将手写初始化代码引入自动加载机制中，将其命名为“初始化脚本”的方式来实现统一协调的。
初始化脚本必须放置在includes/init_includes/目录下，当需要重载某个脚本时，可以将相同名称的文件放置在includes/init_includes/overrides/目录下。
zencart的标准初始化脚本包括了数据库连接初始化、语种初始化、币种初始化、SESSION会话初始化、模块初始化、购物车初始化等等。
