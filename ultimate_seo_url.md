## Ultimate SEO URLs模块

### 安装

1) 上传文件

- 将类文件“seo.class.php”放置在目录“includes/classes/”
- 将文件“reset_seo_cache.php”复制到“admin/includes/”

2) 编辑预处理文件includes/appliation_top.php 

查找:

```php
// include the language translations
require(DIR_WS_LANGUAGES . $language . '.php');
```

在它的下面添加以下代码:

```php
// Ultimate SEO URLs v2.1
include_once(DIR_WS_CLASSES . 'seo.class.php');
if ( !is_object($seo_urls) ){
  $seo_urls = new SEO_URL($languages_id);
}
```

3) 编辑includes/functions/html_output.php

查找 tep_href_link()函数:

```php
////
// The HTML href link wrapper function
function tep_href_link($page = '', $parameters = '', $connection = 'NONSSL',
$add_session_id = true, $search_engine_safe = true) {
  global $request_type, $session_started, $SID;

  if (!tep_not_null($page)) {
    die('<br><br><font color="#ff0000"><b>Error!</b></font><br><br><b>Unable to determine the page link!<br><br>');
  }

  if ($connection == 'NONSSL') {
    $link = HTTP_SERVER . DIR_WS_HTTP_CATALOG;
  } elseif ($connection == 'SSL') {
    if (ENABLE_SSL == true) {
      $link = HTTPS_SERVER . DIR_WS_HTTPS_CATALOG;
    } else {
      $link = HTTP_SERVER . DIR_WS_HTTP_CATALOG;
    }
  } else {
    die('<br><br><font color="#ff0000"><b>Error!</b></font><br><br><b>Unable to determine connection method on a link!<br><br>Known methods: NONSSL SSL</b><br><br>'); 
  }

  if (tep_not_null($parameters)) {
    $link .= $pae . '?' . tep_output_string($parameters);
    $separator = '&';
  } else {
    $link .= $page;
    $separator = '?';
  }

   while ( (substr($link, -1) == '&') || (substr($link, -1) == '?') ) $link = substr($link, 0, -1);

  // Add the session ID when moving from different HTTP and HTTPS servers, or when SID is defined
  if ( ($add_session_id == true) && ($session_started == true) && (SESSION_FORCE_COOKIE_USE == 'False') ) {
    if (tep_not_null($SID)) {
      $_sid = $SID;
    } elseif ( ( ($request_type == 'NONSSL') && ($connection == 'SSL') && (ENABLE_SSL == true) ) || ( ($request_type == 'SSL') && ($connection == 'NONSSL') ) ) {
      if (HTTP_COOKIE_DOMAIN != HTTPS_COOKIE_DOMAIN) {
        $_sid = tep_session_name() . '=' . tep_session_id();
      }
    }
  }

  if ( (SEARCH_ENGINE_FRIENDLY_URLS == 'true') && ($search_engine_safe == true) ) {
    while (strstr($link, '&&')) $link = str_replace('&&', '&', $link);
    $link = str_replace('?', '/', $link);
    $link = str_replace('&', '/', $link);
    $link = str_replace('=', '/', $link);
    $separator = '?';
  }
  
  if (isset($_sid)) {
    $link .= $separator . $_sid;
  }

  return $link;
}
```

替换此函数为:

```php
////
// Ultimate SEO URLs v2.1
// The HTML href link wrapper function
function tep_href_link($page = '', $parameters = '', $connection = 'NONSSL', $add_session_id = true, $search_engine_safe = true) {
  global $seo_urls;

  if ( !is_object($seo_urls) ){
    if ( !class_exists('SEO_URL') ){
      include_once(DIR_WS_CLASSES . 'seo.class.php');
    }

    global $languages_id;
    $seo_urls = new SEO_URL($languages_id);
  }

  return $seo_urls->href_link($page, $parameters, $connection, $add_session_id);
}
```

4) 编辑admin/categories.php 

查找代码:

```php
$action = (isset($HTTP_GET_VARS['action']) ? $HTTP_GET_VARS['action'] : '');
```


在其下添加:

```php
// Ultimate SEO URLs v2.1
// If the action will affect the cache entries
if ( eregi("(insert|update|setflag)", $action) ) include_once('includes/reset_seo_cache.php');
```

5) 编辑admin/includes/functions/general.php

