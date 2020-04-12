# 스프링 빈 스코프(Bean Scopes)
작성자: 전동훈
작성일: 2020.04

## 빈 스코프의 종류

스코프(SCOPE) | 설명
-|-
싱글톤(Singleton) | (기본값) 싱글톤 스코프로 정의된 빈은 스프링 IoC 컨테이너별로 단 한번만 생성(인스턴스)된다.
프로토타입(Prototype) | 프로토타입 스포로 정의된 빈은 빈을 요청할 때마다 생성된다.
리퀘스트(Request) | HTTP request의 생명주기와 동일한 생명주기를 가진다. 즉, 각 HTTP request별로 고유한 빈을 가진다. 이 값은 웹을 인식하는 스프링의 `ApplicationContext`에서만 유효하다.
세션(Session) | HTTP `Session`과 동일한 생명주기를 가진다. 이 값은 웹을 인식하는 스프링의 `ApplicationContext`에서만 유효하다.
애플리케이션(Application) | `ServletContext`와 동일한 생명주기를 가진다. 이 값은 웹을 인식하는 스프링의 `ApplicationContext`에서만 유효하다.
웹소켓(Websocket) | `WebSocket`과 동일한 생명주기를 가진다. 이 값은 웹을 인식하는 스프링의 `ApplicationContext`에서만 유효하다.

## 싱글톤 스코프
오직 하나의 공유 인스턴스로 싱글톤 빈을 관리하고, 스프링 컨테이너는 모든 빈의 정의와 일치하는 요청의 결과로 하나의 빈 인스턴스를 반환한다. 

다시말하면, 싱글톤 스코프로 빈을 정의했을 때, 스프링 IoC 컨테이너는 빈 정의대로 정확히 한번 객체를 인스턴스화 한다. 이 인스턴스는 이러한 싱글톤 빈들의 캐시에 저장되고 같은 빈을 요청하면 캐싱된 객체를 돌려준다. 

스프링의 싱글톤 빈의 개념은 GoF의 싱글톤 패턴과는 개념이 다르다. 싱글톤 패턴은 클래스로더당 한번만 생성되도록 객체의 주기를 하드코딩한 것이다. 스프링의 싱글톤 스코프는 하나의 컨테이너에 하나의 빈으로 설명할 수 있다. 이것의 의미는 하나의 스프링 컨테이너는 싱글톤 스코프로 정의된 빈의 인스턴스를 오직 하나만 가질 수 있다는 것이다.

## 프로토타입 스코프
프로토타입 스코프로 정의된 빈은 해당 빈의 요청이 있을 때마다 새로운 객체를 생성하여 반환한다. 즉, 이 빈은 다른 빈이나 컨테이너에서 `getBean()` 메소드의 호출을 통해 주입이 가능하다. 일반적으로 상태를 가지는 빈은 프로토타입 스코프로 정의하고 상태를 가지지 않는 빈은 싱글톤 빈으로 정의한다.

프로토타입 빈은 스프링의 다른 빈들과 달리 생명주기를 컨테이너에서 관리하지 않는다. 컨테이너는 프로토타입 빈을 생성과 초기화를 담당하지만 객체를 클라이언트에게 주고 나면은 컨테이너는 더이상 그 객체 정보를 가지지 않는다. 따라서 스코프에 관계없이 모든 객체의 초기화 콜백 메소드가 호출되지만, 프로토타입의 경우 설정된 소멸 콜백 메소드는 호출되지 않는다. 클라이언트의 코드는 반드시 프로토타입 객체를 정리해야하고 점유한 리소스들을 해제해야 한다. 스프링 컨테이너가 프로토타입 빈이 점유한 리소스들을 해제하도록 하기 위해 정리가 필요한 빈들의 참조를 가지고 있는 커스텀 빈 후처리기를 사용할 수 있다.

어떤점에서는 프로토타입 빈에 대한 스프링 컨테이너의 역할은 자바의 `new` 키워드를 대신하는 것일 수 있다. 해당 시점 이후의 모든 생명주기 관리는 클라이언트가 해야한다.

### 싱글톤 빈과 프로토타입 빈의 의존성
싱글톤 빈을 프로토타입 빈에 주입할 때, 해당 의존관계는 객체 생성시에 이루어진다는 것을 알아야한다. 따라서, 프로토타입 빈을 싱글톤 빈으로 주입한다면 새로운 프로토타입 빈이 생성되고 싱글톤 빈에 주입된다. 생성된 프로토타입 객체는 싱글톤 빈에 제공되는 유일한 객체이다.

>   싱글톤 빈은 한번만 생성되기 때문에 프로토타입 빈을 싱글톤 빈에 주입할 경우 해당 싱글톤 빈 안에서 하나의 프로토타입 객체만 생성된다.

