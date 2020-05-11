# `Stream` Trait

`Stream` trait은 `Future`와 유사하지만 종료하기 이전에 여러 개의 값을 생성 할 수 있습니다.
이느 표준 라이브러리의 `Iterator` trait과 유사합니다 :

```rust,ignore
{{#include ../../examples/05_01_streams/src/lib.rs:stream_trait}}
```

`Stream`의 일반적인 예로는 `futures` crate의 channel type의 `Receiver`가 있습니다.
`Sender`-end 에서 값이 전송 될 때마다 receiver에서는 `Some(val)`이 나옵니다.
그리고 `Sender`가 삭제되고 보류 중인 모든 메시지가 수신되면 `None`을 산출 할 것입니다.

```rust,edition2018,ignore
{{#include ../../examples/05_01_streams/src/lib.rs:channels}}
```
