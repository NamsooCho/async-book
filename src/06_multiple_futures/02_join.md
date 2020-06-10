# `join!`

`futures::join`매크로는 동시에 future 모두가 실행되는 동안, 다른 여러 future들이 완료 될 때까지 기다릴수 있게 합니다.

# `join!`

여러 비동기 작업들이 수행되어지고 있을때, 단순히 일련의 `.await` 작업들을 하려는
경향이 있습니다:

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:naiive}}
```

그렇지만, `get_book`이 완료된 이후까지도 `get_music`이 시작하지 않을 것이기 
때문에 이것은 필요 이상으로 느립니다. 몇몇 다른 언어들에서, future들은 주변에서
완료됩니다. 그래서 두 작업들은 future들을 시작하기 위해 각 `async fn` 우선 
부름으로써 동시에 실행될 수 있습니다, 그리고 둘다 작업을 기다립니다:


```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:other_langs}}
```

그렇지만, 러스트 future들은 적극적으로 `.await`되어질때까지 작동하지 않을것입니다.
이것은 위의 두 코드 snippet들이 동시에 실행 되는것보다 차례대로 `book_future`와  `music_future`이 둘 다 실행될것입니다. 
두 future들을 동시에 정확하게 실행하기 위해서, `futures::join!`을 사용해보세요:

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:join}}
```

`join!`으로부터 리턴되는 값은 통과된 각 `Future`의 출력을 포함하는 하나의 튜플입니다.

## `try_join!`

`Result`를 반환하는 future들은, `join!`보다 `try_join!`을 사용하는 것을 생각해봅시다. 

`join!`은 모든 subfuture들이 완료하면 완료만 하기 때문에, subfuture들 중 하나가 `Err`를 리턴한 후에도 다른 future들을 처리하는것을 계속할것입니다.

`join!`과 달리, `try_join!`은 subfuture들 중 하나가 에러를 리턴하면 즉시 완료할 것입니다.

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:try_join}}
```

`try_join!`을 통과한 future들은 모두 같은 에러 타입을 가져야 된다는것을 알아야합니다.
`futures::future::TryFutureExt`에서 에러 타입들을 합치기 위해서 `.map_err(|e| ...)` 와 `.err_into()`함수들을 사용하는 것을 생각해보세요.

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:try_join_map_err}}
```
