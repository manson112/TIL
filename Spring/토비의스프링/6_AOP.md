# 6. AOP

# 6.2.3 단위 테스트와 통합 테스트

- 단위테스트
    
    테스트 대상 클래스를 목 오브젝트 등의 테스트 대역(Test Double)을 이용해 의존 오브젝트나 외부의 리소스를 사용하지 않도록 고립시켜서 하는 테스트
    
- 통합테스트
    
    두 개 이상의, 성격이나 계츠이 다른 오브젝트가 연동하도록 만들어 테스트하거나, 또는 외부의 DB나 파일, 서비스 등의 리소스가 참여하는 테스트
    
    스프링 테스트 컨텍스트 프레임워크를 이용해서 컨텍스트에서 생성되고 DI된 오브젝트를 테스트하는 것도 통합 테스트이다.
    

## 통합 테스트와 단위 테스트 중 어떤 것을 쓸지 결정하는 가이드라인

- 항상 단위 테스트를 먼저 고려한다
- 하나의 클래스나 성격과 목적이 같은 긴밀한 클래스 몇 개를 모아서 외부와의 의존관계를 모두 차단하고 필요에 따라 스텁이나 목 오브젝트 등의 테스트 대역을 이용하도록 테스트를 만든다. 단위 테스트는 테스트 작성도 간단하고 실행 속도도 빠르며 테스트 대상 외의 코드나 환경으로부터 테스트 결과에 영향을 받지도 않기 때문에 가장 빠른 시간에 효과적인 테스트를 작성하기에 유리하다.
- 외부 리소스를 사용해야만 가능한 테스트는 통합 테스트로 만든다.
- DAO와 같이 단위 테스트로 만들기 어려운 코드는 DB까지 연동하는 테스트로 만드는 편이 효과적이다. DB를 사용하는 테스트는 DB에 테스트 데이터를 준비하고, DB에 직접 확인을 하는 등의 부가적인 작업이 필요하다.
- DAO를 테스트를 통해 충분히 검증해두면, DAO를 이용하는 코드는 DAO 역할을 스텁이나 목 오브젝트로 대체해서 테스트할 수 있다.
- 여러 개의 단위가 의존관계를 가지고 동작할 때를 위한 통합 테스트는 필요하다. 다만, 단위 테스트를 충분히 거쳤다면 통합 테스트의 부담은 상대적으로 줄어든다.
- 단위 테스트를 만들기가 너무 복잡하다고 판단되는 코드는 처음부터 통합 테스트를 고려해본다. 이때도 통합 테스트에 참여하는 코드 중에서 가능한 한 많은 부분을 미리 단위 테스트로 검증해두는 게 유리하다.
- 스프링 테스트 컨텍스트 프레임워크를 이용하는 테스트는 통합 테스트다. 가능하면 스프링의 지원 없이 직접 코드 레벨의 DI를 사용하면서 단위 테스트를 하는 게 좋겟지만 스프링의 설정 자체도 테스트 대상이고, 스프링을 이용해 좀 더 추상적인 레벨에서 테스트해야 할 경우도 종종 있다.

코드를 작성하면서 테스트는 어떻게 만들 수 있을까를 생각해보는 것은 좋은 습관이다. 테스트하기 편하게 만들어진 코드는 깔끔하고 좋은 코드가 될 가능성이 높다.

스프링이 지지하고 권장하는 깔끔하고 유연한 코드를 만들다 보면 테스트도 그만큼 만들기 쉬워지고, 테스트는 다시 코드의 품질을 높여주고, 리팩토링과 개선에 대한 용기를 주기도 할 것이다. 반대로 좋은 코드를 만들려는 노력을 게을리하면 테스트 작성이 불편해지고, 테스트를 잘 만들지 않게 될 가능성이 높아진다. 

# 6.2.4 목 프레임워크

단위 테스트를 만들기 위해서는 스텁이나 목 오브젝트의 사용이 필수적이다. 의존관계가 없는 단순한 클래스나 세부 로직을 검증하기 위해 메소드 단위로 테스트할 때가 아니라면, 대부분 의존 오브젝트를 필요로 하는 코드를 테스트하게 되기 때문이다.

## Mockito 프레임워크

Mockito와 같은 목 프레임워크의 특징은 목 클래스를 일일이 준비해둘 필요가 없다는 점이다. 간단한 메소드 호추란으로 다이내믹하게 특정 인터페이스를 구현한 테스트용 목 오브젝트를 만들 수 있다.

