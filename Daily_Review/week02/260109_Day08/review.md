# Day 08 수업 내용 정리
부제목: Custom Cell(awakeFromNib), UITableView 데이터 추가 및 삭제

## 1. iOS 13: 대변혁의 기점

iOS 12에서 13으로 올라갈 때 유저 경험뿐만 아니라 개발 방식에서도 가장 큰 변화가 있었다.

* **시스템 변화**: 다크모드 도입, iPadOS 독립, 그리고 앱의 생명주기를 관리하는 **Scene Delegate**가 등장했다.
* **화면 전환**: 기존에는 `fullScreen`이 기본이었으나, iOS 13부터는 카드 형태의 `sheet` 방식이 기본(Default)가 되었다.
* **새로운 기술**: UIKit 위주의 개발에서 SwiftUI가 처음 등장했고, 테이블뷰/컬렉션뷰를 더 유연하게 만드는 Compositional Layout과 Diffable Data Source 같은 기술들이 생겨났다.

## 2. 레이아웃과 UI 디테일에 대한 내용

테이블뷰의 미세한 디자인을 조정하는 방법들이다.

* **구분선(Separator)**: 좌측 여백(Inset)의 Left 값을 0으로 설정하면 구분선이 화면 끝까지 꽉 차게 보인다.
* **Section Index**: 카카오톡 친구 목록처럼 우측에 'ㄱ, ㄴ, ㄷ' 등의 인덱스를 눌러 이동하는 기능을 설정할 수 있다.
* **Header 구성**: 테이블뷰 상단에 배너 등을 넣고 싶을 때, `UIView`를 드래그하여 헤더 영역에 두고 높이를 조절하여 구현한다.

# 유연한 화면 구성을 위한 선택: UIViewController + UITableView

`UITableViewController`는 편리하지만, 화면 전체가 테이블뷰라 검색바를 상단에 고정하는 등의 유연한 레이아웃을 잡기가 어렵다.

* **문제점**: `UITableViewController`는 스크롤 기능을 상속받고 있어서 검색바를 내부에 넣으면 스크롤 시 같이 올라가 버린다.
* **해결책**: 일반 `UIViewController` 위에 `UITableView`를 필요한 영역만큼만 얹어서 만든다. 이렇게 하면 **테이블뷰만 독릭접으로 스크롤**되게 제어할 수 있어서 훨씬 유연하다.

# 4. 이벤트 핸들링: addTarget

`@IBAction` 연결이 어려운 상황이나 코드로 UI를 짤 때 `addTarget`을 사용한다.

* **대상**: `UIControl`을 상속받는 모든 객체(UIButton, UITextField 등)에서 사용 가능하다.
* **문법**: `#selector`를 통해 실행할 함수를 연결하며, 이때 해당 함수는 `@objc` 키워드가 붙어있어야 한다.

```Swift
    override func viewDidLoad() {
        super.viewDidLoad()
        // UIControl
        // Function Type
        addButton.addTarget(
            self,
            action: #selector(addButtonClicked),
            for: .touchUpInside) // 해당 테이블뷰에서 실행할 거라서 self
    }

    @objc func addButtonClicked(_ sender: UIButton) {
        // 1. 텍스트필드 글자 가져오기
        let text = dramaTextField.text!
        // 2. sample 배열에 글자 추가하기
        sample.append(text)
        // 3. 잘 추가되었는 지 print 로 확인
        print(sample)
    }
```

* **주의**: 함수 이름만 쓰는 이유는 '함수 타입' 자체를 전달하기 위해서다. `()`를 붙여 실행해버리면 버튼을 누르기도 전에 함수가 돌아가버리기 때문이다. (리액트에서 했지?)

## 5. 데이터 우선주의와 화면 갱신 (Sync)

UIKit은 데이터가 바뀐다고 화면이 자동으로 바뀌지 않는다. **데이터 처리 후 뷰 갱신**이라는 원칙을 꼭 기억해야 한다.

* **reloadData()**: 데이터(배열 등)에 변화가 생기면 반드시 이 메서드를 호출하여 테이블뷰를 처음부터 다시 그리게 해야 한다.
* **didSelectRowAt**: 셀을 클릭했을 때 실행되는 함수다. 셀을 직접 지우는 것이 아니라 **배열에서 데이터를 삭제한 뒤 `reloadData()`를 호출**하여 화면이 데이터를 따라오게 만든다.

## 6. 셀 재사용 매커니즘

테이블뷰는 수천 개의 데이터가 있어도 눈에 보이는 만큼의 '셀 그릇'만 만들어서 돌려쓴다. 이를 이해하는 것이 효율적인 코드의 핵심이다.

* **awakeFromNib**: 셀 그릇이 처음 만들어질 때 딱 한 번 호출된다. **변하지 않는 디자인**(폰트, 색상 등)을 여기서 설정하면 성능이 좋아진다.
* **prepareForReuse**: 셀이 재사용되기 직전에 '설거지'를 하는 단계다. 이전 셀의 상태(배경색 등)가 남지 않도록 초기화해준다.
* **cellForRowAt**: 재사용되는 그릇에 매번 새로운 데이터(내용)를 담는 곳이다. `if-else`문을 통해 모든 조건에 대응해야 엉뚱한 데이터가 표시되지 않는다.

## 7. 셀 높이 조절

* **고정 높이**: 모든 셀 높이가 같다면 `tableView.rowHeight = 80`처럼 **프로퍼티**를 쓰는 것이 성능에 가장 좋다.
* **유동적 높이 (Automatic Dimension)**: 내용에 따라 높이가 변하게 하려면 `numberOfLines = 0`으로 설정하고 `UITableView.automaticDimension`을 사용한다. 이때 오토레이아웃 제약조건(상하좌우 여백)이 완벽해야 시스템이 높이를 계산할 수 있다.
