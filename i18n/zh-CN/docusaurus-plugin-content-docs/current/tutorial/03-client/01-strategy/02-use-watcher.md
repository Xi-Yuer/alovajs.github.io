---
title: 监听请求
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import EmbedSandpack from "@site/src/components/EmbedSandpack";
import CodeBlock from '@theme/CodeBlock';
import useWatcherSearchVue from '!!raw-loader!@site/codesandbox@3/02-client/02-use-watcher/vueComposition-search.zh.vue';
import useWatcherSearchReact from '!!raw-loader!@site/codesandbox@3/02-client/02-use-watcher/react-search.zh.jsx';
import useWatcherSearchSvelte from '!!raw-loader!@site/codesandbox@3/02-client/02-use-watcher/svelte-search.zh.svelte';
import useWatcherSearchSolid from '!!raw-loader!@site/codesandbox@3/02-client/02-use-watcher/solid-search.zh.jsx';

:::info 策略类型

use hook

:::

在一些需要随数据变化而重新请求的场景下，如分页、数据筛选、模糊搜索、tab 栏切换等，可以使用`useWatcher` 来监听指定的状态变化时立即发送请求。

## 示例

接下来我们以搜索 todo 项为例，尝试改变选择框中的选项，看看 todo 列表是如何变化的。
<Tabs groupId="framework">
<TabItem value="1" label="vue">

<EmbedSandpack template="vue" mainFile={useWatcherSearchVue} editorHeight={800} />

</TabItem>
<TabItem value="2" label="react">

<EmbedSandpack template="react" mainFile={useWatcherSearchReact} editorHeight={800} />

</TabItem>
<TabItem value="3" label="svelte">

<CodeBlock language="html">{useWatcherSearchSvelte}</CodeBlock>

</TabItem>
<TabItem value="4" label="solid">

<EmbedSandpack template="solid" mainFile={useWatcherSearchSolid} editorHeight={800} />

</TabItem>
</Tabs>

## 使用

:::tip 用法提示

useWatcher 支持 useRequest 的所有功能，详情请查看 [useRequest](/tutorial/client/strategy/use-request)。以下是 useWatcher 特有的用法。

:::

### 立即发送请求

与`useRequest`不同的是，`useWatcher`的`immediate`属性默认是`false`。

```javascript
const { send } = useWatcher(() => getTodoList(currentPage), [currentPage], {
  // highlight-start
  immediate: true
  // highlight-end
});
send();
```

### 请求防抖

通常我们都会在频繁触发的事件层面编写防抖代码，这次我们在请求层面实现了防抖功能，这意味着你再也不用在模糊搜索功能中自己实现防抖了，用法也非常简单。

:::info 什么是防抖

防抖（debounce），就是指触发事件后，在 n 秒内函数只能执行一次，如果触发事件后在 n 秒内又触发了事件，则会重新计算函数延执行时间（在这里和节流区分一下，节流是在触发完事件之后的一段时间之内不能再次触发事件）

:::

**设置所有监听状态的防抖时间**

```javascript
const { loading, data, error } = useWatcher(
  () => filterTodoList(keyword, date),
  [keyword, date],
  {
    // highlight-start
    // 设置debounce为数字时表示为所有监听状态的防抖时间，单位为毫秒
    // 如这边表示当状态keyword、date的一个或多个变化时，将在500ms后才发送请求
    debounce: 500
    // highlight-end
  }
);
```

**为单个监听状态设置防抖时间**

很多场景下，我们只需要对某几个频繁变化的监听状态进行防抖，如文本框的`onInput`触发的状态变化，可以这样做：

```javascript
const { loading, data, error } = useWatcher(
  () => filterTodoList(keyword, date),
  [keyword, date],
  {
    // highlight-start
    // 以监听状态的数组顺序分别设置防抖时间，0或不传表示不防抖
    // 这边监听状态的顺序是[keyword, date]，防抖数组设置的是[500, 0]，表示只对keyword单独设置防抖
    debounce: [500, 0]
    // 也可以这么按如下设置:
    // debounce: [500],
    // highlight-end
  }
);
```

### 请求时序

有时候当`useWatcher`监听的状态发生连续的改变导致连续的请求的发起时，后一次的请求先于前一次的请求获得响应，但是当前一次请求获得响应时，会覆盖后一次请求的响应，导致获取到与状态不匹配的响应；例如说有个状态`state`改变后发出了请求`1`，然后在请求`1`还未响应时又改变了`state`值，并发出了请求`2`，如果请求`1`后于请求`2`返回，最终的响应数据会维持在请求`1`。
所以我们设计了`abortLast`参数，它用于标记当下一次请求发出时，是否中断上一次的未响应请求，默认为`true`，这样`useWatcher`所发出的请求只有最后一次有效。

```mermaid
sequenceDiagram
  participant U as 用户
  participant S as 服务器
  U ->> U: 监听state状态
  U ->> S: state改变发起请求1
  U ->> S: state改变发起请求2
  S ->> U: 请求2先响应
  S ->> U: 请求1后响应
  U ->> U: 请求2的响应被覆盖
```

```javascript
useWatcher(
  () => getTodoList($currentPage),
  // 被监听的状态数组，这些状态变化将会触发一次请求
  [state],
  {
    // highlight-start
    abortLast: true // 是否中断上一次的未响应请求，默认为true
    // highlight-end
  }
);
```

:::warning 注意事项

`abortLast`默认为`true`，在正常情况下你不需要关注这个参数，如果修改为`false`，可能会导致状态与响应不匹配的问题。

:::

## API

请查看[API-useWatcher](/api/core-hooks#usewatcher)。
