# 예외 종류


![](https://velog.velcdn.com/images/salgu1998/post/c8310c15-9633-4d69-b86e-61c884a01593/image.png)
## Error
java.lang.Error 클래스의 하위 클래스들이다. 

Error는 메모리가 부족하는 등과 같이 **시스템이 비정상적인 상황**인 경우에 사용한다. 

**주로 JVM에서 발생**시키기 때문에 **애플리케이션 코드에서 잡아서는 안되며**, 잡아서 대응할 수 있는 방법도 없다. 
따라서 시스템 레벨에서 특별한 작업을 하는게 아니라면 이러한 에러 처리는 하지 않아도 된다.

## Exception
java.lang.Exception 클래스와 하위 클래스들은 Error와 달리 **애플리케이션 코드에서 예외가 발생하였을 경우**에 사용된다. 그리고 Exception 은 다시 체크 예외와 언체크 예외로 구분된다.

### Checked Exception
**명시적인 예외처리를 강제**하기 때문에 체크 예외라고 불린다. 

체크 예외는 **RuntimeException 클래스를 상속받지 않은 예외 클래스들**이다. 
체크 예외는 복구 가능성이 있는 예외이므로 **반드시 예외를 처리하는 코드를 함께 작성해야 한다**. 만약 예외를 처리하지 않으면 컴파일 에러가 발생한다.

대표적으로 IOException, SQLException 등이 있으며, 예외를 처리하기 위해서는 catch 문으로 잡거나 throws를 통해 메소드 밖으로 던질 수 있다. 

체크 예외란 Exception 클래스 하위 클래스이며, 언체크 예외란 Exception 하위의 RuntimException 하위의 예외이다. (Java 예외 종류에 대해 모르면 [이 글](https://mangkyu.tistory.com/152)을 참고해주세요)


체크 예외는 **개발자가 실수로 예외 처리를 누락하지 않도록 컴파일러가 도와준다**. 
하지만 개발자가 모든 체크 예외를 처리해주어야 하므로 **번거로우며**, 신경쓰지 않고 싶은 예외까지 처리해야 한다는 단점이 있다.


### Unchecked Exception
명시적인 예외처리를 강제하지 않기 때문에 언체크 예외라고 불린다. 


RuntimeException 클래스를 상속받는 예외 클래스들은 **복구 가능성이 없는 예외들**이므로 **컴파일러가 예외처리를 강제하지 않는다**. 

그래서  언체크 예외라고 불리는데, 언체크 예외는 Error와 마찬가지로 에러를 처리하지 않아도 컴파일 에러가 발생하지 않는다. 
즉, 런타임 예외는 **예상치 못했던 상황에서 발생하는 것이 아니므로 굳이 예외 처리를 강제하지 않는다**. RuntimeException에는 대표적 NullPointerException이나 IllegalArgumentException 등과 같은 것들이 있다. 

## java 예외와 스프링 @Transactional
체크 예외와 언체크 예외의 차이를 아는 것은 매우 중요하다. 왜냐하면 스프링 프레임워크가 제공하는 선언적 트랜잭션(@Transactional)안에서 에러 발생 시 **체크 예외는 롤백이 되지 않고, 언체크 예외는 롤백**이 되기 때문이다. 
이는 자바 언어와는 무관하게 프레임워크의 기능임을 반드시 알고 넘어가도록 하자. (물론 옵션을 변경할 수 있다.)


# 예외 처리 방법과 전략


## 예외 복구
예외 상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것.  


## 예외 처리 회피

예외 처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것.   
`throws` 문. 

JDBCTemplate이 사용하는 콜백 객체의 메소드는 모두 throws SQLException이 붙어있다.  
SQLException을 처리하는 일은 콜백 로브젝트의 역할이 아니라고 보기 때문이다.  

```java
// JdbcTemplate.java

@Override  
public void execute(final String sql) throws DataAccessException {  
	if (logger.isDebugEnabled()) {  
	logger.debug("Executing SQL statement [" + sql + "]");  
}  
  
	/**  
	* Callback to execute the statement.  
	*/  
	class ExecuteStatementCallback implements StatementCallback<Object>, SqlProvider {  
	
		@Override  
		@Nullable  
		public Object doInStatement(Statement stmt) throws SQLException {  
			stmt.execute(sql);  
			return null;  
		}  
		
		@Override  
		public String getSql() {  
			return sql;  
			}  
		}  
	  
	execute(new ExecuteStatementCallback(), true);  
}
```


이렇게 긴밀하게 역할을 분담하는 관계가 아니라면 자신의 코드에서 발생하는 예외를 그냥 던져버리는건 **무책임한 책임 회피일 수 있다**. 

## 예외 전환

예외 회피와 비슷하게 예외를 메서드 밖으로 던지는 것이다.  
그러나 예외 회피와는 달리 발생한 예외를 적절한 예외로 전환해서 던진다.   

두 가지 목적 
- 의미를 분명하게 해주는 예외로 바꾸기 위해
- Unchecked Exception인 런타임 예외로 바꾸기 위해


```java
public void add(User user) throws DuplicateUserIdException, SQLException {
  try {
    // JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
    // 그런 기능을 가진 다른 SQLException을 던지는 메소드를 호출하는 코드
  }
  catch(SQLException e) {
    // ErrorCode가 MySQL의 "Duplicate Entry(1062)"이면 예외 전환
    if (e.getERrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
      throw DuplicateUserException();
    else
      throw e; // 그 외의 경우는 SQLException 그대로
  }
}
```


## **💬 언제 `체크드`로, 언제 `언체크드`로 바꿀까?**

- 비즈니스적으로 의미도 없고, **복구 가능하지도 않은 예외에 대해서는 런타임 예외(언체크드 예외)로 포장**해서 던지는 편이 낫다.
- 반대로 **어플리케이션 로직으로 인한 예외는 체크 예외를 사용하는게 적절**하다. 이에 대한 적절한 대응이나 복구전략이 필요하기 때문이다!

> 어차피 복구가 불가능한 예외라면 가능한 한 빨리 런타임 예외로 포장해 던지게 해서 다른 계층의 메소드를 작성할 때 불필요한 throws 선언이 들어가지 않도록 해줘야 한다.


## 예외 처리 전략

💬 **런타임 예외의 보편화**

> 자바 환경이 서버 환경으로 넘어가면서, 체크 예외의 활용도와 가치는 떨어지고 있다.

체크 예외는 복구할 가능성이 조금이라도 있는, 말 그대로 예외적인 상황이기 때문에 자바는 이를 처리하는 catch 블록이나 throws 선언을 강제하고 있다.

- 예외가 발생할 가능성이 있는 API 메소드를 사용하는 개발자의 실수를 방지하기 위한 배려이다.
- 하지만 실제로는 예외를 제대로 다루고 싶지 않을 만큼 짜증나게 만드는 원인이 되기도 한다.

서버 환경은 일반적인 애플리케이션 환경과 다르다.

어플리케이션 환경

- 자바 초기 AWT, 스윙 에서는 파일을 처리한다고 했을 때, 해당 파일이 처리 불가능하면 복구가 필요했다.
- 워드에서 특정이름의 파일을 검색할 수 없다고 프로그램이 종료되어선 안된다!

**서버 환경**

- 한 번에 다수의 사용자가 접근하여 해당 서비스를 이용하기 때문에 작업을 중지하고 예외 상황을 복구할 수 없다!
- 어플리케이션 차원에서 예외상황을 파악하고 요청에 대한 작업을 취소하는 편이 좋다.

**💬 어플리케이션 예외**

런타임 예외 중심 전략은 굳이 이름을 붙이자면 낙관적 예외처리 기법이다.

- 복구할 수 있는 예외는 없다고 가정한다.
- 예외가 생겨도 어차피 런타임 예외 이므로 시스템에서 알아서 처리해줄 것이고, 꼭 필요한 경우는 런타임 예외라도 잡아서 복구하거나 대응할 수 있으니 문제될 것이 없다는 낙관적 태도를 기반으로 한다.
- 혹시 놓치는 예외가 있을까 처리를 강제하는 체크 예외의 비관적인 접근 방법과 대비된다.
