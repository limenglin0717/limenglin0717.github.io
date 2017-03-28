---
title: es6中的箭头函数
date: 2016-09-21 09:15:30
tags: es6
categories: es6 
---
ES6中引入了一种编写函数的新语法
```
    // ES5
    var selected = allJobs.filter(function (job) {
      return job.isSelected();
    });
    // ES6
    var selected = allJobs.filter(job => job.isSelected());
```
当你只需要一个只有一个参数的简单函数时，可以使用新标准中的箭头函数，它的语法非常简单：标识符=>表达式。你无需输入function和return，一些小括号、大括号以及分号也可以省略。

（我个人对于这个特性非常感激，不再需要输入function这几个字符对我而言至关重要，因为我总是不可避免地错误写成functoin，然后我就不得不回过头改正它。）

如果要写一个接受多重参数（也可能没有参数，或者是不定参数、默认参数、参数解构）的函数，你需要用小括号包裹参数list。
```
    // ES5
    var total = values.reduce(function (a, b) {
      return a + b;
    }, 0);
    // ES6
    var total = values.reduce((a, b) => a + b, 0);
```
我认为这看起来酷毙了。

那么不是非常函数化的情况又如何呢？除表达式外，箭头函数还可以包含一个块语句。回想一下我们之前的示例：
```
    // ES5
    $("#confetti-btn").click(function (event) {
      playTrumpet();
      fireConfettiCannon();
    });
```
这是它们在ES6中看起来的样子：
```
    // ES6
    $("#confetti-btn").click(event => {
      playTrumpet();
      fireConfettiCannon();
    });
```
这是一个微小的改进，对于使用了Promises的代码来说箭头函数的效果可以变得更加戏剧性，}).then(function (result) { 这样的一行代码可以堆积起来。

**注意，使用了块语句的箭头函数不会自动返回值，你需要使用return语句将所需值返回。**

小提示：当使用箭头函数创建普通对象时，你总是需要将对象包裹在小括号里。
```
 // 为与你玩耍的每一个小狗创建一个新的空对象
    var chewToys = puppies.map(puppy => {});   // 这样写会报Bug！
    var chewToys = puppies.map(puppy => ({})); //
```
用小括号包裹空对象就可以了。

不幸的是，一个空对象{}和一个空的块{}看起来完全一样。ES6中的规则是，紧随箭头的{被解析为块的开始，而不是对象的开始。因此，puppy => {}这段代码就被解析为没有任何行为并返回undefined的箭头函数。

更令人困惑的是，你的JavaScript引擎会将类似{key: value}的对象字面量解析为一个包含标记语句的块。幸运的是，{是唯一一个有歧义的字符，所以用小括号包裹对象字面量是唯一一个你需要牢记的小窍门。

## 这个函数的this值是什么呢？

普通function函数和箭头函数的行为有一个微妙的区别，**箭头函数没有它自己的this值**，箭头函数内的this值继承自外围作用域。

在我们尝试说明这个问题前，先一起回顾一下。

JavaScript中的this是如何工作的？它的值从哪里获取？这些问题的答案可都不简单，如果你对此倍感清晰，一定因为你长时间以来一直在处理类似的问题。

这个问题经常出现的其中一个原因是，无论是否需要，function函数总会自动接收一个this值。你是否写过这样的hack代码：

```
    {
      ...
      addAll: function addAll(pieces) {
        var self = this;
        _.each(pieces, function (piece) {
          self.add(piece);
        });
      },
      ...
    }
```

在这里，你希望在内层函数里写的是this.add(piece)，不幸的是，内层函数并未从外层函数继承this的值。在内层函数里，this会是window或undefined，临时变量self用来将外部的this值导入内部函数。（另一种方式是在内部函数上执行.bind(this)，两种方法都不甚美观。）

在ES6中，不需要再hackthis了，但你需要遵循以下规则：

* 通过object.method()语法调用的方法使用非箭头函数定义，这些函数需要从调用者的作用域中获取一个有意义的this值。
* 其它情况全都使用箭头函数。
```
    // ES6
    {
      ...
      addAll: function addAll(pieces) {
        _.each(pieces, piece => this.add(piece));
      },
      ...
    }
```
在ES6的版本中，注意addAll方法从它的调用者处获取了this值，内部函数是一个箭头函数，所以它继承了外围作用域的this值。

超赞的是，在ES6中你可以用更简洁的方式编写对象字面量中的方法，所以上面这段代码可以简化成：
```
    // ES6的方法语法
    {
      ...
      addAll(pieces) {
        _.each(pieces, piece => this.add(piece));
      },
      ...
    }
```
在方法和箭头函数之间，我再也不会错写functoin了，这真是一个绝妙的设计思想！

箭头函数与非箭头函数间还有一个细微的区别，箭头函数不会获取它们自己的arguments对象。诚然，在ES6中，你可能更多地会使用不定参数和默认参数值这些新特性。