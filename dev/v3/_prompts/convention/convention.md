### 꼭 해야할것

- **2. Event handler naming**
    
    **한 줄 설명:** 이벤트 핸들러는 `handle{TargetName}{EventType}` 형태로 작성합니다.
    
    ```tsx
    <Button onClick={handleButtonClick} onFocus={handleButtonFocus} />
    ```
    
- **4. Component boolean props**
    
    **한 줄 설명:** boolean props는 is/can/should 접두어 없이, 명시적으로 true/false 값을 전달합니다.
    
    컴포넌트의 boolean props는 `is`, `can`, `should` 등의 접두어 없이 작성하며, true 값을 생략하지 않고 명시적으로 전달합니다.
    
    ```tsx
    <TheBox open={true} clickable={false} animate={true} />
    <Something open={true} />
    
    // ❌
    // <TheBox isOpen={true} canClick={false} shouldAnimate={true} />
    // <Something open />
    ```
    
- **8. Boolean check**
    
    **한 줄 설명:** boolean이 아닌 값을 boolean으로 type casting하지 않고, null/undefined 체크는 == 연산자를 사용합니다.
    
    ```tsx
    // ✅
    if (something != null) { ... }
    
    // ❌
    // if (!!something) { ... }
    // if (something === null || something === undefined) { ... }
    ```
    
- **9. Non-null Assertion**
    
    **한 줄 설명:** null이 될 수 없는 상황에서는 optional chaining 대신 non-null assertion을 사용합니다.
    
    ```tsx
    // ✅
    const { data } = useData();
    const name = data**!**.contents.name;
    
    // ❌
    const { data } = useData();
    const name = data**?**.contents.name;
    ```
    
- **10. API argument 네이밍 ⭐️⭐️**
    
    **한 줄 설명:** API 호출 시 전달하는 데이터는 params로 통일합니다.
    
    API 호출 시 search parameter, path parameter, body payload, header 등은 모두 params로 통일해 전달합니다.
    
    ```tsx
    // ✅
    function getUser(params: GetUserParams) { ... }
    
    // ❌
    // function getUser(payload: GetUserParams) { ... }
    ```
    
- **12. API 호출 함수 네이밍 ⭐️⭐️**
    
    **한 줄 설명:** HTTP 메서드에 따라 함수 네이밍을 fetch/post/update/delete로 통일합니다.
    
    - get method ⇒ `fetch` prefix
    - post method ⇒ `post` prefix
    - put method ⇒ `update` prefix
    - delete method ⇒ `delete` prefix
    
    ```tsx
    // ✅
    function **fetch**LuckyQuizList() { ... }
    function **post**LuckyQuizDetail() { ... }
    function **update**LuckyQuizDetail() { ... }
    function **delete**LuckyQuizDetail() { ... }
    
    // ❌
    // list.map(({ name }) => name);
    ```
    
- **17. 가능하다면 최대한 응집도 높게 구현 ⭐️⭐️⭐️**
    
    **한 줄 설명:** 관련된 로직과 UI를 한 컴포넌트/함수 내에 응집시켜 구현합니다.
    
    관련된 로직과 UI를 한 곳에 모아, 코드의 응집도를 높이고 유지보수성을 향상시킵니다.
    
    ```tsx
    // ✅
    if(condition) {
        return <Navigate to="/" />
    }
    
    // 다른 코드들
    <WysiwygContentForm
        header={
    	    <WysiwygContentForm.Header
    		    right={<WysiwygContentForm.Header.Right ... />}
    			  ...
    			/>
    	}
    />
    ```
    
    ```tsx
    // ❌ 
    const navigate = useNavigate();
    
    // 다른 코드들
    useEffect(() => {
    	if(condition) {
    		navigate('/');
    	}
    }, [navigate, condition])
    
    // 다른 코드들
    <WysiwygContentForm
        header={
    	    <WysiwygContentForm.Header
    		    right={<WysiwygContentForm.Header.Right ... />}
    			  ...
    			/>
    		}
    />
    ```
    
- **18.  삼항 연산자(ternary) 사용**
    
    **한 줄 설명:** 조건부 렌더링 시 boolean이 아닌 값은 && 대신 삼항 연산자를 사용합니다.
    
    조건부 렌더링에서 boolean이 아닌 값은 && 연산자 대신 삼항 연산자를 사용합니다.
    
    ```tsx
    // ✅
    return <section>{title != null ? <h1>{title}</h1> : null}</section>;
    
    // ❌
    // return <section>{title && <h1>{title}</h1>}</section>;
    ```
    
