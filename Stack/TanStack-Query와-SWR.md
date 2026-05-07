# TanStack Query와 SWR

React에서 서버에서 데이터를 가져올 때, `useEffect` 안에 `fetch`를 넣고 `useState`로 로딩·에러·데이터를 직접 관리할 수 있다. 하지만 다음이 반복되면 코드가 길어지고 버그가 나기 쉽다.

- 같은 API를 여러 컴포넌트에서 호출해 **중복 요청**이 남
- **캐시**가 없어 탭만 바꿔도 매번 다시 불러옴
- **로딩 / 에러 / 성공** 상태를 매번 손으로 맞춤
- 데이터를 **갱신**할 때(수정 후 목록 다시 불러오기 등) 흐름이 흩어짐

**TanStack Query**와 **SWR**은 이런 서버에서 온 데이터(서버 상태)를 **훅과 캐시**로 정리해 주는 라이브러리다. 

---

## 1. 용어 정리

- **서버 상태**: 
  - API 응답처럼 **마음대로 바꿀 수 없고** 네트워크를 통해서만 최신이 되는 데이터다. (반면 버튼 눌림 같은 건 **클라이언트 상태**에 가깝다.)
- **캐시**: 
  - 한 번 받아 둔 응답을 키(주소 같은 것)에 묶어 저장해 두었다가 같은 키로 요청하면 네트워크 없이 먼저 보여 줄 수 있다.
  - 캐시 무효화 : 다시 fetch 하도록 만드는 거 
- **Stale-While-Revalidate**: 
  - 옛날 데이터라도 **일단 화면에 보여 주고** 뒤에서 **조용히 다시 받아서** 맞춘다!는 전략이다. SWR 이름이 이 전략에서 왔고 TanStack Query도 비슷한 감각으로 동작할 수 있다.

---

## 2. TanStack Query (구 React Query)

### 2.1  무엇을 해 주나

쿼리 키(Query Key) 단위로 데이터를 캐시하고 언제 새로 가져올지 / 언제 버릴지 / 수정 후 어떻게 다시 맞출지를 라이브러리가 맡아 준다.  
`GET`뿐 아니라 `POST`/`PATCH` 같은 변경(Mutation)과 캐시 무효화를 한 흐름으로 묶기 좋게 설계되어 있다.

### 2.2 설치와 최소 설정 (React)

```bash
npm install @tanstack/react-query
```

앱 최상단에 `QueryClient`와 `QueryClientProvider`를 둔다.

```tsx
// main.tsx 또는 App.tsx 근처
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // 전역 기본값 예: 1분 동안은 신선해서 자동 재요청 안 함 (프로젝트마다 조정)
      staleTime: 60 * 1000,
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      {/* 라우터, 페이지 등 */}
    </QueryClientProvider>
  );
}
```

### 2.3 `useQuery` — 데이터 가져오기

**쿼리 키**는 보통 `배열`로 쓴다. `['user', userId]`처럼 **의미 단위**로 나누면 나중에 `userId`만 바뀔 때 캐시가 구분된다!

```tsx
import { useQuery } from "@tanstack/react-query";

async function fetchUser(id: string) {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error("유저를 불러오지 못했습니다.");
  return res.json();
}

function UserProfile({ userId }: { userId: string }) {
  const { data, isPending, isError, error, refetch } = useQuery({
    queryKey: ["user", userId],
    queryFn: () => fetchUser(userId),
    // 이 쿼리만 staleTime을 다르게 줄 수도 있다
    staleTime: 30 * 1000,
  });

  // TanStack Query v5 기준: 첫 로딩은 isPending. v4를 쓰면 문서의 isLoading 등을 확인하면 된다.
  if (isPending) return <p>불러오는 중…</p>;
  if (isError) return <p>에러: {(error as Error).message}</p>;

  return (
    <div>
      <p>{data.name}</p>
      <button type="button" onClick={() => refetch()}>
        다시 불러오기
      </button>
    </div>
  );
}
```

 

- `queryKey`가 같으면 여러 컴포넌트가 `같은 캐시`를 본다. 한곳에서 받아 두면 다른 곳은 네트워크 없이 바로 그 데이터를 쓸 수 있다.
- `isPending`은 v5에서 아직 데이터 없음(첫 로딩)에 가깝게 쓴다. (버전에 따라 `isLoading` 등 이름이 문서와 맞춰져 있으니 공식 문서를 보면 된다.)
- `staleTime` 동안은 데이터를fresh하다고 보아, mount 시 자동 재요청을 줄이는 데 쓴다. `staleTime`이 지나면 **stale**이 되고, 탭 전환·포커스 등(기본 옵션)에 따라 **백그라운드에서 다시** 가져올 수 있다.

