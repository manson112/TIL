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

# 6.4 스프링의 프록시 팩토리 빈

스프링은 이런 문제에 대한 해법을 매우 세련되고 깔끔한 방식으로 제공한다.

# 6.4.1 ProxyFactoryBean

자바에는 JDK에서 제공하는 다이내믹 프록시 외에도 편리하게 프록시를 만들 수 있도록 지원해주는 다양한 기술이 존재한다. 따라서 스프링은 일관된 방법으로 프록시를 만들 수 있게 도와주는 추상 레이어를 제공한다.

스프링의 ProxyFactoryBean은 프록시를 생성해서 빈 오브젝트로 등록하게 해주는 팩토리 빈이다. 순수하게 프록시를 생성하는 작업만을 담당하고 프록시를 통해 제공해줄부가기능은 별도의 빈에 둘 수 있다.

ProxyFactoryBean이 생성하는 프록시에서 사용할 부가기능은 MethodInterceptor 인터페이스를 구현해서 만든다. MethodInterceptor는 InvacationHandler와 비슷하지만 한 가지 다른 점이 있다. InvocationHandler의 invoke() 메서드는 타깃 오브젝트에 대한 정보를 제공하지 않는다. 따라서 타깃은 InvocationHandler를 구현한 클래스가 직접 알고 있어야 한다. 

반면에 MethodInterceptor의 invoke() 메서드는 ProxyFactoryBean으로부터 타깃 오브젝트에 대한 정보까지도 함께 제공받는다. 그 차이 덕분에 MethodInterceptor는 타깃 오브젝트에 상관없이 독립적으로 만들어질 수 있다. 따라서 MethodInterceptor 오브젝트는 타깃에 다른 여러 프록시에서 함께 사용할 수 있고, 싱글톤 빈으로 등록 가능하다.

```java
public class DynamicProxyTest {
	@Test
	public void simpleProxy() {
		Hello proxiedHello = (Hello)Proxy.newProxyInstance(
			getClass().getClassLoader(),
			new Class[] {Hello.class},
			new UppercaseHandler(new HelloTarget()));
	}

	@Test
	public void proxyFactoryBean() {
		ProxyFactoryBean pfBean = new ProxyFactoryBean();
		pfBean.setTarget(new HelloTarget());
		pfBean.addAdvice(new UppercaseAdvice());
		Hello proxiedHello = (Hello) pfBean.getObject();
	}

	static class UppercaseAdvice implements MethodInterceptor {
		public Object invoke(MethodInvocation invocation) throws Throwable {
			String ret = (String)invocation.proceed();
			return ret.toUpperCase(); -> 부가기능 추가
		}
	}

}
```

# 어드바이스: 타깃이 필요 없는 순수한 부가기능

InvocationHandler를 구현했을 때와 달리 MethodInterceptor를 구현한 UppercaseAdvice에는 타깃 오브젝트가 등장하지 않는다. MethodInterceptor로는 메소드 정보와 함께 타깃 오브젝트가 담긴 MethodInvocation 오브젝트가 전달된다. MethodInvocation은 타깃 오브젝트의 메소드를 실행할 수 있는 기능이 있기 때문에 MethodInterceptor는 부가기능을 제공하는 데만 집중할 수 있다.

MethodInvocation은 일종의 콜백 오브젝트로, proceed() 메소드를 실행하면 타깃 오브젝트의 메소드를 내부적으로 실행해주는 기능이 있다. 그래서 MethodInvocation 구현 클래스는 일종의 공유 가능한 템플릿처럼 동작하는 것이다. 바로 이 점이 JDK의 다이내믹 프록시를 직접 사용하는 코드와 스프링이 제공해주는 프록시 추상화 기능인 ProxyFactoryBean을 사용하는 코드의 가장 큰 차이점이자 장점이다. 따라서 MethodInvocation은 싱글톤으로 두고 공유할 수 있다. 

MethodInterceptor를 설정해줄 때는 일반적인 DI 경우처럼 수정자 메서드를 사용하는 대신 addAdvice()라는 메서드를 사용한다.