将以下代码添加到?>标签的上面:

```php
// Function to reset SEO URLs database cache entries
// Ultimate SEO URLs v2.1
function tep_reset_cache_data_seo_urls($action){
  switch ($action){
    case 'reset':
      tep_db_query("DELETE FROM cache WHERE cache_name LIKE '%seo_urls%'");
      tep_db_query("UPDATE configuration SET configuration_value='false' WHERE configuration_key='SEO_URLS_CACHE_RESET'");
      break;
    default:
      break;
  }

  # The return value is used to set the value upon viewing
  # It's NOT returining a false to indicate failure!!
  return 'false';
}
```

6) 编辑.htaccess文件

.htaccess 文件必须放置在“catalog”目录下,如果.htaccess 文件不存在,请先创建该文件。 

如果你的网站前台文件放置在“directory”目录下,那么你需要使用下面的代码作为.htacess 文件 的内容,同时将“RewriteBase /directory/”中的“diretory”替换成你的真实目录。

.htaccess 文件内容如下:

```
Options +FollowSymLinks
RewriteEngine On
RewriteBase /directory/
RewriteRule ^(.*)-p-(.*).html$ product_info.php?products_id=$2&%{QUERY_STRING} RewriteRule ^(.*)-c-(.*).html$ index.php?cPath=$2&%{QUERY_STRING}
RewriteRule ^(.*)-m-([0-9]+).html$ index.php?manufacturers_id=$2&%{QUERY_STRING} RewriteRule ^(.*)-pi-([0-9]+).html$ popup_image.php?pID=$2&%{QUERY_STRING} RewriteRule ^(.*)-t-([0-9]+).html$ articles.php?tPath=$2&%{QUERY_STRING} RewriteRule ^(.*)-a-([0-9]+).html$ article_info.php?articles_id=$2&%{QUERY_STRING} RewriteRule ^(.*)-pr-([0-9]+).html$ product_reviews.php?products_id=$2&%{QUERY_STRING}
RewriteRule ^(.*)-pri-([0-9]+).html$
product_reviews_info.php?products_id=$2&%{QUERY_STRING}
RewriteRule ^(.*)-i-([0-9]+).html$ information.php?info_id=$2&%{QUERY_STRING}
RewriteRule ^(.*)-links-(.*).html$ links.php?lPath=$2&%{QUERY_STRING}
```

如果你的网站前台文件放置在站点根目录(osCommerce MS2.2 以前的版本),那么请使用下面的 代码作为.htaccess 文件的内容:

```
Options +FollowSymLinks
RewriteEngine On
RewriteBase /

RewriteRule ^(.*)-p-(.*).html$ product_info.php?products_id=$2&%{QUERY_STRING} RewriteRule ^(.*)-c-(.*).html$ index.php?cPath=$2&%{QUERY_STRING}
RewriteRule ^(.*)-m-([0-9]+).html$ index.php?manufacturers_id=$2&%{QUERY_STRING} RewriteRule ^(.*)-pi-([0-9]+).html$ popup_image.php?pID=$2&%{QUERY_STRING} RewriteRule ^(.*)-t-([0-9]+).html$ articles.php?tPath=$2&%{QUERY_STRING} RewriteRule ^(.*)-a-([0-9]+).html$ article_info.php?articles_id=$2&%{QUERY_STRING} RewriteRule ^(.*)-pr-([0-9]+).html$ product_reviews.php?products_id=$2&%{QUERY_STRING}
RewriteRule ^(.*)-pri-([0-9]+).html$
product_reviews_info.php?products_id=$2&%{QUERY_STRING}
RewriteRule ^(.*)-i-([0-9]+).html$ information.php?info_id=$2&%{QUERY_STRING}
RewriteRule ^(.*)-links-(.*).html$ links.php?lPath=$2&%{QUERY_STRING}
```

### 参数

Ultimate SEO URLS 插件的参数在后台的 Configuration->SEO URLs 里,

