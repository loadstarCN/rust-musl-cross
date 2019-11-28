# rust-musl-cross
该文档在ubuntu16.04上亲测可用，不过openssl部分没有成功（其实是没有搞明白）。

翻译不清楚的地方请直接查看[英文原文](https://github.com/messense/rust-musl-cross)

Docker镜像，用于使用[musl cross make][]编译静态Rust二进制文件，
参考来自[rust-musl-builder](https://github.com/emk/rust-musl-builder)

[![Docker Image](https://img.shields.io/docker/pulls/messense/rust-musl-cross.svg?maxAge=2592000)](https://hub.docker.com/r/messense/rust-musl-cross/)
[![Build Status](https://travis-ci.org/messense/rust-musl-cross.svg?branch=master)](https://travis-ci.org/messense/rust-musl-cross)

## 预编译镜像

目前我们有以下镜像 [prebuilt Docker images on Docker Hub](https://hub.docker.com/r/messense/rust-musl-cross/).

| Rust toolchain | Cross Compile Target                | Docker Image Tag    |
|----------------|-------------------------------------|---------------------|
| stable         | x86\_64-unknown-linux-musl          | x86\_64-musl        |
| stable         | i686-unknown-linux-musl             | i686-musl           |
| stable         | arm-unknown-linux-musleabi          | arm-musleabi        |
| stable         | arm-unknown-linux-musleabihf        | arm-musleabihf      |
| stable         | armv7-unknown-linux-musleabihf      | armv7-musleabihf    |
| stable         | armv5te-unknown-linux-musleabi      | armv5te-musleabi    |
| stable         | mips-unknown-linux-musl             | mips-musl           |
| stable         | mipsel-unknown-linux-musl           | mipsel-musl         |


例如，要使用 `armv7-unknown-linux-musleabihf` 目标，请首先pull镜像文件：

```bash
docker pull messense/rust-musl-cross:armv7-musleabihf
```

然后你可以：

```bash
alias rust-musl-builder='docker run --rm -it -v "$(pwd)":/home/rust/src messense/rust-musl-cross:armv7-musleabihf'
rust-musl-builder cargo build --release
```

此命令假定`$（pwd）`指向的目录是可读写的，它将生成二进制文件在`armv7-unknown-linux-musleabihf`目录中。
目前，它不会尝试在每次编译之间保持缓存，因此最好保留为生成最终relaase版本的编译。

## 工作原理

`rust-musl-cross` 在 [musl-cross-make][]的帮助下使用 [musl-libc][], [musl-gcc][]  其易于编译，并支持新的 [rustup][] `target`.  它包括几个库的静态版本：
libraries:

- 标准的 `musl-libc` 库.
- OpenSSL，这是许多Rust应用程序所需要的。


## 使OpenSSL工作

如果您的应用程序使用OpenSSL，您还需要采取一些额外的步骤来确保它能够找到OpenSSL的可信证书列表，这些证书存储在不同Linux发行版上的不同位置。您可以使用[`OpenSSL probe`]（https://crates.io/crates/OpenSSL-probe）执行此操作，如下所示：

```rust
extern crate openssl_probe;

fn main() {
    openssl_probe::init_ssl_cert_env_vars();
    //... your code
}
```

## 使用 beta/nightly 版本的Rust

目前我们默认安装稳定的Rust，如果你想切换到beta/nightly 版本的Rust，你可以从我们的Docker镜像扩展，例如对target `x86_64-unknown-linux-musl` 使用beta Rust：

```dockerfile
FROM messense/rust-musl-cross:x86_64-musl
RUN rustup update beta && \
    rustup target add --toolchain beta x86_64-unknown-linux-musl
```

## 瘦身二进制文件

您可以在镜像中使用 `musl-strip` 命令瘦身二进制文件，例如：

```bash
docker run --rm -it -v "$(pwd)":/home/rust/src messense/rust-musl-cross:armv7-musleabihf musl-strip /home/rust/src/target/release/example
```

[musl-libc]: http://www.musl-libc.org/
[musl-gcc]: http://www.musl-libc.org/how.html
[musl-cross-make]: https://github.com/richfelker/musl-cross-make
[rustup]: https://www.rustup.rs/

## License

许可证在 [The MIT License](./LICENSE)
