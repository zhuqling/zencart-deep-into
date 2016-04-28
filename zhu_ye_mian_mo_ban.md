# 主页面模板
网面BODY内容的控制由主页面模板加载开始，在index.php前端文件里调用。
require($template->get_template_dir('tpl_main_page.php',DIR_WS_TEMPLATE, $current_page_base,'common'). '/tpl_main_page.php');
template模板类的方法get_template_dir会自动控制加载哪个目录，其优先级以下：
1.	includes/templates/<template>/<page>/ 自定义模板目录下页面目录
2.	includes/templates/template_default/<page>/ 默认模板目录下页面目录
3.	includes/templates/<template>/common/ 自定义模板目录下通用目录
4.	includes/templates/template_default/common/ 默认模板目录下通用目录
可以看出通过这样的优先级安排，自定义模板和单独页面的布局非常容易从默认模板中区分出来，这对于测试新模板或新页面布局，而不影响现有模板是非常有用的。
默认主页面模块是位于includes/templates/template_default/common/目录下的tpl_main_page.php文件。它是一个标准的三栏布局，主要分为头部、主内容、左边栏、右边栏、底部五大块区域。
	其中头部由tpl_header.php模板文件管理，由变量$flag_disable_header控制是否隐藏头部区域；
	主内容由require($body_code); 代码直接调用页面代码生成；
	左边栏由includes/modules/column_left.php模块加载，左边栏的布局在后台layout里控制，是否隐藏左边栏，由变量$flag_disable_left控制的。
if (COLUMN_LEFT_STATUS == 0 || (CUSTOMERS_APPROVAL == '1' and $_SESSION['customer_id'] == '') || (CUSTOMERS_APPROVAL_AUTHORIZATION == 1 && CUSTOMERS_AUTHORIZATION_COLUMN_LEFT_OFF == 'true' and ($_SESSION['customers_authorization'] != 0 or $_SESSION['customer_id'] == ''))) {
  // global disable of column_left
  $flag_disable_left = true;
}
可以看出默认控制是否隐藏左边栏有以下几个因素：
a) COLUMN_LEFT_STATUS（左边栏状态）常量=0，由网站后台操作；
b) 当CUSTOMERS_APPROVAL（登陆限制）常量=1并且未登陆时；
c) CUSTOMERS_APPROVAL_AUTHORIZATION（授权登陆）常量=1，并且CUSTOMERS_AUTHORIZATION_COLUMN_LEFT_OFF（授权下关闭左边栏）常量=true，且用户未登陆或未授权时；
	右边栏由includes/modules/column_right.php模块加载，右边栏的布局同样在后台layout可以控制。$flag_disable_right变量控制是否隐藏右边栏。
if (in_array($current_page_base,explode(",",'list_pages_to_skip_all_right_sideboxes_on_here,separated_by_commas,and_no_spaces')) ) {
    $flag_disable_right = true;
  }
zencart默认代码预留了隐藏右边栏的功能，我们只需要修改上面的代码，将explode函数后面的第二个参数里增加需要隐藏右边栏页面的名称即可，多个页面名称以逗号或空格分隔。
	底部区域由tpl_footer.php模板文件管理，由变量$flag_disable_footer控制是否隐藏底部区域。

Banner广告
Banner分布图
zencart中广告的应用主要体现在Banner的管理上，无论是头部、底部，还是左边栏、右边栏都有Banner的踪影。Banner的分组情况如下：
头部区域为三组：SHOW_BANNERS_GROUP_SET1、SHOW_BANNERS_GROUP_SET2、SHOW_BANNERS_GROUP_SET3
底部区域为三组：SHOW_BANNERS_GROUP_SET4、SHOW_BANNERS_GROUP_SET5、SHOW_BANNERS_GROUP_SET6
侧边栏区域：SHOW_BANNERS_GROUP_SET7、SHOW_BANNERS_GROUP_SET8、SHOW_BANNERS_GROUP_SET_ALL

<?php
  if (SHOW_BANNERS_GROUP_SET1 != '' && $banner = zen_banner_exists('dynamic', SHOW_BANNERS_GROUP_SET1)) {
    if ($banner->RecordCount() > 0) {
?>
<div id="bannerOne" class="banners"><?php echo zen_display_banner('static', $banner); ?></div>
<?php
    }
  }
?>
zen_banner_exists($action, $identifier)，$action可选值有'dynamic'和'static'，当为dynamic时，代表是广告组$identifier参数应填入Banner组，以随机位置返回该分组的所有Banner。当为static时，代表是指定Banner，$identifier参数应填入Banner ID。
zen_display_banner($action, $identifier)，$action可选值也是dynamic和static，当为dynamic时，$identifier应指定广告组，查询数据库后输出所有指定组的广告，当为static时，如将zen_banner_exists函数返回的Banner数据传入$identifier参数，则不需要查询数据库直接按输入Banner输出，否则$identifier被认定为Banner ID，查询数据库后输出该Banner。

