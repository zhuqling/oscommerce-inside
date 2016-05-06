## 编写统计模块

我们要编写的统计模块是一个允许用户购买保险的选项,在确认付款时,我们允许用户选择是否为此笔交易购
买保险。当用户选择保险时,表示对发货出错的情况可以进行重新发货保障。

那么如何将保险选项放入统计的内容里呢?在这里我们将保险选项当成一个保险统计模块进行处理。

保险统计模块提供的参数包括:

-  Display Shipping Insurance:是否生效保险统计模块
-  Sort Order:排序号
-  InsuranceAmount:保险金额(美元USD),当转换币
  种时自动转换此保险金额。
-  ApplyInsuranceFeeToWhichOrders:是否限制使用区域,national:只使用于本国订单,international:只使用于国际(非本国)订单,both:无限制
-  International Multiplier:国际订单系数,当为国际订单 时,收取的实际保险金额等于输入的保险金额乘以国际订单系数。

### 统计模块

保险统计模块 includes/modules/order-total/ot_insurance.php

```php
<?php
class ot_insurance {
  var $title, $output;

  function ot_insurance() {
    $this->code = 'ot_insurance';
    $this->title = MODULE_ORDER_TOTAL_INSURANCE_TITLE;
    $this->description = MODULE_ORDER_TOTAL_INSURANCE_DESCRIPTION;
    $this->enabled = MODULE_ORDER_TOTAL_INSURANCE_STATUS;
    $this->sort_order = MODULE_ORDER_TOTAL_INSURANCE_SORT_ORDER;
    $this->multiplier = MODULE_ORDER_TOTAL_INSURANCE_INT_MULT;
    $this->output = array();
  }

  function process() {
    global $order, $currencies;
    global $ot_shipping;

    $choose_insurance = $_SESSION['choose_insurance'];
    if (MODULE_ORDER_TOTAL_INSURANCE_STATUS == 'true' && $choose_insurance == '1') {
      switch (MODULE_ORDER_TOTAL_INSURANCE_DESTINATION) {
        case 'national':
          if ($order->delivery['country_id'] == STORE_COUNTRY) $pass = true; break;
        case 'international':
          if ($order->delivery['country_id'] != STORE_COUNTRY) $pass = true; break;
        case 'both':
          $pass = true;
          break;
        default:
          $pass = false;
          break;
      }
    } else {
      $pass = false;
    }

    if ($pass == true) {
      $this_amount = MODULE_ORDER_TOTAL_INSURANCE_AMOUNT;

      // If international shipment, multiply insurance charge by multiplier
      if ($order->delivery['country_id'] != STORE_COUNTRY) {
        $this_amount *= $this->multiplier;
      }

      $order->info['total'] += $this_amount;
      $this->output[] = array(
        'title' => $this->title . ':',
        'text' => $currencies->format($this_amount, true, $order->info['currency'], $order->info['currency_value']),
        'value' => $this_amount
      );
    } 
  }

  function check() {
    if (!isset($this->check)) {
      $check_query = tep_db_query("select configuration_value from " . TABLE_CONFIGURATION . " where configuration_key = 'MODULE_ORDER_TOTAL_INSURANCE_STATUS'");
      $this->check = tep_db_num_rows($check_query);
    }
    return $this->check;
  }

  function keys() {
    return array('MODULE_ORDER_TOTAL_INSURANCE_STATUS','MODULE_ORDER_TOTAL_INSURANCE_SORT_ORDER', 'MODULE_ORDER_TOTAL_INSURANCE_AMOUNT', 'MODULE_ORDER_TOTAL_INSURANCE_DESTINATION','MODULE_ORDER_TOTAL_INSURANCE_INT_MULT');
  }

  function install() {
    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, set_function, date_added) values ('Display Shipping Insurance', 'MODULE_ORDER_TOTAL_INSURANCE_STATUS', 'true', 'Do you want to offer Shipping Insurance?', '6', '1','tep_cfg_select_option(array(\'true\', \'false\'), ', now())");
    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, date_added) values ('Sort Order', 'MODULE_ORDER_TOTAL_INSURANCE_SORT_ORDER', '4', 'Sort order of display.', '6', '2', now())");
    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, use_function, date_added) values ('Insurance Amount.', 'MODULE_ORDER_TOTAL_INSURANCE_AMOUNT', '1.99', 'How much do you want to insurance?', '6', '4', 'currencies->format', now())");
    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, set_function, date_added) values ('Apply Insurance Fee To Which Orders', 'MODULE_ORDER_TOTAL_INSURANCE_DESTINATION', 'both', 'Apply insurance fee for orders sent to the set destination.', '6', '8', 'tep_cfg_select_option(array(\'national\', \'international\', \'both\'), ', now())");
    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, date_added) values ('International Multiplier', 'MODULE_ORDER_TOTAL_INSURANCE_INT_MULT', '1', 'For International Orders, multiply the total insurance cost by this number:', '6', '10', now())");
  }

  function remove() {
    $keys = '';
    $keys_array = $this->keys();

    for ($i=0; $i<sizeof($keys_array); $i++) {
      $keys .= "'" . $keys_array[$i] . "',";
    }

    $keys = substr($keys, 0, -1);
    tep_db_query("delete from " . TABLE_CONFIGURATION . " where configuration_key in (" . $keys . ")");
  }
}
?>
```

