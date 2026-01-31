# Day 16 - 수업 내용 복습 정리

## 1. Network: API 통신

서버에 데이터를 요청할 때 단순히 URL만 보내는 것이 아니라 **Header**와 **Parameter**를 통해 정보를 정교하게 전달한다.

### Query String vs Header

- Query String(`?key=value`): URL 뒤에 데이터가 노출된다. 검색어(`query`)처럼 공개되어도 상관없는 정보에 주로 사용한다.
- HTTP Header: 택배 상자 안의 비밀 쪽지와 같음. **인증키(API Key)** 혹은 **토큰** 같은 민감한 정보를 안전하게 전달할 때 사용한다.
  - Authorization: "나는 권한이 있는 사용자다"라고 증명하는 헤더 키다. 카카오의 경우, `KakaoAK {REST_API_KEY}` 형식을 지켜야 한다.

### HTTP 상태 코드 (Status Code)

통신의 결과는 숫자로 온다. 이를 통해 예외 처리를 해야 한다.

- **200~299**: 성공 (OK)
- **401**: 인증 실패 (API 키 오류)
- **404**: 경로를 찾을 수 없음
- **500**: 서버 내부 오류 (서버 개발자의 영역)
- **Alamofire**의 `validate()`: 이 메서드는 기본적으로 200~299 사이가 아니면 `failure` 케이스로 보내주는 **필터** 역할을 한다.

## 2. UICollectionView & Layout 이슈

테이블 뷰와 달리 컬렉션 뷰는 어떻게 보여줄지(Layout)에 대한 정보가 초기화 시점에 반드시 필요하다.

### 왜 `let collectionView = UICollectionView()`는 안 되는 걸까?

- 테이블 뷰는 리스트 형태가 고정이라 애플이 기본값을 주지만, 컬렉션 뷰는 <u>격자, 가로, 비대칭</u> 등 형태가 무궁무진하다.
- 따라서 생성 시점에 `UICollectionViewLayout` 객체가 없으면 **엔진이 뷰를 그릴 수 없어**서 <u>런타임 에러가 발생</u>한다.

### Static 메서드와 Lazy의 관계

- `static func layout()`: 클래스 인스턴스가 생기기도 전에 메모리에 올라가 있는 메서드다. 따라서 인스턴스 생성을 위한 초기값(`collectionViewLayout: layout()`)으로 바로 사용할 수 있다.
- `lazy var`: `self`가 생성된 이후에 접근할 수 있는 변수다. 클로저 내부에서 `delegate = self`처럼 자기 자신을 참조해야 할 때 사용한다.

## 3. Protocol Property Requirements

프로토콜은 메서드뿐만 아니라 변수(Property)도 요구할 수 있다.

### `{ get set }`의 의미

프로토콜은 이 변수가 '저장 프로퍼티'인지 '연산 프로퍼티'인지 상관하지 않는다. 오직 이 값을 읽을 수 있는가(get), 혹은 쓸 수 있는가(set)만 따진다.

- `{ get }`: 읽기만 가능해도 됨 (let, get 전용 연산 프로퍼티 가능)
- `{ get set }`: 읽고 쓰기 모두 가능해야 함 (var, get/set 모두 있는 연산 프로퍼티 가능)

```Swift
class J: UIViewController, ViewDesignProtocol {
    var navigationTitle: String {
        get {
            let bg = UserDefaults.standard.integer(forKey: "theme")

            if bg == 1 {
                return "화이트 테마를 사용하고 있는 고래밥님"
            } else if bg == 2 {
                return "블랙 테마를 사용하고 있는 고래밥님"
            } else {
                return "레드 테마를 사용하고 있는 고래밥님"
            }
        }
        set {
            navigationItem.title = newValue
        }
    }
}
```

### `newValue`는 어디서 온 걸까?

연산 프로퍼티(Computed Property)의 `set` 블록에서만 사용되는 특수한 예약어다.

## 4. 보안 & 기타 팁

- `.gitignore`: API 키와 같이 민감한 파일은 깃허브에 올리지 않도록 설정한다.
- `ActivityIndicator`: 네트워크 통신 중에 유저가 답답하지 않게 '로딩 중'임을 알려주는 UI 요소다.
- QuickType.io: JSON을 Swift 구조체로 바꾸어주는 유용한 도구이지만 `Optional` 처리가 미흡하다는 점 참고.
