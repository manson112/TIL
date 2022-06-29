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