## 所有国家的州/省列表

这个是osCommerce的插件，基础结构是相同的，总共包括305条记录

下载地址：http://www.oscommerce.com/community/contributions,1792

修改SQL代码，先删除DROP TABLE的语句，改为

```php
DELETE FROM `zones`
```

再加上所有的INSERT INTO语句