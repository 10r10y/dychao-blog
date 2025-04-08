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