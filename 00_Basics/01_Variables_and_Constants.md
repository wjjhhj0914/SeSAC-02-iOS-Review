# Swift의 변수와 상수

## 1. 변수 vs 상수

Swift에서는 값을 저장할 때 `var`와 `let` 키워드를 사용한다.

### var

* 변수
* 값을 나중에 **변경할 수 있음** (Mutable)
* JavaScript의 let과 var를 떠올리자

### let

* 상수
* 값을 한 번 정하면 **변경할 수 없음** (Immutable)
* JavaScript의 const를 떠올리자

## 예시 코드

```Swift
// 변수: 선언 후 값 변경 가능
var userAge = 25
userAge = 26 // OK!

// 상수: 선언 후 값 변경 시 컴파일 에러 발생
let userName = "Hyojung"
// userName = "Junghyo" // Error
```

Swift는 성능 최적화와 코드의 안정성을 위해 가급적 **모든 것을 `let`으로 선언**하고, 꼭 필요한 경우에만 `var`로 바꿀 것을 강력히 권장한다고 함.

# Swift의 데이터 타입

Swift는 정적 타이핑(Static Typing) 언어다.

TypeScript처럼 변수명 뒤에 콜론(`:`)을 붙여 타입을 명시하거나 초기값을 통해 타입을 추론한다.

> **정적 타이핑이란?**<br />
>
> 컴파일 시점에 변수의 타입이 결정되는 방식을 말한다.<br />
> TypeScript와 유사하지만, 실행(Runtime) 시가 아닌 **코드를 빌드하는 시점에 모든 타입 에러를 잡아내므로** 타입 안정성(Type Safety)이 매우 높다.

* **엄격한 타입 구분**: 정수(`Int`)와 실수(`Double`)조차 아예 다른 타입으로 간주하며, 자동 형변환을 허용하지 않는다.
* **타입 추론(Type Inference)**: 타입을 명시하지 않아도 초기값을 통해 타입을 자동으로 결정한다.

## 예시 코드

```Swift
var score = 1000 // Int 타입으로 추론됨
// score = "천" // Compile Error
```

# Swift의 함수

Swift의 함수는 다른 언어와 달리 '문장처럼 읽히는 독특한 문법(Argument Label)'이라는 것이 있다고 한다. 이 부분을 잘 정리해두면 나중에 애플 공식 문서를 읽을 때 도움이 된다.

* `func` 키워드 사용
* 가독성 극대화를 위해 **전달인자 레이블**(Argument Label)이라는 독특한 기능 제공

## 1. 기본 구조

반환 타입 앞에 `->`(화살표) 사용

```Swift
func 함수이름(파라미터이름: 타입) -> 반환타입 {
  return 결과값
}
```

## 2. Swift만의 특징: Argument Label

함수를 호출할 때 쓰는 이름과 함수 내부에서 쓰는 이름을 다르게 할 수 있다.<br />
코드를 영어 문장처럼 자연스럽게 읽히게 하기 위함이다.

```Swift
// 'to'는 호출할 때 사용(Argument Label), 'name'은 함수 내부에서 사용(Parameter Name)
func sayHello(to name: String) {
  print("Hello, \(name)!")
}

sayHello(to: "Hyojung") // 호출 시: "Say Hello to Hyojung"처럼 읽힘
```

* 와일드카드 패턴(`_`): 호출할 때 파라미터 이름을 아예 생략하고 싶다면 `_`를 사용

```Swift
func add(_ a: Int, _ b: Int) -> Int {
  return a + b
}

let sum = add(5, 10) // 이름을 생략하고 값만 전달
```

## 3. 반환값이 없는 함수

반환값이 없을 때는 `-> Void`를 생략하거나 명시할 수 있다.

### (1) `-> Void`를 명시한 예시

```Swift
func sayHello(to name: String) -> Void {
  print("Hello, \(name)!")
}
```

### (2) 빈 괄호 `()`를 사용한 예시

Swift에서 `Void`는 사실 빈 튜플`()`의 별칭이다. 그래서 이렇게 쓸 수도 있다.

```Swift
func sayHello(to name: String) -> () {
  print("Hello, \(name)!")
}
```

### (3) 모두 생략한 예시
보통의 경우, 코드를 간결하게 만들기 위해 생략하는 것을 가장 권장한다.

```Swift
func sayHello(to name: String) {
  print("Hello, \(name)!")
}
```

> #### 왜 굳이 `-> Void`를 명시하기도 하는 걸까?
>
> 평소에는 생략하는 것이 국룰이지만, 가끔 명시하는 경우가 있다.
>
> 1. **가독성을 위해**: 함수가 복잡할 때 "이 함수는 return 값이 확실히 없다"는 것을 명확히 보여주고 싶을 때 사용.
> 2. **클로저를 다룰 때**: 나중에 배우게 될 클로저에서는 타입을 정확히 맞춰줘야 할 때가 있어서 `-> Void`라고 꼭 써줘야 하는 상황이 생긴다.
