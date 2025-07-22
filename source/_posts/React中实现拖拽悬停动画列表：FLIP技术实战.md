---
title: React中实现拖拽悬停动画列表：FLIP技术实战
index_img: /img/flip.png
date: 2025-07-22 21:46:24
tags:
- javascript
- react
- flip
---
# React中实现拖拽悬停动画列表：FLIP技术实战

在现代 Web 应用中，用户期望流畅且直观的交互体验。对于可排序列表，拖拽功能是常见的需求，而当列表项在拖拽过程中实时交换位置并伴有平滑动画时，用户体验会得到显著提升。本文将深入探讨如何不依赖任何第三方动画库，仅使用 React Hooks 和原生浏览器 API，通过 FLIP (First, Last, Invert, Play) 技术实现这样一个动态列表。

我们将以一个具体的 React 组件为例，分析其实现细节。在这个组件中，用户可以拖动列表中的某一项，当它悬停在另一项上时，这两项会立即交换位置，并伴随丝滑的过渡动画。


## 什么是 FLIP 技术？

FLIP 是一种高性能的动画技术，尤其适用于处理因 DOM 操作（如添加、删除、排序）而引起的位置变化的元素。它的核心思想是：

1. **FIRST (F)**: 在任何改变发生之前，记录下元素在屏幕上的初始位置和尺寸。
2. **LAST (L)**: 执行 DOM 操作（例如，重新排序列表项），然后读取元素在屏幕上的最终位置和尺寸。
3. **INVERT (I)**: 使用 CSS Transform (`transform: translate()`, `scale()`) 立即使元素在视觉上“跳回”到它在 FIRST 步骤中记录的初始位置。由于 Transform 不会触发浏览器重排（reflow），这一步非常高效。此时，元素在 DOM 中的位置是新的，但在屏幕上看起来还在老位置。
4. **PLAY (P)**: 移除 INVERT 步骤中应用的 Transform，并添加 CSS Transition。浏览器会自动为元素从“伪装”的旧位置平滑过渡到其在 DOM 中的新自然位置创建动画。

这种方法的巧妙之处在于，它将复杂的动画计算简化为位置差值的计算，并利用了浏览器对 CSS Transform 和 Transition 的硬件加速优化。

## React 实现剖析

让我们深入研究一下 `AnimatedList` 组件的关键部分，看看它是如何运用 FLIP 技术的。

### 1. 状态管理与 Refs

组件需要管理几个关键状态：

- `items`: 存储列表项数据的数组。
- `draggedItemId`: 当前正在被拖拽的项目的 ID。
- `dragOverItemId`: 拖拽操作悬停在其上的项目的 ID，用于视觉反馈。
- `itemRefs`: 一个 React Ref 对象，用于存储对每个列表项 DOM 元素的引用。这对于后续读取元素的边界框至关重要。
- `prevBoundingBoxes`: 同样是一个 Ref 对象，用于存储在列表项顺序改变*之前*，每个项目的位置和尺寸信息 (`DOMRect` 对象)。这是 FLIP 中 "FIRST" 步骤的数据。
- `firstRender`: 一个 Ref，用于标记是否是首次渲染，以便在初始加载时不执行动画，仅记录初始位置。

```jsx
const [items, setItems] = useState(initialItems);
const itemRefs = useRef({});
const prevBoundingBoxes = useRef({});
const firstRender = useRef(true);

const [draggedItemId, setDraggedItemId] = useState(null);
const [dragOverItemId, setDragOverItemId] = useState(null);

// 确保每个 item 都有一个 ref
items.forEach(item => {
  if (!itemRefs.current[item.id]) {
    itemRefs.current[item.id] = React.createRef();
  }
});

```

### 2. 捕获初始状态 (FIRST)

FLIP 的第一步是在任何可能导致布局变化的操作之前捕获元素的当前状态。在我们的拖拽场景中，这个操作发生在 `handleDragEnter` 函数内部，即当一个被拖拽的项首次进入另一个项的区域，并且即将触发重新排序时：

```jsx
const getCurrentBoundingBoxes = () => {
  const currentBoxes = {};
  items.forEach(i => { // 注意：这里遍历的是当前状态的 items
    const node = itemRefs.current[i.id]?.current;
    if (node) {
      currentBoxes[i.id] = node.getBoundingClientRect();
    }
  });
  return currentBoxes;
};

// ... 在 handleDragEnter 中
if (draggedItemId && draggedItemId !== targetItemId) {
  // 关键：在更新 items 数组之前捕获当前所有项的位置
  prevBoundingBoxes.current = getCurrentBoundingBoxes();

  // ...后续的重新排序逻辑和 setItems(newItems)
}

```

同时，在组件首次渲染完成时，`useLayoutEffect` 也会调用 `getCurrentBoundingBoxes` 来初始化 `prevBoundingBoxes`。

### 3. 拖拽事件处理与列表重排

组件使用 HTML5 的拖拽 API (`draggable`, `onDragStart`, `onDragOver`, `onDragEnter`, `onDragLeave`, `onDrop`, `onDragEnd`) 来管理拖拽交互。

核心的重新排序逻辑位于 `handleDragEnter`：

