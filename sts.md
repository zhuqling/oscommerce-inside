## STS

## STS的特点

STS 全称为 Simple Template System,意即为简单模板系统,它是一个 osCommerce 的模板插 件,由第三方开发,但其在 osCommerce 的插件中是最受欢迎的一个。

针对 osCommerce 2.2 MS2, 2.2 RC1 和 2.2 RC2 的版本都有相应的 STS 版本,以 STS v4.5.8 为例,其拥有目录 STS 为原始 STS文件,目录Files for MS2是针对osCommerce2.2 MS2的STS安装文件,目录Files for RC1 是针对 osCommerce 2.2 RC1 版本的安装文件,目录 Files for RC2 是针对 osCommerce 2.2 RC2 版 本的安装文件。

所以读者在使用 STS 模板前一定要看清 osCommerce 的版本说明选择对应的 STS 安装文件。

### 安装STS

#### 备份文件和数据库

因为在添加插件时会涉及到对 osCommerce 原始文件的变更,为了保证数据安全,所以建议先进 行 osCommerce 文件和数据库的备份,以防万一。

#### 复制 STS 文件

-  复制STS类文件catalog\includes\classes\sts.php
-  复制STS函数文件catalog\includes\functions\sts.php
-  复制语言文件目录catalog\includes\languages\english\modules\sts (如果需要使用其它语言,那么就需要同时添加其它语言的文件,默认的 STS 支持三种语言:英语、德语、西班牙语)
-  复制模板类目录catalog\includes\modules\sts
-  复制模板代码目录catalog\includes\modules\sts_inc
-  复制模板目录catalog\includes\sts_templates

#### 编辑 OSC 文件

如果你的 osCommerce 是全新的系统,没有修改过任何文件(其实只要是包括下面要修改的
文件没有编辑过就可以),就可以直接找到对应的版本,将对应的目录复制到 osCommerce,在提 示覆盖文件时选择是便可。

但如果 osCommerce 系统已经被修改过(可能是根据某些插件的需要而进行的修改,或者是 网站为了自己的功能需要而进行的修改等),则要进行以下的手工编辑。

1) catalog/admin/modules.php

```php
switch ($set) {
```

查找:

在它后面添加:

```php
// START STS 4.1
case 'sts':
    $module_type = 'sts';
    $module_directory = DIR_FS_CATALOG_MODULES . 'sts/';
    $module_key = 'MODULE_STS_INSTALLED';
    define('HEADING_TITLE', HEADING_TITLE_MODULES_STS);
    break;
    // END STS 4.1
```

2) catalog\admin\includes\boxes\modules.php

查找:

```php
'<a href="' . tep_href_link(FILENAME_MODULES, 'set=payment', 'NONSSL') . '"class="menuBoxContentLink">' . BOX_MODULES_PAYMENT . '</a><br>' .
```

在其后添加以下代码:

```php
// START STS 4.1
'<a href="' . tep_href_link(FILENAME_MODULES, 'set=sts', 'NONSSL') . '"
class="menuBoxContentLink">' . BOX_MODULES_STS . '</a><br>' .
// END STS 4.1
```

3) catalog\admin\includes\languages\english.php

在文件的最后追加:

```php
//START STS 4.1
define('BOX_MODULES_STS', 'STS');
//END STS 4.1
```

如果使用到其它语种,则需要在其它语种的当前目录进行相应的修改

4) catalog\admin\includes\languages\english\modules.php 

在文件最后添加:

```php
//START STS 4.1
define('HEADING_TITLE_MODULES_STS', 'STS Modules');
//END STS 4.1
```

如果使用到其它语种,则需要在其它语种的当前目录进行相应的修改

5) catalog\includes\application_bottom.php

在文件的开头添加:

```php
// START STS 4.4
if ($sts->display_template_output) {
  $sts->stop_capture();
  include DIR_WS_MODULES.'sts_inc/sts_display_output.php';
}
//END STS 4.4
```

6) catalog\includes\application_top.php

查找:

```php
// initialize the message stack for output messages
require(DIR_WS_CLASSES . 'message_stack.php');
$messageStack = new messageStack;
```

在其前面添加:

```php
// START STS 4.5
require (DIR_WS_CLASSES.'sts.php');
$sts= new sts();
$sts->start_capture();
// END STS
```

7) catalog\includes\column_left.php

在文件开头添加:

```php
// START STS 4.1
if ($sts->display_template_output) {
  include DIR_WS_MODULES.'sts_inc/sts_column_left.php';
} else {
//END STS 4.1
```

在文件结尾添加:

```php
// START STS 4.1
}
// END STS 4.1
```

8) catalog\includes\column_right.php

在文件开头添加:

```php
// START STS 4.1
if ($sts->display_template_output) {
  $sts->restart_capture ('content');
} else {
//END STS 4.1
```

