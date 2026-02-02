# Day 22 - 수업 내용 복습 정리

## 1. 네트워크 요청 추상화(Network Abstraction)

네트워크 통신 코드를 하나의 파일(`Manager`)로 몰아넣는 것을 넘어, 요청에 필요한 요소(URL, Method, Parameter 등)를 열거형 등으로 규격화하는 작업이다.

### Router 패턴 (열거형 활용)

- 이유: API가 늘어날 때마다 `fetchPhoto`, `fetchUser` 등 메서드를 무한정 만들지 않기 위함이다.
- 구성: `BaseURL`, `Endpoint`, `Method`, `Parameter` 등을 **연산 프로퍼티**로 정의한다.
- 연관값(Associate Value): `case random(id: Int)`처럼 케이스 옆에 값을 붙여, 호출 시점에 동적인 데이터를 전달할 수 있다.

## 2. 제네릭(Generic) - 타입을 변수처럼

제네릭은 어떤 타입이 들어올지 미리 정하지 않고, **사용(호출)하는 시점에 타입을 결정**하는 기능이다.

### 타입 파라미터 `<T>`

- `T`는 placeholder다. `Int`, `String`, 혹은 수업 시간에 만든 `PhotoInfo` 구조체가 들어갈 자리를 비워두는 것이다.
- Type Constraints(타입 제약): `<T: Decodable>`처럼 작성하면, "아무나 못 들어오고 `Decodable`을 채택한 친구들만 들어와"라고 제한하는 것이다. (안전장치)

### 왜 제네릭이 네트워크 통신에 필수일까?

서버 응답 데이터(식판)는 API마다 다르다. 하지만 <u>요청을 보내고 응답을 받아서 디코딩한다</u>는 로직은 같다. 이때 제네릭을 쓰면 **하나의 메서드로 모든 식판을 처리**할 수 있다.

```Swift
// 제네릭을 사용한 단 하나의 네트워크 메서드
func fetch<T: Decodable>(api: PhotoRouter, type: T.Type, completion: @escaping (T) -> Void) {
    AF.request(api.endpoint, method: api.method).responseDecodable(of: T.self) { response in
        // T.self가 런타임에 [PhotoInfo].self 가 되기도, PhotoInfo.self 가 되기도 함!
        if let value = response.value {
            completion(value)
        }
    }
}
```

## 3. 메타 타입(Meta Type) - 타입의 타입

제네릭 함수를 호출할 때 우리는 `type: PhotoInfo.self`처럼 전달한다. 이게 바로 메타 타입이다.

- `PhotoInfo`: 구조체의 이름 (타입 그 자체)
- `PhotoInfo.self`: 그 타입을 가리키는 **값(Value)**
- `PhotoInfo.Type`: 그 값의 **타입(Meta Type)**

비유를 하자면, `String`이 "사과"라면, `String.Type`은 "과일 종류"라는 카테고리 자체를 의미한다. 컴퓨터에게 내가 지금 주려는 게 어떤 종류의 데이터 구조인지 알려주는 명찰 같은 것이다.

## 4. AppDelegate & SceneDelegate 가이드

- AppDelegate: 앱의 생명주기와 시스템 이벤트를 담당. 화면이 없으므로 여기서 네트워크 통신을 직접 하면 실패 시 Alert 등을 띄울 수 없어서 위험하다.
- SceneDelegate: UI창(Window)을 관리. 코드 기반 UI를 짤 때 `rootViewController`를 여기서 설정한다.
- 팁: 앱 시작 시 데이터가 필요하다면 `SplashViewController` 같은 전용 화면을 만들어서 거기서 통신하는 것이 정석이다. 즉, LaunchScreen과 똑같이 생긴 VC를 하나 만들어서 거기서 네트워크 통신을 하는 것이다.

## 5. Tips

1. Moya 라이브러리: 우리가 만든 `Router` 구조를 더 체계적으로 프로토콜화해서 관리하게 해주는 라이브러리.
2. `@unknown default`: 열거형에 새로운 케이스가 추가될 미래를 대비하는 속성. 미래에 추가될 케이스를 처리하지 않아서 발생할 오류를 미리 경고해 준다.
