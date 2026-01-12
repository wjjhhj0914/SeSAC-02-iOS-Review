# Day 04 수업 복습

## 1. UILayout & Component

### Storyboard & Constraints

* **Aspect Ratio**: 너비나 높이 중 하나가 고정되어 있을 때 비율을 유지하는 속성.
*  **Constant vs Multiplier**
  * `Constant`: 고정된 수치(예: 20px). 기기마다 화면 크기가 달라도 항상 같은 크기를 유지한다.
  * **`Multiplier`**: 비율 기반 수치(예: 1:1 정사각형). 화면 크기에 따라 유동적으로 변하므로 아이폰 대응에 유리하다.
* **Stack View**: 여러 뷰를 묶어 관리할 때 필수적! `Fill Equally`는 내부 요소들의 크기를 똑같이 배분(N빵)한다.

### UIButton & UIImageView

* **Button Style (iOS 15 기준)**
  * **Plain, Tinted, Filled**: **최신** 스타일. 이미지 여백 조절 등이 편리하지만 속성 적용 방식이 다르다.
  * **Default**: `setTitle`, `setTitleColor` 등이 가장 잘 먹히는 **기본** 스타일.
* **SF Symbols vs Assets**
  * `UIImage(systemName: "pencil")`: 애플 제공 아이콘을 불러올 때 사용.
  * `UIImage(name: "pencil")`: Assets에 직접 넣은 이미지를 불러올 때 사용.

## 2. Keyboard & UX

### 키보드 내리기 (Dismiss Keyboard)

1. `view.endEditing(true)`: 바탕 뷰를 기준으로 올라온 모든 편집 요소를 한꺼번에 종료시킨다.
2. `Tap Gesture`: 빈 화면을 눌렀을 때 키보드를 내리고 싶을 때 사용. 단, 배경색 투명도가 너무 낮거나 (`alpha < 0.01`) user interaction이 꺼져 있으면 작동하지 않을 수
있다.

#### UITextField 주요 이벤트 정리

사용자가 Text Field와 상호작용하는 시점에 따라 다양한 이벤트를 활용할 수 있다.

* `Editing Changed`: 텍스트가 입력되거나 삭제될 때마다 **실시간**으로 호출된다.
  * 용도: 글자 수 실시간 제한, 유효성 검사(실시간 빨간 불 띄우기) 등에 사용한다.
* `Editing Did End`: 어떤 이유로든 **편집이 끝났을 때**(포커스가 다른 곳으로 옮겨갔을 때) 호출된다.
  * 주의: Return키를 눌렀을 때뿐만 아니라 다른 필드를 터치하거나 화면 빈 곳을 눌러 편집이 종료될 때도 발생한다.
* `Did End On Exit`: 사용자가 키보드의 **Return키를 눌렀을 때만** 발생하는 이벤트다.
  * 특징: 애플이 Return키 클릭 시 키보드를 자동으로 내려주는 기능을 이 이벤트에 포함시켜 두었다. 따라서 중괄호 `{}` 안에 별도의 `view.endEditing(true)` 코드가 없어도 키보드가 내려간다.

#### 키보드 제어 요약을 해보면

* **전체 화면 클릭 시**: `Tap Gesture`를 연결하고 `view.endEditing(true)`를 실행하여 모든 요소의 편집을 종료한다.
* **Return키 클릭 시**: 특정 필드의 키보드만 내리고 싶다면 `Did End On Exit` 이벤트를 연결하는 것이 가장 간편하다.
* **작동 안 할 때 체크**: 스토리보드의 Connections Inspector에서 이벤트가 `Did End On Exit`에 정확히 연결되어 있는지, 아니면 `Editing Did End`에 잘못 연결되어 있는지는 않은지 확인하라.

### UILabels과 ClipsToBounds

* `UILabel`은 기본적으로 모서리 깎기가 안 보인다. `clipsToBounds = true`를 설정해야 깎여나간 영역 밖을 버리고 둥글게 보인다.
* **주의**: 테두리(`border`)는 깎여도 배경색이나 이미지는 안 깎일 수 있는데, 이때 `clipsToBounds`가 그릇 밖으로 튀어나온 배경을 잘라주는 역할을 한다.

## 3. Swift 개념

### 함수 매개변수의 두 얼굴

함수를 정의할 때 이름을 두 개 쓰는 이유는 '가독성' 때문이다.

* **외부 매개변수(Argument Label)**: 함수를 **호출할 때** 사용한다. `(to recipient: String)`에서 `to`에 해당하며, 코드를 문장처럼 읽히게 합니다.
* **내부 매개변수(Parameter Name)**: 함수 **내부에서** 사용합니다. `recipient`에 해당하며, 로직 안에서 데이터의 성격을 명확히 합니다.
* 와일드카드(`_`): 외부 이름을 생략하고 싶을 때 사용합니다.

### 타입 변환 (Casting)
* **CGFloat vs Int**: UI의 좌표나 굴곡(`cornerRadius`)은 소수점 단위가 가능한 `CGFloat`을 씁니다. `Int`를 넣을 땐 `CGFloat(num)`으로 변환해야 합니다.
* **정수 나눗셈의 함정**: `Int / Int`는 소수점을 버립니다. 정확한 평균값을 구하려면 나누기 전에 `Double(sum) / Double(count)`처럼 변환해야 소수점이 살아납니다.