在文件结尾添加:

```php
// START STS 4.1
}
// END STS 4.1
```

9) catalog\includes\footer.php 

在文件开头添加:

```php
// START STS 4.5
if ($sts->display_template_output) {
  // Get content here, in case column_right is not called.
  if (!isset($sts->template['content']))
    $sts->restart_capture ('content');
  } else {
//END STS
```

在文件结尾添加:

```php
// START STS 4.1
}
// END STS 4.1
```

10) catalog\includes\header.php

在文件开头添加:

```php
// START STS 4.1
$sts->restart_capture ('applicationtop2header');
// END STS 4.1
```

11) catalog\includes\functions\html_output.php 

查找:

```php
function tep_image($src, $alt = '', $width = '', $height = '', $parameters = '') {
```

在其后添加:

```php
// START STS v4.4:
global $sts;
$sts->image($src); // Take image from template folder if exists.
// END STS v4.4
```

查找:

```php
$image_submit = '<input type="image" src="' . tep_output_string(DIR_WS_LANGUAGES . $language . '/images/buttons/' . $image) . '" border="0" alt="' . tep_output_string($alt) . '"';
```

在其前面添加:

```php
// START STS v4.4:
global $sts;
$src = $sts->image_button($image,$language);
if ($src!='')
$image_submit = '<input type="image" src="' . tep_output_string($src) . '" border="0" alt="' . tep_output_string($alt) . '"';
else
// END STS v4.4
```

查找:

```php
function tep_image_button($image, $alt = '', $parameters = '') {
  global $language;
```

在其后添加:

```php
// START STS v4.4:
global $sts;
$src = $sts->image_button($image, $language, true); // 3rd parameter to tell tep_image that file check has been already done
if ($src!='') { // Take image from template folder if exists.
  return tep_image ($src);
}
// END STS v4.4
```

12) catalog\includes\classes\boxes.php

boxes.php 是 boxes 的类文件,负责 boxes 的样式呈现。你可以选择是否替换 box 的样式,如果选 择使用 STS 的 box 样式那么直接复制对应的 boxes.php 文件进行覆盖即可(建议先备份原文件), 如“Files for RC2/includes/classes/boxes.php”。但如果你已经拥有满意的 boxes 样式,或者对 boxes 样式不在意,可以不进行这个文件的变更。

### STS 参数设置

进入后台,点击 Modules->STS 进入 STS 选项设置页,效果如下:


可以看出 STS 允许用户自行开启与关闭 STS 模板,只有在“install”后模板才会生效。

STS 模块只有 4 种类型,包括默认模板、首页模板、弹出图片页模板、产品详细页模板。 

1) 默认模板
 
-  Use Templates:是否启用STS,
  STS模块依赖于默认模板上,所以是否启用默认模板就 等于是否启用 STS
-  Code for debug output:用于调试的代码,
  如“index.php?sts_debug=debug”,那么调试代 码即为“debug”
-  Files for normal template:在显示标准模板前,需要执行的代码文件列表
  多个文件之间 用分号分隔,STS 会运行这些代码文件,以决定最终的模板显示内容,代码文件传递参 数给模板文件,而让模板文件负责显示页面,从而实现了代码与显示的分离,代码文件需保存在目录“catalog/includes/modules/sts_inc”中
-  Basefolder:存在模板的基础目录
  标准情况下为“includes/sts_templates/”,STS允许同 时存在多个模板,但只激活一个,当前激活的模板由下面的 Template folder 指定
-  Template folder:使用的模板目录
-  Default template file:默认模板文件名
-  Use template for infoboxes:是否将样式应用到信息框

> 提示:
>
> STS 允许临时切换模板,这样可以在保持现有的模板不变的情况下,测试新的模板。使用临时模 板只需在 URL 后加上“sts_template=TEMPLATE_DIR”(TEMPLATE_DIR 即想要显示的模板路径) 就可以了。

2) 首页模板

-  Use template for index page:是否对首页启用模板
-  Files for index.php template:用于首页模板的代码文件列表
-  Check parent template:是否检查产品分类模板

> 提示:
>
> STS 为了更大程度的灵活性,允许用户针对不同的产品目录设置不一样的模块样式,因为 osCommerce 显示目录和首页同样 使用 index.php 文件(首页文件),所以是否启用检查产品分类 模板的功能被设置在了首页模板参数中。 
> 
> 产品分类模板的文件名是由首页模板的文件名 “index.php.html”转变而来,如“index.php_1_9.html”代表 ID 为 1 的产品分类属下的 ID 为 9 的子分类所使用的模板

3) 弹出图片页模板

-  Use template for the image popup page:是否启用弹出图片页模板
-  Files to include:在显示弹出图片页模板前,需要执行的文件列表

4) 产品详细页模板