ProxyFactoryBean에는 여러 개의 MethodInterceptor를 추가할 수 있는데, 이는 ProxyFactoryBean 하나 만으로 여러 개의 부가기능을 제공해주는 프록시를 만들 수 있다는 뜻이다. 따라서 앞에서 살펴봤던 프록시 팩토리 빈의 단점 중 하나였던, 새로운 부가기능을 추가할 때마다 프록시와 프록시 팩토리 빈도 추가해줘야 한다는 문제를 해결할 수 있다.

그런데 MethodInterceptor 오브젝트를 추가하는 메서드 이름은 addMethodInterceptor가 아니라 addAdvice다. MethodInterceptor는 Advice 인터페이스를 상속하고 있는 서브인터페이스이기 때문이다. 이름에서 알 수 있듯이 MethodInterceptor처럼 타깃 오브젝트에 적용하는 부가기능을 담은 오브젝트를 스프링에서는 어드바이스(Advice)라고 부른다.

ProxyFactoryBean을 적용한 코드에는 프록시가 구현해야 하는 Hello라는 인터페이스를 제공해주는 부분이 없다. 프록시를 직접 만들 때나 JDK 다이내믹 프록시를 만들 때 반드시 제공해줘야 하는 정보가 Hello 인터페이스였다. 그래야만 다이내믹 프록시 오브젝트의 타입을 결정할 수 있기 때문이다.

그런테 스프링의 ProxyFactoryBean은 인터페이스를 굳이 알려주지 않아도 ProxyFactoryBean에 있는 인터페이스 자동검출 기능을 사용해 타깃 오브젝트가 구현하고 있는 인터페이스 정보를 알아낸다. 그리고 알아낸 인터페이스를 모두 구현하는 프록시를 만들어준다. 

ProxyFactoryBean은 기본적으로 JDK가 제공하는 다이내믹 프록시를 만들어준다. 경우에 따라서는 CGLib이라고 하는 오픈소스 바이트코드 생성 프로임워크를 이용해 프록시를 만들기도 한다.

# 포인트컷: 부가기능 적용 대상 메소드 선정 방법

기존에 InvocationHandler를 직접 구현했을 때는 부가기능 적용 외에도 메소드의 이름을 가지고 부가기능 적용 대상을 선정하는 작업이 있었다. 그렇다면 ProxyFactoryBean과 MethodInterceptor를 사용하는 방식에서도 메소드 선정 기능을 넣을 수 있을까? 

MethodInterceptor는 여러 프록시가 공유해서 사용할 수 있다. 그러기 위해서 타깃 정보를 가지고 있지 않도록 만들었다. 그런데 여기에다 트랜잭션 적용 대상 메소드 이름 패턴을 넣어주는 것은 곤란하다. 트랜잭션 적용 메소드 패턴은 프록시마다 다를 수 있기 때문에 여러 프록시가 공유하는 MethodInterceptor에 특정 프록시에만 적용되는 패턴을 넣으면 문제가 된다.

이 문제는 어떻게 해결할 수 있을까?

함께 두기 곤란한 성격이 다르고 변경 이유와 시점이 다르고, 생성 방식과 의존관계가 다른 코드가 함께 있다면 분리해주면 된다.

프록시의 핵심 가치는 타깃을 대신해서 클라이언트의 요청을 받아 처리하는 오브젝트로서의 존재 자체이므로, 메소드를 선별하는 기능은 프록시로부터 다시 분리하는 편이 낫다. 메소드를 선정하는 일도 일종의 교환 가능한 알고리즘이므로 전략 패턴을 적용할 수있기 때문이다.

스프링의 ProxyFactoryBean 방식은 두 가지 확장 기능인 Advice와 Pointcut을 활용하는 유연한 구졸르 제공한다.

스프링은 부가기능을 제공하는 오브젝트를 어드바이스라고 부르고, 메소드 선정 알고리즘을 담은 오브젝트를 포인트컷이라고 부른다. 어드바이스와 포인트컷은 모두 프록시에 DI로 주입돼서 사용된다. 두 가지 모두 여러 프록시에서 공유가 가능하도록 만들어지기 때문에 스프링의 싱글톤 빈으로 등록이 가능하다.

