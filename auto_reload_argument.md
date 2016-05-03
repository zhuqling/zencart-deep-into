## zencart自动加载机制支持的功能

1、以include方式包含文件

参数：

- autoType = include
- loadFile = 文件路径

等价代码：

```php
include($loadFile);
```

2、以require方式加载文件

参数：

- autoType = require
- loadFile = 文件路径

等价代码：

```php
require($loadFile);
```

3、加载初始脚本

参数：

- autoType = init_script
- loadFile = 脚本文件名

初始脚本位于includes/init_includes目录下，如果在includes/init_includes/overrides目录下有相同文件名存在，则执行overrides目录下的脚本。

等价代码：

```php
require('includes/init_includes/'.$loadFile);
或
require('includes/init_includes/overrides/'.$loadFile);
```

4、加载类文件

参数：

- autoType = class
- loadFile = 类文件名
- classPath = 文件所在路径（可选）

当classPath未填时，系统会使用默认的类文件夹includes/classes中搜索。

等价代码：

1. 当classPath有值时，`include($classPath.$loadFile);`
2. 未设置classPath时，`include('includes/classes/'.$loadFile);`

5、初始类实例

参数：

- autoType = classInstantiate
- objectName = 实例变量名
- className = 类名
- classSession = 是否将变量保存到SESSION会话使其成为全局变量（可选），
- checkInstantiated = 创建变量前是否检查SESSION会话已经存在此变量，存在则不创建（可选，仅当classSession=true时有效）

等价代码：

1. 当未设置classSession时，`$objectName = new className();`
2. 当设置classSession = true，未设置checkInstantiated时，`$_SESSION[$objectName] = new className();`
3. 当classSession = true，checkInstantiated = true时，`if(!isset($_SESSION[$objectName])) $_SESSION[$objectName] = new className();`

6、执行变量方法

参数：

- autoType = objectMethod
- objectName = 变量名
- methodName = 方法名

等价代码：

```php
if(is_object($_SESSION[$objectName]))
  $_SESSION[$objectName]->methodName();
else
	$objectName->methodName();
```

### 表：自动加载参数

| 功能 | autoType值 | 参数 | 参数名称 |
|---|---|---|---|
| 以include方式包含文件 | include | loadFile | 文件名 |
| 以require方式加载文件 | require | loadFile | 文件名 |
| 加载初始脚本 | init_script | loadFile | 文件名 |
| 加载类文件 | class | loadFile | 文件名 |
|  |  | [classPath] | 所在目录 |
| 初始类实例 | classInstantiate | objectName | 实例名 |
|  |  | className | 类名 |
|  |  | [classSession] | 是否SESSION类 |
|  |  | [checkInstantiated] | 是否检查重复 |
| 执行变量方法 |  | objectMethod | objectName	实例名 |
|  |  | methodName | 方法名 |

> 备注：参数加方括号代码可选
