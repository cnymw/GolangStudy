# linux curl 命令详解

curl命令使用支持的协议之一（HTTP、HTTPS、FTP、FTPS、SCP、SFTP、TFTP、DICT、TELNET、LDAP 或 FILE）向网络服务器传输数据或从网络服务器获取数据。它被设计为在没有 UI
的情况下工作，因此非常适合在 shell 脚本中使用。

该命令提供代理支持，用户身份验证，FTP 上传，HTTP post，SSL 连接，Cookie，文件传输恢复，metalink 等功能。

## 语法

```bash
curl [options] [URL...]
```

### 参数

#### -b, --cookie <name=data>

(HTTP)将数据作为 cookie 传输到 HTTP 服务器。

它应该是之前在`Set-Cookie`行为中从服务器接受的数据。 数据的格式为"NAME1=VALUE1;NAME2=VALUE2"。如果行中未使用"="字符，则该操作被视为用于读取存储 cookie
数据的文件名，如果能够匹配上文件，那么会在这个会话中使用该文件。

使用该方法还可以激活"cookie parser"，使得 curl 也可以记录传入的 cookie，将此方法和 --location 选项结合使用，会比较方便。

#### --ciphers <list of ciphers>

（SSL）指定要在连接中使用的密码（cipher）。

列出的密码必须有效。

#### -c, --cookie-jar <file name>

(HTTP)指定要在操作完成后写入所有 Cookie 的文件。

Curl 将之前从指定文件读取的以及从远程服务器接受的所有 cookie 写入到指定文件里。 如果没有已知的 cookie，则不会写入任何文件。

此命令行选项激活 curl 记录和使用 cookie 的 cookie 引擎。

#### -d, --data <data>

（HTTP）将 POST 请求中的指定数据发送到 HTTP 服务器，模拟方式就像用户填写 HTML 表单并按下提交按钮一样。

请注意，数据完全按照指定方式发送，无需额外处理（所有换行符都被截断）。数据应为`url-encoded`。这会导致 curl 使用类型`application/x-www-form-urlencoded`将数据传递给服务器。

与`-F/--form`相比较，如果在同一命令行上多次使用此选项，则指定的数据将与分割符`&`合并在一起。例如，使用`-d name=daniel-d -d skill=loosy`
将生成类似于`name=daniel-d&skill=loosy`的数据块。如果以`@`字符开始，则其余字符应为读取数据的文件名。如果要从`stdin`读取数据，则应为`-`。文件的内容必须是`url-encoded`
编码的。还可以指定多个文件。

#### --data-binary <data>

（HTTP）将完全按照指定的方式提交数据，无需任何额外处理。

#### -e, --referer <URL>

（HTTP）将`Referer Page`信息发送到 HTTP 服务器。

这也可以通过`-H/--header`选项进行设置。当与`-L/--location`一起使用时，可以在`--referer URL`后面添加`auto`,使得 curl 在`Location:header`之后自动设置上一个
URL。`auto`字符串可以单独使用，即使你没有设置初始`--referer`

#### -E, --cert <certificate[:password]>

（SSL）告诉 curl 在获取带有 HTTPS，FTPS 或其他基于 SSL 协议的文件时使用指定的客户端证书文件。

证书必须采用 PEM 格式。如果未指定可选密码，将在终端上查询该密码。请注意，此选项假定"证书"文件是私钥和私钥连接的文件。

#### --cert-type <type>

（SSL）告诉 curl 所提供证书的类型。

PEM，DER 和 ENG 是公认的类型。如果未指定，默认为 PEM。

#### --cacert <CA certificate>

（SSL）告诉 curl 使用指定的证书文件来验证对等方。

该文件可能包含多个 CA 证书。证书必须采用 PEM 格式。通常情况下，curl 使用默认文件，因此该选项用于更改默认文件。

curl 识别`CURL_CA_BUNDLE`的环境变量（如果已经设置），并使用给定路径作为 CA 证书集合的路径。此选项覆盖该变量。

Windows 版本的 curl 会自动查找名为`curl-ca-bundle.crt`的 CA 证书文件，该文件与 curl.exe 位于同一目录中，或者位于当前工作目录中，或者位于 PATH 中的任何文件夹中。

#### --capath <CA certificate directory>

(SSL)告诉 curl 使用指定的证书目录来验证对等方。

证书必须采用 PEM 格式，并且必须使用 openssl 提供的`c_rehash`组件处理目录。

#### -F, --form <name=content>

