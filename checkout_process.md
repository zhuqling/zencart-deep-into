## 普通离线支付方式 checkout_process

直接嵌入checkout_prcess.php模块

```php
require(DIR_WS_MODULES . zen_get_module_directory('checkout_process.php'));

// load the after_process function from the payment modules
$payment_modules->after_process();
```

复位购物车和结算中使用的会话数据

```php
$_SESSION['cart']->reset(true);

// unregister session variables used during checkout
unset($_SESSION['sendto']);
unset($_SESSION['billto']);
unset($_SESSION['shipping']);
unset($_SESSION['payment']);
unset($_SESSION['comments']);
$order_total_modules->clear_posts();//ICW ADDED FOR CREDIT CLASS SYSTEM
转向到付款成功页面
zen_redirect(zen_href_link(FILENAME_CHECKOUT_SUCCESS, (isset($_GET['action']) && $_GET['action'] == 'confirm' ? 'action=confirm' : ''), 'SSL'));
```

在checkout_process.php模块中限制多次提交订单，达3次以上将视为恶意提交，将转向到其它页面

```php
if (!isset($_SESSION['payment_attempt'])) $_SESSION['payment_attempt'] = 0;
$_SESSION['payment_attempt']++;
$zco_notifier->notify('NOTIFY_CHECKOUT_SLAMMING_ALERT');
if ($_SESSION['payment_attempt'] > 3) {
  $zco_notifier->notify('NOTIFY_CHECKOUT_SLAMMING_LOCKOUT');
  $_SESSION['cart']->reset(TRUE);
  zen_session_destroy();
  zen_redirect(zen_href_link(FILENAME_TIME_OUT));
}
```

### 订单的生成过程

```php
// 处理付款模块
require(DIR_WS_CLASSES . 'payment.php');
$payment_modules = new payment($_SESSION['payment']);
// 处理发货模块
require(DIR_WS_CLASSES . 'shipping.php');
$shipping_modules = new shipping($_SESSION['shipping']);
// 初始化订单对象
require(DIR_WS_CLASSES . 'order.php');
$order = new order;

// 确认产品无异常
if (sizeof($order->products) < 1) {
  zen_redirect(zen_href_link(FILENAME_SHOPPING_CART));
}
// 初始化统计模块
require(DIR_WS_CLASSES . 'order_total.php');
$order_total_modules = new order_total;

if (strpos($GLOBALS[$_SESSION['payment']]->code, 'paypal') !== 0) {
  $order_totals = $order_total_modules->pre_confirmation_check();
}
if ($credit_covers === TRUE)
{
	$order->info['payment_method'] = $order->info['payment_module_code'] = '';
}
$order_totals = $order_total_modules->process();

if (!isset($_SESSION['payment']) && $credit_covers === FALSE) {
  zen_redirect(zen_href_link(FILENAME_DEFAULT));
}

// load the before_process function from the payment modules
$payment_modules->before_process();
// 创建订单表数据
$insert_id = $order->create($order_totals, 2);
$payment_modules->after_order_create($insert_id);
// 创建订单所购产品明细
$order->create_add_products($insert_id);
$_SESSION['order_number_created'] = $insert_id;
```

$_SESSION['order_number_created'] 保存新创建的订单号