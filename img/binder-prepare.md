<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta http-equiv="Cache-Control" content="max-age=36000" />
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="author" content="gityuan">
    <meta name="keyword"  content="袁辉辉, Gityuan, Android博客, Android源码, Flutter博客，Flutter源码">
    <meta name="description" content="袁辉辉, Gityuan, Android博客, Android源码, Flutter博客，Flutter源码">
    <meta name="baidu-site-verification" content="LToTaY8RGk" />
    <link rel="shortcut icon" href="/images/favicon.ico"/>
    <title>Binder系列—开篇 - Gityuan博客 | 袁辉辉的技术博客</title>

    <link rel="canonical" href="http://gityuan.com/2015/10/31/binder-prepare/">
    
    <!-- Bootstrap Core CSS -->
    <link rel="stylesheet" href="/css/bootstrap.min.css">
    
    <!-- Custom CSS -->
    <link rel="stylesheet" href="/css/hux-blog.min.css">
    
    <!-- Pygments Github CSS -->
    <link rel="stylesheet" href="/css/syntax.css">
    
    <!-- Custom Fonts -->
    <!-- change font-awesome CDN to qiniu -->
    <link href="http://cdn.staticfile.org/font-awesome/4.2.0/css/font-awesome.min.css" rel="stylesheet" type="text/css">
    
    <!-- add highlight -->
    <link rel="stylesheet" href="/css/androidstudio.css">
    	<script type="text/javascript" src="/js/highlight.pack.js"></script>
    	<script>hljs.initHighlightingOnLoad();</script>

</head>


<!-- hack iOS CSS :active style -->
<body ontouchstart="">

    <!-- Navigation -->
<nav class="navbar navbar-default navbar-custom navbar-fixed-top">
    <div class="container-fluid">
        <!-- Brand and toggle get grouped for better mobile display -->
        <div class="navbar-header page-scroll">
            <button type="button" class="navbar-toggle">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand" href="/">Gityuan</a>
        </div>

        <!-- Collect the nav links, forms, and other content for toggling -->
        <div id="huxblog_navbar">
            <div class="navbar-collapse">
                <ul class="nav navbar-nav navbar-right">
                    <li><a href="/">首页</a></li>
                    <li><a href="/archive">目录</a></li>
                    <li><a href="/job">招聘</a></li>
                    <li><a href="/talk">留言区</a></li>
                    <li><a href="/about">关于我</a></li>
                    <li><a href="/friends">链接</a></li>
                    <!--<li><a href="/tags">标签</a></li>-->
                </ul>
            </div>
        </div>
        <!-- /.navbar-collapse -->
    </div>
    <!-- /.container -->
</nav>
<script>
    // Drop Bootstarp low-performance Navbar
    // Use customize navbar with high-quality material design animation
    // in high-perf jank-free CSS3 implementation
    var $body   = document.body;
    var $toggle = document.querySelector('.navbar-toggle');
    var $navbar = document.querySelector('#huxblog_navbar');
    var $collapse = document.querySelector('.navbar-collapse');

    var __HuxNav__ = {
        close: function(){
            $navbar.className = " ";
            // wait until animation end.
            setTimeout(function(){
                // prevent frequently toggle
                if($navbar.className.indexOf('in') < 0) {
                    $collapse.style.height = "0px"
                }
            },400)
        },
        open: function(){
            $collapse.style.height = "auto"
            $navbar.className += " in";
        }
    }
    
    // Bind Event
    $toggle.addEventListener('click', function(e){
        if ($navbar.className.indexOf('in') > 0) {
            __HuxNav__.close()
        }else{
            __HuxNav__.open()
        }
    })
    
    /**
     * Since Fastclick is used to delegate 'touchstart' globally
     * to hack 300ms delay in iOS by performing a fake 'click',
     * Using 'e.stopPropagation' to stop 'touchstart' event from
     * $toggle/$collapse will break global delegation.
     *
     * Instead, we use a 'e.target' filter to prevent handler
     * added to document close HuxNav.
     *
     * Also, we use 'click' instead of 'touchstart' as compromise
     */
    document.addEventListener('click', function(e){
        if(e.target == $toggle) return;
        if(e.target.className == 'icon-bar') return;
        __HuxNav__.close();
    })