-  Use template for product info page:是否启用产品详细页模板
-  EnableSTS3compatibilitmode:是否激活STS3兼容模式
-  Filesfornormaltemplate:在显示product_info.php前,需执行的文件列表
-  Files for content template:在显示产品详细页模板前,需执行的文件列表

### STS原理

#### STS流程


在初始化 STS 后,当调用“includes/application_top.php”里的$sts->start_capture()时,便开始 了主程序的捕获,当执行完成主程序时,调用“includes/application_bottom.php”里的 $sts->stop_capture 结束捕获,并将捕获结果保存起来,在显示模板时再将其显示出来。

模板代码文件的调用是通过 sts 类的 capture_fields 方法来实现的,当得到所有占位关键字与 替换内容后,获取模板文件,然后调用 sts 类的 replace 方法将模板文件里的全部占位关键字替换 成真实的内容,最后显示出模板的内容。

> 提示:
> 
> 模板文件命名规则:如果程序文件名为 abc.php,则模板文件名应为 abc.php.html 模板变量命名规则:与 PHP 变量相同,如$variable

#### 模板全局变量

1.general.php

| 名称 | 说明 |
|---|---|
|  templatedir |  当前模板名称 |
|  htmlparams |  页面内 HTML 标签的限制参数,如指定文本显示 方向等 |
|  date |  当前日期 |
|  langid |  当前语言 ID |
|  sid |  osCommerce 识别 ID |
|  cataloglogo |  带图片的首页链接 A 标签 |
|  urlcataloglogo |  首页链接 URL |
|  cartlogo |  带图片的购物车链接 A 标签 |
|  myaccountlogo |  带图片的用户账户链接 A 标签 |
|  checkoutlogo |  带图片的结账链接 A 标签 |
|  breadcrumbs |  面包屑导航 |
|  title |  标题 |
|  cartcontents |  购物车链接 A 标签 |
|  urlcartcontents |  购物车链接 URL |
|  checkout |  结账链接 A 标签 |
|  urlcheckout |  结账链接 URL |
|  headertags |  页面内 TITLE 标签 |
|  contactlogo |  带图片的联系页面链接 A 标签 |
|  numrequests |  访问统计 |
|  footer_text |  页脚内容 |
|  banner_only |  468x50 格式的动态 Banner |
|  error_message |  错误消息框 |
|  info_message |   提示消息框 |

| 名称 |    未登录 |    登录后 |
|---|---|---|
| myaccount | 账号链接 A 标签 | 账号链接 A 标签 | 
| urlmyaccount | 账号链接 URL | 账号链接 URL |
| logoff |  注销链接 A 标签 | 空文本 |
| urllogoff | 注销链接 URL | 空文本 |
| myaccountlogoff |  myaccount logoff | myaccount |
| loginofflogo | 带图片的注销链接 A 标 签 |  空文本 |

2.sts_user_code.php

| 名称 | 说明 |
|---|---|
| catmenu |   产品目录 |
| footer |   页脚栏 |
| banner |    468x50 格式的 Banner 框 |

3.sts_column_left.php

| 名称 | 说明 |
|---|---|
| manufacturerbox |   制造商框 |
| categorybox |   目录框 |
| whatsnewbox |   新产品框 |
| searchbox |   搜索框 |
| cartbox |   购物车框 |
| informationbox |    信息提示框 |
|    maninfobox |    当前产品的制造商信息框,仅限产品详细页使用 |
| orderhistorybox |   订单历史记录框 |
| bestsellersbox_only、bestsellersbox |   最畅销产品框 |
| notificationbox |   订阅产品框,仅限产品详细页使用 |
| specialbox |   特价产品框 |
| tellafriendbox |   告知好友框,仅限产品详细页使用 |
| languagebox |   语言选择框 |
| currenciesbox |   币种选择框 |
| reviewsbox |    留言评价框 |

4.product_info.php

| 名称 | 说明 |
|---|---|
| productsid |   当前产品 ID |
| startform |   用于购买产品的 FORM 头 |
| endform |   用于购买产品的 FORM 尾 |
| regularprice |   标准价 |
| regularpricestrike |   标准价显示,当产品有特价时,标准价会被用 S 标签包围,显示成已删除状态 |
| specialprice |   产品特价 |
| productname |   产品名称 |
| productmodel |   产品型号 |
| imagesmall |   小图片 |
| imagelarge |   大图片 |
| product_popup |   弹出图片的 JAVASCRIPT 代码 |
| productdesc |   产品描述 |
| optionheader |   产品选项标志(Available Options:) |
| optionnames |   产品所有属性名 |
| optionchoices |   产品所有属性选择框代码 |
| reviews |   产品评价与留言 |
| moreinfolabel |    产品更多信息标签文本,仅限产品设置有 url 有 效 |
|    moreinfourl |    产品更多信息链接 URL,仅限产品设置有 url 有 效 |
| productdatelabel |   产品可用提示,仅当产品设置有 Date Available 值 |
| productdate |   产品生效日期,仅当产品设置有 Date Available 值 |
| reviewsurl |   产品留言提交 URL |
| reviewsbutton |   留言按钮 |
| addtocartbutton |    添加到购物车按钮 |

