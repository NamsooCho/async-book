# async-book
Rust에서의 비동기 프로그래밍

## Requirements
이 문서는 [`mdbook`]을 사용하여 만들어진다, 이것은 cargo를 사용하여 간편하게 설치할 수 있다..

```
cargo install mdbook
cargo install mdbook-linkcheck
```

[`mdbook`]: https://github.com/rust-lang/mdBook

## Building
완성된 책을 생성하려면, `mdbook build` 명령을 실행하여 `book/` 디렉토리에 콘텐츠를 만들 수 있다.
```
mdbook build
```

## Development
책을 쓰는 중간 중간 당신이 변경한 내용을 확인하려면, `mdbook serve` 명령을 사용하여 로컬 웹서버를 띄워서 편리하게 변경 사항을 확인할 수 있다.
```
mdbook serve
```