</script>


    <!-- _posts目录文章布局-->

<!-- Post Header -->
<style type="text/css">
    header.intro-header{
        position: relative;
        background-image: url('/');
    }

​    
</style>
<header class="intro-header" >
    <div class="header-mask"></div>
    <div class="container">
        <div class="row">
            <div class="col-lg-8 col-lg-offset-2 col-md-10 col-md-offset-1">
                <div class="post-heading">
                    <div class="tags">

                        <a target="_blank" class="tag" href="/tags/#android" title="android">android</a>
                        
                        <a target="_blank" class="tag" href="/tags/#binder" title="binder">binder</a>
                        
                        <a target="_blank" class="tag" href="/tags/#ipc" title="ipc">ipc</a>
                        
                    </div>
                    <h1>Binder系列—开篇</h1>


​                    
                    <h2 class="subheading"></h2>
                    
                    <span class="meta">Posted by Gityuan on October 31, 2015</span>
                </div>
            </div>
        </div>
    </div>
</header>

<!-- Post Content -->
<article>
    <div class="container" style="overflow:hidden">
        <div class="row">

    <!-- Post Container -->
            <div class="
                col-lg-8 col-lg-offset-2
                col-md-10 col-md-offset-1
                post-container">
    
    							<blockquote>
  <p>基于Android 6.0的源码剖析</p>
</blockquote>

<h2 id="一概述">一、概述</h2>
<p>Android系统中，每个应用程序是由Android的<code class="language-plaintext highlighter-rouge">Activity</code>，<code class="language-plaintext highlighter-rouge">Service</code>，<code class="language-plaintext highlighter-rouge">Broadcast</code>，<code class="language-plaintext highlighter-rouge">ContentProvider</code>这四剑客的中一个或多个组合而成，这四剑客所涉及的多进程间的通信底层都是依赖于Binder IPC机制。例如当进程A中的Activity要向进程B中的Service通信，这便需要依赖于Binder IPC。不仅于此，整个Android系统架构中，大量采用了Binder机制作为IPC（进程间通信）方案，当然也存在部分其他的IPC方式，比如Zygote通信便是采用socket。</p>

<p>Binder作为Android系统提供的一种IPC机制，无论从事系统开发还是应用开发，都应该有所了解，这是Android系统中最重要的组成，也是最难理解的一块知识点，错综复杂。要深入了解Binder机制，最好的方法便是阅读源码，借用Linux鼻祖Linus Torvalds曾说过的一句话：<code class="language-plaintext highlighter-rouge">Read The Fucking Source Code</code>。</p>

<h2 id="二-binder">二、 Binder</h2>

<h3 id="21-ipc原理">2.1 IPC原理</h3>

<p>从进程角度来看IPC机制</p>

<p><img src="/images/binder/prepare/binder_interprocess_communication.png" alt="binder_interprocess_communication" /></p>

<p>每个Android的进程，只能运行在自己进程所拥有的虚拟地址空间。对应一个4GB的虚拟地址空间，其中3GB是用户空间，1GB是内核空间，当然内核空间的大小是可以通过参数配置调整的。对于用户空间，不同进程之间彼此是不能共享的，而内核空间却是可共享的。Client进程向Server进程通信，恰恰是利用进程间可共享的内核内存空间来完成底层通信工作的，Client端与Server端进程往往采用ioctl等方法跟内核空间的驱动进行交互。</p>

<h3 id="22-binder原理">2.2 Binder原理</h3>

<p>Binder通信采用C/S架构，从组件视角来说，包含Client、Server、ServiceManager以及binder驱动，其中ServiceManager用于管理系统中的各种服务。架构图如下所示：</p>

<p><img src="/images/binder/prepare/IPC-Binder.jpg" alt="ServiceManager" /></p>

