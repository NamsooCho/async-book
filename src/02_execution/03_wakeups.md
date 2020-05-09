# `Waker`로 타스크 깨우기

futures가 처음에 `poll` 되어서 일을 완성 할 수 없는 경우가 일반적 입니다.
이 경우 futures는 더 진전 될 준비가 되면 polling 되도록 해야 합니다.
이것은 `Waker` type으로 이루어집니다.

futures가 polling 될 때마다 "task"의 일부로 polling됩니다. Task는
executor에게 제출 된 최상위 future 입니다.

`Waker`는 executor에게 해당 task가 깨어나야 한다고 말해주는 `wake()`메소드를 제공 합니다.
`wake()`가 호출되면 executor는
`Waker`와 관련된 작업이 진행될 준비가 되었음을 알고
future는 다시 폴링 됩니다.

`Waker`는 `clone()`도 구현하여 복사하고 저장할 수 있습니다.

`Waker`를 사용하여 간단한 타이머 future를 구현해 봅시다.

## 응용 : 타이머 만들기

예제를 단순회하기 위해 타이머가 시작될 때 새 스레드를 실행 시킵니다.
필요한 시간 동안 sleep 모드로 전환 한 다음, 시간 구간이 경과했을 때 타이머에 신호를 보냅니다.

시작하면서 해야 할 imports는 다음과 같습니다.

```rust
{{#include ../../examples/02_03_timer/src/lib.rs:imports}}
```

future type 자체를 정의하며 시작하겠습니다. 우리의 future는
타이머가 경과하고 future가 완료 되어야 함을 스레드에게 알려주어야 할 방법이 있어야 합니다.
공유 된 `Arc<Mutex<.. >>` 값을 사용하여 스레드와 future 간의 통신을 하도록 하겠습니다.

```rust,ignore
{{#include ../../examples/02_03_timer/src/lib.rs:timer_decl}}
```

이제, 실제로 `Future`를 구현하도록 하겠습니다!

```rust,ignore
{{#include ../../examples/02_03_timer/src/lib.rs:future_for_timer}}
```

아주 간단 하죠? 스레드가 `shared_state.completed = true`를 설정 한 경우
우리는 다한 것 입니다! 그렇지 않으면 현재 작업에 대한 `Waker`를 복제하여
스레드가 작업을 다시 시작할 수 있도록 `shared_state.waker`에게 넘겨주어야 합니다.

중요한 것은 future가 poll 될 때마다 `Waker`를 업데이트 해야 한다는 것입니다.
future는 다른 `Waker`를 가지고 다른 작업으로 이동 했을 수 있기 때문 입니다.
이것은 poll된 이후의 작업들 사이에 futures가 전달 될 때 발생합니다.

마지막으로 실제로 타이머를 구성하고 스레드를 시작하려면 관련 API가 필요합니다.

```rust,ignore
{{#include ../../examples/02_03_timer/src/lib.rs:timer_new}}
```

우와! 이것이 간단한 타이머 future를 만드는 데 필요한 전부입니다. 만약 우리가 future를 실행할 executor를 가지고 있다면 말이죠...
