# Day 14 - 수업 내용 복습 정리

> Protocol, lazy, Frame Based Layout & NSLayoutConstraint & Snapkit

## 1. 프로토콜과 델리게이트 (Delegate)

"부하직원을 부르는 과정"이라는 비유가 아주 찰떡이다. `UISearchBar`나 `UITextView`처럼 복잡한 기능이 담긴 컴포넌트는 단순히 "눌렀다" 그 이상의 정보(글자가 바뀌었나? 편집을 시작했나" 등)를 전달해야 하기 때문에 **대리자(Delegate)** 가 필요하다.

#### 핵심 포인트

- 연결의 중요성: `mySearchBar.delegate = self` 코드가 없으면, 부하직원이 보고할 대상(self, 즉 OneViewController)을 찾지 못해 아무리 타이핑해도 반응이 없다.
- TextView의 Placeholder: `UITextField`와 달리 `UITextView`는 기본 placeholder 프로퍼티가 없다. 그래서 `textColor`가 `.lightGray`인지 확인하는 조건문 등을 통해 직접 구현하는 것이 일반적이다.
- 다중 델리게이트 처리: 한 화면에 여러 텍스트뷰가 있다면 `if textView == topTextView`처럼 매개변수로 들어오는 객체를 비교하여 어떤 녀석이 말을 걸었는지 구분해야 한다.

## 2. UITextField & inputView (커스텀 키보드)

버튼처럼 보이지만 키보드 대신 다른 뷰를 띄우고 싶을 때 가장 많이 쓰이는 기법이다.

- `inputView`: 키보드 영역을 완전히 덮어버리는 뷰다. `UIDatePicker`나 `UIPickerView`를 할당해서 사용한다.
- `tintColor = .clear`: 텍스트 필드인데 글자를 입력받지 않고 피커만 보여줄 때, 깜빡이는 커서를 숨겨주는 코드다. 센스 만점!!
- 위치 선정: `picker` 인스턴스를 클래스 최상단(프로퍼티 영역)에 선언하면, `viewDidLoad` 외에 다른 메서드에서도 피커의 날짜 정보를 읽어오는 등 핸들링하기가 훨씬 유리하다.

## 3. 초기화 시점과 지연 저장 프로퍼티 (Lazy)

"인스턴스 멤버를 프로퍼티 초기화 중에 사용할 수 없다"는 에러는 자주 만났던 벽이다.

- 문제: 클래스가 만들어지는 시점에 `picker`와 `configurePicker()`가 동시에 생기려다 보니, 아직 존재하지도 않는 함수를 호출해서 값을 넣으라고 하니 컴퓨터가 당황하는 것이다.
- 해결 (lazy): "나중에 처음 사용할 때 만들어줄게"라고 미루는 것이다. 그러면 이미 클래스 초기화가 끝난 시점에 함수를 호출하게 되므로 안전하다.
- 즉시 실행 함수 (`{}()`): 함수 이름 짓기도 귀찮고 한 번만 쓰고 말 기능이라면, 클로저 뒤에 `()`를 붙여 바로 실행해버리는 방식이 깔끔하다.

## 4. 레이아웃의 진화와 SnapKit

면접에서 자주 물어본다는 **코드베이스 UI**의 핵심 흐름이다.

### 레이아웃 변천사

1. Frame Based: 좌표(x, y)와 크기(w, h)를 직접 찍음. 기기 크기가 다양해지며 한계 봉착.
2. Autoresizing Mask: Frame 방식에서 유연성을 조금 더함. (코드에서는 반드시 `translatesAutoresizingMaskIntoConstraints = false`로 꺼줘야 AutoLayout과 충돌 안 됨.)
3. AutoLayout (NSLayoutConstraint): 객체 간의 관계를 설정. 코드가 매우 길고 복잡함.
4. SnapKit: AutoLayout을 사람이 읽기 편한 문법으로 바꾼 **라이브러리**.

### SnapKit 주의사항

- 계층 구조(Hierarchy)가 1순위: `addSubView`를 먼저 하지 않고 `snp.makeConstraints`를 호출하면 "불법"!!! 이라서 (맞는지는 모르겠지만 에러 메시지에는 illegal이라고 뜨긴 떴음) 앱이 바로 종료된다. 반드시 뷰를 먼저 부모 뷰에 올린 뒤에 레이아웃을 잡아야 한다.
- Inset vs Offset
  - Offset: 단순히 간격을 띄울 때 사용. `trailing`이나 `bottom`은 화면 안쪽으로 오려면 마이너스 값을 써야 함.
  - Inset: 전체적인 테두리 여백을 잡을 때 사용. 양쪽을 한꺼번에 잡아주므로 훨씬 편리함.

---

SnapKit 어렵다.. 매일 하다 보면 익숙해지겠지.
