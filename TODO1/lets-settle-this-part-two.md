> * 原文地址：[Let’s settle ‘this’ — Part Two](https://medium.com/@nashvail/lets-settle-this-part-two-2d68e6cb7dba)
> * 原文作者：[Nash Vail](https://medium.com/@nashvail?source=post_header_lockup)
> * 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
> * 本文永久链接：[https://github.com/xitu/gold-miner/blob/master/TODO1/lets-settle-this-part-two.md](https://github.com/xitu/gold-miner/blob/master/TODO1/lets-settle-this-part-two.md)
> * 译者：[geniusq1981](https://github.com/geniusq1981)
> * 校对者：

# 让我们一起解决“this”难题 — 第二部分

嘿!欢迎来到让我们解决“这个”的第二部分，我们试图揭开JavaScript中最不了解的一个方面 - “this”关键字。如果您还没有签出[第一部分](https://github.com/xitu/gold-miner/blob/master/TODO1/lets-settle-this-part-one.md)，您可能还想做所以。在第一部分中，我们通过15个不同的例子讨论了默认和隐式绑定规则。我们看到函数的'this'值如何根据函数的调用方式而改变。最后，我们也熟悉了箭头功能以及它们如何进行词法绑定。我希望你能记住一切。

在这部分我们将讨论两个新规则，从_new_绑定开始，我们将进入内部并看看一切是如何工作的。接下来，我们将介绍显式绑定以及如何通过_call(...)，apply(...)_和_bind(...)_方法将任何对象作为'this'绑定到任何函数。

让我们继续我们离开的地方。您的任务是相同的，猜测记录到控制台的内容。还记得WGL吗？

But在我们潜入之前，让我们热身一点。

#### Example #16

```
function foo() {}

foo.a = 2;
foo.bar = {
 b: 3,
 c: function() {
  console.log(this);
 } 
}

foo.bar.c();
```

我知道你现在就像“发生了什么事？为什么在这里将属性分配给函数？这不会导致错误吗？“好吧，首先，这不会导致错误。JavaScript中的每个函数也都是一个对象。就像任何其他常规对象一样，您也可以为函数指定属性!

With那个让我们现在弄清楚什么被记录到控制台。如果您注意到，隐式绑定在此处起作用。直接在_c'_s调用之前的对象引用是_bar_不是吗？因此_c_中的'this'是_bar_，因此_bar_是记录到控制台的内容。

你应该从这个例子中得到的一件事是，JavaScript中的函数也是对象，就像任何其他对象一样，它们可以被赋予属性。

#### Example #17

```
function foo() {
 console.log(this);
}

new foo();
```

So，会记录什么？或者根本没有记录任何东西？

答案是一个空对象。是的，不是_a_不是_foo_只是一个空对象。让我们看看它是如何工作的。

First of notice notice ** _如何_ **函数被调用。它不是一个独立的调用，也没有它前面的对象引用。它有什么在呼叫前_new_。可以使用前面的_new_关键字调用JavaScript中的任何函数。当发生这种情况时，当用_new_调用一个函数时，大约会发生四个事件，其中两个是，

 1。创建一个空对象。
 2。新创建的对象被绑定为函数调用的“this”。

And(2.)是你执行上面显示的代码时得到一个空对象登录到控制台的原因。你可能会问“这怎么会有用？”。我们会发现它有点争议。

#### Example #18

```
function foo(id, name) {
 this.id = id;
 this.name = name;
}

foo.prototype.print = function() {
 console.log( this.id, this.name );
};

var a = new foo(1, ‘A’);
var b = new foo(2, ‘B’);

a.print();
b.print();
```

直观地说，在这个例子中很容易猜到登录到控制台的内容，但是你在技术上是否合适？让我们来看看。

To回顾一下，当使用_new_关键字调用函数时，会发生四个事件。

 1。创建一个空对象。
 2。新创建的对象被绑定为函数调用的“this”。
 3。**新创建的对象原型链接到函数的原型对象。**
 4。**函数正常执行，最后返回新创建的对象*。**

We在前面的例子中已经验证了前两个事件实际上发生了，这就是我们将一个空对象记录到控制台的原因。现在忘了大约3，让我们专注于第四场比赛。没有什么能阻止函数的执行，除了函数内部的“this”是新创建的空对象之外，函数的执行参数与任何其他正常的JavaScript函数一样。因此，当在函数内部_foo_在我们的情况下，我们执行类似_this.id = id_ **的操作时，我们实际上将属性分配给新创建的空对象，该对象在调用函数**中被绑定为“this”。再读一遍。一旦函数完成执行**，就会返回相同的新创建的对象**。由于在上面的示例中，我们为返回的对象具有的对象分配了_id_和_name_属性。然后可以将返回的对象分配给我们想要的任何内容，就像我们在上面的示例中对_a_和_b_所做的那样。

Every函数调用_new_导致创建一个全新的空对象，它在函数内部可选扩充(_this.propName = ...)_并在函数执行结束时返回它。因此，最后我们的变量_a_和_b_看起来像这样。

```
var a = {
 id: 1,
 name: ‘A’
};

var b = {
 id: 2,
 name: ‘B’
};
```

大!我们刚刚学会了创建对象的新方法。但是_a_和_b_有一些共同之处，它们都是**原型链接到_foo_的原型**(事件4)，因此可以访问它的属性(变量，函数e.t.c)。正因为如此，我们可以调用_a.print()_和_b.print()_，因为_print_是我们在_foo_原型中创建的函数。快速问题，当我调用_a.print()_时会发生什么绑定？如果你说Implicit，那你就绝对正确。因此，在调用_a.print()_'时，_print_里面的这个'是_a_，并且首先登录到控制台的是_1，A_，同样当我们调用_b.print()2时，B_会被记录。

#### Example #19

```
function foo(id, name) {
 this.id = id;
 this.name = name;

 return {
  message: ‘Got you!’
 };
}

foo.prototype.print = function() {
 console.log( this.id, this.name );
};

var a = new foo(1, ‘A’);
var b = new foo(2, ‘B’);

console.log( a );
console.log( b );
```

几乎所有内容都与上一个示例中的代码相同，只是注意_foo_ now现在返回一个对象。好吧，做一件事回到上一个例子并重读第四个事件你呢？注意*****？当使用_new_关键字调用函数时，在执行结束时返回新创建的对象**，除非**您返回自己的自定义对象，就像我们在此示例中所做的那样。

所以？记录了什么？很明显，它是返回的对象，具有_message_属性的对象会被记录到控制台，两次。打破整个构造是如此容易，不是吗？只返回一个没有意义的对象，一切都失败了。此外，您现在无法调用_a.print()_或_b.print()_，因为_a_和_b_被分配了返回的内容，并且我们返回的对象未与_foo_的原型进行原型链接。

但是等一下，如果不是返回一个对象，我们返回一个像_'abc'_或数字或布尔值或函数或null或undefined或数组的字符串？事实证明，构造是否破坏取决于你返回的内容。看到这里的模式？

```
return {}; // Breaks
return function() {}; // Breaks
return new Number(3); // Breaks
return [1, 2, 3]; // Breaks
return null; // Doesn’t break
return undefined; // Doesn’t break
return ‘Hello’; // Doesn’t break
return 3; // Doesn’t break
...
```

为什么会发生这是另一篇文章的主题。我的意思是我们已经离这里有点偏僻了，这个例子与'this'绑定没什么关系吗？

通过_new_绑定整个创建对象已经被使用(误用？)，以便在JavaScript中伪造传统类。实际上，在JavaScript中没有类，ES2015中的新_class_语法只是语法。在幕后_new_绑定是发生的事情，那里没有变化。我一个人不关心你是否使用_new_绑定到假类，直到你的程序工作，代码是可扩展，可读和可维护的。但是，你又如何拥有可扩展，可读和可维护的代码，所有的包和脆弱性_new_绑定带来了？

That可能有很多内容。如果你还有点失落，你应该重新阅读。重要的是你要了解_new_绑定的工作原理可能永远不会再使用它:)。

Enough严肃的谈话，让我们继续前进。

Consider下面的代码。不要猜测在这个例子中记录了什么，我们将从下一个例子开始继续“猜谜游戏”:)。

```
var expenses = {
 data: [1, 2, 3, 4, 5],
 total: function(earnings) {
  return this.data.reduce( (prev, cur) => prev + cur ) — (earnings || 0);
 }
};

var rents = {
 data: [1, 2, 3, 4]
};
```

_expenses_对象具有_data_和_total_ properties_。data_包含一些数字而_total_是一个函数，它将_earnings_作为参数并返回_data_中所有数字的总和减去_earnings._非常简单。

Now看看_rents，_就像费用一样，它也有_data_属性。现在说，出于某种原因，只是假设在这里，你想在_rent_的_data_数组上运行_total_函数，因为我们是优秀的程序员，我们不喜欢重复自己。我们绝对不能做_rents.total()_并且_rents_被隐式地绑定为_total_函数的'this'，因为_rents.total()_是一个无效的调用，因为_rents_没有名为_total._的属性。只有当有一种方法将_rents_绑定为'this'到_total_函数时。好吧猜怎么着？有，请允许我向您介绍_call()_和_apply()._

你看_call_和_apply_做同样的事情，它们允许你将你想要的对象绑定到你想要的功能。这意味着我可以做到这一点......

```
console.log( expenses.total.call(rents) ); // 10
```

...and this.

```
console.log( expenses.total.apply(rents) ); // 10
```

Which很棒!上面的两行代码都会导致_total_被调用为'this'作为_rents_对象。_call_和_apply_就'this'绑定而言，只涉及传递参数的方式不同。

Notice函数_total_接受参数_earnings，_让我们传递它。

```
console.log( expenses.total.call(rents, 10) ); // 0 Works!
console.log( expenses.total.apply(rents, 10) ); // Error
```

Passing目标函数的参数(在我们的例子中是_total_)通过_call_很简单，你只需传入一个逗号分隔的参数列表，比如任何其他JavaScript函数_.call(customThis，arg1，arg2，arg3 ......)._在上面的代码我们传入了10个_earnings_参数，一切都按预期工作。

_apply_虽然要求你将参数传递给目标函数(在我们的例子中是_total_)包装在一个数组_.apply(customThis，[arg1，arg2，arg3 ...])W_hich通知我们没有在上面的代码片段中执行一个错误。错误肯定可以通过在数组中包装目标函数的参数来修复，就像这样。

```
console.log( expenses.total.apply(rents, [10]) ); // 0 Works!
```

Enemonic我用来记住_call_和_apply_之间的区别就是这样的。A代表** _ a _ ** _ pply_，A代表** a ** rray!所以通过** _ a _ ** _ pply_对目标函数的参数传递包含在** a ** rray中。只是一个愚蠢的小助记符，但它的作用😬。

Now如果我们传入一个数字或一个字符串或一个布尔值或null / undefined而不是'this'的对象文字传递给** _ call _ ** _，_ ** _ apply _ **和** _ bind _ **(接下来讨论)_._然后会发生什么？没有什么，比如你传入2号'this'它被包裹在它的对象形式_new Number(2)_同样如果你传入一个字符串它变成_new String(...)_ boolean values变为_new Boolean(...)_ so so等等这个新对象是String还是Number或Boolean被绑定为被调用函数的'this'。传入_null_和_undefined_虽然会产生不同的结果。如果为'this'传入_null_或_undefined_，则调用该函数，就好像它接受默认绑定一样，这意味着全局对象被绑定为被调用函数的“this”。

还有另一种方法将'this'绑定到一个函数，这次通过一个名为的方法，等待它，_bind_!

让我们看看你是否可以解决这个问题。在下面的示例中记录了什么？

#### Example #20

```
var expenses = {
 data: [1, 2, 3, 4, 5],
 total: function(earnings) {
  return this.data.reduce( (prev, cur) => prev + cur ) — (earnings   || 0);
 }
};

var rents = {
 data: [1, 2, 3, 4]
};

var rentsTotal = expenses.total.bind(rents);

console.log(rentsTotal());
console.log(rentsTotal(10));
```

这个例子的答案是10后跟0.注意_rents_对象声明下面发生了什么。我们正在从函数_expenses.total._创建一个新函数_rentsTotal_这就是_bind_创建一个新函数，当被调用时，它的'this'关键字设置为提供的值(在我们的例子中是_rents_)。因此，当我们调用_rentsTotal()_时，虽然它是一个独立的调用，但它的'this'已设置为_rents_，而Default Binding不能覆盖它。此调用导致10打印到控制台。

在下一行中，使用参数(10)调用_rentsTotal_与使用相同的参数(10)调用_expenses.total_完全相同，它只是在'this'的值中，它们不同。此调用的结果为0。

Moreover你也可以使用_bind绑定目标函数的参数(在我们的例子中是_expenses.total_)。考虑这个。

```
var rentsTotal = expenses.total.bind(rents, 10);
console.log(rentsTotal());
```

您认为如何登录控制台？0当然因为10已被绑定到目标函数(_expenses.total)_作为_earnings_由_bind._

让我们看一个说明_bind._的真实生活用法的例子。

#### Example #21

```
// HTML

<button id=”button”>Hello</button>

// JavaScript

var myButton = {
 elem: document.getElementById(‘button’),
 buttonName: ‘My Precious Button’,
 init: function() {
  this.elem.addEventListener(‘click’, this.onClick);
 },
 onClick: function() {
  console.log(this.buttonName);
 }
};

myButton.init();
```

We已经在HTML中创建了一个按钮，然后我们将JavaScript代码中的相同按钮定位为_myButton._注意_init_中的内容我们还在按钮上附加了一个click事件监听器。您现在的问题是记录到控制台的内容单击按钮时？

如果您猜对了_undefined_就是记录的内容。这种“巫术”的原因是作为回调(在我们的例子中是_this.onClick_)传递给事件侦听器的函数将目标元素绑定为“this”。这意味着当_onClick_被称为'this'时，它是DOM对象按钮(_elem_)而不是我们的_myButton_对象，因为DOM对象按钮没有名称为_buttonName_的属性，导致_undefined_被记录到控制台。

但是有办法解决这个问题(双关语)。我们需要做的就是添加一个，只需添加一行代码。

#### Fix #1

```
var myButton = {
 elem: document.getElementById(‘button’),
 buttonName: ‘My Precious Button’,
 init: function() {
  this.onClick = this.onClick.bind(this);
  this.elem.addEventListener(‘click’, this.onClick);
 },
 onClick: function() {
  console.log(this.buttonName);
 }
};
```

Notice在上一个片段(#21)中调用函数_init_的方式。确切地说，Implicit绑定将_myButton_绑定为'this'到_init_函数。现在注意我们在新行中如何将_myButton_绑定到_onClick_函数。这样做会创建一个新的函数，它完全是_onClick_，除了它的'this'为_myButton_ object _._然后新创建的函数被重新分配给_myButton.onClick._这就是全部，当你点击按钮时，你现在'将“My Precious Button”记录到控制台。

你也可以使用箭头功能修复代码。就是这样。我会告诉你为什么这些工作。

#### Fix #2

```
var myButton = {
 elem: document.getElementById(‘button’),
 buttonName: ‘My Precious Button’,
 init: function() {
  this.elem.addEventListener(‘click’, () => {
   this.onClick.call(this);
  });
 },
 onClick: function() {
 console.log(this.buttonName);
 }
};
```

#### Fix #3

```
var myButton = {
 elem: document.getElementById(‘button’),
 buttonName: ‘My Precious Button’,
 init: function() {
  this.elem.addEventListener(‘click’, () => {
   console.log(this.buttonName);
  });
 }
};
```

而已。我们差不多完成了。还有一些问题，比如是否有优先顺序？如果试图将“this”绑定到同一个函数的两个规则之间存在冲突怎么办？这是另一篇文章的主题。第3部分？可能并且说实话，你很少遇到这样的冲突。所以现在我们已经完成了，让我们总结一下我们在这两部分学到的东西。

#### 总结

在第一部分中，我们看到函数的“this”如何不固定，并且可以根据函数的调用方式而改变。我们讨论了默认绑定，它适用于函数经历独立调用时，隐式绑定适用于调用函数时，前面有一个对象引用和箭头函数，以及它们如何以词法方式绑定它们。接近第一部分的结尾，我们还在JavaScript对象中进行了自引用的怪癖。

在这一部分，我们开始使用_new_绑定以及它如何工作以及如何轻松地破坏整个结构。这一部分的后半部分致力于使用_call，apply_和_bind._明确地将'this'绑定到函数。我还尴尬地与你分享了关于如何记住_call_和_apply之间差异的助记符。希望你能记住它。

#### 这篇文章很长。非常感谢你能一直读完。我希望这篇文章能让你学到些东西。如果觉得还不错，也请把这篇文章推荐给其他人吧。祝你一天都有好心情！

> 如果发现译文存在错误或其他需要改进的地方，欢迎到 [掘金翻译计划](https://github.com/xitu/gold-miner) 对译文进行修改并 PR，也可获得相应奖励积分。文章开头的 **本文永久链接** 即为本文在 GitHub 上的 MarkDown 链接。


---

> [掘金翻译计划](https://github.com/xitu/gold-miner) 是一个翻译优质互联网技术文章的社区，文章来源为 [掘金](https://juejin.im) 上的英文分享文章。内容覆盖 [Android](https://github.com/xitu/gold-miner#android)、[iOS](https://github.com/xitu/gold-miner#ios)、[前端](https://github.com/xitu/gold-miner#前端)、[后端](https://github.com/xitu/gold-miner#后端)、[区块链](https://github.com/xitu/gold-miner#区块链)、[产品](https://github.com/xitu/gold-miner#产品)、[设计](https://github.com/xitu/gold-miner#设计)、[人工智能](https://github.com/xitu/gold-miner#人工智能)等领域，想要查看更多优质译文请持续关注 [掘金翻译计划](https://github.com/xitu/gold-miner)、[官方微博](http://weibo.com/juejinfanyi)、[知乎专栏](https://zhuanlan.zhihu.com/juejinfanyi)。
