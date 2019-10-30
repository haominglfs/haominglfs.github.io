---
title: hibernate缓存
date: 2018-12-13 12:59:10
tags:
---

1. 缓存级别

   * 第一级别的缓存是Session级别的缓存，它是属于事务范围的缓存。这一级别的缓存由hibernate管理的。
   * 第二级别的缓存是SessionFactory级别的缓存，它是属于进程范围的缓存，sessionFactory级别的缓存分为两类：
     * 内置缓存: Hibernate 自带的, 不可卸载.通常在Hibernate的初始化阶段,Hibernate 会把映射元数据和预定义的 SQL语句放到SessionFactory的缓存中,映射元数据是映射文件中数据（.hbm.xml文件中的数据）的复制.该内置缓存是只读的。
     
     * 外置缓存(二级缓存):一个可配置的缓存插件.在默认情况下,SessionFactory不会启用这个缓存插件.外置缓存中的数据是数据库数据的复制,外置缓存的物理介质可以是内存或硬盘。
     
       <!--more-->

2. 二级缓存的适用场景

   * 适合放入二级缓存中的数据：
     * 很少被修改
     * 不是很重要的数据,允许出现偶尔的并发问题
   * 不适合放入二级缓存中的数据：
     * 经常被修改
     * 财务数据,绝对不允许出现并发问题
     * 与其他应用程序共享的数据

3. 管理二级缓存，二级缓存是可配置的的插件,Hibernate 允许选用以下类型的缓存插件:

   * EHCache: 可作为进程范围内的缓存,存放数据的物理介质可以是内存或硬盘,对Hibernate的查询缓存提供了支持。
   * 其他看官方文档。

4. 使用二级缓存的步骤:

   1. 加入二级缓存插件的 jar 包及配置文件:

      * 复制 \hibernate-release-4.2.4.Final\lib\optional\ehcache\*.jar 到当前 Hibrenate 应用的类路径下
      * 复制 \hibernate-release-4.2.4.Final\project\etc\ehcachexml 到当前 WEB 应用的类路径下

   2. 配置 hibernate.cfg.xml

      ```xml
      <!-- 启用二级缓存 -->
      <property name="cache.use_second_level_cache">true</property>
      <!-- 配置使用的二级缓存的产品 -->
      <property name="hibernate.cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory</property>
      <!-- 配置对哪些类使用 hibernate 的二级缓存 (实际上也可以在 .hbm.xml 文件中配置对哪些类使用二级缓存, 及二级缓存的策略是什么)-->
      <class-cache usage="read-write" class="com.atguigu.hibernate.entities.Employee"/>
      ===============================集合的二级缓存
      <collection-cache usage="read-write" collection="com.atguigu.hibernate.entities.Department.emps"/>
      
      <!--也可以在 .hbm.xml 文件中进行配置-->
      <set name="emps" table="GG_EMPLOYEE" inverse="true" lazy="true">
      	<cache usage="read-write"/>
          <key>
              <column name="DEPT_ID" />
          </key>
          <one-to-many class="com.atguigu.hibernate.entities.Employee" />
      </set>
      <!--注意: 还需要配置集合中的元素对应的持久化类也使用二级缓存! 否则将会多出 n 条 SQL 语句. -->
      =================================查询缓存
      <!-- 查询缓存: 默认情况下, 设置的缓存对 HQL 及 QBC 查询时无效的, 但可以通过以下方式使其是有效的  -->
      <property name="cache.use_query_cache">true</property>
      <!-- 调用 Query 或 Criteria 的 setCacheable(true) 方法  查询缓存依赖于二级缓存 -->
      ```

   3. ehcache 的配置文件: ehcache.xml

      ```xml
      <ehcache>
      
          <!--  
          	指定一个目录：当 EHCache 把数据写到硬盘上时, 将把数据写到这个目录下.
          -->     
          <diskStore path="d:\\tempDirectory"/>
      
          <!--  
          	设置缓存的默认数据过期策略 
          -->    
          <defaultCache
              maxElementsInMemory="10000"
              eternal="false"
              timeToIdleSeconds="120"
              timeToLiveSeconds="120"
              overflowToDisk="true"
              />
      
         	<!--  
         		设定具体的命名缓存的数据过期策略。每个命名缓存代表一个缓存区域
         		缓存区域(region)：一个具有名称的缓存块，可以给每一个缓存块设置不同的缓存策略。
         		如果没有设置任何的缓存区域，则所有被缓存的对象，都将使用默认的缓存策略。即：<defaultCache.../>
         		Hibernate 在不同的缓存区域保存不同的类/集合。
      			对于类而言，区域的名称是类名。如:com.atguigu.domain.Customer
      			对于集合而言，区域的名称是类名加属性名。如com.atguigu.domain.Customer.orders
         	-->
         	<!--  
         		name: 设置缓存的名字,它的取值为类的全限定名或类的集合的名字 
      		maxElementsInMemory: 设置基于内存的缓存中可存放的对象最大数目 
      		
      		eternal: 设置对象是否为永久的, true表示永不过期,
      		此时将忽略timeToIdleSeconds 和 timeToLiveSeconds属性; 默认值是false 
      		timeToIdleSeconds:设置对象空闲最长时间,以秒为单位, 超过这个时间,对象过期。
      		当对象过期时,EHCache会把它从缓存中清除。如果此值为0,表示对象可以无限期地处于空闲状态。 
      		timeToLiveSeconds:设置对象生存最长时间,超过这个时间,对象过期。
      		如果此值为0,表示对象可以无限期地存在于缓存中. 该属性值必须大于或等于 timeToIdleSeconds 属性值 
      		
      		overflowToDisk:设置基于内存的缓存中的对象数目达到上限后,是否把溢出的对象写到基于硬盘的缓存中 
         	-->
          <cache name="com.atguigu.hibernate.entities.Employee"
              maxElementsInMemory="1"
              eternal="false"
              timeToIdleSeconds="300"
              timeToLiveSeconds="600"
              overflowToDisk="true"
              />
      
          <cache name="com.atguigu.hibernate.entities.Department.emps"
              maxElementsInMemory="1000"
              eternal="true"
              timeToIdleSeconds="0"
              timeToLiveSeconds="0"
              overflowToDisk="false"
              />
      
      </ehcache>
      ```

