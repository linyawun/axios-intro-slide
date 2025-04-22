---
highlighter: shiki
css: unocss
colorSchema: dark
transition: fade-out
mdc: true
layout: center
glowSeed: 4
title: 'Exploring Axios: What Does This Library Do for Us?'
info: |
  ## Exploring Axios: What Does This Library Do for Us?
  - speaker：Monica
  - date：2025.5.20
  - presentation at: Langlive Tech Sharing
author: Monica
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
fonts:
  # basically the text
  sans: Robot Noto Sans
  # use with `font-serif` css class from UnoCSS
  serif: Robot Noto Serif
  # for code blocks, inline code, etc.
  mono: Fira Code
# slidev style reference: https://github.com/antfu/talks/tree/main/2024-06-13
---

# Exploring Axios

## What Does This Library Do for Us?

<div class="mt-6">
<p>speaker：Monica</p>
<p>2025.5.20 @Langlive Tech Sharing</p>
</div>

<style>
  .slidev-page h1{
    @apply text-size-5xl;
  }
  .slidev-page h2{
    @apply text-light-700;
  }
  .slidev-layout p{
    margin-top: 0px;
    margin-bottom: 0.5rem;
    opacity: 0.6;
  }
</style>

---

```yaml
glowSeed: 15
glowOpacity: 0.3
```

# What is Axios?

- Promise based HTTP client for the browser and node.js
- Built-in many convenient features, such as request/response interceptors, error handling, request cancellation, etc.
<br>
<br>
<div my-10 w-min flex="~ gap-1" items-center justify-center>
  <mdi:github op50 ma text-xl />
  <a href="https://github.com/axios/axios" target="_blank" class="font-300 mr-4">axios</a>
  <mdi:file-document op50 ma text-xl/>
  <a href="https://github.com/axios/axios" target="_blank" class="font-300">axios</a>
</div>

---

# What is Axios?

Axios and Fetch

| Feature                   | axios | fetch                         |
| ------------------------- | ----- | ----------------------------- |
| Automatic JSON parsing    | ✅    | ❌ (requires `res.json()`)    |
| Request Interceptors      | ✅    | ❌                            |
| Response Interceptors     | ✅    | ❌                            |
| Automatic params handling | ✅    | ❌                            |
| Built-in error handling   | ✅    | ❌ (manual handling required) |

---

```yaml
layout: cover
```

# Basic Axios Usage

---

# Basic Axios Usage

<div class='note-block'>
  This axios example and source code version uses v1.8.4
</div>

- Direct axios request
  - `axios(config)`
    ```js
    axios({
      method: 'get',
      url: 'https://jsonplaceholder.typicode.com/posts/1',
      headers: { 'X-Custom-Header': 'foobar' },
    });
    ```
  - `axios(url[, config])`
    ```js
    axios('https://jsonplaceholder.typicode.com/posts/1', {
      headers: { 'X-Custom-Header': 'foobar' },
    });
    ```
  - `axios.requestMethod`
    ```js
    axios.get('https://jsonplaceholder.typicode.com/posts/1');
    ```

---

# Basic Axios Usage

- Create an axios instance for requests
  ```js
  const apiClient = axios.create({
    baseURL: 'https://jsonplaceholder.typicode.com',
    timeout: 5000,
    headers: { 'X-Custom-Header': 'foobar' },
  });
  apiClient.get('/posts/1').then((res) => console.log(res.data));
  ```

---

# Basic Axios Usage - 🔍 Source Code

### axios is exported in <a href='https://github.com/axios/axios/blob/v1.x/lib/axios.js' target='_blank' class='hover:text-[#c5c3fb]!'>lib/axios.js</a>

1. `createInstance`
<div class='ml-6'>

```js
function createInstance(defaultConfig) {
  const context = new Axios(defaultConfig); //  creates a new Axios instance
  const instance = bind(Axios.prototype.request, context); // creates a bound function using the bind method, This creates a function that will call Axios.prototype.request with the correct this context.

  // Copy axios.prototype to instance
  utils.extend(instance, Axios.prototype, context, { allOwnKeys: true });
  // Copy context to instance
  utils.extend(instance, context, null, { allOwnKeys: true });

  // Factory for creating new instances
  instance.create = function create(instanceConfig) {
    return createInstance(mergeConfig(defaultConfig, instanceConfig));
  };
  return instance; //  instance is both a function AND an object
}
```

