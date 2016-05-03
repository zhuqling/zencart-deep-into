## 如何调试

1. $debug_on，includes/modules/debug_blocks/目录下
2. product_info_prices.php调试价格，需取消tpl_product_info_display.php模板中的`//require(DIR_WS_MODULES . '/debug_blocks/product_info_prices.php'); `注释
3. shopping_cart_contents.php调试购物车，需取消pages/shopping_cart/header.php里的`//  require(DIR_WS_MODULES . 'debug_blocks/shopping_cart_contents.php'); `注释
