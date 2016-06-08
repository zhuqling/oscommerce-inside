
## 目录结构与文件

我将以一个表格来说明 osCommerce V3 与 osCommerce RC2.2 在文件以及目录结构上的区别

|  | osCommerce V3 | osCommerce RC2.2 |
| --- | --- |
| 前台执行文件 | 8个 | 44个 |
| 使用的 Javascript 库* | 3个, 目录位于:ext/ 另外还有两个有关 XMLHTTP 的 Javascript 库 , 位 于 includes/javascript/  | 无  |
|是否有单独的产品图片文件类 | 有,并且产品图片会按图片组再 进行分类 目录位于:images/products/ | 无 |
|  前台类文件的个数 目录:includes/classes/ | 44个 | 17个 |
|  前台函数文件的个数 目录:includes/functions/ | 3个 | 12个 |
|  ext 目录的用途 | 共用的 Javascript 库** | 在线付款模块的回调 |
|  类扩展的支持 | includes/classes/database/目录用于 扩展数据库引擎 includes/classes/session/ 目 录 用 于 扩展 SESSION 会话 | 无 |
|  与逻辑分离的内容显示模块 | 位于目录:includes/content/ | 无 |
|  全局的逻辑模块 | 位于目录:includes/content/actions/ | 无 |
|  功能模块 | 目录:includes/modules/content 但 address_book_details.php 、downloads.php、product_listing.php 仍然保留在 includes/modules/目录 |   目录:includes/modules/ |
|  内容框模块 | 目录:includes/modules/boxes/ | 目录:includes/boxes/ |
|  缓存保存路径 | 默认:includes/work,在安装时可更改 | 无 |
|  Header、Footer 及左、右边栏 | 无*** | 位于:includes/ |
| 国家标志图标(用于语种切换) | 目录:images/worldflags/ | 无 |


- * 此处指专门的 Javascript 库或函数,其用途用于处理 XMLHTTP、或者显示可视化文本编辑器, 而只用于判断页面提交是否正确的 Javascript 不包括在内
- ** osCommerce V3 在线付款模块的回调由各个付款模块自行处理,流程更紧凑,也更合理了
- *** osCommerce V3 通过模块系统控制了页面的显示,从而不再需要单独的 Header、Footer 和左、 右边栏文件
