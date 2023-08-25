Vue3 项目下，如何用 TypeScript 封装一个 useRequest

# 前言
搭建个人网站时，需要一个管理后台，最开始的目的是管理博客内容，就新建了一个 `Vue3` + `Vite` + `TypeScript` 的 CMS 项目  
类似于 `React hooks`，基于 Composition API 的方式组织代码体验良好。但是 `axios` 请求需要很多**重复代码**来**维护状态**，希望由一个类似`ahooks useRequest`的钩子来简化代码量。  
找了一圈，`vueUse`是个不错的选择，其中有`useFetch`和`useAxios`  
 - 不想用 fetch `useFetch` 排除  
 - `useAxios` 调用前声明，每次都要传入 axios 实例，接口维护完全和 useAxios 绑定，既麻烦，又高耦合  

于是参考其他网上的博客和 ahooks, 自己做了一个 useRequest 封装。

# 环境
|        Env | Vision   |
| ---------: | :------- |
|        Vue | v3.2.45  |
| TypeScript | v4.9.4   |
|      axios | v1.2.5   |

# API 声明
只和 `axios` 关联的 api 声明方式是我个人推崇的。纯粹，而且多用，不仅是在 component setup 之中，其他地方也可以直接使用。  
```typescript
// src/api/blog.ts
const BLOG_PREFIX = '/nest-api/blog/'

export const TagApi = {
  findAll() {
    return request<Tag[]>({ url: `${BLOG_PREFIX}tag`, method: 'GET' })
  },
  findOne(id: Tag['id']) {
    return request<Tag>({ url: `${BLOG_PREFIX}tag/${id}`, method: 'GET' })
  },
  add(data: Partial<Tag>) {
    return request<Tag>({ url: `${BLOG_PREFIX}tag`, data, method: 'POST' })
  },
  edit(id: Tag['id'], data: Partial<Tag>) {
    return request<Tag>({ url: `${BLOG_PREFIX}tag/${id}`, data, method: 'PATCH' })
  },
  delete(id: Tag['id']) {
    return request<Tag>({ url: `${BLOG_PREFIX}tag/${id}`, method: 'DELETE' })
  }
}
```

# axios 配置
预设配置，加点拦截器
```typescript
/*
 * @FilePath: src/utils/request.ts
 */
import type { RequestResponse, Response } from '@/types'
import type {
  AxiosInstance,
  AxiosRequestConfig,
  InternalAxiosRequestConfig,
  Canceler
} from 'axios'
import axios from 'axios'
import { ref } from 'vue'

const config = {
  withCredentials: true, // send cookies when cross-domain requests
  timeout: 10 * 1000
  // transformResponse: [
  //   (data) => {
  // Do whatever you want to transform the data
  //     return JSONbig.parse(data)
  //   }
  // ]
}
// create an axios request
export const instance: AxiosInstance = axios.create(config)

// request interceptor
instance.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    return config
  },
  error => {
    // do something with request error
    return Promise.reject(error)
  }
)

// response interceptor
instance.interceptors.response.use(
  (response: Response) => {
    const res = response.data
    if (!res || !res.code || res.code !== 200) {
      return Promise.reject(res)
    }
    return Promise.resolve(response)
  },
  error => {
    if (axios.isCancel(error)) {
      return Promise.reject({
        code: 10000,
        message: 'Cancel',
        data: null
      })
    }
    if (error.code === 'ECONNABORTED') {
      return Promise.reject({
        code: 10001,
        message: 'Timeout',
        data: null
      })
    }
    return Promise.reject(error)
  }
)
// 通过封装暴露 cancel 方法
function request<T>(config: AxiosRequestConfig): RequestResponse<T> {
  const cancel = ref<Canceler>()
  return {
    instance: instance({
      ...config,
      cancelToken: new axios.CancelToken(c => {
        cancel.value = c
      })
    }),
    cancel
  }
}
export default request
```
这里的 `instance.interceptors.response` use 的第一个 `onFulfill(response)` 参数类型 Response 是自己封装的，为了适配后端接口，如下
```typescript
/*
 * @FilePath: /src/types/index.ts
 */
import type { AxiosResponse, Canceler } from 'axios'
import type { Ref } from 'vue'
export interface ResponseWrapper<T = any> {
  message: string
  success: boolean
  code: number
  data: T
}

export type Response<T = any> = AxiosResponse<ResponseWrapper<T>>

export interface RequestResponse<T> {
  instance: Promise<Response<T>>
  cancel: Ref<Canceler | undefined>
}

// useRequest 第一次参数，传入 api
export type Service<T, P extends any[]> = (...args: P) => RequestResponse<T>

```
如果不需要适配，可以用`AxiosResponse<T>`替换  
`RequestResponse` 和 `Service` 是 `useRequest` 的基础，这个类型是要保证后面调用可以有良好的**类型提示**体验。

