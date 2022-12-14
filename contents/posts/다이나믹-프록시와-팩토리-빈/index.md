---
title: "다이나믹 프록시와 팩토리 빈"
description: 
date: 2022-10-15
update: 2022-10-15
tags:
- AOP

series: "Spring AOP 알아보기"

---

## 들어가기전..

Spring AOP는 스프링의 3대 축이라고 불리기도 하지만 그만큼 내용의 깊이가 깊고 어려운 항목이기도 합니다. 이로 인해서, Spring AOP를 계속 사용하고 있음에도 불구하고 개발자는 인지하지 못하기도 합니다. 이번 시리즈에서는 Spring AOP를 조금씩 알아보는 시간을 가져보려고 합니다.

## 프록시 패턴과 데코레이터 패턴의 차이..?

이 두가지 패턴의 차이점을 알기 전에 프록시의 개념에 대해서 간단하게 살펴보겠습니다.

프록시는 클라이언트라 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 대리자, 대리인과 같은 역할을 한다고 해서 **프록시**라고 부릅니다. 그리고 이러한 프록시를 통해 최종적으로 요청을 처리하는 실제 오브젝트를 **타깃**이라고 부릅니다. 다음 그림은 일반적인 **클라이언트 - 프록시 - 타깃의 구조**입니다.

![](img.png)

이러한 프록시는 사용 목적에 따라서 두 가지로 구분할 수 있습니다. 첫째는 클라이언트가 타깃에 접근하는 방법을 제어하기 위해서 사용될 수 있습니다. 두번째는 타깃에 부가적인 기능을 부여해주기 위해서 사용될 수 있습니다. 무엇이 프록시 패턴이고 데코레이터 패턴일까요?? 하나씩 살펴보도록 하겠습니다.

### 데코레이터 패턴

데코레이터 패턴은 타깃에 부가적인 기능을 런타임 시 다이내믹하게 부여해주기 위해 프록시를 사용하는 패턴을 말합니다. 데코레이터 패턴은 이름에서도 의미를 유추할 수 있듯이 **하나의 타깃을 여러 겹으로 포장하고 장식을 붙이는데 사용**이 됩니다. 즉, **프록시가 한 개 이상이 될 수 있음을 의미**합니다.

이러한 **데코레이터는 위임하는 대상에게도 인터페이스로 접근을 하기 때문에, 자신이 최종 타깃으로 위임을 하는지 또 다른 데코레이터 프록시로 위임을 하는지 알 수 없습니다**. 즉, 외부에서 위임을 할 대상을 런타임 시에 주입받을 수 있도록 만들어야 합니다.

![](img_1.png)

이러한 데코레이터 패턴은 타깃의 코드를 손대지 않고, 클라이언트가 호출하는 방법도 변경하지 않은 채로 새로운 기능을 추가할 때 유용한 방법이라고 할 수 있습니다.

### 프록시 패턴

프록시 패턴은 타깃의 기능을 확장하거나 추가하지 않습니다. **대신 클라이언트가 타깃에 접근하는 방식을 변경**해줍니다. 예를 들면, 타깃 오브젝트를 생성하기가 복잡하거나 당장 필요하지 않은 경우, 필요한 시점까지 오브젝트를 생성하지 않는 것이 좋습니다. 하지만 **타깃 오브젝트에 대한 레퍼런스가 미리 필요할 때가 있습니다**. 이럴 때 프록시 패턴을 사용할 수 있습니다.

프록시 패턴은 클라이언트에게 **실제 타깃에 대한 레퍼런스를 넘기는 것이 아닌, 프록시를 넘겨주는 것**을 의미합니다. 그리고 **프록시의 메서드를 통해 타깃을 사용하려고 시도하면, 그때 타깃 오브젝트를 생성해서 요청을 위임하여 객체의 생성을 최대한 늦춥**니다.

![](img_2.png)

이러한 프록시 패턴은 객체의 생성을 요청시점까지 최대한 늦춤으로써 끝까지 사용하지 않거나, 많은 작업이 진행된 후에 사용되는 비효율적인 생성을 보다 효율적으로 사용할 수 있도록 하는 유용한 방법이라고 할 수 있습니다.