| 参数 | 默认值 | 说明 |
|---|---|---|
| Enable SEO URLs | true | 开启/关闭 SEO URLs 插件 |
| Add cPath to product URLs | false | 是否将产品分类路径追加到产品 URL 上 例:some-product-p-1.html?cPath=xx |
| Add category parent to begining of URLs | true | 是否在产品 URL 前面添加上级分类的名称 例:parent-category-p-1.html |
| Choose URL Rewrite Type | Rewrite | URL 重写方式,只支持 Apache Rewrite 方式 |
|  Filter Short Words | 3 | 过滤长度小于多少的词 |
| Output W3C valid URLs (parameter string) | true | 输出 W3C 标准的 URL |
| Enable SEO cache to save queries | true | 是否开启 SEO 缓存 |
| Enable product cache | true | 是否开启产品缓存 |
| Enable categories cache | true | 是否开启产品分类缓存 |
| Enable manufacturers cache | true | 是否开启制造商缓存 |
| Enable articles cache | true | 是否开启文章缓存,仅限已安装 Article 文章模块 |
| Enable information cache | true | 是否开启信息页面缓存,仅限已安装 information 信息模块 |
| Enable topics cache | true | 是否开启话题缓存,仅限已安装 topics 话题模块 |
| Enable automatic redirects | true | 是否启用自动跳转,当以旧的 URL 访问时会自动以 301 方式跳转到新 URL |
| Enter special character conversions | 空 | 特殊字符对应表 <br>格式: char=>conv, char2=>conv2, char3=>conv3 <br>提示: 多个字符替换方案之间用逗号分隔 |
| Remove all non-alphanumeric characters | false | 是否删除识别关键字前的所有非数字字符 例:some-product-p-1.html 将转变成 someproduct-p-1.html  |
| Reset SEO URLs Cache | false | 重置 SEO URLs 缓存 |

> 提示:
> 
> 重置 SEO URLs 缓存,需要选择“reset”然后点击“update”按钮

### 代码分析

动态页面的 URL 静态化,实际为网址重写(URL Rewrite),即指定 URL 规则让 Apache 按新 的 URL 规则进行解析重定向(具体代码请见安装第 6 步:“编辑.htaccess 文件”)。但要做到完全 静态化,则必须在程序中能够轻松地得到指定页面的静态 URL。

所以我们将分析的重点放在静态 URL 的如何得到上面,因为 osCommerce 的 URL 生成都是由 tep_href_link 函数实现,所以我们的 分析重点也就理所当然地放在了“tep_href_link”函数的分析上。

Utimate SEO URLs(以下简称SEO)修改了原有的“tep_href_link”函数,最主要是下面这段代码:

```php
return $seo_urls->href_link($page, $parameters, $connection, $add_session_id);
```

其中$seo_urls 是 SEO 类的对象,可以看出新的 tep_href_link 函数,将实现的细节直接转移到了 SEO 类的方法 href_link 上了。

我们先不直接查看 href_link 方法,首先分析一下 SEO 类的初始化 方法即 SEO_URL 方法,因为接下来你可以看到在具体分析 href_link 方法时,将使用到这里讲的 内容。

#### SEO_URL 类 includes/classes/seo.class.php

