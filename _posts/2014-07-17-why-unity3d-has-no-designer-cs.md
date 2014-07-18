---
layout: post
title: 为啥Unity3d没xx.designer.cs
date: 2014-07-17
categories: tech
publish: true
---

winform时代，在vs的designer里拖啊拖啊，总是可以自动生成一个designer.cs，够(xiang)叼(si)的话你还能自己去改designer里的代码，运气好了vs的designer还能正常的识别，然后再继续拖拖拖。

unity3d的designer没有生成.cs，简直是，耍！流！氓！

先猜个原因再来做点实验：

前方高能预警(纯YY向)

<!--more-->

原因：其实为的就是一点——**performance**，看不到unity源代码，但是从各种资料来看，unity是*build with mono*，而不是*build on mono*，链一张mono官方的图说明此关系：

![relationship](http://www.mono-project.com/files/2/2b/Loaded.png)

So what？designer生成的根本就不是mono的managed code而是native code（或者是unity自己做的某种中间语言），如果能看到unity的源代码，托管部分的代码一定全都是`[DllImport]`才对。没有对应的托管代码生成，哪儿来的designer.cs？

一拍脑袋，啥都想通了，render的部分尽量少搀和托管的代码，从而让整体效率更高。再转念一想，好像又不是这么回事，不管布局和动画参数之类的怎么调整，render的核心部分本来也不是运行在托管的环境下的。那只能理解为unity这样设计的原因让程序员犯错（乱搞）的几率变小，且效率更高。

不过，且不论unity设计的初衷（我会告诉你我越想越觉得前面的结论是瞎扯？），核心算法放在native空间里肯定要比managed空间里快，这点是毫无疑问的。

<del>开始写代码，折腾起（总计折腾了一整个下午，才helloworld出来）：</del>

想要在windows下build一个嵌入mono（Mono JIT compiler version 3.3.0 不确定后续版本还有没改变）运行时的helloworld，步骤如下：

1.  安装Mono的[运行时](http://www.go-mono.com/mono-downloads/download.html)
2.  Path什么的，我忘了是自己加的还是安装程序干的了。反正要确认在CMD下可以mono
3.	用VisualStudio的自带的lib工具做一个.lib出来后续编译用：`lib /nologo /def:mono.def /out:mono.lib /machine:x86`
4.  mono.def内容[在此](https://gist.github.com/rhapsodyn/ec5519c4a65f75d1c699)，官方的tutorial写的是错（至少在windows上）的
5.  c/c++已经玩不来了，研究了半天把include（mono安装后的include目录）和link（上面生成的那个.lib）搞定了
6.	终于可以愉快的写代码了

代码旨在测试同一个算法在托管环境和非托管环境下实现的差距（代码没有经过仔细推敲，可能没啥说服力）：

*cpp:*

{% highlight cpp %}
// EmbedMono.cpp : Defines the entry point for the console application.
//

#include "stdafx.h"
#include <mono/jit/jit.h>
#include <mono/metadata/environment.h>
#include <stdlib.h>

/*
 * Very simple mono embedding example.
 * Compile with: 
 * 	gcc -o teste teste.c `pkg-config --cflags --libs mono-2` -lm
 * 	mcs test.cs
 * Run with:
 * 	./teste test.exe
 */

static MonoString*
NativeFoo () {
	int i = 0;
	const int MAX = 65536;
	char str_result[MAX];
	for (i; i < MAX - 1; i++) 
	{
		str_result[i] = (char)(((int)'0')+i%10);
	}
	str_result[MAX-1] = (char)0;
	return mono_string_new (mono_domain_get (), str_result);
}

static void main_function (MonoDomain *domain, const char *file, int argc, char** argv)
{
	MonoAssembly *assembly;

	assembly = mono_domain_assembly_open (domain, file);
	if (!assembly)
		exit (2);
	/*
	 * mono_jit_exec() will run the Main() method in the assembly.
	 * The return value needs to be looked up from
	 * System.Environment.ExitCode.
	 */
	mono_jit_exec (domain, assembly, argc, argv);
}


int 
main(int argc, char* argv[]) {
	MonoDomain *domain;
	const char *file;
	int retval;
	
	if (argc < 2){
		fprintf (stderr, "Please provide an assembly to load\n");
		return 1;
	}
	file = argv [1];

	/*
	 * Load the default Mono configuration file, this is needed
	 * if you are planning on using the dllmaps defined on the
	 * system configuration
	 */
	//mono_config_parse (NULL);
	/*
	 * mono_jit_init() creates a domain: each assembly is
	 * loaded and run in a MonoDomain.
	 */
	domain = mono_jit_init (file);
	/*
	 * We add our special internal call, so that C# code
	 * can call us back.
	 */
	mono_add_internal_call ("MonoEmbed::NativeFoo", NativeFoo);

	main_function (domain, file, argc - 1, argv + 1);
	
	retval = mono_environment_exitcode_get ();
	
	mono_jit_cleanup (domain);
	return retval;
}
{% endhighlight %}

*csharp:*

{% highlight csharp %}
using System;
using System.Runtime.CompilerServices;
using System.Text;
using System.Diagnostics;

class MonoEmbed {
	[MethodImplAttribute(MethodImplOptions.InternalCall)]
	extern static string NativeFoo();

	static string ManagedFoo()
	{
        var MAX = 65535;
        var sb = new StringBuilder();
        for (int i = 0; i < MAX; i++)
        {
            sb.Append(Convert.ToString(i % 10));
        }
        return sb.ToString();
	}

	static void Main() {
        var nativeStr = NativeFoo();
        var managedStr = ManagedFoo();

        Console.WriteLine(nativeStr == managedStr);
        Console.WriteLine(nativeStr.Length);
        Console.WriteLine(managedStr.Length);

        Benchmark("NativeFoo for 1000 times", () => { NativeFoo(); });
        Benchmark("ManagedFoo for 1000 times", () => { ManagedFoo(); });
    }

    static void Benchmark(string description, Action act)
    {
        // clean up
        GC.Collect();
        GC.WaitForPendingFinalizers();
        GC.Collect();

        //warm up
        act();

        var watch = Stopwatch.StartNew();
        for (int i = 0; i < 1000; i++)
        {
            act();
        }
        watch.Stop();
        Console.Write(description);
        Console.WriteLine(" Time Elapsed {0} ms", watch.Elapsed.TotalMilliseconds);
    }
}
{% endhighlight %}

运行结果如下：

> True
>
> 65535
>
> 65535
>
> NativeFoo for 1000 times Time Elapsed 1681.5873 ms
>
> ManagedFoo for 1000 times Time Elapsed 15382.2008 ms
>

10倍差距！虽然ManagedFoo肯定还有些优化的空间，还是足以验证我前面的YY还是有点道理的。

后面又写了个完全犯规的函数：

{% highlight csharp %}
static private string[] cheat = new string[] { "0", "1", "2", "3", "4", "5", "6", "7", "8", "9" };

static string ManagedFoo2()
{
    var MAX = 65535;
    var sb = new StringBuilder();
    for (int i = 0; i < MAX; i++)
    {
        sb.Append(cheat[i%10]);
    }
    return sb.ToString();
}
{% endhighlight %}

结果`ManagedFoo2 for 1000 times Time Elapsed 2614.927 ms`，还是比不了啊（废话）。