---
title: hibernate映射关系
date: 2018-12-11 20:00:23
tags: hibernate
---

![实体映射关系](https://raw.githubusercontent.com/haominglfs/images/master/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.png)

<!--more-->

### java代码

```java
public class Customer {

	private Integer customerId;
	private String customerName;

	public Integer getCustomerId() {
		return customerId;
	}

	public void setCustomerId(Integer customerId) {
		this.customerId = customerId;
	}

	public String getCustomerName() {
		return customerName;
	}

	public void setCustomerName(String customerName) {
		this.customerName = customerName;
	}

}

public class Order {
	
	private Integer orderId;
	private String orderName;
	
	private Customer customer;

	public Integer getOrderId() {
		return orderId;
	}

	public void setOrderId(Integer orderId) {
		this.orderId = orderId;
	}

	public String getOrderName() {
		return orderName;
	}

	public void setOrderName(String orderName) {
		this.orderName = orderName;
	}

	public Customer getCustomer() {
		return customer;
	}

	public void setCustomer(Customer customer) {
		this.customer = customer;
	}
}
```

### 单向一对多

```xml
<hibernate-mapping package="com.atguigu.hibernate.entities.n21">
    <class name="Order" table="ORDERS">
        <id name="orderId" type="java.lang.Integer">
            <column name="ORDER_ID" />
            <generator class="native" />
        </id>
        
        <property name="orderName" type="java.lang.String">
            <column name="ORDER_NAME" />
        </property>
        
		<!-- 
			映射多对一的关联关系。 使用 many-to-one 来映射多对一的关联关系 
			name: 多这一端关联的一那一端的属性的名字
			class: 一那一端的属性对应的类名
			column: 一那一端在多的一端对应的数据表中的外键的名字
		-->
		<many-to-one name="customer" class="Customer" column="CUSTOMER_ID"></many-to-one>
    </class>
</hibernate-mapping>
```

```java
	@Test
	public void testMany2OneSave(){
		Customer customer = new Customer();
		customer.setCustomerName("BB");
		
		Order order1 = new Order();
		order1.setOrderName("ORDER-3");
		
		Order order2 = new Order();
		order2.setOrderName("ORDER-4");
		
		//设定关联关系
		order1.setCustomer(customer);
		order2.setCustomer(customer);
		
		//执行  save 操作: 先插入 Customer, 再插入 Order, 3 条 INSERT
		//先插入 1 的一端, 再插入 n 的一端, 只有 INSERT 语句.
	    //session.save(customer);
        
		//session.save(order1);
		//session.save(order2);
		
		//先插入 Order, 再插入 Customer. 3 条 INSERT, 2 条 UPDATE
		//先插入 n 的一端, 再插入 1 的一端, 会多出 UPDATE 语句!
		//因为在插入多的一端时, 无法确定 1 的一端的外键值. 所以只能等 1 的一端插入后, 再额外发送 				UPDATE 语句.
		//推荐先插入 1 的一端, 后插入 n 的一端
		session.save(order1);
		session.save(order2);
		session.save(customer);
	}	

	@Test
	public void testMany2OneGet(){
		//1. 若查询多的一端的一个对象, 则默认情况下, 只查询了多的一端的对象. 而没有查询关联的
		//1 的那一端的对象!
		Order order = (Order) session.get(Order.class, 1);
		System.out.println(order.getOrderName()); 
		
		System.out.println(order.getCustomer().getClass().getName());
		
		session.close();
		
		//2. 在需要使用到关联的对象时, 才发送对应的 SQL 语句. 
		Customer customer = order.getCustomer();
		System.out.println(customer.getCustomerName()); 
		
		//3. 在查询 Customer 对象时, 由多的一端导航到 1 的一端时, 
		//若此时 session 已被关闭, 则默认情况下
		//会发生 LazyInitializationException 异常
		
		//4. 获取 Order 对象时, 默认情况下, 其关联的 Customer 对象是一个代理对象!
	}

	@Test
	public void testDelete(){
		//在不设定级联关系的情况下, 且 1 这一端的对象有 n 的对象在引用, 不能直接删除 1 这一端的对象
		Customer customer = (Customer) session.get(Customer.class, 1);
		session.delete(customer); 
	}
	
	@Test
	public void testUpdate(){
		Order order = (Order) session.get(Order.class, 1);
		order.getCustomer().setCustomerName("AAA");
	}
```

### 双向一对多

* java代码

```java
public class Customer {

	private Integer customerId;
	private String customerName;
	
	/*
	 * 1. 声明集合类型时, 需使用接口类型, 因为 hibernate 在获取
	 * 集合类型时, 返回的是 Hibernate 内置的集合类型, 而不是 JavaSE 一个标准的
	 * 集合实现. 
	 * 2. 需要把集合进行初始化, 可以防止发生空指针异常
	 */
	private Set<Order> orders = new HashSet<>();

	public Integer getCustomerId() {
		return customerId;
	}

	public void setCustomerId(Integer customerId) {
		this.customerId = customerId;
	}

	public String getCustomerName() {
		return customerName;
	}

	public void setCustomerName(String customerName) {
		this.customerName = customerName;
	}

	public Set<Order> getOrders() {
		return orders;
	}

	public void setOrders(Set<Order> orders) {
		this.orders = orders;
	}	
}

public class Order {
	
	private Integer orderId;
	private String orderName;
	
	private Customer customer;

	public Integer getOrderId() {
		return orderId;
	}

	public void setOrderId(Integer orderId) {
		this.orderId = orderId;
	}

	public String getOrderName() {
		return orderName;
	}

	public void setOrderName(String orderName) {
		this.orderName = orderName;
	}

	public Customer getCustomer() {
		return customer;
	}

	public void setCustomer(Customer customer) {
		this.customer = customer;
	}	
}
```

* 映射配置(多的一端配置与单向相同)

  ```xml
  <hibernate-mapping package="com.atguigu.hibernate.entities.n21.both">
      
      <class name="Customer" table="CUSTOMERS">
      
          <id name="customerId" type="java.lang.Integer">
              <column name="CUSTOMER_ID" />
              <generator class="native" />
          </id>
      
          <property name="customerName" type="java.lang.String">
              <column name="CUSTOMER_NAME" />
          </property>
          
          <!-- 映射 1 对多的那个集合属性 -->
          <!-- set: 映射 set 类型的属性, table: set 中的元素对应的记录放在哪一个数据表中. 该值需要和多对一的多的那个表的名字一致 -->
          <!-- inverse: 指定由哪一方来维护关联关系. 通常设置为 true, 以指定由多的一端来维护关联关系 -->
          <!-- cascade 设定级联操作. 开发时不建议设定该属性. 建议使用手工的方式来处理 -->
          <!-- order-by 在查询时对集合中的元素进行排序, order-by 中使用的是表的字段名, 而不是持久化类的属性名  -->
          <set name="orders" table="ORDERS" inverse="true" order-by="ORDER_NAME DESC">
          	<!-- 执行多的表中的外键列的名字 -->
          	<key column="CUSTOMER_ID"></key>
          	<!-- 指定映射类型 -->
          	<one-to-many class="Order"/>
          </set>
          
      </class>
      
  </hibernate-mapping>
  ```


* 测试

  ```java
  	@Test
  	public void testCascade(){
  		Customer customer = (Customer) session.get(Customer.class, 3);
  		customer.getOrders().clear();
  	}
  	
  	@Test
  	public void testDelete(){
  		//在不设定级联关系的情况下, 且 1 这一端的对象有 n 的对象在引用, 不能直接删除 1 这一端的对象
  		Customer customer = (Customer) session.get(Customer.class, 1);
  		session.delete(customer); 
  	}
  	
  	@Test
  	public void testUpdat2(){
  		Customer customer = (Customer) session.get(Customer.class, 1);
  		customer.getOrders().iterator().next().setOrderName("GGG"); 
  	}
  	
  	@Test
  	public void testUpdate(){
  		Order order = (Order) session.get(Order.class, 1);
  		order.getCustomer().setCustomerName("AAA");
  	}
  	
  	@Test
  	public void testOne2ManyGet(){
  		//1. 对 n 的一端的集合使用延迟加载
  		Customer customer = (Customer) session.get(Customer.class, 7);
  		System.out.println(customer.getCustomerName()); 
  		//2. 返回的多的一端的集合时 Hibernate 内置的集合类型. 
  		//该类型具有延迟加载和存放代理对象的功能. 
  		System.out.println(customer.getOrders().getClass()); 
  		
  		//session.close();
  		//3. 可能会抛出 LazyInitializationException 异常 
  		
  		System.out.println(customer.getOrders().size()); 
  		
  		//4. 再需要使用集合中元素的时候进行初始化. 
  	}
  	
  	@Test
  	public void testMany2OneGet(){
  		//1. 若查询多的一端的一个对象, 则默认情况下, 只查询了多的一端的对象. 而没有查询关联的
  		//1 的那一端的对象!
  		Order order = (Order) session.get(Order.class, 1);
  		System.out.println(order.getOrderName()); 
  		
  		System.out.println(order.getCustomer().getClass().getName());
  		
  		session.close();
  		
  		//2. 在需要使用到关联的对象时, 才发送对应的 SQL 语句. 
  		Customer customer = order.getCustomer();
  		System.out.println(customer.getCustomerName()); 
  		
  		//3. 在查询 Customer 对象时, 由多的一端导航到 1 的一端时, 
  		//若此时 session 已被关闭, 则默认情况下
  		//会发生 LazyInitializationException 异常
  		
  		//4. 获取 Order 对象时, 默认情况下, 其关联的 Customer 对象是一个代理对象!
  		
  	}
  	
  	@Test
  	public void testMany2OneSave(){
  		Customer customer = new Customer();
  		customer.setCustomerName("AA");
  		
  		Order order1 = new Order();
  		order1.setOrderName("ORDER-1");
  		
  		Order order2 = new Order();
  		order2.setOrderName("ORDER-2");
  		
  		//设定关联关系
  		order1.setCustomer(customer);
  		order2.setCustomer(customer);
  		
  		customer.getOrders().add(order1);
  		customer.getOrders().add(order2);
  		
  		//执行  save 操作: 先插入 Customer, 再插入 Order, 3 条 INSERT, 2 条 UPDATE
  		//因为 1 的一端和 n 的一端都维护关联关系. 所以会多出 UPDATE
  		//可以在 1 的一端的 set 节点指定 inverse=true, 来使 1 的一端放弃维护关联关系!
  		//建议设定 set 的 inverse=true, 建议先插入 1 的一端, 后插入多的一端
  		//好处是不会多出 UPDATE 语句
  		session.save(customer);
  		
  		//session.save(order1);
  		//session.save(order2);
  		
  		//先插入 Order, 再插入 Cusomer, 3 条 INSERT, 4 条 UPDATE
  		//session.save(order1);
  		//session.save(order2);	
  		//session.save(customer);
  	}
  ```

### 一对一(外键)

* 实体类

  ```java
  public class Department {
  
  	private Integer deptId;
  	private String deptName;
  	
  	private Manager mgr;
  
  	public Integer getDeptId() {
  		return deptId;
  	}
  
  	public void setDeptId(Integer deptId) {
  		this.deptId = deptId;
  	}
  
  	public String getDeptName() {
  		return deptName;
  	}
  
  	public void setDeptName(String deptName) {
  		this.deptName = deptName;
  	}
  
  	public Manager getMgr() {
  		return mgr;
  	}
  
  	public void setMgr(Manager mgr) {
  		this.mgr = mgr;
  	}
  }
  
  
  public class Manager {
  
  	private Integer mgrId;
  	private String mgrName;
  	
  	private Department dept;
  
  	public Integer getMgrId() {
  		return mgrId;
  	}
  
  	public void setMgrId(Integer mgrId) {
  		this.mgrId = mgrId;
  	}
  
  	public String getMgrName() {
  		return mgrName;
  	}
  
  	public void setMgrName(String mgrName) {
  		this.mgrName = mgrName;
  	}
  
  	public Department getDept() {
  		return dept;
  	}
  
  	public void setDept(Department dept) {
  		this.dept = dept;
  	}
  }
  ```

* 映射

  ```xml
  <hibernate-mapping>
  
      <class name="com.atguigu.hibernate.one2one.foreign.Manager" table="MANAGERS">
          
          <id name="mgrId" type="java.lang.Integer">
              <column name="MGR_ID" />
              <generator class="native" />
          </id>
          
          <property name="mgrName" type="java.lang.String">
              <column name="MGR_NAME" />
          </property>
          
          <!-- 映射 1-1 的关联关系: 在对应的数据表中已经有外键了, 当前持久化类使用 one-to-one 进行映射 -->
          <!-- 
          	没有外键的一端需要使用one-to-one元素，该元素使用 property-ref 属性指定使用被关联实体主键以外的字段作为关联字段(没有property-ref将使用dept主键做关联，否则使用mgr属性对应表字段做关联)
           -->
          <one-to-one name="dept" 
          	class="com.atguigu.hibernate.one2one.foreign.Department"
          	property-ref="mgr"></one-to-one>
          
      </class>
      
  </hibernate-mapping>
  ====================================================
  <hibernate-mapping>
  
      <class name="com.atguigu.hibernate.one2one.foreign.Department" table="DEPARTMENTS">
  
          <id name="deptId" type="java.lang.Integer">
              <column name="DEPT_ID" />
              <generator class="native" />
          </id>
          
          <property name="deptName" type="java.lang.String">
              <column name="DEPT_NAME" />
          </property>
  		
  		<!-- 使用 many-to-one 的方式来映射 1-1 关联关系 -->
  		<many-to-one name="mgr" class="com.atguigu.hibernate.one2one.foreign.Manager" 
  			column="MGR_ID" unique="true"></many-to-one>	        
  			        
      </class>
  </hibernate-mapping>
  ```

* 测试

  ```java
  	@Test
  	public void testGet2(){
  		//在查询没有外键的实体对象时, 使用的左外连接查询, 一并查询出其关联的对象
  		//并已经进行初始化. 
  		Manager mgr = (Manager) session.get(Manager.class, 1);
  		System.out.println(mgr.getMgrName()); 
  		System.out.println(mgr.getDept().getDeptName()); 
  	}
  	
  	@Test
  	public void testGet(){
  		//1. 默认情况下对关联属性使用懒加载
  		Department dept = (Department) session.get(Department.class, 1);
  		System.out.println(dept.getDeptName()); 
  		
  		//2. 所以会出现懒加载异常的问题. 
  		//session.close();
  		//Manager mgr = dept.getMgr();
  		//System.out.println(mgr.getClass()); 
  		//System.out.println(mgr.getMgrName()); 
  		
  		//3. 查询 Manager 对象的连接条件应该是 dept.manager_id = mgr.manager_id
  		//而不应该是 dept.dept_id = mgr.manager_id
  		Manager mgr = dept.getMgr();
  		System.out.println(mgr.getMgrName()); 
  	}
  	
  	@Test
  	public void testSave(){
  		
  		Department department = new Department();
  		department.setDeptName("DEPT-BB");
  		
  		Manager manager = new Manager();
  		manager.setMgrName("MGR-BB");
  		
  		//设定关联关系
  		department.setMgr(manager);
  		manager.setDept(department);
  		
  		//保存操作
  		//建议先保存没有外键列的那个对象. 这样会减少 UPDATE 语句
  		session.save(department);
  		session.save(manager);
  		
  	}
  }
  ```

### 一对一(基于主键)

* 实体类同上

* 映射

  ```xml
  <hibernate-mapping package="com.atguigu.hibernate.one2one.primary">
  
      <class name="Department" table="DEPARTMENTS">
  
          <id name="deptId" type="java.lang.Integer">
              <column name="DEPT_ID" />
              <!-- 使用外键的方式来生成当前的主键 -->
              <generator class="foreign">
              	<!-- property 属性指定使用当前持久化类的哪一个属性的主键作为外键 -->
              	<param name="property">mgr</param>
              </generator>
          </id>
          
          <property name="deptName" type="java.lang.String">
              <column name="DEPT_NAME" />
          </property>
  		
  		<!--  
  		采用 foreign 主键生成器策略的一端增加 one-to-one 元素映射关联属性,
  		其 one-to-one 节点还应增加 constrained=true 属性, 以使当前的主键上添加外键约束
  		-->
  		<one-to-one name="mgr" class="Manager" constrained="true"></one-to-one>
  					        
      </class>
  </hibernate-mapping>
  =================================================
  <hibernate-mapping>
  
      <class name="com.atguigu.hibernate.one2one.primary.Manager" table="MANAGERS">
          
          <id name="mgrId" type="java.lang.Integer">
              <column name="MGR_ID" />
              <generator class="native" />
          </id>
          
          <property name="mgrName" type="java.lang.String">
              <column name="MGR_NAME" />
          </property>
          
          <one-to-one name="dept" 
          	class="com.atguigu.hibernate.one2one.primary.Department"></one-to-one>
          
      </class>
  ```

* 测试

  ```java
  	@Test
  	public void testGet2(){
  		//在查询没有外键的实体对象时, 使用的左外连接查询, 一并查询出其关联的对象
  		//并已经进行初始化. 
  		Manager mgr = (Manager) session.get(Manager.class, 1);
  		System.out.println(mgr.getMgrName()); 
  		System.out.println(mgr.getDept().getDeptName()); 
  	}
  	
  	@Test
  	public void testGet(){
  		//1. 默认情况下对关联属性使用懒加载
  		Department dept = (Department) session.get(Department.class, 1);
  		System.out.println(dept.getDeptName()); 
  		
  		//2. 所以会出现懒加载异常的问题. 
  		Manager mgr = dept.getMgr();
  		System.out.println(mgr.getMgrName()); 
  	}
  	
  	@Test
  	public void testSave(){
  		
  		Department department = new Department();
  		department.setDeptName("DEPT-DD");
  		
  		Manager manager = new Manager();
  		manager.setMgrName("MGR-DD");
  		
  		//设定关联关系
  		manager.setDept(department);
  		department.setMgr(manager);
  		
  		//保存操作
  		//先插入哪一个都不会有多余的 UPDATE
  		session.save(department);
  		session.save(manager);
  		
  	}
  ```

### 映射多对多

* 实体类

  ```java
  public class Category {
  
  	private Integer id;
  	private String name;
  	
  	private Set<Item> items = new HashSet<>();
  
  	public Integer getId() {
  		return id;
  	}
  
  	public void setId(Integer id) {
  		this.id = id;
  	}
  
  	public String getName() {
  		return name;
  	}
  
  	public void setName(String name) {
  		this.name = name;
  	}
  
  	public Set<Item> getItems() {
  		return items;
  	}
  
  	public void setItems(Set<Item> items) {
  		this.items = items;
  	}
  }
  
  public class Item {
  
  	private Integer id;
  	private String name;
  	
  	private Set<Category> categories = new HashSet<>();
  	
  	public Integer getId() {
  		return id;
  	}
  	public void setId(Integer id) {
  		this.id = id;
  	}
  	public String getName() {
  		return name;
  	}
  	public void setName(String name) {
  		this.name = name;
  	}
  	public Set<Category> getCategories() {
  		return categories;
  	}
  	public void setCategories(Set<Category> categories) {
  		this.categories = categories;
  	}
  }
  ```

* 映射(单向的配置为去掉一个的配置)

  ```xml
  <hibernate-mapping package="com.atguigu.hibernate.n2n">
  
      <class name="Category" table="CATEGORIES">
      
          <id name="id" type="java.lang.Integer">
              <column name="ID" />
              <generator class="native" />
          </id>
      
          <property name="name" type="java.lang.String">
              <column name="NAME" />
          </property>
          
          <!-- table: 指定中间表 -->
          <set name="items" table="CATEGORIES_ITEMS">
              <key>
                  <column name="C_ID" />
              </key>
              <!-- 使用 many-to-many 指定多对多的关联关系. column 执行 Set 集合中的持久化类在中间表的外键列的名称  -->
              <many-to-many class="Item" column="I_ID"></many-to-many>
          </set>
          
      </class>
  </hibernate-mapping>
  ======================================================
  <hibernate-mapping>
  
      <class name="com.atguigu.hibernate.n2n.Item" table="ITEMS">
          
          <id name="id" type="java.lang.Integer">
              <column name="ID" />
              <generator class="native" />
          </id>
          
          <property name="name" type="java.lang.String">
              <column name="NAME" />
          </property>
          
          <set name="categories" table="CATEGORIES_ITEMS" inverse="true">
          	<key column="I_ID"></key>
          	<many-to-many class="com.atguigu.hibernate.n2n.Category" column="C_ID"></many-to-many>
          </set>
          
      </class>
  </hibernate-mapping>
  ```

* 映射继承关系