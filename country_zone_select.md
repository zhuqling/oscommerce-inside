## 国家与省份/区列表联动

由于 osCommerce 系统默认未实现国家列表与省份/区列表的联动,所以当选择国家后,必须刷新页面才能选择该国家的省份/区列表,很不方便,因此我们选择使用Country-State selector插件来 实现实时的国家列表与省份/区列表联动。

插件地址:http://addons.oscommerce.com/info/2028

[安装]

1、复制以下文件到相对应的目录

- catalog/includes/functions/ajax.php
- catalog/includes/ajax.js.php      
   
2、修改文件 catalog/address_book_process.php
查找以下代码(大约在第 13 行): 

`require('includes/application_top.php');`

在其后添加以下代码:

```php
// +Country-State Selector
require(DIR_WS_FUNCTIONS . 'ajax.php');
if (isset($HTTP_POST_VARS['action']) && $HTTP_POST_VARS['action'] == 'getStates' && isset($HTTP_POST_VARS['country'])) {
  ajax_get_zones_html(tep_db_prepare_input($HTTP_POST_VARS['country']), true);
} else {
// -Country-State Selector
```

查找(大约在第 99 行):

```php
if (ACCOUNT_STATE == 'true') {
$zone_id = 0;
$check_query = tep_db_query("select count(*) as total from " . TABLE_ZONES . "
where zone_country_id = '" . (int)$country . "'");
     $check = tep_db_fetch_array($check_query);
     $entry_state_has_zones = ($check['total'] > 0);
     if ($entry_state_has_zones == true) {
$zone_query = tep_db_query("select distinct zone_id from " . TABLE_ZONES . " where zone_country_id = '" . (int)$country . "' and (zone_name = '" . tep_db_input($state) . "' or zone_code = '" . tep_db_input($state) . "')");
       if (tep_db_num_rows($zone_query) == 1) {
        $zone = tep_db_fetch_array($zone_query);
        $zone_id = $zone['zone_id'];
       } else {
        $error = true;
        $messageStack->add('addressbook', ENTRY_STATE_ERROR_SELECT);
      }
} else {
   if (strlen($state) < ENTRY_STATE_MIN_LENGTH) {
     $error = true;
     $messageStack->add('addressbook', ENTRY_STATE_ERROR);
   }
}
}
```
替换成以下代码:

```php
if (ACCOUNT_STATE == 'true') {
  // +Country-State Selector
  if ($zone_id == 0) {
    // -Country-State Selector
    if (strlen($state) < ENTRY_STATE_MIN_LENGTH) {
      $error = true;
      $messageStack->add('addressbook', ENTRY_STATE_ERROR);
    }
  }
}
```

查找(第 224 行): 

`$entry = array();`

替换为:

```php
$entry = array();
// +Country-State Selector
if (!isset($country)) $country = DEFAULT_COUNTRY;
$entry['entry_country_id'] = $country;
// -Country-State Selector
```


查找(大概第 271 行):

```php
<?php
  if (!isset($HTTP_GET_VARS['delete'])) {
   include('includes/form_check.js.php');
  }
<?
```

替换为:

```php
<?php
  if (!isset($HTTP_GET_VARS['delete'])) {
   include('includes/form_check.js.php');
 // +Country-State Selector
   require('includes/ajax.js.php');
 // -Country-State Selector
} ?>
```

在页面最后,查找:

`<?php require(DIR_WS_INCLUDES . 'application_bottom.php'); ?>`

替换为:

```php
<?php require(DIR_WS_INCLUDES . 'application_bottom.php'); ?>
<?php
// +Country-State Selector
}
// -Country-State Selector ?>
```

3、修改文件 catalog/checkout_payment_address.php 

查找(第 13 行):

`require('includes/application_top.php');`

在其下添加:

```php
// +Country-State Selector
require(DIR_WS_FUNCTIONS . 'ajax.php');
if (isset($HTTP_POST_VARS['action']) && $HTTP_POST_VARS['action'] == 'getStates'
&& isset($HTTP_POST_VARS['country'])) {
ajax_get_zones_html(tep_db_prepare_input($HTTP_POST_VARS['country']), true);
} else {
// -Country-State Selector
```

查找(第 93 行):