</div>

---

# Basic Axios Usage - 🔍 Source Code

### axios is exported in <a href='https://github.com/axios/axios/blob/v1.x/lib/axios.js' target='_blank' class='hover:text-[#c5c3fb]!'>lib/axios.js</a>

1. `createInstance`
<div class='ml-6'>

`createInstance` function creates an Axios instance that functions both as a function and an object:

1. Creates a base `Axios` instance (context)
2. Binds `Axios.prototype.request` to context as the main request function
<div class='ml-6'>

- When calling `axios(config)`, it internally executes `Axios.prototype.request.bind(context)(config)`, making the main request function bound to the correct context
</div>

3. Extends instance to include:
<div class='ml-6'>

- Methods from Axios prototype chain (`get`, `post`, etc.)
- Configuration and state from context
</div>

4. Provides `create` method for new instances with merged custom configs

</div>

---

# Basic Axios Usage - 🔍 Source Code

### axios is exported in <a href='https://github.com/axios/axios/blob/v1.x/lib/axios.js' target='_blank' class='hover:text-[#c5c3fb]!'>lib/axios.js</a>

2. Create Default Axios Instance

<div class='ml-6'>

```js
const axios = createInstance(defaults);
```

- Creates an axios instance with default configuration, this is the instance we use directly

</div>

3. Extend Axios Instance with Additional Properties and Methods

<div class='ml-6'>

```js
// ...
axios.spread = spread;

// Expose isAxiosError
axios.isAxiosError = isAxiosError;

// Expose mergeConfig
axios.mergeConfig = mergeConfig;

axios.AxiosHeaders = AxiosHeaders;
// ..
```

</div>

---

# Basic Axios Usage - 🔍 Source Code

<br class='hidden' />

