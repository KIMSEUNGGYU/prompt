## 코드 컨벤션

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
    
- **13. react-query 호출 함수 네이밍**
    
    **한 줄 설명:** get 인 경우엔 hook 이름에 get을 안붙인다.
    
    react query의 useSuspenseQuery, useQuery를 활용할 때, `get`키워드는 사용하지 않는다.
    
    ```tsx
    // ✅
    functioin useOrders() {}
    
    // ❌
    functioin useGetOrders() {}
    ```
    

## 정의 템플릿

### 기본 - 필수

- 1. 스키마 정의 - zod 기반
    
    ```tsx
    // template.schema.ts
    
    import { z } from 'zod';
    
    export const TemplateSchema = z.object({
      id: z.string(),
      title: z.string(),
      amount: z.number().nonnegative(),
      detail: z.object({}).passthrough().optional(),
    });
    export type Template = z.infer<typeof TemplateSchema>;
    
    export const TemplateListSchema = z.array(
      TemplateSchema.pick({ id: true, title: true, amount: true })
    );
    ```
    
- 2. API 정의 - ky 및 커스텀 httpClient (ApiResponse 및 데이터만 반환)
    
    ```tsx
    // template.~~api.~~ts 
    import { httpClient } from '@/remotes/httpClient';
    import { TemplateSchema, TemplateListSchema, Template, TemplateModel } from './Template.schema';
    
    // GET (단건)
    export async function fetchTemplate(params: { id: Template['id'] }) {
      const res = await httpClient.get(`/v1/templates/${params.id}`);
      const parsed = TemplateSchema.**parse**(res.data);
      return new TemplateModel(parsed); // 모델 클래스로 감싸는 방식 (원하면 생 데이터 반환으로 바꿔도 됨)
    }
    
    // GET (목록)
    export async function fetchTemplates() {
      const res = await httpClient.get(`/v1/templates`);
      const parsed = TemplateListSchema.parse(res.data);
      return parsed.map((t) => new TemplateModel({ ...t, detail: {} } as Template));
    }
    
    // POST
    export async function postTemplate(params: Pick<Template, 'title' | 'amount'>) {
      const res = await httpClient.post(`/v1/templates`, params);
      return TemplateSchema.parse(res.data);
    }
    
    // PUT/PATCH
    export async function updateTemplate(params: Partial<Pick<Template, 'title' | 'amount'>> & { id: Template['id'] }) {
      const res = await httpClient.put(`/v1/templates/${params.id}`, params);
      return TemplateSchema.parse(res.data);
    }
    
    // DELETE
    export async function deleteTemplate(params: { id: Template['id'] }) {
      await httpClient.delete(`/v1/templates/${params.id}`);
    }
    
    ```
    
- 3. queryKey 는 도메인 단위로 정의
    
    ```tsx
    // shared/queryKey.ts
    export const templateQueryKey = {
      all: () => ['templates'] as const,
      detail: (id: string) => ['templates', id] as const,
    };
    ```
    

### 심화 - 상세 (기본적인 것은 아니고 필요하면 사용)

> 해당 내용을 API - 상세 페이지로 이동할지 고민중!
> 
- 4. `HttpClient.tsx` - ApiResponse 및 실제 데이터만 parse 하기 위함
    - ApiResponse 가 형태가 있고, 내부적으로 유효한 데이터만 사용하고 싶은 경우
    - 코드
        
        ```tsx
        import { HTTPError, type KyInstance, type Options } from 'ky';
        
        interface ApiSuccessResponse<T> {
          resultType: 'SUCCESS';
          success: T;
        }
        
        export interface ApiErrorResponse {
          statusCode: number;
          message: string;
          error: string;
        }
        
        export class ApiError extends Error {
          constructor(
            public statusCode: number,
            public error: string,
            message: string
          ) {
            super(message);
            this.name = 'ApiError';
          }
        }
        
        export class HttpClient {
          constructor(private client: KyInstance) {}
        
          private async request<T>(path: string, options?: Options): Promise<T> {
            try {
              const response = await this.client(path, options).json<ApiSuccessResponse<T>>();
        
              if (response.resultType === 'SUCCESS') {
                return response.success;
              }
        
              // 정상 응답이지만 SUCCESS가 아닌 경우 (거의 없을 것으로 예상)
              throw new Error('Invalid response type');
            } catch (error) {
              // HTTPError (4xx, 5xx)
              if (error instanceof HTTPError) {
                const errorBody = await error.response.json<ApiErrorResponse>();
                throw new ApiError(errorBody.statusCode, errorBody.error, errorBody.message);
              }
        
              // 네트워크 에러, 타임아웃 등
              throw error;
            }
          }
        
          get<T>(path: string, options?: Options): Promise<T> {
            return this.request(path, { ...options, method: 'get' });
          }
        
          post<T>(path: string, options?: Options): Promise<T> {
            return this.request(path, { ...options, method: 'post' });
          }
        
          put<T>(path: string, options?: Options): Promise<T> {
            return this.request(path, { ...options, method: 'put' });
          }
        
          delete<T>(path: string, options?: Options): Promise<T> {
            return this.request(path, { ...options, method: 'delete' });
          }
        
          patch<T>(path: string, options?: Options): Promise<T> {
            return this.request(path, { ...options, method: 'patch' });
          }
        }
        
        export function isApiError(error: unknown): error is ApiError {
          return error instanceof ApiError;
        }
        
        ```
        
    