<p>可以看出无论是注册服务和获取服务的过程都需要ServiceManager，需要注意的是此处的Service Manager是指Native层的ServiceManager（C++），并非指framework层的ServiceManager(Java)。ServiceManager是整个Binder通信机制的大管家，是Android进程间通信机制Binder的守护进程，要掌握Binder机制，首先需要了解系统是如何首次<a href="http://gityuan.com/2015/11/07/binder-start-sm/">启动Service Manager</a>。当Service Manager启动之后，Client端和Server端通信时都需要先<a href="http://gityuan.com/2015/11/08/binder-get-sm/">获取Service Manager</a>接口，才能开始通信服务。</p>

<p>图中Client/Server/ServiceManage之间的相互通信都是基于Binder机制。既然基于Binder机制通信，那么同样也是C/S架构，则图中的3大步骤都有相应的Client端与Server端。</p>

<ol>
  <li><strong><a href="http://gityuan.com/2015/11/14/binder-add-service/">注册服务(addService)</a></strong>：Server进程要先注册Service到ServiceManager。该过程：Server是客户端，ServiceManager是服务端。</li>
  <li><strong><a href="http://gityuan.com/2015/11/15/binder-get-service/">获取服务(getService)</a></strong>：Client进程使用某个Service前，须先向ServiceManager中获取相应的Service。该过程：Client是客户端，ServiceManager是服务端。</li>
  <li><strong>使用服务</strong>：Client根据得到的Service信息建立与Service所在的Server进程通信的通路，然后就可以直接与Service交互。该过程：client是客户端，server是服务端。</li>
</ol>

<p>图中的Client,Server,Service Manager之间交互都是虚线表示，是由于它们彼此之间不是直接交互的，而是都通过与<a href="http://gityuan.com/2015/11/01/binder-driver/">Binder驱动</a>进行交互的，从而实现IPC通信方式。其中Binder驱动位于内核空间，Client,Server,Service Manager位于用户空间。Binder驱动和Service Manager可以看做是Android平台的基础架构，而Client和Server是Android的应用层，开发人员只需自定义实现client、Server端，借助Android的基本平台架构便可以直接进行IPC通信。</p>

<h3 id="23-cs模式">2.3 C/S模式</h3>

<p>BpBinder(客户端)和BBinder(服务端)都是Android中Binder通信相关的代表，它们都从IBinder类中派生而来，关系图如下：</p>

<p><img src="/images/binder/prepare/Ibinder_classes.jpg" alt="Binder关系图" /></p>

<ul>
  <li>client端：BpBinder.transact()来发送事务请求；</li>
  <li>server端：BBinder.onTransact()会接收到相应事务。</li>
</ul>

<h2 id="三-提纲">三、 提纲</h2>

<p>在后续的Binder源码分析过程中所涉及的源码，会有部分的精简，主要是去掉所有log输出语句，已减少代码篇幅过于长。通过前面的介绍，下面罗列一下关于Binder系列文章的提纲：</p>

<ul>
  <li><a href="http://gityuan.com/2015/11/01/binder-driver/">Binder系列1—Binder Driver初探</a></li>
  <li><a href="http://gityuan.com/2015/11/02/binder-driver-2/">Binder系列2—Binder Driver再探</a></li>
  <li><a href="http://gityuan.com/2015/11/07/binder-start-sm/">Binder系列3—启动Service Manager</a></li>
  <li><a href="http://gityuan.com/2015/11/08/binder-get-sm/">Binder系列4—获取Service Manager</a></li>
  <li><a href="http://gityuan.com/2015/11/14/binder-add-service/">Binder系列5—注册服务(addService)</a></li>
  <li><a href="http://gityuan.com/2015/11/15/binder-get-service/">Binder系列6—获取服务(getService)</a></li>
  <li><a href="http://gityuan.com/2015/11/21/binder-framework/">Binder系列7—framework层分析</a></li>
  <li><a href="http://gityuan.com/2015/11/22/binder-use/">Binder系列8—如何使用Binder</a></li>
  <li><a href="http://gityuan.com/2015/11/23/binder-aidl/">Binder系列9—如何使用AIDL</a></li>
  <li><a href="http://gityuan.com/2015/11/28/binder-summary/">Binder系列10—总结</a></li>
</ul>

