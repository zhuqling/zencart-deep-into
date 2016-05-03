## 标准页面功能代码

以下总结了zencart中最常用的页面功能处理代码，供大家学习参考。

1. 页面处理标准代码之：加载语言控制文件

  ```php
  require(DIR_WS_MODULES . zen_get_module_directory('require_languages.php'));
  ```

  先检查 `includes/modules/<template>/` 有没有require_languages.php语言控制文件，否则使用 `includes/modules/require_languages.php` 

  需要注意，zencart语言文件不会自动加载，必须通过上述代码才能加载语言常量定义，require_languages.php会自动根据当前选择语言，加载语言总控制文件、全局语言定义文件、页面语言文件。

  关于语言文件的详细内容请查看“多语种的实现”之“语言文件的加载”。

2. 页面处理标准代码之：用户登陆限制

  ```php
  if (!$_SESSION['customer_id']) {
    $_SESSION['navigation']->set_snapshot(); // 保存当前页面快照，当登陆后自动返回当前页面
    zen_redirect(zen_href_link(FILENAME_LOGIN, '', 'SSL'));
  }
  ```

3. 删除当前页面快照

  ```php
  $_SESSION['navigation']->remove_current_page();
  ```

  页面快照通常用于在转向其它页面前记录当前页面，在转向后的页面里返回上次访问页面，最常见于访问需要登录的页面里，登录后可自动返回原页面。

4. 页面处理标准代码之：加入面包屑

  ```php
  $breadcrumb->add(NAVBAR_TITLE_1, zen_href_link(FILENAME_ACCOUNT, '', 'SSL'));
  $breadcrumb->add(NAVBAR_TITLE_2);
  ```

  面包屑主要用于内容的层次化导航，方便用户返回上级页面。

5. 页面处理标准代码之：购物车为空返回

  ```php
  if ($_SESSION['cart']->count_contents() <= 0) {
    zen_redirect(zen_href_link(FILENAME_SHOPPING_CART));
  }
  ```

6. 页面处理标准代码之：查询数据表

  ```php
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
  ```

7. 页面处理标准代码之：使用消息堆栈

  ```php
  $password_new = zen_db_prepare_input($_POST['password_new']);
  if (strlen($password_new) < ENTRY_PASSWORD_MIN_LENGTH) 
      $messageStack->add('account_password', ENTRY_PASSWORD_NEW_ERROR);
  ```

  消息堆栈是一种错误传递机制，通过堆栈可以在一个页面验证表单，在本页面或另一个页面显示错误消息。
