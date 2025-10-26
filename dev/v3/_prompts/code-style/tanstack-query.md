## 1. query

- queryOptions 사용
    
    
    - **Q. queryOptions 를 사용했는데 UseQuery 타입으로도 충분히 사용할 수 있는데 queryOptions() 함수로 사용한 이유?**
        
        ```jsx
        **[면접관의 의도]**
        *   TanStack Query V5의 변경점을 잘 이해하고 있는지 확인합니다.
        *   코드의 재사용성, 테스트 용이성, 관심사 분리에 대해 얼마나 고민하는지 보고 싶어 합니다.
        ```
        
        1. **관심사의 분리와 재사용성입니다.** 
            - `addressQueryOptions` 을 보시면, 이 함수는 쿼리 키와 쿼리 함수(어떤 API를 호출할지)를 정의하는 ‘**설정**의’ 역할을 합니다. 그리고 UI 컴포넌트인 `AddressPage.tsx`에서는 이 설정을 가져와 `SuspenseQuery`에 전달하여 **'실행'**만 합니다.
            이렇게 하면 쿼리의 '정의'와 '실행'이 분리됩니다. 만약 다른 곳에서 이 주소 검색 기능을 재사용해야 한다면, 이 `addressQueryOptions` 만 가져다 **`useQuery` 나 `prefetchQuery` 등 다양한 컨텍스트에서 재사용할 수 있습니다**. 쿼리 로직을 여러 컴포넌트에서 중복으로 작성할 필요가 없어지는 것이죠
        2. **타입 안정성입니다.** 
            - `queryOptions` 를 사용하면 타입 추론이 더 원활하게 됩니다.
                
                `queryOptions`가 반환하는 객체의 타입을 기반으로 `useQuery`의 `data`와 `error` 타입이 자동으로 추론되므로, 별도의 타입 선언 없이도 타입스크립트의 이점을 온전히 누릴 수 있습니다.
                
            - 면접관님 말씀대로, `queryOptions`는 런타임에 아무것도 하지 않는 항등 함수가 맞습니다. 그럼에도 불구하고 `UseQueryOptions` 타입을 직접 쓰는 대신 `queryOptions` 함수를 써야만 하는 이유는, **TypeScript의 타입 추론(Type Inference) 능력의 한계와, `queryOptions`가 그 한계를 극복해주는 매우 영리한 장치**이기 때문입니다.
                
                특히 **`select` 옵션을 사용할 때 그 차이가 극명하게 드러납니다.**
                
            - 최종본 정리
                
                네, 맞습니다. 그 두 문장으로 완벽하게 핵심을 파악하셨습니다.
                
                이제 면접관의 "내부적으로 어떻게 처리했길래? 그렇게 되나요?" 라는 질문에 대한 모범 답안을 알려드리겠습니다. 기술적인 용어를 사용하되, 쉽고 명확하게 설명하는 것이 중요합니다.
                
                ---
                
                ### 면접관 질문: `queryOptions`가 내부적으로 어떻게 더 안정적인 타입 추론을 제공하나요?
                
                **[모범 답변]**
                
                > 네, 그 비밀은 queryOptions 함수의 런타임 코드가 아닌, 타입스크립트의 제네릭(Generics)과 타입 추론(Type Inference) 기능에 있습니다.
                > 
                > 
                > `queryOptions`는 그냥 함수가 아니라, 정교하게 설계된 **제네릭 함수**입니다. 이 함수가 호출될 때 타입스크립트 컴파일러는 다음과 같은 단계를 거칩니다.
                > 
                > **첫째, `queryFn`의 반환 타입을 먼저 추론합니다.**
                > 제가 `queryOptions`에 전달한 객체에서 `queryFn` 프로퍼티(예: `fetchUsers`)를 찾아 그 함수의 반환 타입(예: `Promise<User[]>`)을 알아냅니다. 이 타입을 `TQueryFnData`라는 제네릭 변수에 저장합니다.
                > 
                > **둘째, 그 정보를 바탕으로 `select` 함수의 타입을 추론합니다.**`select` 함수는 `(data) => ...` 형태를 가집니다. 타입스크립트는 여기서 `data` 파라미터의 타입이 방금 추론한 `TQueryFnData`(즉, `User[]`)라는 것을 이미 알고 있습니다. 그리고 제가 작성한 `select` 함수의 **실제 반환 값**(예: `data[0]`의 타입인 `User`)을 분석하여, 이것을 최종 데이터 타입인 `TData`로 확정합니다.
                > 
                > **이 과정의 핵심은, 타입스크립트가 코드의 *구현*을 보고 타입을 *역으로 추론*해낸다는 점입니다.**
                > 
                > 제가 만약 `select` 함수를 지우거나 로직을 바꾸면, 타입스크립트는 2단계 추론 과정에서 그 변경을 감지하고 `TData`의 타입을 즉시 다르게 추론합니다. `select`가 없으면 `TData`는 `TQueryFnData`와 같아지는 식이죠.
                > 
                > 따라서, `UseQueryOptions` 타입을 직접 쓰는 것이 "제가 타입스크립트에게 타입을 알려주는" 방식이라면, `queryOptions` 함수를 쓰는 것은 **"타입스크립트가 제 코드를 보고 타입을 스스로 알아내게 하는"** 방식입니다.
                > 
                > 결론적으로 `queryOptions`는 **'구현이 곧 타입이 되도록'** 만들어, 개발자가 타입 동기화에 신경 쓸 필요 없이 로직에만 집중하게 해주는 매우 영리한 타입 헬퍼 함수입니다.
                > 
                
                **[좀 더 쉬운 비유로 덧붙이고 싶다면]**
                
                > 마치 UseQueryOptions를 쓰는 건 제가 직접 "이 문의 열쇠 스펙은 5핀짜리 A타입입니다"라고 열쇠공에게 알려주는 것과 같습니다. 만약 문이 6핀짜리로 바뀌면, 제가 직접 스펙을 다시 알려줘야 합니다.
                > 
                > 
                > 하지만 `queryOptions`를 쓰는 건, 그냥 **열쇠공에게 문 자체를 보여주는 것**과 같습니다. 그러면 열쇠공이 알아서 문을 분석해서 완벽하게 맞는 열쇠를 만들어 줍니다. 문이 바뀌면 당연히 거기에 맞는 다른 열쇠를 만들어 주겠죠. `queryOptions`가 바로 그런 역할을 하는 것입니다.
                > 
            - 최종본 추가 질문
            queryOptions 가 내부적으로 타입 추론을 안정적이게 할 수 있는 이유가 이렇게 정의되어서 그런거야??
                
                ```tsx
                declare function queryOptions<TQueryFnData = unknown, TError = DefaultError, TData = TQueryFnData, TQueryKey extends QueryKey = QueryKey>(options: DefinedInitialDataOptions<TQueryFnData, TError, TData, TQueryKey>): DefinedInitialDataOptions<TQueryFnData, TError, TData, TQueryKey> & {
                    queryKey: DataTag<TQueryKey, TQueryFnData, TError>;
                };
                declare function queryOptions<TQueryFnData = unknown, TError = DefaultError, TData = TQueryFnData, TQueryKey extends QueryKey = QueryKey>(options: UnusedSkipTokenOptions<TQueryFnData, TError, TData, TQueryKey>): UnusedSkipTokenOptions<TQueryFnData, TError, TData, TQueryKey> & {
                    queryKey: DataTag<TQueryKey, TQueryFnData, TError>;
                };
                declare function queryOptions<TQueryFnData = unknown, TError = DefaultError, TData = TQueryFnData, TQueryKey extends QueryKey = QueryKey>(options: UndefinedInitialDataOptions<TQueryFnData, TError, TData, TQueryKey>): UndefinedInitialDataOptions<TQueryFnData, TError, TData, TQueryKey> & {
                    queryKey: DataTag<TQueryKey, TQueryFnData, TError>;
                };
                
                ```
                
                네, 바로 그거예요! 정확히 핵심을 보셨습니다. 보여주신 저 코드가 바로 `queryOptions`가 마법처럼 타입 추론을 해내는 이유의 전부입니다.
                
                저 코드는 타입스크립트의 **함수 오버로딩(Function Overloading)** 이라는 기능을 사용하고 있습니다. `queryOptions`라는 똑같은 이름의 함수가 여러 개의 다른 타입 시그니처(signature)를 가지고 선언되어 있죠.
                
                TypeScript는 당신이 `queryOptions`에 넘겨주는 객체(`options`)의 **모양(shape)을 보고, 이 여러 개의 선언 중에서 가장 적합한 것 하나를 골라서** 타입을 추론합니다.
                
                보여주신 세 가지 선언의 역할을 간단히 설명해 드릴게요.
                
                1. `DefinedInitialDataOptions`: 당신이 `initialData` 옵션에 **실제 값을 제공했을 때** 사용되는 선언입니다. 이 경우 `data`는 절대 `undefined`가 아니므로, 타입스크립트는 `data` 타입을 더 정밀하게 추론할 수 있습니다.
                2. `UnusedSkipTokenOptions`: `skipToken`을 사용했을 때를 위한 특별한 경우입니다.
                3. `UndefinedInitialDataOptions`: `initialData`가 없거나 `initialData`가 함수일 때, 즉 **가장 일반적인 경우**에 사용되는 선언입니다.
                
                ### 핵심은 3번째 선언에 있습니다.
                
                `UndefinedInitialDataOptions`를 위한 선언을 자세히 보죠.
                
                ```tsx
                declare function queryOptions<
                  TQueryFnData = unknown,      // API가 반환할 타입
                  TError = DefaultError,
                  TData = TQueryFnData,        // ✨ "최종 데이터 타입(TData)은 기본적으로 API 반환 타입과 같다"
                  TQueryKey extends QueryKey = QueryKey
                >(
                  options: UndefinedInitialDataOptions<TQueryFnData, TError, TData, TQueryKey>
                ): UndefinedInitialDataOptions<...>;
                
                ```
                
                여기서 가장 중요한 부분은 `TData = TQueryFnData` 입니다. 이것이 **기본값**입니다.
                
                하지만 `UndefinedInitialDataOptions` 타입 정의 안에는 `select` 프로퍼티가 이렇게 정의되어 있습니다.
                
                ```tsx
                // UndefinedInitialDataOptions 내부 (간략화)
                interface UndefinedInitialDataOptions {
                  // ...
                  select?: (data: TQueryFnData) => TData;
                }
                
                ```
                
                이제 모든 조각이 맞춰집니다.
                
                1. 당신이 `select` 옵션 **없이** 객체를 전달하면:
                    - `TData`를 추론할 단서가 없습니다.
                    - 그래서 기본값인 `TData = TQueryFnData`가 사용됩니다. (최종 데이터 타입 = API 반환 타입)
                2. 당신이 `select` 옵션을 **포함해서** 객체를 전달하면:
                    - TypeScript는 당신이 작성한 `select` 함수의 **반환 타입을 분석**합니다.
                    - 그 반환 타입을 `TData`로 **추론하고 확정**합니다.
                    - 이제 `TData`는 더 이상 기본값이 아니라, `select` 함수의 반환 타입이 됩니다. (최종 데이터 타입 = `select`의 반환 타입)
                
                **결론:**
                
                `queryOptions`가 타입 추론을 안정적으로 할 수 있는 이유는, **함수 오버로딩**을 통해 `initialData`의 유무 등 여러 케이스에 따라 다른 타입 규칙을 적용하고, **제네릭의 기본값과 타입 추론**을 영리하게 활용하여 개발자가 전달한 객체의 `select` 함수 구현으로부터 최종 데이터 타입을 역으로 알아내기 때문입니다.
                
                보여주신 저 코드야말로, "구현이 곧 타입이 되도록" 만드는 TypeScript 패턴의 정수라고 할 수 있습니다.
                
                ---
                
                아, 정말 좋은 지적입니다! 제가 설명을 위해 **과도하게 단순화**해서 혼란을 드렸네요. 죄송합니다.
                
                실제로는 `UndefinedInitialDataOptions`라는 직접적인 인터페이스가 따로 정의되어 있지 않습니다. 제가 보여드린 코드는 "이해를 돕기 위한 가상의 예시"였는데, 마치 실제 코드인 것처럼 표현해서 혼란을 드렸습니다.
                
                ---
                
                ## 실제 TanStack Query의 타입 구조
                
                ## 실제 구조 분석
                
                실제로는 다음과 같은 구조입니다:
                
                ```tsx
                // 실제 코드에서
                type UndefinedInitialDataOptions<TQueryFnData = unknown, TError = DefaultError, TData = TQueryFnData, TQueryKey extends QueryKey = QueryKey> =
                  UseQueryOptions<TQueryFnData, TError, TData, TQueryKey> & {
                    initialData?: undefined | InitialDataFunction<NonUndefinedGuard<TQueryFnData>> | NonUndefinedGuard<TQueryFnData>;
                  };
                
                ```
                
                핵심은 `UndefinedInitialDataOptions`가 **`UseQueryOptions<TQueryFnData, TError, TData, TQueryKey>`를 확장(extends)**한다는 점입니다.
                
                즉, `select` 옵션은 `UseQueryOptions` 타입 안에 정의되어 있고, `UndefinedInitialDataOptions`는 그것을 상속받아 `initialData` 관련 제약만 추가한 것입니다.
                
                ## 타입 추론의 진짜 비밀
                
                결국 타입 추론이 가능한 이유는:
                
                1. **함수 오버로딩**: 3가지 다른 시그니처로 다양한 케이스 처리
                2. **제네릭 기본값**: `TData = TQueryFnData` (select 없으면 API 반환 타입 그대로)
                3. **`UseQueryOptions` 내부의 `select` 정의**: 이 부분이 핵심인데, `select?: (data: TQueryFnData) => TData` 형태로 정의되어 있을 것입니다.
                
                제가 직접 `UseQueryOptions` 타입을 찾아서 `select` 부분을 보여드리려 했는데, 파일 구조가 복잡해서 바로 찾기 어렵네요.
                
                ## 제가 실수한 부분
                
                제가 "UndefinedInitialDataOptions 내부"라고 표현한 것이 잘못이었습니다. 정확히는:
                
                - `UndefinedInitialDataOptions`는 `UseQueryOptions`를 확장
                - `select` 옵션은 `UseQueryOptions` 안에 정의됨
                - 따라서 `UndefinedInitialDataOptions`도 `select` 옵션을 가짐
                
                **결론적으로 타입 추론 원리는 제가 설명한 것이 맞지만, 코드 구조에 대한 표현이 부정확했습니다.** 혼란을 드려 죄송합니다!
                
                실제로는 `UseQueryOptions` 타입 정의를 찾아야 `select` 옵션의 정확한 타입을 볼 수 있을 것입니다.
                
            
    1. 관심사의 분리와 재사용성 
    2. 타입 안정성 

