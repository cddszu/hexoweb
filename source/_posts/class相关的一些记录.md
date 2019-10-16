---
title: class相关的一些记录
---
#### 取值函数（getter）和存值函数（setter）
与 ES5 一样, 在“类”的内部可以使用get和set关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。
```
class MyClass {
  constructor() {
    // ...
  }
  get prop() {
    return 'getter';
  }
  set prop(value) {
    console.log('setter: '+value);
  }
}

let inst = new MyClass();

inst.prop = 123;
// setter: 123

inst.prop
// 'getter'
```

存值函数和取值函数是设置在属性的 Descriptor 对象上的。

```
class CustomHTMLElement {
  constructor(element) {
    this.element = element;
  }

  get html() {
    return this.element.innerHTML;
  }

  set html(value) {
    this.element.innerHTML = value;
  }
}

var descriptor = Object.getOwnPropertyDescriptor(
  CustomHTMLElement.prototype, "html"
);

"get" in descriptor  // true
"set" in descriptor  // true
```

#### this指向
类的方法内部如果含有this，它默认指向类的实例。但是，必须非常小心，一旦单独使用该方法，很可能报错。
```
class Logger {
  printName(name = 'there') {
    this.print(`Hello ${name}`);
  }

  print(text) {
    console.log(text);
  }
}

const logger = new Logger();
const { printName } = logger;
printName(); 
```

#### 静态属性（提案）
静态属性指的是 Class 本身的属性，即Class.propName，而不是定义在实例对象（this）上的属性。
```
class MyClass {
  static myStaticProp = 42;

  constructor() {
    console.log(MyClass.myStaticProp); // 42
  }
}
```

#### 私有属性或者方法（提案）
私有属性和私有方法都是在命名前面加 `#`
```
class IncreasingCounter {
  #count = 0;
  get value() {
    console.log('Getting the current value!');
    return this.#count;
  }
  increment() {
    this.#count++;
  }
}
const counter = new IncreasingCounter();
counter.#count // 报错
counter.#count = 42 // 报错
```

#### 继承
子类的构造器中，必须执行`super`函数，并且`super`不能在其他类方法中执行
```
class B extends A {
    constructor() {
        super() // 缺少会报错
    }
    
    fun () {
        super() // 在类方法中也会报错
    }
}
```