# Do: Best Practices

## ✅ Compose with children
```tsx
// Good: composition over configuration
<Card>
  <Card.Header>
    <Card.Title>Title</Card.Title>
  </Card.Header>
  <Card.Body>Content here</Card.Body>
</Card>
```

## ✅ Export Props interface
```tsx
export interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
}
export function Button({ variant = 'primary' }: ButtonProps) { ... }
```

## ✅ Use forwardRef for element wrappers
```tsx
export const Input = forwardRef<HTMLInputElement, InputProps>((props, ref) => (
  <input ref={ref} {...props} />
));
Input.displayName = 'Input';
```

## ✅ Derive state from props
```tsx
// Good: derived directly
function UserList({ users }: { users: User[] }) {
  const activeCount = users.filter(u => u.active).length;
  // No useState needed!
}
```

## ✅ Memoize callbacks for React.memo children
```tsx
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);

return <ExpensiveChild onClick={handleClick} />;
```

## ✅ Handle all states
```tsx
function UserTable({ users, isLoading, error }: Props) {
  if (isLoading) return <Skeleton />;
  if (error) return <ErrorBanner message={error} onRetry={refetch} />;
  if (users.length === 0) return <EmptyState message="No users yet" action={<AddUserButton />} />;
  return <Table data={users} />;
}
```
