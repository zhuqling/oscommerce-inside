## 评论审核

osCommerce 默认的评论系统非常简单,所以我们需要加入最基本审核机制进来,在此我们使用 ReviewApproval System 插件。

插件地址:http://addons.oscommerce.com/info/76

[安装]

1、执行以下 SQL 语句,扩展评论的属性,使得评论需要审核才能被显示出来

```sql
ALTER TABLE `reviews` ADD `approved` TINYINT(3) UNSIGNED DEFAULT "0"
```

2、复制以下文件到相对应的目录

-  catalog/review_notice.php
-  catalog/includes/languages/english/review_notice.php
-  admin/includes/languages/english/images/buttons/review_approve.gif
-  admin/includes/languages/english/images/buttons/review_disapprove.gif

3、修改文件:catalog/product_info.php 

查找(约第 165 行):

```php
$reviews_query = tep_db_query("select count(*) as count from " . TABLE_REVIEWS . " where products_id = '" . (int)$HTTP_GET_VARS['products_id'] . "'");

```

替换为:

```php
$reviews_query = tep_db_query("select count(*) as count from " . TABLE_REVIEWS . " where approved = 1 and products_id = '" . (int)$HTTP_GET_VARS['products_id'] . "'");
```

4、修改文件:catalog/product_reviews.php 

查找(约第 93 行):

```php
$reviews_query_raw = "select r.reviews_id, left(rd.reviews_text, 100) as reviews_text, r.reviews_rating, r.date_added, r.customers_name from " . TABLE_REVIEWS . " r, " . TABLE_REVIEWS_DESCRIPTION . " rd where r.products_id = '" . (int)$product_info['products_id'] . "' and r.reviews_id = rd.reviews_id and rd.languages_id = '" . (int)$languages_id . "' order by r.reviews_id desc";
```

替换为:

```php
$reviews_query_raw = "select r.reviews_id, left(rd.reviews_text, 100) as reviews_text, r.reviews_rating, r.date_added, r.customers_name from " . TABLE_REVIEWS . " r, " . TABLE_REVIEWS_DESCRIPTION . " rd where r.products_id = '" . (int)$product_info['products_id'] . "' and r.reviews_id = rd.reviews_id and rd.languages_id = '" . (int)$languages_id . "' and r.approved = '1' order by r.reviews_id desc";
```

5、修改文件:catalog/product_reviews_write.php 

查找(约第 22 行):

`tep_redirect(tep_href_link(FILENAME_PRODUCT_REVIEWS, tep_get_all_get_params(array('action'))));`

替换为:

`tep_redirect(tep_href_link(FILENAME_REVIEW_NOTICE, tep_get_all_get_params(array('action'))));`

查找(约第 51 行):

`tep_redirect(tep_href_link(FILENAME_PRODUCT_REVIEWS, tep_get_all_get_params(array('action'))));`

替换为:
        
`tep_redirect(tep_href_link(FILENAME_REVIEW_NOTICE,tep_get_all_get_params(array('action'))));`
     

6、修改文件:catalog/reviews.php 

查找(约第 67 行):

```php
$reviews_query_raw = "select r.reviews_id, left(rd.reviews_text, 100) as reviews_text, r.reviews_rating, r.date_added, p.products_id, pd.products_name, p.products_image, r.customers_name from " . TABLE_REVIEWS . " r, " . TABLE_REVIEWS_DESCRIPTION . " rd, " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_status = '1' and p.products_id = r.products_id and r.reviews_id = rd.reviews_id and p.products_id = pd.products_id and pd.language_id = '" . (int)$languages_id . "' and rd.languages_id = '" . (int)$languages_id . "' order by r.reviews_id DESC";
```

替换为:

