## EasyPopulate

Easy Populate插件对于osCommerce来说,是管理产品的一个力器。它不仅可以实现下载、 上传产品,还可以按指定格式下载想要的产品,如下载为 CSV 格式,或者下载 Froogle 格式,又 如只下载某个目录里所有在售的产品,再或者只下载产品的属性等。所以可以说 Easy Populate 是 osCommerce 的必备插件之一。

插件地址:http://addons.oscommerce.com/info/500

下面我们介绍的是 Easy Populate 2.76h 版本,该版本对于 osCommerce 系统有一定的要求,其
中包括:

-  要求 osCommerce 2.2,如果是 osCommerce 2.1 及以下版本都不能适用
-  如果对于数据结构进行过修改,那么 Easy Populate 有可能不能正确执行
-  上传产品时的表格要求必须包括 product_model(产品型号)字段,Easy Populate 会将此字段 作为唯一识别 ID 更新产品信息。
-  上传产品的表格要求首行为标题行,Easy Populate 不允许上传没有标题行的文件,否则会报 错
-  Easy Populate 对数据类型不予判断,所以用户填写数据时要自行判断数据类型是否正确,便 如日期类型可以使用“YYYY-MM-DD”的格式
-  如果上传内容里包含“\”反斜线,则必须使用“\\”两个反斜线来代表“\”,建议避免使用 反斜线
 
[安装]

1、复制以下文件到对应的目录:

- /catalog/admin/easypopulate.php 
- /catalog/admin/easypopulate_functions.php 
- /catalog/admin/EPDocumentation/index.html 
- /catalog/admin/EPDocumentation/contents.htm 
- /catalog/admin/EPDocumentation/links.htm

2、新建文件夹“/catalog/temp/”,并设置 temp 目录权限为 777

>  提示:
>  
>  temp 目录用于存放上传及下载产品时的临时文件,如果想变更 temp 目录的位置,可以在文件 “admin/easypopulate.php”里修改以下语句:
>  
> define ('EP_TEMP_DIRECTORY', DIR_FS_CATALOG . 'temp/');

3、编辑文件/catalog/admin/boxes/catalog.php 

查找代码(大概在第 25 行):

```php
'<a href="' . tep_href_link(FILENAME_PRODUCTS_ATTRIBUTES, '', 'NONSSL') . '"
class="menuBoxContentLink">' . BOX_CATALOG_CATEGORIES_PRODUCTS_ATTRIBUTES .
'</a><br>' .
```

在它的下面添加如下代码:

```php
'<a href="' . tep_href_link('easypopulate.php', '', 'NONSSL') . '"
class="menuBoxContentLink">Easy Populate</a><br>' .
```

4、复制文件/catalog/admin/includes/classes/table_block.php(注:此步骤只针对于 osCommerce RC2a 以前版本)

5、执行以下 SQL 语句(注:此步骤只针对于 osCommerce RC2a 以前版本):

`ALTER TABLE `products` ADD INDEX `idx_products_model` ( `products_model` );`

6、调整 PHP 参数,以便实现最佳性能

- max_execution_time = 30 (单位:秒)
  修改这个参数到一个合适的数值,以便于 PHP 能处理上传比较大的文件
- max_input_time = 60 (单位:秒)
- post_max_size = 2M
- upload_max_filesize = 2M
  单个上传文件的大小限制,如果文件太大,建议使用 FTP 将文件上传到 temp 目录,然后再选择通过临时文件进行上传
- memory_limit = 8M 内存限制,建议设置值:32M
- session.gc_maxlifetime = 1440 (单位:秒)
  SESSION 的存活期,因为处理上传文件时时间会比较长,所以应设置得比较合理,以理解会话不会中途过期。通常应大于 max_execution_time 的值

参数介绍

*基本参数*

文件:admin/easypopulate.php

-  define ('EP_SPLIT_MAX_RECORDS', 300);
  下载并分割文件,每个文件所存放产品的条数
-  define ('EP_DELETE_IT', 'Delete');
  如果在列“v_status”里设置 Delete,将删除此产品。EP_DELETE_IT 用于指定使用 Delete字符或者其它识别符
-  define ('EP_INACTIVATE_ZERO_QUANTITIES', false); 当填入数量为 0 时,是否设置产品为 out of stock 缺货状态
-  define ('EP_MODEL_NUMBER_SIZE', 12); 产品型号字段的长度
-  define ('EP_PRICE_WITH_TAX', false); 产品价格是否含税
-  define ('EP_PRECISION', 2); 价格的精确度,默认为精确到小数点后 2 位
-  $ep_separator = "\t"; 
  字段分隔符,默认为“\t”TAB,其它选项包括:“,”逗号、”;”分号、“~”波浪号和“*” 星号
-  define ('EP_REPLACE_QUOTES', false); 是否对单引号与双引号进行转义
-  define ('EP_EXCEL_SAFE_OUTPUT', true);
  是否进行 EXCEL 安全输出,安全输出下字段分隔符为默认为“,”
-  define ('EP_EXCEL_SAFE_OUTPUT_ALT_PARCE', false); 在 EXCEL 安全输出时,是否进行 ALT 解析
-  define ('EP_PRESERVE_TABS_CR_LF', false); 是否保留 TAB 和换行
-  define ('EP_MAX_CATEGORIES', 7); 产品目录的层级数
-  define ('EP_PRODUCTS_WITH_ATTRIBUTES', true); 输出产品时是否包括产品属性
-  define ('EP_PRODUCTS_ATTRIBUTES_STOCK', false); 是否进行属性库存检查

*插件支持*

-  define ('EP_MORE_PICS_6_SUPPORT', false); 是否加载了 MORE PICS 6 插件
-  define ('EP_HTC_SUPPORT', false); 是否加载了 HTC(Header Tags Control)插件
-  define ('EP_SPPC_SUPPORT', false);
  是否加载了 SPPC(Separate Pricing Per Customer)插件
-  define ('EP_EXTRA_FIELDS_SUPPORT', false); 是否加载了 Extra Fields 插件
-  define ('EP_UNKNOWN_ADD_IMAGES_SUPPORT', false); 是否加载了 Unknown Image 插件


*Froogle 参数:*

Easy Populate 能够很好地输出兼容的 Froogle 表格文件,对于针对 Froogle 输出的参数包括:

-  define ('EP_FROOGLE_PRODUCT_INFO_PATH', DIR_WS_CATALOG . HTTP_CATALOG_SERVER . "product_info.php"); 定义产品页面的 URL 地址
-  define ('EP_FROOGLE_IMAGE_PATH', HTTP_CATALOG_SERVER. DIR_WS_CATALOG_IMAGES); 定义产品图片路径
-  define ('EP_FROOGLE_SEF_URLS', false); 是否开启了搜索引擎友好的 URL 方式
-  define ('EP_FROOGLE_CURRENCY', 'USD'); 默认的币种
