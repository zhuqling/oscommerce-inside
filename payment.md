## 了解付款模块



后台 Modules->Payment,显示了所有付款模块,以下我们以 Paypal Website Payments Standard模块为例子对付款模块的原理进行说明。

> Paypal(贝宝)是全球最大的在线支付系统,支持不同币种的付款,允许 Paypal 余额或者借 记卡、信用卡付款,它还提供其它丰富的服务,Paypal 对于付款方与收款方有着严格的规定,以确保交易安全。其在线支付的安全性上是国际领先的,所以一直以来,都是在线支付的首选。

Paypal Website Payments Standard 是 Paypal 提供的诸多服务之一,是用于普通的Paypal 收款, 可以满足大部分的收款需要。

Paypal 提供了以下的收款功能:

-  PayPal Website Payments Standard:Paypal普通付款,用于大部分的收款;
-  PayPal Website Payments Pro Direct Payments:Paypal 直接付款专业版,
  除了提供收款功能外,还设专门的 API 进行订单查询、账号余额信息查询的功能;
-  PayPal Express Checkout:Paypal快速付款,
  与普通付款不同之处在于,普通付款的运输方式等信息由网站自己实现,所以要求用户必须是网站的注册用户才可付款,而快速付款功能,允许用户未注册就可以付款;
-  PayPal Website Payments Pro (UK) Direct Payments:针对英国(UK)的直接付款; 
-  PayPal Website Payments Pro (UK) Express Checkout:针对英国的快速付款。

Paypal Website Payments Standard 模块包括有三个文件:除了模块主文件、语言文件,还有一 个用于处理 Paypal IPN 的文件,它们的安装位置以下:

-  模块主文件:includes/modules/payment/paypal_standard.php
-  语言文件:includes/languages/LANGUAGE/modules/payment/paypal_standard.php
-  PaypalIPN处理文件:ext/modules/payment/paypal/standard_ipn.php

要安装付款模块,先将三个文件复制到正确的路径。进行后台 Modules/Payment,先选择要 安装的模块,点击“install”按钮。此模块在安装完成后,默认是未激活的,所以一定要先配置好 它的参数才可以使用。
     
### Paypal 网站付款标准版的参数

-  EnablePayPalWebsitePaymentsStandard:是否激活PaypalWebsitePaymentsStandard模块。
-  E-MailAddress:用于收款的Paypal账号。
-  Payment Zone:是否限制只有指定的税区才使用此模块收款。
-  SetPreparingOrderStatus:当Paypal返回Preparing状态时,订单应改成哪个状态。
-  Set PayPal Acknowledged Order Status:成功付款后,订单应改成什么状态。
-  GatewayServer:Paypal的网关选择,Live:真正付款,Sandbox:测试付款。

> 提示:
> Paypal 除了实际的收付款服务外,还专门提供了测试收付款的 Sandbox 测试环境。用户只要注册 Sandbox 账号,就可以使用此账号进行付款的测试,测试付款的流程和效果与实际付款是一样的, 这样便大大方便了程序的调试。
> Paypal Sandbox网址:https://developer.paypal.com/devscr?cmd=_home

-  TransactionMethod:指明交易是否是稍后在https://www.paypal.com上通过Paypal授权与捕 获进行捕获的授权。选择 Authorization 则为是,选择 Sale 则为否。
-  PageStyle:付款的页面样式,此页面样式在Paypal账户的设置里可由用户自定义。
-  DebugE-MailAddress:接收测试信息的Email地址。
-  Sort order of display:付款模块的排序号(功能同运输模块的 Sort Order)
-  EnableEncryptedWebPayments:是否使用加密的EWP传输。以下的5个参数只有当启用加密的传输时才会使用到。
-  Your Private Key:私钥文件
-  Your Public Certificate:公钥文件
-  YourPayPalPublicCertificateID:Paypal账号关联的公共授权ID。
-  Working Directory:用于生成临时文件的文件夹
-  OpenSSL Location:OpenSSL命令所在的路径,加密文件需要使用OpenSSL命令,所以需要指定 OpenSSL 的路径。

在普通收款代码里,所有的需要传输至 Paypal 的参数,是以 Post 方式传送的,所以所有的参 数都是放在 form 的 hidden 元素里,因此这些参数是可以被发现的,如果有人出于恶意的目的, 利用此点,他可能会通过修改 hidden 的内容,使得最后的付款金额产生损失。

Paypal 提供了更安全的传输方式 Encrypted Web Payments(EWP),通过将所有参数进行加密, 并保存为文件,然后传输的只是此文件名。通过这个方法,其它人就不可能再修改的了传递的参 数。

关于加密的Paypal传输,请浏览http://www.paypal.com了解更多详情。
     
### Paypal 网站付款标准版的显示

选择使用Paypal付款后,点击“continue”按钮,在订单确认页面点击“Confirm Order”按钮, 系统会自动连接至 Paypal 网站进行付款。
