# 自动加载机制

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

Tip: 如何开启测试自动加载机制？

> 修改includes/application_top.php文件的第75行
> ```
> define('DEBUG_AUTOLOAD', false);
> ```

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

