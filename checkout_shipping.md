## 选择运输方式 checkout_shipping

首先通常情况下，用户结算是通过购物车或直接点击页面最上方的checkout按钮进入结算流程，结算的首要任务就是确认发货方式，确认发货方式时先要确认收货地址。

在进入结算流程前，zencart要求用户是已经登陆的，并且购物车里必须添加了物品。

```php
// if there is nothing in the customers cart, redirect them to the shopping cart page
if ($_SESSION['cart']->count_contents() <= 0) {
  zen_redirect(zen_href_link(FILENAME_TIME_OUT));
}

// if the customer is not logged on, redirect them to the login page
if (!isset($_SESSION['customer_id']) || !$_SESSION['customer_id']) {
  $_SESSION['navigation']->set_snapshot();
  zen_redirect(zen_href_link(FILENAME_LOGIN, '', 'SSL'));
} else {
  // validate customer
  if (zen_get_customer_validate_session($_SESSION['customer_id']) == false) {
    $_SESSION['navigation']->set_snapshot(array('mode' => 'SSL', 'page' => FILENAME_CHECKOUT_SHIPPING));
    zen_redirect(zen_href_link(FILENAME_LOGIN, '', 'SSL'));
  }
}
```

上面的代码是在结算流程里的每一个环节都要事先检查的部分。然后确认发货环节还要保证所添加的产品符合产品要求，这里的要求包括最小购买量、最大购买量、库存缺货等。

购物车类的get_products方法当$check_for_valid_cart传入true会计算这些限制条件，完成后会将结果保存到会话$_SESSION['valid_to_checkout']里，当结果为false时表示未通过限制检查，$_SESSION['cart_errors']保存了错误的详细描述。

```php
// Validate Cart for checkout
$_SESSION['valid_to_checkout'] = true;
$_SESSION['cart']->get_products(true);
if ($_SESSION['valid_to_checkout'] == false) {
  $messageStack->add('header', ERROR_CART_UPDATE, 'error');
  zen_redirect(zen_href_link(FILENAME_SHOPPING_CART));
}
```

下面的代码检查库存是否充足，这取决于用户在后台的设置。

```php
if ( (STOCK_CHECK == 'true') && (STOCK_ALLOW_CHECKOUT != 'true') ) {
  $products = $_SESSION['cart']->get_products();
  for ($i=0, $n=sizeof($products); $i<$n; $i++) {
    if (zen_check_stock($products[$i]['id'], $products[$i]['quantity'])) {
      zen_redirect(zen_href_link(FILENAME_SHOPPING_CART));
      break;
    }
  }
}
```

默认收货地址，$_SESSION['sendto']保存用户选择的收货地址ID（这个地址是归用户所有，他在登陆后可以增加、编辑、删除地址），在下面的代码里两种情况会默认收货地址，一是没有$_SESSION['sendto']时，二是$_SESSION['sendto']并非当前登陆用户所有。

$_SESSION['shipping']保存当前发货方式的结构，如下：

```php
array('id' => $_SESSION['shipping'],
      'title' => (($free_shipping == true) ?  $quote[0]['methods'][0]['title'] : $quote[0]['module'] . ' (' . $quote[0]['methods'][0]['title'] . ')'),
      'cost' => $quote[0]['methods'][0]['cost'])
```

其中id 为发货方式代码，title为发货方式名称，cost 为运费。

```php
if (!$_SESSION['sendto']) {
  $_SESSION['sendto'] = $_SESSION['customer_default_address_id'];
} else {
// verify the selected shipping address
  $check_address_query = "SELECT count(*) AS total
                          FROM   " . TABLE_ADDRESS_BOOK . "
                          WHERE  customers_id = :customersID
                          AND    address_book_id = :addressBookID";

  $check_address_query = $db->bindVars($check_address_query, ':customersID', $_SESSION['customer_id'], 'integer');
  $check_address_query = $db->bindVars($check_address_query, ':addressBookID', $_SESSION['sendto'], 'integer');
  $check_address = $db->Execute($check_address_query);

  if ($check_address->fields['total'] != '1') {
    $_SESSION['sendto'] = $_SESSION['customer_default_address_id'];
    $_SESSION['shipping'] = '';
  }
```

