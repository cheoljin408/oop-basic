# oop-basic
객체지향 프로그래밍 기초 공부
---
# 캡슐화

- 데이터와 관련 기능을 묶기
- 객체가 기능을 어떻게 구현했는지 외부에 감추는 것
    - 구현에 사용된 데이터의 상세 내용을 외부에 감춤
- 정보 은닉 의미 포함
- 외부에 영향 없이 객체 내부 구현 변경 가능

캡슐화 하지 않으면

```java
if(acc.getMembership() == REGULAR && acc.getExpDate().isAfter(now())) {
	...정회원 기능
}
```

⬇️

```java
if(acc.getMembership() == REGULAR &&
	(
		(acc.getServiceData().isAfter(fiveYearAgo) && acc.getExpDate().isAfter(now())) ||
		(acc.getServiceData().isBefore(fiveYearAgo) && acc.addMonth(acc.getExp()).isAfter(now()))
	)
) {
	...정회원 기능
}
```

요구사항의 변화가 데이터 구조와 사용에 변화를 발생시킴(여러 곳의 코드를 전부 다 수정해야할 수도 있음)

캡슐화 하면

- 기능을 제공하고 구현 상세를 감춤

```java
if(acc.hasRegularPermission()) {
	...정회원 기능
}
```

```java
public class Account {
	private Membership membership;
	private Date expDate;

	public boolean hasRegularPermission() {
		return membership == REGULAR && expDate.isAfter(now())
	}
}
```

요구사항이 바뀐다면

- 내부 구현만 변경

```java
if(acc.hasRegularPermission()) {
	...정회원 기능
}
```

```java
public class Account {
	private Membership membership;
	private Date expDate;

	public boolean hasRegularPermission() {
		return membership == REGULAR && 
			(expDate.isAfter(now()) ||
				(
					serviceDate.isBefore(fiveYearAgo()) &&
					addMonth(expDate).isAfter(now())
				)
			);
	}
}
```

캡슐화된 기능을 사용하는 코드의 연쇄적인 변경(영향)을 최소화 한다.

- 캡슐화는 의도(비즈니스 요구사항)에 대한 이해를 높임. e.g. `hasRegularPermission()` 은 정회원에 대한 정책이구나...

캡슐화를 위한 규칙

- Tell, Don`t Ask 데이터를 달라고 하지 않고 해달라고 하기
- Demeter`s Law
    - 메소드에서 생성한 객체의 메소드만 호출
    - 파라미터로 받은 객체의 메소드만 호출
    - 필드로 참조하는 객체의 메소드만 호출

### 정리

- 캡슐화: 기능의 구현을 외부에 감춤
- 캠슐화를 통해 기능을 사용하는 코드에 영향을 주지 않고(또는 최소화) 내부 구현을 변경할 수 있는 유연함

## 캡슐화 연습1

캡슐과 전

```java
public AuthResult authenticate(String id, String pw) {
	Member mem = findOne(id);
	
	if(mem == null) return AuthResult.NO_MATCH;

	if(mem.getVerificationEmailStatus() != 2) {
		return AuthResult.NO_EMAIL_VERIFIED;
	}

	if(passwordEncoder.isPasswordValid(mem.getPassword(), pw, mem.getId())) {
		return AuthResult.SUCCESS;
	}
	
	return AuthResult.NO_MATCH;
}
```

캡슐화 후

```java
public AuthResult authenticate(String id, String pw) {
	Member mem = findOne(id);
	
	if(mem == null) return AuthResult.NO_MATCH;

	if(!mem.isEmailVerified()) {
		return AuthResult.NO_EMAIL_VERIFIED;
	}

	if(passwordEncoder.isPasswordValid(mem.getPassword(), pw, mem.getId())) {
		return AuthResult.SUCCESS;
	}
	
	return AuthResult.NO_MATCH;
}
```

```java
public class Member {
	private int verificationEmailStatus;

	public boolean isEmailVerified() {
		return verificationEmailStatus == 2;
	}
}
```

## 캡슐화 연습2

캡슐화 전

```java
public class Rental {
	private Movie movie;
	private int daysRented;

	public int getFrequentRenterPoints() {
		if(movie.getPriceCode() == Movie.NEW_RELEASE && daysRented > 1)
			return 2;
		else
			return 1;
	}
}
```

```java
public class Movie {
	public static int REGULAR = 0;
	public static int NEW_RELEASE = 1;
	public int priceCode;

	public int getPriceCode() {
		return priceCode;
	}
}
```

캡슐화 후

```java
public class Rental {
	private Movie movie;
	private int daysRented;

	public int getFrequentRenterPoints() {
		return movie.getFrequentRenterPoints(daysRented);
	}
}
```

```java
public class Movie {
	public static int REGULAR = 0;
	public static int NEW_RELEASE = 1;
	public int priceCode;

	public int getFrequentRenterPoints(int daysRented) {
		if(priceCode() == NEW_RELEASE && daysRented > 1)
			return 2;
		else
			return 1;
	}
}
```

## 캡슐화 연습3

캡슐화 전

```java
Timer t = new Timer();

t.startTime = System.currentTimeMillis();
...
t.stopTime = System.currentTimeMillis();

long elapsedTime = t.stopTime - t.startTime;
```

```java
public class Timer {
	public long startTime;
	public long stopTime
}
```

캡슐화 후

```java
Timer t = new Timer();

t.start();
...
t.stop();

long time = t.elapsedTime(MILLISECOND);
```

```java
public class Timer {
	private long startTime;
	private long stopTime;

	public void start() {
		this.startTime = System.currentTimeMillis();
	}

	public void stop() {
		this.stopTime = System.currentTimeMillis();
	}

	public long elapsedTime(TimemUnit unit) {
		switch(unit) {
			case MILLISECOND:
				return stopTime - startTime;
			...
		}
	}
}
```

