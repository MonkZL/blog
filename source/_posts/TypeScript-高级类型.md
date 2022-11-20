---
title: TypeScript 高级类型
date: 2022-11-20 11:30:18
categories: TypeScript
tags: Advanced Type
---

# 前言

- - -
之前一直使用的是 **js** 现在转过来学习 **ts** 的时候尝到了 **ts** 对类型规范的很多好处，相应的 **ts** 的类型有时候也让人头大,下面简单总结一下自己对 **ts** 高级类型的学习成果。

# 一般类型
- - -

    //给变量指定类型
    const value: string = "value"
    //给方法的形参和返回值指定类型
    const fun0 = (str: string): string => {
        return ""
    }
    //甚至可以直接给方法指定类型
    type funType = (str: string) => string
    const fun1: funType = (str) => {
        return ""
    }

这个时候我们可以看的 `fun1` 这时候是不用指定 `str` 的类型的，IDE会自动提示，如下：

<img src="/images/5125944-2a5d0f6c35cf0f85.png" width="100%">
<br>

但还有个特殊的情况 `fun1` 的形参是可以不写的，比如：

    type funType = (str: string) => string
    const fun2: funType = () => {
        return "fun2 return value"
    }
    console.log(fun2("value"));

这样是不会报错的，可是为什么呢？我们明明指定了`funType`的形参有且是`string`，那大家肯定想问真是为什么，抱歉我也不知道，哈哈哈！如果有知道的朋友希望能留言告知。这样做的问题是在`fun2`里面是拿不到传入的形参的，不知道是不是bug。

# 泛型
- - -

对就是它，我个人来说在学习和使用`ts`的途中最头疼的就是泛型，`ts`中泛型的使用真的让人头晕眼花，但当你真的看明白`ts`中泛型的使用是又不得不说这简直就像是在变魔术。下面记录一下泛型的一般用法：

    //将对象中值为number类型的数据组成一个新对象
    function getObjAllNum<T>(t: T): any {
        let numObj = {} as any;
        for (const tKey in t) {
            if (typeof t[tKey] == "number") {
                numObj[tKey] = t[tKey]
            }
        }
        return numObj
    }
    const value = {
        name: 'zl',
        age: 27,
        sex: 'm'
    }
    //输出 {age: 27}
    console.log(getObjAllNum(value));

可以看到，当我们调用`getObjAllNum`的时候`T`类型已经转变成对象`value`的类型了。

<img src="/images/5125944-9c713ea3c4d78cfd.png" width="100%">
<br>

# 高级类型
- - -

接下来要说的就是`ts`为我们提供的一些类型的高级操作方式

## 类型别名（type）

可以理解成可以给一个类型再取另一个名字，比如：

    type myString = string
    const str: myString = "str"

## 联合类型（|）

`A|B`表示`A`或者`B`中的任意一个类型 ，如下代码，`zhangSan`就是属于`Man`类型，`xiaoLi`属于`Woman`类型。

    interface Man {
        working(): void;
    }
    interface Woman {
        shopping(): void;
    }
    type Person = Man | Woman
    const zhangSan: Person = {
        working() {
            console.log('working')
        }
    }
    zhangSan.working()
    const xiaoLi: Person = {
        shopping() {
            console.log('shopping')
        }
    }
    xiaoLi.shopping()

但需要注意的是下面这种写法：

    const person: Person = {
        working() {
            console.log('working')
        },
        shopping() {
            console.log('shopping')
        }
    }

这样写是不会报错的，但是当试图调用方法的时候就会报错，可以看出调用`shopping`的时候，就会提示`Man`不存在`shopping`属性，其实这是因为`A|B`联合类型表示的是`A`或者`B`中的任意一个类型，只有一个类型，而不是两种类型的合并，下面来说说真正的合并`交叉类型（&）`

<img src="/images/5125944-7709f5ba1f5db227.png" width="100%">
<br>


## 交叉类型（&）

`A&B`高级类型是将两个类型合并成了一个类型，这个类型拥有`A`和`B`的所有属性，所以称其为`合并类型`也没啥毛病。**这东西才学的时候一直以为是交集，其实应该是并集才对。**

    interface Apple {
        size: string,
        color: string,
    }
    interface Pen {
        type: string,
        color: string,
    }
    type Pineapple = Apple & Pen
    const pineapple: Pineapple = {
        size: '大',
        color: '黄色',
        type: '海南凤梨'
    }

