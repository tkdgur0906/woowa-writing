# @Transactional의 원리와 사용방법 알아보기

스프링을 사용하는 개발자라면 `@Transactional`을 한 번쯤 사용해 보셨을 것입니다. 

이번 글에서는 `@Transactional`이 내부적으로 어떻게 구현되어 있는지, 그리고 이를 잘 사용하기 위해서 알아야할 것들에 대해 다뤄보고자 합니다.

---

## 트랜잭션 관리 방법
트랜잭션 관리 방법은 크게 명시적 트랜잭션과 선언적 트랜잭션으로 나눌 수 있습니다. 두 가지의 차이를 살펴보겠습니다.

## 명시적 트랜잭션
명시적 트랜잭션은 개발자가 코드에서 직접 트랜잭션을 관리하는 방식을 말합니다.

### 특징
1. 명확한 트랜잭션 제어: 개발자는 트랜잭션의 시작, 커밋, 롤백 시점을 명시적으로 정의할 수 있어 트랜잭션을 세밀하게 제어할 수 있습니다.
2. 복잡한 트랜잭션 로직 처리 가능: 선언적 트랜잭션과 달리, 트랜잭션 중 일부 작업만 커밋하고 나머지를 롤백하거나, 예외 상황에 따라 트랜잭션 처리를 유연하게 할 수 있습니다.
이와 같은 방식은 복잡한 비즈니스 로직이나 예외 상황을 고려해야 할 때 유용하게 활용될 수 있습니다.

```java
@Service
public class MyService {

    @Autowired
    private PlatformTransactionManager transactionManager;

    public void performService() {
        // 트랜잭션 시작
        TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
        
        try { 
            // 작업 성공 시 트랜잭션 커밋
            transactionManager.commit(status);
        } catch (Exception e) {
            // 예외 발생 시 트랜잭션 롤백
            transactionManager.rollback(status);
            throw e;
        }
    }
}
```
## 선언적 트랜잭션
선언적 트랜잭션은 트랜잭션 관리를 코드에서 직접 수행하지 않고, 어노테이션이나 XML을 사용해 선언적으로 관리하는 방식을 말합니다.

### 특징
코드 간결성: 트랜잭션 관리 로직을 코드에서 직접 처리하지 않기 때문에 비즈니스 로직과 트랜잭션 관리가 분리되어 코드가 더 간결해지고 가독성이 높아집니다.

1. 자동 트랜잭션 관리: 애너테이션이나 XML 설정으로 트랜잭션을 정의하면, 트랜잭션의 시작, 커밋, 롤백이 자동으로 처리됩니다. 개발자는 직접 트랜잭션을 관리할 필요가 없어 편리합니다.

2. 유연한 설정 가능: 트랜잭션의 전파 수준(Propagation), 고립 수준(Isolation), 시간 초과(Timeout), 읽기 전용 여부(ReadOnly) 등 다양한 속성을 세부적으로 설정할 수 있어, 비즈니스 요구에 맞게 트랜잭션을 조정할 수 있습니다.

3. 선언적 트랜잭션은 개발자에게 직관적인 트랜잭션 관리 방식을 제공하며, 코드를 더 깔끔하게 유지할 수 있도록 도와줍니다.

### xml 기반 트랜잭션 처리
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 트랜잭션 매니저 설정 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 트랜잭션 애너테이션 활성화 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
    
</beans>

```


### 어노테이션 기반 트랜잭션 처리
```java
@Service
public class MyService {

