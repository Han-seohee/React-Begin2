# React-Begin2 리액트 입문
2021.09.16
### :blue_heart: useReducer 기초

>reducer : 상태를 업데이트 하는 함수

:file_folder:Counter.js

```
import React, { useReducer } from 'react';

function reducer(state, action) {
    switch (action.type) {
        case 'INCREMENT':
            return state + 1;
        case 'DECREMENT':
            return state -1;
        default:
            throw new Error('Unhandled action');
    }
}

function Counter() {

    const [number, dispatch] = useReducer(reducer, 0);
    const onIncrease = () => {
        dispatch({
            type: 'INCREMENT'
        })
    };

    const onDecrease = () => {
        dispatch({
            type: 'DECREMENT'
        })
    };
    
    return (
        <div>
            <h1>{number}</h1>
            <button onClick={onIncrease}>+1</button>
            <button onClick={onDecrease}>-1</button>
        </div>
    )
}

export default Counter;
```

:file_folder:index.js

```
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import Counter from './Counter';
import reportWebVitals from './reportWebVitals';

ReactDOM.render(<Counter />, document.getElementById('root'));

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

---
### :blue_heart: useReducer - App에서 useReducer 사용하기

:file_folder:index.js

```
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import Counter from './Counter';
import reportWebVitals from './reportWebVitals';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();

```

:file_folder:App.js

```
import React, { useRef, useReducer, useMemo, useCallback } from 'react';
import CreateUser from './CreateUser';
import UserList from './UserList';

function CountActiveUsers(users) {
  console.log('활성 사용자 수를 세는중...');
  return users.filter(user => user.active).length;
}

const initialState = {
  inputs: {
    username: '',
    email: '',
  },
  users: [
    {
        id: 1,
        username: 'seohee',
        email: 'a67114585a@gmail.com',
        active: true,
    },
    {
        id: 2,
        username: 'tester',
        email: 'tester@example.com',
        active: false,
    },
    {
        id: 3,
        username: 'liz',
        email: 'liz@example.com',
        active: false,
    }
]
}