这里有个问题就是两个`Interface`做交叉类型时，如果含有相同的`key`会出现什么问题呢？[详情请见另一篇](https://www.jianshu.com/p/38bc2dac26b5)。

## 类型索引（keyof）

`keyof`的作用是将一个类型拆分开了，将拆分出来的子类型的集合作为类型返回，如下代码，`PersonKeys`的类型（图`keyof.png`所示）为`name|age`这种联合类型

    interface Person {
        name: string,
        age: number
    }
    type PersonKeys = keyof Person

<img src="/images/5125944-20ef51582f5f8d3a.png" width="100%">
<br>

## 类型约束（extends）

`extends`的作用是约束泛型，将泛型的类型限定成某个类型，如下例子:`logPersonInfo<Person>()`可以传入`Man`和`Woman`两种类型，但`logPersonInfo<Man>()`却只能传入`Man`类型,`logPersonInfo<Woman>()`却只能传入`Woman`类型。

    interface Person {
        name: string,
        age: number
    }
    class Man implements Person {
        name = 'zhangSan';
        age = 27;
        working() {}
    }
    class Woman implements Person {
        name = 'xiaoLi';
        age = 26;
        shopping() {}
    }
    function logPersonInfo<T extends Person>(t: T) {
        console.log(`name : ${t.name} , age : ${t.age}`)
    }
    logPersonInfo<Person>(new Man())
    logPersonInfo<Person>(new Woman())
    logPersonInfo<Man>(new Man())
    logPersonInfo<Woman>(new Woman())

下面再说一种情况，来更加深入的了解`extends`:

    interface Person {
        name: string,
        age: number
    }
    class Alien {
        name = "E.T"
        age = 1000
        fly() {
        }
    }
    function logPersonInfo<T extends Person>(t: T) {
        console.log(`name : ${t.name} , age : ${t.age}`)
    }
    logPersonInfo<Person>(new Alien())

这个时候我们传入`logPersonInfo<Person>()`的是一个`Alien`，这时候也不会有什么问题，因为`Alien`包含了`Person`的所有属性。对比两个例子我们能看得出来这里的`extends`和类继承的`extends`是不一样的，这个地方的`extends`起到的作用只是限制泛型`T`的类型为包含了`Person`类型的所有属性（**当然所有属性的类型也要一致**）的类型。

## 类型映射（in）

`in`高级类型起到的作用是做类型的映射，它会遍历**已有类型的所有key**或者是**联合类型的所有类型**，有点类似于`forin`。下面我们写的demo，将已有类型的所有属性转换成可空类型。

    interface Person {
        name: string;
        age: number;
    }
    //此时这里的P就是in遍历出来的key
    //将Person的所有key遍历出来设置成?:可空类型，再赋值上key对应的value
    //这时候的PersonValueCanNull类型就是所有属性可为空的Person类型了
    type PersonValueCanNull = { [P in keyof Person]?: Person[P] }

如上代码，这个时候`PersonValueCanNull`的类型为`{name?: string, age?: number}`,如图：

<img src="/images/5125944-5d14c681a6266115.png" width="100%">
<br>

再来个联合类型的例子：

    type ValueKeyType = "key1" | "key2" | "key3"
    type ValueType = { [P in ValueKeyType]: boolean }
    const value: ValueType = {
        key1: false,
        key2: false,
        key3: false,
    }

结合上面联合类型的例子，我们再来看第一个例子，`[P in keyof Person]`就可以拆分成`[P in keyof "name"|"age"]`，其实最终都是使用联合类型来进行`in`操作

## 条件类型（A ? B : C）

`条件类型`其实就是一个三元运算操作，如果满足条件`A`那么就是`B`类型，否则就是`C`类型，话不多说上代码：

    //传入T和U，如果T包含U所有的key，那么返回类型是T，反之返回的是T和U的交叉类型
    type MergeAction<T, U> = T extends U ? T : T & U

    interface Teacher {
        name: string,
        age: number,
        teachStudentNum: number
    }

    interface Father {
        name: string,
        age: number,
        childNum: number
    }

    //回顾一下之前的 extends， Teacher里面没有包含Father里面的
    //所有key，所以返回的是Teacher&Father的交叉类型
    const person: MergeAction<Teacher, Father> = {
        name: 'zl',
        age: 27,
        teachStudentNum: 60,
        childNum: 1
    }

类型如图：

<img src="/images/5125944-b19d136fdd1c9c65.png" width="100%">
<br>

下面我们再来个`keyof`的例子

    const person :MergeAction<keyof Teacher, keyof Father> = "name"

此时的类型就是 `name|age`的`联合类型`了。
<img src="/images/5125944-742a495f249e6f45.png" width="100%">
<br>