    @Transactional
    public void performTransaction() {
        // 트랜잭션이 자동으로 시작됨
        // 비즈니스 로직 수행
        // 작업이 성공하면 트랜잭션이 커밋됨
        // 예외가 발생하면 트랜잭션이 롤백됨
    }
}
```

여러 가지 트랜잭션 처리 방법이 있지만, 이번 글에서는 `@Transactional`에 대해 자세히 살펴보겠습니다.

---

## @Transactional 속성
`@Transactional`은 트랜잭션의 동작 방식을 세밀하게 제어할 수 있는 여러 속성을 제공합니다. 이 속성을 사용해 트랜잭션의 전파 수준, 고립 수준, 타임아웃, 읽기 전용 여부 등을 설정할 수 있습니다.

각 속성의 자세한 설정 방법은 생략하고, 각각의 의미를 간단하게 설명하겠습니다.

```java
public enum Propagation {
    REQUIRED(0),
    SUPPORTS(1),
    MANDATORY(2),
    REQUIRES_NEW(3),
    NOT_SUPPORTED(4),
    NEVER(5),
    NESTED(6);

    private final int value;

    private Propagation(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }
}
```

### 전파 수준(propagation)
트랜잭션 전파는 현재 실행 중인 메서드가 이미 존재하는 트랜잭션 내에서 실행될 것인지, 아니면 새로운 트랜잭션을 시작할 것인지를 결정하는 속성입니다.

```java
public enum Propagation {
    
    REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),
    SUPPORTS(TransactionDefinition.PROPAGATION_SUPPORTS),
    MANDATORY(TransactionDefinition.PROPAGATION_MANDATORY),
    REQUIRES_NEW(TransactionDefinition.PROPAGATION_REQUIRES_NEW),
    NOT_SUPPORTED(TransactionDefinition.PROPAGATION_NOT_SUPPORTED),
    NEVER(TransactionDefinition.PROPAGATION_NEVER),
    NESTED(TransactionDefinition.PROPAGATION_NESTED);
}
```

### 고립 수준(isolation)
고립 수준은 여러 트랜잭션이 동시에 실행될 때 데이터 일관성을 어떻게 유지할 것인지를 결정하는 속성입니다. 

SQL의 고립 수준과 동일하게 동작하며, 동시성 문제를 방지하기 위해 사용됩니다.

```java
public enum Isolation {
    