- queryOptions 추상화를 통해 options 타입 지정
    
    ```
    // queryOptions 타입 지정 
    export const meQueryOptions = (options?: **Omit<Parameters<typeof queryOptions>[0], 'queryKey' | 'queryFn'>**) =>
      queryOptions({
        queryKey: ['me'],
        queryFn: fetchMe,
        ...options,
      });
    // 이렇게 하니깐 사용처에서 API 응답 타입 추론이 안되어서 
    
    ```
    
    → 해당 방식은 사용처에서 API 응답 타입 추론이 안됨!! 
    
    ### 결론
    
    - `UndefinedInitialDataOptions` 타입 활용
    
    ```jsx
    import { **type UndefinedInitialDataOptions**, queryOptions } from '@tanstack/react-query';
    
    // 이렇게 개선 
    export const meQueryOptions = (options?: Omit<**UndefinedInitialDataOptions**<ApiResponse<Me>>, 'queryKey' | 'queryFn'>) =>
      queryOptions({
        queryKey: ['me'],
        queryFn: fetchMe,
        ...options,
      });
    
    ```
    
    근데 더 좋은 방식은 해당 패턴으로!! 
    
    ```tsx
    // 사용 
    useQuery(**{
    	...meQUeryOptions,
    	[속성들...]
    }**)
    ```
    

