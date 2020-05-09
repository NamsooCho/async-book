# `Future` Trait

`Future` trait은 Rust의 비동기 프로그래밍의 중심에 있습니다.
`Future`는 값을 생성 할 수있는 비동기 연산입니다.
(이 값은 비어있을 수 있습니다. 예 :`()`). *simplified* 버전의
future trait은 다음과 같습니다.

```rust
{{#include ../../examples/02_02_future_trait/src/lib.rs:simple_future}}
```

'poll'함수를 호출하여 선물을 진행할 수 있습니다.
가능한 한 완성을 향한 미래. 미래가 완료되면
`Poll :: Ready (result)`를 반환합니다. 미래가 아직 완료되지 않으면
`Poll :: Pending`을 반환하고`wake ()`함수가 호출되도록 정렬
'미래'가 더 발전 할 준비가되었을 때. `wake ()`가 호출되면
'미래'를 실행하는 집행자는 '미래'를 다시 호출하여 '미래'가
더 많은 진전을 이루십시오.

`wake ()`가 없으면 실행자는 특정 시점을 알 방법이 없습니다.
미래는 진전을 이룰 수 있으며 항상 모든 것을 폴링해야합니다.
미래. `wake ()`를 사용하면, 집행자는 어떤 선물이 준비되어 있는지 정확히 알 수 있습니다
'폴링'됩니다.

예를 들어, 소켓에서 읽을 수있는 경우를 고려하십시오.
사용 가능한 데이터가 없을 수도 있습니다. 데이터가 있으면 읽을 수 있습니다
`Poll :: Ready (data)`를 반환하지만 준비된 데이터가 없으면 미래는
차단되었으며 더 이상 진행할 수 없습니다. 사용 가능한 데이터가 없으면
소켓에서 데이터가 준비되면 호출 될 'wake'를 등록해야합니다.
그것은 우리의 미래가 진보 할 준비가되었다고 집행자에게 알려줄 것입니다.
간단한 'SocketRead'미래는 다음과 같습니다.

Futures can be advanced by calling the `poll` function, which will drive the
future as far towards completion as possible. If the future completes, it
returns `Poll::Ready(result)`. If the future is not able to complete yet, it
returns `Poll::Pending` and arranges for the `wake()` function to be called
when the `Future` is ready to make more progress. When `wake()` is called, the
executor driving the `Future` will call `poll` again so that the `Future` can
make more progress.

Without `wake()`, the executor would have no way of knowing when a particular
future could make progress, and would have to be constantly polling every
future. With `wake()`, the executor knows exactly which futures are ready to
be `poll`ed.

For example, consider the case where we want to read from a socket that may
or may not have data available already. If there is data, we can read it
in and return `Poll::Ready(data)`, but if no data is ready, our future is
blocked and can no longer make progress. When no data is available, we
must register `wake` to be called when data becomes ready on the socket,
which will tell the executor that our future is ready to make progress.
A simple `SocketRead` future might look something like this:

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:socket_read}}
```

This model of `Future`s allows for composing together multiple asynchronous
operations without needing intermediate allocations. Running multiple futures
at once or chaining futures together can be implemented via allocation-free
state machines, like this:

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:join}}
```

This shows how multiple futures can be run simultaneously without needing
separate allocations, allowing for more efficient asynchronous programs.
Similarly, multiple sequential futures can be run one after another, like this:

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:and_then}}
```

These examples show how the `Future` trait can be used to express asynchronous
control flow without requiring multiple allocated objects and deeply nested
callbacks. With the basic control-flow out of the way, let's talk about the
real `Future` trait and how it is different.

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:real_future}}
```

The first change you'll notice is that our `self` type is no longer `&mut self`,
but has changed to `Pin<&mut Self>`. We'll talk more about pinning in [a later
section][pinning], but for now know that it allows us to create futures that
are immovable. Immovable objects can store pointers between their fields,
e.g. `struct MyFut { a: i32, ptr_to_a: *const i32 }`. Pinning is necessary
to enable async/await.

Secondly, `wake: fn()` has changed to `&mut Context<'_>`. In `SimpleFuture`,
we used a call to a function pointer (`fn()`) to tell the future executor that
the future in question should be polled. However, since `fn()` is just a
function pointer, it can't store any data about *which* `Future` called `wake`.

In a real-world scenario, a complex application like a web server may have
thousands of different connections whose wakeups should all be
managed separately. The `Context` type solves this by providing access to
a value of type `Waker`, which can be used to wake up a specific task.

[pinning]: ../04_pinning/01_chapter.md
