

gitlab安装需求:https://docs.gitlab.com.cn/ce/install/requirements.html


gitlab配置: https://docs.gitlab.com.cn/omnibus/settings/configuration.html

全部版本升级文档：http://docs.gitlab.com/omnibus/update/README.html

安装runner文档：https://docs.gitlab.com.cn/runner/

注册RUNNERS https://docs.gitlab.com.cn/runner/register/index.html

https://coderwall.com/p/ltwmtg/gitlab-ci



> 有一个需求需要安装GitLab Runner, But原来gitlab版本是10.7， 因为使用GitLab Runner需要gitlab 9以上才行，所以打算升一次级。1,2,3,4分别是升级gitlab。



#### 1.第一步备份数据

```shell
sudo gitlab-rake gitlab:backup:create STRATEGY=copy
```



使用以上命令会在`/var/opt/gitlab/backups`目录下创建一个名称类似为`1393513186_gitlab_backup.tar`的压缩包, 这个压缩包就是Gitlab整个的完整部分, 其中开头的`1393513186`是备份创建的日期.

Gitlab 修改备份文件默认目录

你也可以通过修改`/etc/gitlab/gitlab.rb`来修改默认存放备份文件的目录(参考目录尾的gitlab配置连接):

```shell
gitlab_rails['backup_path'] = '/diskb/gitlab/backups/'
```

`/mnt/backups`修改为你想存放备份的目录即可, 修改完成之后使用`gitlab-ctl reconfigure`命令重载配置文件即可.



#### 2.第二步gitlab迁移数据

Storing Git data in an alternative directory 设置存储仓库数据的备用目录

默认情况下omnibus-gitlab 将仓库数据存储在 `/var/opt/gitlab/git-data`目录下，仓库存放在子目录 `repositories`里面。 以可以通过修改`/etc/gitlab/gitlab.rb` 的这一行来自定义 `git-data` 的父目录。

```shell
git_data_dirs({ "default" => { "path" => "/diskb/gitlab/git-data" } })

```

自GitLab 8.10开始,可以通过在`/etc/gitlab/gitlab.rb`文件中添加下面的几行配置， 来增加多个 git 数据存储目录。

```shell
git_data_dirs({
  "default" => { "path" => "/var/opt/gitlab/git-data" },
  "alternative" => { "path" => "/diskb/gitlab/git-data" }
})

```

**注意** 目标路径和其子路径必须`不能为软链接`。

运行 `sudo gitlab-ctl reconfigure` 使修改生效。

如果 `/var/opt/gitlab/git-data` 目录已经存在Git仓库数据， 你可以用下面的命令把数据迁移到新的位置:

```
# 准备迁移之前要停止GitLab服务，防止用户写入数据。
sudo gitlab-ctl stop

# 注意 'repositories'后面不带斜杠，而
# 'git-data'后面是有斜杠的。
sudo rsync -av /var/opt/gitlab/git-data/repositories /diskb/gitlab/git-data/

# 如果需要修复权限设置，
# 可运行下面的命令进行修复。
sudo gitlab-ctl reconfigure

# 再次检查下 /diskb/gitlab、git-data的目录. 正常情况应该有下面这个子目录:
# repositories

sudo ls /diskb/gitlab/git-data/

# 完工! 启动GitLab，验证下是否能
# 通过web访问Git仓库。
sudo gitlab-ctl start

```



### 3.使用Gitlab-ce国内镜像更新

如果不用国内镜像，可能会因为墙的原因下载不了。

首先信任 GitLab 的 GPG 公钥:

```
curl https://packages.gitlab.com/gpg.key 2> /dev/null | sudo apt-key add - &>/dev/null

```

再选择你的 Debian/Ubuntu 版本，文本框中内容写进 `/etc/apt/sources.list.d/gitlab-ce.list`

你的Debian/Ubuntu版本:  			Ubuntu 14.04 LTS 	

```
deb https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu trusty main

```

安装 gitlab-ce:

```
sudo apt-get update
sudo apt-get install gitlab-ce
```



### 4.升级PostgreSQL

你在升级gitlab-ce过程中，可能会遇到

```
 Upgrade failed. Retry the upgrade after upgrading your PostgreSQL version.

dpkg: error processing archive /var/cache/apt/archives/gitlab-ce10.2.4-ce.0amd64.deb (--unpack):

 subprocess new pre-installation script returned error exit status 1
```



首先查看PostgreSQL版本，需要 9.6+版本的才行

```
/opt/gitlab/embedded/bin/psql --version
# 如果是9.2.18 则升级psql
sudo gitlab-ctl pg-upgrade 
```









### 5.下载并安装GitLab Runner

?	官网强烈建议，RUNNER不和GitLab安装到同一台机器， 所以我把GItLab Runner安装到了越南服。

> ?	我们强烈建议不要在计划安装GitLab的计算机上安装GitLab Runner。 根据您决定如何配置GitLab Runner以及您在CI环境中使用哪些工具来练习您的应用程序，GitLab Runner可以消耗大量的可用内存。
>
> 如果您决定在同一台计算机上运行GitLab Runner和GitLab Rails应用程序，则以上可用的内存消耗计算将无效。
>
> 由于[安全原因](https://gitlab.com/gitlab-org/gitlab-runner/blob/master/docs/security/index.md) ，将所有内容安装在单台机器上也是不安全的 - 特别是当您计划使用GitLab Runner使用shell执行程序时。
>
> 如果您打算使用CI功能，我们建议为每个GitLab Runner使用一个单独的机器。

?	安装步骤看连接https://docs.gitlab.com.cn/runner/install/linux-repository.html



### 6.注册GitLab Runner

https://docs.gitlab.com.cn/runner/register/index.html

注册的步骤参考了下面链接

https://docs.gitlab.com/runner/register/

```shell
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://139.196.194.74:8800/
Please enter the gitlab-ci token for this runner:
m6nKsnbTg5sXXSG56MiR
Please enter the gitlab-ci description for this runner:
[iZ228wa8ursZ]: luacheck-executor           
Please enter the gitlab-ci tags for this runner (comma separated):
luacheck
Whether to run untagged builds [true/false]:
[false]: true            
Whether to lock the Runner to current project [true/false]:
[true]: true
Registering runner... succeeded                     runner=m6nKsnbT
Please enter the executor: docker-ssh, parallels, shell, ssh, docker+machine, kubernetes, docker, docker-ssh+machine, virtualbox:
docker
Please enter the default Docker image (e.g. ruby:2.1):
alpine:latest
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```

注意上面一个链接第九点

9.If you chose Docker as your executor, you'll be asked for the default image to be used for projects that do not define one in `.gitlab-ci.yml`:

### 7.安装docker, 安装LUA和luarocks和luacheck

因为注册GitLab Runner的时候选择的执行器是选择docker的，那么就准备把lua等文件安装到docker上.

默认镜像是mylua:latest

自己的仓库添加.gitlab-ci.yml文件

```
git add .gitlab-ci.yml
git commit -m "Add .gitlab-ci.yml"
git push origin master
```



