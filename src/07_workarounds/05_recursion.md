# 재귀

내부적으로 `async fn`은 `.await`하고 있는 sub-`Future`들 각각을 포함하는 상태 머신 타입을 만듭니다.
이것은 재귀적인 `async fn`을 조금 tricky하게 만드는데,
그 이유는 결과 상태 머신 타입은 자신를 포함해야하기 때문입니다.

```rust,edition2018
# async fn step_one() { /* ... */ }
# async fn step_two() { /* ... */ }
# struct StepOne;
# struct StepTwo;
// This function:
async fn foo() {
    step_one().await;
    step_two().await;
}
// generates a type like this:
enum Foo {
    First(StepOne),
    Second(StepTwo),
}

// So this function:
async fn recursive() {
    recursive().await;
    recursive().await;
}

// generates a type like this:
enum Recursive {
    First(Recursive),
    Second(Recursive),
}
```

이것은 정상적으로 동작하지 않을 것입니다 - 우리는 무한한 크기의 타입을 만들었습니다!
컴파일러는 다음의 불평을 할 것입니다.

```
error[E0733]: recursion in an `async fn` requires boxing
 --> src/lib.rs:1:22
  |
1 | async fn recursive() {
  |                      ^ an `async fn` cannot invoke itself directly
  |
  = note: a recursive `async fn` must be rewritten to return a boxed future.
```

이것을 허용하기 위해서는, `Box`를 사용하여 우회해야 합니다.
불행히도, 컴파일러의 제한은 단순히 `recursive()` 에 대한 호출을 `Box::pin`으로 둘러 싼다고 해결되지 않을 것이라는 의미입니다.
이것을 해결하려면 우리는 `.boxed()` `async` 블록을 반환하는 non-`async` 함수에 재귀를 하도록 해야 합니다 :

```rust,edition2018
{{#include ../../examples/07_05_recursion/src/lib.rs:example}}
```