```php
$reviews_query_raw = "select r.reviews_id, left(rd.reviews_text, 100) as reviews_text, r.reviews_rating, r.date_added, p.products_id, pd.products_name, p.products_image, r.customers_name from " . TABLE_REVIEWS . " r, " . TABLE_REVIEWS_DESCRIPTION . " rd, " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_status = '1' and p.products_id = r.products_id and r.reviews_id = rd.reviews_id and p.products_id = pd.products_id and pd.language_id = '" . (int)$languages_id . "' and rd.languages_id = '" . (int)$languages_id . "' and r.approved = '1' order by r.reviews_id DESC";
```

7、修改文件:catalog/includes/filenames.php

在“?>”之前追加以下代码:

`define('FILENAME_REVIEW_NOTICE', 'review_notice.php');`

8、修改文件:catalog/includes/boxes/reviews.php 

查找(约第 22 行):

```ph
$random_select = "select r.reviews_id, r.reviews_rating, p.products_id, p.products_image, pd.products_name from " . TABLE_REVIEWS . " r, " . TABLE_REVIEWS_DESCRIPTION . " rd, " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_status = '1' and p.products_id = r.products_id and r.reviews_id = rd.reviews_id and rd.languages_id = '" . (int)$languages_id . "' and p.products_id = pd.products_id and pd.language_id = '" . (int)$languages_id . "'";
```

替换为:

```php
$random_select = "select r.reviews_id, r.reviews_rating, p.products_id, p.products_image, pd.products_name from " . TABLE_REVIEWS . " r, " . TABLE_REVIEWS_DESCRIPTION . " rd, " . TABLE_PRODUCTS . " p, " . TABLE_PRODUCTS_DESCRIPTION . " pd where p.products_status = '1' and p.products_id = r.products_id and r.reviews_id = rd.reviews_id and rd.languages_id = '" . (int)$languages_id . "' and p.products_id = pd.products_id and pd.language_id = '" . (int)$languages_id . "' and r.approved = '1'";
```

9、修改文件:admin/reviews.php 

查找(约第 21 行):

```php
$reviews_rating = tep_db_prepare_input($HTTP_POST_VARS['reviews_rating']);
$reviews_text = tep_db_prepare_input($HTTP_POST_VARS['reviews_text']);
```

替换为:

```php
$reviews_rating = tep_db_prepare_input($HTTP_POST_VARS['reviews_rating']);
$last_modified = tep_db_prepare_input($HTTP_POST_VARS['last_modified']);
$reviews_text = tep_db_prepare_input($HTTP_POST_VARS['reviews_text']);
```

查找(约第 36 行):

```php     
tep_db_query("delete from " . TABLE_REVIEWS_DESCRIPTION . " where reviews_id = '" . (int)$reviews_id . "'");
       tep_redirect(tep_href_link(FILENAME_REVIEWS, 'page=' .
$HTTP_GET_VARS['page']));
break; }
} ?>
```

替换为:

```php
tep_redirect(tep_href_link(FILENAME_REVIEWS, 'page=' . $HTTP_GET_VARS['page']));
       break;
case 'approve_review':
$reviews_id = tep_db_prepare_input($HTTP_GET_VARS['rID']); tep_db_query("update " . TABLE_REVIEWS . " set approved=1 where reviews_id =
" . $reviews_id);
       tep_redirect(tep_href_link(FILENAME_REVIEWS, 'page=' .
$HTTP_GET_VARS['page'] . '&rID=' . $reviews_id));
       break;
case 'disapprove_review':
$reviews_id = tep_db_prepare_input($HTTP_GET_VARS['rID']); tep_db_query("update " . TABLE_REVIEWS . " set approved=0 where reviews_id =
" . $reviews_id);
       tep_redirect(tep_href_link(FILENAME_REVIEWS, 'page=' .
$HTTP_GET_VARS['page'] . '&rID=' . $reviews_id));
       break;
}
}
?>
     
查找(约第 205 行):

```php
<td class="dataTableHeadingContent" align="right"><?php echo
TABLE_HEADING_DATE_ADDED; ?></td>
             <td class="dataTableHeadingContent" align="right"><?php echo
