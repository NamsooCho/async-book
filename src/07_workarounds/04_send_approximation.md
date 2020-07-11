# `Send` 추정

일부 `async fn` 상태 시스템은 스레드를 통해 전송되는 것이 안전하지만
다른 것들은 그렇지 않습니다. `async fn` `Future` 가 `Send` 인지 여부가 결정되는 것은
non-`Send` 타입이 `.await` 지점에서 유지되는지 여부에 의합니다. 컴파일러는
객체가 `.await`에 걸쳐있을 때 `Send`에 근사하기 위해 최선을 다합니다.
그러나 이 분석은 오늘날 여러 곳에서 너무 보수적입니다.

예를 들어, 단순한 non-`Send`타입, 아마도 `Rc`가 들어 있는 타입을 생각해 보세요 :

```rust
use std::rc::Rc;

#[derive(Default)]
struct NotSend(Rc<()>);
```

`NotSend` 타입의 변수는 `async fn` 들에서 `async fn`에 의해 리턴 된 결과 `Future`타입 값이
`Send` 여야 하는 경우에도 일시적으로 나타날 수 있습니다 :

```rust,edition2018
# use std::rc::Rc;
# #[derive(Default)]
# struct NotSend(Rc<()>);
async fn bar() {}
async fn foo() {
    NotSend::default();
    bar().await;
}

fn require_send(_: impl Send) {}

fn main() {
    require_send(foo());
}
```

하지만 만약 우리가 `foo`가 `NotSend`를 변수에 담도록 수정하면 이 예제는 더 이상 컴파일 되지 않습니다 :

```rust,edition2018
# use std::rc::Rc;
# #[derive(Default)]
# struct NotSend(Rc<()>);
# async fn bar() {}
async fn foo() {
    let x = NotSend::default();
    bar().await;
}
# fn require_send(_: impl Send) {}
# fn main() {
#    require_send(foo());
# }
```

```rust
error[E0277]: `std::rc::Rc<()>` cannot be sent between threads safely
  --> src/main.rs:15:5
   |
15 |     require_send(foo());
   |     ^^^^^^^^^^^^ `std::rc::Rc<()>` cannot be sent between threads safely
   |
   = help: within `impl std::future::Future`, the trait `std::marker::Send` is not implemented for `std::rc::Rc<()>`
   = note: required because it appears within the type `NotSend`
   = note: required because it appears within the type `{NotSend, impl std::future::Future, ()}`
   = note: required because it appears within the type `[static generator@src/main.rs:7:16: 10:2 {NotSend, impl std::future::Future, ()}]`
   = note: required because it appears within the type `std::future::GenFuture<[static generator@src/main.rs:7:16: 10:2 {NotSend, impl std::future::Future, ()}]>`
   = note: required because it appears within the type `impl std::future::Future`
   = note: required because it appears within the type `impl std::future::Future`
note: required by `require_send`
  --> src/main.rs:12:1
   |
12 | fn require_send(_: impl Send) {}
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

error: aborting due to previous error

For more information about this error, try `rustc --explain E0277`.
```

이 오류는 정확 합니다. `x`를 변수에 저장하면 `.await` 이후까지 삭제되지 않습니다.
그 시점에서 `async fn`은
다른 스레드에서 실행될 수 있습니다. `Rc`는 `Send`가 아니기 때문에 스레드를 가로 질러 이동하면
매우 안 좋은 것이죠. 이것에 대한 간단한 해결책은 `.await` 이전에 `Rc`를 `drop` 하는 것입니다.
하지만 불행히도 이 방법은 오늘날에는 작동하지 않습니다.

이 문제를 성공적으로 해결하려면 non-`Send`의 변수를 캡슐화하는 블록 스코프를 도입해야 할 수도 있습니다.
이것은 컴파일러에게는 이러한 변수가 `.await` 지점을 가로질러 살아 있지 않다고 말해주므로 더 쉬워집니다 :

```rust,edition2018
# use std::rc::Rc;
# #[derive(Default)]
# struct NotSend(Rc<()>);
# async fn bar() {}
async fn foo() {
    {
        let x = NotSend::default();
    }
    bar().await;
}
# fn require_send(_: impl Send) {}
# fn main() {
#    require_send(foo());
# }
```