<p>文章是从底层驱动往上层写的，这并不适合大家的理解，建议读者还是从上层往底层看。下面说说这个系列文章之间的彼此联系，也是对你阅读顺序的一个建议，更好的建议，大家可以上微博跟<strong><a href="http://weibo.com/gityuan">@Gityuan</a></strong>，或许邮件跟我进行交流与反馈：</p>

<p>首先阅读<a href="http://gityuan.com/2015/11/14/binder-add-service/">Binder系列5—注册服务(addService)</a>和<a href="http://gityuan.com/2015/11/15/binder-get-service/">Binder系列6—获取服务(getService)</a>，这两个过程都需要于ServiceManager打交道，那么这两个过程在开始之前都需要<a href="http://gityuan.com/2015/11/08/binder-get-sm/">Binder系列4—获取Service Manager</a>，既然要获取Service Manager，那么就需要先<a href="http://gityuan.com/2015/11/07/binder-start-sm/">Binder系列3—启动Service Manager</a>。在看Binder服务的注册和获取这两个过程中，不断追溯下去，最终调用到底层Binder底层驱动，这时需要了解<a href="http://gityuan.com/2015/11/01/binder-driver/">Binder系列1—Binder Driver初探</a>和<a href="http://gityuan.com/2015/11/02/binder-driver-2/">Binder系列2—Binder Driver再探</a>。</p>

<p>看完Binder系列1~系列6，那么对Binder的整个流程会有一个比较清晰的认知，这还只是停留在Native层(C/C++)。接下来，可以看看上层<a href="http://gityuan.com/2015/11/21/binder-framework/">Binder系列7—framework层分析</a>的Binder架构情况，Java层 Binder架构的核心逻辑都是交由Native架构来完成，更多的是对Binder的一个封装过程，只有真正理解了Native层Binder架构，才能算掌握的Binder。</p>

<p>前面的这些都是讲述Binder整个流程以及原理，再接下来你可能想要自己写一套Binder的C/S架构服务。如果你是<strong>系统工程师</strong>可能会比较关心Native层和framework层分别该如何实现自己的自定义的Binder通信服务，见<a href="http://gityuan.com/2015/11/22/binder-use/">Binder系列8—如何使用Binder</a>；如果你是<strong>应用开发工程师</strong>则应该更关心App是如何使用Binder的，那么可以查看文章<a href="http://gityuan.com/2015/11/23/binder-aidl/">Binder系列9—如何使用AIDL</a>。</p>

<p>最后是对Binder的一个简单总结<a href="http://gityuan.com/2015/11/28/binder-summary/">Binder系列10—总结</a>。</p>

<h2 id="四-源码目录">四. 源码目录</h2>
<p>从上之下, 整个Binder架构所涉及的总共有以下5个目录:</p>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/framework/base/core/java/               (Java)
/framework/base/core/jni/                (JNI)
/framework/native/libs/binder            (Native)
/framework/native/cmds/servicemanager/   (Native)
/kernel/drivers/staging/android          (Driver)
</code></pre></div></div>

<h4 id="41-java-framework">4.1 Java framework</h4>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/framework/base/core/java/android/os/  
    - IInterface.java
    - IBinder.java
    - Parcel.java
    - IServiceManager.java
    - ServiceManager.java
    - ServiceManagerNative.java
    - Binder.java  


/framework/base/core/jni/    
    - android_os_Parcel.cpp
    - AndroidRuntime.cpp
    - android_util_Binder.cpp (核心类)
</code></pre></div></div>

<h4 id="42-native-framework">4.2 Native framework</h4>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/framework/native/libs/binder         
    - IServiceManager.cpp
    - BpBinder.cpp
    - Binder.cpp
    - IPCThreadState.cpp (核心类)
    - ProcessState.cpp  (核心类)

/framework/native/include/binder/
    - IServiceManager.h
    - IInterface.h

/framework/native/cmds/servicemanager/
    - service_manager.c
    - binder.c
</code></pre></div></div>

<h4 id="43-kernel">4.3 Kernel</h4>

<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/kernel/drivers/staging/android/
    - binder.c
    - uapi/binder.h
</code></pre></div></div>

								<hr>
								<font color="#004B97"><strong>微信公众号 </strong></font>
