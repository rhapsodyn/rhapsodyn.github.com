---
layout: post
title: Callbacks 和(han) Promise 和 Rx*
date: 2014-07-09
categories: tech
publish: true
---

想想还有几个月就去台湾了，心里还有点小激动呢，标题卖个萌。

-------------------------------

回头看了一下，发现自己最近几个月对callback和async这些玩意儿出奇的关心。原因无它，各种callback和async操作实在是太**反 人 类**了（此处人类特指地球上某统治级灵长类生物中跟其祖先最接近的一群——程序猿，BTW，反正都扯蛋了就多扯两句：人人都知道user-friendly重要，但却没有人比微软更注重programmer-friendly，就这一点，简直要为M$点赞，以后蛋疼了一定要写一篇关于大微软爱护程序员的软文），而如何**抽象**这些反人类的东西，正是计算机和编程领域里最有魅力的部分。

<!--more-->

你得承认，类似：内存 = 竖着摆的矩形分栏状图形；stream = 横着摆带节点的水管状图形，都是一些类似“平面直角坐标系”这样的——你根本就不晓得笛卡尔是怎么想出来的——伟大抽象。没了这些抽象，很多更一步的概念和实现就会变得十分的困难。

嗯，图文并茂一下：

*memory:*
![memory](http://www.arjay.bc.ca/Modula-2/Text/Ch12/Figure/Figure_12.5.GIF)

*stream:*
![stream](https://encrypted-tbn1.gstatic.com/images?q=tbn:ANd9GcQqluJSgyDyXLVcXIINByJiDkdTRLvFOgRSwDSW09Ae8QD_B9aG_w)

简单来说，抽象就是让你理解不了的东西变得容易(可以?)理解了，如果达不到这个目的，那就不是成功的抽象。

类似threading和asynchronous这样的东西就是难以理解的（光拼写就很难了），因为人的惯性思维都是串行的，非异步和单线程的编程也是串行的，当我写下

{% highlight shell %}
operation_one
operation_two
{% endhighlight%}

的时候，我一定想的是`operation_one`执行完毕`operation_two`开始执行这样的顺序。如果我没接触过threading或者async，当你告诉我“可能operation\_two会比operation\_one先执行完，可能他们两个操作还会瞬间就return了，不过到底会发生什么我也不能确定呢”，我一定会觉得你在耍流氓。

可async就是要耍流氓（后面都只写async不写threading了，因为multi-threading只是async底层的一种实现方式，async的抽象层次已经比threading高了，没必要在一起讲），至于async到底是怎么耍流氓的，先看下面的代码（nodejs）：

{% highlight javascript %}
var dirName = "foo";
fs.readdir(dirName, function(err, fileNames) {
	if (err) {
		throw err;
	}

	fileNames.forEach(function(fileName) {
		var fullPath = path.resolve(dirName, fileName);
		fs.readFile(fullPath, function(err, data) {
			if (err) {
				throw err;
			}

			console.log(data.length);
		});
	});
});
{% endhighlight %}

看起来挺直白的是吧？不就是读取某文件夹下所有文件，然后print出来他们的文件长度嘛。嗯，加个功能，在读取完所有文件后，打印个`DONE`出来。简单，这样写就行了：

{% highlight javascript %}
var dirName = "foo";
fs.readdir(dirName, function(err, fileNames) {
	if (err) {
		throw err;
	}

	fileNames.forEach(function(fileName) {
		var fullPath = path.resolve(dirName, fileName);
		fs.readFile(fullPath, function(err, data) {
			if (err) {
				throw err;
			}

			console.log(data.length);
		});
	});

	console.log("DONE");
});
{% endhighlight %}

才没有**行了**，**DONE**第一时间就print出来了，因为我们可爱的async，想要在正确的时候显示DONE，你得这样写：

{% highlight javascript %}
var readCount = 0;
fileNames.forEach(function(fileName) {
	var fullPath = path.resolve(dirName, fileName);
	fs.readFile(fullPath, function(err, data) {
		if (err) {
			throw err;
		}

		console.log(data.length);

		if (++readCount == fileNames.length) {
			console.log("DONE");
		}
	});
});
{% endhighlight %}

先不论这种写法严格的正确与否（在有error的情况下，肯定要另说了），无缘无故多出一个变量（`readCount`）和一段逻辑（`++`和`if`）,为的只是实现一个在同步环境下十分顺理成章的操作，这本身就不可接受嘛。究其原因——**nodejs以callback的方式设计async的API，就是渣决定**，这话可不是我第一个说的，至于是在那个大牛的blog上最先看到的忘记了。结合开篇写到的抽象的部分，可以进一步升华出——**以回调的方式抽象异步操作非常反人类**——这一结论。

我不知道nodejs团队有没有承认自己的渣，只知道后来的api都多了类似`readFileSync`这样的对应的同步版本。多强调一次，我并不是说nodejs的异步+事件驱动的设计理念是渣，而是觉得**以回调的方式提供API/抽象异步操作真心很渣**。

不过，勤劳勇敢的程序员有自己的特有的手段，来<del>寻找珍贵的食材</del>解决问题，自己写个library，拯救其他同类。让我们看看引入了npm上依赖数前五的`async`库，情况有何不同：

{% highlight javascript %}
var dirName = "foo";
fs.readdir(dirName, function(err, fileNames) {
	if (err) {
		throw err;
	}

	var fullPaths = fileNames.map(function(fileName) {
		return path.resolve(dirName, fileName);
	});

	async.map(fullPaths, fs.readFile, function(err, datas) {
		if (err) {
			throw err
		}

		datas.forEach(function(data) {
			console.log(data.length);
		});

		console.log("DONE");
	})
})
{% endhighlight %}

async具体的玩法可以去看[文档](https://github.com/caolan/async)，从直观上来讲，代码好像清晰了一点，也多了点FP的高大上的味道。但是说到底，情况从“不知道何时该出现回调”到了“现在我能在all done的时候调用回调了”，有好转，但不多。

再来，用promise:

{% highlight javascript %}
var readdirPromise = function(dirName) {
	var deferred = Q.defer();

	fs.readdir(dirName, function(err, fileNames) {
		if (err) {
			deferred.reject(err);
		} else {
			deferred.resolve(fileNames);
		}
	});

	return deferred.promise;
};

var readFilePromise = function(filePath) {
	var deferred = Q.defer();

	fs.readFile(filePath, function(err, data) {
		if (err) {
			deferred.reject(err);
		} else {
			deferred.resolve(data);
		}
	});

	return deferred.promise;
};
// below two lines are AWESOME！
// var readdirPromise = Q.denodeify(fs.readdir);
// var readFilePromise = Q.denodeify(fs.readFile);

var dirName = "foo";

readdirPromise(dirName).then(function(fileNames) {

	var readFilePromises = fileNames.map(function(fileName) {
		var fullPath = path.resolve(dirName, fileName);
		return readFilePromise(fullPath);
	});

	return Q.all(readFilePromises);
}).then(function(datas) {

	datas.forEach(function(data) {
		console.log(data.length);
	});

	console.log("DONE");
});
{% endhighlight %}

不看前面包装的部分，或者用Q的denodeify来封装，代码似乎更加顺眼了点，乍一看几乎是串行化地在写代码了。这也是我看来拯救callback式API最好的workaround了，注意是**workaround**而不是**solution**，这很重要！

糟，标题里面还有个Rx*完全没提啊，因为我是今天才真正了解了一点Rx\*，刚刚有点make sense的感觉，关于Rx\*的以后再写吧。关于Rx*，[这个文章](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)写的很不错。

FYI:上面所有代码见这个[gist](https://gist.github.com/rhapsodyn/b1daceb4d2f1da514aaf)

洋洋洒洒写了那么多最后还只有个workaround，那真正的solution是啥？下面有几行（我就是数体教，你打我啊）代码，你们自己感受一下：

{% highlight csharp %}
using System;
using System.IO;
using System.Threading.Tasks;
using System.Text;

public class AsyncDemo
{
	public static void Main()
	{
		Task.Run(async ()=>{
			string readResult = await ReadAsync("foo.txt");
			int writeLength = await WriteAsync("foo1.txt", readResult);

			Console.WriteLine(writeLength);
		});

		Console.ReadLine();
	}

	public static async Task<string> ReadAsync(string filePath)
	{
		using(var fs = new FileStream(filePath, FileMode.Open, FileAccess.Read, FileShare.Read, 4096, true))
		{
			StringBuilder sb = new StringBuilder();

	        byte[] buffer = new byte[0x1000];
	        int numRead;
	        while ((numRead = await fs.ReadAsync(buffer, 0, buffer.Length)) != 0)
	        {
	            string text = Encoding.UTF8.GetString(buffer, 0, numRead);
	            sb.Append(text);
	        }

	        return sb.ToString();
		}
	}


	public static async Task<int> WriteAsync(string filePath,string text)
	{
		byte[] encodedText = Encoding.UTF8.GetBytes(text);

	    using (FileStream sourceStream = new FileStream(filePath,
	        FileMode.Append, FileAccess.Write, FileShare.None,
	        bufferSize: 4096, useAsync: true))
	    {
	        await sourceStream.WriteAsync(encodedText, 0, encodedText.Length);
	    }

	    return encodedText.Length;
	}
}
{% endhighlight %}

去掉其他奇奇怪怪的部分，只看这两行：

{% highlight csharp %}
string readResult = await ReadAsync("foo.txt");
int writeLength = await WriteAsync("foo1.txt", readResult);
{% endhighlight %}

天都亮了！

FYI1:上面的代码是在mono 3.3.0的环境下测试通过了，最近也稍微看了一下mono，感觉非常不错。`WaitToWriteStack.push(mono)`;

FYI2:别以为上面那个就是真·solution了，如果你不知道async里的坑，该错的代码一样会写错。只是说C#5.0的async/wait，只从代码层面看，已经很好了。不过，**看起来好**，就是最重要的。Looks good is All!