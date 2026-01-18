# Day 10 - 수업 내용 복습 정리

> Extension & Protocol

## 1. UI 효율성 극대화: Extension

많은 파일에 공통된 디자인을 적용할 때, 한 줄씩 수정하는 비효율을 줄이기 위해 사용한다.

* UIViewController 확장: 모든 화면에 공통적으로 쓰이는 기능(예: 배경색 설정)을 넣기 좋다.
* 특정 객체 확장 (UITextField, UILabel): 모든 화면이 아닌 특정 UI 요소만 디자인할 때는 해당 객체를 직접 확장하는 것이 효율적이다.
  * 이유: `UIViewController`에 너무 많은 메서드를 넣으면, 그 기능을 안 쓰는 화면에서도 해당 메서드를 계속 들고 있어야 하는 '이고지고 가는' 상황이 발생하기 때문이다.

## 2. 테이블뷰 구현: TVC vs (VC + TV)

테이블뷰를 구현하는 두 가지 방식의 차이점이다.

[단계별 구현 흐름]
1. 아울렛 연결: `UITableView`를 코드와 연결한다.
2. 프로토콜 채택: `UITableViewDelegate`, `UITableViewDataSource`를 클래스 이름 뒤에 써준다.
3. 부하 직원 임명(Delegate/DataSource): `viewDidLoad`에서 `tableView.delegate = self`처럼 '내 일은 내가 하겠다'고 명시한다.
4. XIB 등록: 스토리보드가 아닌 별도 파일(XIB)로 만든 셀을 사용한다면 반드시 `register` 과정을 거쳐야 한다. ← 과제할 때도 이 과정을 자주 까먹어서 앱이 크래시됐는데, 꼭 기억하자‼️

## 3. 코드의 거주지: "누가 이 일을 해야 하는가?"

### 📋 UITableViewController vs. UIViewController + UITableView 비교

| 구분 | UITableViewController (TVC) | UIViewController + UITableView |
| :--- | :--- | :--- |
| **View 계층 구조** | **Root View가 곧 TableView**입니다. 별도의 기본 View가 없습니다. | **기본 UIView 위에 TableView를 얹은** 형태입니다. |
| **레이아웃 자유도** | 화면 전체가 테이블뷰라 다른 UI 요소(버튼 등)를 고정 배치하기 어렵습니다. | 테이블뷰의 크기를 자유롭게 조절하고, 주변에 다른 View를 배치하기 쉽습니다. |
| **자동 설정** | `delegate`와 `dataSource`가 자동으로 `self`에 연결되어 있습니다. | `delegate`와 `dataSource`를 **코드로 직접 연결**해줘야 합니다. |
| **아울렛(Outlet)** | 기본 `tableView` 변수가 내장되어 있어 따로 아울렛 연결이 필요 없습니다. | 테이블뷰를 제어하기 위해 반드시 **아울렛 변수**를 만들어야 합니다. |
| **사용 목적** | 설정 화면처럼 화면 전체가 리스트로만 구성될 때 편리합니다. | 복잡한 레이아웃이나, 한 화면에 여러 개의 테이블뷰가 들어갈 때 사용합니다. |

코드의 위치를 옮겨가며 가독성과 효율을 높이는 과정(Refactoring)이다.

* Step 1(VC): `cellForRowAt`에서 레이블의 텍스트, 색상 등을 직접 지정 (VC가 너무 바쁨).
* Step 2(Cell): 셀 안에 `configure` 함수를 만들고 데이터만 넘겨줌(VC가 한결 가벼워짐).
* Step 3(Data Model): 데이터의 가공(예: 금액 뒤에 '원' 붙이기)은 `struct` 내부에서 처리(셀은 출력만 담당).

## 4. 헷갈리는 개념 요약 (Property)

### 저장 프로퍼티 vs 연산 프로퍼티

* 저장 프로퍼티(Stored): 한마디로 "창고"다. 값을 실제로 메모리 공간에 **저장**한다.
  * `var nickname = "고래밥"`
* 연산 프로퍼티(Computed): "터널" 또는 "계산기"다. 값을 저장하지 않고, **부를 때마다 계산**해서 결과만 알려준다.
  * `var introduce: String { return "저는 \(nickname)입니다." }
  * **언제 써요?** 매개변수가 필요 없는 단순 가공(예: 가격 포맷팅)일 때 함수 대신 쓰면 코드가 훨씬 깔끔해진다.

### 인스턴스 vs 타입 (Static)

* 인스턴스: 붕어빵 각각의 특징 (철수네 붕어빵, 영희네 붕어빵). `jack.age`처럼 이름표를 붙여서 호출한다.
* 타입 (Static): 붕어빵 틀 자체의 특징. `User.hello`처럼 클래스/구조체 이름으로 직접 호출한다.
  * **언제 써요?** 앱 전체에서 공통으로 쓰이는 값(예: 셀 identifer `static let identifier = "Cell"`에 사용하면 메모리 효율도 좋고 관리도 편하다.

---

`let cell = BookTableViewCell()`이 안 되는 이유는, 이 방식은 코드만 가져오고 스토리보드와 연결된 정보를 가져오지 못하기 때문이다. 그래서 우리는 `dequeueReusableCell`을 통해 "이미 만들어진(디자인된) 셀을 재사용하겠다"고 시스템에 요청하는 방식을 사용한다.