## 付款模块的工作原理

### 付款模块的方法

#### 安装与删除

includes/modules/payment/paypal_standard.php [571-629] 

付款模块的安装装与删除需要的方法与运输是一样的,分为 keys(返回模块所用的键)、check(模 块是否已安装)、install(安装模块)、remove(移除模块)。

初始化 includes/modules/payment/paypal_standard.php [17-40]

```php
function paypal_standard() {
   global $order;

   $this->signature = 'paypal|paypal_standard|1.0|2.2';

   $this->code = 'paypal_standard';
   $this->title = MODULE_PAYMENT_PAYPAL_STANDARD_TEXT_TITLE;
   $this->public_title = MODULE_PAYMENT_PAYPAL_STANDARD_TEXT_PUBLIC_TITLE;
   $this->description = MODULE_PAYMENT_PAYPAL_STANDARD_TEXT_DESCRIPTION;
   $this->sort_order = MODULE_PAYMENT_PAYPAL_STANDARD_SORT_ORDER;
   $this->enabled = ((MODULE_PAYMENT_PAYPAL_STANDARD_STATUS == 'True') ? true : false);

   if ((int)MODULE_PAYMENT_PAYPAL_STANDARD_PREPARE_ORDER_STATUS_ID > 0) {
     $this->order_status =
MODULE_PAYMENT_PAYPAL_STANDARD_PREPARE_ORDER_STATUS_ID;
   }

   if (is_object($order)) $this->update_status();

   if (MODULE_PAYMENT_PAYPAL_STANDARD_GATEWAY_SERVER == 'Live') {
     $this->form_action_url = 'https://www.paypal.com/cgi-bin/webscr';
   } else {
     $this->form_action_url = 'https://www.sandbox.paypal.com/cgi-bin/webscr';
   }
}
```

上面是 Paypal 标准付款模块的初始化过程。其中包含了标准付款模块所必须的参数设置,另外则 是针对 Paypal 付款的特点而设置的参数。

-  $code:付款模块的ID
-  $title:付款模块的名称
-  $public_title:付款模块的显示名称,此成员是可选的,因为外部程序从不会直接调用模块的 title,所以是否使用不同的 title 来区分内部与公共显示,完全取决于模块编写者的习惯,或 者是否有必要的考虑。
-  $description:付款模块的描述
-  $sort_order:付款选择时的排序号
-  $enabled:是否生效

以下三个成员是只用于 Paypal 付款模块,为了满足特定要求而添加的。

-  order_status:订单默认状态
-  form_action_url:使用哪一个网关进行付款,https://www.paypal.com/cgi-bin/webscr是正常的付
款网关,https://www.sandbox.paypal.com/cgi-bin/webscr是Paypal Sandbox的调试付款网关。
-  signature:Paypal付款模块的版本识别号

#### 付款功能

includes/modules/payment/paypal_standard.php [42-569]

与其它模块不同的是,付款模块针对付款的特点,有其特别的方法。

#### function javascript_validation()

用于校验用户输入内容的 javascript 代码。

此方法一般只使用在针对要求用户输入信息的付款模块(如:PSiGate 模块,此模块要求用户输入 信用卡的有关资料进行付款),在 Paypal 付款模块内不需要用户输入任何信息,所以不返回任何 内容即可(返回 false)。

#### function output_error()

此方法有错误,应该是模块编写的失误所致,正确的方法名应为 get_error,get_error 方法返回当 用户的输入信息出错时的错误提示信息。

> 提示:
> get_error 和 javascript_validation 同时适用于那种需要用户输入资料的付款模块,所以针对在线支 付类的付款模块,由于付款资料都是直接由付款网关执行,方法只需返回空或 false 便可。


#### function selection()

在付款模块选择时如果显示此模块(调用代码见 checkout_payment.php[224])。

includes/modules/payment/paypal_standard.php [89]

```php
return array('id' => $this->code,
             'module' => $this->public_title);
```

