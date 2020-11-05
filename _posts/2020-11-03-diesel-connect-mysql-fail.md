---
layout: post
title: diesel connect mysql fail
date: 2020-11-03 15:45
tags: [diesel, mysql, mariadb]
category: rust
---

使用`diesel`访问 mysql 数据库，在测试的时候发现会出现"Plugin caching_sha2_password could not be loaded: it is already loaded"。

## 问题分析

`caching_sha2_password`是 Mysql 中用户密码验证的模块。在测试时发现，单次的 mysql 连接是没有问题的。

Rust 对测试的处理是并发进行，使用命令`cargo test -- --test-threads=1`可以使用单线程进行测试。使用单线程测试也是没有问题的。

现在怀疑是 Mysql Server 的问题，本来想更换 Mysql8 到 Mysql6 再进行测试。突然想到可以使用 go 编写测试程序测试一下并发连接。

```go
func connectMysql(url string) *sql.DB {
    db, err := sql.Open("mysql", url)
    if err != nil {
        fmt.Println(err)
    }
    err = db.Ping()
    if err != nil {
        fmt.Println(err)
    }
    if db == nil {
        fmt.Println("database connection is nil")
    }
    return db
}

func parallelConnect(url string) {
    n := 50
    conns := make([]*sql.DB, n)
    for i := 0; i < n; i++ {
        go func(idx int) {
            conns[idx] = connectMysql(url)
        }(i)
    }
    time.Sleep(time.Second)
    for _, conn := range conns {
        defer conn.Close()
    }
}
```

> 用 Go 进行测试，一点问题都没有。基本可以确认是跟 diesel 对 Mysql client 的连接处理相关

比对了一下 Go 和 Rust 对 Mysql Client 的实现，Go 是基于原生的 Go 代码进行，Rust 是通过 ffi 封装了 mysql client lib。那么问题一定是出在 Mysql Client lib 上。

为了排除是否是因为单 Mysql 主机造成的错误，在本机启动 5 个 Mysql Server，Rust 并发分别连接，问题依旧。基本可以明确问题了。

```rust
    #[test]
    fn test_connect_many_mysqls() {
        return;
        use std::thread;
        let mut handlers = Vec::new();
        for i in 0..5 {
            handlers.push(thread::spawn(move || {
                let _ = conn_mysql(&format!("mysql://root:root@127.0.0.1:{}/db", 3306 + i));
            }));
        }
        handlers.into_iter().for_each(|h| h.join().unwrap());
    }
```

> 查看我本机的 Mysql Lib，发现 Manjaro 系统基于 Arch，Arch Linux 本身的 mysql library 使用的是 mariadb library，那么是不是由于 mariadb library 实现的问题呢？
> 切换到 Oracle 的 Mysql library 是否可行？

## 切换原版 Mysql Client

费了一些力气，下载了社区版本的 Mysql Client library 进行测试。果然没有任何问题。看来明确是`mariadb`的原因了。

> rust 的`mysqlclient-sys`包通过 ffi 调用系统的 Mysql library。
>
> 首先会使用`pkg-config`找`libmysqlclient`，如果找不到的话，会使用`MSVC`查找，（应该是 Windows 的选项）。
> 再之后是通过`mysql_config`可执行文件查找。在编译的时候把原版 Mysql Client 的`mysql_config`在 PATH 中设置在前面，就可以使用新的 Mysql Client library。
> 当然最后运行的时候，要通过`LD_LIBRARY_PATH`把新的库的路径加上，就可以找到了。
>
> 操作流程如下：
>
> ```bash
> export PATH=/home/jam/Workspace/c/mysql-8.0.22-linux-glibc2.17-x86_64-minimal/bin:$PATH
> export LD_LIBRARY_PATH=/home/jam/Workspace/c/mysql-8.0.22-linux-glibc2.17-x86_64-minimal/lib:$LD_LIBRARY_PATH
> ```
>
> 编译后用`ldd`查看可以看到库已经更新
> `libmysqlclient.so.21 => /home/jam/Workspace/c/mysql-8.0.22-linux-glibc2.17-x86_64-minimal/lib/libmysqlclient.so.21 (0x00007f0f461ab000)`
