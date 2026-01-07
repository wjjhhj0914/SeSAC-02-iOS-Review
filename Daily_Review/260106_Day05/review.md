# Day 05 수업 내용 복습

## 1. 프로퍼티와 메서드

* 프로퍼티(Property): 클래스(class)나 구조체(struct)라는 큰 박스 안에 선언된 **변수**나 **상수**를 부르는 이름이다. 역할은 변수와 같지만, "어디에 속해 있느냐"에 따라 이름이 바뀐다.
  * 예: `@IBOutlet var mySwitch: UISwitch!`에서 `mySwitch`는 이 클래스의 프로퍼티다.
* 메서드(Method): 클래스나 구조체 안에 들어있는 **함수**를 부르는 이름이다.

## 2. UI 구성 및 라이프사이클 (UISwitch & UITextField)

### UISwitch 스타일 적용의 비밀

* **onTintColor**: 스위치가 켜졌을 때의 배경색. `viewDidLoad`에서는 잘 적용되었음.
* `thumbTintColor`: 스위치의 동그란 버튼 색상.
  * 문제 상황: `viewDidLoad`에서 `thumbTintColor`를 설정해도 적용되지 않는 경우가 있음.
  * 해결책: `viewDidAppear` 시점에서 설정하면 적용됨.
    * 이유: 실행 시점의 차이다. view가 메모리에 로드된 직후(`viewDidLoad`)에는 아직 그려지지 않은 속성이 있을 수 있다. 지금은 "나중에 적용해야 하는 속성도 있다"는 점과 "뷰 라이프사이클(Life Cycle)의 중요성을 기억하면 된다.

### TextField Placeholder 색상 바꾸기: attributedPlaceholder 사용

* NS(Next Step): Objective-C 시절부터 사용되던 아주 오래된 접두어임.
* NSAttributedString: 단순히 텍스트만 넣는 게 아니라 색상, 폰트 등 "속성"을 같이 줄 때 사용한다.

```Swift
myTextField.attributedPlaceholder = NSAttributedString(string: "아이디를 입력하세요", attributes: [.foregroundColor: UIColor.red])
```

## 3. 효율적인 UI 관리 (Tag & Outlet Collection)

### View Tag 활용 (반복 줄이기)

* 여러 버튼이 하나의 `@IBAction`을 공유할 때, 어떤 버튼이 눌렸는지 구분하기 위해 `tag`를 사용한다.
* 기본값은 모두 `0`이므로 스토리보드나 `viewDidLoad`에서 각각 고유한 번호를 부여해야 한다.

```Swift
var emotion = ["행복해", "우울해", "신기해"] // 프로퍼티 배열
var count = [0, 0, 0]
@IBAction func emotionButtonClicked(_ sender: UIButton) {
  count[sender.tag] = count[sender.tag] + 1
  print("\(emotion[sender.tag]) \(count)")
}
```

### Outlet Collection

* 동일한 성격의 UI 요소(예: 여러 개의 UILabel)를 하나의 **배열** 형태로 묶어서 관리하는 기능이다. 반복문을 돌리거나 인덱스로 접근하기 매우 편리하다.

## 4. 아키텍처 및 협업 (Storyboard Reference)

* Storyboard Reference: 너무 커진 스토리보드를 여러 개로 나누어 관리하는 기능.
* Entry Point 에러: 나눈 스토리보드에 시작점(Is Initial View Controller)이 지정되지 않으면 빨간 에러 뜸.
* 시작점 설정: 앱의 전체 시작점은 `Info.plist`의 `Storyboard Name`에서 결정하며, 스토리보드 내부의 시작점은 화면 전환의 출발지를 의미함.

## 5. 최신 트렌드: Swift 6 `InlineArray`

이거는 한번 찾아보는 게 좋음.

## 옵셔널(Optional) 중요

### 왜 `Optional("")`이 뜰까?

* `String?` 타입은 `nil`일 수도 있기 때문이다.
* 문자열 보간법(`\()`): 이 안에는 반드시 값이 있어야 하므로 `nil`일 가능성이 있는 옵셔널 타입을 넣으면 경고가 뜨고, 출력 시 `Optional`이라는 글자가 붙는다.

### 옵셔널 처리 방법

1. Forced Unwrapping(`!`): "절대 nil이 아님을 내가 보장해!" (위험하지만 확실할 때 사용)
2. Nil Coalescing(`??`): "만약 nil이면 이 기본값을 대신 써줘."
  * `print(greeting ?? "다시 입력해 주세요")
3. Optional Binding(`if let`): "값이 있으면 변수에 담아서 쓰고, 없으면 안전하게 넘어가자." (가장 권장됨)

```Swift
if let userText = ageTextField.text, let age = Int(userText) {
  print("당신은 \(age)살입니다.")
} else {
  print("숫자를 입력해 주세요.")
}
```

#### 주의사항

* 텍스트필드에 아무것도 안 써도 `nil`이 아니라 `""`(빈 문자열)이 들어온다. 애플의 기본 세팅이다.
* `Int("안녕")`처럼 형변환에 실패하면 `nil`이 반환된다.

### guard let

guard let은 특정 조건이 맞지 않으면 아래 코드를 더 이상 실행하지 않고 **즉시 종료**시키고 싶을 때 사용한다.

#### guard let의 특징

* Early Exit: 값이 `nil`이라면 `else` 블록을 실행하고 함수를 즉시 끝내버린다.
* else문 필수: 조건이 맞지 않을 때 어떻게 나갈지(`return`, `break` 등)를 반드시 적어줘야 한다.
* 넓은 스코프: `guard` 문을 통과해 언래핑된 변수는 함수가 끝날 때까지 아래 전체 코드에서 자유롭게 사용할 수 있다.