# useRequest
`useRequest<T, P = any[]>`, 命名应该都比较语义化了  
泛型参数 T 可以从 api 推导，P 是 api 调用参数， 这个思考了很久，参考了 ahooks 写的，也算是 ts 难点了
```typescript
/*
 * @FilePath: /src/hooks/useRequest.ts
 */
import type { Service } from '@/types'
import type { Canceler } from 'axios'
import { toRefs, watch, reactive } from 'vue'
import type { UnwrapRef, WatchSource, ComputedRef } from 'vue'

interface RequestOptions<T, P extends any[]> {
  manual?: boolean
  defaultParameters?: P
  repeatCancel?: boolean
  refreshDeps?: WatchSource<any>[]
  refreshDepsParameters?: ComputedRef<P>
  onSuccess?: (response: T, params: P) => void
  onError?: (err: any, params: P) => void
}
export interface IRequestResult<T> {
  data?: T
  loading: boolean
  cancel?: Canceler
  error?: any
}

function useRequest<T, P extends any[]>(api: Service<T, P>, options?: RequestOptions<T, P>) {
  const {
    manual = false,
    defaultParameters = [] as unknown as P,
    repeatCancel = false,
    refreshDeps,
    refreshDepsParameters,
    onSuccess,
    onError
  } = options || {}

  const result = reactive<IRequestResult<T>>({
    data: undefined,
    loading: false,
    cancel: undefined,
    error: null
  })

  async function run(...args: P) {
    result.loading = true
    if (repeatCancel) {
      result.cancel && result.cancel()
    }
    const { instance, cancel } = api(...args)
    result.cancel = cancel.value
    return instance
      .then(res => {
        result.data = res.data.data as unknown as UnwrapRef<T>
        if (typeof onSuccess === 'function') {
          onSuccess(res.data.data, args)
        }
        return res.data.data
      })
      .catch(error => {
        if (typeof onError === 'function') {
          onError(error, args)
        }
        return Promise.reject(error)
      })
      .finally(() => {
        result.loading = false
      })
  }

  if (!manual) run(...defaultParameters)

  if (refreshDeps && refreshDeps.length) {
    watch(
      refreshDeps,
      () => {
        const args = refreshDepsParameters
          ? refreshDepsParameters.value
          : defaultParameters
          ? defaultParameters
          : ([] as unknown as P)
        run(...args)
      },
      { deep: true }
    )
  }

  return {
    run,
    ...toRefs(result)
  }
}

export default useRequest

```

# How to Invoke
```typescript
// sample example:
<script lang='ts' setup>
const { data: tags, run: fetchTags } = useRequest(TagApi.findAll)  // auto invoke
const { run: runDeleteTag } = useRequest(TagApi.delete, { manual: true })
const { run: runAddTag, loading: adding } = useRequest(TagApi.add, { manual: true })
const { run: runEditTag, loading: editing } = useRequest(TagApi.edit, { manual: true })
</script>
```
类似 `ahooks`，调用简单

