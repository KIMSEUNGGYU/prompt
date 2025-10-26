## 프론트엔드 개발용, 타입스크립트 

- 타입스크립트 작성시, 새로운 타입만든 것보다 기존에 있던 타입을 있으면 해당 타입을 활용하기
- 라이브러리의 타입을 활용해서 타입 추론 및 타입 정의하기 
- typescript 에서 기본적으로 제공하는 타입들을 활용하여 타입 추론 및 타입 정의하기 (e.g. ComponentProps 등..)

TODO...

```ts
import type { ComponentProps } from 'react';
import { Map as _KakaoMap, useKakaoLoader } from 'react-kakao-maps-sdk';

interface KakaoMapProps extends ComponentProps<typeof _KakaoMap> {
  children?: React.ReactNode;
}
```

```ts
import { type UndefinedInitialDataOptions, queryOptions } from '@tanstack/react-query';

export const meQueryOptions = (options?: Omit<UndefinedInitialDataOptions<ApiResponse<Me>>, 'queryKey' | 'queryFn'>) =>
  queryOptions({
    queryKey: ['me'],
    queryFn: fetchMe,
    ...options,
  });

```
