# 나의 React 코드 스타일 가이드

## 1. 프로젝트 구조
- FSD 아키텍처를 사용함, 
- "아키텍처는 '소유권 기반 FSD(Ownership-Driven FSD)' 모델을 따른다."
- "FSD 구조: Layers(app, pages, widgets, features, entities, shared), Slices(도메인), Segments(ui, api, model, lib, config)."
- "새로운 기능은 'pages' 슬라이스에서 시작하며, 재사용 시점에 상위 레이어로 추출(Promote)한다."

### 내가 사용했던 아키텍처 구조
```
multi-step-form
├─ src
│  ├─ App.tsx
│  ├─ app
│  │  ├─ Routes.tsx
│  │  └─ queryProvider.tsx
│  ├─ entities
│  │  └─ user
│  │     ├─ api
│  │     │  └─ index.ts
│  │     ├─ index.ts
│  │     └─ model
│  │        └─ user.type.ts
│  ├─ index.css
│  ├─ main.tsx
│  ├─ pages
│  │  ├─ enroll-profile
│  │  │  ├─ api
│  │  │  │  └─ onboarding.mutation.ts
│  │  │  ├─ index.ts
│  │  │  ├─ model
│  │  │  │  ├─ enroll-profile.context.tsx
│  │  │  │  ├─ enroll-profile.reducer.ts
│  │  │  │  ├─ enroll-profile.schema.ts
│  │  │  │  └─ useStepNavigation.ts
│  │  │  └─ ui
│  │  │     ├─ EnrollProfileView.tsx
│  │  │     ├─ Step1Nickname.tsx
│  │  │     ├─ Step2Gender.tsx
│  │  │     ├─ Step3FavoriteGenre.tsx
│  │  │     ├─ Step4FavoriteMovieDrama.tsx
│  │  │     ├─ StepGuard.tsx
│  │  │     └─ index.tsx
│  │  └─ enroll-profile-complete
│  │     ├─ api
│  │     │  └─ user.query.ts
│  │     ├─ index.ts
│  │     └─ ui
│  │        ├─ EnrollProfileCompleteSkeleton.tsx
│  │        ├─ EnrollProfileCompleteView.tsx
│  │        ├─ ErrorFallback.tsx
│  │        └─ index.tsx
│  ├─ shared
│  │  ├─ api.ts
│  │  ├─ hooks
│  │  │  └─ useErrorThrow.ts
│  │  ├─ ui
│  │  │  ├─ Button.tsx
│  │  │  ├─ FixedBottom.tsx
│  │  │  ├─ GlobalPortal.tsx
│  │  │  ├─ Input.tsx
│  │  │  ├─ NotFound.tsx
│  │  │  ├─ ProgressBar.tsx
│  │  │  ├─ Skeleton.tsx
│  │  │  ├─ Text.tsx
│  │  │  └─ index.ts
│  │  └─ utils
│  │     ├─ cn.ts
│  │     ├─ index.ts
│  │     ├─ mockApi.ts
│  │     └─ sessionStorage.ts
│  └─ vite-env.d.ts
├─ tsconfig.app.json
├─ tsconfig.json
├─ tsconfig.node.json
└─ vite.config.ts
```

## 2. 라우팅 예제 
```tsx
import { createBrowserRouter, Navigate, RouterProvider } from 'react-router-dom';

import { EnrollProfilePage } from '@/pages/enroll-profile';
import { EnrollProfileCompletePage } from '@/pages/enroll-profile-complete';
import { NotFound } from '@/shared/ui/NotFound';

const router = createBrowserRouter([
  {
    path: '/',
    element: <Navigate to="/enroll-profile?step=1" replace />,
  },
  {
    path: '/enroll-profile',
    element: <EnrollProfilePage />,
  },
  {
    path: '/enroll-profile-complete/:userId',
    element: <EnrollProfileCompletePage />,
  },
  {
    path: '*',
    element: <NotFound />,
  },
]);

export function Routes() {
  return <RouterProvider router={router} />;
}
```


## 3. 도메인 API 예제
```tsx
import { client } from '@/shared/api';

import type { User } from '../model/user.type';

export const onboardingUser = async (payload: Omit<User, 'id'>): Promise<User> => {
  const response = await client.post('/api/users/onboarding', payload);
  return response.data;
};

export const getUser = async (userId: number): Promise<User> => {
  const response = await client.get(`/api/users/${userId}`);
  return response.data;
};

// ---
import axios from 'axios';

const BASE_URL = 'https://onboarding-server-idpj.onrender.com';
export const client = axios.create({
  baseURL: BASE_URL,
});

```

## 4. react-query

