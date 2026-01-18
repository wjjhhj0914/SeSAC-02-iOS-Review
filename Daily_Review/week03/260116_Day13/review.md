# Day 13 - 수업 내용 복습 정리

> UserDefaults의 심화 활용부터 접근 제어자(Private), 그리고 Swift의 핵심인 프로토콜까지!

## 1. UserDefaults: 데이터 싱크와 생명주기

앱을 껐다 켜지 않아도 데이터가 반영되게 하려면 **화면이 나타날 때마다** 데이터를 새로고침해야 한다.

* `viewDidLoad`: 화면이 처음 메모리에 로드될 때 **단 한 번** 실행.
* `viewWillAppear`: 다른 화면에 갔다 돌아올 때를 포함하여, **화면이 나타날 때마다** 실행.

```Swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    // 화면이 뜰 때마다 최신 데이터를 가져와서 UI 업데이트
    setProfile()
}
```

## 2. 매직 스트링 방지와 접근 제어

"nick", "age" 같은 문자열(Magic String)을 직접 쓰는 것은 오타의 위험이 크다. 이를 방지하기 위해 열거형(Enum)이나 구조체(Struct)를 활용한다.

### 왜 Enum이나 `private init`을 쓸까?

불필요한 인스턴스 생성을 막아 **메모리 낭비를 방지**하고, **타입 자체**로만 사용하게 강제하기 위해서다.

```Swift
// 방법 1: 구조체 + private init
struct UDKey {
    static let nick = "nick"
    static let age = "age"

    private init() { } // 외부에서 UDKey() 인스턴스 생성 방지
}

// 방법 2: 열거형 (인스턴스 생성이 아예 불가능함)
enum UDKey {
    static let nick = "nick"
    static let age = "age"
}
```

## 3. UserDefaultManager: 연산 프로퍼티 활용

`get`과 `set`을 활용하면 데이터를 읽고 쓰는 로직을 한곳에서 관리할 수 있어 코드가 매우 간결해진다.

```Swift
struct UserDefaultManager {
    static var nick: String {
        get {
            // 데이터를 가져올 때 (기본값 설정 가능)
            return UserDefaults.standard.string(forKey: UDKey.nick) ?? "손님"
        }
        set {
            // 데이터를 저장할 때 (newValue는 새로 들어온 값)
            UserDefaults.standard.set(newValue, forKey: UDKey.nick)
        }
    }
}

// 사용 예시
print(UserDefaultManager.nick) // get 실행
UserDefaultManager.nick = "고래밥" // set 실행 (newValue에 "고래밥"이 전달됨)
```

## 4. 접근 제어(Private)와 컴파일 최적화

`private`는 단순히 "나만 볼 거야!"가 아니라 **성능**과도 연결된다.

* 안전성: 외부에서 내부 로직(예: `configureLabel`)을 마음대로 호출해 생기는 버그를 방지한다.
* 성능(최적화): `private`이 붙으면 Swift 컴파일러는 "이 메서드는 다른 곳에서 안 쓰이네?"라고 판단하여 호출 과정을 단순화(Direct Dispatch)하므로 **빌드 및 실행 속도**가 미세하게 빨라진다.

## 5. 프로토콜(Protocol): 규격과 타입

프로토콜은 "이 기능은 반드시 구현해!"라고 약속하는 **설계도**다.

### AnyObject와 선택적 요청

* `AnyObject`: 이 프로토콜은 클래스(Class)에서만 채택할 수 있게 제한한다.
* `@objc optional`: 프로토콜 메서드를 반드시 구현하지 않아도 되게 만든다. (단, 클래스에서만 사용 가능)

### 왜 클래스 전용일까?

이유는 참조 타입의 특성과 Objective-C와의 역사 때문이다.

#### AnyObject: "참조형만 들어오세요"

* 개념: `AnyObject`는 모든 클래스가 암시적으로 준수하는 프로토콜이다.
* 이유: 구조체나 열거형은 **값 타입(Value Type)** 으로, 메모리의 **스택(Stack)** 영역에 복사되어 저장된다. 반면 클래스는 **참조 타입**으로 **힙(Heap)** 영역에 저장되고 그 주소값을 공유한다. `AnyObject`는 "이 프로토콜은 주소값을 가진 객체(Object)여야만 한다"는 제약을 거는 것이기 때문에 **클래스에서만 사용 가능**한 것이다.

#### @objc optional: "Objective-C의 유산"

* 개념: 원래 Swift의 프로토콜은 '규격'이므로 채택하면 **모든 메서드를 반드시 구현**해야 한다. 하지만 iOS의 기반인 UIKit(Objective-C 기반)에서는 "필요한 것만 골라 쓰세요"라는 방식이 아주 많았다.
* 이유: `@objc`라는 키워드 자체가 **Objective-C 런타임을 빌려오겠다**는 뜻이다. 그런데 Objective-C는 구조체나 열거형에 메서드를 넣거나 프로토콜을 채택하는 기능이 _아예_ 없었다. 오직 클래스만 가능했다. 따라서 Objective-C의 방식을 빌려온 `optional` 기능 역시 **클래스에서만 작동**하게 된 것이다.

