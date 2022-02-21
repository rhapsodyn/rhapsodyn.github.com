---
layout: post
title: 2021 Retospective
date: 2021-01-31
categories: tech
---

本来 21 年攒了个 Flutter 的 po，结果拖拖拉拉拖到了 22 年也没写完，干脆凑点其他杂七杂八的东西，一起发个年度回顾，阿 Q 一下也算是完成了自己的年度 KPI。

## End2End Me

一个人在工作了十年之后，很难不去回顾自己的职业生涯。在还算特殊的经历的塑造之下（web 全栈 && 游戏客户端），突然发现自己逐渐成为~~究极进化！~~了某种 E2E Dev：from client **GPU-END** to server **CPU-END**。虽然算不上精通，但至少从渲染到计算，都还算的上了解。从前到后，从上到下，也能组个像模像样的技能树：

1. 能写 web UI
1. 能用 Flutter
1. 能用 Unity
1. *稍微能*写 native mobile UI，其实 MFC 也会点
1. *稍微能*写简单的 shader，只针对 programable pipeline，光追的 pipeline 不行，gpgpu 不会
1. *稍微能*改 Flutter engine，能读 game engine
1. 能写 managed server runtime，java 不会且讨厌
1. *稍微能*写应用层 network
1. *稍微能*写 native (Rust or C)
1. *稍微能*做 instruction-level （by valgrind eg.） 级别的优化

