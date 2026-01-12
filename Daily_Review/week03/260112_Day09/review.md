# Day 09: 수업 내용 정리
부제목: alert, class vs struct, opensource license

## 1. UIAlertController: 알림과 액션 시트

사용자에게 메시지를 전달하고 선택을 받는 도구다.

* **구성 단계**
1. 생성: `title`, `message`, `preferredStyle`(`.alert`, `.actionSheet`)을 정한다.

```Swift
let alert = UIAlertController(title: "즐겨찾기", message: "할 일을 즐겨찾기에 등록하였습니다.", preferredStyle: .alert)
```

2. **액션 추가**: `UIAlertAction`을 만들어 버튼을 정의하고, `alert.addAction()`으로 추가한다.

```Swift
let ok1 = UIAlertAction(title: "확인1", style: .default)
```

3. **표시**: `present(alert, animated: true)` 명령어로 화면에 띄운다. 이거 무조건 해줘야 함! 안 하면 안 보여요.

* **버튼 스타일**
  * `.default`: 일반적인 파란색 버튼
  * `.destructive`: 주의가 필요한 빨간색 버튼
  * `.cancel`: 취소 버튼(학습 효과를 위해 위치가 고정되며, 한 알럿에 무조건 **하나만** 사용 가능하다.)

* 주의사항 (시점): `viewDidLoad`에서 `present`를 호출하면 화면이 아직 다 뜨지 않은 상태라 알럿을 띄울 수 없다는 경고가 나온다. 안전하게 화면이 나타난 후인 `viewDidAppear`에서 호출하거나 버튼 클릭 시점에 띄워야 한다.

## 2. Swift 문법: Class vs Struct

UIKit(Class 기반)과 SwiftUI(Struct 기반)의 근본적인 차이를 이해하는 출발점이다.

* **참조 타입(Class) vs 값 타입(Struct)**
  * **Class**: 힙(Heap) 영역에 저장되며, 변수에는 메모리 주소값이 저장된다. 여러 변수가 같은 인스턴스를 가리킬 수 있어서 **데이터 동기화**가 일어난다.
  * **Struct**: 스택(Stack) 영역에 저장되며, 값을 대입할 때 **복사**가 일어난다.

* **let의 차이**
  * **Class**: `let`으로 선언해도 내부의 `var` 프로퍼티는 변경 가능하다. (자물쇠가 주소값만 잠그기 때문!)
  ```Swift
  let format = DateFormatter()
  format.dateFormat = "yyMMdd"
  ```

  * **Struct**: `let`으로 선언하면 내부의 모든 값이 잠겨서 변경할 수 없다.

* **타입 프로퍼티(`static`)**: 인스턴스를 만들지 않아도 사용할 수 있으며, 메모리의 **데이터 영역**에 저장되어 앱이 실해되는 동안 공유된다. 호출 전까지는 공간을 차지하지 않아서 효율적이다.

## 3. 메모리의 4개 영역

앱이 사용하는 자원은 용도에 따라 나뉜다.

1. **Code**: 작성한 기계어 코드가 저장되는 곳.
2. **Data**: 전역 변수나 `static` 변수가 저장되는 곳. 앱 종료 시까지 유지된다.
3. **Heap**: 클래스 인스턴스처럼, 런타임에 크기가 결정되는 참조 타입이 저장되는 곳.
4. **Stack**: 함수 호출 시 생기는 지역 변수나 구조체 등 값 타입이 저장되는 곳.

## 4. 오픈소스와 Dependency Manager

남이 만든 코드를 안전하고 편리하게 가져다 쓰는 방법이다.

* **도구**: **SPM(Swift Package Manager) **이 애플 공식 도구로서 현재 가장 많이 쓰이며, 과거에는 **CocoaPods**가 보편적이었다.

* 버전 관리 (Sementic Versioning): `Major.Minor.Patch` 형식을 따른다.
  * **Major**: 큰 변화, 호환성 깨짐.
  * **Minor**: 신기능 추가, 시각적 변화.
  * **Patch**: 버그 수정.

* 필수 라이브러리
  * **Kingfisher**: 웹 이미지를 가져오고 캐싱할 때 쓰는 iOS 개발자의 필수 도구다.
  * **IQKeyboardManager**: 키보드가 입력창을 가리지 않게 자동으로 조정해 주는 편리한 도구다.

## 5. XIB와 Reusability

여러 화면에서 똑같은 디자인의 셀을 쓰고 싶을 때 사용한다.

* **UINib**: `.xib` 파일을 코드로 불러올 때 사용하는 객체다.

* 등록 과정: 스토리보드가 아닌 외부 파일(XIB)을 쓸 때는 테이블뷰에게 "이 셀 파일을 쓸 거야"라고 알려주는 `register` 코드가 반드시 필요하다!

```Swift
let nib = UINib(nibName: "DiaryTableViewCell", bundle: nil)
tableView.register(nib, forCellReuseIdentifier: DiaryTableViewCell.identifier)
```

여기서 `UINib`은 파일명임.

---

Q: "클래스로 만든 인스턴스를 `let`으로 선언했는데 왜 내부 값을 바꿀 수 있나요?"<br />
A: "클래스는 **참조 타입**이기 때문입니다. `let`은 인스턴스가 저장된 **메모리 주소값**을 변경하지 못하게 잠그는 것이지, 힙 영역에 있는 실제 데이터 내부의 변수까지 잠그는 것은 아니기 때문입니다."