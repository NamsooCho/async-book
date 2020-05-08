# 응용: 간단한 HTTP 서버

`async`/`.await` 를 사용하여 에코 서버를 만들어 보겠습니다.

본 예제를 시작하려면 `rustup update stable`을 실행하여 Rust stable 1.39 이상을 유지하십시오. 일단 완료하면
`cargo new async-await-echo`를 실행하여 새로운 프로젝트를 만들고 여십시오.
실행의 결과는 async-await-echo 폴더에 만들어 집니다.

`Cargo.toml` 파일에 몇몇 의존성을 추가해 봅시다 :

```toml
{{#include ../../examples/01_05_http_server/Cargo.toml:9:18}}
```

이제 의존성이 해결 되었으므로 이제 코딩을 시작하겠습니다.
다음의 imports를 추가합니다 :

```rust,ignore
{{#include ../../examples/01_05_http_server/src/lib.rs:imports}}
```

이제 의존성이 해결 되었으므로 모든 것을 모아서 리퀘스트를 처리하도록 하는 기본 토대를 만드는 작업을 시작하겠습니다 :

```rust,ignore
{{#include ../../examples/01_05_http_server/src/lib.rs:boilerplate}}
```

지금 `cargo run`하면, "Listening on http://127.0.0.1:3000" 이라는 메시지가 터미널에 나타납니다.
여러분이 선택한 브라우저에서 위 주소를 열면 "hello, world!"가 브라우저에 나타납니다.
축하합니다! 방금 Rust에서 첫 번째 비동기 웹 서버를 작성했습니다.

다음과 같은 정보(request URI, HTTP version, headers, and other metadata)가 포함 된 요청을 검사 할 수도 있습니다.
예를 들어, 다음과 같이 요청의 URI를 인쇄 할 수 있습니다.

```rust,ignore
println!("Got request at {:?}", req.uri());
```

우리가 요청을 처리 할 때 비동기적인 것을 아무 것도 아직 하고 있지 않은 것을 눈치 채셨나요?
-우리는 단지 즉시 응답합니다.
따라서 우리는 `async fn`이 제공하는 유연성을 활용하지 않습니다.
정적 메시지를 반환하는 대신 Hyper의 HTTP 클라이언트를 사용하여 다른 웹 사이트에 사용자의 요청을 프록시 하도록 시도해 봅시다.

요청하려는 URL을 파싱하여 시작합니다.

```rust,ignore
{{#include ../../examples/01_05_http_server/src/lib.rs:parse_url}}
```

그런 다음 새로운 `hyper::Client`를 작성하고 이를 사용하여 `GET` 요청을 할 수 있습니다.
이는 응답을 사용자에게 반환합니다 :

```rust,ignore
{{#include ../../examples/01_05_http_server/src/lib.rs:get_request}}
```

`Client::get`은 `Future<Output = Result<Response<Body>>>`을 구현하는 `hyper::client::ResponseFuture`를 반환합니다.
(또는 futures 0.1 버젼에서 `Future<Item = Response<Body>, Error = Error>`).
우리가 그 future를 `.await` 하면 현재 작업 중인 HTTP 요청이 전송됩니다.
현재의 타스크는 일시 중지되고 큐에 들아가서 대기하다가 응답이 오면 계속 실행 됩니다.

이제 `cargo run`을 하고 브라우저에서 `http://127.0.0.1:3000/foo`를 연다면,
Rust 홈페이지와 다음 문자열이 터미널에 나타납니다.

```
Listening on http://127.0.0.1:3000
Got request at /foo
making request to http://www.rust-lang.org/en-US/
request finished-- returning response
```

축하합니닥! 여러분은 HTTP 요청을 프록시 했습니다.
