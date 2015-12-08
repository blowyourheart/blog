---
layout: post
title:  "巧用nc管理memcached缓存"
date:   2015-12-06 15:57:41 +0800
categories: memcached nc
---
memcached作为成熟的缓存系统，被各大互联网公司广泛使用。memcached作为一个分布式的缓存系统，缓存的分片是通过客户端来做的。所以，如果要改动数据，就需要去每一个memcached实例上执行一遍这样的改动，才能确保最终改动了数据。

memcached提供了文本协议和二进制协议来操作缓存，利用文本协议我们可以方便的修改数据，举例如下：
缓存里有一堆实体 entity:1 entity:3 ...，现在根据运营或者数据的需要，需要更新某些实体。反映在memcached上就是把缓存的实体直接删除掉就可以了。假定文件
entityIds.txt存放了要删除的实体id，每行一个。可以使用脚本来生成要删除的命令，保存到cmd.txt文件中
{% highlight bash %}
cat ./entityIds.txt | while read LINE
    do
        echo "delete entity:$LINE"
    done 
{% endhighlight %}
                                                
假定现在有两个memcached实例，部署在 cache1, cache2两台机器上，端口号都是 11211，我们只需要执行一个简单的脚本就可以达到想要的目的。

{% highlight ruby %}
nc cache1 11211 < cmd.txt
nc cache2 11211 < cmd.txt
{% endhighlight %}

其实nc的功能还是很强大的，比如很多公司的线上环境都有跳板机来保障登录的安全性，传文件就必须先经过跳板机传输再传到对应的机器上，或者公司的op会提供一个传输文件的工具，如果线上机器的某些端口可以直接和开发机器连接，就可以用nc来传输文件了。
{% highlight bash %}
nc  -l 10000 > file.txt  #线上机器监听本机10000端口并把收到的内容存到本地file.txt文件。
nc online-server1 10000 < send_file.txt
{% endhighlight %}

传输完了以后用 md5sum 命令检查下是否是一样的就可以了。 和同事之间传输文件也可以使用这种方式方便的传输，尤其是在没有qq的时候这样最方便了，比scp, sftp命令还要快。  

