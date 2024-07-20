---
title: A Peek at useRequest hook
date: 2024-03-04 21:34:11
categories: "Front-end"
tags:
  - Hooks
  - React
  - Web
---

# Introduction

---

`useRequest` is a powerful, well-encapsulated hook from a hook library ahooks to manage async data fetching. When there is multiple async logic in a single component in React, we will deal with a bunch of useState and useEffect hooks, which makes it complicated to call APIs.

What it probably looks like:

```tsx
// Component.ts
const [ data, setData ] = useState<object>(defaultData)
const [ isLoading, setIsloading ] = useState<boolean>(false)

useEffect(() => {
	setIsloading(true)
  request = service.fetchData(...)
  setData(...)
  handlerError(...)
	setIsloading(false)
}, [])
```

With the help of `useRequest`, we can simplify our code:

```tsx
import { useRequest } from "ahooks";

const {
  data,
  run: request,
  loading,
  error,
} = useRequest(service.serviceA, options);
```

# Main features

---

`useRequest` provides sufficient enough functionalities for network request scenarios in React projects including:

- Automatic/manual request
- Polling
- Debounce
- Throttling
- Refresh on window focus
- Error retry
- Loading delay
- SWR(stale-while-revalidate)
- Caching

# A Glance on Basic Usage

---

## Loading delay

Set the delay time for loading to become `true`

```tsx
const { loading, data } = useRequest(getUsername, {
  loadingDelay: 300, //Set the delay time for loading to become true
});

return <div>{loading ? "Loading..." : data}</div>;
```

## Polling

By setting `options.pollingInterval` , enter the polling mode, `useRequest`  will periodically trigger service execution.

```tsx
const { data, run, cancel } = useRequest(getUsername, {
  pollingInterval: 3000, //will periodically trigger service execution.
});
```

## Refresh on window focus

the request will be refreshed when the browser is `refocus`  and `revisible`.

```tsx
const { data } = useRequest(getUsername, {
  refreshOnWindowFocus: true,
});
```

## Debounce & Throttling

Enter the debounce mode by setting `options.debounceWait` / `options.throttleWait`. At this time, if `run` or `runAsync` is triggered frequently, the request will be executed with the debounce/throttle strategy.

```tsx
const { data, run } = useRequest(getUsername, {
  debounceWait: 300,
  throttleWait: 300,
  manual: true,
});
```

## Cache & SWR

If `options.cacheKey`  is set, `useRequest`  will cache the successful data . The next time the component is initialized, if there is cached data, it will return the cached data first, and then send a new request in background, which is the ability of SWR.

```tsx
async function getArticle(): Promise<{ data: string; time: number }> {
  console.log("cacheKey");
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        data: Mock.mock("@paragraph"),
        time: new Date().getTime(),
      });
    }, 1000);
  });
}

const Article = () => {
  const { data, loading } = useRequest(getArticle, {
    cacheKey: "cacheKey-demo",
  });
  if (!data && loading) {
    return <p>Loading</p>;
  }
  return (
    <>
      <p>Background loading: {loading ? "true" : "false"}</p>
      <p>Latest request time: {data?.time}</p>
      <p>{data?.data}</p>
    </>
  );
};
```

## Error retry

By setting `options.retryCount` , set the number of error retries, useRequest will retry after it fails.

```tsx
const { data, run } = useRequest(getUsername, {
  retryCount: 3,
});
```

# Design Pattern

`useRequest` has two main modules that work together to serve its functionality: the main `Fetch` class and plugins

The **Plugin** module uses varieties of different plugins, each of which only works for a specific function.

Fetch module on the other hand is even more simple - to implement the `Fetch` class which aggregates all plugins to this hook to make it robust and easy to maintain.

