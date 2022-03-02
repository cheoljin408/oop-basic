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