```php
// SEO类初始化以$languages_id为参数,即需指定 语言类型。因为不同的语言对应于不同的产品名称/分类名称,而 URL 里会使用到这些名称,所以 SEO 类即使是 在同一个产品的处理上,如果是不同语言的情况下,也可能会生成不同的 URL
function SEO_URL($languages_id){
  global $session_started, $SID;

  $this->installer = new SEO_URL_INSTALLER;
  $this->DB = new SEO_DataBase(DB_SERVER, DB_SERVER_USERNAME, DB_DATABASE, DB_SERVER_PASSWORD);

  // SEO类所在的文件除了SEO类以外,还有两个类,SEO_URL_INSTALLER类是其中之一,SEO_DataBase 类是另外的一个类。这两个类都是 SEO 类的辅助类

  // 其中SEO_URL_INSTALLER类负责SEO类的安装及清除(处理SEO的参数选项以及SEO缓存表等),Utimate SEO URLs插件之所以不像其它插件那样需要手工执行SQL语句,就是因为SEO_URL_INSTALLER类可以判断 SEO 类是否已经安装,并且会自动安装。感兴趣的读者可以仔细研究它的原理。

  // SEO_DataBase 类则是 SEO 类专属的数据库访问类,它封装了 Query 、FetchArray 、NumRows 、 InsertID 、Free 、Slashes 、DBPerform 等方法,其中 DBPerform 方法用于快速执行新增或更新操作。

  $this->languages_id = (int)$languages_id;
  $this->data = array();
  $this->turnOffBrokenUrls(); // 关闭 osCommerce 自带的 SEO 功能选项
  $seo_pages = array(
    FILENAME_DEFAULT,
    FILENAME_PRODUCT_INFO,
    FILENAME_POPUP_IMAGE,
    FILENAME_PRODUCT_REVIEWS,
    FILENAME_PRODUCT_REVIEWS_INFO
  );

  // $seo_pages变量是一个所有需要进行SEO静态化的文件列表,也就是就SEO类有对应的URL改写规则生 成该文件的静态 URL,如果文件不在此列表里,那么在 href_link 方法里会将使用不同的方法得到 URL

  if ( defined('FILENAME_ARTICLES') ) $seo_pages[] = FILENAME_ARTICLES;
  if ( defined('FILENAME_ARTICLE_INFO') ) $seo_pages[] = FILENAME_ARTICLE_INFO;
  if ( defined('FILENAME_INFORMATION') ) $seo_pages[] = FILENAME_INFORMATION;
  if ( defined('FILENAME_LINKS') ) $seo_pages[] = FILENAME_LINKS;

  $this->attributes = array(
    'PHP_VERSION' => PHP_VERSION,
    'SESSION_STARTED' => $session_started,
    'SID' => $SID,
    'SEO_ENABLED' => defined('SEO_ENABLED') ? SEO_ENABLED : 'false',
    'SEO_ADD_CPATH_TO_PRODUCT_URLS' => defined('SEO_ADD_CPATH_TO_PRODUCT_URLS') ? SEO_ADD_CPATH_TO_PRODUCT_URLS : 'false',
    'SEO_ADD_CAT_PARENT' => defined('SEO_ADD_CAT_PARENT') ? SEO_ADD_CAT_PARENT : 'true',
    'SEO_URLS_USE_W3C_VALID' => defined('SEO_URLS_USE_W3C_VALID') ? SEO_URLS_USE_W3C_VALID : 'true',
    'USE_SEO_CACHE_GLOBAL' => defined('USE_SEO_CACHE_GLOBAL') ? USE_SEO_CACHE_GLOBAL : 'false',
    'USE_SEO_CACHE_PRODUCTS' => defined('USE_SEO_CACHE_PRODUCTS') ? USE_SEO_CACHE_PRODUCTS : 'false',
    'USE_SEO_CACHE_CATEGORIES' => defined('USE_SEO_CACHE_CATEGORIES') ? USE_SEO_CACHE_CATEGORIES : 'false',
    'USE_SEO_CACHE_MANUFACTURERS' => defined('USE_SEO_CACHE_MANUFACTURERS') ? USE_SEO_CACHE_MANUFACTURERS : 'false',
    'USE_SEO_CACHE_ARTICLES' => defined('USE_SEO_CACHE_ARTICLES') ? USE_SEO_CACHE_ARTICLES : 'false',
    'USE_SEO_CACHE_TOPICS' => defined('USE_SEO_CACHE_TOPICS') ? USE_SEO_CACHE_TOPICS : 'false',
    'USE_SEO_CACHE_INFO_PAGES' => defined('USE_SEO_CACHE_INFO_PAGES') ? USE_SEO_CACHE_INFO_PAGES : 'false',
    'USE_SEO_CACHE_LINKS' => defined('USE_SEO_CACHE_LINKS') ? USE_SEO_CACHE_LINKS : 'false',
    'USE_SEO_REDIRECT' => defined('USE_SEO_REDIRECT') ? USE_SEO_REDIRECT : 'false',
    'SEO_REWRITE_TYPE' => defined('SEO_REWRITE_TYPE') ? SEO_REWRITE_TYPE : 'false',
    'SEO_URLS_FILTER_SHORT_WORDS' => defined('SEO_URLS_FILTER_SHORT_WORDS') ? SEO_URLS_FILTER_SHORT_WORDS : 'false',
    'SEO_CHAR_CONVERT_SET' => defined('SEO_CHAR_CONVERT_SET') ? $this->expand(SEO_CHAR_CONVERT_SET) : 'false',
    'SEO_REMOVE_ALL_SPEC_CHARS' => defined('SEO_REMOVE_ALL_SPEC_CHARS') ? SEO_REMOVE_ALL_SPEC_CHARS : 'false',
    'SEO_PAGES' => $seo_pages,
    'SEO_INSTALLER' => $this->installer->attributes
  );

  // $attributes成员几乎完全是Ultimate SEO URLs所有参数的复制

  $this->base_url = HTTP_SERVER . DIR_WS_HTTP_CATALOG;
  $this->base_url_ssl = HTTPS_SERVER . DIR_WS_HTTPS_CATALOG; // SEO 类可以根据用户的需要生成 NOSSL 以及 SSL 的链接

  $this->cache = array();
  $this->timestamp = 0;
  $this->reg_anchors = array(
    'products_id' => '-p-',
    'cPath' => '-c-',
    'manufacturers_id' => '-m-',
    'pID' => '-pi-',
    'tPath' => '-t-',
    'articles_id' => '-a-',
    'products_id_review' => '-pr-',
    'products_id_review_info' => '-pri-',
    'info_id' => '-i-',
    'lPath' => '-links-'
  );

  /*
  $reg_anchors 成员是 SEO 类的主角,即 URL 改写规则,可以看到这里的规则必须与在.htaccess 文件里
  应用的规则完全一致,细心的读者一定已经发现,Utimate SEO URLs 插件生成的 URL 都有一个规律,即“名 称”+“关键字”+“识别 ID”,也正因为如此,SEO 类的 URL 改写规则才能如此简单。
  */

  $this->performance = array(
    'NUMBER_URLS_GENERATED' => 0,
    'NUMBER_QUERIES' => 0,
    'NUMBER_STANDARD_URLS_GENERATED' => 0,
    'TOTAL_CACHED_PER_PAGE_RECORDS' => 0,
    // $performance用于统计SEO类的性能
    'CACHE_QUERY_SAVINGS' => 0,
    'TOTAL_TIME' => 0,
    'TIME_PER_URL' => 0,
    'QUERIES' => array()
  );

  if ($this->attributes['USE_SEO_CACHE_GLOBAL'] == 'true'){
    $this->cache_file = 'seo_urls_v2_';
    $this->cache_gc();

    if ( $this->attributes['USE_SEO_CACHE_PRODUCTS'] == 'true' ) $this->generate_products_cache();
    if ( $this->attributes['USE_SEO_CACHE_CATEGORIES'] == 'true' ) $this->generate_categories_cache();
    if ( $this->attributes['USE_SEO_CACHE_MANUFACTURERS'] == 'true' ) $this->generate_manufacturers_cache();
    if ( $this->attributes['USE_SEO_CACHE_ARTICLES'] == 'true' && defined('TABLE_ARTICLES_DESCRIPTION')) $this->generate_articles_cache();
    if ( $this->attributes['USE_SEO_CACHE_TOPICS'] == 'true' && defined('TABLE_TOPICS_DESCRIPTION')) $this->generate_topics_cache();
    if ( $this->attributes['USE_SEO_CACHE_INFO_PAGES'] == 'true' && defined('TABLE_INFORMATION')) $this->generate_information_cache();
    if ( $this->attributes['USE_SEO_CACHE_LINK_PAGES'] == 'true' && defined('TABLE_LINK_CATEGORIES')) $this->generate_links_cache();
  } # end if // 上面的代码用于生成 URL 缓存

  if ($this->attributes['USE_SEO_REDIRECT'] == 'true'){
    $this->check_redirect();
  } # end if
} # end constructor
```

