# Day 11 - 수업 내용 복습 정리

> iOS 개발의 꽃!!! 완전 중요한 내용들을 배웠다. CollectionView, 화면 전환 + 값 전달

## 1. 화면 전환: "어떤 길로 가서, 어떻게 돌아올 것인가?"

화면 전환은 단순히 화면이 바뀌는 게 아니라, **메모리 계층 구조**를 만드는 과정이다. 짝꿍(Push-Pop / Present-Dismiss)을 맞추는 게 가장 중요하다. 이거를 맞추지 않으면 반응하지 않는다.

### 화면 전환의 2가지 방식 비교

| 방식 | 메서드 짝꿍 | 특징 | 비유 |
| :--- | :--- | :--- | :--- |
| **Show (Push)** | `push` / `pop` | 우측에서 좌측으로 등장. 상단 네비게이션 바 유지. | 책장을 한 장씩 넘기는 것 |
| **Modal (Present)** | `present` / `dismiss` | 아래에서 위로 등장. 새로운 맥락(글쓰기, 설정 등)을 띄울 때 사용. | 기존 책 위에 포스트잇을 붙이는 것 |

### 코드로 화면 전환하는 3단계 (암기 필수!)

1. Storyboard 찾기: `UIStoryboard(name: "Main", bundle: nil)`
2. VC 찾기: `instantiateViewController(withIdentifer: "ViewController")`
3. 전환 방식 선택: `present(vc, ...)` 또는 `navigationController?.pushViewController(vc, ...)`
  **주의!** `push`를 썼는데 화면이 안 뜬다면? Entry Point가 되는 뷰 컨트롤러가 Navigation Controller에 Embed 되어 있는지 꼭 확인해야 한다. 왜냐면 `navigationController`가 `nil`이면 아무 일도 일어나지 않는다!!

## 2. 값 전달: "식판 통째로 넘겨주기"

화면을 전환하면서 데이터를 넘겨줄 때 가장 세련된 방법은 **인스턴스 프로퍼티**를 활용하는 것이다.

### ⚠️ 아울렛에 직접 값을 넣으면 안 되는 이유 (런타임 에러)

```Swift
let vc = storyboard.instantiateViewController(...) as! DetailViewController
vc.mainLabel.text = "Hello" // ❌ 여기서 앱이 터짐!
```

* 이유: `instantiateViewController`를 호출한 직후에는 아직 뷰 컨트롤러의 뷰가 로드되지 않은 상태다. 즉, `mainLabel`이라는 아울렛 변수는 아직 메모리에 올라오지 않은 `nil` 상태이기 때문에 값을 넣으려고 하면 `Fatal Error`가 발생한다.
* 해결책: 일반 변수를 `DetailViewController`에 만들어서 거기에 값을 담아주고, 실제 화면에 보여주는 것은 `DetailViewController`의 `viewDidLoad` 시점에 수행한다.

## 3. CollectionView: "자유로운 레이아웃의 끝판왕"

테이블뷰가 '리스트'라면, 콜렉션뷰는 '그리드'나 '수평 스크롤' 등 훨씬 자유로운 배치가 가능하다.

### FlowLayout 설정의 핵심 요소

테이블뷰와 달리 **셀의 너비와 높이를 직접 계산**해줘야 한다.

* itemSize: 셀의 크기(width, height).
* sectionInset: 전체 콘텐츠의 상하좌우 여백(안쪽 여백).
* minimumLineSpacing: 줄 간격(수직 스크롤이면 행 사이, 수평이면 열 사이).
* minimumInteritemSpacing: 셀 사이의 최소 간격.

### 화면 너비에 맞게 'N등분' 하는 법

화면 크기가 기기마다 다르기 때문에, 고정 수치보다는 화면 전체 너비를 기준으로 계산하는 게 좋다.

```Swift
let layout = UICollectionViewFlowLayout()
let spacing: CGFloat = 20
let width = UIScreen.main.bounds.width - (spacing * 3) // 여백 3군데 제외
layout.itemSize = CGSize(width: width / 2, height: width / 2) // 2단 배열
```

## 4. iOS 이후의 변화

수업 중에 잠깐 언급된 Compositional Layout이나 Diffable Data Source는 iOS 13부터 등장한 현대적인 방식이다. FlowLayout이 기본기라면, 나중에는 더 복잡한 레이아웃을 훨씬 적은 코드로 짤 수 있게 된다고 한다.