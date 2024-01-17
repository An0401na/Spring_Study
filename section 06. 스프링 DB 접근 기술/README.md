> ##### 이 글은 인프런에서 김영한 님의 "스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술"을 수강 후 개인적으로 공부한 내용을 정리한 게시글입니다. 잘못된 점이나 부족한 부분이 있다면 언제든 지적 부탁드립니다.
>

## ✏️ H2 데이터베이스

>**자바 기반의 오픈소스 관계형 데이트베이스 관리 시스템**

- 스프링 부트가 지원하는 인메모리 관계형 데이터베이스
    - 인메모리 특징 : 검색이 빠름, 휘발성(전원이 꺼지면 데이터 삭제)
- 브라우저 기반의 콘솔 프로그램
- 적은 용량으로 설치 가능
- Embedded mode(내장모드) & Server mode(서버모드) 지원
    - Embedded mode(내장모드) : Application 서버 실행 종료시 데이터 모두 휘발
    - Server mode(서버모드) : Application 서버 종료시에도 지속적으로 해당 데이터를 사용
- 로컬환경이나 테스트 환경에서 많이 사용됨.

---

## ✏️ 순수 JDBC

### JDBC(Java Database Connectivity)

> **자바에서 데이터베이스에 접속할 수 있도록 하는 자바 API**
JDBC는 데이터베이스에서 자료를 쿼리하거나 업데이트하는 방법을 제공한다.
>

### 환경설정

1. bulid.gradle 파일에 jdbc, h2 데이터베이스 관련 라이브러리 추가

```java
implementation 'org.springframework.boot:spring-boot-starter-jdbc'
runtimeOnly 'com.h2database:h2
```

2. resource/application.properties 파일에 스프링 부트 데이터 베이스 연결 설정 추가

```java
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
```

### JDBC 레포지토리 구현

**JDBC 회원 레포지토리**

```java
package hello.hellospring.repository;
import hello.hellospring.domain.Member;
import org.springframework.jdbc.datasource.DataSourceUtils;
import javax.sql.DataSource;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;
public class JdbcMemberRepository implements MemberRepository {
    private final DataSource dataSource;
    public JdbcMemberRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    @Override
    public Member save(Member member) {
        String sql = "insert into member(name) values(?)";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
            pstmt.setString(1, member.getName());
            pstmt.executeUpdate();
            rs = pstmt.getGeneratedKeys();
            if (rs.next()) {
                member.setId(rs.getLong(1));
            } else {
                throw new SQLException("id 조회 실패");
            }
            return member;
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
    @Override
    public Optional<Member> findById(Long id) {
        String sql = "select * from member where id = ?";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql);
            pstmt.setLong(1, id);
            rs = pstmt.executeQuery();
            if(rs.next()) {
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                return Optional.of(member);
            } else {
                return Optional.empty();
            }
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
    @Override
    public List<Member> findAll() {
        String sql = "select * from member";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql);
            rs = pstmt.executeQuery();
            List<Member> members = new ArrayList<>();
            while(rs.next()) {
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                members.add(member);
            }
            return members;
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
    @Override
    public Optional<Member> findByName(String name) {
        String sql = "select * from member where name = ?";
        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql);
            pstmt.setString(1, name);
            rs = pstmt.executeQuery();
            if(rs.next()) {
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                return Optional.of(member);
            }
            return Optional.empty();
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
    private Connection getConnection() {
        return DataSourceUtils.getConnection(dataSource);
    }
    private void close(Connection conn, PreparedStatement pstmt, ResultSet rs) {
        try {
            if (rs != null) {
                rs.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            if (pstmt != null) {
                pstmt.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            if (conn != null) {
                close(conn);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    private void close(Connection conn) throws SQLException {
        DataSourceUtils.releaseConnection(conn, dataSource);
    }
}
```

> **JDBC 사용시 반복되는 부분**
>
> 1. 커넥션 획득
> 2. statement를 준비하고 실행결과를 반복하도록 루프를 실행
> 3. 커넥션 종료, statement 및 resultset 종료트
> 4. 랜잭션을 다루기 위한 커넥션 동기화
> 5. 예외 발생 시 스프링 예외 변환기 실행

**SpringConfig 변경**