错误/消息提醒
前面我们已经介绍过，在页面处理代码里可以设置消息堆栈。
$messageStack->add('account_password', ENTRY_PASSWORD_NEW_ERROR);
消息的显示，是在模板里实现的，消息的显示与常规消息类型见如下代码。
1、Header类型消息和error_message以及info_message
头部tpl_header.php
if ($messageStack->size('header') > 0) {
    echo $messageStack->output('header');
  }
  if (isset($_GET['error_message']) && zen_not_null($_GET['error_message'])) {
  echo htmlspecialchars(urldecode($_GET['error_message']));
  }
  if (isset($_GET['info_message']) && zen_not_null($_GET['info_message'])) {
   echo htmlspecialchars($_GET['info_message']);
其中header类型的消息将在网页最上方显示。同时$_GET['error_message']和$_GET['info_message']如被指定值时也将显示，$_GET['error_message']定位为错误类型，$_GET['info_message']定位为通知类型。
2、Upload类型
主页面tpl_main_page.php，页面内容上方。
<!-- bof upload alerts -->
<?php if ($messageStack->size('upload') > 0) echo $messageStack->output('upload'); ?>
<!-- eof upload alerts -->
upload类型定位为上传错误类型。

3、自定义
<?php if ($messageStack->size('addressbook') > 0) echo $messageStack->output('addressbook'); ?> 
用户片定义的类型，由用户自己命名，只要在页面代码里的类型名称与模板输出时一致即可。




四、具体页面分析
1、首页
允许输入参数
当$cPath有输入时，判断分类类型products（有产品或无子分类也无产品）/nested（无产品但有子分类），付值给变量$category_depth。
加载$define_page，静态HTML介绍文件
加载语言
$_GET['typefilter']参数，标准使用includes/index_filters/default_filter.php，该文件中有参数有：$_GET['sort'](仅前三位)，$_GET['alpha_filter_id']（且值>0，产品名称的第一个字母）$alpha_sort = " and pd.products_name LIKE '" . chr((int)$_GET['alpha_filter_id']) . "%' ";$_GET['manufacturers_id']（非空，供应商），$_GET['filter_id']（分类ID），$select_column_list变量定义查询SQL字段，
当指宝了供应商manufacturers_id时，filter_id为分类ID，当未指定供应商ID时，$_GET['filter_id']为供应商ID，分类为当前分类ID，两者都未指定时，仅使用当前分类ID过滤。
PRODUCT_LISTING_DEFAULT_SORT_ORDER为默认排序方式
$column_list变量为显示的列，可选值有文本类型：PRODUCT_LIST_MODEL 产品型号、PRODUCT_LIST_NAME 产品名称、PRODUCT_LIST_MANUFACTURER 供应商、PRODUCT_LIST_QUANTITY 数量、PRODUCT_LIST_IMAGE 实为标题、PRODUCT_LIST_WEIGHT 重量、PRODUCT_LIST_PRICE 产品排序（p.products_price_sorter）

PRODUCT_LIST_FILTER >0是否有过滤条件，当$_GET['manufacturers_id']有值时，使用供应商过滤查询出分类，否则使用当前分类过滤查询出供应商。

为nested类型输出 tpl_index_categories.php模板，当为products类型输出tpl_index_product_list.php模板，其它输出tpl_index_default.php模板，$current_categories_description保存当前分类描述，
  if ($categories_image = zen_get_categories_image($current_category_id)) {
?>
<div id="categoryImgListing" class="categoryImg"><?php echo zen_image(DIR_WS_IMAGES . $categories_image, '', SUBCATEGORY_IMAGE_TOP_WIDTH, SUBCATEGORY_IMAGE_TOP_HEIGHT); ?></div>
<?php
  }
输出分类图片，tpl_modules_category_row.php模板输出所有子分类，它又调用category_row.php模块文件得到结果，组建$list_box_contents[$row][$col]数组，并交由tpl_columnar_display.php模板完全输出。
$list_box_contents[$row][$col] = array('params' => 'class="categoryListBoxContents"' . ' ' . 'style="width:' . $col_width . '%;"',
                                           'text' => '<a href="' . zen_href_link(FILENAME_DEFAULT, $cPath_new) . '">' . zen_image(DIR_WS_IMAGES . $categories->fields['categories_image'], $categories->fields['categories_name'], SUBCATEGORY_IMAGE_WIDTH, SUBCATEGORY_IMAGE_HEIGHT) . '<br />' . $categories->fields['categories_name'] . '</a>');


includes/classes/db/mysql/define_queries.php定义了查询语句




2、分类页
3、产品页
4、购物车
5、搜索
6、功能模块
7、页面模块
tpl_modules_featured_products.php推荐产品，tpl_modules_specials_default.php特价产品，tpl_modules_whats_new.php新品，upcoming_products.php即将到货，

安装，环境要求
安全
SEO URL
全局变量和SESSION
类
函数

八、付款流程
付款流程分5步进行，
checkout_shipping、checkout_payment、checkout_confirmation、checkout_process、checkout_success；
其中checkout_shipping选择运输方式时可以修改发货地址checkout_shipping_address；checkout_payment选择付款方式时允许修改billing address账号地址checkout_payment_address。
首先通常情况下，用户结算是通过购物车或直接点击页面最上方的checkout按钮进入结算流程，结算的首要任务就是确认发货方式，确认发货方式时先要确认收货地址。
在进入结算流程前，zencart要求用户是已经登陆的，并且购物车里必须添加了物品。
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
上面的代码是在结算流程里的每一个环节都要事先检查的部分。然后确认发货环节还要保证所添加的产品符合产品要求，这里的要求包括最小购买量、最大购买量、库存缺货等。购物车类的get_products方法当$check_for_valid_cart传入true会计算这些限制条件，完成后会将结果保存到会话$_SESSION['valid_to_checkout']里，当结果为false时表示未通过限制检查，$_SESSION['cart_errors']保存了错误的详细描述。
// Validate Cart for checkout
  $_SESSION['valid_to_checkout'] = true;
  $_SESSION['cart']->get_products(true);
  if ($_SESSION['valid_to_checkout'] == false) {
    $messageStack->add('header', ERROR_CART_UPDATE, 'error');
    zen_redirect(zen_href_link(FILENAME_SHOPPING_CART));
  }
下面的代码检查库存是否充足，这取决于用户在后台的设置。
if ( (STOCK_CHECK == 'true') && (STOCK_ALLOW_CHECKOUT != 'true') ) {
    $products = $_SESSION['cart']->get_products();
    for ($i=0, $n=sizeof($products); $i<$n; $i++) {
      if (zen_check_stock($products[$i]['id'], $products[$i]['quantity'])) {
        zen_redirect(zen_href_link(FILENAME_SHOPPING_CART));
        break;
      }
    }
  }

默认收货地址，$_SESSION['sendto']保存用户选择的收货地址ID（这个地址是归用户所有，他在登陆后可以增加、编辑、删除地址），在下面的代码里两种情况会默认收货地址，一是没有$_SESSION['sendto']时，二是$_SESSION['sendto']并非当前登陆用户所有。$_SESSION['shipping']保存当前发货方式的结构，如下
array('id' => $_SESSION['shipping'],
                                'title' => (($free_shipping == true) ?  $quote[0]['methods'][0]['title'] : $quote[0]['module'] . ' (' . $quote[0]['methods'][0]['title'] . ')'),
                                'cost' => $quote[0]['methods'][0]['cost'])
其中id 为发货方式代码，title为发货方式名称，cost 为运费。
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
$_SESSION['customer_default_address_id']是用户默认收货地址ID，这是在用户登陆后获取的，在用户登陆时还获取了其它一些资料，包括有
 $_SESSION['customer_id'] = $check_customer->fields['customers_id'];
        $_SESSION['customer_default_address_id'] = $check_customer->fields['customers_default_address_id'];
        $_SESSION['customers_authorization'] = $check_customer->fields['customers_authorization'];
        $_SESSION['customer_first_name'] = $check_customer->fields['customers_firstname'];
        $_SESSION['customer_last_name'] = $check_customer->fields['customers_lastname'];
        $_SESSION['customer_country_id'] = $check_country->fields['entry_country_id'];
        $_SESSION['customer_zone_id']
用户默认地址即用户在注册时所填写的地址，当用户未更改时默认它为主地址，主地址是不能删除的，所以每个用户都至少拥有一个地址。

创建订单结构
下面两句看上去不起眼，但实际上作用很大，它实现了将购物车的内容填充到订单结构，为最终生成订单数据做好准备。
 require(DIR_WS_CLASSES . 'order.php');
  $order = new order;
未指定任何参数初始化订单类实例会自动用购物车的内容生成订单数据结构，如将订单号Order ID填入，使用自动以加载该订单的数据生成订单数据结构，这两个订单数据结构大致相同，但细微之处有所不同，详细的订单结构请参考《订单数据结构》。

超时限制，这要求整个结算过程在要求的时间限制范围内，可以避免过度占用服务器资源，以及系统安全的考虑。
if (isset($_SESSION['cart']->cartID)) {
  if (!isset($_SESSION['cartID']) || $_SESSION['cart']->cartID != $_SESSION['cartID']) {
    $_SESSION['cartID'] = $_SESSION['cart']->cartID;
  }
} else {
  zen_redirect(zen_href_link(FILENAME_TIME_OUT));
}
zencart还支持虚拟物品的交易，当为虚拟物品时是不需要发货的，所以我们可以绕过发货直接进入付款环节
  if ($order->content_type == 'virtual') {
    $_SESSION['shipping'] = 'free_free';
    $_SESSION['shipping']['title'] = 'free_free';
    $_SESSION['sendto'] = false;
    zen_redirect(zen_href_link(FILENAME_CHECKOUT_PAYMENT, '', 'SSL'));
  }
计算出总重量和总价值，这两个数值主要是给运输模块提供计算依据，绝大多数发货方式都是按重量、体积计费的，所以提供所有物品的总重量是非常必要的，那么总价值与发货方式有什么用处呢？总价值主要是用于用户的个性化定制，例如可以限制当总价值小于某个金额时，某个发货方式不可用；或当金额在某个区间内时，发货方式需要增加多少运费等等。
$total_weight = $_SESSION['cart']->show_weight();
  $total_count = $_SESSION['cart']->count_contents();

处理免运费
免运费需要用户在后台安装free shipping的统计模块，免运费有几个参数控制：
一是MODULE_ORDER_TOTAL_SHIPPING_DESTINATION发往地区，分国内national、国际international和both三个选项，当发货地址的国家与后台所设店铺国家一致认为是国内。
二是MODULE_ORDER_TOTAL_SHIPPING_FREE_SHIPPING_OVER最小金额，当购买物品的总价值小于规定金额时，免运费不生效。
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
当$free_shipping为false时，表示非免运费。
用户留言
  if (isset($_SESSION['comments'])) {
    $comments = $_SESSION['comments'];
  }
当用户选择了发货方式提交表单后，我们会重新收集用户留言，会确认发货方式
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
列举出所有可选的发货方式
  $quotes = $shipping_modules->quote();
发货模块的quote方法是一个非常重要的方法，它决定了是否需要显示出该发货方式给用户选择，以及用户选择了发货方式后的运费确认过程。
zencart系统中允许一个发货方式支持多种子发货方式，这在实际的环境中是非常常见的，例如中国邮政同时支持小包、挂号等邮寄方式，DHL支持朝九特派、环球快递、限日快递等等服务，quote方法有一个参数$method，当未指定时会返回该发货方式可选的所有子发货方式，当指定了$method时，代表用户选择了指定的子发货方式。
在zencart默认的8种发货方式中，都没有子发货方式，所以它们的quote方法的$method参数都没有使用。quote方法返回如下结构：
array('id' => 发货模块代码,
    'module' => 模块名称,
    'methods' => 
array(array('id' => 子发货方式代码,
    'title' => 子发货方式名称
    'cost' => 运费 
))
);


自动选择最便宜的发货方式
  if ( !$_SESSION['shipping'] || ( $_SESSION['shipping'] && ($_SESSION['shipping'] == false) && (zen_count_shipping_modules() > 1) ) ) $_SESSION['shipping'] = $shipping_modules->cheapest();

页面输出
循环所有可选发货方式，输出其它主名称以及所有子发货方式，当有错误时需要显示出来。
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
标准的输出金额的方法是：$currencies->format(zen_add_tax($quotes[$i]['methods'][$j]['cost'], (isset($quotes[$i]['tax']) ? $quotes[$i]['tax'] : 0)))，这段代码兼容了金额与税额的计算，并自动按当前币种进行展示。

更改发货地址checkout_shipping_address
更改发货地址预处理代码中，与前面的确认发货方式一样需要处理用户是否已登陆、购物车是否有产品等常规判断，在加载了当前页面的语言文件后，我们还需要判断物品类型是否为虚拟物品，如果是虚拟物品就不需要收货地址，直接转向到确认付款环节。
这里通过将新建地址的部分分离到单独的模块，增加这一功能的重用性。
$addressType = "shipto";
require(DIR_WS_MODULES . zen_get_module_directory('checkout_new_address'));
$addressType在checkout_new_address模块中使用，当值为shipto时，表示是发货地址，当值为billto时，表示是账单地址。模块会根据该值自动设置$_SESSION['sendto']还是$_SESSION['billto']的值。
当用户填写新的发货地址并提交后，checkout_new_address模块会检查相关数据的有效性，如果所有数据都正确时，会保存地址信息到用户的名下，将根据$addressType的值，自动设置$_SESSION会话，然后转向到相应的发货或付款环节。
当前用户地址簿数量
$addresses_count = zen_count_customer_address_book_entries();
选择发货地址涉及到了三个模板文件，分别是默认页面模板文件tpl_checkout_shipping_address_default.php、多地址选择模板tpl_modules_checkout_address_book.php、新添加地址模板tpl_modules_checkout_new_address.php。
其中多地址选择模板的数据源来自模块checkout_address_book.php。
控制国家、州和省份的选择
  $selected_country = (isset($_POST['zone_country_id']) && $_POST['zone_country_id'] != '') ? $country : SHOW_CREATE_ACCOUNT_DEFAULT_COUNTRY; // 默认选择的国家
  $flag_show_pulldown_states = ((($process == true || $entry_state_has_zones == true) && $zone_name == '') || ACCOUNT_STATE_DRAW_INITIAL_DROPDOWN == 'true' || $error_state_input) ? true : false; // 是否使用下拉框式州选择器
  $state = ($flag_show_pulldown_states) ? $state : $zone_name; // 当使用下拉框式选择这里state 为州的ID，否则为名称
  $state_field_label = ($flag_show_pulldown_states) ? '' : ENTRY_STATE;

付款模块checkout_payment
我们略过常规检查不再说明，首先检查是否已经正确设置发货方式，
if (!$_SESSION['shipping']) {
  zen_redirect(zen_href_link(FILENAME_CHECKOUT_SHIPPING, '', 'SSL'));
}
if (isset($_SESSION['shipping']['id']) && $_SESSION['shipping']['id'] == 'free_free' && defined('MODULE_ORDER_TOTAL_SHIPPING_FREE_SHIPPING_OVER') && $_SESSION['cart']->show_total() < MODULE_ORDER_TOTAL_SHIPPING_FREE_SHIPPING_OVER) {
  zen_redirect(zen_href_link(FILENAME_CHECKOUT_SHIPPING, '', 'SSL'));
}
上述代码检查有无设置发货方式，并且检查免运费方式是否设置正确。
检查库存状态，当设置库存不足不发货时，会限制进入付款环节
// Stock Check
if ( (STOCK_CHECK == 'true') && (STOCK_ALLOW_CHECKOUT != 'true') ) {
  $products = $_SESSION['cart']->get_products();
  for ($i=0, $n=sizeof($products); $i<$n; $i++) {
    if (zen_check_stock($products[$i]['id'], $products[$i]['quantity'])) {
      zen_redirect(zen_href_link(FILENAME_SHOPPING_CART));
      break;
    }
  }
}
检查Coupon优惠码
if ($_SESSION['cc_id']) {
  $discount_coupon_query = "SELECT coupon_code
                            FROM " . TABLE_COUPONS . "
                            WHERE coupon_id = :couponID";

  $discount_coupon_query = $db->bindVars($discount_coupon_query, ':couponID', $_SESSION['cc_id'], 'integer');
  $discount_coupon = $db->Execute($discount_coupon_query);
}
确认账单地址，默认为用户默认地址
// if no billing destination address was selected, use the customers own address as default
if (!$_SESSION['billto']) {
  $_SESSION['billto'] = $_SESSION['customer_default_address_id'];
} else {
  // verify the selected billing address
  $check_address_query = "SELECT count(*) AS total FROM " . TABLE_ADDRESS_BOOK . "
                          WHERE customers_id = :customersID
                          AND address_book_id = :addressBookID";

  $check_address_query = $db->bindVars($check_address_query, ':customersID', $_SESSION['customer_id'], 'integer');
  $check_address_query = $db->bindVars($check_address_query, ':addressBookID', $_SESSION['billto'], 'integer');
  $check_address = $db->Execute($check_address_query);

  if ($check_address->fields['total'] != '1') {
    $_SESSION['billto'] = $_SESSION['customer_default_address_id'];
    $_SESSION['payment'] = '';
  }
}
初始发货模块，会生成当前选择的发货方式对象
require(DIR_WS_CLASSES . 'shipping.php');
$shipping_modules = new shipping($_SESSION['shipping']);
初始化统计模块
require(DIR_WS_CLASSES . 'order_total.php');
$order_total_modules = new order_total;
$order_total_modules->collect_posts(); // 先各自统计模块收集提交信息，判断是否正确，并完成相关设置工作
$order_total_modules->pre_confirmation_check(); // 汇总所有统计模块应增加或扣减的金额

总重量和总数量
$total_weight = $_SESSION['cart']->show_weight();
$total_count = $_SESSION['cart']->count_contents();
加载所有付款模块，获取它们的显示样式
require(DIR_WS_CLASSES . 'payment.php');
$payment_modules = new payment;
$flagOnSubmit = sizeof($payment_modules->selection());
出错控制
if (isset($_GET['payment_error']) && is_object(${$_GET['payment_error']}) && ($error = ${$_GET['payment_error']}->get_error())) {
  $messageStack->add('checkout_payment', $error['error'], 'error');
}
模板输出
<?php
  $selection = $payment_modules->selection();

  if (sizeof($selection) > 1) {
?>
<p class="important"><?php echo TEXT_SELECT_PAYMENT_METHOD; ?></p>
<?php
  } elseif (sizeof($selection) == 0) {
?>
<p class="important"><?php echo TEXT_NO_PAYMENT_OPTIONS_AVAILABLE; ?></p>

<?php
  }
?>

<?php
  $radio_buttons = 0;
  for ($i=0, $n=sizeof($selection); $i<$n; $i++) {
?>
<?php
    if (sizeof($selection) > 1) {
        if (empty($selection[$i]['noradio'])) {
 ?>
<?php echo zen_draw_radio_field('payment', $selection[$i]['id'], ($selection[$i]['id'] == $_SESSION['payment'] ? true : false), 'id="pmt-'.$selection[$i]['id'].'"'); ?>
<?php   } ?>
<?php
    } else {

?>
<?php echo zen_draw_hidden_field('payment', $selection[$i]['id'], 'id="pmt-'.$selection[$i]['id'].'"'); ?>
<?php
    }
?>
<label for="pmt-<?php echo $selection[$i]['id']; ?>" class="radioButtonLabel"><?php echo $selection[$i]['module']; ?></label>

<?php
    if (defined('MODULE_ORDER_TOTAL_COD_STATUS') && MODULE_ORDER_TOTAL_COD_STATUS == 'true' and $selection[$i]['id'] == 'cod') {
?>
<div class="alert"><?php echo TEXT_INFO_COD_FEES; ?></div>
<?php
    } else {
      // echo 'WRONG ' . $selection[$i]['id'];
?>
<?php
    }
?>
<br class="clearBoth" />

<?php
    if (isset($selection[$i]['error'])) {
?>
    <div><?php echo $selection[$i]['error']; ?></div>

<?php
    } elseif (isset($selection[$i]['fields']) && is_array($selection[$i]['fields'])) {
?>

<div class="ccinfo">
<?php
      for ($j=0, $n2=sizeof($selection[$i]['fields']); $j<$n2; $j++) {
?>
<label <?php echo (isset($selection[$i]['fields'][$j]['tag']) ? 'for="'.$selection[$i]['fields'][$j]['tag'] . '" ' : ''); ?>class="inputLabelPayment"><?php echo $selection[$i]['fields'][$j]['title']; ?></label><?php echo $selection[$i]['fields'][$j]['field']; ?>
<br class="clearBoth" />
<?php
      }
?>
</div>
<br class="clearBoth" />
<?php
    }
    $radio_buttons++;
?>
<br class="clearBoth" />
<?php
  }
?>
显示统计信息
<?php
  if (MODULE_ORDER_TOTAL_INSTALLED) {
    $order_totals = $order_total_modules->process();
?>
<?php $order_total_modules->output(); ?>
<?php
  }
?>
当$order_total_modules的output方法传入参数$return_html=true是，会使用模板文件tpl_modules_order_totals.php，否则使用直接构造方式输出HTML代码。

修改账单地址checkout_payment_address
与修改发货地址相类似
$addressType = "billto";
require(DIR_WS_MODULES . zen_get_module_directory('checkout_new_address'));
// if no billing destination address was selected, use their own address as default
if (!$_SESSION['billto']) {
  $_SESSION['billto'] = $_SESSION['customer_default_address_id'];
}

$addresses_count = zen_count_customer_address_book_entries();
主要还是依赖于checkout_new_address模块，来进行。

确认环节checkout_confirmation
保存付款方式和客户留言到会话
if (isset($_POST['payment'])) $_SESSION['payment'] = $_POST['payment'];
$_SESSION['comments'] = zen_db_prepare_input($_POST['comments']);
检查是否勾选了同意条款
if (DISPLAY_CONDITIONS_ON_CHECKOUT == 'true') {
  if (!isset($_POST['conditions']) || ($_POST['conditions'] != '1')) {
    $messageStack->add_session('checkout_payment', ERROR_CONDITIONS_NOT_ACCEPTED, 'error');
  }
}
信贷覆盖？需要研究用处
会直接设置付款功能失效。
if (!isset($credit_covers)) $credit_covers = FALSE;

//echo 'credit covers'.$credit_covers;

if ($credit_covers) {
  unset($_SESSION['payment']);
  $_SESSION['payment'] = '';
}
在includes/classes/order_total.php里，当总金额与将减少的金额相差在0.009以内并且不是免运费的情况下，设置$credit_covers=true
$difference = $order->info['total'] - $total_deductions;
      if ( $difference <= 0.009 && $_SESSION['payment'] != 'freecharger') {
        $credit_covers = true;
      }
初始化相应的付款模块，update_status方法用于确认此付款方式是否可能，通常该处的代码只检查是否满足zone的限制要求，除此之外，我们也可以增加一些其它的限制条件，比如金额过大时限制使用，或者强制要求使用SSL连接方式才能使用本付款方式。
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
当使用了Coupon时，设置用户访问来源
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
CUSTOMERS_REFERRAL_STATUS的值在后台设置

付款转向，这里有两种情况，一种是在线支付方式，一种是离线支付方式，对于在线支付方式，我们需要自动转向到相应的支付功能提供商的页面完成支付过程，而对于离线支付方式，zencart有一套标准的订单处理步骤，我们只需要转向到checkout_process便可。
if (isset($$_SESSION['payment']->form_action_url)) {
  $form_action_url = $$_SESSION['payment']->form_action_url;
} else {
  $form_action_url = zen_href_link(FILENAME_CHECKOUT_PROCESS, '', 'SSL');
}

禁止修改账单地址，通常的付款方式都会允许可修改账单地址，但有一些特殊的情况下会使用到此限制，你只要设置付款模块的flagDisablePaymentAddressChange变量=true即可，通常这可以在付款模块的in_special_checkout方法里按情况设置。
$flagDisablePaymentAddressChange = false;
if (isset($$_SESSION['payment']->flagDisablePaymentAddressChange)) {
  $flagDisablePaymentAddressChange = $$_SESSION['payment']->flagDisablePaymentAddressChange;
}

模板输出
再次确认过程，主要针对需要填写其它资料的付款方式，如直接输入信用卡完成付款的付款方式等。
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
当我们需要再次确认时，可以仿照下面的代码来编写，否则可以直接返回false即可。
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
付款转向参数，通过隐藏域的方法，向将转向的页面传递参数
<?php
  echo zen_draw_form('checkout_confirmation', $form_action_url, 'post', 'id="checkout_confirmation" onsubmit="submitonce();"');

  if (is_array($payment_modules->modules)) {
    echo $payment_modules->process_button();
  }
?>

普通离线支付方式checkout_process
直接嵌入checkout_prcess.php模块
require(DIR_WS_MODULES . zen_get_module_directory('checkout_process.php'));

// load the after_process function from the payment modules
  $payment_modules->after_process();

复位购物车和结算中使用的会话数据
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

在checkout_process.php模块中
限制多次提交订单，达3次以上将视为恶意提交，将转向到其它页面
if (!isset($_SESSION['payment_attempt'])) $_SESSION['payment_attempt'] = 0;
$_SESSION['payment_attempt']++;
$zco_notifier->notify('NOTIFY_CHECKOUT_SLAMMING_ALERT');
if ($_SESSION['payment_attempt'] > 3) {
  $zco_notifier->notify('NOTIFY_CHECKOUT_SLAMMING_LOCKOUT');
  $_SESSION['cart']->reset(TRUE);
  zen_session_destroy();
  zen_redirect(zen_href_link(FILENAME_TIME_OUT));
}

订单的生成过程
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

$_SESSION['order_number_created']保存新创建的订单号

神奇的订单类
初始化方法，即可以直接从购物车生成，也可以从指定订单生成。
当从购物车生成时，会自动提取当前的发货地址、账单地址，这样便可以让结算环节里的每一步都即时同步订单资料，使用结算环节变得非常容易实现。
除此之外，订单类还会自动处理coupon、自动转换币种、自动记录用户IP、自动记录用户留言。
通过订单号初始化的过程不具备以上功能。

九、三大模块
1、运输
2、付款
3、统计

所有国家的州/省列表
这个是osCommerce的插件，基础结构是相同的，总共包括305条记录，下载地址：http://www.oscommerce.com/community/contributions,1792
修改SQL代码，先删除DROP TABLE的语句，改为
DELETE FROM `zones`
再加上所有的INSERT INTO语句

如何调试
$debug_on，includes/modules/debug_blocks/目录下，product_info_prices.php调试价格，需取消tpl_product_info_display.php模板中的//require(DIR_WS_MODULES . '/debug_blocks/product_info_prices.php'); 注释、
shopping_cart_contents.php调试购物车，需取消pages/shopping_cart/header.php里的//  require(DIR_WS_MODULES . 'debug_blocks/shopping_cart_contents.php'); 注释

 
4、多语种的实现
语言类
语言切换
默认语言

语言文件的加载
zencart中的语言文件不是自动加载的，必须由页面功能代码里手工添加以下代码：
require(DIR_WS_MODULES . zen_get_module_directory('require_languages.php'));

通过require_languages.php语言控制文件来控制如何加载语言文件。
语言文件主要分成三种类型：语言总控制文件、全局语言定义文件、页面语言文件。我们介绍一下这三种语言文件的含义及其加载顺序。
1、语言总控制文件，优先级1级，此语言文件最先被加载，一般为includes/languages/<language>.php文件。该文件主要是针对语言级的格式及规范汇总，如日期时间格式、语言默认币种、称谓、基础名词的翻译等等。
语言总控制定义的详细内容请看“附录之语言总控制定义明细表”

2、全局语言定义文件，优先级2级，全局语言定义文件不只是单个文件，而一组文件，它们是由includes/languages/<language>.php加载的。以下为几个基本全局语言定义文件，这些文件加载所检查的目录都是相同的，首先检查includes/languages/<language>/<template>/模板目录下是否含有文件，有则加载，否则加载includes/languages/<language>/目录下的对应文件。
a) header.php 用于页面头部的常用功能定义
b) email_extras.php 邮件相关的内容
c) button_names.php 定义按钮名称及文件名
d) icon_names.php 图标名称及文件名
e) other_images_names.php 其它图片名称及文件名
f) credit_cards.php 信用卡
g) whos_online.php 在线
h) meta_tags.php META和Tag相关

