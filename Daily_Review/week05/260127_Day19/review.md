# Day 19 - 수업 내용 복습 정리

## 1. Apperance: 앱의 전반적인 스타일 설정

iOS 13부터 도입된 `Appearance` API를 사용하면 앱 전체의 네비게이션 바 혹은 탭 바 스타일을 일관되게 관리할 수 있다.

- Standard Style: 콘텐츠가 스크롤 될 때의 배경색과 스타일.
- Scroll Edge Style: 콘텐츠가 최상단/최하단에 닿아 있을 때의 스타일.
- AppDelegate 활용: `UILabel.appearance().textColor = .white`처럼 설정하면 모든 레이블의 기본 색상을 한 번에 바꿀 수 있다.

## 2. Code-based Start: Main.storyboard 삭제하기

스토리보드 없이 코드로만 첫 화면을 띄우는 과정이다.

1. Info.plist 삭제: `Main storyboard file base name` 항목 제거.
2. Scene Configuration 삭제: `Storyboard Name` 항목 제거.
3. SceneDelegate 설정:

```Swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
    guard let scene = (scene as? UIWindowScene) else { return }
    window = UIWindow(windowScene: scene) // 앱의 뼈대가 되는 창 생성

    let vc = BookViewController()
    let nav = UINavigationController(rootViewController: vc)

    window?.rootViewController = nav // 첫 화면 지정
    window?.makeKeyAndVisible() // 윈도우를 보이게 설정
}
```

## 3. First Class Object & Escaping Closure

함수를 변수처럼 다루는 **일급 객체**의 특성과, 비동기 상황에서 필수인 **탈출 클로저**를 이해해야 한다.

### @escaping Closure가 필요한 이유

함수 내부에서 정의된 클로저가 함수의 실행이 끝난 뒤(예: 버튼 클릭 시점)에 호출되어야 할 때, 메모리에서 사라지지 않도록 **탈출(@escaping)** 시켜주는 것이다.

```Swift
// 실전 예시: 재사용 가능한 Alert 메서드
func showAlert(title: String, message: String, okAction: @escaping () -> Void) {
    let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)

    let ok = UIAlertAction(title: "확인", style: .default) { _ in
        okAction() // 확인 버튼을 눌렀을 때(함수 종료 후) 실행됨
    }

    alert.addAction(ok)
    present(alert, animated: true)
}
```

## 4. ScrollView: 창문(Frame)과 풍경(Content)

스크롤 뷰의 핵심은 **창문 크기(Frame Layout Guide)** 와 **내부 내용물 크기(Content Layout Guide)** 를 구분하는 것이다.

### SnapKit 레이아웃 공식

- **수평 스크롤**: <u>높이</u>를 `frameLayoutGuide`에 맞추어 고정.
- **수직 스크롤**: <u>너비</u>를 `frameLayoutGuide`에 맞추어 고정.

```Swift
// 수직 스크롤 레이아웃 예시
scrollView.snp.makeConstraints { make in
    make.edges.equalTo(view.safeAreaLayoutGuide)
}

contentView.snp.makeConstraints { make in
    make.width.equalTo(scrollView.snp.width) // 너비 고정 (수직 스크롤의 핵심)
    make.verticalEdges.equalTo(scrollView.contentLayoutGuide) // 내부 콘텐츠 길이에 따라 가변
}
```

## Tips

1. View Hierarchy: 앱을 실행하고 `Debug View Hierarchy`를 클릭하면 윈도우(Window) 위에 네비게이션 컨트롤러, 그 위에 뷰가 층층이 쌓인 구조를 3D로 확인할 수 있다. <- 이거 완전 쩖