几乎所有付款模块 selection 方法最后的代码都是上面一样的(如果没有 public_title,则为 title),可以说此方法的作用仅依靠这一小段代码就可以完成了,不过对于不同的模块可能还需要 进行一些诸如清理、初始化的工作等。


#### function update_status()

重新更新付款模块,检查模块的有效性(调用代码见 checkout_confirmation.php[53], checkout_process.php[69])。

因为付款模块的限制比较少,因此大多数情况下,此方法都只是完成税区限制的检验即可。 Paypal 标准付款模块也不例外,当为 Paypal 付款模块设置了税区限制,则检验当前的区是否满足 税区要求,以便调整付款模块的有效性。

> 提示:
> 当前的区使用的是账单的区信息 billing[‘zone_id’],而不是发货的区信息 shipping[‘zone_id’]。

以下五个方法都将在付款过程中的不同环节起到非常重要作用的因素,它们分别是:

#### function pre_confirmation_check()

在确认之前需做的准备(调用代码见 checkout_confirmation.php[60])

标准的代码是由$cart 购物车类生成新的识别 ID,新建会话将其保存,等到调用 confirmation 时, 再核对识别 ID 是否正确。

Paypal 付款模块使用了标准的代码,此代码内容如下:

includes/modules/payment/paypal_standard.php [93]
    
```php
function pre_confirmation_check() {
    global $cartID, $cart;

    if (empty($cart->cartID)) {
       $cartID = $cart->cartID = $cart->generate_cart_id();
    }

    if (!tep_session_is_registered('cartID')) {
       tep_session_register('cartID');
    }
}
``` 

#### function confirmation()

正式确认调用代码见 checkout_confirmation.php[250]

> 提示:
> 
> Paypal 付款模块在 confirmation 的过程中就会生成订单,而并非是在确认以后,所以在这种情况 下,就有可能发生已经选择了 Paypal 付款方式,进入了确认页面(已生成新订单),而用户又需 要更改原先的内容,导致每二次执行 confirmation 的情况,如果不进行特殊处理,就会产生多余订单的现象(用户的本意是要更改当前的订单,而实际却生成了两个订单,所以之前的订单是毫 无意义的)。
> 
> 对于在线支付的模块都会有这种情况发生,因为用户可以在未付款成功前,有返回更改订单的权 限。所以这里的代码很值得我们进行讨论。

