---
title: Promise 手写实现
date: 2023-05-25 17:11:16
tags:
- 手写
- Pormise
categories: 
- JavaScript
---

#### 实现步骤
1. 状态
2. 成功回调
3. 失败回调
4. 执行器
5. then
6. 在同步代码之后再执行：setTimeout 0
7. pending状态时不直接执行回调，而是压入回调数组，触发resolve或reject时执行
8. then 的链式调用

#### JS版本
```javascript
class MyPromise {
    constructor(executor) {
      // Promise 状态：pending, fulfilled, rejected
      this.status = 'pending';
      // Promise 结果值
      this.value = undefined;
      // Promise 拒绝原因
      this.reason = undefined;
      // 成功回调函数数组
      this.onFulfilledCallbacks = [];
      // 失败回调函数数组
      this.onRejectedCallbacks = [];
  
      // resolve 函数定义
      const resolve = (value) => {
        if (this.status === 'pending') {
          this.status = 'fulfilled';
          this.value = value;
          // 执行所有成功回调
          this.onFulfilledCallbacks.forEach(callback => callback(this.value));
        }
      };
  
      // reject 函数定义
      const reject = (reason) => {
        if (this.status === 'pending') {
          this.status = 'rejected';
          this.reason = reason;
          // 执行所有失败回调
          this.onRejectedCallbacks.forEach(callback => callback(this.reason));
        }
      };
  
      // 执行器立即执行
      try {
        executor(resolve, reject);
      } catch (error) {
        reject(error);
      }
    }
  
    // then 方法
    then(onFulfilled, onRejected) {
      // 处理入参不是函数情况
      onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
      onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason };
  
      // 返回新的 Promise 实现链式调用
      const promise2 = new MyPromise((resolve, reject) => {
        if (this.status === 'fulfilled') {
          // 使用 setTimeout 模拟异步
          setTimeout(() => {
            try {
              const x = onFulfilled(this.value);
              resolve(x);
            } catch (error) {
              reject(error);
            }
          }, 0);
        }
  
        if (this.status === 'rejected') {
          setTimeout(() => {
            try {
              const x = onRejected(this.reason);
              resolve(x);
            } catch (error) {
              reject(error);
            }
          }, 0);
        }
  
        if (this.status === 'pending') {
          // 存储回调
          this.onFulfilledCallbacks.push(() => {
            setTimeout(() => {
              try {
                const x = onFulfilled(this.value);
                resolve(x);
              } catch (error) {
                reject(error);
              }
            }, 0);
          });
  
          this.onRejectedCallbacks.push(() => {
            setTimeout(() => {
              try {
                const x = onRejected(this.reason);
                resolve(x);
              } catch (error) {
                reject(error);
              }
            }, 0);
          });
        }
      });
  
      return promise2;
    }
  
    // catch 方法
    catch(onRejected) {
      return this.then(null, onRejected);
    }
  
    // 静态方法 resolve
    static resolve(value) {
      return new MyPromise((resolve) => {
        resolve(value);
      });
    }
  
    // 静态方法 reject
    static reject(reason) {
      return new MyPromise((resolve, reject) => {
        reject(reason);
      });
    }
  }
```

#### TS版本
```typescript
class CustomPromise<T> {
  private status: 'pending' | 'fulfilled' | 'rejected';
  private value: T | undefined;
  private reason: any;
  private onFulfilledCallbacks: Array<() => void>;
  private onRejectedCallbacks: Array<() => void>;

  constructor(executor: (resolve: (value: T | PromiseLike<T>) => void, reject: (reason: any) => void) => void) {
    // Promise 状态：pending, fulfilled, rejected
    this.status = 'pending';
    // Promise 结果值
    this.value = undefined;
    // Promise 拒绝原因
    this.reason = undefined;
    // 成功回调函数数组
    this.onFulfilledCallbacks = [];
    // 失败回调函数数组
    this.onRejectedCallbacks = [];

    // resolve 函数定义
    const resolve = (value: T | PromiseLike<T>): void => {
      if (this.status === 'pending') {
        this.status = 'fulfilled';
        this.value = value as T;
        // 执行所有成功回调
        this.onFulfilledCallbacks.forEach(callback => callback());
      }
    };

    // reject 函数定义
    const reject = (reason: any): void => {
      if (this.status === 'pending') {
        this.status = 'rejected';
        this.reason = reason;
        // 执行所有失败回调
        this.onRejectedCallbacks.forEach(callback => callback());
      }
    };

    // 执行器立即执行
    try {
      executor(resolve, reject);
    } catch (error) {
      reject(error);
    }
  }

  // then 方法
  then<TResult1 = T, TResult2 = never>(
    onFulfilled?: ((value: T) => TResult1 | PromiseLike<TResult1>) | undefined | null,
    onRejected?: ((reason: any) => TResult2 | PromiseLike<TResult2>) | undefined | null
  ): CustomPromise<TResult1 | TResult2> {
    // 处理参数可选情况
    const realOnFulfilled = typeof onFulfilled === 'function' ? onFulfilled : (value: T): T => value;
    const realOnRejected = typeof onRejected === 'function' ? onRejected : (reason: any): never => { throw reason };

    // 返回新的 Promise 实现链式调用
    const promise2 = new CustomPromise<TResult1 | TResult2>((resolve, reject) => {
      if (this.status === 'fulfilled') {
        // 使用 setTimeout 模拟异步
        setTimeout(() => {
          try {
            const x = realOnFulfilled(this.value as T);
            resolve(x as TResult1 | TResult2);
          } catch (error) {
            reject(error);
          }
        }, 0);
      }

      if (this.status === 'rejected') {
        setTimeout(() => {
          try {
            const x = realOnRejected(this.reason);
            resolve(x);
          } catch (error) {
            reject(error);
          }
        }, 0);
      }

      if (this.status === 'pending') {
        // 存储回调
        this.onFulfilledCallbacks.push(() => {
          setTimeout(() => {
            try {
              const x = realOnFulfilled(this.value as T);
              resolve(x as TResult1 | TResult2);
            } catch (error) {
              reject(error);
            }
          }, 0);
        });

        this.onRejectedCallbacks.push(() => {
          setTimeout(() => {
            try {
              const x = realOnRejected(this.reason);
              resolve(x);
            } catch (error) {
              reject(error);
            }
          }, 0);
        });
      }
    });

    return promise2;
  }

  // catch 方法
  catch<TResult = never>(
    onRejected?: ((reason: any) => TResult | PromiseLike<TResult>) | undefined | null
  ): CustomPromise<T | TResult> {
    return this.then(null, onRejected);
  }

  // 静态方法 resolve
  static resolve<TValue>(value: TValue | PromiseLike<TValue>): CustomPromise<TValue> {
    return new CustomPromise<TValue>((resolve) => {
      resolve(value);
    });
  }

  // 静态方法 reject
  static reject<T = never>(reason: any): CustomPromise<T> {
    return new CustomPromise<T>((resolve, reject) => {
      reject(reason);
    });
  }
} 
```

