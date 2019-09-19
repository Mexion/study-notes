## 1.Redux工作流
我将会把这个过程比拟成图书馆的一个流程来帮助理解。
- Action Creator（具体借书的表达） 
想借书的人向图书馆管理员说明要借的书的那句话。

- Store（图书馆管理员） 
负责整个图书馆的管理。是Redux的核心。

- Reducers（图书馆的记录本） 
管理员需要借助Reducer（图书馆的记录本）来记录。

- React Component(借书的人)
React中需要数据的组件，既为借书的人。


整个工作流可以理解为:
需要借书的人(component)向图书馆管理员(store)说明想要借什么书(action)，图书馆管理员自己不知道书在哪需要借助图书馆的记录本(reducer)，通过reducer查询后明确需要借的书，借给借书者。
通俗理解为:
组件想要获取State， 用ActionCreator创建了一个请求交给Store,Store借助Reducer确认了该State的状态，Reducer返回给Store一个结果，Store再把这个State转给组件。


## 2.实战演示

首先要通过npm安装Redux
```
npm install redux -S
```
安装完毕后我们在项目中新建store目录，并在目录中新建一个index.js文件，在index.js中创建store:
```js
import { createStore } from 'redux';

const store = createStore();

export default store;
```
这样子store虽然创建出来了，但是正如图书管理员需要图书记录本一样，store也需要reducer来管理数据，我们需要创建reducer并且将其传递给store。新建reducer.js文件，输入如下代码:
```js
const defaultState = {
    inputValue: '',
    list: []
};
export default (state = defaultState, action) =>  {
    return state;
};
```
reducer.js需要导出一个函数，这个函数的第一个参数state代表了整个store存储的数据，我们在上方定义了一个对象并传给了state，这就是整个store存储的默认数据。第二个参数action含义上面已经解释过了，代表了借书的那句话。现在再把记录本(reducer)传递给图书管理员(store)，修改index.js:
```javascript
import { createStore } from 'redux';
import { reducer } from './reducer';

const store = createStore(reducer);

export default store;
```
在需要使用store数据的组件中，我们可以引入index.js，然后使用getState()方法获取里面的数据，假设下方代码是一个TodoList组件:
```jsx
import React, { Component, Fragment } from 'react';
import store from '../store';

class TodoList extends Component {
  constructor(props) {
    super(props);
    this.state = store.getState();
  }

  render() {
    return (
      <Fragment>
        <input 
          type="text"
          className="input"
          placeholder="input your todo"
          value={ this.state.inputValue }
          />
        <button className="button">提交</button>
        <ul>
          {
            this.state.list.map((item, index) => {
              return (
                <li key={ index }>{ item }</li>)
            })
          }
        </ul>
      </Fragment>
    )
  }
}

export default TodoList;
```
这样就将整个组件state的数据和store中的数据绑定了起来。



那么如何修改store中的数据呢，假如我们要在input的值改变的时候修改store中的inputValue，首先需要给input添加一个onChange事件，如下：

```jsx
import React, { Component, Fragment } from 'react';
import store from '../store';

class TodoList extends Component {
  constructor(props) {
    super(props);
    this.state = store.getState();
    this.handleInputChange = this.handleInputChange.bind(this);
  }

  render() {
    return (
      <Fragment>
        <input 
          type="text"
          className="input"
          placeholder="input your todo"
          value={ this.state.inputValue }
          onChange={ this.handleInputChange }
          />
        <button className="button">提交</button>
        <ul>
          {
            this.state.list.map((item, index) => {
              return (
                <li key={ index }>{ item }</li>)
            })
          }
        </ul>
      </Fragment>
    )
  }

  handleInputChange (e) {
    const action = {
      type: 'change_input_value',
      value: e.target.value
    }

    store.dispatch(action);
  }
}

export default TodoList;
```

