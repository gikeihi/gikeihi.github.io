---
title: single promise
date: 2020-02-21 10:02:52
categories: javascript
tags: 
    - javascript
    - promise
    - 小程序
---

实现一个单例的promise。
<!-- more -->

# 问题  
javascript中对网络的请求都是异步调用。  
比如小程序在访问页面时，如果需要用到用户的openid，那么避免不了需要先做登录处理。  
但是当所有的请求都同时发出时，所有的请求都会执行一遍登录，这显然效率不高。  
那么如果有一个请求已经发起了登录请求，那么后续的请求只要排队等着即可。  

# 方案
解决方案有两种。
- 做一个队列，用一个flag来标识是否在登录请求中，如果是的话，就把后续的请求放到队列里。
```js
let localToken = wx.getStorageSync(LOCAL_TOKEN_KEY);
const loginQueue = [];
let isLoginning = false;
function getToken() {
  return new Promise((resolve, reject) => {
    if (!localToken) {
      loginQueue.push({
        resolve,
        reject
      })
      if (!isLoginning) {
        isLoginning = true
        internalLogin().then(token => {
          while (loginQueue.length) {
            loginQueue.shift().resolve(token);
          }
        }).catch(loginError).finally(() => {
          isLoginning = false;
        })
      }
    } else {
      resolve(localToken)
    }
  })
}
```
- 实现一个单例的promise，所有请求都用这个promise来获取token
```js
let localToken = wx.getStorageSync(LOCAL_TOKEN_KEY);
const promiseSingleton = (fn) => {
  let _ret = null
  return (...args) => {
    if (_ret) {
      return _ret
    }
    _ret = fn(...args)
    _ret.then(res => {
      _ret = null
      return res
    }).catch(err => {
      _ret = null
      return Promise.reject(err)
    })
    return _ret
  }
}
const singleLogin = promiseSingleton(internalLogin)
function getToken() {
  console.log("gettoken")
  return new Promise((resolve, reject) => {
    if (!localToken) {
      singleLogin().then(token => {
        resolve(token)
      }).catch(loginError)
    } else {
      resolve(localToken)
    }
  })
}
```