```php
$zone_id = 0;
$check_query = tep_db_query("select count(*) as total from " . TABLE_ZONES . " where zone_country_id = '" . (int)$country . "'");
       $check = tep_db_fetch_array($check_query);
       $entry_state_has_zones = ($check['total'] > 0);
       if ($entry_state_has_zones == true) {
$zone_query = tep_db_query("select distinct zone_id from " . TABLE_ZONES . " where zone_country_id = '" . (int)$country . "' and (zone_name like '" . tep_db_input($state) . "%' or zone_code like '%" . tep_db_input($state) . "%')");
        if (tep_db_num_rows($zone_query) == 1) {
          $zone = tep_db_fetch_array($zone_query);
          $zone_id = $zone['zone_id'];
        } else {
          $error = true;
          $messageStack->add('checkout_address', ENTRY_STATE_ERROR_SELECT);
        }
       } else {
          $error = true;
          $messageStack->add('checkout_address', ENTRY_STATE_ERROR_SELECT);
        }
       } else {
        if (strlen($state) < ENTRY_STATE_MIN_LENGTH) {
          $error = true;
             $messageStack->add('checkout_address', ENTRY_STATE_ERROR);
   }
}
}
```

替换为:

```php
if (ACCOUNT_STATE == 'true') {
  // +Country-State Selector
  if ($zone_id == 0) {
  // -Country-State Selector
   if (strlen($state) < ENTRY_STATE_MIN_LENGTH) {
     $error = true;
     $messageStack->add('checkout_address', ENTRY_STATE_ERROR);
   }
  }
}
```

查找(第 187 行):

```php
// if no billing destination address was selected, use their own address as default
if (!tep_session_is_registered('billto')) {
  $billto = $customer_default_address_id;
}
```

替换为:

```php
// if no billing destination address was selected, use their own address as default 
if (!tep_session_is_registered('billto')) {
   $billto = $customer_default_address_id;
  }
  // +Country-State Selector
  if (!isset($country)){$country = DEFAULT_COUNTRY;}
// -Country-State Selector
```

查找(大概在第 271 行): 

`<?php require(DIR_WS_INCLUDES . 'form_check.js.php'); ?>`

替换为:

```php
<?php
require(DIR_WS_INCLUDES . 'form_check.js.php');
require(DIR_WS_INCLUDES . 'ajax.js.php');
?>
```

在页面的最后查找:

`<?php require(DIR_WS_INCLUDES . 'application_bottom.php'); ?>`

替换为:

```php
<?php require(DIR_WS_INCLUDES . 'application_bottom.php'); ?>
<?php
// +Country-State Selector
}
// -Country-State Selector
?>
```

4、修改文件 catalog/checkout_shipping_address.php 

查找(第 13 行):

`require('includes/application_top.php');`

紧接着它的下面添加以下代码:

```php
// +Country-State Selector
require(DIR_WS_FUNCTIONS . 'ajax.php');
  if (isset($HTTP_POST_VARS['action']) && $HTTP_POST_VARS['action'] == 'getStates'
&& isset($HTTP_POST_VARS['country'])) {
ajax_get_zones_html(tep_db_prepare_input($HTTP_POST_VARS['country']), true);
} else {
// -Country-State Selector
```

查找(第 93 行):

```php
$zone_id = 0;
$check_query = tep_db_query("select count(*) as total from " . TABLE_ZONES . " where zone_country_id = '" . (int)$country . "'");
       $check = tep_db_fetch_array($check_query);
       $entry_state_has_zones = ($check['total'] > 0);
       if ($entry_state_has_zones == true) {
$zone_query = tep_db_query("select distinct zone_id from " . TABLE_ZONES . " where zone_country_id = '" . (int)$country . "' and (zone_name like '" . tep_db_input($state) . "%' or zone_code like '%" . tep_db_input($state) . "%')");
        if (tep_db_num_rows($zone_query) == 1) {
          $zone = tep_db_fetch_array($zone_query);
          $zone_id = $zone['zone_id'];
        } else {
          $error = true;
          $messageStack->add('checkout_address', ENTRY_STATE_ERROR_SELECT);
        }
       } else {
          $error = true;
          $messageStack->add('checkout_address', ENTRY_STATE_ERROR_SELECT);
        }
       } else {
        if (strlen($state) < ENTRY_STATE_MIN_LENGTH) {
          $error = true;
          $messageStack->add('checkout_address', ENTRY_STATE_ERROR);
        }
  }
}
```

