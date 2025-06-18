# React Performance Optimization Guide

This guide covers low-level (code-focused) and high-level (architectural) performance improvements for React apps. It’s structured for both beginners and experienced developers.

---

## Low-Level Optimizations (Component Level)

### 1. Memoization with `React.memo`, `useMemo`, and `useCallback`
While these hooks help prevent unnecessary re-renders in React 18 and below, React 19 introduces an automatic memoization system that reduces the need for manual usage in many scenarios. However, for performance-critical or complex components, explicit memoization may still provide benefits.

```js
const ExpensiveComponent = React.memo(({ data }) => <div>{data.value}</div>);

const computedValue = useMemo(() => heavyCalc(input), [input]);
const handleClick = useCallback(() => doSomething(), []);
```

### 2. Avoid Anonymous Functions in JSX
```js
// Not ideal
<button onClick={() => doSomething()} />

// Recommended
const handleClick = useCallback(() => doSomething(), []);
<button onClick={handleClick} />
```

### 3. Proper Key Usage in Lists
```js
{items.map(item => <li key={item.id}>{item.name}</li>)}
```
Avoid using array index as a key.

### 4. Component Decomposition
Break large components into smaller ones to isolate renders.

### 5. Avoid Unnecessary State
Use refs for values that don’t trigger re-renders:
```js
const interval = useRef();
```

### 6. Throttle and Debounce Events
```js
const debounced = useCallback(debounce((val) => search(val), 300), []);
```

### 7. Virtualize Large Lists
```jsx
import { FixedSizeList as List } from 'react-window';

<List height={400} itemCount={1000} itemSize={35} width={300}>
  {({ index, style }) => <div style={style}>{items[index]}</div>}
</List>
```

### 8. Lazy Load Components
```js
const LazyComponent = React.lazy(() => import('./HeavyComponent'));

<Suspense fallback={<Loading />}>
  <LazyComponent />
</Suspense>
```

---

## High-Level Optimizations (App Level)

### 1. Production Builds
```bash
npm run build
```
Use minified, tree-shaken builds.

### 2. Code Splitting
```js
const AdminPanel = React.lazy(() => import('./AdminPanel'));
```
Split on route or component level.

### 3. Image Optimization
- Use WebP, AVIF formats
- `loading="lazy"` for non-visible images

### 4. Bundle Size Analysis
```bash
npm install --save-dev source-map-explorer
npx source-map-explorer 'build/static/js/*.js'
```

### 5. Optimize Library Imports
```js
// Not recommended
import _ from 'lodash';

// Recommended
import debounce from 'lodash/debounce';
```

### 6. SSR & SSG
Use Next.js for server-side rendering or static export.

### 7. Data Fetching & Caching
Use React Query or SWR:
```js
const { data } = useQuery('posts', fetchPosts);
```

### 8. Service Workers & PWA
Enable offline mode with:
```js
serviceWorker.register();
```

### 9. CDN for Assets
Deliver static files via CDN for reduced latency.

### 10. Tree Shaking
Use ES module versions like `lodash-es`, `date-fns` for better dead-code elimination.

---

## Benchmarking Tools

| Tool | Purpose |
|------|---------|
| React DevTools Profiler | Analyze component re-renders |
| Chrome DevTools | JS profiling, memory leaks |
| Lighthouse | Audit PWA, performance, accessibility |
| Webpack Bundle Analyzer | Visualize bundle size |
| Why Did You Render | Warn about unnecessary re-renders |

---

Happy optimizing!


***

***
# React-Performance for Class Based Components

  Please go through React Performance part 1 as well. 
 
1.Should React Update The Component?

2.Debounce Input Handlers

3.Memoize React Components

4.Avoid Async Initialization in componentWillMount()

5.Using Optimizing tools like  why-did-you-update

6.Beware of Object Literals in JSX

7.Problems with mutating the state directly 
---
### 1.Should React Update The Component?
when the application is growing, attempting to re-render and compare the entire virtual DOM at every action will eventually slow down.
React provides a weapon to take a call whether to update the component or not. This is where the shouldComponentUpdate method comes into play.
```
function shouldComponentUpdate(nextProps, nextState) {
    return true;
} 
```