3、页面语言文件，优先级3级，页面语言文件是最后被加载的。页面语言文件主要是针对当前页面定义相应语言常量。页面语言文件的加载分两种情况：
a) 当<template>模板目录下有符合的页面语言文件时
先加载includes/languages/<language>/<template>/<page>***.php分模板语言文件，再加载includes/languages/<language>/<page>***.php页面语言文件，主语言会覆盖分模板语言中相同的定义。
b) 当<template>目录下无符合的语言文件时
直接加载includes/languages/<language>/<page>***.php页面语言文件。
大家应该已经注意到页面语言文件的文件名为<page>***.php格式，只要以页面名称开头，以php为后缀的都被应为是当前页面语言文件。笔者在此认为这应该是zencart的一个bug，是否应该加载所有以<page>开头的语言文件，值得商榷。




Meta控制与显示
Meta相关文件
1、语言文件，由includes/languages/<language>.php自动载入
includes/languages/<language>/<template>/meta_tags.php
includes/languages/<language>/meta_tags.php
2、内容控制
includes/modules/meta_tags.php
3、页面模板，由模板牵头调用（2）内容控制，再显示出来
includes/templates/<template>/common/html_header.php
includes/templates/common/html_header.php

require(DIR_WS_MODULES . zen_get_module_directory('meta_tags.php')); 
?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" <?php echo HTML_PARAMS; ?>>
<head>
<title><?php echo META_TAG_TITLE; ?></title>
<meta http-equiv="Content-Type" content="text/html; charset=<?php echo CHARSET; ?>" />
<meta name="keywords" content="<?php echo META_TAG_KEYWORDS; ?>" />
<meta name="description" content="<?php echo META_TAG_DESCRIPTION; ?>" />