```tsx
import { onboardingUser } from '@/entities/user';
import { mutationOptions } from '@suspensive/react-query';

export const createOnboardingOptions = () => {
  return mutationOptions({
    mutationFn: onboardingUser,
  });
};

// ...

import { getUser } from '@/entities/user';
import { queryOptions } from '@tanstack/react-query';

export const userQueryOptions = (userId: number) =>
  queryOptions({
    queryKey: ['user', userId] as const,
    queryFn: () => getUser(userId),
  });

// use
export function EnrollProfileCompletePage() {
  const { userId } = useParams();

  return (
    <QueryErrorResetBoundary>
      {({ reset }) => (
        <ErrorBoundary fallback={ErrorFallback} onReset={reset}>
          <Suspense fallback={<EnrollProfileCompleteSkeleton />}>
            <SuspenseQuery {...userQueryOptions(Number(userId))}>
              {({ data }) => <EnrollProfileCompleteView user={data} />}
            </SuspenseQuery>
          </Suspense>
        </ErrorBoundary>
      )}
    </QueryErrorResetBoundary>
  );
}

export function EnrollProfilePage() {
  const { currentStep } = useStepNavigation();

  return (
    <ErrorBoundary fallback={ErrorFallback}>
      <EnrollProfileProvider>
        {/* GYU-TODO: StepGuard 처리 방식이 완전 추상화 되어 있지 않은거 같음.  */}
        <StepGuard currentStep={currentStep} allowedSteps={[1, 2, 3, 4]} redirectTo="/enroll-profile?step=1">
          <EnrollProfileView />
        </StepGuard>
      </EnrollProfileProvider>
    </ErrorBoundary>
  );
}
```

## 선언적인 프로그래밍
- suspensive@3 라이브러리를 활용하여 선언적인 프로그래밍을 하는 것을 선호 

```tsx
import { queryOptions, useSuspenseQuery, useSuspenseQueries, SuspenseQuery } from '@suspensive/react-query'
import { useQuery, useQueries, useQueryClient } from '@tanstack/react-query'
 
const postQueryOptions = (postId) =>
  queryOptions({
    queryKey: ['posts', postId] as const,
    queryFn: ({
      queryKey: [, postId], // queryKey를 활용할 수 있습니다.
    }) => fetch(`https://example.com/posts/${postId}`),
  })
 
// 커스텀 쿼리 훅을 만들 필요가 없습니다.
// useQuery, useQueries, useSuspenseQuery, useSuspenseQueries, SuspenseQuery에서 직접 queryOptions를 활용할 수 있습니다.
const post1Query = useQuery(postQueryOptions(1))
const { data: post1 } = useSuspenseQuery({
  ...postQueryOptions(1),
  refetchInterval: 2000, // 사용처에서 확장성을 명확히 표현됩니다.
})
const [post1Query, post2Query] = useQueries({
  queries: [postQueryOptions(1), { ...postQueryOptions(2), refetchInterval: 2000 }],
})
const [{ data: post1 }, { data: post2 }] = useSuspenseQueries({
  queries: [postQueryOptions(1), { ...postQueryOptions(2), refetchInterval: 2000 }],
})
const Example = () => <SuspenseQuery {...postQueryOptions(1)}>{({ data: post1 }) => <>{post1.text}</>}</SuspenseQuery>
 
// queryClient의 메소드에서 queryKey와 queryFn을 쉽게 활용할 수 있습니다.
const queryClient = useQueryClient()
queryClient.refetchQueries(postQueryOptions(1))
queryClient.prefetchQuery(postQueryOptions(1))
queryClient.invalidateQueries(postQueryOptions(1))
queryClient.fetchQuery(postQueryOptions(1))
queryClient.resetQueries(postQueryOptions(1))
queryClient.cancelQueries(postQueryOptions(1))
 
 
const editPostMutationOptions = (postId: number) =>
  mutationOptions({
    mutationFn: (content: string) => fetch(`https://example.com/posts/${postId}`, {
      method: 'PATCH',
      body: JSON.stringify({ content }),
    }).then(res => res.json()),
  })
```

```tsx
import { SuspenseQuery } from '@suspensive/react-query'
import { Suspense, ErrorBoundary } from '@suspensive/react'
import { PostListItem, UserProfile } from '~/components'
 
const PostsPage = ({ userId }) => (
  <ErrorBoundary fallback={({ error }) => <>{error.message}</>}>
    <Suspense fallback={'loading...'}>
      <SuspenseQuery {...userQueryOptions(userId)}>
        {({ data: user }) => <UserProfile key={user.id} {...user} />}
      </SuspenseQuery>
      <SuspenseQuery
        {...postsQueryOptions(userId)}
        select={(posts) => posts.filter(({ isPublic }) => isPublic)}
      >
        {({ data: posts }) =>
          posts.map((post) => <PostListItem key={post.id} {...post} />)
        }
      </SuspenseQuery>
    </Suspense>
  </ErrorBoundary>
)
```