```tsx
// 가장 최근 사용 패턴
useQuery(**{
	...meQUeryOptions,
	[속성들...]
}**)
```

```tsx
// 정의
export const addressSearchQueryOptions = (keyword: string) =>
  queryOptions({
    queryKey: addressQueryKey.search(keyword),
    queryFn: () => fetchAddressResults(keyword),
  });

// 사용 
// 1. suspensive SuspenseQuery
<SuspenseQuery {...addressSearchQueryOptions(debouncedKeyword)}>
  {({ data }) => { ...}
</SuspenseQuery>

  
// 2. react-query 
const { data } = useSuspenseQuery({
  ...merchantListQueryOptions(),
});
```

- SuspenseQuery 로 선언적으로 하는 것도.. data-fetching 도 SuspenseQuery 로 위임해서 컴포넌트 내부에서 data-fetching 을 신경쓰지 않고, 컴포넌트는 UI 만 담당
    
    
    - **선언적(What/관계)**: 비즈니스·서비스 로직은 **관계 기반의 What**을 선언하고, **시간 흐름의 How**는 하위(인프라/훅/유틸)에 **위임**
        
        효과: 사람이 인지해야 할 요소가 줄어 가독성과 변경 용이성이 올라감
        
        e.g. **로딩은 Suspense, 에러는 ErrorBoundary, 데이터 동기화는 Query가 담당 → 컴포넌트는 “상태→뷰 매핑”에만 집중**
        

