## 编写运输模块

好了,通过上面的介绍,我们已经对运输模块的情况了如指掌了,那么让我们动手来编写一
个运输模块吧。

我们要编写的是一个叫 flytexpress 的模块,此模块是通过调用国内一家物流公司 flytexpress.com 的 web service 来实现的。

此模块实现的功能有:

-  传递订单信息到flytexpress.com的WebService,返回应计的运费。
-  自定义税类
-  自定义税区
-  自定义排序号

下面是 flytexpress 模块的参数表:
 
### 运输模块

flytexpress 运输模块 includes/modules/shipping/flytexpress.php <?php

```php
/**
* Flytexpress Shipping Method
* Data from flytexpress.com
*/
class flytexpress {
  var $code, $title, $description, $icon, $enabled;

  // class constructor
  function flytexpress () {
    global $order;

    $this->code = 'flytexpress';
    $this->title = MODULE_SHIPPING_FLYTEXPRESS_TEXT_TITLE;
    $this->description = MODULE_SHIPPING_FLYTEXPRESS_TEXT_DESCRIPTION; $this->sort_order = MODULE_SHIPPING_FLYTEXPRESS_SORT_ORDER;
    $this->icon = '';
    $this->tax_class = MODULE_SHIPPING_FLYTEXPRESS_TAX_CLASS;
    $this->enabled = ((MODULE_SHIPPING_FLYTEXPRESS_STATUS == 'True') ? true : false);

    // remote web service call parameters
    $this->wsdl = 'http://www.flytexpress.com/FreightInquiryService.asmx?WSDL';
    $this->postType = 'USPS3DAYS';

    if ( ($this->enabled == true) && ((int)MODULE_SHIPPING_FLYTEXPRESS_ZONE > 0) ) {
      $check_flag = false;
      $check_query = tep_db_query("select zone_id from " . TABLE_ZONES_TO_GEO_ZONES . " where geo_zone_id = '" . MODULE_SHIPPING_FLYTEXPRESS_ZONE . "' and zone_country_id = '" . $order->delivery['country']['id'] . "' order by zone_id");
      
      while ($check = tep_db_fetch_array($check_query)) {
        if ($check['zone_id'] < 1) {
          $check_flag = true;
          break;
        } elseif ($check['zone_id'] == $order->delivery['zone_id']) {
          $check_flag = true;
          break;
        }
      }

      if ($check_flag == false) {
        $this->enabled = false;
      }
    }
  }

  // class methods
  function quote($method = '') {
    global $order,$shipping_weight;

    $cost = $this->_getCost();
    
    if('E' == strtoupper(substr($cost,0,1)) || (int)$cost<=0){
      // error
      return false;
    }
    $this->quotes = array('id' => $this->code,
                       'module' => MODULE_SHIPPING_FLYTEXPRESS_TEXT_TITLE,
                       'methods' => array(array('id' => $this->code,
                                            'title' => sprintf(MODULE_SHIPPING_FLYTEXPRESS_TEXT_WAY,round($shipping_weight/1000,2)),
                                            'cost' => $cost)));

    if ($this->tax_class > 0) {
      $this->quotes['tax'] = tep_get_tax_rate($this->tax_class,
$order->delivery['country']['id'], $order->delivery['zone_id']);
    }

    if (tep_not_null($this->icon)) $this->quotes['icon'] = tep_image($this->icon, $this->title);

    return $this->quotes;
  }

  function check() {
    if (!isset($this->_check)) {
      $check_query = tep_db_query("select configuration_value from " .
TABLE_CONFIGURATION . " where configuration_key =
'MODULE_SHIPPING_FLYTEXPRESS_STATUS'");
       $this->_check = tep_db_num_rows($check_query);
     }
     return $this->_check;
  }

  function install() {
    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title,configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, set_function, date_added) values ('Enable Flytexpress Shipping', 'MODULE_SHIPPING_FLYTEXPRESS_STATUS', 'True', 'Do you want to offer Flytexpress shipping?', '6', '0', 'tep_cfg_select_option(array(\'True\', \'False\'), ', now())");

    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, use_function, set_function, date_added) values ('Tax Class', 'MODULE_SHIPPING_FLYTEXPRESS_TAX_CLASS', '0', 'Use the following tax    class on the shipping fee.', '6', '0', 'tep_get_tax_class_title', 'tep_cfg_pull_down_tax_classes(', now())");

    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, use_function, set_function, date_added) values ('Shipping Zone', 'MODULE_SHIPPING_FLYTEXPRESS_ZONE', '0', 'If a zone is selected, only enable this shipping method for that zone.', '6', '0', 'tep_get_zone_class_title', 'tep_cfg_pull_down_zone_classes(', now())");

    tep_db_query("insert into " . TABLE_CONFIGURATION . " (configuration_title, configuration_key, configuration_value, configuration_description, configuration_group_id, sort_order, date_added) values ('Sort Order', 'MODULE_SHIPPING_FLYTEXPRESS_SORT_ORDER', '0', 'Sort order of display.', '6', '0', now())");
  }

  function remove() {
    tep_db_query("delete from " . TABLE_CONFIGURATION . " where configuration_key in ('" . implode("', '", $this->keys()) . "')");
  }

  function keys() {
    return array('MODULE_SHIPPING_FLYTEXPRESS_STATUS',
'MODULE_SHIPPING_FLYTEXPRESS_TAX_CLASS', 'MODULE_SHIPPING_FLYTEXPRESS_ZONE',
'MODULE_SHIPPING_FLYTEXPRESS_SORT_ORDER');
  }

  function _getCost() {
    global $order,$shipping_weight;

    $parameters = array('postType' => $this->postType,
                        'fromAddressId' => '',
                        'toCountry' => $order->delivery['country']['iso_code_2'],
                        'weight' => round($shipping_weight/1000,2) // 换算成千克单位
                       );

    if(class_exists('SoapClient')){ // 通过 SoapClient 类实现 Web Service 的调用
      $client = new SoapClient($this->wsdl);
      $result = $client->InquiryFreight($parameters); // InquireFreight是 flytexpress web service的一个方法;
      $result = $result->InquiryFreightResult; // 获取运费
    } else {
      $result = 0;
    }

    return $result;
  }
}
?>
```

### 语言文件

flytexpress 运输模块语言文件 includes/languages/english/modules/shipping/flytexpress.php

```php
<?php
/**
* Flytexpress Shipping Method
* Data from flytexpress.com
*/
define('MODULE_SHIPPING_FLYTEXPRESS_TEXT_TITLE', 'FlytExpress.com');
define('MODULE_SHIPPING_FLYTEXPRESS_TEXT_DESCRIPTION', 'Shipping Cost return from
flyexpress.com');
define('MODULE_SHIPPING_FLYTEXPRESS_TEXT_WAY', 'Weight: %s KG.<br/>Delivery
Estimates: 2 - 6 days to major destinations.');
?>
```

### Web Service说明

因为我们要调用Web Service ,所以我们使用了SoapClient类。SoapClient类需要在PHP里安装 SOAP的扩展,并非默认的模块。如果未启用SOAP,请参考PHP帮助的SOAP扩展安装章节,或者 可以使用Nusoap类,此类实现了Web Service的调用,使用方法与SoapClient相似。要了解Nusoap 类的详细情况,请登陆http://www.nusoap.com/,查看相关介绍。

> 提示:
> Flytexpress的web service使用千克为单位,而在此处我们使用的单位是克,所以第99行处使用了单位换算。将原来的克换算成千克,然后将重量传给 Web Service。

我们来看最终的显示效果:

