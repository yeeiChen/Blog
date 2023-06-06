关于 this 的指向问题是每个新人都需要抓狂的必经之路。许多新人包括我自己都是凭借着以往对 this 的简单总结，来模糊地判断 this 的指向问题，以至于给自己挖下许多大坑。这篇文章带你从本质和定义上出发，彻底弄明白 this 的指向问题。

首先想要真正弄明白 this 的指向问题，我们需要对 **作用域** 以及 **闭包** 有深刻的认识。因此在正文开始前我们通过图文和代码结合的方式先粗略地讲解一下两者。

## 什么是作用域？

首先来看看第一段代码以及在浏览器里看到的结果。

```js
var a = 1;

{
  var b = 2;
}
console.log(a, b);
debugger;
```

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84cf80285eda43e9aae0b218ba2c5518~tplv-k3u1fbpfcp-watermark.image?" alt="image.png" width="100%">

这里可以很清楚地看见，我们定义的 a 和 b 被挂载到 **作用域**-**全局**(**winodw**) 下面了

再来看看第二段代码以及在浏览器里看到的结果。

```js
{
  let blockC = 3;
  console.log(blockC);
  debugger;
}
```

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a688d8a997e4928b2d2d2d5bd07017c~tplv-k3u1fbpfcp-watermark.image?" alt="image.png" width="100%">

这里可以很清楚地看见，我们定义的 blockC 被挂载到 **作用域**-**代码块**(**block**) 下面了。

最后再来看下第三段代码以及在浏览器里看到的结果

```js
function localTest() {
  const localA = 1;
  console.log(localA);
  debugger;
}

localTest();
```

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/615d5d5b5d104c55bee404f9539beb24~tplv-k3u1fbpfcp-watermark.image?" alt="image.png" width="100%">

这里可以很清楚地看见，我们定义的 localA 被挂载到 **作用域**-**本地**(**local**) 下面了。

### 小结

通过上面三个例子我们可以明白 js 中作用域分别是**全局(window)、代码块(block)、本地(local)**

## 什么是闭包？

我们依旧通过图文和代码结合的方式来讲解。

来看看此段代码以及浏览器里看到的结果。

```js
function scoping() {
  const closureA = 3;

  return function () {
    const scA = 4;
    console.log(closureA, scA);
    debugger;
  };
}

const sp = scoping();

sp();
```

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4f6e5cd5cdd4ded90e7bbe787fdeb9d~tplv-k3u1fbpfcp-watermark.image?" alt="image.png" width="100%">

图中可以看见函数 sp 用来接收高阶函数的返回值,其 **本地**(**local**) 作用域下存有 scA 的值，更关键的是在其 **Scopes** 中，存放着一层 **Closure** ，内部有我们定义的 closureA，并且 Closure 是内部第一层的存在！！！从这里你已经大概能明白了，这个 Closure 才是闭包的核心。当然闭包有几个特殊例子，感兴趣的朋友们可以自己去看下，这里只作简略说明。

