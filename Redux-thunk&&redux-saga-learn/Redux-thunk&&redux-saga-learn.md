# Redux-thunk和Redux-saga的使用



有时我们在处理一些异步请求或者复杂逻辑的时候会让组件变得十分臃肿并且难以维护，redux-thunk可以帮助我们对异步操作放到action中进行处理。
redux-thunk是redux的中间件，我们首先通过npm进行安装

```bash
npm install redux-thunk -S
```
需要使用中间件，我们在创建store时需要从redux中引入applyMiddleware方法，
```js
import { createStore, applyMiddleware } from "redux";
import thunk from "redux-thunk";
import reducer from "./reducer";

export default const store = createStore(
    reducer,
    applyMiddleware(thunk)
);
```
如果要同时使用中间件和redux的devtool插件，可以通过redux的compose函数进行组合。
```js
import { createStore, applyMiddleware, compose } from "redux";
import thunk from "redux-thunk";
import reducer from "./reducer";

const composeEnhancers = typeof window === 'object' && window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ? window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({ }) : compose;

const enhancer = composeEnhancers( applyMiddleware(thunk) );

export default const store = createStore(
    reducer,
    enhancer
);
```

在使用redux-thunk之后我们创建action的时候可以不再是一个对象，而可以是一个函数，这个函数可以进行一些异步或者复杂的操作。
比如我们要使用axios在页面挂载时请求数据，我们就可以将这个操作放到action中，并在componentDidMount这个生命周期中通过store.dispatch(action)将这个函数action传入，这时会自动执行这个action函数，并将dispatch作为参数传入这个函数。在这个函数中我们获取到数据后还需要创建一个action对象，并在这里使用传过来的dispatch发送action。

其实所谓中间件就是在两者之间进行一定的操作，在redux中是介于action和store之间，也就是dispatch方法，在没有使用中间件时，我们创建了action通过dispatch直接将其传递给store，store通过reducer返回新的state。但使用了中间件之后就可以对dispatch进行修改，例如thunk就是对dispatch进行了重新封装，假如传递过来的action是一个对象，那么就照旧直接传给store，如果传过来的是一个函数就将dispatch传给这个函数并运行它。

## 2.redux-saga
redux-saga和thunk一样，也是一个处理异步的中间件，区别在于saga是将操作与action区分开来，在单独的位置进行管理。
我们需要先安装saga

```bash
npm install redux-saga -S
```

然后导入createSagaMiddleware创建一个saga中间件

```js
import { createStore, applyMiddleware, compose } from "redux";
import createSagaMiddleware from "redux-saga";
import reducer from "./reducer";

const sagaMiddleware = createSagaMiddleware();

const composeEnhancers = typeof window === 'object' && window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ? window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({ }) : compose;

const enhancer = composeEnhancers( applyMiddleware(sagaMiddleware) );

export default const store = createStore(
    reducer,
    enhancer
);
```

接下来我们需要创建一个sagas.js文件，因为saga是用单独的文件来管理异步的操作。创建好后import导入，然后使用sagaMiddleware的run方法将导入的saga作为参数传入。
```js
import { createStore, applyMiddleware, compose } from "redux";
import createSagaMiddleware from "redux-saga";
import reducer from "./reducer";
import mySagas from “./sagas”

const sagaMiddleware = createSagaMiddleware();

const composeEnhancers = typeof window === 'object' && window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ? window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({ }) : compose; 
const enhancer = composeEnhancers( applyMiddleware(sagaMiddleware) );

export default const store = createStore(
    reducer,
    enhancer
);

sagaMiddleware.run(mySagas);
```

接下来就可以编写创建的sagas.js文件，

```js
import { takeEvery } from 'redux-saga/effects';

function* mySagas() {
    yield takeEvery(GET_INIT_LIST, getInitList);
}

export default mySagas;
```

sagas.js必须返回一个generator函数，我们从redux-saga/effects中导入takeEvery方法，并用这个方法去拦截type为GET_INIT_LIST的acion，如果拦截到这个action那么就运行getInitList这个函数，我们的异步操作就可以放到这个函数中：

```js
import { takeEvery, put } from 'redux-saga/effects';
import axios from 'axios';

function* getTodoList() {
    try {
        const res = yield axios.get('./list.json');
    	const action = {
                type: 'init_list_action',
                list: res.data
        		}
    	yield put(action);
    }catch(e) {
        console.log(`can't to fetch list.json,${ e }`)
    }
}

function* mySagas() {
    yield takeEvery(GET_TODO_LIST, getTodoList);
}

export default mySagas;
```

这里假设使用axios去请求本地的list文件，当mySagas拦截到GET_TODO_LIST这个acion时，就会执行getTodoList方法，而在getTodoList方法中我们使用了axios获取到了数据，并且创建了一个acion，使用saga导入的put方法将action发送给store，这里的put相当于我们平常使用redux的dispatch。