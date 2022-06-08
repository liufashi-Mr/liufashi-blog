---
title: typescript封装axios
date: 2022-05-21 16:48:59
categories:
  - FE
tags:
  - typescript
  - FE
  - axios
plugins:
  - indent
---

## 前言

最近工作繁忙，继上篇{% post_link typescript-h5-template%}已经一个月没有更新内容了。最近有个 h5 的项目正好使用到之前搭建的模板，于是将上述的模板拉过来，配个路由就直接可以使用了，节省不少时间。现在再将 axios 添加到项目中。这项下次再使用到的时候就会更加轻松。

## 功能

1. 常见配置（跨域携带 cookie，token，超时设置，请求头）
2. 请求拦截器和响应拦截器
3. 请求封装，使用同一套写法
4. 请求失败的提示信息
5. 支持单个请求的请求或者相应拦截单独处理

## 开始动手

首先安装 axios `npm i axios -S`，引入 axios，添加默认配置，其中基础路径 baseURL，一般是在.env 中拿[在 cra 中使用 env](https://www.html.cn/create-react-app/docs/adding-custom-environment-variables/)，如果是复杂的情况也可以单独建一个文件，通过 window.location.host 在不同环境的区别等等修改 baseURL

```ts
// 引入axios和后面会用到的ts类型
import axios, {
  AxiosRequestConfig,
  AxiosRequestHeaders,
  AxiosInstance,
  AxiosResponse,
} from "axios";
// 使用QueryString,当content-type为表单时“application/x-www-form-urlencoded” 将参数拼接为类似于get请求的形式 (a=1&b=3)
import QueryString from "qs";
import { message } from "antd";

interface RequestConfig extends AxiosRequestConfig {
  headers: AxiosRequestHeaders;
}
const defaultConfig: RequestConfig = {
  baseURL: "",
  timeout: 120 * 1000, //超时
  withCredentials: true, //允许跨域携带cookie
  headers: {
    // content-type 可以先和后端沟通使用JSON还是表单，后面有少数不一样的特殊处理就行
    "content-type": "application/x-www-form-urlencoded",
  },
};
```

上面就实现了基础配置的添加

```ts
const request = (
  method: "get" | "post" | "put" | "delete" | "patch" | "head" | "options",
  url: string,
  params?: any,
  config?: AxiosRequestConfig, //自定义的配置
  interceptor?: {
    request?: any;
    response?: any;
  }
): any => {
  const finalConfig: RequestConfig = { ...defaultConfig, ...config };
  const instance: AxiosInstance = axios.create(finalConfig); //使用传入的config覆盖默认
  instance.interceptors.request.use((request) => {
    // 请求拦截器
    return interceptor?.request && typeof interceptor.request === "function"
      ? //对单独的请求处理,执行请求传过来的
        interceptor.request(request)
      : // 在这里可以对请求参数做统一处理
        (() => {
          return request;
        })();
  });
  // 相应拦截器，与上面基本一致
  instance.interceptors.response.use(
    (
      response: AxiosResponse<COMMON.ResponseData>
    ): COMMON.ResponseData | Promise<never> => {
      if (response.status === 200 && response.data.success) {
        return interceptor?.response &&
          typeof interceptor.response === "function"
          ? interceptor.response(response)
          : (() => {
              return response.data;
            })();
      } else {
        // 弹出后端返回的错误信息
        message.error(response.data.msg);
        return Promise.reject(response.data);
      }
    },
    (error) => {
      return Promise.reject(error);
    }
  );

```

最后是处理请求参数

```ts
// 删除请求参数为undefined或者null的参数，让请求看起来更清爽。若有时需要传null等可以省略
 Object.keys(params as object).forEach((item) => {
    if (item && (params[item] === undefined || params[item] === null)) {
      delete params[item];
    }
  });

  // 当使用表单时“application/x-www-form-urlencoded”，使用qs修改参数形式
  const data =
    finalConfig.headers["content-type"] === "application/json"
      ? params
      : QueryString.stringify(params);
  // 两种不同的传参方式包含的请求方法
  const useParams = ["get","delete","head","options"]
  const useData = ["put","post","patch"]
  return instance({
    method,
    url,
    params:useParams.includes(method)&& params,
    data: useData.includes(method)  && data,
  });
};
export default request;
```

## 使用

首先呢，现在全局的 types 加上一个规定请求参数和相应参数的泛型

```ts
// 根据后端返回的参数定义
interface ResponseData<T = any> {
  code: number;
  data: T;
  msg: string;
  success: true | false;
}
// 这样我们在使用的时候只需要规定返回的data中的内容了
interface REQ<P, T> {
  (props: P): Promise<ResponseData<T>>;
}
```

然后在请求中使用,可以看下面的例子

```ts
interface RequestProps {
  username: string;
}

interface RecordResponseItem {
  ...
  username:string
  list: any[];
  ...
}


export const getTestContent: REQ<RequestProps, RecordResponseItem[]> = (
  params
) =>
  request(
    "post",
    "/xxx",
    params,
    {
      headers: {
        "content-type": "application/json",
      },
    },
    {
      request: (request: any) => {
        // 处理请求
        console.log(request);
      },
      response: (response: any) => {
        // 处理响应
        console.log(response);
      },
    }
  );


```

重点来啦

- 如果这个请求所有的都需要拦截操作可以直接写在这里，如上
- 如果是某次使用需要拦截可以在使用的时候传过来，如下

```ts
import { getTestContent } from "@apis";
// 这个时候要将请求参数的类型加上interceptors
getTestContent({
  username: "liufashi",
  interceptors: {
    request: (request: any) => {
      // 处理请求
      console.log(request);
    },
    response: (response: any) => {
      // 处理响应
      console.log(response);
    },
  },
});
// 简单的加一下类型
interface RequestProps {
  username: string;
  interceptors：any
}
```

到此一个相对好用的 axios 已经完成啦！
