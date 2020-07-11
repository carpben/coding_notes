# Redux + Typescript - Writing good code doesn’t have to mean more boilerplate 

# Redux and Typscript both require boilerplate, but when combining them 

# Redux + Typescript - A winning combination 

# Redux + Typescript - Quality fast coding 

# Redux + Typescript - Writing good/quality code could actually mean less boilerplate 
# Redux + Typescript - Best practices 
# Redux + Typescript - Writing quality Javascript was never so fast 
# Redux + Typescript - Writing quality Javascript just became way faster
# Redux + Typescript + Immer - Writing safe maintainble code just became blazing fast 


Redux is rightly known as the gold standard for managing application state in Javascript. Why? 
It separates app state from ui elements, and organizes the most important part of a front end application - app state and app logic. And it scales well. 
The complaint most users have is that it requires a lot of boilerplate. Due to this reason many avoid Redux, or try Mobx instead. 

Typescript on the other hand is the gold standart for writing good Javascript. It provides two huge/enourmous/essential benefits - code safety and better editing experience. But here again, some are reluctant to use Typescript since it requires type definitions, and hence - more boilerplate. 

But I suggest that Typescript integrates perfectly with Redux (and Redux like patterns), and by using the right patterns they allow us to write code which isn’t just safe organized and maintainable, but also lightning fast to write (less code and better support from the editor). 

## Traditional Redux pattern 
First let's look ata a traditional redux pattern, by looking at Redux official basic tutorial. 

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

All this boilerplate for just 3 possible actions. All this boilerplate and we still haven't added Typescript, so our code isn't type safe, isn't mutation safe, and we don't get any support from the editor. Here are some examples for possible errors: 
1. In the reducer checking for an action.type that can't possibly be dispatched. 
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

The way to solve this is adding Typescript. 
We can solve it by adding Typescript. 
But adding Typescript above/to this structure, while a common practice, means that our code will become even more verbose. Traditionally we'll have to add the following files (partially based on Redux official documentation https://redux.js.org/recipes/usage-with-typescript): 

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

Then we need to add Types to our reducers, containers and actions. 

## Can we do things differently? Rethinking Typescript - Redux integration 
The first aspect/space/angle we might be able to handle better are actions. Action creators and action types are verbose and somewhat trivial. In certain repositories/projects they might have an important rule - enforcing אחדות from a dispatcher to the reducer. But when we have Typescript in our tool chain, can we remove them all together and remain with just ... a type. 

A type can much better enforce unity between a dispatcher and a reducer than any constant or function. All action code will be replaced by the following: 
```ts
//actionType.ts
type Action =  {
	type: "ADD_TODO"
	text: string
} | {
	type: "TOGGLE_TODO"
	index: number
} | {
	type: "SET_VISIBILITY_FILTER"
	filter: "SHOW_ALL" | "SHOW_COMPLETED" | "SHOW_COMPLETED"
}
```

`ADD_TODO` is a "literal type". Together with Typescript's "Discriminated Unions" feature it allows for a very strong pattern. Lets see how we can use the in our reducers. 
```ts
function todos(state = [], action:Action) {
  switch (action.type) {
    case "
```
At this point Typescript knows that the only possible values are `"ADD_TODO"|"TOGGLE_TODO"|"SET_VISIBILITY_FILTER"` and the editor should offer those options as autocomplete, and no other options are allowed. 
```ts
	case 'ADD_TODO' : 
```
at this point Typescript knows the exact structure of you action. 
```ts
	case 'ADD_TODO' : 
		action. // autoComplete will show that there is only one field text of type string. 
```

The same principle applies to the dispatcher. All we need to do is let Typescript know that it can only accept type Action. 
Personally I'd create a custom `useDispatch` hook, but there are other ways to go as well. 

```js
// containers/FilterLink.js 
const mapDispatchToProps = (dispatch:Dispatch<Action>, ownProps) => ({
  onClick: (text:string) => dispatch(  // After typing dispatch we can only dispatch a valid action
    type: 'ADD_TODO' // Even if code is slightly longer, coding is much faster, since we get full support from the editor
    text
  )
})

export default connect(mapStateToProps, mapDispatchToProps)(Link)
```

```js 
// containers/ConnectedAddTodo using the useDispatch hook

const useTodosDispatch = useDispatch as () => Dispatch<Action>

const ConnectedAddTodo = ({ dispatch }) => {
  ...
  const dispatch = useDispatch()

  const handleSubmit = (e) => {
    e.preventDefault()
    if (!input.current.value.trim()) {
      return
    }
    dispatch({ // Full type support. We can only dispatch a valid action, full support from the editor
      type: "ADD_TODO", 
      text: input.current.value
    })
  }}
  }
  ...
}
```

We finished the first and most significant step. We now have safe maintainble code which is shorter and faster/quick to write. But we can do better. In the next articles we'll look at adding Immer to the mix, and typing which is based on implentation rather than definition. 











