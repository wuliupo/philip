# Beginner’s Guide

This guide gives a basic introduction to nginx and describes some simple tasks that can be done with it. It is supposed that nginx is already installed on the reader’s machine. If it is not, see the [Installing nginx](http://nginx.org/en/docs/install.html) page. This guide describes how to start and stop nginx, and reload its configuration, explains the structure of the configuration file and describes how to set up nginx to serve out static content, how to configure nginx as a proxy server, and how to connect it with a FastCGI application.
> 这篇教程简单介绍了 nginx 并且讲解了一些 nginx 可以解决的简单任务。这里，我们假设 nginx 已经安装在读者的机器上。如果没有，可以看一下如何安装 nginx。这篇教程主要讲解的是如何启用和停止nginx，和重新加载配置，描述配置文件的基本结构和怎样搭建一个 nginx 静态辅助器，怎样配置 nginx 作为一个代理服务器来以及如何用它连接FastCGI应用。

nginx has one master process and several worker processes. The main purpose of the master process is to read and evaluate configuration, and maintain worker processes. Worker processes do actual processing of requests. nginx employs event-based model and OS-dependent mechanisms to efficiently distribute requests among worker processes. The number of worker processes is defined in the configuration file and may be fixed for a given configuration or automatically adjusted to the number of available CPU cores (see [worker_processes](http://nginx.org/en/docs/ngx_core_module.html#worker_processes)).
> nginx 有一个主进程和其他子进程。主进程的主要工作是加载和执行配置文件，并且驻留子进程。子进程用来作为实际的请求处理。nginx 采取基于事件的模型和 OS 依赖的机制，在多个子进程之间高效的分配请求。子进程的个数会直接写在配置文件中并且，对于给定的配置可以是固定的，或者根据可用的 CPU 核数自动的进行调整（参考 子进程）。

The way nginx and its modules work is determined in the configuration file. By default, the configuration file is named `nginx.conf` and placed in the directory `/usr/local/nginx/conf`, `/etc/nginx`, or` /usr/local/etc/nginx`.
> nginx 和它模块的工作方式是在配置文件中写好的。默认情况下，这个配置文件通常命名为 nginx.conf 并且会放置在 `/usr/local/nginx/conf`，`/etc/nginx`，或者 `/usr/local/etc/nginx`。

### Starting, Stopping, and Reloading Configuration

To start nginx, run the executable file. Once nginx is started, it can be controlled by invoking the executable with the -s parameter. Use the following syntax:
> 运行可执行文件就可以开启 nginx，一旦nginx 开启，那么它就可以通过使用 -s 参数的可执行命令控制。使用下列格式：

``` bash
# -c 为 nginx 的配置文件
nginx -c /usr/local/nginx/conf/nginx.conf
```

``` bash
nginx -s signal
```

Where signal may be one of the following:
> signal 可以为下列命令之一：

``` bash
# 直接关闭 nginx
stop — fast shutdown
# 会在处理完当前正在的请求后退出，也叫优雅关闭
quit — graceful shutdown
# 重新加载配置文件，相当于重启
reload — reloading the configuration file
# 重新打开日志文件
reopen — reopening the log files
```

For example, to stop nginx processes with waiting for the worker processes to finish serving current requests, the following command can be executed:
> 比如，等待当前子进程处理完正在执行的请求后，结束 nginx 进程，可以使用下列命令：

``` bash
nginx -s quit
```

This command should be executed under the same user that started nginx.
> 执行该命令的用户需要和启动的 nginx 的用户一致。

Changes made in the configuration file will not be applied until the command to reload configuration is sent to nginx or it is restarted. To reload configuration, execute:
> 如果重载配置文件的命令没有传递给 nginx 或者 nginx 没有重启，那么配置文件的改动是不会被使用的。重载配置文件的命令可以使用：

```bash
nginx -s reload
```

Once the master process receives the signal to reload configuration, it checks the syntax validity of the new configuration file and tries to apply the configuration provided in it. If this is a success, the master process starts new worker processes and sends messages to old worker processes, requesting them to shut down. Otherwise, the master process rolls back the changes and continues to work with the old configuration. Old worker processes, receiving a command to shut down, stop accepting new connections and continue to service current requests until all such requests are serviced. After that, the old worker processes exit.
> 一旦主进程接收到重载配置文件的命令后，它会先检查配置文件语法的合法性，如果没有错误，则会重新加载配置文件。如果成功，则主进程会重新创建一个子进程并且发送关闭请求给以前的子进程。如果没有成功，主进程会回滚改动并且继续使用以前的配置。老的子进程在接受关闭的命令后，会停止接受新的请求并且继续处理当前的请求，直到处理完毕。之后，该子进程就直接退出了。

A signal may also be sent to nginx processes with the help of Unix tools such as the kill utility. In this case a signal is sent directly to a process with a given process ID. The process ID of the nginx master process is written, by default, to the `nginx.pid` in the directory `/usr/local/nginx/logs` or `/var/run`. For example, if the master process ID is 1628, to send the QUIT signal resulting in nginx’s graceful shutdown, execute:
> 在 Unix 工具的帮助下，比如使用 kill 工具，该信号会被发送给 nginx 进程。在这种情况下，信号会被直接发送给带有进程 ID 的进程。nginx 的主进程的进程 ID 是写死在 nginx.pid 文件中的。该文件通常放在 /usr/local/nginx/logs 或者 /var/run 目录下。比如，如果主进程的 ID 是 1628，为了发送 QUIT 信号来使 nginx 优雅退出，可以执行：

``` bash
kill -s QUIT 1628
```

For getting the list of all running nginx processes, the ps utility may be used, for example, in the following way:
> 为了得到所有正在运行的 nginx 进程，我们可能会使用到 ps 工具，比如，像下列的方式：

``` bash
ps -ef | grep nginx
```

For more information on sending signals to nginx, see [Controlling nginx](http://nginx.org/en/docs/control.html).
> 更多关于发送信号给 nginx，可以参考 nginx 控制。

### Configuration File’s Structure

nginx consists of modules which are controlled by directives specified in the configuration file. Directives are divided into simple directives and block directives. A simple directive consists of the name and parameters separated by spaces and ends with a semicolon (;). A block directive has the same structure as a simple directive, but instead of the semicolon it ends with a set of additional instructions surrounded by braces ({ and }). If a block directive can have other directives inside braces, it is called a context (examples: [events](http://nginx.org/en/docs/ngx_core_module.html#events), [http](http://nginx.org/en/docs/http/ngx_http_core_module.html#http), [server](http://nginx.org/en/docs/http/ngx_http_core_module.html#server), and [location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)).
> nginx 是由一些模块组成，我们一般在配置文件中使用一些具体的指令来控制它们。指令被分为简单指令和块级命令。一个简单的指令是由名字和参数组成，中间用空格分开，并以分号结尾。块级指令和简单指令一样有着类似的结构，但是末尾不是分号而是用 { 和 } 大括号包裹的额外指令集。如果一个块级指令的大括号里有其他指令，则它被叫做一个上下文（比如：events，http，server，和 location）。

Directives placed in the configuration file outside of any contexts are considered to be in the [main](http://nginx.org/en/docs/ngx_core_module.html) context. The `events` and `http` directives reside in the `main` context, `server` in `http`, and `location` in `server`.
> 在配置文件中，没有放在任何上下文中的指令都是处在主上下文中。events 和 http 的指令是放在主上下文中，server 放在 http 中, location 放在 server 中。

The rest of a line after the # sign is considered a comment.
> 以 # 开头的行，会被当做注释。

### Serving Static Content

An important web server task is serving out files (such as images or static HTML pages). You will implement an example where, depending on the request, files will be served from different local directories: `/data/www` (which may contain HTML files) and `/data/images` (containing images). This will require editing of the configuration file and setting up of a [server](http://nginx.org/en/docs/http/ngx_http_core_module.html#server) block inside the [http](http://nginx.org/en/docs/http/ngx_http_core_module.html#http) block with two [location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location) blocks.
> 一个重要的网络服务器的任务是处理文件（比如图片或者静态 HTML 文件）。这里，你会实践一个例子，文件会从不同的目录中映射（取决于请求）：/data/www（放置 HTML 文件）和 /data/images（放置图片）。这需要配置一下文件，将带有两个 location 的指令的 server 的块级命令放在 server 指令中。

First, create the `/data/www` directory and put an index.html file with any text content into it and create the `/data/images` directory and place some images in it.
> 首先，创建一个 /data/www 目录，然后放置一个事先写好内容的 index.html 文件。接着，创建一个 /data/images 目录，然后放置一些图片。

Next, open the configuration file. The default configuration file already includes several examples of the server block, mostly commented out. For now comment out all such blocks and start a new server block:
> 下一步，打开配置文件。默认的配置文件已经包含了一些关于 server 指令的样式，大多数情况下直接把他们给注释掉。现在，注释掉其他的区块，然后写一个新的 server 区块：

```bash
http {
   server {
   }
}
```

Generally, the configuration file may include several server blocks [distinguished](http://nginx.org/en/docs/http/request_processing.html) by ports on which they [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) to and by [server names](http://nginx.org/en/docs/http/server_names.html). Once nginx decides which server processes a request, it tests the URI specified in the request’s header against the parameters of the location directives defined inside the server block.
> 通常，该配置文件可能会包含多个 server 指令。这些 server 指令监听不同的端口和服务器名。一旦 nginx 决定哪个服务进程处理请求，它会根据在 server 块级指令中定义好的 location 指令的参数，来匹配请求头中指定的 URI。

Add the following location block to the server block:
> 将下列 location 指令添加到 server 指令中：

```bash
location / {
   root /data/www;
}
```

This location block specifies the "/" prefix compared with the URI from the request. For matching requests, the URI will be added to the path specified in the [root](http://nginx.org/en/docs/http/ngx_http_core_module.html#root) directive, that is, to `/data/www`, to form the path to the requested file on the local file system. If there are several matching `location` blocks nginx selects the one with the longest prefix. The location block above provides the shortest prefix, of length one, and so only if all other `location` blocks fail to provide a match, this block will be used.
> 该 location 指令相对于请求中的 URI 执行了 “/” 的前缀。为了匹配请求，URI 会被添加到 root 命令指定的路径后，即 /data/www，得到本地文件系统中请求文件的路径。如果，有几个 location 同时满足匹配要求，那么 nginx 会选择最长的前缀。上面的 location 提供了长度为 1 的前缀，所以，仅当其他的 location 匹配失败后，该指令才会使用。

Next, add the second location block:
> 接着，添加第二个 location 区块：

```bash
location /images/ {
   root /data;
}
```

It will be a match for requests starting with `/images/` (`location` / also matches such requests, but has shorter prefix).
> 它会匹配到以 /images/ 开头的请求（location / 也会匹配到该请求，只是前缀更短）

The resulting configuration of the `server` block should look like this:
> server 块级命令的配置结果如下：

```bash
server {
   location / {
       root /data/www;
   }

   location /images/ {
       root /data;
   }
}
```

This is already a working configuration of a server that listens on the standard port 80 and is accessible on the local machine at `http://localhost/`. In response to requests with URIs starting with `/images/`, the server will send files from the `/data/images` directory. For example, in response to the `http://localhost/images/example.png` request nginx will send the `/data/images/example.png` file. If such file does not exist, nginx will send a response indicating the 404 error. Requests with URIs not starting with `/images/` will be mapped onto the `/data/www` directory. For example, in response to the `http://localhost/some/example.html` request nginx will send the `/data/www/some/example.html` file.
> 这已经是一个可用的服务器配置，它监听标准的 80 端口并且可以在本地上通过 `http://localhost/` 访问。对于 URI 以 /images/ 开头的请求，服务器会从 /data/images 目录中，返回对应的文件。例如，nginx 会返回 /data/images/example.png 文件，当接收到 http://localhost/images/example.png 的请求响应时。如果该文件不存在，nginx 会返回一个 404 错误的响应。没有以 /images/ 开头的 URI 的请求，将会直接映射到 /data/www 目录中。比如，响应 http://localhost/some/example.html 的请求，nginx 会发送 /data/www/some/example.html 文件。

To apply the new configuration, start nginx if it is not yet started or send the reload signal to the nginx’s master process, by executing:
> 为了使用新的配置文件，如果还没开启 nginx 需要先开启，然后将重载信号发送给 nginx 的主进程，通过执行：

```bash
nginx -s reload
```

In case something does not work as expected, you may try to find out the reason in access.log and error.log files in the directory `/usr/local/nginx/logs` or `/var/log/nginx`.
> 如果你发现有些地方出了问题，你可以在 /usr/local/nginx/logs 或者 /var/log/nginx 目录下的 access.log 和 error.log 文件中，找到原因。

### Setting Up a Simple Proxy Server

One of the frequent uses of nginx is setting it up as a proxy server, which means a server that receives requests, passes them to the proxied servers, retrieves responses from them, and sends them to the clients.
> nginx 常常用来作为代理服务器，这代表着服务器接收请求，然后将它们传递给被代理服务器，得到请求的响应，再将它们发送给客户端。

We will configure a basic proxy server, which serves requests of images with files from the local directory and sends all other requests to a proxied server. In this example, both servers will be defined on a single nginx instance.
> 我们将配置一个基本的代理服务器，它会处理本地图片文件的请求并返回其他的请求给被代理的服务器。在这个例子中，两个服务器都会定义在一个 nginx 实例中。

First, define the proxied server by adding one more server block to the nginx’s configuration file with the following contents:
> 首先，通过在 nginx 配置文件中添加另一个 server 区块，来定义一个被代理的服务器，像下面的配置：

```bash
server {
   listen 8080;
   root /data/up1;

   location / {
   }
}
```

This will be a simple server that listens on the port 8080 (previously, the listen directive has not been specified since the standard port 80 was used) and maps all requests to the `/data/up1` directory on the local file system. Create this directory and put the `index.html` file into it. Note that the root directive is placed in the server context. Such `root` directive is used when the `location` block selected for serving a request does not include own `root` directive.
> 上面就是一个简单的服务器，它监听在 8080 端口（之前，listen 并没被定义，是因为默认监听的 80 端口）并且会映射所有的请求给 本地文件目录 /data/up1。创建该目录，然后添加 index.html 文件。注意，root 指令是放在 server 上下文中。当响应请求的 location 区块中，没有自己的 root 指令，上述的 root 指令才会被使用。

Next, use the server configuration from the previous section and modify it to make it a proxy server configuration. In the first `location` block, put the [proxy_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) directive with the protocol, name and port of the proxied server specified in the parameter (in our case, it is `http://localhost:8080`):
> 接着，使用前面章节中的 server 配置，然后将它改为一个代理服务配置。在第一个 location 区块中，放置已经添加被代理服务器的协议，名字和端口等参数的 proxy_pass 指令（在这里，就是 `http://localhost:8080`）:

```bash
server {
   location / {
       proxy_pass http://localhost:8080;
   }

   location /images/ {
       root /data;
   }
}
```

We will modify the second location block, which currently maps requests with the /images/ prefix to the files under the /data/images directory, to make it match the requests of images with typical file extensions. The modified location block looks like this:
> 我们将修改第二个 location 区块，现在它只会映射带有 /images/ 前缀的请求到 /data/images 目录下，我们将它修改为满足特定图片文件扩展名的请求。修改后的 location 指令如下：

```bash
location ~ \.(gif|jpg|png)$ {
   root /data/images;
}
```

The parameter is a regular expression matching all URIs ending with `.gif`, `.jpg`, or `.png`. A regular expression should be preceded with `~`. The corresponding requests will be mapped to the `/data/images` directory.
> 该参数是一个正则表达式，它会匹配所有以 .gif，.jpg 或者 .png 结尾的 URIs。一个正则表达式需要以 ~ 开头。匹配到的请求会被映射到 /data/images 目录下。

When nginx selects a location block to serve a request it first checks [location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location) directives that specify prefixes, remembering location with the longest prefix, and then checks regular expressions. If there is a match with a regular expression, nginx picks this location or, otherwise, it picks the one remembered earlier.
> 当 nginx 在选择 location 去响应一个请求时，它会先检测满足特定前缀的 location 指令，记住满足条件的最长前缀的 location，然后检测正则表达式。如果有一个正则的匹配的规则，nginx 会选择该 location，否则，会选择之前缓存的规则。

The resulting configuration of a proxy server will look like this:
> 最终，一个代理服务器的配置结果如下：

```bash
server {
   location / {
       proxy_pass http://localhost:8080/;
   }

   location ~ \.(gif|jpg|png)$ {
       root /data/images;
   }
}
```

This server will filter requests ending with `.gif`, `.jpg`, or `.png` and map them to the `/data/images` directory (by adding URI to the root directive’s parameter) and pass all other requests to the proxied server configured above.
> 该服务器会选择以 .gif，.jpg，或者 .png 结束的请求并且映射到 /data/images 目录（通过添加 URI 给 root 指令的参数），接着将其他所有的请求映射到上述被代理的服务器。

To apply new configuration, send the reload signal to nginx as described in the previous sections.
> 为了使用新的配置，像前几个章节描述的一样，需要向 nginx 发送重载信号。

There are many more directives that may be used to further configure a [proxy](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) connection.
> 这还有很多其他的指令，可以用于进一步配置代理连接。

### Setting Up FastCGI Proxying

nginx can be used to route requests to FastCGI servers which run applications built with various frameworks and programming languages such as PHP.

The most basic nginx configuration to work with a FastCGI server includes using the [fastcgi_pass](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass) directive instead of the proxy_pass directive, and [fastcgi_param](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_param) directives to set parameters passed to a FastCGI server. Suppose the FastCGI server is accessible on localhost:9000. Taking the proxy configuration from the previous section as a basis, replace the proxy_pass directive with the fastcgi_pass directive and change the parameter to localhost:9000. In PHP, the `SCRIPT_FILENAME` parameter is used for determining the script name, and the `QUERY_STRING` parameter is used to pass request parameters. The resulting configuration would be:

```bash
server {
   location / {
       fastcgi_pass  localhost:9000;
       fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
       fastcgi_param QUERY_STRING    $query_string;
   }

   location ~ \.(gif|jpg|png)$ {
       root /data/images;
   }
}
```

This will set up a server that will route all requests except for requests for static images to the proxied server operating on localhost:9000 through the `FastCGI` protocol.