프록시는 클라이언트로부터 요청을 받으면 먼저 포인트컷에게 부가기능을 부여할 메소드인지를 확인해달라고 요청한다. 

포인트컷은 Pointcut 인터페이스를 구현해서 만들면 된다. 프록시는 포인트컷으로부터 부가기능을 적용할 대상 메소드인지 확인받으면, MethodInterceptor 타입의 어드바이스를 호출한다. 어드바이스는 JDK의 다이내믹 프록시의 InvocationHandler와 달리 직접 타깃을 호출하지 않는다. 자신이 공유돼야 하므로 타깃 정보라는 상태를 가질 수 없다. 따라서 타깃에 직접 의존하지 않도록 일종의 템플릿 구조로 설계되어 있다. 어드바이스가 부가기능을 부여하는 중에 타깃 메소드의 호출이 필요하면 프록시로부터 전달받은 MethodInvocation 타입 콜백 오브젝트의 proceed() 메소드를 호출해주기만 하면 된다.

실제 위임 대상인 타깃 오브젝트의 레퍼런스를 갖고 있고, 이를 이용해 타깃 메소드를 직접 호출하는 것은 프록시가 메소드 호출에 따라 만드는 Invocation 콜백의 역할이다. 이는 재사용 가능한 기능을 만들어두고 바뀌는 부분만 외부에서 주입해서 이를 작업 흐름 중에 사용하도록 하는 전형적인 템플릿/콜백 구조다. 

템플릿은 한 번 만들면 재사용이 가능하고 여러 빈이 공유해서 사용할 수 있듯이, 어드바이스도 독립적인 싱글톤 빈으로 등록하고 DI를 주입해서 여러 프록시가 사용하도록 만들 수 있다.

```java
@Test
public void pointcutAdvisor() {
	ProxyFactoryBean pfBean = new ProxyFactoryBean();
	pfBean.setTraget(new HelloTarget());
	
	NameMatchMethodPoiontcut pointcut = new NameMatchMethodPointcut();
	pointcut.setMappedName("sayH*") // 이름 비교 조건 설정

	pfBean.addAdvisor(new DefaultPointcutAdvisor(pointcut, new UppercasetAdvice()));
	
	Hello proxiedHello = (Hello) pfBean.getObject();
	
}
```

포인트컷이 필요 없을 때는 ProxyFactoryBean의 addAdvice 메소드를 호출해서 어드바이스만 등록하면 됐다. 그런데 포인틐컷을 함께 등록할 때는 어드바이스와 포인트컷을 Advisor 타입으로 묶어서 addAdvice 메소드를 호출해야 한다. ProxyFactoryBean에는 여러 개의 어드바이스와 포인트컷이 추가될 수 있기 때문이다. 포인트컷과 어드바이스를 따로 등록하면 어떤 어드바이스에 대해 어떤 포인트컷을 적용할지 애매해지기 때문이다.

이렇게 어드바이스와 포인트컷을 묶은 오브젝트를 인터페이스 이름을 따서 어드바이저 라고 부른다.

> Advisor = Pointcut + Advice
> 

# 6.4.2 ProxyFactoryBean 적용

JDK 다이내믹 프록시의 구조를 그대로 이용해서 만들었던 TxProxyFactoryBean을 이제 스프링이 제공하는 ProxyFactoryBean을 이용하도록 수정해보자

# TransactionAdvice

```java
public class TransactionAdvice implements MethodInterceptor {
	PlatformTransactionManager transactionManager;
	
	public void setTransactionManager(PlatformTransactionManager transactionManager) {
		this.transactionManager = transactionManager;
	}

	public Object invoke(MethodInvocation invocation) throws Throwable {
		TransactionStatus status = this.transactionManager.getRansaction(new DefaultTransactionDefinition());
		
		try {
			Object ret = invocation.proceed(); // 콜백을 호출해서 타깃의 메소드를 실행한다. 타깃 메소드 호출 전후로 필요한 부가기능을 넣을 수 있다.
			this.transactionManager.commit(status);
			return ret;
		} catch (RuntimeException e) { // JDK 다이내믹 프록시가 제공하는 Method와는 달리 예외가 포장되지 않고 타깃에서 보낸 그대로 전달된다.
			this.transactionManager.rollback(status);
			throw e;
		}	
	}
}
```