TABLE_HEADING_ACTION; ?>&nbsp;</td>
</tr>
```

替换为:

```php
<td class="dataTableHeadingContent" align="right"><?php echo
TABLE_HEADING_DATE_ADDED; ?></td>
             <td class="dataTableHeadingContent" align="center"><?php echo
TEXT_APPROVED; ?></td>
             <td class="dataTableHeadingContent" align="right"><?php echo
TABLE_HEADING_ACTION; ?>&nbsp;</td>
</tr>
```


查找(约第 210 行):

`$reviews_query_raw = "select reviews_id, products_id, date_added, last_modified, reviews_rating from " . TABLE_REVIEWS . " order by date_added DESC";`

替换为:

`$reviews_query_raw = "select reviews_id, products_id, date_added, last_modified, reviews_rating, approved from " . TABLE_REVIEWS . " order by date_added DESC";`

查找(约第 240 行):
     
```php
<td class="dataTableContent" align="right"><?php echo tep_date_short($reviews['date_added']); ?></td>
<td class="dataTableContent" align="right"><?php if ( (is_object($rInfo)) && ($reviews['reviews_id'] == $rInfo->reviews_id) ) { echo tep_image(DIR_WS_IMAGES . 'icon_arrow_right.gif'); } else { echo '<a href="' . tep_href_link(FILENAME_REVIEWS, 'page=' . $HTTP_GET_VARS['page'] . '&rID=' . $reviews['reviews_id']) . '">' . tep_image(DIR_WS_IMAGES . 'icon_info.gif', IMAGE_ICON_INFO) . '</a>'; } ?>&nbsp;</td>
</tr>
```

替换为:

```php
<td class="dataTableContent" align="right"><?php echo tep_date_short($reviews['date_added']); ?></td>
<td class="dataTableContent" align="center"><?php echo $reviews['approved']==1? '<a href="' . tep_href_link(FILENAME_REVIEWS, tep_get_all_get_params(array('action', 'info')) . 'action=disapprove_review&rID=' . $reviews['reviews_id'], 'NONSSL') . '">' . tep_image(DIR_WS_IMAGES . 'icon_status_green.gif', IMAGE_ICON_STATUS_GREEN, 10, 10) . '</a>&nbsp;<a href="' . tep_href_link(FILENAME_REVIEWS, tep_get_all_get_params(array('action', 'info')) . 'action=disapprove_review&rID=' . $reviews['reviews_id'], 'NONSSL') . '">' . tep_image(DIR_WS_IMAGES . 'icon_status_red_light.gif', IMAGE_ICON_STATUS_RED, 10, 10) . '</a>':'<a href="' . tep_href_link(FILENAME_REVIEWS, tep_get_all_get_params(array('action', 'info')) . 'action=approve_review&rID=' . $reviews['reviews_id'], 'NONSSL') . '">' . tep_image(DIR_WS_IMAGES . 'icon_status_green_light.gif', IMAGE_ICON_STATUS_GREEN, 10, 10) . '</a>&nbsp;<a href="' . tep_href_link(FILENAME_REVIEWS, tep_get_all_get_params(array('action', 'info')) . 'action=approve_review&rID=' . $reviews['reviews_id'], 'NONSSL') . '">' . tep_image(DIR_WS_IMAGES . 'icon_status_red.gif', IMAGE_ICON_STATUS_RED, 10, 10) . '</a>' ?></td>
<td class="dataTableContent" align="right"><?php if ( (is_object($rInfo)) && ($reviews['reviews_id'] == $rInfo->reviews_id) ) { echo tep_image(DIR_WS_IMAGES . 'icon_arrow_right.gif'); } else { echo '<a href="' . tep_href_link(FILENAME_REVIEWS, 'page=' . $HTTP_GET_VARS['page'] . '&rID=' . $reviews['reviews_id']) . '">' . tep_image(DIR_WS_IMAGES . 'icon_info.gif', IMAGE_ICON_INFO) . '</a>'; } ?>&nbsp;</td> </tr>
```

查找(约第 281 行):

```php
$contents[] = array('text' => '<br>' . TEXT_INFO_PRODUCTS_AVERAGE_RATING . ' ' . number_format($rInfo->average_rating, 2) . '%');
}
break;
```

替换为:

```php
if($rInfo->approved==0){
$contents[] = array('align' => 'left', 'text' => '<br>&nbsp;' . TEXT_APPROVED
. ': ' . TEXT_NO );
        $contents[] = array('align' => 'center', 'text' => '<a href="' .
tep_href_link(FILENAME_REVIEWS, tep_get_all_get_params(array('action', 'info')) .
'action=approve_review&rID=' . $rInfo->reviews_id, 'NONSSL') . '">' .
tep_image_button('review_approve.gif', TEXT_APPROVE) . '</a>');
        }
       elseif($rInfo->approved==1) {
$contents[] = array('align' => 'left', 'text' => '<br>&nbsp;' . TEXT_APPROVED . ': ' . TEXT_YES );
        $contents[] = array('align' => 'center', 'text' => '<a href="' .
tep_href_link(FILENAME_REVIEWS, tep_get_all_get_params(array('action', 'info')) .
'action=disapprove_review&rID=' . $rInfo->reviews_id, 'NONSSL') . '">' .
                        $contents[] = array('text' => '<br>' . TEXT_INFO_PRODUCTS_AVERAGE_RATING . ' ' .
number_format($rInfo->average_rating, 2) . '%');
} break;
                                              

   tep_image_button('review_disapprove.gif', TEXT_DISAPPROVE) . '</a>');
        }
