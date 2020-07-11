```js
//actions.js
import ADD_TODO, TOGGLE_TODO, SET_VISIBILITY_FILTER from "types/actions.ts "

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
}

export function addTodo(text):ADD_TODO {
  return { type: ADD_TODO, text }
}

export function toggleTodo(index):TOGGLE_TODO {
  return { type: TOGGLE_TODO, index }
}

export function setVisibilityFilter(filter):SET_VISIBILITY_FILTER {
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
import Action from "types/actions.ts "
import {ToDos, Visibility, State} from "types/state.ts"
const { SHOW_ALL } = VisibilityFilters

function visibilityFilter(state = SHOW_ALL, action:Action):Visibility {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action:Action): Todos {
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

const todoApp:State = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

```js
// containers/FilterLink.js 
import { Dispatch } from "redux"
import { connect } from 'react-redux'
import { setVisibilityFilter } from '../actions'
import Link from '../components/Link'
import Action from "types/actions "
import state from "actions/state"

const mapStateToProps = (state:State, ownProps:{filter:string}) => ({
  active: ownProps.filter === state.visibilityFilter
})

const mapDispatchToProps = (dispatch:Dispatch<Action>, ownProps:{filter:string}) => ({
  onClick: () => dispatch(setVisibilityFilter(ownProps.filter))
})

export default connect(mapStateToProps, mapDispatchToProps)(Link)
```

```js 
// containers/ConnectedAddTodo.js using the useDispatch hook
import React from 'react'
import { useDispatch } from 'react-redux'
import { addTodo } from '../actions'
import Action from "types/actions.ts "

const ConnectedAddTodo = () => {
  const input = useRef()
  const dispatch = useDispatch<Action>()

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

```ts
// types/actions.ts 
import Visibility from "types/state.ts"

export interface AddTodoAction {
  type: "ADD_TODO"
  text: string
}

export interface ToggleTodoAction {
  type: "TOGGLE_TODO"
  index: number
}

export interface SetVisibilityFilter {
  type: "SET_VISIBILITY_FILTER"
  filter:  Visibility
}
export Action = AddTodoAction | ToggleTodoAction | SetVisibilityFilter
```

```ts
//types/state.ts
type Todo = {
	text: string 
	completed: boolean 
}

export type ToDos = Todo[]

export type Visibility = 'SHOW_ALL' | "SHOW_COMPLETED" | 'SHOW_COMPLETED'

export type State = {
	visibilityFilter: Visibility
	todos: ToDos
}
```
