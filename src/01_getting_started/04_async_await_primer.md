# `async`/`.await` 입문

`async`/`.await`는 비동기 함수를 동기 코드처럼 보이도록 해주기 위한 Rust의 내장 툴입니다.
`async`는 코드 블록을
`Future`라는 trait을 구현하는 상태 머신으로 변환해 줍니다.
동기 함수에서 blocking함수의 호출은 전체 스레드를 blocking하는 반면에
blocking된 `Future`는 스레드의 제어를 내어 놓아 다른 타스크의 실행을 허용합니다.
이는 다른 `Future`의 실행을 허용합니다.

비동기 함수를 만들려면 `async fn` 구문을 사용할 수 있습니다 :

```rust,edition2018
async fn do_something() { /* ... */ }
```

`async fn`에 의해 반환되는 값은 `Future`입니다. 무슨 일이 일어나려면
`Future`는 executor 프로그램에서 실행 되어야 합니다.

```rust,edition2018
{{#include ../../examples/01_04_async_await_primer/src/lib.rs:hello_world}}
```

`async fn` 안에서 `.await`를 사용하여
`Future` trait을 구현하는 다른 type (예 :
또 다른`async fn`의 반환 값)의 실행이 완성되기를 기다릴 수 있습니다.
`block_on`과 달리`.await`는 현재의 스레드를 block하지 않습니다.
대신 Future가 완료 될 때 까지 비동기 적으로 기다립니다.
이는 만약 현재의 future가 진행될 수 없는 경우 다른 타스크가 실행되도록 해줍니다.

예를 들어, 우리가 3개의 `async fn`: `learn_song`, `sing_song`,
그리고 `dance` 를 가지고 있다고 해봅시다.

```rust,ignore
async fn learn_song() -> Song { /* ... */ }
async fn sing_song(song: Song) { /* ... */ }
async fn dance() { /* ... */ }
```

배우고, 노래하고, 춤을 추는 한 가지 방법은 각각을 개별적으로 block하는 것입니다:

```rust,ignore
{{#include ../../examples/01_04_async_await_primer/src/lib.rs:block_on_each}}
```

그러나 우리는 이런 식으로 최고의 성능을 제공하지 않습니다.
한 번에 한 가지 작업만 수행하고 있습니다! 분명히 우리는 노래를 부르기 전에 노래를 배워야 합니다.
하지만 우리는 노래를 배우거나 부르는 동시에 춤을 출 수 있습니다.
이를 위해 동시에 실행할 수 있는 두 개의 `async fn`을 만들 수 있습니다:

```rust,ignore
{{#include ../../examples/01_04_async_await_primer/src/lib.rs:block_on_main}}
```

이 예에서 노래를 부르기 전에 노래를 배우는 것이 필요하지만
노래 학습과 노래 부르기는 춤과 동시에 일어날 수 있습니다. 우리가 
`learn_and_sing`의 `learn_song().await` 대신 `block_on(learn_song())`을 사용했다면,
`learn_song`이 실행되는 동안 스레드는 다른 작업을 수행 할 수 없습니다.
이렇게 하면 노래와 동시에 춤을 출 수 없게됩니다.
`learn_song` future를 `.await`~ing 함으로써, 우리는 `learn_song`이 block 된 경우
다른 작업이 현재 스레드를 인계받을 수 있도록 할 수 있습니다.
이를 통해 여러 타스크를 동일한 스레드에서 동시에 운영 할 수 있습니다.
이제 `async`/`await`의 기본 사항을 배웠으므로
예제를 살펴봅시다.
