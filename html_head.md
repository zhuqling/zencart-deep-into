# HTML头部

HTML头部包括了页面标题、Meta信息、ICON图标、CSS样式表、JS代码等，HTML头部文件通常使用位于includes/templates/<template>/common/html_header.php或includes/templates/template_default/common/html_header.php文件。

HTML头部初始化的顺序如下：

1. 首先加载Meta内容控制文件includes/modules/meta_tags.php，此文件控制如何定义不同页面的标题和Meta常量。

2. 显示Title、Meta内容（包括keywords和description）

3. 根据后台设置确定是否屏蔽搜索引擎收录

```php
<?php if (defined('ROBOTS_PAGES_TO_SKIP') && in_array($current_page_base,explode(",",constant('ROBOTS_PAGES_TO_SKIP'))) || $current_page_base=='down_for_maintenance' || $robotsNoIndex === true) { ?>
<meta name="robots" content="noindex, nofollow" />
<?php } ?>
```

ROBOTS_PAGES_TO_SKIP常量在meta_tags.php里定义，它控制了哪些页面需要加入上述的屏蔽搜索引擎收录的代码。

4. 显示收藏及显示用ICON图标

图标地址由常量FAVICON定义，同样在meta_tags.php里控制。

5. 显示标准地址

是否显示标准地址是由初始脚本init_canonical.php控制的。

6. 加载CSS样式表文件

在zencart中CSS样式表是有区分的，是否加载CSS样式表取决于样式表的文件名。

CSS样式表文件通常放置在includes/templates/<template>/css/或includes/templates/template_default/css/目录下。

| 文件名格式 | 说明 | 示例 |
|---|---|---|
| style***.css | 全局模板 | stylesheet.css |
| <language>_stylesheet.css | 语言相关 | en_stylesheet.css |
| <page>.css | 访问页面相关 | shoppingcart.css |
| <language>_<page>.css | 语言和页面相关 | en_account.css |
| c_<cPath>.css | 分类路径相关 | c_123.css |
| <language>_c_<cPath>.css | 语言和分类路径 | en_c_123.css |
| m_<manufacture_id>.css | 供应商相关 | m_88.css |
| <language>_m_<manufacture_id>.css | 语言和供应商 | en_m_88.css |
| p_<product_id>.css | 产品相关 | p_100.css |
| <language>_p_<product_id>.css | 语言和产品相关 | en_p_100.css |

7. 用于打印的CSS样式

当打印页面时，使用的CSS样式，要求文件名格式为print***.css。

8. 加载全局JS代码文件

全局JS代码放置在includes/templates/<template>/jscript/或includes/templates/template_default/jscript/目录下，全局JS代码会在所有页面加载。文件名格式为：jscript_***.js

9. 加载页面JS代码

页面JS代码仅针对当前页面使用，放置在includes/modules/<page>/jscript/目录下，文件名格式为：jscript_***.js

10. 加载全局JS的PHP文件

此文件用于在PHP程序中合成JS代码，通常用于动态JS代码的实现，文件放置位置的与全局JS代码文件相同，文件名格式为jscript_***.php。

11. 加载页面JS的PHP文件

用于针对某个页面的动态JS代码，与页面JS代码文件放置在同一目录下，文件名格式为jscript_***.php。
