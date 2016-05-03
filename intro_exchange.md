## 税率的支持 

### 数据库分析

首先来看有关税率的数据库关系图。

由图可以看出,最后计算税率的是 tax_rates 表,而 geo_zone,tax_class 都是为了更好的管理税率而设置的。

税率由税类进行分类管理,税类即对各种税区设置税率的方法。至于关联到产品最 后是否要计算税率,以及税率的多少,是由 products 表的 products_tax_class_id 字段进行关联。


### 税率函数分析

*function tep_get_tax_rate($class_id, $country_id, $zone_id)*

功能:获取税率

参数:

- $class_id:税类
- $country_id:国家
- $zone_id:省份/区

当未设置$country_id 和$zone_id 时,默认使用当前用户的所在国家和区。

> 提示:
> 当一个税类具多个税率时,计算方法为:所有税率的重复相乘。

*function tep_get_tax_description($class_id, $country_id, $zone_id)*

功能:获取税率描述

参数:

- $class_id:税类
- $country_id:国家
- $zone_id:省份/区

> 提示:
> 当一个税类具多个税率时,将返回所有税率的描述相加。

*function tep_add_tax($price, $tax)*

功能:加总产品价格与税率

参数:
- $price:产品价格
- $tax:税率

> 此函数调用了下面讲的 tep_calculate_tax 方法得到税额后,再与价格相加并返回。

> *提示:*
> 此函数的结果还取决于configuration-> My Store->Display Prices with Tax项的设置,只有该项为 true 时,才会加总税额。

*function tep_calculate_tax($price, $tax)*

功能:计算税率

参数:
- $price:产品价格
- $tax:税率

*function tep_display_tax_value($value, $padding)*

功能:显示税额

> 提示:
> 此函数的作用在于使用指定的样式格式化税率,决定样式的设置项为:configuration-> My Store->Tax Decimal Places,此项设置决定了最终显示的数值格式。

### 程序分析

税率的显示 product_info.php[80]
```php
$products_price = $currencies->display_price($product_info['products_price'], tep_get_tax_rate($product_info['products_tax_class_id']));
```

以及 shopping_cart.php[152]

```php
$info_box_contents[$cur_row][] = array('align' => 'right',
                                     'params' => 'class="productListing-data"
valign="top"',
                                     'text' => '<b>' .
$currencies->display_price($products[$i]['final_price'],
tep_get_tax_rate($products[$i]['tax_class_id']), $products[$i]['quantity']) .
'</b>');
```

通过 tep_get_tax_rate 函数,得到税率,然后传入$currencies 的 display_price 方法,这个方法通过 调用 tep_add_tax 加总价格税额,并使用适当的样式格式化输出。

### 税率的执行

大家都知道,最终的价格是多少,取决于购物车的计算以及最终计单的生成,所以税率的计
算首先要放在了购物车类里,然后再是订单类。

购物车类 includes/classes/shopping_cart.php[262]

```php
$products_tax = tep_get_tax_rate($product['products_tax_class_id']);
```

includes/classes/shopping_cart.php [272]

```php
$this->total += $currencies->calculate_price($products_price, $products_tax, $qty);
```

这样的代码是不是很熟悉了呢?本人在此就不再累赘啦。

至于$currencies 的 calculate_price 方法与上面的 display_price 原理是一样的,不同之处在于
display_price 会输出价格,而 calculate_price 只负责计算。

订单类 includes/classes/order.php[284]

```php
$this->products[$index] = array('qty' => $products[$i]['quantity'],
                                 'name' => $products[$i]['name'],
                                 'model' => $products[$i]['model'],
                                 'tax' =>
tep_get_tax_rate($products[$i]['tax_class_id'], $tax_address['entry_country_id'],
$tax_address['entry_zone_id']),
                                 'tax_description' =>
tep_get_tax_description($products[$i]['tax_class_id'],
$tax_address['entry_country_id'], $tax_address['entry_zone_id']),
                                 'price' => $products[$i]['price'],
                                 'final_price' => $products[$i]['price'] +
$cart->attributes_price($products[$i]['id']),
                                 'weight' => $products[$i]['weight'],
                                 'id' => $products[$i]['id']);
```

第 287 行是得到单个产品税率的代码,第 288 行得于税率的描述。

读者可以注意到,这里计算税率使用的是用户的国家及区信息,而不是收货方的信息。

订单类 includes/classes/order.php [312]

```php
$shown_price = $currencies->calculate_price($this->products[$index]['final_price'],
$this->products[$index]['tax'], $this->products[$index]['qty']);
$this->info['subtotal'] += $shown_price;
```

然后便是金额小计的部分,使用了$currencies 的 calculate_price 方法,同时传入刚得到的税率 值作为参数。

第 317-331 行的代码都是负责显示的,而显示则区分了包含税率与不包含税率两种情形,其 中上半部分说明了包含税率应如何显示。

订单类 includes/classes/order.php [336]

```php
if (DISPLAY_PRICE_WITH_TAX == 'true') {
       $this->info['total'] = $this->info['subtotal'] +
$this->info['shipping_cost'];
} else {
       $this->info['total'] = $this->info['subtotal'] + $this->info['tax'] +
$this->info['shipping_cost'];
}
```

这段代码考虑是否将税额包含在总计里。

最后税率信息又是怎么写入数据库的呢?让我们接着来看。

订单处理 checkout_process.php[186]

```php
$sql_data_array = array('orders_id' => $insert_id,
                       'products_id' => tep_get_prid($order->products[$i]['id']),
                       'products_model' => $order->products[$i]['model'],
                       'products_name' => $order->products[$i]['name'],
                       'products_price' => $order->products[$i]['price'],
                       'final_price' => $order->products[$i]['final_price'],
                       'products_tax' => $order->products[$i]['tax'],
                       'products_quantity' => $order->products[$i]['qty']);
tep_db_perform(TABLE_ORDERS_PRODUCTS, $sql_data_array);
```

将所有产品的税率写入最终的订单明细里,这样在查看订单的时候就可以显示产品的税额了。
