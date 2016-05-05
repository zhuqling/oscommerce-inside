## 编写付款模块

在编写我们的付款模块之前,先介绍一下 Paypal IPN(即时付款通知),参考一下 paypal 模块
是如何处理付款返回信息,将对我们编写自己的付款模块非常有帮助。

### Paypal IPN介绍

在前面 Paypal 标准付款模块的 process_button 方法里,我们提到了 notify_url 的值,当时我们 将其赋值为:“catalog/ext/modules/payment/paypal/standard_ipn.php”,所以我们的 Paypal IPN 处理 分析工作就从 standard_ipn.php 开始。

路径的处理 /ext/modules/payment/paypal/standard_ipn.php [13]

```php
chdir('../../../../');
require('includes/application_top.php');
```

由于我们需要根据返回信息更新数据库的数据,所以需要包含 application_top.php 文件,而 standard_ipn.php 文件的存放路径是/ext/modules/payment/paypal/,所以可以看到这里使用了一个巧 妙的处理方法:通过使用 chdir('../../../../')切换当前路径后,就可以像在主目录的其它文件一样直接 包括 includes/application_top.php,不需要做任何其它修改。

接收参数 /ext/modules/payment/paypal/standard_ipn.php [16]

```php
$parameters = 'cmd=_notify-validate';

reset($HTTP_POST_VARS);
while (list($key, $value) = each($HTTP_POST_VARS)) {
  $parameters .= '&' . $key . '=' . urlencode(stripslashes($value));
}
```

设置 cmd 值为_notify-validate,并将返回的数据附加为参数,与 cmd 值一起回传到 Paypal 网关。

> 提示:
>
> cmd 值为 Paypal 服务的关键字,通过设置 cmd 为不同的值可以实现不同的功能,例如:设置_xclick 时,表示要制作一个付款按钮,又如:设置_cart 时,表示要使用 Paypal 的购物车功能。

网关的切换 /ext/modules/payment/paypal/standard_ipn.php [23]

```php
if (MODULE_PAYMENT_PAYPAL_STANDARD_GATEWAY_SERVER == 'Live') {
  $server = 'www.paypal.com';
} else {
  $server = 'www.sandbox.paypal.com';
}
```

在 Paypal 付款模块的参数一节,我们讲到 Paypal 模块提供了普通模式和调试模式,所以在这里也 要相应的切换到正确的网关。

初始化 SSL 连接 /ext/modules/payment/paypal/standard_ipn.php [33]

```php
if ( (PHP_VERSION >= 4.3) && ($fp = @fsockopen('ssl://' . $server, 443, $errno, $errstr, 30)) ) {
   $fsocket = true;
} elseif (function_exists('curl_exec')) {
   $curl = true;
} elseif ($fp = @fsockopen($server, 80, $errno, $errstr, 30)) {
   $fsocket = true;
}
```

Paypal 网关需要使用 SSL 安全连接,所以需要考虑使用 SOCKET 连接方式或者 CURL 连接方式。 除了这两种方式,上面的代码还提供了一种非 SSL 连接方式。

> 提示
> 
> 据笔者观察,Paypal 网关只提供 SSL 安全连接,并不支持普通连接,所以上面的第三种情况永远 都不会有效(有一种可能就是 Paypal 网关在此之前曾经提供过普通连接,而之后就取消了)。
> 
> 根据经验,CURL 的方式比 SOCKET 方式运用的更普遍,其中一个原因就是如果你使用的是空间, 而不是自己配置服务器的话,提供商提供 CURL 库几率远远大于 SOCKET 库;另一个原因就是 CURL 库更易使用,因为它设置参数的灵活性,所以大部分情况都使用 CURL 库。

SOCKET 连接方式 /ext/modules/payment/paypal/standard_ipn.php [42-62]

```php
$header = 'POST /cgi-bin/webscr HTTP/1.0' . "\r\n" .
          'Host: ' . $server . "\r\n" .
          'Content-Type: application/x-www-form-urlencoded' . "\r\n" .
          'Content-Length: ' . strlen($parameters) . "\r\n" .
          'Connection: close' . "\r\n\r\n";

@fputs($fp, $header . $parameters);
$string = '';

while (!@feof($fp)) {
  $res = @fgets($fp, 1024);
  $string .= $res;

  if ( ($res == 'VERIFIED') || ($res == 'INVALID') ) {
    $result = $res;
    break;
  }
}

@fclose($fp);
```

 提示:
 
         

CURL 连接方式 /ext/modules/payment/paypal/standard_ipn.php [64-76]

```php
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, 'https://' . $server . '/cgi-bin/webscr');
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $parameters);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, false);
curl_setopt($ch, CURLOPT_TIMEOUT, 30);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);

$result = curl_exec($ch);
curl_close($ch);
```

以上说明了 SOCKET 连接方式与 CURL 连接方式是如何进行 SSL 连接的,SOCKET 连接方 式的特点是参数由字符串组成,使用的连接函数与普通连接文件一样,唯一的区别在于建立连接 的函数不同,普通文件时使用 fopen ,SOCKET 使用 fsockopen。

CURL 连接方式由 curl_init 初始化句柄,由 curl_close 关闭句柄,然后通过 curl_setopt 设置参 数,CURL 拥有特定的常量来表示不同的参数值,正式建立连接是通过 curl_exec 函数来完成。

> 提示:
> 如果读者了解过网页返回的 header,就应该不会对 SOCKET 提交的数据感到陌生。
> 
> 其中 POST 表明这个连接将提交一个 POST 数据,Host 指定要求连接的 URL,因为要提交 POST 数据,所以设置 Content-Type 为 application/x-www-form-urlencoded。然后指定请求数据的长度 Content-Length,最后指定 Connection: close 关闭此次连接。

处理返回数据 /ext/modules/payment/paypal/standard_ipn.php [79]