## 다이내믹 프록시

프록시는 기존 코드에 영향을 주지 않으면서 타깃의 기능을 확장하거나 접근 방법을 제어할 수 있는 유용한 방법입니다. 하지만 해당방법은 상당히 귀찮은 작업이라고도 할 수 있습니다. **매번 새로운 클래스를 정의해야하고, 인터페이스 처리된 메서드들에 대해서 위임하는 작업들을 전부 해줘야 하기 때문**입니다.

이렇게 불편한 작업들을 우리는 `java.lang.reflect`에 있는 패키지에서 프록시를 손쉽게 만들 수 있도록 기능을 제공하고 있습니다.

### 프록시의 구성과 프록시 작성의 문제점

프록시는 다음의 두 가지 기능으로 구성됩니다.

- 타깃과 같은 메서드를 구현하고 있다가 메소드가 호출되면 타킷 오브젝토로 위임한다.
- 지정된 요청에 대해서는 부가기능을 수행한다.

이렇게 프록시를 만들게 되면 다음과 같이 **위에서 말한 귀찮은 작업**들을 매번 해줘야 합니다.

- 타겟의 인터페이스를 구현하고 위임하는 코드를 작성하기가 번거롭다. 부가기능이 필요없는 메서드도 이를 적용해야 한다. 또한, 타겟의 인터페이스 메서드가 추가되거나 변경될 때마다 함께 수정해줘야 한다.
- 부가기능 코드가 중복될 가능성이 많다. 트랜잭션은 DB를 사용하는 대부분의 로직에 적용될 필요가 있다. 예를들어 add()에 부가 기능을 추가해주고, update()에도 부가 기능을 추가해줘야 한다면 트랜잭션 기능을 제공하는 유사한 코드가 여러 매서드에 중복되서 나타날 것이다.

이러한 문제를 어떻게 해결해야 할까요? 바로 **JDK 다이내믹 프록시**를 통해서 해결할 수 있습니다.

### 리플랙션

다이내믹 프록시는 리플렉션 기능을 이용해서 만들 수 있습니다. 리플랙션을 간단히 설명하자면 자바의 코드 자체를 추상화에서 접근하도록 만든 것이라고 할 수 있습니다.

자바의 모든 클래스는 그 클래스 자체의 구성정보를 담은 `Class` 타입의 오브젝트를 하나씩 갖고 있습니다. 이러한 `Class` 타입의 오브젝트를 이용하면 클래스 코드에 대한 메타정보를 가져오거나 오브젝트를 조작할 수 있습니다.

조금 자세하게 예를 들어서 설명하면 `String.class`가 가진 `length()`라는 메서드의 정보를 가져오기 위해서는 다음과 같이 메서드 정보를 가져올 수 있습니다.

```java
Method lengthMethod = String.class.getMethod("length");
```

그리고 이렇게 가져온 메서드의 정보를 가지고 실행도 시킬 수 있습니다.

```java
int length = lengthMethod.invoke(name);
```

리플랙션 사용까지 알았다면 다이내믹 프록시를 적용하는데 큰 어려움이 없을 것 입니다.

### 다이내믹 프록시를 적용해보자

다이내믹 프록시는 **프록시 팩토리에 의해 런타임 시 다이내믹하게 만들어지는 오브젝트**를 말합니다. 다이내믹 프록시 오브젝트는 타깃의 인터페이스와 같은 타입으로 만들어집니다. 이 덕분에 **프록시를 만들 때, 인터페이스를 모두 구현해가면서 클래스를 정의하지 않아도 됩니다**.

여기서 개발자는 다이내믹 프록시를 통해서 만들어주는 오브젝트를 기반으로, 제공하려는 부가기능 제공 코드만 작성을 하면 됩니다. 이런 부가기능 제공 코드는 `InvocationHandler`를 구현한 오브젝트에 담으면 됩니다.

```java
public Object invoke(Object proxy, Method method, Object[] args)
```

