# Day 12 - 수업 내용 복습 정리

> 중요한 내용!!! CollectionView Layout & UserDefaults(데이터 영속성)

오늘 배운 내용은 iOS 개발에서 '화면'을 넘어 '데이터'와 '성능(시점)'의 영역으로 들어가는 아주 중요한 지점들이다. 특히 View Drawing Cycle이나 Sandbox System 같은 개념은 면접에서도 왜 그렇게 코드를 짰는지 그러한 질문으로 자주 등장하는 단골 주제다.

## 1. CollectionView Layout: 수식의 비밀

컬렉션뷰에서 셀의 크기를 계산할 때 왜 특정 숫자를 곱하고 빼는지 명확하게 이해하는 것이 중요하다.

### 셀 너비 계산 공식

```Swift
let width = UIScreen.main.bounds.width - (inset * 2) - (spacing * 2)
```

왜 $inset \times 2$와 $spacing \times 2$를 빼는가? 3단 배열을 예로 들어보자.

* Inset (여백): 화면의 왼쪽 끝과 오른쪽 끝, 총 2군데에 존재한다. $\rightarrow (inset \times 2)$
* Spacing (간격): 셀이 3개라면 그 사이의 간격은 2군데다. 셀이 $n$개라면 간격은 $(n-1)$개.
* 공식:
$$\text{Cell Width} = \frac{\text{Device Width} - (\text{Inset} \times 2) - (\text{Spacing} \times (\text{Cell Count} - 1))}{\text{Cell Count}}$$

### UIScreen.main.bounds가 Deprecated된 이유

iOS 26(및 최신 버전)에서 이 코드가 경고를 띄우는 이유는 **멀티태스킹과 아이패드 대응** 때문이다.

* 앱이 전체 화면이 아니라 Split View 등으로 화면의 절반만 차지할 수도 있는데, `UIScreen`은 기기 전체의 크기를 가져오기 때문이다.
* 따라서 현재 앱이 보여지는 창의 크기를 뜻하는 `view.window?.windowScene?.screen.bounds` 사용을 권장하는 것이다.

## 2. ⭐️ UserDefaults & iOS Sandbox System

데이터를 저장할 때 가장 먼저 만나는 '사물함' 시스템이다.

### iOS Sandbox System이란?

애플은 보안을 위해 앱마다 독립된 사물함을 배정한다.<br />
다른 앱의 사물함을 절대 열어볼 수 없다.<br />
이 사물함은 크게 4가지 공간으로 나뉜다.

1. Documents: 사용자가 생성한 콘텐츠나 설정 파일 저장 (백업 가능, 예: 카톡 채팅 데이터).
2. Library: 앱의 설정, 캐시 등을 저장.
  * UserDefaults가 바로 이 Library 안의 `Preferences` 폴더에 `.plist` 형태로 저장된다.
3. Caches: 임시 파일 (용량이 부족하면 OS가 자동 삭제).
4. System Data: 앱 실행을 위한 필수 시스템 파일.

### UserDefaults vs Database 비교

| 비교 항목 | UserDefaults | Database (Core Data, Realm, SQLite) |
| :--- | :--- | :--- |
| **데이터 규모** | 경량 데이터 (단일 값, 간단한 정보) | 중량 데이터 (복잡한 구조, 대량의 정보) |
| **저장 방식** | Key-Value (딕셔너리 형태) | Table / Object (엑셀 또는 객체 형태) |
| **데이터 형식** | String, Int, Bool, Data 등 기본 타입 | 모델 객체, 관계형 데이터 |
| **사용 목적** | 사용자 설정(다크모드, 알림), 자동 로그인 여부 | 게시글 목록, 채팅 내역, 영화 정보 관리 |
| **특징** | 쓰기/읽기가 간편하나 대용량 시 성능 저하 | 쿼리(Query)를 통한 검색, 정렬, 필터링 용이 |
| **영속성** | 로컬(샌드박스) 저장, 앱 삭제 시 함께 삭제 | 로컬(샌드박스) 저장, 앱 삭제 시 함께 삭제 |

### Nil vs 0 (데이터의 존재 유무 확인)

* String: "글자"가 없으면 아예 상자 자체가 비어있는 `nil`을 반환한다.
* Int: 숫자는 상자가 비어있어도 기본값인 `0`을 반환한다.
  * 따라서 닉네임은 `nil`인지 확인하고, 버튼 클릭 횟수는 `0`인지 확인하는 방식으로 데이터의 존재 여부를 판단하는 것이 효율적이다.

## 3. View Drawing Cycle: "왜 내 동그라미는 마름모인가?"

이 부분은 정말 중요한 **시점**의 문제다.

### 원인과 해결 (DispatchQueue.main.async)

1. 문제 발생: `cellForRowAt`에서 `cornerRadius`를 설정할 때, 셀은 아직 XIB에 설정된 '임시 크기'를 가지고 있다. 실제 오토레이아웃이 적용되기 전이다.
2. 결과: 작은 크기일 때 반으로 깎았다가, 나중에 셀이 화면 크기에 맞춰 길어지면 깎인 모양이 유지되어 마름모처럼 보인다.
3. 해결: `DispatchQueue.main.async` 블록에 코드를 넣으면, "메인 스레드야, 지금 하고 있는 '뷰 그리기' 일 다 끝나고 나서 이 코드를 실행해줘"라고 예약하는 것과 같다.
4. 효과: 뷰의 크기가 확정된 직후에 `cornerRadius`가 계산되므로 완벽한 정원이 된다.

즉, 나중에 커질 걸 모르고 미리 깎아버려서 생기는 문제다. 다 커진 다음에 깎으라고 시키는 것이 `async`의 역할!