（HTTP）这个选项会让 curl 模拟用户按下提交表单的按钮。

curl 会根据 RFC1867 标准使用 Content-type 为 `multipart/formdata` 提交数据。这个选项也允许上传二进制文件等。要强制让`content`部分成为文件，请在文件名前加`@`
字符。要获取文件的内容部分，请在文件名前面加上字符`<`。`@`和`<`之间的区别在于，`@`使指定的文件作为上传的一部分内容，而`<`使文件内容作为文本字段提交。

例如，要将密码文件发送到服务器，其中"password"是输入"/etc/passwd"文件的表单字段名称：

```bash
curl -F password=@/etc/passwd www.mypasswords.com
```

要从`stdin`而不是文件总读取文件内容，请在文件名应该出现的位置使用`-`，这适用于`@`和`<`。你还可以使用`Type=`告诉 curl 使用特定的`Content-type`：

```bash
curl -F "web=@index.html;Type=text/html" url.com
```

或

```bash
curl -F "name=daniel;Type=text/foo" url.com
```

你还可以通过设置"filename="，来显式更改文件上传部分的"name"字段，如下所示：

```bash
curl -F "file=@localfile;filename=nameinpost" url.com
```

如果文件名或路径包含","或";",那么它必须加上双引号，例如：

```bash
curl -F "file=@\"localfile\";filename=\"nameinpost\"" url.com
```

或

```bash
curl -F 'file=@"localfile";filename="nameinpost"' url.com
```

请注意，如果文件名或路径是双引号，则文件名中的任何双引号或反斜杠都必须用反斜杠转义。

#### -H, --header <header>

（HTTP）获取网页时要使用的额外 header。

你可以指定任意数量的额外 header。请注意，如果添加与 curl 将使用的 header 同名的自定义 header，则将使用你外部设置的 header，而不是内置的 header。这使得你可以实现更复杂的场景。

你应该知道在什么时候该替换内置的 header，通过在冒号右侧提供不包含内容的值，来删除内置 header，例如，-H "Host:"。如果发送没有值的自定义 header，则 header 必须以分号终止，例如 -H "
X-Custom-Header;"

#### -i, --include

(HTTP)在输出中包含 HTTP header。

HTTP header包括服务器名称，文档日志，HTTP 版本等内容。

#### --interface <name>

使用指定的接口执行操作。

你可以输入接口名称，IP 地址或主机名，例如：

```bash
curl --interface eth0:1 http://www.netscape.com/
```

#### -I, --head

(HTTP/FTP/FILE) 仅获取 HTTP header。

HTTP 服务器的特点是 HEAD 命令只获取文档的 header。在 FTP 或文件上使用时，curl 仅显示文件大小和上次修改时间。

#### -k, --insecure

(SSL)此选项明确允许 curl 执行"不安全"的 SSL 连接和传输。

所有的 SSL 连接都会使用默认安装的 CA 证书。 除非使用`-k/--insecure`，否者所有被视为"不安全"的连接都会失败。

#### --key <key>

（SSL/SSH）私钥文件名。

允许你在此单独文件中提供私钥。

#### --key-type <type>

（SSL）私钥文件类型。

指定你的`--key`提供的私钥类型。支持 DER，PEM 和 ENG，如果未指定，默认为 PEM。

#### --limit-rate <speed>

指定要使用的最大传输速率。

如果你的带宽有限，并且不希望使用整个带宽，那么这个功能非常有用。

给定的速度以字节/秒为单位，除非添加了后缀。添加`k`或`K`将以千字节做单位，`m`或`M`表示兆字节，`g`或`G`表示千兆字节。

给定的速度是整个传输过程中计算的平均速度，意思是 curl 可能在短时间内使用更高的传输速度，但随着时间的推移，它使用的传输速度不会超过给定的速度。

#### -L, --location

（HTTP/HTTPS）如果服务器返回请求的页面已移动到其他位置（用 Location：header 和 3XX 响应代码表示），此选项将使 curl 在新位置上重定向请求。

如果与`-i/--include`和`-I/--head`一起使用，则会显示所有请求的页面。 使用身份验证时，curl 只将其凭据发送到初始主机。如果重定向到其他主机，它将无法截获用户名+密码。

#### --location-trusted

（HTTP/HTTPS）类似于`-L/--location`，但允许向所有重定向站点发送用户名+密码。

如果站点重定向到你的站点来发送你的身份验证信息（在 HTTP 基本身份验证的情况下为明文），则这可能会导致安全漏洞。

#### --no-keepalive

