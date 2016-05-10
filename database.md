
## 数据库关系图

### 产品

产品内容由 products 和 products_description 两个表保存,products 保存产品的数量、型号、 价格、重量等信息,而 products_description 则保存产品名称、描述、浏览次数等,products_description 表通过字段 language_id 关联 languages 表实现不同语言情况下显示不同的产品描述。

产品属于哪个分类是由 products_to_categories 表控制,其中的 products_id 字段关联 products 表,categories_id 字段关联 categories 表(产品分类表)

产品的税率由字段 tax_class_id 关联表 tax_class 来实现,制造商信息由字段 manufacturers_id 关联表 manufacturers 来实现。

产品特价设置由表 specials 实现,specials 表的 products_id 指定了特价产品的产品 ID 产品属性由表 products_attributes 进行管理,具体的实现请看下面的介绍。

### 产品属性

products_option 表保存所有的产品属性,products_options_values 表保存所有的产品属性值, 产品属性与属性值的一一对应关系由 products_options_values_to_products_options 表管理。
 
products_options_values_to_products_options 表的功能是管理产品属性,而真正的产品属性设 置是由 products_attributes 表来管理的,其中的 products_id 字段指定了产品 ID,字段 options_id 指定产品的属性 ID,字段 options_values_id 指定了此属性的值 ID,字段 options_values_price 和 price_prefix 分别指定了此属性值所变更的价格和价格前缀

>  提示:
>  
> 价格前缀的值可以是“+”或者“-”,也就是说对应于此属性值,产品的价格将加上 options_values_price,或者减去 options_values_price

### 产品分类

产品分类由表 categories 和 categories_description 管理,categories 表保存产品分类的基本资料, 包括产品分类图片、上级分类 ID、排序号等,categories_description 表保存对应不同语言的产品 分类名称
产品分类与产品的从属关系由 products_to_categories 表管理

### 制造商

manufacturers 表保存制造商的基本资料,manufacturers_info 表保存不同语言情况下的制造商 信息,如制造商的网址、访问次数等

单个产品对应于哪个制造商由 products 表的 manuafacturers_id 字段关联 manufacturers 表来进行
 
### 客户

客户资料分别由 customers 和 customers_info 表保存,customers 表保存客户的基础资料, customers_info 则保存附加资料。

与客户关联的购物车信息由 customers_basket 表保存,主要保存产品 ID、购买数量、最终价 格等信息,而购物车内所有产品的属性则由 customers_basket_attributes 表保存。customers_basket 表和 customers_basket_attributes 表同时保存有产品 ID,所以程序在获得购物车内产品的属性时需 要使用产品 ID 进行查找。

### 订单

订单相关的表包括:orders、orders_products、orders_products_attributes 三个表
其中 orders 表负责管理订单基础资料,包括客户资料(以“customers_”为前缀的字段)、收 件人资料(以“delivery_”为前缀的字段)、付款方式(payment_method)、发货方式(shipping_method)、运费(shipping_cost)、订单状态(orders_status)、货币(currency 和 currency_value)以及购买日期(date_purchased)等

orders_products 表负责管理订单所包含产品的资料,包括产品 ID、产品名称、最终价格、购 买数量、税率等

orders_products_attributes 表由负责管理所购买产品的参数

> 提示:
> 
> 值得注意的是,orders_products_attributes 表所保存的是产品参数的最终结果,而不是属性 ID 和属 性值 ID,也就是说它并不与 products_options 表和 products_options_values 关联。其中字段 products_options 保存属性名称,字段 products_options_values 保存参数值

### 地址簿

osCommerce 的客户地址信息是独立于客户资料的,所以每个客户都可以拥有多个地址信息, 因为客户的地址信息除了保存他的住址外,还包括两种重要的内容:发货地址和账单地址,所以 独立的地址信息就可以满足用户需要将货物或者将账单发送发送到不同地址的需求。

adress_book 表负责保存地址信息,customers_id 和 address_book_id 字段作为表的主键, customers_id 字段则关联到 customers 表,以便通过客户查找到他的地址,字段 entry_countries_id
 
关联到 countries 表,取得地址所处国家,不同国家显示地址的格式会有所不同,所以程序可以通 过 countries 表的 address_format_id 关联到 address_format 取得地址格式

字段 entry_zones_id 关联到 zones 表,取得省份/区的信息

### 留言/评价

reviews 表和 reviews_description 表负责管理用户的留言评价,reviews 表保存留言的基本资料, 包括产品 ID、用户 ID、用户评分(reviews_rating)等,reviews_description 表保存具体的留言内 容,其中 languages_id 关联 languages 表,用于实现不同语言的留言
  
### 选项

系统选项由表 configuration_group 和表 configuration 管理,configuration_group 是选项组, configuration 是选项。configuration 表通过字段 configuration_group_id 关联到 configuration_group 表

configuration_group 表的字段 visible 允许隐藏选项组,osCommerce 使用这个特性的唯一地方 在于管理模块选项(包括了付款、发货、统计模块)。

configuration 表除了字段 configuration_title(选项名)、configuration_key(选项对应的常量名)、 configuration_value(选项值)等外,还有两个重要的字段,即 use_function 和 set_function。

它们 分别是读取选项值的函数(取得选项值)和设置选项值的函数(修改选项时)。以用户所处省份/ 区 Zone 的选项为例(configuration_id 为 6),它的 use_function 值是“tep_cfg_get_zone_name”, set_function 值 是 “ tep_cfg_pull_down_zone_list( ”, 这 两 个 函 数 都 定 义 在 “admin/includes/functions/general.php”文件中

tep_cfg_get_zone_name 函数的完整定义是“function tep_cfg_get_zone_name($zone_id)”

tep_cfg_pull_down_zone_list 函数的定义是“function tep_cfg_pull_down_zone_list($zone_id)” 

在调用 use_function 所设定的函数时,系统会将选项值作为函数的唯一参数值入,而在调用set_function 时,最终完整的调用函数是:set_function . '"' . htmlspecialchars($configuration_value) . '"),其中$configuration_value 为选项值,所以 set_function 字段的值必须包含单个左括号,而不是 双括号或者无括号。
 
### 广告位

广告位由 banners 表管理,banners_history 表负责管理 banner 的统计

banners 表的 expires_impressions 字段和 expires_date 用于管理 banner 的过期设置, expires_impressions 值表示浏览量超过多少将停止 banner 的展示,expires_date 值指定具体的 banner 过期时间

### 语言

languages 表管理 osCommerce 多语言功能,其中 name 为语言名称、code 为语言代码、image 为语言标识图标、directory 为语言文件的保存目录

### 币种

多币种功能由 currencies 表负责管理,其中 title 为货币名称、code 为货币代码、symbol_left 为货币左前缀、symbol_right 为货币右后缀、decimal_point 为小数点符号、thousands_point 为千位 符号、decimal_places 为小数点位数

### 税率

tax_class 表负责管理税类,tax_rates 表管理税率(税收比率)

tax_rates 表的字段 tax_priority 为税率计算优先级,字段 tax_rate 为税率,字段 tax_zone_id 关 联到 zones 表,tax_class_id 字段关联税率 tax_class 表,从而实现了用税类来管理不同的省份/区赋 于不同的税率的特性
