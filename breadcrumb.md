# 面包屑流程

1. includes/init_includes/init_add_crumbs.php 初始化面包屑

加入主页，$breadcrumb->add(HEADER_TITLE_CATALOG, zen_href_link(FILENAME_DEFAULT));

	1. 当有$cPath和$cPath_array时，自动加入分类导航
	2. 当有产品过滤条件时，自动加入过滤条件至导航
	3. 当有$_GET['products_id']时，自动加入产品页面导航

2. includes/modules/pages/<page>/header_php***.php文件

	1. 视情况加入

```php
	$breadcrumb->add(NAVBAR_TITLE_1, zen_href_link(FILENAME_ACCOUNT, '', 'SSL'));
	$breadcrumb->add(NAVBAR_TITLE_2);
```

	2. 先$breadcrumb->reset(); 再加入

3. 显示，在includes/templates/<template>/common/tpl_main_page.php，或includes/templates/template_default/common/tpl_main_page.php，调用$breadcrumb->trail(BREAD_CRUMBS_SEPARATOR)显示出面包屑。

```php
<?php if (DEFINE_BREADCRUMB_STATUS == '1' || (DEFINE_BREADCRUMB_STATUS == '2' && !$this_is_home_page) ) { ?>
    <div id="navBreadCrumb"><?php echo $breadcrumb->trail(BREAD_CRUMBS_SEPARATOR); ?></div>
<?php } ?>
```