```java
package hello.hellospring;
import hello.hellospring.repository.JdbcMemberRepository;
import hello.hellospring.repository.JdbcTemplateMemberRepository;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.sql.DataSource;
@Configuration
public class SpringConfig {
    private final DataSource dataSource;
    public SpringConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }
    @Bean
    public MemberRepository memberRepository() {
// return new MemoryMemberRepository();
        return new JdbcMemberRepository(dataSource);
    }
}
```

![](https://velog.velcdn.com/images/dksek3050/post/2a515731-359f-430b-acd9-3805c018cfbc/image.png)


> **개방-폐쇄 원칙(OCP, Open-Closed Principle)** : 확장에는 열려있고, 수정, 변경에는 닫혀있다.
>
> - 기존 메모리를 사용하여 회원을 저장하던 방식인 MemoryMemberRepository를 주입하지 않고 Jdbc 방식인 JdbcMemberRepository를 주입하도록 수정하였다.
> - 스프링의 DI (Dependencies Injection)을 사용하면 기존 코드를 전혀 손대지 않고, 설정만으로 구현 클래스를 변경할 수 있다.

---

## ✏️ 스프링 통합 테스트

>
>
> - @SpringBootTest : 스프링 컨테이너와 테스트를 함께 실행한다.
> - @Transactional : 테스트 케이스에 이 애노테이션이 있으면, 테스트 시작 전에 트랜잭션을 시작하고, 테스트
    > 완료 후에 항상 롤백한다. 이렇게 하면 DB에 데이터가 남지 않으므로 다음 테스트에 영향을 주지 않는다.

## ✏️ 스프링 JdbcTemplate

> **개발자가 JDBC 기술을 쉽게 사용할 수 있도록 단순화하고 반복잡업을 대신 처리해주는 클래스**
>
>
> JdbcTemplate은 spring-jdbc에 포함되어 있다.
>

**JdbcTemplate 회원 레포지토리**

```java
package hello.hellospring.repository;
import hello.hellospring.domain.Member;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.simple.SimpleJdbcInsert;
import javax.sql.DataSource;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;
public class JdbcTemplateMemberRepository implements MemberRepository {
    private final JdbcTemplate jdbcTemplate;
    public JdbcTemplateMemberRepository(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }
    @Override
    public Member save(Member member) {
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");
        Map<String, Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());
        Number key = jdbcInsert.executeAndReturnKey(new
                MapSqlParameterSource(parameters));
        member.setId(key.longValue());
        return member;
    }
    @Override
    public Optional<Member> findById(Long id) {
        List<Member> result = jdbcTemplate.query("select * from member where id
                = ?", memberRowMapper(), id);
        return result.stream().findAny();
    }
    @Override
    public List<Member> findAll() {
        return jdbcTemplate.query("select * from member", memberRowMapper());
    }
    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = jdbcTemplate.query("select * from member where
                name = ?", memberRowMapper(), name);
        return result.stream().findAny();
    }
    private RowMapper<Member> memberRowMapper() {
        return (rs, rowNum) -> {
            Member member = new Member();
            member.setId(rs.getLong("id"));
            member.setName(rs.getString("name"));
            return member;
        };
    }

```

기존 JDBC 보다 단계가 훨씬 줄어든걸 확인할 수 있다.

Connection하고 자원을 할당하고 관리하는 부분이 알아서 처리되었다.

**그러나 SQL문은 직접 작성해야한다.**

## ✏️ JPA(Java Persistence API)

> **자바에서 ORM(Object-Relational Mapping) 기술 표준으로 사용되는 인터페이스의 모음**
>
>
> ![](https://velog.velcdn.com/images/dksek3050/post/c46e3098-84a7-405c-8bab-4fd24a7f76c6/image.png)
>
> - 실제적으로 구현된것이 아니라 구현된 클래스와 매핑을 해주기 위해 사용되는 프레임워크이다. JPA를 구현한 대표적인 오픈소스로는 Hibernate가 있다.
> - SQL문이 아닌 Method를 통해 DB를 조작할 수 있어, 개발자는 객체 모델을 이용하여 비즈니스 로직을 구성하는데만 집중할 수 있다. (내부적으로는 쿼리를 생성하여 DB를 조작)
> - 객체지향적인 코드 작성이 가능하다. 오직 객체지향적 접근만 고려하면 되기 때문에 생산성 증가
> - 매핑하는 정보가 Class로 명시 되었기 때문에 ERD를 보는 의존도를 낮출 수 있고 유지보수 및 리팩토링에 유리

### 환경설정

1. **build.gradle 파일에 JPA, h2 데이터베이스 관련 라이브러리 추가**

```java
dependencies{
        implementation'org.springframework.boot:spring-boot-starter-thymeleaf'
        implementation'org.springframework.boot:spring-boot-starter-web'
        implementation'org.springframework.boot:spring-boot-starter-data-jpa'
        runtimeOnly'com.h2database:h2'
        testImplementation('org.springframework.boot:spring-boot-starter-test'){
        exclude group:'org.junit.vintage',module:'junit-vintage-engine'
        }
}
```

spring-boot-starter-data-jpa 는 내부에 jdbc 관련 라이브러리를 포함

1. resource/application.properties 파일에 JPA설정 추가

```java
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa

spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
```

- show-sql : JPA가 생성하는 SQL을 출력

    ```java
    spring.jpa.properties.hibernate.format_sql=true
    ```

  format_sql 설정을 true 로 하면 한눈에 보기 더 좋다.

- ddl-auto : JPA는 테이블을 자동으로 생성하는 기능을 제공하는데 none 를 사용하면 해당 기능을 끈다.
    - create 를 사용하면 엔티티 정보를 바탕으로 테이블도 직접 생성해준다.

### JPA 엔티티 매핑

```java
package hello.hellospring.domain;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
@Entity
public class Member {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    public Long getId() {
        return id;
    }
    public void setId(Long id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

### JPA 회원 레포지토리

```java
package hello.hellospring.repository;
import hello.hellospring.domain.Member;
import javax.persistence.EntityManager;
import java.util.List;
import java.util.Optional;
public class JpaMemberRepository implements MemberRepository {
    private final EntityManager em;
    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }
    public Member save(Member member) {
        em.persist(member);
        return member;
    }
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where
                m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
        return result.stream().findAny();
    }
}
```

### 서비스 계층에 트랜젝션 추가

```java
import org.springframework.transaction.annotation.Transactional
@Transactional
public class MemberService {}