当触发事件后，我们在事件中生成一个action，aciton实际上就是一个对象，它的type表明需要做什么事情，同时将修改需要的value值也写入action中，使用store的dispatch方法将action传递给store，store接收到action后，会将当前的state数据和和action一并传递给reducer，正如上面所写的reducer的两个参数。

reducer接收到数据和action之后，我们就可以根据需要进行对应的操作了： 

```js
const defaultState = {
    inputValue: '',
    list: []
}

export default (state = defaultState, action) => {
    if(action.type === 'change_input_value') {
        const newState = JSON.parse(JSON.stringify(state));
        newState.inputValue = action.value;
        return newState;
    }
    return state;
}
```

我们在里面进行判断，如果action的type等于change_input_value，那么就把state中的inputValue替换为actoin中传递过来的value，并且返回修改后的数据，这里我们对state进行了一次深拷贝，因为在redux中我们不能直接对state进行修改。



虽然我们已经可以修改store中的state，但是组件内的value并不会跟随store中的数据进行更新，我们还需要进一步的操作：

```jsx
import React, { Component, Fragment } from 'react';
import store from '../store';

class TodoList extends Component {
  constructor(props) {
    super(props);
    this.state = store.getState();
    this.handleInputChange = this.handleInputChange.bind(this);
    this.handleStoreChange = this.handleStoreChange.bind(this);
    store.subscribe(this.handleStoreChange);
  }

  render() {
    return (
      <Fragment>
        <input 
          type="text"
          className="input"
          placeholder="input your todo"
          value={ this.state.inputValue }
          onChange={ this.handleInputChange }
          />
        <button className="button">提交</button>
        <ul>
          {
            this.state.list.map((item, index) => {
              return (
                <li key={ index }>{ item }</li>)
            })
          }
        </ul>
      </Fragment>
    )
  }

  handleInputChange(e) {
    const action = {
      type: 'change_input_value',
      value: e.target.value
    }
    store.dispatch(action);
  }

  handleStoreChange() {
    this.setState(store.getState());
  }
}

export default TodoList;
```

我们在构造函数中使用store的subscribe方法，参数传入一个函数，只要store的数据发生改变就会触发这个函数，我们在这个函数里使用setState方法并传入store.getState()来同步更新组件的数据。

接下来我们完善整个TodoList，包括点击按钮新增todo以及点击todo删除自身的功能：

```jsx
// this is the TodoList.jsx

import React, { Component, Fragment } from 'react';
import store from '../store';

class TodoList extends Component {
  constructor(props) {
    super(props);
    this.state = store.getState();
    this.handleInputChange = this.handleInputChange.bind(this);
    this.handleStoreChange = this.handleStoreChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
    store.subscribe(this.handleStoreChange);
  }

  render() {
    return (
      <Fragment>
        <input 
          type="text"
          className="input"
          placeholder = "input your todo"
          value={ this.state.inputValue }
          onChange={ this.handleInputChange }
          />
        <button
          className="button"
          onClick={ this.handleSubmit }>提交</button>
        <ul>
          {
            this.state.list.map((item, index) => {
              return (
                <li 
                  key={ index }
                  onClick={ this.handleDeleteItem.bind(this, index) }>{ item }</li>)
            })
          }
        </ul>
      </Fragment>
    )
  }

  handleInputChange(e) {
    const action = {
      type: 'change_input_value',
      value: e.target.value
    }
    store.dispatch(action);
  }

  handleStoreChange() {
    this.setState(store.getState());
  }

  handleSubmit() {
    if(this.state.inputValue === '') {
      alert("You can't submit a null value!");
      return false;
    }
    const action = {
      type: 'add_todo_item'
    }
    store.dispatch(action);
  }

  handleDeleteItem(index) {
    const action = {
      type: 'delete_todo_item',
      index
    }
    store.dispatch(action);
  }
}

export default TodoList;
```

