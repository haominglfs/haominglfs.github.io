---
title: hibernate的session管理方式
date: 2018-12-14 20:21:26
tags:
---

1. Hibernate 自身提供了三种管理 Session对象的方法：

   * Session 对象的生命周期与本地线程绑定。
   * –Session对象的生命周期与JTA事务绑定。
   * –Hibernate委托程序管理Session对象的生命周期。

2. 在Hibernate的配置文件中,hibernate.current_session_context_class属性用于指定Session管理方式,可选值包括：

   * thread:Session 对象的生命周期与本地线程绑定。
   * jta:Session 对象的生命周期与 JTA 事务绑定。
   * managed:Hibernate 委托程序来管理 Session对象的生命周期。

3. 配置与使用

   ```xml
   <!-- 配置管理 Session 的方式 -->
   <property name="current_session_context_class">thread</property>
   ```

   ```java
   public class HibernateUtils {
   	
   	private HibernateUtils(){}
   	
   	private static HibernateUtils instance = new HibernateUtils();
   	
   	public static HibernateUtils getInstance() {
   		return instance;
   	}
   
   	private SessionFactory sessionFactory;
   
   	public SessionFactory getSessionFactory() {
   		if (sessionFactory == null) {
   			Configuration configuration = new Configuration().configure();
   			ServiceRegistry serviceRegistry = new ServiceRegistryBuilder()
   					.applySettings(configuration.getProperties())
   					.buildServiceRegistry();
   			sessionFactory = configuration.buildSessionFactory(serviceRegistry);
   		}
   		return sessionFactory;
   	}
   	
   	public Session getSession(){
   		return getSessionFactory().getCurrentSession();
   	}
   }
   
   ==============================================================
   public class DepartmentDao {
   
   	public void save(Department dept){
   		//内部获取 Session 对象
   		//获取和当前线程绑定的 Session 对象
   		//1. 不需要从外部传入 Session 对象
   		//2. 多个 DAO 方法也可以使用一个事务!
   		Session session = HibernateUtils.getInstance().getSession();
   		System.out.println(session.hashCode());
   		
   		session.save(dept);
   	}
   	
   	/**
   	 * 若需要传入一个 Session 对象, 则意味着上一层(Service)需要获取到 Session 对象.
   	 * 这说明上一层需要和 Hibernate 的 API 紧密耦合. 所以不推荐使用此种方式. 
   	 */
   	public void save(Session session, Department dept){
   		session.save(dept);
   	}
   }
   ================================================================
   @Test
       public void testManageSession(){
   
       //获取 Session
       //开启事务
       Session session = HibernateUtils.getInstance().getSession();
       System.out.println("-->" + session.hashCode());
       Transaction transaction = session.beginTransaction();
   
       DepartmentDao departmentDao = new DepartmentDao();
   
       Department dept = new Department();
       dept.setName("ATGUIGU");
   
       departmentDao.save(dept);
       departmentDao.save(dept);
       departmentDao.save(dept);
   
       //若 Session 是由 thread 来管理的, 则在提交或回滚事务时, 已经关闭 Session 了. 
       transaction.commit();
       System.out.println(session.isOpen()); 
   }
   ```

4. 如果把 Hibernate 配置文件的 hibernate.current_session_context_class 属性值设为 thread, Hibernate 就会按照与本地线程绑定的方式来管理 Session，Hibernate 按一下规则把 Session 与本地线程绑定：

   * 当一个线程(threadA)第一次调用SessionFactory对象的getCurrentSession()方法时,该方法会创建一个新的Session(sessionA)对象,把该对象与threadA绑定,并将sessionA返回。
   * 当threadA再次调用SessionFactory对象的getCurrentSession()方法时,该方法将返回sessionA对象。
   * 当 threadA提交sessionA对象关联的事务时,Hibernate 会自动flushsessionA对象的缓存,然后提交事务,关闭sessionA对象.当threadA撤销sessionA对象关联的事务时,也会自动关闭sessionA对象。
   * 若 threadA再次调用SessionFactory对象的getCurrentSession()方法时,该方法会又创建一个新的Session(sessionB)对象,把该对象与threadA绑定,并将sessionB返回。

5. 批量操作

   推荐使用原生的JDBC方式。