替换为:

```php
if (ACCOUNT_STATE == 'true') {
  // +Country-State Selector
  if ($zone_id == 0) {
  // -Country-State Selector
   if (strlen($state) < ENTRY_STATE_MIN_LENGTH) {
     $error = true;
     $messageStack->add('checkout_address', ENTRY_STATE_ERROR);
   }
  }
}
```

查找(第 187 行):

```php
// if no shipping destination address was selected, use their own address as default 
if (!tep_session_is_registered('billto')) {
  $billto = $customer_default_address_id;
 }
```

替换为:

```php
// if no shipping destination address was selected, use their own address as default 
if (!tep_session_is_registered('sendto')) {
    $sendto = $customer_default_address_id;
  }
  // +Country-State Selector
  if (!isset($country)){$country = DEFAULT_COUNTRY;}
  // -Country-State Selector
```

查找(约第 271 行): 

`<?php require(DIR_WS_INCLUDES . 'form_check.js.php'); ?>`

替换为:

```php
<?php
require(DIR_WS_INCLUDES . 'form_check.js.php');
require(DIR_WS_INCLUDES . 'ajax.js.php');
?>
```

在页面最后查找:

`<?php require(DIR_WS_INCLUDES . 'application_bottom.php'); ?>`

替换为:

```php
<?php require(DIR_WS_INCLUDES . 'application_bottom.php'); ?>
<?php
// +Country-State Selector
}
// -Country-State Selector
?>
```

5、修改文件 catalog/create_account.php 

查找(第 13 行):

`require('includes/application_top.php');`

追加以下代码:

```php
// +Country-State Selector
require(DIR_WS_FUNCTIONS . 'ajax.php');
  if (isset($HTTP_POST_VARS['action']) && $HTTP_POST_VARS['action'] == 'getStates'
&& isset($HTTP_POST_VARS['country'])) {
ajax_get_zones_html(tep_db_prepare_input($HTTP_POST_VARS['country']), true);
} else {
// -Country-State Selector
```

查找(第 130 行):

```php
$zone_id = 0;
$check_query = tep_db_query("select count(*) as total from " . TABLE_ZONES . " where zone_country_id = '" . (int)$country . "'");
$check = tep_db_fetch_array($check_query);
$entry_state_has_zones = ($check['total'] > 0);
if ($entry_state_has_zones == true) {
$zone_query = tep_db_query("select distinct zone_id from " . TABLE_ZONES . " where
zone_country_id = '" . (int)$country . "' and (zone_name like '" . tep_db_input($state) . "%' or zone_code like '%" . tep_db_input($state) . "%')");
     if (tep_db_num_rows($zone_query) == 1) {
        $zone = tep_db_fetch_array($zone_query);
        $zone_id = $zone['zone_id'];
       } else {
        $error = true;
        $messageStack->add('create_account', ENTRY_STATE_ERROR_SELECT);
}
} else {
       } else {
       if (strlen($state) < ENTRY_STATE_MIN_LENGTH) {
        $error = true;
        $messageStack->add('create_account', ENTRY_STATE_ERROR);
       }
} }
```

替换为:

```php
if (ACCOUNT_STATE == 'true') {
  // +Country-State Selector
  if ($zone_id == 0) {
  // -Country-State Selector
   if (strlen($state) < ENTRY_STATE_MIN_LENGTH) {
     $error = true;
     $messageStack->add('create_account', ENTRY_STATE_ERROR);
   }
 }
}
```

查找(第 252 行):

`$breadcrumb->add(NAVBAR_TITLE, tep_href_link(FILENAME_CREATE_ACCOUNT, '', 'SSL'));`

替换为:

```php
// +Country-State Selector
 if (!isset($country)) $country = DEFAULT_COUNTRY;
 // -Country-State Selector
$breadcrumb->add(NAVBAR_TITLE,tep_href_link(FILENAME_CREATE_ACCOUNT, '', 'SSL'));
```

查找(约第 260 行): 

