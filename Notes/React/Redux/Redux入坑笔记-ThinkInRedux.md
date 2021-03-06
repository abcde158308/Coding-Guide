<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Redux入坑笔记-Think in Redux](#redux%E5%85%A5%E5%9D%91%E7%AC%94%E8%AE%B0-think-in-redux)
  - [定义常量、state和Action](#%E5%AE%9A%E4%B9%89%E5%B8%B8%E9%87%8Fstate%E5%92%8Caction)
  - [定义Reducer](#%E5%AE%9A%E4%B9%89reducer)
  - [combineReducers](#combinereducers)
  - [注入到React](#%E6%B3%A8%E5%85%A5%E5%88%B0react)
  - [子组价获取props和dispatch](#%E5%AD%90%E7%BB%84%E4%BB%B7%E8%8E%B7%E5%8F%96props%E5%92%8Cdispatch)
  - [优势](#%E4%BC%98%E5%8A%BF)
  - [UPDATE](#update)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Redux入坑笔记-Think in Redux

> Redux 简要的说就是一个事件分发器和全局`state`控制台。

Redux 有一个全局的`state`，通过将根组件包进`Provider`，将`store`分发给所有的子组件，而子组件通过`connect`方法，获取`dispatch`事件分发函数，以及需要的`props`(如果有需要也可以通过`connect`传入想分发给子组件的`action`)

### 定义常量、state和Action

```
// Reducer/ConstValue.js
export const ADD_NEW = 'add_new';
export const INCREASE = 'increase';
export const DECREASE = 'decrease';
export const DELETE = 'delete';
export INITIAL_STATE = {
	objList: [
		{
			number: 0
		}
	]
};

// Reducer/ActionCreator.js
import {
	ADD_NEW,
	DELETE
	INCREASE,
	DECREASE
} from './ConstValue';

export function addNew(obj) {
	return {
		type: ADD_NEW,
		obj
	}
}

export function delete(index) {
	return {
		type: DELETE,
		index
	}
}

export function increase(number, index) {
	return {
		type: INCREASE,
		number,
		index
	}
}

export function decrease(number, index) {
	return {
		type: DECREASE,
		number,
		index
	}
}
```

### 定义Reducer

> Reducer 会注册给 store,用于处理 action 事件

```javascript
// Reducer/Reducers.js
import {
	ADD_NEW,
	DELETE
	INCREASE,
	DECREASE,
	INITIAL_STATE
} from './ConstValue';

let objList = INITIAL_STATE.objList;

export function objList(state=objList, action) {
	switch (action.type) {
		case ADD_NEW:
			return [...state, action.obj];
		case DELETE:
			return [...state.slice(0, action.index),
				...state.slice(action.index + 1)
			];
		case INCREASE:
		case DECREASE:
			return [...state.slice(0, action.index),
				Object.assign({}, state[action.index], {number: number(state[action.index].number, action)}),
				...state.slice(action.index + 1)
			];
		default:
			return state;
	}
}

export function number(state=objList[0].number, action) {
	switch (action.type) {
		case INCREASE:
			return state + action.number;
		case DECREASE:
			return state - action.number;
		default:
			return state;
	}
}
```

**IMPORTANT**

Reducer 传入(state, action), 通过判断`action`的`type`, 进行事件分发处理。当事件很多的时候可以把 Reducer 拆分, 最后通过`combineReducers`进行组合。

每个 Reducer 都要有明确的返回值,当`siwtch`到`default`的时候则返回传入的`state`本身。在处理`state`的时候，不要在原有参数上修改, 而应该返回一个新的参数, 例如

```javascript
return Object.assign({}, state[0], {number: action.number});
// 通过Object.assign获取一份state[0]的拷贝, 并修改state[0]中的number数据

return [...state, action.obj];
// 通过[...XXX]得到一个新的数组, 并在最后加入action.obj
```

### combineReducers

```javascript
// Reducer/index.js
import { combineReducers } from 'redux';

import * as appReducers from './Reducers';

export const AppReducer = combineReducers(appReducers);
```

上述代码中, 通过`import * as appReducers`导出全部的 Reducer, 之后利用`combineReducers`黑魔法快速的组合 Reducer 

**黑魔法发动条件**

每个 Reducer 的名称, 必须与它获取的 state 参数名称一样, 例如:

```javascript
export function objList(state=objList, action){}

export function number(state=objList[0].number, action){}
```

如果你任性的不想那么写, 那么就要:

```javascript
// Reducer/Reducers.js
export function reducerA(state=objList, action){}
export function reducerB(state=objList[0].number, action){}

// Reducer/index.js
import { combineReducers } from 'redux';
import {
	reducerA
	reducerB,
} from './Reducers';

export function AppReducer(state, action) {
	return {
		objList: reducerA(state.objList, action),
		number: reducerB(state.objList[0].number, action)
	}
}
```

### 注入到React

```scss
// 安装React绑定库
sudo npm install --save react-redux
```
在根组件上, 通过`Provider`注入store

**只有一个Store**

```javascript
// App/index.js
import { Provider } from 'react-redux';
import { createStore } from 'redux';
import {
  AppReducer
} from '../Reducer/index';
import RootComponent from './RootComponent';

let appStore = createStore(AppReducer);

render(
  <Provider store={appStore}>
    <RootComponent />
  </Provider>,
    document.getElementById('index_body')
);

// App/RootComponent.js
import ContentComponent from './ContentComponent';

class RootComponent extends Component {
	render() {
		return (
			<div>
				<ContentComponent />
			<div>
		)
	}
}

export default ContentComponent;
```

注入`store`后, 根组件所有的组件都可以获取到`dispatch`函数, 以便进行`action`的分发处理

### 子组价获取props和dispatch

```javascript
// App/ContentComponent.js
import {
	increase,
	decrease
} from '../Reducer/ActionCreator';

class ContentComponent extends Component {
	render() {
		const {obj, increaseNumber, decreaseNumber}
		return (
			<div>
				<span 
				  onClick={() => {
				  	increaseNumber(1);
				  }}
				 >加一</span>
				<span>{obj.number}</span>
				<span
				  onClick={() => {
				  	decreaseNumber(1);
				  }}
				>减一</span>
			</div>
		)
	}
}

// 定义props筛选函数, 以state作为传入参数, 选出需要注入该组件作为props的state. 不是必须, 不写则state作为props全部注入
const mapStateToProps = (state) => {
	return {
		obj: state.objList[0]
		// objList: state.objList 
		// 本来想写list增删逻辑的但是太懒了暂时搁浅..
	}
}

// 定义action筛选函数, 以dispatch作为传入参数, 选出需要注入该组件需要使用的action. 不是必须
const mapDispatchToProps = (dispatch) => {
	return {
		increaseNumber: (number) => {
			dispatch(increase(number));
		},
		decreaseNumber: (number) => {
			dispatch(decrease(number));
		}
	}
}

import { connect } from 'react-redux';
export default connect(mapStateToProps, mapDispatchToProps)(ContentComponent);
// 不使用筛选函数的时候:
// export default connect(mapStateToProps)(ContentComponent);
// export default connect()(ContentComponent);
```

在通过`connect`进行注入的时候, `dispatch`已经作为组件的 props 而存在了。所以当需要传入的事件很多, 感觉写`mapDispatchToProps`很繁琐的时候, 还有另外一种写法:

```javascript
// App/ContentComponent.js
import {
	increase,
	decrease
} from '../Reducer/ActionCreator';

class ContentComponent extends Component {
	render() {
		const {obj, dispatch}
		return (
			<div>
				<span 
				  onClick={() => {
				  	dispatch(increaseNumber(1));
				  }}
				 >加一</span>
				<span>{obj.number}</span>
				<span
				  onClick={() => {
				  	dispatch(decreaseNumber(1));
				  }}
				>减一</span>
			</div>
		)
	}
}

import { connect } from 'react-redux';
export default connect(mapStateToProps)(ContentComponent);
```

### 优势

通过 Redux, 我们可以少些很多繁琐的事件传输。在 Redux 之前, 顶层组件处理 state 的改变, 而触发的事件则有可能需要层层传递给底层的子组件, 子组件触发之后再次层层回调传到顶层。

但 Redux 的 state 是全局的, 不必关心哪个组件触发`setState()`函数, 只需要设定好`action`和处理 action 的`reducer`, 由`store`进行分发处理。

那样的话, 我们可以在底层触发 state 的改变而不必担心向上调用 --- 触发的 action 改变将被 store 监听, dispatch 给 reducer, reducer通过判断`action.type`, 做出适当的反应处理 state

------

### UPDATE

关于`combineReducer`,`createStore`,`state`的一些问题

在`createStore`的时候，可以传入多个参数:
```javascript
let store = createStore(reducer, [initialState], [enhancer]);
````
其中，initialState是传入reducer的初始化state，enhancer是applyMiddleware一类的中间件插件

在不使用`combineReducer`的时候，可以这么写：
```javascript
// app.js
import appReducer from './reducer';
import {
  SHOW_ALL
} from './ConstValue';
let initialState = {
  list: [],
  status: SHOW_ALL
};
let store = createStore(appReducer, initialState);

// reducer.js
const appReducer = (state, action) => {
  switch (action.type) {
    case ACTION.ADD_ITEM:
      return Object.assign({}, {
        list: [...state.list, action.item]
      });
    case ACTION.SHOW_COMPLETE:
      return Object.assign({}, {
        status: action.filter
      });
    default:
      return state
  }
}
export default appReducer;
```
即初始化的state为通过`createStore`传入的initialState

而使用了`combineReducer`的话，则：
```javascript
// app.js
import appReducer from './index';
let initialState = {
  list: [],
  status: SHOW_ALL
};
let store = createStore(appReducer, initialState);
// 与之前的相比不变

// reducer/reducers.js
export function List(List = [], action) {
  switch (action.type) {
    case ACTION.ADD_ITEM:
      return [...List, action.item];
    default:
      return List;
  }
}

export function status(status = ACTION.SHOW_ALL, action) {
  switch (action.type) {
    case ACTION.SHOW_COMPLETE:
      return action.filter;
    default:
      return status;
  }
}

//reducer/index.js
import * as reducers from './reducers'
import { combineReducers } from 'redux';
export const appReducer = combineReducers(reducers);
```
即通过`combineReducer`合成reducer的话，必须给每个reducer赋一个初始值（不能为undefined）

因为`combineReducer`黑魔法要求函数名与传入的state参数名相同，所以可能会在reducer进行合成的时候，提取reducer参数进行一次state的合并，成为一个默认的INIT-STATE