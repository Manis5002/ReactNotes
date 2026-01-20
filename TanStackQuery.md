# TanStack Query: Intermediate to Advanced Guide

## 1. Core Concepts: Caching & Query Keys

### The Cache
TanStack Query is not just a fetching hook; it is an intelligent caching layer.
*   **Structure:** It stores data like an object.
    *   **Key:** The `queryKey` (unique identifier).
    *   **Value:** The response data + metadata (status, timestamp, etc.).
*   **Mechanism:** When `useQuery` is called:
    1.  It checks the cache for the provided `queryKey`.
    2.  If the key exists, it evaluates if the data is **Fresh** or **Stale**.
    3.  If **Fresh**, it returns cached data immediately without a network request.
    4.  If **Stale** (or doesn't exist), it fetches new data from the backend.

### Fresh vs. Stale Data
*   **Default Behavior:** By default, data is marked **Stale** immediately (0ms). This means every component mount triggers a background refetch, even if data exists in the cache.
*   **`staleTime`:** The duration (in milliseconds) data remains "Fresh."
    *   **Usage:** Saves network resources by preventing refetches until the time expires.
    *   **Infinity:** `staleTime: Infinity` means data never automatically refetches.

**Code Example:**
```typescript
// Data remains fresh for 1 minute (60,000ms)
const { data } = useQuery({
  queryKey: ['users'],
  queryFn: getUsers,
  staleTime: 60000, 
});
```

---

## 2. Query Invalidation (CRUD Operations)

When you modify data (POST, DELETE, PUT), the local cache becomes outdated. You must tell TanStack Query the data is invalid to trigger a refresh.

*   **Invalidate Queries:** Marks a specific query key as stale.
*   **Behavior:**
    1.  Marks data as stale.
    2.  If the data is currently being used (active on screen), it triggers an immediate refetch.
*   **Best Practice:** Always `await` the mutation/API call before invalidating to ensure the backend has processed the change.

**Code Example:**
```typescript
const queryClient = useQueryClient();

const handleCreateUser = async (newUser) => {
  // 1. Await the backend operation
  await createUser(newUser);
  
  // 2. Invalidate to trigger a refetch of the list
  queryClient.invalidateQueries({ queryKey: ['users'] });
};

// Note: You can await invalidateQueries if you need the fresh data immediately for subsequent logic.
```

---

## 3. Mutations (`useMutation`)

Used for creating, updating, or deleting data. While you can use standard API calls, `useMutation` provides lifecycle hooks and state management.

### Key Features
*   **`mutate`:** The function to trigger the mutation.
*   **Lifecycle Callbacks:**
    *   `onSuccess`: Runs if the request succeeds (ideal for invalidating queries).
    *   `onError`: Runs if the request fails.
    *   `onSettled`: Runs regardless of success or failure.
    *   `onMutate`: Runs immediately (used for Optimistic Updates).
*   **State:** Provides `isPending`, `isError`, etc.

**Code Example:**
```typescript
const mutation = useMutation({
  mutationFn: (user: UserType) => createUser(user),
  onSuccess: () => {
    // Handling invalidation centrally in the mutation
    queryClient.invalidateQueries({ queryKey: ['users'] });
  },
  onError: (error) => {
    console.error("Mutation failed", error);
  },
  onSettled: (data, error) => {
    console.log("Mutation finished");
  }
});

// Usage
<button onClick={() => mutation.mutate(newUserObject)}>Create</button>
```
*Note: If you are not using the callbacks or loading states, a simple async function + `invalidateQueries` is a valid alternative to `useMutation`.*

---

## 4. Dynamic & Reusable Query Options

To make code maintainable and flexible (e.g., enabling a query in one component but not another), use a "Query Option Factory" pattern.

### Implementation Strategy
1.  Create a separate file/function that returns the query options object.
2.  Accept arguments for API parameters (like `page`, `limit`).
3.  Accept a second argument for TanStack specific options (`enabled`, `staleTime`).
4.  Use TypeScript to ensure type safety, omitting `queryKey` and `queryFn` from the passed options to prevent overrides.

**Code Example:**
```typescript
import { UseQueryOptions } from '@tanstack/react-query';

export function createUsersQueryOptions(
  params: { page: number }, 
  options?: Omit<UseQueryOptions<GetUsersResponse, Error, GetUsersResponse>, 'queryKey' | 'queryFn'>
) {
  return {
    queryKey: ['users', params],
    queryFn: () => getUsers(params),
    ...options, // Spread dynamic options here
  };
}

// Usage in Component
const { data } = useQuery(
  createUsersQueryOptions({ page: 1 }, { enabled: !!userIsLoggedIn, staleTime: 5000 })
);
```

---

## 5. Infinite Querying & Pagination

Used when the backend returns paginated data (pages of results).

### Prerequisites
*   The backend must support pagination (usually via `limit` and `page` query parameters).
*   Hook: `useInfiniteQuery`.

### Configuration
1.  **`initialPageParam`:** The starting page index (usually 1 or 0).
2.  **`getNextPageParam`:** Logic to determine the next page index.
    *   Receives `lastPage` (response from the last fetch).
    *   Return `undefined` if there is no more data (stops the query).
    *   Otherwise, return `lastPage.currentPage + 1`.
3.  **Query Function:** Must accept `{ pageParam }` and pass it to the API call.

### Accessing Data
*   **`data.pages`:** An array where each index contains the response object for that page.
*   **Flattening:** To show a "Load More" list, use `.flatMap()` to combine all pages into one array.
*   **Utilities:**
    *   `fetchNextPage()`: Function to load the next batch.
    *   `hasNextPage`: Boolean.
    *   `isFetchingNextPage`: Boolean (for loading spinners).

**Code Example:**
```typescript
const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
  queryKey: ['users', 'infinite'],
  initialPageParam: 1,
  queryFn: ({ pageParam }) => getUsers({ page: pageParam, limit: 10 }),
  getNextPageParam: (lastPage) => {
    // Check if backend says there are more pages
    return lastPage.hasMore ? lastPage.currentPage + 1 : undefined;
  }
});

// Flattening for UI
const allUsers = data?.pages.flatMap((page) => page.users) || [];
```

---

## 6. Advanced Query Options

### `select`
Transforms or filters data *after* fetching but *before* it is returned to the component.
*   **Use Cases:** extracting specific properties, sorting arrays, filtering list.
*   **Benefit:** Keeps the component logic clean and can optimize re-renders (if the selected data doesn't change, the component won't re-render).

```typescript
// Example: Sort users by date
select: (data) => data.users.sort((a, b) => a.createdAt - b.createdAt)
```

### `refetchInterval` (Polling)
Automatically refetches data at a specific interval.
*   **Use Case:** Real-time data (stock prices, live chats).
*   **Usage:** Pass a number (ms) or a function returning a number.
*   **Note:** Only runs when the window/component is active.

### `refetchOnWindowFocus`
*   **Default:** `true`. Refetches data when the user switches tabs and comes back.
*   **Interaction with `staleTime`:** If `staleTime` is set, `refetchOnWindowFocus` will **not** trigger if the data is still fresh.
*   **Override:** Set to `'always'` to force refetch on focus regardless of stale time.

### `placeholderData`
Data shown temporarily while the real query is loading. It is **not** cached.
*   **Preventing Flicker:** A common pattern for pagination is keeping the *previous* page's data while the next page loads.
*   **Code:** `placeholderData: (previousData) => previousData`. This keeps the list visible instead of showing a loading spinner when changing pages.

### `initialData`
Pre-populates the cache with data. Unlike placeholder data, this **is cached** and persists.
*   **Behavior:** It is treated as "real" data. By default, it is marked stale immediately unless `staleTime` is provided.
*   **Optimization:** You can set `initialData` by searching the cache of *other* queries.
    *   *Example:* If you have a list of users, and you click on one user, you can use the data from the "list" query to immediately populate the "single user" query.

---

## 7. Typing & Zod
*   **Validation:** It is highly recommended to use **Zod** to validate backend responses at runtime.
*   **Types:** If not using Zod, explicitly type the return of the `queryFn` (e.g., `Promise<UserResponse>`) so TanStack Query can infer the `data` type in the component.

---

## 8. Prefetching
Fetches data before the user actually navigates or clicks, making the app feel instant.

*   **Trigger:** Often used on `onMouseEnter` (hover).
*   **Method:** `queryClient.prefetchQuery`.
*   **Logic:** Assumes if a user hovers a button, they are likely to click it.

**Code Example:**
```typescript
const handleMouseEnter = () => {
  queryClient.prefetchQuery({
    queryKey: ['users'],
    queryFn: getUsers,
  });
};

<button onMouseEnter={handleMouseEnter}>Show Users</button>
```
