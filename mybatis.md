# 마이바티스를 사용한 자바 퍼시스턴스 개발

## Spring 프로젝트에서 연동하기

먼저 의존성 추가
```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.3.0</version>
</dependency>

<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis-spring</artifactId>
	<version>1.2.3</version>
</dependency>
```

mybatis-spring 모듈을 사용하면, 스프링의 ApplicationContext에 마이바티스 빈을 설정할 수 있다.

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
	<property name="dataSource" ref="dataSource" />
    <property name="configLocation" value="classpath:/mybatis-config.xml"/>
    <property name="mapperLocations" value="classpath:/sql-map/*.xml"/>
    <!-- statement 선언의 오류를 좀 더 빠르게 파악하기 위해서 true로 설정 -->
    <property name="failFast" value="true"/>
</bean>

<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg index="0" ref="sqlSessionFactory"></constructor-arg>
    <constructor-arg index="1" value="BATCH" />
</bean>
```
스프링에서 SqlSessionFactory빈을 만들고 SqlSessionTemplate빈을 만들어서 생성자로 넣어준다.
SqlSessionTemplate빈은 thread safe하게 SqlSession 객체를 제공하기 때문에, 여러 개의 스프링 빈에서 동일한 SqlSessionTemplate 객체를
공유할 수 있다. 개념적으로는 SqlSessionTemplate이 스프링 DAO 모듈의 JdbcTemplate과 동일하다. 


mapper interface가 있는 패키지를 체크하고 자동으로 mapper interface를 mapper bean으로 등록하기 위해
MapperScannerConfigurer를 사용할 수 있다.
```xml
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	<property name="basePackage" value="com.test.mapper" />
	<property name="processPropertyPlaceHolders" value="true" />
</bean>
```

java config code 
```java
@Configuration
@MapperScan("com.test.mapper")
public class TestConfig {

	@Autowired
	ApplicationContext applicationContext;

	@Bean
	public SqlSessionFactory sqlSessionFactory() throws Exception {
		SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
		sqlSessionFactoryBean.setDataSource(dataSource());
		sqlSessionFactoryBean.setConfigLocation(new ClassPathResource("mybatis-config.xml"));
		sqlSessionFactoryBean.setMapperLocations(applicationContext.getResources("classpath:/sql-map/*.xml"));
		return sqlSessionFactoryBean.getObject();
	}

	@Bean
	public SqlSessionTemplate sqlSessionTemplate() throws Exception {
		SqlSessionTemplate sqlSessionTemplate = new SqlSessionTemplate(sqlSessionFactory());
		return sqlSessionTemplate;
	}
}
```

MapperScanner를 사용하면 mapper namespace로 맵핑시켜서 사용하면 된다.
```java
@Repository(value = "TestMapper")
public interface TestMapper {
}
```
```xml
<mapper namespace="com.test.TestMapper">
```

만약 MapperScanner를 사용안한다면 sqlSessionTemplate을 직접 DI받아서 사용하면 된다.
```java
@Repository
public class TestDaoImpl implements TestDao {
		@Autowired
    	private SqlSessionTemplate sqlSessionTemplate;
}
```

스프링을 사용한 트랜잭션 관리
```xml
<tx:annotation-driven></tx:annotation-driven>

<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
```
트랜잭션매니저에서 사용한 dataSource는 sqlSessionFactory 빈이 사용하는 것과 동일한 dataSource여야 한다.

## XML을 사용한 SQL 매퍼
ResultMap 확장이 가능하다. 
```xml
<resultMap type="Student" id="StudentResult">
    <result property="name" column="name"/>
    <result property="email" column="email"/>
    ...
</resultMap>

<resultMap type="Student" id="StudentWithAddressResult" extends="StudentResult">
    <result property="address.addrId" column="addr_id"/>
    <result property="address.street" column="street"/>
    ...
