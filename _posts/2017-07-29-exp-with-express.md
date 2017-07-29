---
layout: post
title: Expressjs三五事
date: 2017-07-29
categories: tech
publish: true
---

最近在用[Expressjs](https://expressjs.com/)折腾后台，遇到一些坑（很多都不算坑，只怪我自己不会走路），也积累的一些tips，可以一记：

## EXPRESS 

框架本身的质量和成熟度还是比较高的，最出色/难得的一点是——十分的**克制**（连`body-parser`都要自己`require`），跟前辈RoR和Rjango相比，轻量的不像实力派。虽然支持*View*，但是没有任何默认的Render Engine；根本就没有任何连带的ORM或者类似的数据访问的组件，也就跟*Model*也沾不上啥边。

所以，你已经很难把Express定位为MVC Web Framework了，而更为准确的定位是——带middleware chain的router，仅此而已。

轻量带来的优点和缺点同样显而易见

Pros：
1. 源码短，容易读，想要看清楚一个请求怎么来怎么去debug个10来次都能弄明白个大概
2. 优化容易，iceberg少，能看到的东西基本上就是所有了
3. 接口简单，学习成本和记忆成本少。`(req, res, next) => {}`就基本上你需要写的所有东西了。
4. res和req只是在nodejs本来的request和response上封装的一层，该有什么就有什么
5. app和router的设计真的是出色，各种sub-app和router的分离可以让每个请求走的middleware chain既清晰又精简

Cons：
1. 各种middleware需要自己折腾，麻烦
1. 各种middleware需要自己折腾，麻烦
1. 各种middleware需要自己折腾，麻烦
1. 各种middleware需要自己折腾，麻烦
1. 各种middleware需要自己折腾，麻烦

## 开发

#### Nodemon

任何用到了node的项目，都该在`package.json`里有如下配置才对：

``` javascript
  "scripts": {
    "dev": "set DEBUG=zd:* & nodemon ./bin/www",
  },
```

#### Debug()

这是一个很有用的module: 

1. 不是所有所有代码都能用ide的debug搞定，比如时序相关的
1. 很多时候也就是一个print就能搞明白的东西，何必要F5？
1. 其他各种module，包括express自己也都是用的`debug()`，只要`set DEBUG=*`就能看到所有的print，岂不痛哉
1. 关于nodejs的一切都是tj大神写的，这个也不例外

#### Promise

Promise是一味解决callback hell的好药，但并不能根治，还是希望`await/async`早点成为正式标准吧。promise的缺点，随便google一下多的是。

#### Sequelize

算是个合格的ORM，该有的功能都有，只是：

1. 文档略渣
1. 时区问题有点烦，需要在配置里加个`+8:00`，DB里存utc确实是解决globalization的好办法，但几个项目需要呢？
1. migration的功能看起来很美，但是动不动就要drop table，你敢用？
1. model里没定义的field，写在query的attributes里，并不会报错，你看debug信息里生成的sql也有，但最后的model就是没有，有点蛋疼
``` javascript
const foo = sequelize.define('foo', {
    a: DataTypes.STRING(512),
    //b: DataTypes.STRING(512),
});
     
db.foo.findOne({
    attributes: ['a', 'b']
}).then(ret => {
    // DEBUG: EXECUTE SQL: select a, b from foo
    ret.b === undefined;
});
```

#### Morgan

全局日志，没啥好说的。只是很多时候只能抛出`Error: write after end`，但实际上可能是读写permission的问题。不过这也怪不得实现，只能自己好生检查了。

## 部署

#### Serve Static 

Expressjs自带的server static十分简单，就是单纯的每次读文件（304不用读，废话），正确的返回304或者200，也没有任何缓存（view engine的模板倒是有缓存）。静态资源还是应该放在专业的地方。

#### PM2

监控和部署全款它了，好用还是好用，只是：

1. web上的监控服务收费，倒是无可厚非
1. deploy localhost有点麻烦（需要把ecosystem.js里的host写成localhost之类的，google一下有很多人问过），因为设计初衷是部署到remote机器（确实是合理的）
1. startup脚本用其他用户，必须要`sudo pm2 startup --user other --hp /var/other`，su到其他user，报一些风马牛不相及的error
1. 执行deploy的目录必须要`git init`过 ？？？
![nickyong](http://img.mp.itc.cn/upload/20160828/df8db1060c5342af82aeb6a9dc43ca42_th.jpeg)

#### Newbie-thing

- 非root用户无法bind 80端口：现在还没lbs，有了自然不会bind 80，看起来过得去的解决方案就只有iptable做本地转发了
- 开发环境用crontab做些自动部署之类的东西挺方便的
- shell script真不是人写的 