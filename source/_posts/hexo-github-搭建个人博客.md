---
title: hexo + github 搭建个人博客
date: 2018-04-02 11:56:46
tags: [hexo, Anisina]
categories: Hexo
---


现在hexo+github搭建个人博客的文章已经非常多了，写这篇文章的目的是因为喜欢Anisina这个主题，而这个主题的作者有一段时间没更新了，缺少一些功能。为了用这个主题添加自己想要的功能，所以就有了这篇博文。希望这边文章能帮助喜欢Anisina主题的小伙伴，少踩一点坑。


### 参考博客
[博客](https://www.jianshu.com/p/13e64c9e2295)
[Anisina主题](https://haojen.github.io/2017/05/09/Anisina-%E4%B8%AD%E6%96%87%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/#undefined)
[主题配置](https://www.jianshu.com/p/f054333ac9e6)
[Next主题配置](https://www.jianshu.com/p/f054333ac9e6)


### Hexo安装
#### 安装node.js
#### 安装Hexo
在terminal中执行以下命令
> $ sudo npm install -g hexo 

### 初始化自己的博客项目
#### 创建你用来保存博客的文件夹，比如：Blog  创建文件夹的命令如下
> $ mkdir Blog

#### cd到刚创建的文件夹中
> $ cd Blog

#### 初始化hexo
> $ hexo init
    
#### 安装hexo工程所需包
> $ npm install

#### 启动hexo服务
> $ hexo s

此时，浏览器中打开网址http://localhost:4000，能看到如下页面：
![此处输入图片的描述](https://ws3.sinaimg.cn/large/006tNc79gy1fp4hdtduzcj31kw0w47wh.jpg)



### 统计博客总字数

> $ npm install hexo-wordcount --save
```
<span class="post-count">博客全站共 <%= totalcount(site) %> 字</span>
```

### 统计博文字数以及阅读时间
同上安装插件,在post.ejs中加入以下代码
```
<span>字数: <%= wordcount(page.content) %></span>
<span class="post-count"> 阅读时长: <%= min2read(page.content) %> 分钟</span>
```

### 宠物
> npm install --save hexo-helper-live2d
> npm install {your model's package name} 


#### 系统自带宠物名
    live2d中自带的宠物模型
    
```
live2d-widget-model-chitose
live2d-widget-model-epsilon2_1
live2d-widget-model-gf
live2d-widget-model-haru/01 (use npm install --save live2d-widget-model-haru)
live2d-widget-model-haru/02 (use npm install --save live2d-widget-model-haru)
live2d-widget-model-haruto
live2d-widget-model-hibiki
live2d-widget-model-hijiki
live2d-widget-model-izumi
live2d-widget-model-koharu
live2d-widget-model-miku
live2d-widget-model-ni-j
live2d-widget-model-nico
live2d-widget-model-nietzsche
live2d-widget-model-nipsilon
live2d-widget-model-nito
live2d-widget-model-shizuku
live2d-widget-model-tororo
live2d-widget-model-tsumiki
live2d-widget-model-unitychan
live2d-widget-model-wanko
live2d-widget-model-z16
```
    
#### 配置宠物
    
    在_config.yml中添加以下配置
    
```
# #宠物
live2d:
  enable: true
  scriptFrom: local
  model:
    use: live2d-widget-model-wanko
  display:
    position: right
    width: 120
    height: 240
  mobile:
    show: false
```

### 动态背景
    只需要在 footer.ejs中添加以下代码即可
```
<!-- 动态背景 -->
<script type="text/javascript" src="//cdn.bootcss.com/canvas-nest.js/1.0.0/canvas-nest.min.js"></script>
```
    
### 文章结尾

在post.ejs中添加一下代码
```
  <!-- 文章结束 -->
    <% if(is_post()) { %>
        <div style="text-align:center;color: #ccc;font-size:14px;">-------------本文结束<i class="fa fa-paw"></i>感谢您的阅读-------------</div>
    <% } %>
    <br>
    <hr>
```


### 博文版权声明
#### 在hexo-theme-Anisina主题目录中，找到 source/css文件夹，新建文件copyright.styl,文件中写入一下内容
    
```
    .copyright {
      width: 85%;
      max-width: 45em;
      margin: 2.8em auto 0;
      padding: 0.5em 1.0em;
      border: 1px solid #d3d3d3;
      font-size: 0.93rem;
      line-height: 1.6em;
      word-break: break-all;
      background: rgba(255,255,255,0.4);
    }
    .copyright p{margin:0;}
    .copyright span {
      display: inline-block;
      width: 5.2em;
      color: #b5b5b5;
      font-weight: bold;
    }
    .copyright .raw {
      margin-left: 1em;
      width: 5em;
    }
    .copyright a {
      color: #808080;
      border-bottom:0;
    }
    .copyright a:hover {
      color: #a3d2a3;
      text-decoration: underline;
    }
    .copyright:hover .fa-clipboard {
      color: #000;
    }
    .copyright .post-url:hover {
      font-weight: normal;
    }
    .copyright .copy-path {
      margin-left: 1em;
      width: 1em;
      +mobile(){display:none;}
    }
    .copyright .copy-path:hover {
      color: #808080;
      cursor: pointer;
    }
```

#### 在head.ejs 中 添加一下代码

```
    <%- css('css/copyright') %>
```

####  在post.ejs中的*<%- page.content %>*这行代码下面加入以下代码

```
<!-- 版权声明 -->
<% if(is_post()) { %>
<div class="copyright">
    <script src="//cdn.bootcss.com/clipboard.js/1.5.10/clipboard.min.js"></script>
    
    <!-- JS库 sweetalert 可修改路径 -->
    <script src="https://cdn.bootcss.com/jquery/2.0.0/jquery.min.js"></script>
    <script src="https://unpkg.com/sweetalert/dist/sweetalert.min.js"></script>
    <p><span>本文标题:</span><a href="<%= url_for(page.path) %>"> <%= page.title %></a></p>
    <p><span>文章作者:</span><a href="/" title="访问 <%= theme.author %> 的个人博客"><%= config.author %></a></p>
    <p><span>发布时间:</span><%= page.date.format("YYYY年MM月DD日 - HH:MM") %></p>
    <p><span>最后更新:</span><%= page.updated.format("YYYY年MM月DD日 - HH:MM") %></p>
    <p><span>原始链接:</span><a href="<%= url_for(page.path) %>" title="<%= page.title %>"><%= page.permalink %></a>
      <span class="copy-path"  title="点击复制文章链接"><i class="fa fa-clipboard" data-clipboard-text="<%= page.permalink %>"  aria-label="复制成功！"></i></span>
    </p>
    <p><span>许可协议:</span><i class="fa fa-creative-commons"></i> <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank" title="Attribution-NonCommercial-NoDerivatives 4.0 International (CC BY-NC-ND 4.0)">署名-非商业性使用-禁止演绎 4.0 国际</a> 转载请保留原文链接及作者。</p>  
  </div>
  <script> 
      var clipboard = new Clipboard('.fa-clipboard');
        $(".fa-clipboard").click(function(){
        clipboard.on('success', function(){
          swal({   
            title: "",   
            text: '复制成功',
            icon: "success", 
            showConfirmButton: true
            });
          });
      });  
  </script>
<% } %>
```

### 集成来必力评论系统
        在你的 _config.yml 里添加use_livere: true ，然后注册一个来比力后，再配置livere_uid: 你的来比力 UID，然后 hexo clean && hexo deploy -g  附上[来必力官网](https://livere.com/)
        来必力UID获取方式如下: 登录->个人头像->管理页面->代码管理->一般网站，框起来的为UID
![](https://ws2.sinaimg.cn/large/006tNc79gy1fp5fs0fou2j310w0l5dju.jpg)


### 添加简书
在footer.ejs中  添加如下代码
```
 <% if (config.jianshu_username) { %>
    <li>
        <a target="_blank"  href="https://www.jianshu.com/users/<%= config.jianshu_username %>">
        <span class="fa-stack fa-lg">
        <i class="fa fa-circle fa-stack-2x"></i>
        <i class="fa fa-jianshu fa-stack-1x fa-inverse">简</i>
        </span>
        </a>
    </li>
 <% } %>
```

在_config.xml中添加如下代码
```
jianshu_username: ecf4f97c26fa # 你的简书ID
```

### daovoice即时聊天系统
#### 注册daovoice账号

点[这里][1]注册，注册后按以后步骤拿到你的appid![获取appid][2]

#### 添加配置
在_config.yml中添加一下配置
```
# daovoice 及时聊天
daovoice: true
daovoice_app_id: 7dfaf13b #这里填你的刚才获得的 app_id
```
#### 在head.esj模板中添加以下代码
```
<!-- daovoice -->
<% if(config.daovoice) { %>
<script>
(function(i,s,o,g,r,a,m){i["DaoVoiceObject"]=r;i[r]=i[r]||function(){(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;a.charset="utf-8";m.parentNode.insertBefore(a,m)})(window,document,"script",('https:' == document.location.protocol ? 'https:' : 'http:') + "//widget.daovoice.io/widget/0f81ff2f.js","daovoice")
  daovoice('init', {
      app_id: "<%= theme.daovoice_app_id%>"
    });
  daovoice('update');
</script>
<% } %>
```

### 增加打赏功能

#### 截图自己微信以及支付宝的收款款码
#### 在_config.yml中添加如下配置
```
#打赏二维码
qrcodes: [
  {
      img: https://ws1.sinaimg.cn/large/006tNc79gy1fp6mjdk6jmj30hc0hcmy3.jpg,
      text: 支付宝打赏
  },
  {
      img: https://ws1.sinaimg.cn/large/006tNc79gy1fp6mjdbp68j30hc0hcdh3.jpg,
      text: 微信打赏
  }
] 
```

#### 在post.ejs中添加一下代码
```
 <!-- 打赏 -->
 <% if(is_post()) { %>
    <% if(config.qrcodes) { %>
        <div class="qrcode">
            <!-- <h5 class="text-center">打赏</h5> -->
            <% config.qrcodes.forEach(function(code){ %>
            <div class="qrcode-item">
                <img src="<%= code.img %>" class="qrcode-item-img" />
                <div class="qrcode-item-text"><%= code.text %></div>
            </div>
            <% }); %>
        </div>
    <% } %>
<% } %>
```

#### 在source/css文件夹中新建qrcode.styl文件，该文件的内容如下
```
.qrcode {
    display: flex;
    width: 100%;
    flex-direction: row;
    justify-content: space-around;
    align-items: space-between;
    flex-wrap: wrap;
}

.qrcode-tem {
    text-align: center;
    justify-content: center;
    align-items: center;
}

.qrcode-item-img {
    // flex: 1;
    width: 250px;
    height: 250px;
    // height: 40%;
    min-width : 200px;
}

.qrcode-item-text {
    color: rgb(73, 177, 245);
    font-size: 14px;
    // align-content: center;
    text-align: center;
}
```

####  在head.esj中加入一下代码
```
    <!-- 打赏 -->
    <%- css('css/qrcode') %>
```


## Next主题
### 文字背景色使用

<span id="inline-blue"> 站点配置文件 </span>
<span id="inline-purple"> 主题配置文件 </span>
<span id="inline-yellow"> 站点配置文件 </span>
<span id="inline-green"> 主题配置文件 </span>

```
<span id="inline-blue"> 站点配置文件 </span>
<span id="inline-purple"> 主题配置文件 </span>
<span id="inline-yellow"> 站点配置文件 </span>
<span id="inline-green"> 主题配置文件 </span>
```


> if you have any questions, leave a message.

  [1]: http://dashboard.daovoice.io/
  [2]: https://ws2.sinaimg.cn/large/006tKfTcgy1fpuvr58pfbj30xi0nc7al.jpg