面包屑流程
A)  includes/init_includes/init_add_crumbs.php 初始化面包屑
加入主页，$breadcrumb->add(HEADER_TITLE_CATALOG, zen_href_link(FILENAME_DEFAULT));
1、当有$cPath和$cPath_array时，自动加入分类导航
2、当有产品过滤条件时，自动加入过滤条件至导航
3、当有$_GET['products_id']时，自动加入产品页面导航
B)  includes/modules/pages/<page>/header_php***.php文件
1、视情况加入
$breadcrumb->add(NAVBAR_TITLE_1, zen_href_link(FILENAME_ACCOUNT, '', 'SSL'));
$breadcrumb->add(NAVBAR_TITLE_2);
2、先$breadcrumb->reset(); 再加入
C) 显示，在includes/templates/<template>/common/tpl_main_page.php，或includes/templates/template_default/common/tpl_main_page.php，调用$breadcrumb->trail(BREAD_CRUMBS_SEPARATOR)显示出面包屑。
<?php if (DEFINE_BREADCRUMB_STATUS == '1' || (DEFINE_BREADCRUMB_STATUS == '2' && !$this_is_home_page) ) { ?>
    <div id="navBreadCrumb"><?php echo $breadcrumb->trail(BREAD_CRUMBS_SEPARATOR); ?></div>
<?php } ?>

