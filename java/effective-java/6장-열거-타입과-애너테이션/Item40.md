# Item 40. @Override 애너테이션을 일관되게 사용하라

자바가 기본으로 제공하는 애너테이션 중 보통의 프로그래머에게 가장 중요한 것은 `@Override` 일 것이다. `@Override` 는 메서드 선언에만 달 수 있으며, 이 애너테이션이 달렸다는 것은 상위 타입의 메서드를 재정의했음을 뜻한다. 이 애너테이션을 일관되게 사용하면 여러가지 악명 높은 버그들을 예방해준다. 다음의 Bigram 프로그램을 살펴보자. 이 클래스는 바이그램, 즉 여기서는 영어 알파벳 2개로 구성된 문자열을 표현한다.

* 영어 알파벳 2개로 구성된 문자열을 표현하는 클래스 - 버그를 찾아보자.

```java
public class Bigram {
	private final char first;
	private final char second;
	
	public Bigram(char first, char second) {
		this.first = first;
		this.second = second;
	}
	
	public boolean equals(Biagram b) {
		return b.first == first && b.second == second;
	}
	
	public int hashCode() {
		return 31 * first + second;
	}
	
	public static void main(String[] args) {
		Set<Bigram> s = new HashSet<>();
		for (int i = 0; i < 10; i++) 
			for (char ch = 'a'; ch <= 'z'; ch++) 
				s.add(new Bigram(ch, ch));
		System.out.println(s.size());	
	}
}
```

main 메서드를 보면 똑같이 소문자 2개로 구성된 바이그램 26개를 10번 반복해 집합에 추가한 다음, 그 집합의 크기를 출력한다. Set 은 중복을 허용하지 않으니 26이 출력될 거 같지만, 실제로는 260이 출력된다. 무언가 잘못됐다!

확실히 Bigram 작성자는 equals 메서드를 재정의하려 한 것으로 보이고 hashCode 도 함께 재정의해야 한다는 사실을 잊지 않았다. 그런데 안타깝게도 equals 를 '재정의(overriding)' 한게 아니라 '다중정의(overloading)' 해버렸다. Object 의 equals 를 재정의하려면 매개변수 타입을 Object 로 해야만 하는데, 그렇게 하지 않은 것이다. 그래서 Object 에서 상속한 equals 와는 별개인 equals 를 새로 정의한 꼴이 되었다. Object 의 equals 는 == 연산자와 똑같이 객체 식별성 (identity) 만을 확인한다. 따라서 같은 소문자를 소유한 바이그램 10개 각각이 서로 다른 객체로 인식되고, 결국은 260을 출력한 것이다.

다행히 이 오류는 컴파일러가 찾아낼 수 있지만, 그러려면 Object.equals 를 재정의한다는 의도를 명시해야 한다. 다음 코드처럼 말이다.

```java
@Override public boolean equals(Bigram b) {
	return b.first == first && b.second == second;
}
```

이처럼 `@Override` 애너테이션을 달고 다시 컴파일하면 다음의 컴파일 오류가 발생한다.

```java
Bigram.java:10 method does not override or implement a method
from a supertype
	@Override public boolean equals(Bigram b)
	^
```

잘못된 부분을 명확히 알려주므로 곧장 올바르게 수정할 수 있다.

```java
@Override public boolean equals(Object o) {
	if(!(o instanceof Bigram))
		return false;
	Bigram b = (Bigram) o;
	return b.first == first && b.second == second;
}
```

그러니 **상위 클래스의 메서드를 재정의하려는 모든 메서드에 `@Override` 애너테이션을 달자.** 예외는 한 가지뿐이다. 구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때는 굳이 `@Override` 를 달지 않아도 된다. 구체 클래스인데 아직 구현하지 않은 추상 메서드가 남아 있다면 컴파일러가 그 사실을 바로 알려주기 때문이다. 물론 재정의 메서드 모두에 `@Override` 를 일괄로 붙여두는게 좋아 보인다면 그래도 상관없다. 또한 대부분의 IDE 는 재정의할 메서드를 선택하면 `@Override` 를 자동으로 붙여주니 참고하자.

한편, IDE 는 `@Override` 를 일관되게 사용하도록 부추기기도 한다. IDE 에서 관련 설정을 활성화해놓으면 `@Override` 가 달려있지 않은 메서드가 실제로는 재정의를 했다면 경고를 준다. `@Override` 를 일관되게 사용한다면 이처럼 실수로 재정의했을 때 경고해줄 것이다. 재정의할 의도였으나 실수로 새로운 메서드를 추가했을 때 알려주는 컴파일 오류의 보완재 역할로 보면 되겠다. IDE 와 컴파일러 덕분에 우리는 의도한 재정의만 정확하게 해낼 수 있는 것이다.

`@Override` 는 클래스뿐 아니라 인터페이스의 메서드를 재정의할 때도 사용할 수 있다. 디폴트 메서드를 지원하기 시작하면서, 인터페이스 메서드를 구현한 메서드에도 `@Override` 를 다는 습관을 들이면 시그니처가 올바른지 재차 확신할 수 있다. 구현하려는 인터페이스에 디폴트 메서드가 없음을 안다면 이를 구현한 메서드에서는 `@Override` 를 생략해 코드를 조금 더 깔끔히 유지해도 좋다. 하지만 추상 클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 `@Override` 를 다는 것이 좋다. 상위 클래스가 구체 클래스든 추상 클래스든 마찬가지다. 예컨대 Set 인터페이스는 Collection 인터페이스를 확장했지만 새로 추가한 메서드는 없다. 따라서 모든 메서드 선언에 `@Override` 를 달아 실수로 추가한 메서드가 없을을 보장했다.

> 핵심 정리
>
> 재정의한 모든 메서드에 `@Override` 애너테이션을 의식적으로 달면 여러분이 실수했을 때 컴파일러가 바로 알려줄 것이다. 예외는 한 가지뿐이다. 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우엔 이 애너테이션을 달지 않아도 된다. (단다고 해서 해로울 것도 없다.)