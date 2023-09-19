# ts漫游指南

## 一、类型声明

```Vue
let foo:string

function toString(num:number):string{
    return String(num)
}

//报错
let foo:string = 123;

let x:number;
console.log(x) //报错 变量只有赋值后才能使用

unknown 类型
    和any类似，所有类型的值都可以分配给unknown类型
    unknown类型的变量，不能直接赋值给其他类型的变量，只能赋值给any类型和unknown类型
    let v:unknown = 123;
    let v1:number = v; //报错

    不能直接调用unknown类型变量的方法和属性
    let v1:unknown = { foo:123 }
    v1.foo //报错
    let v2:unknown = 'hello'
    v2.trim() //报错
    let v3:unknown = (n = 0) => n + 1
    v3() //报错

    那如何使用unknown的变量呢 只有经过类型缩小才可以使用
    let a:unknown = 1;
    if (typeof a === 'number') {
        let r = a + 10; // 正确
    }
```
