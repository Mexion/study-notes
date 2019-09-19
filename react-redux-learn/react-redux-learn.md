#  React-redux的使用

react-redux是Redux的官方React绑定。本身redux不只是react使用，这个库可以让我们更方便地使用redux。

首先依然是通过npm安装

```bash
npm install react-redux -S
```

接下来我们在index.js中编写

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import store from './store'
//从react-redux中导出Provider组件
import { Provider } from 'react-redux';

//将整个App组件作为子组件，使用react-redux提供的Provider进行包裹
//并将redux的store作为属性传入Provider之中
ReactDOM.render(<Provider store={ store }><App></App></Provider>, document.getElementById('root'))

```

