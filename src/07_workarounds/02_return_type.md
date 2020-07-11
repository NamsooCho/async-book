# 타입 에러의 반환

일반적인 Rust 함수에서 잘못된 타입의 값을 반환하면
다음과 같은 오류가 발생했습니다.

```rust
error[E0308]: mismatched types
 --> src/main.rs:2:12
  |
1 | fn foo() {
  |           - expected `()` because of default return type
2 |     return "foo"
  |            ^^^^^ expected (), found reference
  |
  = note: expected type `()`
             found type `&'static str`
```

그러나 현재 `async fn`지원은 리턴 타입을 "신뢰" 할 수 없습니다.
함수 시그니쳐와 타입이 일치하지 않거나, 심지어 리턴 값으로부터 추정되는 함수 시그니쳐 오류도
이러한 문제에 포함됩니다.
예를 들어, 함수
`async fn foo () { "foo" }`는 다음의 오류를 발생시킵니다 :

```rust
error[E0271]: type mismatch resolving `<impl std::future::Future as std::future::Future>::Output == ()`
 --> src/lib.rs:1:16
  |
1 | async fn foo() {
  |                ^ expected &str, found ()
  |
  = note: expected type `&str`
             found type `()`
  = note: the return type of a function must have a statically known size
```

에러 메시지는 `&str`을 예상하고 `()`를 찾았다고 말합니다.
실제로 원하는 것과 정반대 입니다. 그 이유는
컴파일러가 함수 본문을 잘못 신뢰하여 본분이 올바른 타입을 리턴 한다고 생각하기 때문입니다.

이 문제의 해결 방법은
"`SomeType`을 예상하고`OtherType`을 찾았습니다"라는 메시지가있는 함수 시그니쳐를 가리키는 오류는
일반적으로 하나 이상의 리턴 포인트가 잘못 되었음을 나타낸다고 알아차리면 됩니다.

이 이슈는 현재 이 트랙에서 논의되고 있습니다. [this bug](https://github.com/rust-lang/rust/issues/54326).