리플렉션을 통한 타깃 메소드 호출 작업의 번거로움은 MethodInvocation 타입의 콜백을 이용한 덕분에 대부분 제거할 수 있다. 타깃 메소드가 던지는 예외도 InvocationTargetException으로 포장돼서 오는 것이 아니기 때문에 그대로 잡아서 처리하면 된다.

# 스프링 XML 설정파일

```xml
<bean id="transactionAdvice" class="springbook.user.service.TransactionAdvice">
	<property name="transactionManager" ref="transactionManager" />
</bean>

<!-- upgrade로 시작하는 모든 메소드를 선택하도록 만든다. -->
<bean id="transactionPointcut" class="org.springframework.aop.support.NameMatchMethodPointcut">
	<property name="mappedName" value="upgrade*"/>
</bean>

<bean id="transactionAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
	<property name="advice" ref="transactionAdvice"/>
	<property name="pointcut" ref="transactionPointcut" />
</bean>

<bean id="userService" class="org.springframework.aop.framework.ProxyFactoryBean">
	<property name="target" ref="userServiceImpl" />
	<property name="interceptorNames">
		<list>
			<value>transactionAdvisor</value>
		</list>
	</property>
</bean>
```

어드바이저는 interceptorNames라는 프로퍼티를 통해 넣는다. 프로퍼티 이름이 advisor가 아닌 이유는 어드바이스와 어드바이저를 혼합해서 설정할 수 있도록 하기 위해서다.

그래서 property 태그의 ref 애트리뷰트를 통한 설정 대신 list와 value 태그를 통해 여러 개의 값을 넣을 수 있도록 하고 있다.

# 테스트

```java
@Test
@DirtiesContext
public void upgradeAllOrNothing() {
	TestUserService testUserService = new TestUserService(users.get(3).getId());
	testUserService.setUserDao(userDao);
	testUserService.setMailSender(manilSender);

	ProxyFactoryBean txProxyFactoryBean = context.getBean("&userService", ProxyFactoryBean.class);
	txProxyFactoryBean.setTarget(testUserService);
	UserService txUserService = (UserService) txProxyFactoryBean.getObject();
	...
}
```

# 6.5 스프링 AOP

지금까지 해왔던 작업의 목표는 비즈니스 로직에 반복적으로 등장해야만 했던 트랜잭션 코드를 깔끔하고 효과적으로 분리해내는 것이다. 이렇게 분리해낸 트랜잭션 코드는 투명한 부가기능 형태로 제공돼야 한다. 투명하다는 것은 부가기능을 적용한 후에도 기존 설계와 코드에는 영향을 주지 않는다는 뜻이다. 마치 투명한 유리를 사이에 둔 것처럼 다른 코드에서는 그 존재가 보이지 않지만, 메소드가 호출되는 과정에 다이내믹하게 참여해서 부가적인 기능을 제공해주도록 만드는 것이다. 

# 6.5.1 자동 프록시 생성

앞에서 한 작업들 덕분에, 타깃코드는 여전히 깔끔한 채로 남아 있고, 부가기능은 한 번만 만들어 모든 타깃과 메소드에 재사용이 가능하고, 타깃의 적용 메소드를 선정하는 방식도 독립적으로 작성할 수 있도록 분리되어 있다.

하지만 아직 한 가지 해결할 과제가 남아 있다. 

프록시 팩토리 빈 방식의 접근 방법의 한계라고 생각했던 두 가지 문제가 있었다. 그중에서 부가기능이 타깃 오브젝트마다 새로 만들어지는 문제는 스프링 ProxyFactoryBean의 어드바이스를 통해 해결됐다. 남은 것은 부가기능의 적용이 필요한 타깃 오브젝트마다 거의 비슷한 내용의 ProxyFactoryBean 설정 정보를 추가해주는 부분이다.

