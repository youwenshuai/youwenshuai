---
title: gitlab部署
tags: gitlab
---

##### 配置阿里云的yum源

```sh
yum install curl policycoreutils openssh-server openssh-clients postfix  -y
systemctl start postfix
systemctl enable postfix
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | bash
yum -y install gitlab-ce
sed -i '/^external_url/s/gitlab.example.com/192.168.1.3/' /etc/gitlab/gitlab.rb
gitlab-ctl reconfigure        #初始化、启动服务
gitlab-ctl status
gitlab-ctl start               # 启动所有 gitlab 组件；
gitlab-ctl stop                # 停止所有 gitlab 组件；
gitlab-ctl restart             # 重启所有 gitlab 组件；
gitlab-ctl status              # 查看服务状态；
gitlab-ctl reconfigure         # 启动服务；
vim /etc/gitlab/gitlab.rb      # 修改默认的配置文件；
gitlab-rake gitlab:check SANITIZE=true --trace    # 检查gitlab；
gitlab-ctl tail                # 查看日志；
```

> 遇到502报错查看8080端口是否被占用
>
>  vim /etc/gitlab/gitlab.rb
>
> 更改端口 unicorn 
>
> 重新初始化gitlab-ctl reconfigure 
>
> 更改密码gitlab-rails console production
>
> user = User.where(id: 1).first 
>
> user.password=12345678
>
> user.password_confirmation=12345678
>

\-----------------------------------------------------------------

版本2

```sh
yum install -y curl policycoreutils-pythonopenssh-server
systemctl start firewalld
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
yum install postfix -y
systemctl start postfix
systemctl enable postfix
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-10.0.0-ce.0.el7.x86_64.rpm
yum -y  install policycoreutils-python
rpm -ivh gitlab-ce-10.0.0-ce.0.el7.x86_64.rpm
vim  /etc/gitlab/gitlab.rb
external_url 'http://localhost'
gitlab-ctl reconfigure
```

> 遇到502报错查看8080端口是否被占用
>
>  vim /etc/gitlab/gitlab.rb
>
> 更改端口 unicorn 
>
> 重新初始化gitlab-ctl reconfigure 
>
> 汉化
>
> 查看gitlab版本
>
> cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
>
> 10.0.0
>
> 下载对应的中文版
>
> wget https://gitlab.com/xhang/gitlab/-/archive/10-0-stable-zh/gitlab-10-0-stable-zh.zip
>
> unzip gitlab-10-0-stable-zh.zip
>
> cp -r /opt/gitlab/embedded/service/gitlab-rails{,.ori}
>
> /bin/cp -rf  /root/gitlab-10-0-stable-zh/* /opt/gitlab/embedded/service/gitlab-rails/
>
> 报错不用管，因为设置过root密码
>
> gitlab-ctl reconfigure 重新配置
>
> gitlab-ctl restart  重新启动
>

##### 汉化

> yum -y install git
>
> git clone https://gitlab.com/xhang/gitlab.git
>
> cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
>
> cat gitlab/VERSION
>
> cd /root/gitlab/
>
> git diff v10.0.0 v10.0.0-zh >/tmp/10.0.0-zh.diff
>
> yum install patch -y
>
> patch -d /opt/gitlab/embedded/service/gitlab-rails -p1 < /tmp/10.0.0-zh.diff
>
> gitlab-ctl restart
>
> gitlab-ctl reconfigure
>
> netstat -ntlp
>

