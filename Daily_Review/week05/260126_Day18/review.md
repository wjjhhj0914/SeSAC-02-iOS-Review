# Day 18 - 수업 내용 복습 정리

## 1. 열거형(Enum) - 안전하고 직관적인 코드!

단순한 정수(`Int`)나 문자열(`String`) 대신 의미를 가진 Case를 사용하여 코드의 가독성과 안전성을 높인다.

### 주요 특징

- RawValue(원시값): 멤버에 `Int`, `String` 등의 값을 매칭할 수 있다.
  - `Int`를 채택하면 0부터 시작하는 index가 자동으로 부여되지만, 직접 지정도 가능하다.
- CaseIterable: `allCases`를 통해 모든 케이스를 배열로 만들어 준다. 테이블 뷰 섹션/행 처리에 매우 유용하다.
- Optional: 옵셔널 자체가 내부적으로는 `none`, `some(Wrapped)` 케이스를 가진 열거형으로 구현되어 있다.

### Enum 실전 활용 예시 코드

```Swift
import UIKit

// 1. 원시값(RawValue)과 CaseIterable 채택
enum SearchType: Int, CaseIterable {
    case movie = 0  // 0을 지정하면 아래는 자동으로 1, 2...
    case drama
    case book
    case shopping

    // 2. 연산 프로퍼티를 활용한 '데이터 매칭' (로직의 캡슐화)
    var navigationTitle: String {
        switch self {
        case .movie: return "영화를 검색해 보세요"
        case .drama: return "드라마를 검색해 보세요"
        case .book: return "책을 검색해 보세요"
        case .shopping: return "쇼핑 아이템을 검색해 보세요"
        }
    }

    var placeholder: String {
        return "\(self.navigationTitle) (입력 필수)"
    }
}

// 3. 실제 ViewController에서의 활용
class SearchViewController: UIViewController {

    var currentType: SearchType = .movie // 초기값 설정

    override func viewDidLoad() {
        super.viewDidLoad()

        // enum 덕분에 코드가 한 줄로 깔끔해짐!
        navigationItem.title = currentType.navigationTitle
    }

    // 버튼 클릭 시 타입 전환 예시
    @IBAction func typeChanged(_ sender: UIButton) {
        // rawValue를 이용해 안전하게 인스턴스 생성 (failable initializer)
        if let selectedType = SearchType(rawValue: sender.tag) {
            self.currentType = selectedType
            print("현재 모드: \(selectedType), 제목: \(selectedType.navigationTitle)")
        }
    }
}
```

## 2. 오버로딩(Overloading)

함수의 이름이 같아도 **매개변수의 타입**, **개수**, 또는 **반환값의 타입**이 다르면 다른 함수로 인식하는 특성이다.

```Swift
func hello() { }               // 1번
func hello(name: String) { }   // 2번: 매개변수 있음
func hello() -> String { }     // 3번: 반환값 있음
```

## 3. 일급 객체(First-class Object)

Swift에서 함수는 **일급 객체**로 취급된다. 이는 다음 세 가지가 가능함을 의미한다.

1. **변수/상수에 저장 가능**: `let a = hello`
2. **매개변수로 전달 가능**: 다른 함수의 인자로 함수를 보냄.
3. **반환값으로 사용 가능**: 함수가 함수를 반환함.

### 함수 타입(Function Types)

변수에 함수를 담을 때 생기는 타입이다.

- `() -> Void`: 매개변수와 반환값이 없음.
- `(String) -> Int`: 문자열을 받고 정수를 반환함.

## 4. 함수 표기법(Function Notation)

오버로딩으로 인해 이름이 겹치는 함수를 명확히 지칭할 때 사용한다. `_:`의 의미는 다음과 같다.

- `hello`: 함수 이름 자체. (모호할 수 있음)
- `hello(_:)`: 와일드카드 식별자를 사용하여 이름이 없는 매개변수를 가진 함수임을 명시.
- `hello(name:)`: `name`이라는 Argument Label을 가진 함수임을 명시.
- `#selctor(clicked(_:))`: 셀렉터에서 특정 매개변수를 받는 함수를 정확히 가리킬 때 필수적.

## 5. 클로저(Closure) - 이름 없는 함수

함수를 굳이 정의하지 않고 코드 블록 형태로 사용하는 것이다.

### 구조 (Closure Expression)

```Swift
let study = { (name: String) -> Void in
    // Closure Body
    print("\(name) 공부 중")
}
```

- `in` 키워드: 클로저의 Header(타입 정의)와 Body(실제 로직)를 구분하는 기준점이다.
- **Trailing Closure**: 함수의 마지막 인자가 클로저라면 호출 시 소괄호 바깥으로 뺄 수 있어서 가독성이 좋아진다.