我们使用 SESSION 会话值$HTTP_SESSION_VARS['choose_insurance']来传递用户是否选择了保 险选项,在确认付款页面将如果修改以便支持保险选项生效将在接下来的内容里讲到。

### 语言文件

语言文件 includes/languages/english/modules/order_total/ot_insurance.php

```php
<?php
define('MODULE_ORDER_TOTAL_INSURANCE_TITLE', 'Shipping Insurance');
define('MODULE_ORDER_TOTAL_INSURANCE_DESCRIPTION', 'Shipping Insurance');
?>
```

### 其它修改

要完成 insurance 保险统计模块,还必须对选择付款方式页面 checkout_payment.php 做一些必 要的修改。

修改 checkout_payment.php 文件,要求查找以下代码:

```php
<?php
  $radio_buttons++;
}
?>
          </table>
        </td>
      </tr>
    </table>
  </td>
</tr>
```

在其后添加如下的代码:

```php
<tr>
  <td><?php echo tep_draw_separator('pixel_trans.gif', '100%', '10'); ?></td>
</tr>
<tr>
  <td class="main"><b><?php echo TEXT_SHIPPING_INSURANCE_TITLE; ?></b></td>
</tr>
<tr>
  <td>
    <table border="0" width="100%" cellspacing="1" cellpadding="2" class="infoBox">
      <tr class="infoBoxContents">
        <td class="main" width="100%" align="left"><input type="checkbox" name="choose_insurance" value="1" checked>&nbsp;&nbsp;&nbsp;<?PHP echo TEXT_SHIPPING_INSURANCE_CHOICE; ?>&nbsp;&nbsp;<div class="smallText"><?PHP printf(TEXT_SHIPPING_INSURANCE_DISCLAIMER,$currencies->format(MODULE_ORDER_TOTAL_INSURANCE_AMOUNT)); ?></div></td>
      </tr>
    </table>
  </td>
</tr>
<tr>
  <td><?php echo tep_draw_separator('pixel_trans.gif', '100%', '10'); ?></td>
</tr>
```

这里代码的作用是显示出保险选项以供用户选择是否需要购买保险。 

#### 修改 checkout_confirmation.php 文件,查找以下代码:

```php
$shipping_modules = new shipping($shipping);
```

在其后添加以下代码:

```php
$_SESSION['choose_insurance'] = $_POST[choose_insurance];
```

  
### 打开文件 includes/languages/english.php

在其最后添加如下代码:

```php
define('TEXT_SHIPPING_INSURANCE_TITLE', 'Shipping Insurance');
define('TEXT_SHIPPING_INSURANCE_CHOICE', 'Add Shipping Insurance to your order?');
define('TEXT_SHIPPING_INSURANCE_DISCLAIMER', 'We offer Shipping Insurance %s to protect your package against any lost or damaged shipments. Any missing issues reported, we will reship your order immediately.');
```

最终的保险选项显示效果以下:

当选择了购买保险,订单确认页面会增加一条保险费用的统计栏
