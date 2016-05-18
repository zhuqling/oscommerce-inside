## 修改产品页标题

本节将介绍如何将产品页面的标题(页面的标题),以及 TITLE(浏览器的标题)修改成当前产 品名称。

文件:product_info.php

查找(第 18 行):

`$product_check = tep_db_fetch_array($product_check_query);`

在其后追加如下代码:

```php
$query_title = tep_db_query(sprintf(‘select products_name from %s inner join %s
using(products_id) where products_id=%d and language_id=%d and
products_status=1’,TABLE_PRODUCTS,TABLE_PRODUCT,
(int)$HTTP_GET_VARS['products_id'], (int)$languages_id));
$result_title = tep_db_fetch_array($query_title);
```

查找: 

`<title><?php echo TITLE; ?></title>`

替换为:

`<title><?php echo TITLE,’ - ’,$result_title[‘products_name’]; ?></title>`
