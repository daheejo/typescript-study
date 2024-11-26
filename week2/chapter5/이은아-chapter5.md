# 5장 타입 활용하기

## 5.1 조건부 타입

### 5.1.1 extends 와 제네릭을 활용한 조건부 타입

- T extends U ? X : Y
  - T를 U 에 할당할 수 있으면 X 타입, 아니면 Y 타입으로 결정된다.

```tsx
interface Bank {
  financialCode: string;
  companyName: string;
  name: string;
  fullName: string;
}

interface Card {
  financialCode: string;
  companyName: string;
  name: string;
  appCardType: string;
}

type PayMethod<T> = T extends 'card' ? Card : Bank;
type CardPayMethod = PayMethod<'card'>;
type BankPayMethod = PayMethod<'bank'>;
```

### 5.1.2 infer를 활용해서 타입 추론하기

- extends 를 사용할 때 infer 키워드를 사용할 수 있다.
- 삼항 연산자를 사용한 조건문의 형태를 가지는데, extends 로 조건을 서술하고 infer 로 타입을 추론하는 방식을 취한다.

```tsx
/* 
	UnpackPromise 타입은 제네릭으로 T를 받아 T가 Promise로 래핑된 경우라면 K를 반환하고, 아닌 경우 any를 반환
*/
type UnpackPromise<T> = T extends Promise<infer K>[] ? K : any;
```

