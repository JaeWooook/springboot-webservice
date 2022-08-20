## 스프링부트로 웹서비스 출시하기

### 개발환경
 - IDE : IntelliJ IDEA Ultimate
 - Git Tools : Git Bash
 - OS : Window 10
 - SpringBoot 2.7.2
 - Java 17
 - Gradle

### SpringBoot & JPA로 간단한 API 만들기
 - @RestController는 @ResponseBody를 모든 메소드에서 적용해줍니다.
 - 즉 @RestController를 통해서 생성되는 메소드의 결과는 JSON형태로 반환하게 된다.

#### Post 클래스에는 JPA에서 제공하는 어노테이션이 있다.
 - @Entity
   - 테이블과 링크될 클래스임을 나타낸다.
   - 언더스코어 네이밍(_)으로 네이밍한다.
 - @Id
   - 해당 테이블의 PK 필드를 나타낸다.
 - @GeneratedValue
   - PK의 생성 규칙을 나타낸다.
   - 기본값은 AUTO로, MySQL의 auto_increment와 같이 자동증가하는 정수형 값이 된다.
   - 스프링부트 2.0에서는 옵션을 추가해야만 auto_increment가 된다.
 - @Column
   - 태이블의 컬럼을 나타내면, 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 컬럼이 된다.
   - 사용하는 이유는 기본값 외에 추가로 변경이 필요한 옵션이 있을 경우 사용한다.

> 웬만하면 Entity의 PK는 Long 타입의 Auto_increment를 추천한다.

 - @NoArgsConstructor : 기본 생성자 자동 추가
   - access = AccessLevel.PROTECTED : 기본생성자의 접근 권한을 protected로 제한
   - 생성자로 protected Posts() {} 와 같은 효과
   - __Entity 클래스를 프로젝트 코드상에서 기본생성자로 생성하는 것은 막되, JPA에서 Entity 클래스를 생성하는것은 허용하기 위해 추가__
 - @Builder : 해당 클래스의 빌더패턴 클래스를 생성
   - 생성자 상단에 선언시 생성자에 포함된 필드만 빌더에 포함
   
> Entity 클래스를 생성할때, 주의할 것은 무분별한 setter 메서드 생성이다.
> 
> 해당 클래스의 인스턴스 값들이 언제 어디서 변해야하는지 코드상으로 명확히 구분할 수가 없어, 차후에 복잡해진다.

> ibatis/MyBatis 등에서는 DAO라고 불리는 DB Layer 접근자
> JPA에선 Repository라고 부르며 인터페이스로 생성
> JpaRepository<Entity클래스, PK타입> 을 상속하면 기본적인 CRUD 메서드가 자동 생성된다.

#### JUnit 테스트

 - given
   - 테스트 기반 환경을 구축하는 단계
   - 여기선 @builder의 사용법도 같이 확인
 - when
   - 테스트하고자 하는 행위 선언
   - 여기선 Posts가 DB에 insert되는 것을 확인하기 위함
 - then
   - 테스트 결과 검증
   - 실제로 DB에 insert 되었는지 확인하기 위해 조회 후, 입력된 값 확인

#### Junit 테스트시 유의사항!
```java
Posts posts  = postsList.get(0);
        assertThat(posts.getTitle(), is("테스트 게시글"));
        assertThat(posts.getContent(), is("테스트 본문"));
```
 - assertThat을 사용할 때 JUnit의 라이브러리를 사용하고 있는지, Assertion의 라이브러리를 사용하는지 확인할 것!

#### 생성자로 주입 받는 방식
@Autowired를 사용하지 않고, @AllArgsConstructor를 사용하면
생성자를 코드로 작성하지 않아도 직접 생성해준다.

#### H2 database 연결 not found 에러 해결 방법
> 먼저 H2 database가 설치 되어있지 않다면 설치 부터 해준다.
> 
> 동일하게 Embedded로 해도된다.

```yaml
spring:
  h2:
    console:
      enabled: true

  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb;DB_CLOSE_ON_EXIT=FALSE
```
 - 이런식으로 아래의 datasource를 설정해줘야 한다. 
 