## 2. mutation

- mutationOptions 사용
    
    ```tsx
    // 정의 
    export const createMerchantMutationOptions = () => {
      const queryClient = useQueryClient();
    
      return mutationOptions({
        mutationFn: postMerchant,
        onSuccess: () => {
          queryClient.invalidateQueries({ queryKey: merchantQueryKey.merchantList });
        },
      });
    };
    
    // 사용 
    const { mutate } = useMutation({
      ...createMerchantMutationOptions(),
    });
    ```
    

---

- [부록] useMutation API 스키마
    
    ### 3. useMutation
    
    ```tsx
    UseMutationOptions<TData, TError, TVariables, TContext>
    ```
    
    - **TData**: mutation이 성공했을 때 반환하는 데이터의 타입 (즉, mutationFn의 반환값)
    - **TError**: mutation이 실패했을 때의 에러 타입
    - **TVariables**: mutationFn에 전달되는 변수(파라미터)의 타입
    - **TContext**: (선택) onMutate에서 반환하는 context의 타입 (주로 optimistic update에서 사용)
    

## 3. 에러 핸들링

[5. react-qeury API 요청시 정의, queryProvider 정의 ](https://www.notion.so/5-react-qeury-API-queryProvider-289fc844aaad803db955ce56c42168c5?pvs=21) 

```tsx
// /src/lib/queryProvider.tsx
import * as React from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { isAppError } from 'lib/error/AppError';

function createQueryClient() {
  return new QueryClient({
    defaultOptions: {
      **queries: {
        // AppError 기준 재시도 규칙: Auth/NotFound는 재시도 제외, 나머지는 최대 2회
        retry: (failureCount, error) =>
          !(isAppError(error) && (error.kind === 'Auth' || error.kind === 'NotFound')) &&
          failureCount < 2,
      },
      mutations: {
        retry: 0,
      },**
    },
  });
}

const queryClient = createQueryClient();

export function QueryProvider({ children }: { children: React.ReactNode }) {
  return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>;
}
```

---