```java
UserDao mockUserDao = mock(UserDao.class); // 이렇게 만들어진 목 오브젝트는 아직 아무런 기능이 없다.

when(mockUserDao.getAll()).thenReturn(this.users); // mockUserDao.getAll()이 호출됐을 때, users 리스트를 리턴해주라는 선언

verify(mockUserDao, times(2)).update(any(User.class)); // update() 호출이 두번 있었는지를 검증하는 부분
```

이렇게 UserDao 인터페이스를 구현한 클래스를 만들 필요도 없고 리턴 값을 생성자를 통해 넣어줬다가 메소드 호출 시 리턴하도록 코드를 만들 필요도 없다. 

편리하게 작성된 메소드 몇 개로 목 오브젝트를 사용할 수 있게해준다.

Mockito 목 오브젝트는 다음의 네 단계를 거쳐서 사용하면 된다.

- 인터페이스를 이용해 목 오브젝트를 만든다.
- 목 오브젝트가 리턴할 값이 있으면 이를 지정해준다. 메소드가 호출되면 예외를 강제로 던지게 만들 수도 있다.
- 테스트 대상 오브젝트에 DI 해서 목 오브젝트가 테스트 중에 사용되도록 만든다.
- 테스트 대상 오브젝트를 사용한 후에 목 오브젝트의 특정 메소드가 호출됐는지, 어떤 값을 가지고 몇 번 호출됐는지를 검증한다.

```java
@Test
public void mockUpgradeLevels() throws Exception {
	UserServiceImpl userServiceImpl = new UserServiceImpl();
	
	UserDao mockUserDao = mock(UserDao.class);
	when(mockUserDao.getAll()).thenReturn(this.users);
	userServiceImpl.setUserDao(mockUserDao);
	
	MailSender mockMailSender = mock(MailSender.class);
	userServiceImpl.setMailSender(mockMailSender);

	userServiceImpl.upgradeLevels();

	// 검증
	verify(mockUserDao, times(2)).update(any(User.class)); // 메소드 호출 횟수 검증
	verify(mockUserDao, times(2)).update(any(User.class)); 
	veryfy(mockUserDao).update(users.get(1)); // update에 넘겨진 객체가 두번째 사용자인지 검증
	assertThat(users.get(1).getLevel(), is(Level.SILVER)); // 두번쨰 사용자의 레벨이 SILVER로 승급되었는지 검증
	veryfy(mockUserDao).update(users.get(3));
	assertThat(users.get(3).getLevel(), is(Level.GOLD));

	// 실제 MailSender 목 오브젝트에 전달된 파라미터를 가져와 내용을 검증하는 방법을 사용
	ArgumentCaptor<SimpleMailMessage> mailMessageArg = ArgumentCaptor.forClass(SimpleMailMessage.class); 
	verify(mockMailSender, times(2)).send(mailMessageArg.capture()); // 파라미터 캡쳐
	List<SimpleMailMessage> mailMessages = mailMessageArg.getAllValues(); 
	assertThat(mailMessages.get(0).getTo()[0], is(users.get(1).getEmail()));
	assertThat(mailMessages.get(1).getTo()[0], is(users.get(3).getEmail()));
}
```

# 6.3 다이내믹 프록시와 팩토리 빈

# 6.3.1 프록시와 프록시 패턴, 데코레이터 패턴

트랜잭션이라는 기능은 사용자 관리 비즈니스 로직과는 성격이 다르기 때문에 아예 그 적용 사실 자체를 밖으로 분리할 수있다.이 방법을 이용해 UserServiceTx를 만들었고, UserServiceImpl에는 트랜잭션 관련 코드가 하나도 남지 않게 됐다.

이렇게 분리된 부가기능을 담은 클래스는 부가기능 외의 나머지 모든 기능은 원래 핵심기능을 가진 클래스로 위임해줘야 하는 중요한 특징이 있다.

핵심 기능은 부가기능을 가진 클래스의 존재자체를 모른다. 따라서 부가기능이 핵심기능을 사용하는 구조가 되는 것이다.

문제는 클라이언트가 핵심기능을 가진 클래스를 직접 사용해버리면 부가기능이 적용될 기회가 없다는 점이다. 그래서 부가기능은 마치 자신이 핵심 기능을 가진 클래스인 것처럼 꾸며서, 클라이언트가 자신을 거쳐서 핵심기능을 사용하도록 만들어야 한다.  그러기 위해서는 클라이언트는 인터페이스를 통해서만 핵심기능을 사용하게 하고, 부가기능 자신도 같은 인터페이스를 구현한 뒤에 자신이 그 사이에 끼어들어야 한다.

이렇게 마치 자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 것을 프록시(proxy) 라고 부른다. 그리고 프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트를 타깃(target) 또는 실체(real subject)라고 부른다.

프록시의 특징은 타깃과 같은 인터페이스를 구현 했다는 것과 프록시가 타깃을 제어할 수 있는 위치에 있다는 것이다.

프록시는 사용 목적에 따라 두 가지로 구분할 수 있다. 첫째는 클라이언트가 타깃에 접근하는 방법을 제어하기 위해서다.  두 번째는 타깃에 부가적인 기능을 부여해주기 위해서다.

# 데코레이터 패턴

데코레이터 패턴은 타깃에 부가적인 기능을 런타임 시 다이내믹하게 부여해주기 위해 프록시를 사용하는 패턴을 말한다. 

다이내믹 하다는 말은 컴파일 시점, 즉 코드상에서는 어떤 방법과 순서로 프록시와 타깃이 연결되어 사용되는지 정해져 있지 않다는 뜻이다.

데코레이터 패턴에서는 같은 인터페이스를 구현한 타겟과 여러 개의 프록시를 사용할 수 있다. 프록시로서 동작하는 각 데코레이터는 위임하는 대상에도 인터페이스로 접근하기 때문에 자신이 최종 타깃으로 위임하는지, 아니면 다음 단계의 데코레이터 프록시로 위임하는지 알지 못한다. 그래서 데코레이터의 다음 위임 대상은 인터페이스로 선언하고 런타임 시에 주입받을 수 있도록 만들어야 한다.

데코레이터 패턴은 타깃의 코드를 손대지 않고, 클라이언트가 호출하는 방법도 변경하지 않은 채로 새로운 기능을 추가할 때 유용한 방법이다.

# 프록시 패턴

여기서 말하는 프록시 패턴은 타깃에 대한 접근 방법을 제어하려는 목적을 가진 경우이다.

프록시 패턴의 프록시는 타깃의 기능을 확장하거나 추가하지 않는다. 대신 클라이언트가 타깃에 접근하는 방식을 변경해준다.

타깃 오브젝트가 생성하기 복잡하거나 당장 필요하지 않은 경우에는 꼭 필요한 시점까지 오브젝트를 생성하지 않는 편이 좋다.

그런데 타깃 오브젝트에 대한 레퍼런스가 미리 필요할 수 있다. 이럴 때 프록시 패턴을 적용하면 된다. 레퍼런스가 필요할 때 실제 타깃 오브젝트를 만드는 대신 프록시를 넘겨주는 것이다. 그리고 프록시의 메소드를 통해 타깃을 사용하려고 시도하면, 그떄 프록시가 다깃 오브젝트를 생성하고 요청을 위임해주는 식이다.

프록시는 코드에서 자신이 만들거나 접근할 타깃 클래스 정보를 알고 있는 경우가 많다. 생성을ㅈ ㅣ연하는 프록시라면 구체적인 생성 방법을 알아야 하기 때문에 타깃 클래스에 대한 직접적인 정보를 알아야 한다. 물론 프록시 패턴이라고 하더라도 인터페이스를 통해 위임하도록 만들 수도 있다.

# 6.3.2 다이내믹 프록시

프록시를 생성하는 일은 매우 번거롭게 느껴진다. 매번 새로운 클래스를 정의해야 하고, 인터페이스의 구현해야 할 메소드가 많으면 모든 메소드를 일일이 구현해서 위임하는 코드를 넣어야 하기 때문이다. 일일이 모든 인터페이스를 구현해서 클래스를 새로 정의하지 않고도 편리하게 만들어서 사용할 방법은 없을까? 

자바에는 java.lang.reflect 패키지 안에 프록시를 손쉽게 만들 수 있도록 지원해주는 클래스들이 있다.

프록시 클래스를 정의하지 않고도 몇 가지 API를 이용해 프록시처럼 동작하는 오브젝트를 다이내믹하게 생성하는 것이다.

# 프록시의 구성과 프록시 작성의 문제점

프록시는 다음의 두 가지 기능으로 구성된다.

- 타깃과 같은 메소드를 구현하고 있다가 메소드가 호출되면 타깃 오브젝트로 위임한다.
- 지정된 요청에 대해서는 부가기능을 수행한다.

