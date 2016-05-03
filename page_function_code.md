# 页面功能代码

所有页面都有特定的功能，通常编码人员都是针对页面开展针对性的工作，如登录页面需要处理响应用户表单的提交及验证是否通过、以及检查输入是否合法等，产品页面需要显示产品基础信息如标题、价格、描述等等内容。 

zencart的页面功能代码集中放置在includes/modules/page/中，每个功能页面再在其下建立相应文件夹，功能代码文件文件名必须以header_php开头，即必须为header_php***.php格式。

页面功能代码文件由index.php文件自动根据当前页面调用。

代码：index.php自动加载页面功能代码文件
```php
  $directory_array = $template->get_template_part($code_page_directory, '/^header_php/');
  foreach ($directory_array as $value) { 
    require($code_page_directory . '/' . $value);
  }
```

代码中的$code_page_directory为全局变量，保存当前页面功能代码的文件夹路径，其值应为includes/modules/pages/<page>/。通常上面的代码我们可以看到，只要是以header_php开头，以php为后缀的文件都会被自动加载（模板类的get_template_part方法默认后缀名为php）。


## Page Not Found 页面未找到

> zencart会自动检查请求当前页面是否拥有功能目录，功能目录默认为includes/modules/pages/<page>文件夹，如果此目录不存在，则被认定为Page Not Found。
> 
> 当出现Page Not Found情况，会根据zencart后台设置出现两种情况。
> 
> 1. 当常量MISSING_PAGE_CHECK = On / True，系统会自动转向网站首页。
> 2. 否则将显示PageNotFound功能页面。
> 
> Page Not Found判断逻辑代码在includes/init_includes/init_sanitize.php初始脚本里。

