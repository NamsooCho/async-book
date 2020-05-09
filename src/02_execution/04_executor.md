# 응용: Executor 만들기

Rust의 `Future`는 게으르다 : 그들은 적극적으로 추진하지 않으면 아무 것도 하지 않을 것이다.
future를 완료시키는 한 가지 방법은 `async` 함수 안에서 `.await`를 사용 하는 것 입니다.
이지만 문제를 한 단계 위로 올릴 뿐입니다.
최상위 `async` 함수에서 반환 된 future을 누가 실행 합니까? 정답은
우리는`Future` executor가 필요합니다.

`Future` executor 들은 최상위 `Future` 세트를 가져 와서 완성까지
`Future`가 진행될 수 있을 때 마다 `poll`을 호출하여 타스크를 진행 시킵니다.
일반적으로
executor는 future에 먼저 한 번 `polling`을 시작합니다. `Future`가
`wake()`를 호출하여 진행할 준비가 되었음을 표시하면, 그들은 큐에 다시 배치되고
`poll`이 다시 호출되어 `Future`가
완료될 때까지 이 과정이 반복됩니다.

이 섹션에서는 대규모로 future를 실행할 수 있는 간단한 executor 프로그램을 작성합니다.

이 예에서는 `ArcWake` trait을 구현한 `futures` crate에 의존합니다.
이 것은 `Waker`를 구성하는 쉬운 방법을 제공합니다.

```toml
[package]
name = "xyz"
version = "0.1.0"
authors = ["XYZ Author"]
edition = "2018"

[dependencies]
futures = "0.3"
```

다음으로, 우리는 다음의 의존성을 `src/main.rs`의 맨 위에 추가합니다 :

```rust,ignore
{{#include ../../examples/02_04_executor/src/lib.rs:imports}}
```

우리의 executor는 채널을 통해 실행할 작업을 보내서 작동합니다. Executor는
채널에서 이벤트를 가져 와서 실행합니다. 작업이 재 실행 될 준비가 되면(awoken)
그것은 자신을 재 스케즆링 하여 다시 폴링되도록 예약 할 수 있습니다.
재 스케쥴링은 자신을 채널로 다시 넘겨 줌으로써 이루어 집니다.

이 디자인에서 executor 프로그램 자체는 타스크 채널의 수신 end-point만 필요합니다.
사용자는 새로운 futures을 spawn할 수 있도록 송신 end-point을 얻습니다.
타스크 자체는 자신을 재 스케쥴링이 가능한 future 일 뿐이므로
작업을 다시 큐에 보내는 데 사용할 수 있는 sender와 pairing 된 future로 저장 됩니다.

```rust,ignore
{{#include ../../examples/02_04_executor/src/lib.rs:executor_decl}}
```

새로운 futures을 쉽게 spawn 할 수 있는 method을 spawner에 추가합시다.
이 방법은 future의 type을 취하여 box에 넣고, 다음과 같이 새로운 `Arc<Task>`를 만듭니다.
이것은 executor의 큐에 넣을 수 있는 객체가 됩니다.

```rust,ignore
{{#include ../../examples/02_04_executor/src/lib.rs:spawn_fn}}
```

future을 폴링하려면 `Waker`를 만들어야 합니다.
[Task 깨우기 섹션]에서 논의한 바와 같이, `Waker`는
`wake`가 호출되면 작업이 다시 폴링되도록 예약하는 책임이 있습니다.
`Waker`는 executor에게 어떤 작업이 준비 되었는지 정확하게 알려 준다는 점을 기억하세요.
그들은 전진 할 준비가 되어 있는 future에 대해서만 polling을 합니다. 새로운 `Waker`를 만들려면 가장 쉬운 방법이
`ArcWake` trait을 구현 한 다음
`waker_ref` 또는 `.into_waker()` 함수를 사용하여 `Arc<impl ArcWake>`
를 `Waker`로 만드는 것입니다. 우리의 타스크를 위해 `ArcWake`를 구현해 봅시다.

```rust,ignore
{{#include ../../examples/02_04_executor/src/lib.rs:arcwake_for_task}}
```

`Arc<Task>`에서 `Waker`가 생성되면 `wake()`를 호출하는 행위는
`Arc`의 복사본을 만들게 되고, 이 복사본은 타스크 채널로 전송 되도록 합니다. 우리의 executor는 그 다음
타스크를 선택하고 폴링 해야 합니다. 그것을 구현해 봅시다 :

```rust,ignore
{{#include ../../examples/02_04_executor/src/lib.rs:executor_run}}
```

축하합니다! 우리는 이제 futures executor를 갖고 있습니다. 우리는 그것을 `Async/.await`코드와 우리가 이미 작성 해본
`TimerFuture`와 같은 커스텀 future을 실행하기 위해
사용할 수 있습니다 :

```rust,edition2018,ignore
{{#include ../../examples/02_04_executor/src/lib.rs:main}}
```

[Task 깨우기 섹션]: ./03_wakeups.md
