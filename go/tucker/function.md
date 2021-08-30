# 7. 함수

## 7.1 함수 정의

함수는 함수 키워드, 함수명, 매개변수, 반환 타입, 함수 코드 블록으로 구성된다.

func - 함수 정의 키워드 Add - 함수명 a int, b int - 매개변수 int - 반환 타입

```go
func Add(a int, b int) int {
// a 와 b 를 더한 값을 반환한다.
return a + b
}
```

함수명의 명명 규칙은 변수명과 같다. **첫 글자가 대문자인 함수는 패키지 외부로 공개되는 함수이다.**

```go
package main

import "fmt"

func Add(a int, b int) int {
	return a + b
}

func main() {
	c := Add(3, 6)
	fmt.Println(c)
}
```

## 7.2 함수를 호출하면 생기는 일

함수를 호출할 때 입력하는 값을 argument라고 한다. 아규먼트 혹은 인수라고 한다. 반명 함수가 외부로부터 입력 받는 변수를 parameter라고 한다. 매개변수 혹은 파라미터라고 한다.

함수를 호출하며 입력한 값은 실제 함수에 어떻게 전달될까? 보낸 값을 그대로 사용하는 것이 아니라 값을 복사해 사용한다.

```go
package main

import "fmt"

func Add(a int, b int) int { // 매개변수 복사
	return a + b // 값 반환 (c에 대입 복사)
}

func main() {
	c := Add(3, 6) // 함수 호출
	fmt.Println(c)
}
```

> 핵심 포인트
>
> **인수는 매개변수로 복사된다.**
>
> **매개변수와 함수 내에서 선언된 변수는 함수가 종료되면 변수 범위를 벗어나서 접근하지 못한다.**

## 7.3 함수는 왜 쓰나?

자주 사용되거나 변경 가능성이 있는 코드 블록을 묶어서 함수로 만들면 효율적으로 코딩할 수 있고 추후 프로그램 변경 요구에도 간단히 대처할 수 있다. 또 관련된 코드를 묶어서 이름을 부여하기 때문에 코드를 읽기에도
훨씬 편하다.

## 7.3.1 멀티 반환 함수

함수는 값을 여러 개 반환할 수 있다. 반환 값이 여럿일 때는 반환 타입들을 소괄호로 묶어서 표현한다.

```go
package main

import "fmt"

func Divide(a, b int) (int, bool) { // 함수 선언
	if b == 0 {
		return 0, false // 제수가 0일 때 반환
	}

	return a / b, true // 제수가 0이 아닐 때 반환
}

func main() {
	c, success := Divide(9, 3) // 제수가 0이 아닌 경우
	fmt.Println(c, success)
	d, success := Divide(9, 0) // 제수가 0인 경우
	fmt.Println(d, success)
}
```

## 7.3.2 변수명을 지정해 반환하기

함수 선언부에 반환 타입을 적을 때 변수명까지 적어주면, return문으로 해당 변수를 명시적으로 반환하지 않아도 값을 반환할 수 있다.

```go
package main

import "fmt"

func Divide(a, b int) (result int, success bool) { // 반환할 함수 명시
	if b == 0 {
		result = 0
		success = false
		return // 명시적으로 반환할 값을 지정하지 않은 return문
	}
	result = a / b
	success = true
	return
}

func main() {
	c, success := Divide(9, 3) // 제수가 0이 아닌 경우
	fmt.Println(c, success)
	d, success := Divide(9, 0) // 제수가 0인 경우
	fmt.Println(d, success)
}
```

함수 결과를 반환할 때 명시적으로 result, success를 지정하지 않았지만 두 값이 반환된다.

* 반환할 변수에 이름을 지정할 경우 모든 반환 변수에 이름을 지정해야 한다. 모두 지정하거나, 모두 지정하지 않거나 둘 중 하나여야 한다.

## 7.4 재귀 호출

재귀호출이란 함수 안에서 자기 자신 함수를 다시 호출하는 것을 말한다.

```go
package main

import "fmt"

func printNo(n int) {
	if n == 0 { // 재귀 탈출 조건
		return
	}
	fmt.Println(n)
	printNo(n - 1)          // 재귀 호출
	fmt.Println("After", n) // 재귀 호출 이후 출력
}

func main() {
	printNo(3) // 함수 호출
}
```

3, 2, 1, After 1, After 2, After 3

호출 순서를 보면 printNo(3)이 먼저 호출되고 이어서 printNo(2), printNo(1), printNo(0) 순으로 호출된다. 최종적으로 printNo(0)이 호출됐을 때 탈줄 조건인 n==0 을
만족해 return 이 되어 종료된다. 재귀 호출된 printNo() 함수가 종료되면 호출자인 printNo()의 위치로 반환되고 거기에서 명령을 수행해서 "After "메시지가 출력된다.

printNo() 함수 내에서 printNo(n-1)을 호출하면, main() 함수에서 printNo(3)을 호출한 뒤 printNo(2), printNo(1), printNo(0)을 차례로 호출한다.
printNo(0)에서 재귀 호출 탈출 조건 (n==0)을 만족하므로 자신을 호출한 위치로 차례대로 연속해서 돌아가게 되어 최종적으로 최초 호출했던 main() 함수 위치로 돌아간다.

> 재귀 호출을 사용할 때는 항상 탈출 조건을 정해야 한다.
>
> 재귀 호출이 종료되는 시점을 명확히하자. 그렇지 않으면 재귀 호출이 무한히 반복되어 비정상적으로 종료된다.

> 핵심 요약
> 1. 함수란 특수한 코드 묶음을 말한다. 함수를 만들면 코드를 재사용할 수 있다.
> 2. 함수 정의
> 3. 멀티 반환 함수는 값을 여러 개 반환할 수 있다. 반환 타입 자리에 소괄호로 여러 반환 타입을 묶어서 표시한다.
> 4. 재귀 호출은 함수 안에서 같은 함수를 호출하는 기법을 말한다. 재귀 호출 시에는 항상 탈출 조건을 명확히 해야 한다.