When this function returns true for any component, it allows the render-diff process to be triggered.
This gives the you a simple way of controlling the render-diff process. Whenever you need to prevent a component from being re-rendered at all, simply return false from the function. Inside the function, you can compare the current and next set of props and state to determine whether a re-render is necessary:
```
function shouldComponentUpdate(nextProps, nextState) {
    return nextProps.id !== this.props.id;
}
```

##### Using a React.PureComponent
To ease and automate a bit this optimization technique, React provides what is known as “pure” component. A React.PureComponent is exactly like a React.Component that implements a shouldComponentUpdate()function with a shallow prop and state comparison.
A React.PureComponent is more or less equivalent to this:
```
class MyComponent extends React.Component {
    shouldComponentUpdate(nextProps, nextState) {
        return shallowCompare(this.props, nextProps) && shallowCompare(this.state, nextState);
    }
    …
}
```

It performs shallow comparison, So, It works perfectly with primitive data types, when coming to objects unless you avoid mutating the object, you can’t take full use of object. 
This will cause the render() function to create a new function on every render. 


 ---
### 2. Debounce Input Handlers
This concept is not specific to React, or any other front-end library.
 Debouncing has long been used in JavaScript to successfully run expensive tasks without hindering performance.

 A debounce function can be used to delay certain events so that it doesn’t get fired up every millisecond. This helps you limit the number of API calls, DOM updates, and time consuming tasks. 
For instance, when you’re typing something into the search menu, there is a short delay before all of the suggestions pop up. The debounce function limits the calls to onChange event handler.
 In many cases, the API request is usually made after the user has stopped typing.
Let me focus more on the debouncing strategy for input handlers and DOM updates. Input handlers are a potential cause of performance issues.

 Let me explain how:

##### Without Debounce

```
export default class SearchBar extends React.Component {
  constructor(props) {
    super(props);
    this.onChange = this.onChange.bind(this);
  }
  onChange(e) {
    this.props.onChange(e.target.value);
  }
  render () {
    return (
      <label> Search </label>
      <input onChange={this.onChange}/>
    );
  }
}
 ```
On every input change, we’re calling this.props.onChange(). What if there is a long-running sequence of actions inside that callback? That’s where debounce comes it handy. 
##### With Debounce:
```
import {debounce} from 'throttle-debounce';

export default class SearchBarWithDebounce extends React.Component {
  constructor(props) {
    super(props);
    this.onChange = this.onChange.bind(this);
    this.onChangeDebounce = debounce( 300, 
         value => this.props.onChange(value) //onChange will be called only once for 300 ms
    );
  }
  onChange(e) {
    this.onChangeDebounce(e.target.value);
  }
  render () {
    return (
      <input onChange={this.onChange}/>
    );
  }
}
 ```
The DOM update happens after the user has finished typing, which is exactly what we need. If you’re curious about debouncing AJAX calls, you can do something like this:
```
import {debounce} from 'throttle-debounce';

export default class Comp extends Component {
constructor(props) {
    super(props);
    this.callAPI = debounce(500, this.callAPI);onChange will be called only once for 500 ms
  }
  onChange(e) {
    this.callAPI(e.target.value);
  }
  callAjax(value) {
    console.log('value :: ', value);
    // AJAX call here
  }
  render() {
    return (
      <div>
        <input type="text" onKeyUp={this.onChange.bind(this)}/>
      </div>
    );
  }
}

```
---
### 3. Memoize React Components
Memoization is an optimization technique used primarily to speed up computer programs by storing the results of expensive function calls and returning the cached result when the same inputs occur again.
 A memoized function is usually faster because if the function is called with the same values as the previous one then instead of executing function logic it would fetch the result from cache.

Let's consider below simple stateless UserDetails React component.
```
const UserDetails = ({user, onEdit}) => {
    const {title, full_name, profile_img} = user;

    return (
        <div className="user-detail-wrapper">
            <img src={profile_img} />
            <h4>{full_name}</h4>
            <p>{title}</p>
        </div>
    )
}
 ```
