# 开发

## 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--对应的接口mapper-->
<mapper namespace="mapper接口的位置">
<!--对应的结果类-->
  <resultMap id="ResultMap" type="查询结果对应的类">
      <result column="sql字段" jdbcType="VARCHAR" property="类字段" />
  </resultMap>
    
    <select id="查询接口的方法名" resultMap="ResultMap">
        <include refid="sql片段引用"/>
        <if test="sortClause != null">
            order by ${sortClause}
        </if>
    </select>
    
    <sql id="cityData">
        select 
          xxx,yyy
        from ai_sco_day_city_analyse
        <where>
            <!--会在前边去掉多余的and，后边去掉多余的逗号suffixOverrides=","-->
            <trim prefixOverrides="and" >
                <if test="startDate != null">
                    <!--大于小于等需要转义-->
                    and call_date &gt;= #{startDate}
                </if>
                    and xxx like concat('%',#{node},'%')
            </trim>
        </where>
        group by city
    </sql>

    <!--foreach用于循环列表，将列表用逗号分隔前后加括号拼接,collection中的要么是参数名，要么用@Param注解-->
    <foreach collection="whereTenantIds" open="(" close=")" separator="," item="item">
        (#{item.taskId})
         <!--#{listItem} 预编译阶段会生成?占位符，但是如果${groupByClause}，就会做纯替换-->
    </foreach>
  
  <update id="updateAuthorIfNecessary">
      update Author
        <set><!--动态更新-->
          <if test="username != null">username=#{username},</if>
          <if test="password != null">password=#{password},</if>
          <if test="email != null">email=#{email},</if>
          <if test="bio != null">bio=#{bio}</if>
        </set>
      where id=#{id}
	</update>
  
  <select id="findActiveBlogLike" resultType="Blog">
  				SELECT * FROM BLOG WHERE state = ‘ACTIVE’
          <choose>
                <when test="title != null">
                  AND title like #{title}
                </when>
                <when test="author != null and author.name != null">
                  AND author_name like #{author.name}
                </when>
                <otherwise>
                  AND featured = 1
                </otherwise>
         </choose>
		</select>
</mapper>
```

## 传递多个参数

- **@Param注解传参法**

  ```xml
  public User selectUser(@Param("userName") String name, int @Param("deptId") deptId);
  
  <select id="selectUser" resultMap="UserResultMap">
      select * from user
      where user_name = #{userName} and dept_id = #{deptId}
  </select>
  ```

- **Map传参法**

  ```xml
  public User selectUser(Map<String, Object> params);
  
  <select id="selectUser" parameterType="java.util.Map" resultMap="UserResultMap">
      select * from user
      where user_name = #{userName} and dept_id = #{deptId}
  </select>
  ```

- **Java Bean传参法**

  ```xml
  public User selectUser(User user);
  
  <select id="selectUser" parameterType="com.jourwon.pojo.User" resultMap="UserResultMap">
      select * from user
      where user_name = #{userName} and dept_id = #{deptId}
  </select>
  ```

## 属性字段映射

- 用别名映射

  ```sql
  <select id="getOrder" parameterType="int" resultType="com.jourwon.pojo.Order">
         select order_id id, order_no orderno ,order_price price form orders where order_id=#{id};
  </select>
  ```

- 通过resultMap  中的<result>来映射字段名和实体类属性名的一一对应的关系

## 一对一、一对多查询

- 一对一

  ```xml
  public class Order {
      private Integer orderId;
      /**
       * 另一个对象
       */
      private Pay pay;
  }
  
  <resultMap id="resultMap" type="Order">
      <id property="orderId" column="order_id" />
      <!--一对一结果映射-->
      <association property="pay" javaType="cn.fighter3.entity.Pay">
          <id column="payId" property="pay_id"/>
          <result column="account" property="account"/>
      </association>
  </resultMap>
  
  <select id="getTeacher" resultMap="resultMap" parameterType="int">
          select * from order o 
           left join pay p on o.order_id=p.order_id
          where  o.order_id=#{orderId}
  </select>
  ```

- 一对多

  ```xml
  public class Category {
      private int categoryId;
      private String categoryName;
    
      /**
      * 商品列表
      **/
      List<Product> products;
  }
        
  <resultMap type="Category" id="categoryBean">
          <id column="categoryId" property="category_id" />
          <result column="categoryName" property="category_name" />
  
          <!-- 一对多的关系 -->
          <!-- property: 指的是集合属性的值, ofType：指的是集合中元素的类型 -->
          <collection property="products" ofType="Product">
              <id column="product_id" property="productId" />
              <result column="productName" property="productName" />
              <result column="price" property="price" />
          </collection>
  </resultMap>
        
  <select id="listCategory" resultMap="categoryBean">
         select c.*, p.* from category_ c left join product_ p on c.id = p.cid
   </select>       
  ```

## 插入以后获取主键

```xml
<insert id="insert" useGeneratedKeys="true" keyProperty="userId" >
    insert into user( 
    user_name, user_password, create_time) 
    values(#{userName}, #{userPassword} , #{createTime, jdbcType= TIMESTAMP})
</insert>

<!--注意，插入以后返回的仍然是插入成功几条，但是设置的对象的id属性值会被改写-->
mapper.insert(user);
user.getId;
```

## 批量保存

- 使用foreach标签
- 使用ExecutorType.BATCH

## mybatis-generator

maven插件

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>${mybatis-generator.version}</version>
                <configuration>
                    <verbose>true</verbose>
                    <overwrite>true</overwrite>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>org.mybatis.generator</groupId>
                        <artifactId>mybatis-generator-core</artifactId>
                        <version>${mybatis-generator.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>${mysql-driver.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>com.softwareloop</groupId>
                        <artifactId>mybatis-generator-lombok-plugin</artifactId>
                        <version>1.0</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
```

配置文件，放在resourcs文件夹下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<!-- 配置生成器 -->
<generatorConfiguration>

    <context id="mysql" targetRuntime="MyBatis3">

        <!--true自动给表名加反引号关键字-->
        <property name="autoDelimitKeywords" value="false"/>
        <property name="javaFileEncoding" value="UTF-8"/>
        <property name="javaFormatter" value="org.mybatis.generator.api.dom.DefaultJavaFormatter"/>
        <property name="xmlFormatter" value="org.mybatis.generator.api.dom.DefaultXmlFormatter"/>
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>

        <!--实现序列化接口-->
        <!--<plugin type="org.mybatis.generator.plugins.SerializablePlugin">
            <property name="suppressJavaInterface" value="false"/>
        </plugin>-->
        <!--重写equal和hashcode方法-->
        <!--<plugin type="org.mybatis.generator.plugins.EqualsHashCodePlugin"/>-->
        <plugin type="com.softwareloop.mybatis.generator.plugins.LombokPlugin" >
            <property name="hasLombok" value="true"/>
            <!--<property name="accessors" value="true"/>-->
            <property name="builder" value="true"/>
            <property name="builder.fluent" value="true"/>
        </plugin>

        <commentGenerator>
            <!--是否取消注释-->
            <property name="suppressAllComments" value="true"/>
            <!--是否生成注释带时间戳-->
            <property name="suppressDate" value="true"/>
        </commentGenerator>

        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false"
                        userId="root"
                        password="20131983"/>

        <javaTypeResolver type="org.mybatis.generator.internal.types.JavaTypeResolverDefaultImpl">
            <!--是否强制将DECIMAL和NUMERIC类型的JDBC字段转换为Java类型的BigDecimal-->
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <javaModelGenerator targetPackage="com.wyj.entity"
                            targetProject="/Users/wyj/Desktop/java"/>

        <sqlMapGenerator targetPackage="mappers"
                         targetProject="/Users/wyj/Desktop/java/xml"/>

        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.wyj.mapper"
                             implementationPackage="com.wyj.mapper.impl"
                             targetProject="/Users/wyj/Desktop/java"/>

        <table tableName="testgen_%">
            <generatedKey column="id" sqlStatement="mysql" identity="true"/>
            <!--将所有以test开头的对象都替换成空字符串-->
            <domainObjectRenamingRule searchString="^Test" replaceString=""/>
            <!--将is_visible都替换成visible-->
            <columnRenamingRule searchString="is_visible" replaceString="visible"/>
            <columnOverride column="add_time" javaType="LocalDateTime" isGeneratedAlways="true"/>
            <columnOverride column="update_time" javaType="LocalDateTime" isGeneratedAlways="true"/>
            <!--枚举转换-->
            <columnOverride column="user_type" jdbcType="VARCHAR" javaType="com.enums.UserType"/>
        </table>
    </context>

</generatorConfiguration>
```

# 问题

## #与$符的区别

```
（1）
　　1）#{} 为参数占位符 ?，即sql 预编译
　　2）${} 为字符串替换，即 sql 拼接
（2）
　　1）#{}：动态解析 -> 预编译 -> 执行
　　2）${}：动态解析 -> 编译 -> 执行
（3）
　　1）#{} 的变量替换是在DBMS 中
　　2）${} 的变量替换是在 DBMS 外
（4）
　　1）变量替换后，#{} 对应的变量自动加上单引号 ''
　　2）变量替换后，${} 对应的变量不会加上单引号 ''
（5）
　　1）#{} 能防止sql 注入
　　2）${} 不能防止sql 注入

#{} 和 ${} 的实例：假设传入参数为 1
（1）开始
　　1）#{}：select * from t_user where uid= #{uid}
　　2）${}：select * from t_user where uid= '${uid}'
（2）然后
	 1）#{}：select * from t_user where uid= ?
	 2）${}：select * from t_user where uid= '1'
（3）最后
　　1）#{}：select * from t_user where uid= '1'
　　2）${}：select * from t_user where uid= '1'

#{} 和 ${} 在使用中的技巧和建议
（1）不论是单个参数，还是多个参数，一律都建议使用注解@Param("")
（2）能用 #{} 的地方就用 #{}，不用或少用 ${}
（3）表名作参数时，必须用 ${}。如：select * from ${tableName}
（4）order by 时，必须用 ${}。如：select * from t_user order by ${columnName}
		 3,4因为预编译加了单引号，会导致不起作用
（5）使用 ${} 时，要注意何时加或不加单引号，即 ${} 和 '${}'
```

## 生命周期

- SqlSessionFactoryBuilder

  >一旦创建了 SqlSessionFactory，就不再需要它了。因此 SqlSessionFactoryBuilder 实例的生命周期只存在于方法的内部。

- SqlSessionFactory

  >SqlSessionFactory 是用来创建SqlSession的，相当于一个数据库连接池，每次创建SqlSessionFactory都会使用数据库资源，多次创建和销毁是对资源的浪费。所以SqlSessionFactory是应用级的生命周期，而且应该是单例的。

- SqlSession

  > SqlSession相当于JDBC中的Connection，SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的生命周期是一次请求或一个方法。

- Mapper

  > 映射器是一些绑定映射语句的接口。映射器接口的实例是从 SqlSession 中获得的，它的生命周期在sqlsession事务方法之内，一般会控制在方法级。

## 延迟加载及其原理

>- Mybatis支持association关联对象和collection关联集合对象的延迟加载，association指的就是一对一，collection指的就是一对多查询。在Mybatis配置文件中，可以配置是否启用延迟加载lazyLoadingEnabled=true|false。
>- 它的原理是，使用CGLIB创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null值，那么就会单独发送事先保存好的查询关联B对象的sql，把B查询上来，然后调用a.setB(b)，于是a的对象b属性就有值了，接着完成a.getB().getName()方法的调用。这就是延迟加载的基本原理。

## MyBatis执行流程

1. 读取 MyBatis 配置文件——mybatis-config.xml 、加载映射文件——映射文件即 SQL 映射文件，文件中配置了操作数据库的 SQL 语句，最后生成一个配置对象。
2. 构造会话工厂：通过 MyBatis 的环境等配置信息构建会话工厂 SqlSessionFactory。
3. 创建会话对象：由会话工厂创建 SqlSession 对象，该对象中包含了执行 SQL 语句的所有方法。
4. Executor 执行器：MyBatis 底层定义了一个 Executor 接口来操作数据库，它将根据 SqlSession 传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护。
5. StatementHandler：数据库会话器，串联起参数映射的处理和运行结果映射的处理。
6. ParameterHandler：对输入参数的类型进行处理，并预编译。
7. ResultSetHandler：对返回结果的类型进行处理，根据对象映射规则，返回相应的对象。

## 功能架构

一般把Mybatis的功能架构分为三层：

- API接口层：提供给外部使用的接口API，开发人员通过这些本地API来操纵数据库。接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理。
- 数据处理层：负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作。
- 基础支撑层：负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑。

## 为什么Mapper接口不需要实现类

- 动态代理

## 都有哪些Executor执行器

Mybatis有三种基本的Executor执行器，SimpleExecutor、ReuseExecutor、BatchExecutor。

- **SimpleExecutor**：每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。
- **ReuseExecutor**：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map<String, Statement>内，供下一次使用。简言之，就是重复使用Statement对象。
- **BatchExecutor**：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同。

## 如何指定使用哪一种Executor执行器

- 在Mybatis配置文件中，在设置（settings）可以指定默认的ExecutorType执行器类型，也可以手动给DefaultSqlSessionFactory的创建SqlSession的方法传递ExecutorType类型参数，如SqlSession openSession(ExecutorType execType)。
- 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）；BATCH 执行器将重用语句并执行批量更新。

## 插件原理

- Mybatis会话的运行需要ParameterHandler、ResultSetHandler、StatementHandler、Executor这四大对象的配合，插件的原理就是在这四大对象调度的时候，插入一些我我们自己的代码
- Mybatis使用JDK的动态代理，为目标对象生成代理对象。它提供了一个工具类`Plugin`，实现了`InvocationHandler`接口
- 使用`Plugin`生成代理对象，代理对象在调用方法的时候，就会进入invoke方法，在invoke方法中，如果存在签名的拦截方法，插件的intercept方法就会在这里被我们调用，然后就返回结果。如果不存在签名方法，那么将直接反射调用我们要执行的方法

## 如何编写插件

- 实现Mybatis的Interceptor接口并重写intercept()方法
- 然后再给插件编写注解，确定要拦截的对象，要拦截的方法
- 最后，再MyBatis配置文件里面配置插件

## 分页

- MyBatis使用RowBounds对象进行分页，它是针对ResultSet结果集执行的内存分页，而非物理分页。可以在sql内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

- 分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，拦截Executor的query方法

- 在执行查询的时候，拦截待执行的sql，然后重写sql，根据dialect方言，添加对应的物理分页语句和物理分页参数。

## 缓存

- 一级缓存: 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为SqlSession，各个SqlSession之间的缓存相互隔离，当 Session flush 或 close 之后，该 SqlSession 中的所有 Cache 就将清空，MyBatis默认打开一级缓存。
- 二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同之处在于其存储作用域为 Mapper(Namespace)，可以在多个SqlSession之间共享，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态),可在它的映射文件中配置。
