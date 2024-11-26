# 4장 타입 확장하기 , 좁히기

## 4.1 타입 확장 하기

---

### 4.1.1 타입 확장의 장점

- 코드 중복 제거, 명시적인 코드 작성

```tsx
// 메뉴 요소 타입
interface BaseMenuItem {
  itemName: string | null;
  itemImageUrl: string | null;
  itemDiscountAmount: number;
  stock: number | null;
}

// 장바구니 요소 타입 (메뉴 타입에 수량 정보 추가)
interface BaseCartItem extends BaseMenuItem {
  quantity: number;
}

// 메뉴 요소 타입
type BaseMenuItem = {
  itemName: string | null;
  itemImageUrl: string | null;
  itemDiscountAmount: number;
  stock: number | null;
};

type BaseCartItem = {
  quantity: number;
} & BaseMenuItem;
```

### 4.1.2 유니온 타입

- 주의) 유니온 타입으로 선언된 값은 유니온 타입에 포함된 모든 타입이 공통으로 갖고 있는 속성에만 접근할 수 있다.

```tsx
interface CookingStep {
  orderId: string;
  price: number;
}

interface DeliveryStep {
  orderId: string;
  time: number;
  distance: string;
}

function getDeliveryDistance(step: CookingStep | DeliveryStep) {
  return step.distance; // distance 는 DeliveryStep 에만 존재하는 속성이기에 에러 발생
}
// step은 CookingStep 혹은 DeliveryStep 타입에 해당할 뿐이지 CookingStep 이면서 DeliveryStep 인 것은 아니다.
```

### 4.1.3 교차 타입

- 타입스크립트의 타입을 속성 집합이 아니라 값의 집합으로 이해해야 한다.
- BaedalProgress 교차 타입은 CookingStep 이 가진 속성과 DeliveryStep 이 가진 속성을 모두 만족하는 값의 타입(집합) 이라고 해석할 수 있다.

```tsx
interface CookingStep {
  orderId: string;
  price: number;
}

interface DeliveryStep {
  orderId: string;
  time: number;
  distance: string;
}

type BaedalProgress = CookingStep & DeliveryStep;

// 유니온 타입과 다르게 BaedalProgress 는 CookingStep 과 DeliveryStep 타입을 합쳐 모든 속성을 가진 단일 타입이 된다.
function logBaedalInfo(progress: BaedalProgress) {
  console.log(progress.price);
  console.log(progress.distance);
}
```

### 4.1.3 extends 와 교차 타입

- extends 키워드를 사용해서 교차 타입을 작성할 수 있다.
- BaseCartItem 은 BaseMenuItem을 확장
  - BaseCartItem 은 BaseMenuItem 의 속성을 모두 포함하는 상위 집합
  - BaseMenuItem 은 BaseCartItem 의 부분집합
- 유니온 타입과 교차 타입은 오직 type 키워드로 선언할 수 있다.

```tsx
// 메뉴 요소 타입
interface BaseMenuItem {
  itemName: string | null;
  itemImageUrl: string | null;
  itemDiscountAmount: number;
  stock: number | null;
}

// 장바구니 요소 타입 (메뉴 타입에 수량 정보 추가)
interface BaseCartItem extends BaseMenuItem {
  quantity: number;
}

// 메뉴 요소 타입
type BaseMenuItem = {
  itemName: string | null;
  itemImageUrl: string | null;
  itemDiscountAmount: number;
  stock: number | null;
};

type BaseCartItem = {
  quantity: number;
} & BaseMenuItem;
```

- extends 키워드를 사용한 타입이 교차 타입과 100% 상응하지 않는다.

```tsx
interface DeliveryTip {
  tip: number;
}

interface Filter extends DeliveryTip {
  tip: string; // 타입 호환이 되지 않는 다고 에러 발생
}
```

```tsx
type DeliveryTip = {
  tip: number;
};

type Filter = {
  tip: string;
} & DeliveryTip;
/* 
  type 키워드는 교차타입으로 선언되었을 때 새롭게 추가되는 속성에 대해 미리 알 수 없어 선언시 에러가 발생하지 않는다.
  tip 이라는 속성에 대해 서로 호환되지 않는 타입이 선언되어 결국 never 타입
*/
```

