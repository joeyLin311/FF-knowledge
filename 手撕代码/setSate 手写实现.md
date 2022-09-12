---
date created: 2022-07-01 16:50
---
#React #编程题 

```jsx
let workInProgressHook;
// 首次render是否为mount
let isMount = true;

// fiber 节点，App组件对应的fiber对象
const fiber = {
  // 保存该FunctionComponent对应的Hooks链表
  memoizedState: null,
  // 指向 App 函数
  stateNode: App
};

function schedule() {
  // workInProgressHook 指向当前工作的hook
  // 更新前将workInProgressHook重置为fiber保存的第一个Hook
  workInProgressHook = fiber.memoizedState;
  // 触发组件 render
  const app = fiber.stateNode();
  // 组件首次 render 为mount，以后再触发的更新为 update
  isMount = false;
  return app;
}

function dispatchAction(queue, action) {
  // 创建 update 
  const update = {
    action,
    next: null
  }
  // 环状单向链表操作
  if (queue.pending === null) {
    update.next = update;
  } else {
    update.next = queue.pending.next;
    queue.pending.next = update;
  }
  queue.pending = update;
  // 模拟 React 调度更新
  schedule();
}

function useState(initialState) {
  let hook;

  if (isMount) {
    // mount 时需要生成 hook 对象
    hook = {
      // 保存update的queue，即上文介绍的queue
      queue: {
        pending: null
      },
      // 保存 hook 对应的 state
      memoizedState: initialState,
      // 与下一个 hook 连接形成单向无环链表
      next: null
    }
    // 将hook插入fiber.memoizedState链表末尾
    if (!fiber.memoizedState) {
      fiber.memoizedState = hook;
    } else {
      workInProgressHook.next = hook;
    }
    // 移动workInProgressHook指针
    workInProgressHook = hook;
  } else {
    // update时从workInProgressHook中取出该useState对应的hook
    hook = workInProgressHook;
    // 移动 workInProgressHook 指针指向下一个fiber
    workInProgressHook = workInProgressHook.next;
  }
  // update执行前的初始state
  let baseState = hook.memoizedState;
  if (hook.queue.pending) {
    // ...根据queue.pending中保存的update更新state
    // 获取update环状单向链表中第一个update
    let firstUpdate = hook.queue.pending.next;

    do {
      // 执行update action
      const action = firstUpdate.action;
      baseState = action(baseState);
      firstUpdate = firstUpdate.next;

      // 最后一个update执行完后跳出循环
    } while (firstUpdate !== hook.queue.pending)

    // 清空queue.pending
    hook.queue.pending = null;
  }

  // 将update action执行完后的state作为memoizedState
  hook.memoizedState = baseState;

  return [baseState, dispatchAction.bind(null, hook.queue)];
}

function App() {
  const [num, updateNum] = useState(0);

  console.log(`${isMount ? 'mount' : 'update'} num: `, num);

  return {
    click() {
      updateNum(num => num + 1);
    }
  }
}

window.app = schedule();
```
