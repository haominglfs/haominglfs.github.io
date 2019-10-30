---
title: jpa
date: 2018-12-12 14:15:40
tags:
---

* persistence.xml

  JPA规范要求在类路径的META-INF目录下放置persistence.xml，文件的名称是固定的。

  <!--more-->

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <persistence version="2.0"
  	xmlns="http://java.sun.com/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  	xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd">
    	<!-- 1.name 属性用于定义持久化单元的名字, 必选 
  		 2.transaction-type：指定 JPA  的事务处理策略。
  			RESOURCE_LOCAL：默认值，数据库级别的事务，只能针对一种数据库，不支持分布式事务。
  			如果需要支持分布式事务，使用JTA：transaction-type="JTA“
  	-->  
  	<persistence-unit name="jpa-1" transaction-type="RESOURCE_LOCAL">
  		<!-- 
  		配置使用什么 ORM 产品来作为 JPA 的实现 
  		1. 实际上配置的是  javax.persistence.spi.PersistenceProvider 接口的实现类
  		2. 若 JPA 项目中只有一个 JPA 的实现产品, 则也可以不配置该节点. 
  		-->
  		<provider>org.hibernate.ejb.HibernatePersistence</provider>
  	
  		<!-- 添加持久化类 -->
  		<class>com.atguigu.jpa.helloworld.Customer</class>
  		<class>com.atguigu.jpa.helloworld.Order</class>
  	
  		<class>com.atguigu.jpa.helloworld.Department</class>
  		<class>com.atguigu.jpa.helloworld.Manager</class>
  	
  		<class>com.atguigu.jpa.helloworld.Item</class>
  		<class>com.atguigu.jpa.helloworld.Category</class>
  		
  		<!-- 
  		配置二级缓存的策略 
  		ALL：所有的实体类都被缓存
  		NONE：所有的实体类都不被缓存. 
  		ENABLE_SELECTIVE：标识 @Cacheable(true) 注解的实体类将被缓存
  		DISABLE_SELECTIVE：缓存除标识 @Cacheable(false) 以外的所有实体类
  		UNSPECIFIED：默认值，JPA 产品默认值将被使用
  		-->
  		<shared-cache-mode>ENABLE_SELECTIVE</shared-cache-mode>
  	
  		<properties>
  			<!-- 连接数据库的基本信息 -->
  			<property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver"/>
  			<property name="javax.persistence.jdbc.url" value="jdbc:mysql:///jpa"/>
  			<property name="javax.persistence.jdbc.user" value="root"/>
  			<property name="javax.persistence.jdbc.password" value="1230"/>
  			
  			<!-- 配置 JPA 实现产品的基本属性. 配置 hibernate 的基本属性 -->
  			<property name="hibernate.format_sql" value="true"/>
  			<property name="hibernate.show_sql" value="true"/>
  			<property name="hibernate.hbm2ddl.auto" value="update"/>
  			
  			<!-- 二级缓存相关 -->
  			<property name="hibernate.cache.use_second_level_cache" value="true"/>
  			<property name="hibernate.cache.region.factory_class" value="org.hibernate.cache.ehcache.EhCacheRegionFactory"/>
  			<property name="hibernate.cache.use_query_cache" value="true"/>
  		</properties>
  	</persistence-unit>
  </persistence>
  ```

* 实体类

  ```java
  @NamedQuery(name="testNamedQuery", query="FROM Customer c WHERE c.id = ?")
  @Cacheable(true)
  @Table(name="JPA_CUTOMERS")
  @Entity
  public class Customer {
  
  	private Integer id;
  	private String lastName;
  
  	private String email;
  	private int age;
  	
  	private Date createdTime;
  	private Date birth;
  	
  	public Customer() {
  		// TODO Auto-generated constructor stub
  	}
  	
  	public Customer(String lastName, int age) {
  		super();
  		this.lastName = lastName;
  		this.age = age;
  	}
  
  
  
  	private Set<Order> orders = new HashSet<>();
  
  //	@TableGenerator(name="ID_GENERATOR",
  //			table="jpa_id_generators",
  //			pkColumnName="PK_NAME",
  //			pkColumnValue="CUSTOMER_ID",
  //			valueColumnName="PK_VALUE",
  //			allocationSize=100)
  //	@GeneratedValue(strategy=GenerationType.TABLE,generator="ID_GENERATOR")
  	@GeneratedValue(strategy=GenerationType.AUTO)
  	@Id
  	public Integer getId() {
  		return id;
  	}
  
  	public void setId(Integer id) {
  		this.id = id;
  	}
  
  	@Column(name="LAST_NAME",length=50,nullable=false)
  	public String getLastName() {
  		return lastName;
  	}
  
  	public void setLastName(String lastName) {
  		this.lastName = lastName;
  	}
  
  	public String getEmail() {
  		return email;
  	}
  
  	public void setEmail(String email) {
  		this.email = email;
  	}
  
  	public int getAge() {
  		return age;
  	}
  
  	public void setAge(int age) {
  		this.age = age;
  	}
  	
  	@Temporal(TemporalType.TIMESTAMP)
  	public Date getCreatedTime() {
  		return createdTime;
  	}
  
  	public void setCreatedTime(Date createdTime) {
  		this.createdTime = createdTime;
  	}
  
  	@Temporal(TemporalType.DATE)
  	public Date getBirth() {
  		return birth;
  	}
  
  	public void setBirth(Date birth) {
  		this.birth = birth;
  	}
  	
  	//映射单向 1-n 的关联关系
  	//使用 @OneToMany 来映射 1-n 的关联关系
  	//使用 @JoinColumn 来映射外键列的名称
  	//可以使用 @OneToMany 的 fetch 属性来修改默认的加载策略
  	//可以通过 @OneToMany 的 cascade 属性来修改默认的删除策略. 
  	//注意: 若在 1 的一端的 @OneToMany 中使用 mappedBy 属性, 则 @OneToMany 端就不能再使用 @JoinColumn 属性了. 
  //	@JoinColumn(name="CUSTOMER_ID")
  	@OneToMany(fetch=FetchType.LAZY,cascade={CascadeType.REMOVE},mappedBy="customer")
  	public Set<Order> getOrders() {
  		return orders;
  	}
  
  	public void setOrders(Set<Order> orders) {
  		this.orders = orders;
  	}
  
  	//工具方法. 不需要映射为数据表的一列. 
  	@Transient
  	public String getInfo(){
  		return "lastName: " + lastName + ", email: " + email;
  	}
  
  	@Override
  	public String toString() {
  		return "Customer [id=" + id + ", lastName=" + lastName + ", email="
  				+ email + ", age=" + age + ", createdTime=" + createdTime
  				+ ", birth=" + birth + "]";
  	}
  
  }
  ```

* 执行持久化操作流程

  ```java
  		//1. 创建 EntitymanagerFactory
  		String persistenceUnitName = "jpa-1";
  		
  		//可选
  		//Map<String, Object> properites = new HashMap<String, Object>();
  		//properites.put("hibernate.show_sql", true);
  		
  		EntityManagerFactory entityManagerFactory = 
  				Persistence.createEntityManagerFactory(persistenceUnitName);
  				//Persistence.createEntityManagerFactory(persistenceUnitName, properites);
  				
  		//2. 创建 EntityManager. 类似于 Hibernate 的 SessionFactory
  		EntityManager entityManager = entityManagerFactory.createEntityManager();
  		
  		//3. 开启事务
  		EntityTransaction transaction = entityManager.getTransaction();
  		transaction.begin();
  		
  		//4. 进行持久化操作
  		Customer customer = new Customer();
  		customer.setAge(12);
  		customer.setEmail("tom@atguigu.com");
  		customer.setLastName("Tom");
  		customer.setBirth(new Date());
  		customer.setCreatedTime(new Date());
  		
  		entityManager.persist(customer);
  		
  		//5. 提交事务
  		transaction.commit();
  	
  		//6. 关闭 EntityManager
  		entityManager.close();
  		
  		//7. 关闭 EntityManagerFactory
  		entityManagerFactory.close();
  ```

* 基本注解

  1. @Entity
     标注用于实体类声明语句之前，指出该Java类为实体类，将映射到指定的数据库表。如声明一个实体类Customer，它将映射到数据库中的customer表上。

  2. @Table

     * 当实体类与其映射的数据库表名不同名时需要使用 @Table 标注说明，该标注与 @Entity 标注并列使用，置于实体类声明语句之前，可写于单独语句行，也可与声明语句同行。

     * @Table 标注的常用选项是 name，用于指明数据库的表名

     * @Table标注还有一个两个选项 catalog 和 schema 用于设置表所属的数据库目录或模式，通常为数据库名。uniqueConstraints选项用于设置约束条件，通常不须设置。

  3. @Id

     * @Id标注用于声明一个实体类的属性映射为数据库的主键列。该属性通常置于属性声明语句之前，可与声明语句同行，也可写在单独行上。
     * •@Id标注也可置于属性的getter方法之前。

  4. @GeneratedValue

     * @GeneratedValue  用于标注主键的生成策略，通过 strategy属性指定。默认情况下，JPA自动选择一个最适合底层数据库的主键生成策略：SqlServer对应identity，MySQL对应auto increment。

     * 在 javax.persistence.GenerationType 中定义了以下几种可供选择的策略：

       –IDENTITY：采用数据库 ID自增长的方式来自增主键字段，Oracle 不支持这种方式；

       –AUTO： JPA自动选择合适的策略，是默认选项；

       –SEQUENCE：通过序列产生主键，通过 @SequenceGenerator 注解指定序列名，MySql 不支持这种方式

       –TABLE：通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植。

  5. @Basic

     * @Basic 表示一个简单的属性到数据库表的字段的映射,对于没有任何标注的 getXxxx() 方法,默认即为@Basic
     * fetch: 表示该属性的读取策略,有EAGER 和 LAZY两种,分别表示主支抓取和延迟加载,默认为EAGER.
     * optional:表示该属性是否允许为null,默认为true

  6. @Column

     * 当实体的属性与其映射的数据库表的列不同名时需要使用@Column标注说明，该属性通常置于实体的属性声明语句之前，还可与@Id标注一起使用。
     * @Column标注的常用属性是name，用于设置映射数据库表的列名。此外，该标注还包含其它多个属性，如：unique 、nullable、length 等。
     * @Column标注的columnDefinition属性:表示该字段在数据库中的实际类型.通常ORM框架可以根据属性类型自动判断数据库中字段的类型,但是对于Date类型仍无法确定数据库中字段类型究竟是DATE,TIME还是TIMESTAMP.此外,String的默认映射类型为VARCHAR,如果要将String类型映射到特定数据库的BLOB或TEXT字段类型。
     * @Column标注也可置于属性的getter方法之前。

  7. @Transient

     * 表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性.
     * 如果一个属性并非数据库表的字段映射,就务必将其标示为@Transient,否则,ORM框架默认其注解为@Basic

  8. @Temporal

     * 在核心的JavaAPI 中并没有定义Date类型的精度(temporalprecision).  而在数据库中,表示Date类型的数据有DATE,TIME, 和 TIMESTAMP三种精度(即单纯的日期,时间,或者两者兼备). 在进行属性映射时可使用@Temporal注解来调整精度。

* JPA API

  ```java
  public class JPATest {
  
  	private EntityManagerFactory entityManagerFactory;
  	private EntityManager entityManager;
  	private EntityTransaction transaction;
  	
  	@Before
  	public void init(){
  		entityManagerFactory = Persistence.createEntityManagerFactory("jpa-1");
  		entityManager = entityManagerFactory.createEntityManager();
  		transaction = entityManager.getTransaction();
  		transaction.begin();
  	}
  	
  	@After
  	public void destroy(){
  		transaction.commit();
  		entityManager.close();
  		entityManagerFactory.close();
  	}
  	
  	//可以使用 JPQL 完成 UPDATE 和 DELETE 操作. 
  	@Test
  	public void testExecuteUpdate(){
  		String jpql = "UPDATE Customer c SET c.lastName = ? WHERE c.id = ?";
  		Query query = entityManager.createQuery(jpql).setParameter(1, "YYY").setParameter(2, 12);
  		
  		query.executeUpdate();
  	}
  
  	//使用 jpql 内建的函数
  	@Test
  	public void testJpqlFunction(){
  		String jpql = "SELECT lower(c.email) FROM Customer c";
  		
  		List<String> emails = entityManager.createQuery(jpql).getResultList();
  		System.out.println(emails);
  	}
  	
  	@Test
  	public void testSubQuery(){
  		//查询所有 Customer 的 lastName 为 YY 的 Order
  		String jpql = "SELECT o FROM Order o "
  				+ "WHERE o.customer = (SELECT c FROM Customer c WHERE c.lastName = ?)";
  		
  		Query query = entityManager.createQuery(jpql).setParameter(1, "YY");
  		List<Order> orders = query.getResultList();
  		System.out.println(orders.size());
  	}
  	
  	/**
  	 * JPQL 的关联查询同 HQL 的关联查询. 
  	 */
  	@Test
  	public void testLeftOuterJoinFetch(){
  		String jpql = "FROM Customer c LEFT OUTER JOIN FETCH c.orders WHERE c.id = ?";
  		
  		Customer customer = 
  				(Customer) entityManager.createQuery(jpql).setParameter(1, 12).getSingleResult();
  		System.out.println(customer.getLastName());
  		System.out.println(customer.getOrders().size());
  		
  //		List<Object[]> result = entityManager.createQuery(jpql).setParameter(1, 12).getResultList();
  //		System.out.println(result);
  	}
  	
  	//查询 order 数量大于 2 的那些 Customer
  	@Test
  	public void testGroupBy(){
  		String jpql = "SELECT o.customer FROM Order o "
  				+ "GROUP BY o.customer "
  				+ "HAVING count(o.id) >= 2";
  		List<Customer> customers = entityManager.createQuery(jpql).getResultList();
  		
  		System.out.println(customers);
  	}
  	
  	@Test
  	public void testOrderBy(){
  		String jpql = "FROM Customer c WHERE c.age > ? ORDER BY c.age DESC";
  		Query query = entityManager.createQuery(jpql).setHint(QueryHints.HINT_CACHEABLE, true);
  		
  		//占位符的索引是从 1 开始
  		query.setParameter(1, 1);
  		List<Customer> customers = query.getResultList();
  		System.out.println(customers.size());
  	}
  	
  	//使用 hibernate 的查询缓存. 
  	@Test
  	public void testQueryCache(){
  		String jpql = "FROM Customer c WHERE c.age > ?";
  		Query query = entityManager.createQuery(jpql).setHint(QueryHints.HINT_CACHEABLE, true);
  		
  		//占位符的索引是从 1 开始
  		query.setParameter(1, 1);
  		List<Customer> customers = query.getResultList();
  		System.out.println(customers.size());
  		
  		query = entityManager.createQuery(jpql).setHint(QueryHints.HINT_CACHEABLE, true);
  		
  		//占位符的索引是从 1 开始
  		query.setParameter(1, 1);
  		customers = query.getResultList();
  		System.out.println(customers.size());
  	}
  	
  	//createNativeQuery 适用于本地 SQL
  	@Test
  	public void testNativeQuery(){
  		String sql = "SELECT age FROM jpa_cutomers WHERE id = ?";
  		Query query = entityManager.createNativeQuery(sql).setParameter(1, 3);
  		
  		Object result = query.getSingleResult();
  		System.out.println(result);
  	}
  	
  	//createNamedQuery 适用于在实体类前使用 @NamedQuery 标记的查询语句
  	@Test
  	public void testNamedQuery(){
  		Query query = entityManager.createNamedQuery("testNamedQuery").setParameter(1, 3);
  		Customer customer = (Customer) query.getSingleResult();
  		
  		System.out.println(customer);
  	}
  	
  	//默认情况下, 若只查询部分属性, 则将返回 Object[] 类型的结果. 或者 Object[] 类型的 List.
  	//也可以在实体类中创建对应的构造器, 然后再 JPQL 语句中利用对应的构造器返回实体类的对象.
  	@Test
  	public void testPartlyProperties(){
  		String jpql = "SELECT new Customer(c.lastName, c.age) FROM Customer c WHERE c.id > ?";
  		List result = entityManager.createQuery(jpql).setParameter(1, 1).getResultList();
  		
  		System.out.println(result);
  	}
  	
  	@Test
  	public void testHelloJPQL(){
  		String jpql = "FROM Customer c WHERE c.age > ?";
  		Query query = entityManager.createQuery(jpql);
  		
  		//占位符的索引是从 1 开始
  		query.setParameter(1, 1);
  		List<Customer> customers = query.getResultList();
  		System.out.println(customers.size());
  	}
  	
  	@Test
  	public void testSecondLevelCache(){
  		Customer customer1 = entityManager.find(Customer.class, 1);
  		
  		transaction.commit();
  		entityManager.close();
  		
  		entityManager = entityManagerFactory.createEntityManager();
  		transaction = entityManager.getTransaction();
  		transaction.begin();
  		
  		Customer customer2 = entityManager.find(Customer.class, 1);
  	}
  	
  	//对于关联的集合对象, 默认使用懒加载的策略.
  	//使用维护关联关系的一方获取, 还是使用不维护关联关系的一方获取, SQL 语句相同. 
  	@Test
  	public void testManyToManyFind(){
  //		Item item = entityManager.find(Item.class, 5);
  //		System.out.println(item.getItemName());
  //		
  //		System.out.println(item.getCategories().size());
  		
  		Category category = entityManager.find(Category.class, 3);
  		System.out.println(category.getCategoryName());
  		System.out.println(category.getItems().size());
  	}
  	
  	//多对所的保存
  	@Test
  	public void testManyToManyPersist(){
  		Item i1 = new Item();
  		i1.setItemName("i-1");
  	
  		Item i2 = new Item();
  		i2.setItemName("i-2");
  		
  		Category c1 = new Category();
  		c1.setCategoryName("C-1");
  		
  		Category c2 = new Category();
  		c2.setCategoryName("C-2");
  		
  		//设置关联关系
  		i1.getCategories().add(c1);
  		i1.getCategories().add(c2);
  		
  		i2.getCategories().add(c1);
  		i2.getCategories().add(c2);
  		
  		c1.getItems().add(i1);
  		c1.getItems().add(i2);
  		
  		c2.getItems().add(i1);
  		c2.getItems().add(i2);
  		
  		//执行保存
  		entityManager.persist(i1);
  		entityManager.persist(i2);
  		entityManager.persist(c1);
  		entityManager.persist(c2);
  	}
  	
  	//1. 默认情况下, 若获取不维护关联关系的一方, 则也会通过左外连接获取其关联的对象. 
  	//可以通过 @OneToOne 的 fetch 属性来修改加载策略. 但依然会再发送 SQL 语句来初始化其关联的对象
  	//这说明在不维护关联关系的一方, 不建议修改 fetch 属性. 
  	@Test
  	public void testOneToOneFind2(){
  		Manager mgr = entityManager.find(Manager.class, 1);
  		System.out.println(mgr.getMgrName());
  		
  		System.out.println(mgr.getDept().getClass().getName());
  	}
  	
  	//1.默认情况下, 若获取维护关联关系的一方, 则会通过左外连接获取其关联的对象. 
  	//但可以通过 @OntToOne 的 fetch 属性来修改加载策略.
  	@Test
  	public void testOneToOneFind(){
  		Department dept = entityManager.find(Department.class, 1);
  		System.out.println(dept.getDeptName());
  		System.out.println(dept.getMgr().getClass().getName());
  	}
  	
  	//双向 1-1 的关联关系, 建议先保存不维护关联关系的一方, 即没有外键的一方, 这样不会多出 UPDATE 语句.
  	@Test
  	public void testOneToOnePersistence(){
  		Manager mgr = new Manager();
  		mgr.setMgrName("M-BB");
  		
  		Department dept = new Department();
  		dept.setDeptName("D-BB");
  		
  		//设置关联关系
  		mgr.setDept(dept);
  		dept.setMgr(mgr);
  		
  		//执行保存操作
  		entityManager.persist(mgr);
  		entityManager.persist(dept);
  	}
  	
  	@Test
  	public void testUpdate(){
  		Customer customer = entityManager.find(Customer.class, 10);
  		
  		customer.getOrders().iterator().next().setOrderName("O-XXX-10");
  	}
  	
  	//默认情况下, 若删除 1 的一端, 则会先把关联的 n 的一端的外键置空, 然后进行删除. 
  	//可以通过 @OneToMany 的 cascade 属性来修改默认的删除策略. 
  	@Test
  	public void testOneToManyRemove(){
  		Customer customer = entityManager.find(Customer.class, 8);
  		entityManager.remove(customer);
  	}
  	
  	//默认对关联的多的一方使用懒加载的加载策略. 
  	//可以使用 @OneToMany 的 fetch 属性来修改默认的加载策略
  	@Test
  	public void testOneToManyFind(){
  		Customer customer = entityManager.find(Customer.class, 9);
  		System.out.println(customer.getLastName());
  		
  		System.out.println(customer.getOrders().size());
  	}
  	
  	//若是双向 1-n 的关联关系, 执行保存时
  	//若先保存 n 的一端, 再保存 1 的一端, 默认情况下, 会多出 n 条 UPDATE 语句.
  	//若先保存 1 的一端, 则会多出 n 条 UPDATE 语句
  	//在进行双向 1-n 关联关系时, 建议使用 n 的一方来维护关联关系, 而 1 的一方不维护关联系, 这样会有效的减少 SQL 语句. 
  	//注意: 若在 1 的一端的 @OneToMany 中使用 mappedBy 属性, 则 @OneToMany 端就不能再使用 @JoinColumn 属性了. 
  	
  	//单向 1-n 关联关系执行保存时, 一定会多出 UPDATE 语句.
  	//因为 n 的一端在插入时不会同时插入外键列. 
  	@Test
  	public void testOneToManyPersist(){
  		Customer customer = new Customer();
  		customer.setAge(18);
  		customer.setBirth(new Date());
  		customer.setCreatedTime(new Date());
  		customer.setEmail("mm@163.com");
  		customer.setLastName("MM");
  		
  		Order order1 = new Order();
  		order1.setOrderName("O-MM-1");
  		
  		Order order2 = new Order();
  		order2.setOrderName("O-MM-2");
  		
  		//建立关联关系
  		customer.getOrders().add(order1);
  		customer.getOrders().add(order2);
  		
  		order1.setCustomer(customer);
  		order2.setCustomer(customer);
  		
  		//执行保存操作
  		entityManager.persist(customer);
  
  		entityManager.persist(order1);
  		entityManager.persist(order2);
  	}
  	
  	/*
  	@Test
  	public void testManyToOneUpdate(){
  		Order order = entityManager.find(Order.class, 2);
  		order.getCustomer().setLastName("FFF");
  	}
  	
  	//不能直接删除 1 的一端, 因为有外键约束. 
  	@Test
  	public void testManyToOneRemove(){
  //		Order order = entityManager.find(Order.class, 1);
  //		entityManager.remove(order);
  		
  		Customer customer = entityManager.find(Customer.class, 7);
  		entityManager.remove(customer);
  	}
  	
  	//默认情况下, 使用左外连接的方式来获取 n 的一端的对象和其关联的 1 的一端的对象. 
  	//可使用 @ManyToOne 的 fetch 属性来修改默认的关联属性的加载策略
  	@Test
  	public void testManyToOneFind(){
  		Order order = entityManager.find(Order.class, 1);
  		System.out.println(order.getOrderName());
  		
  		System.out.println(order.getCustomer().getLastName());
  	}
  	*/
  	
  	/**
  	 * 保存多对一时, 建议先保存 1 的一端, 后保存 n 的一端, 这样不会多出额外的 UPDATE 语句.
  	 */
  	/*
  	@Test
  	public void testManyToOnePersist(){
  		Customer customer = new Customer();
  		customer.setAge(18);
  		customer.setBirth(new Date());
  		customer.setCreatedTime(new Date());
  		customer.setEmail("gg@163.com");
  		customer.setLastName("GG");
  		
  		Order order1 = new Order();
  		order1.setOrderName("G-GG-1");
  		
  		Order order2 = new Order();
  		order2.setOrderName("G-GG-2");
  		
  		//设置关联关系
  		order1.setCustomer(customer);
  		order2.setCustomer(customer);
  		
  		//执行保存操作
  		entityManager.persist(order1);
  		entityManager.persist(order2);
  		
  		entityManager.persist(customer);
  	}
  	*/
  	
  	/**
  	 * 同 hibernate 中 Session 的 refresh 方法. 
  	 */
  	@Test
  	public void testRefresh(){
  		Customer customer = entityManager.find(Customer.class, 1);
  		customer = entityManager.find(Customer.class, 1);
  		
  		entityManager.refresh(customer);
  	}
  	
  	/**
  	 * 同 hibernate 中 Session 的 flush 方法. 
  	 */
  	@Test
  	public void testFlush(){
  		Customer customer = entityManager.find(Customer.class, 1);
  		System.out.println(customer);
  		
  		customer.setLastName("AA");
  		
  		entityManager.flush();
  	}
  	
  	//若传入的是一个游离对象, 即传入的对象有 OID. 
  	//1. 若在 EntityManager 缓存中有对应的对象
  	//2. JPA 会把游离对象的属性复制到查询到EntityManager 缓存中的对象中.
  	//3. EntityManager 缓存中的对象执行 UPDATE. 
  	@Test
  	public void testMerge4(){
  		Customer customer = new Customer();
  		customer.setAge(18);
  		customer.setBirth(new Date());
  		customer.setCreatedTime(new Date());
  		customer.setEmail("dd@163.com");
  		customer.setLastName("DD");
  		
  		customer.setId(4);
  		Customer customer2 = entityManager.find(Customer.class, 4);
  		
  		entityManager.merge(customer);
  		
  		System.out.println(customer == customer2); //false
  	}
  	
  	//若传入的是一个游离对象, 即传入的对象有 OID. 
  	//1. 若在 EntityManager 缓存中没有该对象
  	//2. 若在数据库中也有对应的记录
  	//3. JPA 会查询对应的记录, 然后返回该记录对一个的对象, 再然后会把游离对象的属性复制到查询到的对象中.
  	//4. 对查询到的对象执行 update 操作. 
  	@Test
  	public void testMerge3(){
  		Customer customer = new Customer();
  		customer.setAge(18);
  		customer.setBirth(new Date());
  		customer.setCreatedTime(new Date());
  		customer.setEmail("ee@163.com");
  		customer.setLastName("EE");
  		
  		customer.setId(4);
  		
  		Customer customer2 = entityManager.merge(customer);
  		
  		System.out.println(customer == customer2); //false
  	}
  	
  	//若传入的是一个游离对象, 即传入的对象有 OID. 
  	//1. 若在 EntityManager 缓存中没有该对象
  	//2. 若在数据库中也没有对应的记录
  	//3. JPA 会创建一个新的对象, 然后把当前游离对象的属性复制到新创建的对象中
  	//4. 对新创建的对象执行 insert 操作. 
  	@Test
  	public void testMerge2(){
  		Customer customer = new Customer();
  		customer.setAge(18);
  		customer.setBirth(new Date());
  		customer.setCreatedTime(new Date());
  		customer.setEmail("dd@163.com");
  		customer.setLastName("DD");
  		
  		customer.setId(100);
  		
  		Customer customer2 = entityManager.merge(customer);
  		
  		System.out.println("customer#id:" + customer.getId());
  		System.out.println("customer2#id:" + customer2.getId());
  	}
  	
  	/**
  	 * 总的来说: 类似于 hibernate Session 的 saveOrUpdate 方法.
  	 */
  	//1. 若传入的是一个临时对象
  	//会创建一个新的对象, 把临时对象的属性复制到新的对象中, 然后对新的对象执行持久化操作. 所以
  	//新的对象中有 id, 但以前的临时对象中没有 id. 
  	@Test
  	public void testMerge1(){
  		Customer customer = new Customer();
  		customer.setAge(18);
  		customer.setBirth(new Date());
  		customer.setCreatedTime(new Date());
  		customer.setEmail("cc@163.com");
  		customer.setLastName("CC");
  		
  		Customer customer2 = entityManager.merge(customer);
  		
  		System.out.println("customer#id:" + customer.getId());
  		System.out.println("customer2#id:" + customer2.getId());
  	}
  	
  	//类似于 hibernate 中 Session 的 delete 方法. 把对象对应的记录从数据库中移除
  	//但注意: 该方法只能移除 持久化 对象. 而 hibernate 的 delete 方法实际上还可以移除 游离对象.
  	@Test
  	public void testRemove(){
  //		Customer customer = new Customer();
  //		customer.setId(2);
  		
  		Customer customer = entityManager.find(Customer.class, 2);
  		entityManager.remove(customer);
  	}
  	
  	//类似于 hibernate 的 save 方法. 使对象由临时状态变为持久化状态. 
  	//和 hibernate 的 save 方法的不同之处: 若对象有 id, 则不能执行 insert 操作, 而会抛出异常. 
  	@Test
  	public void testPersistence(){
  		Customer customer = new Customer();
  		customer.setAge(15);
  		customer.setBirth(new Date());
  		customer.setCreatedTime(new Date());
  		customer.setEmail("bb@163.com");
  		customer.setLastName("BB");
  		customer.setId(100);
  		
  		entityManager.persist(customer);
  		System.out.println(customer.getId());
  	}
  	
  	//类似于 hibernate 中 Session 的 load 方法
  	@Test
  	public void testGetReference(){
  		Customer customer = entityManager.getReference(Customer.class, 1);
  		System.out.println(customer.getClass().getName());
  		
  		System.out.println("-------------------------------------");
  //		transaction.commit();
  //		entityManager.close();
  		
  		System.out.println(customer);
  	}
  	
  	//类似于 hibernate 中 Session 的 get 方法. 
  	@Test
  	public void testFind() {
  		Customer customer = entityManager.find(Customer.class, 1);
  		System.out.println("-------------------------------------");
  		
  		System.out.println(customer);
  	}
  }
  ```

* 单向多对一映射

  ```java
  @Table(name="JPA_ORDERS")
  @Entity
  public class Order {
  
  	private Integer id;
  	private String orderName;
  
  	private Customer customer;
  
  	@GeneratedValue
  	@Id
  	public Integer getId() {
  		return id;
  	}
  
  	public void setId(Integer id) {
  		this.id = id;
  	}
  
  	@Column(name="ORDER_NAME")
  	public String getOrderName() {
  		return orderName;
  	}
  
  	public void setOrderName(String orderName) {
  		this.orderName = orderName;
  	}
  
  	//映射单向 n-1 的关联关系
  	//使用 @ManyToOne 来映射多对一的关联关系
  	//使用 @JoinColumn 来映射外键. 
  	//可使用 @ManyToOne 的 fetch 属性来修改默认的关联属性的加载策略
  	@JoinColumn(name="CUSTOMER_ID")
  	@ManyToOne(fetch=FetchType.LAZY)
  	public Customer getCustomer() {
  		return customer;
  	}
  
  	public void setCustomer(Customer customer) {
  		this.customer = customer;
  	}
  
  }
  ```

* 单向一对多

  ```java
  @NamedQuery(name="testNamedQuery", query="FROM Customer c WHERE c.id = ?")
  @Cacheable(true)
  @Table(name="JPA_CUTOMERS")
  @Entity
  public class Customer {
  
  	private Integer id;
  	private String lastName;
  
  	private String email;
  	private int age;
  	
  	private Date createdTime;
  	private Date birth;
  	
  	public Customer() {
  		// TODO Auto-generated constructor stub
  	}
  	
  	public Customer(String lastName, int age) {
  		super();
  		this.lastName = lastName;
  		this.age = age;
  	}
  
  
  
  	private Set<Order> orders = new HashSet<>();
  
  //	@TableGenerator(name="ID_GENERATOR",
  //			table="jpa_id_generators",
  //			pkColumnName="PK_NAME",
  //			pkColumnValue="CUSTOMER_ID",
  //			valueColumnName="PK_VALUE",
  //			allocationSize=100)
  //	@GeneratedValue(strategy=GenerationType.TABLE,generator="ID_GENERATOR")
  	@GeneratedValue(strategy=GenerationType.AUTO)
  	@Id
  	public Integer getId() {
  		return id;
  	}
  
  	public void setId(Integer id) {
  		this.id = id;
  	}
  
  	@Column(name="LAST_NAME",length=50,nullable=false)
  	public String getLastName() {
  		return lastName;
  	}
  
  	public void setLastName(String lastName) {
  		this.lastName = lastName;
  	}
  
  	public String getEmail() {
  		return email;
  	}
  
  	public void setEmail(String email) {
  		this.email = email;
  	}
  
  	public int getAge() {
  		return age;
  	}
  
  	public void setAge(int age) {
  		this.age = age;
  	}
  	
  	@Temporal(TemporalType.TIMESTAMP)
  	public Date getCreatedTime() {
  		return createdTime;
  	}
  
  	public void setCreatedTime(Date createdTime) {
  		this.createdTime = createdTime;
  	}
  
  	@Temporal(TemporalType.DATE)
  	public Date getBirth() {
  		return birth;
  	}
  
  	public void setBirth(Date birth) {
  		this.birth = birth;
  	}
  	
  	//映射单向 1-n 的关联关系
  	//使用 @OneToMany 来映射 1-n 的关联关系
  	//使用 @JoinColumn 来映射外键列的名称
  	//可以使用 @OneToMany 的 fetch 属性来修改默认的加载策略
  	//可以通过 @OneToMany 的 cascade 属性来修改默认的删除策略. 
  	//注意: 若在 1 的一端的 @OneToMany 中使用 mappedBy(类似inverse,由多的一方来维护关联关系) 属性, 则 @OneToMany 端就不能再使用 @JoinColumn 属性了. 
  //	@JoinColumn(name="CUSTOMER_ID")
  	@OneToMany(fetch=FetchType.LAZY,cascade={CascadeType.REMOVE},mappedBy="customer")
  	public Set<Order> getOrders() {
  		return orders;
  	}
  
  	public void setOrders(Set<Order> orders) {
  		this.orders = orders;
  	}
  
  	//工具方法. 不需要映射为数据表的一列. 
  	@Transient
  	public String getInfo(){
  		return "lastName: " + lastName + ", email: " + email;
  	}
  
  	@Override
  	public String toString() {
  		return "Customer [id=" + id + ", lastName=" + lastName + ", email="
  				+ email + ", age=" + age + ", createdTime=" + createdTime
  				+ ", birth=" + birth + "]";
  	}
  }
  ```

* 双向一对多(多对一)

  单向一对多和单向多对一的组合(注意使用mappedBy提高性能，防止执行多余的update语句)。

* 双向一对一

  ```java
  @Table(name="JPA_DEPARTMENTS")
  @Entity
  public class Department {
  
  	private Integer id;
  	private String deptName;
  	
  	private Manager mgr;
  
  	@GeneratedValue
  	@Id
  	public Integer getId() {
  		return id;
  	}
  
  	public void setId(Integer id) {
  		this.id = id;
  	}
  
  	@Column(name="DEPT_NAME")
  	public String getDeptName() {
  		return deptName;
  	}
  
  	public void setDeptName(String deptName) {
  		this.deptName = deptName;
  	}
  
  	//使用 @OneToOne 来映射 1-1 关联关系。
  	//若需要在当前数据表中添加外键则需要使用 @JoinColumn 来进行映射. 注意, 1-1 关联关系, 所以需要添加 unique=true
  	@JoinColumn(name="MGR_ID", unique=true)
  	@OneToOne(fetch=FetchType.LAZY)
  	public Manager getMgr() {
  		return mgr;
  	}
  
  	public void setMgr(Manager mgr) {
  		this.mgr = mgr;
  	}
  }
  ====================================================
  @Table(name="JPA_MANAGERS")
  @Entity
  public class Manager {
  
  	private Integer id;
  	private String mgrName;
  	
  	private Department dept;
  
  	@GeneratedValue
  	@Id
  	public Integer getId() {
  		return id;
  	}
  
  	public void setId(Integer id) {
  		this.id = id;
  	}
  
  	@Column(name="MGR_NAME")
  	public String getMgrName() {
  		return mgrName;
  	}
  
  	public void setMgrName(String mgrName) {
  		this.mgrName = mgrName;
  	}
  
  	//对于不维护关联关系, 没有外键的一方, 使用 @OneToOne 来进行映射, 建议设置 mappedBy=true
  	@OneToOne(mappedBy="mgr")
  	public Department getDept() {
  		return dept;
  	}
  
  	public void setDept(Department dept) {
  		this.dept = dept;
  	}
  }
  ```

* 双向多对多

  ```java
  @Table(name="JPA_CATEGORIES")
  @Entity
  public class Category {
  
  	private Integer id;
  	private String categoryName;
  	
  	private Set<Item> items = new HashSet<>();
  
  	@GeneratedValue
  	@Id
  	public Integer getId() {
  		return id;
  	}
  
  	public void setId(Integer id) {
  		this.id = id;
  	}
  
  	@Column(name="CATEGORY_NAME")
  	public String getCategoryName() {
  		return categoryName;
  	}
  
  	public void setCategoryName(String categoryName) {
  		this.categoryName = categoryName;
  	}
  
  	@ManyToMany(mappedBy="categories")
  	public Set<Item> getItems() {
  		return items;
  	}
  
  	public void setItems(Set<Item> items) {
  		this.items = items;
  	}
  }
  ================================================
  @Table(name="JPA_ITEMS")
  @Entity
  public class Item {
  
  	private Integer id;
  	private String itemName;
  	
  	private Set<Category> categories = new HashSet<>();
  
  	@GeneratedValue
  	@Id
  	public Integer getId() {
  		return id;
  	}
  
  	public void setId(Integer id) {
  		this.id = id;
  	}
  
  	@Column(name="ITEM_NAME")
  	public String getItemName() {
  		return itemName;
  	}
  
  	public void setItemName(String itemName) {
  		this.itemName = itemName;
  	}
  
  	//使用 @ManyToMany 注解来映射多对多关联关系
  	//使用 @JoinTable 来映射中间表
  	//1. name 指向中间表的名字
  	//2. joinColumns 映射当前类所在的表在中间表中的外键
  	//2.1 name 指定外键列的列名
  	//2.2 referencedColumnName 指定外键列关联当前表的哪一列
  	//3. inverseJoinColumns 映射关联的类所在中间表的外键
  	@JoinTable(name="ITEM_CATEGORY",
  			joinColumns={@JoinColumn(name="ITEM_ID", referencedColumnName="ID")},
  			inverseJoinColumns={@JoinColumn(name="CATEGORY_ID", referencedColumnName="ID")})
  	@ManyToMany
  	public Set<Category> getCategories() {
  		return categories;
  	}
  
  	public void setCategories(Set<Category> categories) {
  		this.categories = categories;
  	}
  }
  ```