`<?php require(DIR_WS_INCLUDES . 'form_check.js.php'); ?>`

替换为:

```php
<?php
require(DIR_WS_INCLUDES . 'form_check.js.php');
require(DIR_WS_INCLUDES . 'ajax.js.php');
?>
```

查找(第 355 行):

```php
<!-- body_text //-->
   <td width="100%" valign="top"><?php echo tep_draw_form('create_account',
tep_href_link(FILENAME_CREATE_ACCOUNT, '', 'SSL'), 'post', 'onSubmit="return
check_form(create_account);"') . tep_draw_hidden_field('action', 'process'); ?>
     <table border="0" width="100%" cellspacing="0" cellpadding="0" >
```

替换为:

```php
<!-- body_text //-->
      <td width="100%" valign="top"><?php echo tep_draw_form('create_account', tep_href_link(FILENAME_CREATE_ACCOUNT, '', 'SSL'), 'post', 'onSubmit="return check_form(create_account);"') . tep_draw_hidden_field('action', 'process'); ?> <div id="indicator"><?php echo tep_image(DIR_WS_IMAGES . 'indicator.gif'); ?></div>
     <table border="0" width="100%" cellspacing="0" cellpadding="0" >
```

查找(第 418 行):

```php
<?php if ($process == true) {
     if ($entry_state_has_zones == true) {
       $zones_array = array();
$zones_query = tep_db_query("select zone_name from " . TABLE_ZONES . " where zone_country_id = '" . (int)$country . "' order by zone_name");
       while ($zones_values = tep_db_fetch_array($zones_query)) {
        $zones_array[] = array('id' => $zones_values['zone_name'], 'text' =>
$zones_values['zone_name']);
       }
       echo tep_draw_pull_down_menu('state', $zones_array);
     } else {
       echo tep_draw_input_field('state');
     }
   } else {
     echo tep_draw_input_field('state');
} ?>
```

替换为:

```php
<div id="states">
<?php
// +Country-State Selector
echo ajax_get_zones_html($country,'',false);
// -Country-State Selector
?>
</div>
```

查找(第 442 行):

`<td class="main"><?php echo tep_get_country_list('country') . ' ' . (tep_not_null(ENTRY_COUNTRY_TEXT) ? '<span class="inputRequirement">' . ENTRY_COUNTRY_TEXT . '</span>': ''); ?></td>`

替换为:

```php
<?php // +Country-State Selector ?>
<td class="main" colspan="3"><?php echo tep_get_country_list('country',$country,'onChange="getStates(this.value, \'states\');"') . '&nbsp;' . (tep_not_null(ENTRY_COUNTRY_TEXT) ? '<span class="inputRequirement">' . ENTRY_COUNTRY_TEXT . '</span>': ''); ?></td>
<?php // -Country-State Selector ?>
```

在页面的最后查找:

`<?php require(DIR_WS_INCLUDES . 'application_bottom.php'); ?>`

替换为:

```php
<?php require(DIR_WS_INCLUDES . 'application_bottom.php'); ?>
<?php
// +Country-State Selector
}
// -Country-State Selector
?>
```

6、修改文件 catalog/includes/languages/english.php 

添加以下定义

```php
// +Country-State Selector
define ('DEFAULT_COUNTRY', '223');
// -Country-State Selector
```

修改 ENTRY_STATE_TEXT 和 ENTRY_COUNTRY_TEXT 定义:

```php
define('ENTRY_STATE_TEXT', '* (Select country first)');
define('ENTRY_COUNTRY_TEXT', '* (State Dropdown will auto-update when changed)');
```

7、修改文件 catalog/includes/modules/address_book_details.php 

查找(第 28 行):

```php
<tr class="infoBoxContents">
  <td><table border="0" cellspacing="2" cellpadding="2">
```

替换为:

```php
<tr class="infoBoxContents">
       <td><table border="0" cellspacing="2" cellpadding="2">
<div id="indicator"><?php echo tep_image(DIR_WS_IMAGES . 'indicator.gif'); ?></div>
```

查找(第 97 行):
     
