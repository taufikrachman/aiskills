# React Native Patterns

Build React Native apps with TypeScript. Production patterns.

## Rules

### 1. Component Structure
```tsx
// Screen component pattern
export function ProductListScreen() {
  const { data, isLoading, error } = useProducts();
  if (isLoading) return <LoadingSkeleton />;
  if (error) return <ErrorBanner error={error} onRetry={refetch} />;
  return <FlatList data={data} renderItem={renderProduct} />;
}
```
- Use React Query (TanStack Query) for data fetching.
- Always handle loading, error, and empty states.
- Use `FlatList`/`SectionList` for scrollable content (virtualized).

### 2. Navigation (React Navigation)
```tsx
// Stack for linear flows, Tab for main sections
const Stack = createNativeStackNavigator();
function RootNavigator() {
  return (
    <Stack.Navigator screenOptions={{ headerShown: false }}>
      <Stack.Screen name="Main" component={MainTabs} />
      <Stack.Screen name="ProductDetail" component={ProductDetail} />
      <Stack.Screen name="Checkout" component={Checkout} />
    </Stack.Navigator>
  );
}
```
- Type-safe navigation with typed route params.
- Deep linking: configure `linking` for universal links.

### 3. Platform Differences
- iOS SafeArea: `useSafeAreaInsets()` or `<SafeAreaView>`.
- Keyboard avoidance: `KeyboardAvoidingView behavior="padding"`.
- Status bar: `<StatusBar barStyle="dark-content" />`.
- Platform-specific files: `Component.ios.tsx`, `Component.android.tsx`.

### 4. Performance
- `React.memo` for list items. `useCallback` for handlers.
- Hermes JS engine (default in RN 0.70+).
- Image optimization: use `fast-image` for caching.
- Avoid inline functions in FlatList renderItem.

## Anti-Patterns
- ❌ `ScrollView` for long lists (use FlatList)
- ❌ No keyboard avoidance on forms
- ❌ Hardcoded dimensions (use `useWindowDimensions`)
