# Redux patterns - Writing safe maintainable code just became blazing fast 
## Intro 

Redux recently celebrated 5 years of being released. The library was originally developed by Dan Abramov and shortly after became Javascript’s most popular state management library. 

IMHO Redux deserves the credit it receives from the community. It separates app state from UI elements, and organizes the most important parts of an application - app state and app logic - in a coherent way. At the same time it is composable and independent, and therefore can work with various frontend frameworks and environments. 

However, a common complaint is that Redux requires too much boilerplate and is too verbose and therefore is just not worth it. For this reason many avoid it entirely. Some use Mobx instead. 

But since Redux's release the JS community has progressed a great deal. The 2020 JS ecosystem isn’t anything like that of 2015. Static typing has greatly improved, proxies were standardized, new immutability tools were developed by the community, React introduced hooks, and much more. All those mean that we can use Redux differently. Perhaps even to redefine the way we think about Redux.  Not just a library which allows us to manage our state in a centralized predictable way, but one which could also be safe to use, fast to use, all while enjoying exceptional developer experience. 

## Part 1 - Redux + Typescript 

Redux is currently recognized as the gold standard for managing application state in JavaScript. But it is considered verbose / boilerplate oriented. 
 
Typescript on the other hand is the gold standard for writing Javascript. It provides two important benefits - code safety and better editor/developer experience. But again, some are reluctant to use TypeScript since it requires type definitions and hence more verbose code.

One could imagine that using both frameworks together would be exhausting to develop in. But it doesn't have to be this way. I suggest that TypeScript integrates perfectly with Redux (and Redux like patterns). Together they allow for code which isn’t just safe, organized and maintainable, but also fast to write (less code and better support from the editor). That is if we don't just add Typescript on top of our Redux code, but design our code according to the features Typescript has to offer.


### Traditional Redux pattern 
First let's take a look at a traditional redux pattern, by looking at Redux's official basic tutorial. 

```js
//actions.js
/*
 * action types
 */

export const ADD_TODO = 'ADD_TODO'
export const TOGGLE_TODO = 'TOGGLE_TODO'
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER'

/*
 * other constants
 */

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
}

/*
 * action creators
 */

export function addTodo(text) {
  return { type: ADD_TODO, text }
}

export function toggleTodo(index) {
  return { type: TOGGLE_TODO, index }
}

export function setVisibilityFilter(filter) {
  return { type: SET_VISIBILITY_FILTER, filter }
}
```

```js
// reducers.js
import { combineReducers } from 'redux'
import {
  ADD_TODO,
  TOGGLE_TODO,
  SET_VISIBILITY_FILTER,
  VisibilityFilters
} from './actions'
const { SHOW_ALL } = VisibilityFilters

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

```js
// containers/FilterLink.js 
import { connect } from 'react-redux'
import { setVisibilityFilter } from '../actions'
import Link from '../components/Link'

const mapStateToProps = (state, ownProps) => ({
  active: ownProps.filter === state.visibilityFilter
})

const mapDispatchToProps = (dispatch, ownProps) => ({
  onClick: () => dispatch(setVisibilityFilter(ownProps.filter))
})

export default connect(mapStateToProps, mapDispatchToProps)(Link)
```

```js 
// containers/ConnectedAddTodo using the useDispatch hook
import React from 'react'
import { useDispatch } from 'react-redux'
import { addTodo } from '../actions'

const ConnectedAddTodo = (props) => {
  const input = useRef()
  const dispatch = useDispatch()

  return (
    <div>
      <form
        onSubmit={e => {
          e.preventDefault()
          if (!input.value.trim()) {
            return
          }
          dispatch(addTodo(input.current.value))
        }}
      >
        <input ref={input} />
        <button type="submit">Add Todo</button>
      </form>
    </div>
  )
}
```

Our code handles 3 possible actions. One might say that it leans towards the verbose. And yet much could go wrong, here are some examples:
1. The reducer checking for an action.type that can't possibly be dispatched. 
```js
// reducers.js
function todos(state = [], action) {
  switch (action.type) {
    case SOME_UNRLATED_CONSTANT_OR_STRING:
		....
	.....
}
```
2. Mutating state 
```js
// reducers.js
function todos(state = [], action) {
  switch (action.type) {
    case REMOVE_TASK:
	 	return state.splice(...)
		....
	...
}
```

3. Breaking state structure 
```js
// reducers.js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return {
			text: action.text,
			completed: false
		}
		...
	...
  }
}
```
4. Dispatching a non action value 
```js
// containers/FilterLink.js 
const mapDispatchToProps = (dispatch, ownProps) => ({
  onClick: () => dispatch(Whatever)
})
```


Currently our code isn't type safe, isn't mutation safe, and we don’t get much support from the editor either.

We can solve those with TypeScript. But adding TypeScript on top of the current structure, while a common practice, means that our code will become even more verbose. Traditionally we'll have to add the following files (partially based on Redux official documentation https://redux.js.org/recipes/usage-with-typescript): 

```ts
// types/actions.ts 
interface AddTodoAction {
  type: "ADD_TODO"
  text: string
}

interface ToggleTodoAction {
  type: "TOGGLE_TODO"
  index: number
}

