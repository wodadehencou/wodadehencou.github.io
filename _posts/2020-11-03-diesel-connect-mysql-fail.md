---
layout: post
title: TITLE
date: 2020-11-03 15:45
tags: [diesel, mysql, mariadb]
category: rust 
summary: Using diesel to connect mysql in parallel, Error: "Plugin caching_sha2_password could not be loaded: it is already loaded" 
---

# diesel connect mysql fail

使用`diesel`访问mysql数据库，在测试的时候发现会出现"Plugin caching_sha2_password could not be loaded: it is already loaded"。

## 问题分析

`caching_sha2_password`是Mysql中用户密码验证的模块。在测试时发现，单次的mysql连接是没有问题的。

Rust对测试的处理是并发进行，使用命令`cargo test -- --test-threads=1`可以使用单线程进行测试。使用单线程测试也是没有问题的。

现在怀疑是Mysql Server的问题，本来想更换Mysql8到Mysql6再进行测试。突然想到可以使用go编写测试程序测试一下并发连接。

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

*用Go进行测试，一点问题都没有。基本可以确认是跟diesel对Mysql client的连接处理相关*

比对了一下Go和Rust对Mysql Client的实现，Go是基于原生的Go代码进行，Rust是通过ffi封装了mysql client lib。那么问题一定是出在Mysql Client lib上。

为了排除是否是因为单Mysql主机造成的错误，在本机启动5个Mysql Server，Rust并发分别连接，问题依旧。基本可以明确问题了。

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

> 查看我本机的Mysql Lib，发现Manjaro系统基于Arch，Arch Linux本身的mysql library使用的是mariadb library，那么是不是由于mariadb library实现的问题呢？
> 切换到Oracle的Mysql library是否可行？  

## TODO

换到Oracle的Mysql Client试试，这个问题在C上应该也是存在的。

