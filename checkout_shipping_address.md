## 更改发货地址 checkout_shipping_address

更改发货地址预处理代码中，与前面的确认发货方式一样需要处理用户是否已登陆、购物车是否有产品等常规判断，在加载了当前页面的语言文件后，我们还需要判断物品类型是否为虚拟物品，如果是虚拟物品就不需要收货地址，直接转向到确认付款环节。

这里通过将新建地址的部分分离到单独的模块，增加这一功能的重用性。

```php
$addressType = "shipto";
require(DIR_WS_MODULES . zen_get_module_directory('checkout_new_address'));
```

$addressType在checkout_new_address模块中使用，当值为shipto时，表示是发货地址，当值为billto时，表示是账单地址。模块会根据该值自动设置$_SESSION['sendto']还是$_SESSION['billto']的值。

当用户填写新的发货地址并提交后，checkout_new_address模块会检查相关数据的有效性，如果所有数据都正确时，会保存地址信息到用户的名下，将根据$addressType的值，自动设置$_SESSION会话，然后转向到相应的发货或付款环节。

### 当前用户地址簿数量

```php
$addresses_count = zen_count_customer_address_book_entries();
```

选择发货地址涉及到了三个模板文件，分别是默认页面模板文件tpl_checkout_shipping_address_default.php、多地址选择模板tpl_modules_checkout_address_book.php、新添加地址模板tpl_modules_checkout_new_address.php。

其中多地址选择模板的数据源来自模块checkout_address_book.php。

### 控制国家、州和省份的选择

```php
  $selected_country = (isset($_POST['zone_country_id']) && $_POST['zone_country_id'] != '') ? $country : SHOW_CREATE_ACCOUNT_DEFAULT_COUNTRY; // 默认选择的国家
  $flag_show_pulldown_states = ((($process == true || $entry_state_has_zones == true) && $zone_name == '') || ACCOUNT_STATE_DRAW_INITIAL_DROPDOWN == 'true' || $error_state_input) ? true : false; // 是否使用下拉框式州选择器
  $state = ($flag_show_pulldown_states) ? $state : $zone_name; // 当使用下拉框式选择这里state 为州的ID，否则为名称
  $state_field_label = ($flag_show_pulldown_states) ? '' : ENTRY_STATE;
```