Here, all the children in UserDetails are based on props. This stateless component will re-render whenever props changes. If the UserDetailscomponent attribute is less likely to change, then it's a good candidate for using the memoize version of the component:
```
import moize from 'moize';

const UserDetails = ({user, onEdit}) =>{
    const {title, full_name, profile_img} = user;

    return (
        <div className="user-detail-wrapper">
            <img src={profile_img} />
            <h4>{full_name}</h4>
            <p>{title}</p>
        </div>
    )
}

export default moize(UserDetails,{
    isReact: true
}); 
```
This method will do a shallow equal comparison of both props and context of the component based on strict equality.


If you are using React V16.6.0 or greater version, then you can use React.memo and rewrite the above code like:
```
const UserDetails = ({user, onEdit}) =>{
    const {title, full_name, profile_img} = user;

    return (
        <div className="user-detail-wrapper">
            <img src={profile_img} />
            <h4>{full_name}</h4>
            <p>{title}</p>
        </div>
    )
}

export default React.memo(UserDetails)
```
---
### 4. Avoid Async Initialization in componentWillMount()
componentWillMount() is only called once and before the initial render. Since this method is called before render(), our component will not have access to the refs and DOM element.
Here’s a bad example:
```
function componentWillMount() {
  axios.get(`api/comments`)
    .then((result) => {
      const comments = result.data
      this.setState({
        comments: comments
      })
    })}
```

Let’s make it better by making async calls for component initialization in the componentDidMount lifecycle hook:
```
function componentDidMount() {
  axios.get(`api/comments`)
    .then((result) => {
      const comments = result.data
      this.setState({
        comments: comments
      })
    })
} 
```
The componentWillMount() is good for handling component configurations and performing synchronous calculation based on props since props and state are defined during this lifecycle method

---
### 5.Using Optimizing tools like  why-did-you-update

The why-did-you-update library detects unnecessary component renders.

 Specifically, it points out instances where a component’s render method gets called even though no changes have occurred. 

A great way to monitor your application for avoidable renders is to install an npm package called why-did-you-update which will monitor and log avoidable component renders straight into your browser console at runtime.

### 6.Beware of Object Literals in JSX
Once your components become more "pure", you start detecting bad patterns that lead to useless rerenders.
 The most common is the usage of object literals in JSX, which is "The infamous {{". Let me give you an example:
 ```
import React from 'react';
import MyTableComponent from './MyTableComponent';

const Datagrid = (props) => (
    <MyTableComponent style={{ marginTop: 10 }}>
        ...
    </MyTableComponent>
)
```

The style prop of the <MyTableComponent> component gets a new value every time the <Datagrid> component is rendered. So even if <MyTableComponent> is pure, it will be rendered every time <Datagrid> is rendered.

 In fact, each time you pass an object literal as prop to a child component, you break purity. The solution is simple:
```
import React from 'react';
import MyTableComponent from './MyTableComponent';

const tableStyle = { marginTop: 10 };
const Datagrid = (props) => (
    <MyTableComponent style={tableStyle}>
        ...
    </MyTableComponent>
)
```
This looks very basic,You can see this mistake so many times that I've developed a sense for detecting the infamous {{ in JSX. I routinely replace it with constants.

Another usual suspect for hijacking pure components is React.cloneElement(). If you pass a prop by value as second parameter, the cloned element will receive new props at every render.
```
// bad
const MyComponent = (props) => <div>{React.cloneElement(Foo, { bar: 1 })}</div>;

// good
const additionalProps = { bar: 1 };
const MyComponent = (props) => <div>{React.cloneElement(Foo, additionalProps)}</div>;

```
### 7.Problems with mutating the state directly 
So what’s wrong with mutating state directly? 

Let’s say we overwrite shouldComponentUpdate and are checking nextState against this.state to make sure that we only re-render components when changes happen in the state.
```
shouldComponentUpdate(nextProps, nextState) {
    if (this.state.users !== nextState.users) {
      return true;
    }
    return false;
  }
```
Even if changes happen in the user's array, React won’t re-render the UI as it’s the same reference.
The easiest way to avoid this kind of problem is to avoid mutating props or state. 


