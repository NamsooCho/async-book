# `async`/`.await`

[1 장]에서 우리는 `async`/`.await`에 대해 간단히 살펴 보았고
간단한 서버를 구축하는데 사용했습니다. 이 장에서는 `async`/`.await`에 대해 일반론 적인 설명과
작동 방식 및 비동기 코드와 전통적인 Rust 프로그램 과의 차이점에 대해 자세히 설명합니다.

`async`/`.await`는 Rust 구문의 특별한 부분으로
block하지 않고 현재 스레드의 제어를 내어 놓아서 작업이 완료되기를 기다리는 동안 다른 코드가 진행될 수 있게 해줍니다.

`async`를 사용하는 두 가지 주요 방법이 있습니다 : `async fn` 및 `async` 블록.
각각은 `Future` trait을 구현하는 값을 반환합니다 : 

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:async_fn_and_block_examples}}
```

첫 장에서 보았듯이 `async` 본문과 다른 futures는 게으릅니다.
그들은 execute 될 때까지 아무 것도 하지 않습니다. `Future`를 실행하는 가장 일반적인 방법은
`.await` 입니다. `.await`가 `Future` 에서 호출되면 완료까지 실행을 시도합니다.
`Future`가 block되면 제어권을 내어 놓아 다른 future가 제어를 얻을 수 있습니다.
더 많은 진전이 가능할 때 해당 `Future`가 executor에 의해 선택됩니다.
Executor에 의해 실행을 재개하여 `.await`가 완결 되도록 합니다.

## `async` 샘영주기

전통적인 함수와 달리, reference 또는
non-static인 인수를 취하는 `async fn`는 생명주기가 인수의 생명주기에 묶여 있는 `Future`를 반환합니다 :

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:lifetimes_expanded}}
```

이는 `async fn`에서 반환 된 future가 정적이 아닌 인수가 여전히 유효한 동안에 `.await` 되여야 함을 의미합니다.
함수를 호출 한 직후에 future를 기다리는 `.await`의 경우
(`foo(&x).await` 와 같이) 이것은 문제가 되지 않습니다. 그러나 future를 저장하거나
다른 작업이나 스레드로 전송하면 문제가 될 수 있습니다.

reference를 인수로 갖는 `async fn`을 `static` future로 설정하는 일반적인 해결 방법은
`async` 블록 안에서 `async fn` 을 호출하면서 인수를 함께 엮어 주는 것입니다 :

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:static_future_with_borrow}}
```

인수를 `async` 블록으로 옮기면 수명이 연장되어 `good`에 대한 호출에서 리턴된 `Future`의 수명과 일치 되었습니다.

## `async move`

`async` 블록과 클로저는 보통의 클로져와 마찬가지로 `move` 키워드를 허용합니다.
`async move` 블록은 변수가 참조하는 데이터의 소유권을 갖고
, 현재 scope보다 오래 생염이 유지되도록 허용해 주지만
해당 변수를 다른 코드와 공유하는 기능을 포기합니다.

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:async_move_examples}}
```

## 멀티 스레드 Executor 에서의 `.await`

멀티 스레드 `Future` executor를 사용할 때 `Future`가 스레드 간에 이동 할 수 있습니다.
따라서 비동기 바디에 사용 된 모든 변수는 스레드 사이에 이동 가능 해야 합니다.
`.await`는 잠재적으로 새로운 스레드로 전환 되는 결과를 가질 수 있습니다.

이것은 `Rc`, `&RefCell` 또는 다른 `Send` trait을 구현하지 않는 type을 사용하는 것이
안전하지 않다는 것을 의미합니다. 이 것은 `Sync`를 구현하지 않은 type을 사용하는 것에도 해당됩니다.

(주의: 이 type이 `.await`를 호출하는 동안의 scope 내에 있지 않는한 이 type 들을 사용할 수 있습니다.)

마찬가지로, future를 인식하지 못하는 전통적인 lock을 `.await`를 가로 질러
유지하는 것은 좋지 않습니다.
왜냐하면 스레드 풀이 잠길 수 있기 때문입니다 : 하나의 작업이
`.await` lock을 갖고 있다가 executor에 제어를 양보하여 다른 작업을 수행 할 수 있는데
그 다른 작업이 lock하려고 하면 교착 상태를 유발합니다. 이것을 피하려면
`std::sync`에 있는 것이 아니라 `futures::lock`에 있는 `Mutex`를 사용하십시오.

[1 장]: ../01_getting_started/04_async_await_primer.md