참고 ) [https://velog.io/@from_numpy/TypeScript-infer](https://velog.io/@from_numpy/TypeScript-infer)

## 5.2 템플릿 리터럴 타입 활용하기

- 템플릿 리터럴 타입은 자바스크립트의 템플릿 리터럴 문법을 사용해 특정 문자열에 대한 타입을 선언할 수 있는 기능이다.

```tsx
type HeadingNumber = 1 | 2 | 3 | 4 | 5;
type HeaderTag = `h${HeadingNumber}`;
```

- 타입스크립트 컴파일러가 유니온을 추론하는데 시간이 오래 걸리면 비효율적이기 때문에 타입스크립트가 타입을 추론하지 않고 에러를 내뱉을 때가 있다.
  - 유니온 조합의 경우의 수가 너무 많지 않게 적절하게 나누어 타입을 정의하는 것이 좋다.

## 5.3 커스텀 유틸리티 타입 활용하기

- 유틸리티 타입을 활용한 커스텀 유틸리티 타입을 제작해서 사용할 수 있다.

### 5.3.1 유틸리티 함수를 활용해 styled-components 의 중복 타입 선언 피하기

**Props 타입과 styled-components 타입의 중복 선언 및 문제점**

- pick 유틸리티 타입을 통해 props 에서 필요한 부분만 선택하여 styled-components 컴포넌트의 타입을 정의할 수 있다.
  - 중복된 코드를 작성하지 않아도 되고 유지보수를 더욱 편리하게 할 수 있다.

```tsx
// HrComponent.tsx
export type Props = {
	height?: string;
	color?: keyof typeof colors;
	isFull?: boolean;
	className?: string;
	...
}

export const Hr: VFC<Props> = ({height, color, isFull, className}) => {
	return <HrComponent ...  />;
};

// style.ts
import {Props} from "...";
type styledProps = Pick<Props, "height" | "color" | "isFull" >;
const HrComponent = styled.hr<StyledProps>`
	...
`;
```

### 5.3.2 PickOne 유틸리티 함수

- 서로 다른 2개 이상의 객체를 유니온 타입으로 받을 때 타입 검사가 제대로 진행되지 않는 이슈가 있다.
  - 이런 문제 해결을 위해 PickOne 이라는 유틸리티 함수를 구현해보자!

```tsx
type Card = {
	card: string;
};

type Account = {
	account: string;
};

function withdraw(type: Card | Account) {
	...
}

withdraw({card: "hyundai", account: "hana"});

/*
	타입 에러가 발생하지 않는 이유는?
	집합 관점으로 볼때 유니온은 합집합이 되기 때문이다.
	따라서 card, account 속성이 하나씩 할당된 상태도 허용하지만, card / account 속성이 모두 포함되어도
	합집합의 범주에 들어가기에 타입 에러가 발생하지 않는다.
*/
```

**식별할 수 있는 유니온으로 객체 타입을 유니온으로 받기**

- 식별할 수 있는 유니온(Discriminated unions)은 각 타입에 type 이라는 공통된 속성을 추가하여 구분하는 방법이다.
  - type을 일일이 넣어줘야 하는 불편함이 생긴다.
  - 이미 구현된 상태에서 적용하게 되면 해당 함수를 사용하는 부분을 모두 수정해야한다.

```tsx
type Card = {
	type: "card";
	card: string;
};

type Account = {
	type: "account";
	account: string;
};

function withdraw(type: Card | Account) {
	...
}

withdraw({type: "card" , card: "hyundai"});
withdraw({type: "account", account: "hana"});
```

**PickOne 커스텀 유틸리티 타입 구현하기**

- 구현하고자 하는 타입
  - account 또는 card 속성 하나만 존재하는 객체를 받는 타입이다.
  - account 일 때는 card를 받지 못하고, card 일 때는 account 를 받지 못하게 하려면 하나의 속성이 들어왔을 때 다른 타입을 옵셔널한 undefined 값으로 지정하는 방법을 생각해볼 수 있다.

```tsx
{ account: string; card?: undefined} | {account?: undefined; card: string;}

type PayMethod =
| {account: string; card?: undefined; payMoney?: undefined}
| {account?: undefined; card: string; payMoney?: undefined}
| {account?: undefined; card?: undefined; payMoney: string}
```

→ 결국, 하나의 속성을 제외하고 나머지 값을 옵셔널 타입 + undefined 로 설정하면 원하고자 하는 속성만 받도록 구현할 수 있다.

**PickOne 살펴보기**

```tsx
type PickOne<T> = {
  [P in keyof T]: Record<P, T[P]> & Partial<Reacord<Exclude<keyof T, P>, undefined>>;
}[keyof T];
```

- T 에는 객체가 들어온다고 가장하자!

```tsx
type One<T> = {[P in keyof T]: Record<P, T[P]>}[keyof T];

/*
	1. [P in keyof T] 에서는 T 는 객체로 가정하기의 P는 T의 객체 키 값이다.
	2. Record<P, T[P]> 는 P를 키 로 가지고, P를 키로둔 T 객체의 값의 레코드 타입이다.
	3. {[P in keyof T]: Reacord<P, T[P]>} 키는 T 객체의 키 모음이고, value 는 해당 키의 원본 객체 T 이다.
	4. 3의 타입에서 다시 [keyof T] 의 키값으로 접근하기에 최종결과는 전달받은 T와 같다.
*/

type Card = { card: string };
const one: One<Card> = { card: "hyundai" }

type ExcludeOne<T> = {[P in keyof T] : Partial<Reacord<Exclude<keyof T, P>, undefined>>;
}[keyof T];

/*
	1. [P in keyof T] 에서는 T 는 객체로 가정하기의 P는 T의 객체 키 값이다.
	2. Exclude<keyof T, P> 는 T 객체가 가진 키 값에서 P타입과 일치하는 키값을 제거 한다.
	3. Record<A, undefined> 키로 를, 값으로 undefined 타입을 갖는 레코드 타입이다.
	즉, 전달 받은 객체 타입을 모두 {[key] : undefined } 형태로 만든다.
	4. Partial<B>는 B타입을 옵셔널로 만든다. 따라서 {[key]? : undefined} 와 같다.
	5. [P in keyof T]로 매핑된 타입에서 동일한 객체인 [keyof T] 로 접근하기에 4번 타입이 반환된다.
	최종적으로 4번 타입이 반환된다.
*/

type PickOne<T> = One<T> & ExcludeOne<T>;

/*
	1. [P in keyof T] 를 공통으로 갖기 때문에 아래와 같이 교차
	2. 아래 타입을 해설하면 전달된 T 타입의 1개의 키는 값을 가지고 있으며, 나머지 키는 옵셔널한 undefined 객체를 의미한다.
*/
[P in keyof T]: Record<P, T[P]> & Partial<Reacord<Exclude<keyof T, P>, undefined>>;
```

### 5.3.3 NonNullable 타입 검사 함수를 사용하여 간편하게 타입 가드하기

- null 을 가질 수 있는 값의 null 처리는 자주 사용되는 타입 가드 패턴의 하나이다.

**NonNullable 타입이란**

- 유틸리티 타입으로 제네릭으로 받는 T가 null 또는 undefined 일 때, never 또는 T를 반환하는 타입이다.

```tsx
type NonNullable<T> = T extends null | undefined ? never | T;
```

**null, undefined 를 검사해주는 NonNullable 함수**

- NonNullable 유틸리티 타입을 사용하여 null 또는 undefined를 검사해주는 타입 가드 함수를 만들어 쓸 수 있다.
- 매개변수인 value 가 null 또는 undefined 라면 false 를 반환한다.
- is 키워드가 쓰였기에 NonNullable 함수를 사용하는 쪽에서 true 가 반환된다면 넘겨준 인자는 null 이나 undefined 가 아닌 타입으로 타입가드가 된다.

```tsx
function NonNullable<T>(value: T): value is NonNullable<T> {
  return value !== null && value !== undefined;
}
```

## 5.4 불변 객체 타입으로 활용하기

```tsx
const colors = {
  red: '#F45452',
  green: '#0C952A',
  blue: '#1A7CFF',
};

// key 의 타입을 string 으로 선언하여 colors 에 어떤 값이 추가될 지 모르기에 getColorHex 반환값은 any 가 된다.
const getColorHex = (key: string) => colors[key];
```

- as const 키워드로 불변 객체를 선언하고, keyof 연산자를 사용하여 getColorHex 함수 인자로 실제로 colors 객체에 존재하는 키 값만 받도록 설정 할 수 있다.
- 타입에 맞지 않는 값을 전달할 경우 타입 에러가 반환되기에 컴파일 단계에서 발생할 수 있는 실수를 방지할 수 있다.
- 자동완성 기능을 통해 객체 어떤 값이 있는 지 쉽게 파악할 수 있다.

### 5.4.1 Atom 컴포넌트에서 theme style 객체 활용하기

- Atom 단위의 작은 컴포넌트는 다양한 환경에서 유연하게 구현되어야 한다.
- 프로젝트의 스타일 값을 관리해주는 theme 객체를 두고 관리한다.
- Atom 컴포넌트에서는 theme 객체의 색상, 폰트 사이즈의 키값을 받은 뒤 theme 객체에서 값을 받아오도록 설계한다.

**타입스크립트 keyof 연산자로 객체의 키 값을 타입으로 추론하기**

- keyof 연산자는 객체 타입을 받아 해당 객체의 키 값을 string 또는 number 의 리터럴 유니온 타입을 반환한다.
- 인덱스 시그니처가 사용되었다면 인덱스 시그니처의 키 타입을 반환한다.

```tsx
interface ColorType {
  red: string;
  green: string;
  blue: string;
}

type ColorKeyType = keyof ColorType; // 'red' | 'green' | 'blue'
```

**타입스크립트 typeof 연산자로 값을 타입으로 다루기**

- 변수 혹은 속성의 타입을 추론하는 역할을 한다.
- typeof 연산자는 단독으로 사용되기 보다 주로 유틸리티 타입이나 keyof 연산자 같이 타입을 받는 연산자와 함께 쓰인다.

```tsx
const colors = {
  red: '#F45452',
  green: '#0C952A',
  blue: '#1A7CFF',
};

type ColorsType = typeof colors;
/*
{
	red: string;
	green: string;
	blue: string;
}
*/
```

**객체의 타입을 활용해서 컴포넌트 구현하기**

- theme 객체로 타입을 구체화

```tsx
const colors = {
  red: '#F45452',
  green: '#0C952A',
  blue: '#1A7CFF',
};

const theme = {
  colors: {
    default: colors.gray,
    ...colors,
  },
  backgroundColor: {
    default: colors.white,
    gray: colors.gray,
    mint: colors.mint,
    black: colors.black,
  },
};

type ColorType = keyof typeof theme.colors;
type BackgroundColorType = keyof typeof theme.backgroundColor;

interface Props {
  color?: ColorType;
  backgroundColor?: BackgroundColorType;
}
```

참고) [https://inpa.tistory.com/entry/TS-📘-타입스크립트-keyof-typeof-사용법](https://inpa.tistory.com/entry/TS-%F0%9F%93%98-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-keyof-typeof-%EC%82%AC%EC%9A%A9%EB%B2%95)

## 5.5 Record 원시 타입 키 개선하기

### 5.5.1 무한한 키를 집합으로 가지는 Record

```tsx
type Category = string;
interface Food {
  name: string;
}

const foodByCategory: Record<Category, Food[]> = {
  한식: [{ name: '김찌' }, { name: '된찌' }],
  일식: [{ name: '초밥' }, { name: '텐동' }],
};

// foodByCategory["양식"]을 Food[] 로 추론
// foodByCategory["양식"] 은 런타임에서 undefined 가 되어 오류 반환
foodByCategory['양식'].map((food) => console.log(food));

// 옵셔널 체이닝을 통해 런타임에러를 방지할 수 있지만 어떤 값이 undefined 인지 판단해야 하는 번거로움이 있다.
foodByCategory['양식']?.map((food) => console.log(food));
```

### 5.5.2 유닛 타입으로 변경하기

- 키가 유한한 집합이라면 유닛 타입을 사용할 수 있다.
- 키가 무한해야 하는 상황에서는 적합하지 않다.

```tsx
type Category = '한식' | '일식';
interface Food {
  name: string;
}

const foodByCategory: Record<Category, Food[]> = {
  한식: [{ name: '김찌' }, { name: '된찌' }],
  일식: [{ name: '초밥' }, { name: '텐동' }],
};
```

### 5.5.3 Partial을 활용하여 정확한 타입 표현하기

- 키가 무한한 상황에서는 Partial을 사용하여 해당 값이 undefined 일 수 있는 상태임을 표현할 수 있다.

```tsx
type PartialRecord<K extends string, T> = Partial<Record<K, T>>;
type Category = string;

interface Food {
  name: string;
}

const foodByCategory: PartialRecord<Category, Food[]> = {
  한식: [{ name: '김찌' }, { name: '된찌' }],
  일식: [{ name: '초밥' }, { name: '텐동' }],
};

foodByCategory['양식']; // Food[] 또는 undefined 타입으로 추론 (사전 조치 가능)
```

- 참고 ) [https://www.typescriptlang.org/ko/docs/handbook/utility-types.html#partialtype](https://www.typescriptlang.org/ko/docs/handbook/utility-types.html#partialtype)
