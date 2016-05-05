## 运输模块的工作原理

我们仍以 Zone Rates 模块为例来说明一般的运输模块的运行原理。

### 运输模块的方法

*初始化* includes/modules/shipping/zones.php [101]

```php
function zones() {
  $this->code = 'zones';
  $this->title = MODULE_SHIPPING_ZONES_TEXT_TITLE;
  $this->description = MODULE_SHIPPING_ZONES_TEXT_DESCRIPTION;
  $this->sort_order = MODULE_SHIPPING_ZONES_SORT_ORDER;
  $this->icon = '';
  $this->tax_class = MODULE_SHIPPING_ZONES_TAX_CLASS;
  $this->enabled = ((MODULE_SHIPPING_ZONES_STATUS == 'True') ? true : false);
```

运输模块要初始化的基本内容包括:$code、$title、$description、$sort_order、$enabled。   $code:此模块的唯一标识符。

- $title:显示的模块名称
- $description:显示的描述
- $sort_order:序号
- $enabled:是否生效

其它的内容如 icon、tax-class,又如上面的 Zone Rates 扩展参数等,根据不同的模块需要而定。

*安装与删除* includes/modules/shipping/zones.php [172-209]

安装与删除涉及了 4 个方法,分别为 install、remove、key、check,它们的功能分别如下:  

- install:添加模块的所有参数至数据库
- remove:从数据库删除此模块的所有参数
- key:模块的所有参数键值
- check:判断模块是否已经安装

*运费计算* includes/modules/shipping/zones.php [115]

```php
function quote($method = '')
```

quote 的作用有两个:

- 获取可用的运输方式,如果一个运输方式有多个子运输方式,也一并输出。
- 在选定此运输方式后,获取最后的运费。

quote 方法只有唯一的 method 参数,此参数的作用是在选定此运输方式时,如果此模块拥有 多种不同子运输方式,可以通过 method 传递此子运输方式。

这里我们拿美国邮政 USPS(United States Postal Service)提供的服务来举例:

> USPS 提供有:
> - Global Express Guaranteed
> - Global Express Guaranteed Non-Document Rectangular
> - Global Express Guaranteed Non-Document Non-Rectangular
> - Express Mail International (EMS)
> - Express Mail International (EMS) Flat Rate Envelope
> - Priority Mail International
> - Priority Mail International Flat Rate Envelope
> - Priority Mail International Flat Rate Box
> - First-Class Mail International

不同的服务运费也不一样,所以如果我们使用一个 USPS 运输模块来包含其提供的所有服务时,则必须提供子运输方式给用户选择,最后通过子运输方式来计算最终的运费。

*返回值格式* includes/modules/shipping/zones.php [155]

```php
$this->quotes = array('id' => $this->code,
                       'module' => MODULE_SHIPPING_ZONES_TEXT_TITLE,
                       'methods' => array(array('id' => $this->code,
                                            'title' => $shipping_method,
                                            'cost' => $shipping_cost)));
```

运输模块返回的是一个数组,数组结构如上,id 是此模块的唯一识别 ID,module 是模块的 名称,methods 数组是一个二维数组,它指定了可用的运输方式,如果有多个子运输方式,应将 所有子运输方式放在 methods 数组里一起返回。

### 运输模块的调用流程

*选择运输方式* checkout_shipping.php[64-95]

```php
  // load all enabled shipping modules
  require(DIR_WS_CLASSES . 'shipping.php');
  $shipping_modules = new shipping;
  if ( defined('MODULE_ORDER_TOTAL_SHIPPING_FREE_SHIPPING') &&
(MODULE_ORDER_TOTAL_SHIPPING_FREE_SHIPPING == 'true') ) {
   $pass = false;
   switch (MODULE_ORDER_TOTAL_SHIPPING_DESTINATION) {
    case 'national':
      if ($order->delivery['country_id'] == STORE_COUNTRY) {
        $pass = true;
      }
      break;
    case 'international':
      if ($order->delivery['country_id'] != STORE_COUNTRY) {
        $pass = true;
      }
      break;
    case 'both':
      $pass = true;
      break;
  }
  $free_shipping = false;
  if ( ($pass == true) && ($order->info['total'] >=
MODULE_ORDER_TOTAL_SHIPPING_FREE_SHIPPING_OVER) ) {
   $free_shipping = true;
   include(DIR_WS_LANGUAGES . $language .
'/modules/order_total/ot_shipping.php');
  }
} else {
  $free_shipping = false;
}
```

首先加载 shipping 类,并新建一个 shipping 类的对象$shipping_modules。shipping 类的初始化如下:

运输方式类 includes/classes/shipping.php[16-42]