- **5. API interceptor**
    - 순환 참조 조심
        - *기존에 순환참조 위험이 있어서 refresh 전용 API instance 를 만들었습니다.> 기존에는 apiClient 에서 api 요청 (create가맹점) -> 토큰 만료 -> apiClient 의 interceptor 훅에서 API 요청 처리 -> API 요청 처리에서 토큰 만료 -> 또 apiClient 에서 요청 실패 등 -> 순환참조 발생
    - 코드
        
        ```tsx
        import { getPhase } from '@ishopcare/phase';
        import { is } from '@tossteam/is';
        import ky, { HTTPError, type NormalizedOptions } from 'ky';
        import { HttpClient } from 'lib/HttpClient';
        import { logout, saveTokens } from 'lib/auth';
        import { sentryService } from 'lib/sentry-service';
        import { useAuthStore } from 'stores/auth.store';
        import { postRefreshToken } from './auth';
        
        const API_ENDPOINTS = {
          local: 'https://retool-api.dev.ishopcare.bz',
          dev: 'https://retool-api.dev.ishopcare.bz',
          live: 'https://public-api.ishopcare.bz',
        } as const;
        
        /**
         * @description refresh API 전용 ky 인스턴스 (순환 참조 방지)
         * @memo 인터셉터가 없어서 refresh API가 401 발생 시 무한 루프 방지
         */
        export const refreshApiClient = ky.create({
          prefixUrl: getApiPrefixUrl(),
        });
        
        /**
         * @description 일반 API용 ky 인스턴스 (인터셉터 포함)
         */
        **export const apiClient = ky.create({
          prefixUrl: getApiPrefixUrl(),
          retry: {
            limit: 2, // 최대 2번 재시도 (401 토큰 갱신 포함)
            methods: ['get', 'post', 'put', 'delete', 'patch'],
            statusCodes: [401], // 401 에러만 재시도
          },
          hooks: {
            beforeRequest: [beforeRequestInterceptor],
            afterResponse: [afterResponseInterceptor, afterResponseErrorReporter],
          },
        });**
        
        **export const httpClient = new HttpClient(apiClient);**
        
        /**
         * @description refresh API 전용 HttpClient (순환 참조 방지)
         */
        **export const refreshHttpClient = new HttpClient(refreshApiClient);**
        
        export function getApiPrefixUrl(): string {
          const phase = getPhase();
          const apiUrl = API_ENDPOINTS[phase];
        
          if (is.falsy(apiUrl)) {
            throw new Error('서버와 연결할 수 없습니다.');
          }
        
          return apiUrl;
        }
        
        /**
         * @description 모든 요청에 AccessToken을 자동으로 주입
         */
        **function beforeRequestInterceptor(request: Request) {
          const token = useAuthStore.getState().tokens?.accessToken;
          if (is.truthy(token)) {
            const headers = new Headers(request.headers);
            headers.set('Authorization', `Bearer ${token}`);
            return new Request(request, { headers });
          }
          return request;
        }**
        
        /**
         * @description 에러 응답을 Sentry에 리포팅
         * @memo 401은 정상적인 토큰 만료이므로 리포팅 제외
         */
        **async function afterResponseErrorReporter(request: Request, options: NormalizedOptions, response: Response) {
          // 401은 토큰 만료로 정상적인 케이스이므로 리포팅 제외
          if (is.falsy(response.ok) && response.status !== 401) {
            const error = new HTTPError(response, request, options);
            sentryService.captureApiError(error);
          }
        
          return response;
        }**
        
        /**
         * @description 401 응답이 동시에 발생하는 경우 여러 토큰 갱신 요청을 방지하기 위해 Promise를 공유
         * @memo Race Condition 방지
         */
        **let refreshTokenPromise: Promise<void> | null = null;**
        
        /**
         * @description 401 에러 발생 시 토큰 갱신 처리
         * @memo afterResponse에서 토큰 갱신 후, ky의 retry 메커니즘이 자동으로 재시도
         */
        **async function afterResponseInterceptor(_request: Request, _options: NormalizedOptions, response: Response) {
          if (response.status === 401) {
            const refreshToken = useAuthStore.getState().tokens?.refreshToken ?? '';
            if (is.emptyString(refreshToken)) {
              logout();
              return response;
            }
        
            // 이미 토큰 갱신 중이면 해당 Promise를 기다림 (Race Condition 방지)
            if (refreshTokenPromise === null) {
              refreshTokenPromise = (async () => {
                try {
                  // accessToken 만료 되어 refreshToken 으로 갱신 시도
                  const token = await postRefreshToken({ refreshToken });
                  saveTokens({
                    accessToken: token.accessToken,
                    refreshToken,
                  });
                } catch (error) {
                  // refreshToken 만료시 로그아웃 처리 및 리디렉션
                  if (error instanceof HTTPError) {
                    sentryService.captureApiError(error);
                  }
                  logout();
                  throw error;
                } finally {
                  refreshTokenPromise = null;
                }
              })();
            }
        
            // 토큰 갱신 완료 대기
            await refreshTokenPromise;
          }
        
          return response;
        }**
        
        export { ApiError, isApiError } from 'lib/HttpClient';
        
        ```
        
