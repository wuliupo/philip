# Server names

> # 虚拟主机名

Server names are defined using the [server_name](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) directive and determine which server block is used for a given request. See also “[How nginx processes a request](http://nginx.org/en/docs/http/request_processing.html)”. They may be defined using exact names, wildcard names, or regular expressions:
> 虚拟主机名使用server_name指令定义，用于决定由某台虚拟主机来处理请求。具体请参考《nginx如何处理一个请求》。虚拟主机名可以使用确切的名字，通配符，或者是正则表达式来定义：

```bash
server {
    listen       80;
    server_name  example.org  www.example.org;
    ...
}

server {
    listen       80;
    server_name  *.example.org;
    ...
}

server {
    listen       80;
    server_name  mail.*;
    ...
}

server {
    listen       80;
    server_name  ~^(?<user>.+)\.example\.net$;
    ...
}
```

When searching for a virtual server by name, if name matches more than one of the specified variants, e.g. both wildcard name and regular expression match, the first matching variant will be chosen, in the following order of precedence:
> nginx以名字查找虚拟主机时，如果名字可以匹配多于一个主机名定义，比如同时匹配了通配符的名字和正则表达式的名字，那么nginx按照下面的优先级别进行查找，并选中第一个匹配的虚拟主机：

1. exact name
1. longest wildcard name starting with an asterisk, e.g. “`*.example.org`”
1. longest wildcard name ending with an asterisk, e.g. “`mail.*`”
1. first matching regular expression (in order of appearance in a configuration file)

> 1. 确切的名字；
> 1. 最长的以星号起始的通配符名字：*.example.org；
> 1. 最长的以星号结束的通配符名字：mail.*；
> 1. 第一个匹配的正则表达式名字（按在配置文件中出现的顺序）。

### Wildcard names

> ### 通配符名字

A wildcard name may contain an asterisk only on the name’s start or end, and only on a dot border. The names “`www.*.example.org`” and “`w*.example.org`” are invalid. However, these names can be specified using regular expressions, for example, “`~^www\..+\.example\.org$`” and “`~^w.*\.example\.org$`”. An asterisk can match several name parts. The name “`*.example.org`” matches not only `www.example.org` but `www.sub.example.org` as well.
> 通配符名字只可以在名字的起始处或结尾处包含一个星号，并且星号与其他字符之间用点分隔。所以，“`www.*.example.org`”和“`w*.example.org`”都是非法的。不过，上面的两个名字可以使用正则表达式描述，即“`~^www\..+\.example\.org$`”和“`~^w.*\.example\.org$`”。星号可以匹配名字的多个节（各节都是以点号分隔的）。“`*.example.org`”不仅匹配`www.example.org`，也匹配`www.sub.example.org`。

A special wildcard name in the form “`.example.org`” can be used to match both the exact name “`example.org`” and the wildcard name “`*.example.org`”.
> 有一种形如“.example.org”的特殊通配符，它可以既匹配确切的名字“example.org”，又可以匹配一般的通配符名字“*.example.org”。

### Regular expressions names

> ### 正则表达式名字

The regular expressions used by nginx are compatible with those used by the Perl programming language (PCRE). To use a regular expression, the server name must start with the tilde character:
> nginx使用的正则表达式兼容PCRE。为了使用正则表达式，虚拟主机名必须以波浪线“~”起始：

```bash
server_name  ~^www\d+\.example\.net$;
```

otherwise it will be treated as an exact name, or if the expression contains an asterisk, as a wildcard name (and most likely as an invalid one). Do not forget to set “^” and “$” anchors. They are not required syntactically, but logically. Also note that domain name dots should be escaped with a backslash. A regular expression containing the characters “{” and “}” should be quoted:
> 否则该名字会被认为是个确切的名字，如果表达式含星号，则会被认为是个通配符名字（而且很可能是一个非法的通配符名字）。不要忘记设置“^”和“$”锚点，语法上它们不是必须的，但是逻辑上是的。同时需要注意的是，域名中的点“.”需要用反斜线“\”转义。含有“{”和“}”的正则表达式需要被引用，如：

```bash
server_name  "~^(?<name>\w\d{1,3}+)\.example\.net$";
```

otherwise nginx will fail to start and display the error message:
> 否则nginx就不能启动，错误提示是：

```bash
directive "server_name" is not terminated by ";" in ...
```

A named regular expression capture can be used later as a variable:
> 命名的正则表达式捕获组在后面可以作为变量使用：

```bash
server {
    server_name   ~^(www\.)?(?<domain>.+)$;

    location / {
        root   /sites/$domain;
    }
}
```

The PCRE library supports named captures using the following syntax:
> PCRE使用下面语法支持命名捕获组：

```bash
?<name> Perl 5.10 compatible syntax, supported since PCRE-7.0
?'name' Perl 5.10 compatible syntax, supported since PCRE-7.0
?P<name> Python compatible syntax, supported since PCRE-4.0
```

If nginx fails to start and displays the error message:
> 如果nginx不能启动，并显示错误信息：

```bash
pcre_compile() failed: unrecognized character after (?< in ...
```

this means that the PCRE library is old and the syntax “`?P<name>`” should be tried instead. The captures can also be used in digital form:
> 说明PCRE版本太旧，应该尝试使用`?P<name>`。捕获组也可以以数字方式引用：

```bash
server {
    server_name   ~^(www\.)?(.+)$;

    location / {
        root   /sites/$2;
    }
}
```

However, such usage should be limited to simple cases (like the above), since the digital references can easily be overwritten.
> 不过，这种用法只限于简单的情况（比如上面的例子），因为数字引用很容易被覆盖。

### Miscellaneous names

> ### 其他类型的名字

There are some server names that are treated specially.
> 有一些主机名会被特别对待。

If it is required to process requests without the “Host” header field in a server block which is not the default, an empty name should be specified:
> 如果需要用一个非默认的虚拟主机处理请求头中不含“Host”字段的请求，需要指定一个空名字：

```bash
server {
    listen       80;
    server_name  example.org  www.example.org  "";
    ...
}
```

If no [server_name](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) is defined in a [server](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) block then nginx uses the empty name as the server name.
> 如果server块中没有定义server_name，nginx使用空名字作为虚拟主机名。

If a server name is defined as “`$hostname`” (0.9.4), the machine’s hostname is used.
> 如果以“$hostname”（nginx 0.9.4及以上版本）定义虚拟主机名，机器名将被使用。

If someone makes a request using an IP address instead of a server name, the “Host” request header field will contain the IP address and the request can be handled using the IP address as the server name:
> 如果使用IP地址而不是主机名来请求服务器，那么请求头的“Host”字段包含的将是IP地址。可以将IP地址作为虚拟主机名来处理这种请求：

```bash
server {
    listen       80;
    server_name  example.org
                 www.example.org
                 ""
                 192.168.1.1
                 ;
    ...
}
```

In catch-all server examples the strange name “_” can be seen:
> 在匹配所有的服务器的例子中，可以见到一个奇怪的名字“_”：

```bash
server {
    listen       80  default_server;
    server_name  _;
    return       444;
}
```

There is nothing special about this name, it is just one of a myriad of invalid domain names which never intersect with any real name. Other invalid names like “--” and “!@#” may equally be used.
> 这没什么特别的，它只不过是成千上万的与真实的名字绝无冲突的非法域名中的一个而已。当然，也可以使用“--”和“!@#”等等。

nginx versions up to 0.6.25 supported the special name “`*`” which was erroneously interpreted to be a catch-all name. It never functioned as a catch-all or wildcard server name. Instead, it supplied the functionality that is now provided by the [server_name_in_redirect](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name_in_redirect) directive. The special name “`*`” is now deprecated and the [server_name_in_redirect](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name_in_redirect) directive should be used. Note that there is no way to specify the catch-all name or the default server using the server_name directive. This is a property of the [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive and not of the [server_name](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) directive. See also “[How nginx processes a request](http://nginx.org/en/docs/http/request_processing.html)”. It is possible to define servers listening on ports `*:80` and `*:8080`, and direct that one will be the default server for port `*:8080`, while the other will be the default for port `*:80`:

> nginx直到0.6.25版本还支持一个特殊的名字“`*`”，这个名字一直被错误地理解成是一个匹配所有的名字。但它从来没有像匹配所有的名字，或者通配符那样工作过，而是用来支持一种功能，此功能现在已经改由server_name_in_redirect指令提供支持了。所以，现在这个特殊的名字“`*`”已经过时了，应该使用server_name_in_redirect指令取代它。需要注意的是，使用server_name指令无法描述匹配所有的名字或者默认服务器。这是listen指令的属性，而不是server_name指令的属性。具体请参考《nginx如何处理一个请求》。可以定义两个服务器都监听`*:80`和`*:8080`端口，然后指定一个作为端口`*:8080`的默认服务器，另一个作为端口`*:80`的默认服务器：

```bash
server {
    listen       80;
    listen       8080  default_server;
    server_name  example.net;
    ...
}

server {
    listen       80  default_server;
    listen       8080;
    server_name  example.org;
    ...
}
```

### Optimization

> ### 优化

Exact names, wildcard names starting with an asterisk, and wildcard names ending with an asterisk are stored in three hash tables bound to the listen ports. The sizes of hash tables are optimized at the configuration phase so that a name can be found with the fewest CPU cache misses. The details of setting up hash tables are provided in a separate [document](http://nginx.org/en/docs/hash.html).
> 确切名字和`*`号在头部或者在末尾的通配符名字存储在三张哈希表中。哈希表和监听端口关联。哈希表的尺寸在配置阶段进行了优化，可以以最小的CPU缓存命中失败来找到名字。设置哈希表的细节参见这篇文档

The exact names hash table is searched first. If a name is not found, the hash table with wildcard names starting with an asterisk is searched. If the name is not found there, the hash table with wildcard names ending with an asterisk is searched.
> nginx首先搜索确切名字的哈希表，如果没有找到，搜索以星号起始的通配符名字的哈希表，如果还是没有找到，继续搜索以星号结束的通配符名字的哈希表。

Searching wildcard names hash table is slower than searching exact names hash table because names are searched by domain parts. Note that the special wildcard form “.example.org” is stored in a wildcard names hash table and not in an exact names hash table.
> 因为名字是按照域名的部分来搜索的，所以搜索通配符名字的哈希表比搜索确切名字的哈希表慢。注意特殊的通配符名字“.example.org”存储在通配符名字的哈希表中，而不在确切名字的哈希表中。

Regular expressions are tested sequentially and therefore are the slowest method and are non-scalable.
> 正则表达式是一个一个串行的测试，所以是最慢的，而且不可扩展。

For these reasons, it is better to use exact names where possible. For example, if the most frequently requested names of a server are example.org and www.example.org, it is more efficient to define them explicitly:
> 鉴于以上原因，请尽可能使用确切的名字。举个例子，如果使用example.org和www.example.org来访问服务器是最频繁的，那么将它们明确的定义出来就更为有效：

```bash
     server {
        listen       80;
        server_name  example.org  www.example.org  *.example.org;
        ...
    }
```

than to use the simplified form:
> 而不是简单的像下面这种方式使用：

```bash
server {
    listen       80;
    server_name  .example.org;
    ...
}
```

 If a large number of server names are defined, or unusually long server names are defined, tuning the [server_names_hash_max_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_max_size) and [server_names_hash_bucket_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size) directives at the http level may become necessary. The default value of the [server_names_hash_bucket_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size) directive may be equal to 32, or 64, or another value, depending on CPU cache line size. If the default value is 32 and server name is defined as “too.long.server.name.example.org” then nginx will fail to start and display the error message:

> 如果定义了大量名字，或者定义了非常长的名字，那可能需要在http配置块中使用server_names_hash_max_size和server_names_hash_bucket_size指令进行调整。server_names_hash_bucket_size的默认值可能是32，或者是64，或者是其他值，取决于CPU的缓存行的长度。如果这个值是32，那么定义“too.long.server.name.example.org”作为虚拟主机名就会失败，而nginx显示下面错误信息：

```bash
could not build the server_names_hash,
you should increase server_names_hash_bucket_size: 32
```

 In this case, the directive value should be increased to the next power of two:
> 出现了这种情况，那就需要将指令的值扩大一倍：

```bash
 http {
    server_names_hash_bucket_size  64;
    ...
```

 If a large number of server names are defined, another error message will appear:
> 如果定义了大量名字，得到了另外一个错误：

 ```bash
 could not build the server_names_hash,
you should increase either server_names_hash_max_size: 512
or server_names_hash_bucket_size: 32
 ```

In such a case, first try to set [server_names_hash_max_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_max_size) to a number close to the number of server names. Only if this does not help, or if nginx’s start time is unacceptably long, try to increase [server_names_hash_bucket_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size).
> 那么应该先尝试设置server_names_hash_max_size的值差不多等于名字列表的名字总量。如果还不能解决问题，或者服务器启动非常缓慢，再尝试提高server_names_hash_bucket_size的值。

If a server is the only server for a listen port, then nginx will not test server names at all (and will not build the hash tables for the listen port). However, there is one exception. If a server name is a regular expression with captures, then nginx has to execute the expression to get the captures.
> 如果只为一个监听端口配置了唯一的主机，那么nginx就完全不会测试虚拟主机名了（也不会为监听端口建立哈希表）。不过，有一个例外，如果定义的虚拟主机名是一个含有捕获组的正则表达式，这时nginx就不得不执行这个表达式以得到捕获组。

### Compatibility

> ### 兼容性


* The special server name “$hostname” has been supported since 0.9.4.
* A default server name value is an empty name “” since 0.8.48.
* Named regular expression server name captures have been supported since 0.8.25.
* Regular expression server name captures have been supported since 0.7.40.
* An empty server name “” has been supported since 0.7.12.
* A wildcard server name or regular expression has been supported for use as the first server name since 0.6.25.
* Regular expression server names have been supported since 0.6.7.
* Wildcard form example.* has been supported since 0.6.0.
* The special form .example.org has been supported since 0.3.18.
* Wildcard form *.example.org has been supported since 0.1.13.

> * 从0.9.4版本开始，支持特殊的虚拟主机名“$hostname”。
> * 从0.8.48版本开始，默认的虚拟主机名是空名字“”。
> * 从0.8.25版本开始，支持虚拟主机名中使用命名的正则表达式捕获组。
> * 从0.7.40版本开始，支持虚拟主机名中使用正则表达式的捕获组。
> * 从0.7.12版本开始，支持空名字“”。
> * 从0.6.25版本开始，通配符和正则表达式名字可以作为第一个虚拟主机名。
> * 从0.6.7版本开始，支持正则表达式的虚拟主机名。
> * 从0.6.0版本开始，支持形如example.*的通配符名字。
> * 从0.3.18版本开始，支持形如.example.org的特殊通配符名字。
> * 从0.1.13版本开始，支持形如*.example.org的通配符名字。