![useRequest diagram](https://cdn.jsdelivr.net/gh/playitsafe/cdn/img/2022-07-30-useRequest-diagram.png)

# Source code

---

## `Fetch` - the core

Structure of `Fetch` class:

```tsx
export default class Fetch<TData, TParams extends any[]> {
  pluginImpls: PluginReturn<TData, TParams>[];

  count: number = 0;

  state: FetchState<TData, TParams> = {
    loading: false,
    params: undefined,
    data: undefined,
    error: undefined,
  };

  constructor(
    public serviceRef: MutableRefObject<Service<TData, TParams>>,
    public options: Options<TData, TParams>,
    public subscribe: Subscribe,
    public initState: Partial<FetchState<TData, TParams>> = {},
  ) {
    this.state = {
      ...this.state,
      loading: !options.manual,
      ...initState,
    };
  }

  setState(s: Partial<FetchState<TData, TParams>> = {}) {...}

  runPluginHandler(event: keyof PluginReturn<TData, TParams>, ...rest: any[]) {...}

  async runAsync(...params: TParams): Promise<TData> {...}

  run(...params: TParams) {...}

  cancel() {...}

  refresh() {...}

  refreshAsync() {...}

  mutate(data?: TData | ((oldData?: TData) => TData | undefined)) {...}
```

Most of its API is provided for the user to call such as `run`、`runAsync`、`cancel`、`refresh`、`refreshAsync`、`mutate`, while `runPluginHandler`、`setState` are for internal use.

### `pluginImpls`

As per its properties, we can see it has a `pluginImpls` property, from its type `PluginReturn<TData, TParams>[]` it seems to contain results of all plugins after execution.

```tsx
export interface PluginReturn<TData, TParams extends any[]> {
  onBefore?: (params: TParams) =>
    | ({
        stopNow?: boolean;
        returnNow?: boolean;
      } & Partial<FetchState<TData, TParams>>)
    | void;

  onRequest?: (
    service: Service<TData, TParams>,
    params: TParams
  ) => {
    servicePromise?: Promise<TData>;
  };

  onSuccess?: (data: TData, params: TParams) => void;
  onError?: (e: Error, params: TParams) => void;
  onFinally?: (params: TParams, data?: TData, e?: Error) => void;
  onCancel?: () => void;
  onMutate?: (data: TData) => void;
}
```

Inside the `PluginReturn<TData, TParams>` type, it stores some lifecycle callback hooks which will be called at a certain phase of the request.

### `state`

There’s also a `state` property of `FetchState<TData, TParams>` type. The type definition below shows it stores the context of the request. `loading` ,`data`, `errors` are the results we’d like to get from `useRequest`

```tsx
export interface FetchState<TData, TParams extends any[]> {
  loading: boolean;
  params?: TParams;
  data?: TData;
  error?: Error;
}
```

```tsx
const { data, error, loading } = useRequest(service);
```

And the `setState` API is used to update the state.

Two main APIs of the `Fetch` class are `runPluginHandler` and `runAsync` , which are called by all of the other APIs to do some extra work.

### `runPluginHandler`

```tsx
runPluginHandler(event: keyof PluginReturn<TData, TParams>, ...rest: any[]) {
	// @ts-ignore
  const r = this.pluginImpls.map((i) => i[event]?.(...rest)).filter(Boolean);
  return Object.assign({}, ...r);
}
```

This function accepts an event parameter which is of the union type `onBefore | onRequest | onSuccess | onError | onFinally | onCancel | onMutate` and other extra parameters. What this handler does is to call the relevant lifecycle hook from `pluginImpls` and return its result.

### `runAsync`

```tsx
async runAsync(...params: TParams): Promise<TData> {
    this.count += 1;
    const currentCount = this.count;

    const {
      stopNow = false,
      returnNow = false,
      ...state
    } = this.runPluginHandler('onBefore', params);

    // stop request
    if (stopNow) {
      return new Promise(() => {});
    }

    this.setState({
      loading: true,
      params,
      ...state,
    });

    // return now
    if (returnNow) {
      return Promise.resolve(state.data);
    }

    this.options.onBefore?.(params);

    try {
      // replace service
      let { servicePromise } = this.runPluginHandler('onRequest', this.serviceRef.current, params);

      if (!servicePromise) {
        servicePromise = this.serviceRef.current(...params);
      }

      const res = await servicePromise;

      if (currentCount !== this.count) {
        // prevent run.then when request is canceled
        return new Promise(() => {});
      }

      // const formattedResult = this.options.formatResultRef.current ? this.options.formatResultRef.current(res) : res;

      this.setState({
        data: res,
        error: undefined,
        loading: false,
      });

      this.options.onSuccess?.(res, params);
      this.runPluginHandler('onSuccess', res, params);

      this.options.onFinally?.(params, res, undefined);

      if (currentCount === this.count) {
        this.runPluginHandler('onFinally', params, res, undefined);
      }

      return res;
    } catch (error) {
      if (currentCount !== this.count) {
        // prevent run.then when request is canceled
        return new Promise(() => {});
      }

      this.setState({
        error,
        loading: false,
      });

      this.options.onError?.(error, params);
      this.runPluginHandler('onError', error, params);

      this.options.onFinally?.(params, undefined, error);

      if (currentCount === this.count) {
        this.runPluginHandler('onFinally', params, undefined, error);
      }

      throw error;
    }
  }
```

What this long function does is to implement callbacks that are passed in to give users the opportunity to process the result of the request instead of handling it automatically.

For example, In an `onBefore` hook, user can cancel a request before it’s been sent out ; In an `onRequest` hook, the function to fetch data can be overwritten, etc.

### Other APIs

Other APIs such as `run`、`cancel`、`refresh` will eventually call `runPluginHandler`
 and `runAsync` .

The main responsibility of this `Fetch` class is to run callbacks in different phases of a request lifecycle and update the state.

## Plugins

The implementation of `useRequest` separates the core logic and the complicity of each different functionality by the plugin mechanism. `Fetch` only care about when to call those plugin hooks and each plugin itself will only focus on customizing and doing its own logic.

Take `usePollingPlugin` as an example, the main logic of this plugin is to set a timeout in `onFinally` callback after each request using `pollingInterval` passed by users and run `refresh` function of the `Fetch` instance.

```tsx
const usePollingPlugin: Plugin<any, any[]> = (
  fetchInstance,
  { pollingInterval, pollingWhenHidden = true }
) => {
  const timerRef = useRef<NodeJS.Timeout>();
  const unsubscribeRef = useRef<() => void>();

  const stopPolling = () => {
    if (timerRef.current) {
      clearTimeout(timerRef.current);
    }
    unsubscribeRef.current?.();
  };

  useUpdateEffect(() => {
    if (!pollingInterval) {
      stopPolling();
    }
  }, [pollingInterval]);

  if (!pollingInterval) {
    return {};
  }

  return {
    onBefore: () => {
      stopPolling();
    },
    onFinally: () => {
      // if pollingWhenHidden = false && document is hidden, then stop polling and subscribe revisible
      if (!pollingWhenHidden && !isDocumentVisible()) {
        unsubscribeRef.current = subscribeReVisible(() => {
          fetchInstance.refresh();
        });
        return;
      }

      timerRef.current = setTimeout(() => {
        fetchInstance.refresh();
      }, pollingInterval);
    },
    onCancel: () => {
      stopPolling();
    },
  };
};
```

## Adding up

To hook up the core `Fetch` class and plugins together to make this hook work, `useRequestImplement` is called and accepts request options and plugins from a higher level and `Fetch` will be instantiated inside the function.

```tsx
function useRequestImplement<TData, TParams extends any[]>(
  service: Service<TData, TParams>,
  options: Options<TData, TParams> = {},
  plugins: Plugin<TData, TParams>[] = []
) {
  const { manual = false, ...rest } = options;

  const fetchOptions = {
    manual,
    ...rest,
  };

  const serviceRef = useLatest(service);

  const update = useUpdate();

  const fetchInstance = useCreation(() => {
    const initState = plugins
      .map((p) => p?.onInit?.(fetchOptions))
      .filter(Boolean);

    return new Fetch<TData, TParams>(
      serviceRef,
      fetchOptions,
      update,
      Object.assign({}, ...initState)
    );
  }, []);
  fetchInstance.options = fetchOptions;
  // run all plugins hooks
  fetchInstance.pluginImpls = plugins.map((p) =>
    p(fetchInstance, fetchOptions)
  );

  useMount(() => {
    if (!manual) {
      // useCachePlugin can set fetchInstance.state.params from cache when init
      const params = fetchInstance.state.params || options.defaultParams || [];
      // @ts-ignore
      fetchInstance.run(...params);
    }
  });

  useUnmount(() => {
    fetchInstance.cancel();
  });

  return {
    loading: fetchInstance.state.loading,
    data: fetchInstance.state.data,
    error: fetchInstance.state.error,
    params: fetchInstance.state.params || [],
    cancel: useMemoizedFn(fetchInstance.cancel.bind(fetchInstance)),
    refresh: useMemoizedFn(fetchInstance.refresh.bind(fetchInstance)),
    refreshAsync: useMemoizedFn(fetchInstance.refreshAsync.bind(fetchInstance)),
    run: useMemoizedFn(fetchInstance.run.bind(fetchInstance)),
    runAsync: useMemoizedFn(fetchInstance.runAsync.bind(fetchInstance)),
    mutate: useMemoizedFn(fetchInstance.mutate.bind(fetchInstance)),
  } as Result<TData, TParams>;
}

export default useRequestImplement;
```

Finally, this function will be returned in a `useRequest` function with custom plugins along with its native plugins passed in.

```tsx
function useRequest<TData, TParams extends any[]>(
  service: Service<TData, TParams>,
  options?: Options<TData, TParams>,
  plugins?: Plugin<TData, TParams>[]
) {
  return useRequestImplement<TData, TParams>(service, options, [
    ...(plugins || []),
    useDebouncePlugin,
    useLoadingDelayPlugin,
    usePollingPlugin,
    useRefreshOnWindowFocusPlugin,
    useThrottlePlugin,
    useRefreshDeps,
    useCachePlugin,
    useRetryPlugin,
    useReadyPlugin,
  ] as Plugin<TData, TParams>[]);
}
```

# Summarise

The main idea of implementing a plugin is to find out the appropriate phase of the request lifecycle and plug in the core logic of the hook. The most important takeaway from the exploration of the hook’s source code is the approach of separating its core `Fetch` function and its plugins, which makes it more reusable and maintainable. Users are able to extend the plugins easily as they wish and each of the plugins works independently. I believe it’s a great example of the single responsibility principle and that’s something I could borrow from when customizing a hook or implementing complicated logic.

## Reference

https://ahooks.js.org/hooks/use-request/basic
https://github.com/alibaba/hooks/tree/master/packages/hooks/src/useRequest/src