그런데 런타임시 싱글톤 빈내의 프로토타입 빈을 계속해서 새로운 객체로 생성하고 싶을 수 있다. 하지만 이런 경우 스프링 컨테이너는 싱글톤 빈을 한번만 생성 및 의존성을 주입하는 초기화 작업을 하기 때문에 프로토타입 빈을 싱글톤 빈으로 주입할 수 없다. 만약 런타임시에 하나 이상의 새로운 프로토타입 빈의 객체가 필요하다면 **메소드 주입**을 참고해라.

## 리퀘스트, 세션, 애플리케이션 그리고 웹소켓 스코프
`request`, `session`, `application`, `websocket` 스코프는 오직 웹을 알고있는 (웹을 사용하는) 스프링 `ApplicationContext` (`XmlWebApplicationContext` 와 같은)에서만 유효한 값이다. `ClassPathXmlApplication`과 같은 기본적인 스프링 컨테이너에서 이 스코프들을 사용한다면 알 수 없는 빈 스코프라는 `IllegalStateException` 예외를 던질것이다.

### 웹 설정 초기화
리퀘스트, 세션, 애플리케이션, 웹소켓 스코프(web-scoped beans)들을 사용하기 위해서는 빈을 정의하기 전에 몇가지 설정이 필요하다.

>   이 작업은 기본 스코프인 싱글톤과 프로토타입 빈을 위해서는 필요하지 않다.

이 초기 설정 작업은 서블릿에 따라 다르다.

스프링 Web MVC 내에서 정의된 빈(사실상 스프링의 `DispatcherServlet`이 처리하는 요청 내에서)에 접근하는 경우는 특별한 설정이 필요하지 않다. `DispatcherServlet`이 이미 모든 관련 상태를 노출하고 있다.

만약 스프링의 `DispatcherServlet` 밖에서 요청을 처리하는 서블릿 2.5의 웹 컨테이너를 사용한다면 `org.springframework.web.context.request.RequestContextListener`, `ServletRequestListener`를 등록해야 한다. 서블릿 3.0 이상부터는 이러한 작업을 프로그램적으로 구현된 `WebApplicationInitializer` 인터페이스를 이용해 설정할 수 있다. 또는 더 이전 버전의 컨테이너의 경우 다음 선언을 웹 애플리케이션의 `web.xml` 파일에 추가하면 된다.

```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextlistener
        </listener-class>
    </listener>
    ...
</web-app>
```

만약 리스너를 설정하는 것과 관련한 이슈가 있다면 스프링의 `RequestContextFilter` 사용을 고려하자. 필터 매핑은 주변 웹 애플리케이션 환경에 영향을 받기 때문에 적절히 환경을 변경해주어야 한다. 다음은 웹 애플리케이션의 일부인 필터 설정을 보여준다.

