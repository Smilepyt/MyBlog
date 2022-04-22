## call实现
- fn.apply(obj,arg1, arg2, argN)
```js
Function.prototype.myCall = function(context) {
    // 将函数挂载到传入的对象上
    context.fn = this || windows
    // 获取剩余参数
    const arg = [...arguments].slice(1)
    // 执行对象的方法，得到返回值
    const res = context.fn(...arg)
    // 删除对象的方法
    delete context.fn
    // 返回
    return res
}
let obj = {
    name: 'pyt'
}
function say() {
    console.log(this.name, ...arguments)
}
say()
say.myCall(obj, 1, 2, 3) // pyt 1 2 3
```
## apply实现
- fn.apply(obj,数组参数）
```js
Function.prototype.myApply = function (context) {
    context.fn = this || window
    const arg = [...arguments].slice(1)
    if (!Array.isArray(arg)) {
        throw new Error('arg must be array')
    }
    const ans = context.fn(arg)
    delete context.fn
    return ans
}
let obj = {
    name: 'pyt'
}
function say() {
    console.log(this.name, ...arguments)
}
say()
say.myApply(obj, [3, 2, 1]) // pyt [3,2,1]
```
## bind实现
- fn.bind(obj, arg1, arg2, argN)

```js
Function.prototype.myBind = function (context) {
    const self = this
    const args = [...arguments].slice(1)
    function F() {
        return self.apply(this instanceof F ? this : context, args.concat(...arguments))

    }
    F.prototype = Object.create(self.prototype)
    return F
}
// test
function foo(name) {
    this.name = name;
}
let obj = {};
let bar = foo.myBind(obj);
bar('Jack');
console.log(obj.name);  // Jack
let alice = new bar('Alice');
console.log(obj.name);  // Jack
console.log(alice.name);    // Alice
```

- 关键点1：new 操作符修改 this 指向的优先级更高
```js 
function F() {
    return self.apply(this instanceof F ? this : context, args.concat(...arguments))
}
```

- 关键点2：每次绑定的时候，将绑定函数的原型指向原函数的原型，如果 new 调用绑定函数，得到的实例的原型，也是原函数的原型。这样在 new 执行过程中，执行绑定函数的时候对 this 的判断就可以判断出是否是 new 操作符调用
```js 
F.prototype = Object.create(self.prototype)
```

- 关键点3：当绑定函数直接连接原函数的原型的时候，如果绑定函数的原型有修改时，原函数的原型也会受到影响。所以，为了解决这个问题，我们需要一个空函数，作为中间人。
```js
// F.prototype = self.prototype   // 错误

// 方法1
F.prototype = Object.create(self.prototype)

// 方法2
function X() {}
if(this.prototype) {
    X.prototype = this.prototype
}
F.prototype = new X()
```
- 参考
    - [bind 函数的实现原理](https://zhuanlan.zhihu.com/p/38968174)