好了,看完了初始化方法,下面让我们来看 href_link 方法吧:

```php
function href_link($page = '', $parameters = '', $connection = 'NONSSL', $add_session_id = true){
  // Some sites have hardcoded &amp;
  $parameters = str_replace('&amp;', '&', $parameters);
  $this->start($this->timestamp);
  $this->performance['NUMBER_URLS_GENERATED']++;

  if ( !in_array($page, $this->attributes['SEO_PAGES']) || $this->attributes['SEO_ENABLED'] == 'false' ) { // 如果没有指定页面的 URL 规则就需要使用默认的 URL 方案
    return $this->stock_href_link($page, $parameters, $connection,
    $add_session_id);
  }

  $link = $connection == 'NONSSL' ? $this->base_url : $this->base_url_ssl;
  $separator = '?';

  if ($this->not_null($parameters)) {
    $link .= $this->parse_parameters($page, $parameters,
    $separator); // parse_parameters 方法分析传入的参数,根据 URL 规则得到优化后的 URL
  } else {
    $link .= $page;
  }

  $link = $this->add_sid($link, $add_session_id, $connection, $separator); //add_sid方法用于决定是否在URL里追加会话ID
  $this->stop($this->timestamp, $time);
  $this->performance['TOTAL_TIME'] += $time;
  
  switch($this->attributes['SEO_URLS_USE_W3C_VALID']){
    case ('true'): // 当设置了生成的 URL 必须兼容 W3C 时执行下面的程序
      if (!isset($_SESSION['customer_id']) &&
        defined('ENABLE_PAGE_CACHE') && ENABLE_PAGE_CACHE == 'true' && class_exists('page_cache')){
        return $link;
      } else {
        return htmlspecialchars(utf8_encode($link));
      }
      break;
    case ('false'):
      return $link;
      break;
  }
} # end function
```