```

- **JPA를 통한 모든 데이터 변경은 트랜잭션 안에서 실행해야 한다.**
- 스프링은 해당 클래스의 메서드를 실행할 때 트랜잭션을 시작하고, 메서드가 정상 종료되면 트랜잭션을 커밋한다.
  만약 런타임 예외가 발생하면 롤백한다.

### JPA를 사용하도록 스프링 설정 변경

```java
package hello.hellospring;
import hello.hellospring.repository.*;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.persistence.EntityManager;
import javax.sql.DataSource;
@Configuration
public class SpringConfig {
    private final DataSource dataSource;
    private final EntityManager em;
    public SpringConfig(DataSource dataSource, EntityManager em) {
        this.dataSource = dataSource;
        this.em = em;
    }
    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }
    @Bean
    public MemberRepository memberRepository() {
// return new MemoryMemberRepository();
// return new JdbcMemberRepository(dataSource);
// return new JdbcTemplateMemberRepository(dataSource);
        return new JpaMemberRepository(em);
    }
}
```

- 엔티티를 관리하는 EntityManager 주입
- 엔티티 매니저 내부에는 영속성 컨텍스트(PersistanceContext)가 존재하는데 이를 통해 엔티티를 관리한다.

---

## ✏️ 스프링 데이터 JPA

>**스프링 데이터의 공통적인 기능에서 JPA의 유용한 기술이 추가된 기술**
>![](https://velog.velcdn.com/images/dksek3050/post/9f58945b-fd9d-49fa-9a18-18f8da473fd4/image.png)
>
>- CRUD를 포함된 여러 메소드를 포함하여 알아서 쿼리를 생성
>- 페이징 처리 기능과 메서드를 이름으로 자동으로 쿼리를 빌딩하는 기능 등 여러 기능 제공

### 스프링 데이터 JPA 회원 레포지토리

```java
package hello.hellospring.repository;
import hello.hellospring.domain.Member;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;
public interface SpringDataJpaMemberRepository extends JpaRepository<Member,
        Long>, MemberRepository {
    Optional<Member> findByName(String name)
}
```

### 스프링 데이터 JPA를 사용하도록 스프링 설정 변경

```java
package hello.hellospring;
import hello.hellospring.repository.*;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class SpringConfig {
    private final MemberRepository memberRepository;
    public SpringConfig(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository);
    }
}
```

스프링 데이터 JPA가 `SpringDataJpaMemberRepository` 를 스프링 빈으로 자동 등록
