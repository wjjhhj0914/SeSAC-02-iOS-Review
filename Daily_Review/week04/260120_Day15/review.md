# Day 15 - 수업 내용 복습 정리

## 1. SnapKit: Safe Area vs Superview

SnapKit으로 레이아웃을 잡을 때 이 둘의 차이를 명확히 아는 것이 중요하다.

- `equalToSuperview()` : 부모 뷰의 전체 영역(스크린 전체)을 꽉 채운다. 이 경우 상단의 노치나 하단의 홈 바 영역까지 콘텐츠가 침범하게 되어 글자가 가려지거나 터치가 안 될 수 있다.
- `equalTo(view.safeAreaLayoutGuide)` : 시스템이 정한 '안전 구역' 내에만 콘텐츠를 배치한다. 노치와 홈 바를 피해서 사용자에게 실제로 보이는 영역 안에 뷰를 안착시킨다.

> 즉, 콘텐츠가 노치나 홈 바에 가려지지 않도록, 전체 화면(`equalToSuperview`) 대신 실제 표시 가능 영역인 `safeAreaLayoutGuide`를 기준으로 레이아웃을 잡아야 한다.

```Swift
// ❌ 위험: 상단 상태바나 하단 홈 바에 콘텐츠가 겹칠 수 있음
tableView.snp.makeConstraints { make in
    make.edges.equalToSuperview()
}

// ✓ 권장: 기기별 노치 디자인에 상관없이 안전하게 노출됨
tableView.snp.makeConstraints { make in
    make.edges.equalTo(view.safeAreaLayoutGuide)
}
```

## 1. Code-based UI & Custom View

코드베이스로 UI를 작성할 때는 스토리보드(Xib)의 `awakeFromNib` 대신 `init` 구문을 사용한다.

### Designed Initializer vs Required Initializer

- `override init(frame: CGRect)` : 코드베이스로 뷰를 생성할 때 호출되는 **지정 생성자**.
- `required init?(coder: NSCoder)` : 스토리보드나 Xib에서 뷰를 불러올 때 사용되는 **필수 생성자**.
  - 코드베이스로만 작성하더라도 `NSCoding` 프로토콜 때문에 반드시 구현해야 함.
  - 보통 `fatalError`를 통해 코드베이스 전용임을 명시한다.

### Custom Initializer

반복되는 설정을 줄이기 위해 커스텀 `init`을 만들 수 있다.

```Swift
init(placeholder: String, keyboard: UIKeyboardType, isSecure: Bool) {
    super.init(frame: .zero) // .zero는 CGRect(x: 0, y: 0, width: 0, height: 0)
    self.placeholder = placeholder
    self.keyboardType = keyboard
    self.isSecureTextEntry = isSecure

    configureView() // 공통 디자인 메서드 호출
}
```

> Tip: 커스텀 `init`을 만들면 부모 클래스의 `init(frame:)` 상속이 끊길 수 있으므로 내부에서 `super.init(frame:)`을 반드시 호출해서 초기화 시점을 잡아주어야 한다.

## 2. UITableView in Code-base

코드베이스에서 테이블뷰를 사용할 때는 **Cell의 등록(Register)** 이 핵심이다.

### Swift Meta Type (`.self`)

- `tableView.register(BookTableViewCell.self, ...)`에서 `.self`는 인스턴스가 아닌 **타입 그 자체**를 의미한다.
- Xib가 없는 코드베이스 셀은 `cellClass` 타입으로 등록해야 하므로 `.self`를 붙여서 **메타 타입을 전달**한다.

### 즉시 실행 함수 (Immediate Invoked Function Expression)

**클로저**를 활용하여 프로퍼티를 초기화하면 코드가 간결해진다.

- 장점 : `viewDidLoad` 이전에 설정이 완료되며, `let`을 사용할 수 있어서 불변성을 유지하기 좋다.
- 주의 : 내부에서 `self`를 참조해야 하는 경우(delegate 설정 등)에는 `lazy var`를 사용해야 한다.

## 3. Network Communication (HTTP & Alamofire)

서버와 통신할 때는 HTTP 프로토콜의 특성을 이해해야 한다.

### HTTP의 특성

1. 비연결성(Connectionless) : 응답을 보내면 연결을 즉시 끊음.
2. 무상태성(Stateless) : 서버가 클라이언트를 기억하지 못함.
3. Client-Driven : 항상 클라이언트의 요청이 있어야 응답이 발생함.

### Serialization & Deserialization

- Encoding(직렬화) : Swift 객체 → JSON 문자열 (서버로 보낼 때)
- Decoding(역직렬화) : JSON 문자열 → Swift 객체 (서버에서 받을 때)
- Codable : `Encodable` + `Decodable` 합쳐진 프로토콜.

### Alamofire 라이브러리 사용 Flow

1. `responseString` : 통신 자체가 성공하는지, 어떤 형태의 데이터가 오는지 먼저 확인하는 용도.
2. `responseDecodable` : 정의한 구조체(식판)에 맞춰 데이터를 파싱.
3. `reloadData()` : 네트워크 응답은 **비동기**로 오기 때문에, 데이터가 도착한 시점에 반드시 테이블뷰를 갱신해야 화면에 반영된다.