새로운 타깃이 등장했다고해서 코드를 손댈 필요는 없어졌지만, 설정은 매번 복사해서 붙이고 target 프로퍼티의 내용을 수정해줘야 한다. 단순하지만, 단순하고 쉬운만큼 실수하기도 쉽다.

# 중복 문제의 접근 방법

반복적인 프록시 메소드 구현을 코드 자동생성 기법을 이용해 해결했다면 반복적인 ProxyFactoryBean 설정 문제는 설정 자동등록 기법으로 해결할 수 없을까? 또는 실제 빈 오브젝트가 되는 것은 ProxyFactoryBean을 통해 생성되는 프록시 그 자체이므로 프록시가 자동으로 빈으로 생성되게 할 수는 없을까? 마치 다이내믹 프록시가 인터페이스만 제공하면 모든 메소드에 대한 구현 클래스를 자동으로 만들듯이, 일정한 타깃 빈의 목록을 제공하면 자동으로 각 타깃 빈에 대한 프록시를 만들어주는 방법이 있다면 ProxyFactoryBean 타입 빈 설정을 매번 추가해서 프록시를 만들어내는 수고를 덜 수 있을 것 같다.

# 빈 후처리기를 이용한 자동 프록시 생성기

스프링은 OCP의 가장 중요한 요소인 유연한 확장 이라는 개념을 스프링 컨테이너 자신에게도 다양한 방법으로 적용하고 있다. 

스프링은 컨테이너로서 제공하는 기능 중에서 변하지 않는 핵심적인 부분 외에는 대부분 확장할 수 있도록 확장 포인트를 제공해준다.

그중에서 관심을 가질 만한 확장 포인트는 바로 BeanPostProcessor 인터페이스를 구현해서 만드는 빈 후처리기다. 빈 후처리기는 이름 그대로 스프링 빈 오브젝트로 만들어지고 난 후에, 빈 오브젝트를 다시 가공할 수 있게 해준다. 

여기서는 스프링이 제공하는 빈 후처리기 중 하나인 DefaultAdvisorAutoProxyCreator를 살펴보겠다. DefaultAdvisorAutoProxtCreator는 Advisor를 이용한 자동 프록시 생성기다. 

빈 후처리기를 스프링에 적용하는 방법은 간단하다. 빈 후처리기 자체를 빈으로 등록하는 것이다. 스프링은 빈 후처리기가 빈으로 등록되어 있으면 빈 오브젝트가 생성될 때마다 빈 후처리기에 보내서 후처리 작업을 요청한다. 빈 후처리기는 빈 오브젝트의 프로퍼티를 강제로 수정할 수도 있고 별도의 초기화 작업을 수행할 수도 있다. 심지어는 만들어진 빈 오브젝트 자체를 바꿔치기할 수도 있다. 따라서 스프링이 설정을 참고해서 만든 오브젝트가 아닌 다른 오브젝트를 빈으로 등록시키는 것이 가능하다.

이를 잘 활용하면 스프링이 생성하는 빈 오브젝트의 일부를 프록시로 포장하고, 프록시를 빈으로 대신 등록할 수도 있다.

DefaultAdvisorAutoProxyCreator 빈 후처리기가 등록되어 있으면 스프링은 빈 오브젝트를 만들 때마다 후처리기에게 빈을 보낸다. DefaultAdvisorAutoProxyCreator는 빈으로 등록된 모든 어드바이저 내의 포인트컷을 이용해 전달받은 빈이 프록시 적용 대상인지 확인한다. 프록시 적용 대상이면 그때는 내장된 프록시 생성기에게 현재 빈에 대한 프록시를 만들게 하고, 만들어진 프록시에 어드바이저를 연결해준다. 그리고 프록시 오브젝트를 스프링 컨테이너에게 돌려준다.

적용할 빈을 선정하는 로직이 추가된 포인트컷이 담긴 어드바이저를 등록하고 빈 후처리기를 사용하면 일일이 ProxyFactoryBean 빈을 등로갛지 않아도 타깃 오브젝트에 자동으로 프록시가 적용되게 할 수 있다.

# 확장된 포인트컷