- **25. 복잡한 분기처리의 경우에도 하나의 상태에만 의존 ⭐️**
    
    **한 줄 설명:** 여러 상태값 대신 하나의 상태값만을 기준으로 분기처리합니다.
    
    여러 상태값을 조합하지 않고, 하나의 상태값만을 기준으로 분기처리합니다.
    
    ```tsx
    // ✅
    const [contentStatus, setContentStatus] = useState(ContentStatus.기본);
    ...
    <SwitchCase
       value={contentStatus}
       caseBy={{
          ContentStatus.수정중: ...,
          ContentStatus.보류중: ...,
          ContentStatus.기본: ...
        }}
     />
     
     const severity: 'success' | 'error' | 'warning' | 'info' = 'info'
     
      const icon = match(severity)
        .with('success', () => 'icon-check-circle-green')
        .with('warning', () => 'icon-warning-circle')
        .with('error', () => 'icon-warning-circle-red')
        .with('info', () => 'icon-info-circle') // ✅
        .exhaustive();
    ```
    
    ```tsx
    ❌ 
    const [소재보류, 소재보류하기, 소재보류_취소] = use광고소재보류State();
    const [소재수정, set소재수정, set소재수정취소] = useBooleanState(false);
    ...
    {!소재보류 && !소재수정 && (...)}
    {소재보류 && (...)}
    {소재수정 && (...)}
    ```
    
- **27.  디렉토리 구조 컨벤션 ⭐️⭐️⭐️**
    
    **한 줄 설명:** 지역성 원칙을 고려해, 실제 사용하는 곳과 가까운 위치에 파일을 둡니다.
    
    공통 컴포넌트는 전역 디렉토리에, 도메인 컴포넌트는 도메인별 디렉토리에 위치시키고, 사용하는 곳과 가까운 위치에 파일을 둡니다.
    
    ## **✅ 지역성 원칙을 고려한 패키지 구조**
    
    - `/pages` next app에서 route를 정의하기 위한 디렉토리
    - `/src/pages` 페이지 별로 **flat하게 관리**, 이 안에서 hooks, components 등 디렉토리를 나눔
    - `/src/pages/**/components` 컴포넌트와 컨테이너의 구분없이 컴포넌트라 지칭하고 hooks와 함께 둔다.
    - `/src/components` 도메인 로직과는 전혀 상관없는 재사용 가능한 공통 컴포넌트들
    - `/src/utils` 도메인 로직과는 전혀 상관없는 유틸성 함수들이 위치
    1. components만 모아두는 directory에는 반드시 애플리케이션 전역에서 공유하는 컴포넌트만 위치해야 한다. 그렇지 않은 경우는 최대한 공유하거나 사용하는 곳에서 가장 가까운 디렉토리에 위치시킨다.
    2. **도메인을 알고 있는 컴포넌트는 `modules` 디렉토리에서 도메인 별로 관리한다.**
        
        **모듈은 **특정 기능이나 로직을 하나의 단위로 묶은 소프트웨어 부품 또는 독립적인 하드웨어 구성 요소**를 의미합니다.
        
    
    ## why?
    
    - 실제 사용하는 곳과 정의된 곳이 멀리 떨어져 있는 것이 불편했고 이를 해소하기 위함.
    - 불편했던 점은 해당 컴포넌트의 변경에 어디까지 영향을 주는지 확인하기 어렵다.
- **28.  Zod 사용, 스키마 작성법 ⭐️⭐️⭐️**
    
    **한 줄 설명:** 기초가 되는 스키마는 서버에 1개의 엔티티에 대한 read(GET) 요청을 했을때를 기준으로 한다.
    
    ```tsx
    // ✅
    // src/domain/Payment/Payment.schema.ts
    // 작성되는 위치는 (컴포넌트와 마찬가지로) 상황에 따라 다를 수 있다.
    // page-specific한 모델 스키마의 경우 해당 page 디렉토리에 작성할 수 있다.
    
    import { z } from 'zod';
    
    export const PaymentSchema = z.object({
      key: z.string(),
      amount: z.number(),
      detail: z.object({
        // ...
      }),
      // ...
    });
    
    export type Payment = z.infer<typeof PaymentSchema>;
    
    ```
    
    이걸 기반으로 remote API를 정의한다.
    
    ```tsx
    // src/domain/Payment/Payment.api.ts
    import { httpClient } from '@app/remote/httpClient';
    import { PaymentSchema, Payment } from './Payment.schema';
    
    export async function getPayment(key: Payment['key']) {
      const response = await httpClient.get(`/v1/payments/${key}`);
      // parse를 하면 타입이 추론되므로 Generic을 사용할 필요가 없음.
      return PaymentSchema.parse(response);
    }
    
    ```
    
    Create, List API 등 다른 API 들도 기초 스키마를 기반으로 확장한다.
    
    ```tsx
    // src/domain/Payment/Payment.api.ts
    
    // ...
    
    export async function createPayment(params: Pick<Payment, 'amount' | '...'>) {
      const response = await httpClient.post(`/v1/payments`, {
        body: params,
      });
    
      return PaymentSchema.parse(response);
    }
    
    const ListPaymentsSchema = z.array(
      PaymentSchema.omit({
        detail: true,
      })
    );
    
    export async function listPayments() {
      const response = await httpClient.get(`/v1/payments`);
    
      return ListPaymentsSchema.parse(response);
    }
    
    ```
    
