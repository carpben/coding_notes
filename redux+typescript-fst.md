# Redux + Typescript - Writing good code could mean less boilerplate
# Redux + Typescript - Quality fast coding 
# Redux + Typescript - Writing good/quality code could actually mean less boilerplate 
# Redux + Typescript - Writing quality Javascript was never so fast 
# Redux + Typescript - Writing quality Javascript just became way faster
# Redux + Typescript + Immer - Writing safe maintainble code just became blazing fast 


Redux is rightly known as the gold standard for managing application state in Javascript. It separates app state from ui elements, and organizes the most important part of an application - app state and app logic in a coherent way. But many programmers complain that Redux requires too much boilerplate/ is too verbose, and is just not worth it. Many avoid it, some use Mobx instead. 

Typescript on the other hand is the gold standard for writing Javascript. It provides two important benefits - code safety and better editor/developer experience. But here again, some are reluctant to use Typescript since it requires type definitions, and hence - more verbose code. 

I suggest that Typescript integrates perfectly with Redux (and Redux like patterns). Together they allow for code which isn’t just safe organized and maintainable, but also lightning fast to write (less code and better support from the editor). That is if we don't just add Typescript on top of our Redux code,  but design our code according the the features Typescript has to offer. 

## Traditional Redux pattern 
First let's look at a traditional redux pattern, by looking at Redux official basic tutorial. 

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
import { connect } from 'react-redux'
import { addTodo } from '../actions'

const ConnectedAddTodo = ({ dispatch }) => {
  const input = useRef()
  const dispatch - useDispatch()

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

All this boilerplate for just 3 possible actions. All this boilerplate and we still haven't added Typescript, so our code isn't type safe, isn't mutation safe, and we doesn't get much support from the editor. Here are some examples for possible errors: 
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

5. And more 

We can solve those by adding Typescript. 
But adding Typescript on top of this structure, while a common practice, means that our code will become even more verbose. Traditionally we'll have to add the following files (partially based on Redux official documentation https://redux.js.org/recipes/usage-with-typescript): 

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

## Can we do things differently? Rethinking Typescript - Redux integration 
The first aspect/space/angle we should redsign are actions. Action creators and action types are verbose and somewhat trivial. In certain repositories/projects they might have an important rule - enforcing unity/uniformity from a dispatcher to the reducer. But when we have Typescript in our tool chain, do we really need action creators? Can we remove them all together and remain with just a type? 

A type can much better enforce uniformity between a dispatcher and a reducer than any constant or function. All action code can be replaced by the following: 
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

Lets see how `Action` can be used in our reducers. 
```ts
function todos(state = [], action:Action) {
  switch (action.type) {
    case "
```
At this point Typescript knows that the only possible values are `"ADD_TODO"|"TOGGLE_TODO"|"SET_VISIBILITY_FILTER"` and the editor should offer those options as autocomplete, and no other options are allowed. 
```ts
	case 'ADD_TODO' : 
```
`ADD_TODO` is a "literal type". Together with Typescript's "Discriminated Unions" feature, Typescript can at this point recognize the exact structure of you action. 
```ts
	case 'ADD_TODO' : 
		action. // autoComplete will show that there is only one field text of type string. 
```

The get the same benefits from the dispatcher side. All we need to do is let Typescript know that dispatch can only accept type Action. 
Personally I'd create a custom `useDispatch` hook, but there are other ways to go as well. 

```js
// containers/FilterLink.js 
const mapDispatchToProps = (dispatch:Dispatch<Action>, ownProps) => ({
  onClick: (text:string) => dispatch(  // After typing dispatch we can only dispatch a valid action
    type: 'ADD_TODO' // Even if code is slightly longer, coding is much faster, since we get full support from the editor
    text
  )
})
```

```js 
// containers/ConnectedAddTodo using the useDispatch hook

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
Look here to see how Redux code has shortened. We finished the first and most significant step. We now have safe maintainble code which is shorter and faster/quick to write. But we can further improve. In the next articles we'll look at adding Immer to the mix, and typing which is based on implentation rather than definitions.

Until the next time, I'd love to read your comments and thoughts. 











