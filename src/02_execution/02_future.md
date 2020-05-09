# `Future` Trait

`Future` trait은 Rust의 비동기 프로그래밍의 중심에 있습니다.
`Future`는 값을 생성 할 수있는 비동기 연산입니다.
(이 값은 비어있을 수 있습니다. 예 :`()`). *simplified* 버전의
future trait은 다음과 같습니다.

```rust
{{#include ../../examples/02_02_future_trait/src/lib.rs:simple_future}}
```

`poll` 함수를 호출하여 `Future`을 실행할 수 있습니다.
이러한 호출은 future를 가능한 한 완수를 목표로 합니다. `Future`가 완료되면
`Poll::Ready(result)`를 반환합니다. Future가 아직 완료되지 않았으면
`Poll::Pending`을 반환하고 `Future`가 더 진행 할 준비가 되었을 때`wake()`함수가 호출되도록 해 주어야 합니다.
`wake()`가 호출되면
`Future`를 실행하는 executor는 `Future`를 다시 호출하여 `Future`가
중단되었던 지점에서 실행을 계속 하도록 합니다.

`wake()`가 없으면 executor는 특정 future가 언제 다시 실행 가능한 지를 알 방법이 없으므로,
항상 모든 future를 polling 해야 합니다.
`wake()`를 사용하면, executor는 어떤 future가 실행 가능한지를 polling 해야 하는지 정확히 알 수 있습니다.

예를 들어, 사용 가능한 데이터가 있을 수도, 없을 수도 있는 소켓을 읽는 경우를 고려하십시오.
데이터가 있으면 읽을 수 있고 `Poll::Ready(data)`를 반환하지만
준비된 데이터가 없으면 future는
block되며 더 이상 진행할 수 없습니다. 사용 가능한 데이터가 없으면
소켓에서 데이터가 준비되면 호출 될 `wake`를 등록해야 합니다.
그것은 우리의 future가 재실행 할 준비가 되었다고 ezecutor에게 알려줄 것입니다.
간단한 `SocketRead` future는 다음과 같습니다.

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:socket_read}}
```

이 `Future` 모델은 중간 할당이 필요없는 여러 개의 비동기 동작을 하는 여러 future를 구성 할 수 있습니다.
한 번에 여러 개의 futures, 또는 chained futures를 allocation-free
state machines을 사용하여 실행되는 코드를 구현할 수 있습니다.

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:join}}
```

이것은 여러 futures를 독립적인 각각의 할당 없이 동시에 실행할 수 있는 방법을 보여 줍니다.
이러한 기능은 보다 효율적인 비동기식 프로그램을 허용합니다.
마찬가지로 다음과 같이 여러 순차 futures을 차례로 실행할 수도 있습니다.

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:and_then}}
```

이 예는 `Future` trait을 사용하여 여러 개의 할당된 객체와 깊이 중첩된 콜백 없이
비동기 제어를 표현하는 방법을 보여줍니다.
기본적인 제어 흐름에 대해 설명하였으니. 
실제 `Future` trait과 그것의 차별성을 살펴보겠습니다.

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:real_future}}
```

가장 먼저 눈에 띄는 변화는 `self` type이 더 이상 `&mut self`가 아니라는 것입니다.
이 것은 `Pin<&mut Self>`로 변경 되었습니다. 나중에 Pin에 대해 자세히 설명하겠습니다[pinning].
지금은 우리가 만드는 future가 재배치 될 수 없다는 것 정도로 이해하고 넘어 갑시다.
재배치 될 수 없는 객체는 필드 사이에 포인터를 저장할 수 있습니다.
예 : `struct MyFut { a: i32, ptr_to_a: * const i32 }`.
Pinning은 async/await를 활성화 하는데 필요합니다.

둘째, `wake: fn()`은 `&mut Context<'_>`로 변경되었습니다.
`SimpleFuture`에서
우리는 함수 포인터 (`fn()`)을 호출하여 future의 executor에게
해당 future는 poll되어야 한다고 알려 주었습니다. 그러나 fn()은
함수 포인터일 뿐이고 이를 사용하면 future에 대한 호출과 관련된 데이터를 저장할 수 없습니다.

실제 시나리오에서, 웹 서버와 같은 복잡한 응용 프로그램은
수천 개의 서로 다른 커넥션과 관련된 웨이크 업이 모두 별도로 관리 되어야 
됩니다. `Context` type은
특정 작업을 깨우는 데 사용할 수있는 `Waker` type의 값에 접근하도록 해 주어서 이 문제를 해결 합니다.

[pinning]: ../04_pinning/01_chapter.md
