# spring_DB_1
스프링 DB 1편 - 데이터 접근 핵심 원리

# 스프링 DB 1편 - 데이터 접근 핵심 원리



![image-20220515204542049](/Users/geumbit/Library/Application Support/typora-user-images/image-20220515204542049.png)



build.gradle에 아래 코드를 추가하여 @Slfj4등을 사용 가능

```java
//테스트에서 lombok 사용
testCompileOnly 'org.projectlombok:lombok'
testAnnotationProcessor 'org.projectlombok:lombok'
```



### JDBC 표준 인터페이스

JDBC(Java Database Connectivity)는 자바에서 데이터베이스에 접속할 수 있도록 하는 자바API다.

​	JDBC는 Java에서 DB를 사용할 경우 Mysql, Oracle등 여러 종류의 DB들을 연결, SQL전달, 결과응답하는 방법의 <표준 인터페이스>를 만들어놓았다. 결과적으로 각 드라이버구현체만 넣으면 Java와 DB의 연결을 손쉽게 할 수 있다.

![image-20220515213319483](/Users/geumbit/Library/Application Support/typora-user-images/image-20220515213319483.png)



### JDBC와 최신 데이터 접근 기술

SQL Mapper와 ORM 기술로 나눌 수 있다.



### 1. SQL Mapper

![image-20220515213354927](/Users/geumbit/Library/Application Support/typora-user-images/image-20220515213354927.png)

1. SQL Mapper

   - 장점: 
     - JDBC를 편리하게 사용하도록 도와준다.
     - SQL 응답 결과를 객체로 편리하게 변환해준다.
     - JDBC의 반복 코드를 제거해준다.
   - 단점: 개발자가 SQL을 직접 작성해야한다. 

   - 대표 기술: 스프링 JdbcTemplate, MyBatis



### 2. ORM 기술

![image-20220515213526223](/Users/geumbit/Library/Application Support/typora-user-images/image-20220515213526223.png)

- ORM은 객체를 관계형 데이터베이스 테이블과 매핑해주는 기술이다. <br>이 기술 덕분에 개발자는 반복적인 SQL을 직접 작성하지 않고, ORM 기술이 개발자 대신에 SQL을 동적으로 만들어 실행해준다. <br>추가로 각각의 데이터베이스마다 다른 SQL을 사용하는 문제도 중간에서 해결해준다. 
- 대표 기술: JPA, 하이버네이트, 이클립스링크
- JPA는 자바 진영의 ORM 표준 인터페이스이고, 이것을 구현한 것으로 하이버네이트와 이클립스 링크 등의 구현 기술이 있다.

----

# 데이터베이스 연결



**ConnectionConst**

```java
public abstract class ConnectionConst {
        public static final String URL = "jdbc:h2:tcp://localhost/~/test";
        public static final String USERNAME = "sa";
        public static final String PASSWORD = "";
}
```



**DBConnectionUtil**

```java
@Slf4j
public class DBConnectionUtil {
    public static Connection getConnection() {
        try {
            Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);
            log.info("get connection={}, class{}", connection, connection.getClass());
            return connection;
        } catch (SQLException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```



DBConnectionUtil.getConnection()가 진짜 DB연결이 잘되는지 TestCode로 확인해보자.

```java
@Slf4j
public class DBConnectionUtilTest {

    @Test
    void connection(){
        Connection connection = DBConnectionUtil.getConnection();
        Assertions.assertThat(connection).isNotNull();
    }
}
```

![image-20220515222004610](/Users/geumbit/Library/Application Support/typora-user-images/image-20220515222004610.png)



성공은 떨어졌는데 여기서 궁금점!

DBConnectionUtil에서 사용된 **Connection** interface인데 <br>어떻게 h2 드라이드 구현체를 찾아가서 연결해준걸까?

**DriverManager.getConnection()**에서 설치된 Driver구현체들을 모두 DriverManager에 등록된다. 구현체들은 Connection객체를 상속받았다.

![image-20220515222525891](/Users/geumbit/Library/Application Support/typora-user-images/image-20220515222525891.png)

 

DriverManager에 등록된 DB구현체들이 for문을 돌아 올바른 Connection객체를 반환해준다

![image-20220515222301907](/Users/geumbit/Library/Application Support/typora-user-images/image-20220515222301907.png) <br>그림으로 설명하면 아래 화면과 같다.



![image-20220515222109678](/Users/geumbit/Library/Application Support/typora-user-images/image-20220515222109678.png)

1. 애플리케이션 로직에서 커넥션이 필요하면 DriverManager.getConnection() 을 호출한다.

2. DriverManager 는 라이브러리에 등록된 드라이버 목록을 자동으로 인식한다. 이 드라이버들에게순서대로 다음 정보를 넘겨서 커넥션을 획득할 수 있는지 확인한다.
   - URL: 예) jdbc:h2:tcp://localhost/~/test
   - 이름, 비밀번호 등 접속에 필요한 추가 정보
   - 여기서 각각의 드라이버는 URL 정보를 체크해서 본인이 처리할 수 있는 요청인지 확인한다. 예를 들어서 URL이 jdbc:h2 로 시작하면 이것은 h2 데이터베이스에 접근하기 위한 규칙이다. 따라서 H2 드라이버는 본인이 처리할 수 있으므로 실제 데이터베이스에 연결해서 커넥션을 획득하고 이 커넥션을 클라이언트에 반환한다. 반면에 URL이 jdbc:h2 로 시작했는데 MySQL 드라이버가 먼저 실행되면 이 경우 본인이 처리할 수 없다는 결과를 반환하게 되고, 다음 드라이버에게 순서가 넘어간다.

3. 이렇게 찾은 커넥션 구현체가 클라이언트에 반환된다.



