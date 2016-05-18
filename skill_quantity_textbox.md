## 增加产品数量文本框

在将产品添加到购物车时,osCommerce 默认没有提供产品数量的选项,本节介绍如何添加一个 产品数量文本框,以供用户输入需购买的产品数量。

1、文件 product_info.php,将以下代码复制到“Add to Cart”按钮附近(form 标签以内)

`< input type="text" name="quantity" value="1" maxlength="2" size="2">`

2、修改文件 includes/application_top.php 

查找:

` $HTTP_POST_VARS['id']))+1`

将其替换为以下代码:

`$HTTP_POST_VARS['id']))+$HTTP_POST_VARS[‘quantity’]`
