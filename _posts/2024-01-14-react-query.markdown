---
layout:     post
title:      "React Query"
subtitle:   "리액트 쿼리 시작하기"
date:       2024-01-14 22:40:00
author:     "Hyunz"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Frontend
    - React
    - ReactQuery
---
> 리액트 쿼리는 서버 상태를 관리하는 라이브러리입니다. 클라이언트 상태와 분리하여 관리할 수 있습니다.
   

## React Query ?

서버 상태를 관리하는 라이브러리 

React에서 데이터의 fetching, caching, synchronizing, updating server state를 위한 라이브러리

- React Query는 서버에게 요청을 보내는 Query와 요청에 대한 응답(서버 데이터)를 캐싱하고 서버 데이터에 대한 적절한 핸들링 등 여러 편리한 기능들을 제공해줌
- React Query는 클라이언트에서 쿼리와 쿼리에 대한 서버 데이터를 **클라이언트 캐시로 관리**합니다. React 코드에서 서버 데이터가 필요할 때 Fetch API나 Axios를 통해 바로 서버에게 요청을 보내지 않고 React Query 캐시를 요청합니다.
- React Query 클라이언트를 어떻게 구성했느냐에 따라 캐시의 데이터를 유지 관리합니다. 데이터를 관리하는 것은 React Query이지만 서버의 새 데이터로 캐시를 업데이트하는 시기를 설정하는 것은 사용자의 몫입니다.


### **서버 상태(Server State)란?**

클라이언트에 표시하는 데 필요한 서버 데이터. 즉, DB에 존재하는 데이터

### **React Query 장점**

1. **모든 쿼리의 로딩 및 오류 상태를 유지**해주기 때문에 수동으로 다루어야 할 필요가 없어집니다.

2. 사용자를 위해 데이터의 Pagination 또는 Infinity Scroll일 때 필요한 경우에 데이터를 가져올 수 있도록하는 **Lazy loading 도구도 제공**됩니다.

3. **서버 데이터를 미리 pre-fetch하고 캐싱**하여 사용자가 데이터를 필요로할 때 pre-fetch되어 캐싱된 데이터를 제공하기 때문에 요청과 응답에 대한 시간을 줄일 수도 있습니다.

4. React Query를 통해 **서버의 데이터를 업데이트(Mutation)하여 관리**할 수도 있습니다.

5. 각 쿼리들은 키로 식별되기 때문에 페이지 내 여러 컴포넌트가 **동일한 데이터를 요청하는 경우 한 번만 요청하도록 중복 요청을 제거**할 수 있습니다.

6. 서버에서 **오류가 발생하는 경우에 대한 재시도를 관리**할 수 있습니다.

7. 쿼리가 성공하거나 오류가 났을 때를 구별하여 **특정 조건에 따라 실행될 콜백함수를 전달**할 수도 있습니다.

### Install React Query

```tsx
npm i @tanstack/react-query 
npm i -D @tanstack/eslint-plugin-query 
```

### React Query 셋팅

최상단에 `QueryClientProvider` 셋팅

```tsx
function App() {
    const queryClient = new QueryClient();
    return (
        <div>
            <QueryClientProvider client={queryClient}>
                <App/>
            </QueryClientProvider>
        </div>
    );
}
```



### ReactQueryDevTools
- React Query Devtools로 리액트 쿼리 개발자도구 사용 가능

- `npm i @tanstack/react-query-devtools`로 리액트쿼리 개발자도구 설치

- 상위 컴포넌트에 해당 컴포넌트 추가

```tsx
<ReactQueryDevtools initialIsOpen={true} buttonPosition='top-right' position='left'/>
```


### Query 상태

**LifeCycle(라이프 사이클)**

React Query를 통해 관리되는 쿼리 데이터는 라이프 사이클에 따라 각 상태를 가진다.

- fetching : 요청중인 쿼리
- fresh : 유효한 쿼리. 컴포넌트가 마운트, 업데이트 되어도 데이터를 재요청하지 않음
- stale : 만료된 쿼리. 컴포넌트가 마운트, 업데이트되면 데이터를 다시 요청한다. 만료되었더라도 캐시되어 있으면 캐시에 남아있다.
- inactive : 사용하지 않는 쿼리. 일정 시간이 지나면 가비지 컬렉터가 캐시에서 제거
- delete : 캐시에서 제거된 쿼리.