根据上面的代码,具体得到 URL 的方法是 parse_parameters,所以我们需要继续分析 parse_parameters 方法:

```php
function parse_parameters($page, $params, &$separator){
  $p = @explode('&', $params);
  krsort($p);

  $container = array();
  foreach ($p as $index => $valuepair){
    $p2 = @explode('=', $valuepair);
    switch($p2[0]){
      //这里判断GET参数,选择适当的URL替换规则
      case 'products_id': // 适用产品页 URL 替换规则
        switch(true){
          case ( $page == FILENAME_PRODUCT_INFO && !$this->is_attribute_string($p2[1]) ):
            // make_url方法实现了URL生成过程
             $url = $this->make_url($page, urlencode($this->get_product_name($p2[1])), $p2[0], $p2[1], '.html', $separator);
            break;
          case ( $page == FILENAME_PRODUCT_REVIEWS ):
            // 产品评论页
            $url = $this->make_url($page, urlencode($this->get_product_name($p2[1])), 'products_id_review', $p2[1], '.html', $separator);
            break;
          case ( $page == FILENAME_PRODUCT_REVIEWS_INFO):
            //评论详细页
            $url = $this->make_url($page, urlencode($this->get_product_name($p2[1])), 'products_id_review_info', $p2[1], '.html', $separator);
            break;
          default:
            $container[$p2[0]] = $p2[1];
            break;
        } # end switch
        break;
      case 'cPath': // 适用产品分类页 URL 替换规则
        switch(true){
          case ($page == FILENAME_DEFAULT):
            $url = $this->make_url($page, urlencode($this->get_category_name($p2[1])), $p2[0], $p2[1], '.html', $separator);
            break;
          case ( !$this->is_product_string($params)):
            if ($this->attributes['SEO_ADD_CPATH_TO_PRODUCT_URLS'] == 'true' ){
               $container[$p2[0]] = $p2[1];
            }
            break;
          default:
            $container[$p2[0]] = $p2[1];
            break;
        } # end switch
        break;
      case 'manufacturers_id': // 适用制造商页 URL 替换规则
        switch(true){
          case ($page == FILENAME_DEFAULT && !$this->is_cPath_string($params) && !$this->is_product_string($params) ):
            $url = $this->make_url($page, urlencode($this->get_manufacturer_name($p2[1])), $p2[0], $p2[1], '.html', $separator);
            break;
          case ($page == FILENAME_PRODUCT_INFO):
            break;
          default:
            $container[$p2[0]] = $p2[1];
            break;
        } # end switch
        break;
      case 'pID': // 适用产品图片页 URL 替换规则
        switch(true){
          case ($page == FILENAME_POPUP_IMAGE):
            $url = $this->make_url($page, urlencode($this->get_product_name($p2[1])), $p2[0], $p2[1], '.html', $separator);
            break;
          default:
            $container[$p2[0]] = $p2[1];
            break;
        } # end switch
        break;
      case 'tPath': // 适用文章页 URL 替换规则(需安装文章 Article 模块)
        switch(true){
        case ($page == FILENAME_ARTICLES):
          $url = $this->make_url($page, urlencode($this->get_topic_name($p2[1])), $p2[0], $p2[1], '.html', $separator);
          break;
        default:
          $container[$p2[0]] = $p2[1];
          break;
      } # end switch
      break;
      //ojp lPath
      case 'lPath': // 适用链接页 URL 替换规则(需安装链接 Link 模块)
        switch(true){
          case ($page == FILENAME_LINKS):
            $url = $this->make_url($page, urlencode($this->get_link_name($p2[1])), $p2[0], $p2[1], '.html', $separator);
            break;
          default:
            $container[$p2[0]] = $p2[1];
            break;
        } # end switch
        break;
      case 'articles_id': // 适用详细文章内容页 URL 替换规则(需安装文章 Article 模块)
        switch(true){
          case ($page == FILENAME_ARTICLE_INFO):
            $url = $this->make_url($page, urlencode($this->get_article_name($p2[1])), $p2[0], $p2[1], '.html', $separator);
            break;
          default:
            $container[$p2[0]] = $p2[1];
            break;
        } # end switch
        break;
      case 'info_id': // 适用介绍页 URL 替换规则(需安装信息Information 模块)
        switch(true){
          case ($page == FILENAME_INFORMATION):
            $url = $this->make_url($page, urlencode($this->get_information_name($p2[1])), $p2[0], $p2[1], '.html', $separator);
            break;
          default:
            $container[$p2[0]] = $p2[1];
            break;
        } # end switch
        break;
      default:
        if( isset($p2[1]) ) $container[$p2[0]] = $p2[1];
        break;
    } # end switch
  } # end foreach $p

  $url = isset($url) ? $url : $page;
  if ( sizeof($container) > 0 ){
    if ( $imploded_params = $this->implode_assoc($container) ){
      $url .= $separator . $this->output_string($imploded_params );
      $separator = '&';
    } 
  }

  return $url;
} # end function
```

