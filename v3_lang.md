
## 语言文件

osCommerce V3的语言分成两部分:一个是语言文件,区别于osCommerce RC2.2的语言文件, V3 的语言文件采用 XML 格式;第二个是语言表,前面加载语言时只使用到语言表,并且语言表的内容会被写入到缓存文件,这样就可以大大提高语言加载的效率。

下面我们将分步骤讲解 osCommerce V3 是如何使用语言表,并得到需要的语言定义,以及语言文件是如何转化为语言表的。

### 文件语言的加载 

语言文件的加载由语言服务提供支持,语言服务通过加载语言类,设置正确的当前语言,便会开始加载通用模块语言。

语言服务 includes/modules/services/language.php

```php
class osC_Services_language {
  function start() {
    global $osC_Language, $osC_Session;

    require('includes/classes/language.php');
    $osC_Language = new osC_Language();
    if (isset($_GET['language']) && !empty($_GET['language'])) {
      $osC_Language->set($_GET['language']); // 设置当前语言
    }

    $osC_Language->load('general'); // 加载通用语言定义
    $osC_Language->load('modules-boxes'); // 加载通用内容框语言定义 
    $osC_Language->load('modules-content'); // 加载通用功能模块语言定义

    header('Content-Type: text/html; charset=' . $osC_Language->getCharacterSet()); // 按照语言的设置,设定页面的字符编码
    osc_setlocale(LC_TIME, explode(',', $osC_Language->getLocale())); // 设置时区 return true;
  }

  function stop() {
   return true;
  }
}
```

加载指定语言定义 includes/classes/language.php[55]

```php
function load($key, $language_code = null) {
  global $osC_Database;
  
  if ( is_null($language_code) ) {
    $language_code = $this->_code;
  }

  if ( $this->_languages[$language_code]['parent_id'] > 0 ) {
    $this->load($key, $this->getCodeFromID($this->_languages[$language_code]['parent_id'])); // osCommerce V3拥有父语言的概念,当加载的语言拥有父语言时,首先会加载其父语言的定义
  }

  $Qdef = $osC_Database->query('select * from :table_languages_definitions where languages_id = :languages_id and content_group = :content_group');
  $Qdef->bindTable(':table_languages_definitions', TABLE_LANGUAGES_DEFINITIONS);
  $Qdef->bindInt(':languages_id', $this->getData('id', $language_code));
  $Qdef->bindValue(':content_group', $key);
  $Qdef->setCache('languages-' . $language_code . '-' . $key); // 通过 SQL 查询得到语言定义,并保存到缓存文件
  $Qdef->execute();
  
  while ($Qdef->next()) {
    $this->_definitions[$Qdef->value('definition_key')] = $Qdef->value('definition_value'); // 内部变量 $_definitions 保存了所有的语言定义
  }
  
  $Qdef->freeResult();
}
```

如何得到某个语言定义?

要得到单个语言定义,只需要使用 osC_Language 类的 get 方法即可,如: $osC_Language->get('my_account')。osC_Language 类的 get 方法定义如下:

includes/classes/language.php[80]

```php
function get($key) {
  if (isset($this->_definitions[$key])) {
    return $this->_definitions[$key]; //内部变量$_definitions保存了所有的语言定义, 所以可能简单地获得指定的语言定义
  }

  return $key;
}
```

### 模块与内容框语言

功能模块是如何加载它的语言定义的呢?下面是购买推荐模块的初始化方法:

osC_Content_also_purchased_products 类

includes/modules/contents/also_purchased_products.php[24]

```php
function osC_Content_also_purchased_products() {
  global $osC_Language;

  $this->_title = $osC_Language->get('customers_also_purchased_title');
}
```

可以看到,得到语言定义只需要简单地使用 osC_language 类的 get 方法即可。 

同样内容框加载语言定义的方法与功能模块一样,如币种切换的内容框的初始化如下:

osC_Boxes_currencies 类 includes/modules/boxes/currencies.php[22]

```php
function osC_Boxes_currencies() {
  global $osC_Language;
  $this->_title = $osC_Language->get('box_currencies_heading');
}
```

> 提示:
> 
> 因为功能模块语言所使用的类型为“modules-content”,而内容框语言的类型是“modules-boxes”, 在前面语言服务加载时默认已经加载了这两种类型的语言,所以可以在模块和内容框里直接取得语言定义

运输模块与付款模块的语言

运输模块的语言加载位于运输模块类的初始化方法里,运输模块使用语言的类型是 “modules-shipping”。

osC_Shipping 类 includes/classes/shipping.php[44]

`$osC_Language->load('modules-shipping');`

而付款模块语言的加载同样位于付款模块类的初始化方法,付款模块语言的类型是 “modules-payment”

osC_Payment 类 includes/classes/payment.php[45]

`$osC_Language->load('modules-payment');`

   
### 如何编辑语言

语言文件如何转化到语言表?

语言文件是如何转化到语言表,我们将在管理后台找到答案。 