```java
public class UserServiceTx implements UserService {
	UserService userService; // 타깃 오브젝트
	...
	
	public void add(User user) {
		this.userService.add(user);
	}

	public void upgradeLevels() {
		TransactionStatus status = this.trasactionManager.getTransaction(new DefaultTransactionDefinition());
		try {
			userService.upgradeLevels(); // 위임
			this.transactionManager.commit(status);
		} catch (RuntimeException e) {
			this.transactionManager.rollback(status);
			throw e;
		}
	}

}
```

프록시를 만들기가 번거로운 이유는 두가지를 찾아볼 수 있다.

- 첫째, 타깃의 인터페이스를 구현하고 위임하는 코드를 작성하기가 번거롭다.
- 둘째, 부가기능 코드가 중복될 가능성이 많다는 점이다.

사용자 관리 로직 외에도 다양한 비즈니스 로직을 담은 클래스가 만들어질 것인데, 그때마다 메소드에 트랜잭션을 부여하는 코드가 중복돼야 할지 모른다. 트랜잭션 외의 프록시를 활용할 만한 부가기능, 접근제어 기능은 일반적인 성격을 띤 것들이 많다. 따라서 다양한 타깃 클래스와 메소드에 중복돼서 나타날 가능성이 높다.

두번째 문제인 부가기능의 중복 문제는 중복되는 코드를 분리해서 어떻게든 해결될 것 같지만, 첫 번째 문제인 인터페이스 메소드의 구현과 위임 기능 문제는 간단해 보이지 않는다. 이는 JDK의 다이내믹 프록시를 사용해 해결할 수 있다.

# 리플렉션

다이내믹 프록시는 리플렉션 기능을 이용해서 프록시를 만들어준다. 리플렉션은 자바의 코드 자체를 추상화해서 접근하도록 만든 것이다.

자바의 모든 클래스는 그 클래스 자체의 구성정보를 담은 Class 타입의 오브젝트를 하나씩 갖고 있다. ‘클래스명.class’ 또는 오브젝트의 getClass() 메소드를 호출하면 클래스 정보를 담은 Class 타입의 오브젝트를 가져올 수 있다. 클래스 오브젝트를 이용하면 클래스 코드에 대한 메타정보를 가져오거나 오브젝트를 조작할 수 있다. 더 나아가서 오브젝트 필드의 값을 읽고 수정할 수도 있고, 원하는 파라미터 값을 이용해 메소드를 호출할 수도 있다.

```java
Method lengthMethod = String.class.getMethod("length");
int length = lengthMethod.invoke(name); // int length = name.length();
```

# 프록시 클래스

다이내믹 프록시를 이용한 프록시를 만들어보자.

```java
interface Hello {
	String sayHello(String name);
	String sayHi(String name);
	String sayThankYou(String name);
}

public class HelloTarget implements Hello {
	public String sayHello(String name) {
		return "Hello " + name;
	}

	public String sayHi(String name) {
		return "Hi " + name;
	}
	
	public String sayThankYou(String name) {
		return "Thank You " + name;
	}
}

// 프록시
public class HelloUppercase implements Hello {
	Hello hello;
	
	public HelloUppercase(Hello hello) {
		this.hello = hello;
	}

	public String sayHello(String name) {
		return hello.sayHello(name).toUpperCase();
	}

	public String sayHi(String name) {
		return hello.sayHi(name).toUpperCase();
	}

	public String sayThankYou(String name) {
		return hello.sayThankYou(name).toUpperCase();
	}
}
```

이 프록시는 프록시 적용의 일반적인 문제점 두가지 모두를 가지고 있다. 인터페이스의 모든 메소드를 구현해 위임하도록 코드를 만들어야 하며, 부가기능인 리턴 값을 대문자로 바꾸는 기능이 모든 메소드에 중복돼서 나타난다.

# 다이내믹 프록시 적용

클래스로 만든 프록시인 HelloUppercase를 다이내믹 프록시를 이용해 만들어보자.

다이내믹 프록시는 프록시 팩토리에 의해 런타임 시 다이내믹하게 만들어지는 오브젝트다. 

다이내믹 프록시 오브젝트는 타깃의 인터페이스와 같은 타입으로 만들어진다. 덕분에 프록시를 만들 때 인터페이스를 모두 구현해가면서 클래스를 정의하는 수고를 덜 수 있다. 프록시 팩토리에게 인터페이스 정보만 제공해주면 해당 인터페이스를 구현한 클래스의 오브젝트를 자동으로 만들어주기 때문이다.