<a target="_blank" href="http://gityuan.com/images/about-me/gityuan_weixin_logo.png">
	<font color="#f57e42"><strong>Gityuan</strong></font>
</a>

<font color="#004B97"><strong> | 微博</strong></font>
<a target="_blank" href="http://weibo.com/gityuan">
	<font color="#f57e42"><strong>weibo.com/gityuan</strong></font>
</a>

<font color="#004B97"><strong> | 博客</strong></font>
<a target="_blank" href="http://gityuan.com/talk/">
	<font color="#f57e42"><strong>留言区交流</strong></font>
</a>
<!-- <img src="/images/about-me/gityuan_weixin_logo.png" alt="gityuan">-->

                <hr>


                <ul class="pager">
                    
                    <li class="previous">
                        <a target="_blank" href="/2015/10/30/kernel-memory/" data-toggle="tooltip" data-placement="top" title="Linux内存管理">
                        上一篇<br>
                        <span>Linux内存管理</span>
                        </a>
                    </li>


​                    
                    <li class="next">
                        <a target="_blank" href="/2015/11/01/binder-driver/" data-toggle="tooltip" data-placement="top" title="Binder系列1—Binder Driver初探">
                        下一篇<br>
                        <span>Binder系列1—Binder Driver初探</span>
                        </a>
                    </li>
                    
                </ul>


​                

            </div>
    
    <!-- Side Catalog Container -->
        
            <div class="
                col-lg-2 col-lg-offset-0
                visible-lg-block
                sidebar-container
                catalog-container">
                <div class="side-catalog">
                    <hr class="hidden-sm hidden-xs">
                    <h5>
                        <a class="catalog-toggle" href="#">CATALOG</a>
                    </h5>
                    <ul class="catalog-body"></ul>
                </div>
            </div>


    <!-- Sidebar Container -->
            <div class="
                col-lg-8 col-lg-offset-2
                col-md-10 col-md-offset-1
                sidebar-container">
    
                <!-- Featured Tags -->
                
                <section>
                    <hr class="hidden-sm hidden-xs">
                    <h5><a target="_blank" href="/tags/">标签</a></h5>
                    <div class="tags">
                      
                            <a target="_blank" href="/tags/#android" title="android" rel="153">
                                    android
                            </a>
                      
                            <a target="_blank" href="/tags/#组件系列" title="组件系列" rel="19">
                                    组件系列
                            </a>
                      
                            <a target="_blank" href="/tags/#else" title="else" rel="3">
                                    else
                            </a>
                      
                            <a target="_blank" href="/tags/#debug" title="debug" rel="19">
                                    debug
                            </a>
                      
                            <a target="_blank" href="/tags/#权限" title="权限" rel="2">
                                    权限
                            </a>
                      
                            <a target="_blank" href="/tags/#web" title="web" rel="2">
                                    web
                            </a>
                      
                            <a target="_blank" href="/tags/#tool" title="tool" rel="12">
                                    tool
                            </a>
                      
                            <a target="_blank" href="/tags/#java" title="java" rel="12">
                                    java
                            </a>
                      
                            <a target="_blank" href="/tags/#performance" title="performance" rel="4">
                                    performance
                            </a>
                      
                            <a target="_blank" href="/tags/#app" title="app" rel="2">
                                    app
                            </a>
                      
                            <a target="_blank" href="/tags/#algorithm" title="algorithm" rel="1">
                                    algorithm
                            </a>
                      
                            <a target="_blank" href="/tags/#进程系列" title="进程系列" rel="13">
                                    进程系列
                            </a>
                      
                            <a target="_blank" href="/tags/#虚拟机" title="虚拟机" rel="1">
                                    虚拟机
                            </a>
                      
                            <a target="_blank" href="/tags/#memory" title="memory" rel="5">
                                    memory
                            </a>
                      
                            <a target="_blank" href="/tags/#jvm" title="jvm" rel="5">
                                    jvm
                            </a>
                      
                            <a target="_blank" href="/tags/#linux" title="linux" rel="11">
                                    linux
                            </a>
                      
                            <a target="_blank" href="/tags/#binder" title="binder" rel="19">
                                    binder
                            </a>
                      
                            <a target="_blank" href="/tags/#ipc" title="ipc" rel="3">
                                    ipc
                            </a>
                      
                            <a target="_blank" href="/tags/#handler" title="handler" rel="3">
                                    handler
                            </a>
                      
                            <a target="_blank" href="/tags/#process" title="process" rel="6">
                                    process
                            </a>
                      
                            <a target="_blank" href="/tags/#power" title="power" rel="1">
                                    power
                            </a>
                      
                            <a target="_blank" href="/tags/#系统启动" title="系统启动" rel="6">
                                    系统启动
                            </a>
                      
                            <a target="_blank" href="/tags/#AMS" title="AMS" rel="2">
                                    AMS
                            </a>
                      
                            <a target="_blank" href="/tags/#PMS" title="PMS" rel="1">
                                    PMS
                            </a>
                      
                            <a target="_blank" href="/tags/#自学编程" title="自学编程" rel="1">
                                    自学编程
                            </a>
                      
                            <a target="_blank" href="/tags/#stability" title="stability" rel="7">
                                    stability
                            </a>
                      
                            <a target="_blank" href="/tags/#组件" title="组件" rel="3">
                                    组件
                            </a>
                      
                            <a target="_blank" href="/tags/#art" title="art" rel="2">
                                    art
                            </a>
                      
                            <a target="_blank" href="/tags/#graphic" title="graphic" rel="1">
                                    graphic
                            </a>
                      
                            <a target="_blank" href="/tags/#NativeDebug" title="NativeDebug" rel="3">
                                    NativeDebug
                            </a>
                      
                            <a target="_blank" href="/tags/#实战案例" title="实战案例" rel="7">
                                    实战案例
                            </a>
                      
                            <a target="_blank" href="/tags/#flutter" title="flutter" rel="22">
                                    flutter
                            </a>
                      
                    </div>
                </section>


                <!-- Friends Blog -->
                <!--
                 -->
            </div>
        </div>
    </div>
