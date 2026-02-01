# Day 17 - 수업 내용 복습 정리

## 1. Base Structure: 상속을 통한 코드 최적화

반복되는 `viewDidLoad` 설정과 `required init` 호출을 줄이기 위해 **Base 클래스**를 도입한다.

### BaseViewController & BaseTableViewCell

- 목적: 공통적인 UI 설정 메서드(`configureHierarchy`, `configureLayout`, `configureView`)의 호출 시점을 부모 클래스에서 관리하여 자식 클래스의 코드 양을 줄인다.
- Cell의 이점: `UITableViewCell`의 `init(style:reuseIdentifier:)` 내부에서 configure 메서드들을 미리 호출해두면, 자식 셀은 메서드 구현만 하면 된다.
- `@available(*, unavailable)`: 코드베이스 UI에서 사용하지 않는 `init?(coder:)`를 명시적으로 금지하여 안전성을 높인다.

## 2. identifier 개선: Protocol + Extension

클래스 이름을 수동으로 입력하던 방식에서 **메타 타입(`self`)** 을 활용한 자동화 방식으로 개선한다.

### ReusableViewProtocol

- `String(describing: self)`: 클래스 타입 그 자체를 문자열로 반환한다. (예: `BookTableViewCell`)
- Protocol Default Implementation: 프로토콜과 익스텐션을 결합해 `UIViewController`, `UITableViewCell` 등이 자동으로 자신의 이름을 `identifier`로 가지게 한다.

## 3. Pagination: 연속적인 데이터 처리

사용자가 리스트의 끝에 도달했을 때 다음 데이터를 불러오는 기술이다.

### 구현 방식(willDisplay)

- 시점: `willDisplay` 메서드에서 `indexPath.row`가 `list.count - 1`인 시점을 포착한다.
- 주의사항: 새로운 검색 시에는 `page = 1` 초기화와 기존 `list`를 `removeAll()` 해야 하며, 추가 데이터는 `.append(contentsOf:)`를 사용한다.
- 마지막 페이지 처리: 카카오 API의 `is_end` 같은 메타 데이터를 활용해 불필요한 네트워크 요청을 방지한다.

### Offset-based vs Cursor-based

면접에서 단골로 나오는 **도메인 지식**이다.

| 구분              | **Offset-based Pagination**                | **Cursor-based Pagination**             |
| :---------------- | :----------------------------------------- | :-------------------------------------- |
| **작동 방식**     | 페이지 번호(page)와 데이터 개수(size) 기준 | 마지막 데이터의 고유 ID(Cursor) 기준    |
| **요청 예시**     | `?page=2&size=30`                          | `?after_id=12345&size=20`               |
| **장점**          | 구현이 쉽고 특정 페이지로 직접 이동 가능   | 데이터 변화가 잦아도 **중복/누락 없음** |
| **단점**          | 데이터 삽입/삭제 시 결과 중복 위험 있음    | 구현이 까다롭고 페이지 건너뛰기 불가    |
| **성능**          | 뒤 페이지로 갈수록 서버 부하 증가 ($O(N)$) | 데이터 양에 상관없이 성능 일정 ($O(1)$) |
| **적합한 서비스** | 게시판, 도서 검색, 공지사항 등             | 인스타그램 피드, 채팅, 실시간 타임라인  |

## 4. iOS의 3가지 Pagination 구현법

1. `willDisplay` 메서드: 가장 직관적이고 구현이 쉽다.
2. `scrollViewDidScroll` (Offset 기준): 스크롤 위치(`contentOffset.y`)를 계산하여 특정 지점에서 미리 호출할 수 있다.
3. `UITableViewDataSourcePrefetching`:

- iOS 10 이상 지원.
- 데이터를 미리 다운로드(`prefetch`)하거나 빠르게 스쳐 지나가는 셀의 다운로드를 취소(`cancelPrefetching`)할 수 있어서 대용량 이미지 처리에 유리하다.

```Swift
extension BookViewController: UITableViewDataSourcePrefetching {
    // 용량이 엄청 큰 상황에 필요한 친구
    func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath]) {
        // 미리 데이터를 다운로드 받기
    }

    // 취소하는 거
    func tableView(_ tableView: UITableView, cancelPrefetchingForRowsAt indexPaths: [IndexPath]) {
        // 스쳐지나간 이미지들은 다운로드받을 필요 없으니까, 취소하자 하는 기능
    }
}
```