The explanation is already given in Should Component Update. 




# Few points to consider by developers while writing react application 

1. Mandatory points to follow
2. Better to follow these as well

### Mandatory points to follow: 
1. Strictly, **Don’t mutate the object which is part of state**. If we Mutate, we are disturbing the react lifecycle. 
2. Use **only Unique keys while rendering the lists**(unique in the sense, unique among that particular list and should be same after every rerender, may be some id’s,..), If we don’t give unique values to the keys then you are not using the main feature of react for which facebook has created this library. 
3. which life cycle method is better to use for api calls? **ComponentDidMount**, Avoiding api calls in constructor or ComponentWillMount will reduce unnecessary re-render calls. 

4. Flux listener’s should be used appropriately, Using unwanted listeners will trigger the Updation lifecycle, Even Though React is intelligent enough not to change the dom, But we are executing too much javascript and creating Virtual doms and triggering the diffing algorithm. 
5. Proper use of Stateful, Stateless and PureComponents. Limit the use of Stateful Component and let’s not misuse the PureComponents. We should not neglect splitting of components, It’s plays a major role in performance, It will described in detail in the attached document.
 

6. Misuse of State variables. Including all the variables as part of state should not be done, Include only those variables in state that should be rendered, any other variables should never be part of state, It should be part of class variables or local scope to a function. 

7. Avoid inline CSS, This is the basic thing of all which we should not encourage. 


**If we couldn’t follow the above things, then we are misusing the react library. It’s like we are continuously hitting the tree without sharpening the axe.**


### Better to follow these as well
1. Using sprite images, This is a basic thing which we are still missing. Too many request to the server just for an icon will make the browser busy and wastage of infrastructure and troubling user’s bandwidth as well. 
 

2. Removing Unused imports, Even Though webpack doesn’t include them in the final bundle let’s import only those modules that are required and follow a pattern while importing. First import the external libraries then our generic components and then siblings or any other  components. 

3. Avoid binding the functions in render function. Use ES6 Arrow Keys or bind them in constructor. when you wan’t to pass parameters to the function, please curry the function and bind it. 

4. Use SetState carefully, For example you got some data from backend and you want to manipulate the data and show to the user. It’s better to manipulate the data first and then set it in the state, But mostly we are setting the state immediately while we get the data from the backend and then applying modification  on the state which triggers the updation lifecycle too much(Can be easily avoided).

5. Reducing the updation lifecycles. If we need 6 api calls to render a page, Then it’s better we update the state only after the mandatory api calls return the data. We can use axios or any other comfortable library to do the parallel calls and update the state when mandatory data is returned. 

6. Use Debounce/Throtlle to avoid unnecessery callback(Please use these functions especially in autosuggestions and scrolling, if not used you can't have smooth flow). Libraries like React virtualized help to create less no of Dom nodes when rendering huge no of nodes.

7. It’s better to avoid copying the props to state with out proper purpose. Instead you can directly use the props where required and use ComponentWillRecevieProps to find out the changes. 

8. Please use react-perf tool to calculate the performance of the project. There are lot more other tools like lightbox, profiling, memory,...provided by Google chrome developer tools. While calculating the performance please use an incognitive  mode so that other addons installed on browser won't effect the performance while calculating it.  



### Few honourable mentions 

1. Please use lodash and immutable.js. There are good functionalities provided by lodash, for ex to checking the values of objects when the chain is too long. we are using endless && operator which hampers the readability, instead we can use lodash or write our own lib. 

2. please use  eslint-plugin-react 
3. Code splitting(resource splitting and on demand code splitting)
4. CommonsChunkPlugin→  you can extract common code (such as all external libraries) into a “chunk” file of its own. 
5. Using ExtractTextWebpackPlugin, you can extract all CSS code into a separate CSS file.
This kind of splitting will help in two ways. It helps the browser to cache those less frequently changing resources. It will also help the browser to take advantage of parallel downloading to potentially reduce the load time.

Our code should be our documentation. :)