```php
 // class constructor
 function shipping($module = '') {
   global $language, $PHP_SELF;
   if (defined('MODULE_SHIPPING_INSTALLED') &&
tep_not_null(MODULE_SHIPPING_INSTALLED)) {
     $this->modules = explode(';', MODULE_SHIPPING_INSTALLED);
     $include_modules = array();
if ( (tep_not_null($module)) && (in_array(substr($module['id'], 0, strpos($module['id'], '_')) . '.' . substr($PHP_SELF, (strrpos($PHP_SELF, '.')+1)), $this->modules)) ) {
$include_modules[] = array('class' => substr($module['id'], 0, strpos($module['id'], '_')), 'file' => substr($module['id'], 0, strpos($module['id'], '_')) . '.' . substr($PHP_SELF, (strrpos($PHP_SELF, '.')+1)));
    } else {
      reset($this->modules);
      while (list(, $value) = each($this->modules)) {
        $class = substr($value, 0, strrpos($value, '.'));
        $include_modules[] = array('class' => $class, 'file' => $value);
      }
    }
    for ($i=0, $n=sizeof($include_modules); $i<$n; $i++) {
      include(DIR_WS_LANGUAGES . $language . '/modules/shipping/' .
$include_modules[$i]['file']);
      include(DIR_WS_MODULES . 'shipping/' . $include_modules[$i]['file']);

      $GLOBALS[$include_modules[$i]['class']] = new
$include_modules[$i]['class'];
    }
  }
}
```

在这里请注意第 20 行的 MODULE_SHIPPING_INSTALLED,这个常量是 OSC 系统维护的一个“已安装运输方式”的表。 它的修改与更新由后台的“admin/modules.php” 文件管理,具体代码位于 186-195 行。

MODULE_SHIPPING_INSTALLED 的值将 所有已安装的运输模块文件名用“;”分隔开来, 所以在 21 行首先将其分解为数组。

接下来的代码,将所有运输模块文件名,组 织成由 class 和 file 为成员的数组 $include_modules。

然后再遍历$include_modules,加载每个运输 模块的语言文件与主文件,并一一新建各自的类 对象。对象的名称与类名一致(其实为文件名去 除后缀名的那部分)。并且这些类对象都为全局变 量。

接着继续来看 checkout_shipping.php 文件, 在 68-95 行的代码讨论了免运费的问题。只有在 激活了 Free Shipping 时,才会进行此计算,Free Shipping 的设置在后台的 Modules->Order Total 的 Shipping 选项里,以图所示:

其中的参数如下:

-  Allow Free Shipping:指定是否开启免运费功能。
-  Free Shipping For Orders Over:选项设置总金额大于多少才可以免运费。
-  Provide free Shipping For Orders Made:是否限定免运费只用于国内或国际,或者不限定。
 
其中 68 行是对“Allow Free Shipping”的检验,71-85 行的代码即是对“Provide free Shipping For Orders Made”的检验,而 88 行的代码是对“Free Shipping For Orders Over”订单额是否达到指定 金额选项的检验。

为了适应一般的条理性,我们按照先显示可用运输方式,然后选择指定运输方式的顺序来看运输
模块的执行过程。

运输模块的列表获取 checkout_shipping.php[140]

```php
// get all available shipping quotes
$quotes = $shipping_modules->quote();
```
此处调用了 shipping 类的 quote 方法。

运输模块类 includes/classes/shipping.php[44]

```php
function quote($method = '', $module = '') {
    global $total_weight, $shipping_weight, $shipping_quoted, $shipping_num_boxes;

    $quotes_array = array();

    if (is_array($this->modules)) {
      $shipping_quoted = '';
      $shipping_num_boxes = 1;
      $shipping_weight = $total_weight;

      if (SHIPPING_BOX_WEIGHT >= $shipping_weight*SHIPPING_BOX_PADDING/100) {
        $shipping_weight = $shipping_weight+SHIPPING_BOX_WEIGHT;
      } else {
        $shipping_weight = $shipping_weight + ($shipping_weight*SHIPPING_BOX_PADDING/100);
       }

      if ($shipping_weight > SHIPPING_MAX_WEIGHT) { // Split into many boxes
        $shipping_num_boxes = ceil($shipping_weight/SHIPPING_MAX_WEIGHT);
        $shipping_weight = $shipping_weight/$shipping_num_boxes;
      }

      $include_quotes = array();

      reset($this->modules);
      while (list(, $value) = each($this->modules)) {
        $class = substr($value, 0, strrpos($value, '.'));
        if (tep_not_null($module)) {
          if ( ($module == $class) && ($GLOBALS[$class]->enabled) ) {
            $include_quotes[] = $class;
          }
        } elseif ($GLOBALS[$class]->enabled) {
          $include_quotes[] = $class;
        }
      }
      $size = sizeof($include_quotes);
      for ($i=0; $i<$size; $i++) {
        $quotes = $GLOBALS[$include_quotes[$i]]->quote($method);
        if (is_array($quotes)) $quotes_array[] = $quotes;
      }
    }

    return $quotes_array;
}
```

我在前面已经讲过,每个运输模块 quote 方法具有两个作用,它不仅可以列表出所有可用的 运输方式,而且可以指定对应的运输方式及子运输方式来计算运费。