后台 

文本编辑器
添加ckeditor下载http://www.zen-cart.com/index.php?main_page=product_contrib_info&products_id=1626
将下载的ckeditor解压后，可以看到有两个目录admin和editors，将admin目录下的所有文件复制到管理后台目录，将editors目录下的ckeditor文件夹复制到editors目录下，当编辑器有更新时只需要更新editors/ckeditor目录下的文件即可。进入后台configuration -> My store -> HTML Editor编辑选择新的编辑器，进行Category -> Categories/Products 下拉框TextEditor选择ckeditor。进入产品编辑即可看到已经可以使用新的编辑器了。
includes/init_includes/overriders/init_html_editor.php文件可以不复制，直接修正zencart一处Bug,在includes/init_includes/init_html_editor.php第一行：
if (!defined('DIR_WS_EDITORS')) define('DIR_WS_EDITORS', 'editors');
改为if (!defined('DIR_WS_EDITORS')) define('DIR_WS_EDITORS', 'editors/');



或tinyMCE编辑器http://www.zen-cart.com/index.php?main_page=product_contrib_info&products_id=269
下载ckeditor，下载地址http://ckeditor.com/download 或tinyMCE，下载地址http://www.tinymce.com/download/download.php 需下载jQuery版本源文件，解压至editors目录，即cheditor放置在editors/cheditor目录，tinyMCE放置在editors/tiny_mce目录下，zencart已经内置这些编辑器的支持代码，可以先测试使用像这样的网址：http://localhost/web/editors/ckeditor/_samples/index.html或者http://localhost/web/editors/tiny_mce/examples/index.html  ，
修改文件：includes/languages/<language>/extra_definitions/editors_list.php
添加：
if (!defined('EDITOR_HTMLAREA')) define('EDITOR_HTMLAREA', 'HTMLArea');
if (!defined('EDITOR_CKEDITOR')) define('EDITOR_CKEDITOR', 'CKEditor');
if (!defined('EDITOR_TINYMCE')) define('EDITOR_TINYMCE', 'tinyMCE');