- **29. 라이브러리 Class 타입 체크 ⭐️**
    
    **한 줄 설명:** 라이브러리에서 가져온 클래스의 타입 체크는 instanceof 대신 타입 체크 함수를 사용합니다.
    
    > 라이브러리 버전이 여럿일 경우 번들링 과정에서 클래스의 싱글톤이 보장안될 수 있거나 혹은 babel 등 사용시 transpile 과정에서 class가 아니게 될 수 있어서 `instanceof` 대신 사용합니다.
    > 
    
    ## ✅ 좋은 케이스
    
    ```tsx
    export function isTossPaymentsError(err: unknown): err is TossPaymentsError {
      if (typeof err !== 'object' || err == null) {
        return false;
      }
      return ['message', 'success'].every(prop => prop in err);
    }
    if (isTossPaymentsError(error)) {
      console.log(error.message);
    }
    ```
    
    ## ❌ 안좋은 케이스
    
    ```tsx
    import { TossPaymentsError } from '@tosspayments/error';
    if (error instanceof TossPaymentsError) {
      console.log(error.message);
    }
    ```
    

### 기본 (하고 있던것들)

- **1. effective react component**
    
    **한 줄 설명:** 컴포넌트에서 리턴하는 element가 없을 때, `return null`을 한다.
    
- **3. Props deconstruct**
    
    **한 줄 설명:** props를 bypass 하는 경우가 아니라면 구조분해할당 합니다.
    
    ```tsx
    function Wrapper({ children }: { children: ReactNode }) {
        //
    }
    ```
    
- **7. typescript enum 사용 금지**
    
    **한 줄 설명:** typescript의 enum 대신 as const 객체를 사용합니다.
    
    ```tsx
    // ✅
    const Something = {
      Foo: '...',
      Bar: '...',
    } as const;
    ```
    
- **11. Nilable vs Optional Parameter**
    
    **한 줄 설명:** undefined 허용과 optional parameter를 구분하여 상황에 맞는 타입을 사용합니다.
    
    함수 파라미터에서 undefined 허용과 optional parameter를 구분하여 타입을 명확히 작성합니다.
    
    ```tsx
    // ✅
    function apply(name?: string) { ... }
    function apply(name: string | undefined) { ... }
    
    // ❌
    // 무지성 nullable은 안좋아요.
    // function apply(name?: string) { ... }
    ```
    
- **13. react-query 호출 함수 네이밍**
    
    **한 줄 설명:** get 인 경우엔 hook 이름에 get을 안붙인다.
    
    react query의 useSuspenseQuery, useQuery를 활용할 때, `get`키워드는 사용하지 않는다.
    
    ```tsx
    // ✅
    functioin useOrders() {}
    
    // ❌
    functioin useGetOrders() {}
    ```
    