else{
$contents[] = array('align' => 'left', 'text' => '<br>&nbsp;' . TEXT_APPROVED
. ': ' . "Unknown" );
}
```

10、修改语言文件:admin/includes/languages/english/reviews.php 

追加如下定义:

```php
define('TEXT_APPROVED', 'Approved') ;
define('TEXT_APPROVE', 'Approve') ;
define('TEXT_DISAPPROVE', 'Disapprove') ;
define('TEXT_YES', 'Yes') ;
define('TEXT_NO', 'No') ;
```

以下修改功能为当添加有新评论时,自动向管理员邮箱发送邮箱通知。如果不需要此功能可以不进行以下的修改

11、修改文件:product_reviews_write.php

查找(约第 51 行):

```php
tep_db_query("insert into " . TABLE_REVIEWS_DESCRIPTION . " (reviews_id, languages_id, reviews_text) values ('" . (int)$insert_id . "', '" . (int)$languages_id . "', '" . tep_db_input($review) . "')");
```

在其后添加以下代码:

```php     
$subject = 'Product Review - Approval Required';
$message = 'There is a new product review awaiting approval from your online store, please click the link below to view this review:<br><a href="' . tep_href_link(FILENAME_PRODUCT_REVIEW_EMAIL) . '">' . tep_href_link(FILENAME_PRODUCT_REVIEW_EMAIL) . '</a>';
    $from_name = 'Product Reviews';
    tep_mail(STORE_OWNER, STORE_OWNER_EMAIL_ADDRESS, $subject, $message, $from_name, STORE_OWNER_EMAIL_ADDRESS);
```

再将原代码下的:

```php
tep_redirect(tep_href_link(FILENAME_PRODUCT_REVIEWS,
tep_get_all_get_params(array('action'))));
```

替换为:

```php
tep_redirect(tep_href_link(FILENAME_REVIEW_NOTICE,
tep_get_all_get_params(array('action'))));
```

12、修改文件:catalog/includes/filenames.php 

追加以下定义:

`define('FILENAME_PRODUCT_REVIEW_EMAIL', 'admin/reviews.php');`
