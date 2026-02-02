# Day 20 - 수업 내용 복습 정리

## 1. 역값전달: Closure vs Delegate vs Notification

데이터가 '앞 화면(Push/Present된 화면)'에서 '뒷 화면(이전 화면)'으로 넘어가는 과정이다.

### 🔄 역값전달 방식 비교 (Closure vs Delegate vs Notification)

| 구분            | **Closure (클로저)**                | **Delegate (대리자)**                | **Notification (알림)**                   |
| :-------------- | :---------------------------------- | :----------------------------------- | :---------------------------------------- |
| **연결 관계**   | 1 : 1 (직접 연결)                   | 1 : 1 (프로토콜 기반)                | **1 : N (방송국 기반)**                   |
| **주요 특징**   | 함수를 변수에 담아 전달함           | 프로토콜로 약속된 메서드만 대행함    | 센터에 등록된 모든 곳에 신호 보냄         |
| **장점**        | 코드가 간결하고 흐름이 직관적임     | 역할 분담이 명확하고 구조가 안정적임 | 서로 몰라도 데이터 전달 가능 (Decoupling) |
| **단점**        | 순환 참조(Memory Leak) 발생 위험    | 작성할 코드(프로토콜 등)가 많음      | 어디서 신호를 받았는지 추적이 어려움      |
| **적합한 상황** | 간단한 값 전달, 한 곳에만 전달할 때 | 복잡한 로직 전달, 구조적 설계 시     | 앱 전체에 동시 알림이 필요할 때           |

### 역값전달 실전 흐름

1. Closure: `NextVC`에 `var closureData: ((String) -> Void)?` 공간을 만들고, `pop` 하기 직전에 `closureData?(텍스트)`를 실행한다.

2. Delegate: `protocol`을 선언하고 `NextVC`에 `delegate` 변수를 만든다. `PrevVC`에서 `vc.delegate = self`로 대리인을 자처한다.

3. Notification: `Post`는 <u>방송 송출</u>, `AddObserver`는 <u>채널 고정</u>이다. 반드시 **송출 전에 채널 고정**이 되어 있어야 정보를 받는다.

## 2. Singleton Pattern

앱 전체에서 **단 하나의 인스턴스**만 존재하도록 보장하는 디자인 패턴이다.

- **구현**: `static let shared = 클래스명()`과 `private init()`을 통해 외부 생성을 차단한다.
- **활용**: `NetworkManager`, `UserDefaultManager` 등 공유 자원을 관리할 때 사용한다.
- **주의**: 전역 상태를 가지므로 메모리 관리에 유의해야 하며, 구조체(`struct`)는 값 복사가 일어나므로 싱글톤의 목적(유일성)에 어긋나 주로 클래스로 구현한다.

## 3. GCD(Grand Central Dispatch): 동기 vs 비동기

iOS에서 멀티 쓰레딩을 관리하는 가장 기본적인 방식이다. 큐라는 대기열에 작업을 던지면, 시스템이 알아서 쓰레드에 분배한다.

### 핵심 개념 비교

- Sync(동기): 작업이 끝날 때까지 기다림. (현재 쓰레드 정지)
- Async(비동기): 작업을 던져두고 바로 다음 코드 실행. (현재 쓰레드 진행)
- Serial(직렬): 작업을 한 쓰레드에 몰아넣음. (순서 보장) → `DispatchQueue.main`
- Concurrent(동시): 여러 쓰레드에 작업을 분배함. (순서 미보장) → `DispatchQueue.global()`

### 주의해야 할 조합: Sync + Serial(Main)

`DispatchQueue.main.sync`는 절대 사용하면 안 된다. 메인 쓰레드가 자기 자신에게 작업을 맡기고 기다리는 **데드락(Deadlock)** 상태에 빠져 앱이 즉시 터진다.

```Swift
// ✅ 가장 많이 쓰는 패턴: 비동기로 데이터 받고, UI 업데이트는 메인에서!
DispatchQueue.global().async {
    // 1. 시간이 오래 걸리는 작업 (네트워크, 이미지 처리)
    let data = try? Data(contentsOf: url)

    DispatchQueue.main.async {
        // 2. UI 업데이트는 반드시 메인 쓰레드에서!
        self.imageView.image = UIImage(data: data)
    }
}
```

### `asyncConcurrent` 결과가 매번 다른 이유

결과가 매번 다른 이유는 **일 분배 방식**과 **쓰레드의 컨디션** 때문이다.

1. 동시(Concurrent) 분배: 매니저(Queue)가 작업을 받자마자 여러 알바생(Thread)에게 일을 뿌린다. 1번 작업은 알바A에게, 2번 작업은 알바B에게 주는 것이다.
2. 비동기(Async) 실행: 매니저는 일을 맡기자마자 다음 일을 하러 떠난다. 알바생이 일을 끝냈는지 확인하지 않는다.
3. 예측 불가능한 변수:

- 알바A는 손이 빠를 수도, 알바B는 갑자기 재채기가 나와서 0.1초 늦어질 수도 있다.
- CPU(사장님)가 갑자기 다른 중요한 앱의 작업을 먼저 처리하라고 쓰레드에게 명령할 수도 있다.
  결론적으로, 작업을 **동시에 여러 명**에게 맡겼는데(`Concurrent`), **끝나는 걸 기다려주지도 않으니**(`Async`), 누구의 작업이 먼저 프린트문에 도달할지는 실행할 때마다 시스템의 상황에 따라 달라지게 되는 것이다.

## 4. Tips

1. Alamofire의 비밀: `AF.request`는 내부적으로 이미 `global().async`하게 동작하며, 응답 클로저(`responseDecodable`)는 `main.async`하게 동작하도록 설계되어 있어서 우리가 편하게 UI를 바꿀 수 있는 것이다.
