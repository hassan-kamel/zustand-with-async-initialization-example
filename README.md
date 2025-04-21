# Zustand with Async Initialization Example

Based on your question, here's an example of how to initialize a Zustand store with async data while avoiding hydration issues:

```tsx
import { createStore } from 'zustand'
import { useQuery } from '@tanstack/react-query'
import { useStore } from 'zustand'

// 1. Create context for our store
const BearStoreContext = React.createContext(null)

// 2. Create provider component with async initialization
const BearStoreProvider = ({ children }) => {
  // Fetch data asynchronously
  const { data, isPending } = useQuery({
    queryKey: ['bears'],
    queryFn: async () => {
      const response = await fetch('/api/bears')
      return response.json()
    }
  })

  // Create store only when data is available
  const [store] = React.useState(() => {
    if (!data) return null
    return createStore((set) => ({
      bears: data.initialBears,
      actions: {
        increasePopulation: (by) => set((state) => ({ bears: state.bears + by })),
        removeAllBears: () => set({ bears: 0 }),
      },
    }))
  })

  // Show loading state while data is being fetched
  if (isPending || !store) {
    return <div>Loading bears...</div>
  }

  return (
    <BearStoreContext.Provider value={store}>
      {children}
    </BearStoreContext.Provider>
  )
}

// 3. Custom hook to access the store
const useBearStore = (selector) => {
  const store = React.useContext(BearStoreContext)
  if (!store) {
    throw new Error('Missing BearStoreProvider')
  }
  return useStore(store, selector)
}

// 4. Usage in app
const App = () => {
  return (
    <BearStoreProvider>
      <BearCounter />
    </BearStoreProvider>
  )
}

const BearCounter = () => {
  const bears = useBearStore((state) => state.bears)
  return <div>Number of bears: {bears}</div>
}
```

## Key Points:

1. **Async Initialization**: We use React Query to fetch data before creating the store
2. **Hydration Safety**: The provider won't render children until data is loaded
3. **Server-Side Rendering**: On the server, only the loading state will render
4. **Client-Side**: Once hydrated, the data loads and the store initializes

## Alternative Approach (Suspense):

If you're using React Suspense, you could do:

```tsx
const BearStoreProvider = ({ children }) => {
  const { data } = useQuery({
    queryKey: ['bears'],
    queryFn: fetchBears,
    suspense: true
  })

  const [store] = React.useState(() => createStore(/* ... */))
  
  return (
    <BearStoreContext.Provider value={store}>
      {children}
    </BearStoreContext.Provider>
  )
}

// Then wrap with Suspense boundary
const App = () => (
  <Suspense fallback="Loading...">
    <BearStoreProvider>
      <BearCounter />
    </BearStoreProvider>
  </Suspense>
)
```

This approach ensures:
- No hydration mismatches
- Clean separation of async data fetching and state management
- Reusable store instances when needed
- Proper loading states during data fetching