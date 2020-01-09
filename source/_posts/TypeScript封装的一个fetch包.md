---
title: TypeScript封装的一个fetch包
date: 2019-8-30 21:06:19
tags: moka
---

# 自己封了个 fetch ，总结一下几个关键点

## 1. 定义好要用的interface

```typescript
import { Omit } from './util'

export enum EMethod {
  Get = 'GET',
  Post = 'POST',
  Put = 'PUT',
  Delete = 'DELETE'
}

export interface IFetchResponse {
  data: any
  msg: string
  success: boolean
  RetCode: number
}

export interface IFetchConfig extends Omit<RequestInit, 'body'> {
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE'
  body?: object
  dataType?: 'hlj' | 'json' | 'text'
  bodyType?: 'json' | 'file'
  codeKey?: string
  ignore?: any[]
  onError?: ((RetCode, msg) => void) | null | false
  expect?: any
}
```
请求的方法、返回的格式、请求体的格式、都定义好按照这个来，有一点是原生的RequestInit的body是必传的，根据我的需求body可以不传，所以omit掉了。（注： 最新的ts好像内置omit方法）附omit方法：
```typescript
export type Omit<T, K> = Pick<T, Exclude<keyof T, K>>
```

## 2.参数的处理
原本的参数是一个对象的形式，get、delete请求方法时候要把对象转化成字符串拼接到url上

## 3.文件的处理
为了满足上传文件的需要，定义了一个 ‘file’ 类型的请求方式用于上传文件,使用FormData实现
```typescript
  // 上传文件
  if (bodyType === 'file') {
    const data = new FormData()
    Object.keys(body).forEach(key => {
      data.append(key, body[key])
    })
    requestInit.body = data
    delete requestInit.headers['Content-Type']
  }
```
## 4. fetch可配置化
每个项目中可能有不同的配置，比如自定义的统一错误提示、自定义的header
```typescript
export const configure = ({ config, ...rest }) => {
  if (isFunction(config)) {
    globalConfig = config(globalConfig)
  }
  globalConfig = { ...globalConfig, ...rest }
}
```
本地项目中可以调用configure方法 把自己自定义的配置merge上去。
## 5.写测试
```json
"dependencies": {
    "@babel/core": "^7.4.5",
    "@babel/preset-env": "^7.4.5",
    "@babel/preset-typescript": "^7.3.3",
    "@ryzen/eslint-config-ryzen": "^1.0.9",
    "@types/jest": "^24.0.13",
    "babel-jest": "^24.8.0",
    "eslint": "4.19.1",
    "global": "^4.3.2",
    "jest": "^24.8.0",
    "microbundle": "^0.11.0"
  },
```
```javascript
import http, { createApi } from '../src'

test('http, createApi, should have 4 method, get, post, post, delete', () => {
  ;[http, createApi].forEach(item => {
    expect(item).toHaveProperty('post')
    expect(item).toHaveProperty('get')
    expect(item).toHaveProperty('delete')
    expect(item).toHaveProperty('put')
  })
})

```
![](/image/20191007202531.jpg)
对每一个必要的模块加上单元测试，这样再次迭代之后跑一下jest就不怕改坏了。

## 6.使用lerna进行多包管理
因为这个包依赖了其他自己写的包（eslint-config包）并且使用lerna可以方便的管理package之间的依赖、一键发布、打changelog等 

![](/image/20191007203309.jpg)
![lerna的多包管理](/image/20191007203431.jpg)




