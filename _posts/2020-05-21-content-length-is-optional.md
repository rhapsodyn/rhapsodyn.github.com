---
layout: post
title: 原来Content-Length如此不重要
date: 2020-05-21
categories: tech
---

一直以为HTTP1.0/HTTP1.1中的response header里的[content-length](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Length)
是个挺重要的Key，毕竟body读多长都要靠这个字段控制，~~种地全靠它~~协议能正常工作全靠它。

现在反思一下，这个误解主要来自两个方面：一是自己实现过的协议，基本都是靠类似的实现去读header和body；
二是根本没认真思考过http1.1时代的流式传输，server往client写个巨大的文件流，难道还要全部读完了读出length才能传输吗？显然不是，这都不叫流式传输了啊。

直到最近想cache一个gzip过的response，才发现了这个知识的盲区——[transfer-encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding)。
简单的说，就是response header有`transfer-encoding: chunked`的时候，不用给`content-length`，直接按某种简单的编码方式一个一个chunk传body就行。

```javascript
const http = require('http')

/**
 * @param {String} str 
 */
function encode(str) {
    const lengthFrag = str.length.toString(16)
    const rst = `${lengthFrag}\r\n${str}\r\n`
    console.log(rst)
    return rst
}

http.createServer((req, res) => {
    if (req.url === '/foo') {
        console.log('req received')
        res.socket.write(
            'HTTP/1.1 200 OK\r\n' +
            'Content-Type: text/plain\r\n' +
            'Transfer-Encoding: chunked\r\n' +
            '\r\n')

        let counter = 0
        const clock = setInterval(() => {
            console.log(counter)
            if (counter < 10) {
                const chunk = encode(`the time is: ${new Date().getTime()}\r\n`)
                res.socket.write(chunk)
                counter++
            } else {
                clearInterval(clock)
                // end
                res.socket.write('0\r\n\r\n')
                // res.socket.end()
            }
        }, 200);
    } else {
        res.writeHead(404)
        res.end()
    }
}).listen(9000)
```

代码一目了然，浏览器测了下也没啥问题。然后随手看了下其他网站的header，发现很多请求好像没带`content-length`也没带`transfer-encoding`，一样也挺正常。

小问号就多了很多小朋友，然后就改了两行代码，1. 不传header 2. 写完了把socket直接`end`掉，而不是write特殊的tail

```javascript
const http = require('http')

/**
 * @param {String} str 
 */
function encode(str) {
    const lengthFrag = str.length.toString(16)
    const rst = `${lengthFrag}\r\n${str}\r\n`
    console.log(rst)
    return rst
}

http.createServer((req, res) => {
    if (req.url === '/foo') {
        console.log('req received')
        res.socket.write(
            'HTTP/1.1 200 OK\r\n' +
            'Content-Type: text/plain\r\n' +
            // 'Transfer-Encoding: chunked\r\n' +
            '\r\n')

        let counter = 0
        const clock = setInterval(() => {
            console.log(counter)
            if (counter < 10) {
                const chunk = encode(`the time is: ${new Date().getTime()}\r\n`)
                res.socket.write(chunk)
                counter++
            } else {
                clearInterval(clock)
                // end
                res.socket.end()
            }
        }, 200);
    } else {
        res.writeHead(404)
        res.end()
    }
}).listen(9000)
```

嗯，果然，只要把socket关了就好了，不想传`content-length`就不传吧，浏览器一样正正常常的。

http果然是个兼容性很强的协议呢！

