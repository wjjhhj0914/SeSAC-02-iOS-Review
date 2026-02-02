# Day 21 - 수업 내용 복습 정리

## 1. QoS(Quality of Service): 작업의 우선순위

모든 백그라운드 작업이 똑같이 중요한 것은 아니다. `global()` 큐에 작업을 보낼 때, 중요도에 따라 에너지를 다르게 배분할 수 있다.

**Note**: 메인 스레드는 항상 최우선순위이므로 QoS 설정이 따로 없다.

## 2. DispatchGroup: 여러 비동기 작업의 끝을 알고 싶을 때

여러 개의 API 통신이나 이미지 다운로드가 **모두 완료된 시점**에 무언가(예: 화면 갱신)를 하고 싶을 때 사용한다.

### 2-1. 단순 notify 방식 (동기 코드용)

작업들이 단순히 `print`나 계산 같은 동기 코드라면 `group.notify`만으로 충분하다.

### 2-2. enter & leave 방식 (비동기 내부의 비동기용) ← 이게 핵심! ⭐️

네트워크 통신(Alamofire 등)은 그 자체로 비동기다. `async(group: group) 내부에 또 비동기 코드가 있으면, 시스템은 내부 작업이 끝나기도 전에 겉 작업이 끝났다고 착각한다. 이때 **참조 카운팅** 개념을 사용한다.

- `group.enter()`: "나 이제 비동기 작업 시작한다! (카운트 + 1)"
- `group.leave()`: "드디어 비동기 작업 끝났다! (카운트 - 1)"
- `group.notify()`: "카운트가 드디어 0이 되었구나! 최종 마무리하자~"

```Swift
let group = DispatchGroup()

group.enter() // 카운트 +1
NetworkManager.shared.fetchImage { info in
    // 성공하든 실패하든 무조건 호출되어야 함!
    defer { group.leave() } // 카운트 -1
    print("이미지 1 완료")
}

group.notify(queue: .main) {
    print("모든 이미지가 로드되었습니다. UI 업데이트!")
}
```

### 주의: `leave`가 호출되지 않으면 카운트가 0이 되지 않아서 `notify`가 영원히 실행되지 않는다. (UI Freezing 유발 가능)

## 3. 왜 이런 기술들이 등장했을까?

### Main Thread(닭 벼슬)의 임무

메인 스레드는 오직 **UI 업데이트**와 **유저 이벤트(클릭 등)** 처리에만 집중해야 한다. 무거운 작업(네트워크, 이미지 처리)을 메인에서 하면 화면이 멈추는 **UI Freezing**이 발생한다.

### GCD vs Swift Concurrency

- **GCD**: 알바생(Thread)을 **필요할 때마다 부르는** 방식. 너무 많이 부르면 알바생이 폭증하는 **Thread Explosion** 현상이 발생하여 메모리 효율이 떨어진다.
- **Swift Concurrency**: iOS 13+ 이후 도입. 시스템이 리소스를 효율적으로 관리하여 **최적의 알바생 수만 유지**한다.

## 4. Tips

1. Error Handling: 네트워크 통신 시 `case .failure` 블록에서도 반드시 `group.leave()`를 해주어야 한다. 안 그러면 실패한 작업 때문에 전체 `notify`가 안 뜬다.
2. Alamofire의 배려: Alamofire는 **내부적으로 이미 비동기 처리**를 하고 응답을 메인 스레드로 보내주기 때문에, 우리가 직접 `DispatchQueue.global().async`로 감쌀 필요는 없다. (알고 쓰는 것과 모르고 쓰는 것의 차이!)
3. UI 업데이트는 언제나 Main: `notify(queue: .main)`으로 설정하면 클로저 내부에서 바로 `tableView.reloadData()`를 안전하게 호출할 수 있다.
