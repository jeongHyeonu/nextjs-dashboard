# Next.js 학습하기


소모임 [사이드나우]
- Next.js 스터디 기록 및 정리


## Next.js 튜토리얼
출처 : https://nextjs.org/learn/dashboard-app/getting-started 

(번역본) : https://wontory.gitbook.io/next.js-journey/learn-next.js/docs/introduction

### pnpm 설치 및 실행

pnpm은 기존 npm보다 빠르고 효율적이라고 한다. (공식 문서 : https://pnpm.io/)

- pnpm 설치 : `npm install -g pnpm`

- 프로젝트 패키지 설치 : `pnpm i`
- 개발서버 시작 : `pnpm dev`

### CSS 스타일링

#### 전역 스타일

최상단 루트 경로의 [`/app/layout.tsx`](https://github.com/jeongHyeonu/nextjs-dashboard/blob/main/app/layout.tsx)에 스타일을 추가하면 된다.

추가하면 `/app/page.tsx` 뿐만 아니라, `/app/dashboard/customers/page.tsx` 에도 스타일이 추가되는 방식이다.

모든 페이지에 tailwind와 같은 CSS 프레임워크를 추가해서 디자인하거나, 전역 폰트를 적용하는 데에 효과적이다.

- CSS 모듈을 사용하여 스타일 충돌의 위험을 낮추고, 모듈성을 향상시킬 수 있다.

#### clsx 라이브러리

클래스를 쉽게 전환해 줄 수 있고, 조건부 스타일링에 유용하다. 

아래는 예시 코드이다.


```
import clsx from 'clsx';

export default function InvoiceStatus({ status }: { status: string }) {
  return (
    <span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-sm',
        {
          'bg-gray-100 text-gray-500': status === 'pending',
          'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
    // ...
)}
```
status 에 따라 색상을 변경할 수 있다.

이외에도 CSS 라이브러리인 [styled-jsx](https://github.com/vercel/styled-jsx), [styled-components](https://github.com/vercel/next.js/tree/canary/examples/with-styled-components), [emotion](https://github.com/vercel/next.js/tree/canary/examples/with-emotion) 등 다른 스타일링 라이브러리를 적용해도 좋다.

### 글꼴/이미지 최적화

#### 글꼴 최적화가 필요한 이유?

글꼴 사용 시, 글꼴 로딩으로 인해 성능에 영향을 미칠 수 있음 (레이아웃 시프트)

이때 Next.js는 모듈을 사용할 때 글꼴을 자동으로 최적화해 다른 정적 에셋과 함께 호스팅 → **추가 네트워크 요청 발생X**

> 기본 글꼴 추가 예시 : [/app/ui/fonts.ts](https://github.com/jeongHyeonu/nextjs-dashboard/blob/main/app/ui/fonts.ts) 에 폰트 추가 후 [root layout](https://github.com/jeongHyeonu/nextjs-dashboard/blob/main/app/layout.tsx)에 적용 

> 보조 글꼴 추가 예시 : [/app/ui/fonts.ts](https://github.com/jeongHyeonu/nextjs-dashboard/blob/main/app/ui/fonts.ts) 에 폰트 추가 후 [page](https://github.com/jeongHyeonu/nextjs-dashboard/blob/main/app/page.tsx)에 적용




#### 이미지 최적화가 필요한 이유?

이미지가 화면 크기에 반응하고, 레이아웃 시프트를 방지하고, 이미지를 지연로드 해야 함

이때, Next.js는 `next/image` 컴포넌트를 사용해 이미지를 자동으로 최적화함

> 데스크톱/모바일에서 반응형 이미지 예시 : [`/app/page.tsx`](https://github.com/jeongHyeonu/nextjs-dashboard/blob/main/app/page.tsx) 에서 `<Image>` 컴포넌트 참조



### 페이지/레이아웃 생성

page 생성 예시 : [`/app/dashboard/page.tsx`](https://github.com/jeongHyeonu/nextjs-dashboard/blob/main/app/dashboard/page.tsx)

layout 생성 예시 : [`/app/dashboard/layout.tsx`](https://github.com/jeongHyeonu/nextjs-dashboard/blob/main/app/dashboard/layout.tsx)
- 이 레이아웃은 사이드바 역할을 하여 [`/app/dashboard/invoices/page.tsx`](https://github.com/jeongHyeonu/nextjs-dashboard/blob/main/app/dashboard/invoices/page.tsx) 또는 [`/app/dashboard/customers/page.tsx`](https://github.com/jeongHyeonu/nextjs-dashboard/blob/main/app/dashboard/customers/page.tsx) 와 같은 하위 폴더 page에도 적용된다.


### 페이지 탐색

페이지 간의 링크를 만들려면 기존 `<a>` 태그 대신, next/link 의 `<Link>` 태그를 이용한다.
 - 기존 `<a>` 태그는 페이지 전채를 다시 로딩하기 때문에, 속도가 느리고 깜빡거리는 현상 발생

next.js는 연결된 경로 코드를 사전로드 및 자동 코드 분할하여 페이지를 전환한다. *(대상 페이지를 미리 백그라운드에 로드 - 캐싱/부분렌더링 지원)*
 - react의 SPA와는 조금 다르다. *(react는 초기 로드에 모든 코드를 로드함)*

페이지간 라우팅에 대한 자세한 설명은 아래 공식 문서 참조

([next.js Docs : Linking and Navigating](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#how-routing-and-navigation-works))

### 활성화된 링크 표시

'next/navigation' 의 `usePathname` 훅을 사용해서 현재 경로를 URL 에서 불러오고, `clsx` 로 현재 링크와 일치하는 링크에 css 스타일링을 적용한다. 

자세한 사용법은 아래 '사이드바 예시' 링크를 확인하자.

([참고 : 사이드바 예시](https://github.com/jeongHyeonu/nextjs-dashboard/blob/main/app/ui/dashboard/nav-links.tsx) 
)