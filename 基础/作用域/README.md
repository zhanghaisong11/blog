# 作用域

我的理解中作用域就是函数可访问变量，对象，函数的集合

作用域规定了如何查找变量，也就是确定当前执行代码对变量的访问权限。  
JavaScript 采用词法作用域(lexical scoping)，也就是静态作用域。


#### 词法作用域：
词法作用域就是定义在词法阶段的作用域。换句话说，词法作用域是由你在写代码时将变量和块作用域写在哪里来决定的，因此当词法分析器处理代码时会保持作用域不变 。

因为 JavaScript 采用的是词法作用域，函数的作用域在函数定义的时候就决定了。

而与词法作用域相对的是动态作用域，函数的作用域是在函数调用的时候才决定的。


```javascript
var value = 1;

function foo() {
    console.log(value);
}

function bar(a,b,c) {
  
    var value = 2;
  console.log(value)
    foo();
}

function  sum (){
  let s = 0
  arguments.forEach(item){
    s = s + item
  }
  return s
}

sum(1)
sum(1 ,2  ,3,4)


bar();

// 结果是 ???

```

### 作用域链

当查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级(词法层面上的父级)执行上下文的变量对象中查找，一直找到全局上下文的变量对象，也就是全局对象。这样由多个执行上下文的变量对象构成的链表就叫做作用域链。

AO：Activive Object，即函数的活动对象。
VO：Variable Object，即变量对象。

### 1.什么是变量对象（Variable Object）
变量对象是与执行上下文对应的概念，在执行上下文的创建阶段，它依次存储着在上下文中定义的以下内容：

#### 1.1 函数的所有形参（如果是函数上下文中）：
建立arguments对象。检查当前上下文中的参数，建立该对象下的属性与属性值。没有实参的话，属性值为undefined。

#### 1.2. 所有函数声明：(FunctionDeclaration, FD)
检查当前上下文的函数声明，也就是使用function关键字声明的函数。在变量对象中以函数名建立一个属性，属性值为指向该函数所在内存地址的引用。如果变量对象已经存在相同名称的属性，则完全替换这个属性。

#### 1.3. 所有变量声明:（var, VariableDeclaration）
检查当前上下文中的变量声明，每找到一个变量声明，就在变量对象中以变量名建立一个属性，属性值为undefined。如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性。


### 2.什么是活动对象？(activation object, AO)

未进入执行阶段前，变量对象中的属性都不能访问！但是进入到执行阶段之后，变量对象转变成了活动对象，里面的属性都能被访问了，然后开始进行执行阶段的操作。

因此，对于函数上下文来讲，活动对象与变量对象其实都是同一个对象，只是处于执行上下文的不同生命周期。不过只有处于执行上下文栈栈顶的函数执行上下文中的变量对象，才会变成活动对象。



### 例子

```javascript
var scope = "global scope";
function checkscope(){
    var scope2 = 'local scope';
    return scope2;
}
checkscope();
```

执行过程如下：

1.checkscope 函数被创建，保存作用域链到 内部属性[[scope]]

```javascript
checkscope.[[scope]] = [
    globalContext.VO
];
```
2.执行 checkscope 函数，创建 checkscope 函数执行上下文，checkscope 函数执行上下文被压入执行上下文栈

```javascript
ECStack = [
    checkscopeContext,
    globalContext
];
```

3.checkscope 函数并不立刻执行，开始做准备工作，第一步：复制函数[[scope]]属性创建作用域链

```javascript
checkscopeContext = {
    Scope: checkscope.[[scope]],
}
```

4.第二步：用 arguments 创建活动对象，随后初始化活动对象，加入形参、函数声明、变量声明

```javascript
checkscopeContext = {
    AO: {
        arguments: {
            length: 0
        },
        scope2: undefined
    },
    Scope: checkscope.[[scope]],
}
```

5.第三步：将活动对象压入 checkscope 作用域链顶端

```javascript
checkscopeContext = {
    AO: {
        arguments: {
            length: 0
        },
        scope2: undefined
    },
    Scope: [AO, [[Scope]]]
}
```

6.准备工作做完，开始执行函数，随着函数的执行，修改 AO 的属性值

```javascript
checkscopeContext = {
    AO: {
        arguments: {
            length: 0
        },
        scope2: 'local scope'
    },
    Scope: [AO, [[Scope]]]
}
```

7.查找到 scope2 的值，返回后函数执行完毕，函数上下文从执行上下文栈中弹出

```javascript
ECStack = [
    globalContext
];
```


###问题

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
//输出结果是？

```
```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();
//输出结果是？

```
