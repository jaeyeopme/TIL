# Item 63. 문자열 연결은 느리니 주의하라

문자열 연결 연산자 (+) 는 여러 문자열을 하나로 합쳐주는 편리한 수단이다. 그런데 한 줄짜리 출력값 혹은 작고 크기가 고정된 객체의 문자열 표현을 만들 때라면 괜찮지만, 본격적으로 사용하기 시작하면 **성능 저하**를 감내하기 어렵다. 문자열 연결 연산자로 문자열 n 개를 잇는 시간은 `n^2` 에 비례한다. 문자열은 **불변** (String literal - String constant pool(Heap), new - Heap 별도의 객체를 가리킴) 이라서 값이 변경되지 않기 때문에 여러 스레드가 데이터를 공유하더라도 **동기화를 신경 쓸 필요가 없어 안정성이 유지되는 장점**이 있다. **연산이 적게 사용되고, 문자열 값의 수정 없이 읽기가 많은 경우**에는 String 클래스의 사용이 더 적절하다. 하지만, 두 문자열을 연결할 경우 연산이 수행될 때마다 두 문자열을 모두 읽어 들이고 새로운 메모리에 복사해야하므로 성능 저하는 피할 수 없는 결과다. 

예를 들어 다음 메서드는 청구서의 품목 (item) 을 전부 하나의 문자열로 연결해준다.

```java
public String statement() {
	String result = "";
	for (int i = 0; i < numItems(); i++) 
		result += lineForItem(i); // 문자열 연결
	return result;
}
```

품목이 많은 경우 이 메서드는 심각하게 느려질 수 있다. 성능을 포기하고 싶지 않다면 **String 대신 StringBuilder 를 사용하자.** StringBuffer, StringBuilder 에서도 마찬가지로 문자열 복사를 하긴 하지만 가변 크기 배열을 이용해서 필요한 경우에만 문자열을 복사한다.  StringBuffer, StringBuilder 에서는 `+` 연산자 대신 `append()` 라는 함수를 사용한다. value 에 사용되지 않고 남아있는 공간에 새로운 문자열이 들어갈 정도의 크기가 있다면 그대로 삽입하고, 그렇지 않다면 value 배열의 크기를 **두배로 증가**시키면서 기존의 문자열을 복사하고 새로운 문자열을 삽입한다. StringBuffer와 StringBuilder는 문자열의 더하더라도 매번 문자열을 복사할 필요가 없어서 성능을 높일 수 있다.

```java
public String statement2() {
	StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
	for (int i = 0; i < numItems(); i++)
		b.append(lineForItem(i));
  return b.toString();
}
```

자바 6 이후 문자열 연결 성능을 다방면으로 개선했지만, 이 두 메서드의 성능 차이는 여전히 크다. 품목을 100개로 하고 lineForItem 이 길이 80인 문자열을 반환하게 하여 내 컴퓨터에서 실행해보니 statement2가 6.5배나 빨랐다. statement 메서드의 수행 시간은 품목 수의 제곱이 비례해 늘어나고 statement2는 선형으로 늘어나므로, 품목 수가 늘어날수록 성능 격차도 점점 벌어질 것이다. statement2에서 StringBuilder 를 전체 결과를 담기에 충분한 크기로 초기화한 점을 잊지 말자. 하지만 기본 값을 사용하더라도 여전히 5.5배나 빨랐다.

> 핵심 정리
>
> 원칙은 간단하다. 성능에 신경써야 한다면 **많은 문자열을 연결할 때는 문자열 연결 연산자(+) 를 피하자.** 대신 StringBuilder 의 append 메서드를 사용하라. 문자 배열을 사용하거나, 문자열을 (연결하지 않고) 하나씩 처리하는 방법도 있다.