```php
function confirmation(){
  global $cartID, $cart_PayPal_Standard_ID, $customer_id, $languages_id, $order, $order_total_modules;

  if (tep_session_is_registered('cartID')) {
    $insert_order = false;

    if (tep_session_is_registered('cart_PayPal_Standard_ID')) {
      $order_id = substr($cart_PayPal_Standard_ID, strpos($cart_PayPal_Standard_ID, '-')+1);

      $curr_check = tep_db_query("select currency from " . TABLE_ORDERS . " where orders_id = '" . (int)$order_id . "'");
      $curr = tep_db_fetch_array($curr_check);

      if ( ($curr['currency'] != $order->info['currency']) || ($cartID !=
substr($cart_PayPal_Standard_ID, 0, strlen($cartID))) ) {
        $check_query = tep_db_query('select orders_id from ' . TABLE_ORDERS_STATUS_HISTORY . ' where orders_id = "' . (int)$order_id . '" limit 1');
        if (tep_db_num_rows($check_query) < 1) {
          tep_db_query('delete from ' . TABLE_ORDERS . ' where orders_id = "' .
(int)$order_id . '"');
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
              if (tep_not_null($GLOBALS[$class]->output[$i]['title']) && tep_not_null($GLOBALS[$class]->output[$i]['text'])) {
                $order_totals[] = array(
                  'code' => $GLOBALS[$class]->code,
                  'title' => $GLOBALS[$class]->output[$i]['title'],
                  'text' => $GLOBALS[$class]->output[$i]['text'],
                  'value' => $GLOBALS[$class]->output[$i]['value'],
                  'sort_order' => $GLOBALS[$class]->sort_order);
              }
            }
          }
        }
      }

      $sql_data_array = array(
        'customers_id' => $customer_id,
        'customers_name' => $order->customer['firstname'] . ' ' . $order->customer['lastname'],
        'customers_company' => $order->customer['company'],
        'customers_street_address' =>$order->customer['street_address'],
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
        'billing_company' => $order->billing['company'],         'billing_street_address' => $order->billing['street_address'],
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
              $attributes_query = "select popt.products_options_name,
              poval.products_options_values_name, pa.options_values_price, pa.price_prefix, pad.products_attributes_maxdays, pad.products_attributes_maxcount , pad.products_attributes_filename from " . TABLE_PRODUCTS_OPTIONS . " popt, " . TABLE_PRODUCTS_OPTIONS_VALUES . " poval, " . TABLE_PRODUCTS_ATTRIBUTES . " pa left join " . TABLE_PRODUCTS_ATTRIBUTES_DOWNLOAD . " pad on pa.products_attributes_id=pad.products_attributes_id where pa.products_id = '" . $order->products[$i]['id'] . "' and pa.options_id = '" . $order->products[$i]['attributes'][$j]['option_id'] . "' and pa.options_id = popt.products_options_id and pa.options_values_id = '" . $order->products[$i]['attributes'][$j]['value_id'] . "' poval.products_options_values_id
and pa.options_values_id = and popt.language_id = '" . $languages_id . "' and poval.language_id = '" . $languages_id . "'";

              $attributes = tep_db_query($attributes_query);
            } else {
              $attributes = tep_db_query("select popt.products_options_name, poval.products_options_values_name, pa.options_values_price, pa.price_prefix from " . TABLE_PRODUCTS_OPTIONS . " popt, " . TABLE_PRODUCTS_OPTIONS_VALUES . " poval, " . TABLE_PRODUCTS_ATTRIBUTES . " pa where pa.products_id = '" . $order->products[$i]['id'] . "' and pa.options_id = '" . $order->products[$i]['attributes'][$j]['option_id'] . "' and pa.options_id = popt.products_options_id and pa.options_values_id = '" . $order->products[$i]['attributes'][$j]['value_id'] . "' and pa.options_values_id = poval.products_options_values_id and popt.language_id = '" . $languages_id . "' and poval.language_id = '" . $languages_id . "'");
            }

            $attributes_values = tep_db_fetch_array($attributes);
            $sql_data_array = array(
              'orders_id' => $insert_id,
              'orders_products_id' => $order_products_id,
              'products_options' => $attributes_values['products_options_name' ],
              'products_options_values' =>   $attributes_values[' products_options_values_name'],
              'options_values_price' => $attributes_values['options_values_price'],
              'price_prefix' => $attributes_values['price_prefix']);

            tep_db_perform(TABLE_ORDERS_PRODUCTS_ATTRIBUTES, $sql_data_array);

            if ((DOWNLOAD_ENABLED == 'true') && isset($attributes_values['products_attributes_filename']) && tep_not_null($attributes_values['products_attributes_filename'])) {
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

      $cart_PayPal_Standard_ID = $cartID . '-' . $insert_id;
      tep_session_register('cart_PayPal_Standard_ID');
    }
  }

  return false;
}
```

在 108 行验证 pre_confirmation_check 生成的识别 ID (cartID),每次生成新订单后会生成新 的会话值 cart_PayPal_Standard_ID(第 267-268 行代码的功能),如果未设置会话值 cart_PayPal_Standard_ID 说明之前没有新订单的生成,则不有必要执行删除旧订单的代码部分。 除了判断会话值 cart_PayPal_Standard_ID,考虑是否要删除旧的订单,还取决于另外两个条件, 一是币种是否有更改,二是 cart_PayPal_Standard_ID 的值是否有变。当币种有改变或者 cart_PayPal_Standard_ID 的值有变化时,说明确实需要创建新订单来保存当前的订单,所以要删 除旧订单。

第 134-269 行完成了整个订单的数据库实现。它们包括:新建一个 Order 订单,保存统计数 据 Order_total,保存订单的所有产品以及所有产品的属性,当产品为在线下载内容时,还需要更 新下载数据表。

