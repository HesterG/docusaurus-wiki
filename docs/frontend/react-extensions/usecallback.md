# useCallBack

## [掘金](https://juejin.cn/post/7074938135544594463)

* 无论使不使用 useCallback 每次渲染都是会新创建一个函数的，不会因依赖没有变化而减少，只是返回的函数是新创建的函数还是已经缓存下来的函数。
* 使用场景: 利用 memoize 减少不必要的子组件重新渲染，需要搭配 React.memo 或 shouldComponentUpdate 一起使用。

## [bilibili](https://www.bilibili.com/video/BV1uG411V7m3)

### useMemo

* 组件里的表每秒都在变，会导致组件每秒都在渲染，导致函数每次都被调用。也就是尽管input里的数字不变，因为时间变了，function每秒也在被call
*   使用memo来解决这个问题:

    useMemo takes two arguments:

    1. A chunk of work to be performed, wrapped up in a function
    2. A list of dependencies

    For every subsequent render, however, **React has a choice to make.** Should it:

    1. Invoke the function again, to re-calculate the value, or
    2. Re-use the data it already has, from the _last_ time it did this work.
*   另一种解决方式是把两个组件分开，但是有的情况还是需要状态提升,那么可以使用`React.memo`:

    ```
    ...
    // Transform our PrimeCalculator into a pure component:
    const PurePrimeCalculator = React.memo(PrimeCalculator);
    function App() {
      const time = useTime();

      // Come up with a suitable background color,
      // based on the time of day:
      const backgroundColor = getBackgroundColorFromTime(time);

      return (
        <div style={{ backgroundColor }}>
          <Clock time={time} />
          <PurePrimeCalculator />
        </div>
      );
    }
    ...
    ```

    Like a force field, `React.memo` wraps around our component and protects it from unrelated updates. Our `PurePrimeCalculator` will only re-render when it receives new data, or when its internal state changes.

    **A more conventional approach**

    In the example above, I'm applying `React.memo` to the _imported_ `PrimeCalculator` component.

    In truth, this is a bit unusual. I chose to structure it this way so that everything was visible in the same file, to make it easier to understand.

    In practice, I often apply `React.memo` to the component `export`, like this:

    ```
    // PrimeCalculator.js
    function PrimeCalculator() {
      /* Component stuff here */
    }

    export default React.memo(PrimeCalculator);
    ```
*   但是使用purecomponent有时候也会出现奇怪的事，也会render比较多，这就是`useMemo`可以解决的第二种问题

    boxes的例子中，每次`input`改变时，明明看起来没有改变Boxes, Boxes组件还是重新渲染了, 这是因为:

    The `Boxes` component only has 1 prop, `boxes`, and it appears as though we're giving it the exact same data on every render. It's always the same thing: a red box, a wide purple box, a yellow box. We _do_ have a `boxWidth` state variable that affects the `boxes` array, but we aren't changing it!

    **Here's the problem:** every time React re-renders, we're producing a _brand new array_. They're equivalent in terms of _value_, but not in terms of _reference_.

    也就是boxes array内容虽然都一样，但是reference不一样，也会触发Boxes的重新渲染

    ```
    // Every time we render this component, we call this function...
    function App() {
      // ...and wind up creating a brand new array...
      const boxes = [
        { flex: boxWidth, background: 'hsl(345deg 100% 50%)' },
        { flex: 3, background: 'hsl(260deg 100% 40%)' },
        { flex: 1, background: 'hsl(50deg 100% 60%)' },
      ];
      // ...which is then passed as a prop to this component!
      return (
        <Boxes boxes={boxes} />
      );
    }
    ```

    When the `name` state changes, our `App` component re-renders, which re-runs all of the code. We construct a brand-new `boxes` array, and pass it onto our `Boxes` component.

    **And `Boxes` re-renders, because we gave it a brand new array!**

    The _structure_ of the `boxes` array hasn't changed between renders, but that isn't relevant. All React knows is that the `boxes` prop has received a freshly-created, never-before-seen array.

    也就是每次name input改变，App组件会重新渲染，就创建了一个新的boxes array, 并传给Boxes组件, Boxes就会重新渲染, 因为这个array的reference变了

    To solve this problem, we can use the `useMemo` hook:

    ```
    const boxes = React.useMemo(() => {
      return [
        { flex: boxWidth, background: 'hsl(345deg 100% 50%)' },
        { flex: 3, background: 'hsl(260deg 100% 40%)' },
        { flex: 1, background: 'hsl(50deg 100% 60%)' },
      ];
    }, [boxWidth]);
    ```

