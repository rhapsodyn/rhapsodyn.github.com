---
layout: post
title: 用node_redis，不需要pooling
date: 2017-03-19
categories: tech
publish: true
---

周末看到个 XX 公司 redis 最佳实践的文章，中间有一条是 client 最好做 pooling，提高效率。

手上的项目后台是 nodejs，redis client 用的[redis_node](https://github.com/NodeRedis/node_redis)这个实现，没有额外实现连接池。因为印象中似乎看过有 stackoverflow 的答案说是已经默认实现了，一搜，果然有这个帖子[https://stackoverflow.com/questions/21976270/node-js-redis-connection-pooling](https://stackoverflow.com/questions/21976270/node-js-redis-connection-pooling)，只是说的是不用去管 pool 了，一个 client 就足够高效了。

略感兴趣是如何的高效法，难道是默认实现了 connection pool？就看了看源代码，结果一个 client 就一个 socket（|| tls），丝毫没发现 connection pool 的影子。

遂又搜了一下 github 的 issue，发现了个关键字——pipeline。

读了 redis[官方的文档](https://redis.io/topics/pipelining)，才弄明白是如何个高效法：

redis 的协议十分的简洁，标准的 request-response 模式，除了 sub/pub 和 moniter 等命令外，都是你发一个 request 我回一个 response，简单直白。而所谓 pipeline，就是 client 可以一次性发送多个 request，而不需要等到前一个 request 收到对应的 response 了以后再发送第二个 request，server 也会把所有的 response 按收到 request 顺序打包一次性的回复。从而减少了 RRT 和 syscall 的次数，在 throughput 高的情况下可以提升不少的效率。

等等，一次多个请求一起发送，redis 的协议又没有包 ID 之类的东西，不会出现乱序么？这个还真不会！因为服务器是按照收到请求的顺序准备好 response，并放到 buffer 里统一发送的。你能保证请求的顺序，也就不用担心相应的顺序了。正好，nodejs 还就是单线程模型的，请求顺序没得乱，巧了吗这不是！

再等等，我看 node_redis 的源代码，每次调用 redis 的 command，都会往 socket 里写，好像并没有合成一个 tcp 包的打包操作啊？因为本来 nodejs 的 socket.write 也不是直接往 system 的 socket 里写的啊：

> socket.write(data[, encoding][, callback])#
> Added in: v0.1.90
> Sends data on the socket. The second parameter specifies the encoding in the case of a string--it defaults to UTF8 encoding.
>
> Returns true if the entire data was flushed successfully to the kernel buffer. Returns false if all or part of the data was queued in user memory. 'drain' will be emitted when the buffer is again free.
>
> The optional callback parameter will be executed when the data is finally written out - this may not be immediately.

启示：

1.  没事别瞎优化 —— premature optimization is the root of all evil
2.  多读源代码 —— show me the code