以下是 make_url 方法的代码:

```php
function make_url($page, $string, $anchor_type, $id, $extension = '.html', &$separator){
  // 参数
  // $page:页面(保留用法)
  // $string:URL前导文本
  // $anchor_type:识别符
  // $id:识别ID
  // $extension:扩展名
  // $separator:分隔符(保留用法)  
  // Right now there is but one rewrite method since cName was dropped
  // In the future there will be additional methods here in the switch
  switch ( $this->attributes['SEO_REWRITE_TYPE'] ){
    case 'Rewrite':
      return $string . $this->reg_anchors[$anchor_type] . $id .  $extension;
      break;
    default:
      break;
  } # end switch
} # end function
```

因为在指定参数时,需要根据 ID,得到名称(如指定产品 ID,最后生成的 URL 前导为产品 名称),所以 Ultimate SEO URLs 使用了缓存来保存相关信息,以减少查询数据库的次数提高效率。

Ultimate SEO URLs 将每个此类功能都封装成单独的函数,如 get_product_name 函数用于获得指定 ID 的产品名称,我们下面以 get_product_name 函数为例介绍缓存技术在 Ultimate SEO URLs 里的应用,其它类似功能请读者参考源码。


```php
function get_product_name($pID){
  switch(true){
    case ($this->attributes['USE_SEO_CACHE_GLOBAL'] == 'true' && defined('PRODUCT_NAME_' . $pID)): // 当开启了全局变量选项时,需要通过全局变量获得缓存的数据
      $this->performance['CACHE_QUERY_SAVINGS']++;
      $return = constant('PRODUCT_NAME_' . $pID);
      $this->cache['PRODUCTS'][$pID] = $return;
      break;
    case ($this->attributes['USE_SEO_CACHE_GLOBAL'] == 'true' && isset($this->cache['PRODUCTS'][$pID])):
      // 如果没有开启全局变量选项,需要访问成员 cache 数 组的值获得缓存数据
      $this->performance['CACHE_QUERY_SAVINGS']++;
      $return = $this->cache['PRODUCTS'][$pID];
      break;
    default: // 如果没有缓存数据,则必须查询数据库
      $this->performance['NUMBER_QUERIES']++; $sql = "SELECT products_name as pName FROM ".TABLE_PRODUCTS_DESCRIPTION." WHERE products_id='".(int)$pID."' AND language_id='".(int)$this->languages_id."' LIMIT 1";
      $result = $this->DB->FetchArray( $this->DB->Query( $sql ));

      $pName = $this->strip( $result['pName'] );
      $this->cache['PRODUCTS'][$pID] = $pName; // 保存结果至成员cache 数组实现缓存
      $this->performance['QUERIES']['PRODUCTS'][] = $sql;
      $return = $pName;
      break;
    } # end switch

    return $return;
  } # end function
```

