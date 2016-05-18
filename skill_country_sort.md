## 调整国家列表的排序

将用户最常用的国家调整到国家列表的前列,会使用户在选择国家时更加方便。

如果想修改整个 osCommerce 系统的所有国家列表,可以按以下方式修改: 

页面包括:

- creat_account.php
- checkout_payment_address.php
- checkout_shipping_address.php
- address_book_process.php

1、修改文件:includes/functions/html_output.php

查找(约第 289 行,位于函数 tep_get_country_list 里):

`$countries_array = array((array('id' => '', 'text' => PULL_DOWN_DEFAULT));`

修改为以下代码:

```php
$countries_array = array(array('id' => '', 'text' => PULL_DOWN_DEFAULT),array('id' => '223', 'text' => 'United States')); 
// 在将“美国”提到仅次于“please select”下的位 置,新添加的 223 是美国的 ID 值,如果想再添加其它国家,只需要将该国家的 ID 及名称加入数组即可
```

如果只想修改单个页面,而又不想影响到其它页面时,并且只想将某个国家做为默认的选择,应使用下面的方法修改:

1、以新建用户为例,修改 create_account.php 查找(大约第 442 行):

`echo tep_get_country_list('country')`

替换成以下代码:

`echo tep_get_country_list('country', '223') // 将“美国”作为默认选择项`
