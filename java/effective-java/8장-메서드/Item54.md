# Item 54. null 이 아닌, 빈 컬렉션이나 배열을 반환하라

## 방어 코드

컬렉션이나 배열같은 컨테이너가 비었을 때 null 을 반한하는 메서드를 사용할 때면 항시 방어 코드를 넣어줘야한다.

```java
List<Cheese> cheeses = shop.getCheeses();

if (cheeses != null && cheeses.contains(Cheese.STILTON))
	System.out.prinln("좋았어, 바로 그거야.")
```

## 비어있는 객체

빈 컨테이너를 할당하는 데도 비용이 드니 null 을 반환하는 쪽이 낫다는 주장도 있다. 하지만 이는 두가지 면에서 틀린 주장이다. 첫 번째, 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한 이 정도의 성능 차이는 신경 쓸 수준이 못된다. 두 번째, 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다. 사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨릴 수 있다. 

```java
public List<Cheese> getCheeses() {
  return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
```

## 비어있는 불변 객체

해법은 매번 똑같은 빈 **불변** 컬렉션을 반환하는 것이다. 불변 객체는 자유롭게 공유해도 안전하다. 배열을 쓸 때도 마찬가지다. 절대 null 을 반환하지 말고 길이가 0인 배열을 반환하라. 이 최적화 버전의 getCheeses 는 항상 EMPTY_CHEESE_ARRAY 를 인수로 넘겨 toArray 를 호출한다. 따라서 cheesesInStock 이 비었을 때면 언제나 EMPTY_CHEESE_ARRAY 를 반환하게 된다. 단순히 성능을 개선할 목적이라면 toArray 에 넘기는 배열을 미리 할당하는 건 추천하지 않는다. 오히려 성능이 떨어진다는 연구 결과도 있다.

```java
private final static Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheese() {
	return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

> 핵심 정리
>
> null 이 아닌, 빈 배열이나 컬렉션을 반환하라. null 을 반환하는 API 는 사용하기 어렵고 오류 처리 코드도 늘어난다. 그렇다고 성능이 좋은 것도 아니다.