부가기능은 직접 작성해야하므로 프록시 오브젝트와 독립적으로 InvocationHandler를 구현한 오브젝트에 담는다. 

InvocationHandler는 한 개의 메소드만 가진 간단한 인터페이스이다.

```java
public Object invoke(Object proxy, Method method, Object[] args)
```

Invoke() 메소드는 리플렉션의 Method 인터페이스를 파라미터로 받는다. 메소드를 호출할 때 전달되는 파라미터도 args로 받는다. 다이내믹 프록시 오브젝트는 클라이언트의 모든 요청을 리플렉션 정보로 변환해서 InvocationHandler 구현 오브젝트의 invoke() 메소드로 넘기는 것이다. 타깃 인터페이스의 모든 메소드 요청이 하나의 메소드로 집중되기 때문에 중복되는 기능을 효과적으로 제공할 수 있다.

```java
public class UppercaseHandler implements InvocationHandler {
	Hello target;
	public UppercaseHandler(Hello target) {
		this.target = target;
	}

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		String ret = (String)method.invoke(target, args);
		return ret.toUpperCase();
	}
}
```

다이내믹 프로시가 클라이언트로부터 받는 모든 요청은 invoke() 메소드로 전달된다. 다이내믹 프록시를 통해 요청이 전달되면 리플렉션 API를 이용해 타깃 오브젝트의 메소드를 호출한다.

```java
Hello proxiedHello = (Hello) Proxy.newProxyInstance(
		getClass().getClassLoader(), // 클래스 로딩에 사용될 클래스 로더
		new Class[] {Hello.class}, // 구현할 인터페이스
		new UppercaseHandler(new HelloTarget())); // 부가기능과 위임 코드를 담은 InvocationHandler
```

# 다이내믹 프록시의 확장

다이내믹 프록시 방식이 직접 정의해서 만든 프록시보다 훨씬 유연하고 많은 장점이 있다.

Hello 인터페이스의 메소드가 30개로 늘어나면, 클래스로 직접 구현한 프록시는 매번 코드를 추가해야 한다. 하지만 UppercaseHandler와 다이내믹 프록시를 생성해서 사용하는 코드는 전혀 손댈 게 없다.

InvocationHandler 방식의 또 한가지 장점은 타깃의 종류에 상관없이도 적용이 가능하다는 점이다. 

어차피 리플렉션의 Method 인터페이스를 이용해 타깃의 메소드를 호출하는 것이니 Hello 타입의 타깃으로 제한할 필요도 없다.

어떤 종류의 인터페이스를 구현한 타깃이든 상관없이 재사용할 수 있고, 메소드의 리턴 타입이 스트링인 경우만 대문자로 결과를 바꿔주도록 UppercaseHandler를 만들 수있다.

```java
public class UppercaseHandler implements InvocationHandler {
	Object target;
	public UppercaseHandler(Object target) {
		this.target = target;
	}

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		Object ret = method.invoke(target, args);
		return ret.toUpperCase();
	}
}

/// 메소드명으로 적용 구분
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	Object ret = method.invoke(target, args);
	if (ret instanceof String && method.getName().startsWith("say")) {
		return ((String)ret).toUpperCase();
	} else {
		return ret;
	}
}

```

InvocationHandler는 단일 메소드에서 모든 요청을 처리하기 때문에 어떤 메소드에 어떤 기능을 적용할지를 선택하는 과정이 필요할 수도 있다. 호출하는 메소드 이름, 파라미터의 개수와 타입, 리턴 타입 등의 정보를 가지고 부가적인 기능을 적용할 메소드를 선택할 수 있다. 

# 6.3.3 다이내믹 프록시를 이용한 트랜잭션 부가기능

# 트랜잭션 InvocationHandler

```java
public class TransactionHandler implements InvocationHandler {
	private Object target;
	private PlatformTransactionManager transactionManager;
	private String pattern;

	public void setTarget(Object target) {
		this.target = target;
	}

	public void setTransactionManager(PlatformTransactionManager transactionManager) {
		this.transactionManager = transactionManager;
	}

	public void setPattern(String pattern) {
		this.pattern = pattern;
	}

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		if (method.getName().startsWith(pattern)) {
			return invokeTransaction(method, args);
		} else {
			return method.invoke(target, args);
		}
	}	
	
	public Object invokeTransaction(Method method, Object[] args) throws Throwable {
		TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
		try {
			Object ret = method.invoke(target, args);
			this.transactionManager.commit(status);
			return ret;
		} catch (InvocationTargetException e) {
			this.transactionManager.rollback(status);
			throw e.getTargetException();
		}
	}

}
```

