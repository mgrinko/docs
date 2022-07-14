# React + Typescript

- Use `.tsx` extension in all the files having JSX (e.g. `index.tsx`)
- Create a `types` folder and put global types there (1 file = 1 interface)
- Put types for `Props` and `State` in its component file

## Functional component
```tsx
import React from 'react';

const App: React.FC = () => (
  <div className="App">
    <UsersList users={usersFromServer} />
  </div> 
);

type Props = {
  users: User[];
};

const UsersList: React.FC<Props> = ({ users }) => (
  // your markup
);
```

## Class component
```tsx
import React from 'react';

type State = {
  title: string;
  users: User[];
  selectedUser: User | null;
};

type Props = {
  todos: Todo[];
  onAdd: (title: string) => void;
};

class Counter extends React.Component<Props, State> {

  // Readonly protects you from a mutation `this.state.title = 'Hello'`
  state: Readonly<State> = { 
    title: '',
    users: [],
    selectedUser: null,
    // without `state: State` typescript can't understand
    // that `selectedUser` can also be a `User`, not only null
  };
  
  render() {
    const { title, users } = this.state;
    
    return (
      <>
        <h1>{title}</h1>
        <ul>
          {users.map(user => (
            <li key={user.id}>
              {user.name}
            </li>
          ))}
        </ul>
      </>
    );
  }
}
```

## Global events
```typescript
// we can import only `Component` instead of whole React
import { Component } from 'react';

type State = {
  key: string;
};

// {} because Props are empty
class App extends Component<{}, State> {
  state: Readonly<State> = {
    key: '',
  };

  handleKeyup = (event: KeyboardEvent) => { 
    console.log(event.key);
  };

  componentDidMount() {
    // we add a global handler when the component appears
    window.addEventListener('keyup', this.handleKeyup);
  }
  
  componentWillUnmount() {
    // we must remove global handler when the commponent disappears
    window.removeEventListener('keyup', this.handleKeyup);
  }
}
```

## Events
```tsx
const App: React.FC = () => {
  const handleClick = (event: React.MouseEvent<HTMLButtonElement>) => {
    console.log(event.currentTarget.tagName); // BUTTON
    
    // but `event.target` does not exist in React.MouseEvent type
    console.log(event.target.tagName); // type error
  };

  return (
    <div className="App">
      <button type="button" onClick={handleClick}>
        My button
      </button>
      
      <button
        type="button"
        onClick={(event) => {
          // TS sees the type from the usage `<button onClick={event => ...}`
          console.log(event.currentTarget.tagName); // BUTTON
        }}
    </div>
  );
};
```

## useState
```typescript
type Props = {
  user: User,
  onUpdate: (user: User) => void
};

export const UserInfo: React.FC<Props> = ({ user, onUpdate }) => {
  const [message, setMessage] = useState(''); // string is obvious
  const [name, setName] = useState(user.name); // string is taken from user.name
  const [age, setAge] = useState(user.age); // number is taken from user.age
  const [user, setUser] = useState<User | null>(null); // we set type manually
  const [friends, setFriends] = useState<User[]>([]); // we set type manually
  
  // ...
}
```

## useEffect
```typescript
const App = () => {
  const [query, setQuery] = useState('');
  const [userId, setUserId] = useState(0);

  useEffect(() => {
    // this function is called after each render
    // Because the dependencies array is not passed
  });

  useEffect(() => {
    // componentDidMount
    // this function is called once after the first render
    // Because the dependencies array is empty
    
    return () => {
      // componentWillUnmount
      // this function is called before the component is removed from the page
    };
  }, []);

  useEffect(() => {
    // this function is called after each render 
    // if `query` or `userId` is changed

    return () => {
      // this functions is called before every effect call 
      // if `query` or `userId` is changed
    } 
  }, [query, userId]);
};
```

## Forms events
```tsx
const App: React.FC = () => {
  const [title, setTitle] = useState('');
  const [userId, setUserId] = useState(0);

  const handleTitleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    setTitle(event.target.value); // or event.currentTarget.value
  };

  const handleUserChange = (event: React.ChangeEvent<HTMLSelectElement>) => {
    setUserId(+event.target.value);
  };

  const handleSubmit = (event: React.FormEvent) => {
    event.preventDefault();
    console.log('submit');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input onChange={handleTitleChange} />

      <select onChange={handleUserChange}>
        <option value="" disabled>Choose a user</option>

        {users.map(user => (
          <option value={user.id} key={{user.id}}>
            {user.name}
          </option>
        ))}
      </select>
    
      <button type="submit">Save</button>
    </form>
  );
}
```

## Fetching data
```typescript
interface User {
  id: number;
  name: string;
}

export function getUser(userId: number): Promise<User> {
  return fetch(`${BASE_URL}/users/${userId}`)
    .then(response => response.json())
}

getUser(123)
  .then(user => {
    console.log(user.id, user.name);
  })
```
or
```typescript
export const getUser = async (userId: number): Promise<User> => {
  const response = await fetch(`${BASE_URL}/users/${userId}`);

  return response.json();
};
```

```typescript
function getAll<T>(url: strnig): Promise<T[]> {
  return fetch(BASE_URL + url)
    .then(response => response.json());
}

getAll<User>('/users'); // Promise<User[]>
getAll<Todo>('/todos'); // Promise<Todo[]>
```
