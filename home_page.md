## 首页

### 允许输入参数

当$cPath有输入时，判断分类类型products（有产品或无子分类也无产品）/nested（无产品但有子分类），赋值给变量$category_depth。

1. 加载$define_page，静态HTML等文件
2. 加载语言


- $_GET['typefilter']参数，标准使用includes/index_filters/default_filter.php，
该文件中有参数有：

- $_GET['sort'](仅前三位)，
- $_GET['alpha_filter_id']（且值>0，产品名称的第一个字母）
- $alpha_sort = " and pd.products_name LIKE '" . chr((int)$_GET['alpha_filter_id']) . "%' ";
- $_GET['manufacturers_id']（非空，供应商），
- $_GET['filter_id']（分类ID），
- $select_column_list变量定义查询SQL字段，

    - 当指宝了供应商manufacturers_id时，filter_id为分类ID，
    - 当未指定供应商ID时，$_GET['filter_id']为供应商ID，分类为当前分类ID，
    - 两者都未指定时，仅使用当前分类ID过滤。

- PRODUCT_LISTING_DEFAULT_SORT_ORDER为默认排序方式

- $column_list变量为显示的列，可选值有文本类型：

    - PRODUCT_LIST_MODEL 产品型号
    - PRODUCT_LIST_NAME 产品名称
    - PRODUCT_LIST_MANUFACTURER 供应商
    - PRODUCT_LIST_QUANTITY 数量
    - PRODUCT_LIST_IMAGE 实为标题
    - PRODUCT_LIST_WEIGHT 重量
    - PRODUCT_LIST_PRICE 产品排序（p.products_price_sorter）

- PRODUCT_LIST_FILTER >0是否有过滤条件，当$_GET['manufacturers_id']有值时，使用供应商过滤查询出分类，否则使用当前分类过滤查询出供应商。

- 为nested类型输出 tpl_index_categories.php模板，当为products类型输出tpl_index_product_list.php模板，其它输出tpl_index_default.php模板，

- $current_categories_description保存当前分类描述，

```php
if ($categories_image = zen_get_categories_image($current_category_id)) {
?>
<div id="categoryImgListing" class="categoryImg"><?php echo zen_image(DIR_WS_IMAGES . $categories_image, '', SUBCATEGORY_IMAGE_TOP_WIDTH, SUBCATEGORY_IMAGE_TOP_HEIGHT); ?></div>
<?php
}
```

输出分类图片，

tpl_modules_category_row.php模板输出所有子分类，它又调用category_row.php模块文件得到结果，组建$list_box_contents[$row][$col]数组，并交由tpl_columnar_display.php模板完全输出。

```php
$list_box_contents[$row][$col] = array('params' => 'class="categoryListBoxContents"' . ' ' . 'style="width:' . $col_width . '%;"',
                                           'text' => '<a href="' . zen_href_link(FILENAME_DEFAULT, $cPath_new) . '">' . zen_image(DIR_WS_IMAGES . $categories->fields['categories_image'], $categories->fields['categories_name'], SUBCATEGORY_IMAGE_WIDTH, SUBCATEGORY_IMAGE_HEIGHT) . '<br />' . $categories->fields['categories_name'] . '</a>');

```

`includes/classes/db/mysql/define_queries.php` 定义了查询语句