$_SESSION['customer_default_address_id']是用户默认收货地址ID，这是在用户登陆后获取的，在用户登陆时还获取了其它一些资料，包括有
 $_SESSION['customer_id'] = $check_customer->fields['customers_id'];

 ```php
$_SESSION['customer_default_address_id'] = $check_customer->fields['customers_default_address_id'];
$_SESSION['customers_authorization'] = $check_customer->fields['customers_authorization'];
$_SESSION['customer_first_name'] = $check_customer->fields['customers_firstname'];
$_SESSION['customer_last_name'] = $check_customer->fields['customers_lastname'];
$_SESSION['customer_country_id'] = $check_country->fields['entry_country_id'];
$_SESSION['customer_zone_id']
```

用户默认地址即用户在注册时所填写的地址，当用户未更改时默认它为主地址，主地址是不能删除的，所以每个用户都至少拥有一个地址。

### 创建订单结构

下面两句看上去不起眼，但实际上作用很大，它实现了将购物车的内容填充到订单结构，为最终生成订单数据做好准备。

```php
require(DIR_WS_CLASSES . 'order.php');
$order = new order;
```

未指定任何参数初始化订单类实例会自动用购物车的内容生成订单数据结构，如将订单号Order ID填入，使用自动以加载该订单的数据生成订单数据结构，这两个订单数据结构大致相同，但细微之处有所不同，详细的订单结构请参考《订单数据结构》。

超时限制，这要求整个结算过程在要求的时间限制范围内，可以避免过度占用服务器资源，以及系统安全的考虑。

```php
if (isset($_SESSION['cart']->cartID)) {
  if (!isset($_SESSION['cartID']) || $_SESSION['cart']->cartID != $_SESSION['cartID']) {
    $_SESSION['cartID'] = $_SESSION['cart']->cartID;
  }
} else {
  zen_redirect(zen_href_link(FILENAME_TIME_OUT));
}
```

zencart还支持虚拟物品的交易，当为虚拟物品时是不需要发货的，所以我们可以绕过发货直接进入付款环节

```php
if ($order->content_type == 'virtual') {
  $_SESSION['shipping'] = 'free_free';
  $_SESSION['shipping']['title'] = 'free_free';
  $_SESSION['sendto'] = false;
  zen_redirect(zen_href_link(FILENAME_CHECKOUT_PAYMENT, '', 'SSL'));
}
```

计算出总重量和总价值，这两个数值主要是给运输模块提供计算依据，绝大多数发货方式都是按重量、体积计费的，所以提供所有物品的总重量是非常必要的，那么总价值与发货方式有什么用处呢？总价值主要是用于用户的个性化定制，例如可以限制当总价值小于某个金额时，某个发货方式不可用；或当金额在某个区间内时，发货方式需要增加多少运费等等。

```php
$total_weight = $_SESSION['cart']->show_weight();
$total_count = $_SESSION['cart']->count_contents();
```

### 处理免运费

免运费需要用户在后台安装free shipping的统计模块，免运费有几个参数控制：

1. MODULE_ORDER_TOTAL_SHIPPING_DESTINATION发往地区，分国内national、国际international和both三个选项，当发货地址的国家与后台所设店铺国家一致认为是国内。
2. MODULE_ORDER_TOTAL_SHIPPING_FREE_SHIPPING_OVER最小金额，当购买物品的总价值小于规定金额时，免运费不生效。