> properties파일의 형식과 의미이다

```properties
# 설명

#h2 console 활성화
spring.h2.console.enabled=true 

# 원격접속 허용
spring.h2.console.settings.web-allow-others=true

# 유일한 이름 생성 여부
spring.datasource.generate-unique-name = false 

# h2 console 경로
spring.h2.console.path=/h2-console

# h2 url 경로
spring.datasource.url=jdbc:h2:mem:testdb

# class 명칭
spring.datasource.driverClassName=org.h2.Driver
```

위와 같이 설정하면 localhost:8080/h2-console에서 확인할 수 있다.
jdbc url은 jdbc:h2:mem:testdb로 해야된다. 경로를 그렇게 지정해주었기 때문이다.

#### 포스트맨을 사용하면서 몰랐던 점
> POST 방식이던, GET이던 무언가 값을 받아서 던지게 된다면 parameter 값을 던져줘야한다. 
> 던질때는 반드시, Body가 JSON형태인지를 확인해줄 필요가 있다. 그렇지 않다면 400에러가 발생할 수 있다.

 - 참고 이미지

![image](https://user-images.githubusercontent.com/55322459/184493856-da2b868a-4d3e-4c15-9db3-e3bfe6eb1ad5.png)

#### BaseTimeEntity 클래스의 모든 Entity들의 상위 클래스가 되어 Entity들의 createdDate, modifiedDate를 자동으로 관리하는 역할을 한다.

 - @MapperSuperclass
   - JPA Entity 클래스들이 BaseTimeEntity를 상속할 경우 필드들(createdDate, modifiedDate)도 컬럼으로 인식하도록 한다.
 - @EntityListeners(AuditingEntityListener.class)
   - BaseTimeEntity클래스에 Auditing 기능을 포함 시킨다.
 - @CreatedDate
   - Entity가 생성되어 저장될 때 시간이 자동 저장된다.
 - @LastModifiedDate
   - 조회한 Entity의 값을 변경할 때 시간이 자동 저장된다.


### SpringBoot & Handlebars로 화면 만들기

#### SpringBoot 순환 참조 금지 해제
> 스프링 부트 2.6 부터 순환 참조를 기본적으로 금지하도록 되어있다.
> 그래서 __Requested bean is currently in creation: Is there an unresolvable circular reference?__ 와 같은 에러가 발생한다.
> 
> 이것은 application.yml에 아래와 같은 코드를 추가하면된다.

```yaml
spring:
  h2:
    console:
      enabled: true

  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb;DB_CLOSE_ON_EXIT=FALSE

  main:
    allow-circular-references: true
```
 - main부터 allow-circular-references: true값을 추가하면 된다.

#### @Transactional(readOnly = true) 트랜잭션에서 빨간줄 생기는 에러
트랜잭션의 import 가 alt+enter로 추가하다보니 마구잡이로 추가가 되면서

import javax.transaction.Transactional;이 선택되었다.
이걸로 하게되면 readOnly=true에 대한 옵션에서 에러가 발생한다.
해당 어노테이션에는 위와 같은 옵션이 없기 때문이다.

```java
import org.springframework.transaction.annotation.Transactional;
```
 - 위와 같은 package로 올바르게 import 하면 문제를 해결할 수 있다.

#### 트랜잭션에 readOnly 기능 옵션추가
```java
@Transactional(readOnly = true)
```
옵션(readOnly = true)을 주면 트랜잭션 범위는 유지하되, 조회 기능만 남겨두어 조회 속도가 개선되기 때문에 특별히 등록/수정/삭제 기능이 없는 메소드에선 사용하는걸 추천한다.

### AWS EC2 & AWS RDS 운영 환경 구축

### EC2 수동 배포해보기

### TravisCI & AWS CodeDeploy로 CI환경 구축하기

### Nginx로 무중단 배포 구축하기

### 스프링부트 운영 환경 설정

### Google GSuite & AWS로 도메인, Email 할당 받기, 타임존 수정