```xml
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

`DispatcherServlet`, `RequestContextListener`, `RequestContextFilter` 는 모두 요청에 대한 서비스를 제공하기 위한 `Thread`에 HTTP request 객체를 바인딩하는 정확히 같은 작업을 한다. 이를 통해 요청 및 세션 스코프의 빈을 여러 호출을 통해 추가로 사용할 수 있다.

### 리퀘스트 스코프
스프링 컨테이너는 매 HTTP 요청마다 리퀘스트 스코프인 빈의 새로운 인스턴스를 생성한다. 이말은 생성한 인스턴스가 리퀘스트 스코프를 가지는 것을 의미한다. 같은 빈의 정의로 생성된 다른 인스턴스들은 현재 상태의 변화를 알지 못하기 때문에 생성된 인스턴스의 내부 상태를 원하는 만큼 변경해도 괜찮다. 이 인스턴스들은 개별 요청에 따라 서로 다르다. 요청에 대한 처리가 끝난다면 리퀘스트 스코프인 빈은 버려질것이다. 

어노테이션 설정 또는 자바 코드 설정을 할 경우 `@RequestScope` 어노테이션을 컴포넌트(또는 빈)에 부여하여 리퀘스트 스코프를 설정할 수 있다. 다음은 그 예시이다.

```java
@RequestScope 또는 @Scope("request")
@Component
public class LoginAction{
    // ...
}
```


### 세션 스코프
세션 스코프의 빈은 스프링 컨테이너가 하나의 HTTP Session의 생명주기동안 빈 정의를 사용하여 빈을 새로 생성한다. 리퀘스트 스코프와 마찬가지로 생성된 빈의 내부 상태를 원하는대로 변경해도 괜찮다. HTTP Session 인스턴스가 소멸하면 해당 스코프로 설정된 빈들도 같이 소멸된다.

어노테이션 및 자바 설정으로 스코프를 설정하는 방법은 다음과 같다.

```java
@SessionScope 또는 @Scope("session")
@Component
public class UserPreferences {
    // ...
}
```

### 애플리케이션 스코프
스프링 컨테이너는 웹 애플리케이션이 전체 생명주기동안 한번 빈의 인스턴스를 생성한다. 즉, 애플리케이션 스코프 빈은 서블릿 컨텍스트 레벨이며 일반적인 서블릿 컨텍스트의 속성으로 저장된다. 이는 스프링의 싱글톤 빈과 유사하지만 매우 중요한 두가지 차이점이 있다.
1.  스프링의 애플리케이션 컨텍스트가 아니라 하나의 서블릿 컨텍스트안에서는 싱글톤이다.
2.  애플리케이션 스코프 빈은 노출되어 있어있기 때문에 서블릿 컨텍스트의 속성으로 접근할 수 있다.

어노테이션 및 자바 설정으로 애플리케이션 스코프를 설정하는 방법은 다음과 같다.

```java
@ApplicationScope 또는 @Scope("application")
@Component
public class AppPreferences {
    // ...
}
```
### 스코프가 있는 빈들의 의존성
스프링 IoC 컨테이너는 빈들의 인스턴스화 뿐만 아니라 그들의 의존성 주입도 관리한다. 리퀘스트 스코프 빈을 더 긴 생명주기를 가진 빈에 주입하고싶다면 AOP proxy를 주입할 수도 있다. 스코프가 있는 객체의 같은 퍼블릭 인터페이스로 접근할 수 있지만 관련된 스코프로부터 실제 대상 객체를 검색하고 실제 객체의 메소드 호출을 위임하는 프록시 객체를 주입해야 한다.

> 	You may also use `<aop:scoped-proxy/>` between beans that are scoped as singleton, with the reference then going through an intermediate proxy that is serializable and therefore able to re-obtain the target singleton bean on deserialization.
>
>   When declaring `<aop:scoped-proxy/>` against a bean of scope `prototype`, every method call on the shared proxy leads to the creation of a new target instance to which the call is then being forwarded.
>
>   Also, scoped proxies are not the only way to access beans from shorter scopes in a lifecycle-safe fashion. You may also declare your injection point (that is, the constructor or setter argument or autowired field) as `ObjectFactory<MyTargetBean>`, allowing for a `getObject()` call to retrieve the current instance on demand every time it is needed — without holding on to the instance or storing it separately.
>
>   As an extended variant, you may declare `ObjectProvider<MyTargetBean>`, which delivers several additional access variants, including `getIfAvailable` and `getIfUnique`.
>
>   The JSR-330 variant of this is called `Provider` and is used with a `Provider<MyTargetBean>` declaration and a corresponding `get()` call for every retrieval attempt. See here for more details on JSR-330 overall.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/> 
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```

프록시를 생성하기 위해 빈 정의에서 `<aop:scoped-proxy/>` 자식 엘레먼트를 추가해야한다. 왜 리퀘스트, 세션 또는 커스텀 스코프로 정의된 빈들은 `<aop:scoped-proxy/>가 필요할까? 다음 싱글톤 빈 정의와 위에서 정의된 것과 비교해 보자.

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

위 예제의 `userManager`에는 세션 스코프인 `userPreferences`에 대한 참조가 주입된다. 여기서 중요한 점은 `userManager`가 싱글톤 빈이라는 점이다. 이는 하나의 컨테이너에서 정확히 한번 객체가 생성되며 의존성 주입또한 한번 일어난다는 것이다. 때문에 `userManager`는 동일한 `userPreferences` 객체를 가지고 동작한다. 이는 우리가 넓은 스코프를 가진 빈에 더 짧은 스코프의 빈을 주입할 때 원하는 동작 방식이 아니다. 이렇다면 차라리 `userManager`를 세션 스코프로 설정하는게 낫다. 이러한 문제로 인해서 `userManager` 객체가 생성될 때 실제 `userPreferences` 객체를 주입하는 것이 아니라 프록시 객체를 생성하여 준다. 이후 `userManager`에서 `userPreferences` 객체를 사용할 때 프록시는 적절한 인스턴스를 검색하여 해당 인스턴스에 동작을 위임하여 처리한다. 

이같은 이유때문에 리퀘스트 및 세션 스코프의 빈을 다른 빈에 주입할 때 다음과 같이 설정할 필요가 있다.

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

### 생성을 위한 프록시 타입의 선택
기본적으로 스프링 컨테이너는 `<aop:scoped-proxy/>` 요소를 가진 빈에 대해 CGLIB 기반의 프록시 객체를 생성한다.

>   CGLIB 프록시는 오직 퍼블릭 메소드 호출에 대해서만 인터셉트하여 동작한다. 퍼블릭이 아닌 메소드는 프록시의 실제 대상 객체로의 위임이 일어나지 않는다.

대신에 `<aop:scoped-proxy/>`의 속성으로 `proxy-target-class`의 값을 `false`로 주어 표준 JDK 인터페이스 기반의 프록시를 생성하도록 설정할 수 있다. 표준 JDK 인터페이스 기반의 프록시를 사용하는 것은 추가적인 라이브러리가 필요없다. 하지만 이 방법을 사용할 경우 스코프를 가진 빈은 적어도 하나이상의 인터페이스를 구현해야하며 해당 빈을 주입받는 모든 빈들이 인터페이스를 통해 접근해야한다. 다음은 프록시 기반 설정 인터페이스이다.

```xml
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.stuff.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.stuff.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

