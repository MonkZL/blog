---
title: TypeScript自带的工具泛型
date: 2022-11-20 11:42:42
categories: TypeScript
tags: Advanced Type
---

# 前言

前面总结了`ts`的高级类型，下面再来说说`ts`给我们提供的一些基于这些高级类型而成的工具泛型。

## Partial

    /**
     * Make all properties in T optional
     * 将T中的所有属性设置为可选
     */
    type Partial<T> = {
        [P in keyof T]?: T[P];
    };

官方提供的这个方法，就是将`T`中的所有属性设置为可选类型。可以看到它使用了`in`和`keyof T`来遍历`T`中的所有`key`（此时`P`就代表`key`），可以看到它使用了`?:`来将类型改为了可空类型，最后再通过`T[P]`取值的形式,拿到`key`对应的属性，最后就生成了一个所有`key`都和`T`一样，但都是可空类型的新`T`类型。下面我们看看效果：

<img src="/images/5125944-397285fdc655de54.png" width="100%">
<br>

可以看到此处的类型已经发生变化。

## Required

    /**
     * Make all properties in T required
     * 使T中的所有属性都是必需的
     */
    type Required<T> = {
        [P in keyof T]-?: T[P];
    };

`Required`和`Partial`是相反的，它的作用是`使T中的所有属性都是必需的`，`[P in keyof T]-?: T[P]`里面的`-?`可以看做是取消掉`?`的意思，直接贴效果：

<img src="/images/5125944-7de6dcb0d9577306.png" width="100%">
<br>

可以看到，所有可选类型都变成必选了。

## Readonly

    /**
     * Make all properties in T readonly
     * 将T中的所有属性设置为只读
     */
    type Readonly<T> = {
        readonly [P in keyof T]: T[P];
    };

顾名思义，`Readonly`的作用就是将`T`中的所有元素设置为只读类型。

<img src="/images/5125944-29db75e060ecad47.png" width="100%">
<br>

我们可以看的类型`age`和`name`已经变成只读类型了。

## Pick

    /**
     * From T, pick a set of properties whose keys are in the union K
     * 从T中，选择一组键位于联合K中的属性
     */
    type Pick<T, K extends keyof T> = {
        [P in K]: T[P];
    };

根据 `K extends keyof T` 可以得知 `K` 是单个类型都位于 `T` 中的联合类型，首先通过 `[P in K]` 取出联合类型 `K` 中所有的类型，再将其和 `T[P]` 的值关联起来，最后返回了一个新的 `interface` 类型，这个类型里面只包含了联合类型 `K` 的所有类型。

    interface Person {
        name: string,
        age: number,
        sex: string
    }

    type nameAndAge = "name" | "age"

    const nameAndAgePerson: Pick<Person, nameAndAge> = {
        name: "zl",
        age: 27
    }

如上示例，因为 `K(nameAndAge)` 的类型为 `name|age` ，所以通过 `Pick<Person,nameAndAge>` 得到的类型是 `{ name:string , age:number }` 。

## Record

    /**
     * Construct a type with a set of properties K of type T
     * 将联合类型K的所有值作为Key，T作为类型，生成一个新的类型
     */
    type Record<K extends keyof any, T> = {
        [P in K]: T;
    };

同样的根据 `K extends keyof any` 我们可以知道 `K` 是个联合类型，通过 `[P in K]` 将联合类型的所有值都取了出来，再将其和 `T` 关联起来，最后返回了一个新的 `interface` 类型，这个类型里面的所有属性的类型都是 `T` 类型。

    interface Person {
        name: string,
        age: number,
        sex: string
    }

    type TeacherAndWorker = "teacher" | "worker"

    const person: Record<TeacherAndWorker, Person> = {
        teacher: { name: 'zl', age: 27, sex: "M" },
        worker: { name: 'zl', age: 27, sex: "M" }
    }

如上示例可以看出，`Record` 的作用就是将 `K(TeacherAndWorker)` 的所有子类型作为 `新interface` 子属性的 `key` ，将 `T(Person)` 作为 `新interface` 所有子属性的值。

## Exclude

    /**
     * Exclude from T those types that are assignable to U
     * 从T中排除可分配给U的类型
     */
    type Exclude<T, U> = T extends U ? never : T;

`Exclude` 最常用的还是结合两个联合类型来使用的，我们能通过 `Exclude` 取出 `T` 联合类型在 `U` 联合类型中没有的子类型。

    interface Person {
        name: string,
        age: number,
        sex: string
    }

    interface Alien {
        name: string,
        age: number
    }

    const personKeysExcludeAlienKeys: Exclude<keyof Person, keyof Alien> = "sex"

如上所示，`T(keyof Person = name|age|sex)` 联合类型在 `U(keyof Alien = name|age)` 联合类型中没有的子类型只有 `sex`，所以通过 `Exclude<keyof Person, keyof Alien>` 返回的类型就是 `sex`。下面我们再来看看 `interface` 之间的 `Exclude`。

    const alien: Exclude<Alien, Person> = {
        name: "zl",
        age: 27
    }    