- **15. 함수/컴포넌트는 하나의 역할만 담당 |** 기본 하고 있지만 컨벤션 예제가.. 좀 빡쎔 ㅋㅋ
    
    **한 줄 설명:** 함수/컴포넌트는 최대한 하나의 역할만 담당하도록 작성합니다.
    
    복잡한 함수는 역할별로 분리하여 가독성과 유지보수성을 높입니다.
    
    ```tsx
    // ✅
    const handleTargetSelectionComplete = async () => {
      try {
        await validateFields(['segmentOptions', 'targetSegmentCnt']);
        if (getValues('viewType') === 'CREATE') {
          const { id, name } = await createDisplayAdsSegment();
          openSuccessToaster(`${name} 타겟이 생성되었어요.`);
          setValue('segmentId', id);
        } else {
          await editDisplayAdsSegment({ id: getValues('segmentId')! });
        }
        clearErrors('segmentId');
        setValue('viewType', 'SUMMARY');
      } catch (error: any) {
        openErrorToaster(error.message);
      }
    };
    
    ```
    
    ```tsx
    const handleTargetSelectionComplete = async () => {
        const [isValidSegmentOptions, isValidTargetCount] = await Promise.all([
          validate('segmentOptions', { shouldFocus: true }),
          validate('targetSegmentCnt'),
        ]);
        if (!isValidSegmentOptions) {
          openErrorToaster('입력되지 않은 항목이 있어요.');
          return;
        }
        if (!isValidTargetCount) {
          openErrorToaster(`최소 타겟 규모는 ${commaizeNumber(DisplayAdsPolicy.Contract.최소_타겟_규모)}명이에요.`);
          return;
        }
        const payload = convertSegmentOptionsToCreateSegmentPayload({
          segmentOptions: getValues('segmentOptions'),
          count: getValues('targetSegmentCnt') ?? undefined,
          treeNodeModels,
          adType: '배너광고',
        });
        const payloadDescription = getCreateSegmentPayloadDescription({
          payload,
          treeNodeModels,
          nameMapper: { ...nameMapper, ...idNameMapper },
        });
        try {
          if (getValues('viewType') === 'CREATE') {
            const { id: segmentId } = await createSegment(
              { ...payload, sampling: getValues('sampling'), description: payloadDescription },
              { adType: '배너광고' },
            );
            openSuccessToaster(`${payload.name} 타겟이 생성되었어요.`);
            setValue('segmentId', segmentId);
          } else {
            await editSegment(getValues('segmentId')!, payload);
          }
          clearErrors('segmentId');
          setValue('viewType', 'SUMMARY');
        } catch (error: any) {
          openErrorToaster(error.message);
        }
      };
    
    ```
    
- **16. reduce에서 타입단언 금지**
    
    **한 줄 설명:** Array.prototype.reduce 를 사용 할 때 initialValue를 필수로 넣고, 타입단언하지 않습니다.
    
    `as casting` 을 통한 타입단언을 금지합니다.
    
    ```tsx
    // ✅
    arr.reduce<Item>((acc, curr) => {
      acc[curr.id] = curr;
      return acc;
    }, {});
    
    // ❌
    arr.reduce((**acc = {}**, curr) => {
      acc[curr.id] = curr;
      return acc;
    });
    
    arr.reduce((acc, curr) => {
      acc[curr.id] = curr;
      return acc;
    }, **{} as Item**);
    ```
    
- **21.  export default vs export**
    
    **한 줄 설명:** export default는 사용하지 않고, 필요시 명시적 export만 사용합니다.
    
    export default는 next.js의 pages 디렉토리에서만 사용하고, 나머지는 명시적 export를 사용합니다.
    
    ```tsx
    // ✅
    export { ListItem };
    export { MyPage as default } from ‘pages/MyPage’;
    
    // ❌
    // export default ListItem;
    ```
    
- **23. 하드코딩 값은 상수로 사용**
    
    **한 줄 설명:** 하드코딩해야 하는 값은 의미 있는 상수로 선언하여 사용합니다.
    
    하드코딩 값(매직 넘버 등)은 반드시 상수로 선언하여 코드의 의도를 명확히 드러냅니다.
    
    ```tsx
    // ✅
    const 최대_퀴즈_개수 = 3;
    ...
    if(fields.length > 최대_퀴즈_개수) {...}
    ```
    
- **24.  파일에 함수 여러개 있는 경우 배치 순서 ⭐️⭐️**
    
    **한 줄 설명:** 고수준에서 저수준 순으로 함수/컴포넌트를 배치합니다.
    
    한 파일에 여러 함수/컴포넌트가 있을 때, 고수준(상위) → 저수준(하위) 순서로 배치합니다.
    
    ```tsx
    // ✅
    interface Props {...}
    
    **function ChangePasswordPage() {**
      ...
      return (
        <>
         {...}
         <SubmitButton />
        </>
      )
    }
    
    **function SubmitButton () { ... }
    const styledButton = {...}**
    ```
    
    ```tsx
    // ❌
    interface Props {...}
    
    function SubmitButton () { ... }
    
    function ChangePasswordPage() {
      ...
      return (
        <>
         {...}
         <SubmitButton />
        </>
      )
    }
    ```