롤백을 적용하기 위한 예외는 RuntimeException 대신에 InvocationTargetException을 잡도록 해야한다. Method.invoke()를 이용해 타깃 오브젝트의 메소드를 호출할 때는 타깃 오브젝트에서 발생하는 예외가 InvocationTargetException으로 한번 포장돼서 전달된다.

# 6.3.4 다이내믹 프록시를 위한 팩토리 빈

이제 TransactionHandler와 다이내믹 프록시를 스프링의 DI를 통해 사용할 수 있도록 만들어야 할 차례다.

그런데 문제는 DI의 대상이 되는 다이내믹 프록시 오브젝트는 일반적인 스프링의 빈으로는 등록할 방법이 없다는 것이다. 스프링 빈은 기본적으로 클래스 이름과 프로퍼티로 정의된다. 스프링은 지정된 클래스 이름을 가지고 리플렉션을 이용해서 해당 클래스의 오브젝트를 만든다. 클래스의 이름을 갖고 있다면 다음과 같은 방법으로 새로운 오브젝트를 생성할 수 있다.

```java
Date now = (Date) Class.forName("java.util.Date").newInstance();
```

스프링은 내부적으로 리플렉션 API를 이용해서 빈 정의에 나오는 클래스 이름을 가지고 빈 오브젝트를 생성한다. 문제는 다이내믹 프록시는 이런 식으로 프록시 오브젝트가 생성되지 않는다는 점이다.

다이내믹 프록시는 Proxy 클래스의 newProxyInstance()라는 스태틱 팩토리 메소드를 통해서만 만들 수 있다.

# 팩토리 빈

스프링은 클래스 정보를 가지고 디폴트 생성자를 통해 오브젝트를 만드는 방법 외에도 빈을 만들 수 있는 여러 가지 방법을 제공한다. 대표적으로 팩토리 빈을 이용한 빈 생성 방법이 있다.

팩토리 빈이란 스프링을 대신해서 오브젝트의 생성로직을 담당하도록 만들어진 특별한 빈을 말한다.

팩토리 빈을 만드는 방법에는 여러 가지가 있는데, 가장 간단한 방법은 스프링의 FactoryBean이라는 인터페이스를 구현하는 것이다. 

```java
public interface FactoryBean<T> {
	T getObject() throws Exception;
	Class<? extends T> getObjectType();
	boolean isSingleton();
}
```

FactoryBean 인터페이스를 수현한 클래스를 스프링의 빈으로 등록하면 팩토리 빈으로 동작한다. 

```java
public class Message {
	String text;
	private Message(String text) { // 외부에서 생성자를 통해 Object를 만들 수 없다.
		this.text = text;
	}

	public String getText() {
		return text;
	} 

	public static Message newMessage(String text) { // 생성자 대신 사용할 수있는 스태틱 팩토리 메소드를 제공한다.
		return new Message(text);
	}
}
```

Message 클래스의 생성자가 private이므로 다음과 같이 사용할 수 없다.

```xml
<bean id="m" class="***.***.Message">
```

리플렉션은 private으로 선언된 접근 규약을 위반할 수 있는 기능이 있기 때문에, 사실 스프링은 private 생성자를 가진 클래스도 빈으로 등록해주면 리플렉션을 이용해 오브젝트를 만들어준다. 하지만 생성자를 private으로 만들었다는 것은 스태틱 메소드를 통해 오브젝트가 만들어져야 하는 중요한 이유가 있기 떄문이므로 이를 무시하고 오브젝트를 강제로 생성하면 위험하다.

Message 클래스의 오브젝트를 생성해주는 팩토리 빈 클래스를 만들어보자

```java
public class MessageFactoryBean implements FactoryBean<Message> {
	String text;
	
	public void setText(String text) {
		this.text = text;
	}
	
	public Message getObject() throws Exception { // 실제 Bean으로 사용 될 오브젝트 직접 생성
		return Message.newMessage(this.text);
	}

	public Class<? extends Message> getObjectType() {
		return Message.class;
	}

	public boolean isSingleton() { // 이는 팩토리 빈의 동작방식에 대한 설정이고, 만들어진 빈 오브젝트는 싱글톤으로 스프링이 관리해줄 수있다.
		return false;
	}
}
```

