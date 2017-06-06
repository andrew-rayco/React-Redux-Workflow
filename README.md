# React-Redux-Workflow
A place for me to keep notes on setting up redux in a react project
reference:
http://redux.js.org/docs/api/createStore.html
the repo I'm working on which this note refers to:
https://github.com/cici-chen/Fifty-Shades-Of-Pink/tree/refactor

### Install node modules
```
npm i --save-dev redux react-redux redux-thunk
```

### Create Store
To keep index.js in root/client pristine, we are creating a store file then import it into index.js, but we could have just added it in index.js
> in root/client/store.js
```
import {createStore, applyMiddleware, compose} from 'redux'
//There should only be a single store in your app
//{compose} is used to apply several store enhancers in a row
import thunkMiddleware from 'redux-thunk'

import reducers from './reducers'

export default function store = createStore(reducers, compose(
  applyMiddleware(thunkMiddleware),
  window.devToolsExtension ? window.devToolsExtension() : f => f
)) 
```
### Link Store to our app
> in root/client/index.js
```
import {Provider} from 'react-redux'
//Makes the Redux store available to the connect() calls in the component wrapped in them.
import store from './store'
```
Then in the part where you render your <App/>
```
document.addEventListener('DOMContentLoaded', () => {
  ReactDOM.render(
    <Provider store={store}>
      <App />
    </Provider>,
    document.getElementById('app')
  )
})
```
I wanna see the state of the store, so I added this in the file too
```
console.log('State of the entire app', store.getState())
```
