# 主页面模板

页面BODY内容的控制由主页面模板加载开始，在index.php前端文件里调用。

```php
require($template->get_template_dir('tpl_main_page.php',DIR_WS_TEMPLATE, $current_page_base,'common'). '/tpl_main_page.php');
```

template模板类的方法get_template_dir会自动控制加载哪个目录，其优先级以下：

1. includes/templates/<template>/<page>/ 自定义模板目录下页面目录
2. includes/templates/template_default/<page>/ 默认模板目录下页面目录
3. includes/templates/<template>/common/ 自定义模板目录下通用目录
4. includes/templates/template_default/common/ 默认模板目录下通用目录

可以看出通过这样的优先级安排，自定义模板和单独页面的布局非常容易从默认模板中区分出来，这对于测试新模板或新页面布局，而不影响现有模板是非常有用的。

默认主页面模块是位于includes/templates/template_default/common/目录下的tpl_main_page.php文件。它是一个标准的三栏布局，主要分为头部、主内容、左边栏、右边栏、底部五大块区域。

1. 其中头部由tpl_header.php模板文件管理，由变量$flag_disable_header控制是否隐藏头部区域；
2. 主内容由require($body_code); 代码直接调用页面代码生成；
3. 左边栏由includes/modules/column_left.php模块加载，左边栏的布局在后台layout里控制，是否隐藏左边栏，由变量$flag_disable_left控制的。

```php
if (COLUMN_LEFT_STATUS == 0 || (CUSTOMERS_APPROVAL == '1' and $_SESSION['customer_id'] == '') || (CUSTOMERS_APPROVAL_AUTHORIZATION == 1 && CUSTOMERS_AUTHORIZATION_COLUMN_LEFT_OFF == 'true' and ($_SESSION['customers_authorization'] != 0 or $_SESSION['customer_id'] == ''))) {
  // global disable of column_left
  $flag_disable_left = true;
}
```

可以看出默认控制是否隐藏左边栏有以下几个因素：

    1. COLUMN_LEFT_STATUS（左边栏状态）常量=0，由网站后台操作；
    2. 当CUSTOMERS_APPROVAL（登陆限制）常量=1并且未登陆时；
    3. CUSTOMERS_APPROVAL_AUTHORIZATION（授权登陆）常量=1，并且CUSTOMERS_AUTHORIZATION_COLUMN_LEFT_OFF（授权下关闭左边栏）常量=true，且用户未登陆或未授权时；

4. 右边栏由includes/modules/column_right.php模块加载，右边栏的布局同样在后台layout可以控制。$flag_disable_right变量控制是否隐藏右边栏。

```php
if (in_array($current_page_base,explode(",",'list_pages_to_skip_all_right_sideboxes_on_here,separated_by_commas,and_no_spaces')) ) {
    $flag_disable_right = true;
  }
```

zencart默认代码预留了隐藏右边栏的功能，我们只需要修改上面的代码，将explode函数后面的第二个参数里增加需要隐藏右边栏页面的名称即可，多个页面名称以逗号或空格分隔。

5. 底部区域由tpl_footer.php模板文件管理，由变量$flag_disable_footer控制是否隐藏底部区域。


