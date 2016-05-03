## Banner广告

Banner分布图

zencart中广告的应用主要体现在Banner的管理上，无论是头部、底部，还是左边栏、右边栏都有Banner的踪影。Banner的分组情况如下：

1. 头部区域为三组：SHOW_BANNERS_GROUP_SET1、SHOW_BANNERS_GROUP_SET2、SHOW_BANNERS_GROUP_SET3
2. 底部区域为三组：SHOW_BANNERS_GROUP_SET4、SHOW_BANNERS_GROUP_SET5、SHOW_BANNERS_GROUP_SET6
3. 侧边栏区域：SHOW_BANNERS_GROUP_SET7、SHOW_BANNERS_GROUP_SET8、SHOW_BANNERS_GROUP_SET_ALL

```php
<?php
  if (SHOW_BANNERS_GROUP_SET1 != '' && $banner = zen_banner_exists('dynamic', SHOW_BANNERS_GROUP_SET1)) {
    if ($banner->RecordCount() > 0) {
?>
<div id="bannerOne" class="banners"><?php echo zen_display_banner('static', $banner); ?></div>
<?php
    }
  }
?>
```

### zen_banner_exists($action, $identifier)

$action可选值有'dynamic'和'static'，当为dynamic时，代表是广告组$identifier参数应填入Banner组，以随机位置返回该分组的所有Banner。当为static时，代表是指定Banner，$identifier参数应填入Banner ID。

### zen_display_banner($action, $identifier)

$action可选值也是dynamic和static，当为dynamic时，$identifier应指定广告组，查询数据库后输出所有指定组的广告，当为static时，如将zen_banner_exists函数返回的Banner数据传入$identifier参数，则不需要查询数据库直接按输入Banner输出，否则$identifier被认定为Banner ID，查询数据库后输出该Banner。
