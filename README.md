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
