# Promise

Promise 对象用于表示一个异步操作的最终完成 (或失败)及其结果值。

一个 Promise 必然处于以下几种状态之一：
- 待定（pending）: 初始状态，既没有被兑现，也没有被拒绝。
- 已兑现（fulfilled）: 意味着操作成功完成。
- 已拒绝（rejected）: 意味着操作失败。

```js
class MyPromise {
    static PENDING = 'pending'
    static FULFILED = 'fulfiled'
    static REJECTED = 'rejected'
    constructor(executor) {
        // promise 初始状态
        this.PromiseState = MyPromise.PENDING
        this.PromiseResult = null
        // 利用数组 保存链式调用执行的方法
        this.onFulfiledCallbacks = []
        this.onRejectCallbacks = []
        // try catch 处理 Promise 中有 throw 的情况
        try {
            executor(this.resolve.bind(this), this.reject.bind(this))
        } catch (error) {
            this.reject(error)
        }
    }

    resolve(result) {
        // resolve 只能 PENDING -> FULFILED
        if (this.PromiseState !== MyPromise.PENDING) return
        this.PromiseState = MyPromise.FULFILED
        this.PromiseResult = result
        while (this.onFulfiledCallbacks.length) {
            this.onFulfiledCallbacks.shift()(result)
        }
    }

    reject(reason) {
         // reject 只能 PENDING > REJECTED
        if (this.PromiseState !== MyPromise.PENDING) return
        this.PromiseState = MyPromise.REJECTED
        this.PromiseResult = reason
        while (this.onRejectCallbacks.length) {
            this.onRejectCallbacks.shift()(reason)
        }
    }

    then(onFulfiled, onReject) {
        // 保证参数都是 function
        onFulfiled = typeof onFulfiled === 'function' ? onFulfiled : result => result
        onReject = typeof onReject === 'function' ? onReject : reason => { throw (reason) }

        var thenPromise = new MyPromise((resolve, reject) => {
            const resolvePromise = cb => {
                // Promise.then 微任务，利用 setTimeout 解决
                setTimeout(() => {
                    try {
                        const x = cb(this.PromiseResult)
                        if (x === thenPromise) {
                            throw new Error('回调不能是本身')
                        } else if (x instanceof MyPromise) {
                            x.then(resolve, reject)
                        } else {
                            resolve(this.PromiseResult)
                        }
                    } catch (error) {
                        reject(error)
                        throw new Error(error)
                    }
                })
            }

            if (this.PromiseState === MyPromise.PENDING) {
                this.onFulfiledCallbacks.push(resolvePromise.bind(this, onFulfiled))
                this.onRejectCallbacks.push(resolvePromise.bind(this, onReject))
            }
            if (this.PromiseState === MyPromise.FULFILED) {
                resolvePromise(onFulfiled)
            }
            if (this.PromiseState === MyPromise.REJECTED) {
                resolvePromise(onReject)
            }
        })

        return thenPromise
    }

}

const test1 = new MyPromise((resolve, reject) => {
    resolve('成功')
})
console.log(test1)
// MyPromise {PromiseState: 'fulfiled', PromiseResult: '成功', onFulfiledCallbacks: Array(0), onRejectCallbacks: Array(0)}

const test2 = new MyPromise((resolve, reject) => {
    reject('失败')
})
console.log(test2)
// MyPromise {PromiseState: 'rejected', PromiseResult: '失败', onFulfiledCallbacks: Array(0), onRejectCallbacks: Array(0)}

const test3 = new MyPromise((resolve, reject) => {
    resolve('成功')
    reject('失败')
})
console.log(test3)
// MyPromise {PromiseState: 'fulfiled', PromiseResult: '成功', onFulfiledCallbacks: Array(0), onRejectCallbacks: Array(0)}

const test4 = new MyPromise((resolve, reject) => {
    throw ('失败')
})
console.log(test4)
// MyPromise {PromiseState: 'rejected', PromiseResult: '失败', onFulfiledCallbacks: Array(0), onRejectCallbacks: Array(0)}

const test5 = new MyPromise((resolve, reject) => {
    resolve('成功')
}).then(res => console.log(res), err => console.log(err))
// 成功

const test6 = new MyPromise((resolve, reject) => {
    setTimeout(() => {
        resolve('成功') 
    }, 1000)
}).then(res => console.log(res), err => console.log(err))
// 1秒后输出 成功

const test7 = new MyPromise((resolve, reject) => {
    resolve(100) // 成功 200
    // reject(300) // 成功 300
}).then(res => 2 * res, err => 3 * err).then(res => console.log('成功', res), err => console.log('失败', err))
const test8 = new MyPromise((resolve, reject) => {
    resolve(100) // 失败 200
    // reject(100) // 成功 300
}).then(res => new MyPromise((resolve, reject) => reject(2 * res)), err => new Promise((resolve, reject) => resolve(3 * err))).then(res => console.log('成功', res), err => console.log('失败', err))
```

- 参考文章
    - [看了就会，手写Promise原理，最通俗易懂的版本！！！](https://juejin.cn/post/6994594642280857630#heading-9) 
    - [手把手一行一行代码教你“手写Promise“](https://juejin.cn/post/7043758954496655397#heading-12) 