SEO Url
修改includes/classees/seo_install.php
function install_settings(){
		$this->uninstall_settings();
		$sort_order_query = "SELECT MAX(sort_order) as max_sort FROM `".TABLE_CONFIGURATION_GROUP."`";
		$sort = $this->db->Execute($sort_order_query);
		$next_sort = $sort->fields['max_sort'] + 1;
		$insert_group = "INSERT INTO `".TABLE_CONFIGURATION_GROUP."` VALUES ('', 'SEO URLs', 'Options for Ultimate SEO URLs by Chemo', '".$next_sort."', '1')";
		$this->db->Execute($insert_group);
		$group_id = $this->db->insert_ID();

		foreach ($this->default_config as $key => $value){
			$sql = str_replace('GROUP_INSERT_ID', $group_id, $value['QUERY']);
			$this->db->Execute($sql);
		}

		$insert_cache_table = "CREATE TABLE " . TABLE_SEO_CACHE . " (
		  `cache_id` varchar(32) NOT NULL default '',
		  `cache_language_id` tinyint(1) NOT NULL default '0',
		  `cache_name` varchar(255) NOT NULL default '',
		  `cache_data` mediumtext NOT NULL,
		  `cache_global` tinyint(1) NOT NULL default '1',
		  `cache_gzip` tinyint(1) NOT NULL default '1',
		  `cache_method` varchar(20) NOT NULL default 'RETURN',
		  `cache_date` datetime NOT NULL default '0000-00-00 00:00:00',
		  `cache_expires` datetime NOT NULL default '0000-00-00 00:00:00',
		  PRIMARY KEY  (`cache_id`,`cache_language_id`),
		  KEY `cache_id` (`cache_id`),
		  KEY `cache_language_id` (`cache_language_id`),
		  KEY `cache_global` (`cache_global`)
		) ENGINE=MyISAM;"; // 原为TYPE=MyISAM;
		$this->db->Execute($insert_cache_table);
	} # end function	
} # end class
在admin/includes/languages/<language>.php
在define('BOX_CONFIGURATION_EZPAGES_SETTINGS', 'EZ-Pages Settings');之下加入
define('BOX_CONFIGURATION_SEO_URL', 'SEO URL');