inactive -> fetching -> fresh -> stale 

 `refetchOnMount` / `refetchOnWindowFocus`/`refetchOnReconnect` 가 true이면 Mount / window focus / reconect시점에 data가 stale이라고 판단되어 refetch한다. false이면 캐시 데이터를 가져올 수도 있다. (기본값:true)

![reactquery-lifecycle](/img/in-post/reactquery-lifecycle.png)
출처 : https://sooleeandtomas.tistory.com/59 




## useQuery

api를 GET 메소드로 조회하려할 때 사용

`queryKey`로 구분해서 data fetching 수행

- **{ isLoading, isFetching, data, isError, error, refetch }** : 쿼리 결과를 반환함
- **queryKey** : useQuery의 key. 해당 key로 쿼리를 구분하고, 적재되어 있는게 있으면 api를 호출하지 않고 데이터를 가져옴.
    - 배열 형식으로 저장
    - 배열의 첫번째 인자는 key값. 두번째 인자부터는 쿼리 파라미터. api가 변수에 의존하면 인자로 무조건 넣어줘야 함.
- **queryFn** : 수행할 콜백함수(api 조회)
- **select** : 가져온 data 정제
- **staleTime** : 유효 기간. staleTime이 지나면 refetch(api조회)함. stale 되지 않았으면(fresh 상태임), 해당 쿼리에 대한 데이터를 가져옴. 설정해주지 않으면 기본값 0임

```jsx
function Todos() {
  const { isPending, isError, data, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodoList,
  })

  if (isPending) {
    return <span>Loading...</span>
  }

  if (isError) {
    return <span>Error: {error.message}</span>
  }

  // success 상태 이면 아래 jsx 출력
  return (
    <ul>
      {data.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  )
}
```



### 강제로 refetch

> **클릭 시 fetch**      

- `enabled` : 자동으로 query를 실행시킬지 여부          
useQuery로 데이터를 가져올 때, 기본값은 enabled:true이다.       
컴포넌트 mount 시 또는 종속 parameter 변경 시 자동으로 쿼리를 실행한다.         
enabled:false 하면 자동으로 쿼리를 실행하지 않고, `refetch()` 호출 시에만 쿼리를 실행한다



### 새로 fetch 시 이전 데이터 유지하기 (pagination에서 사용)

`placeholderData: keepPreviousData`       
- useQuery 내에서 `placeholderData: keepPreviousData` 옵션 사용       
react query v4까지는 keepPreviousData:true 였는데 v5에서 바뀜 
- 기존 → page1에서 page2를 선택하면 page2의 데이터를 가져올 때 Loading 상태가 되어서 데이터를 가지고 있지 않음.     
- 옵션 사용 → page1에서 page2를 선택하면 page2의 데이터를 가져오기 전까지 page1의 데이터를 가지고 있음



### **주기적으로 데이터 최신화해야할 때**

유저의 행위와는 별도로 refetchInterval 옵션으로 Polling 작업을 수행

- `refetchIntervalInBackground` : 백그라운드에서 데이터를 리패치 할 것인지 여부
- `refetchInterval` : 리패치 주기
아래코드 : 2초 간격으로 api를 호출하여 데이터를 리패치한다

```tsx
const { isLoading, isFetching, data, isError, error } = useQuery(
  'get-product',
  fetchProducts,
  {
    refetchInterval: 2000,
    refetchIntervalInBackground: true,
  },
)
```



### 에러 핸들링

v4까지는 useQuery의 onSuccess, onError 콜백리스너도 있었음. 

그런데, 이 콜백리스너가 더 혼란을 줄 수 있다고해서 v5에서는 없어짐. 제일 상위 컴포넌트에서 `QueryClient`인스턴스 생성할 때 옵션으로 onError 콜백함수를 정의해줄 수 있음.

```tsx
const queryClient = new QueryClient({
    queryCache: new QueryCache({
        onError: (error:any) => {
            const err = error as AxiosError;
            if (err.response?.data?.error === "invalid_token") {
                alert('토큰 만료')
            }else{
                alert('에러가 발생했습니다.');
            }
        }
    })
});
```

