---
layout: post
title: "gitlab迁移"
date: 2014-12-20 14:44:40 -0500
comments: true
categories: gitlab
---
gitlab 从 A 服务器迁移至 B 服务器

##一、在B服务器上部署gitlab环境

可参考gitlab官方的安装教程，上面介绍的已经很详细，在这里就不多说了

##二、打包A服务器上的数据

    cd /home/git/
    tar zcvf /bk/repositories.tar.gz ./repositories   #打包仓库文件
    mysqldump -uroot -pxxxx gitlabhq_production > /bk/gitlabhq_production.sql  #备份sql文件
    cp /home/git/.ssh/authorized_keys  /bk/authorized_keys #备份keys

    cd /bk
    scp repositories.tar.gz  gitlabhq_production.sql  authorized_keys root@B:/bk/ #复制到B服务器的/bk目录下

在B服务器上导入数据

    cd /bk/
    tar zxvf repositories.tar.gz
    /bin/cp -r repositories /home/git/  #导入仓库
    chown -R git.git /home/git/repositories/ #授权
    cat authorized_keys >> /home/git/.ssh/authorized_keys #导入keys
    mysql -uroot -pxxx gitlabhq_production < gitlabhq_production.sql #导入数据库

    cd /home/git/gitlab/
    sudo -u git -H bundle exec rake gitlab:import:repos RAILS_ENV=production
    sudo -u git -H bundle exec rake gitlab:satellites:create RAILS_ENV=production
    sudo chmod -R ug+rwX,o-rwx /home/git/repositories/
    sudo chmod -R ug-s /home/git/repositories/
    find /home/git/repositories/ -type d -print0 | sudo xargs -0 chmod g+s

检测
    
    sudo -u git -H bundle exec rake gitlab:check RAILS_ENV=production

检测没有问题的话就可以重启下gitlab，然后打开看看数据什么的都有了。


