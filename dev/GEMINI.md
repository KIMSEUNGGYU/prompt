# GEMINI.md: 프론트엔드 개발 헌법

이 문서는 Gemini가 당신의 프론트엔드 개발을 돕기 위한 핵심 가이드입니다. 모든 코드 생성, 리팩토링, 리뷰 요청 시 이 헌법을 최우선으로 따릅니다.

## 제 1장: 최상위 개발 철학 (Core Philosophy)

- **최우선 가치: 변경 용이성 (Easy to Change).** 코드는 비즈니스 변화를 쉽게 수용할 수 있어야 합니다.
- **높은 응집도, 낮은 결합도 (High Cohesion, Low Coupling):** 함께 변경될 코드는 함께 위치시키고, 모듈 간의 의존성은 최소화하여 코드의 영향 범위를 제어합니다.
- **100% 선언적 비동기 처리:** `if(isLoading/isError)`와 같은 명령형 코드를 금지합니다. **`@suspensive/react`**와 **`@suspensive/react-query`**를 사용하여 `<Suspense>`와 `<ErrorBoundary>`에 로딩/에러 상태 처리를 위임합니다.
- **SOLID 원칙:** 특히 단일 책임 원칙(SRP)과 관심사 분리(SoC)를 모든 설계의 기반으로 삼습니다.
- **추상화:** 복잡한 조건문이나 매직넘버는 가독성을 위해 이름 있는 함수/상수로 추상화합니다.

## 제 2장: 기술 스택 (Tech Stack)

- **필수 (Required):** `React`, `TypeScript`, `TanStack Query`, `React-Router-Dom`, `Axios`, `@suspensive/react`, `@suspensive/react-query@v3`.
- **표준 옵션 (Optional but Standard):** `Next.js`, `Zustand`, `React-Hook-Form`, `Zod`, `Jest` + `RTL`, `Playwright`, `Tailwind CSS`, `shadcn/ui` + `Radix UI`.

## 제 3장: 아키텍처 (Ownership-Driven FSD)

- **모델:** 소유권 기반 FSD(Ownership-Driven FSD) 모델을 따릅니다.
- **구조:**
  - **Layers:** `app`, `pages`, `widgets`, `features`, `entities`, `shared`
    - `app`: 애플리케이션의 전역 설정 (라우팅, global style, 다국어 설정 등)
    - `pages`: 페이지 단위 컴포넌트
    - `widgets`: 재사용 가능한 복잡한 UI 블록
    - `features`: **(비즈니스 로직 전용)** `entities`를 조합하거나 클라이언트의 요구사항을 반영하여 실제 기능을 구현합니다.
    - `entities`: **(도메인 전용)** 서버 API의 스키마를 1:1로 반영하는 순수한 도메인 모델과 관련 API 함수를 정의합니다.
    - `shared`: 공통 유틸리티 및 컴포넌트
  - **Slices:** 도메인 단위 (e.g., `user`, `enroll-profile`)
  - **Segments:** `ui`, `api`, `model`, `lib`, `config`
  - `ui`: 컴포넌트를 정의합니다.
  - `api`: 서버 API 연동 및 `TanStack Query`를 사용한 서버 상태 관리를 담당합니다.
  - `model`: `*.type.ts`, `*.store.ts`, `*.schema.ts` 등을 통해 **타입, 클라이언트 상태(store), 유효성 검사(schema) 등 해당 도메인의 데이터 모델과 관련된 로직**을 정의합니다.
  - `lib`: 비즈니스 로직이 없는 순수 유틸리티 함수를 정의합니다.
  - `config`: 상수, 환경 변수 등 설정 값을 정의합니다.
- **핵심 워크플로우:**
  1.  **페이지에서 시작:** 새로운 기능은 해당 기능이 처음 사용되는 `pages` 슬라이스 내부에 모든 관련 코드(`ui`, `api`, `model`)와 함께 구현하는 것을 원칙으로 합니다.
  2.  **재사용 시점에 추출:** 다른 곳에서 재사용이 필요할 때, 해당 코드를 `features`, `widgets`, `entities` 등 적절한 상위 레이어로 추출(Promote)합니다.