```Swift
@objc protocol ViewBasicProtocol: AnyObject {
    func configureView()
    @objc optional func setButton() // 필수가 아닌 선택 사항
}
```

### ⭐️ 타입으로서의 프로토콜 (Protocol as Types)

프로토콜을 타입으로 사용하면 **다형성**을 구현할 수 있다. "어떤 종류든 상관없어, 이 규겨만 지키면 다 받아줄게"라는 뜻이다.

```Swift
protocol SeSAC { func addShot() }
class Latte: SeSAC { func addShot() { print("샷 추가") } }
struct Americano: SeSAC { func addShot() { print("샷 추가") } }

// SeSAC 타입을 공유하므로 클래스든 구조체든 담을 수 있음
var coffee: SeSAC = Latte()
coffee = Americano()
```

## 6. Protocol vs private: 언제 무엇을 쓸까?

이 둘은 목적 자체가 완전히 다르다. **외부와의 약속**이냐, **내부의 비밀**이냐의 차이다.

### 프로토콜을 써야 할 때

* 공통 규격이 필요할 때: 여러 클래스가 같은 형태의 메서드를 가져야 할 때(예: 모든 뷰컨트롤러는 `setUpUI()`를 가져야 한다).
* 협업/소통이 필요할 때: "너 내 부하직원(Delegate) 하려면 이 규칙은 지켜!"라고 명세서를 줄 때.
* 다형성을 활용할 때: 어떤 클래스인지 몰라도 "이 프로토콜을 가졌다면 이 메서드는 있겠군"하고 믿고 호출하고 싶을 때.

### private(접근 제어)을 써야 할 때

* 내부 로직을 숨길 때: 밖에서 이 메서드를 건드렸다가 데이터가 꼬일 위험이 있을 때.
* 컴파일 성능을 높일 때: "이건 여기서만 쓰니까 다른 곳 찾지 마!"라고 컴파일러에게 알려줘서 빌드 속도를 높일 때.
* 복잡도를 낮출 때: 외부에서 도트(`.`)를 찍었을 때 꼭 필요한 메서드만 자동완성에 나오게 하고 싶을 때.

### Protocol vs private 차이점 비교

| 구분 | 프로토콜 (Protocol) | 접근 제어 (private) |
| :--- | :--- | :--- |
| **핵심 목적** | **규격 정의 및 표준화** (외부 공개) | **내부 구현 은닉 및 보호** (내부 폐쇄) |
| **비유** | 식당의 **메뉴판** (누구나 볼 수 있음) | 주방의 **비밀 레시피** (주방장만 볼 수 있음) |
| **의무 사항** | 채택(Adopt) 시 명세된 기능을 반드시 구현해야 함 | 구현 여부를 외부에 알리거나 강제할 필요 없음 |
| **사용 범위** | 여러 타입 간의 공통된 인터페이스 제공 | 정의된 범위(대개 현재 중괄호 `{ }`) 내부로 한정 |
| **성격** | 외부와의 통로를 만드는 것 (**Interface**) | 외부와의 통로를 차단하는 것 (**Encapsulation**) |
| **상호 관계** | 프로토콜 요구 사항은 `private`으로 선언 불가 | 프로토콜과 관계없이 내부 로직에 자유롭게 사용 가능 |

### 왜 둘을 동시에 못 쓸까? (Trade-off)

수업 내용 중 "충돌이 난다"고 했던 부분의 핵심이다.

프로토콜은 "이 메서드는 밖에서 누구나 부를 수 있는 '공식 규격'이다"라고 선언하는 것이다. 그런데 그 메서드에 `private`을 붙이면 "이건 '나만 아는 비밀'이라 밖에서 못 부른다"는 뜻이 된다.

**컴파일러의 황당함**: "아니, 메뉴판(Protocol)에는 '김치찌개'가 있다고 써놓고, 주방 문(private)은 걸어 잠가서 아무도 주문을 못 하게 하면 어떡해? 둘 중 하나만 해!"

결국 프로토콜은 외부와의 통로를 여는 것이고, private은 통로를 닫는 것이기 때문에 논리적으로 공존할 수 없는 것이다.

---

"왜 프로포콜과 private을 동시에 못 쓸까?"

프로토콜은 외부와 소통하기 위한 **공개된 규격**인데, `private`은 나만 보겠다는 **폐쇄적 제어**이기 때문이다. 모순이 발생하므로 함께 쓸 수 없는 것이지요~