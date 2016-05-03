# 修改账单地址 checkout_payment_address

与修改发货地址相类似

```php
$addressType = "billto";
require(DIR_WS_MODULES . zen_get_module_directory('checkout_new_address'));
// if no billing destination address was selected, use their own address as default
if (!$_SESSION['billto']) {
  $_SESSION['billto'] = $_SESSION['customer_default_address_id'];
}

$addresses_count = zen_count_customer_address_book_entries();
```

主要还是依赖于checkout_new_address模块，来进行。