5.popup_image.php

|  名称 | 说明 |
|---|---|
| productname |   产品名称 |
| productmodel |   产品型号 |
| popupimage |   产品图片 |
| back |   保留参数,上一张图片 |
| next |   保留参数,下一张图片 |
| count |    保留参数,产品图片数量指示器如“2/7 ”,表 示总图片有 7 张,当前显示第 2 张图片 |
       
### STS技巧

#### 替换产品分类页面标题

查找文件 index.php: 

```php
<?php echo HEADING_TITLE; ?>
```

找到第一个时,将其替换为如下代码: 

```php
<?php echo $category['categories_name'] ?>
```

找到第二个时,替换成以下代码:

```php
<?php
if (isset($HTTP_GET_VARS['manufacturers_id'])) {
  $category_query = tep_db_query("select manufacturers_name from " . TABLE_MANUFACTURERS . " where manufacturers_id = '" . (int)$HTTP_GET_VARS['manufacturers_id'] . "'");

  $category = tep_db_fetch_array($category_query);
  if ($category['manufacturers_name'] != "") {
    echo $category['manufacturers_name'];
  } else {
    echo HEADING_TITLE;
  }
} else {
  $category_query = tep_db_query("select cd.categories_name from " . TABLE_CATEGORIES . " c, " . TABLE_CATEGORIES_DESCRIPTION . " cd where c.categories_id = '" . (int)$current_category_id . "' and cd.categories_id = '" . (int)$current_category_id . "' and cd.language_id = '" . (int)$languages_id . "'");

  $category = tep_db_fetch_array($category_query);
  if ($category['categories_name'] != "") {
    echo $category['categories_name'];
  } else {
    echo HEADING_TITLE;
  }
}
?>
```

#### 多语言下的图片

1) 不同语言下的图片按钮

如果需要替换现有的按钮图片,只需要将所有按钮图片放置在模板目录下的“images
/LANGUAGE//buttons”文件夹即可,其中 LANGUAGE 为语言名称,如“english”等,

osCommerce 在显示按钮图片时,会自动查找模板目录下的图片,如此便实现了在不同语言下显示不同的图片 按钮,这一点的好处是为实现不同模板拥有不同按钮方案提供了快速的解决方法

2) 不同语言下的图片

语言的 ID 对应到模板的变量名为$langid,所以如果需要显示图片可以使用下面的代码

`<img border="0" src="images/myimage$langid.jpg">`

假设当前语言是 English,其语言 ID 是 1,那么上面的代码将显示图片“images/myimage1.jpg”, 再如语言为 Italian 时,其语言 ID 是 2,以上代码将显示图片“images/myimage2.jpg”,从而实现 了不同语言下显示不同图片的效果。

#### STS兼容插件

STS Power Pack 是 STS 豪华包,使用 STS 时,一些插件必须进行修改,从而形成了该插件的针对 STS 的特性而设置的特定版本,当您在使用 STS 模板系统时,同时使用到以下的插件,那么可以 考虑选择 STS 豪华包

-  Header Tags Controller (STS Power Pack) :页面头标签插件
-  Ez new fields (STS Power Pack) :
-  Master Products (STS Power Pack) :产品管理插件
-  Dynamenu & STSv4 :动态产品目录菜单和 STS
-  Custom Header and Footer Includes for STSv4x :基于 STS 的自定义页首与页尾
-  STSv4.3.3andHTCv2.6.0Bundle:页面头标签插件与STS包
-  MoPics6 popup:多图片展示插件
-  AddNewPagesUsingSTS:基于STS的添加新页面
-  Top of page for STS:基于 STS 的热门页面
-  STSv4.5.2andHTCv2.6.3Bundle:STS和页面头标签插件包   
- STS_MEGA_POWER_PACK_BUNDLE:STS豪华插件包
-  LIGHTBOXImageforSTS4+:基于STS的产品图片展示插件
-  AttributesSorterCopierforSTSv4:基于STS的产品属性操作插件
-  STSv4.5.2andPRODUCT_EXTRA_FIELDS:STS和产品附加属性
-  Dynamic Sitemap and STS :STS 与动态网站地图
-  STSandMarqueeTextBanner:STS与滚动文字Banner
-  EasyMetaTags–CUSTOMProductTagsforSTSv4:简易Meta标签,基于STS的自定义产品
标签

> 关于STS Power Pack的更多内容请访问:http://www.oscommerce.com/community/contributions,4456

  