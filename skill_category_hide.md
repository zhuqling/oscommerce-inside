## 隐藏不包含产品的产品分类

文件:includes/boxes/categories.php 替换函数 tep_show_category:

```php          
function tep_show_category($counter) {
  global $tree, $categories_string, $cPath_array;
  $products_in_category = tep_count_products_in_category($counter);

  if($products_in_category>0){
    for ($i=0; $i<$tree[$counter]['level']; $i++) {
      $categories_string .= "&nbsp;&nbsp;";
    }

    $categories_string .= '<a href="';
    
    if ($tree[$counter]['parent'] == 0) {
      $cPath_new = 'cPath=' . $counter;
    } else {
      $cPath_new = 'cPath=' . $tree[$counter]['path'];
    }

    $categories_string .= tep_href_link(FILENAME_DEFAULT, $cPath_new) . '">';

    if (isset($cPath_array) && in_array($counter, $cPath_array)) {
      $categories_string .= '<b>';
    }

    // display category name
    $categories_string .= $tree[$counter]['name'];
    if (isset($cPath_array) && in_array($counter, $cPath_array)) {
      $categories_string .= '</b>';
    }

    if (tep_has_category_subcategories($counter)) {
      $categories_string .= '-&gt;';
    }

    $categories_string .= '</a>';
    if (SHOW_COUNTS == 'true') {
      $categories_string .= '&nbsp;(' . $products_in_category . ')';
    }

    $categories_string .= '<br>';
  }

  if ($tree[$counter]['next_id'] != false) {
    tep_show_category($tree[$counter]['next_id']);
  }
}
```