	DEFAULT(TransactionDefinition.ISOLATION_DEFAULT),
	READ_UNCOMMITTED(TransactionDefinition.ISOLATION_READ_UNCOMMITTED),
	READ_COMMITTED(TransactionDefinition.ISOLATION_READ_COMMITTED),
	REPEATABLE_READ(TransactionDefinition.ISOLATION_REPEATABLE_READ),
	SERIALIZABLE(TransactionDefinition.ISOLATION_SERIALIZABLE);
}
```

### 타임 아웃(timeout)
트랜잭션이 완료되기까지의 최대 시간을 초 단위로 설정합니다. 설정된 시간이 초과되면 트랜잭션은 자동으로 롤백됩니다

```java
int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;
```

### 읽기 전용 여부(readOnly)

트랜잭션이 데이터 읽기만 수행할 때 `readOnly = true`로 설정하면 성능 최적화를 할 수 있습니다.

만약 read/write DB가 분리되어 있다면, 읽기 전용 트랜잭션은 읽기 전용 데이터베이스에서 처리되어 데이터베이스의 부하를 줄이고, 성능을 더욱 향상시킬 수 있습니다.

```java
boolean readOnly() default false;
```

---
## @Transactional의 작동 원리

스프링의 3대 요소라고 칭해지는 기술은 IoC/DI, PSA, AOP입니다.
`@Transactional`은 내부적으로 AOP를 사용하여 트랜잭션을 처리합니다.

앞서 말했듯이 선언적 트랜잭션 방식인 `@Transactional`의 장점은 트랜잭션 처리 로직과 비즈니스 로직을 분리할 수 있다는 점입니다.

AOP를 활용한 `@Transactional`의 작동 방식은 다음 그림처럼 진행됩니다.

![Transactional 구조](./image/transactional_structure.png)


### 트랜잭션 처리 로직
AOP라는 개념이 생소하기 때문에 어렵게 느껴질 수 있지만, 
선언적 트랜잭션은 명시적 트랜잭션의 코드를 다른 곳으로 이동한 것이라 보면 편합니다. 트랜잭션 처리 로직이 어떻게 수행되는지 살펴보겠습니다.

#### 1. 트랜잭션 매니저 획득
- `TransactionInterceptor`에서 사용자가 호출한 메서드에 `@Transactional`이 붙어 있는지 검증합니다.
- 만약 `@Transactional`이 붙어 있는 메서드라면 트랜잭션 매니저를 반환하고, 그렇지 않으면 `null`을 반환합니다.
```java
//TransactionAspectSupport.class
protected Object invokeWithinTransaction(Method method, @Nullable Class<?> targetClass,
			final InvocationCallback invocation) throws Throwable {

    // If the transaction attribute is null, the method is non-transactional.
    TransactionAttributeSource tas = getTransactionAttributeSource();
    final TransactionAttribute txAttr = (tas != null ? tas.getTransactionAttribute(method, targetClass) : null);
    final TransactionManager tm = determineTransactionManager(txAttr);
```

#### 2. 트랜잭션 획득
- 트랜잭션 매니저가 트랜잭션을 시작합니다.
```java
//TransactionAspectSupport.class
protected TransactionInfo createTransactionIfNecessary(@Nullable PlatformTransactionManager tm,
                                                       @Nullable TransactionAttribute txAttr, final String joinpointIdentification) {

    TransactionStatus status = null;

    //...
    if (txAttr != null) {
        if (tm != null) {
            status = tm.getTransaction(txAttr);
        }
    }
    //...
}
```
```java
//AbstractPlatformTransactionManager.class
public final TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
			throws TransactionException {
    //...
    try {
        return startTransaction(def, transaction, false, debugEnabled, suspendedResources);
    } catch (RuntimeException | Error ex) {
        resume(null, suspendedResources);
        throw ex;
    }
    //...
}
```
- 트랜잭션을 시작하기 위해 트랜잭션 매니저에서 `DataSource`를 가져옵니다.
- `DataSource`에서 새로운 커넥션을 생성합니다.
```java
//JpaTransactionManager.class
protected void doBegin(Object transaction, TransactionDefinition definition) {
    //...
    if (this.getDataSource() != null) {
        ConnectionHandle conHandle = this.getJpaDialect().getJdbcConnection(em, definition.isReadOnly());
    }
    /...
}
```
- 자동 커밋 모드를 해제합니다.
```java
//AbstractLogicalConnectionImplementor.class
public void begin() {
        try {
            if (!this.doConnectionsFromProviderHaveAutoCommitDisabled()) {
                log.trace("Preparing to begin transaction via JDBC Connection.setAutoCommit(false)");
                this.getConnectionForTransactionManagement().setAutoCommit(false);
                log.trace("Transaction begun via JDBC Connection.setAutoCommit(false)");
            }

            this.status = TransactionStatus.ACTIVE;
        } catch (SQLException var2) {
            SQLException e = var2;
            throw new TransactionException("JDBC begin transaction failed: ", e);
        }
    }
```
- 커넥션을 트랜잭션 동기화 매니저에 저장함으로써 전체 트랜잭션에서 동일한 커넥션을 재사용할 수 있도록 합니다.
```java
//JpaTransactionManager.class
protected void doBegin(Object transaction, TransactionDefinition definition) {
    if (this.getDataSource() != null) {
        ConnectionHandle conHandle = this.getJpaDialect().getJdbcConnection(em, definition.isReadOnly());
        if (conHandle != null) {
            ConnectionHolder conHolder = new ConnectionHolder(conHandle);
            if (timeoutToUse != -1) {
                conHolder.setTimeoutInSeconds(timeoutToUse);
            }

            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Exposing JPA transaction as JDBC [" + conHandle + "]");
            }
            //DataSource, 커넥션을 트랜잭션 동기화 매니저에 보관
            TransactionSynchronizationManager.bindResource(this.getDataSource(), conHolder);
            txObject.setConnectionHolder(conHolder);
        } else if (this.logger.isDebugEnabled()) {
            this.logger.debug("Not exposing JPA transaction [" + em + "] as JDBC transaction because JpaDialect [" + this.getJpaDialect() + "] does not support JDBC Connection retrieval");
        }
    }
}
```
#### 3. 비즈니스 로직 및 데이터 접근 로직 실행
- 트랜잭션이 시작된 후 비즈니스 로직을 실행합니다.
- 데이터 접근 로직을 수행할 때, 트랜잭션 동기화 매니저에 저장된 커넥션을 사용하여 작업을 진행합니다. 

#### 4. 트랜잭션 종료
- 모든 트랜잭션 작업이 종료 된 뒤 정상적으로 작업이 처리됐다면 변경사항을 데이터베이스에 커밋합니다.
```java
//AbstractLogicalConnectionImplementor.class
public void commit() {
        try {
            log.trace("Preparing to commit transaction via JDBC Connection.commit()");
            if (this.isPhysicallyConnected()) {
                this.getConnectionForTransactionManagement().commit();
            } else {
                this.errorIfClosed();
            }

            this.status = TransactionStatus.COMMITTED;
            log.trace("Transaction committed via JDBC Connection.commit()");
        } catch (SQLException var2) {
            SQLException e = var2;
            this.status = TransactionStatus.FAILED_COMMIT;
            throw new TransactionException("Unable to commit against JDBC Connection", e);
        }

        this.afterCompletion();
    }
