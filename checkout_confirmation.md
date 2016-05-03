## 确认环节 checkout_confirmation

### 保存付款方式和客户留言到会话

```php
if (isset($_POST['payment'])) $_SESSION['payment'] = $_POST['payment'];
$_SESSION['comments'] = zen_db_prepare_input($_POST['comments']);
```

### 检查是否勾选了同意条款

```php
if (DISPLAY_CONDITIONS_ON_CHECKOUT == 'true') {
  if (!isset($_POST['conditions']) || ($_POST['conditions'] != '1')) {
    $messageStack->add_session('checkout_payment', ERROR_CONDITIONS_NOT_ACCEPTED, 'error');
  }
}
```

### 信贷覆盖？需要研究用处

会直接设置付款功能失效。

```php
if (!isset($credit_covers)) $credit_covers = FALSE;

//echo 'credit covers'.$credit_covers;

if ($credit_covers) {
  unset($_SESSION['payment']);
  $_SESSION['payment'] = '';
}
```

在includes/classes/order_total.php里，当总金额与将减少的金额相差在0.009以内并且不是免运费的情况下，设置$credit_covers=true

```php
$difference = $order->info['total'] - $total_deductions;
if ( $difference <= 0.009 && $_SESSION['payment'] != 'freecharger') {
  $credit_covers = true;
}
```

初始化相应的付款模块，update_status方法用于确认此付款方式是否可能，通常该处的代码只检查是否满足zone的限制要求，除此之外，我们也可以增加一些其它的限制条件，比如金额过大时限制使用，或者强制要求使用SSL连接方式才能使用本付款方式。

```php
$payment_modules = new payment($_SESSION['payment']);
$payment_modules->update_status();
if ( ($_SESSION['payment'] == '' || !is_object($$_SESSION['payment']) ) && $credit_covers === FALSE) {
  $messageStack->add_session('checkout_payment', ERROR_NO_PAYMENT_MODULE_SELECTED, 'error');
}

if (is_array($payment_modules->modules)) {
  $payment_modules->pre_confirmation_check(); // 再次确认检查相关内容
}

if ($messageStack->size('checkout_payment') > 0) { // 当付款内容有错误时，会重新返回到付款方式选择环节
  zen_redirect(zen_href_link(FILENAME_CHECKOUT_PAYMENT, '', 'SSL'));
}
```

### 当使用了Coupon时，设置用户访问来源

```php
if ($_SESSION['cc_id']) {
  $discount_coupon_query = "SELECT coupon_code
                            FROM " . TABLE_COUPONS . "
                            WHERE coupon_id = :couponID";

  $discount_coupon_query = $db->bindVars($discount_coupon_query, ':couponID', $_SESSION['cc_id'], 'integer');
  $discount_coupon = $db->Execute($discount_coupon_query);

  $customers_referral_query = "SELECT customers_referral
                               FROM " . TABLE_CUSTOMERS . "
                               WHERE customers_id = :customersID";

  $customers_referral_query = $db->bindVars($customers_referral_query, ':customersID', $_SESSION['customer_id'], 'integer');
  $customers_referral = $db->Execute($customers_referral_query);

  // only use discount coupon if set by coupon
  if ($customers_referral->fields['customers_referral'] == '' and CUSTOMERS_REFERRAL_STATUS == 1) {
    $sql = "UPDATE " . TABLE_CUSTOMERS . "
            SET customers_referral = :customersReferral
            WHERE customers_id = :customersID";

    $sql = $db->bindVars($sql, ':customersID', $_SESSION['customer_id'], 'integer');
    $sql = $db->bindVars($sql, ':customersReferral', $discount_coupon->fields['coupon_code'], 'string');
    $db->Execute($sql);
  } else {
    // do not update referral was added before
  }
}
```

CUSTOMERS_REFERRAL_STATUS的值在后台设置

付款转向，这里有两种情况，一种是在线支付方式，一种是离线支付方式，对于在线支付方式，我们需要自动转向到相应的支付功能提供商的页面完成支付过程，而对于离线支付方式，zencart有一套标准的订单处理步骤，我们只需要转向到checkout_process便可。

```php
if (isset($$_SESSION['payment']->form_action_url)) {
  $form_action_url = $$_SESSION['payment']->form_action_url;
} else {
  $form_action_url = zen_href_link(FILENAME_CHECKOUT_PROCESS, '', 'SSL');
}
```

禁止修改账单地址，通常的付款方式都会允许可修改账单地址，但有一些特殊的情况下会使用到此限制，你只要设置付款模块的flagDisablePaymentAddressChange变量=true即可，通常这可以在付款模块的in_special_checkout方法里按情况设置。

```php
$flagDisablePaymentAddressChange = false;
if (isset($$_SESSION['payment']->flagDisablePaymentAddressChange)) {
  $flagDisablePaymentAddressChange = $$_SESSION['payment']->flagDisablePaymentAddressChange;
}
```

### 模板输出

再次确认过程，主要针对需要填写其它资料的付款方式，如直接输入信用卡完成付款的付款方式等。

```php
<?php
  if (is_array($payment_modules->modules)) {
    if ($confirmation = $payment_modules->confirmation()) {
?>
<div class="important"><?php echo $confirmation['title']; ?></div>
<?php
    }
?>
<div class="important">
<?php
      for ($i=0, $n=sizeof($confirmation['fields']); $i<$n; $i++) {
?>
<div class="back"><?php echo $confirmation['fields'][$i]['title']; ?></div>
<div ><?php echo $confirmation['fields'][$i]['field']; ?></div>
<?php
     }
?>
      </div>
<?php
  }
?>
```

当我们需要再次确认时，可以仿照下面的代码来编写，否则可以直接返回false即可。

```php
function confirmation() {
  $confirmation = array('title' => $this->title . ': ' . $this->cc_card_type,
                        'fields' => array(array('title' => MODULE_PAYMENT_LINKPOINT_API_TEXT_CREDIT_CARD_OWNER,
                                                'field' => $_POST['linkpoint_api_cc_owner']),
                                          array('title' => MODULE_PAYMENT_LINKPOINT_API_TEXT_CREDIT_CARD_NUMBER,
                                                'field' => str_repeat('X', (strlen($this->cc_card_number) - 4)) . substr($this->cc_card_number, -4)),
                                          array('title' => MODULE_PAYMENT_LINKPOINT_API_TEXT_CREDIT_CARD_EXPIRES,
                                                'field' => strftime('%B, %Y', mktime(0,0,0,$_POST['linkpoint_api_cc_expires_month'], 1, '20' . $_POST['linkpoint_api_cc_expires_year'])))));

  return $confirmation;
}
```

付款转向参数，通过隐藏域的方法，向将转向的页面传递参数

```php
<?php
  echo zen_draw_form('checkout_confirmation', $form_action_url, 'post', 'id="checkout_confirmation" onsubmit="submitonce();"');

  if (is_array($payment_modules->modules)) {
    echo $payment_modules->process_button();
  }
?>
```