`InvocationHandler`의 유일한 메서드인 `invoke()` 메서드는 리플레션의 `Method`와 메서드를 호출할 때 전달되는 파라미터도 `args`로 받습니다. 즉, 다이내믹 프록시 오브젝트는 **클라이언트의 모든 요청을 리플렉션 정보로 변환해서 `InvocationHandler` 를 구현한 오브젝트의 `invoke()` 메서드로 넘긴다**고 할 수 있습니다.

코드를 통해서 이를 한번 확인해보겠습니다.

```java
public class TxHandler implements InvocationHandler {

    private Object target;
    private PlatformTransactionManager transactionManager;

    public TxHandler(Object target, PlatformTransactionManager transactionManager) {
        this.target = target;
        this.transactionManager = transactionManager;
    }

    @Override
    public Object invoke(final Object proxy, final Method method, final Object[] args) throws Throwable {
        TransactionStatus transactionStatus = transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            Object result = method.invoke(target, args);
            transactionManager.commit(transactionStatus);
            return result;
        } catch (InvocationTargetException e) {
            transactionManager.rollback(transactionStatus);
            throw e.getTargetException();
        }
    }
}
```

중요한 내용이라서 흐름을 한번 더 정리해보겠습니다.

1. 다이내믹 프록시로부터 요청을 전달받으려면 `InvocationHandler`를 구현해야 합니다. 즉, 다이내믹 프록시가 클라이언트로부터 받는 모든 요청은 `invoke()` 메서드로 전달이 됩니다.
2. 이렇게 전달이 되면 **리플렉션 API**를 이용해서 **타깃 오브젝트**의 메서드를 호출합니다.
3. 이후 지정한 부가기능을 수행하고, 결과를 리턴하면 해당 값을 **다이내믹 프록시**가 받아서 최종적으로 클라이언트에게 전달합니다.

이러한 프록시는 다음과 같이 생성할 수 있습니다.

```java
final var txHandler= new TxHandler(target, transactionManager);
final var userService = (UserService) Proxy.newProxyInstance(
     getClass().getClassLoader(),
	   new Class[]{UserService.class},
     txHandler);
```

이제 우리는 **다이내믹 프록시**를 이용해서 조금 더 깔끔하게 `Transactional` 을 적용할 수 있게 되었습니다.

