## 全局变量

osCommerce 对于全局变量的使用十分普遍,包括多货币、多语言、浏览历史、消息堆栈以 及付款流程在内(包括但不只限于)的这些功能都广泛使用了全局变量,所以了解这些全局变量对于我们进行开发将是非常有帮助的。

### 全局变量的存续

全局变量要求在用户的整个会话中有效,也就是说即使用户在切换到另一个页面时,也能够还原 此前的全局变量数据。但 PHP 语言本身是不支持会话级别的全局变量,所以 osCommerce 在处理 全局变量的存继时,使用了如下的方法来实现:

includes/application_top.php[247]
```php
// create the shopping cart & fix the cart if necesary
if (tep_session_is_registered('cart') && is_object($cart)) {
  if (PHP_VERSION < 4) {
    $broken_cart = $cart;
    $cart = new shoppingCart;
    $cart->unserialize($broken_cart);
  }
} else {
  tep_session_register('cart');
  $cart = new shoppingCart;
}
```

可以看出 osCommerce 使用 SESSION 来注册并还原全局变量,要支持这样的操作还必须将 PHP 的参数 register_globals 设置为 true,所以 osCommerce 在安装时会检测用户系统的 register_globals 属性,如果未设置将无法继续安装。

### 基础类型变量

- $SID:SESSION ID
- $SESSION_SSL_ID:SSL 连接方式的 SESSION ID 
- $http_user_agent:浏览器信息
- $ip_address:浏览 IP
- $PHP_SELF:当前 PHP 文件
- $cart:购物车对象
- $currencies:货币对象
- $currency:当前币种
- $lng:语言对象 
- $language:当前语言目录(可以是像“english”的完整单词,也可以像“eng”的缩写,是用户在 管理后台设置的)
- $languages_id:当前语言 ID
- navigation:浏览历史对象
- $current_category_id:当前浏览的产品分类 ID(在分类页面和产品详细内容面有效) 
- $breadcrumb:面包屑导航对象
- $messageStack:消息对象

### 用户相关变量

- $customer_id:用户 ID 
- customer_default_address_id:用户默认地址 ID 
- customer_first_name:用户名字 
- customer_country_id:用户所属国家 ID 
- customer_zone_id:用户所属省/区 ID

> 提示:
> 
> 用户相关变量只有在用户登陆后才会生效。

### 订单相关变量

- $order:订单对象
- $cartID:购物车 ID 
- $sendto:收货地址 ID 
- $billto:账单地址 ID 
- $shipping:发货方式 
- $payment:付款方式 
- $free_shipping:是否 FREE SHIPPING 
- $comments:用户备注 
- $total_weight:产品总重量 
- $total_count:产品总数量 
- $shipping_weight:发货所计的重量 
- $shipping_num_boxes:产品包裹数量

> 提示:
> 
> 订单相关变量只有当用户进入订单流程后才会生效。