同样 shipping 类的 quote 也 具有同样的作用,或者换句话说,shipping 类的 quote 方法即是通过所有运输模块的 quote 方法来 达到这两个功能的(事实上的确如此,上面的 quote 方法第 67-83 行的代码已经说明了这点)。

在 shipping 类的初始化时,我们已经将所有已安装的运输模块文件名分解成数组,存放在成 员变量$modules 里。

所以在 49 行,首先判断$this->modules 是否为数组来确定有可用的运输模块。

54-63 的代码实现了包装的重量计算,以及包装的分拆。相关的三个参数在后台的 Configuration->Shipping/Pachaging 里。

详细的参数如下:

-  Enter the Maximum Package Weight you will ship:单个包裹的最大容量,对应于常量
SHIPPING_MAX_WEIGHT
-  PackageTareweight:单个包装箱的重量,对应于常量SHIPPING_BOX_WEIGHT
-  Largerpackages-percentageincrease:超出标准尺寸的包装重量计算倍数(包装计算倍数),
对应于常量 SHIPPING_BOX_PADDING
-  Largerpackages-percentageincrease的单位为“%”,如填入10,则为10%

> 提示: OSC 的包装重量及分拆原理是
> 1. 如果总重量*包装计算倍数<=单个包装箱的重量,则按单个包装箱的重量计算包装重量;否则,包装重量为总重量*包装计算倍数的值。
> 2. 如果总重量超过单个包裹的最大容量,则按最大容量分拆成若干包裹。否则,使用单个包裹。

第 67-77 行的代码找出所有可安装的运输模块,并确认其是否生效。

第 70-76 行显示了 quote 方法两个不同作用的区别所在,当指定了$module 参数时,只将指定 的$module 包含进最终执行 quote 方法的数组$include_quotes 里。

剩余的代码是运行所有可用的运输模块的 quote 方法,并将返回结果保存至数组,并将此数 组作为返回值。

*运输模块列表的显示*

运输模块列表的显示从行 264 开始,到 361 行结束。其中显示了如何以单选框的方式输出运输模块选项。代码比较简单,请读者自行分析。 

值得提及的一点是,单选框的名称为“shipping”,值为运输模块的 code +“_”+ 子运输方式
的 code,如无子运输方式,则为运输模块的 code +“_”+ 运输模块的 code。如:Zone Rates 选 项的单选框的值为“zones_zones”。

运输模块的选取 checkout_shipping.php[104]

```php
if (!tep_session_is_registered('shipping')) tep_session_register('shipping');
if ( (tep_count_shipping_modules() > 0) || ($free_shipping == true) ) {
  if ( (isset($HTTP_POST_VARS['shipping'])) &&
(strpos($HTTP_POST_VARS['shipping'], '_')) ) {
    $shipping = $HTTP_POST_VARS['shipping'];
    list($module, $method) = explode('_', $shipping);
    if ( is_object($$module) || ($shipping == 'free_free') ) {
      if ($shipping == 'free_free') {
        $quote[0]['methods'][0]['title'] = FREE_SHIPPING_TITLE;
        $quote[0]['methods'][0]['cost'] = '0';
      } else {
        $quote = $shipping_modules->quote($method, $module);
      }
      if (isset($quote['error'])) {
        tep_session_unregister('shipping');
      } else {
        if ( (isset($quote[0]['methods'][0]['title'])) &&
(isset($quote[0]['methods'][0]['cost'])) ) {
          $shipping = array('id' => $shipping,
                         'title' => (($free_shipping == true) ?
$quote[0]['methods'][0]['title'] : $quote[0]['module'] . ' (' .
$quote[0]['methods'][0]['title'] . ')'),
                         'cost' => $quote[0]['methods'][0]['cost']);
          tep_redirect(tep_href_link(FILENAME_CHECKOUT_PAYMENT, '', 'SSL'));
        }
      }
    } else {
      tep_session_unregister('shipping');
    }
  }
} else {
   $shipping = false;
   tep_redirect(tep_href_link(FILENAME_CHECKOUT_PAYMENT, '', 'SSL'));
}
```

osCommerce 使用 session 的方法传递 shipping 的选择。所以在最开始就使用tep_session_register 注册 shipping。而在选择无效时,则使用 tep_session_unregister 注销 shipping。

第 106 行的 tep_count_shipping_modules 函数,返回已安装并已生效的运输模块的数量。
针对指定的运输模块的处理从第 107 行开始,因为我们在上面提及到 shipping 单选框的值的 格式,所以 110 行,将此值分解为$module 和$method。

在判断了运输模块$module 的正确性后,在第 116 行,通过 shipping 类($shipping_modules 为 其对象)的 quote 方法,间接执行选取的运输模块的 quote 方法。

如果 quote 方法返回的值是正确的,则在第 122 行,构造正确的$shipping 值用于 session 的传 递。
第 126 行显示选择了正确运输模块后,通过 tep_redirect 自动转入付款方式的选择页面。