```php
$pass = true;
if ( defined('MODULE_ORDER_TOTAL_SHIPPING_FREE_SHIPPING') && (MODULE_ORDER_TOTAL_SHIPPING_FREE_SHIPPING == 'true') ) {
  $pass = false;

  switch (MODULE_ORDER_TOTAL_SHIPPING_DESTINATION) {
    case 'national':
      if ($order->delivery['country_id'] == STORE_COUNTRY) {
        $pass = true;
      }
      break;
    case 'international':
      if ($order->delivery['country_id'] != STORE_COUNTRY) {
        $pass = true;
      }
      break;
    case 'both':
      $pass = true;
      break;
  }

  $free_shipping = false;
  if ( ($pass == true) && ($_SESSION['cart']->show_total() >= MODULE_ORDER_TOTAL_SHIPPING_FREE_SHIPPING_OVER) ) {
    $free_shipping = true;
  }
} else {
  $free_shipping = false;
}
```

当$free_shipping为false时，表示非免运费。

### 用户留言

```php
if (isset($_SESSION['comments'])) {
  $comments = $_SESSION['comments'];
}
```

当用户选择了发货方式提交表单后，我们会重新收集用户留言，会确认发货方式

```php
if ( isset($_POST['action']) && ($_POST['action'] == 'process') ) {
  if (zen_not_null($_POST['comments'])) {
    $_SESSION['comments'] = zen_db_prepare_input($_POST['comments']);
  }
  $comments = $_SESSION['comments'];
  $quote = array();

  if ( (zen_count_shipping_modules() > 0) || ($free_shipping == true) ) {
    if ( (isset($_POST['shipping'])) && (strpos($_POST['shipping'], '_')) ) {
      /**
       * check to be sure submitted data hasn't been tampered with
       */
      if ($_POST['shipping'] == 'free_free' && ($order->content_type != 'virtual' && !$pass)) { // 这里检查仅允许物品是虚拟或前面已经pass的才可以使用免运费方式。
        $quote['error'] = 'Invalid input. Please make another selection.';
      } else {
        $_SESSION['shipping'] = $_POST['shipping'];
      }

      list($module, $method) = explode('_', $_SESSION['shipping']);
      if ( is_object($$module) || ($_SESSION['shipping'] == 'free_free') ) { // is_object($$module)的检查可以阻止外部故意传入错误参数的恶意攻击，确保发货方式是存在的。
        if ($_SESSION['shipping'] == 'free_free') {
          $quote[0]['methods'][0]['title'] = FREE_SHIPPING_TITLE;
          $quote[0]['methods'][0]['cost'] = '0';
        } else {
          $quote = $shipping_modules->quote($method, $module); // 调用指定发货模块的quote方式，传入子发货方式，得出的结束与上面的结构类似，当有错误时使用error参数返回
        }
        if (isset($quote['error'])) { // 当出现错误时，清除发货方式的选择
          $_SESSION['shipping'] = '';
        } else {
          if ( (isset($quote[0]['methods'][0]['title'])) && (isset($quote[0]['methods'][0]['cost'])) ) {
            $_SESSION['shipping'] = array('id' => $_SESSION['shipping'],
                              'title' => (($free_shipping == true) ?  $quote[0]['methods'][0]['title'] : $quote[0]['module'] . ' (' . $quote[0]['methods'][0]['title'] . ')'),
                              'cost' => $quote[0]['methods'][0]['cost']);

            zen_redirect(zen_href_link(FILENAME_CHECKOUT_PAYMENT, '', 'SSL')); // 正确时转向确认付款方式环节
          }
        }
      } else {
        $_SESSION['shipping'] = false;
      }
    }
  } else { //  当没有发货方式可选时，也会转向到确认付款方式环节
    $_SESSION['shipping'] = false;

    zen_redirect(zen_href_link(FILENAME_CHECKOUT_PAYMENT, '', 'SSL'));
  }
}
```

### 列举出所有可选的发货方式

```php
$quotes = $shipping_modules->quote();
```

发货模块的quote方法是一个非常重要的方法，它决定了是否需要显示出该发货方式给用户选择，以及用户选择了发货方式后的运费确认过程。

