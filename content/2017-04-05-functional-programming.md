+++
date = 2017-04-05
title = "函数式编程笔记(1)-基本概念"
description = "functional programming concepts"

[taxonomies]
categories = ["Programming"]
tags = ["function"]
+++

## 一等公民

不要把函数理解为特殊的一个类型，他们跟普通数据类型一样，可以被存在数组里，当作参数传递，赋值给变量等等。
把函数当作普通数据类型这样的代码会更容易理解和维护，同时需要正确的为函数命名。  

## 纯函数
> 纯函数是指对于一个函数，如果有相同的输入，那么输出永远都是相同的，并且没有任何副作用。

比如一个map函数
```javascript
const double = (x) => {
  return x.map(n => {return n * 2});
}
```
如果输入一个数组a，就会得到一个double后的新数组，只要a是相同的，那么结果永远都是相同的，并且原来的数组a不会被修改。

在函数式编程中，我们希望的都是这样的纯函数，他们在调用中是:

1. 可靠的；
2. 适合作cache；
3. 非常容易去测试；

调用纯函数会很大程度减少bug的产生几率和你为了找到bug而花费的debug时间。

当被调用的时候，纯函数一定可以被它的计算结果所替换掉，并且不会影响程序的运行结果，这被称为`透明引用` ([referential transparency](https://en.wikipedia.org/wiki/Referential_transparency))。

纯函数只依赖于它被调用时候的输入。

来看看几个不属于纯函数的例子：

```javascript
//splice change the value of parameter
const arr = [1,2,3];
arr.splice(0, 2);

//the result of add depends variable out of it's scope.
const baseNum = 8;
const add = (x) => {
  return x + baseNum;
}

// impure function producing a side effect
function showAlert() {
  alert('This is a side effect!');
  console.log('alert triggerd.');
}

// impure function calling pure functions procedurally
function proceduralFn() {
  const result1 = pureFnFirst(1);
  const result2 = pureFnLast(2);
  console.log(`Done with ${result1} and ${result2}!`);
}

// impure function that resembles a pure function,
// but returns different results given the same inputs
function getRandomRange(min, max) {
  return Math.random() * (max - min) + min;
}
```

总结一下纯函数的一些特征：
 * 纯函数必定要接收参数。
 * 相同的输入参数肯定会得到相同的结果。
 * 纯函数只依赖于自己的scope，并且不会对外部scope的状态做出改变。
 * 纯函数不会有任何副作用。
 * 纯函数不会调用任何非纯函数。

非纯函数的一些特征：

* 会修改它自己scope之外的状态。
* 没有返回值。
* 对于相同的输入，输出结果不确定。

#### 关于副作用(side effects)
副作用的关键部分在于“副”，大部分bug都是从这里出来的。对于副作用(side effects)在javascript里有下面一些定义:

如果有函数或者表达式修改了它自己的scope之外的状态，那么这个结果就叫做副作用。一些常见例子有：

1. 调用API
2. 更改文件系统
3. 操作DOM
4. 弹出alert，或者调用console打印log
5. 写数据库等。

# 柯里化(curry)
在了解了纯函数的概念后，我们肯定都期望后面运用函数式编程写出来的都是纯函数，我们需要一些工具来辅助我们做这些任务，柯里化(curry)就是一个非常重要的工具。

> 柯里化后的函数可以接收一部分参数，返回一个新的函数来接收剩余的参数. 我们可以一次性传所有参数来得到计算结果，也可以顺序传递多次参数，分多次调用得最终到结果。

举个例子：
```javascript
const add = R.curry((x, y) => {
  return x + y;
});

//get 3
add(1, 2)

//get a new function
const f1 = add(1)

//call the new function, get the result 3
f1(2)
```

我们定义了一个柯里化后的加法函数，可以直接传递2个参数给这个函数它会立即返回结果。也可以只传递一个参数，它会返回一个新的函数，这个函数接收一个参数，再次调用这个函数会返回最终结果。

再看一个例子：

```javascript
const formatName = R.curry((firstName, middleName, lastName) => {
  return `${firstName} ${middleName} ${lastName}`;
});

const formatDavid = formatName('David', 'M')

//get 'David M Xie'
formatDavid('Xie')

//get 'David M Dou'
formatDavid('Dou')

const formatJames = formatName('James')

//get 'James L Jones'
formatJames('L', 'Jones')
//get 'James R Paul'
formatJames('R', 'Paul')
```

会发现代码的reuse非常方便。而且我们特意把变化的数据，放在最后，函数的逻辑跟数据完全没有关系，让reuse会更方便，一会就能看到这样的好处。

#### 实际应用
最近在写一个api通过solr来作搜索查询，需要从用户的requestModel中拿到请求的条件，然后转成solr的query去solr中作search，其中用了不少柯里化，使整个代码看起来清晰不少。

```javascript
module.exports = (request) => {
 let query = solr.createClient().query()

 return R.compose(
    buildRows(request),
    buildStart(request),
    buildSearchRange('price', request.minPrice, request.maxPrice),
    buildSearchRange('area', request.minArea, request.maxArea),
    buildSearchRooms('bedroom', request.bedroom),
    buildCustomText(request),
    buildSortBy,
  )(query)
}

const buildRows = R.curry((request, query) => {
  if (request.rows) {
    query.rows(request.rows)
  } else {
    query.rows(20)
  }
  return query
})

const buildStart = R.curry((request, query) => {
  if (request.offset) {
    query.start(request.offset)
  }
  return query
})

const buildCustomText = R.curry((request, query) => {
  if (request.keyword) {
    query.q(request.keyword).df('keywords')
  }

  return query
})

const buildSearchRange = R.curry((fqField, min, max, query) => {
  if (min && max) {
    query.matchFilter(fqField, `[${min} TO ${max}]`)
    return query
  }

  if (min) {
    query.matchFilter(fqField, `[${min} TO *]`)
  }

  if (max) {
    query.matchFilter(fqField, `[* TO ${max}]`)
  }

  return query
})

const buildSearchRooms = R.curry((fqField, filterValue, query) => {
  if (filterValue) {
    query.matchFilter(fqField, filterValue)
  }

  return query
})

const buildSortBy = (query) => {
  query.sort({feature: 'desc', dorder: 'desc'})
  return query
}
```
在上面的代码中，主要逻辑是要根据requestModel中的不同field来构造不同的solr query条件，通过几个不同的函数来处理自己的逻辑，
最后调用compose把她们组合起来得到最终的结果, 这几个函数都被柯里化了，在组合的时候只传给他们一部分参数，返回一个新的function，
然后在组合调用最外面把最后一个参数传递进去，经过这一串的函数处理后，便得到了最终计算结果。
其实compose中就是一个pipeline，我更倾向于使用pipe操作。

# 组合 (compose)
组合简单来看就是把多个函数通过组合让他们生成一个新的函数，然后传入数据进行处理得到最后结果。
简单的例子

```javascript
const compose = (f, g) => {
  return (x) => {
    return f(g(x));
  }
};

```
这个例子中，f,g都是函数，x是要处理的数据。
```javascript
//get 'T'
compose(upperCase, getFirst)('test')
```
函数g会先于f执行，因此从调用角度来看，数据'test'在一个函数处理pipeline中先进入`getFirst`, 处理后的结果再进入`upperCase`, 最后结果返回。
在compose中数据是从右向左的顺序来在函数中流动。

compose的顺序反过来就是pipeline，喜欢用pipe(f1, f2, f3, f4)(data)的方式来处理数据。


# PointFree
> Pointfree style means never having to say your data

永远不要说出你的数据除非你马上就要使用它，即函数式变成你无需提及你将要操作的数据是什么样子的，只有当你要处理这个数据时候它才需要出场。
我们上面的那个构建solr query例子已经很明显，可以再看一个小例子：
```javascript
// 非 pointfree，因为提到了数据：data
var snakeCase = function (name) {
  return name.toLowerCase().replace('-', '_');
};

// pointfree
var snakeCase = compose(replace('-', '_'), toLowerCase);

//调用时候才需要数据
snakeCase(name)
```

函数式编程和柯里化对于pointfree的实现帮助太大了，通过柯里化和组合或者管道你可以不用关心传进来的数据是什么，只需要关注处理它的规则即可，这也是 `Ramda`和`lodash`或者`underscore`的最大区别。

#### 参考
https://drboolean.gitbooks.io/mostly-adequate-guide/content/
https://auth0.com/blog/glossary-of-modern-javascript-concepts