팩토리 빈은 빈 오브젝트를 생성하는 과정에서만 사용된다.

# 팩토리 빈의 설정 방법

```xml
<bean id="message" class="***.***.MessageFactoryBean">
	<property name="text" value="Factory Bean" />
</bean>
```

여타 빈 설정과 다른 점은 Message bean object의 타입이 class 애트리뷰트에 정의된 MessageFactoryBean이 아니라 Message 타입이라는 것이다. Message 빈의 타입은 MessageFactoryBean의 getObjectType() 메소드가 돌려주는 타입으로 결정된다. 또 getObject() 메소드가 생성해주는 오브젝트가 message 빈의 오브젝트가 된다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration
public class FactoryBeanTest {
	@Autowired
	ApplicationContext context;

	@Test
	public void getMessageFromFactoryBean() {
		Object message = context.getBean("message");
		assertThat(message, is(Message.class));
		assertThat(((Message)message).getText(), is("Factory Bean"));
	}

}
```

이제 FactoryBean 인터페이스를 구현한 클래스를 스프링 빈으로 만들면 getObject() 메소드가 생성해주는 오브젝트가 실제 빈의 오브젝트로 대치된다는 사실을 알 수 있다.

팩토리 빈 자체를 가져오고 싶다면 ‘&’을 빈 이름 앞에 붙여주면 된다.

# 다이내믹 프록시를 만들어주는 팩토리 빈

팩토리 빈을 사용하면 다이내믹프록시 오브젝트를 스프링의 빈으로 만들어줄 수 있다. 팩토리 빈의 getObject() 메소드에 다이내믹 프록시 오브젝트를 만들어주는 코드를 넣으면 되기 때문이다.

스프링 빈에는 팩토리 빈과 UserServiceImpl만 빈으로 등록한다. 팩토리 빈은 다이내믹 프록시가 위임할 타깃 오브젝트인 UserServiceImpl에 대한 레퍼런스를 프로퍼티를 통해 DI 받아둬야 한다.

# 트랜잭션 프록시 팩토리 빈

```java
public class TxProxyFactoryBean implements FactoryBean<Object> {
	Object target;
	PlatformTransactionManager transactionManager;
	String pattern;
	Class<?> serviceInterface; // UserService 외의 인터페이스를 가진 타겟에도 적용할 수 있다.
	
	public void setTarget(Object target) {
		this.target = target;
	}

	public void setTransactionManager(PlatformTransactionManager transactionManager) {
		this.transactionManager = transactionManager;
	}

	public void setPattern(String pattern) {
		this.pattern = pattern;
	}

	public void setServiceInterface(Class<?> serviceInterface) {
		this.serviceInterface = serviceInterface;
	}

	public Object getObject() throws Exception {
		TransactionHandler txHandler = new TransactionHandler();
		txHandler.setTarget(target);
		txHandler.setTransactionManager(transactionManager);
		txHandler.setPattern(pattern);
		return Proxy.newProxyInstance(
						getClass().getClassLoader(), new Class[] { serviceInterface },
						txHandler);
	}
	
	public Class<?> getObjectType() {
		return serviceInterface; 
	}

	public boolean isSingleton() {
		return false;
	}
	
}
```

팩토리 빈이 만드는 다이내믹 프록시는 구현 인터페이스나 타깃의 종류에 제한이 없다. 따라서 UserService 외에도 트랜잭션 부가기능이 필요한 오브젝트를 위한 프록시를 만들 때 얼마든지 재사용이 가능하다. 설정이 다른 여러 개의 TxProxyFactoryBean 빈을 등록하면 된다.

```xml
<bean id="userService" class="***.***.TxProxyFactoryBean">
	<property name="target" ref="userServiceImpl" />
	<property name="transactionManager" ref="transactionManager" />
	<property name="pattern" value="upgradeLevels" />
	<property name="serviceInterface" value="springbook.user.service.UserService"/>
