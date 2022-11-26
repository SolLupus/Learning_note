# PHP强等于绕过小trick

绕过`$row['username']==="admin"`

```php
$username = $mysqli->escape_string($_POST['username']);
    $studentid = addslashes_deep($_POST['studentid']);

    if ($mysqli->connect_errno) {
        exit("something err0r!<br>姓名或学号重复");
    }
    if($result = $mysqli->query("select * from users where studentid='$studentid'")) {
        if ($result->num_rows) {
            $result->close();
            echo "<script>alert(\"用户已存在\");</script>";
        } else {
            $arrend = $mysqli->query("select count(*) from challengesAll")->fetch_row()[0];
            $arr = getRandom($arrend);
            $query = "insert into users values (NULL, '$username', '$studentid');";
            $query .= "insert into randChallenges values (NULL, '$username', '$arr[0]', '$arr[1]', '$arr[2]', '$arr[3]', '$arr[4]');";
            if ($mysqli->multi_query($query)===TRUE) {
                $mysqli->close();
                header("Location: login.php");
            } else {
                exit("something error！".$mysqli->error);
            }
        }
    }

```



$mysqli->escape_string出来的username在后端没有被存进去，导致数据库直接存了一个新的admin账号

![QQ图片20220721200206](E:\桌面\QQ图片20220721200206.jpg)

前后端异步操作

![QQ图片20220721200317](E:\桌面\QQ图片20220721200317.jpg)

php传递sql语句的时候带了%00，但是sql处理的时候没有认

urlencode了，就没有问题

![QQ图片20220721200250](E:\桌面\QQ图片20220721200250.jpg)

