## 修改产品分类页面标题

本节将介绍如何将产品分类页面的标题(页面的标题),以及 TITLE(浏览器的标题)修改成当 前产品分类名称。

1、 修改面包屑导航类

文件:includes/classes/breadcrumb.php

添加方法 last:

```php
function last(){
  return $this->_trail[count($this->_trail)-1]['title'];
}
```

2、 修改 index.php 文件:index.php 查找:

`<title><?php echo TITLE; ?></title>`

替换为:

```php
<title>
<?php
if(‘top’ != $category_depth)
  echo $breadcrumb->last();
else
  echo TITLE;
?>
</title>
```

查找:

`<td class="pageHeading"><?php echo HEADING_TITLE; ?></td>`

在 index.php 文件里,查找的结果会有三条,我们需要替换前面两条为下面的代码(第 66 行和第232 行),最后的一条不修改。

替换为:

`<td class="pageHeading"><?php echo $breadcrumb->last(); ?></td>`