</article>






<!-- async load function -->
<script>
    function async(u, c) {
      var d = document, t = 'script',
          o = d.createElement(t),
          s = d.getElementsByTagName(t)[0];
      o.src = u;
      if (c) { o.addEventListener('load', function (e) { c(null, e); }, false); }
      s.parentNode.insertBefore(o, s);
    }
</script>

<script>
    async("http://cdn.bootcss.com/anchor-js/1.1.1/anchor.min.js",function(){
        anchors.options = {
          visible: 'always',
          placement: 'right',
          icon: '#'
        };
        anchors.add().remove('.intro-header h1').remove('.subheading').remove('.sidebar-container h5');
    })
</script>
<style>
    /* place left on bigger screen */
    @media all and (min-width: 800px) {
        .anchorjs-link{
            position: absolute;
            left: -0.75em;
            font-size: 1.1em;
            margin-top : -0.1em;
        }
    }
</style>



    <!-- Footer -->
<footer>
    <div class="container">
        <div class="row">
            <div class="col-lg-8 col-lg-offset-2 col-md-10 col-md-offset-1">
                <ul class="list-inline text-center">

                    <li>
                        <a target="_blank" href="http://weibo.com/gityuan">
                            <span class="fa-stack fa-lg">
                                <i class="fa fa-circle fa-stack-2x"></i>
                                <i class="fa fa-weibo fa-stack-1x fa-inverse"></i>
                            </span>
                        </a>
                    </li>


​                    
                    <li>
                        <a target="_blank" href="https://www.zhihu.com/people/gityuan">
                            <span class="fa-stack fa-lg">
                                <i class="fa fa-circle fa-stack-2x"></i>
                                <i class="fa  fa-stack-1x fa-inverse">知</i>
                            </span>
                        </a>
                    </li>


​										
​                    
​                    
​                    
​                    
                    <li>
                        <a target="_blank" href="https://www.linkedin.com/in/gityuan">
                            <span class="fa-stack fa-lg">
                                <i class="fa fa-circle fa-stack-2x"></i>
                                <i class="fa fa-linkedin fa-stack-1x fa-inverse"></i>
                            </span>
                        </a>
                    </li>


