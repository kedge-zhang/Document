----  
tags: hexo
----
date: 2017-12-27 16:07:39
---
## 1.安装前提
    *Node.js
    *Git
## 2.部署
### 2.1 git
    yum install asciidoc xmlto curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker -y
    cd /usr/local/src && wget https://www.kernel.org/pub/software/scm/git/git-2.13.0.tar.gz --no-check-certificate
    tar fvx git-2.13.0.tar.gz && cd git-2.13.0 && make configure && ./configure && make all doc && make install install-doc install-html
    
### 2.2 node.js
    * 1.安装nvm
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.1/install.sh | bash
    source ~/.bashrc
    * 2.安装node
    nvm install stable

### 2.3 hexo

    * 1.安装hexo
    npm install -g hexo-cli
    * 2. path
    mkdir -p /data/hexo  ##部署路径
    * 3. hexo初始化
    hexo init
    * 4.安装依赖
    npm install
    * 5.生成hexo静态页面
    hexo generate
    * 6.启动本地服务
    hexo server
    * 7.主题
    # path:/data/hexo
    # 主题网站:https://hexo.io/themes
    # 找到theme:修改后面的参数，默认是landscape
    git clone https://github.com/iissnan/hexo-theme-next themes/next
    theme: next
    配置完成后保存 wq ，执行以下命令
    hexo clean   //清除缓存文件 (db.json) 和已生成的静态文件 (public)
    hexo g       //生成缓存和静态文件
    hexo d       //重新部署到服务器  

## 3. 整合 github
### 3.1 仓库格式：
    your-user-name.github.io
    PS： 
      your-user-name：GitHub 帐户名
      github.io		：固定格式
    Example：
    我的github 帐户名为 kedge，那么新建仓库名为kedge.github.io

### 3.2 配置免密钥
    ssh-keygen -t RSA -C "name@eamil.com"
    将公钥添加到github里即可
    
### 3.3 整合 hexo 和 github
    * 1.vim /data/hexo/_config.yml
    deploy:
    type: git
    repository: https://github.com/kedge/kedge.github.io.git
    branch: master
    #注意格式：在配置所有的_config.yml文件时（包括theme中的），在所有的冒号:后边都要加一个空格，否则执行hexo命令会报错
    * 2.生成静态页面
    hexo generate        或者：hexo g
    
### 3.4 此时若出现如下报错：
    * 1 ERROR Local hexo not found in ~/blog
        ERROR Try runing: 'npm install hexo --save'
        则执行命令：
        npm install hexo --save
    * 2 若无报错，自行忽略此步骤，然后执行以下步骤
        hexo deploy            或者：hexo d
    * 3 若步骤 2 报错，则执行：
        npm install hexo-deployer-git --save

## 4.文章发布
终端cd到/data/hexo文件夹下，执行如下命令新建文章，名为`postName.md`的文件会建在目录`/blog/source/_posts`下; 
######
       hexo new "postName"  
文章编辑完成后，终端cd到blog文件夹下，执行如下命令来发布:
#####
       hexo generate    //生成静态页面
       hexo deploy      //将文章部署到Github

     