可以看到，通过 `Exclude<Alien, Person>` 我们得到的类型就是 `Alien` 类型，因为 `T(Alien) extends U(Person)` 为 `false` ，所以返回 `T(Alien)`。这种方式我是想不到什么使用场景。

## Extract

    /**
     * Extract from T those types that are assignable to U
     * 从T中提取可分配给U的类型
     */
    type Extract<T, U> = T extends U ? T : never;

`Extract` 和 `Exclude` 是相反的，最常用的还是结合两个联合类型来使用的，我们能通过 `Extract` 取出 `T` 联合类型在 `U` 联合类型中所有重复的子类型。

<img src="/images/5125944-fc2ef49fbd1c8bfe.png" width="100%">
<br>

如图所示，此时通过 `Extract<keyof Person, keyof Alien>` 返回的是联合类型 `name|age`。

## Omit

    /**
     * Construct a type with the properties of T except for those in type K.
     * 构造一个属性为T的类型，但类型K中的属性除外。
     */
    type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;

`Omit` 就是 `Pick` 和 `Exclude` 的组合，首先我们可以通过 `K extends keyof any` 得知 `K` 为联合类型，然后通过 `Exclude<keyof T, K>` 我们取出了在联合类型 `keyof T` 中有，而在 `K` 中没有的子属性(这里我们假设是 `联合类型M` )，最后再通过 `Pick<T,M>` 得到一个新的 `interface` 类型，这个类型里面只包含了联合类型 `M` 的所有类型。***简单的讲 `Omit` 就是从类型 `T` 中取出在 `联合类型K` 中所没有的类型。***

    interface Person {
        name: string,
        age: number,
        sex: string
    }

    const person: Omit<Person, "name" | "age"> = { sex: "M" };

如上示例，我们取出了在 `T(Person)` 里面除去 `K("name"|"age")` 以外的类型 `sex` ，并组成了一个新的 `interface`。

## NonNullable

    /**
     * Exclude null and undefined from T
     *  从T中剔除null和undefined
     */
    type NonNullable<T> = T extends null | undefined ? never : T;

就是字面意思，去除掉联合类型中的 `null` 和 `undefined` 类型。如下图所示，

<img src="/images/5125944-be1bc2dc54865aaa.png" width="100%">
<br>

使用 `NonNullable` 之后去掉了联合类型 `null | undefined | number | string` 中的 `null` 和 `undefined` 类型，剩下了 `number|string`。

## Parameters

     /**
     * Obtain the parameters of a function type in a tuple
     * 获取元组中函数类型的参数
     */
    type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;

根据 `<T extends (...args: any) => any>` 我们能够知道 `T` 是个函数类型，再通过 `T extends (...args: infer P) => any` 将 `T` 的形参列表约束成了泛型 `P` ，所以通过三元运算返回的是函数类型 `T` 的形参元组。

    function fn(str: string, num: number) {}
    type fnParamtersTuple = Parameters<typeof fn>

<img src="/images/5125944-4d0cac21ce6fafd5.png" width="100%">
<br>

如图所示，`fnParamtersTuple` 的类型为，`fn` 的形参类型的元组。

## ConstructorParameters

    /**
     * Obtain the parameters of a constructor function type in a tuple
     * 获取元组中构造函数类型的参数
     */
    type ConstructorParameters<T extends abstract new (...args: any) => any> = T extends abstract new (...args: infer P) => any ? P : never;

和 `Parameters` 类似，只不过 `ConstructorParameters` 获取的是构造函数的形参元组。下面以官网上介绍 `interface类静态部分与实例部分的区别` 中的例子改变一下作为demo。

    interface ClockConstructor {
        new(hour: number, minute: number): ClockInterface;
    }
    interface ClockInterface {
        tick();
    }

    //通过 ConstructorParameters<ClockConstructor> 获取构造函数中的形参元组
    function createClock(ctor: ClockConstructor, ...arg: ConstructorParameters<ClockConstructor>): ClockInterface {
        return new ctor(...arg);
    }

    class DigitalClock implements ClockInterface {
        constructor(h: number, m: number) { }
        tick() {
            console.log("beep beep");
        }
    }

    let digital = createClock(DigitalClock, 12, 17);

## ReturnType

    /**
     * Obtain the return type of a function type
     * 获取函数类型的返回类型
     */
    type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;

和 `Parameters` 类似，只不过 `ReturnType` 获取的是函数类型的返回类型。

     function fn() : string {
         return ""
     }
     type fnReturnType = ReturnType<typeof fn>

<img src="/images/5125944-83e80e5357848ba4.png" width="100%">
<br>

## InstanceType

    /**
     * Obtain the return type of a constructor function type
     */
    type InstanceType<T extends abstract new (...args: any) => any> = T extends abstract new (...args: any) => infer R ? R : any;

和 `Parameters` 类似，只不过 `ConstructorParameters` 获取的是构造函数返回值的类型。

    class Person { }

    class Alien { }

    type p0 = InstanceType<typeof Person> // Person

    interface PersonConstructor {
        new(): Person
    }

    type p1 = InstanceType<PersonConstructor> // Person

    interface PersonAndAlienConstructor {
        new(): Person | Alien
    }

    type p2 = InstanceType<PersonAndAlienConstructor> // Person | Alien