```php
if ($result == 'VERIFIED') {
  if (isset($HTTP_POST_VARS['invoice']) && is_numeric($HTTP_POST_VARS['invoice']) && ($HTTP_POST_VARS['invoice'] > 0)) {
    $order_query = tep_db_query("select orders_status, currency, currency_value from " . TABLE_ORDERS . " where orders_id = '" . $HTTP_POST_VARS['invoice'] . "' and customers_id = '" . (int)$HTTP_POST_VARS['custom'] . "'");

    if (tep_db_num_rows($order_query) > 0) {
      $order = tep_db_fetch_array($order_query);
      if ($order['orders_status'] == MODULE_PAYMENT_PAYPAL_STANDARD_PREPARE_ORDER_STATUS_ID) {
        $sql_data_array = array(
          'orders_id' => $HTTP_POST_VARS['invoice'],
          'orders_status_id' => MODULE_PAYMENT_PAYPAL_STANDARD_PREPARE_ORDER_STATUS_ID,
          'date_added' => 'now()',
          'customer_notified' => '0',
          'comments' => '');

        tep_db_perform(TABLE_ORDERS_STATUS_HISTORY, $sql_data_array);
        tep_db_query("update " . TABLE_ORDERS . " set orders_status = '" . (MODULE_PAYMENT_PAYPAL_STANDARD_ORDER_STATUS_ID > 0 ? (int)MODULE_PAYMENT_PAYPAL_STANDARD_ORDER_STATUS_ID : (int)DEFAULT_ORDERS_STATUS_ID) . "', last_modified = now() where orders_id = '" . (int)$HTTP_POST_VARS['invoice'] . "'");
      }

      $total_query = tep_db_query("select value from " . TABLE_ORDERS_TOTAL . " where orders_id = '" . $HTTP_POST_VARS['invoice'] . "' and class = 'ot_total' limit 1");
      $total = tep_db_fetch_array($total_query);
      $comment_status = $HTTP_POST_VARS['payment_status'] . ' (' . ucfirst($HTTP_POST_VARS['payer_status']) . '; ' . $currencies->format($HTTP_POST_VARS['mc_gross'], false, $HTTP_POST_VARS['mc_currency']) . ')';

      if ($HTTP_POST_VARS['payment_status'] == 'Pending') {
        $comment_status .= '; ' . $HTTP_POST_VARS['pending_reason'];
      } elseif ( ($HTTP_POST_VARS['payment_status'] == 'Reversed') ||
($HTTP_POST_VARS['payment_status'] == 'Refunded') ) {
        $comment_status .= '; ' . $HTTP_POST_VARS['reason_code'];
      }

      if ($HTTP_POST_VARS['mc_gross'] != number_format($total['value'] * $order['currency_value'], $currencies->get_decimal_places($order['currency']))) {
        $comment_status .= '; PayPal transaction value (' . tep_output_string_protected($HTTP_POST_VARS['mc_gross']) . ') does not match order value (' . number_format($total['value'] * $order['currency_value'], $currencies->get_decimal_places($order['currency'])) . ')';
      }

      $sql_data_array = array(
        'orders_id' => $HTTP_POST_VARS['invoice'],
        'orders_status_id' => (MODULE_PAYMENT_PAYPAL_STANDARD_ORDER_STATUS_ID > 0 ? (int)MODULE_PAYMENT_PAYPAL_STANDARD_ORDER_STATUS_ID : (int)DEFAULT_ORDERS_STATUS_ID),
        'date_added' => 'now()',
        'customer_notified' => '0',
        'comments' => 'PayPal IPN Verified [' . $comment_status . ']');

      tep_db_perform(TABLE_ORDERS_STATUS_HISTORY, $sql_data_array);
    }
  }
} else {
  if (tep_not_null(MODULE_PAYMENT_PAYPAL_STANDARD_DEBUG_EMAIL)) {
    $email_body = '$HTTP_POST_VARS:' . "\n\n";

    reset($HTTP_POST_VARS);
    while (list($key, $value) = each($HTTP_POST_VARS)) {
      $email_body .= $key . '=' . $value . "\n";
    }

    $email_body .= "\n" . '$HTTP_GET_VARS:' . "\n\n";
    reset($HTTP_GET_VARS);
    while (list($key, $value) = each($HTTP_GET_VARS)) {
      $email_body .= $key . '=' . $value . "\n";
    }

    tep_mail('', MODULE_PAYMENT_PAYPAL_STANDARD_DEBUG_EMAIL, 'PayPal IPN Invalid Process', $email_body, STORE_OWNER, STORE_OWNER_EMAIL_ADDRESS);
  }

  if (isset($HTTP_POST_VARS['invoice']) && is_numeric($HTTP_POST_VARS['invoice']) && ($HTTP_POST_VARS['invoice'] > 0)) {
    $check_query = tep_db_query("select orders_id from " . TABLE_ORDERS . " where orders_id = '" . $HTTP_POST_VARS['invoice'] . "' and customers_id = '" . (int)$HTTP_POST_VARS['custom'] . "'");
    
    if (tep_db_num_rows($check_query) > 0) {
      $comment_status = $HTTP_POST_VARS['payment_status'];
      
      if ($HTTP_POST_VARS['payment_status'] == 'Pending') {
        $comment_status .= '; ' . $HTTP_POST_VARS['pending_reason'];
      } elseif ( ($HTTP_POST_VARS['payment_status'] == 'Reversed') ||
($HTTP_POST_VARS['payment_status'] == 'Refunded') ) {
        $comment_status .= '; ' . $HTTP_POST_VARS['reason_code'];
      }
      
      tep_db_query("update " . TABLE_ORDERS . " set orders_status = '" . ((MODULE_PAYMENT_PAYPAL_STANDARD_ORDER_STATUS_ID > 0) ? MODULE_PAYMENT_PAYPAL_STANDARD_ORDER_STATUS_ID : DEFAULT_ORDERS_STATUS_ID) . "', last_modified = now() where orders_id = '" . $HTTP_POST_VARS['invoice'] . "'");

      $sql_data_array = array(
        'orders_id' => $HTTP_POST_VARS['invoice'],
        'orders_status_id' => (MODULE_PAYMENT_PAYPAL_STANDARD_ORDER_STATUS_ID > 0) ? MODULE_PAYMENT_PAYPAL_STANDARD_ORDER_STATUS_ID : DEFAULT_ORDERS_STATUS_ID,
        'date_added' => 'now()',
        'customer_notified' => '0',
        'comments' => 'PayPal IPN Invalid [' . $comment_status . ']');

      tep_db_perform(TABLE_ORDERS_STATUS_HISTORY, $sql_data_array);
    }
  }
}
```

