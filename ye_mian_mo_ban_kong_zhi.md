# 页面模板控制

zencart中页面需要显示的模板可以自动控制，也就是说一个页面可以自行控制切换显示几个模板，或者直接输出内容。这种灵活的页面模板控制机制可以让我们实现例如这样的例子：我们搜索产品时，当搜索结果存在时控制显示一个模板，当未搜索到任何产品时显示另一个模板。这样便大大方便了模板的功能区分，非常利于模板的管理的修改。
页面模板控制权的转移有以下步骤：
A) index.php前台文件加载主模板控制文件
require($template->get_template_dir('main_template_vars.php',DIR_WS_TEMPLATE, $current_page_base,'common'). '/main_template_vars.php');

默认会加载includes/templates/common/main_template_vars.php主模板控制文件，默认模板控制文件会自动判断继续传导加载其它模板控制文件或者直接加载模板文件。当然主模板控制文件可以被重载（由模板类和get_template_dir方法自动实现），主模板控制文件优先级是：
1) 模板页面目录 includes/templates/<template>/<page>/main_template_vars.php 
2) 默认模板页面目录 includes/templates/template_default/<page>/main_template_vars.php 
3) 模板通用目录 includes/templates/<template>/common/main_template_vars.php 
4) 默认模板通用目录 includes/templates/template_default/common/main_template_vars.php 

B) 默认主模板控制文件
因为非默认主模板控制文件只是针对特殊情况，并且几乎不使用，所以在这里只介绍默认主模板控制文件。
默认主模板控制文件会首先判断includes/modules/<page>/目录是否存在main_template_vars.php模板控制文件，如果存在则加载，否则直接加载includes/templates/<template>/templates/目录或includes/templates/template_default/templates/目录下的tpl_<page>_default.php模板文件。

实际上，默认主模板并没有直接加载其它页面模板控制文件或模板文件，确切的是主模板文件会定义变量$body_code等于需要加载的文件名，然后再由tpl_main_page.php模板利用该变量加载对应的文件。

C) 页面模板控制文件
主模板控制文件将控制权移交给页面模板控制文件时，页面模板控制文件便拥有了最为灵活了页面控制能力。它将决定加载相应模板还是直接输出，在这里如果是继续加载模板文件，那么一般模板文件将被命名为tpl_<page>_***.php，通常文件避免命名为tpl_<page>_default.php，以免与标准模板文件名混淆。
页面模板控制文件的作用有：
	在加载模板之前设置模板参数
	定义不同模板，按需加载
	处理功能直接输出，无需模板介入

示例：加载可变模板
require($template->get_template_dir($tpl_page_body, DIR_WS_TEMPLATE, $current_page_base,'templates'). '/' . $tpl_page_body);

示例：加载标准模板文件
require($template->get_template_dir('tpl_specials_default.php',DIR_WS_TEMPLATE, $current_page_base,'templates'). '/tpl_specials_default.php');


D) 标准模板文件
标准模板文件位于includes/template/<template>/templates/目录或includes/template/template_default/templates/目录，文件名为tpl_<page>_default.php，模板文件可以再加载代码、模块或其它模板。
