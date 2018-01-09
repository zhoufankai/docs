

gitlab��װ����:https://docs.gitlab.com.cn/ce/install/requirements.html


gitlab����: https://docs.gitlab.com.cn/omnibus/settings/configuration.html

ȫ���汾�����ĵ���http://docs.gitlab.com/omnibus/update/README.html

��װrunner�ĵ���https://docs.gitlab.com.cn/runner/

ע��RUNNERS https://docs.gitlab.com.cn/runner/register/index.html

https://coderwall.com/p/ltwmtg/gitlab-ci



> ��һ��������Ҫ��װGitLab Runner, Butԭ��gitlab�汾��10.7�� ��Ϊʹ��GitLab Runner��Ҫgitlab 9���ϲ��У����Դ�����һ�μ���1,2,3,4�ֱ�������gitlab��



#### 1.��һ����������

```shell
sudo gitlab-rake gitlab:backup:create STRATEGY=copy
```



ʹ�������������`/var/opt/gitlab/backups`Ŀ¼�´���һ����������Ϊ`1393513186_gitlab_backup.tar`��ѹ����, ���ѹ��������Gitlab��������������, ���п�ͷ��`1393513186`�Ǳ��ݴ���������.

Gitlab �޸ı����ļ�Ĭ��Ŀ¼

��Ҳ����ͨ���޸�`/etc/gitlab/gitlab.rb`���޸�Ĭ�ϴ�ű����ļ���Ŀ¼(�ο�Ŀ¼β��gitlab��������):

```shell
gitlab_rails['backup_path'] = '/diskb/gitlab/backups/'
```

`/mnt/backups`�޸�Ϊ�����ű��ݵ�Ŀ¼����, �޸����֮��ʹ��`gitlab-ctl reconfigure`�������������ļ�����.



#### 2.�ڶ���gitlabǨ������

Storing Git data in an alternative directory ���ô洢�ֿ����ݵı���Ŀ¼

Ĭ�������omnibus-gitlab ���ֿ����ݴ洢�� `/var/opt/gitlab/git-data`Ŀ¼�£��ֿ�������Ŀ¼ `repositories`���档 �Կ���ͨ���޸�`/etc/gitlab/gitlab.rb` ����һ�����Զ��� `git-data` �ĸ�Ŀ¼��

```shell
git_data_dirs({ "default" => { "path" => "/diskb/gitlab/git-data" } })

```

��GitLab 8.10��ʼ,����ͨ����`/etc/gitlab/gitlab.rb`�ļ����������ļ������ã� �����Ӷ�� git ���ݴ洢Ŀ¼��

```shell
git_data_dirs({
  "default" => { "path" => "/var/opt/gitlab/git-data" },
  "alternative" => { "path" => "/diskb/gitlab/git-data" }
})

```

**ע��** Ŀ��·��������·������`����Ϊ������`��

���� `sudo gitlab-ctl reconfigure` ʹ�޸���Ч��

��� `/var/opt/gitlab/git-data` Ŀ¼�Ѿ�����Git�ֿ����ݣ� ���������������������Ǩ�Ƶ��µ�λ��:

```
# ׼��Ǩ��֮ǰҪֹͣGitLab���񣬷�ֹ�û�д�����ݡ�
sudo gitlab-ctl stop

# ע�� 'repositories'���治��б�ܣ���
# 'git-data'��������б�ܵġ�
sudo rsync -av /var/opt/gitlab/git-data/repositories /diskb/gitlab/git-data/

# �����Ҫ�޸�Ȩ�����ã�
# �������������������޸���
sudo gitlab-ctl reconfigure

# �ٴμ���� /diskb/gitlab��git-data��Ŀ¼. �������Ӧ�������������Ŀ¼:
# repositories

sudo ls /diskb/gitlab/git-data/

# �깤! ����GitLab����֤���Ƿ���
# ͨ��web����Git�ֿ⡣
sudo gitlab-ctl start

```



### 3.ʹ��Gitlab-ce���ھ������

������ù��ھ��񣬿��ܻ���Ϊǽ��ԭ�����ز��ˡ�

�������� GitLab �� GPG ��Կ:

```
curl https://packages.gitlab.com/gpg.key 2> /dev/null | sudo apt-key add - &>/dev/null

```

��ѡ����� Debian/Ubuntu �汾���ı���������д�� `/etc/apt/sources.list.d/gitlab-ce.list`

���Debian/Ubuntu�汾:  			Ubuntu 14.04 LTS 	

```
deb https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu trusty main

```

��װ gitlab-ce:

```
sudo apt-get update
sudo apt-get install gitlab-ce
```



### 4.����PostgreSQL

��������gitlab-ce�����У����ܻ�����

```
 Upgrade failed. Retry the upgrade after upgrading your PostgreSQL version.

dpkg: error processing archive /var/cache/apt/archives/gitlab-ce10.2.4-ce.0amd64.deb (--unpack):

 subprocess new pre-installation script returned error exit status 1
```



���Ȳ鿴PostgreSQL�汾����Ҫ 9.6+�汾�Ĳ���

```
/opt/gitlab/embedded/bin/psql --version
# �����9.2.18 ������psql
sudo gitlab-ctl pg-upgrade 
```









### 5.���ز���װGitLab Runner

?	����ǿ�ҽ��飬RUNNER����GitLab��װ��ͬһ̨������ �����Ұ�GItLab Runner��װ����Խ�Ϸ���

> ?	����ǿ�ҽ��鲻Ҫ�ڼƻ���װGitLab�ļ�����ϰ�װGitLab Runner�� �����������������GitLab Runner�Լ�����CI������ʹ����Щ��������ϰ����Ӧ�ó���GitLab Runner�������Ĵ����Ŀ����ڴ档
>
> �����������ͬһ̨�����������GitLab Runner��GitLab RailsӦ�ó��������Ͽ��õ��ڴ����ļ��㽫��Ч��
>
> ����[��ȫԭ��](https://gitlab.com/gitlab-org/gitlab-runner/blob/master/docs/security/index.md) �����������ݰ�װ�ڵ�̨������Ҳ�ǲ���ȫ�� - �ر��ǵ����ƻ�ʹ��GitLab Runnerʹ��shellִ�г���ʱ��
>
> ���������ʹ��CI���ܣ����ǽ���Ϊÿ��GitLab Runnerʹ��һ�������Ļ�����

?	��װ���迴����https://docs.gitlab.com.cn/runner/install/linux-repository.html



### 6.ע��GitLab Runner

https://docs.gitlab.com.cn/runner/register/index.html

ע��Ĳ���ο�����������

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

ע������һ�����ӵھŵ�

9.If you chose Docker as your executor, you'll be asked for the default image to be used for projects that do not define one in `.gitlab-ci.yml`:

### 7.��װdocker, ��װLUA��luarocks��luacheck

��Ϊע��GitLab Runner��ʱ��ѡ���ִ������ѡ��docker�ģ���ô��׼����lua���ļ���װ��docker��.

Ĭ�Ͼ�����mylua:latest

�Լ��Ĳֿ����.gitlab-ci.yml�ļ�

```
git add .gitlab-ci.yml
git commit -m "Add .gitlab-ci.yml"
git push origin master
```



