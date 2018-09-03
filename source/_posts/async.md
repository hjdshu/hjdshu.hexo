---
title: 理解 Promise，Async, awit
date: 2018-08-07 17:48:25
---

### 1.Promise
#### 1.1 promise的基本用法
``` javascript
const task1 = (start) => {
  return new Promise((resolve, reject) => {
    let A = start
    setTimeout(() => {
      A = A + 1
      if (A) {
          resolve(A)
      } else {
          reject('task1错误')
      }
    }, 1000)
  })
}

const task2 = (result) => {
  return new Promise((resolve, reject) => {
    let B = result
    setTimeout(() => {
      B = result + 1
      if (B > 2) {
          resolve(B)
      } else {
          reject('task2错误')
      }
    }, 1000)
  })
}

const task3 = (result) => {
  return new Promise((resolve, reject) => {
    let C = result
    setTimeout(() => {
      C = result * 2
      if (C) {
          resolve(C)
      } else {
          reject('task3错误')
      }
    }, 1000)
  })
}
```

#### 1.2 利用promise的链式调用来解决回调问题
``` javascript
task1(1).then(task2).then(task3).then((result)=>{console.log(result)}).catch((err)=>{console.log(err)})
```

#### 1.3 【终极写法】利用async, awit解决回调问题
``` javascript
async function foo(start, call) {
  let a = await task1 (start)
  let b = await task2 (a)
  let c = await task3 (b)
  call(c)
}
foo(2, (res)=>{console.log('yes:'+res)}).catch((err)=>{console.log(err)})
```