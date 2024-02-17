## JDBC 트랜잭션 경계 설정

```java
Connection c = dataSource.getConnection();

c.setAutoCommit(false); // 트랜잭션 경계 시작
try {

  ...
  user.upgradeLevel();
  userDao.update(c, user);

  c.commit(); // 트랜잭션 경계 끝지점 (커밋)
} catch(Exception e) {
  c.rollback(); // 트랜잭션 경계 끝지점 (롤백)
}

c.close();
```

```java
public interface UserDao {
  public void add(Connection c, User user);
  public User get(Connection c, String id);
  ...
  public void update(Connection c, User user);
}
```

이렇게 트랜잭션을 관리하는 작업을 트랜잭션 경계 설정이라고 한다.  
이렇게 하나의 DB 커넥션 안에서 만들어지는 트랜잭션을 로컬 트랜잭션이라고도 한다.  

### 문제점
Connection은 DAO가 아닌 서비스에서 관리해야 하는데, 그래야만 같은 트랜잭션 안에서 동작해야 하기 때문이다.  
즉 서비스의 메서드가 트랜잭션에서 말하는 원자가 되어야 하기 때문이다.  

위 방식의 문제점은 3가지이다.  
- JDBCTemplate을 더 이상 사용할 수 없다. JDBC API를 사용하던 초기 방식으로 돌아가게 되었다.
- UserService의 메서드에서 Connection을 다뤄야 하고 메서드 추출을 할 경우에 Connection 파라미터로 지저분해지게 된다.  
- DAO 인터페이스에서 특정 DB의 Connection에 의존하므로, DAO는 더 이상 데이터 접근 기술에 독립적일 수 없게 된다.  DI가 깨지게 된다.  


## 트랜잭션 기술 설정의 분리
위 문제를 해결하기 위해서는 JTA를 적용하는 DataSourceTransactionManager 객체를 빈으로 등록하고 UserService쪽에서 DI받도록 해야 한다.  
빈 객체는 기본적으로 싱글톤이기 때문에 멀티 스레드에 대한 걱정을 하지 않아도 된다.   

여기서 여러 DB에 걸쳐 트랜잭션 경계를 만들어야 하는 글로벌 트랜잭션이라는 새로운 요구사항이 들어왔다고 가정하자.  

지금까지 사용한 JDBC의 Connection을 이용한 트랜잭션 방식은 로컬 트랜잭션이라 글로벌 트랜잭션을 이용하려면 무언가 다른 방법이 필요하다.  

왜냐하면 로컬 트랜잭션은 하나의 DB Connection에 종속되기 때문이다.   
따라서 각 DB와 독립적으로 만들어지는 Connection을 통해서가 아니라, 별도의 트랜잭션 관리자를 통해 트랜잭션을 관리하는 글로벌 트랜잭션(Global Transaction) 방식을 사용해야 한다.  

글로벌 트랜잭션을 적용해야 트랜잭션 매니저를 통해 여러 개의 DB가 참여하는 작업을 하나의 트랜잭션으로 만들 수 있다

```java
// JNDI를 이용해 서버의 Transaction 오브젝트를 가져온다.
InitialContext ctx = new InitialContext();
UserTransaction tx = (UserTransaction)ctx.lookup(USER_TX_JNDI_NAME);

tx.begin();
// JNDI(Java Naming and Directory Interface)로 가져온 dataSource를 사용해야 한다.
Connection c = dataSource.getConnection();
try {
  // 데이터 액세스 코드
  tx.commit();
} catch (Exception e) {
  tx.rollback();
  throw e;
} finally {
  c.close();
}
```

## PSA



스프링은 트랜잭션 기술의 공통점을 담은 트랜잭션 추상화 기술을 제공한다. 이를 이용하면 특정 기술에 종속되지 않고 트랜잭션 경계 설정 작업이 가능해진다.
드를 분리한 것이다.

![image](https://github.com/1suckk/toby-spring-vol1/assets/72124326/14f27eb0-7582-4ee0-9939-b853b8efee77)


UserDao와 UserService는 각각 담당하는 코드의 기능적인 관심에 따라 분리되었다. 사실 둘은 같은 애플리케이션 로직을 담은 코드이지만 내용에 따라 분리하여, 수평적으로 분리했다고 볼 수 있다.
트랜잭션의 추상화는 이와는 좀 다르다. 애플리케이션의 비즈니스 로직과 그 하위에서 동작하는 로우레벨의 트랜잭션 기술이라는 아예 다른 계층의 특성을 갖는 코드를 분리한 것이다.

![image](https://github.com/1suckk/toby-spring-vol1/assets/72124326/28ef44c7-8996-498e-9cb3-6791f2ea44d2)


```
