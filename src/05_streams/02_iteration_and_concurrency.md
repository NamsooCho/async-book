# 반복자와 동시성

동기 `Iterator`와 유사하게 `Stream`에서 값을 처리하고 반복하는 방법에는 여러 가지가 있습니다.
콤비네이터 스타일의 `map`, `filter` 및 `fold`와 early-exit-on-error 류의
`try_map`, `try_filter` 및 `try_fold` 메소드가 있습니다.

불행히도 `for` 루프는 `Stream`과 함께 사용할 수 없지만
imperative-style 코드, `while` 및 `next`/`try_next` 함수는
사용될 수 있습니다 :

```rust,edition2018,ignore
{{#include ../../examples/05_02_iteration_and_concurrency/src/lib.rs:nexts}}
```

그러나 한 번에 하나의 요소만 처리하면 잠재적으로
동시성에 대한 기회를 버리게 되고 결국 처음으로 돌아가서 우리가 왜
비동기 코드를 작성하는지를 묻게 됩니다. 스트림에서 여러 항목을 동시에 처리하려면
`for_each_concurrent`와 `try_for_each_concurrent`를 사용하십시오 :

```rust,edition2018,ignore
{{#include ../../examples/05_02_iteration_and_concurrency/src/lib.rs:try_for_each_concurrent}}
```