```js
// this is the reducer.js

const defaultState = {
    inputValue: '',
    list: []
}

export default (state = defaultState, action) => {
    if(action.type === 'change_input_value') {
        const newState = JSON.parse(JSON.stringify(state));
        newState.inputValue = action.value;
        return newState;
    }

    if (action.type === 'add_todo_item') {
        const newState = JSON.parse(JSON.stringify(state));
        newState.list.push(newState.inputValue);
        newState.inputValue = '';
        return newState;
    }

    if (action.type === 'delete_todo_item') {
        const newState = JSON.parse(JSON.stringify(state));
        newState.list.splice(action.index, 1)
        return newState;
    }
    return state;
}
```

## 3.拆分actionTypes

虽然现在整个todolist的功能没有任何问题，但是我们没做一个功能都要派发一个action，action的type是一个字符串，一旦拼写错误并不会报错，可能因为拼写错误的问题导致功能的异常，但是却很难找到bug，这是我们可以将action的type拆分出来成为一个个常量，在需要的地方直接导入使用即可，即使拼写发生了错误，因为它是一个常量，也会直接报错，能立即找到bug的所在。

首先我们需要在store文件夹下新建一个actionTypes.js文件，在里面直接定义需要的actionType常量：

```js
//this is the actionTypes.js
export const CHANGE_INPUT_VALUE = 'change_input_value';
export const ADD_TODO_ITEM = 'add_todo_item';
export const DELETE_TODO_ITEM = 'delete_todo_item';
```

```jsx
//this is the TodoList.jsx
import React, { Component, Fragment } from 'react';
import store from '../store';
import { CHANGE_INPUT_VALUE, ADD_TODO_ITEM, DELETE_TODO_ITEM } from '../store/actionTypes';

class TodoList extends Component {
  constructor(props) {
    super(props);
    this.state = store.getState();
    this.handleInputChange = this.handleInputChange.bind(this);
    this.handleStoreChange = this.handleStoreChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
    store.subscribe(this.handleStoreChange);
  }

  render() {
    return (
      <Fragment>
        <input 
          type="text"
          className="input"
          placeholder = "input your todo"
          value={ this.state.inputValue }
          onChange={ this.handleInputChange }
          />
        <button
          className="button"
          onClick={ this.handleSubmit }>提交</button>
        <ul>
          {
            this.state.list.map((item, index) => {
              return (
                <li 
                  key={ index }
                  onClick={ this.handleDeleteItem.bind(this, index) }>{ item }</li>)
            })
          }
        </ul>
      </Fragment>
    )
  }

  handleInputChange(e) {
    const action = {
      type: CHANGE_INPUT_VALUE,
      value: e.target.value
    }
    store.dispatch(action);
  }

  handleStoreChange() {
    this.setState(store.getState());
  }

  handleSubmit() {
    if(this.state.inputValue === '') {
      alert("You can't submit a null value!");
      return false;
    }
    const action = {
      type: ADD_TODO_ITEM
    }
    store.dispatch(action);
  }

  handleDeleteItem(index) {
    const action = {
      type: DELETE_TODO_ITEM,
      index
    }
    store.dispatch(action);
  }
}

export default TodoList;
```

```js
// this is the reducer.js
import { CHANGE_INPUT_VALUE, ADD_TODO_ITEM, DELETE_TODO_ITEM } from '../store/actionTypes';

const defaultState = {
    inputValue: '',
    list: []
}

export default (state = defaultState, action) => {
    if (action.type === CHANGE_INPUT_VALUE) {
        const newState = JSON.parse(JSON.stringify(state));
        newState.inputValue = action.value;
        return newState;
    }

    if (action.type === ADD_TODO_ITEM) {
        const newState = JSON.parse(JSON.stringify(state));
        newState.list.push(newState.inputValue);
        newState.inputValue = '';
        return newState;
    }

    if (action.type === DELETE_TODO_ITEM) {
        const newState = JSON.parse(JSON.stringify(state));
        newState.list.splice(action.index, 1)
        return newState;
    }
    return state;
}
```

