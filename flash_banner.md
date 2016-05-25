## FlashBanner

如何将 Flash swf 文件放入 Banner 使网站更加生动?如何避免直接加入的 Flash 受到浏览器的 限制(浏览器对 Flash 动画增加了限制,以免恶意的 Flash 攻击,只有当用户点击 Flash 后,Flash的动作才会被激活)?幸好有flash_banners插件,它不仅可以让我们制作Flash Banner就像制作普通图片广告一样简单,而且利用脚本突破了浏览器对 Flash 的限制。 

插件地址:http://addons.oscommerce.com/info/3009

[安装]

1、复制 Javascript 脚本文件 AC_ActiveX.js 和 AC_RunActiveContent.js 到 Catalog 根目录 

2、修改文件:catalog/includes/functions/html_output.php

添加以下函数:

```php
function mm_output_flash_movie($name, $movie, $width = '' , $height = '' , $background = '' , $parameters = '') {
  if(tep_not_null($width)) {
    $movie_width = 'width="'.$width.'"' ;
  }

  if(tep_not_null($height)) {
    $movie_height = 'height="'.$height.'"' ;
  }

  if(tep_not_null($parameters)) {
    $flash_movie = $movie . '?' . $parameters ;
  } else {
    $flash_movie = $movie ;
  }

  //fix ie 1 :: begins
  $flash = '<script src="AC_RunActiveContent.js" type="text/javascript"></script>' . "\n" ;
  $flash .= '<script type="text/javascript">AC_FL_RunContent(\'codebase\',\'http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=6,0,29,0\', \'width\',\' '. $width .'\', \'height\',\' '. $height .'\', \'quality\',\'high\', \'pluginspage\',\'http://www.macromedia.com/go/getflashplayer\', \'movie\',\' '. $movie .'\' );' . "\n" ;
  $flash .= '</script>' . "\n" ;
  $flash .= '<noscript><EM>' . "\n" ;
  //fix ie 1 :: ends

  $flash .= '<object type="application/x-shockwave-flash" data="'. $movie .'" '. $movie_width .' '. $movie_height.'>'."\n";
  $flash .= '<param name="movie" value="'.$flash_movie.'" />' . "\n";

  if(tep_not_null($background)) {
    $flash .= '<param name="bgcolor" value="#'.$background.'" />' . "\n" ;
  } else {
    $flash .= '<param name="wmode" value="transparent">' . "\n" ;
  }

  $flash .= '</object>' . "\n\n" ;

  //fix ie 2 :: begins
  $flash .= '</EM></noscript><EM>' . "\n" ;
  //fix ie 2 :: ends

  return $flash;
  return $flash;
}
```

3、修改文件:catalog/includes/functions/banner.php 

查找:

```php
// Display a banner from the specified group or banner id ($identifier)
function tep_display_banner($action, $identifier) {
  if ($action == 'dynamic') {
    $banners_query = tep_db_query("select count(*) as count from " . TABLE_BANNERS . " where status = '1' and banners_group = '" . $identifier . "'");
    $banners = tep_db_fetch_array($banners_query);
    if ($banners['count'] > 0) {
      $banner = tep_random_select("select banners_id, banners_title, banners_image, banners_html_text from " . TABLE_BANNERS . " where status = '1' and banners_group = '" . $identifier . "'");
    } else {
      return '<b>TEP ERROR! (tep_display_banner(' . $action . ', ' . $identifier . ') -> No banners with group \'' . $identifier . '\' found!</b>';
    }
  } elseif ($action == 'static') {
    if (is_array($identifier)) {
      $banner = $identifier;
    } else {
      $banner_query = tep_db_query("select banners_id, banners_title,  banners_image, banners_html_text from " . TABLE_BANNERS . " where status = '1' and banners_id = '" . (int)$identifier . "'");
      if (tep_db_num_rows($banner_query)) {
        $banner = tep_db_fetch_array($banner_query);
      } else {
        return '<b>TEP ERROR! (tep_display_banner(' . $action . ', ' . $identifier . ') -> Banner with ID \'' . $identifier . '\' not found, or status inactive</b>'; }
      }
    } else {
      return '<b>TEP ERROR! (tep_display_banner(' . $action . ', ' . $identifier . ') -> Unknown $action parameter value - it must be either \'dynamic\' or \'static\'</b>';
    }

    if (tep_not_null($banner['banners_html_text'])) {
      $banner_string = $banner['banners_html_text'];
    } else {
      $banner_string = '<a href="' . tep_href_link(FILENAME_REDIRECT, 'action=banner&goto=' . $banner['banners_id']) . '" target="_blank">' .   tep_image(DIR_WS_IMAGES . $banner['banners_image'], $banner['banners_title']) . '</a>';
    }

  tep_update_banner_display_count($banner['banners_id']);
  return $banner_string;
}
```

替换为:

```php
function tep_display_banner($action, $identifier) {
  if ($action == 'dynamic') {
    $banners_query = tep_db_query("select count(*) as count from " . TABLE_BANNERS . " where status = '1' and banners_group = '" . $identifier . "'");
    $banners = tep_db_fetch_array($banners_query);
    if ($banners['count'] > 0) {
      $banner = tep_random_select("select banners_id, banners_title, banners_image, banners_html_text from " . TABLE_BANNERS . " where status = '1' and banners_group = '" . $identifier . "'");
    } else {
    return '<b>TEP ERROR! (tep_display_banner(' . $action . ', ' . $identifier .') -> No banners with group \'' . $identifier . '\' found!</b>';
    }
  } elseif ($action == 'static') {
    if (is_array($identifier)) {
      $banner = $identifier;
    } else {
      $banner_query = tep_db_query("select banners_id, banners_title, banners_image, banners_html_text from " . TABLE_BANNERS . " where status = '1' and banners_id = '" . (int)$identifier . "'");
      if (tep_db_num_rows($banner_query)) {
        $banner = tep_db_fetch_array($banner_query);
      } else {
        return '<b>TEP ERROR! (tep_display_banner(' . $action . ', ' . $identifier . ') -> Banner with ID \'' . $identifier . '\' not found, or status inactive</b>'; 
      }
    }
  } else {
    return '<b>TEP ERROR! (tep_display_banner(' . $action . ', ' . $identifier . ') -> Unknown $action parameter value - it must be either \'dynamic\' or \'static\'</b>';
  }

  if (tep_not_null($banner['banners_html_text'])) {
    $banner_string = $banner['banners_html_text'];
  } else {
    if ( substr($banner['banners_image'], -3, 3) == 'swf' ) {
      $size = getimagesize(DIR_WS_IMAGES . $banner['banners_image']);
      $banner_string = '<a href="' . tep_href_link(FILENAME_REDIRECT, 'action=banner&goto=' . $banner['banners_id']) . '" target="_blank">' . mm_output_flash_movie( $banner['banners_title'], DIR_WS_IMAGES . $banner['banners_image'] , $size[0] , $size[1]) . '</a>';
    } else {
      $banner_string = '<a href="' . tep_href_link(FILENAME_REDIRECT, 'action=banner&goto=' . $banner['banners_id']) . '" target="_blank">' . tep_image(DIR_WS_IMAGES . $banner['banners_image'], $banner['banners_title']) . '</a>';
    }
  }

  tep_update_banner_display_count($banner['banners_id']);
  return $banner_string;
}
```