#### function process_button()

输出完成订单的按钮代码，调用代码见 checkout_confirmation.php[323]

在确认订单页面时,会显示一个“Confirm Order”的按钮,process_button 方法的功能是通过 hidden 元素输出一些必要的参数给下一步付款的界面。

Paypal 通过 form 的 post 方法来传递参数,Paypal 付款的参数表详情参见附录:《Paypal 网 站付款标准版参数表》

其中参数里的 notify_url 值是 Paypal IPN(Instant Payment Notification,即时付款通知)时的 URL。关于 Paypal IPN 的返回将如何处理我们将在下面的章节里讨论。


> 提示:
> Paypal IPN 是 Paypal 提供的一种通知确认功能,通常用于订单的核对,订单状态的变更等用途。
> 是 Paypal 支付系统向网站提供数据更新的最常用方法(除非直接使用 Paypal API 进行开发,否则 IPN 方式将是数据同步的唯一途径)。
> 
> Paypal IPN的介绍请参见: https://cms.paypal.com/us/cgi-bin/?cmd=_render-content&content_ID=developer/e_howto_html_IPNan dPDTVariables
> 
> Paypal IPN的代码示例请见:
   https://cms.paypal.com/us/cgi-bin/?cmd=_render-content&content_ID=developer/library_code

#### function before_process()

生成正式订单前的准备工作，调用代码见 checkout_process.php[81]

如果当前$order_id 的订单状态等于参数表里设置的“Prepare Order Status”时,则在订单状态 记录表里添加一个“Prepare Order Status”的状态。

然后更新订单状态为已付款,并添加一条已付款的状态记录。

第 428-460 行的代码只有在启用了“真实库存”时才有效,其作用有在产品数量上减去已下 订单的数量,如果启用了“缺货不能下单”的设置时,还会设置产品的状态为 out of stock,使产品下架。

第 463 行更新产品的已购买资料。

第 465-543 行代码的实现了向用户信箱发送完整订单信息的邮件,如果有额外需要通知的人 员(可以指定管理邮箱来实时了解所有的下订单的情况),也会一并发送相同内容的邮件。

includes/modules/payment/paypal_standard.php [548]

```php
$cart->reset(true);

// unregister session variables used during checkout
tep_session_unregister('sendto');
tep_session_unregister('billto');
tep_session_unregister('shipping');
tep_session_unregister('payment');
tep_session_unregister('comments');
tep_session_unregister('cart_PayPal_Standard_ID');
```

最后的清理工作包括:复位购物车,清理付款相关的会话值。

> 提示:
> 在 Paypal 付款模块的 before_process 方法最后,显示了自动跳转到 checkout_success.php 文件。 因为在其它一些的付款模块中,实现的订单数据库生成的过程是由 checkout_process.php 来完成 的,所以当 checkout_process 调用 before_process 后,才会真正启动订单数据库生成的程序。
> 而在前面的内容里,我们已经看到了 Paypal 付款模块会自己处理订单生成,所以 Paypal 模块已经没有 必要再重复生成订单,因而直接跳转到 checkout_success 是完全正确的。

#### function after_process()

生成订单完成的善后处理

调用代码见 checkout_process.php[292]

此方法在绝大多数付款模块里都没有实际的用处,因为都是直接返回 false。

### 付款模块的调用流程


上图完整的展示了在一个付款的完整过程中,与付款相关的页面是如何与付款模块进行交互的。

#### 选择付款

在完全一个订单,首先要从选择付款方式开始(暂时忽略其它因素),checkout_payment.php 文件 负责呈现所有可用的付款模块并提交它。

checkout_payment.php [224]

```php
$selection = $payment_modules->selection();
```

$payment_modules 变量在第 79 行生成,它为一个 payment 类的实例。

checkout_payment.php [77]

```php
// load all enabled payment modules
require(DIR_WS_CLASSES . 'payment.php');
$payment_modules = new payment;
```