进入到后台的“languages”语言设置页面,第一眼看上去,与其它的页面没有什么太大的不同。

但真实情况并非如它表面那么简单。

首先,osCommerce V3 的语言选项丰富了很多,因为我们本节的主题是如何编辑语言定义,所以 对于新增加的选项在此不一一讲解,请读者自行查看。

第二便是右上角的“Import”按钮,点击以后出现以下内容:

这里便是语言文件转化为语言表的关键之处。

在 osCommerce V3 如果存在已经编写好的 XML 形式语言文件,那么就可以使用“Import”导入语言文件,语言文件内的所有定义便会添加到语言表,以供程序使用。

那么如果需要将语言表还原成 XML 语言文件,是不是也可以做到呢?

是的,在 osCommerce V3 这个功能叫做“Export”导出。它的图标位于编辑图标后面,它的作用是实现语言表到语言文件的转化。

第三,我们点击语言名称,如上图的“English(default)”,便可以进入到语言组的页面 osCommerce V3 里所有语言都归属于各自的组,例如付款模块的语言定义归于“modules-payment”组;“index.php”页面相关的语言归属于“index”组,这样对于语言的管理 便非常有条理

第四,当点击语言组名称时,我们可以进入到详细的语言编辑页面,在这个页面列出了所有归属于所选语言组的语言定义,在这里我们可以新建一条新的语言定义,也可以删除现有的语言定义,或者编辑语言定义的内容
 
下面便是编辑语言定义的页面:

> 提示:
> 
> 因为新的语言文件采用了 XML 格式存储,所以比以前的语言文件修改难度增大了。因此一般建 议在后台的语言页面新增语言定义,这样新的语言定义即可生效,如果需要保存或者转移语言定义,则可以使用 Export 导出功能,将语言定义导出到 XML 文件。

### 模块的语言安装与卸载

付款模块、运输模块的语言会在其安装时被自动转化到语言表,而其被卸载时也会自动从语言表里删除它所有的语言定义。

osC_Modules 类的安装与卸载 includes/classes/modules.php

```php
function install() {
  global $osC_Database, $osC_Language;
  // ......

  foreach ($osC_Language->getAll() as $key => $value) { // 查找所有可用的语言
    if (file_exists(dirname(__FILE__) . '/../languages/' . $key . '/modules/' . $this->_group . '/' . $this->_code . '.xml')) { // 找到适应语言的模块语言文件
      foreach ($osC_Language->extractDefinitions($key . '/modules/' . $this->_group . '/' . $this->_code . '.xml') as $def) { osC_Language 类的 extractDefinitions 方法解析 XML 语言文件,并返回所有语言定义
        $Qcheck = $osC_Database->query('select id from :table_languages_definitions where definition_key = :definition_key and content_group = :content_group and languages_id = :languages_id limit 1');
        $Qcheck->bindTable(':table_languages_definitions',
TABLE_LANGUAGES_DEFINITIONS);
        $Qcheck->bindValue(':definition_key', $def['key']);
        $Qcheck->bindValue(':content_group', $def['group']);
        $Qcheck->bindInt(':languages_id', $value['id']);
        $Qcheck->execute();

        if ($Qcheck->numberOfRows() === 1) { // 如果存在相同的定义,则替换它
          $Qdef = $osC_Database->query('update :table_languages_definitions set definition_value = :definition_value where definition_key = :definition_key and content_group = :content_group and languages_id = :languages_id');
        } else { // 否则新建一条语言定义
          $Qdef = $osC_Database->query('insert into :table_languages_definitions (languages_id, content_group, definition_key, definition_value) values (:languages_id, :content_group, :definition_key, :definition_value)');
        }

        $Qdef->bindTable(':table_languages_definitions', TABLE_LANGUAGES_DEFINITIONS);
        $Qdef->bindInt(':languages_id', $value['id']);
        $Qdef->bindValue(':content_group', $def['group']);
        $Qdef->bindValue(':definition_key', $def['key']);
        $Qdef->bindValue(':definition_value', $def['value']);
        $Qdef->execute();
      }
    }
  }
  
  osC_Cache::clear('languages');
}

// 移除模块
function remove() {
  global $osC_Database, $osC_Language;

  //......

  if (file_exists(dirname(__FILE__) . '/../languages/' . $osC_Language->getCode() . '/modules/' . $this->_group . '/' . $this->_code . '.xml')) {
    foreach ($osC_Language->extractDefinitions($osC_Language->getCode() . '/modules/' . $this->_group . '/' . $this->_code . '.xml') as $def) { // 同样是通过 XML 语言文件,逐一查找
      $Qdel = $osC_Database->query('delete from :table_languages_definitions where definition_key = :definition_key and content_group = :content_group'); // 执行删除
      $Qdel->bindTable(':table_languages_definitions',
TABLE_LANGUAGES_DEFINITIONS);
      $Qdel->bindValue(':definition_key', $def['key']);
      $Qdel->bindValue(':content_group', $def['group']);
      $Qdel->execute();
    }

    osC_Cache::clear('languages');
  }
}
```