```php
<?php
   if ($process == true) {
if ($entry_state_has_zones == true) {
$zones_array = array();
$zones_query = tep_db_query("select zone_name from " . TABLE_ZONES . " where
zone_country_id = '" . (int)$country . "' order by zone_name");
       while ($zones_values = tep_db_fetch_array($zones_query)) {
        $zones_array[] = array('id' => $zones_values['zone_name'], 'text' =>
$zones_values['zone_name']);
}
       echo tep_draw_pull_down_menu('state', $zones_array);
     } else {
       echo tep_draw_input_field('state');
     }
   } else {
     echo tep_draw_input_field('state',
tep_get_zone_name($entry['entry_country_id'], $entry['entry_zone_id'],
$entry['entry_state']));
} ?>
```                                     

替换为:

```php
<div id="states">
<?php
// +Country-State Selector
echo ajax_get_zones_html($entry['entry_country_id'],($entry['entry_zone_id']==0 ? $entry['entry_state'] : $entry['entry_zone_id']),false);
// -Country-State Selector
?>
</div>
```


查找(第 122 行):

```php
<td class="main"><?php echo tep_get_country_list('country', $entry['entry_country_id']) . ' ' . (tep_not_null(ENTRY_COUNTRY_TEXT) ? '<span class="inputRequirement">' . ENTRY_COUNTRY_TEXT . '</span>': ''); ?></td>
```

替换为:

```php
<?php // +Country-State Selector ?>
          <td class="main"><?php echo tep_get_country_list('country',
$entry['entry_country_id'],'onChange="getStates(this.value,\'states\');"') . '&nbsp;' . (tep_not_null(ENTRY_COUNTRY_TEXT) ? '<span class="inputRequirement">' . ENTRY_COUNTRY_TEXT . '</span>': ''); ?></td>
          <?php // -Country-State Selector ?>
```

catalog/includes/modules/checkout_new_address.php

查找(第 15 行): 

`<table border="0" width="100%" cellspacing="0" cellpadding="2">`

替换为:

```php
<table border="0" width="100%" cellspacing="0" cellpadding="2">
<div id="indicator"><?php echo tep_image(DIR_WS_IMAGES . 'indicator.gif'); ?></div>
```

查找(第 80 行):

```php
<?php
   if ($process == true) {
if ($entry_state_has_zones == true) {
$zones_array = array();
$zones_query = tep_db_query("select zone_name from " . TABLE_ZONES . " where
zone_country_id = '" . (int)$country . "' order by zone_name");
       while ($zones_values = tep_db_fetch_array($zones_query)) {
        $zones_array[] = array('id' => $zones_values['zone_name'], 'text' =>
$zones_values['zone_name']);
}
       echo tep_draw_pull_down_menu('state', $zones_array);
     } else {
       echo tep_draw_input_field('state');
     }
   } else {
     echo tep_draw_input_field('state');
} ?>
```

替换为:

```php
<div id="states">
<?php
// +Country-State Selector
echo ajax_get_zones_html($country,'',false);
// -Country-State Selector
?>
</div>
```

查找(第 104 行):

`<td class="main"><?php echo tep_get_country_list('country') . ' ' . (tep_not_null(ENTRY_COUNTRY_TEXT) ? '<span class="inputRequirement">' .ENTRY_COUNTRY_TEXT . '</span>': ''); ?></td>`

替换为:
        
```php
<?php // +Country-State Selector ?>
   <td class="main"><?php echo tep_get_country_list('country',$country,'onChange="getStates(this.value,\'states\');"') . '&nbsp;' . (tep_not_null(ENTRY_COUNTRY_TEXT) ? '<span class="inputRequirement">' . ENTRY_COUNTRY_TEXT . '</span>': ''); ?></td>
<?php // -Country-State Selector ?>
```

8、修改文件 catalog/admin/customers.php 

查找(第 15 行):

`$action = (isset($HTTP_GET_VARS['action']) ? $HTTP_GET_VARS['action'] : '');`

追加以下代码:

```php
// +Country-State Selector
 $refresh = (isset($HTTP_POST_VARS['refresh']) ? $HTTP_POST_VARS['refresh'] :
'false');
 // -Country-State Selector
```

查找(第 43 行):

```php
if (isset($HTTP_POST_VARS['entry_zone_id'])) $entry_zone_id = tep_db_prepare_input($HTTP_POST_VARS['entry_zone_id']);
```

替换为:

