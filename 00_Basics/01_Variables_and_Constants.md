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