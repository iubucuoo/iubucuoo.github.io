# php内存占用过高



<!--more-->


[参考文章](https://blog.51cto.com/u_14142911/2459262)
php-fpm.conf就是php-fpm的配置文件，路径一般在`/etc`下
不清楚的可以通过命令行找到
查看php-fpm位置:
```liunx
ps -ef | grep 'php-fpm'
```
由于方便管理把php-fpm配置文件拆开了。在/etc/php-fpm.d的的`www.conf`内
配置重要的参数说明如下：
``` php
pm = dynamic #指定进程管理方式，有3种可供选择：static、dynamic和ondemand。
pm.max_children = 16 #static模式下创建的子进程数或dynamic模式下同一时刻允许最大的php-fpm子进程数量。
pm.start_servers = 10 #动态方式下的起始php-fpm进程数量。
pm.min_spare_servers = 8 #动态方式下服务器空闲时最小php-fpm进程数量。
pm.max_spare_servers = 16 #动态方式下服务器空闲时最大php-fpm进程数量。
pm.max_requests = 2000 #php-fpm子进程能处理的最大请求数。
pm.process_idle_timeout = 10s
request_terminate_timeout = 120
```
pm三种进程管理模式说明如下：

pm = static，始终保持一个固定数量的子进程，这个数由pm.max_children定义，这种方式很不灵活，也通常不是默认的。
pm = dynamic，启动时会产生固定数量的子进程（由pm.start_servers控制）可以理解成最小子进程数，而最大子进程数则由pm.max_children去控制，子进程数会在最大和最小数范围中变化。闲置的子进程数还可以由另2个配置控制，分别是pm.min_spare_servers和pm.max_spare_servers。如果闲置的子进程超出了pm.max_spare_servers，则会被杀掉。小于pm.min_spare_servers则会启动进程（注意，pm.max_spare_servers应小于pm.max_children）。

pm = ondemand，这种模式和pm = dynamic相反，把内存放在第一位，每个闲置进程在持续闲置了pm.process_idle_timeout秒后就会被杀掉，如果服务器长时间没有请求，就只会有一个php-fpm主进程。弊端是遇到高峰期或者如果pm.process_idle_timeout的值太短的话，容易出现504 Gateway Time-out错误，因此pm = dynamic和pm = ondemand谁更适合视实际情况而定。

## 解决php-fpm进程占用内存大问题

### 3.1 调整管理模式

static管理模式适合比较大内存的服务器，而dynamic则适合小内存的服务器，你可以设置一个pm.min_spare_servers和pm.max_spare_servers合理范围，这样进程数会不断变动。ondemand模式则更加适合微小内存，例如512MB或者256MB内存，以及对可用性要求不高的环境。

### 减少php-fpm进程数

如果你的VPS主机的内存被占用耗尽，可以检查一下你的php-fpm进程数，按照php-fpm进程数=内存/2/30来计算，1GB内存适合的php-fpm进程数为10-20之间，具体还得根据你的PHP加载的附加组件有关系。

### php-fpm配置示例

这里以1GB内存的VPS配置php-fpm为演示，实际操作来看设置数值还得根据服务器本身的性能、PHP等综合考虑。
```php
pm = dynamic #dynamic和ondemand适合小内存。
pm.max_children = 15 #static模式下生效，dynamic不生效。
pm.start_servers = 8 #dynamic模式下开机的进程数量。
pm.min_spare_servers = 6 #dynamic模式下最小php-fpm进程数量。
pm.max_spare_servers = 15 #dynamic模式下最大php-fpm进程数量。
```

## 解决php-fpm进程不释放内存问题

上面通过减少php-fpm进程总数来达到减少php-fpm内存占用的问题，实际使用过程中发现php-fpm进程还存长期占用内存而不释放的问题。解决的方法就是减少pm.max_requests数。

最大请求数max_requests，即当一个 PHP-CGI 进程处理的请求数累积到 max_requests 个后，自动重启该进程，这样达到了释放内存的目的了。以1GB内存的VPS主机设置为例（如果你设置的数值没有达到释放内存可以继续调低）：

```undefined
pm.max_requests = 500 
```

当php-fpm进程达到了pm.max_requests设定的数值后，就会重启该进程，从而释放内存。
