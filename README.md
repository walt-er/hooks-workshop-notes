# React Hooks Workshop

*"Don't think about Hooks in terms of classes. This is a whole new way of thinking about React" - James Long*

### React createElement
* params: tag name, props, child
* if  > 3 arguments, 3+ are converted to an array of children

### Style object
* camelcase is actually just the JS API for the style properties of a DOM node
* without mapping, React is less magical
* same goes for "className"

### useState
* think about calling use state as a language feature, not a function call
* useState returns an array to allow you to name your variables
* useState is just saving the previous values to an array outside the scope of the component
* useState just uses callorder to save state values by ID
    * all hooks use call order
* THAT'S why you can't use hooks conditionals, call order must be the same forever
    * if you need to do this, create a component that uses those hooks and wrap the component usage in a conditional instead

### useEffect
* an "Effect" is anything outside of rendering
* also uses call order to determine if new values for variables in dependency array are different
    * dependency array performs shallow comparison against provided values
    * no dependency array means always run on change
    * empty dependency array means only run once
* useEffect cleanup function
    * not just run on unmount, it's run whenever we go to run useEffect again
        * also stored in an array according to call order
        you can use closures in useEffect, use local variables

            ```jsx
                useEffect(() => {
                    let isValidAsyncRequest = true;
                    someAsyncNetworkCall()
                        .then(() => {
                            if (isValidAsyncRequest) {
                                ...
                            }
                        });
                    return () => { isValidAsyncRequest = false; };
                });
            ```

        * you can conditionally return cleanup functions
    * useEffect callback is synchronous, cannot await
        * because the cleanup function needs to be set immediately
        * inner asnyc functions within useEffect are fine
        * for quick actions, use promises
* useEffect (and all other hooks) can be broken out into separate functions, known as custom hooks, which allows them to be more flexible and therefore powerful
    * Abstract out useEffect (and useState) into custom hooks order to share with other components, and to provide more context around it's usage: useAppState(), useTitle()

### compound components
* dangers of adding too many features to a component
    * you can compare "god components" to a single function with a million params, adding params each time you need something, to see how overcomplicated components are dangerous
    * even if you hide it in an options object, it's too many conditionals
* instead, split functionality across many smaller components and share state with context
    * easier to swap out individual components

### context
* first use local state and props, then consider context
* context is especially useful for library authors, as they don't know what your children might be
* context methods are `createContext` (React method) and `useContext` (hook)
* anything can be passed into context
* after the context is created, a consumer anywhere below the provider in the component hierarchy can get that value
* user auth is a great candidate for context

### dealing with variable children
* React has a Children API: React.Children.map, React.Children.forEach, etc
    * `{React.Children.map(props.children, () => {})}`
* single children will be returned as a string instead of an array. make sure to handle this case

### useReducer
* facebook hired the redux guy, now reducers are native to React
* why use reducers over useState
    * setting multiple nuggets of state at once - we want to control the ways in which complex state changes can occur
    * more descriptive - dispatching a "SINGUP_FAILED" action is more clear than three lines of setState calls
* reducers definition (separate from React/Redux)
    * iterates over an array and runs some logic over each (accumulating) and updates current state
    * takes current state, takes type of action, applies value to current state according to type of action
    * reducer spreads current state and then makes changes
        * creating a new object with spread old state, does not mutate state
        * spreading old state ensures all untouched state properties remain
    * reducer returns old state when action is not recognized
        * alternatively, you can throw an error
* useReducer function
    * accepts 1) a reducer and 2) and object of initial values,
    * returns an array, array[0] is a `state` object and array[1] a `dispatch` function
        ```jsx
        const [state, dispatch] = useReducer(reducer, {
            error: null,
            loading: false,
            startDate: new Date("March 1, 2019")
        });
        ```
* share reducers/state across the app with context
    * provide return of useReducer as value of context provider (both current state and dispatch function)
* userReducer does about 90% of what you could do with React Redux
    * React Redux gives you more fine grained subscriptions to specific elements of your data store
    * useReducer will have your component rerender whenever any part of the state changes (!)
        * on the other hand React Redux only rerenders when specified state properties change
* writing reducer middleware
    * you just need a function that accepts the action, does its own thing, then returns `dispatch(action)`
    * set this custom dispatch function as the value of `dispatch` in your context, instead of providing `dispatch` from `useReducer` directly
    * this would allow you to do crazier things, like a middleware to handle Promises in `dispatch`
* optimistic updates
    * before waiting for state to update and build a new array of items to render, for example, go ahead and update the array directly so it updates right away
* one big benefit of using reducers is a single state object that can be persisted across loads

### animation
* three kinds of animations
    * interpolating between values
    * enter/exit a component
    * transitioning between elements
* two ways to interpolate
    * time based accoding to some easing
    * physics-based animations (React Spring)
* tween number example (interpolation between values)
    * check on a prop to use either raw total or total passed through new `useTween` hook
    * hook has internal state for acc value
    * adds to internal state until it reaches pased total
    * to isolate rerenders on internal hook state, move hook consumption to separate component
* exit animations
    * best practices??
        * react doesn't natively support any sort of callbacks that block unmount
    * a parent component is required to maintain a record of which components are coming in or out

### performance
* classes are slightly more performant than hooks, but in most cases this is likely negligible
    * [Walt: I'm not sure this is always true. I was confused by this conversation in the class because most arguments for hooks include a note about functional components being more performant than classes. I'd like to investigate what exactly he meant here]
* diffing the virtual React DOM and the real DOM is actually not inherently costly, it's the side effects that get ya
* memoization
    * old-school word for "caching something"
    * in React terms, it means saving the value of something for reuse instead of recalculating/rendering each time a component renders
* `useMemo`
    * takes a function which returns some value to memoize
    * arg two is a dependencies array, just like useEffect
    * you can memoize entire components so they only re-render if their props change
        * this is especially useful for long lists
* `useRef`
    * allows you to save any value to a persistent "bucket", which lives outside the component lifecycle/state, like a DOM node 
    * for example, use could use a ref to use the DOM Node API to control things like focus and scroll behavior in `useEffect` (since it is an "effect" outside of rendering
* React profiler
    * progress steps represent commit events
        * React "commits" a set of changes from its virtual DOM to the page DOM when a diff is found
    * as you step through commit events, gray bars are components that were not re-rendered

### misc
* React's virtual DOM is just a blueprint, or a description of component trees - could be rendered with ReactDOM, React Native, to ASCII, etc
* calling reactDOM render is the same as internal component rerender lifecycle
* imperitive = how
    * go get dom node, calculate value, set value
* declarative = what
    * value label = value state
* static analysis with the React linter runs on functions with `use` prefix
    * linter will warn you if you forgot a dependency
    * linter well let you know that some dependencies are unnecessary
* it's ok to be imperitive when there are no declarative alternatives (focus management)
* you can always return an array (of React componnts) from a JS expression
    * the third arg in React.createElement is already just an array of components
* e.target.elements = get form elements on submit
* `typeof` does not recognize Promise, use `instanceof` instead
* testing will likely be harder with hooks and should be at the component level
    * unit testing probably not feasible with hooks
* React dev tool profiler can record and see everything that re-rendered
* pattern for context is still being figured out, in prod it's recommended to use an established pattern like Redux
