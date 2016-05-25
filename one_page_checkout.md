## 单页面结算

在上面付款流程的介绍中,标准的付款流程是分成三个步骤:首先选择运输方式、接首选择付款方式、最后是订单确认然后生成订单。

如果能将运输方式的选择与付款方式的选择合二为一并为一个页面,并可以实现快速切换发货地址和账单地址,实时更新最终金额,那么将使流程更多顺畅,也使付款的时间缩短,从而提高用户的购物体验。这样无疑对电子商务网站是大有裨益的。

为了整合付款流程。我们需要使用一个osCommerce的插件,插件的名字叫:One Page Checkout ( http://addons.oscommerce.com/info/6646 ),同时有开发人员针对One Page Checkout插件开发出了 一套自动安装的系统Autoinstaller for One Page Checkout,针对最新版本的One Page Checkout,总 会有热心的开发人员及时更新自动安装系统到最新版本。所以我们使用Autoinstaller for One Page Checkout进行安装说明。

[安装]

Autoinstaller for One Page Checkout 要求 osCommerce 系统是全新的系统,所以建议安装此插件前备份所有文件。另外 Autoinstaller for One Page Checkout 会自动备份所修改的文件,即使如果没有及时备份文件,安装后才发现出现错误,也可以通过还原安装文件夹的 backup 子目录里的文件,实现系统恢复。

将解压后的 oc_autoinstaller 目录,放入 catalog 目录下,然后访问 URL: http://www.my-site.com/oc_autoinstaller (假设www.my-site.com是你的网址)点击“install”按钮,当提示安装完成后即告成功。