interface SetVisibilityFilter {
  type: "SET_VISIBILITY_FILTER"
  filter:  'SHOW_ALL' | "SHOW_COMPLETED" | 'SHOW_COMPLETED'
}
```

```ts
//types/state.ts
type Todo = {
	text: string 
	completed: boolean
}

type ToDos = Todo[]

type Visibility = 'SHOW_ALL' | "SHOW_COMPLETED" | 'SHOW_COMPLETED'
```

Then we need to add Types to our reducers, containers and actions creators. 

### Rethinking Typescript - Redux integration 
The first aspect of Redux's traditional pattern we should reconsider are actions. Action creators and action type constants (`const ADD_TASK = "ADD_TASK") are verbose and somewhat trivial. For projects written in vanilla JS they have an important role - standardizing the actions that can be thrown from a dispatcher on the one end and to be expected by a reducer on the other end. But when we have TypeScript in our tool chain, do we really need functions, which don’t have any functionality, just for standardizing?

TypeScript has a better tool for standardizing objects - types. Can we discard action creators altogether and remain with just a type? 

My experience is that it could have significant benefits. Let's see what it might look like. All action code can be replaced by the following: 
```ts
//actionType.ts
type Action =  {
	type: "ADD_TODO" // Typescript literal type
	text: string
} | {
	type: "TOGGLE_TODO"
	index: number
} | {
	type: "SET_VISIBILITY_FILTER"
	filter: "SHOW_ALL" | "SHOW_COMPLETED" | "SHOW_COMPLETED"
}
```
The most difficult half of our job is done - we have a type to represent an action. This type is at the core of the suggested pattern. Now that we have a type, we can use actions conveniently. Lets see what it could look like.

### Reducer - traditional pattern 
```ts
function todos(state = [], action:Action) {
  switch (action.type) {
    case ADD_TODO: // 
```
In the traditional pattern, since we use an identifier, the editor can't be of much help. We are on our own - we need to know exactly what identifier we are looking for for each `case`. 

### Reducer - Suggested pattern
```ts
function todos(state = [], action:Action) {
  switch (action.type) {
    case "ADD_TODO" :
```
As soon as we type `case "` the editor has our back / is right behind us. 
At this point Typescript knows that the only possible values are `"ADD_TODO"|"TOGGLE_TODO"|"SET_VISIBILITY_FILTER"` and the editor will offer those options, and no other option, as autocomplete. 
This is significant. Imagine a big project with over a hundred possible actions. Wouldn't it be helpful if the editor tells you what actions flow through the reducer, and you should consider?  

```ts
	case 'ADD_TODO' :
		action. // autoComplete will show that there is only one property - `text` of type string. 
```
`ADD_TODO` is a "literal type". Due to Typescript's "literal type" and "Discriminated Unions" features, at this point Typescript recognizes the exact structure of our action. 

### dispatcher - traditional pattern 
```ts
import addTodo from "actions"

const mapDispatchToProps = (dispatch:Dispatch, ownProps) => ({
  onClick: (text:string) => dispatch(addTodo (text)) 
})
```
- We can dispatch anything as long as it's an object. This means that we can also dispatch actions that our reducers aren't aware of and therefor doesn't handle. 
- We need to know in advance what identifier (`addToDo` in the example above) we are looking for. 


### dispatcher - suggested pattern 
All we need to do is let Typescript know that dispatch can only accept type Action. 

```js
// connected / container component form
const mapDispatchToProps = (dispatch:Dispatch<Action>, ownProps) => ({
  onClick: (text:string) => dispatch({  
    type: 'ADD_TODO', 
    text
  )
})

// useDispatch form
const useTodosDispatch = useDispatch as () => Dispatch<Action>

const ConnectedAddTodo = ({ dispatch }) => {
  ...
  const dispatch = useDispatch()

  const handleSubmit = (e) => {
    ...
    dispatch({ // Full type support. We can only dispatch a valid action, full support from the editor
      type: "ADD_TODO", 
      text: input.current.value
    })
  }}
  }
  ...
}
```
After typing dispatch we can only dispatch a valid action. Even if code is slightly longer, coding is much faster, since we get full support from the editor

### Possible caveats: 
1. If we decide to change action.type property (e.x from "ADD_TODO" to "APPEND_TODO") or the names of certain properties, we can't refactor it in one centralized place, since we don't have a const or a function. My experience so far is that it is a minor limitation/drawback. 
2. Changing our actions into a type doesn't go along with the popular Redux Thunk pattern. It's not a function, so an action can't dispatch an action. Later in this series I'll present a more efficient pattern IMO to handle side effects. 

-----------
[Link to our Redux code in the traditional pattern](https://github.com/carpben/coding_notes/blob/master/Redux%20patterns/traditional-pattern.md) 

[Link to our Redux code in the suggested pattern](https://github.com/carpben/coding_notes/blob/master/Redux%20patterns/suggestedPattern.md)

In terms of length, our redux files (actions, reducers and connected components) contains 21% less characters. In terms of development, it's a different experience. 

We finished the first and most significant step. We now have safe maintainable code which is shorter and faster to write. But we can further improve. In the next articles we'll look at adding Immer to the mix, typing based on implementation, and managing all side effects in a Redux like middleware. 

Until the next time, I'd love to read your comments and thoughts. 

