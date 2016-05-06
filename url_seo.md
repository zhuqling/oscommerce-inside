## 自制URL SEO类

通过前面对 Ultimate SEO URLs 插件的介绍,我们可以了解到优化后的 URL 由三部分组成, URL识别关键字、名称、识别ID。

因为Ultimate SEO URLs是通过修改href_link函数实现自动 URL 转换的,而在普通动态 URL 里只有识别 ID 并没有名称,所以 SEO 类必须通过识别 ID 查询 到名称,然后得到优化的 URL。

虽然 Ultimate SEO URLs 通过使用缓存技术提高了 URL 的转换效率,但对于通常情况下,能 够得到识别 ID 也必定能够轻易地得到名称,因此我们要设计的 SEO 类将舍弃通过 ID 自动查询名 称的方式,改由调用前由用户赋值。

> 提示:
> 
> 自制 SEO 类不同于 Ultimate SEO URLs 插件,它的使用必须用户自己通过新方法来调用 URL 转换,而不是改造 href_link 函数(如果读者有兴趣的话,完全可以将此类扩展成自动调用的方式)。 
> 
> 所以这里只侧重于介绍 SEO URL 转换过程的实现,而并不是希望开发出一个完整的插件

### Url_Ruler类

Url_Ruler 类 include/classes/url_ruler.class.php

