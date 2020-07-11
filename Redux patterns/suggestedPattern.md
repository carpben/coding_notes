```ts
// actions.ts
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

```js
// reducers.js

import { combineReducers } from 'redux'
import Action from "types/actions.ts "
import {ToDos, Visibility, State} from "types/state.ts"
const { SHOW_ALL } = VisibilityFilters


function visibilityFilter(state = SHOW_ALL, action:Action):Visibility {
  switch (action.type) {
    case "SET_VISIBILITY_FILTER":
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action:Action): Todos {
  switch (action.type) {
    case "ADD_TODO":
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case "TOGGLE_TODO":
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

const todoApp: State = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

```js
// containers/FilterLink.js 
import { Dispatch } from "redux"
import { connect } from 'react-redux'
import Link from '../components/Link'
import Action from "types/actions"
import State from "teyps/state"

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
import Action from "types/actions.ts "

const ConnectedAddTodo = () => {
  	const input = useRef()
  	const dispatch = useDispatch<Action>()
	const handleSubmit = (e) => {
		e.preventDefault()
		if (!input.value.trim()) {
			return
		}
   	dispatch({ // Full type support. We can only dispatch a valid action, full support from the editor
      	type: "ADD_TODO", 
      	text: input.current.value
    	})
  	}}        
  }

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input ref={input} />
        <button type="submit">Add Todo</button>
      </form>
    </div>
  )
}
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