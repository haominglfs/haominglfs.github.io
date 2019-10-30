---
title: tomcat源码4-Pipeline-Value管道
date: 2019-09-02 15:04:43
tags:
---

![](https://raw.githubusercontent.com/haominglfs/images/master/20190902163506.png)

<!--more-->

每个容器（Engine/Host/Context/Wrap）包含一个pipeline，每个pipeline包含一个valve集合，位于前面的valve做完业务处理后将调用后面的valve做业务处理，而容器的缺省valve位于集合的最后一个位置，负责调用下层容器的pipeline的第一个valve做请求处理。调用会从Engine的第一个valve调用开始，一直执行到调用Wrapper的缺省valve：StandardWrapperValve，而filter与servlet的处理就是在这个valve中进行的 。Engine的第一个valve是由Adapter调用的，在connector章节中也看到CoyoteAdapter在处理完request以后会执行`connector.getContainer().getPipeline().getFirst().invoke(request, response)`。

```java
<<ContainerBase>>
/**
     * The Pipeline object with which this Container is associated.
     */
    protected final Pipeline pipeline = new StandardPipeline(this);

    @Override
    protected synchronized void startInternal() throws LifecycleException {
       	.......
        // Start the Valves in our pipeline (including the basic), if any
        if (pipeline instanceof Lifecycle) {
            ((Lifecycle) pipeline).start();
        }
				.......
    }
```