- **6. VO 클래스 정의**
    - https://github.dev/KIMSEUNGGYU/movies/blob/515325f89fb0dbc33a0c4d0386f9aa54f8932eaf/apps/movies/src/entities/tmdb/model/movie.service.ts#L8-L9
    
    **정의**
    
    - `shared/utils/createBaseModel.ts`
        
        ```tsx
        export const createBaseModel = <T>() => {
          return class {
            constructor(props: T) {
              Object.assign(this, props);
            }
          } as { new (args: T): Exclude<T, null | undefined> };
        };
        
        ```
        
    - `model/movie.service.ts`
        
        ```tsx
        import { createBaseModel } from '@/shared/utils/createBaseModel';
        
        import { MovieDetail } from './movie.type';
        
        type Cast = MovieDetail['credits']['cast'][number];
        type Crew = MovieDetail['credits']['crew'][number];
        
        abstract class MovieBase extends createBaseModel<MovieDetail>() {
          protected constructor(detail: MovieDetail) {
            super(detail);
          }
        
          // People 관련 로직
          get directors(): Crew[] {
            return this.credits.crew.filter((item) => item.department === 'Directing');
          }
          get mainActors(): Cast[] {
            return this.credits.cast.slice(0, 6);
          }
          get allCast(): Cast[] {
            return this.credits.cast;
          }
          get allCrew(): Crew[] {
            return this.credits.crew;
          }
        
          // 비즈니스 로직
          get formattedRuntime(): string {
            const hours = Math.floor(this.runtime / 60);
            const minutes = this.runtime % 60;
            return `${hours}시간 ${minutes}분`;
          }
        
          get movieInfo(): string {
            const releaseYear = this.releaseDate.split('-')[0];
            const runtime = this.formattedRuntime;
            const genres = this.genres.map((item) => item.name).join(' • ');
            return `${releaseYear} • ${runtime} • ${genres}`;
          }
        
          get formattedRating(): string {
            return this.voteAverage.toFixed(1);
          }
        
          get formattedVoteCount(): string {
            return this.voteCount.toLocaleString();
          }
        }
        
        // 실제 구현체
        export class Movie extends MovieBase {
          constructor(detail: MovieDetail) {
            super(detail);
          }
        }
        
        ```
        
    
    **사용**
    
    - `api/movie.ts`
        
        ```tsx
        import { api } from '@/shared/api';
        
        **import { Movie } from '../model/movie.service';**
        import { MovieDetail } from '../model/movie.type';
        
        export const fetchMovieDetail = (id: string) => {
          // axios 를 사용하면 params 를 객체로 전달하면 자동으로 쿼리 파라미터로 변환해줌
          const params = {
            language: 'ko-KR',
            append_to_response: 'credits,videos',
          };
        
          return api
            .get<MovieDetail>(`/movie/${id}`, { params })
            .then((res) => res.data)
            .then((json) => **new Movie(json))**;
        };
        
        ```
        
- **7. API 응답과 adapter 패턴**
    
    > VO 패턴으로 대체? 아니면 심플한 것은 adaptor 패턴으로?
    > 
    
    [select (어댑터 패턴) 적용 유무](https://www.notion.so/select-19ffc844aaad80958d8ce1ab3df84bdd?pvs=21) 
    
    - useQuery 과 adapter 패턴(mapper 함수 등) 을 같은 파일? 레벨로 두는게 좋음
        
        → 변경함수를 다른 파일 등에서 관리하면 응집도가 떨어짐  (컨테스트 스위칭 + 한눈에 보이지 않는 단점)
        
    - 따로 mapper 함수 정의 없이 useQuery 의 select 함수로 정의 가능