payment 类是一个管理付款模块的类,它的功能主要是列举出所有可用的付款模块,然后将 自己的方法与付款模块的方法一一关联(此处使用完全名称关联),当执行类的方法时(例如方法 名:F),自动循环所有可用付款模块的方法(即调用付款模块的方法 F)。
     
> 提示:
> 上面说到的情况是在未选定付款模块时的情况,当只选定了某个付款模块时(假设为 P),我们调 用$payment 的方法 F($payment->F()),那么它将只调用付款模块 P 的方法 F,即 P->F()了,而不会循环调用所有可用的付款模块的 F 方法。

虽然 payment 类的其它方法都只是单纯的付款模块方法的间接调用,但它的初始化过程还是值得我们了解的。

付款模块类 includes/classes/payment.php[16]

```php
// class constructor
function payment($module = '') {
  global $payment, $language, $PHP_SELF;

  if (defined('MODULE_PAYMENT_INSTALLED') && tep_not_null(MODULE_PAYMENT_INSTALLED)) {
    $this->modules = explode(';', MODULE_PAYMENT_INSTALLED);

    $include_modules = array();
    
    if ( (tep_not_null($module)) && (in_array($module . '.' . substr($PHP_SELF, (strrpos($PHP_SELF, '.')+1)), $this->modules)) ) {
      $this->selected_module = $module;

      $include_modules[] = array('class' => $module, 'file' => $module . '.php');
    } else {
      reset($this->modules);
      while (list(, $value) = each($this->modules)) {
        $class = substr($value, 0, strrpos($value, '.'));
        $include_modules[] = array('class' => $class, 'file' => $value);
      }
    }

    for ($i=0, $n=sizeof($include_modules); $i<$n; $i++) {
      include(DIR_WS_LANGUAGES . $language . '/modules/payment/' . $include_modules[$i]['file']);
      include(DIR_WS_MODULES . 'payment/' . $include_modules[$i]['file']);

      $GLOBALS[$include_modules[$i]['class']] = new $include_modules[$i]['class'];
    }

    // if there is only one payment method, select it as default because in
    // checkout_confirmation.php the $payment variable is being assigned the
    // $HTTP_POST_VARS['payment'] value which will be empty (no radio button selection possible)
    if ( (tep_count_payment_modules() == 1) && (!isset($GLOBALS[$payment]) || (isset($GLOBALS[$payment]) && !is_object($GLOBALS[$payment]))) ) {
      $payment = $include_modules[0]['class'];
    }
    
    if ( (tep_not_null($module)) && (in_array($module, $this->modules)) && (isset($GLOBALS[$module]->form_action_url)) ) {
      $this->form_action_url = $GLOBALS[$module]->form_action_url;
    }
  }
}
```

细心的读者应该已经注意到,payment 类的初始化过程几乎可以说是 shipping 类的初始化过 程的翻版,虽然如此,但 payment 类还是与 shipping 类有所不同,其中一个差别就是当指定了模 块名称时(选择了付款模块或运输模块)的处理,

从 26 行可以看到 payment 类直接将选定模块变量$selected_module赋值为参数$module的值,使用的类名与文件直接由$module构成。而shipping 类则需要将$module 值进行分解后,才能使用(见 includes/classes/shipping.php[26])。


> 提示:
> shipping 类需要将$module 值进行分解后才能使用,是因为一个 shipping 运输模块可以拥有多个子 运输方式(method),所以$module 值是由 shipping 名称与子运输方式组合而成;而 payment 付款 模块不具有此种特性,所以 payment 类直接使用$module 传递付款模块的名称即可。

第二个差别在于 payment 类会在选定了付款模块后设置全局变量$payment 为当前选定模块的名称。

最后一个差别是针对付款模块的成员$form_action_url 的处理,在运输模块里没有相似的成员,所以它不需要处理。

$form_action_url 是针对在线付款的模块而设置的,这些模块需要跳转到相关的付款网关(如: Paypal)处理剩下的付款环节,所以在付款确认时,将$form_action_url 设置为跳转 URL,页面就 会自动跳转到付款网关,同时将所有的参数以 hiddend 元素传递过去。(代码见: checkout_confirmation.php [116-122])