</bean>
```

여기서 serviceInterface는 숫자, 문자와 같은 단순한 타입이 아니라 Class 타입이다. Class 타입은 위와 같이 value를 이용해 클래스 또는 인터페이스의 이름을넣어주며 된다. 스프링은 수정자 메소드의 파라미터의 타입을 확인해서 프로퍼티의 타입이 Class인 경우는 value로 설정한 이름을 가진 Class Object로 자동 변환해준다.

# 트랜잭션 프록시 팩토리 빈 테스트

```java
public class UserServiceTest {
	@Autowired ApplicationContext context; // 팩토리 빈을 가져오려면 애플리케이션 컨텍스트가 필요하다.
	
	@Test
	@DirtiesContext
	public void upgradeAllOrNothing() throws Exception {
		TestUserService testUserService = new TestUserService(users.get(3).getId());
		testUserService.setUserDao(userDao);
		testUserService.setMailSender(mailSender);
		
		TxProxyFactoryBean txProxyFactoryBean = context.getBean("&userService", TxProxyFactoryBean.class);
		txProxyFactoryBean.setTarget(testUserService);
		UserService txUserService = (UserService) txProxyFactoryBean.getObject();

		userDao.deleteAll();
		for(User user: users) userDao.add(user);

		try {
			txUserService.upgradeLevels();
			fail("TestUserServiceException expected");
		} catch (TestUserServiceException e) {
			
		}
		
		checkLevelUpgraded(users.get(1), false);
	}
}
```

# 6.3.5 프록시 팩토리 빈 방식의 장점과 한계

# 프록시 팩토리 빈의 재사용

TransactionHandler를 이용하는 다이내믹 프록시를 생성해주는 TxProxyFactoryBean은 코드의 수정 없이도 다양한 클래스에 적용할 수 있다. 타깃 오브젝트에 맞는 프로퍼티 정보를 설정해서 빈으로 등록해주기만 하면 된다. 하나 이상의 TxProxyFactoryBean을 동시에 빈으로 등록해도 상관없다. 팩토리 빈이기 때문에 각 빈의 타입은 타깃 인터페이스와 일치한다.

```java
<bean id="coreServiceTarget" class="complex.module.CoreServiceImpl">
	<property name="coreDao" ref="coreDao"/>
</bean>

<bean id="coreService" class="***.***.TxProxyFactoryBean">
	<property name="target" ref="coreServiceTarget"/>
	...
	<property name="serviceInterface" value="complex.module.CoreService"/>

</bean>
```

# 프록시 팩토리 빈 방식의 장점

프록시 팩토리 빈은 위에서 발생하던 두 가지 문제를 해결해준다.

다이내믹 프록시를 이용하면 타깃 인터페이스를 구현하는 클래스를 일일이 만드는 번거로움을 제거할 수 있다. 하나의 핸들러 메소드를 구현하는 것만으로도 수많은 메소드에 부가 기능을 부여해줄 수 있으니 부가기능 코드의 중복 문제도 사라진다. 다이내믹 프록시에 팩토리 빈을 이용한 DI 까지 더해주면 번거로운 다이내믹 프록시 생성 코드도 제거할 수 있다.

# 프록시 팩토리 빈의 한계

프록시를 통해 타깃에 부가기능을 제공하는 것은 메소드 단위로 일어나는 일이다. 하나의 클래스 안에 존재하는 여러 개의 메소드에 부가기능을 한 번에 제공하는 건 어렵지 않게 가능했다. 하지만 한번에 여러 개의 클래스에 공통적인 부가기능을 제공하는 일은 지금까지 살펴본 방법으로는 불가능하다. 

트랜잭션과 같이 비즈니스 로직을 담은 많은 클래스의 메소드에 적용할 필요가 있다면 거의 비슷한 프록시 팩토리 빈의 설정이 중복되는 것을 막을 수 없다.

하나의 타깃에 여러 개의 부가기능을 적용하려고 할 때도 문제다. 설정 파일이 급격히 복잡해지고 이는 바람직하지 못하다.

게다가 타깃과 인터페이스만 다른, 거의 비슷한 설정이 자꾸 반복된 다는 점이 뭔가 찜찜하다.

또 한가지 문제점은 TransactionHandler 오브젝트가 프록시 팩토리 빈 개수만큼 만들어진다는 점이다. TransactionHandler는 타깃 오브젝트를 프로퍼티로 갖고 있다. 따라서 트랜잭션 부가기능을 제공하는 동일한 코드임에도 불구하고 타깃 오브젝트가 달라지면 새로운 TransactionHandler 오브젝트를 만들어야 한다. 

TransactionHandler의 중복을 없애고 모든 타깃에 적용 가능한 싱글톤 빈으로 만들어서 적용할 수는 없을까?