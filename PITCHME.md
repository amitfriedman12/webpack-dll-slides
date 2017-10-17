# redux-little-router
<p style="text-align: center">A tiny router for Redux that lets the URL do the talking.</p>
---
redux-little-router makes URL state a first-class citizen of your Redux store and abstracts cross-browser navigation and routing into a pure Redux API.
### Main principals
* The URL is just another member of the state tree.
* URL changes are just plain actions.
* Route matching should be simple and extensible.
* While the core router does not depend on any view library, it provides flexible React bindings and components.
---
### What does it have going for it?
* Great url pattern matching.
* A single __LOCATION_CHANGED__ action dispatched for url changes.
* Lots of ways to change the url.
  * Action Creators
  * Link Component
  * PersistentQueryLink Component
+++
* Fragment Component.
  * Good: 
  ```<Fragment forRoute='/wef-wef'><SomeComponent {...someprops} /></Fragment>```
  * Bad:
  ```<Route path='/wef-wef' component={SomeComponent}/>```
  * Really?:
  ```<Route path='/wef-wef' children={() => (<SomeComponent {...someprops} />)}/>```
* Not locked into using components.
+++
* Repo Quality.
  * Strict semver.
  * Stable for prod. Incremental API changes only.
  * Router state slice always in sync.
* Flexible integration.
---
### Some Code
+++
#### Configuring Routes
```javascript
// routes.js
export const routes = {
  '/playa': {
    title: 'Home'
  },
  '/playa/facebook': {
    title: 'Facebook'
  },
  '/playa/snapchat': {
    title: 'Snapchat'
  },
  '/playa/twitter': {
    title: 'Twitter'
  },
  '/playa/pinterest': {
    title: 'Pinterest'
  }
}
// routes.js (better)
export const HOME = '/playa'
export const FACEBOOK = '/playa/facebook'
export const SNAPCHAT = '/playa/snapchat'
export const TWITTER = '/playa/twitter'
export const PINTEREST = '/playa/pinterest'
export const routes = {
  [HOME]: {
    title: 'Home'
  },
  [FACEBOOK]: {
    title: 'Facebook'
  },
  [SNAPCHAT]: {
    title: 'Snapchat'
  },
  [TWITTER]: {
    title: 'Twitter'
  },
  [PINTEREST]: {
    title: 'Pinterest'
  }
}
```
@[1-19](A simple example of a route config.)
@[20-44](Using constants is better though. You'll see why later.)
+++
#### Setting Up In The Store
```javascript
// config_store_dev.js
import { createStore, applyMiddleware, combineReducers, compose } from 'redux'
import createSagaMiddleware, { END } from 'redux-saga'
import { routerForBrowser, initializeCurrentLocation } from 'redux-little-router'
import routes from './routes'
import customMiddleware from './app-middleware'
import * as rootReducers from './reducers'
const { reducer, enhancer, middleware } = routerForBrowser({ routes })
const sagaMiddleware = createSagaMiddleware()
const composedEnhancers = compose(
  enhancer,
  applyMiddleware(
    sagaMiddleware,
    middleware,
    customMiddleware
  )
)
const combinedReducers = combineReducers({
  router: reducer,
  ...rootReducers
})
export default function configureStore (initialState) {
  const store = createStore(combinedReducers, initialState, composedEnhancers)
  store.initLocation = () => {
    const initialLocation = store.getState().router
    if (initialLocation) store.dispatch(initializeCurrentLocation(initialLocation))
  }
  store.runSaga = sagaMiddleware.run
  store.close = () => store.dispatch(END)
  return store
}
```
@[1-8](The imports.)
@[9-10](This creates the router reducer, enhancer, and middleware based on your route config.)
@[9-19](Set up other middleware and compose the router things with our things.)
@[20-24](Same thing for the reducers.)
@[25-37](The configure stor function.)
@[26-27](Nothing really changes here.)
@[28-37](Sometimes all that configuration isnt quite done before you start sagas and the router. We'll need controll over when things happen as our app grows.)
+++
#### Using It The Views
```javascript
// content_container.js
import React from 'react'
import { Fragment } from 'redux-little-router'
import * as ROUTES from 'routes'
import { subAppRoots } from 'sub_apps'
import SideNav from 'side_nav/side_nav'
import Styles from './content_container.css'
const noop = () => (null)
const { SnapChat = noop } = subAppRoots
const ContentContainer = () => (
  <div className={Styles.contentContainer}>
    Channel Content
    <Fragment forRoute={ROUTES.SNAPCHAT}>
      <SnapChat />
    </Fragment>
    <Fragment forRoute={ROUTES.FACEBOOK}>
      <h1>This is where the Facebook sub app will be.</h1>
    </Fragment>
    <Fragment forRoute={ROUTES.TWITTER}>
      <h1>This is where the Twitter sub app will be.</h1>
    </Fragment>
    <Fragment forRoute={ROUTES.PINTEREST}>
      <h1>This is where the Pinterest sub app will be.</h1>
    </Fragment>
    <SideNav />
  </div>
)
export default ContentContainer
```
@[1-32](Content Container File)
@[3-4](Imports specific to the router.)
@[15-17](This is an example using the Fragment component. Any children here will render when the route matches)
@[15-28](Easy as that. You can also show a Fragment when no routes are matched, like a 404 page. There is also a property to only show the Fragment when certain criteria is met in the location object.)
+++
#### Making the app do stuff
```javascript
// platform status item
export const PlatformStatusItem = ({ url, label, icon, status }) => (
  <Link
    href={url}
    activeProps={{ className: `${Styles.item} ${Styles.itemActive}` }}
    className={Styles.item}
  >
    <div className={Styles.itemContent}>
      <span className={Styles.icon}>{ icon }</span>
      <span className={Styles.name} data-field='name'>{label}</span>
      {status === true && <span className={`${Styles.status}`} />}
    </div>
  </Link>
)
```
@[1-15](The Link component has a few props so you can change the url in different ways.)
@[4-4](The href is just that. The url you want the router to go to.)
@[5-5](The activeProps is cool. If the route matches the href it will apply the props object you define. We won't need to sweat setting an active or selected flag anymore.)
---
---
# ...
---
### Now For Something Completely Different
The first react/redux app I made was a mess. I created an action, action creator, saga, and reducer for each api call in the app. It was a fairly small app but still had a ton of calls to make. Shit got crazy.
Luckily I had some time to fix it and then start on a completely different app that would be much larger.
---
### The Initial Plan
* **Action** for every api call.
* **Action Creator** for every api call.
* **Saga** for every api call.
* **Reducer** for every api call.
---
### What's Wrong With That?
You may already be familiar with how many files you need to create or modify just
to support a new api call. This is a single page app so there will be a lot of api
calls. Things get out of hand fairly quickly.
We end up with:
* More code.
* More opportunities to introduce bugs.
* More things to write tests for.
---
### How We Can Simplify
Look for ways to generalize all the things for the average api call. They all
work basically the same with some slight variation. After generlizing the code, clear
patterns will emerge.
---
#### Here's an example
Every api call functions basically the same. There's a request and it's either
successful or it fails.
```javascript
// action types
const API_CALL = 'API_CALL'
      API_CALL.REQUEST = 'API_CALL_REQUEST'
      API_CALL.SUCCESS = 'API_CALL_SUCCESS'
      API_CALL.FAILURE = 'API_CALL_FAILURE'
// action creators
const apiCall = {
  request: (payload) => ({ type: API_CALL.REQUEST, payload }),
  success: (payload) => ({ type: API_CALL.SUCCESS, payload }),
  failure: (payload) => ({ type: API_CALL.FAILURE, payload })
}
```
+++
#### Named object parameters
Named parameters are self documenting and pretty nice to work with when there are optional params.
```
// action creators with object params, too bad it's ugly
const apiCall = {
  request: ({ endpoint, params, ...rest }) => ({ type: API_CALL.REQUEST, payload: { endpoint, params, ...rest } }),
  success: ({ data, ...rest }) => ({ type: API_CALL.SUCCESS, payload: { data, ...rest } }),
  failure: ({ error, ...rest }) => ({ type: API_CALL.FAILURE, payload: { error, ...rest } }),
}
```
+++
#### Lets take it a step further
Because the patterns are easier to spot we can reduce boilerplate by creating some code to write
the code for us using factories. One to create the action types and one to create the actions.
```javascript
const createApiCallTypes = base => ({ REQUEST: `${base}_REQUEST`, SUCCESS: `${base}_SUCCESS`, FAILURE: `${base}_FAILURE` })
const action = (type, payload = {}) => ({ type, payload })
// action types
export const API_CALL = createApiCallTypes('API_CALL')
// action creators (with object params that aren't quite as ugly)
export const apiCall = {
  request: ({ endpoint, params = {}, ...rest }) => action(API_CALL.REQUEST, { endpoint, params, ...rest }),
  success: ({ data, ...rest }) => action(API_CALL.SUCCESS, { data, ...rest }),
  failure: ({ error, ...rest }) => action(API_CALL.FAILURE, { error, ...rest })
}
```
---
#### What about reducers?
Reducers check the action type to handle data for
different slices of the store. If this is all we did then each result for an api call would
overwrite the previous one in the store.
There are a few solutions...
+++
#### Option 1 : API Call Id
Use an id in the payload to help identify the specific call in each reducer. This works but
you still need to write a reducer for each response you are expecting.
Reducers would look something like this for the characters slice of the store.  
There would be a 1:1 ratio of api calls to reducers.
```javascript
export const characters = (state = null, action) => {
  const { type, payload } = action
  const { data, callId } = payload
  if (type === API_CALL.SUCCESS && callId === 'characters') {
    return data
  }
  return state
}
```
+++
#### Option 2 : Actions and Action Creators For All
Make actions and action creators for every api call. We could write more factories to
make all the action creators for each api call. They would all have unique types and unique
action creators. They would also have a reducer and saga for each api call.
+++
#### Option 3 : Key -> Value (winning)
Since we will likely want to store the raw data entities we get from each api call in the store we
can simply drop them into a key on an entities slice of the store. One reducer to rule them all.
With something like this we'll have a generic reducer to support our generic action things.
```javascript
export const entities = (state = {}, action) => {
  const { type, payload } = action
  const { data, key } = payload
  if (type === API_CALL.SUCCESS) {
    return state[key] = data
  }
  return state
}
```
---
#### You Forgot Sagas Dude!