function reducer(state, action) {
  switch (action.type) {
    case 'CHANGE_INPUT':
      return {
        ...state,
        inputs: {
          ...state.inputs,
          [action.name]: action.value
        }
      };
      case 'CREATE_USER':
        return {
          inputs: initialState.inputs,
          users: state.users.concat(action.user)
        }
      case 'TOGGLE_USER':
        return {
          ...state,
          users: state.users.map(user =>
            user.id === action.id
              ? { ...user, active: !user.active }
              : user
            )
        };
      case 'REMOVE_USER':
        return {
          ...state,
          users: state.users.filter(user => user.id !== action.id)
        }
      default:
        throw new Error('Unhandled action');
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  const nextId = useRef(4);
  const { users } = state;
  const { username, email } = state.inputs;

  const onChange = useCallback(e => {
    const { name, value } = e.target;
    dispatch({
      type: 'CHANGE_INPUT',
      name,
      value
    })
  }, []);

  const onCreate = useCallback(() => {
    dispatch({
      type: 'CREATE_USER',
      user: {
        id: nextId.current,
        username,
        email,
      }
    });
    nextId.current += 1;
  }, [username, email]);

  const onToggle = useCallback(id => {
    dispatch({
      type: 'TOGGLE_USER',
      id
    });
  }, []);

  const onRemove = useCallback(id => {
    dispatch({
      type: 'REMOVE_USER',
      id
    });
  }, [])

  const count = useMemo(() => CountActiveUsers(users), [users])

  return (
    <>
      <CreateUser 
      username={username} 
      email={email} 
      onChange={onChange} 
      onCreate={onCreate}
      />
      <UserList users={users} 
      onToggle={onToggle} 
      onRemove={onRemove} 
      />
      <div>활성 사용자 수 : {count}</div>
    </>
  )
}

export default App;
```

:mag:

<img src=https://user-images.githubusercontent.com/86407453/133541588-4b26e787-ac67-42f7-8d5e-c83bd4009a2c.png>

---
### :blue_heart: 커스텀 Hook 만들어서 사용하기

:file_folder:App.js

```
import React, { useRef, useReducer, useMemo, useCallback } from 'react';
import CreateUser from './CreateUser';
import UserList from './UserList';
import useInputs from './useInputs';

function CountActiveUsers(users) {
  console.log('활성 사용자 수를 세는중...');
  return users.filter(user => user.active).length;
}

const initialState = {
  users: [
    {
        id: 1,
        username: 'seohee',
        email: 'a67114585a@gmail.com',
        active: true,
    },
    {
        id: 2,
        username: 'tester',
        email: 'tester@example.com',
        active: false,
    },
    {
        id: 3,
        username: 'liz',
        email: 'liz@example.com',
        active: false,
    }
]
}

function reducer(state, action) {
  switch (action.type) {
      case 'CREATE_USER':
        return {
          inputs: initialState.inputs,
          users: state.users.concat(action.user)
        }
      case 'TOGGLE_USER':
        return {
          ...state,
          users: state.users.map(user =>
            user.id === action.id
              ? { ...user, active: !user.active }
              : user
            )
        };
      case 'REMOVE_USER':
        return {
          ...state,
          users: state.users.filter(user => user.id !== action.id)
        }
      default:
        throw new Error('Unhandled action');
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  const [form, onChange, reset] = useInputs({
    username: '',
    email: '',
  });
  const { username, email } = form;
  const nextId = useRef(4);
  const { users } = state;

  const onCreate = useCallback(() => {
    dispatch({
      type: 'CREATE_USER',
      user: {
        id: nextId.current,
        username,
        email,
      }
    });
    nextId.current += 1;
    reset();
  }, [username, email, reset]);

  const onToggle = useCallback(id => {
    dispatch({
      type: 'TOGGLE_USER',
      id
    });
  }, []);

  const onRemove = useCallback(id => {
    dispatch({
      type: 'REMOVE_USER',
      id
    });
  }, [])

  const count = useMemo(() => CountActiveUsers(users), [users])

  return (
    <>
      <CreateUser 
      username={username} 
      email={email} 
      onChange={onChange} 
      onCreate={onCreate}
      />
      <UserList users={users} 
      onToggle={onToggle} 
      onRemove={onRemove} 
      />
      <div>활성 사용자 수 : {count}</div>
    </>
  )
}

export default App;
```

:file_folder:useInputs.js

```
import React, { useRef, useReducer, useMemo, useCallback } from 'react';
import CreateUser from './CreateUser';
import UserList from './UserList';
import useInputs from './useInputs';

function CountActiveUsers(users) {
  console.log('활성 사용자 수를 세는중...');
  return users.filter(user => user.active).length;
}

const initialState = {
  users: [
    {
        id: 1,
        username: 'seohee',
        email: 'a67114585a@gmail.com',
        active: true,
    },
    {
        id: 2,
        username: 'tester',
        email: 'tester@example.com',
        active: false,
    },
    {
        id: 3,
        username: 'liz',
        email: 'liz@example.com',
        active: false,
    }
]
}

function reducer(state, action) {
  switch (action.type) {
      case 'CREATE_USER':
        return {
          inputs: initialState.inputs,
          users: state.users.concat(action.user)
        }
      case 'TOGGLE_USER':
        return {
          ...state,
          users: state.users.map(user =>
            user.id === action.id
              ? { ...user, active: !user.active }
              : user
            )
        };
      case 'REMOVE_USER':
        return {
          ...state,
          users: state.users.filter(user => user.id !== action.id)
        }
      default:
        throw new Error('Unhandled action');
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  const [form, onChange, reset] = useInputs({
    username: '',
    email: '',
  });
  const { username, email } = form;
  const nextId = useRef(4);
  const { users } = state;

  const onCreate = useCallback(() => {
    dispatch({
      type: 'CREATE_USER',
      user: {
        id: nextId.current,
        username,
        email,
      }
    });
    nextId.current += 1;
    reset();
  }, [username, email, reset]);

  const onToggle = useCallback(id => {
    dispatch({
      type: 'TOGGLE_USER',
      id
    });
  }, []);

  const onRemove = useCallback(id => {
    dispatch({
      type: 'REMOVE_USER',
      id
    });
  }, [])

  const count = useMemo(() => CountActiveUsers(users), [users])

  return (
    <>
      <CreateUser 
      username={username} 
      email={email} 
      onChange={onChange} 
      onCreate={onCreate}
      />
      <UserList users={users} 
      onToggle={onToggle} 
      onRemove={onRemove} 
      />
      <div>활성 사용자 수 : {count}</div>
    </>
  )
}

export default App;
```

:mag:

<img src=https://user-images.githubusercontent.com/86407453/133546273-77804dd7-0654-4184-8345-7dd3647950a7.png>

---
### :blue_heart: Context API 를 사용한 전역 값 관리

:file_folder:index.js

```
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import ContextSample from './ContextSample';

ReactDOM.render(
  <ContextSample />,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

:file_folder: ContextSample.js

```
import React, { createContext, useContext, useState } from 'react';

const MyContext = createContext('defaultValue');

function Child() {
    const text = useContext(MyContext);
    return <div>안녕하세요? {text}</div>
}

function Parent() {
    return <Child />
}

function GrandParent() {
    return <Parent />
}

function ContextSample() {
    const [value, setValue] = useState(true);
    return (
    <MyContext.Provider value={value ? 'GOOD' : 'BAD'}>
        <GrandParent />
        <button onClick={() => setValue(!value)}>CLICK ME</button>
    </MyContext.Provider>
    )
}

export default ContextSample;
```

:file_folder:App.js

```
import React, { useRef, useReducer, useMemo, useCallback } from 'react';
import CreateUser from './CreateUser';
import UserList from './UserList';
import useInputs from './useInputs';

function CountActiveUsers(users) {
  console.log('활성 사용자 수를 세는중...');
  return users.filter(user => user.active).length;
}

const initialState = {
  users: [
    {
        id: 1,
        username: 'seohee',
        email: 'a67114585a@gmail.com',
        active: true,
    },
    {
        id: 2,
        username: 'tester',
        email: 'tester@example.com',
        active: false,
    },
    {
        id: 3,
        username: 'liz',
        email: 'liz@example.com',
        active: false,
    }
]
}

function reducer(state, action) {
  switch (action.type) {
      case 'CREATE_USER':
        return {
          inputs: initialState.inputs,
          users: state.users.concat(action.user)
        }
      case 'TOGGLE_USER':
        return {
          ...state,
          users: state.users.map(user =>
            user.id === action.id
              ? { ...user, active: !user.active }
              : user
            )
        };
      case 'REMOVE_USER':
        return {
          ...state,
          users: state.users.filter(user => user.id !== action.id)
        }
      default:
        throw new Error('Unhandled action');
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  const [form, onChange, reset] = useInputs({
    username: '',
    email: '',
  });
  const { username, email } = form;
  const nextId = useRef(4);
  const { users } = state;

  const onCreate = useCallback(() => {
    dispatch({
      type: 'CREATE_USER',
      user: {
        id: nextId.current,
        username,
        email,
      }
    });
    nextId.current += 1;
    reset();
  }, [username, email, reset]);

  const onToggle = useCallback(id => {
    dispatch({
      type: 'TOGGLE_USER',
      id
    });
  }, []);

  const onRemove = useCallback(id => {
    dispatch({
      type: 'REMOVE_USER',
      id
    });
  }, [])

  const count = useMemo(() => CountActiveUsers(users), [users])

  return (
    <>
      <CreateUser 
      username={username} 
      email={email} 
      onChange={onChange} 
      onCreate={onCreate}
      />
      <UserList users={users} 
      onToggle={onToggle} 
      onRemove={onRemove} 
      />
      <div>활성 사용자 수 : {count}</div>
    </>
  )
}

export default App;
```

:mag:

<img src=https://user-images.githubusercontent.com/86407453/133548949-598e6d2d-d716-40e6-b572-918a26d3e805.png>

<img src=https://user-images.githubusercontent.com/86407453/133548977-be50a92f-f0f8-4616-bd61-a0ff7695ce8e.png>

---
### :blue_heart: UserDispatch Context 만들기

:file_folder:index.js

```
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';

ReactDOM.render(
  <App />,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

:file_folder:UserList.js

```
import React, { useContext } from 'react';
import { UserDispatch } from './App';

const User = React.memo(function User({ user }) {
    const { username, email, id, active } = user;
    const dispatch = useContext(UserDispatch);
    
    useContext(() => {
        console.log(user);
    });

    return (
        <div>
            <b
                style={{
                color: active ? 'green' : 'black',
                cursor: 'pointer'
            }}
            onClick={() => dispatch({
                type: 'TOGGLE_USER',
                id
            })}
            >
            {username}
            </b> 
            &nbsp;
            <span>({email})</span>
            <button onClick={() => dispatch({
                type: 'REMOVE_USER',
                id
            })}>삭제</button>
        </div>
    );
});

function UserList({ users }) {
    return (
        <div>
            {
                users.map(
                    (user) => (
                    <User 
                    user={user} 
                    key={user.id}
                    />
                    )
                )
            }
        </div>
    );
}

export default React.memo(
    UserList, (prevProps, nextProps) => nextProps.users === prevProps.users
    );
```

:file_folder:App.js

```
import React, { useRef, useReducer, useMemo, useCallback, createContext } from 'react';
import CreateUser from './CreateUser';
import UserList from './UserList';
import useInputs from './useInputs';

function CountActiveUsers(users) {
  console.log('활성 사용자 수를 세는중...');
  return users.filter(user => user.active).length;
}

const initialState = {
  users: [
    {
        id: 1,
        username: 'seohee',
        email: 'a67114585a@gmail.com',
        active: true,
    },
    {
        id: 2,
        username: 'tester',
        email: 'tester@example.com',
        active: false,
    },
    {
        id: 3,
        username: 'liz',
        email: 'liz@example.com',
        active: false,
    }
]
}

function reducer(state, action) {
  switch (action.type) {
      case 'CREATE_USER':
        return {
          inputs: initialState.inputs,
          users: state.users.concat(action.user)
        }
      case 'TOGGLE_USER':
        return {
          ...state,
          users: state.users.map(user =>
            user.id === action.id
              ? { ...user, active: !user.active }
              : user
            )
        };
      case 'REMOVE_USER':
        return {
          ...state,
          users: state.users.filter(user => user.id !== action.id)
        }
      default:
        throw new Error('Unhandled action');
  }
}

export const UserDispatch = createContext(null);

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  const [form, onChange, reset] = useInputs({
    username: '',
    email: '',
  });
  const { username, email } = form;
  const nextId = useRef(4);
  const { users } = state;

  const onCreate = useCallback(() => {
    dispatch({
      type: 'CREATE_USER',
      user: {
        id: nextId.current,
        username,
        email,
      }
    });
    nextId.current += 1;
    reset();
  }, [username, email, reset]);

  const count = useMemo(() => CountActiveUsers(users), [users])

  return (
    <UserDispatch.Provider value={dispatch}>
      <CreateUser 
      username={username} 
      email={email} 
      onChange={onChange} 
      onCreate={onCreate}
      />
      <UserList users={users} />
      <div>활성 사용자 수 : {count}</div>
    </UserDispatch.Provider>
  )
}

export default App;
```

:mag:

<img src=https://user-images.githubusercontent.com/86407453/133550330-f5b10c47-b004-4a37-afe5-afd8b92ecd37.png>

---
### :blue_heart: 클래스형 컴포넌트

:file_folder:index.js

```
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import Hello from './Hello';


ReactDOM.render(
  <Hello name="react" isSpecial />,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();

```

:file_folder:Hello.js

```
import React, { Component } from 'react';

class Hello extends Component {
    static defaultProps = {
        name: '이름없음',
    };
    render() {
        const { color, isSpecial, name } = this.props;
        return (
            <div style={{ color }}>
                {isSpecial && <b>*</b>}
                안녕하세요 {name}
            </div>
        );
    }
}

export default Hello;
```

:file_folder:App.js

```
import React, { useRef, useReducer, useMemo, useCallback, createContext } from 'react';
import CreateUser from './CreateUser';
import UserList from './UserList';
import useInputs from './useInputs';

function CountActiveUsers(users) {
  console.log('활성 사용자 수를 세는중...');
  return users.filter(user => user.active).length;
}

const initialState = {
  users: [
    {
        id: 1,
        username: 'seohee',
        email: 'a67114585a@gmail.com',
        active: true,
    },
    {
        id: 2,
        username: 'tester',
        email: 'tester@example.com',
        active: false,
    },
    {
        id: 3,
        username: 'liz',
        email: 'liz@example.com',
        active: false,
    }
]
}

function reducer(state, action) {
  switch (action.type) {
      case 'CREATE_USER':
        return {
          inputs: initialState.inputs,
          users: state.users.concat(action.user)
        }
      case 'TOGGLE_USER':
        return {
          ...state,
          users: state.users.map(user =>
            user.id === action.id
              ? { ...user, active: !user.active }
              : user
            )
        };
      case 'REMOVE_USER':
        return {
          ...state,
          users: state.users.filter(user => user.id !== action.id)
        }
      default:
        throw new Error('Unhandled action');
  }
}

export const UserDispatch = createContext(null);

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  const [form, onChange, reset] = useInputs({
    username: '',
    email: '',
  });
  const { username, email } = form;
  const nextId = useRef(4);
  const { users } = state;

  const onCreate = useCallback(() => {
    dispatch({
      type: 'CREATE_USER',
      user: {
        id: nextId.current,
        username,
        email,
      }
    });
    nextId.current += 1;
    reset();
  }, [username, email, reset]);

  const count = useMemo(() => CountActiveUsers(users), [users])

  return (
    <UserDispatch.Provider value={dispatch}>
      <CreateUser 
      username={username} 
      email={email} 
      onChange={onChange} 
      onCreate={onCreate}
      />
      <UserList users={users} />
      <div>활성 사용자 수 : {count}</div>
    </UserDispatch.Provider>
  )
}

export default App;
```

:mag:

<img src=https://user-images.githubusercontent.com/86407453/133573105-16aa1590-d173-4423-a7e7-847e12c895dd.png>

---
### :blue_heart: 클래스형 컴포넌트의 state와 setState

:file_folder:Index.js

```
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import Counter from './Counter';


ReactDOM.render(
  <Counter />,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

:file_folder:Counter.js

```
import React, { Component } from 'react';

class Counter extends Component {

    constructor(props) {
        super(props);
        this.handleIncrease = this.handleIncrease.bind(this);
        this.handleDecrease = this.handleDecrease.bind(this);
    }

    handleIncrease() {
        console.log('increase');
    }

    handleDecrease() {
        console.log('decrease');
    }

    render() {
        return (
            <div>
                <h1>0</h1>
                <button onClick={this.handleIncrease}>+1</button>
                <button onClick={this.handleDecrease}>-1</button>
            </div>
        )
    }
}

// function reducer(state, action) {
//     switch (action.type) {
//         case 'INCREMENT':
//             return state + 1;
//         case 'DECREMENT':
//             return state -1;
//         default:
//             throw new Error('Unhandled action');
//     }
// }

// function Counter() {

//     const [number, dispatch] = useReducer(reducer, 0);
//     const onIncrease = () => {
//         dispatch({
//             type: 'INCREMENT'
//         })
//     };

//     const onDecrease = () => {
//         dispatch({
//             type: 'DECREMENT'
//         })
//     };
    
//     return (
//         <div>
//             <h1>{number}</h1>
//             <button onClick={onIncrease}>+1</button>
//             <button onClick={onDecrease}>-1</button>
//         </div>
//     )
// }

export default Counter;
```

:point_down: 화살표함수

:file_folder:Counter.js

```
import React, { Component } from 'react';

class Counter extends Component {
    handleIncrease = () => {
        console.log(this);
        console.log('increase');
    }

    handleDecrease = () => {
        console.log('decrease');
    }

    render() {
        return (
            <div>
                <h1>0</h1>
                <button onClick={this.handleIncrease}>+1</button>
                <button onClick={this.handleDecrease}>-1</button>
            </div>
        )
    }
}

// function reducer(state, action) {
//     switch (action.type) {
//         case 'INCREMENT':
//             return state + 1;
//         case 'DECREMENT':
//             return state -1;
//         default:
//             throw new Error('Unhandled action');
//     }
// }

// function Counter() {

//     const [number, dispatch] = useReducer(reducer, 0);
//     const onIncrease = () => {
//         dispatch({
//             type: 'INCREMENT'
//         })
//     };

//     const onDecrease = () => {
//         dispatch({
//             type: 'DECREMENT'
//         })
//     };
    
//     return (
//         <div>
//             <h1>{number}</h1>
//             <button onClick={onIncrease}>+1</button>
//             <button onClick={onDecrease}>-1</button>
//         </div>
//     )
// }

export default Counter;
```

:point_down: 상태관리

:file_folder:Counter.js

```
import React, { Component } from 'react';

class Counter extends Component {
    constructor(props) {
        super(props);
        this.state = {
            counter: 0
        };
    }

    handleIncrease = () => {
        this.setState({
            counter: this.state.counter + 1
        });
    }

    handleDecrease = () => {
        this.setState({
            counter: this.state.counter - 1
        });
    }

    render() {
        return (
            <div>
                <h1>{this.state.counter}</h1>
                <button onClick={this.handleIncrease}>+1</button>
                <button onClick={this.handleDecrease}>-1</button>
            </div>
        )
    }
}

// function reducer(state, action) {
//     switch (action.type) {
//         case 'INCREMENT':
//             return state + 1;
//         case 'DECREMENT':
//             return state -1;
//         default:
//             throw new Error('Unhandled action');
//     }
// }

// function Counter() {

//     const [number, dispatch] = useReducer(reducer, 0);
//     const onIncrease = () => {
//         dispatch({
//             type: 'INCREMENT'
//         })
//     };

//     const onDecrease = () => {
//         dispatch({
//             type: 'DECREMENT'
//         })
//     };
    
//     return (
//         <div>
//             <h1>{number}</h1>
//             <button onClick={onIncrease}>+1</button>
//             <button onClick={onDecrease}>-1</button>
//         </div>
//     )
// }

export default Counter;
```

:mag:

<img src=https://user-images.githubusercontent.com/86407453/133576267-601a3fc4-da1a-440e-8ef9-fefc5007dd6b.png>


:point_down: 값을 바로 지정해 주는 방법

:file_folder:Counter.js

```
import React, { Component } from 'react';

class Counter extends Component {
    
    state = {
        counter: 0
    }

    handleIncrease = () => {
        this.setState({
            counter: this.state.counter + 1
        });
    }

    handleDecrease = () => {
        this.setState({
            counter: this.state.counter - 1
        });
    }

    render() {
        return (
            <div>
                <h1>{this.state.counter}</h1>
                <button onClick={this.handleIncrease}>+1</button>
                <button onClick={this.handleDecrease}>-1</button>
            </div>
        )
    }
}

// function reducer(state, action) {
//     switch (action.type) {
//         case 'INCREMENT':
//             return state + 1;
//         case 'DECREMENT':
//             return state -1;
//         default:
//             throw new Error('Unhandled action');
//     }
// }

// function Counter() {

//     const [number, dispatch] = useReducer(reducer, 0);
//     const onIncrease = () => {
//         dispatch({
//             type: 'INCREMENT'
//         })
//     };

//     const onDecrease = () => {
//         dispatch({
//             type: 'DECREMENT'
//         })
//     };
    
//     return (
//         <div>
//             <h1>{number}</h1>
//             <button onClick={onIncrease}>+1</button>
//             <button onClick={onDecrease}>-1</button>
//         </div>
//     )
// }

export default Counter;
```
