# typescript

[官网](https://www.tslang.cn/)


### 1）、介绍

​        TypeScript 是 JavaScript 的一个超集，主要提供了**类型系统**和对 ES6+ 的支持，让javascript变成**强类型**语言。它由 Microsoft 开发，代码开源于 GitHub 上。



### 2）、特点

- 可以在开发阶段和编译阶段就发现大部分错误，这总比在运行时候出错好
- 不显式的定义类型，也能够自动做出类型推论
- 即使 TypeScript 编译报错，也可以生成 JavaScript 文件
- Google 开发的 Angular 就是使用 TypeScript 编写的
- TypeScript 拥抱了 ES6+ 规范



### 3）、安装

```js
npm install -g typescript  \  yarn global add typescript
tsc -v 测试  

tsc的c就是compile（编译）：把ts代码编译成js代码
```



### 4）、编译

#### 1、编译指定的文件

编译所完成的事情，就是把ts的代码转换成js的代码。因为浏览器或者node环境最终识别的是js，而不是ts。

```js
 tsc test.ts
//监听
 tsc test.ts --watch
```

> 类型不匹配时，编辑报错，但可以生成js（编辑通过），如果不希望编译通过需要配tsconfig.json



使用步骤：

```
1、新建test.ts
    随便写上一些代码，js的代码就行（因为ts完全支持js）
2、使用命令：tsc  test.ts 把ts编译成js
    在当前目录下就会产生一个test.js文件。
3、新建html文件，引入test.js文件，就ok了。
```

> 编辑器推荐使用vscode，因为，vscode对ts的提示非常好。



2、使用tsconfig.json文件配置编译选项

```js
 1、tsc --init  初始化配置，tsconfig.json文件
 2、创建ts文件夹和js文件夹
 3、打开配置文件tsconfig.json

  "target": "es5",    将TS转换为JS的那个版本
  "module": "es6",   模块类型
  "rootDir": "./ts",    根路径，要转换的ts路径
   "outDir": "./js",   目标输出路径，将ts转换为js到什么路径
   "sourceMap": true,   源映射，可以通过这个映射找到ts对应的位置
   "declaration": true,  针对代码的提示描述功能（声明）
   "declarationMap": true,  描述映射位置（声明）

  4、tsc -w ：监听即可。
```



## 1、数据类型

### 1）、ts中的类型：

#### ts中的原始类型

​    1）、 js里本来就有的(基本)类型：number、string、boolean、null、undefined、Symbol

​    2）、ts里新增的类型：**void、any、联合类型、函数、数组类型、类 ....**

#### 内置对象类型

##### DOM

​        Document、HTMLElement、NodeList ...

#### 内置类

​       Boolean、Error、Date、RegExp、Math，Array，XMLHttpRequest ... 







### 2）、ts类型的解释:



#### 布尔值

最基本的数据类型就是简单的true/false值，在JavaScript和TypeScript里叫做`boolean`（其它语言中也一样）。

```ts
let isDone: boolean = false;
```

#### 数字

和JavaScript一样，TypeScript里的所有数字都是浮点数。 这些浮点数的类型是 `number`。 除了支持十进制和十六进制字面量，TypeScript还支持ECMAScript 2015中引入的二进制和八进制字面量。

```ts
let decLiteral: number = 6;
let hexLiteral: number = 0xf00d;
let binaryLiteral: number = 0b1010;
let octalLiteral: number = 0o744;
```

#### 字符串

JavaScript程序的另一项基本操作是处理网页或服务器端的文本数据。 像其它语言里一样，我们使用 `string`表示文本数据类型。 和JavaScript一样，可以使用双引号（ `"`）或单引号（`'`）表示字符串。

```ts
let name: string = "bob";
name = "smith";
```

你还可以使用*模版字符串*，它可以定义多行文本和内嵌表达式。 这种字符串是被反引号包围（ ```），并且以`${ expr }`这种形式嵌入表达式

```ts
let name: string = `Gene`;
let age: number = 37;
let sentence: string = `Hello, my name is ${ name }.

I'll be ${ age + 1 } years old next month.`;
```

这与下面定义`sentence`的方式效果相同：

```ts
let sentence: string = "Hello, my name is " + name + ".\n\n" +
    "I'll be " + (age + 1) + " years old next month.";
```

#### 数组

TypeScript像JavaScript一样可以操作数组元素。 有两种方式可以定义数组。

第一种，可以在**元素类型**后面接上 `[]`，表示由此类型元素组成的一个数组：

```ts
let list: number[] = [1, 2, 3];
```

第二种方式是使用**数组泛型**，`Array<元素类型>`：

```ts
let list: Array<number> = [1, 2, 3];
```



#### 联合类型

一个变量的取值可以是多种类型。

```js
如：
let temp: string | number; //表示temp变量的取值可以是string，也可以是number

temp = 12;//没有问题
temp = "hi";//没有问题
temp = false;//不行。

```



#### any

有时候，我们会想要为那些在编程阶段还不清楚类型的变量指定一个类型。 这些值可能来自于动态的内容，比如来自用户输入或第三方代码库。 这种情况下，我们不希望类型检查器对这些值进行检查而是直接让它们通过编译阶段的检查。 那么我们可以使用 `any`类型来标记这些变量：

```ts
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

在对现有代码进行改写的时候，`any`类型是十分有用的，它允许你在编译时可选择地包含或移除类型检查。 你可能认为 `Object`有相似的作用，就像它在其它语言中那样。 但是 `Object`类型的变量只是允许你给它赋任意值 - 但是却不能够在它上面调用任意的方法，即便它真的有这些方法：

```ts
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
```

当你只知道一部分数据的类型时，`any`类型也是有用的。 比如，你有一个数组，它包含了不同的类型的数据：

```ts
let list: any[] = [1, true, "free"];

list[1] = 100;
```

#### void

某种程度上来说，`void`类型像是与`any`类型相反，它表示没有任何类型。 当一个函数没有返回值时，你通常会见到其返回值类型是 `void`：

```ts
function warnUser(): void {
    console.log("This is my warning message");
}
```

声明一个`void`类型的变量没有什么大用，因为你只能为它赋予`undefined`和`null`：

```ts
let unusable: void = undefined;
```



#### unknown

any 与 unknown 的共同点是，都可以将任何其他类型的值赋值给 any 类型或者 unknown 类型的变量。比如：

```js
const n:number = 1;
const n1: any = n;
const n2: unknown = n;

```


​         any 类型和 unknown 类型的区别是：

​         变量 n1 具有 any 类型，它表示 n1 可以是任何类型，可以对 n1 做任何操作；即使 n1 是 number 类型，n1.push 的调用依然是合法的，这无形中失去了 Typescript 给我们带来的类型安全性。

​          而 n2 具有 unknown 类型，它表示 n2 可以是任何类型，但是 n2 不支持任何操作。 所以在某些场合中，使用 unknown 类型，可以限制某些变量的行为，使我们的编码习惯更加谨慎。



#### 类型推论

定义变量时，没有明确的指定数据类型，ts会依照**值**推断出一个类型。

```ts
let a = "hi"; //虽然没有明确类型，但是ts推论出a变量是字符串类型
a=200; //这句话不能通过

```

##### 基础

TypeScript里，在有些没有明确指出类型的地方，类型推论会帮助提供类型。如下面的例子

```ts
let x = 3;
```

变量`x`的类型被推断为数字。 这种推断发生在初始化变量和成员，设置默认参数值和决定函数返回值时。

大多数情况下，类型推论是直截了当地。 后面的小节，我们会浏览类型推论时的细微差别。

##### 最佳通用类型

当需要从几个表达式中推断类型时候，会使用这些表达式的类型来推断出一个最合适的通用类型。例如，

```ts
let x = [0, 1, null];
```

为了推断`x`的类型，我们必须考虑所有元素的类型。 这里有两种选择： `number`和`null`。 计算通用类型算法会考虑所有的候选类型，并给出一个兼容所有候选类型的类型。

由于最终的通用类型取自候选类型，有些时候候选类型共享相同的通用类型，但是却没有一个类型能做为所有候选类型的类型。例如：

```ts
let zoo = [new Rhino(), new Elephant(), new Snake()];
```

这里，我们想让zoo被推断为`Animal[]`类型，但是这个数组里没有对象是`Animal`类型的，因此不能推断出这个结果。 为了更正，当候选类型不能使用的时候我们需要明确的指出类型：

```ts
let zoo: Animal[] = [new Rhino(), new Elephant(), new Snake()];
```

如果没有找到最佳通用类型的话，类型推断的结果为联合数组类型，`(Rhino | Elephant | Snake)[]`。

##### 上下文类型

TypeScript类型推论也可能按照相反的方向进行。 这被叫做“按上下文归类”。按上下文归类会发生在表达式的类型与所处的位置相关时。比如：

```ts
window.onmousedown = function(mouseEvent) {
    console.log(mouseEvent.button);  //<- Error
};
```

这个例子会得到一个类型错误，TypeScript类型检查器使用`Window.onmousedown`函数的类型来推断右边函数表达式的类型。 因此，就能推断出 `mouseEvent`参数的类型了。 如果函数表达式不是在上下文类型的位置， `mouseEvent`参数的类型需要指定为`any`，这样也不会报错了。

如果上下文类型表达式包含了明确的类型信息，上下文的类型被忽略。 重写上面的例子：

```ts
window.onmousedown = function(mouseEvent: any) {
    console.log(mouseEvent.button);  //<- Now, no error is given
};
```

这个函数表达式有明确的参数类型注解，上下文类型被忽略。 这样的话就不报错了，因为这里不会使用到上下文类型。

上下文归类会在很多情况下使用到。 通常包含函数的参数，赋值表达式的右边，类型断言，对象成员和数组字面量和返回值语句。 上下文类型也会做为最佳通用类型的候选类型。比如：

```ts
function createZoo(): Animal[] {
    return [new Rhino(), new Elephant(), new Snake()];
}
```

这个例子里，最佳通用类型有4个候选者：`Animal`，`Rhino`，`Elephant`和`Snake`。 当然， `Animal`会被做为最佳通用类型。





## 2、ts变量作用域

### 全局作用域(项目)  **默认**

​      在一个项目目录下，所有文件的变量都暴露在全局

### 模块作用域(文件)

​        变量封闭在模块内部，在内部全局使用。所以，只有 在文件内部使用了export（export default）就会让全局变量变成模块作用域。

如：

```ts
let temp:number; //定义变量，作用域范围是当前文件（模块）
temp = 12;

export {};

```



### 函数作用域，块级作用域等等



## 3、函数



对函数的限定，能够限定形参的类型，返回值的类型。

形参：输入

返回值：输出

即：函数的限定就是限定它的输入和输出

### 1）、声明式

```js
//注意：下面的代码是定义函数
//function 函数名(参数:类型):返回类型{函数体}
function sum(x:number, y:string):number {
    return 123;
}
```



### 2）、表达式

```jsx
//一、定义和赋值分开：

//注意：下面的代码是定义函数类型
//定义函数类型，不赋值
//let 变量:输入类型 => 输出类型 

// 1)、定义一个变量show3，show3是函数类型
let show3:(m:number,n:string,a?:string)=>number;

// 2)、给变量show赋值（相当于定义函数）
show3 = function(t:number,s:string){
    return t;
}
// 3)、调用函数
let b = show3(12,"23","12");


//二、定义的同时赋值
//let 变量:输入类型 => 输出类型 = function(参数){}
let show3:(a:number,b:string) => number = function (a,b){
  return a;
}


```



### 3）、接口定义函数类型

```js
//1、定义函数类型（自定义函数类型）
interface Func {
    (a:number,b:string):boolean
}

//2、定义函数变量，限定类型为Func。并且给函数赋值
let show5:Func = function(a:number,b:string){
    return true;
}

show5(12,"hi");

// void类型修饰函数返回值时，表示函数不能有任何返回值
function fn():void{
    return "hi"; //出错
}
```



### 4）、可选参数

```js
//函数表达式
//定义的同时赋值
//let 变量:输入类型 => 输出类型 = function(参数){}
let show3:(a:number,b:string,c?) => number = function (a,b){
  return a;
}
```



> 
>
> 可选参数必须写在参数列表的最后
>
> 



### 5）、参数默认值

```js
function fn(a:number,b:number=2):void{
    console.log(a,b);
}

fn(12,23);

fn(12)
```



### 6）、剩余参数

```js
function fn02(a:number,...b):void{
    console.log(a,b);
}

fn02(12,23,34,45,56);

fn02(12)
```







## 4、类型断言（断定）

绕过编译器的类型推断，手动指定一个值的类型

**格式：**
1）、<类型>变量 
2）、变量 as 类型

```ts
(<string>a).length   //断言字符串
(a as string).length //断言字符串

如：
function fn(a:string|number):boolean{    
    
    //a.length; 这么写是错的，因为，a的取值有两种可能性，length没法直接使用，会报错
    // 使用断言，就不会报错
    
    let n:number = (<string>a).length;  //当a是string时，使用length属性。如果是number，忽略掉这句话（没有给n赋值）。那么，n是undefined，
    
    console.log(n);
    return <boolean>0;//这是不对的，因为，类型断言不做类型转换
}

fn("hello"); //5
fn(120);//undefined

```

**类型断言的注意点：**

1）、类型断言不做类型转换 
2）、自定义类型也可以用来做断言
3）、权重最低





## 5、自定义类型

对象的类型使用class或者**接口**（interface）来描述，也可以使用type定义。class我们已经学过，接口如何使用

接口的定义，就是为了描述对象。

**1）、接口：interface**

```js
//定义类型（自定义类型）
//格式
interface 接口名{
    键:类型。
}

//还记得nodejs中的那个schema吗。

interface IPerson{
    name:string,//必须的属性值
    readonly sex:string|number, //只读,联合类型
    age?:number//可选的属性值,
    eat:(food:string)=>void
}

let p1:IPerson={
    name:"张三疯",
    sex:"男",
    // wife:"梅超风" //不行，我们定义的p1类型IPerson里没有wife
    eat:function(food:string):void{
        console.log("吃"+food);        
    }
};

p1.name="张四疯";
// p1.sex="女";//不行，不能改
p1.age = 12; //没有问题
// p1.wife ="梅超风"; //这句话在编译时会报错，因为，我们定义的p1类型IPerson里没有wife

p1.eat("油条");

console.log(p1);

       

```

**2、type**

```js
type IPerson={
    name:string,
    sex:string|number,
    age?:number
}

```



## 6、类(自定义类型)



在TS中，不支持es6类的写法。支持es7类的写法。

```ts
//es6：给类声明实例属性时，直接写在构造函数里
class Person{
    constructor(){
        //声明了一个属性，并且赋值
        this.name = "张三疯";//这不行了，这是es6的写法
    }
}

//es7：给类声明属性，不能写在构造函数里，需要在类（类名对应的一对花括号）里写。
class Person{
    //声明一个属性name，并且赋初值。
    name = "张三疯";//这可以，es7的写法
    
    constructor(){
        this.name="张四疯";//这句话仅仅只是赋值，不是声明。
    }    
    
}

```

es7类的写法：

```ts
class Person{
    //声明实例属性
    name1="张三疯"; //定义属性name，值为 张三疯
    age:number;//age属性没有赋值

    //实例方法
    show(){
        console.log(this.name1,this.age);
    }
    
    //构造函数
    constructor(name,age){
        //赋值
        this.name1 = name;
        this.age = age;
    }

    //静态属性(类的属性)
    static count=0;
    
    //静态方法（类的方法）
    static max(){
        return 110;
    }
    
}


let p = new Person("张三疯","男");
p.show();

console.log(Person.count);
console.log(Person.max());


```



**ES7类的访问控制符**

控制类内部的属性(实例，类)|方法(实例，类) 的被访问权限

```js
//public   公共的，谁都可以访问(类内、外)，默认就是public
//private  私有的，当前类内使用
//protected  受保护的，当前类内部+子类内部可以使用。

class Person {

  public namee:string='张三丰';//实例属性
  private age:number; //私有的，只能当前类内部使用
  protected address:string; //受保护的，当前类内部及其子类内部
  address2:string; //不加的情况下 是public

  protected static VER:string='1.11.1' //类属性 静态属性
	
	constructor(namee:string,age:number){
	   	this.namee = namee;//实例属性第一次的修改
    	this.age = age||0;//实例属性第一次的修改
  }

  private show(){}   //私有的实例方法
  private static show2(){} //私有的类方法（静态方法）
}

```



```js
class Box{
    
    private _checked:boolean=false;

    public set checked(value:boolean)//不能写:void
    {
        this._checked=value;
    }

    public get checked():boolean
    {
        return this._checked;
    }
    
}


```



## 7、泛型



在定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型的一种特性。

其实就是，把**类型也当作变量**（类型是可变的，未知的），在使用时，把类型变量放在<> 里，表示类型的声明。

### **1）、函数泛型**

```ts
//定义 function 函数名<类型变量声明>(length: number, value: 类型变量): Array<类型变量> {}

//如： function 函数<T>(length: number, value: T): Array<T> {}；其中T就是类型的变量名

//调用 函数<string>(参数) //给变量T赋值为string。

function fn<T>(a1:T,a2:T):Array<T>{
    return [a1,a2];
}

console.log(fn<number>(12,23));

console.log(fn<string>("hello","world"));

console.log(fn<number>("hello","world")); //这个不行，因为，要求T 是number类型，而传入的参数"hello"是string类型。

```

> 补充理解函数的泛型
>
> 如：
>
> function fn<T>(){
>
> }
>
> 你可以认为，函数的名字不是fn，而是 fn<T>。其中T是个变量。
>
> 调用时：
>
> fn<string>();    // string就传给T。



### **2）、类泛型**

```js
//定了了Person类，并声明了两个类型T和U。
// 二、类里面使用泛型
// 1、定义一个类型，使用泛型
//   定义类时，声明了两个类型变量，分别是T和P
class Person<T,P>{
    public a:T;
    public b:P;

    constructor(aaaa:T,b:P){
        this.a = aaaa;
        this.b = b;
    }
}

// 2、使用类（实例化对象）时，传入类型变量。
let p1 =new Person<number,string>(12,"");

let p2 = new Person<boolean,number>(true,15);


interface IBook{
    name:string,
    author:string,
    price:number
}

let b1:IBook ={
    name:"西游记",
    author:"张杰",
    price:12
}

let p3 = new Person<IBook,number>(b1,12);

```

 	

[**ts的完整内容的视频**](https://www.bilibili.com/video/BV1H44y157gq/?spm_id_from=333.337.search-card.all.click&vd_source=7914db3d40d8703cd557090c9c3bb770)：陆总监录制