### 2.4 `useMutation` + `invalidateQueries` — 수정 후 목록 갱신 (like 새로 고침..)

게시글을 삭제한 뒤 **목록 쿼리만 다시** 맞추고 싶을 때가 많다. 이때 키를 무효화(invalidate)하면 이 키들은 오래됐다고 표시되고 활성 쿼리는 다시 `queryFn`이 돈다.

```tsx
import { useMutation, useQueryClient } from "@tanstack/react-query";

function DeletePostButton({ postId }: { postId: string }) {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: async () => {
      const res = await fetch(`/api/posts/${postId}`, { method: "DELETE" });
      if (!res.ok) throw new Error("삭제 실패");
    },
    onSuccess: () => {
      // 목록이 ['posts'] 키로 캐시되어 있다고 가정
      queryClient.invalidateQueries({ queryKey: ["posts"] });
    },
  });

  return (
    <button
      type="button"
      disabled={mutation.isPending}
      onClick={() => mutation.mutate()}
    >
      {mutation.isPending ? "삭제 중…" : "삭제"}
    </button>
  );
}
```

**이해 포인트**

- 삭제 API 호출과 화면의 posts 캐시를 최신으로를 **한곳에서 연결**할 수 있다.
- `invalidateQueries`는 **패턴**도 지정할 수 있어, `['posts', { type: 'draft' }]`처럼 세분화된 무효화도 가능하다.

### 2.5 (참고) 의존 쿼리 · 무한 스크롤

- **의존 쿼리**: `userId`가 있을 때만 `['settings', userId]`를 불러오게 `enabled: !!userId` 같은 옵션으로 순서를 제어한다.
- **무한 스크롤**: `useInfiniteQuery`로 페이지를 누적해 붙이는 패턴이 문서화되어 있다.

규모가 커질수록 이런 패턴을 **공식 API 하나로** 가져가기 쉬운 점이 TanStack Query의 강점이다.

---

## 3. SWR (Stale-While-Revalidate)

### 3.1 WHAT

**`useSWR(키, fetcher)`로 키 하나 = 캐시 하나를 만들고 캐시가 있으면 먼저 보여 준 뒤 필요할 때 백그라운드에서 다시 검증(revalidate)한다.**  
API 면적은 TanStack Query보다 **얇고 단순**한 편 👉 읽기 위주 화면에 빠르게 얹기 좋다

### 3.2 설치와 최소 예시

```bash
npm install swr
```

Provider 없이도 `useSWR`만으로 시작할 수 있다. (전역 설정은 `SWRConfig`로 줄 수 있다.)

```tsx
import useSWR from "swr";

const fetcher = (url: string) =>
  fetch(url).then((res) => {
    if (!res.ok) throw new Error("요청 실패");
    return res.json();
  });

function UserProfile({ userId }: { userId: string }) {
  const { data, error, isLoading, mutate } = useSWR(
    userId ? `/api/users/${userId}` : null,
    fetcher,
    {
      revalidateOnFocus: true,
      dedupingInterval: 2000,
    }
  );

  if (error) return <p>에러가 났습니다.</p>;
  if (isLoading) return <p>불러오는 중…</p>;
  if (!data) return null;

  return (
    <div>
      <p>{data.name}</p>
      <button type="button" onClick={() => mutate()}>
        다시 불러오기 (mutate = 재검증)
      </button>
    </div>
  );
}
```

**이해 포인트**

- 첫 인자가 `null`이면 **요청 안 함**. (TanStack Query의 `enabled: false`와 비슷한 용도로 `userId`가 없을 때 쓴다.)
- `mutate()`는 **같은 키의 데이터를 다시 검증**하는 데 자주 쓴다.
- `dedupingInterval` 안에서는 같은 키로 **중복 요청을 합친다**는 식으로 동작해, 실수로 연타되는 요청을 줄이는 데 도움이 된다.

### 3.3 `mutate`로 낙관적 업데이트 (서버 응답 전에 UI 먼저 반영)

서버에 `POST` 요청을 내면서 화면은 먼저 바꾸고 실패 시 되돌리려면 `optimisticData`와 `rollbackOnError`를 같이 쓰는 패턴이 흔하다. (여러 키를 한 번에 맞추거나 롤백을 세밀하게 하려면 TanStack Query의 mutation 쪽이 더 정리되기 쉬운 편이다.)