그런데 한 가지 이상한 점이 있다. 지금까지 포인트컷이란 타깃 오브젝트의 메소드 중에서 어떤 메소드에 부가기능을 적용할지를 선정해주는 역할을 한다고 했다. 그런데 여기서는 갑자기 포인트컷이 등록된 빈 중에서 어떤 빈에 프록시를 적용할지를 선택한다는 식으로 설명하고 있다. 

사실 포인트컷은 두 가지 기능을 모두 가지고 있다. 포인트컷은 클래스 필터와 메소드 매처 두 가지를 돌려주는 메소드를 가지고 있다.

```java
public interface Pointcut {
	ClassFilter getClassFilter();   // 프록시를 적용할 클래스인지 확인
	MethodMatcher getMethodMatcher();  // 어드바이스를 적용할 메소드인지 확인
}
```

지금까지는 Method를 선별하는 기능만 사용해 온 것이다. 메소드만 선별한다는 건 클래스 필터는 모든 클래스를 다 받아주도록 만들어져 있다는 뜻이다. 따라서 클래스의 종류는 상관없이 메소드만 판별한다.

만약 Pointcut 선정 기준을 모두 적용한다면 먼저 프록시를 적용할 클래스인지 판단하고 나서, 적용 대상 클래스인 경우에는 어드바이스를 적용할 메소드인지 확인하는 식으로 동작한다. 클래스 자체가 프록시 적용 대상이 아니라면 어드바이스를 통한 부가기능 부여는 물 건너간 셈이다. 결국 이 두 가지 조건이 모두 충족되는 타깃의 메소드에 어드바이스가 적용되는 것이다.

ProxyFactoryBean에서는 굳이 클래스 레벨의 필터는 필요 없었지만, 모든 빈에 대해 프록시 자동 적용 대상을 선별해야 하는 빈 후처리기인 DefaultAdvisorAutoProxyCreator는 클래스와 메소드 선정 알고리즘을 모두 갖고 있는 포인트컷이 필요하다. 정확히는 그런 포인트컷과 어드바이스가 결합되어 있는 어드바이저가 등록되어 있어야 한다.

# 6.5.2 DefaultAdvisorAutoProxyCreator의 적용

# 클래스 필터를 적용한 포인트컷 작성

NameMatchMethodPointcut을 상속해서 프로퍼티로 주어진 이름 패턴을 가지고 클래스 이름을 비교하는 ClassFilter를 추가하도록 만들 것이다.

```java
public class NameMatchClassMethodPointcut extends NameMatchMethodPointcut {
	public void setMappedClassName(String mappedClassName) {
		this.setClassFilter(new SimpleClassFilter(mappedClassName));
	}

	static class SimpleClassFilter implements ClassFilter {
		String mappedName;
		
		private SimpleClassFilter(String mappedName) {
			this.mappedName = mappedName;
		}

		public boolean matches(Class<?> clazz) {
			return PatternMatchUtils.simpleMatch(mappedName, class.getSimpleName()); // *가 들어간 문자열 비교를 지원하는 스프링의 유틸리티 메소드
		}
	}

}
```

# 어드바이저를 이용하는 자동 프록시 생성기 등록

```java
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator" />
```

이 빈 정의에는 특이하게도 id 애트리뷰트가 없고 class뿐이다. 다른 빈에서 참조되거나 코드에서 빈 이름으로 조회될 필요가 없는 빈이라면 아이디를 등록하지 않아도 무방하다.

# 포인트컷 등록

기존의 포인트컷 설정을 삭제하고 새로 만든 클래스 필터 지원 포인트컷을 빈으로 등록한다.

```java
<bean id="transactionPointcut" class="springbook.service.NameMatchClassMethodPointcut">
	<property name="mappedClassName" value="*ServiceImpl"/>
	<property name="mappedName" value="upgrade*"/>
</bean>
```

# 어드바이스와 어드바이저