해당 내용을 구현한 코드는 다음 링크에서 확인할 수 있습니다. → [링크](https://github.com/woowacourse/jwp-dashboard-jdbc/pull/177/commits/76e7dc9bfb17be6275378bbd7039a29430dd6242)

## 팩토리 빈

위의 내용들을 통해서 조금 더 깔끔하게 `Transactional`을 사용할 수 있는 코드를 만들었습니다. 그렇다면 이를 **스프링의 DI** 통해 사용할 수 있도록 만들어 볼 수 있을 것 같습니다. 하지만 **DI의 대상이 되는 다이내믹 프록시 오브젝트는 일반적인 스프링 빈으로 등록이 불가능**합니다.

스프링의 빈은 기본적으로 클래스 이름과 프로퍼티로 정의됩니다. 이를 기반으로 스프링은 지정된 클래스 이름을 가지고 리플렉션을 통해서 해당 클래스의 오브젝트를 만듭니다.

```java
Date now = (Date) Class.forName("java.util.Date").newInstance();
```

여기서 문제는 **다이내믹 프록시는 이런 식으로 프록시 오브젝트가 생성되지 않는다는 점**입니다. 그렇다면 방법이 없는 것일까요?? 스프링은 무언가 다른 방법을 제시해줄 것 같습니다.

### 팩토리 빈

팩토리 빈은 클래스 정보를 기반으로 디폴트 생성자를 통해 오브젝트를 만드는 방법 외에 특별하게 빈을 만들 수 있는 방법 중 하나입니다. 여러가지 방법이 있지만 가장 간단한 방법은 스프링의 `FactoryBean` 인터페이스를 구현하는 것 입니다.

```java
public interface FactoryBean<T> {
    T getObject() throws Exception;     // 빈 오브젝트를 생성해서 돌려준다. 
    Class<? extends T> getObjectType(); // 생성되는 오브젝트의 타입을 알려준다.
    boolean isSingleton();              // getObject()가 돌려주는 오브젝트가 항상 같은 싱글톤 오브젝트인지 알려준다.
}
```

스프링은 `FactoryBean` 인터페이스를 구현한 클래스가 빈의 클래스로 지정되면, `FactoryBean` 클래스의 오브젝트의 `getObject()`를 이용해 오브젝트를 가져오고, 이를 빈 오브젝트로 사용합니다.

### 다이내믹 프록시를 팩토리 빈으로 만들어보자

이러한 팩토리 빈을 통해서 다이내믹 프록시를 만들 수 있습니다. 팩토리 빈의 `getObject()` 메서드에 다이내믹 오브젝트를 만들어주는 코드를 작성하면 되기 때문입니다.

코드를 통해서 이를 확인해보겠습니다.

```java
public class TxProxyFactoryBean implements FactoryBean<Object> { // 범용적으로 사용하기 위해 Object로 지정했다.
    
    Object target;
    PlatformTransactionManager transactionManager;
    String pattern;
    Class<?> serviceInterface; // 다이내믹 프록시를 생성할 때 필요하다.

    public Object getObject() throws Exception {
        TxHandler txHandler = new TxHandler();
        txHandler.setTarget(target);
        txHandler.setTransactionManager(transactionManager);
        txHandler.setPattern(pattern);

        return Proxy.newProxyInstance(getClass().getClassLoader(), new Class[] { serviceInterface }, txHandler);
    }

    public Class<?> getObjectType() {
        return serviceInterface; // 팩토리 빈이 생성하는 오브젝트의 타입은 DI 받은 인터페이스 타입에 따라 달라진다. 따라서 다양한 타입의 프록시 오브젝트 생성에 재사용할 수 있다.
    }

    public boolean isSingleTon() {
        return false; // 싱글톤 빈이 아니라는 뜻이 아니라 getObject()가 매번 같은 오브젝트를 리턴하지 않는다는 뜻이다.
    }

    // setter ..
}
```

위의 코드를 보면 알 수 있듯이 팩토리 빈이 만드는 다이내믹 프록시는 구현 인터페이스나, 타깃의 종류에 제한이 없습니다. 따라서 **DI를 통해 TxHandler를 이용하는 다이내믹 프록시를 재사용해서 사용**할 수 있습니다.

### 팩토리 빈의 한계

`TransactionHandler`를 이용하는 다이내믹 프록시를 생성해주는 `TxProxyFactoryBean`은 코드의 수정 없이도 다양한 클래스에 적용이 가능합니다. 타깃 오브젝트에 맞는 프로퍼티 정보만 설정해서 빈으로 등록해주면 되기 때문입니다.

하지만 이러한 방법 역시 명확한 한계점을 가지고 있습니다. 바로 한 번에 여러 클래스에 공통적인 부가기능을 제공하는 것 입니다. 지금과 같은 방법은 **적용 대상인 클래스가 N개라면 그만큼의 프록시 패턴의 빈의 설정이 중복**되는 구조를 가지게 됩니다.

또한 **`Handler` 오브젝트가 프록시 팩토리 빈 갯수만큼 만들어진다는 점입니다.** 이는 타깃 오브젝트르 프로퍼티로 가지고 있음으로 생기는 문제입니다.

이러한 문제들을 해결할 수 있는 방법은 없을까요?? 이를 다음글에서 한번 살펴보도록 합시다.

## 글을 마무리하며..

이번 글을 정리하면서 Proxy에 대해서 다시 한번 정리할 수 있는 시간이였습니다. 매번 해당 내용이 추상적으로 다가왔는데 이번에는 조금 이해가 잘되었던 것 같습니다. 아마 진행하고 있는 테크코스 과제와 스터디가 큰 영향을 미친거 같기도 합니다. 이번 6장을 끝으로 토비의 스프링 스터디를 마무리 할 것 같은데 유의미하게 마무리할 수 있었으면 좋겠습니다.

<br>

> 참고한 글
>
- 토비의 스프링 3.1
- [https://github.com/woowacourse/jwp-dashboard-jdbc](https://github.com/woowacourse/jwp-dashboard-jdbc)
- [https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/InvocationHandler.html](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/InvocationHandler.html)