</resultMap>
```

## DB 전환위한 dual write

두개의 트랜잭션매니저를 묶는 트랜잭션매니저가 필요하다.
```xml
<!-- Chained Transaction manager for a multi tx -->
<bean id="chainedTransactionManager" class="org.springframework.data.transaction.ChainedTransactionManager">
	<constructor-arg>
		<list>
			<ref bean="transactionManager" />
			<ref bean="secondaryTransactionManager" />
		</list>
	</constructor-arg>
</bean>

<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"/>
</bean>

<bean id="secondaryTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="secondaryDataSource"/>
</bean>
```

그리고 secondaryDataSource 빈을 등록한다.

```xml
<bean id="secondaryDataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
	<!-- 프로퍼티 설정 -->
</bean>
```

그리고 동적 db 판별을 위한 설정으로 databaseIdProvider 빈을 등록한다.
서로 호환이 안되는 쿼리가 있을 경우 사용한다.

```xml
<bean id="databaseIdProvider" class="org.apache.ibatis.mapping.VendorDatabaseIdProvider">
	<property name="properties">
		<props>
			<prop key="MSSQL">MSSQL</prop>
			<prop key="MySQL">MYSQL</prop>
		</props>
	</property>
</bean>
```

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 동적DB 판별을 위한 설정 -->
    <property name="databaseIdProvider" ref="databaseIdProvider"/>
    <property name="dataSource" ref="dataSource" />
</bean>

<bean id="secondarySqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
	<!-- 동적DB 판별을 위한 설정 -->
	<property name="databaseIdProvider" ref="databaseIdProvider"/>
	<property name="dataSource" ref="secondaryDataSource" />
</bean>

<bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg index="0" ref="sqlSessionFactory"></constructor-arg>
</bean>

<bean id="secondarySqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
	<constructor-arg index="0" ref="secondarySqlSessionFactory"></constructor-arg>
</bean>
```

설정이 끝나면 쿼리에서 다음과 같이 분기해서 사용하면 된다.
```xml
<select>
    <if test="_databaseId == 'MSSQL'">
        <!-- MSSQL 전용쿼리 -->
    </if>

    <if test="_databaseId == 'MYSQL'">
        <!-- MYSQL 전용쿼리 -->
    </if>
</select>
```

자바코드에서 사용할 때는 Dao 를 두개 만들어야한다. ex) TaskDao, TaskSecondaryDao
그래서 서비스레이어에서 각 dao를 호출해서 dual-write를 수행하고, TaskSecondaryDao 에서는 secondarySqlSessionTemplate 를 주입받아 사용한다.

```java
@Autowired
@Qualifier("secondarySqlSessionTemplate")
private SqlSessionTemplate secondarySqlSessionTemplate;
```

참고로 호환이 안되는 쿼리 분기는 master, secondary 관계없이 작업해야한다.
mssql -> mysql 로 바꿨을 때 master, secondary 가 서로 바뀌는데 이때 영향을 받지 않는다.


## 동적 SQL
마이바티는 `<if>`, `<choose>`, `<where>`, `<foreach>`, `<trim>`과 같은 엘리먼트를 사용해서 동적인 SQL 쿼리를 만들도록 지원한다.

### if 조건
if 엘리먼트는 test 조건이 true가 될 때만 해당되는 SQL이 쿼리에 추가된다.
```xml
<resultMap type="Course" id="CourseResult">
    <id property="courseId" column="course_id"/>
    <result property="name" column="name"/>
    <result property="description" column="description"/>
    <result property="start_date" column="startDate"/>
    <result property="end_date" column="endDate"/>
</resultMap>

<select id="searchCourses" parameterType="hashmap" resultMap="CourseResult">
    <![CDATA[
        select * from courses where tutor_id=#{tutorId}
        <if test="courseName != null">
            and name like #{courseName}
        </if>
        <if test="startDate != null">
            and start_date >= #{startDate}
        </if>
        <if test="endDate != null">
            and end_date <= #{endDate}
        </if>
    ]]>
</select>
```