```jsx
const handleDragEnter = (e, targetItemId) => {
  e.preventDefault();
  if (draggedItemId && draggedItemId !== targetItemId) {
    // 步骤 1: 捕获当前位置 (FLIP - FIRST)
    prevBoundingBoxes.current = getCurrentBoundingBoxes();

    const draggedItemIndex = items.findIndex(item => item.id === draggedItemId);
    const targetItemIndex = items.findIndex(item => item.id === targetItemId);

    if (draggedItemIndex === -1 || targetItemIndex === -1) return;

    const newItems = [...items];
    const [draggedElement] = newItems.splice(draggedItemIndex, 1);
    newItems.splice(targetItemIndex, 0, draggedElement);

    // 步骤 2: 更新状态，触发重新渲染 (这将间接触发 FLIP - LAST)
    setItems(newItems);
    setDragOverItemId(targetItemId); // 用于视觉指示
  }
};

```

当 `setItems(newItems)`被调用后，React 会重新渲染列表。此时，列表项在 DOM 中的顺序已经改变。

### 4. 执行动画 (LAST, INVERT, PLAY) - `useLayoutEffect` 的魔力

`useLayoutEffect` Hook 是实现 FLIP 动画的关键。它在 React 完成所有 DOM 变更*之后*，但在浏览器实际将这些变更绘制到屏幕*之前*同步运行。这使得我们有机会在浏览器绘制前读取新的布局信息，并立即应用反向变换。

```jsx
useLayoutEffect(() => {
  // 跳过首次渲染时的动画逻辑
  if (firstRender.current) {
    firstRender.current = false;
    prevBoundingBoxes.current = getCurrentBoundingBoxes(); // 仅记录初始位置
    return;
  }

  items.forEach(item => {
    const node = itemRefs.current[item.id]?.current;
    if (!node) return;

    // 步骤 3: 获取元素的新位置 (FLIP - LAST)
    const newBox = node.getBoundingClientRect();
    // 从 prevBoundingBoxes 获取旧位置 (在 handleDragEnter 中记录的)
    const oldBox = prevBoundingBoxes.current[item.id];

    if (oldBox) {
      const deltaX = oldBox.left - newBox.left;
      const deltaY = oldBox.top - newBox.top;

      // 如果位置有变化，则执行动画
      if (deltaX !== 0 || deltaY !== 0) {
        // 步骤 4: 反向变换 (FLIP - INVERT)
        // 立刻将项目“瞬移”回旧位置的视觉效果 (不带动画)
        node.style.transform = `translate(${deltaX}px, ${deltaY}px)`;
        node.style.transition = 'transform 0s';

        // 步骤 5: 播放动画 (FLIP - PLAY)
        // 使用 requestAnimationFrame 确保 "INVERT" 的样式已应用，
        // 然后再应用 "PLAY" 的样式以触发过渡。
        requestAnimationFrame(() => {
          node.style.transition = 'transform 0.3s ease-in-out'; // 添加过渡效果
          node.style.transform = ''; // 清除 transform，使其回到自然的新位置
        });
      }
    }
  });
  // 注意：这里 prevBoundingBoxes.current 不需要显式更新为 newBox，
  // 因为下一次 handleDragEnter 触发 setItems 之前，
  // getCurrentBoundingBoxes() 会重新捕获最新的布局作为新的 "FIRST" 状态。
}, [items]); // 依赖于 items 数组的变化

```

**代码解释：**

- **LAST**: `const newBox = node.getBoundingClientRect();` 获取了 DOM 更新后元素的新位置。
- **INVERT**: `node.style.transform = \`translate(${deltaX}px, ${deltaY}px)`; node.style.transition = 'transform 0s';`计算出新旧位置的差值，并通过`transform`将元素瞬间“拉回”到它视觉上的旧位置。关键在于`transition = 'transform 0s'`，确保这个“拉回”操作没有动画。
- **PLAY**: 在下一个动画帧 (`requestAnimationFrame`)，我们将 `transition` 设置为期望的动画效果（例如 `'transform 0.3s ease-in-out'`），然后将 `transform` 清空 (`node.style.transform = '';`)。浏览器现在会平滑地将元素从它被“拉回”的视觉位置动画到它在 DOM 中的实际新位置。

### 5. 视觉反馈

为了提升用户体验，组件还实现了一些视觉反馈：

- 被拖拽的项会变得半透明 (`opacity: isDragging ? 0.5 : 1`)。
- 当一个项被拖拽悬停在另一个项上时，目标项会显示一个虚线边框 (`border: \`2px ${isDragOver ? 'dashed #007bff' : 'solid transparent'}``)。

这些样式通过内联 `style` 属性根据 `draggedItemId` 和 `dragOverItemId` 状态动态应用。

## 总结与优势

通过手动实现 FLIP 技术，我们成功创建了一个高性能、交互流畅的拖拽排序列表，而无需引入额外的 JavaScript 动画库。这种方法的优势在于：

- **性能优异**: 主要依赖 CSS Transform 和 Transition，这些通常由 GPU 加速。
- **精细控制**: 开发者对动画的每个阶段都有完全的控制权。
- **轻量级**: 不增加额外的库依赖，减少了项目的包体积。
- **学习价值**: 理解 FLIP 技术有助于更深入地掌握浏览器渲染原理和动画优化。