어드바이스인 transactionAdvice 빈의 설정은 수정할 게 없다. 어드바이저인 transactionAdvisor 빈드 수정할 필요는 없다. 하지만 어드바이저로서 사용되는 방법이 바뀌었다는 사실은 기억해두자. 이제는 ProxyFactoryBean으로 등록한 빈에서처럼 transactionAdvisor를 명시적으로 DI하는 빈은 존재하지 않는다. 대신 어드바이저를 이용하는 자동 프록시 생성기인 DefaultAdvisorAutoProxyCreator에 의해 자동수집되고, 프록시 대상 선정 과정에 참여하며, 자동생성된 프록시에 다이내믹하게 DI돼서 동작하는 어드바이저가 된다.

# ProxyFactoryBean 제거와 서비스 빈의 원상복구

프록시를 도입했던 때부터 아이디를 바꾸고 프록시에 DI 돼서 간접적으로 사용돼야 했던 userServiceImpl 빈의 아이디를 이제는 당당하게 userService로 되돌려놓을 수 있다. 더 이상 명시적인 프록시 팩토리 빈을 등록하지 않기 때문이다. 남았ㄷ너 ProxyFactoryBean 타입의 빈은 삭제해도 좋다. UserService와 관련된 빈 설정은 이제 userService 빈 하나로 충분하다

```java
<bean id="userService" class="springbook.service.UserServiceImpl">
	<property name="userDao" ref="userDao"/>
	<property name="mailSender" ref="mailSender"/>
</bean>
```

# 자동 프록시 생성기를 사용하는 테스트

스프링 컨테이너에 종속적인 기법을 사용했기 때문에, 예외상황을 위한 테스트 대상도 빈으로 등록해줄 필요가 있다.

```java
static class TestUserServiceImpl extends UserServiceImpl {
	private String id="madnite1";
	
	protected void upgradeLevel(User user) {
		if (user.getId().equals(this.id)) throw new TestUserServiceException();
		super.upgradeLevel(user);
	}
}
```

```java
<bean id="testUserService" class="springbook.user.service.UserServiceTest$TestUserServiceImpl" parent="userService" />
```

스태틱 멤버 클래스는 $로 지정한다. 

parent를 이용해 프로퍼티 정의를 포함해서 설정을 상속받는다.

- 테스트코드

```java
public class UserServiceTest {
	@Autowired UserService userService;
	@Autowired UserService testUserService; -> 필드 이름 기준으로 주입될 빈이 결정된다.

	@Test
	public void upgradeAllOrNothing() {
		userDao.deleteAll();
		for(User user: users) userDao.add(user);

		try {
			this.testUserService.upgradeLevels();
			fail("TestUserServiceException expected");
		} catch(TestUserServiceException e) {

		}
		
		checkLevelUpgraded(users.get(1), false);
	}

}
```

# 자동생성 프록시 확인

지금까지 트랜잭션 어드바이스를 적용한 프록시 자동생성기를 빈 후처리기 메커니즘을 통해 적용했다. 최소한 두 가지는 확인해야 한다.

첫째는 트랜잭션이 필요한 빈에 트랜잭션 부가기능이 적용됐는가이다. 트랜잭션이 정상적으로 커밋되는 경우에는 트랜잭션이 롤백되게 함으로써 트랜잭션 적용 여부를 테스트해야 한다. 이는 앞에서 만든 upgradeAllOrNothing() 테스트를 통해 검증했다. 

둘째는 아무 빈에나 트랜잭션 부가기능이 적용된 것은 아닌지 확인해야 한다. 포인트컷  빈의 이름 패턴을 변경해서 이번엔 testUserService빈에 트랜잭션이 적용되지 않게 해보자. 

또 다른 방법으로 자동생성된 프록시를 확인할 수 있다.

DefaultAdvisorAutoProxyCreator에 의해 userService 빈이 프록시로 바꿔치기됐다면 getBean(”userService”)로 가져온 오브젝트는 TestUserService 타입이 아니라 JDK의 Proxy타입일 것이다. 모든 JDK 다이내믹 프록시 방식으로 만들어지는 프록시는 Proxy 클래스의 서브클래스이기 때문이다.

```java
@Test
public void advisorAutoProxyCreator() {
	assertThat(testUserService, is(java.lang.reflect.Proxy.class));
}
```