## 4.2 타입 좁히기 - 타입 가드

---

- 타입 좁히기는 변수 또는 표현식의 타입 범위를 더 작은 범위로 좁혀나가는 과정을 말한다.
  - 정확하고 명시적인 타입 추론 및 복잡한 타입을 작은 범위로 축소하여 타입 안정성을 높인다.

### 4.2.1 타입 가드에 따라 분기 처리 하기

- 조건문과 타입 가드를 활용하여 변수나 표현식의 타입 범위를 좁힌다.
- 타입 가드는 런타임에 조건문을 사용하여 타입 검사하고 타입 범위를 좁혀주는 기능이다.
- 타입 가드는 크게 자바스크립트 연산자를 사용한 타입 가드와 사용자 정의 타입 가드로 구분할 수 있다.
  - 자바스크립트 연산자를 활용한 타입 가드 : typeof, instanceof, in 과 같은 연산자 사용
    - 런타임에 유효한 타입 가드를 만들기 위해서
  - 사용자 정의 타입 가드 : 사용자가 직접 어떤 타입으로 값을 좁실지를 직접 지정하는 방식

### 4.2.2 원시 타입을 추론 할 때 : typeof 연산자 활용하기

- typeof 연산자를 활용하면 원시 타입에 대해 추론할 수 있다.
  - string, number, boolean, undefined, object, function, bigint, symbol
- null 과 배열 타입 등이 object 타입으로 판별되는 등.. 복잡한 타입을 검증하기에는 한계가 있다.

### 4.2.3 인스턴스화된 객체 타입을 판별할 때 : instanceof 연산자 활용하기

- instanceof 연산자는 인스턴스화된 객체 타입을 판별하는 타입 가드로 사용할 수 있다.
- A instanceof B 형태로 사용하며 A에는 타입을 검사할 대상 변수, B에는 특정 객체의 생성자가 들어간다.
- instanceof는 A의 프로토타입 체인에 생성자 B 가 존재하는 지 검사하여 존재하면 true, 아니면 false 를 반환한다.

```tsx
const onKeyDown = (event: React.KeyboardEvent) => {
  if (event.target instanceof HTMLInputElement && event.key === 'Enter') {
    // event.target 의 타입이 HTMLInputElement 이며 event.key 가 "Enter" 인 경우 이다.
    event.target.blur();
    onCTAButtonClick(event);
  }
};
```

### 4.2.4 객체의 속성이 있는 지 없는 지에 따른 구분 : in 연산자 활용하기

- 객체에 속성이 있는 지 확인한 다음에 true 혹은 false 를 반환 한다.
- A in B 형태로 사용하며, A라는 속성이 B 객체에 존재하는 지를 검사한다.
  - 프로토타입 체인으로 접근할 수 있는 속성이면 전부 true 반환한다.

```tsx
type NoticeDialogProps = BasicNoticeDialogProps | NoticeDialogWithCookieProps;

/* cookieKey를 갖는 NoticeDialogWithCookieProps 로 추론되고, 얼리 리턴을 했기에 if 문 스코프 밖에 위치하는 props 객체는 BasicNoticeDialogProps 타입으로 해석한다.
 */
const NoticeDialog: React.FC<NoticeDialogProps> = (props) => {
  if ('cookieKey' in props) return <NoticeDialogWithCookie {...props} />;
  return <NoticeDialogBase {...props} />;
};
```

### 4.2.5 is 연산자로 사용자 정의 타입 가드 만들어 활용하기

- A is B 형태로 사용하며, A는 매개변수 이름이고 B는 타입이다.
- true/false 의 진릿값을 반환하면서 반환 타입을 타입 명제로 지정하게 되면 반환 값이 참일 때, A 매개변수의 타입을 B 타입으로 취급하게 된다.

```tsx
/*
	만약에 isDestinationCode의 반환 값에 is 를 사용하지 않고 boolean 을 사용하면
	x 의 타입을 좁히지 못하고 string 타입으로만 추론한다.
*/
const isDestinationCode = (x: string): x is DestinationCode => destinationCodeList.includes(x);
```

## 4.3 타입 좁히기 - 식별할 수 있는 유니온 (Discriminated Unions)