虽然这技能树跟各种[攻略](https://coolshell.cn/articles/18360.html)上的可能差距较大，但自己玩的还算快乐。各种代码（从 three.js 到 tokio.rs）和 paper （从 SIGGRAPH 到 USENIX）读起来也都没啥障碍，想学点新东西还是挺快的。可能乱点技能树后触发了隐藏 bonus：可用技能点+++。

也看过一些[职业规划的建议](http://www.ruanyifeng.com/blog/2019/02/weekly-issue-42.html)：尽量找交叠的领域，作为发展方向。代入我自己的变量，感觉只有[已经过气](https://www.engadget.com/google-stadia-game-studios-shut-down-montreal-los-angeles-201811811.html)的云游戏最合适了。

抛开那些莫须有的职业发展焦虑，扪心自问一直还算**热爱**编程，那给自己前 10 年的职业生涯打个 90 分不算过分吧？

## Rust is MY new Ruby

Rust 在 2021 正式成为了我的新 Ruby —— AKA: 想顺手/随手写个啥（eg: 写个[Json Seriailizer](https://github.com/rhapsodyn/rust-json-foo)、写个[Ray Tracing](https://github.com/rhapsodyn/rust-ray-tracing-foo)），就用 Rust（Ruby）吧。从 StackOverflow 的年度统计来看，Rust 早就蝉联好几年 [Most Loved Language](https://insights.stackoverflow.com/survey/2021#section-most-loved-dreaded-and-wanted-programming-scripting-and-markup-languages)，我算是后知后觉，不过最后还是着了 Rust 的道。

虽然 Rust 和 Ruby 在 strong-type/weak-type 上基本上在两个极端：一个非常严谨、另一个异常灵活。用 Ruby 你总是能发现“茴香豆”的 100 种写法，并且还能挑个自己看得最顺眼的写法。而 Rust 却总是告诉你“艹字头”应该写得更工整一点，不然会被认错~~的哦~~。人人都抱怨 java 写起来太 verbose 了，但人人都喜欢 Rust 的编译报错太 verbose 了。不夸张的说，能看懂 Rust 的各种编译报错（what & why），也就算得上是入门了。

诚然，除了类型系统，Rust 和 Ruby 还有许许多多的差异：比如 Native VS VM 等等。但从另一些角度来看，这两门语言还是有点共性的。

比如：**Expressive**。随便找了两段 rails 和 axum 的代码：

```ruby
class PeopleController < ActionController::Base
  # This will raise an ActiveModel::ForbiddenAttributesError exception
  # because it's using mass assignment without an explicit permit
  # step.
  def create
    Person.create(params[:person])
  end

  # This will pass with flying colors as long as there's a person key
  # in the parameters, otherwise it'll raise an
  # ActionController::ParameterMissing exception, which will get
  # caught by ActionController::Base and turned into a 400 Bad
  # Request error.
  def update
    person = current_account.people.find(params[:id])
    person.update!(person_params)
    redirect_to person
  end

  private
    # Using a private method to encapsulate the permissible parameters
    # is just a good pattern since you'll be able to reuse the same
    # permit list between create and update. Also, you can specialize
    # this method with per-user checking of permissible attributes.
    def person_params
      params.require(:person).permit(:name, :age)
    end
end
```

```rust
#[tokio::main]
async fn main() {
    // initialize tracing
    tracing_subscriber::fmt::init();

    // build our application with a route
    let app = Router::new()
        // `GET /` goes to `root`
        .route("/", get(root))
        // `POST /users` goes to `create_user`
        .route("/users", post(create_user));

    // run our app with hyper
    // `axum::Server` is a re-export of `hyper::Server`
    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    tracing::debug!("listening on {}", addr);
    axum::Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

// basic handler that responds with a static string
async fn root() -> &'static str {
    "Hello, World!"
}

async fn create_user(
    // this argument tells axum to parse the request body
    // as JSON into a `CreateUser` type
    Json(payload): Json<CreateUser>,
) -> impl IntoResponse {
    // insert your application logic here
    let user = User {
        id: 1337,
        username: payload.username,
    };

    // this will be converted into a JSON response
    // with a status code of `201 Created`
    (StatusCode::CREATED, Json(user))
}

// the input to our `create_user` handler
#[derive(Deserialize)]
struct CreateUser {
    username: String,
}

// the output to our `create_user` handler
#[derive(Serialize)]
struct User {
    id: u64,
    username: String,
}
```

不难发现这两门语言（严格来说：也跟框架有关），在开发 web server 的时候，他们的代码都可以算作是 DSL 了。神奇的是，Ruby 是靠的 meta programming 和 weak type，而 Rust 是靠的 type inference 和 strong type，殊途同归。

关于 Rust 和 Ruby 另一个比较悲伤的共性则是：大多数程序员不会在 everyday work 中使用他们，因为这两门语言以前不是主流，以后也不太可能成为主流。原因么，不外乎就是现在同样定位的技术（Java C++)，已经非常成熟而且还在不断发展，再加上惯性的作用，让后来者很难去替代他们。考虑到距离产生美 or“得不到的永远在骚动”，就不难理解为啥这两门语言是**Most Loved**了。

## Flutter

Flutter 算是我的“年度真香”项目，完全没想到 google 不是玩票而是挺认真的做了套跨平台的 UI Framework

### Q&A

每种技术都是对某个特定问题的解答。

从 Flutter 这个解答不难反推得到 google 定义需要解决的问题：目前没有一套跨平台&高性能的 UI 框架。跨平台是技术需求：减少双平台总的代码量；高性能是用户需求：顺滑才是良好的用户体验，而用户并不关心 app 最终多写了或者少写了多少代码。

而 google 对这个问题的解决方案也很直接：再做一个 browser / game engine，多一层抽象，开发者（大部分时间）只面向 engine 编程。

### Cons

1. engine 本身复杂度太高，即使是 google，issues 依旧维持在 5K+，而且 engine 一旦有 bug，开发者基本上只能等更新
1. engine 打包进 app，增加了 app 的包大小，compile 和 startup 的速度也变慢了
1. 在 Flutter 里“画”出来的组件，跟 iOS/android 的原生组件始终存在差异（包括表现和行为），体验缺乏一致性
1. 需要平台相关功能的时候，还是逃不过写 ObjC 和 Java
1. 大团队和大项目肯定是分平台开发，小项目没法反哺，项目基本只能靠 google 推

### Pros

1. engine 本身复杂度太高，但是有人帮你写了
1. 现代 UI 的编程模型：申明式组件树（跟 React 和 SwiftUI 长得很像）
1. 使用起来有一定的自由度，可以用默认的 host（by `flutter create`），也可以把已有 app 当成 host，用 flutter 做扩展

### Highlight

#### Seperating trees

现代 UI 框架基本都以树来组织*组件*这个没什么好说的，无论是是从理论还是实践都是最优解了，但 Flutter “三棵树”的设计还是挺有意思的：

- Widget Tree： “可编程”的树，面向开发的基本接口，函数式风格（但 dart 本身是 OO 风格的，为了更契合 flutter，dart 中的*new*是可选而且大家都不选的）。有几分神似 React 中的 hook 风格，每次 render 的起点在于 UI `init`或者某个 widget 的`setState`，然后依次调用 UI tree 上每个相关 widget 的`build`。跟 React 不同的是，flutter 的风格更加*简洁*和*显式*，没有了 reactive property，只能使用显示的`setState`来让 UI 重绘。需要关注的“lifetime hook”也只集中在`initState`和`dispose`，因为每次都是“new”的，没有“update”了。虽然 Flutter 提供的 widget 很多，但其实都是各种 wrapper，只是为了减少 boilerplate 用的，实际核心的 API 非常的少，就连各种 animator，也就是每帧调用`setState`罢了。
- Element tree：沟通上下两棵树的中间树，因为上层 widget tree 的“函数式”和“不可变”，所以所有状态就需要 element tree 来管理。而 element tree 需要管理的也仅仅只有状态，就连`setState`也只是改变了一个 dirty flag 而已。实际的渲染交给了另一个树和另一个线程（对 flutter create 出来的 ios/android app 是独立的 render 线程）
- Render tree：实际负责渲染的树，渲染完成了以后更新 element tree 中的状态。render tree 的核心代码都在 c++里，包括 raster cache 等等，部分代码如`canvas.drawRect`等以 FFI 的形式提供给 dart。

#### Seperating concerns

分离三棵树带来的好处之一，就是能很自然的做出符合直觉的多线程设计，rasterization 和 ui 跑在不同线程里，ui（widget tree）只能提交“draw command”，而 rasterization 在另外的线程中执行，尽量让图像表现更 smooooooooth。

事实上，flutter 整个项目的模块化都做的很出色：从 flutter 和 flutter engine 两个 repo 的[分离](https://github.com/flutter/flutter/wiki/Why-we-have-a-separate-engine-repo)，再到 engine 中每个模块的划分：flow 负责缓存和组织 Skia Picture & Layer、shell 负责对 host 提供唯一的 flutter engine api、etc。

确实是 google 认真做的项目，质量还是挺可靠的。

#### Skia

顺便看了下地球上使用最广的 2d 图形库（主要是靠 chrome 和 android \doge），因为只专注于 2D 渲染，所以 API 风格是类似 OpenGL 固定管线的（UI 编程，能有一个场景需要自己写 shader 么？好奇）。

最厉害的地方在帮开发者摆平了跨平台的问题，底层可以使用 OpenGl、Vulkan、Metal 或是 CPU 来渲染，甚至还可以渲染成 PDF。虽然 cpp 代码看起来有点费劲，但好在跟 flutter engine 一样挺 modern 的（c++14），连格式化都很 new school 的：

> We use 4 spaces, not tabs.

#### Dart

跟 Swift 一样专门为了 UI 而优化的语言，没啥好说的，就是挺好用的：符合现代 UI 开发的直觉，学习成本极小（to me）。

## Not End Yet

疫情重塑世界的同时，也顺手影响了下我微不足道的世界观。我现在是不太相信“明天会更好”了，但我相信：[明知会很难，还要努力做](https://www.bilibili.com/video/av286917494/)，才比较厉害！