终于进入了我们的主题,如何利用 Paypal 返回的数据来更新订单数据,我们将一一道来。

这里涉及两种情况,一种是确认付款成功,另一种是未成功的情况。这两种情况由上面 SSL 连接的返回值来决定,如果返回“VERIFIED”文本,则表示确认付款成功,否则未成功。我们先来讲付款成功的处理过程。

付款成功

IPN 返回的参数由 POST 形式返回,所以我们使用$HTTP_POST_VARS($_POST 的长命名变量)来查询参数值。

在第[80]行,订单号由参数 invoice 返回,所以我们先检查 invoice 是否正确。 在第[81]行,通过 invoice 和 custom 查询本地数据库,查找相对应的订单,当查找的订单存在时,则提取出订单的数据。

在第[85-96]行,判断订单当前的状态是否为 Prepare 状态,如果是则会增加一条订单状态历史记录(此时历史记录一定为空),并更新订单状态为已付款状态,已付款状态由用户在参数里设 置,如未设置则为默认状态。

第[98-119]行,最后增加一条已付款状态的历史记录,与一般的情况不同,在这里备注信息 comment 显示的非常重要。

如果付款成功则备注信息应该像如下形式:

`PayPal IPN Verified [Completed (Verified; 99.50)]`

Completed 是付款状态,表示此次付款已经成功,Verified 表示此付款用户的 Paypal 是认证账号, 后面紧跟着付款的金额。

第103-106行说明了Paypal返回付款状态的非正常形式,Pending为付款保持(未成功支付), Reversed 为保留状态,Refunded 为退款。

第 109 行说明了付款的金额与实际订单金额不符的处理方法。

> 提示:
> 为什么要提供非常详细的备注信息呢?主要的考量在于 Paypal 在线付款的敏感性,因为是在线付 款并不是即时到账,用户可以提出退款,
> 所以 IPN 的作用不仅仅限于处理确认付款成功的情况, 还包括用户退款,投诉,撤诉等等。因此提供有用的详尽的原始信息是非常重要的,它将为网站提供强有力的数据支持。

付款未成功

当付款未成功时,需要处理两件事情,一是如果设置了调试 Email,则将发送一封提示邮件;
二是与付款成功一样,也需要增加一条付款状态的历史记录。

邮件内容是 Paypal 返回参数的封装,以便跟踪处理订单情况。

新增付款状态历史记录的代码与付款成功时的一样,所以也就不多讲了。

> 欲了解更多IPN的内容,可以查询http://paypal.ebay.cn/integrationcenter/list__resource_2.htm(l中文), 或者https://www.paypal.com/IntegrationCenter/ic_ipn.html(英文)
> 
> 详细的 IPN 返回参数请见附录:《Paypal IPN 和 PDT 变量表》
   
### 国内银行在线支付

国内在线支付主要有两种形式,一种是直接银行网关支付,另一种则是专业在线支付平台。

国内 各大银行如工商银行、交通银行、招商银行等都支持在线支付,用户只需要开通网上银行服务即可使用(部分银行默认已开通网上银行服务)。

支付宝、财付通、安付通等则是专业的在线支付平台,用户支付必须先注册成为其用户,通过银行充值到平台的账号,才可以进行在线支付(类似 Paypal 的模式,不同之处在于 paypal 允许用户注册 paypal 账号后,便可以立即使用信用卡付款)。 

笔者在这里选择使用国内的“网银在线”,作为付款模块的介绍,网银在线只是集成多家银行于一 身,从而实现一个商户账号便可以接收所有不同银行的付款。

该服务在用户最后付款时依然使用 的是各家银行自身的网关,所以可以保证用户的付款安全,减少其担心。也因为网银在线只是集 成银行间的服务,所以我将其归为专业在线支付平台一类形式。

> 网银在线介绍:http://www.chinabank.com.cn/gateway/help.shtml

#### 付款模块

网银在线模块文件: includes/modules/payment/chinabank.php

