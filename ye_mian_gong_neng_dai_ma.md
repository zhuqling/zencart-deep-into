# 页面功能代码
所有页面都有特定的功能，通常编码人员都是针对页面开展针对性的工作，如登录页面需要处理响应用户表单的提交及验证是否通过、以及检查输入是否合法等，产品页面需要显示产品基础信息如标题、价格、描述等等内容。 
zencart的页面功能代码集中放置在includes/modules/page/中，每个功能页面再在其下建立相应文件夹，功能代码文件文件名必须以header_php开头，即必须为header_php***.php格式。
页面功能代码文件由index.php文件自动根据当前页面调用。
代码：index.php自动加载页面功能代码文件
  $directory_array = $template->get_template_part($code_page_directory, '/^header_php/');
  foreach ($directory_array as $value) { 
    require($code_page_directory . '/' . $value);
  }

代码中的$code_page_directory为全局变量，保存当前页面功能代码的文件夹路径，其值应为includes/modules/pages/<page>/。通常上面的代码我们可以看到，只要是以header_php开头，以php为后缀的文件都会被自动加载（模板类的get_template_part方法默认后缀名为php）。


Page Not Found页面未找到
zencart会自动检查请求当前页面是否拥有功能目录，功能目录默认为includes/modules/pages/<page>文件夹，如果此目录不存在，则被认定为Page Not Found。
当出现Page Not Found情况，会根据zencart后台设置出现两种情况。
1、当常量MISSING_PAGE_CHECK = On / True，系统会自动转向网站首页。
2、否则将显示PageNotFound功能页面。
Page Not Found判断逻辑代码在includes/init_includes/init_sanitize.php初始脚本里。

以下总结了zencart中最常用的页面功能处理代码，供大家学习参考。
页面处理标准代码
1、页面处理标准代码之：加载语言控制文件
require(DIR_WS_MODULES . zen_get_module_directory('require_languages.php'));
先检查includes/modules/<template>/有没有require_languages.php语言控制文件，否则使用includes/modules/require_languages.php
需要注意，zencart语言文件不会自动加载，必须通过上述代码才能加载语言常量定义，require_languages.php会自动根据当前选择语言，加载语言总控制文件、全局语言定义文件、页面语言文件，关于语言文件的详细内容请查看“多语种的实现”之“语言文件的加载”。

2、页面处理标准代码之：用户登陆限制
if (!$_SESSION['customer_id']) {
  $_SESSION['navigation']->set_snapshot(); // 保存当前页面快照，当登陆后自动返回当前页面
  zen_redirect(zen_href_link(FILENAME_LOGIN, '', 'SSL'));
}

3、删除当前页面快照
$_SESSION['navigation']->remove_current_page();
页面快照通常用于在转向其它页面前记录当前页面，在转向后的页面里返回上次访问页面，最常见于访问需要登录的页面里，登录后可自动返回原页面。

4、页面处理标准代码之：加入面包屑
$breadcrumb->add(NAVBAR_TITLE_1, zen_href_link(FILENAME_ACCOUNT, '', 'SSL'));
$breadcrumb->add(NAVBAR_TITLE_2);
面包屑主要用于内容的层次化导航，方便用户返回上级页面。

5、页面处理标准代码之：购物车为空返回
if ($_SESSION['cart']->count_contents() <= 0) {
  zen_redirect(zen_href_link(FILENAME_SHOPPING_CART));
}

6、页面处理标准代码之：查询数据表
$addresses_query = "SELECT *
                    FROM   " . TABLE_ADDRESS_BOOK . "
                    WHERE  customers_id = :customersID
                    ORDER BY firstname, lastname";

$addresses_query = $db->bindVars($addresses_query, ':customersID', $_SESSION['customer_id'], 'integer'); // 绑定参数
$addresses = $db->Execute($addresses_query);

while (!$addresses->EOF) { // 循环直到结束
  $format_id = zen_get_address_format_id($addresses->fields['country_id']);

  $addressArray[] = array('firstname'=>$addresses->fields['firstname'],
  'lastname'=>$addresses->fields['lastname'],
  'address_book_id'=>$addresses->fields['address_book_id'],
  'format_id'=>$format_id,
  'address'=>$addresses->fields);

  $addresses->MoveNext(); // 移动到下一条记录
}

7、页面处理标准代码之：使用消息堆栈
$password_new = zen_db_prepare_input($_POST['password_new']);
if (strlen($password_new) < ENTRY_PASSWORD_MIN_LENGTH) 
    $messageStack->add('account_password', ENTRY_PASSWORD_NEW_ERROR);
消息堆栈是一种错误传递机制，通过堆栈可以在一个页面验证表单，在本页面或另一个页面显示错误消息。