```

지금까지 @Transactional의 작동 과정을 다음과 같이 그림으로 나타낼 수 있다.

![Transactional 전체 구조](./image/transactional_whole_structure.png)


## @Transactional 사용 시 주의점

### 1. public 메서드에만 적용

`@Transactional`은 Spring의 AOP 프록시 방식을 기반으로 구현되었습니다.

스프링에서 해당 방식은 기본적으로 public 메서드에만 트랜잭션을 적용하도록 설정되었습니다.

따라서 private 메서드에 `@Transactional`을 붙이더라도 오류가 발생하진 않지만 트랜잭션 적용은 무시됩니다.
추가로 스프링 3.0 부터는 protected, package-visible에도 `@Transactional`이 적용됩니다.

하지만 `@Transactional`을 붙인 public 메서드에서 내부적으로 호출하는 private 메서드는 같은 객체 내에서 실행되므로 트랜잭션이 전파됩니다.
따라서 같은 트랜잭션이 적용됩니다.


### 2. @Transactional 자기 호출 시 미적용
```java
@Service
public class AService {

    public void func() {
        doSomething();
    }
    
    @Transactional
    public void doSomething() {
        // do something
    }
}
```
위 코드에서는 func()안에 `@Transactional`이 적용된 doSomething()이 존재합니다.
과연 이 코드는 의도대로 동작할까요?

정답은 그렇지 않다입니다.

앞서 `@Transactional`은 비즈니스 로직이 수행되기 전, 프록시가 먼저 호출되어 트랜잭션 작업을 수행한다고 말씀드렸습니다.
하지만 func()은 doSomething()의 프록시를 호출하는 것이 아닌 메서드를 바로 호출하고 있기 때문에 트랜잭션이 수행되지 않습니다.

이 문제를 가장 쉽게 해결하는 방법은 doSomething()을 별도의 클래스로 분리하여 해당 클래스를 주입받아 호출하는 것입니다.

```java
@Service
public class AService {

    private final BService bService;

    public AService(BService bService) {
        this.bService = bService;
    }
    
    public void func() {
        bService.doSomething();
    }
    
}
```
```java
@Service
public class BService {
    
    @Transactional
    public void doSomething() {
        // do something
    }
}
```

스프링에서는 AOP를 적용한 경우 대상 객체 대신 프록시를 빈으로 등록합니다.
따라서 doSomething()을 외부 객체로 분리하여 호출하는 경우에는 항상 프록시가 대신 호출되어 트랜잭션이 정상적으로 작동합니다.