```php
<?php
class chinabank {
  var $code, $title, $description, $enabled;
  
  function chinabank() {
    global $order;

    $this->code = 'chinabank';
    $this->codeTitle = 'ChinaBank';

    $this->title = MODULE_PAYMENT_CHINABANK_TEXT_TITLE;
    $this->description = MODULE_PAYMENT_CHINABANK_TEXT_DESCRIPTION; $this->sort_order = MODULE_PAYMENT_CHINABANK_SORT_ORDER;
    $this->enabled = ((MODULE_PAYMENT_CHINABANK_STATUS == 'True') ? true : false);
    $this->money_type = "CNY"; // 使用人民币作为支付币种
    //$this->form_action_url='https://pay.chinabank.com.cn/select_bank'; 这个URL 是旧的网银在线支付网关
    $this->form_action_url = 'https://Pay3.chinabank.com.cn/PayGate'; // 这里是新的支付网关
  }

  function javascript_validation() {
    return false;
  }

  function selection() {
    return array('id' => $this->code, 'module' => $this->title);
  }

  function pre_confirmation_check() {
   return false;
  }

  function confirmation() {
  // 由于是在线支付模块,所以必须自行处理订单的生成,这里的代码是来自于 paypal 的代码片段
     global $cartID, $cart_ChinaBank_Identify_ID, $customer_id, $languages_id, $order, $order_total_modules;

    if (tep_session_is_registered('cartID')) {
      $insert_order = false;
      if (tep_session_is_registered('cart_ChinaBank_Identify_ID')) {

        $order_id = substr($cart_ChinaBank_Identify_ID,strpos($cart_ChinaBank_Identify_ID, '-')+1);
        $curr_check = tep_db_query("select currency from " . TABLE_ORDERS . " where orders_id = '" . (int)$order_id . "'");

        $curr = tep_db_fetch_array($curr_check);

        if ( ($curr['currency'] != $order->info['currency']) || ($cartID !=
substr($cart_ChinaBank_Identify_ID, 0, strlen($cartID))) ) {
          $check_query = tep_db_query('select orders_id from ' . TABLE_ORDERS_STATUS_HISTORY . ' where orders_id = "' . (int)$order_id . '" limit 1');

          if (tep_db_num_rows($check_query) < 1) {
            tep_db_query('delete from ' . TABLE_ORDERS . ' where orders_id = "' .(int)$order_id . '"');
            tep_db_query('delete from ' . TABLE_ORDERS_TOTAL . ' where orders_id = "' . (int)$order_id . '"');
            tep_db_query('delete from ' . TABLE_ORDERS_STATUS_HISTORY . ' where orders_id = "' . (int)$order_id . '"');
            tep_db_query('delete from ' . TABLE_ORDERS_PRODUCTS . ' where orders_id = "' . (int)$order_id . '"');
            tep_db_query('delete from ' . TABLE_ORDERS_PRODUCTS_ATTRIBUTES . ' where orders_id = "' . (int)$order_id . '"');
            tep_db_query('delete from ' . TABLE_ORDERS_PRODUCTS_DOWNLOAD . ' where orders_id = "' . (int)$order_id . '"');
          }
          $insert_order = true;
        }
      } else {
        $insert_order = true;
      }

      if ($insert_order == true) {
        $order_totals = array();

        if (is_array($order_total_modules->modules)) {
          reset($order_total_modules->modules);
          while (list(, $value) = each($order_total_modules->modules)) {
            $class = substr($value, 0, strrpos($value, '.'));

            if ($GLOBALS[$class]->enabled) {
              for ($i=0, $n=sizeof($GLOBALS[$class]->output); $i<$n; $i++) {
                if (tep_not_null($GLOBALS[$class]->output[$i]['title']) &&
tep_not_null($GLOBALS[$class]->output[$i]['text'])) {
                  $order_totals[] = array(
                    'code' => $GLOBALS[$class]->code,
                    'title' => $GLOBALS[$class]->output[$i]['title'],
                    'text' => $GLOBALS[$class]->output[$i]['text'],
                    'value' => $GLOBALS[$class]->output[$i]['value'],
                    'sort_order' => $GLOBALS[$class]->sort_order
                  );
                }
              }
            }
          }
        }

        $sql_data_array = array(
          'customers_id' => $customer_id,
          'customers_name' => $order->customer['firstname'] . ' ' . $order->customer['lastname'],
          'customers_company' => $order->customer['company'],
          'customers_street_address' =>
$order->customer['street_address'],
          'customers_suburb' => $order->customer['suburb'],
          'customers_city' => $order->customer['city'],
          'customers_postcode' => $order->customer['postcode'],
          'customers_state' => $order->customer['state'],
          'customers_country' => $order->customer['country']['title'],
          'customers_telephone' => $order->customer['telephone'],
          'customers_email_address' => $order->customer['email_address'],
          'customers_address_format_id' => $order->customer['format_id'],
          'delivery_name' => $order->delivery['firstname'] . ' ' . $order->delivery['lastname'],
          'delivery_company' => $order->delivery['company'],
          'delivery_street_address' => $order->delivery['street_address'],
          'delivery_suburb' => $order->delivery['suburb'],
          'delivery_city' => $order->delivery['city'],
          'delivery_postcode' => $order->delivery['postcode'],
          'delivery_state' => $order->delivery['state'],
          'delivery_country' => $order->delivery['country']['title'],
          'delivery_address_format_id' => $order->delivery['format_id'],
          'billing_name' => $order->billing['firstname'] . ' ' . $order->billing['lastname'],
          'billing_company' => $order->billing['company'],
          'billing_street_address' => $order->billing['street_address'],
          'billing_suburb' => $order->billing['suburb'],
          'billing_city' => $order->billing['city'],
          'billing_postcode' => $order->billing['postcode'],
          'billing_state' => $order->billing['state'],
          'billing_country' => $order->billing['country']['title'],
          'billing_address_format_id' => $order->billing['format_id'],
          'payment_method' => $order->info['payment_method'],
          'cc_type' => $order->info['cc_type'],
          'cc_owner' => $order->info['cc_owner'],
          'cc_number' => $order->info['cc_number'],
          'cc_expires' => $order->info['cc_expires'],
          'date_purchased' => 'now()',
          'orders_status' => $order->info['order_status'],
          'currency' => $order->info['currency'],
          'currency_value' => $order->info['currency_value']);
        tep_db_perform(TABLE_ORDERS, $sql_data_array);
        $insert_id = tep_db_insert_id();

        for ($i=0, $n=sizeof($order_totals); $i<$n; $i++) {
          $sql_data_array = array(
            'orders_id' => $insert_id,
            'title' => $order_totals[$i]['title'],
            'text' => $order_totals[$i]['text'],
            'value' => $order_totals[$i]['value'],
            'class' => $order_totals[$i]['code'],
            'sort_order' => $order_totals[$i]['sort_order']);
          tep_db_perform(TABLE_ORDERS_TOTAL, $sql_data_array);
        }

        for ($i=0, $n=sizeof($order->products); $i<$n; $i++) {
          $sql_data_array = array(
            'orders_id' => $insert_id,
            'products_id' => tep_get_prid($order->products[$i]['id']),
            'products_model' => $order->products[$i]['model'],
            'products_name' => $order->products[$i]['name'],
            'products_price' => $order->products[$i]['price'],
                                    'final_price' => $order->products[$i]['final_price'],
            'products_tax' => $order->products[$i]['tax'],
            'products_quantity' => $order->products[$i]['qty']);
          tep_db_perform(TABLE_ORDERS_PRODUCTS, $sql_data_array);
          $order_products_id = tep_db_insert_id();
          $attributes_exist = '0';

          if (isset($order->products[$i]['attributes'])) {
            $attributes_exist = '1';

            for ($j=0, $n2=sizeof($order->products[$i]['attributes']); $j<$n2; $j++){
              if (DOWNLOAD_ENABLED == 'true') {
                $attributes_query = "select popt.products_options_name, poval.products_options_values_name, pa.options_values_price, pa.price_prefix, pad.products_attributes_maxdays, pad.products_attributes_maxcount , pad.products_attributes_filename from " . TABLE_PRODUCTS_OPTIONS . " popt, " . TABLE_PRODUCTS_OPTIONS_VALUES . " poval, " . TABLE_PRODUCTS_ATTRIBUTES . " pa left join " . TABLE_PRODUCTS_ATTRIBUTES_DOWNLOAD . " pad on pa.products_attributes_id=pad.products_attributes_id where pa.products_id = '" . $order->products[$i]['id'] . "' and pa.options_id = '" . $order->products[$i]['attributes'][$j]['option_id'] . "' and pa.options_id = popt.products_options_id and pa.options_values_id = '" . $order->products[$i]['attributes'][$j]['value_id'] . "' and pa.options_values_id = poval.products_options_values_id
  and popt.language_id = '" . $languages_id . "' and poval.language_id = '" . $languages_id . "'";
                $attributes = tep_db_query($attributes_query);
              } else {
                $attributes = tep_db_query("select popt.products_options_name,
poval.products_options_values_name, pa.options_values_price, pa.price_prefix from " . TABLE_PRODUCTS_OPTIONS . " popt, " . TABLE_PRODUCTS_OPTIONS_VALUES . " poval, " . TABLE_PRODUCTS_ATTRIBUTES . " pa where pa.products_id = '" . $order->products[$i]['id'] . "' and pa.options_id = '" . $order->products[$i]['attributes'][$j]['option_id'] . "' and pa.options_id = popt.products_options_id and pa.options_values_id = '" . $order->products[$i]['attributes'][$j]['value_id'] . "' and pa.options_values_id = poval.products_options_values_id and popt.language_id = '" . $languages_id . "' and poval.language_id = '" . $languages_id . "'");
              }

              $attributes_values = tep_db_fetch_array($attributes);

              $sql_data_array = array(
                'orders_id' => $insert_id,
                'orders_products_id' => $order_products_id,
                'products_options' => $attributes_values['products_options_name'],
                'products_options_values' => $attributes_values['products_options_values_name'],
                'options_values_price' => $attributes_values['options_values_price'],
                'price_prefix' => $attributes_values['price_prefix']);
              tep_db_perform(TABLE_ORDERS_PRODUCTS_ATTRIBUTES, $sql_data_array);

              if ((DOWNLOAD_ENABLED == 'true') &&
isset($attributes_values['products_attributes_filename']) && tep_not_null($attributes_values['products_attributes_filename'])) {
                $sql_data_array = array(
                  'orders_id' => $insert_id,
                  'orders_products_id' => $order_products_id,
                  'orders_products_filename' => $attributes_values['products_attributes_filename'],
                  'download_maxdays' => $attributes_values['products_attributes_maxdays'],
                  'download_count' => $attributes_values['products_attributes_maxcount']);
                tep_db_perform(TABLE_ORDERS_PRODUCTS_DOWNLOAD, $sql_data_array);
              }
            }
          }
        }

        $cart_ChinaBank_Identify_ID = $cartID . '-' . $insert_id;
        tep_session_register('cart_ChinaBank_Identify_ID');
      }
    }

    return false;
  }

  function process_button() {
    // process_button 处理提交支付网关的参数,对于不同服务商提供的支付服务,最重要的不同也就是在于这里,另一个不同之处在于对支付返回的处理
    global $customer_id, $order, $sendto, $currency, $cart_ChinaBank_Identify_ID, $shipping;

    $process_button_string = '';
    $orderId = substr($cart_ChinaBank_Identify_ID, strpos($cart_ChinaBank_Identify_ID, '-')+1);

    $parameters = array(
      'v_mid' => MODULE_PAYMENT_CHINABANK_ACCOUNT, // v_mid:注册的账号
      'v_oid' => strlen($orderId)==0?date('Ymd',time())."-".MODULE_PAYMENT_CHINABANK_ACCOUNT."-".d ate('His',time()):$orderId, // 'v_oid:订单号
      'v_amount' => $this->format_raw($order->info['total'] - $order->info['shipping_cost'] - $order->info['tax']), // v_amount:总金额
      'v_moneytype' => $this->money_type, // v_moneytype:支付币种
      'v_url' => tep_href_link('ext/modules/payment/chinabank/receive.php', '', 'SSL', false, false) // v_url:处理返回结果的URL
    );

    $text = $parameters['v_amount'].$this->money_type
            .$parameters['v_oid'].MODULE_PAYMENT_CHINABANK_ACCOUNT
            .$parameters['v_url'].MODULE_PAYMENT_CHINABANK_KEY;
    $parameters['v_md5info'] = strtoupper(md5($text)); // v_md5info:加密字段,用于 验证提交信息

    /**
    * MD5校验串生成方法:
    * 当消费者在商户端生成最终订单的时候,将订单中的 v_amount v_moneytype v_oid v_mid v_url key 六个参数的 value 值拼成一个无间隔的字符串(顺序不要改变)。参数 key 是商户的 MD5 密钥(该密匙可在 登陆商户管理界面后自行更改。)
    **/
    reset($parameters);
    while (list($key, $value) = each($parameters)) {
      $process_button_string .= tep_draw_hidden_field($key, $value);
    }

    return $process_button_string;
  }

  function before_process() {
    global $customer_id, $order, $order_totals, $sendto, $billto, $languages_id,
$payment, $currencies, $cart, $cart_ChinaBank_Identify_ID;
    global $$payment;

    $order_id = substr($cart_ChinaBank_Identify_ID, strpos($cart_ChinaBank_Identify_ID, '-')+1);

    $check_query = tep_db_query("select orders_status from " . TABLE_ORDERS . " where orders_id = '" . (int)$order_id . "'");

    if (tep_db_num_rows($check_query)) {
       $check = tep_db_fetch_array($check_query);
      if ($check['orders_status'] == MODULE_PAYMENT_CHINABANK_STANDARD_PREPARE_ORDER_STATUS_ID) {
        $sql_data_array = array(
          'orders_id' => $order_id,
          'orders_status_id' => MODULE_PAYMENT_CHINABANK_STANDARD_PREPARE_ORDER_STATUS_ID,
          'date_added' => 'now()',
          'customer_notified' => '0',
          'comments' => '');
        tep_db_perform(TABLE_ORDERS_STATUS_HISTORY, $sql_data_array); // 添加一条新 的订单状态记录
      }
    }

    tep_db_query("update " . TABLE_ORDERS . " set orders_status = '" . (MODULE_PAYMENT_CHINABANK_STANDARD_PREPARE_ORDER_STATUS_ID > 0 ? (int)MODULE_PAYMENT_CHINABANK_STANDARD_PREPARE_ORDER_STATUS_ID : (int)DEFAULT_ORDERS_STATUS_ID) . "', last_modified = now() where orders_id = '" . (int)$order_id . "'"); // 更新订单状态

    $products_ordered = '';
    $subtotal = 0;
    $total_tax = 0;

    for ($i=0, $n=sizeof($order->products); $i<$n; $i++) {
      if (STOCK_LIMITED == 'true') {
        if (DOWNLOAD_ENABLED == 'true') {
          $stock_query_raw = "SELECT products_quantity, pad.products_attributes_filename  FROM " . TABLE_PRODUCTS . " p LEFT JOIN " . TABLE_PRODUCTS_ATTRIBUTES . " pa ON p.products_id=pa.products_id LEFT JOIN " . TABLE_PRODUCTS_ATTRIBUTES_DOWNLOAD . " pad ON pa.products_attributes_id=pad.products_attributes_id WHERE p.products_id = '" . tep_get_prid($order->products[$i]['id']) . "'";

          // Will work with only one option for downloadable products
          // otherwise, we have to build the query dynamically with a loop
          $products_attributes = $order->products[$i]['attributes'];
          if (is_array($products_attributes)) {
            $stock_query_raw .= " AND pa.options_id = '" . $products_attributes[0]['option_id'] . "' AND pa.options_values_id = '" .$products_attributes[0]['value_id'] . "'";
          }

          $stock_query = tep_db_query($stock_query_raw);
        } else {
          $stock_query = tep_db_query("select products_quantity from " . TABLE_PRODUCTS . " where products_id = '" . tep_get_prid($order->products[$i]['id']) . "'");
        }

        if (tep_db_num_rows($stock_query) > 0) {
          $stock_values = tep_db_fetch_array($stock_query);

          // do not decrement quantities if products_attributes_filename exists
          if ((DOWNLOAD_ENABLED != 'true') || (!$stock_values['products_attributes_filename'])) {
            $stock_left = $stock_values['products_quantity'] - $order->products[$i]['qty'];
          } else {
            $stock_left = $stock_values['products_quantity'];
          }

          tep_db_query("update " . TABLE_PRODUCTS . " set products_quantity = '" . $stock_left . "' where products_id = '" . tep_get_prid($order->products[$i]['id']) . "'");

          if ( ($stock_left < 1) && (STOCK_ALLOW_CHECKOUT == 'false') ) {
            tep_db_query("update " . TABLE_PRODUCTS . " set products_status = '0' where products_id = '" . tep_get_prid($order->products[$i]['id']) . "'");
          }
        }
      }

      // Update products_ordered (for bestsellers list)
      tep_db_query("update " . TABLE_PRODUCTS . " set products_ordered = products_ordered + " . sprintf('%d', $order->products[$i]['qty']) . " where products_id = '" . tep_get_prid($order->products[$i]['id']) . "'");

      //------insert customer choosen option to order--------
      $attributes_exist = '0';
      $products_ordered_attributes = '';

      if (isset($order->products[$i]['attributes'])) {
        $attributes_exist = '1';

        for ($j=0, $n2=sizeof($order->products[$i]['attributes']); $j<$n2; $j++) {
          if (DOWNLOAD_ENABLED == 'true') {
            $attributes_query = "select popt.products_options_name, poval.products_options_values_name, pa.options_values_price, pa.price_prefix, pad.products_attributes_maxdays, pad.products_attributes_maxcount , pad.products_attributes_filename from " . TABLE_PRODUCTS_OPTIONS . " popt, " . TABLE_PRODUCTS_OPTIONS_VALUES . " poval, " . TABLE_PRODUCTS_ATTRIBUTES . " pa left join " . TABLE_PRODUCTS_ATTRIBUTES_DOWNLOAD . " pad on pa.products_attributes_id=pad.products_attributes_id where pa.products_id = '" . $order->products[$i]['id'] . "' and pa.options_id = '" . $order->products[$i]['attributes'][$j]['option_id'] . "' and pa.options_id = popt.products_options_id and pa.options_values_id = '" . $order->products[$i]['attributes'][$j]['value_id'] . "' and pa.options_values_id = poval.products_options_values_id and popt.language_id = '" . $languages_id . "' and poval.language_id = '" . $languages_id . "'";
            $attributes = tep_db_query($attributes_query);
          } else {
            $attributes = tep_db_query("select popt.products_options_name, poval.products_options_values_name, pa.options_values_price, pa.price_prefix from " . TABLE_PRODUCTS_OPTIONS . " popt, " . TABLE_PRODUCTS_OPTIONS_VALUES . " poval, " . TABLE_PRODUCTS_ATTRIBUTES . " pa where pa.products_id = '" . $order->products[$i]['id'] . "' and pa.options_id = '" . $order->products[$i]['attributes'][$j]['option_id'] . "' and pa.options_id = popt.products_options_id and pa.options_values_id = '" . $order->products[$i]['attributes'][$j]['value_id'] . "' and pa.options_values_id = poval.products_options_values_id and popt.language_id = '" . $languages_id . "' and poval.language_id = '" . $languages_id . "'");
          }

          $attributes_values = tep_db_fetch_array($attributes);
          $products_ordered_attributes .= "\n\t" .
          $attributes_values['products_options_name'] . ' ' .
          $attributes_values['products_options_values_name'];
        } 
      }

      $total_weight += ($order->products[$i]['qty'] *
      $order->products[$i]['weight']);
      $total_tax += tep_calculate_tax($total_products_price, $products_tax) *
      $order->products[$i]['qty'];
      $total_cost += $total_products_price;
      $products_ordered .= $order->products[$i]['qty'] . ' x ' .
      $order->products[$i]['name'] . ' (' . $order->products[$i]['model'] . ') = ' .
      $currencies->display_price($order->products[$i]['final_price'],
      $order->products[$i]['tax'], $order->products[$i]['qty']) .
      $products_ordered_attributes . "\n";
    }

    // load the after_process function from the payment modules
    $this->after_process();
    $cart->reset(true);
    // unregister session variables used during checkout
    tep_session_unregister('sendto');
    tep_session_unregister('billto');
    tep_session_unregister('shipping');
         tep_session_unregister('payment');
    tep_session_unregister('comments');
    tep_session_unregister('cart_ChinaBank_Identify_ID');
    tep_redirect(tep_href_link(FILENAME_CHECKOUT_SUCCESS, '', 'SSL'));
  }

  function after_process() {
    return false;
  }

  function output_error() {
    return false;
  }

  function check() {
    if (!isset($this->_check)) {
      $check_query = tep_db_query("select configuration_value from " . TABLE_CONFIGURATION . " where configuration_key = 'MODULE_PAYMENT_CHINABANK_STATUS'");
       $this->_check = tep_db_num_rows($check_query);
    }
    return $this->_check;
  }

  function install() {
    // 安装 chinabank 模块要用到的参数
    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, set_function, date_added) values ('Enable China Bank Module', 'MODULE_PAYMENT_CHINABANK_STATUS', 'True', 'Do you want to accept China Bank payments?', '6', '0', 'tep_cfg_select_option(array(\'True\', \'False\'), ', now())");
    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, date_added) values ('Business Account', 'MODULE_PAYMENT_CHINABANK_ACCOUNT','', '', '6', '0', now())");
    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, date_added) values ('Business Key', 'MODULE_PAYMENT_CHINABANK_KEY','', '', '6', '', now())");
    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description,
configuration_group_id, sort_order, date_added) values ('Money Type',
'MODULE_PAYMENT_CHINABANK_MONEYTYPE','CNY', '', '6', '', now())");
    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, set_function, use_function, date_added) values ('Set Preparing Order Status', 'MODULE_PAYMENT_CHINABANK_STANDARD_PREPARE_ORDER_STATUS_ID', '" . $status_id . "', 'Set the status of prepared orders made with this payment module to this value', '6', '0', 'tep_cfg_pull_down_order_statuses(', 'tep_get_order_status_name', now())");
    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, set_function, use_function, date_added) values ('Set Success Order Status', 'MODULE_PAYMENT_CHINABANK_SUCCESS_ORDER_STATUS_ID', '" . $status_id . "', 'Set the status of complete orders made with this payment module to this value', '6', '0', 'tep_cfg_pull_down_order_statuses(', 'tep_get_order_status_name', now())");
    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, date_added) values ('Sort order of display.', 'MODULE_PAYMENT_CHINABANK_SORT_ORDER', '0', 'Sort order of display. Lowest is displayed first.', '6', '0', now())");
  }
   
  function remove() {
    tep_db_query("delete from " . TABLE_CONFIGURATION . " where configuration_key in ('" . implode("', '", $this->keys()) . "')");
  }

  function keys() {
    return array(
      'MODULE_PAYMENT_CHINABANK_STATUS',
      'MODULE_PAYMENT_CHINABANK_ACCOUNT',
      'MODULE_PAYMENT_CHINABANK_KEY',
      'MODULE_PAYMENT_CHINABANK_MONEYTYPE',
      'MODULE_PAYMENT_CHINABANK_STANDARD_PREPARE_ORDER_STATUS_ID',
      'MODULE_PAYMENT_CHINABANK_SUCCESS_ORDER_STATUS_ID',
      'MODULE_PAYMENT_CHINABANK_SORT_ORDER'
    );
  }
  
  // format prices without currency formatting
  function format_raw($number, $currency_code = '', $currency_value = '') {
    global $currencies, $currency;

    if (empty($currency_code) || !$this->is_set($currency_code)) {
      $currency_code = $currency;
    }

    if (empty($currency_value) || !is_numeric($currency_value)) {
      $currency_value = $currencies->currencies[$currency_code]['value'];
    }

    return number_format(tep_round($number * $currency_value, $currencies->currencies[$currency_code]['decimal_places']), $currencies->currencies[$currency_code]['decimal_places'], '.', '');
  }
}
?>
```