## 캡슐화 연습4

캡슐화 전

```java
public void verifyEmail(String token) {
	Member mem = findByToken(token);
	
	if(mem == null) throw new BadTokenException();
	
	if(mem.getVerificationEmailStatus() == 2) {
		throw new AlreadyVerifiedException();
	} else {
		mem.setVerificationEmailStatus(2);
	}
	// ...수정사항 DB 반영
}
```

캡슐화 후

```java
public void verifyEmail(String token) {
	Member mem = findByToken(token);
	
	if(mem == null)
		throw new BadTokenException();

	mem.verifyEmail();

	// ...수정사항 DB 반영
}
```

```java
public class Member {
	private int verificationEmailStatus;

	public void verifyEmail() {
		if(isEmailVerified()) {
			throw new AlreadyVerifiedException();
		} else {
			this.verificationEmailStatus = 2;
		}
	}

	public boolean isEmailVerified() {
		return verificationEmailStatus == 2;
	}
}
```
## 다형성

- 여러 모습을 갖는 것
- 객체 지향에서는 한 객체가 여러 타입을 갖는 것
    - 즉 한 객체가 여러 타입의 기능을 제공
    - 타입 상속으로 다형성 구현

```java
public class Timer {
	public void start() {}
	public void stop() {}
}

public interface Rechargeable {
	void charge();
}
```

```java
public class IotTimer extends Timer implements Rechargeable {
	public void charge() {
		...
	}
}
```

```java
IotTimer it = new IotTimer();
it.start();
it.stop();

Timer t = it;
t.start();
t.stop();

Rechargeable r = it;
r.charge();
```

## 추상화

- 데이터나 프로세스 등을 의미가 비슷한 개념이나 의미있는 표현으로 정의하는 과정
- 두 가지 방식의 추상화
    - 특정한 성질
        
        DB의 USER 테이블: 아이디, 이름, 이메일
        
        Money 클래스: 통화, 금액
        
    - 공통 성질(일반화)
        
        프린터: HP MXXX, 삼성 SL-M2XXX
        
        GPU: 지포스, 라데온
        

## 서로 다른 구현 추상화

SCP로 파일 업로드, HTTP로 데이터 전송, DB 테이블 삽입 → 추상화 → 푸시 발송 요청

## 타입 추상화

- 여러 구현 클래스를 대표하는 상위 타입 도출
    - 흔히 인터페이스 타입으로 추상화
    - 추상화 타입과 구현은 타입 상속으로 연결

```java
												<<interface>>
													Notifier
															|
							+---------------+--------------+
							|               |              |
				EmailNotifier    SMSNotifier    KakaoNotifier
```

## 추상 타입 사용에 따른 이점: 유연함

- 콘크리트 클래스를 직접 사용하면
    
    ```java
    private SmsSender smsSender;
    
    public void cancel(String ono) {
    	...주문 취소 처리
    
    	smsSender.sendSms();
    }
    ```
    
    ```java
    private SmsSender smsSender;
    private KakaoPush kakaoPush;
    
    public void cancel(String ono) {
    	...주문 취소 처리
    
    	if(pushEnabled) {
    		kakaoPush.push();
    	} else {
    		smsSender.sendSms();
    	}
    }
    ```
    
    ```java
    private SmsSender smsSender;
    private KakaoPush kakaoPush;
    private MailService mailSvc;
    
    public void cancel(String ono) {
    	...주문 취소 처리
    
    	if(pushEnabled) {
    		kakaoPush.push();
    	} else {
    		smsSender.sendSms();
    	}
    	mailSvc.sendMail();
    }
    ```
    
    요구사항 변경에 따라 주문 취소 코드도 함께 변경
    
    주문 취소라는 본질 자체랑 상관없는 sms, kakao, mail 때문에 주문 취소를 구현한 cancel() 메소드의 코드가 함께 바뀐다.
    

- 추상화 하면

sms전송, 카카오톡보냄, 이메일 발송 → 추상화 → 통지(Notifier)

```java
public void cancel(String ono) {
	...주문 취소 처리
	Notifier notifier = NotifierFactory.instance().getNotifier();
	notifier.notify();
}

public interface NotifierFactory {
	Notifier getNotifier();

	static NotifierFactory instance() {
		return new DefaultNotifierFactory();
	}
}
	
public class DefaultNotifierFactory implements NotifierFactory {
	public Notifier getNotifier() {
		if(pushEnabled) return new KakaoNotifier();
		else return new SmsNotifier();
	}
}
```

## 상속과 재사용

- 상위 클래스의 기능을 재사용, 확장하는 방법으로 활용함..그러나

## 상속을 통한 기능 재사용시 발생할 수 있는 단점

- 상위 클래스 변경 어려움
- 클래스 증가
- 상속 오용

## 상속을 통한 재사용의 단점1

- 상위 클래스의 변경이 어려움
- 상위 클래스를 변경하면 변경의 여파가 계층도를 따라 하위 클래스로 전파 됨

## 상속을 통한 재사용의 단점2

## 상속의 단점 해결 방법 → 조립

![스크린샷 2022-03-01 오전 11.59.21.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a86d140b-a837-41ee-9ba7-0f5a6e48e62d/스크린샷_2022-03-01_오전_11.59.21.png)

- 상속을 이용한 확장을 통해 기능을 추가하는게 아니라 해당 기능을 제공하는 클래스를 만들어서 조립하여 사용
- 상속을 오용하는 문제도 크게 줄어듦

## 상속보다는 조립

- 상속하기에 앞서 조립으로 풀 수 없는지 검토
- 진짜 하위 타입인 경우에만 상속 사용
