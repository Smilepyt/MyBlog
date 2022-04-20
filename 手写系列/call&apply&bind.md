```js
Function.prototype.myCall = function(context) {
    context.fn = this
    const arg = [...arguments].slice(1)
    const res = context.fn(...arg)
    delete context.fn
    return res
}
let obj = {
    name: 'pyt'
}
function say() {
    console.log(this.name, ...arguments)
}
say()
say.myCall(obj, 1, 2, 3)
```