```php
// +Country-State Selector
if (isset($HTTP_POST_VARS['entry_zone_id'])) {
  $entry_zone_id = tep_db_prepare_input($HTTP_POST_VARS['entry_zone_id']);
} else {
  $entry_zone_id = 0;
}
if ($refresh != 'true') {
// -Country-State Selector
```

查找以下代码并删除它(第 110 行):

```php
if (ACCOUNT_STATE == 'true') {
  if ($entry_country_error == true) {
    $entry_state_error = true;
        } else {
$zone_id = 0;
$entry_state_error = false;
$check_query = tep_db_query("select count(*) as total from " . TABLE_ZONES
. " where zone_country_id = '" . (int)$entry_country_id . "'");
          $check_value = tep_db_fetch_array($check_query);
          $entry_state_has_zones = ($check_value['total'] > 0);
          if ($entry_state_has_zones == true) {
$zone_query = tep_db_query("select zone_id from " . TABLE_ZONES . " where zone_country_id = '" . (int)$entry_country_id . "' and zone_name = '" . tep_db_input($entry_state) . "'");
            if (tep_db_num_rows($zone_query) == 1) {
             $zone_values = tep_db_fetch_array($zone_query);
             $entry_zone_id = $zone_values['zone_id'];
            } else {
             $error = true;
             $entry_state_error = true;
}
} else {
            if ($entry_state == false) {
             $error = true;
             $entry_state_error = true;
      }
    }
  }
}
```

查找(第 152 行): 

` if ($error == false) {`

替换为:

```php    
// +Country-State Selector
      } // End if (!$refresh)
     if (($error == false) && ($refresh != 'true')) {
// -Country-State Selector
```

查找(第 196 行): 

`$processed = true;`

替换为:

```php
$processed = true;
// +Country-State Selector
} else if ($refresh == 'true') {
  $cInfo = new objectInfo($HTTP_POST_VARS);
}
// -Country-State Selector
```

查找(第 238 行): 

`if ($action == 'edit' || $action == 'update') {`

替换为:

```php     
// +Country-State Selector
if ($refresh == 'true') {
  $entry_state = '';
  $cInfo->entry_state = '';
}
// -Country-State Selector
if ($action == 'edit' || $action == 'update') {
```
查找(第 333 行):

` //--></script>`

替换为:

```php
function refresh_form(form_name) {
  form_name.refresh.value = 'true';
  form_name.submit();
  return true;
}
//--></script>
```

查找(第 369 行):

```php
<tr><?php echo tep_draw_form('customers', FILENAME_CUSTOMERS, tep_get_all_get_params(array('action')) . 'action=update', 'post', 'onSubmit="return check_form();"') . tep_draw_hidden_field('default_address_id', $cInfo->customers_default_address_id); ?>
```

替换为:

```php
<tr><?php echo tep_draw_form('customers', FILENAME_CUSTOMERS, tep_get_all_get_params(array('action')) . 'action=update', 'post', 'onSubmit="return check_form();"') . tep_draw_hidden_field('default_address_id', $cInfo->customers_default_address_id); ?>
<?php
// +Country-State Selector
echo tep_draw_hidden_field('refresh','false');
// -Country-State Selector
   ?>
```

查找(第 581 行):

```php
$entry_state = tep_get_zone_name($cInfo->entry_country_id,
$cInfo->entry_zone_id, $cInfo->entry_state);
   if ($error == true) {
     if ($entry_state_error == true) {
if ($entry_state_has_zones == true) {
$zones_array = array();
$zones_query = tep_db_query("select zone_name from " . TABLE_ZONES . " where
zone_country_id = '" . tep_db_input($cInfo->entry_country_id) . "' order by
zone_name");
        while ($zones_values = tep_db_fetch_array($zones_query)) {
          $zones_array[] = array('id' => $zones_values['zone_name'], 'text' =>
$zones_values['zone_name']);
        }
echo tep_draw_pull_down_menu('entry_state', $zones_array) . ' ' .
ENTRY_STATE_ERROR;
       } else {
        echo tep_draw_input_field('entry_state',
tep_get_zone_name($cInfo->entry_country_id, $cInfo->entry_zone_id,
$cInfo->entry_state)) . ' ' . ENTRY_STATE_ERROR;
}
} else {
       echo $entry_state . tep_draw_hidden_field('entry_zone_id') .
tep_draw_hidden_field('entry_state');
}
} else {
     echo tep_draw_input_field('entry_state',
tep_get_zone_name($cInfo->entry_country_id, $cInfo->entry_zone_id,
$cInfo->entry_state));
}
```