```php
class Url_Ruler
{
  var $_rulers = array(array(),array());

  // SEO 参数
  const prefixPageInclude = array('product_info.php','product_list.php','product_reviews_info.php','product_r eviews.php'); // 执行添加 URL 前缀的页面
  const postfixPageInclude = array(); // 执行添加 URL 后缀的页面
  const prefixUrlString = 'Wholesale-'; // URL前缀
  const postfixUrlString = ''; // URL 后缀

  const SEO_REPLACE_PUNCTION_WORDS = array(
    ' -',' -','- ','- ',
    ' ',' ',' ', '---','--',
    ':','#','>','<','?','^',
    '\\'
  ); // 以上字符将替换成“-”

 // 特殊字符替换规则表
  const SEO_REPLACE_SPECIAL_WORDS = array(
    '\\' => '-', // \
    '&#33' => '!', // !

    '&#34' => ' inch ', // "
    '&quot;' => ' inch ', // "
    '&#39&#39' => ' inch ', // "

    '&#35' => '-', // # 
    '&#36' => ' dollars ', // $
    '&#37' => ' percent ', // %

    '&#38' => ' and ', // &
    '&amp;' => ' and ', // &

    '&#39'=>'-', //'
    '\'' =>'-',
    '&#40' => '(',
    '&#41' => ')',
    '&#42' => '+', // *
    '&#43' => '+',
    '&#44'=>',', // ,
    '&#45' => '-', // -
    '&#46' => '.',
    '&#47'=>'or', // /

    '&#48' => '0',
    '&#49' => '1',
    '&#50' => '2',
    '&#51' => '3',
    '&#52' => '4',
    '&#53' => '5',
    '&#54' => '6',
    '&#55' => '7',
    '&#56' => '8',
    '&#57' => '9',
    '&#58' => '-', // :
    '&#59' => ';',

    '&#60' => '(', // <
    '&lt;' => '(',

    '&#61'=>'=', //=
    '&#62' => ')', // >
    '&gt;' => ')', // >

    '&#63' => '-', // ?
    '&#64' => ' at ', // @

    '&#91' => '[',
    '&#92' => '-', // \ 
    '&#93' => ']',
    '&#94' => '-', // ^ 
    '&#95' => '_', 
    '&#96' => '`', // `

    '&#123' => '{',
    '&#124' => ' or ', // | '&#125' => '}',
    '&#126' => '~', // ~

    '&#' => '-',
    '&' => 'and',
    '@' => ' at ',
    '\'\'' => ' inch ',
    '"' => ' inch ',
    '%' => ' pencent ',
    '$' => ' dollar ',
    '|' => ' or ',
    '*' => '+',
    ', ' => ',',
    '/' => ' or ',
    '+-' => '+',
    '-+-' => '+',
    '-+' => '+'
  );

  /*
  * 新增URL规则
  * $page:页面名称
  * $normalRuler:动态URL的构成规则
  * $seoRuler:SEO URL的构成规则
  */
  function addRuler($page,$normalRuler,$seoRuler) {
    $this->_rulers[0][$page] = $normalRuler;
    $this->_rulers[1][$page] = $seoRuler;
  }

  /**
  * 获取优化的 URL
  * $page:要调用的页面
  * $title:名称
  * $id:识别ID
  * $ssl_enable:是否启用SSL连接
  **/
  function getUrl($page,$title = null,$id = null, $ssl_enable=false) {
    $title = $this->seo_string_replace($title); // tep_seo_string_replace函数用于过滤 URL 特殊字符

    if(defined('SEO_ENABLED') && SEO_ENABLED=='true'){
      // Url_Ruler类可以根据 osCommerce后台设置自动切换是否启用SEO URL
      if(isset($id))
        $result = sprintf($this->_rulers[1][$page],$title,$id); // 静态 URL
      else
        $result=$this->_rulers[1][$page]; //如果没有指定ID,说明URL无须识别ID的参与
    } else {
      if(isset($id))
        $result = sprintf($this->_rulers[0][$page],$id); // 动态 URL
      else
        $result = $this->_rulers[0][$page];
    }

    $result = $this->extra_url_string($page,$result); // 附加 URL 前缀或者后缀

    // 使用 SSL 连接还是普通连接方式
    if('/' == substr($result,0,1)) $result = substr($result,1); if(ENABLE_SSL && $ssl_enable)
      $result = HTTPS_SERVER . '/' . $result;
    else
      $result = HTTP_SERVER . '/' . $result;
    return $result;
  }

  // 修正 URL 里的特殊字符
  function seo_string_replace($value) {
    $origWord = array_keys(Url_Ruler::SEO_REPLACE_SPECIAL_WORDS);
    $destWord = array_values(Url_Ruler::SEO_REPLACE_SPECIAL_WORDS);
    $anchor = str_replace($origWord,$destWord,$value);
    $anchor = str_replace(Url_Ruler::SEO_REPLACE_PUNCTION_WORDS,'-',$anchor);
    return $anchor;
  }

  //附加 URL 前缀或者后缀
  function extra_url_string($page,$url) {
    $tmpPage = strtolower($page);

    if(false === strpos($tmpPage,'.php')) $tmpPage .= '.php';

    foreach (Url_Ruler::prefixPageInclude as $value) {
      if((false === strpos($url,'.php')) &&($tmpPage == $value)) {
        // not real  php file
        $url = Url_Ruler::prefixUrlString . $url;
        break;
      }

      foreach (Url_Ruler::postfixPageInclude as $value) {
        if((false === strpos($url,'.php')) && ($tmpPage == $value) ) {
          $url .= Url_Ruler::postfixUrlString;
          break;
        }
      }

      return $url;
    }
  }
}
```

### 指定URL替换规则

```php
require_once DIR_WS_CLASSES . 'url_ruler.class.php';
$urlRuler = new Url_Ruler(); 
$urlRuler->addRuler('product_info','product_info.php?products_id=%s','%s-p-%s.htm l'); // 产品详细页
$urlRuler->addRuler('product_reviews','product_reviews.php?products_id=%d','%s-pr -%d.html');// 产品评论页
$urlRuler->addRuler('popup_image','show_image.php?products_id=%d&active=%d','%s-s i-%3$d.html?active=%2$d');// 弹出产品图片页
// ...
```

不同于 Utimate SEO URLs 插件,Url_Ruler 类所使用的 URL 转换规则是由用户指定的,而不是默 认在 Url_Ruler 类里,所以我们可以将上面的代码放置在本地配置文件 “includes/local/configure.php”里,或者是预处理文件“includes/application_top.php”里。

### 使用Url_Ruler类

示例: 输出所有可售产品的 URL

```php
$urls_query = tep_db_query("select products_id,products_name,products_last_modified, products_date_added from " . TABLE_PRODUCTS . " INNER JOIN ".TABLE_PRODUCTS_DESCRIPTION." USING(products_id) where language_id=”. $languages_id .” and products_status = 1 order by products_id");
while(false !== ($urls = tep_db_fetch_array($urls_query))) {
  echo $urlRuler->getUrl('product_info',$urls['products_name'],$urls['products_id']),”\n”;
}
```