关于作用域和闭包，大家可以自行查阅 [《你不知道的 JavaScript(上卷)》](https://book.douban.com/subject/26351021/)
第一部分，这里不再用过多篇幅简述。

接下来让我们进入正文。

## this 真的不简单

首先我们来看看《你不知道的 JavaScript》中对 this 的定义

> this 既不指向函数自身也不指向函数的词法作用域，**this 的绑定和函数声明的位置没有任何关系**，只取决于函数的调用方式。 当一个函数被调用时，会创建一个活动记录（有时候也称为**执行上下文**）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。this 就是记录的其中一个属性，会在函数执行的过程中用到。

划重点:

1.  this 的绑定和函数声明的位置没有任何关系
2.  函数被调用时,会创建一个活动记录(也称执行上下文)

所以不要再 _执行上下文_ 和 _作用域_ 傻傻分不清，执行上下文只有当函数调用时才会生成！！！

废话不多说,我们直接做几道题(不严格模式)来判断你是否真的了解 this。

_建议先手写一遍然后检验答案。_

```js
var name = "window";

function foo1() {
  var name = "foo";
  console.log(this.name);
}

function foo2() {
  var name = "foo1";
  return function () {
    console.log(this.name);
  };
}

function foo3() {
  var name = "foo3";
  return () => {
    console.log(this.name);
  };
}

var obj = {
  name: "obj",
};

foo1();
foo1.call(obj);

foo2()();
foo2.call(obj)();
foo2().call(obj);

foo3()();
foo3.call(obj)();
foo3().call(obj);
```

---

---

答案如下:

```js
foo1(); // window
foo1.call(obj); // obj

foo2()(); // window
foo2.call(obj)(); // window
foo2().call(obj); // obj

foo3()(); // window
foo3.call(obj)(); // obj
foo3().call(obj); // window
```

对比你的答案，不知道你做对没有。

我们来分析一下:

首先 `foo1()` 和 `foo1.call(obj)` 很好理解。分别是 window 和 obj 调用了 foo1 函数

`foo2`是一个高阶函数,不了解的朋友可以将内部的返回函数看作一个整体。`foo2()()`就相当于定义了一个新函数接受 `foo2` 函数的返回值

```js
const Foo2 = function () {
  console.log(this.name);
};

Foo2();
```

因此 `foo2()()` 相当于 `window.Foo2()`，所以输出为 window

了解了高阶函数后 `foo2.call(obj)()` 和 `foo2().call(obj)` 也就好理解`foo2.call(obj)()` 就是通过 `obj` 调用了高阶函数 `foo2`,可以看作 `obj.(Foo2())`,但是括号内函数已经形成输出,所以还是输出 window.`foo2().call(obj)`就是通过 obj 调用了 foo2 高阶函数的返回值,可以看作 obj.Foo2(),所以输出 obj

其实上面四个都比较好理解，接下来关键来了！`foo3` 同样是一个高阶函数，与 `foo2` 不同的是 `foo3` 返回的是一个 `箭头函数`。大家都知道 ES6 后引入了箭头函数,箭头函数没有自己的 this,它的 this 指向吧啦吧啦吧啦......网上一堆答案看的我头晕目眩,最后我也没搞清 `箭头函数` 的 this 指向什么。_所以箭头函数的 this 到底指向什么呢？_

来看下《你不知道的 JavaScript》中对箭头函数的 this 的说明。

> 箭头函数根据当前的**词法作用域**来决定 this，会**继承外层函数调用的 this 绑定**（无论 this 绑定到什么）。和 ES6 之前代码中的 self = this 机制一样。

划重点:

1.  词法作用域
2.  继承外层函数调用的 this 绑定

说结论:

**箭头函数没有自己的 this,它的 this 指向函数定义时最近执行上下文所在的位置，没有则指向 window.同时 this 指向一旦确定就无法被修改.**

如果对箭头函数 this 指向不是很清楚的朋友建议结合书中说明，然后反复品读这句话。如果不清楚`执行上下文`的概念的朋友可以回到目录`-this真的不简单`开头查看。

为了更好地理解和加深记忆，我们做一套题来验证一下。同之前一样，建议先手写答案。

```js
var name = "window";

var person1 = {
  name: "person1",
  show1: function () {
    return () => console.log(this.name);
  },
  show2: () => {
    return () => console.log(this.name);
  },
};

var obj = {
  name: "obj",
};

person1.show1()();
person1.show1().call(obj);
person1.show1.call(obj)();

person1.show2()();
person1.show2().call(obj);
person1.show2.call(obj)();
```

---

---

答案如下:

```js
person1.show1()(); // person1
person1.show1().call(obj); // person1
person1.show1.call(obj)(); // obj

person1.show2()(); // window
person1.show2().call(obj); // window
person1.show2.call(obj)(); // window
```

不知道你是不是做对了,接下来我们通过对箭头函数的小结深入分析一下。

> **箭头函数没有自己的 this,它的 this 指向函数定义时最近执行上下文所在的位置，没有则指向 window.同时 this 指向一旦确定就无法被修改.**

`person1.show1()()` 是个高阶函数,返回一个箭头函数,箭头函数没有自己的 this,会寻找离其最近的执行上下文,最近的执行上下文为`show1()` ，`show1()`所在的位置是 person1,所以输出 person1。这样前大段是不是都一一对上了？

再来看`person1.show1().call(obj)`, person1.show1( ) 时,高阶函数 show1 内部的箭头函数执行,箭头函数没有自己的 this,寻找**最近**的执行上下文，找到 **show1** ,show1 所在的位置为 **person1**,此时箭头函数的 this 确定，即**person1**。**箭头函数的 this 指向确定后无法修改**,所以输出 person1。按照这个思路来理解`person1.show1.call(obj)()` 就很容易了,在箭头函数执行前,show1 高阶函数所在的位置已经从 person1 变化到 obj,最终输出 obj。

同上所述，我们一样根据小结来分析 show2 高阶函数的 this 指向。是不是有朋友发现自己的答案不一样?那是因为这里其实还考察了对 `执行上下文` 概念的理解。

在 `person1.show2()()` 执行时,内部返回的箭头函数因为自身没有 this,就向外寻找,找到 `show2` ,但是由于 `show2` 本身也是箭头函数，它也没有 this，因此它会继续向外寻找。注意,关键点来了,因为 person1 本身是个**对象**,**没有执行上下文**,所以内部会继续往外寻找,直到找到 **window** 并且确定指向。

> 当一个**函数**被调用时，会创建一个活动记录（有时候也称为执行上下文）。

同理: `person1.show2().call(obj)` 和 `person1.show2.call(obj)()` 输出都是 window。

## 构造函数中的 this 指向

其实构造函数中的 this 和上面小结的是一样的，只是因为结构的不同，以及会涉及到其它术语的概念，也会造成很多误解。废话不多说，我们再来做几道题。

```js
function Person(sex) {
  this.sex = sex;
  this.log1 = function () {
    console.log(this.sex);
  };
  this.log2 = () => console.log(this.sex);
  this.log3 = function () {
    return function () {
      console.log(this.sex);
    };
  };
  this.log4 = function () {
    return () => console.log(this.sex);
  };
  this.log5 = () => {
    return () => console.log(this.sex);
  };
}

var boy = new Person("boy");
var girl = new Person("girl");

boy.log1();
boy.log1.call(girl);

boy.log2();
boy.log2.call(girl);

boy.log3()();
boy.log3().call(girl);
boy.log3.call(girl)();

boy.log4()();
boy.log4().call(girl);
boy.log4.call(girl)();

boy.log5()();
boy.log5().call(girl);
boy.log5.call(girl)();
```

答案如下:

```js
boy.log1(); // boy
boy.log1.call(girl); // girl

boy.log2(); // boy
boy.log2.call(girl); // boy

boy.log3()(); // window
boy.log3().call(girl); // girl
boy.log3.call(girl)(); // window

boy.log4()(); // boy
boy.log4().call(girl); // boy
boy.log4.call(girl)(); // girl

boy.log5()(); // boy
boy.log5().call(girl); // boy
boy.log5.call(girl)(); // boy
```

相信很多朋友的答案并不完全正确,让我们从概念和本质出发。

来看一下《javascript 高级程序设计》(红宝书)中的概念描述，通过构造函数创建对象(也就是 new)时整个过程发生了什么。

> 1.  创建一个新对象；
> 2.  将构造函数的作用域赋给新对象（因此 this 就指向了这个新对象）；
> 3.  执行构造函数中的代码（为这个新对象添加属性）；
> 4.  返回新对象。

关键在于第二条，构造函数创建对象时会将 **构造函数的作用域赋给新对象** ，这话是什么意思呢，如果你把这句话简单理解成`将构造函数创建时的执行上下文赋给新对象`是不是好理解多了？记住这句话，然后来分析一下题目答案。

`boy.log1()`和`boy.log1.call(girl)`不用多说，相信大家都明白.

`boy.log2()`和`boy.log2.call(girl)`开始就可以验证红宝书上的概念理论了,构造函数的作用域全部赋给了新对象，箭头函数没有自己的 this,执行时向外寻找找到`boy`并且确定,后续无法再修改.

`boy.log3()()`和`boy.log3().call(girl)`以及`boy.log3.call(girl)()`同第一套题中相似的题目原因一样,这里不再作过多解释,**普通函数的 this 指向它的直接调用者**.

`boy.log4()()`和`boy.log4().call(girl)`以及`boy.log4.call(girl)()`与第一二道题原因一样,`箭头函数的this一旦确定，就无法再修改`.

`boy.log5()()`和`boy.log5().call(girl)`以及`boy.log5.call(girl)()`需要讲一下,为什么输出都是 boy?其实原因和概念一样。log5 作为一个高阶函数,返回一个没有自身 this 指向的箭头函数,但是其本身也是没有 this 指向的箭头函数,和内部返回函数不同的是两者执行的时机不一样.log5 本身在 new 时就已经被确定,根据 new 时第二步中会**将构造函数的作用域赋给新对象**的结论可知,`log5`会找到`boy`并且确定无法修改。然后在内部箭头函数执行时,它会寻找最近的执行上下文所在的位置,此时就是 log5,而 log5 又指向 boy，形成了一条 `boy5->log5->log5的内部返回函数` 链。因此无论做什么操作，输出都是 boy。

是不是发现，构造函数创建对象，以及箭头函数的 this 指向问题，就像闭包和作用域链？

## 总结

在学习的过程中我们应该热衷深入浅出的方法,从概念以及理论入手,最后通过通俗的话来总结。

现在我们可以对函数中 this 的指向问题下结论了。

**普通函数中的 this 指向它的直接调用者**，**箭头函数没有本身的 this,其指向最近执行上下文所在的位置**。

箭头函数的 this 指向读起来还是有点拗口,我们通过以下几点再把它简化一下。

1. 箭头函数本身没有 this,我们可以把 this 看作一个变量对象
2. 箭头函数的 this 类似原型链中的继承,其会沿着链条一层一层往上找
3. 箭头函数的 this 一旦确定,就无法修改

## 引用

《你不知道的 JavaScript》

《javascript 高级程序设计》第四版

---

第一次在掘金输出文章,作为学习记录,同样也希望一直在别人的帮助下走来的自己也可以走上授之以鱼的道路,如果有什么不正确或者疑问的地方欢迎指出
