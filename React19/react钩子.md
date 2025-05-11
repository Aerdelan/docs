<!--
 * @Author: wangyecheng 1874863790@qq.com
 * @Date: 2025-04-16 17:15:05
 * @LastEditors: wangyecheng 1874863790@qq.com
 * @LastEditTime: 2025-04-17 18:32:40
 * @FilePath: \docs\docs\React\ahooks.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->

## 功能性 hook

# useActionState 钩子

接收两个参数

```js
const [state, formAction, isPending] = useActionState(fn, initialState, permalink?);
```

state 为定义的原始数据，formAction 为操作函数，在 formAction 中对 state 进行修改。
可在 fn 中进行接口请求
可在 fn 中进行数据处理

formAction 为 fn 的回调数据

# useCallback

场景：用于父子组件之间切换，数据不更新需要重新渲染 dom 的场景
当使用 memo 方法来跳过组件重新渲染时

# useState

```js
const [count, setCount] = useState(0);
setCount(count + 1);
```

通过 setCount 改变定义的变量

用来声明可以直接更新的变量

# useReducer

```js
function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { ...state, count: state.count + 1 };
    case "decrement":
      return { ...state, count: state.count - 1 };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  function handleIncrement() {
    dispatch({ type: "increment" });
  }

  function handleDecrement() {
    dispatch({ type: "decrement" });
  }

  return (
    <div>
      <h1>{state.count}</h1>
      <button onClick={handleIncrement}>+</button>
      <button onClick={handleDecrement}>-</button>
    </div>
  );
}
```

在 reducer()中声明有更新逻辑的变量
可以对不同类型的的数据做不同的处理，以上案例中相当与使用 reducer 给 数据做了一个处，针对这个数据封装了一个数据处理的方法。

# \*useContext

用于将父节点的 ui 传给子节点使用

# useRef

用于保存 state 数据

# useEffect

结合了 componentDidMount、componentDidUpdate 和 componentWillUnmount 三个生命周期

一个文件中可以有多个 useEffect

模擬 componentDidMount（初始化） 當組件首次渲染時執行一次：
当依赖项为空数组时只会在组件挂载时执行一次（mounting）

```js
useEffect(() => {
  console.log("組件已掛載");
}, []); // 空依賴數組
```

模擬 componentDidUpdate（更新） 當依賴項發生變化時執行：

```js
useEffect(() => {
  console.log("依賴項變化");
}, [依賴項]); // 依賴項改變時觸發
```

模擬 componentWillUnmount（清理） 返回一個清理函數，用於組件卸載時執行：

```js
useEffect(() => {
  const timer = setInterval(() => {
    console.log("計時器運行中");
  }, 1000);

  return () => {
    clearInterval(timer); // 清理計時器
    console.log("組件已卸載");
  };
}, []);
```

監聽多個依賴項 當多個依賴項中的任意一個發生變化時執行：

```js
useEffect(() => {
  console.log("依賴項之一發生變化");
}, [依賴1, 依賴2]);
```

## 性能类 hook

# useMemo

```js
const memoizedValue = useMemo(() => {
  // 計算邏輯
  return 計算結果;
}, [依賴項]);
```

可以说是监听传入依赖项的变化，如果该依赖项变化则执行方法，否则返回之前缓存的值

和 watch 区别在于 useMemo 用于优化计算逻辑，watch 用来监听数据变化，而且 watch 没有返回值