### **invalidateQueries**

서버에서 변경된 데이터를 내려주지 않을 수도 있다. 그러면, 클라이언트에서는 서버에 변경된 데이터를 다시 조회해서 문제를 해결할 수 있다. 즉, 클라이언트는 순서에 맞게 서버에 데이터 변경 요청을 하고, 재조회 요청을 해서 원하는 결과를 얻는 것이다.

```tsx
export const useDeleteDoc = () => {
  const queryClient = useQueryClient(); // queryClient를 가져와야 함

  return useMutation<DriveFile | string, AxiosError, string>({
    mutationKey: [DIARY_KEY],
    mutationFn: (fileId: string) => deleteDoc(fileId),
    onSuccess(data) {
      queryClient.invalidateQueries({ queryKey: [DIARY_KEY] });
    },
  });
};
```



## useMutation

`mutation` 은 query와 다르게 **생성**(POST), **수정**(PUT or PATCH), **삭제**(DELETE)와 같은 데이터를 변경하는 작업을 할 수 있도록 하는 함수다. `useMutation` 훅을 사용한다. 

- query는 컴포넌트가 마운트 될 때 즉시 실행되어 서버에서 데이터를 가져온다. 훅으로 선언만 해두면 알아서 데이터를 조회한다.
- 하지만 mutation은 **서버 데이터를 변형**시키는 것이기 때문에, 일반적으로 컴포넌트가 마운트 될 때 실행되면 안된다. 
대신, useMutation 훅을 사용해 mutation 객체를 가져오고, 명령적으로 필요할 때 호출하도록 하고 있다.
    
    useMutation 또는 mutate에서 데이터 post 성공시, 에러시 동작을 설정할 수 있다.
    

따라서 query는 사이드 이펙트를 발생시키지 않으며, 선언적으로 사용할 수 있고, mutation은 사이드 이펙트를 발생시킬 수 있으며, 명령형으로 사용할 수 있다. React Query는 이런 차이점 때문에 둘을 나누어 사용하도록 했다.

```jsx
function App() {
	// 뮤테이션 객체를 먼저 생성 후 이벤트가 일어났을 때 사용함
  const mutation = useMutation({
    mutationFn: (newTodo) => {
      return axios.post('/todos', newTodo)
    },
  })

  return (
    <div>
      {mutation.isPending ? (
        'Adding todo...'
      ) : (
        <>
            {mutation.isError ? (
            <div>An error occurred: {mutation.error.message}</div>
            ) : null}

            {mutation.isSuccess ? <div>Todo added!</div> : null}
                    
            {/* 이벤트 발생 시 사용 */}
            <button
            onClick={() => {
                mutation.mutate({ id: new Date(), title: 'Do Laundry' })
            }}
            >
            Create Todo
            </button>
        </>
      )}
    </div>
  )
}
```

위 예제처럼 mutation 객체로 받을 수도 있고, 아래처럼 mutation 상태를 따로 받을 수도 있다.

```jsx
const {isIdle, isPending, isError, isSuccess, data, error, mutate} = useMutation({
  mutationFn: (variables) => {
    return axios.post('/todos', variables)
  },
})
```

```jsx
const {
  data,
  error,
  isError,
  isIdle,
  isPending,
  isPaused,
  isSuccess,
  failureCount,
  failureReason,
  mutate,
  mutateAsync,
  reset,
  status,
  submittedAt,
  variables,
} = useMutation({
  mutationFn,
  gcTime,
  mutationKey,
  networkMode,
  onError,
  onMutate,
  onSettled,
  onSuccess,
  retry,
  retryDelay,
  throwOnError,
  meta,
})

mutate(variables, {
  onError,
  onSettled,
  onSuccess,
})
```

- isIdle, isPending, isError, isSuccess는 데이터 가져오는 상태
    - isIdle : 현재 유휴 상태이거나 재설정 상태
    - isPending : 현재 api 호출 중
    - isError : 오류 발생
    - isSuccess : api 성공적으로 호출했고, data 사용 가능한 상태
