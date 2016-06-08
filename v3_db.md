
## 数据库

默认的 osCommerce RC2.2 拥有表的数量有 47 个,而 osCommerce V3 的表数量为 57 个。
先来介绍一下那些新增加的表

| 表 | 说明 |
|---|---|
|  administrators_access | 管理员权限控制,用于对管理员进行角色控制,限制其的访问内容 |
| administrators_log | 管理员日志,用于记录管理员每次的操作 |
| credit_cards | 信用卡卡号格式 |
| languages_definitions | 语言定义 |
| newsletters_log | 邮件列表的日志 |
| orders_products_variants | 用于替代表 orders_products_attributes |
| products_variants | 用 于 替 代 表 products_attributes , 而 products_attributes 表被改用作产品扩展属性 |
| products_variants_groups | 用于替代表 products_options |
| products_variants_values | 用于替代表 products_options_values |
| orders_transactions_history | 订单事务历史记录 |
| orders_transactions_status | 订单事务状态 |
| products_images | 管理产品图片 |
| products_images_groups | 产品图片组 |
| shipping_availability | 运输方式是否有效 |
| shopping_carts | 用于替代表 customers_basket |
| shopping_carts_custom_variants_values | 用于替代表 customers_basket_attributes |
| templates | 模板 |
| templates_boxes | 模板的功能模块与内容框 |
| templates_boxes_to_pages | 功能模块、内容框位置等配置 |
| weight_classes | 重量单位类型 |
| weight_classes_rules | 重量单位的换算 |

被精减的表

| 表 | 说明 |
|---|---|
| address_format |  |
| products_attributes_download |  |
| products_attributes | 虽然此表还存在,但用途已经改变,所以字段结构也完全不同 |
| products_options_values_to_products_options | 此表的目的只在于方便后台管理,但与原始数据存在冗余,所以被删 |
| reviews_description | 因为一个评价只会对应一种语言,所以将 reviews_description 合并入 reviews 表 |

以下是几个新增的重要表之间的关系图:

### 重量单位

weight_classes 表是重量单位表,weight_classes_rules 表是重量单位换算表,产品表 products 通过字段 products_weight_class 关联到 weight_classes 的字段 weight_class_id,实现重量单位的关 联。

weight_classes_rules 表的字段 weight_class_from_id 和 weight_class_to_id 同时关联到表 weight_classes,字段 weight_class_rule 是重量单位换算比率值,如当 weight_class_from_id 为克(g), weight_class_to_id 为千克时,weight_class_rule 的值为 0.001(千分之一)
 
### 模板

表 templates 保存了所有可用的模板,表 templates_boxes 保存所有可用的功能模块和内容框, 字段 modules_group 表明了该内容所属的类型

其中的类型有:boxes(内容框)、content(功能模 块)、product_attributes(产品扩展属性)、payment(付款模块)、shipping(运输模块)、order_total (统计模块)

templates_boxes_to_pages 表定义了所有的功能模块与内容框在模块里的位置,字段 content_page 表明此模块将在哪些页面出现,字段 boxes_group 代表位置组,表示此模块将出现在页面哪个位置,位置组由模块最终控制其显示的位置,所以位置组名称会因使用不同的模板而导 致最终显示位置可能有所不同

### 产品与属性选项

表 products_variants_groups 为 变 量 组 , 相 当 于 osCommerce RC2.2 的 选 项 表 , 表 products_variants_values 为变量表,相当于 RC2.2 的选项值表,表products_variants_values 通过字 段 products_variants_groups_id 定义变量的所属组。

表 products_variants 相当于 RC2.2 的 products_attributes 表,即产品变量表(属性表),字段products_variants_values_id 关联到 products_variants_values 表,同时通过 products_variants_values 表自身的关联,从而可以得到产品的变量组及变量值,字段 default_combo 表示此变量值为默认选项
 

### 产品与图片

表 images_groups 为图片组,图片组的概念是:每一个产品的每一张图片都将会生成图片组所规定的每一种规格,字段 code 不仅表示图片组的代码,而且此 code 将会被做为目录,实现图 片的存取。

字段 size_width 和size_height 为该图片组的图片宽度及高度。

表 products_images 的字段 default_flag 表示该图片是否为默认图片

### 管理员

表 administrators_log 保存所有管理员的针对数据的所有修改记录,字段 module 指定修改的模块,module_action 表示操作的内容(如“update”表示更新数据),field_key 表示修改的字段, old_value 表示修改前的值,new_value 表示修改后的值,administrators_id 代表操作的管理员。

表 administrators_access 负责控制管理员的操作权限

### 购物车

表 shopping_carts 为购物车表,customers_id 表示客户的 ID。

表 shopping_carts_custom_variants_values 保存了所购产品的属性,字段 shopping_carts_item_id 关联到 shopping_carts 表的字段 item_id,字段 products_variants_values_id 关联到表 products_variants_values 的 id,字段 products_variants_values_text 保存了属性值的文本(因为 osCommerce V3 增加文本选项的属性,所以增加了此字段保存属性的文本)