​                    
                    <li>
                        <a target="_blank" href="/feed.xml">
                            <span class="fa-stack fa-lg">
                                <i class="fa fa-circle fa-stack-2x"></i>
                                <i class="fa fa-rss fa-stack-1x fa-inverse"></i>
                            </span>
                        </a>
                    </li>
                    
                </ul>
                <p class="copyright text-muted">
                    Copyright &copy; Gityuan  2021 | Powered by Jekyll with Hux Theme
                </p>
                <!-- 流量统计 -->
    							<div style="display:none">
                                    <!--
    								<script type="text/javascript">var cnzz_protocol = (("https:" == document.location.protocol) ? " https://" : " http://");document.write(unescape("%3Cspan id='cnzz_stat_icon_1000098804'%3E%3C/span%3E%3Cscript src='" + cnzz_protocol + "s22.cnzz.com/z_stat.php%3Fid%3D1000098804%26show%3Dpic' type='text/javascript'%3E%3C/script%3E"));</script>-->
                                    
                                    <!-- Baidu Tongji -->
                                    <script>
                                    var _hmt = _hmt || [];
                                    (function() {
                                      var hm = document.createElement("script");
                                      hm.src = "https://hm.baidu.com/hm.js?3b1c71bfd9641a47009f87240c135d75";
                                      var s = document.getElementsByTagName("script")[0]; 
                                      s.parentNode.insertBefore(hm, s);
                                    })();
                                    </script>
    							</div>
            </div>
        </div>
    </div>
</footer>

<!-- jQuery -->
<script src="/js/jquery.min.js "></script>

<!-- Bootstrap Core JavaScript -->
<script src="/js/bootstrap.min.js "></script>

<!-- Custom Theme JavaScript -->
<script src="/js/hux-blog.min.js "></script>


<!-- async load function -->
<script>
    function async(u, c) {
      var d = document, t = 'script',
          o = d.createElement(t),
          s = d.getElementsByTagName(t)[0];
      o.src = u;
      if (c) { o.addEventListener('load', function (e) { c(null, e); }, false); }
      s.parentNode.insertBefore(o, s);
    }
</script>


<!-- jquery.tagcloud.js -->
<script>
    // only load tagcloud.js in tag.html
    if($('#tag_cloud').length !== 0){
        async('/js/jquery.tagcloud.js',function(){
            $.fn.tagcloud.defaults = {
                //size: {start: 1, end: 1, unit: 'em'},
                color: {start: '#bbbbee', end: '#0085a1'},
            };
            $('#tag_cloud a').tagcloud();
        })
    }
</script>

<!--fastClick.js -->
<script>
    async("http://cdn.bootcss.com/fastclick/1.0.6/fastclick.min.js", function(){
        var $nav = document.querySelector("nav");
        if($nav) FastClick.attach($nav);
    })
</script>


<!-- Google Analytics -->


<!-- Side Catalog -->

<script type="text/javascript">
    function generateCatalog (selector) {
        var P = $('div.post-container'),a,n,t,l,i,c;
        a = P.find('h1,h2,h3,h4,h5,h6');
        a.each(function () {
            n = $(this).prop('tagName').toLowerCase();
            i = "#"+$(this).prop('id');
            t = $(this).text();
            c = $('<a href="'+i+'" rel="nofollow">'+t+'</a>');
            l = $('<li class="'+n+'_nav"></li>').append(c);
            $(selector).append(l);
        });
        return true;
    }

    generateCatalog(".catalog-body");
    
    // toggle side catalog
    $(".catalog-toggle").click((function(e){
        e.preventDefault();
        $('.side-catalog').toggleClass("fold")
    }))
    
    async("/js/jquery.nav.js", function () {
        $('.catalog-body').onePageNav({
            currentClass: "active",
            changeHash: !1,
            easing: "swing",
            filter: "",
            scrollSpeed: 700,
            scrollOffset: 0,
            scrollThreshold: .2,
            begin: null,
            end: null,
            scrollChange: null,
            padding: 80
        });
    });
</script>



</body>

</html>