替换为:

```php
// +Country-State Selector
    $entry_state = tep_get_zone_name($cInfo->entry_country_id,
$cInfo->entry_zone_id, $cInfo->entry_state);
    $zones_array = array();
$zones_query = tep_db_query("select zone_name, zone_id from " . TABLE_ZONES . " where zone_country_id = '" . (int)$cInfo->entry_country_id . "' order by zone_name");
    while ($zones_values = tep_db_fetch_array($zones_query)) {
       $zones_array[] = array('id' => $zones_values['zone_id'], 'text' =>
$zones_values['zone_name']);
    }
      if (count($zones_array) > 0) {
       echo tep_draw_pull_down_menu('entry_zone_id', $zones_array,
$cInfo->entry_zone_id);
       echo tep_draw_hidden_field('entry_state', '');
     } else {
       echo tep_draw_input_field('entry_state', $entry_state);
      }
// -Country-State Selector
```

查找(第 610 行):

```php
if ($error == true) {
   if ($entry_country_error == true) {
     echo tep_draw_pull_down_menu('entry_country_id', tep_get_countries(),
$cInfo->entry_country_id) . ' ' . ENTRY_COUNTRY_ERROR;
   } else {
     echo tep_get_country_name($cInfo->entry_country_id) .
tep_draw_hidden_field('entry_country_id');
   }
  } else {
   echo tep_draw_pull_down_menu('entry_country_id', tep_get_countries(),
$cInfo->entry_country_id);
  }
```

替换为:

```php
// +Country-State Selector
echo css_get_country_list('entry_country_id',
$cInfo->entry_country_id,'onChange="return refresh_form(customers);"');
// -Country-State Selector
```

9、修改文件 catalog/admin/includes/functions/html_output.php 

添加下面的 css_get_countries 和 css_get_country_list 函数:

```php     
// +Country-State Selector
// Adapted from functions in catalog/includes/general.php and html_output.php for Country-State Selector
// Returns an array with countries
// TABLES: countries
  function css_get_countries($countries_id = '', $with_iso_codes = false) {
   $countries_array = array();
   if (tep_not_null($countries_id)) {
           if ($with_iso_codes == true) {
       $countries = tep_db_query("select countries_name, countries_iso_code_2,
countries_iso_code_3 from " . TABLE_COUNTRIES . " where countries_id = '" .
(int)$countries_id . "' order by countries_name");
       $countries_values = tep_db_fetch_array($countries);
       $countries_array = array('countries_name' =>
$countries_values['countries_name'],
                           'countries_iso_code_2' =>
$countries_values['countries_iso_code_2'],
                           'countries_iso_code_3' =>
$countries_values['countries_iso_code_3']);
     } else {
       $countries = tep_db_query("select countries_name from " . TABLE_COUNTRIES .
" where countries_id = '" . (int)$countries_id . "'");
       $countries_values = tep_db_fetch_array($countries);
       $countries_array = array('countries_name' =>
$countries_values['countries_name']);
     }
   } else {
     $countries = tep_db_query("select countries_id, countries_name from " .
TABLE_COUNTRIES . " order by countries_name");
     while ($countries_values = tep_db_fetch_array($countries)) {
       $countries_array[] = array('countries_id' =>
$countries_values['countries_id'],
                             'countries_name' =>
$countries_values['countries_name']);
}
}
   return $countries_array;
  }
////

  // Creates a pull-down list of countries
  function css_get_country_list($name, $selected = '', $parameters = '') {
   $countries_array = array();
   $countries = css_get_countries();
   for ($i=0, $n=sizeof($countries); $i<$n; $i++) {
     $countries_array[] = array('id' => $countries[$i]['countries_id'], 'text' => $countries[$i]['countries_name']);
   }
return tep_draw_pull_down_menu($name, $countries_array, $selected, $parameters); 
}
 // -Country-State Selector
```