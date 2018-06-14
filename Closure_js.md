# 闭包 Closure #
javascript 采用**词法作用域**（lexical scoping，通过阅读包含变量定义在内的数行源码就能知道变量的作用域），
函数的执行依赖于变量作用域，这个作用域是在函数定义时决定的，不是调用时决定的。
JavaScript 函数对象的内部状态不仅包含函数的代码逻辑，还必须引用当前的作用域和作用域链。函数对象可以通过作用域链相互关联起来，函数体内部的变量都可以保存在函数作用域内。

>从技术角度，所有的javascr函数都是闭包，都是对象，都关联到作用域链。

>理解闭包，首先要了解嵌套函数的词法作用域规则。

<pre><code>
var scope="global scope";
function checkscop(){
	var scope="local scope";
	function f(){return scope;}
	return f();
}
checkscope();

//checkscope函数声明了一个局部变量，并定义了一个函数f(), f()返回了这个变量的值，最后将函数f()的执行结果返回。
</code></pre>

词法作用域的基本规则：javascript函数的执行用到了作用域链，这个作用域链是函数定义的时候创建的。嵌套的函数定义在这个作用域链里，其中的变量scope一定是局部变量，不管什么时候什么地点执行函数，这种绑定在执行函数时依然有效。
返回结果是“local scope”

闭包的这个特性：可以捕捉到局部变量和参数，并一直保持，看起来像这些变量绑定到了在其中定义它们的外部函数。

# 直觉问题： #
在外部函数中定义的局部变量在函数返回后就不存在了，那么嵌套的函数如何能调用不存在的作用域链呢？
首先我们需要了解基于栈的CPU架构：如果一个函数的局部变量定义在CPU的栈中，那么当函数返回时它们的确就不存在了。
那JavaScript是如何定义作用域链的？我们**将作用域链描述为一个对线列表**（堆上？），不是绑定在栈上，
每次调用JavaScript函数时，都会为它创建一个新的对象用来保存局部变量，把这个对象添加到作用域链中。函数返回时，就从作用域链中将这个绑定变量的对象删除。
* 如果没有嵌套函数，就没有其他引用指向这个绑定对象，它就会被当做垃圾回收掉。
* 如果定义了嵌套函数，每个嵌套的函数都各自对应一个作用域链，并且这个作用域链指向一个变量绑定对象。但如果这些嵌套的函数对象在外部函数中保存下来，那它们也会和所指向的变量绑定对象一样当垃圾回收掉。
* 但如果这个函数定义了嵌套的函数，并将它作为返回值返回或存储在某处的属性里。这时就会有一个外部引用指向这个嵌套的函数，而不会被当做垃圾回收掉，它所指向的变量绑定对象也不会被当做垃圾回收掉。

>需要避免“循环引用”，特别是DOM对象和JavaScript对象之间存在循环引用时，很容易在一些浏览器下造成内存泄漏。


# 作用域链 Scope chain？ #
是一个对象列表或链表，这组对象定义了这段代码作用域中的变量。
当JavaScript需要查找变量时（也称变量解析 variable resolution），它会从链中的第一个对象开始查找，如果这个对象有一个名为x的属性，则直接使用之，否则继续查找下一个对象，最后没找到就抛出一个引用异常（reference error）。
>在一个嵌套的函数体内，作用域链上至少有三个对象，
理解对象链的创建规则很重要，当定义一个函数时，它实际上保存一个作用域链。当调用这个函数时，它创建一个新的对象来存储它的局部变量，并将这个对象添加到保存的哪个作用域链上，同时创建一个新的更长的表示函数调用作用域的“链”。
对于嵌套函数，每次调用外部函数时，内部函数又会重新定义一遍，因为每次调用外部函数的时候，作用域链都是不同的。内部函数在每次定义时都有一些差别：在每次调用外部函数时，内部函数的代码都是相同的，而且关联这段代码的作用域链也不相同。