더 자세한 정보는 프로시 메커니즘을 공부하자.

## 커스텀 스코프
스프링의 빈 스코프는 확장이 가능하다. 우리는 자신만의 스코프를 정의하거나 기존의 스코프를 재정의또한 가능하다. 후자의 경우 권장되지 않는다. 그리고 내장 싱글톤과 프로토타입 스코프는 변경할 수 없다. (리퀘스트, 세션, 애플리케이션은 가능)

### 커스텀 스코프 만들기
스프링 컨테이너에 커스텀 스코프를 등록하려면 `org.springframework.beans.factory.config.Scope` 인터페이스를 구현해야한다. 자신만의 스코프를 어떻게 구현하는지 알아보기 위해 스프링에서 제공하는 `Scope` 자바독 문서를 참조하자. 

`Scope` 인터페이스는 객체를 가져오고, 객체를 제거하고 객체를 소멸시키는 4개의 메소드를 가지고 있다.


#### get
```java
Object get(String name, ObjectFactory<?> objectFactory)
```
세션 스코프는 세션 스코프의 빈을 리턴하며 해당 세션에 객체가 없다면 새로은 객체를 생성하고 이후에 참조하기 위해 생성한 객체 참조를 세션에 저장한다.

#### remove
```java
Object remove(String name)
```
세션 스코프의 경우 세션에 저장된 세션 스코프 빈 객체를 제거한다. 그 객체는 반드시 리턴되어야 하지만 해당 이름의 빈을 찾을 수 없는 경우 널을 리턴할 수 있다. 

#### registerDestructionCallback
```java
void registerDestructionCallback(String name, Runnable destructionCallback)
```
이 메소드는 스코프에 객체 소멸 메소드가 실행되거나 객체가 소멸할 때 반드시 실행되어야 하는 콜백을 등록한다.

#### getConversationId
```java
String getConversationId()
```
이 메소드는 스코프 내에서 사용되는 식별자를 반환한다.

식별자는 각 스코프 별로 다르며 세션 스코프의 경우 세션 식별자일 수 있다.

### 커스텀 스코프의 사용
커스텀 `Scope`를 구현했다면 스프링 컨테이너가 새로운 스코프를 알 수 있도록 조치해야한다. 새 스코프를 스프링 컨테이너에 등록하기 위해 `registerScope` 메소드를 이용한다. 

```java
void registerScope(String scopeName, Scope scope);
```

이 메소드는 스프링에서 제공되는 대부분의 `ApplicationContext`의 구현 클래스의 `BeanFactory`속성을 통해 사용할 수 있는  `ConfigurableBeanFactory` 인터페이스에 선언되어 있다.

첫번째 인자로 스코프와 관련된 고유한 이름을 주어야한다. 예를들어, 스프링 컨테이너는 `singleton`과 `prototype`을 사용한다. 두번째 인자로는 실제 `Scope` 인터페이스를 구현한 인스턴스를 주어야 한다.

다음 예제를 통해 스코프 등록을 확인해보자.

>   이 예제는 `SimpleThreadScope`를 사용한다. 이 스코프 클래스는 스프링에서 제공하지만 기본값으로 컨테이너에 등록되어 있지 않는 스코프이다. 

```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

위 코드를 실행하면 이제 `thread` 스코프로 빈을 정의할 수 있다.

스프링은 프로그램적으로 스코프를 등록하도록 제한하지 않는다. 위 방법 이외에 `CustomScopeCongifurer` 클래스를 사용해 스코프를 선언적으로 등록할 수 있다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="thing2" class="x.y.Thing2" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="thing1" class="x.y.Thing1">
        <property name="thing2" ref="thing2"/>
    </bean>

</beans>
```

>   `<aop:scoped-proxy/>`를 `FactoryBean`구현에 두면 `getObject()`의 결과가 아니라 팩토리빈 그자체를 빈으로 등록하게 된다.