> 提示:
> 其实除了上面提示的三个差别外,其中的不同还包括:使用不同的数据库常量保存已安装的模块, 模块文件路径不同。

我们回到付款选择页面 checkout_payment.php,第 246-309 行的代码用于输出所有可用付款模块。 当用户点击“Continue”的按钮时,我们接着进入下一环节:付款确认。

#### 确认付款

checkout_confirmation.php[46]

```php
// load the selected payment module
require(DIR_WS_CLASSES . 'payment.php');
$payment_modules = new payment($payment);
```

初始化一个 payment 类,并将选择的模块名$payment 作为参数传入。然后执行付款模块的 update_status 方法[53],调用 pre_confirmation_check 方法[60],做好预准备工作。执行 confirmation 方法[250],输出确认按钮代码[323]。

在这里需要说明一下$form_action_url 的处理,代码如下:

includes/classes/payment.php [116]

```php
if (isset($$payment->form_action_url)) {
  $form_action_url = $$payment->form_action_url;
} else {
  $form_action_url = tep_href_link(FILENAME_CHECKOUT_PROCESS, '', 'SSL');
}

echo tep_draw_form('checkout_confirmation', $form_action_url, 'post');
```

由于在 payment 类里已经将所以付款模块都注册成了全局变量(直接以类名作为变量名),所以 $$payment 便得到选定的付款模块的类实例,如果模块有$form_action_url 成员,则将提交表单的 Action 值设置为$form_action_url(由 tep_draw_form 完成),否则设置为默认订单处理页面 checkout_process.php。

#### 订单处理

checkout_process.php [44]

```php
// load selected payment module
require(DIR_WS_CLASSES . 'payment.php');
$payment_modules = new payment($payment);
```

一开始,如确认订单页面 checkout_confirmation.php 一样,创建 payment 类的实例 $payment_modules。前面我们说明如果在实例化 payment 类时指定了$payment 参数值,则表明选定了单个付款模块,以后由 payment 类的所有间接调用都只作用于该模块上。

checkout_process.php [69]

```php
$payment_modules->update_status();
```

先执行付款模块的 update_status 方法,检验有关的数据的正确性,或者执行一些更新操作。

checkout_process.php [80]

```php
// load the before_process function from the payment modules
$payment_modules->before_process();
```

在正式进行订单的数据生成前,调用付款模块的 before_process 方法,为订单的生成作准备。

也就是在这里,成为普通付款模块与在线支付类付款模块的分水岭,前面在的分析 before_process 方法的代码时,我们讲到对于普通付款模块而言,before_process 方法只需要返回 false,也就是可以什么都不用做;

而在线支付模块则会自行处理订单的生成,然后前往在线支付的网关进行付款。      

> 提示:
> 
> 在线支付类付款模块的订单生成过程与 checkout_process.php 文件的过程是一样的,有兴趣的读者可以对比其中的代码。
> 
> 在线支付类付款模块因为通过 before_process 方法会跳转到支付网关(外部 URL),所以该类
模块将不涉及以下所谈的内容。

具体的订单生成代码说明在 Paypal Website Payment Standard 模块的 confirmation 方法中已经讲过, 在此不再累赘。

checkout_process.php [291]

```php
// load the after_process function from the payment modules
$payment_modules->after_process();
$cart->reset(true);
// unregister session variables used during checkout
tep_session_unregister('sendto');
tep_session_unregister('billto');

tep_session_unregister('shipping');
tep_session_unregister('payment');
tep_session_unregister('comments');
tep_redirect(tep_href_link(FILENAME_CHECKOUT_SUCCESS, '', 'SSL'));
```

处理完订单的生成,然后执行付款模块的 after_process 方法。此方法一般返回 false。 接下来是清理工作,首先是清空购物车,接着注销所有付款相关的 SESSION 会话值,最后直接 跳转到显示订单成功完成信息的 checkout_success.php 文件。

由此,付款及订单生成的整个过程也就宣告完成。
