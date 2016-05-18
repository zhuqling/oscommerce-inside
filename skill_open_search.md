## 支持OpenSearch搜索

什么是 OpenSearch?OpenSearch 就是开放的搜索框架,最通俗的解释就是新一代浏览器的搜索 框,用户可以直接通过这样的搜索框搜索网站的内容,显示最终搜索结果,而无需打开网站。 

1、OpenSearch 文件

新建文件:opensearch.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<OpenSearchDescription xmlns="http://a9.com/-/spec/opensearch/1.1/">
  <ShortName>Product Search</ShortName>
  <LongName>Product Search</LongName>
  <Description>Search products in my site.</Description>
  <Developer>Developer</Developer>
  <InputEncoding>UTF-8</InputEncoding>
  <Url type="text/html"
template="http://www.mysite.com/advanced_search_result.php?inc_subcat=1&amp;searc
h_in_description=0&amp;categories_id=0&amp;keywords={searchTerms}" />
 <Image width="64" height="64" type="image/png">http://www.
mysite.com/images/favicon_64x64.png</Image>
 <Image width="16" height="16" type="image/png">http://www.
mysite.com/images/favicon_16x16.png</Image>
 <Image width="16" height="16" type="image/vnd.microsoft.icon">http://www.
mysite.com/images/favicon.ico</Image>
</OpenSearchDescription>
```

2、使用方法

在页面的“<head>”标签之间添加如下语句:

`<link rel="search" type="application/opensearchdescription+xml"
href="http://www.mysite.com/opensearch.xml" title="Product Search" />`

修改完成后,便会在浏览器的搜索栏显示有新的可用搜索项,点击确认后便可以安装,安装完成后,就可以直接使用浏览器的搜索框搜索网站产品了。
