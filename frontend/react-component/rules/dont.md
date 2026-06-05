# Don't: Anti-Patterns

## ❌ Using index as key
```tsx
// Bad: index resets on add/remove
{items.map((item, index) => <Item key={index} {...item} />)}

// Good: stable unique id
{items.map(item => <Item key={item.id} {...item} />)}
```

## ❌ useEffect for derived state
```tsx
// Bad: unnecessary effect
const [fullName, setFullName] = useState('');
useEffect(() => { setFullName(`${first} ${last}`); }, [first, last]);

// Good: compute inline
const fullName = `${first} ${last}`;
```

## ❌ Storing derivable values in state
```tsx
// Bad: duplicate state
const [items, setItems] = useState(data);
const [count, setCount] = useState(data.length);

// Good: derive from source
const [items, setItems] = useState(data);
const count = items.length;
```

## ❌ Mutation
```tsx
// Bad: direct mutation
state.items.push(newItem);
setState({ ...state });

// Good: immutable update
setState(prev => ({ ...prev, items: [...prev.items, newItem] }));
```

## ❌ Skipping accessibility
```tsx
// Bad: no label
<div onClick={handleClick}>×</div>

// Good: accessible button
<button onClick={handleClick} aria-label="Close dialog">×</button>
```

## ❌ Inline objects in JSX
```tsx
// Bad: new object every render → forces re-render
<ExpensiveList style={{ padding: 16 }} />

// Good: constant reference
const listStyle = { padding: 16 };
<ExpensiveList style={listStyle} />
```

## ❌ Nested ternaries beyond 2 levels
```tsx
// Bad
const color = status === 'active' ? 'green' : status === 'pending' ? 'yellow' : status === 'error' ? 'red' : 'gray';

// Good: map or switch
const colorMap = { active: 'green', pending: 'yellow', error: 'red' };
const color = colorMap[status] ?? 'gray';
```