> 提示:
> 
> get_product_name 方法里,Ultimate SEO URLs 使用了 switch 语句替代 if...else 语句的代码习惯, 因为在 PHP 较代版本前,有分析称这样可以提高代码效率,但在新版 PHP 里,这种优点已经是 不存在的了。
> 
> 不过诸位可以在适当的时候使用一下这种使用 switch(true)代替 if...else 的代码风格,因为这种代码 风格可能会使条理更清晰一些。

通过上面的介绍,我们已经了解了在指定了 URL 替换规则时的 URL 该如何生成,接下来介绍没 有指定 URL 替换规则时的处理。

```php
function stock_href_link($page = '', $parameters = '', $connection = 'NONSSL', $add_session_id = true, $search_engine_safe = true) {
  global $request_type, $session_started, $SID;

  if (!$this->not_null($page)) {
    die('</td></tr></table></td></tr></table><br><br><font color="#ff0000"><b>Error!</b></font><br><br><b>Unable to determine the page link!<br><br>');
  }

  if ($page == '/') $page = '';
  if ($connection == 'NONSSL') { // 判断是 SSL 连接方式还是普通连接方式
    $link = HTTP_SERVER . DIR_WS_HTTP_CATALOG;
  } elseif ($connection == 'SSL') {
    if (ENABLE_SSL == true) {
      $link = HTTPS_SERVER . DIR_WS_HTTPS_CATALOG;
    } else {
      $link = HTTP_SERVER . DIR_WS_HTTP_CATALOG;
    }
  } else {
    die('</td></tr></table></td></tr></table><br><br><font color="#ff0000"><b>Error!</b></font><br><br><b>Unable to determine connection method on a link!<br><br>Known methods: NONSSL SSL</b><br><br>');
  }

  if ($this->not_null($parameters)) { // 是否设置了参数
    $link .= $page . '?' . $this->output_string($parameters); $separator = '&';
  } else {
    $link .= $page;
    $separator = '?';
  }

  while ( (substr($link, -1) == '&') || (substr($link, -1) == '?') ) $link = substr($link, 0, -1);
  
  if ( ($add_session_id == true) && ($session_started == true) && (SESSION_FORCE_COOKIE_USE == 'False') ) {
    if ($this->not_null($SID)) {
      $_sid = $SID;
    } elseif ( ( ($request_type == 'NONSSL') && ($connection == 'SSL') && (ENABLE_SSL == true) ) || ( ($request_type == 'SSL') && ($connection == 'NONSSL') ) ) {
      if (HTTP_COOKIE_DOMAIN != HTTPS_COOKIE_DOMAIN) {
        $_sid = $this->SessionName() . '=' . $this->SessionID();
      }
    }
  }

  if ( (SEARCH_ENGINE_FRIENDLY_URLS == 'true') && ($search_engine_safe == true) ) {
    // 如果设置了搜索引擎友好方法 Search Engine Friendly URL 选项,则需要替换掉不合要求的特殊字符
    while (strstr($link, '&&')) $link = str_replace('&&', '&', $link);
    $link = str_replace('?', '/', $link);
    $link = str_replace('&', '/', $link);
    $link = str_replace('=', '/', $link);
    $separator = '?';
  }

  switch(true){ // 追加会话 ID
    case (!isset($_SESSION['customer_id']) && defined('ENABLE_PAGE_CACHE') && ENABLE_PAGE_CACHE == 'true' && class_exists('page_cache')):
      $page_cache = true;
      $return = $link . $separator . '<osCsid>';
      break;
    case (isset($_sid)):
      $page_cache = false;
      $return = $link . $separator . tep_output_string($_sid);
      break;
    default:
      $page_cache = false;
      $return = $link;
      break;
  } # end switch

  $this->performance['NUMBER_STANDARD_URLS_GENERATED']++;
  $this->cache['STANDARD_URLS'][] = $link;
  $time = 0;
  $this->stop($this->timestamp, $time);
  $this->performance['TOTAL_TIME'] += $time;

  switch(true){
    case ($this->attributes['SEO_URLS_USE_W3C_VALID'] == 'true' && !$page_cache):
      return htmlspecialchars(utf8_encode($return)); // 输出 W3C 兼容格式的 URL
      break;
    default:
      return $return;
      break;
  }# end swtich
} # end default tep_href function
```