補充：從 [`index.d.ts`](https://github.com/axios/axios/blob/v1.x/index.d.ts) 看預設 axios instance 和 `axios.create` 回傳的 instance 差異

- Global axios is defined as `AxiosStatic`
<div class='ml-6'>

```js {*}{maxHeight:'250px'}
export interface AxiosStatic extends AxiosInstance {
  create(config?: CreateAxiosDefaults): AxiosInstance;
  Cancel: CancelStatic;
  CancelToken: CancelTokenStatic;
  Axios: typeof Axios;
  AxiosError: typeof AxiosError;
  HttpStatusCode: typeof HttpStatusCode;
  readonly VERSION: string;
  isCancel: typeof isCancel;
  all: typeof all;
  spread: typeof spread;
  isAxiosError: typeof isAxiosError;
  toFormData: typeof toFormData;
  formToJSON: typeof formToJSON;
  getAdapter: typeof getAdapter;
  CanceledError: typeof CanceledError;
  AxiosHeaders: typeof AxiosHeaders;
  mergeConfig: typeof mergeConfig;
}

declare const axios: AxiosStatic;
```

</div>

---

# Basic Axios Usage - 🔍 Source Code

<br class='hidden' />

補充：從 [`index.d.ts`](https://github.com/axios/axios/blob/v1.x/index.d.ts) 看預設 axios instance 和 `axios.create` 回傳的 instance 差異

- Instance returned by `axios.create` is defined as `AxiosInstance`

<div class='ml-6'>

```js
export interface AxiosInstance extends Axios {
  <T = any, R = AxiosResponse<T>, D = any>(config: AxiosRequestConfig<D>): Promise<R>;
  <T = any, R = AxiosResponse<T>, D = any>(url: string, config?: AxiosRequestConfig<D>): Promise<R>;

  defaults: Omit<AxiosDefaults, 'headers'> & {
    headers: HeadersDefaults & {
      [key: string]: AxiosHeaderValue
    }
  };
}
```

- `AxiosInstance` doesn't have methods like `isCancel()`, `isAxiosError()`, and can't access properties like `Cancel`, `CancelToken`, `HttpStatusCode`
  - `AxiosInstance` lacks the extension of the `axios` instance from step 3 above

</div>

---

# Basic Axios Usage - 🔍 Source Code

<br class='hidden' />

補充：從 [`index.d.ts`](https://github.com/axios/axios/blob/v1.x/index.d.ts) 看預設 axios instance 和 `axios.create` 回傳的 instance 差異

- `AxiosInstance` doesn't define a `create` method, so can the instance returned by `axios.create` call `create`?

  - In TypeScript type checking, using `AxiosInstance.create` will show an error
    <img src='/image/axiosInstance-create-TS.jpg'/>
  - In JavaScript runtime, using `AxiosInstance.create` still works

    ```js
    const instance1 = axios.create({...});
    const instance2 = instance1.create({...});
    instance2.get('/posts/1').then((res) => console.log(res.data)); // Successfully prints data
    ```

    <div class='mt-2'/>

    > The `createInstance` called by `axios.create` assigns a function to `instance.create`

---

```yaml
layout: cover
```

# Axios URL Encoding

---

# Axios URL Encoding

- What does Axios do for us?
  - The `params` object is automatically converted to query strings
  - Uses [`encodeURIComponent`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent) for encoding by default
  - Supports [`paramsSerializer`](https://github.com/axios/axios?tab=readme-ov-file#request-config) for customization

---

# Axios URL Encoding - axios 與 fetch 範例

<div class="grid grid-cols-[140px_1fr_240px] gap-x-4 mt4">

<div />

###### code

###### console

<v-clicks :every='3'>

<div class="my-auto leading-6 text-base opacity-75">
axios: ParamsInUrl
</div>

```js
function axiosParamsInUrl() {
  const search = 'hello world!';
  const symbol = '&$';
  const response = await axios.get(
    `${API_URL}/url-encoded?search=${search}&symbol=${symbol}`
  );
  console.log(response.data);
}
```

```js
receivedQuery: {
  search: 'hello world!',
  symbol: '',
  $: ''
}
```

<div class="my-auto leading-6 text-base opacity-75">
axios: AutoEncodeParams
</div>

```js
function axiosAutoEncodeParams() {
  const search = 'hello world!';
  const symbol = '&$';
  const response = await axios.get(`${API_URL}/url-encoded`, {
    params: { search, symbol },
  });
  console.log(response.data);
}
```

```js
receivedQuery: {
  search: 'hello world!',
  symbol: '&$'
}
```

</v-clicks>

</div>

---

# Axios URL Encoding - axios 與 fetch 範例

<div class="grid grid-cols-[140px_1fr_240px] gap-x-4 mt4">

<div />

###### code

###### console

<v-clicks :every='3'>

<div class="my-auto leading-6 text-base opacity-75">
fetch: WithoutEncode
</div>

```js
function fetchWithoutEncode() {
  const search = 'hello world!';
  const symbol = '&$';
  const res = await fetch(
    `${API_URL}/url-encoded?search=${search}&symbol=${symbol}`
  );
  const data = await res.json();
  console.log(data);
}
```

```js
receivedQuery: {
  search: 'hello world!',
  symbol: '',
  $: ''
}
```

<div class="my-auto leading-6 text-base opacity-75">
fetch: ManualEncode
</div>

```js
function fetchManualEncode() {
  const search = 'hello world!';
  const symbol = '&$';
  const res = await fetch(
    `${API_URL}/url-encoded?search=${encodeURIComponent(
        search
      )}&symbol=${encodeURIComponent(symbol)}`
  );
  const data = await res.json();
  console.log(data);
}
```

```js
receivedQuery: {
  search: 'hello world!',
  symbol: '&$'
}
```

</v-clicks>

</div>

---

# Axios URL Encoding - 🔍 Source Code

<br class='hidden'/>

axios URL 編碼在 [`lib/helpers/buildURL.js`](https://github.com/axios/axios/blob/v1.x/lib/helpers/buildURL.js)

### When does Axios call [`buildURL`](https://github.com/axios/axios/blob/v1.x/lib/helpers/buildURL.js)?

When you call `axios.get()`, here's what happens:

1. The request starts in the Axios class's` _request` method ([`lib/core/Axios.js`](https://github.com/axios/axios/blob/v1.x/lib/core/Axios.js)). This is the core method that handles all requests.
2. Inside `_request`, after handling interceptors, it calls `dispatchRequest` :
<div class='ml-6'>
```js {all|5}{maxHeight:'120px'}
// lib/core/Axios.js
_request(configOrUrl, config) {
    // ... 
    try {
      promise = dispatchRequest.call(this, newConfig);
    } catch (error) {
      return Promise.reject(error);
    }
    // ...
}
```
</div>

--- 


# Axios URL Encoding - 🔍 Source Code



### When does Axios call [`buildURL`](https://github.com/axios/axios/blob/v1.x/lib/helpers/buildURL.js)?
3. [`dispatchRequest`](https://github.com/axios/axios/blob/v1.x/lib/core/dispatchRequest.js) then gets the appropriate adapter (XHR for browsers, HTTP for Node.js) and calls it:

<div class='ml-6'>
```js {all|5}{maxHeight:'250px'}
// lib/core/dispatchRequest.js
export default function dispatchRequest(config) {
    // ...
    const adapter = adapters.getAdapter(config.adapter || defaults.adapter);
      return adapter(config).then(function onAdapterResolution(response) {
        throwIfCancellationRequested(config);
        // Transform response data
        response.data = transformData.call(
          config,
          config.transformResponse,
          response
        );
        response.headers = AxiosHeaders.from(response.headers);
        return response;
      }
    // ...
}
```
</div>

---

# Axios URL Encoding - 🔍 Source Code



### When does Axios call [`buildURL`](https://github.com/axios/axios/blob/v1.x/lib/helpers/buildURL.js)?
4. For browser environments, it uses the XHR adapter ([`lib/adapters/xhr.js`](https://github.com/axios/axios/blob/v1.x/lib/adapters/xhr.js)). The first thing the XHR adapter does is call `resolveConfig`


<div class='ml-6'>
```js {all|4}{maxHeight:'100px'}
// lib/adapters/xhr.js
function (config) {
  return new Promise(function dispatchXhrRequest(resolve, reject) {
    const _config = resolveConfig(config);
    // ...
});
}
```
</div>

5. [`resolveConfig`](https://github.com/axios/axios/blob/v1.x/lib/helpers/resolveConfig.js) is where the URL and params are processed. This is where your params get encoded:

<div class='ml-6'>
```js {all|4}{maxHeight:'100px'}
// lib/helpers/resolveConfig.js
export default (config) => {
// ...
    newConfig.url = buildURL(
      buildFullPath(newConfig.baseURL, newConfig.url, newConfig.allowAbsoluteUrls), 
      config.params, 
      config.paramsSerializer
    );
// ...
}
```
</div>


---

# Axios URL Encoding - 🔍 Source Code


### When does Axios call [`buildURL`](https://github.com/axios/axios/blob/v1.x/lib/helpers/buildURL.js)?

```js
axios.get()
    │
    ▼
Axios._request()
    │
    ▼
dispatchRequest()
    │
    ▼
xhrAdapter()
    │
    ▼
resolveConfig()
    │
    ▼
buildURL()  // This is where params are encoded
```

---

# Axios URL Encoding - 🔍 Source Code

### What does [`buildURL`](https://github.com/axios/axios/blob/v1.x/lib/helpers/buildURL.js) do?

- Purpose: takes a base URL and appends query parameters to it in a properly encoded format.
- Parameters:
    - `url`: The base URL (e.g., `"http://www.google.com"`)
    - `params`: An object containing the query parameters to append
    - `options`: Optional configuration for how to encode/serialize the parameters
        - Can be an object containing serialize function or encode function
        - Can also be directly passed as a serialize function

---

# Axios URL Encoding - 🔍 Source Code

### What does [`buildURL`](https://github.com/axios/axios/blob/v1.x/lib/helpers/buildURL.js) do?

```js {all|12-16|18|20-28|30-36}{maxHeight:'350px'}
/**
 * Build a URL by appending params to the end
 *
 * @param {string} url The base of the url (e.g., http://www.google.com)
 * @param {object} [params] The params to be appended
 * @param {?(object|Function)} options
 *
 * @returns {string} The formatted url
 */
export default function buildURL(url, params, options) {
  /*eslint no-param-reassign:0*/
  // If no parameters are provided, it returns the original URL unchanged.
  // This explains why parameters in axios.get(url) won't be encoded - they need to be in the params field of the axios config to be encoded.
  if (!params) { 
    return url;
  }

  const _encode = options && options.encode || encode; // Use options.encode if provided; otherwise use the built-in encode function

  if (utils.isFunction(options)) { // If options is passed as a function (e.g., custom serialize), it will be converted to { serialize: fn } format
    options = {
      serialize: options
    };
  } 

  const serializeFn = options && options.serialize;

  let serializedParams;

  if (serializeFn) {
    serializedParams = serializeFn(params, options); // If a custom serialize function is provided, use it to process params
  } else {
    serializedParams = utils.isURLSearchParams(params) ? // If params is a URLSearchParams instance, call .toString() directly - means developer has already transformed the params, axios doesn't need to process it
      params.toString() :
      new AxiosURLSearchParams(params, options).toString(_encode); // Use Axios's own AxiosURLSearchParams class to handle the params
  }

  if (serializedParams) { // 如果有序列化後的參數
    const hashmarkIndex = url.indexOf("#"); // 尋找 URL 中的 # 符號位置

    if (hashmarkIndex !== -1) { // 如果找到 # 符號，就移除 # 及其後面的內容
      url = url.slice(0, hashmarkIndex);
    }
    //  根據 URL 是否已有查詢參數來決定使用 ? 或 & 來連接新參數，例如 url 是 https://example.com/search?type=user 就是用 & 繼續加後面的參數
    url += (url.indexOf('?') === -1 ? '?' : '&') + serializedParams;
  }

  return url;
}
```

---

# Axios URL Encoding - 🔍 Source Code

### What does [`buildURL`](https://github.com/axios/axios/blob/v1.x/lib/helpers/buildURL.js) do?
- What does `AxiosURLSearchParams` do?
  - `AxiosURLSearchParams` is a custom class in Axios that handles the conversion of parameters into URL-encoded query strings

<div class='ml-6'>
```js {*}{maxHeight:'280px'}
/**
  * It takes a params object and converts it to a FormData object
  *
  * @param {Object<string, any>} params - The parameters to be converted to a FormData object.
  * @param {Object<string, any>} options - The options object passed to the Axios constructor.
  *
  * @returns {void}
  */
function AxiosURLSearchParams(params, options) {
  this._pairs = []; // 初始化 key-value pairs
  params && toFormData(params, this, options); // 如果有 params 就呼叫 toFormData，toFormData 會呼叫 append 將 key-pairs 加入 _pairs 陣列
}

const prototype = AxiosURLSearchParams.prototype;

prototype.append = function append(name, value) { // 定義 append 方法，將新的 key-value pair 新增到內部的 pairs 陣列
  this._pairs.push([name, value]);
};

prototype.toString = function toString(encoder) { // 定義 toString 方法，將所有 pairs 轉換為 URL-encoded string
  const _encode = encoder ? function(value) { // 如果 toString 參數有指定 encoder，就回傳此 custom encoder， encoder.call(this, value, encode) 允許 custom encoder 存取要 encode 的 value 和 axios 預設的 encode function
    return encoder.call(this, value, encode); 
  } : encode; // 沒有指定 encoder，就使用預設 encode function

  return this._pairs.map(function each(pair) { // 遍歷 pairs 陣列，對每個 pair 操作：Encodes the key (pair[0])、Adds an equals sign (=)、Encodes the value (pair[1])，最後用 & 連接每個 encoded pairs
    return _encode(pair[0]) + '=' + _encode(pair[1]);
  }, '').join('&');
};
```
</div>



---

# Code

Use code snippets and get the highlighting directly, and even types hover!

```ts {all|5|7|7-8|10|all} twoslash
// TwoSlash enables TypeScript hover information
// and errors in markdown code blocks
// More at https://shiki.style/packages/twoslash

import { computed, ref } from 'vue';

const count = ref(0);
const doubled = computed(() => count.value * 2);

doubled.value = 2;
```

<arrow v-click="[4, 5]" x1="350" y1="310" x2="195" y2="334" color="#953" width="2" arrowSize="1" />

<!-- This allow you to embed external code blocks -->

<<< @/snippets/external.ts#snippet

<!-- Footer -->

[Learn more](https://sli.dev/features/line-highlighting)

<!-- Inline style -->
<style>
.footnotes-sep {
  @apply mt-5 opacity-10;
}
.footnotes {
  @apply text-sm opacity-75;
}
.footnote-backref {
  display: none;
}
</style>

<!--
Notes can also sync with clicks

[click] This will be highlighted after the first click

[click] Highlighted with `count = ref(0)`

[click:3] Last click (skip two clicks)
-->

---

## level: 2

# Shiki Magic Move

Powered by [shiki-magic-move](https://shiki-magic-move.netlify.app/), Slidev supports animations across multiple code snippets.

Add multiple code blocks and wrap them with <code>````md magic-move</code> (four backticks) to enable the magic move. For example:

````md magic-move {lines: true}
```ts {*|2|*}
// step 1
const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery',
  ],
});
```

```ts {*|1-2|3-4|3-4,8}
// step 2
export default {
  data() {
    return {
      author: {
        name: 'John Doe',
        books: [
          'Vue 2 - Advanced Guide',
          'Vue 3 - Basic Guide',
          'Vue 4 - The Mystery',
        ],
      },
    };
  },
};
```

```ts
// step 3
export default {
  data: () => ({
    author: {
      name: 'John Doe',
      books: [
        'Vue 2 - Advanced Guide',
        'Vue 3 - Basic Guide',
        'Vue 4 - The Mystery',
      ],
    },
  }),
};
```

Non-code blocks are ignored.

```vue
<!-- step 4 -->
<script setup>
const author = {
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery',
  ],
};
</script>
```
````

---

# Components

<div grid="~ cols-2 gap-4">
<div>

You can use Vue components directly inside your slides.

We have provided a few built-in components like `<Tweet/>` and `<Youtube/>` that you can use directly. And adding your custom components is also super easy.

```html
<Counter :count="10" />
```

<!-- ./components/Counter.vue -->
<Counter :count="10" m="t-4" />

Check out [the guides](https://sli.dev/builtin/components.html) for more.

</div>
<div>

```html
<Tweet id="1390115482657726468" />
```

<Tweet id="1390115482657726468" scale="0.65" />

</div>
</div>

<!--
Presenter note with **bold**, *italic*, and ~~striked~~ text.

Also, HTML elements are valid:
<div class="flex w-full">
  <span style="flex-grow: 1;">Left content</span>
  <span>Right content</span>
</div>
-->

---

## class: px-20

# Themes

Slidev comes with powerful theming support. Themes can provide styles, layouts, components, or even configurations for tools. Switching between themes by just **one edit** in your frontmatter:

<div grid="~ cols-2 gap-2" m="t-2">

```yaml
---
theme: default
---
```

```yaml
---
theme: seriph
---
```

<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-default/01.png?raw=true" alt="">

<img border="rounded" src="https://github.com/slidevjs/themes/blob/main/screenshots/theme-seriph/01.png?raw=true" alt="">

</div>

Read more about [How to use a theme](https://sli.dev/guide/theme-addon#use-theme) and
check out the [Awesome Themes Gallery](https://sli.dev/resources/theme-gallery).

---

# Clicks Animations

You can add `v-click` to elements to add a click animation.

<div v-click>

This shows up when you click the slide:

```html
<div v-click>This shows up when you click the slide.</div>
```

</div>

<br>

<v-click>

The <span v-mark.red="3"><code>v-mark</code> directive</span>
also allows you to add
<span v-mark.circle.orange="4">inline marks</span>
, powered by [Rough Notation](https://roughnotation.com/):

```html
<span v-mark.underline.orange>inline markers</span>
```

</v-click>

<div mt-20 v-click>

[Learn more](https://sli.dev/guide/animations#click-animation)

</div>

---

# Motions

Motion animations are powered by [@vueuse/motion](https://motion.vueuse.org/), triggered by `v-motion` directive.

```html
<div
  v-motion
  :initial="{ x: -80 }"
  :enter="{ x: 0 }"
  :click-3="{ x: 80 }"
  :leave="{ x: 1000 }"
>
  Slidev
</div>
```

<div class="w-60 relative">
  <div class="relative w-40 h-40">
    <img
      v-motion
      :initial="{ x: 800, y: -100, scale: 1.5, rotate: -50 }"
      :enter="final"
      class="absolute inset-0"
      src="https://sli.dev/logo-square.png"
      alt=""
    />
    <img
      v-motion
      :initial="{ y: 500, x: -100, scale: 2 }"
      :enter="final"
      class="absolute inset-0"
      src="https://sli.dev/logo-circle.png"
      alt=""
    />
    <img
      v-motion
      :initial="{ x: 600, y: 400, scale: 2, rotate: 100 }"
      :enter="final"
      class="absolute inset-0"
      src="https://sli.dev/logo-triangle.png"
      alt=""
    />
  </div>

  <div
    class="text-5xl absolute top-14 left-40 text-[#2B90B6] -z-1"
    v-motion
    :initial="{ x: -80, opacity: 0}"
    :enter="{ x: 0, opacity: 1, transition: { delay: 2000, duration: 1000 } }">
    Slidev
  </div>
</div>

<!-- vue script setup scripts can be directly used in markdown, and will only affects current page -->
<script setup lang="ts">
const final = {
  x: 0,
  y: 0,
  rotate: 0,
  scale: 1,
  transition: {
    type: 'spring',
    damping: 10,
    stiffness: 20,
    mass: 2
  }
}
</script>

<div
  v-motion
  :initial="{ x:35, y: 30, opacity: 0}"
  :enter="{ y: 0, opacity: 1, transition: { delay: 3500 } }">

[Learn more](https://sli.dev/guide/animations.html#motion)

</div>

---

# LaTeX

LaTeX is supported out-of-box. Powered by [KaTeX](https://katex.org/).

<div h-3 />

Inline $\sqrt{3x-1}+(1+x)^2$

Block

$$
{1|3|all}
\begin{aligned}
\nabla \cdot \vec{E} &= \frac{\rho}{\varepsilon_0} \\
\nabla \cdot \vec{B} &= 0 \\
\nabla \times \vec{E} &= -\frac{\partial\vec{B}}{\partial t} \\
\nabla \times \vec{B} &= \mu_0\vec{J} + \mu_0\varepsilon_0\frac{\partial\vec{E}}{\partial t}
\end{aligned}
$$

[Learn more](https://sli.dev/features/latex)

---

# Diagrams

You can create diagrams / graphs from textual descriptions, directly in your Markdown.

<div class="grid grid-cols-4 gap-5 pt-4 -mb-6">

```mermaid {scale: 0.5, alt: 'A simple sequence diagram'}
sequenceDiagram
    Alice->John: Hello John, how are you?
    Note over Alice,John: A typical interaction
```

```mermaid {theme: 'neutral', scale: 0.8}
graph TD
B[Text] --> C{Decision}
C -->|One| D[Result 1]
C -->|Two| E[Result 2]
```

```mermaid
mindmap
  root((mindmap))
    Origins
      Long history
      ::icon(fa fa-book)
      Popularisation
        British popular psychology author Tony Buzan
    Research
      On effectiveness<br/>and features
      On Automatic creation
        Uses
            Creative techniques
            Strategic planning
            Argument mapping
    Tools
      Pen and paper
      Mermaid
```

```plantuml {scale: 0.7}
@startuml

package "Some Group" {
  HTTP - [First Component]
  [Another Component]
}

node "Other Groups" {
  FTP - [Second Component]
  [First Component] --> FTP
}

cloud {
  [Example 1]
}

database "MySql" {
  folder "This is my folder" {
    [Folder 3]
  }
  frame "Foo" {
    [Frame 4]
  }
}

[Another Component] --> [Example 1]
[Example 1] --> [Folder 3]
[Folder 3] --> [Frame 4]

@enduml
```

</div>

Learn more: [Mermaid Diagrams](https://sli.dev/features/mermaid) and [PlantUML Diagrams](https://sli.dev/features/plantuml)

---

foo: bar
dragPos:
square: 691,32,167,\_,-16

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos:
square: -114,0,0,0

---

dragPos: