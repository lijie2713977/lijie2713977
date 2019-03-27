# Hexo博客部署纪要
* 安装服务：
npm install hexo-cli -g
npm install hexo-server —save 

* 初始化项目
hexo init HexoJiadongpo
cd HexoJiadongpo

* 下载主题：
git clone https://github.com/theme-next/hexo-theme-next themes/next

npm install
hexo server

问题：
ERROR Deployer not found: git
npm install --save hexo-deployer-git


##系统设置 _config.yml文件

### 修改语言
根据主要所使用的主题对应的,如theme: next，/themes/next/languages下对应的中文
language: zh-CN


## Next主题设置
next/_config.yml配置相关修改

###默认是居中
修改为scheme: Mist。

### 显示对应的标题
menu下，打开注释即可。

### 去见面尾部的脚本
footer.swig

### 标题图标
favicon:
 small: /images/favicon-16x16-next.png
 medium: /images/favicon-32x32-next.png

### 侧边头像
sidebar-author.styl
avatar.gif

### 首页文章只显示预览
进入hexo博客项目的themes/next目录
用文本编辑器打开_config.yml文件
搜索"auto_excerpt",找到如下部分：
`auto_excerpt:
  enable: false
  length: 150`
把enable修改为true即可。


### 添加社交链接
social:
  GitHub: https://github.com/jiadongpo || github-square
   medium: /images/favicon-32x32-next.png
  其中图标去图标库：http://www.fontawesome.com.cn/icons找，不用带fa-即可
  
### 添加打赏
支持微信支付 、支付宝。编辑如下。把自己的收款图片放入对应目录。
reward_comment: 坚持原创技术分享，您的支持将鼓励我继续创作！
wechatpay: /images/wechatpay.jpg
alipay: /images/alipay.jpg


### ERROR Deployer not found: git
发布时出现错误：
bogon% hexo deploy
ERROR Deployer not found: git
解决
npm install --save hexo-deployer-git

### Cannot GET/xxxx
hexo s后访问localhost:4000出现错误Cannot GET/xxxx

解决：检查主题配置文件_config.yml。