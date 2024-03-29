# 6.1 트랜잭션 코드의 분리
- 비즈니스 로직보다 트랜잭션 코드가 더 많이 차지하는 문제 잔존
  - 메소드를 정확히 분리하기
    - 문자 그대로 메소드 내에서 코드를 분리하는 방법을 의미한다.
  - DI 이용해서 트랜잭션을 분리하기
    - UserService라는 클래스를 인터페이스로 바꾸고 그 아래에 비즈니스 로직과 트랜잭션을 담당하는 클래스를 두도록 한다.
    - 트랜잭션 경계 설정에는 자바에서의 TransactionManager를 사용한다.
# 6.2 고립된 단위 테스트
단위 테스트는 가능한 작은 단위에서 테스트 하는 것이 제일 좋다. 그러나, 코드를 작성하다보면 의존관계는 계속 복잡해질 수 밖에 없다. 
여기에서 테스트 대상을 고립시키고 테스트를 하는 것이 시간을 절약하고 계산 비용을 줄일 수 있다.

단위 테스트를 진행할 때 스텁이나 목 오브젝트를 사용하게 되는데 이때 mockito라는 프레임워크를 사용하는 것이 편리하다.
# 6.3 다이내믹 프록시와 팩토리 빈
프록시는 타깃과 같은 인터페이스를 구성해서 클라이언트로 하여금 클라이언트와 타깃을 연결해주는 역할을 지칭한다. 
이때 프록시는 핵심기능 인터페이스와 부가기능 코드로 구분되는데 자신이 가진 부가기능과 타깃의 핵심 기능을 함께 부여해준다.
예를 들면, 비즈니스 로직 코드(주기능)와 함께 트랜잭션 기능(부가기능)을 부여하는 것처럼. 

데코레이터 패턴은 여러 겹으로 구성되며 여러 개의 프록시가 생성된다.
대표적으로 자바 io 패키지에서 InputStream과 OutputStream 구현 클래스가 이에 해당한다. 

java.lang.reflect에서 패키지 안에 프록시를 손쉽게 만들 수 있도록 도와준다. 또한 다이내믹 프록시에는 TransactionHandler가 사용된다.

팩토리 빈이란 스프링을 대신해서 오브젝트의 생성로직을 담당하도록 만들어진 특별한 빈을 의미한다.
# 6.4 스프링의 프록시 팩토리 빈
ProxyFactoryBean을 사용하면 된다. 이때, 어드바이스인 MethodInterceptor의 invoke() 메서드를 사용하여 타깃 오브젝트의 정보를 제공받을
수 있게 된다. 어드바이스는 타깃에게 제공할 부가기능을 담은 모듈을 의미한다.
# 6.5 스프링 AOP
빈 후처리기를 이용한 자동 프록시 생성기를 이용한다. 스프링에서 제공하는 DefaultAdvisorAutoProxyCreator를 활용하면 된다.

AOP: 애스펙트 지향 프로그래밍, 애스펙트란 트랜잭션 경계설정과 같이 비즈니스 로직과 분리되는 영역을 어떻게 모듈화할까에 대한 고민을
해결하기 위한 일종의 방법론이라고 할 수 있다. 
# 6.6 트랜잭션 속성
트랜잭션의 경계는 매니저에게 트랜잭션을 가져오는 것, commit(), rollback()을 호출하는 것으로 설정되고 있다. 

트랜잭션 전파는 트랜잭션의 경계에서 이미 진행 중인 트랜잭션이 있을 때 또는 없을 때 어덯게 동작할 것인지를 결정하는 방식을 의미한다. 

격리수준은 모든 DB 트랜잭션에 해당하는 요소인데, 한 서버에 여러 개의 트랜잭션이 동시에 진행될 때 정해줘야 하는 지표이다. 
# 6.7 애노태이션 트랜잭션 속성과 포인트컷 
트랜잭션 애노테이션에는 @Transactional을 주로 사용한다.
구성 요소를 살펴보면, 먼저 @Target 부분은 애노테이션을 사용할 대상을 지정하는 부분, @Retention은 애노테이션 정보가 언제까지 유지되는지를 정하는 부분을 의미한다.

결국 DB에서 사용하는 트랜잭션 속성을 정의하기 위한 목적이다. 
# 6.8 트랜잭션 지원 테스트
선언적 트랜잭션은  Spring Framework 의 강력한 AOP(Aspect Oriented Programming) 기능을 이용해 일련의 작업들을 트랜잭션으로 묶는 과정을 의미한다.
개발자는 단순히 Spring AOP 에서 제공하는 @Transactional 어노테이션을 트랜잭션으로 묶고자하는 메서드에 선언해주면 된다.

트랜잭션 동기화란 데이터베이스 시스템에서 여러 개의 트랜잭션이 동시에 실행될 때,
이러한 트랜잭션들 사이에서 데이터 일관성을 유지하고 충돌을 방지하기 위해 동기화 메커니즘을 적용하는 것을 말한다. 
