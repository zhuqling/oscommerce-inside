## 修改搜索结果页面标题

1、修改文件 catalog/advanced_search_result.php (约第 156 行)

查找: 

`<td class="pageHeading"><?php echo HEADING_TITLE_2 ; ?></td>`

替换为以下代码:

```php
<td class="main"><?php echo sprintf(HEADING_TITLE_2, $HTTP_GET_VARS['keywords']);?></td>
```  

2、修改语言文件 catalog/includes/languages/english/advanced_search.php (约第 17 行)

查找:

`define('HEADING_TITLE_2', 'Products meeting the search criteria');`

替换为以下代码:

```php
define('HEADING_TITLE_2', '<font size="2">Your search for
</font><br><font size="4"><b>%s</b></font><br><font size="2">returned the following results:</font>');
```