在 TCP 连接上禁用 keepalive 的使用，因为默认情况下 curl 会启用他们。

#### --no-sessionid

（SSL）禁用 curl 对 SSL session-ID 缓存的使用。

默认情况下，所有传输都使用缓存完成。请注意，尽管尝试重用 SSL session-ID 不会损坏任何东西，但在特殊情况下存在损坏 SSL 的实现，可能需要你禁用它才能成功。

#### --noproxy <no-proxy-list>

不使用指定列表里的代理。

唯一的通配符是`*`字符，它匹配所有主机，并有效地禁用代理。该列表中的每个名称都匹配为包含主机名的域或主机名本身。例如，`local.com` 将匹配 `local.com`,`local.com:80` 和 `www.local.com`
，但不匹配 `www.notlocal.com`。

#### -N, --no-buffer

禁用输出流的缓冲。

在正常工作情况下，curl 使用标准的缓冲输出流，效果是以块的形式输出数据，而不一定是在数据到达时。使用该选项将禁用该缓冲。

#### -o, --output <file>

将输出写入 `<file>` 而不是 stdout。

如果使用{}或[]获取多个文档，则可以在<file>字符中使用`#`后跟数字。该变量将替换为正在获取的 URL 的当前字符串，例如：

```bash
curl http://{one,two}.site.com -o "file_#1.txt"
```

或者使用以下变量：

```bash
curl http://{site,host}.host[1-5].com -o "#1_#2"
```

你可以根据 URL 的数量多次使用该选项。

#### -O, --remote-name

将输出写入本地文件，文件名和我们获取的远程文件一样（仅使用远程文件的文件名部分，路径被忽略）。

用于保存的远程文件名是从给定的 URL 提取的。该文件将保存在当前工作目录中，如果要将文件保存在其他目录中，请确保在使用该命令之前更改当前工作目录。

#### --pass <phrase>

（SSL/SSH）私钥的密码短语。

#### --pubkey <key>

(SSH)公钥文件名。

允许你在此文件中提供公钥。

#### --raw

（HTTP）使用该选项时，会禁用所有内部 HTTP 编码内容或传输编码内容，而是将内容原封不动地传递给用户。

#### --resolve < host : port : address >

为特定主机和端口提供自定义地址。

使用此选项，可以使 curl 请求指定的地址，并防止使用正常解析的地址。可以认为这是 curl 提供的`/etc/hosts`
的替代方案。端口号应为主机将要用的特定协议的端口号。这意味着，如果要为同一主机但不同端口提供地址，则需要配置多个条目。

#### --trace <file>

启用后会对将所有传入和传出数据（包括描述性信息）的完整信息输出到指定文件。

#### --trace-ascii <file>

启用后会对将所有传入和传出数据（包括描述性信息）的完整信息输出到指定文件。

该选项与`--trace`非常相似，但省略了十六进制部分，只显示信息的 ASCII 数据部分。它使输出更小，对于未经训练的人来说更容易阅读。

#### -v, --verbose

使抓取的内容更加的详细。

主要用于调试。以`>`开头的行表示 curl 发送的`header`数据，`<`表示 curl 接受的`header`数据，这个数据在正常情况下是隐藏的，以`*`开头的行表示 curl 提供的其他信息。

#### -X, --request <command>

(HTTP) 指定与 HTTP 服务器通信时所要使用的自定义的请求方法。

发起的请求将会使用指定的方法，而不是其他的方法，默认的方法为 GET。通常你不需要这个选项，GET，HEAD，POST 和 PUT 请求都是使用专用的选项来调用的。

#### --max-redirs <num>

设置可跳转的最大重定向数。

如果使用了 -L/--location 支持重定向，再使用该选项可以防止 curl 无休止地执行重定向。默认情况下，限制 50 次重定向。此选项设置为 -1 表示为无限重定向。

## 示例

```bash
curl https://www.computerhope.com/index.htm
```

使用 HTTP 协议从 www.computerhope.com 获取 index.htm 文件，并将内容输出到标准输出。这和浏览器的"查看源代码"功能基本相同，将打印原始 HTML。

```bash
curl https://www.computerhope.com/index.htm > index.htm
```

和上面获取相同的内容，但是将输出重定向到了文件 index.htm。

```bash
curl -O https://www.computerhope.com/index.htm
```

和上面获取相同的内容，并且将输出写入了文件 index.htm。

# 思维导图

![linux-curl命令详解.png](https://cnymw.github.io/GolangStudy/docs/img/linux-curl命令详解.png)