- `mutate` : mutationFn을 실행하는 함수. `mutate()` 로 사용 가능
- `mutationFn` : api요청하는 함수
- `mutationKey` : useQuery의 queryKey와 같은 역할
- `onMutate` : mutationFn 실행되기 전에 먼저 실행되는 함수. Optimistic update 적용할 때 유용하다
    - Optimistic update : 낙관적으로 성공할 것이라 보고 ui적으로 변경하는 것. ex) 좋아요 버튼 클릭 시 미리 ui 변경
- onError(실패 시), onSuccess(성공 시), onSettled(실패 또는 성공 시)는 api 호출 완료 후 실행되는 콜백함수.


       

### 쿼리 무효화 : invalidateQueries

쿼리를 다시 가져오기 전에 쿼리에 대한 데이터가 무효까지 기다리지 않고 수동으로 무효하도록 설정할 수 있다.

```jsx
// 캐시에 있는 모든 쿼리 무효화
queryClient.invalidateQueries()
// 해당 쿼리키에 대한 쿼리 무효화
queryClient.invalidateQueries({ queryKey: ['todos'] })
```

useMutation의 콜백 함수 내에서도 쿼리 무효화할 수 있다. 쿼리 무효화하면, 렌더링 중인 데이터였다면 새로 api를 호출하여 데이터를 fetch하고, 렌더링 중이지 않고 있으면 데이터가 캐시에서 없어진다

```jsx
import { useMutation, useQueryClient } from '@tanstack/react-query'

const queryClient = useQueryClient()

// When this mutation succeeds, invalidate any queries with the `todos` or `reminders` query key
const mutation = useMutation({
  mutationFn: addTodo,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['todos'] })
    queryClient.invalidateQueries({ queryKey: ['reminders'] })
  },
})
```

       

### 데이터 업데이트 : setQueryData

useMutation에서 받은 데이터를 setQueryData를 통해 기존 쿼리를 새 데이터로 업데이트할 수 있다

```jsx
const queryClient = useQueryClient()

const mutation = useMutation({
  mutationFn: editTodo,
  onSuccess: (data) => {
    queryClient.setQueryData(['todo', { id: 5 }], data)
  },
})

mutation.mutate({
  id: 5,
  name: 'Do the laundry',
})

// 해당 쿼리는 mutation의 결과값으로 추후 업데이트 됨
const { status, data, error } = useQuery({
  queryKey: ['todo', { id: 5 }],
  queryFn: fetchTodoById,
})
```

       

### useInfiniteQuery : 무한스크롤

```jsx
const {
  fetchNextPage,
  fetchPreviousPage,
  hasNextPage,
  hasPreviousPage,
  isFetchingNextPage,
  isFetchingPreviousPage,
  ...result
} = useInfiniteQuery({
  queryKey,
  queryFn: ({ pageParam }) => fetchPage(pageParam),
  initialPageParam: 1,
  ...options,
  getNextPageParam: (lastPage, allPages, lastPageParam, allPageParams) =>
    lastPage.nextCursor,
  getPreviousPageParam: (firstPage, allPages, firstPageParam, allPageParams) =>
    firstPage.prevCursor,
})
```



> reference         
- [tanstack query](https://tanstack.com/query/v5)
- [https://abangpa1ace.tistory.com/entry/React-Query-1-React-Query-입문하기](https://abangpa1ace.tistory.com/entry/React-Query-1-React-Query-%EC%9E%85%EB%AC%B8%ED%95%98%EA%B8%B0) 
- [https://mycodings.fly.dev/blog/2023-09-17-react-query-cachetime-staletime-refetch-poll](https://mycodings.fly.dev/blog/2023-09-17-react-query-cachetime-staletime-refetch-poll) 
- [https://pozafly.github.io/react-query/mutation-after-data-update/](https://pozafly.github.io/react-query/mutation-after-data-update/) 
- [https://gusrb3164.github.io/web/2022/01/23/react-query-typescript/](https://gusrb3164.github.io/web/2022/01/23/react-query-typescript/) 
- [https://velog.io/@familyman80/React-Query-한글-메뉴얼](https://velog.io/@familyman80/React-Query-%ED%95%9C%EA%B8%80-%EB%A9%94%EB%89%B4%EC%96%BC)