> 提示:
> 
> 详细的网银在线支付表单中的变量定义:
> —  必填项
> —  与网上支付货款无关项,建议不用

| 变量名称 | 变量命名 | 长度 | 说明 | 举例 |
|---|---|---|---|---|
| 商户编号 | v_mid |  | 不可为空,以初始单所填商户编号为准。 | 20000400 |
| 订单编号 | v_oid | 64 | 不可为空值,订单编号标准格 式为:订单生成日期 (yyyymmdd)-商户编号-商户 流水号。订单编号所有字符总 和不可超过 64 位。 | 19990720-20000400- 12345。商户流水号为 数字,订单号当日内不可重复 |
| 订单总金额 | v_amount | 8 | 不可为空,单位:元,小数点后保留两位。 | 0.01 |
| 币种 | v_moneytype |  | CNY 为人民币 | CNY |
| URL 地址 | v_url | 200 | 消费者完成购物后返回的商 户页面,URL 参数是以 http:// 开头的完整 URL 地址 |  http://domain/chinaba nk/receive.php |
| MD5 校验码 | v_md5info | 32 |  |  |
| 备注1 | remark1 | 150 |  | 备注1 |
| 备注2 | remark2 | 150 |  | 备注2 |
| 收货人姓名 | v_rcvname | 80 |  | 张三 |
| 收货人地址 | v_rcvaddr | 200 |  | 北京海淀 1 |
| 收货人电话 | v_rcvtel | 50 |  | 588156661 |
| 收货人邮编 | v_rcvpost | 10 | | 100081 |
| 收货人 Email | v_rcvemail | 100 |  | test1@test.com |
| 收货人手机号 | v_rcvmobile | 13 |  | 1311311311311 |
| 订货人姓名 | v_ordername | 80 |  | 李四 |
| 订货人地址 | v_orderaddr | 200 |  | 北京海淀 2 |
| 订货人电话 | v_ordertel | 50 |  | 588156662 |
| 订货人邮编 | v_orderpost | 10 |  | 100082 |
| 订货人 Email | v_orderemail | 100 |  | test2@test.com |
| 订货人手机号 | v_ordermobile | 13 |  | 1311311311312 |

