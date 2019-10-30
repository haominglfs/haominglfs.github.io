---
title: tomcat源码3-container
date: 2019-08-29 14:14:18
tags:
---

### Container 的4 个子容器

![](https://raw.githubusercontent.com/haominglfs/images/master/20190828151929.png)

<!--more-->

Container 的子容器Engine 、Host 、Context 、Wrapper 是逐层包含的关系，其中Engine是最顶层，每个service 最多只能有一个Engine, Engine 里面可以有多个Host ，每个Host 下可以有多个Context ，每个Context 下可以有多个Wrapper，它们的装配关系如下图所示。

![](https://raw.githubusercontent.com/haominglfs/images/master/20190828152057.png)

* Engine ：引擎，用来管理多个站点， 一个Service 最多只能有一个Engine。
* Host ：代表一个站点，也可以叫虚拟主机，通过配置Host 就可以添加站点。
* Context ：代表一个应用程序，对应着平时开发的一套程序，或者一个WEB-INF 目录以及下面的web.xml 文件。
* Wrapper ：每个Wrapper 封装着一个servlet。

​           Context 和Host 的区别是Context 表示一个应用，比如，默认配置下webapps 下的每个目录都是一个应用，其中ROOT目录中存放着主应用，其他目录存放着别的子应用，而整个webapps 是一个站点。假如www.haominglfs.com 域名对应着webapps 目录所代表的站点，其中的ROOT 目录里的应用就是主应用，访问时直接使用域名就可以，而webapps/test 目录存放的是test 子应用，访问时需要www.excelib.com/test ，每一个应用对应一个Context ，所有webapps 下的应用都属于www.haominglfs.com 站点，而blog.haominglfs.com 则是另外一个站点，属于另外一个Host。

##### 配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--定义了－个Server ，在8005 端口监听关闭命令“ SHUTDOWN ”；-->
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <!-- Security listener. Documentation at /docs/config/listeners.html
  <Listener className="org.apache.catalina.security.SecurityListener" />
  -->
  <!--APR library loader. Documentation at /docs/apr.html -->
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <!-- Prevent memory leaks due to use of particular java/javax APIs-->
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

  <!-- Global JNDI resources
       Documentation at /docs/jndi-resources-howto.html
  -->
  <GlobalNamingResources>
    <!-- Editable user database that can also be used by
         UserDatabaseRealm to authenticate users
    -->
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>

  <!-- A "Service" is a collection of one or more "Connectors" that share
       a single "Container" Note:  A "Service" is not itself a "Container",
       so you may not define subcomponents such as "Valves" at this level.
       Documentation at /docs/config/service.html
   -->
  <!-- 定义了一个名为Catalina的Service  -->
  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

    <!-- Define an AJP 1.3 Connector on port 8009 -->
    <!-- AJP 主要用于集成（如与Apache 集成） -->
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

    <!-- You should set jvmRoute to support load-balancing via AJP ie :
    <Engine name="Catalina" defaultHost="localhost" jvmRoute="jvm1">
    -->
    <!--定义了一个名为Catalina 的Engine 
			defaultHost 属性，它表示接收到请求的域名如果在所有的Host 的name 和Alias 中都找不到时使用的默认Host
		-->
    <Engine name="Catalina" defaultHost="localhost">

      <!-- Use the LockOutRealm to prevent attempts to guess user passwords
           via a brute-force attack -->
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <!-- This Realm uses the UserDatabase configured in the global JNDI
             resources under the key "UserDatabase".  Any edits
             that are performed against this UserDatabase are immediately
             available for use by the Realm.  -->
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>
			<!--定义了一个名为localhost 的Host  name属性代表域名，所以上面定义的站点可以通过localhost 访问
			 appBase 属性指定站点的位置，比如，上面定义的站点就是默认的webapps 目录， unpackWARs 属性表示是否自动解压war 文件， autoDeploy 属性表示是否自动部署，如果autoDeploy 为true 那么Tomcat 在运行过程中在webapps 目录中加入新的应用将会自动部署并启动。另外Host 还有一个Alias 子标签，可以通过这个标签来定义别名，如果有多个域名访问同一个站点就可以这么定义
			-->
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />

      </Host>
    </Engine>
  </Service>
</Server>

```

##### Context 通过文件配置的方式一共有5 个位置可以配置：

* conf/server.xml 文件中的Context 标签。
* conf/[ enginename ]/[hostname ］／目录下以应用命名的xml 文件。
* 应用自己的／META-INF/context.xml 文件。
* conf/context.xml 文件。
* conf/[ enginename ]/[hostname ]/context.xml.default 文件。

其中前三个位置用于配置单独的应用，后两个配置的Context 是共享的， conf/context.xml文件中配置的内容在整个Tomcat 中共享;第5 种配置的内容在对应的站点（ Host ）中共享。另外，因为conf/server.xrnl 文件只有在Tomcat 重启的时候才会重新加载，所以第一种配置方法不推荐使用。 

Wrapper 的配置就是我们在web.xml 中配置的Servlet ， 一个Servlet 对应一个Wrapper。另外也可以在conf/web.xml 文件中配置全局的Wrapper，处理Jsp 的JspServlet 就配置在这里，所以不需要自己配置Jsp 就可以处理Jsp 请求了。 

### Container 的启动

Container 的启动是通过init 和start 方法来完成的，在前面分析过这两个方法会在Tomcat启动时被Service 调用。Container 也是按照Tomcat 的生命周期来管理的， init 和start 方法也会调用initlntemal 和startintemal 方法来具体处理，不过Container 和前面讲的Tomcat 整体结构启动的过程稍微有点不一样，主要有三点区别：

* Container 的4 个子容器有一个共同的父类ContainerBase ，这里定义了Container 容器的initlntemal和startlnternal 方法通用处理内容，具体容器还可以添加向己的内容；
* 除了最顶层容器的init 是被Service 调用的,子容器的init 方法并不是在容器中逐层循环调用的，而是在执行start 方法的时候通过状态判断还没有初始化才会调用；
* start 方法除了在父容器的startlnternal 方法中调用，还会在父容器的添加子容器的addChild 方法中调用，这主要是因为Context 和Wrapper 是动态添加的，我们在站点目录下放一个应用的文件夹或者war 包就可以添加一个Context ，在web.xml 文件中配置一个Servlet 就可以添加一个Wrapper ，所以Context 和Wrapper 是在容器启动的过程中才动态查找出来添加到相应的父容器中的。

#### ContainerBase

```java
	<<ContainerBase>>
	@Override
    protected void initInternal() throws LifecycleException {
    		//初始化ThreadPoolExecutor 类型的startStopExecutor属性，用于管理启动和关闭的线程
        BlockingQueue<Runnable> startStopQueue = new LinkedBlockingQueue<>();
        startStopExecutor = new ThreadPoolExecutor(
                getStartStopThreadsInternal(),
                getStartStopThreadsInternal(), 10, TimeUnit.SECONDS,
                startStopQueue,
                new StartStopThreadFactory(getName() + "-startStop-"));
        startStopExecutor.allowCoreThreadTimeOut(true);
        super.initInternal();
    }
    
    //ContainerBase 的startlntemal 方法主要做了5件事：
      //如果有Cluster 和Realm 则调用其start方法；
      //调用所有子容器的start方法启动子容器；
      //调用管道中Value的start方法来启动管道；
      //启动完成后将生命周期状态设置为LifecycleState.STARTING状态；
      //启用后台线程定时处理一些事情。
		@Override
    protected synchronized void startInternal() throws LifecycleException {
        // Start our subordinate components, if any
        logger = null;
        getLogger();
      	//Cluster用于配置集群,它的作用就是同步Session
        Cluster cluster = getClusterInternal();
        if (cluster instanceof Lifecycle) {
            ((Lifecycle) cluster).start();
        }
      	//Realm是Tomcat的安全域，可以用来管理资源的访问权限
        Realm realm = getRealmInternal();
        if (realm instanceof Lifecycle) {
            ((Lifecycle) realm).start();
        }
      
     		//子容器是使用startStopExecutor调用新线程来启动的，这样可以用多个线程来同时启动，效率更高
      	//遍历Future 主要有两个作用：①其get方法是阻塞的，只有线程处理完之后才会向下走，这就保证了管道Pipeline 启动之前容器已经启动完成了；②可以处理启动过程中遇到的异常。 
        // Start our child containers, if any
        Container children[] = findChildren();
        List<Future<Void>> results = new ArrayList<>();
        for (int i = 0; i < children.length; i++) {
            results.add(startStopExecutor.submit(new StartChild(children[i])));
        }

        MultiThrowable multiThrowable = new MultiThrowable();

        for (Future<Void> result : results) {
            try {
                result.get();
            } catch (Throwable e) {
                log.error(sm.getString("containerBase.threadedStartFailed"), e);
                multiThrowable.add(e);
            }

        }
        if (multiThrowable.size() > 0) {
            throw new LifecycleException(sm.getString("containerBase.threadedStartFailed"),
                    multiThrowable.getThrowable());
        }
      	//启动容器的管道
        // Start the Valves in our pipeline (including the basic), if any
        if (pipeline instanceof Lifecycle) {
            ((Lifecycle) pipeline).start();
        }
        setState(LifecycleState.STARTING);
        // Start our thread
        threadStart();
    }

/**
     * Start the background thread that will periodically check for
     * session timeouts.
     */
		//其实这个私有方法是start()方法中最重要的方法
    protected void threadStart() {
        if (thread != null)
            return;
      	//如果backgroundProcessorDelay 不大于0，那么方法就停止,默认值为 -1。
      	//子容器中，只有StandardEngine设置这个值为10，其他三个容器默认为-1，说明只有StandardEngine在start()方法调用的时候才会走这个方法，其他容器这个方法是走不到下面代码的
        if (backgroundProcessorDelay <= 0)
            return;
        threadDone = false;
        String threadName = "ContainerBackgroundProcessor[" + toString() + "]";
        thread = new Thread(new ContainerBackgroundProcessor(), threadName);
        thread.setDaemon(true);
        thread.start();
    }

// -------------------------------------- ContainerExecuteDelay Inner Class

    /**
     * Private thread class to invoke the backgroundProcess method
     * of this container and its children after a fixed delay.
     */
    protected class ContainerBackgroundProcessor implements Runnable {
				//ContainerBackgroundProcessor是个Runnable接口的实现类，查看其run方法
        @Override
        public void run() {
            Throwable t = null;
            String unexpectedDeathMessage = sm.getString(
                    "containerBase.backgroundProcess.unexpectedThreadDeath",
                    Thread.currentThread().getName());
            try {
                while (!threadDone) {
                    try {
                        Thread.sleep(backgroundProcessorDelay * 1000L);
                    } catch (InterruptedException e) {
                        // Ignore
                    }
                    if (!threadDone) {
                        processChildren(ContainerBase.this);
                    }
                }
            } catch (RuntimeException|Error e) {
                t = e;
                throw e;
            } finally {
                if (!threadDone) {
                    log.error(unexpectedDeathMessage, t);
                }
            }
        }

        protected void processChildren(Container container) {
            ClassLoader originalClassLoader = null;
            try {
                if (container instanceof Context) {
                    Loader loader = ((Context) container).getLoader();
                    // Loader will be null for FailedContext instances
                    if (loader == null) {
                        return;
                    }

                    // Ensure background processing for Contexts and Wrappers
                    // is performed under the web app's class loader
                    originalClassLoader = ((Context) container).bind(false, null);
                }
                container.backgroundProcess();
              	//获取所有的子容器，然后遍历每个子容器来调用他们的processChildren()方法
                Container[] children = container.findChildren();
                for (int i = 0; i < children.length; i++) {
                    if (children[i].getBackgroundProcessorDelay() <= 0) {
                        processChildren(children[i]);
                    }
                }
            } catch (Throwable t) {
                ExceptionUtils.handleThrowable(t);
                log.error("Exception invoking periodic operation: ", t);
            } finally {
                if (container instanceof Context) {
                    ((Context) container).unbind(false, originalClassLoader);
               }
            }
        }
    }

/**
     * Execute a periodic task, such as reloading, etc. This method will be
     * invoked inside the classloading context of this container. Unexpected
     * throwables will be caught and logged.
     */
    @Override
    public void backgroundProcess() {

        if (!getState().isAvailable())
            return;
				//Catalina的集群组件，通过cluster组件可以实现：集群应用部署，即多个Tomcat实例，不需要每个都分别部署应用，只需要在某个实例上部署，整个集群中的各个实例都会自动同步应用进行部署。那么他的backgroundProcess()方法主要的功能是监听指定文件夹下有没有新增的war包或者文件是新增的还是修改的已决定来重新部署和通知其他tomcat集群成员。
        Cluster cluster = getClusterInternal();
        if (cluster != null) {
            try {
                cluster.backgroundProcess();
            } catch (Exception e) {
                log.warn(sm.getString("containerBase.backgroundProcess.cluster",
                        cluster), e);
            }
        }
      //Catalina中的安全组件，backgroundProcess()方法主要的功能是，貌似是跟servlet的安全校验有关。
        Realm realm = getRealmInternal();
        if (realm != null) {
            try {
                realm.backgroundProcess();
            } catch (Exception e) {
                log.warn(sm.getString("containerBase.backgroundProcess.realm", realm), e);
            }
        }
      	//容器的pipeline组件，这里是遍历整个pipeline链表，分别调用backgroundProcess()方法
        Valve current = pipeline.getFirst();
        while (current != null) {
            try {
                current.backgroundProcess();
            } catch (Exception e) {
                log.warn(sm.getString("containerBase.backgroundProcess.valve", current), e);
            }
            current = current.getNext();
        }
        fireLifecycleEvent(Lifecycle.PERIODIC_EVENT, null);
    }
```

```java
<<StandardContext>>
@Override
    public void backgroundProcess() {

        if (!getState().isAvailable())
            return;
				//Catalina在启动过程中创建的classloader的实例,backgroundProcess()方法主要的功能是查看Context容器是否需要重新加载,热部署就是利用这个机制来完成的
        Loader loader = getLoader();
        if (loader != null) {
            try {
                loader.backgroundProcess();
            } catch (Exception e) {
                log.warn(sm.getString(
                        "standardContext.backgroundProcess.loader", loader), e);
            }
        }
        //Catalina中的Session管理器，backgroundProcess()方法主要的功能是将过期会话(session)置为无效
        Manager manager = getManager();
        if (manager != null) {
            try {
                manager.backgroundProcess();
            } catch (Exception e) {
                log.warn(sm.getString(
                        "standardContext.backgroundProcess.manager", manager),
                        e);
            }
        }
        WebResourceRoot resources = getResources();
        if (resources != null) {
            try {
                resources.backgroundProcess();
            } catch (Exception e) {
                log.warn(sm.getString(
                        "standardContext.backgroundProcess.resources",
                        resources), e);
            }
        }
        InstanceManager instanceManager = getInstanceManager();
        if (instanceManager instanceof DefaultInstanceManager) {
            try {
                ((DefaultInstanceManager)instanceManager).backgroundProcess();
            } catch (Exception e) {
                log.warn(sm.getString(
                        "standardContext.backgroundProcess.instanceManager",
                        resources), e);
            }
        }
        super.backgroundProcess();
    }
```

```java
<<StandardWrapper>>
    @Override
    public void backgroundProcess() {
        super.backgroundProcess();

        if (!getState().isAvailable())
            return;

        if (getServlet() instanceof PeriodicEventListener) {
            ((PeriodicEventListener) getServlet()).periodicEvent();
        }
    }
```

threadStart 方法启动的后台线程是一个while 循环，内部会定期调用backgroundProcess 方法做一些事情，间隔时间的长短是通过ContainerBase 的backgroundProcessor Delay 属性来设置的，单位是秒，如果小于0 就不启动后台线程了，不过其backgroundProcess 方法会在父容器的后台线程中调用。backgroundProcess 方法是Container 接口中的一个方法， 一共有3 个实现，分别在ContainerBase 、StandardContext 和StandardWrapper 中， ContainerBase 中提供了所有容器共同的处理过程， StandardContext 和StandardWrapper 的backgroundProcess 方法除了处理自己相关的业务，也调用ContainerBase 中的处理。ContainerBase 的backgroundProcess 方法中调用了Cluster 、Realm 和管道的backgroundProcess 方法； StandardContext 的backgroundProcess方法中对Session 过期和资源变化进行了处理； StandardWrapper 的backgroundProcess方法会对Jsp 生成的Servlet 定期进行检查。

### Engine

```java
<<StandardEngine>>
@Override
    protected void initInternal() throws LifecycleException {
        // Ensure that a Realm is present before any attempt is made to start
        // one. This will create the default NullRealm if necessary.
        //如果没有配置Realm ，则使用一个默认的NullRealm 
        getRealm();
        super.initInternal();
    }
    
@Override
    protected synchronized void startInternal() throws LifecycleException {

        // Log our server identification information
        if(log.isInfoEnabled())
            log.info( "Starting Servlet Engine: " + ServerInfo.getServerInfo());

        // Standard container startup
        super.startInternal();
    }
```

### Host

```java
<<StandardHost>>
//没有重写initInternal方法

//这里的代码看起来虽然比较多，但功能却非常简单，就是检查Host的管道中有没有指定的Value ，如果没有则添加进去。检查的方法是遍历所有的Value然后通过名字判断的，检查的Value的类型通过getErrorReportValveClass方法获取，它返回errorReportValveClass属性，可以配置，默认值是org.apache.catalina.valves.ErrorReportValve 
@Override
    protected synchronized void startInternal() throws LifecycleException {
        // Set error report valve
        String errorValve = getErrorReportValveClass();
        if ((errorValve != null) && (!errorValve.equals(""))) {
            try {
                boolean found = false;
                Valve[] valves = getPipeline().getValves();
                for (Valve valve : valves) {
                    if (errorValve.equals(valve.getClass().getName())) {
                        found = true;
                        break;
                    }
                }
                if(!found) {
                    Valve valve =
                        (Valve) Class.forName(errorValve).getConstructor().newInstance();
                    getPipeline().addValve(valve);
                }
            } catch (Throwable t) {
                ExceptionUtils.handleThrowable(t);
                log.error(sm.getString(
                        "standardHost.invalidErrorReportValveClass",
                        errorValve), t);
            }
        }
        super.startInternal();
    }
    
private String errorReportValveClass =
        "org.apache.catalina.valves.ErrorReportValve";
public String getErrorReportValveClass() {
        return (this.errorReportValveClass);
    }
```

注册监听器

```java
<<Catalina>>
/**
     * Create and configure the Digester we will be using for startup.
     * @return the main digester to parse server.xml
     */
    protected Digester createStartDigester() {
        long t1=System.currentTimeMillis();
        // Initialize the digester
        Digester digester = new Digester();
        digester.setValidating(false);
        digester.setRulesValidation(true);
        HashMap<Class<?>, List<String>> fakeAttributes = new HashMap<>();
        ..........
        // Add RuleSets for nested elements
        digester.addRuleSet(new NamingRuleSet("Server/GlobalNamingResources/"));
        digester.addRuleSet(new EngineRuleSet("Server/Service/"));
        digester.addRuleSet(new HostRuleSet("Server/Service/Engine/"));
        digester.addRuleSet(new ContextRuleSet("Server/Service/Engine/Host/"));
        addClusterRuleSet(digester, "Server/Service/Engine/Host/Cluster/");
        digester.addRuleSet(new NamingRuleSet("Server/Service/Engine/Host/Context/"));
        ..........
        
 <<HostRuleSet>> 
 @Override
    public void addRuleInstances(Digester digester) {

        digester.addObjectCreate(prefix + "Host",
                                 "org.apache.catalina.core.StandardHost",
                                 "className");
        digester.addSetProperties(prefix + "Host");
        digester.addRule(prefix + "Host",
                         new CopyParentClassLoaderRule());
        //添加hostConfig监听器
        digester.addRule(prefix + "Host",
                         new LifecycleListenerRule
                         ("org.apache.catalina.startup.HostConfig",
                          "hostConfigClass"));
       .........
    }

```

Host 的启动除了startlntemal 方法，还有HostConfig 中相应的方法， HostConfig 继承自LifecycleListener 的监听器（ Engine 也有对应的EngineConfig 监昕器，不过里面只是简单地做了日志记录），在接收到Lifecycle.START_EVENT 事件时会调用start 方法来启动， HostConfig 的start 方法会检查配置的Host 站点配置的位置是否存在以及是不是目录，最后调用deployApps 方法部署应用， deployApps 方法代码如下

```java
<<HostConfig>>
protected void deployApps() {
        File appBase = host.getAppBaseFile();
        File configBase = host.getConfigBaseFile();
        String[] filteredAppPaths = filterAppPaths(appBase.list());
        // Deploy XML descriptors from configBase
        deployDescriptors(configBase, configBase.list());
        // Deploy WARs
        deployWARs(appBase, filteredAppPaths);
        // Deploy expanded folders
        deployDirectories(appBase, filteredAppPaths);
    }
```

一共有三种部署方式：通过XML 描述文件、通过WAR 文件和通过文件夹部署。XML文件指的是conf/[enginename ]/[hostname ]/* .xml 文件， WAR 文件和文件夹是Host 站点目录下的WAR 文件和文件夹，这里会自动找出来并部署上，所以我们如果要添加应用只需要直接放在Host 站点的目录下就可以了。部署完成后，会将部署的Context 通过StandardHost 的add Child 方法添加到Host 里面。StandardHost 的addChild 方法会调用父类ContainerBase 的addChild 方法， 其中会调用子类（这里指Context ）的start 方法来启动子容器。 

### Context

```java
<<StandardContext>>
@Override
    protected synchronized void startInternal() throws LifecycleException {

        if(log.isDebugEnabled())
            log.debug("Starting " + getBaseName());

        // Send j2ee.state.starting notification
        if (this.getObjectName() != null) {
            Notification notification = new Notification("j2ee.state.starting",
                    this.getObjectName(), sequenceNumber.getAndIncrement());
            broadcaster.sendNotification(notification);
        }

        setConfigured(false);
        boolean ok = true;

        // Currently this is effectively a NO-OP but needs to be called to
        // ensure the NamingResources follows the correct lifecycle
        if (namingResources != null) {
            namingResources.start();
        }

        // Post work directory
        postWorkDirectory();

        // Add missing components as necessary
        if (getResources() == null) {   // (1) Required by Loader
            if (log.isDebugEnabled())
                log.debug("Configuring default Resources");

            try {
                setResources(new StandardRoot(this));
            } catch (IllegalArgumentException e) {
                log.error(sm.getString("standardContext.resourcesInit"), e);
                ok = false;
            }
        }
        if (ok) {
            resourcesStart();
        }

        if (getLoader() == null) {
            WebappLoader webappLoader = new WebappLoader(getParentClassLoader());
            webappLoader.setDelegate(getDelegate());
            setLoader(webappLoader);
        }

        // An explicit cookie processor hasn't been specified; use the default
        if (cookieProcessor == null) {
            cookieProcessor = new Rfc6265CookieProcessor();
        }

        // Initialize character set mapper
        getCharsetMapper();

        // Validate required extensions
        boolean dependencyCheck = true;
        try {
            dependencyCheck = ExtensionValidator.validateApplication
                (getResources(), this);
        } catch (IOException ioe) {
            log.error(sm.getString("standardContext.extensionValidationError"), ioe);
            dependencyCheck = false;
        }

        if (!dependencyCheck) {
            // do not make application available if dependency check fails
            ok = false;
        }

        // Reading the "catalina.useNaming" environment variable
        String useNamingProperty = System.getProperty("catalina.useNaming");
        if ((useNamingProperty != null)
            && (useNamingProperty.equals("false"))) {
            useNaming = false;
        }

        if (ok && isUseNaming()) {
            if (getNamingContextListener() == null) {
                NamingContextListener ncl = new NamingContextListener();
                ncl.setName(getNamingContextName());
                ncl.setExceptionOnFailedWrite(getJndiExceptionOnFailedWrite());
                addLifecycleListener(ncl);
                setNamingContextListener(ncl);
            }
        }

        // Standard container startup
        if (log.isDebugEnabled())
            log.debug("Processing standard container startup");


        // Binding thread
        ClassLoader oldCCL = bindThread();

        try {
            if (ok) {
                // Start our subordinate components, if any
                Loader loader = getLoader();
                if (loader instanceof Lifecycle) {
                    ((Lifecycle) loader).start();
                }

                // since the loader just started, the webapp classloader is now
                // created.
                setClassLoaderProperty("clearReferencesRmiTargets",
                        getClearReferencesRmiTargets());
                setClassLoaderProperty("clearReferencesStopThreads",
                        getClearReferencesStopThreads());
                setClassLoaderProperty("clearReferencesStopTimerThreads",
                        getClearReferencesStopTimerThreads());
                setClassLoaderProperty("clearReferencesHttpClientKeepAliveThread",
                        getClearReferencesHttpClientKeepAliveThread());
                setClassLoaderProperty("clearReferencesObjectStreamClassCaches",
                        getClearReferencesObjectStreamClassCaches());

                // By calling unbindThread and bindThread in a row, we setup the
                // current Thread CCL to be the webapp classloader
                unbindThread(oldCCL);
                oldCCL = bindThread();

                // Initialize logger again. Other components might have used it
                // too early, so it should be reset.
                logger = null;
                getLogger();

                Realm realm = getRealmInternal();
                if(null != realm) {
                    if (realm instanceof Lifecycle) {
                        ((Lifecycle) realm).start();
                    }

                    // Place the CredentialHandler into the ServletContext so
                    // applications can have access to it. Wrap it in a "safe"
                    // handler so application's can't modify it.
                    CredentialHandler safeHandler = new CredentialHandler() {
                        @Override
                        public boolean matches(String inputCredentials, String storedCredentials) {
                            return getRealmInternal().getCredentialHandler().matches(inputCredentials, storedCredentials);
                        }

                        @Override
                        public String mutate(String inputCredentials) {
                            return getRealmInternal().getCredentialHandler().mutate(inputCredentials);
                        }
                    };
                    context.setAttribute(Globals.CREDENTIAL_HANDLER, safeHandler);
                }

                // Notify our interested LifecycleListeners
                fireLifecycleEvent(Lifecycle.CONFIGURE_START_EVENT, null);

                // Start our child containers, if not already started
                for (Container child : findChildren()) {
                    if (!child.getState().isAvailable()) {
                        child.start();
                    }
                }

                // Start the Valves in our pipeline (including the basic),
                // if any
                if (pipeline instanceof Lifecycle) {
                    ((Lifecycle) pipeline).start();
                }

                // Acquire clustered manager
                Manager contextManager = null;
                Manager manager = getManager();
                if (manager == null) {
                    if (log.isDebugEnabled()) {
                        log.debug(sm.getString("standardContext.cluster.noManager",
                                Boolean.valueOf((getCluster() != null)),
                                Boolean.valueOf(distributable)));
                    }
                    if ( (getCluster() != null) && distributable) {
                        try {
                            contextManager = getCluster().createManager(getName());
                        } catch (Exception ex) {
                            log.error("standardContext.clusterFail", ex);
                            ok = false;
                        }
                    } else {
                        contextManager = new StandardManager();
                    }
                }

                // Configure default manager if none was specified
                if (contextManager != null) {
                    if (log.isDebugEnabled()) {
                        log.debug(sm.getString("standardContext.manager",
                                contextManager.getClass().getName()));
                    }
                    setManager(contextManager);
                }

                if (manager!=null && (getCluster() != null) && distributable) {
                    //let the cluster know that there is a context that is distributable
                    //and that it has its own manager
                    getCluster().registerManager(manager);
                }
            }

            if (!getConfigured()) {
                log.error(sm.getString("standardContext.configurationFail"));
                ok = false;
            }

            // We put the resources into the servlet context
            if (ok)
                getServletContext().setAttribute
                    (Globals.RESOURCES_ATTR, getResources());

            if (ok ) {
                if (getInstanceManager() == null) {
                    javax.naming.Context context = null;
                    if (isUseNaming() && getNamingContextListener() != null) {
                        context = getNamingContextListener().getEnvContext();
                    }
                    Map<String, Map<String, String>> injectionMap = buildInjectionMap(
                            getIgnoreAnnotations() ? new NamingResourcesImpl(): getNamingResources());
                    setInstanceManager(new DefaultInstanceManager(context,
                            injectionMap, this, this.getClass().getClassLoader()));
                }
                getServletContext().setAttribute(
                        InstanceManager.class.getName(), getInstanceManager());
                InstanceManagerBindings.bind(getLoader().getClassLoader(), getInstanceManager());
            }

            // Create context attributes that will be required
            if (ok) {
                getServletContext().setAttribute(
                        JarScanner.class.getName(), getJarScanner());
            }

            // Set up the context init params
            mergeParameters();

            // Call ServletContainerInitializers
            for (Map.Entry<ServletContainerInitializer, Set<Class<?>>> entry :
                initializers.entrySet()) {
                try {
                    entry.getKey().onStartup(entry.getValue(),
                            getServletContext());
                } catch (ServletException e) {
                    log.error(sm.getString("standardContext.sciFail"), e);
                    ok = false;
                    break;
                }
            }

            // Configure and call application event listeners
            //配置listeners
            if (ok) {
                if (!listenerStart()) {
                    log.error(sm.getString("standardContext.listenerFail"));
                    ok = false;
                }
            }

            // Check constraints for uncovered HTTP methods
            // Needs to be after SCIs and listeners as they may programmatically
            // change constraints
            if (ok) {
                checkConstraintsForUncoveredMethods(findConstraints());
            }

            try {
                // Start manager
                Manager manager = getManager();
                if (manager instanceof Lifecycle) {
                    ((Lifecycle) manager).start();
                }
            } catch(Exception e) {
                log.error(sm.getString("standardContext.managerFail"), e);
                ok = false;
            }

            // Configure and call application filters
            //配置filters
            if (ok) {
                if (!filterStart()) {
                    log.error(sm.getString("standardContext.filterFail"));
                    ok = false;
                }
            }

            // Load and initialize all "load on startup" servlets
            //配置load-on-startup
            if (ok) {
                if (!loadOnStartup(findChildren())){
                    log.error(sm.getString("standardContext.servletFail"));
                    ok = false;
                }
            }

            // Start ContainerBackgroundProcessor thread
            super.threadStart();
        } finally {
            // Unbinding thread
            unbindThread(oldCCL);
        }

        // Set available status depending upon startup success
        if (ok) {
            if (log.isDebugEnabled())
                log.debug("Starting completed");
        } else {
            log.error(sm.getString("standardContext.startFailed", getName()));
        }

        startTime=System.currentTimeMillis();

        // Send j2ee.state.running notification
        if (ok && (this.getObjectName() != null)) {
            Notification notification =
                new Notification("j2ee.state.running", this.getObjectName(),
                                 sequenceNumber.getAndIncrement());
            broadcaster.sendNotification(notification);
        }

        // The WebResources implementation caches references to JAR files. On
        // some platforms these references may lock the JAR files. Since web
        // application start is likely to have read from lots of JARs, trigger
        // a clean-up now.
        getResources().gc();

        // Reinitializing if something went wrong
        if (!ok) {
            setState(LifecycleState.FAILED);
        } else {
            setState(LifecycleState.STARTING);
        }
    }
```

listenerStart 、filterStart 和loadOnStartup 方法分别调用配置在Listener 的contextlnitialized 方法以及Filter 和配置了load-on-startup 的Servlet 的init 方法。 

```java
<<ContextConfig>>
/**
     * Process a "contextConfig" event for this Context.
     */
    protected synchronized void configureStart() {
        // Called from StandardContext.start()

        if (log.isDebugEnabled()) {
            log.debug(sm.getString("contextConfig.start"));
        }

        if (log.isDebugEnabled()) {
            log.debug(sm.getString("contextConfig.xmlSettings",
                    context.getName(),
                    Boolean.valueOf(context.getXmlValidation()),
                    Boolean.valueOf(context.getXmlNamespaceAware())));
        }

        webConfig();

        if (!context.getIgnoreAnnotations()) {
            applicationAnnotationsConfig();
        }
        if (ok) {
            validateSecurityRoles();
        }

        // Configure an authenticator if we need one
        if (ok) {
            authenticatorConfig();
        }

        // Dump the contents of this pipeline if requested
        if (log.isDebugEnabled()) {
            log.debug("Pipeline Configuration:");
            Pipeline pipeline = context.getPipeline();
            Valve valves[] = null;
            if (pipeline != null) {
                valves = pipeline.getValves();
            }
            if (valves != null) {
                for (int i = 0; i < valves.length; i++) {
                    log.debug("  " + valves[i].getClass().getName());
                }
            }
            log.debug("======================");
        }

        // Make our application available if no problems were encountered
        if (ok) {
            context.setConfigured(true);
        } else {
            log.error(sm.getString("contextConfig.unavailable"));
            context.setConfigured(false);
        }

    }
    
/**
     * Scan the web.xml files that apply to the web application and merge them
     * using the rules defined in the spec. For the global web.xml files,
     * where there is duplicate configuration, the most specific level wins. ie
     * an application's web.xml takes precedence over the host level or global
     * web.xml file.
     */
    protected void webConfig() {
        /*
         * Anything and everything can override the global and host defaults.
         * This is implemented in two parts
         * - Handle as a web fragment that gets added after everything else so
         *   everything else takes priority
         * - Mark Servlets as overridable so SCI configuration can replace
         *   configuration from the defaults
         */

        /*
         * The rules for annotation scanning are not as clear-cut as one might
         * think. Tomcat implements the following process:
         * - As per SRV.1.6.2, Tomcat will scan for annotations regardless of
         *   which Servlet spec version is declared in web.xml. The EG has
         *   confirmed this is the expected behaviour.
         * - As per http://java.net/jira/browse/SERVLET_SPEC-36, if the main
         *   web.xml is marked as metadata-complete, JARs are still processed
         *   for SCIs.
         * - If metadata-complete=true and an absolute ordering is specified,
         *   JARs excluded from the ordering are also excluded from the SCI
         *   processing.
         * - If an SCI has a @HandlesType annotation then all classes (except
         *   those in JARs excluded from an absolute ordering) need to be
         *   scanned to check if they match.
         */
        WebXmlParser webXmlParser = new WebXmlParser(context.getXmlNamespaceAware(),
                context.getXmlValidation(), context.getXmlBlockExternal());

        Set<WebXml> defaults = new HashSet<>();
        defaults.add(getDefaultWebXmlFragment(webXmlParser));

        WebXml webXml = createWebXml();

        // Parse context level web.xml
        InputSource contextWebXml = getContextWebXmlSource();
        if (!webXmlParser.parseWebXml(contextWebXml, webXml, false)) {
            ok = false;
        }

        ServletContext sContext = context.getServletContext();

        // Ordering is important here

        // Step 1. Identify all the JARs packaged with the application and those
        // provided by the container. If any of the application JARs have a
        // web-fragment.xml it will be parsed at this point. web-fragment.xml
        // files are ignored for container provided JARs.
        Map<String,WebXml> fragments = processJarsForWebFragments(webXml, webXmlParser);

        // Step 2. Order the fragments.
        Set<WebXml> orderedFragments = null;
        orderedFragments =
                WebXml.orderWebFragments(webXml, fragments, sContext);

        // Step 3. Look for ServletContainerInitializer implementations
        if (ok) {
            processServletContainerInitializers();
        }

        if  (!webXml.isMetadataComplete() || typeInitializerMap.size() > 0) {
            // Step 4. Process /WEB-INF/classes for annotations and
            // @HandlesTypes matches
            Map<String,JavaClassCacheEntry> javaClassCache = new HashMap<>();

            if (ok) {
                WebResource[] webResources =
                        context.getResources().listResources("/WEB-INF/classes");

                for (WebResource webResource : webResources) {
                    // Skip the META-INF directory from any JARs that have been
                    // expanded in to WEB-INF/classes (sometimes IDEs do this).
                    if ("META-INF".equals(webResource.getName())) {
                        continue;
                    }
                    processAnnotationsWebResource(webResource, webXml,
                            webXml.isMetadataComplete(), javaClassCache);
                }
            }

            // Step 5. Process JARs for annotations and
            // @HandlesTypes matches - only need to process those fragments we
            // are going to use (remember orderedFragments includes any
            // container fragments)
            if (ok) {
                processAnnotations(
                        orderedFragments, webXml.isMetadataComplete(), javaClassCache);
            }

            // Cache, if used, is no longer required so clear it
            javaClassCache.clear();
        }

        if (!webXml.isMetadataComplete()) {
            // Step 6. Merge web-fragment.xml files into the main web.xml
            // file.
            if (ok) {
                ok = webXml.merge(orderedFragments);
            }

            // Step 7. Apply global defaults
            // Have to merge defaults before JSP conversion since defaults
            // provide JSP servlet definition.
            webXml.merge(defaults);

            // Step 8. Convert explicitly mentioned jsps to servlets
            if (ok) {
                convertJsps(webXml);
            }

            // Step 9. Apply merged web.xml to Context
            if (ok) {
                configureContext(webXml);
            }
        } else {
            webXml.merge(defaults);
            convertJsps(webXml);
            configureContext(webXml);
        }

        if (context.getLogEffectiveWebXml()) {
            log.info("web.xml:\n" + webXml.toXml());
        }

        // Always need to look for static resources
        // Step 10. Look for static resources packaged in JARs
        if (ok) {
            // Spec does not define an order.
            // Use ordered JARs followed by remaining JARs
            Set<WebXml> resourceJars = new LinkedHashSet<>();
            for (WebXml fragment : orderedFragments) {
                resourceJars.add(fragment);
            }
            for (WebXml fragment : fragments.values()) {
                if (!resourceJars.contains(fragment)) {
                    resourceJars.add(fragment);
                }
            }
            processResourceJARs(resourceJars);
            // See also StandardContext.resourcesStart() for
            // WEB-INF/classes/META-INF/resources configuration
        }

        // Step 11. Apply the ServletContainerInitializer config to the
        // context
        if (ok) {
            for (Map.Entry<ServletContainerInitializer,
                    Set<Class<?>>> entry :
                        initializerClassMap.entrySet()) {
                if (entry.getValue().isEmpty()) {
                    context.addServletContainerInitializer(
                            entry.getKey(), null);
                } else {
                    context.addServletContainerInitializer(
                            entry.getKey(), entry.getValue());
                }
            }
        }
    }
```



  Context 和 Host 一样也有一个LifecycleListener 类型的监听器ContextConfig ， 其中configureStart 方法用来处理CONFTGURE_START_EVENT 事件，这个方法里面调用webConfig方法， webConfig 方法解析了web.xml文件，相应地创建了Wrapper 并使用addChild 添加到了Context 里面。 

### Wrapper

```java
<<Wrapper>>
//没有重写initlntemal 方法

@Override
    protected synchronized void startInternal() throws LifecycleException {

        // Send j2ee.state.starting notification
        if (this.getObjectName() != null) {
            Notification notification = new Notification("j2ee.state.starting",
                                                        this.getObjectName(),
                                                        sequenceNumber++);
            broadcaster.sendNotification(notification);
        }

        // Start up this component
        super.startInternal();

        setAvailable(0L);

        // Send j2ee.state.running notification
        if (this.getObjectName() != null) {
            Notification notification =
                new Notification("j2ee.state.running", this.getObjectName(),
                                sequenceNumber++);
            broadcaster.sendNotification(notification);
        }

    }
```

这里主要做了三件事情：

* 用broadcaster 发送通知，主要用于JMX;
* 调用了父类ContainerBase 中的startlntemal 方法；
* 调用setAvailable 方法让Servlet 有效。

这里的setAvailable 方法是Wrapper 接口中的方法，其作用是设置Wrapper 所包含的Servlet 有效的起始时间，如果所设置的时间为将来的时间，那么调用所对应的Servlet 就会产生错误，直到过了所设置的时间之后才可以正常调用，它的类型是long，如果设置为Long.MAX VALUE 就一直不可以调用了。Wrapper 没有别的容器那种XXXConfig 样式的LifecycleListener 监听器。