zencart系统中允许一个发货方式支持多种子发货方式，这在实际的环境中是非常常见的，例如中国邮政同时支持小包、挂号等邮寄方式，DHL支持朝九特派、环球快递、限日快递等等服务，quote方法有一个参数$method，当未指定时会返回该发货方式可选的所有子发货方式，当指定了$method时，代表用户选择了指定的子发货方式。
在zencart默认的8种发货方式中，都没有子发货方式，所以它们的quote方法的$method参数都没有使用。

quote方法返回如下结构：

```php
array(
  'id' => 发货模块代码,
  'module' => 模块名称,
  'methods' => array(
    array(
      'id' => 子发货方式代码,
      'title' => 子发货方式名称
      'cost' => 运费 
  ))
);
```

### 自动选择最便宜的发货方式

```php
if ( !$_SESSION['shipping'] || ( $_SESSION['shipping'] && ($_SESSION['shipping'] == false) && (zen_count_shipping_modules() > 1) ) ) $_SESSION['shipping'] = $shipping_modules->cheapest();
```

### 页面输出

循环所有可选发货方式，输出其它主名称以及所有子发货方式，当有错误时需要显示出来。

```php
$radio_buttons = 0;
for ($i=0, $n=sizeof($quotes); $i<$n; $i++) { // 循环所有发货方式
// bof: field set
// allows FedEx to work comment comment out Standard and Uncomment FedEx
//      if ($quotes[$i]['id'] != '' || $quotes[$i]['module'] != '') { // FedEx
if ($quotes[$i]['module'] != '') { // Standard
?>
<fieldset>
<legend><?php echo $quotes[$i]['module']; ?>&nbsp;<?php if (isset($quotes[$i]['icon']) && zen_not_null($quotes[$i]['icon'])) { echo $quotes[$i]['icon']; } ?></legend>

<?php
  if (isset($quotes[$i]['error'])) {
?>
<div><?php echo $quotes[$i]['error']; ?></div>
<?php
  } else {
    for ($j=0, $n2=sizeof($quotes[$i]['methods']); $j<$n2; $j++) { // 循环所有子发货方式
// set the radio button to be checked if it is the method chosen
      $checked = (($quotes[$i]['id'] . '_' . $quotes[$i]['methods'][$j]['id'] == $_SESSION['shipping']['id']) ? true : false);

?>
<?php
      if ( ($n > 1) || ($n2 > 1) ) { // 这里确保当仅有一个可选的发货方式时，仅选择可选的那个
?>
<div class="important forward"><?php echo $currencies->format(zen_add_tax($quotes[$i]['methods'][$j]['cost'], (isset($quotes[$i]['tax']) ? $quotes[$i]['tax'] : 0))); ?></div>
<?php
      } else {
?>
<div class="important forward"><?php echo $currencies->format(zen_add_tax($quotes[$i]['methods'][$j]['cost'], $quotes[$i]['tax'])) . zen_draw_hidden_field('shipping', $quotes[$i]['id'] . '_' . $quotes[$i]['methods'][$j]['id']); ?></div>
<?php
      }
?>

<?php echo zen_draw_radio_field('shipping', $quotes[$i]['id'] . '_' . $quotes[$i]['methods'][$j]['id'], $checked, 'id="ship-'.$quotes[$i]['id'] . '-' . str_replace(' ', '-', $quotes[$i]['methods'][$j]['id']) .'"'); ?>
<label for="ship-<?php echo $quotes[$i]['id'] . '-' . str_replace(' ', '-', $quotes[$i]['methods'][$j]['id']); ?>" class="checkboxLabel" ><?php echo $quotes[$i]['methods'][$j]['title']; ?></label>
<!--</div>-->
<br class="clearBoth" />
<?php
      $radio_buttons++;
    }
  }
?>

</fieldset>
<?php
}
// eof: field set
}
```

标准的输出金额的方法是：`$currencies->format(zen_add_tax($quotes[$i]['methods'][$j]['cost'], (isset($quotes[$i]['tax']) ? $quotes[$i]['tax'] : 0)))`，这段代码兼容了金额与税额的计算，并自动按当前币种进行展示。


