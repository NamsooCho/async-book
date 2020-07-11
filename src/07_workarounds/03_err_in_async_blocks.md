# `?` in `async` 블록

`async fn` 에서와 마찬가지로 `async` 블록 안에서 `?` 를 사용하는 것이 일반적입니다.
그러나 `async` 블록의 반환 타입은 명시 적으로 언급되지 않았습니다.
이로 인해 컴파일러가 `async` 블록의 오류 타입을 유추하지 못할 수 있습니다.

예를 들자면 다음의 코드 입니다 :

```rust,edition2018
# struct MyError;
# async fn foo() -> Result<(), MyError> { Ok(()) }
# async fn bar() -> Result<(), MyError> { Ok(()) }
let fut = async {
    foo().await?;
    bar().await?;
    Ok(())
};
```

위 코드는 다음의 오류를 만들어 냅니다 :

```rust
error[E0282]: type annotations needed
 --> src/main.rs:5:9
  |
4 |     let fut = async {
  |         --- consider giving `fut` a type
5 |         foo().await?;
  |         ^^^^^^^^^^^^ cannot infer type
```

불행히도, 현재는 "타입을 `fut`에 부여" 하는 방법이 없습니다.
또한 `async` 블록의 반환 타입을 명시적으로 지정할 방법도 없습니다.
이 문제를 해결 하려면 "turbofish" 연산자를 사용하여 `async` 블록의 성공 및
오류 타입을 제공하여야 합니다 :

```rust,edition2018
# struct MyError;
# async fn foo() -> Result<(), MyError> { Ok(()) }
# async fn bar() -> Result<(), MyError> { Ok(()) }
let fut = async {
    foo().await?;
    bar().await?;
    Ok::<(), MyError>(()) // <- 여기에 명시적인 타입을 준 것을 눈여겨 보세요.
};
```