---

- 종종 태그된 유니온 (Tagged Union) 으로도 불린다.
  - 식별할 수 있는 (Discriminated Unions) 유니온은 타입 좁히기에 종종 사용된다.

### 4.3.1 에러 정의하기 / 식별할 수 있는 유니온

- 예를 들어, 사용자의 유효성 검사를 위해
  - 테스트 에러, 토스트 에러, 얼럿 에러가 있다고 가장하자.
  - 유효성 에러로, 에러 코드와 에러 메시지, 노출 방식에 대한 추가 정보가 필요할 것이다.

```tsx
type TextError = {
  errorCode: string;
  errorMessage: string;
};

type ToastError = {
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number; // 토스트 노출 시간
};

type AlertError = {
  errorCode: string;
  errorMesaage: string;
  onConfirm: () => void; // 얼럿 창 확인 버튼 누른 뒤 액션
};

type ErrorFeedbackType = TextError | ToastError | AlertError;
const errorArr: ErrorFeedbackType[] = [
  { errorCode: '1', errorMessage: '텍스트 에러!' },
  { errorCode: '1', errorMessage: '토스트 에러!', toastShowDuration: 3000 },
  { errorCode: '1', errorMessage: '얼럿 에러!', onConfirm: () => {} },

  ...{ errorCode: '1', errorMessage: '토스트 에러!', toastShowDuration: 3000, onConfirm: () => {} },
  /* 
    타입 에러를 뱉어야 하지만, 타입 에러 발생 X
    이런 상황에 대해서 타입 에러가 발생하지 않는 다면 의미없는 에러 객체가 생겨날 수 있다.
  */
];
```

**에러 타입을 구분할 방법이 필요하다!**

- 각 타입이 비슷한 구조를 가지지만 서로 호환되지 않도록 만들어주기 위해 타입들이 서로 포함관계를 가지지 않도록 정의
  - 식별할 수 있는 유니온을 활용
  - 타입마다 구분할 수 있는 판별자(태그)를 달아주어 포함 관계를 제거하는 것이다!

```tsx
type TextError = {
  errorType: 'TEXT';
  errorCode: string;
  errorMessage: string;
};

type ToastError = {
  errorType: 'TOAST';
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number; // 토스트 노출 시간
};

type AlertError = {
  errorType: 'ALERT';
  errorCode: string;
  errorMesaage: string;
  onConfirm: () => void; // 얼럿 창 확인 버튼 누른 뒤 액션
};

type ErrorFeedbackType = TextError | ToastError | AlertError;
```

### 4.3.2 식별할 수 있는 유니온의 판별자 선정

- 식별할 수 있는 유니온의 판별자는 유닛 타입으로 오직 하나의 정확한 값을 나타내는 타입을 말한다.
- 유니온의 판별자로 사용할 수 있는 타입
  - 리터럴 타입
  - 판별자로 선정한 값에 적어도 하나 이상의 유닛 타입이 포함되어야 하며, 인스턴스화할 수 있는 타입은 포함되지 않아야 한다.
    - void, string, number, Error 유닛 타입 적용 X

## 4.4 Exhaustiveness Checking 으로 정확한 타입 분기 유지하기

---

- Exhaustiveness(철저함/완전함) Checking 은 모든 케이스에 대해 철저하게 타입을 검사하는 것이다.
  - 모든 케이스에 대한 타입 분기 처리를 해주지 않았을 때, 컴파일 타임 에러가 발생하게 하는 것.

```tsx
type ProductPrice = '10000' | '20000' | '5000';

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === '10000') return '만원';
  if (productPrice === '20000') return '이만원';
  // if (productPrice === "5000") return "오천원";
  else {
    exhaustiveCheck(productPrice); // 컴파일 타임 에러가 발생
    return '상품권';
  }
};

/*
  매개변수를 never 타입으로 선언하여 매개 변수로 어떤 값도 받을 수 없으며 에러를 내뱉는다.
  타입 처리 조건문 마지막 else 문에 사용하면 앞의 조건 문에서 모든 타입에 대한 분기 처리를 강제할 수 있다.
*/
const exhaustiveCheck = (param: never) => {
  throw new Error('type error!');
};
```
