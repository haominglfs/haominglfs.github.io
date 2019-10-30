---
title: tomcat源码2
date: 2019-08-27 22:15:05
tags: tomcat
---

# Tomcat拾遗--BootStrap类的静态代码块和反射调用Catalina的意义是什么

首先我们需要知道一个潜规则：即如果我们在A类中调用B类，如果B类没有被classloader加载或者就算加载了 但是该classloader和A类的classloader属于平行的，即我们在A的classloader中找不到B类的class，那么A会使用自己的classloader去加载B。

<!--more-->

## 反射调用Catalina的意义

因为Bootstrap这个类在Tomcat打包发布时是放在bin\bootstrap.jar中，
 而Catalina类是放在lib\catalina.jar中,两个jar是用不同的ClassLoader加载的，
 所以不能在Bootstrap类中直接引用Catalina类，只能通过反射。
 `这也意味着 后续我们在tomcat的Catalina类里面启动的类默认都是使用catalinaLoader（除了我们的context使用webappclassloader去加载的）`，进而tomcat使用的类只能被tomcat自己使用，而不会被其他应用使用

### 组件图

![](https://raw.githubusercontent.com/haominglfs/images/master/20190821095811.png)

多个 Connector 和一个 Container 就形成了一个 Service，Service 的概念大家都很熟悉了，有了 Service 就可以对外提供服务了，但是 Service 还要一个生存的环境，必须要有人能够给她生命、掌握其生死大权，那就非 Server 莫属了。所以整个 Tomcat 的生命周期由 Server 控制。

### connector

![](https://raw.githubusercontent.com/haominglfs/images/master/20190826152751.png)

![](https://raw.githubusercontent.com/haominglfs/images/master/20190822092423.png)

![](https://raw.githubusercontent.com/haominglfs/images/master/20190822092926.png)

在tomcat中，connector负责接收来自客户端的连接，并交由后续的代码进行处理。connector对象持有ProtocolHandler对象；ProtocolHandler对象持有AbstractEndpoint对象。AbstractEndpoint负责创建服务器套接字，并绑定到监听端口；同时还创建accepter线程来接收客户端的连接以及poller线程来处理连接中的读写请求。其结构如上图所示。

```java
public Connector(String protocol) {
        //设置协议
        setProtocol(protocol);
        // Instantiate protocol handler
        ProtocolHandler p = null;
        try {
            //反射生成ProtocolHandler实例
            Class<?> clazz = Class.forName(protocolHandlerClassName);
            p = (ProtocolHandler) clazz.getConstructor().newInstance();
        } catch (Exception e) {
            log.error(sm.getString(
                    "coyoteConnector.protocolHandlerInstantiationFailed"), e);
        } finally {
            this.protocolHandler = p;
        }

        if (Globals.STRICT_SERVLET_COMPLIANCE) {
            uriCharset = StandardCharsets.ISO_8859_1;
        } else {
            uriCharset = StandardCharsets.UTF_8;
        }
    }

public void setProtocol(String protocol) {

        boolean aprConnector = AprLifecycleListener.isAprAvailable() &&
                AprLifecycleListener.getUseAprConnector();

        if ("HTTP/1.1".equals(protocol) || protocol == null) {
            if (aprConnector) {
                setProtocolHandlerClassName("org.apache.coyote.http11.Http11AprProtocol");
            } else {
                setProtocolHandlerClassName("org.apache.coyote.http11.Http11NioProtocol");
            }
        } else if ("AJP/1.3".equals(protocol)) {
            if (aprConnector) {
                setProtocolHandlerClassName("org.apache.coyote.ajp.AjpAprProtocol");
            } else {
                setProtocolHandlerClassName("org.apache.coyote.ajp.AjpNioProtocol");
            }
        } else {
            setProtocolHandlerClassName(protocol);
        }
    }
```

Connector的构造函数带有协议属性，该协议属性是server.xml中Connector标签的protocol的属性值。Tomcat 8中默认值为HTTP/1.1，因此在Connector的构造函数中生成的是Http11NioProtocol对象。在setProtocol()方法中可以看到，tomcat8还包括其他几个协议处理器。协议处理器中带有Apr命名的都是使用Apr库来处理http请求的。通过使用APR库，Tomcat将使用JNI的方式来读取文件以及进行网络传输，可以大大提升Tomcat对静态文件的处理性能，同时如果你使用了HTTPS方式传输的话，也可以提升SSL的处理性能。AJP/1.3协议是Http服务器和应用服务器之间数据交互的协议，比如Apache服务器或IIS服务器与tomcat服务器之间进行数据交互。 
Http11NioProtocol是非阻塞模式的Http1.1协议处理器，使用java的nio包来实现非阻塞。可以看到，在tomcat 8中，默认使用的是非阻塞IO。 

```java
public AbstractHttp11Protocol(AbstractEndpoint<S> endpoint) {
        super(endpoint);
        setConnectionTimeout(Constants.DEFAULT_CONNECTION_TIMEOUT);
        ConnectionHandler<S> cHandler = new ConnectionHandler<>(this);
        setHandler(cHandler);
        getEndpoint().setHandler(cHandler);
    }
```

在创建Http11NioProtocol实例的时候，会创建NioEndpoint、ConnectionHandler实例。

```java
//<<AbstractEndpoint>>
public void init() throws Exception {
        if (bindOnInit) {
            bind();//调用子类bind方法初始化
            bindState = BindState.BOUND_ON_INIT;
        }
        if (this.domain != null) {
            // Register endpoint (as ThreadPool - historical name)
            oname = new ObjectName(domain + ":type=ThreadPool,name=\"" + getName() + "\"");
            Registry.getRegistry(null, null).registerComponent(this, oname, null);

            for (SSLHostConfig sslHostConfig : findSslHostConfigs()) {
                registerJmx(sslHostConfig);
            }
        }
    }
//<<NioEndpoint>>
/**
     * Initialize the endpoint.
     */
    @Override
    public void bind() throws Exception {

        if (!getUseInheritedChannel()) {
            // 打开serverSocketChannel
            serverSock = ServerSocketChannel.open();
            // 设置socket属性
            socketProperties.setProperties(serverSock.socket());
            InetSocketAddress addr = (getAddress()!=null?new InetSocketAddress(getAddress(),getPort()):new InetSocketAddress(getPort()));
            // 绑定监听端口
            serverSock.socket().bind(addr,getAcceptCount());
        } else {
            // Retrieve the channel provided by the OS
            Channel ic = System.inheritedChannel();
            if (ic instanceof ServerSocketChannel) {
                serverSock = (ServerSocketChannel) ic;
            }
            if (serverSock == null) {
                throw new IllegalArgumentException(sm.getString("endpoint.init.bind.inherited"));
            }
        }
        // 设为阻塞模式
        //这里为什么要设置成阻塞呢，Tomcat的设计初衷主要是为了操作方便。这样这里就跟BIO模式下一样了。只不过在BIO下这里返回的是Socket，NIO下这里返回的是SocketChannel。
        serverSock.configureBlocking(true); //mimic APR behavior

        // Initialize thread count defaults for acceptor, poller
        if (acceptorThreadCount == 0) {
            // FIXME: Doesn't seem to work that well with multiple accept threads
            acceptorThreadCount = 1;
        }
        if (pollerThreadCount <= 0) {
            //minimum one poller thread
            pollerThreadCount = 1;
        }
        setStopLatch(new CountDownLatch(pollerThreadCount));

        // Initialize SSL if needed
        initialiseSsl();
		// 打开阻塞模式的selector
        selectorPool.open();
    }
```

在bind()方法中，首先打开serverSocketChannel，并绑定到监听端口，此处将其该channel设置为阻塞模式。对于SSL部分，此处略过不讲。在最后的 selectorPool.open()执行语句中，会先获得共享的selector，并且创建线程在该selector上检测事件。

```java
//<<AbstractEndPoint>>
public final void start() throws Exception {
        if (bindState == BindState.UNBOUND) {
            bind();
            bindState = BindState.BOUND_ON_START;
        }
        startInternal();//调用子类startInternal方法初始化启动
    }

//<<NioEndpoint>>
/**
     * Start the NIO endpoint, creating acceptor, poller threads.
     */
    @Override
    public void startInternal() throws Exception {

        if (!running) {
            running = true;
            paused = false;

            processorCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                    socketProperties.getProcessorCache());
            eventCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                            socketProperties.getEventCache());
            nioChannels = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                    socketProperties.getBufferPool());

            // Create worker collection
            if ( getExecutor() == null ) {
                createExecutor();
            }
			// 初始化计数器Latch
            initializeConnectionLatch();

            // 创建Poller线程
            // Start poller threads
            pollers = new Poller[getPollerThreadCount()];
            for (int i=0; i<pollers.length; i++) {
                pollers[i] = new Poller();
                Thread pollerThread = new Thread(pollers[i], getName() + "-ClientPoller-"+i);
                pollerThread.setPriority(threadPriority);
                pollerThread.setDaemon(true);
                pollerThread.start();
            }
			// 创建Acceptor线程
            startAcceptorThreads();
        }
    }
```

在startInternal()方法中，最重要的是创建Poller和Acceptor线程。Acceptor线程处理serverSocketChannel的请求接收事件；Poller处理serverSocketChannel的读写事件。此时可以预想到，Acceptor线程专门负责接收客户端连接socketChannel，然后将socketChannel交给Poller线程读写。在实际中，Poller线程将socketChannel再次封装之后又开启另一个线程进行实际的数据处理。这样设计的目的是避免当某一个请求出现阻塞的时候，影响到整个服务器的接收、处理能力。 按接收请求，处理请求的逻辑，我们先观察Acceptor线程。

```java

// --------------------------------------------------- Acceptor Inner Class
    /**
     * The background thread that listens for incoming TCP/IP connections and
     * hands them off to an appropriate processor.
     */
    protected class Acceptor extends AbstractEndpoint.Acceptor {

        @Override
        public void run() {

            int errorDelay = 0;
			// 一直循环直到接收停止命令
            // Loop until we receive a shutdown command
            while (running) {

                // Loop if endpoint is paused
                while (paused && running) {
                    state = AcceptorState.PAUSED;
                    try {
                        Thread.sleep(50);
                    } catch (InterruptedException e) {
                        // Ignore
                    }
                }

                if (!running) {
                    break;
                }
                state = AcceptorState.RUNNING;

                try {
                    //通过同步计数器来限制连接数目
                    //当连接数目超过上限时，则等待
                    //其中同步计算器是通过继承AQS实现的
                    //默认的最大连接数是10000
                    //if we have reached max connections, wait
                    countUpOrAwaitConnection();

                    SocketChannel socket = null;
                    try {
                        // Accept the next incoming connection from the server
                        // socket
                        //接收连接，此处并不是使用selector实现,在前面的代码中已知serverSock是阻塞模式的。
                        socket = serverSock.accept();
                    } catch (IOException ioe) {
                        // We didn't get a socket
                        countDownConnection();
                        if (running) {
                            // Introduce delay if necessary
                            errorDelay = handleExceptionWithDelay(errorDelay);
                            // re-throw
                            throw ioe;
                        } else {
                            break;
                        }
                    }
                    // Successful accept, reset the error delay
                    errorDelay = 0;

                    // Configure the socket
                    if (running && !paused) {
                        // setSocketOptions() will hand the socket off to
                        // an appropriate processor if successful
                        // 在setSocketOptions中将接收到的socket传给poller线程进行处理
                        if (!setSocketOptions(socket)) {
                            closeSocket(socket);
                        }
                    } else {
                        closeSocket(socket);
                    }
                } catch (Throwable t) {
                    ExceptionUtils.handleThrowable(t);
                    log.error(sm.getString("endpoint.accept.fail"), t);
                }
            }
            state = AcceptorState.ENDED;
        }


        private void closeSocket(SocketChannel socket) {
            countDownConnection();
            try {
                socket.socket().close();
            } catch (IOException ioe)  {
                if (log.isDebugEnabled()) {
                    log.debug(sm.getString("endpoint.err.close"), ioe);
                }
            }
            try {
                socket.close();
            } catch (IOException ioe) {
                if (log.isDebugEnabled()) {
                    log.debug(sm.getString("endpoint.err.close"), ioe);
                }
            }
        }
    }

protected boolean setSocketOptions(SocketChannel socket) {
        // Process the connection
        try {
            //disable blocking, APR style, we are gonna be polling it
            // 设置为非阻塞模式
            socket.configureBlocking(false);
            Socket sock = socket.socket();
            socketProperties.setProperties(sock);
			// 从NioChannel容器中获得一个NioChannel
            // NioChannel可以理解为socketChannel的代理类，提供更多的功能
            NioChannel channel = nioChannels.pop();
            if (channel == null) {
                // SocketBufferHandler维护了在处理过程中的读写缓存
                SocketBufferHandler bufhandler = new SocketBufferHandler(
                        socketProperties.getAppReadBufSize(),
                        socketProperties.getAppWriteBufSize(),
                        socketProperties.getDirectBuffer());
                if (isSSLEnabled()) {
                    channel = new SecureNioChannel(socket, bufhandler, selectorPool, this);
                } else {
                    // 将socket、bufHandler封装到NioChannel中
                    channel = new NioChannel(socket, bufhandler);
                }
            } else {
                channel.setIOChannel(socket);
                channel.reset();
            }
            // 将niochannel注册到poller线程中进行处理
            getPoller0().register(channel);
        } catch (Throwable t) {
            ExceptionUtils.handleThrowable(t);
            try {
                log.error("",t);
            } catch (Throwable tt) {
                ExceptionUtils.handleThrowable(tt);
            }
            // Tell to close the socket
            return false;
        }
        return true;
    }
```

### poller和pollerEvent

待补充

![](https://raw.githubusercontent.com/haominglfs/images/master/20190826152622.png)



```java
public void run() {
            // Loop until destroy() is called
    		// 循环直到destroy()方法被调用
            while (true) {
                boolean hasEvents = false;
                try {
                    if (!close) {
                        hasEvents = events();
                        // wakeupCounter > 0，表示有事件，故直接用selectNow，否则用												select(selectorTimeout)以阻塞一段时间等待事件到来
                        if (wakeupCounter.getAndSet(-1) > 0) {
                            //if we are here, means we have other stuff to do
                            //do a non blocking select
                            keyCount = selector.selectNow();
                        } else {
                            keyCount = selector.select(selectorTimeout);
                        }
                        wakeupCounter.set(0);
                    }
                    // 关闭
                    if (close) {
                        events();
                        timeout(0, false);
                        try {
                            selector.close();
                        } catch (IOException ioe) {
                            log.error(sm.getString("endpoint.nio.selectorCloseFail"), ioe);
                        }
                        break;
                    }
                } catch (Throwable x) {
                    ExceptionUtils.handleThrowable(x);
                    log.error("",x);
                    continue;
                }
                //either we timed out or we woke up, process events first
                 // 执行队列中的PollerEvent事件，注册读或写，
                 // hasEvents表示是否有读写事件注册
                if ( keyCount == 0 ) hasEvents = (hasEvents | events());

                Iterator<SelectionKey> iterator =
                    keyCount > 0 ? selector.selectedKeys().iterator() : null;
                // Walk through the collection of ready keys and dispatch
                // any active event.
                while (iterator != null && iterator.hasNext()) {
                    SelectionKey sk = iterator.next();
                    NioSocketWrapper attachment = (NioSocketWrapper)sk.attachment();
                    // Attachment may be null if another thread has called
                    // cancelledKey()
                    if (attachment == null) {
                        iterator.remove();
                    } else {
                        iterator.remove();
                        // 将sk和attachtment包装，交由后续线程继续处理
                        processKey(sk, attachment);
                    }
                }//while

                //process timeouts
                timeout(keyCount,hasEvents);
            }//while

            getStopLatch().countDown();
        }
//在events()方法中，通过调用PollerEvent的run()方法将socket注册到selector中
public boolean events() {
            boolean result = false;
            PollerEvent pe = null;
    		// 从队列中获得PollerEvent事件
            for (int i = 0, size = events.size(); i < size && (pe = events.poll()) != null; i++ ) {
                result = true;
                try {
                    // 调用PollerEvent的run()方法执行事件注册
                    pe.run();
                    pe.reset();
                    if (running && !paused) {
                        eventCache.push(pe);
                    }
                } catch ( Throwable x ) {
                    log.error("",x);
                }
            }

            return result;
        }

public void run() {
    // 如果是注册，则把socket注册到selector中
            if (interestOps == OP_REGISTER) {
                try {
                    socket.getIOChannel().register(
                            socket.getPoller().getSelector(), SelectionKey.OP_READ, socketWrapper);
                } catch (Exception x) {
                    log.error(sm.getString("endpoint.nio.registerFail"), x);
                }
            } else {
                final SelectionKey key = socket.getIOChannel().keyFor(socket.getPoller().getSelector());
                try {
                    if (key == null) {
                        // The key was cancelled (e.g. due to socket closure)
                        // and removed from the selector while it was being
                        // processed. Count down the connections at this point
                        // since it won't have been counted down when the socket
                        // closed.
                        socket.socketWrapper.getEndpoint().countDownConnection();
                        ((NioSocketWrapper) socket.socketWrapper).closed = true;
                    } else {
                        final NioSocketWrapper socketWrapper = (NioSocketWrapper) key.attachment();
                        if (socketWrapper != null) {
                            //we are registering the key to start with, reset the fairness counter.
                            int ops = key.interestOps() | interestOps;
                            socketWrapper.interestOps(ops);
                            key.interestOps(ops);
                        } else {
                            socket.getPoller().cancelledKey(key);
                        }
                    }
                } catch (CancelledKeyException ckx) {
                    try {
                        socket.getPoller().cancelledKey(key);
                    } catch (Exception ignore) {}
                }
            }
        }

//processKey()方法处理准备完毕的事件
protected void processKey(SelectionKey sk, NioSocketWrapper attachment) {
            try {
                // 如果close，则取消sk
                if ( close ) {
                    cancelledKey(sk);
                } else if ( sk.isValid() && attachment != null ) {
                    if (sk.isReadable() || sk.isWritable() ) {
                        if ( attachment.getSendfileData() != null ) {
                            // 处理文件
                            processSendfile(sk,attachment, false);
                        } else {
                            unreg(sk, attachment, sk.readyOps());
                            boolean closeSocket = false;
                            // Read goes before write
                            if (sk.isReadable()) {// 处理可读
                                if (!processSocket(attachment, SocketEvent.OPEN_READ, true)) {
                                    closeSocket = true;
                                }
                            }
                            if (!closeSocket && sk.isWritable()) { //处理可写
                                if (!processSocket(attachment, SocketEvent.OPEN_WRITE, true)) {
                                    closeSocket = true;
                                }
                            }
                            if (closeSocket) {
                                cancelledKey(sk);
                            }
                        }
                    }
                } else {
                    //invalid key
                    cancelledKey(sk);
                }
            } catch ( CancelledKeyException ckx ) {
                cancelledKey(sk);
            } catch (Throwable t) {
                ExceptionUtils.handleThrowable(t);
                log.error("",t);
            }
        }

// 将selectionKey包装为SocketProcessor 
public boolean processSocket(SocketWrapperBase<S> socketWrapper,
            SocketEvent event, boolean dispatch) {
        try {
            if (socketWrapper == null) {
                return false;
            }
            SocketProcessorBase<S> sc = processorCache.pop();
            if (sc == null) {
                sc = createSocketProcessor(socketWrapper, event);
            } else {
                sc.reset(socketWrapper, event);
            }
            Executor executor = getExecutor();
            if (dispatch && executor != null) {
                // 交给线程池处理或直接运行
                executor.execute(sc);
            } else {
                sc.run();
            }
        } catch (RejectedExecutionException ree) {
            getLog().warn(sm.getString("endpoint.executor.fail", socketWrapper) , ree);
            return false;
        } catch (Throwable t) {
            ExceptionUtils.handleThrowable(t);
            // This means we got an OOM or similar creating a thread, or that
            // the pool and its queue are full
            getLog().error(sm.getString("endpoint.process.fail"), t);
            return false;
        }
        return true;
    }
```

![](https://raw.githubusercontent.com/haominglfs/images/master/20190827095247.png)

![](https://raw.githubusercontent.com/haominglfs/images/master/20190827095432.png)



#### LifeCycle接口

![](https://raw.githubusercontent.com/haominglfs/images/master/20190821095719.png)