执行SQL加入菜单
INSERT INTO admin_pages (page_key, language_key, main_page, page_params, menu_key, display_on_menu, sort_order) VALUES ('configSEOUrl', 'BOX_CONFIGURATION_SEO_URL','FILENAME_CONFIGURATION','gID=32','configuration','Y',26)

多图片支持
zencart原生支持多图片，初接触zencart系统的使用者可能丈二和尚摸不到头脑，因为我们在后台编辑产品看到只有一个提供上传图片/写入图片地址的地方，其实中zencart中实现多图片的功能比想像的还要简单，它是通过图片文件名自动匹配实现的。
控制是否支持多图片，默认开启，进入Catelog -> Product types ，选择 Product - General点击edit layout，修改参数Show Product Additional Images = true时，便会显示出多图片。
相当文件夹下，图片文件名规则：imagename***.jpg
如/images/products/image001.jpg，会匹配到/images/products/image001_001.jpg或/images/products/image0010001.jpg等图片文件。

邮件
1、新注册邮件、欢迎
由includes/modules/create_account.php里调用，使用email_template_welcome.html模板
if (trim(EMAIL_SUBJECT) != 'n/a') 决定是否发送邮件
会发送welcome_extra邮件

2、忘记密码
由pages/password_forgotten/header_php.php文件里调用，使用email_template_password_forgotten.html邮件模板
始终发送

3、新订单
通过order类的function send_order_email($zf_insert_id, $zf_mode)方法实现，传递给zen_mail的参数module为checkout_extra，使用email_template_checkout.html模板文件，
始终发送
会发送checkout_extra邮件

4、订单状态变更
在admin/orders.php里，调用email_template_order_status.html邮件模板
当notify参数为1时发送

5、联系我们
在pages/contact_us/header_php.php里实现，调用email_template_contact_us.html邮件模板文件
始终发送

6、Coupon
由admin/coupon_admin.php调用，使用email_template_coupon.html模板
始终发送
会发送coupon_extra邮件

7、GV mail
在admin/gv_mail.php里调用，使用email_template_gv_mail.html模板
始终发送
会发送gv_mail_extra邮件

8、GV queue
在admin/gv_queue.php里调用 ，使用email_template_gv_queue.html模板
始终发送
会发送gv_queue_extra邮件

9、GV send
在pages/gv_send/header_php.php，使用email_template_gv_send.html模板
始终发送
会发送gv_send_extra邮件

10、直接邮件
在admin/mail.php，使用email_template_direct_email.html模板，直接发送邮件给指定的顾客



11、低库存
在生成新订单时，order类的function send_order_email($zf_insert_id, $zf_mode)方法同时发磅低库存提醒邮件，传递给zen_mail的参数module为low_stock，使用email_template_low_stock.html邮件模板文件
当后台设置需发提醒并添加了收件地址时发送

12、订阅邮件
在admin/includes/modules/newsletter/newsletter.php，调用 email_template_newsletters.html模板

13、产品提醒
在admin/includes/modules/newsletter/product_notification.php里，调用  email_template_product_notification.html模板


14、告诉朋友
email_template_tell_a_friend.html，此功能已取消

15、用户授权
在admin/customers.php里，使用email_template_default.html模板

16、留评价
pages/product_reviews_write/header_php.php，使用reviews_extra作为module参数，没有email_template_reviews_extra.html模板存在，使用email_template_default.html模板


17、

	
发送邮件的任务实际是交给functions/functions_email.php中的zen_mail($to_name, $to_address, $email_subject, $email_text, $from_email_name, $from_email_address, $block=array(), $module='default', $attachments_list='' ) 方法执行的
Zen_mail再根据调用时的参数决定是使用文本构造方式还是使用模块，模块的提取是由function zen_build_html_email_from_template($module='default', $content='')方法实现的，方法会自动调用相应的邮件模板并将参数替换。
邮件模板位于email目录，所有文件以email_template_开头，以.html为后缀
查找模板文件的方法为，由指定的模板名称以email_template_<module>.html规则的文件名在email文件里查找，当以指定的模板名称找不到，会自动将模板名称里的extra或admin删除后，重新查找
Email目录支持分语言，在email文件夹下新建各语言代码的目录，使会自动查找相应的邮件模板，英语的在email根目录。
模板文件夹的查找顺序为
1、Emails/email_template_<current_page_base>.html
2、$block['EMAIL_TEMPLATE_FILENAME'].html
3、Emails/email_template_<module>.html 注：module已经将其中的_extra/_admin删除
4、Emails/email_template_default.html



