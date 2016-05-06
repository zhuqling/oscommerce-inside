
## 统计模块的工作原理

### 付款流程

统计模块在付款确认页面里才开始出现它的身影,在确认了运输方式与付款方式后,为了让用户了解清楚各种明细的金额,所以在确认付款时应该输出应统计的所有项目。

checkout_confirmation.php[67]

```php
require(DIR_WS_CLASSES . 'order_total.php');
$order_total_modules = new order_total;
$order_total_modules->process();
```

所有的统计模块由 order_total 类管理,order_total 类有三个方法:

### 初始化方法

与运输模块和付款模块一样,order_total 类的初始化过程也大致相同,所以已安装的付款模 块名称如常量 MODULE_ORDER_TOTAL_INSTALLED 以字符串的形式保存,通过分拆 MODULE_ORDER_TOTAL_INSTALLED,得到各个模块的文件名,然后循环加载每个统计模块 的语言文件与主文件,接着创建每个模块的全局变量,变量名称与模块名称相同。

### process 方法

通过循环调用每个统计模块的 process 方法,得出统计模块的结果。结果并不是以 return 的形
式返回,而是存放在统计模块的$output 成员变量(二维数组)里。所以要得到所有统计模块的数 据还必须再循环一次该模块的$output 成员变量。

统计模块类 includes/classes/order_total.php [44]

```php
for ($i=0, $n=sizeof($GLOBALS[$class]->output); $i<$n; $i++) {
  if (tep_not_null($GLOBALS[$class]->output[$i]['title']) && tep_not_null($GLOBALS[$class]->output[$i]['text'])) {
    $order_total_array[] = array(
      'code' => $GLOBALS[$class]->code,
      'title' => $GLOBALS[$class]->output[$i]['title'],
      'text' => $GLOBALS[$class]->output[$i]['text'],
      'value' => $GLOBALS[$class]->output[$i]['value'],
      'sort_order' => $GLOBALS[$class]->sort_order
    );
  }
}
```

统计模块的$output 成员变量数组是一个关联数组,具体内容如下:

-  $code:统计模块的ID
-  $title:统计模块名称
-  $text:统计模块结果的字面显示(常规情况下此处都显示为金额,所以要先将金额进行币种转换,然后以对应的格式显示)
-  $value:统计模块的结果(实际金额,数据型)
-  $sort_order:统计模块的排序号

得到所有统计模块的结果后,然后将其返回。

### output 方法

output 方法与 process 方法的不同之处在于,process 方法是获取统计模块的所有信息,而 output 方法只是简单的显示所有统计模块的结果。

统计模块类 includes/classes/order_total.php [240]

```php
if (MODULE_ORDER_TOTAL_INSTALLED) {
  echo $order_total_modules->output();
}
```
    
### 订单生成流程

除了显示所有应计的明细费用外,将明细结果写入最终的订单也非常重要。

checkout_process.php[75]

```php
require(DIR_WS_CLASSES . 'order_total.php');
$order_total_modules = new order_total;
$order_totals = $order_total_modules->process();
```


同样先创建 order_total 类的实例,然后要先调用 process 方法,这里使用$order_totals 保存 process 方法的返回值(在显示统计结果时并无必要保存返回值)。

统计模块类 includes/classes/order_total.php [124]

```php
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
```

这段代码是在紧跟着创建新订单后的代码,$insert_id 便是新订单的 ID。通过循环$order_totals (process 方法的返回值),将所以统计模块的结果保存到 order_total 表里。

由于标准的统计模块代码都比较简单,如果读者感兴趣的话,笔者相信你完全可以对其进行分析了解清楚。

所以本人对标准统计模块也就不再介绍了,下面让我们直接进入如何编写统计模块的介绍。