```tsx
import useSWR from "swr";

function ToggleLike({ postId }: { postId: string }) {
  const key = `/api/posts/${postId}`;
  const { data, mutate } = useSWR(key, fetcher);

  const liked = data?.liked ?? false;

  const onClick = () => {
    const next = { ...data, liked: !liked };

    mutate(
      async () => {
        const res = await fetch(`/api/posts/${postId}/like`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ liked: !liked }),
        });
        if (!res.ok) throw new Error("좋아요 실패");
        return res.json();
      },
      {
        optimisticData: next,
        rollbackOnError: true,
        populateCache: true,
        // 서버가 이미 최종 JSON을 주면 false로 두고 캐시만 맞춰도 된다
        revalidate: true,
      }
    );
  };

  return (
    <button type="button" onClick={onClick}>
      {liked ? "좋아요 취소" : "좋아요"}
    </button>
  );
}
```

목록 캐시까지 한 번에 갱신하려면 `useSWRConfig`의 `mutate("/api/posts")`처럼 **다른 키**를 이어서 호출하면 된다.

### 3.4 POST / DELETE는?

SWR은 기본이 **키 + GET fetch** 패턴이다. `POST`는 보통 **그냥 `fetch`를 호출한 뒤**, 성공하면 `mutate(관련키)`로 목록을 다시 검증하는 식으로 짜는 경우가 많다. TanStack Query처럼 `useMutation`이 한 세트로 있는 건 아니다.

```tsx
async function deletePost(postId: string) {
  const res = await fetch(`/api/posts/${postId}`, { method: "DELETE" });
  if (!res.ok) throw new Error("삭제 실패");
}

function DeletePostButton({ postId }: { postId: string }) {
  const { mutate } = useSWRConfig();

  return (
    <button
      type="button"
      onClick={async () => {
        await deletePost(postId);
        await mutate("/api/posts"); // 목록 키를 알고 있다면 재검증
      }}
    >
      삭제
    </button>
  );
}
```

---

## 4. 비교

**목표**: `/api/posts`에서 목록을 가져와 보여 준다.

### TanStack Query

```tsx
const { data, isPending, error } = useQuery({
  queryKey: ["posts"],
  queryFn: () => fetch("/api/posts").then((r) => r.json()),
});
```

### SWR

```tsx
const { data, error, isLoading } = useSWR("/api/posts", fetcher);
```

둘 다 **로딩 / 에러 / 데이터**를 훅 하나로 줄인다는 점은 같다. 차이는 **키 설계·무효화·Mutation·고급 쿼리**를 얼마나 라이브러리가 깊게 받아 주느냐에 가깝다  <br/>
확실히 SWR이 문법이 더 간결하고 이해하기는 쉬운 듯 하다.

---

## 5. TanStack Query vs SWR  

| 구분 | TanStack Query | SWR |
|------|----------------|-----|
| **첫인상** | 쿼리 클라이언트 + 키 배열 + queryFn이 중심 | URL(또는 문자열) 키 + fetcher가 중심 |
| **키** | 보통 **배열** `['posts', filter]` — 객체·파라미터 표현이 자연스럽다 | 보통 **문자열**(URL) — 단순하지만 복잡한 필터는 직렬화를 신경 써야 할 수 있다 |
| **변경 요청 (POST 등)** | `useMutation` + `invalidateQueries` 등 **패턴이 문서·커뮤니티와 정렬**되어 있다 | `fetch` 직접 + `mutate`로 **직접 연결**하는 경우가 많다 |
| **기능 범위** | 무한 스크롤, 의존 쿼리, 프리페치, Devtools 등 **넓다** | 읽기·재검증·캐시에 **집중**하고 상대적으로 얇다 |
| **번들·학습** | 개념과 옵션이 많아 **배울 양이 많지만** 대형 앱에서 이득이 크다 | **가볍고** 시작이 빠르다 |
| **적합한 경우** | CRUD 많음, 화면 많음, 캐시 전략을 팀 단위로 통일하고 싶을 때 | 대시보드·랜딩·GET 위주·빠른 도입 |

---

 

---

## 7. 참고 
- [TanStack Query VS SWR](https://velog.io/@tkddn_dev8430/TanStack-Query-vs-SWR-%EC%96%B4%EB%96%A4-%EA%B8%B0%EC%A4%80%EC%9C%BC%EB%A1%9C-%EC%84%A0%ED%83%9D%ED%95%A0%EA%B9%8C)
- [TanStack Query 공식 문서](https://tanstack.com/query/latest)
- [SWR 공식 문서](https://swr.vercel.app/)
