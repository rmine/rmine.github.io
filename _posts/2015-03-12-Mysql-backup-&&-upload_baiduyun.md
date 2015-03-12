---
layout: post
title: Mysql自动备份和自动上传百度云盘
categories: [mysql, bypy, bash, crontab， ubuntu]
---

当你意识到数据库备份的重要性时，往往措手不及。这是血的教训，不说了，都是泪～～～。为了避免重蹈覆辙，此次下定决心把备份做好。

基本的思路是：利用crontab对mysql进行定时物理备份，并且利用百度云盘的超大空间将备份文件自动上传到云盘。

### 1.mysql备份
---

这里我直接用的mysql物理备份，将整个数据库备份到dump文件中，因为我是准备每天备份，所以加上了时间戳，命令如下：
{% highlight bash %}
mysqldump -uroot -pxxxxxx my_database > /opt/mysql_backup/dumps/my_database_`date +%Y%m%d`.dump
{% endhighlight %}

为了可以方便之后的crontab调用，所以可以先放入shell脚本中，以`backup_mysql.sh`文件名为例子：
{% highlight bash %}
#创建脚本文件
touch backup_mysql.sh

#给上可执行权限
chmod +x backup_mysql.sh

#将以下内容放入文件中，其中的数据库名称自己更改
#!/bin/bash
mysqldump -uroot -pxxxxxx my_database > /opt/mysql_backup/dumps/my_database_`date +%Y%m%d`.dump
{% endhighlight %}

### 2.安装百度云/百度网盘Python客户端
---

这是一个百度云/百度网盘的Python客户端。具体的[github地址](https://github.com/houtianze/bypy)。选这个是因为本身ubuntu内置python，所以不需要额外的安装部署。

此工具通过`pip`工具安装，如果没有，请用一下命令安装：
{% highlight bash %}
sudo apt-get install python-pip python-dev
{% endhighlight %}

然后安装此工具用到的相应依赖：
{% highlight bash %}
sudo pip install requests
{% endhighlight %}

安装（有洁癖的可以指定全路径执行文件，比如我）：
{% highlight bash %}
git clone到任意目录。（为了运行方便，可以把bypy.py和bypygui.pyw拷贝至/usr/bin/目录）
{% endhighlight %}

运行（我这里是远程服务器，所以不考虑图形界面）：
{% highlight bash %}
cd 到clone的目录，运行： ./bypy.py 可见命令行支持的全部命令和参数。
{% endhighlight %}

我所需要的仅仅是上传功能，所以我需要了解upload功能。
{% highlight bash %}
#查看upload命令可以看到如下
upload [localpath] [remotepath] [ondup] - upload a file or directory (recursively)

#然后自己试试，在bypy的目录下
./bypy.py upload ./requirements.txt test/requirements.txt
{% endhighlight %}

这时候应该就能在云盘里看到了(注：云盘上传的根目录是/apps/bypy，这里看github主的说明是百度权限的问题)
![baiduyun](/img/2015-03-12_002.jpg)

另外，调用此工具还要授权一次，只要把工具给出的授权url拷贝到浏览器，然后把百度返回的授权码粘贴到控制台就可以了。此番也将这工具如何用搞清楚了，为后面的事情做准备。


### 3.利用shell脚本完成操作
---

虽然之前没有仔细学过shell语法，但是此次用到的东西应该不多，所以不会太难，边学编写，完成了如下的脚本。

主要功能有2点：

1. **上传备份好的mysql文件**
2. **将超过一个月以上的备份文件删除，节省空间。**

{% highlight bash %}
#创建脚本文件
touch upload.sh

#给上可执行权限
chmod +x upload.sh

#开始编辑文件
#!/bin/bash

#upload file with bypy
function upload_file(){
        folder="/opt/mysql_backup/dumps"
        filename="my_database_`date +%Y%m%d`.dump"
        filePath=$folder/$filename
        if [ ! -x "$folder" ]; then
                echo "[ERROR]["`date +%Y-%m-%d' '%H:%M:%S`"] $folder is not exist."
                mkdir $folder
        fi


        if [ ! -f $filePath ]; then
                echo "[ERROR]["`date +%Y-%m-%d' '%H:%M:%S`"] $folder/$filename not found."
        else
                echo "[INFO]["`date +%Y-%m-%d' '%H:%M:%S`"] $folder/$filename has been found. Start uploading ......"
                /opt/bypy/bypy.py upload "$filePath" "mysql_backup/$filename"
                echo "[INFO]["`date +%Y-%m-%d' '%H:%M:%S`"] Uploading end."
        fi

}

#delete file for more than a month to save my poor space:(
function delete_before_month(){
        folder="/opt/mysql_backup/dumps"
        beforeAMonth=$(date -d "-1 month" +%Y%m%d)

        if [ ! -d $folder ]; then
                echo "[INFO]["`date +%Y-%m-%d' '%H:%M:%S`"] Folder not found. Nothing to do."
        else
                echo "[INFO]["`date +%Y-%m-%d' '%H:%M:%S`"] Iteration is beginning ......"
                for file in $folder/*
                do
                        if [ -f $file  ]; then
                                extension=${file##*.}
                                if [ "dump" = $extension ]; then
                                        baseName=$(basename $file .dump)
                                        timeStr=${baseName##*_}
                                        if (( $timeStr <= $beforeAMonth )); then
                                                rm -rf $file
                                                echo "[INFO]["`date +%Y-%m-%d' '%H:%M:%S`"] Delete file $file."
                                        fi
                                else
                                        echo $file" is not a dump file."
                                fi
                        else
                                        echo $file" is not a file."
                        fi
                done


                echo "[INFO]["`date +%Y-%m-%d' '%H:%M:%S`"] Iteration end ......"
        fi

}

upload_file;
delete_before_month;
{% endhighlight %}

### 4.crontab完成定时任务
---

第一步，先让mysql备份好，第二步，将备份好的文件上传。所以配置mysql的备份时间应在前面，至于第二步应该相隔多久，自己手动备份估算下时间即可。

我的配置，每天晚上11点55备份，59分开始上传：
{% highlight bash %}
55 23 * * * /bin/bash -l -c '/opt/mysql_backup/backup_mysql.sh >> /opt/mysql_backup/log/cron_log.log 2>&1'

59 23 * * * /bin/bash -l -c '/opt/mysql_backup/upload.sh >> /opt/mysql_backup/log/upload_log.log 2>&1'
{% endhighlight %}

### 5.总结
---

有时候侥幸心理会让你跌的头破血流，所以在力所能及的情况下，还是把防护措施做的踏实一点好。