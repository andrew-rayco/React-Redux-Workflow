# React-Redux-Workflow
A place for me to keep notes on setting up redux in a react project
reference:
http://redux.js.org/docs/api/createStore.html
the repo I'm working on which this note refers to:
https://github.com/cici-chen/Fifty-Shades-Of-Pink/tree/refactor
for workflow logic:
https://quickleft.com/blog/redux-plain-english-workflow/

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

### Make a component smart
copied from: https://quickleft.com/blog/redux-plain-english-workflow/
The notion is pretty straight-forward: if you want to wisen up your component, use the connect method from react-redux. Connecting your component gives it access to the global state object. It’s highly unlikely however, that our component needs access to everything in the state object. We can slice off only what we want the component to be aware of in the mapStateToProps function. The global state object will have various other keys in it, but in the example below, we’re only allowing questions through.

Create a container for the component & import the component in it

> in root/Client/Containers/StoryLibrary.js
```
import React from 'react'
import { connect } from 'react-redux';

import StoryLibrary from '../components/StoryLibrary.jsx'

const StoryLibrary = (props) => (
  <div>
    <StoryLibrary stories={props.stories}/>
  </div>
)

function mapStateToProps(state) {
   return { stories: state.stories };
}

export default connect(mapStateToProps)(StoryLibrary)
```
### Dispatch an action
Within the QuestionsPage component, we can use one of React’s component callbacks (i.e. componentWillMount) to initiate the fetching of questions. All smart components have access to the dispatch method via props.  The dispatch method allows us to execute an action creator, perform logic, and ultimately update the global state object.
> in root/client/containers/StoryLibrary.js
```
import { fetchStories } from '../actions'

const StoryLibrary = (props) => (
 ...
  componentWillMount(){
    props.dispatch(fetchStories())
  }
 ...
);
```
### Write the action creater
> in root/client/actions/index.js
```
import {getStories} from '../api/stories'

export function fetchStories(){
  return getStories()
}

export function receiveStories(stories){
  return{
    type:'RECEIVE_STORIES',
    stories
  }
}
```
### Test redux
reference:http://redux.js.org/docs/recipes/WritingTests.html
1. Install Jest testing engine
To use it together with Babel, you will need to install babel-jest:
```
npm install --save-dev jest babel-jest
```
2. Configure in package.json
> in root/package.json
Make sure you have
```
"babel": {
    "presets": [
      "es2015",
      "react"
    ]
  },
```
Mine already had es2015 and react set up in it, not sure when did that happen.
Add test scrtipt too
```
"scripts": {
    "test-jest": "jest'tests/redux/*.test.js'",
    "test:watch": "npm test -- --watch"
```

and run `npm run test-jest` to run it once, or `npm run test:watch` to test on every file change.

### Test Async Action Creators
For async action creators using Redux Thunk or other middleware, it's best to completely mock the Redux store for tests. You can apply the middleware to a mock store using redux-mock-store. You can also use nock to mock the HTTP requests.
My async action creator:
```
import {FETCH_STORIES_REQUEST,RECEIVE_STORIES} from './actionTypes'

import {getStories} from '../api/stories'

function fetchStoriesRequest(){
  return{
    type: FETCH_STORIES_REQUEST
    //Notice there's no quotation marks surrounding it because we are importing them from actionTypes so they are variables here. They turn into strings in actionTypes
  }
}

export function receiveStories(stories){
  return{
    type:RECEIVE_STORIES,
    stories
  }
}

export function fetchStories(){
  return dispatch => {
    dispatch(fetchStoriesRequest())
    return getStories(dispatch)
  }
}
```
Following redux document's recommendation, I exported all action types to another file. Which seems redundant but it doesn't hurt so why not.
> create root/client/actions/actionTypes.js
```
export const FETCH_STORIES_REQUEST = 'FETCH_STORIES_REQUEST'
export const RECEIVE_STORIES = 'RECEIVE_STORIES'

```
My test:
>create root/tests/redux/actions.test.js
```
import configureMockStore from 'redux-mock-store'
import thunk from 'redux-thunk'
import * as actions from '../../client/actions'
import * as types from '../../client/actions/actionTypes'
import nock from 'nock'
//I'm using jest here, so apparently I don't need to import the library

const middlewares = [ thunk ]
const mockStore = configureMockStore(middlewares)

const testStoriesInfo=[
  {
    id:1,
    name:"fifty-shades-of-pink"
  },
  {
    id:2,
    name:"hotel-room"
  }
]

describe('async actions', () => {
  afterEach(() => {
    nock.cleanAll()
  })

  it('create RECEIVE_STORIES upon FETCH_STORIES_REQUEST', () => {
    nock('http://localhost:80')
      .get('/api/v1/stories')
      .reply(200, testStoriesInfo)

    const expectedActions = [
      { type: types.FETCH_STORIES_REQUEST },
      //or you can just type the strings out here.
      { type: types.RECEIVE_STORIES, stories: testStoriesInfo }
    ]
    const store = mockStore({ stories: [] })

    return store.dispatch(actions.fetchStories())
    expect(store.getActions()).toEqual(expectedActions)
    //In the official document it was in a promise. But my fetchStories doesn't return a promise.So I didn't have to use .then
  })
})

```
### Test action creators
When testing action creators we want to test whether the correct action creator was called and also whether the right action was returned.
