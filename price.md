# 价格原理

显示前的计算

1. 基础价 `zen_get_products_base_price($products_id)`
2. 特价 `zen_get_products_special_price($products_id, true);`

    特价会查询表Specials，要求status=1,得到specials_new_products_price，当产品编号前四位为GIFT时，直接采用specials_new_products_price

3. 售价 `zen_get_products_special_price($products_id, false);`

    当第二参数为false时，在计算完specials_new_products_price后，查询products表的master_categories_id字段，再查询salemaker_sales表，查询条件有：

    - Sale_categories_all 存在master_categories_id
    - sale_status = '1'  状态开启
    - (sale_date_start <= now() or sale_date_start = '0001-01-01')  开始时间
    - (sale_date_end >= now() or sale_date_end = '0001-01-01')  结束时间
    - (sale_pricerange_from <= '" . $product_price . "' or sale_pricerange_from = '0')  价格范围
    - (sale_pricerange_to >= '" . $product_price . "' or sale_pricerange_to = '0')

    无满足条件的记录时，直接返回special_price

以下计算会同时以special_price和基础价格两种结果，在最后才会决定采用哪个价格

当sale_deduction_type字段为：

- 0：原价-sale_deduction_value字段值
- 1：原价-原价*sale_deduction_value/100
- 2：采用sale_deduction_value 作为价格

当sale_specials_condition字段为：

- 0：返回以基础价计算的结果
- 1：直接返回特价（未按上述公式计算）
- 2：返回特价计算的结果
- 默认：返回特价

### 产品属性价格

当products_priced_by_attribute =1（后台产品参数Product Priced by Attributes:YES）

基础价会采用其中的产品属性价格，否则使用products_price

### 价格的显示

1. $show_special_price . $show_sale_price . $show_sale_discount 当基础价=0时，特价、售价、折扣标志
2. $show_normal_price . $show_special_price . $show_sale_price . $show_sale_discount 基础价、特价、售价、折扣标志


### One Time Charge 单次购买费用

- Product 字段product_is_free 显示免费标记
- Product 字段product_is_call 显示联系价格标记
- 永久免运费 Product字段product_is_always_free_shipping

### 价格与购买

在shopping_cart.php类的calculate方法里，当product_is_always_free_shipping=1和products_virtual=1时，不会加总产品重量，即产品重量会自动计为0。

产品编号以GIFT为前缀也会算入免运费，其中free_shipping_item保存免运费数量，free_shipping_weight保存免运费重量。

### 数量区间价格

- products_discount_type （Discount Type:）价格扣减类型
- products_discount_type_from（Discount Priced from) = 0 特价系统将失效，其它采用特价进行扣减

通过表products_discount_quantity查询discount_quantity<=添加到购物车的数量

1. 当查询不到符合条件的数据时，返回实际价格
2. 当products_discount_type = 0时，返回实际价格
3. 当products_discount_type = 1时，代表扣减百分比
4. 当products_discount_type = 2时，代表实际金额
5. 当products_discount_type = 3 时，代表扣减金额

返回的实际价格按sale_price、 special_price、normal_price排序继承。

产品表的products_mixed_discount_quantity字段（Discount Qty Applies to Mixed Attributes）当=0代表只计算当前产品的数量，做为区间满足数量，当不为0时，代表相同属性的产品可以合并计算数量作为区间数量。

### 价格与数量（在shopping_cart的get_products方法检查）

- 最大不超过 products_quantity_order_max 
- 最小起拍 products_quantity_order_min
- 批量 products_quantity_order_units，当设置为10时，代表输入数量必须为10的几次倍数量，不能有零数
products_quantity_mixed（Product Qty Min/Unit Mix） 代表属性产品数量将汇总进行计算

### 属性价格

- product_attribute_is_free（Attribute is Free When Product is Free: ）当产品为免费时，标记属性为免费
- attributes_discounted （Apply Discounts Used by Product Special/Sale:）属性参与折扣计算

#### 文本类型属性

1. 字数计费attributes_price_letters 单个字价格，attributes_price_letters_free免费字数
2. 按单词计费，attributes_price_words 单个词价格，attributes_price_words_free免费词数


- 价格基数： attributes_price_factor,attributes_price_factor_offset，增加费用计算公式：$price/$special_price(当ATTRIBUTES_PRICE_FACTOR_FROM_SPECIAL=1时) * ($factor - $offset)
- 单次价格基数：attributes_price_factor_onetime，attributes_price_factor_onetime_offset，与价格基数相同，增加公式相同

- 数量价格区间： attributes_qty_prices（ Attributes Qty Price Discount:）输入文本格式应为： 数量1：价格1，数量2：价格2.....，此价格为增加价格
- 数量价格区间单次增加费用：attributes_qty_prices_onetime，与数量价格区间公式相同

- 单次价格：attributes_price_onetime（One Time: ），直接输入增加金额

- 属性重量变化：products_attributes_weight，products_attributes_weight_prefix



价格与顾客

特价
Saler marker

促销手段
GV
Coupon
Discount Price