#### 语言文件

网银在线语言文件: includes/languages/english/modules/payment/chinabank.php

```php
<?php
  define('MODULE_PAYMENT_CHINABANK_TEXT_TITLE', 'China Bank Payment');
  define('MODULE_PAYMENT_CHINABANK_TEXT_DESCRIPTION', 'China Bank Payment');
?>
```

#### 处理支付返回

网银在线支付返回处理文件:ext/modules/payment/chinabank/receive.php

```php
<?php
  chdir('../../../../');
  require('includes/application_top.php');

  $v_oid = trim($_POST['v_oid']); // 订单号
  $v_pmode = trim($_POST['v_pmode']); // 支付银行
  $v_pstatus = trim($_POST['v_pstatus']); // 支付是否成功,“20”为成功,“30”为失败 $v_pstring = trim($_POST['v_pstring']); // 支付结果信息
  $v_amount = trim($_POST['v_amount']); // 总金额
  $v_moneytype = trim($_POST['v_moneytype']); // 支付币种
  $v_md5str = trim($_POST['v_md5str']); // 加密过的验证码

  $md5string=strtoupper(md5($v_oid.$v_pstatus.$v_amount.$v_moneytype.MODULE_PAYMENT_CHINABANK_KEY));

  if ($v_md5str==$md5string){
    if($v_pstatus=="20"){
      $sql = sprintf('select * from %s where orders_id=%d and class="ot_total" and value="%s"',TABLE_ORDERS_TOTAL,int($v_oid),$v_amount); // 需要验证订单号与总金额是否一 致
      $query = tep_db_query($sql);
      
      if(1 == tep_db_num_rows($query)){
        $sql_data_array = array(
          'orders_id' => int($v_oid),
          'orders_status_id' => MODULE_PAYMENT_CHINABANK_SUCCESS_ORDER_STATUS_ID,
          'date_added' => 'now()',
          'customer_notified' => '0',
          'comments' => '');
        tep_db_perform(TABLE_ORDERS_STATUS_HISTORY, $sql_data_array); // 增加一条 付款成功的订单状态记录

        tep_db_query("update " . TABLE_ORDERS . " set orders_status = '" . (MODULE_PAYMENT_CHINABANK_SUCCESS_ORDER_STATUS_ID > 0 ? (int)MODULE_PAYMENT_CHINABANK_SUCCESS_ORDER_STATUS_ID : (int)DEFAULT_ORDERS_STATUS_ID) . "', last_modified = now() where orders_id = '" . int($v_oid) . "'"); // 更新订单状态为已付款
        
        tep_redirect(tep_href_link(FILENAME_CHECKOUT_SUCCESS));
      }
    }

    $messageStack->add('header','Your payment information is not correct.','warning');
    tep_redirect(tep_href_link(FILENAME_CHECKOUT_PAYMENT));
  }

  $messageStack->add('header','Payment failure.','warning');
  tep_redirect(tep_href_link(FILENAME_SHOPPING_CART));

  require('includes/application_bottom.php');
?>
```

详细的网银在线返回信息,见表:

| 变量名称 | 变量命名 | 返回值说明 |
|---|---|---|
| 订单编号 | v_oid | 商户发送的 v_oid 定单编号。|
| 支付状态 | v_pstatus | 20(表示支付成功) 30(表示支付失败) |
| 支付结果信息 | v_pstring | 支付完成 |
| 支付银行 | v_pmode | 支付银行,例如工商银行 |
| 订单 MD5 校验码 | v_md5str | 该参数的 MD5 字符串的顺序为:<br> *v_oid, v_pstatus,v_amount,v_moneytype,key* <br> MD5 字符串示例: 20050320-2000400-0000012342012.340key <br> 用 MD5 函数加密上述字符串后得到的值如果和 v_md5str 值相等即表明返回的信息没有被纂改 |
| 订单总金额 | v_amount | 订单实际支付金额 |
| 币种 | v_moneytype | 订单实际支付币种 |
| 备注字段 1 | remark1 |  |
| 备注字段 2 |remark2 |  |

