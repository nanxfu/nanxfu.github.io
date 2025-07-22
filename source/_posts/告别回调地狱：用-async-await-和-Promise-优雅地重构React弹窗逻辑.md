---
title: 告别回调地狱：用 async/await 和 Promise 优雅地重构React弹窗逻辑
index_img: /img/react.png
date: 2025-07-22 21:43:32
tags:
- javascript
- react
---
在现代前端开发中，我们经常需要处理异步操作，尤其是在与用户交互时。一个典型的场景就是弹窗（Dialog/Modal）：我们弹出一个窗口，等待用户操作（例如点击“继续”或“取消”），然后根据用户的选择执行后续逻辑。当这个流程变得复杂，比如一个弹窗接着另一个弹窗时，我们很容易陷入“回调地狱”（Callback Hell）。

今天，我们就以一个真实的React项目代码为例，探讨如何利用 `async/await` 和 `Promise` 将一个深度嵌套的回调逻辑，重构为清晰、可读、可维护的线性代码。

### 问题剖析：我们面临的“回调地狱”

让我们先来看看重构前的代码。这段逻辑的目标是：在用户上传文件进行批量抓取时，根据不同的URL数量，弹出不同的确认弹窗组合。

**原始代码片段：**

```tsx
// ...
if (urls.length > 10 && urls.length < 2000) {
  cfg.use('dialog').show({
    content: (
      <NotificationModal
        onClose={onClose}
        onContinue={() => {
          cfg.use('dialog').close(); // 关闭第一个弹窗
          cfg.use('dialog').show({   // 弹出第二个弹窗
            content: (
              <WebScraperScrapingMethodModal
                scraperId={scraper.id}
                onClose={onClose}
                onContinue={() => {
                  cfg.use('dialog').close(); // 关闭第二个弹窗
                  onExecute(); // 执行最终操作
                }}
              />
            ),
          });
        }}
      />
    ),
  });
  return;
}
// ...

```

这段代码虽然能正常工作，但存在几个典型的问题：

1. **深度嵌套（The Pyramid of Doom）**：`onContinue` 回调函数里又包含了一个 `cfg.use('dialog').show`，形成了视觉上的“右倾金字塔”，可读性极差。
2. **责任链模糊**：外层 `NotificationModal` 的 `onContinue` 回调，承担了弹出 `WebScraperScrapingMethodModal` 的责任。这两个组件被紧密地耦合在了一起。
3. **难以维护和扩展**：如果未来需要在两个弹窗之间再增加一个，或者调整顺序，修改起来会非常痛苦，并且容易出错。
4. **重复的逻辑**：`cfg.use('dialog').close()` 这样的调用散落在各处，显得冗余。

### 重构目标：化繁为简，理清逻辑

我们的目标是消除回调嵌套，将异步的用户交互流程变得像写同步代码一样直观。我们希望最终的代码能像这样：

```tsx
// 伪代码
console.log('开始操作');
await showDialog('你确定吗？');
await showDialog('请选择一个方法');
executeFinalAction();
console.log('操作完成');

```

要实现这个目标，关键在于将基于回调的弹窗API，封装成返回 `Promise` 的函数。

### 一步步走向优雅：重构实战

### 第1步：核心思想 —— 将弹窗操作 Promise 化

`async/await` 是 `Promise` 的语法糖。因此，我们需要创建一个函数，它负责显示弹窗，并返回一个 `Promise`。这个 `Promise` 的状态由用户的操作来决定：当用户点击“继续”或“关闭”时，我们 `resolve` 这个 `Promise`，从而让 `await` 的等待结束，代码继续向下执行。

### 第2步：创建通用的异步弹窗函数

为了提高复用性，我们创建一个通用的 `showDialog` 函数。它接受一个React组件作为参数，并自动为其注入 `onClose` 和 `onContinue` 回调来控制 `Promise` 的状态。

```tsx
// 通用的异步弹窗函数
const showDialog = (
  Component: React.ComponentType<any>,
  props: any = {}
): Promise<{ continued: boolean }> => {
  return new Promise((resolve) => {
    cfg.use('dialog').show({
      content: (
        <Component
          {...props}
          onClose={() => {
            cfg.use('dialog').close();
            // 用户关闭了弹窗，我们认为是“未继续”
            resolve({ continued: false });
          }}
          onContinue={() => {
            cfg.use('dialog').close();
            // 用户点击了继续
            resolve({ continued: true });
          }}
        />
      ),
    });
  });
};

```

在这个函数中：

- 它返回一个 `Promise`，该 `Promise` 在未来的某个时刻会解析为一个对象 `{ continued: boolean }`。
- `Promise` 的 `resolve` 函数被传递给了弹窗组件的 `onClose` 和 `onContinue` 回调。
- 当用户与弹窗交互时，对应的回调被触发，`resolve` 被调用，`await` 的等待结束。我们通过 `continued` 标志位来判断用户是点击了“继续”还是直接关闭。

### 第3步：应用重构，享受线性逻辑

现在，我们可以用这个强大的 `showDialog` 工具来重写我们原来的业务逻辑了。

**重构后的代码：**

```tsx
// ...

// 将业务逻辑标记为 async 函数
const handleBulkScraping = async () => {
  // ... 其他逻辑 ...

  // check if the scrapeSourceType is UPLOAD_FILE ...
  if (
    crx.scraperMgr.getBulkScrapingSourceType() === IstWebScrapeSourceType.UPLOAD_FILE &&
    crx.scraperMgr.getScrapeBulkFileConfig()?.bulkUrlsFile?.name
  ) {
    if (urls.length === 5000) {
      cfg.use('toast').warn('超出最大限制，只爬取前5000条');
      const { continued } = await showDialog(NotificationModal);
      if (continued) {
        onExecute();
      }
      return;
    }

    if (urls.length > 10 && urls.length < 2000) {
      // 1. 等待第一个弹窗
      const { continued: step1Continued } = await showDialog(NotificationModal);

      // 如果用户在第一步就关闭了，则直接返回
      if (!step1Continued) return;

      // 2. 等待第二个弹窗
      const { continued: step2Continued } = await showDialog(
        WebScraperScrapingMethodModal,
        { scraperId: scraper.id }
      );

      // 3. 执行最终操作
      if (step2Continued) {
        onExecute();
      }
      return;
    }

    // 其他情况...
    const { continued } = await showDialog(NotificationModal);
    if (continued) {
      onExecute();
    }
  }

  // ...
};

// ...

```

### 总结：为什么这样更好？

对比一下重构前后的代码，优势显而易见：

1. **可读性（Readability）**：代码从上到下执行，逻辑清晰直观，就像一份操作手册。`await` 关键字明确地告诉我们“此处需要等待用户操作”。
2. **可维护性（Maintainability）**：
    - **修改顺序**：交换两个 `await showDialog(...)` 的位置即可。
    - **增加步骤**：在两个 `await` 之间插入一个新的 `await` 调用即可。
    - **删除步骤**：直接删除对应的 `await` 行即可。
3. **关注点分离（Separation of Concerns）**：`showDialog` 封装了处理异步和弹窗显示的底层细节。业务逻辑函数 `handleBulkScraping` 只关心“做什么”（业务流程），而不关心“怎么做”（弹窗的具体实现）。

通过将回调模式转换为 Promise-based 的 `async/await` 模式，我们不仅解决了“回调地狱”的问题，更让代码的健壮性和可维护性提升到了一个新的水平。这个模式可以广泛应用于任何基于回调的异步API，是每一位现代前端开发者都应该掌握的强大技巧。