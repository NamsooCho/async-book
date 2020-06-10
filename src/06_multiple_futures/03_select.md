# `select!`

`futures::select` 매크로는 어떤 future가 완료하자마자 사용자가 응답하는 것을 허락하는 다양한 future들을 실행합니다.

```rust,edition2018
{{#include ../../examples/06_03_select/src/lib.rs:example}}
```

위의 함수는 동시에 `t1`과 `t2`를 실행할것입니다.

`t1`나 `t2`가 끝날때, 해당하는 핸들러는 `println!`을 호출할것입니다. 

그리고 함수는 남아있는 작업을 완료하는것 없이 끝낼것입니다.

`select`의 기본 구문은 당신이 선택하고 싶은만큼의 많은 future들로 반복 되어지는    
`<pattern> = <expression> => <code>,`입니다.

## `default => ...` and `complete => ...`

`select` 또한 `defalut`와 `complete`분기를 지원합니다.

`default`분기는 `select`된 future들이 아무것도 완료되지 않으면 실행할것입니다.
`default`는 준비된 다른 future들이 없다면 실행되므로 `default`분기와 함께 `select`은 항상 즉시 반환할것입니다. 

`complete`분기들은 `select`가 된 모든 future들이 완료되어 더 이상 진행되지 않는
경우에 처리를 하는데 사용될 수 있습니다.
이것은 `select!`에 대해서 looping 할 때 꽤 편합니다.


```rust,edition2018
{{#include ../../examples/06_03_select/src/lib.rs:default_and_complete}}
```

## `Unpin` 및 `FusedFuture`와의 상호작용

위의 첫번째 예제에서 당신이 알렸을지도 모르는 것은 우리가 `pin_mut`로 future들을
고정할뿐만 아니라 두개의 `async fn`로부터 반환되는 future들에서 `.fuse()`를 불렀어야 했던것입니다.
이 두 호출은 `select`에서 사용되는 future들이 `Unpin`트레잇 과 `FusedFutre` 트레잇 둘 다 구현해야 하기때문에 필연적입니다.

`Unpin`은 `select`로부터 사용되는 future들이 값으로 가지는것이 아니라 mutable
reference로 되는 것이기 때문에 필수적입니다.
future의 소유권을 가지지 않음으로써, 완료되지 않은 future들은 `select`를 부른
이 후에 사용 될 수 있습니다.

마찬가지로, `select`가 완료한 후의 future를 폴링해서는 안되기 때문에 `FusedFuture`트레잇은 필요합니다. `FusedFuture`은 완료 여부를 추적하는 future들에 의해 구현됩니다.
이것은 아직 완료되지 않은 future들만 폴링하는 loop에서 `select`를 사용할 수 있게 합니다. 이것은 loop를 통해서 `a_fut` 또는 `b_fut`이 두번째 완료되는 곳을 위의 예제에서 볼 수 있습니다.
`future::ready`로부터 리턴되는 future는 `FusedFuture`를 구현하기 때문에, 다시 
poll하지 않는 `select`를 말할 수 있습니다.

스트림들은 상응하는 `FusedStream`을 가지는것을 알아야 합니다. 이 트레잇을 구현하거나 `.fuse()`를 사용하여 감싸진 스트림들은  스트림들의 `.next()`/ `.try_next()` 결합자들로부터 `FusedFuture`를 yiled할 것입니다.

```rust,edition2018
{{#include ../../examples/06_03_select/src/lib.rs:fused_stream}}
```

## Concurrent tasks in a `select` loop with `Fuse` and `FuturesUnordered`
## `Fuse`와 `FuturesUnordered` 와 함께 `select` 루프에서 동시발생 작업들 

다소 발견하기 어렵지만 편리한 함수는 `Fuse::terminated()`입니다,
이 함수는 이미 종료된 텅빈 future를 생성하는것을 허락하며, 나중에 실행해야 하는 future로 채워질 수 있습니다.

이것은 `select` loop 내부에 스스로 만들어지고 `select` loop 동안 실행되야할 작업이 있을때, 편리할 수 있습니다.

`.select_next_some()`의 사용을 알아야합니다. 이것은 `None`들을 무시하는 스트림에서 반환되는 Some(_)값들의 분기만을 실행하기 위해 `select`와 함께 사용될 수 있습니다.

```rust,edition2018
{{#include ../../examples/06_03_select/src/lib.rs:fuse_terminated}}
```

같은 future들의 많은 복사본들은 동시에 실행될때, `FutureUnordered` 타입을 사용하세요.

    아래의 예제는 위에것과 비슷하지만, 새로운 것이 만들어질때 중단하는것보다 `run_on_new_num_fut`의 각 복사본을 완료할것입니다. 또한 `run_on_new_num_fut`으로 리턴된 값을 출력할 것입니다.

```rust,edition2018
{{#include ../../examples/06_03_select/src/lib.rs:futures_unordered}}
```