### useCallBack

* **Here's the short version:** It's the exact same thing, but for _functions_ instead of arrays / objects.Similar to arrays and objects, functions are compared by reference, not by value:This means that if we define a function within our components, it'll be re-generated on every single render, producing an identical-but-unique function each time.
*   在例子中，点击click me也会触发megaboost组件的渲染

    Like we saw with the `boxes` array, the problem here is that we're generating a brand new function on every render. If we render 3 times, we'll create 3 separate `handleMegaBoost` functions, breaking through the `React.memo` force field.Using what we've learned about `useMemo`, we could solve the problem like this:

    ```
    const handleMegaBoost = React.useMemo(() => {
      return function() {
        setCount((currentValue) => currentValue + 1234);
      }
    }, []);
    ```

    Instead of returning an array, we're returning a _function_. This function is then stored in the `handleMegaBoost` variable.This works… but there's a better way:

    ```
    const handleMegaBoost = React.useCallback(() => {
      setCount((currentValue) => currentValue + 1234);
    }, []);
    ```

    `useCallback` serves the same purpose as `useMemo`, but it's built specifically for functions. We hand it a function directly, and it memoizes that function, threading it between renders.

    ```js
    // This:
    React.useCallback(function helloWorld(){}, []);

    // ...Is functionally equivalent to this:
    React.useMemo(() => function helloWorld(){}, []);
    ```
* **useCallback is syntactic sugar.** It exists purely to make our lives a bit nicer when trying to memoize callback functions.

### When to use these hooks

* The best way to use these hooks is in response to a problem. If you notice your app becoming a bit sluggish, you can use the React Profiler to hunt down slow renders. In some cases, you'll be able to improve performance by restructuring your application. In other cases, `useMemo` and `useCallback` can help speed things up.
*   #### Inside generic custom hooks

    One of my favourite lil’ custom hooks is `useToggle`, a friendly helper that works almost exactly like `useState`, but can only toggle a state variable between `true` and `false`:

    ```
    function App() {
      const [isDarkMode, toggleDarkMode] = useToggle(false);
      return (
        <button onClick={toggleDarkMode}>
          Toggle color theme
        </button>
      );
    }
    ```

    Here's how this custom hook is defined:

    ```
    function useToggle(initialValue) {
      const [value, setValue] = React.useState(initialValue);

      const toggle = React.useCallback(() => {
        setValue(v => !v);
      }, []);

      return [value, toggle];
    }
    ```

    Notice that the `toggle` function is memoized with `useCallback`.

    When I build custom reusable hooks like this, I like to make them as efficient as possible, because **I don't know where they'll be used in the future.** It might be overkill in 95% of situations, but if I use this hook 30 or 40 times, there's a good chance this will help improve the performance of my application.
*   #### Inside context providers

    When we share data across an application with context, it's common to pass a big object as the `value` attribute.

    It's generally a good idea to memoize this object:

    ```
    const AuthContext = React.createContext({});

    function AuthProvider({ user, status, forgotPwLink, children }){
      const memoizedValue = React.useMemo(() => {
        return {
          user,
          status,
          forgotPwLink,
        };
      }, [user, status, forgotPwLink]);

      return (
        <AuthContext.Provider value={memoizedValue}>
          {children}
        </AuthContext.Provider>
      );
    }
    ```

    **Why is this beneficial?** There might be dozens of pure components that consume this context. Without `useMemo`, all of these components would be forced to re-render if `AuthProvider`'s parent happens to re-render.
