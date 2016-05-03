## 多币种的支持

多币种的支持由 currencies 类支援,我们先来看执行过程。

### 前提

程序预处理 includes/application_top.php[260]

```php
require(DIR_WS_CLASSES . 'currencies.php');
$currencies = new currencies();
```

此处初始化 currencies 全局对象$currencies。$currencies 对象由用户定义的 currencies 表生成了币 种数组,其结构如下:


了解了 currencies 数组结构后,我们再看接下来如果切换币种。

程序预处理 includes/application_top.php[291]

```php
if (!tep_session_is_registered('currency') || isset($HTTP_GET_VARS['currency']) || ( (USE_DEFAULT_LANGUAGE_CURRENCY == 'true') && (LANGUAGE_CURRENCY != $currency) ) ) {
  if (!tep_session_is_registered('currency')) tep_session_register('currency');
if (isset($HTTP_GET_VARS['currency'])) {
    if (!$currency = tep_currency_exists($HTTP_GET_VARS['currency'])) $currency =
(USE_DEFAULT_LANGUAGE_CURRENCY == 'true') ? LANGUAGE_CURRENCY : DEFAULT_CURRENCY;
  } else {
     $currency = (USE_DEFAULT_LANGUAGE_CURRENCY == 'true') ? LANGUAGE_CURRENCY : DEFAULT_CURRENCY;
  }
}
```

此段代码与多语言切换的代码有一曲同工之妙。这里的代码风格与语言切换是一致的。

通过$HTTP_GET_VARS['currency']传递币种的 code 值来转换币种,第 295 行代码指定了全局 $currency 变量使用$HTTP_GET_VARS['currency']的 code 值,在切换之前通过$currencies 的 is_set 方法来确定所设置的值是正确性。

在默认情况下,分成两种情况,一是如有全局的$currency 变量时,会继续使用它的值,使得 币种可以持久作用。

每二种情况又分成两个情况,在 admin 后台有设置“是否使用语种的币种 (USE_DEFAULT_LANGUAGE_CURRENCY)”与“默认币种(DEFAULT_CURRENCY)”的选 项,如果设置了 USE_DEFAULT_LANGUAGE_CURRENCY,则会使用语言文件里的 LANGUAGE_CURRENCY 值。


全局语言定义 includes/languages/english.php[39]

```php
define('LANGUAGE_CURRENCY', 'USD');
```

否则,使用设置为 DEFAULT_CURRENCY。

### 显示效果

产品详细页 product_info.php[78]

```php
$products_price = '<s>' . $currencies->display_price($product_info['products_price'], tep_get_tax_rate($product_info['products_tax_class_id'])) . '</s> <span class="productSpecialPrice">' . $currencies->display_price($new_price, tep_get_tax_rate($product_info['products_tax_class_id'])) . '</span>';
```

币种类 includes/classes/currencies.php[72]

```php
function display_price($products_price, $products_tax, $quantity = 1) {
return $this->format($this->calculate_price($products_price, $products_tax,
$quantity));
}
```

$currencies 的方法 display_price,通过传入要显示的价格,税率,数量,则会返回格式化的价格文本。

其中 calculate_price 方法的作用是计算价格、税率与数量的结果,format 方法通过全局 $currency 变量对应的币种数组数据来格式化货币文本。