#### Promise A+ 版本
```javascript
// 记录Promise的三种状态
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

/**
 * 运行一个微队列任务
 * 把传递的函数放到微队列中
 * @param {Function} callback
 */
function runMicroTask(callback) {
  // 判断node环境
  // 为了避免「变量未定义」的错误，这里最好加上前缀globalThis
  // globalThis是一个关键字，指代全局对象，浏览器环境为window，node环境为global
  if (globalThis.process && globalThis.process.nextTick) {
    process.nextTick(callback);
  } else if (globalThis.MutationObserver) {
    const p = document.createElement("p");
    const observer = new MutationObserver(callback);
    observer.observe(p, {
      childList: true, // 观察该元素内部的变化
    });
    p.innerHTML = "1";
  } else {
    setTimeout(callback, 0);
  }
}

/**
 * 判断一个数据是否是Promise对象
 * @param {any} obj
 * @returns
 */
function isPromise(obj) {
  return !!(obj && typeof obj === "object" && typeof obj.then === "function");
}

class MyPromise {
  /**
   * 创建一个Promise
   * @param {Function} executor 任务执行器，立即执行
   */
  constructor(executor) {
    this._state = PENDING; // 状态
    this._value = undefined; // 数据
    this._handlers = []; // 处理函数形成的队列
    try {
      executor(this._resolve.bind(this), this._reject.bind(this));
    } catch (error) {
      this._reject(error);
      console.error(error);
    }
  }

  /**
   * 向处理队列中添加一个函数
   * @param {Function} executor 添加的函数
   * @param {String} state 该函数什么状态下执行
   * @param {Function} resolve 让then函数返回的Promise成功
   * @param {Function} reject 让then函数返回的Promise失败
   */
  _pushHandler(executor, state, resolve, reject) {
    this._handlers.push({
      executor,
      state,
      resolve,
      reject,
    });
  }

  /**
   * 根据实际情况，执行队列
   */
  _runHandlers() {
    if (this._state === PENDING) {
      // 目前任务仍在挂起
      return;
    }
    while (this._handlers[0]) {
      const handler = this._handlers[0];
      this._runOneHandler(handler);
      this._handlers.shift();
    }
  }

  /**
   * 处理一个handler
   * @param {Object} handler
   */
  _runOneHandler({ executor, state, resolve, reject }) {
    runMicroTask(() => {
      if (this._state !== state) {
        // 状态不一致，不处理
        return;
      }

      if (typeof executor !== "function") {
        // 传递后续处理并非一个函数
        this._state === FULFILLED ? resolve(this._value) : reject(this._value);
        return;
      }
      try {
        const result = executor(this._value);
        if (isPromise(result)) {
          result.then(resolve, reject);
        } else {
          resolve(result);
        }
      } catch (error) {
        reject(error);
        console.error(error);
      }
    });
  }

  /**
   * Promise A+规范的then
   * @param {Function} onFulfilled
   * @param {Function} onRejected
   */
  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      this._pushHandler(onFulfilled, FULFILLED, resolve, reject);
      this._pushHandler(onRejected, REJECTED, resolve, reject);
      this._runHandlers(); // 执行队列
    });
  }

  /**
   * 仅处理失败的场景
   * @param {Function} onRejected
   */
  catch(onRejected) {
    return this.then(null, onRejected);
  }

  /**
   * 无论成功还是失败都会执行回调
   * @param {Function} onSettled
   */
  finally(onSettled) {
    return this.then(
      (data) => {
        onSettled();
        return data;
      },
      (reason) => {
        onSettled();
        throw reason;
      }
    );
  }

  /**
   * 更改任务状态
   * @param {String} newState 新状态
   * @param {any} value 相关数据
   */
  _changeState(newState, value) {
    if (this._state !== PENDING) {
      // 目前状态已经更改
      return;
    }
    this._state = newState;
    this._value = value;
    this._runHandlers(); // 状态变化，执行队列
  }

  /**
   * 标记当前任务完成
   * @param {any} data 任务完成的相关数据
   */
  _resolve(data) {
    this._changeState(FULFILLED, data);
  }

  /**
   * 标记当前任务失败
   * @param {any} reason 任务失败的相关数据
   */
  _reject(reason) {
    this._changeState(REJECTED, reason);
  }

}

```