调试邮件模板
在functions_email.php，将  if (!defined('EMAIL_SYSTEM_DEBUG')) define('EMAIL_SYSTEM_DEBUG','0');里的EMAIL_SYSTEM_DEBUG常量设置为preview即可。

决定是否发送邮件的条件：
后台
其它对应功能


价格原理

显示前的计算
基础价 zen_get_products_base_price($products_id)
特价 zen_get_products_special_price($products_id, true);
特价会查询表Specials，要求status=1,得到specials_new_products_price，当产品编号前四位为GIFT时，直接采用specials_new_products_price

售价 zen_get_products_special_price($products_id, false);
当第二参数为false时，在计算完specials_new_products_price后，查询products表的master_categories_id字段，再查询salemaker_sales表，查询条件有：
Sale_categories_all 存在master_categories_id
sale_status = '1'  状态开启
(sale_date_start <= now() or sale_date_start = '0001-01-01')  开始时间
(sale_date_end >= now() or sale_date_end = '0001-01-01')  结束时间
(sale_pricerange_from <= '" . $product_price . "' or sale_pricerange_from = '0')  价格范围
(sale_pricerange_to >= '" . $product_price . "' or sale_pricerange_to = '0')
无满足条件的记录时，直接返回special_price
以下计算会同时以special_price和基础价格两种结果，在最后才会决定采用哪个价格
当sale_deduction_type字段为
0：原价-sale_deduction_value字段值
1：原价-原价*sale_deduction_value/100
2：采用sale_deduction_value 作为价格
当sale_specials_condition字段为
0：返回以基础价计算的结果
1：直接返回特价（未按上述公式计算）
2：返回特价计算的结果
默认：返回特价

产品属性价格
当products_priced_by_attribute =1（后台产品参数Product Priced by Attributes:YES）
基础价会采用其中的产品属性价格，否则使用products_price

价格的显示
1、$show_special_price . $show_sale_price . $show_sale_discount 当基础价=0时，特价、售价、折扣标志
2、$show_normal_price . $show_special_price . $show_sale_price . $show_sale_discount 基础价、特价、售价、折扣标志

One Time Charge 单次购买费用
Product字段product_is_free 显示免费标记
Product 字段product_is_call 显示联系价格标记
永久免运费 Product字段product_is_always_free_shipping

价格与购买
在shopping_cart.php类的calculate方法里
当product_is_always_free_shipping=1和products_virtual=1时，不会加总产品重量，即产品重量会自动计为0。产品编号以GIFT为前缀也会算入免运费，其中free_shipping_item保存免运费数量，free_shipping_weight保存免运费重量。

数量区间价格
products_discount_type （Discount Type:）价格扣减类型
products_discount_type_from（Discount Priced from) = 0 特价系统将失效，其它采用特价进行扣减
通过表products_discount_quantity查询discount_quantity<=添加到购物车的数量
1、当查询不到符合条件的数据时，返回实际价格
2、当products_discount_type = 0时，返回实际价格
3、当products_discount_type = 1时，代表扣减百分比
4、当products_discount_type = 2时，代表实际金额
5、当products_discount_type = 3 时，代表扣减金额
返回的实际价格按sale_price、 special_price、normal_price排序继承。
产品表的products_mixed_discount_quantity字段（Discount Qty Applies to Mixed Attributes）当=0代表只计算当前产品的数量，做为区间满足数量，当不为0时，代表相同属性的产品可以合并计算数量作为区间数量。

价格与数量（在shopping_cart的get_products方法检查）
最大不超过 products_quantity_order_max 
最小起拍 products_quantity_order_min
批量 products_quantity_order_units，当设置为10时，代表输入数量必须为10的几次倍数量，不能有零数
products_quantity_mixed（Product Qty Min/Unit Mix） 代表属性产品数量将汇总进行计算

属性价格
product_attribute_is_free（Attribute is Free When Product is Free: ）当产品为免费时，标记属性为免费
attributes_discounted （Apply Discounts Used by Product Special/Sale:）属性参与折扣计算
文本类型属性
1、字数计费attributes_price_letters 单个字价格，attributes_price_letters_free免费字数
2、按单词计费，attributes_price_words 单个词价格，attributes_price_words_free免费词数

价格基数： attributes_price_factor,attributes_price_factor_offset，增加费用计算公式：$price/$special_price(当ATTRIBUTES_PRICE_FACTOR_FROM_SPECIAL=1时) * ($factor - $offset)
单次价格基数：attributes_price_factor_onetime，attributes_price_factor_onetime_offset，与价格基数相同，增加公式相同

数量价格区间： attributes_qty_prices（ Attributes Qty Price Discount:）输入文本格式应为： 数量1：价格1，数量2：价格2.....，此价格为增加价格
数量价格区间单次增加费用：attributes_qty_prices_onetime，与数量价格区间公式相同

单次价格：attributes_price_onetime（One Time: ），直接输入增加金额

属性重量变化：products_attributes_weight，products_attributes_weight_prefix



价格与顾客



特价
Saler marker

促销手段
GV
Coupon
Discount Price


Index_filter原理

购物车原理

Extra_configures使用

Extra_datafiles使用

Functions/Extra_functions的使用

Auto_loader的覆盖
Init_include的覆盖

地址格式
格式号	格式规则	默认影响国家	示例
1	$firstname $lastname
$streets
$city, $postcode
$state,$country	所有以下未提及国家	James Logan
496 Victoria Street 
Sydney, 2010
New South Wales, Australia
2	$firstname $lastname
$streets
$city, $state   $postcode
$country	United States	John Doe
123 Magnolia Street
Dallas, TX   55803-0034
United States
3	$firstname $lastname
$streets
$city
$postcode - $state, $country	Spain	Martina Gonzalez
General Castanos, 86 
Barcelona
08003 - Barcelona, Spain
提示：上面的城市和州都为Barcelona，西班牙使用省的称呼代替州
4	$firstname $lastname
$streets
$city ($postcode)
$country	Singapore	ohn Low
16 Whampoa Drive
Singapore (260042)
Singapore
5	$firstname $lastname
$streets
$postcode $city
$country	Austria, Germany	Heidi Kohler
Lentzeallee 194 
14195 Berlin
Germany
6	$firstname $lastname
$streets
$city
$state
$postcode
$country	United Kingdom	James Watson
1 St. Georges Business Centre
Portsmouth
PO1 3AX
United Kingdom
提示：上面的城市和州使用了相同的Portsmouth，在此处会省略 


