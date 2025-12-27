# Swift의 변수와 상수

## 1. 변수 vs 상수

Swift에서는 값을 저장할 때 `var`와 `let` 키워드를 사용한다.

### var

- 변수
- 값을 나중에 **변경할 수 있음** (Mutable)
- JavaScript의 let과 var를 떠올리자

### let

- 상수
- 값을 한 번 정하면 **변경할 수 없음** (Immutable)
- JavaScript의 const를 떠올리자

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