
# 동시에 다양한 Future들을 실행하는 것

지금까지, 우리는 `.await`을 사용해서 하나의 특정 `Future`가 완료할때까지 
현재의 Task를 막는 future들을 보통 실행해왔습니다.
그러나, 실제 비동기적인 어플리케이션들은 동시에 다른 작업들을 자주 실행해야합니다.

이번 챕터에서, 우리는 같은 시간에 여러 비동기 작업들을 실행하는 방법을 다룰것입니다.

- `join!`: futures이 모두 완료될때까지 기다립니다.
- `select!`: future들 중 하나가 완료될때까지 기다립니다.
- `Spawning`: 하나의 future가 완료될때까지 주변에서 실행되는 top-level 작업을 만듭니다.
- `FuturesUnordered`: 각 subfuture의 결과를 yield하는 future들의 그룹.