- **구조 예시:**
  ```
  src
  ├─ app
  ├─ pages
  │  └─ enroll-profile
  │     ├─ api
  │     ├─ model
  │     └─ ui
  ├─ widgets
  ├─ features
  ├─ entities
  │  └─ user
  │     ├─ api
  │     └─ model
  └─ shared
     ├─ ui
     ├─ hooks
     └─ utils
  ```

## 제 4장: 핵심 구현 패턴 (Core Implementation Patterns)

- **데이터 페칭 (TanStack Query + Suspensive):**

  - **관심사 분리:**
    1.  순수 API 요청 함수 (`axios`): `entities/{domain}/api/index.ts`에 위치시킵니다.
    2.  `queryOptions` / `mutationOptions` 객체: `pages/{page}/api/*.query.ts` 또는 `features/{feature}/api/*.mutation.ts`에 위치시켜 API 호출과 상태 로직을 분리합니다.
  - **선언적 사용:** `useSuspenseQuery` 또는 `<SuspenseQuery>` 컴포넌트를 사용합니다.
  - **예시 (`queryOptions`):**

    ```typescript
    // entities/user/api/index.ts
    export const getUser = async (userId: number): Promise<User> => {
      const response = await client.get(`/api/users/${userId}`);
      return response.data;
    };

    // pages/enroll-profile-complete/api/user.query.ts
    import { getUser } from '@/entities/user';
    import { queryOptions } from '@tanstack/react-query';

    export const userQueryOptions = (userId: number) =>
      queryOptions({
        queryKey: ['user', userId] as const,
        queryFn: () => getUser(userId),
      });
    ```

  - **예시 (컴포넌트):**

    ```tsx
    import { SuspenseQuery } from '@suspensive/react-query';
    import { ErrorBoundary, Suspense } from '@suspensive/react';

    <ErrorBoundary fallback={ErrorFallback}>
      <Suspense fallback={<Skeleton />}>
        <SuspenseQuery {...userQueryOptions(Number(userId))}>
          {({ data }) => <EnrollProfileCompleteView user={data} />}
        </SuspenseQuery>
      </Suspense>
    </ErrorBoundary>;
    ```

- **상태 관리 (State Management):**
  - **전역 (Global):** `Zustand` (사용자 인증, 테마 등 앱 전역 상태)
  - **도메인 (Domain):** `Context API + useReducer` (다단계 폼 등 특정 도메인 내 복잡한 상태)
  - **지역 (Local):** `useState` (단일 컴포넌트 내부의 간단한 상태)
- **파일 명명 규칙 (File Naming):**
  - 파일의 역할을 명확히 하기 위해 `*.query.ts`, `*.mutation.ts`, `*.schema.ts`, `*.type.ts` 와 같은 접미사를 사용합니다.
- **폼 (Forms):**
  - `React-Hook-Form` + `Zod` 조합을 표준으로 사용합니다.

## 제 5장: 코드 리뷰 가이드라인 (Code Review Guidelines)

코드 리뷰 요청 시, 아래 항목들을 기준으로 검토를 진행합니다.

- **[ ] 아키텍처:** 코드가 Ownership-Driven FSD 원칙(페이지에서 시작, 필요시 추출)을 따르는가?
- **[ ] 선언적 프로그래밍:** `isLoading`, `isError` 분기문 없이 Suspensive를 통해 비동기 로직이 100% 선언적으로 작성되었는가?
- **[ ] 관심사 분리:**
  - API 요청 함수와 `queryOptions`가 명확히 분리되었는가?
  - `entities`는 순수 도메인 모델을, `features`는 비즈니스 로직을 처리하는 역할 분리가 잘 되었는가?
  - UI 로직과 비즈니스 로직(커스텀 훅 등)이 분리되었는가?
- **[ ] 상태 관리:** 상태의 범위(전역/도메인/지역)에 맞는 적절한 도구를 사용했는가?
- **[ ] 컨벤션:** 파일 명명 규칙, FSD 구조, 기술 스택 등 헌법에 명시된 규칙을 준수하는가?
- **[ ] 변경 용이성:** 코드가 헌법의 최상위 가치인 '변경 용이성'에 기여하는가?
