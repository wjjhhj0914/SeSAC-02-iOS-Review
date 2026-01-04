# Swift의 제어 흐름과 집단 자료형

## 1. 집단 자료형 (Collection Types)

Swift의 모든 컬렉션은 **타입 안정성**을 위해 동일한 타입의 데이터만 저장할 수 있다.

### ① Array(배열)

* 순서가 있는 데이터 리스트.
* 인덱스로 접근.

#### 코드 예시

```Swift
// 생성
var tasks: [String] = []

// 추가
tasks.append("Swift 기초 공부") // ← push와 같음

// 삽입
// 특정 위치에 데이터 삽입
tasks.insert("러닝", at: 0)

// 삭제
// 인덱스로 삭제
tasks.remove(at: 1)

// 확인
// 개수 및 빈 배열 여부 확인
tasks.count
tasks.isEmpty
```

### ② 딕셔너리(Dictionary)

* 키와 값의 쌍으로 이루어진 컬렉션.
* 특징은 키를 통해 값에 접근하면 결과는 항상 **Optional**임. 키가 없을 수도 있기 때문임.

#### 코드 예시

```Swift
var userInfo: [String: Int] = ["Age": 26]

print(userInfo["Age"] ?? 0) // 키가 없으면 0을 반환하도록 기본값 설정

userInfo["Score"] = 100 // 새로운 데이터 추가
userInfo.updateValue(95, forKey: "Score") // 값 수정
```

### ③ Set(집합)

* 순서가 없고, **중복을 허용하지 않는** 유일한 값들의 모임.

#### 코드 예시

```Swift
var tags: Set<String> = ["Swift", "iOS", "Swift"]
print(tags.count) // 결과: 2 (중복된 "Swift"는 하나만 저장됨)
```

## 2. 조건문 (Conditional Statements)

### if / else if / else

* JS와 다르게 소괄호 `()`는 생략 가능
* 중괄호 `{}`는 필수

```Swift
let temperature = 25
if temperature > 30 {
  print("에어컨 켜 주세요.")
} else if temperature < 5 {
  print("히터 켜 주세요.")
} else {
  print("쾌적한 날씨입니다.")
}
```

### switch 문

* Swift의 switch는 매우 강력, `break`를 명시하지 않아도 됨!
* **범위 연산자**와 결합하여 효율적으로 사용

```Swift
let score = 88

switch score {
  case 90...100:
  print("A")
  case 80..<90>:
  print("B")
  default: // default 안 쓰면 에러 발생 (모든 경우를 다 다뤄야 함)
  print("F")
}
```

## 3. 반복문 (Loop Statements)

### for in 문

* 배열, 딕셔너리, 범위를 순회할 때 사용

```Swift
// 범위 반복
for i in 1...3 {
  print("\(i)번째 반복 중...")
}

// 딕셔너리 순회 (튜플 활용)
let favouriteColours = ["Blue": "파랑", "Red": "빨강"]

for (eng, kor) in favouriteColours {
  print("\(eng)은 한국어로 \(kor)입니다.")
}
```

### while 문

* 조건이 `false`가 될 때까지 무한 반복

```Swift
var energy = 3

while energy > 0 {
  print("에너지 잔량: \(energy)")
  energy -= 1
}
```