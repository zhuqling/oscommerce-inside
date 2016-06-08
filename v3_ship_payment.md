## 运输与付款

### 运输模块

osCommerce V3的运输模块与osCommerce RC2.2工作原理上没什么分别,不过对于组织结构方 面,osCommerce V3 还是有所改进。

osCommerce V3 的运输模块由前后与后台两部分组成,也就是说一个运输模块必须有前后与后台 两套代码。

前后运输模块类继承自 osC_Shipping 类,保存在“includes/modules/shipping/”,它拥有初始 化方法、initialize 方法和 quote 方法。

其中 initialize 方法用于进行必要的数据检查,如区域限制检 查等。quote 方法与 osCommerce RC2.2 一样,用于输出此运输模块 ID、名称以及运费。

后台部分的运输模块类继承于 osC_Shipping_Admin 类,它保存在 “admin/includes/modules/shipping/”,主要负责运输模块的安装、卸载。它的方法包括初始化方法、 install 方法、getKeys 方法、isInstalled 方法,其中 install 方法用于安装运输模块,getkeys 方法用 于在卸载运输模块时罗列出所有参数 KEY,isInstalled 方法返回是否已经安装了此运输模块
  
### 付款模块

同样,在 osCommerce V3 里付款模块也被分为前台与后台两部分代码。

前后的付款模块类,是 osC_Payment 类的一个子类,文件保存位置是 “includes/modules/payment/”,它的方法有初始化方法、selection 方法、process 方法、confirmation 方法、process_button 方法、callback 方法。

selection 方法用于在选择付款模块时输出付款模块的 ID、名称等信息,process 方法对于离线付款方式,会进行订单的生成;而对于在线支付模块,则 会进行在线支付前的准备工作。

confirmation、process_button 和 callback 方法只针对在线支付方式 有效。当为在线支付时,会选择在 confirmation 方法里生成订单,在 process_button 方法里生成在 线支付所需提交的代码,而 callback 方法是 osCommerce V3 所新增的一个方法,用于处理在线支 付的即时反馈,如Paypal IPN信息。

在osCommerce RC2.2里,处理在线支付的即时反馈需要另 外的文件进行处理,通常此文件存放于“ext/modules/payment/PAYMENT_CODE/PAYMENT.php”, 在 osCommerce V3 中能够使得即时反馈处理并入付款模块里,可以说是一个进步。

付款模块的后台部分,是 osC_Payment_Admin 类的一个子类,文件存放位置是 “admin/includes/modules/payment/”,它拥有的方法与运输模块一致。有初始化方法、install 安装 方法、getKeys 枚举付款模块参数方法以及 isInstalled 判断是否已经安装的方法。


> 提示:
> 
> 要将即时反馈处理在付款模块的 callback 方法里面实现,还需要在传递支付代码时,指定即时反 馈的调用地址为:checkout.php?callback&module=PAYMENT,其中 PAYMENT 为此付款模块的 CODE,这样当支付网关有响应时,便会自动访问付款模块 PAYMENT 类的 callback 方法。