### choose, when, otherwise 조건
마이바티스는 `<choose>`의 test 조건을 확인해서 가장 먼저 true가 되는 조건을 사용한다. 어느 조건도 true가 되지 못하면, `<otherwise>`절이 사용된다.
```xml
select * from courses
    <choose>
        <when test="searchBy == 'Tutor'">
            where tutor_id = #{tutorId}
        </when>
        <when test="searchBy == 'CourseName'">
            where name like #{courseName}
        </when>
        <otherwise>
            where tutor start_date &gt; = now()
        </otherwise>
    </choose>
```

### where 조건
가끔은 모든 검색 조건 중 하나도 선택하지 않을 수 있다 마이바티스는 이러한 SQL문을 만들기 위해 `<where>` 엘리먼트를 제공한다.
내부조건을 나타내는 엘리먼트에 의해 리턴되는 내용이 있을 때에만 where를 추가한다(모든 검색 조건이 필수가 아니라 선택할 수 있을 때 사용된다).
```xml
select * from courses
    <where> 
        <if test="tutorId != null">
            tutor_id=#{tutorId}    
        </if>
        <if test="courseName != null">
            and name like #{courseName}
        </if>
        <if test="startDate != null">
            and start_date >= #{startDate}
        </if>
        <if test="endDate != null">
            and end_date <= #{endDate}
        </if>
    </where>
```
### trim 조건
`<trim>` 엘리먼트는 `<where>` 엘리먼트와 유사하지만 접두사/접미사를 추가하거나 제거하는 기능을 추가로 제공한다.
`<if>`조건이 true이면 where절을 추가하고 where 뒤에 접두사 AND나 OR가 있으면 제거한다.
```xml
select * from courses
    <trim prefix="where" prefixOverrides="AND | OR">
        <if test="tutorId != null">
            tutor_id=#{tutorId}    
        </if>
        <if test="courseName != null">
            and name like #{courseName}
        </if>
    </trim>
```
### foreach 루프
배열이나 리스트를 통해 반복적인 처리를 하고 AND나 OR 조건을 붙이거나 IN 절을 처리하는 공통적인 요구사항을 담당한다.

tutor_id의 아이디가 1,3,6인 교사가 가르치는 모든 교육과정을 찾는다고 해보자. tutor_id의 아이디 목록을 매핑 구문에 전달하고 `<foreach>` 엘리먼트를
사용해서 리스트를 반복 처리해서 동적 쿼리를 만들 수 있다.
```xml
<select id="searchCoursesByTutors" parameterType="map" resultMap="CourseResult">
    select * from courses
        <if test="tutorIds != null">
            <where>
                <foreach item="tutorId" collections="tutorIds">
                    OR tutor_id=#{tutorId}
                </foreach>
            </where>
        </if>
</select>        
```

IN절을 만드는법을 알아보자.
```xml
<select id="searchCoursesByTutors" parameterType="map" resultMap="CourseResult">
    select * from courses
        <if test="tutorIds != null">
            <where>
                tutor_id IN
                    <foreach item="tutorId" collections="tutorIds" open="(" seperator="," close=")">
                        #{tutorId}
                    </foreach>
            </where>
        </if>
</select>        
```
### set 조건
`<where>` 엘리먼트와 유사하고 내부 조건이 리턴하는 내용이 있을 경우 SET을 추가할 것이다.
```xml
<update id="updateStudent" parameterType="Student">
    update students
        <set>
            <if test="name != null">name=#{name},</if>
            <if test="email != null">email=#{email},</if>
            <if test="phone != null">phone=#{phone},</if>
        </set>
    where stud_id=#{id}    
</update>
```
여기서 `<set>` 엘리먼트는 `<if>`조건이 텍스트를 리턴한다면 set 키워드를 추가하고 마지막에 콤마(,)를 제거한다.
