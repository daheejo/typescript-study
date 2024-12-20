# 3장 고급타입

## 3-1 타입스크립트 만의 독자적인 타입시스템

타입스크립트는 자바스크립트의 슈퍼셋으로  **타입스크립트의 타입 시스템이 내포하고 있는 개념은 모두 자바스크립트에서 기인한 타입 시스템**입니다.

단지 자바스크립트로 표현할 수단과 필요성이 없었을 뿐이다. 자바스크립트의 수퍼셋으로 정적 타이핑을 할 수 있는 타입스크립트가 등장하면서 비로소 타입스크립트의 타입시스템이 구축되었습니다.

예를 들어 타입스크립트의 any 타입은 자바스크립트의 typeof 연산자나 Object.prototype.toSting.call() 을 사용하여서 콘솔에서 변수 타입을 추척해봐도 any 문자열을 반환하는 경우를 찾을수 없습니다.

따라서 any 타입은 타입스크립트에만 존재하는 독자적인 타입시스템으로 간주 할 수있습니다. 그러나 any 타입의 개념은 이미 자바스크립트에서 널리 사용되고 있습니다. any라는 이름 그대로 어떤 타입이든 매핑할 수 있는 성질을 가지고 있는데 이는 원래 자바스크립트의 사용 방식과 일치하기 때문입니다.

타입스크립트의 타입 계층구조

![image.png](3%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%80%E1%85%A9%E1%84%80%E1%85%B3%E1%86%B8%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8%20f5cf0151cc8042b499cb87e203bea2d5/image.png)

### any 타입

any 타입은 즉 자바스크립트에서 타입을 명시하지 않은것과 동일한 효과를 나타냅니다.

 아래 예시에서 어떠한 값을 할당하더라도 오류가 발생하지 않는것을 확인할 수 있습니다.

```jsx
let state: any;

state = { value: 0 }; // 객체를 할당해도
state = 100; // 숫자를 할당해도
state = "hello world" // 문자열을 할당해도
state.foo.bar = () => console.log("type"); // 중첩구조로 함수를 할당해도 

// 문제가 없는것을 확인 할 수있습니다.
```

### any 타입의 문제

어떠한 값이라도 받아올수 있는 any 타입은 타입 안정성을 저해하고, 정적타이핑을 적용하는것이 목적인 타입스크립트에서 지양해야하는 패턴이다.  any 타입을 사용할 수 있는 3가지 경우가 있는데 이는 아래와 같다.

<aside>
📌

tsconfig.json 에서 noImplictitAny 옵션을 활성화 하면 any 타입 사용시 에러를 발생시킬수 있다.

</aside>

## Typescript 에서 Any 타입을 사용하는 경우 3가지

### 1. 개발 단계에서 임시로 값을 지정해야 할 때

복잡한 기능 개발 과정에서 추후 값이 변경될 가능성이 있거나 아직 세부항목에 대한 타입이 확정되지 않은 경우 any 로 지정해서 타입을 명시하는데 소요하는 시간을 절약할수 있고 개발을 진행할 수 있다. 

any 타입을 남발하면 타입 안정성을 저해하기 때문이 임시로 타입을 지정할때 주로 사용한다.

세부 스펙이 나오는 시점에는 다른 타입으로 대체해야하며 이 과정이 누락될 경우 문제가 발생할 수 있다.

### 2. 어떤 값을 받아올지 또는 넘겨줄지 정할 수 없을 때

예상할수 없는 API 요청 및 응답 처리, 콜백 함수 전달, 외부 라이브러리를 사용할 때는 어떤 유형의 데이터를 받을지 특정하기 어려울때 열린 타입을 선언해야 할 때 사용할 수 있다. 

예시 코드

```jsx
type FeedbackModalParams = {
	show: boolean,
	content: string,
	cancelButtonText?: string,
	confirmButtonText?: string,
	beforeOnClose?: () => void,
	action?: any,
}
```

피드백을 나타내기 위해 모달창을 그릴때 사용되는 인자를 나타내는 타입 FeedbackModalParams에서action 이라는 속성이 any로 선언된 것을 볼 수 있다. action 속성은 모달 창을 그릴 때 실행될 함수를 의미한다. 모달창을 화면에 그릴 때 다양한 범주의 액션에 따라 인자의 개수나 타입을 일일이 명시하기 힘들 수 있다. 이럴 때 any 타입을 사용하면 다양한 액션 함수를 전달 할 수있다.

### 3. 값을 예측할 수 없을 때 암묵적으로 사용한다.

외부 라이브러리나 웹 API의 요청에 따라 다양한 값을 반환하는 API가 존재할 수 있다. 대표적인 예로 브라우저의 Fetch API를 들 수 있다. Fetch API의 일부 메서드는 요청 이후의 응답을 특정 포맷으로 파싱하는데 이때 반환 타입이 any 로 매핑되어있는 것을 볼 수있다.

이렇게 예외적으로 any타입을 사용해야 하는 상황이 있음에도 되도록 any 타입은 지양하는것이 좋다.

왜냐하면 타입스크립틔 타입검사를 무색하게 만들고 잠재적으로 위험한 상황을 초래할 가능성이 크기 때문이다.

앞선 예시에서 개발자의 의도에 따르면 action 속성에 함수만 넘겨주어야 하지만, 실수 또는 악의적으로 함수가 아닌 값을 넘기더라도 타입스크립트는 이를 에러로 간주하지 않는다. 타입스크립트의 컴파일러에서는 아무런 에러가 도출되지 않지만, 실제 런타임에서 심각한 오류가 발생할 수있다.

Any타입은 개발자에게 편의성과 확장성을 제공하기도 하지만 해당 값을 컨트롤 하려면 파악해야할 정보도 많다, 즉 도구의 도움을 받을 수 없는 상태에서 온전히 개발자 스스로 책임져야함을 의미한다. 이런 맥락을 이해하지 못하면 협업시 실수할 가능성이 커진다.

## Unkown 타입

unknown 타입은 any 타입과 유사하게 모든 타입의 값이 할당 될 수 있다. 그러나 any를 제외한 다른 타입으로 선언된 변수에는 unknown 타입 값을 할당 할 수 없다. 아래표는 any 와 unknown 타입의 비교내용을 담고있다.

**unknown타입과 any 타입 비교**

| any | unknown |
| --- | --- |
| 어떤 타입이든 any 타입에 할당이 가능하다. | 어떤 타입이든 unknown 타입에 할당이 가능하다. |
| any 타입은 어떤 타입으로 재할당 가능하다 (never 제외) | unknown 타입은 any 타입외에 다른 타입으로 할당이 불가능하다. |

⇒ unknown 타입 ⇒ any 타입 ⇒ 모든 타입으로 재할당이 가능하다 (never 제외)

unknown 타입은 타입스크립트 3.0이 릴리스 될때 추가되었는데 기존 타입 시스템에서 부족한 부분을 보완하기 위해 등장했다. Unknown에 대응되는 자바스크립트자료형이 무엇인지 쉽게 떠오르지 않을만큼 타입스크립트 만의타입 시스템이라고 볼 수 있는데, unknown 타입은 이름처럼 무엇이 할당될지 아직 모르는 상태의타입을 말한다.  이렇게 보면 any 와 비슷한데 왜 unknwon타입이 추가되었을까?

## unknown 타입과 any 타입은 무엇이 다른걸까?

```jsx
// 타입을 할당하는 시점에는 에러가 발생하지 않는다.
const unknownFunction: unknown = () => console.log("unknown type");

// 런타임시 에러가 발생한다.  Error: Object is of type 'unknwon' ts(2571)
unknownFunction()
```

unknown 타입을 할당할때 컴파일시에는 에러가 발생하지 않지만 런타임시 에러가 발생한다.

비단 함수 분 아니라 객체의 속성 접근, 클래스 생성자 호출을 통한 인스턴스 생성 등 객체 내부에 접근하는 모든 시도에서 에러가 발생한다. 할당시점에는 아무런 문제가 업는데 왜 호출시에 문제가 생기는걸까?

unknown타입은 어떤 타입이 할당되었는지 알 수 없음을 나타내기때문에 unknown 타입으로 선언된 변수는 값을 가져오거나 내부 속성에 접근할 수없다. 이는 unknown 타입으로 할당된 변수는 어떤 값이든 올 수 있음을 의미하는 동시에 개발자에게 엄격한 타입검사를 강제하는 의도를 담고있습니다.

any 타입을 사용하면 어떤 값이든 허용된다. 앞서 어떤 값이 할당될지 파악하기 어려운 상황에서 ant 타입을 지정하여 임시로 문제를 회피하는 예시도 살펴보았다.

## void 타입

> 함수 타입을 지정할 때 함수에 전달되는 매개변수의 타입과 반환하는 타입을 지정해야하는데, 이때 매개변수를 전달하지 않는 경우에는 그냥 괄호를 비워두면 되지만 아무런 값을 반환하지 않는 경우에는 void 타입을 매핑한다.
> 
> 
> 예를 들어 콘솔로그를 찍는 거나 다른 함수를 실행하는 역할만 하는 함수의 경우 특정 값을 반환하지 않으므로 void를 타입으로 매핑해주면 된다.
> 

```tsx
function showModal(type: ModalType): void {
  feedbackSlice.actions.createModal(type);
}

// 화살표 함수로 작성 시
const showModal = (type: ModalType): void => {
  feedbackSlice.actions.createModal(type);
}
```

자바스크립트에서 함수에 명식적 반환문을 작성하지 않으면 undefined가 반환되나, 타입스크립트에서 void 타입을 사용하는데 이것은 undefined가 아니다.

### 즉, 타입스크립트에서 어떤 값을 반환하지 않는 함수는 void 타입을 지정하여 사용하면 된다!

## never 타입

> never 타입도 일반적으로 함수와 관련해 많이 사용되는 타입이다. never는 값을 반환할 수 없는 타입이다. 자바스크립트에서 두 가지 상황에서 값을 반환할 수 없는데 예시로 알아보자.
> 

### 에러를 던지는 경우

> 자바스크립트에서 런타임에 의도적으로 에러를 발생시키고 캐치할 수 있다. throw 키워드를 사용하면 에러를 발생시킬 수 있는데, 이는 값을 반환하는 것으로 간주하지 않는다. 따라서 특정 함수가 실행 중 마지막에 에러를 던지는 작업을 수행한다면 해당 함수의 반환 타입은 never다.
> 

```tsx
function generateError(res: Response): never {
  throw new Error(res.getMessage());
}
```

### 무한히 함수가 실행되는 경우

> 드물지만 함수 내에서 무한 루프를 실행하는 경우가 있을 수 있다. 무한 루프는 결국 함수가 종료되지 않음을 의미하기 때문에 값을 반환하지 못한다.
> 

```tsx
function checkStatus(): never {
  while(true) {
    // ...
  }
}
```

### never 타입은 모든 타입의 하위 타입이다. 즉, never 자신을 제외한 어떤 타입도 never 타입에 할당될 수 없다는 것을 의미한다. any 타입이라 할지라도 never 타입에 할당될 수 없디.

## Array 타입

> 배열 타입을 가리키는 Array 키워드는 자바스크립트에서도 Object.prototype.toString.call(...) 연산자를 사용하여 확인할 수 있다. typeof는 객체 타입을 단순 object 타입으로 알려주지만, Object.prototype.toString.call(...) 함수는 객체의 인스턴스까지 알려주기 때문이다.
> 

```tsx
const arr = [];
console.log(Object.prototype.toString.call(arr)); // [object Array]
```

### 배열은 두 가지 형식으로 선언할 수 있다.

```tsx
const array: number[] = [1, 2, 3];
const array: Array<number> = [1, 2, 3];

const array1: number[] | string[] = [1, 'abc'];
// const array1: (number | string)[] = [1, 'abc']; 이렇게도 가능하다.
const array2: Array<number | string> = [1, 'abc'];
```

### 배열 타입을 명시하는 것으로 배열의 길이까지 제한할 수 없다. 그러나 튜플은?

> 튜플은 배열 타입의 하위 타입으로 기존 타입스크립트의 배열 기능에 길이 제한까지 추가한 ㅏㅌ입 시스템이다.
> 

```tsx
let tuple: [number] = [1];

tuple = [1, 2]; // 불가능
tuple = [1, 'abc']; // 불가능

let tuple: [number, string, boolean] = [1, 'abc', true]; // 여러 타입과 혼합도 가능
```

> 배열은 사전에 허용하지 않은 타입이 서로 섞이는 것을 방지하여 타입 안정성을 제공하고, 튜플은길이까지 제한하여 원소 개수와 타입을 보장한다.
> 

```tsx
// 튜플과 배열의 성질을 혼합해 사용한 예시
const httpStatusFromPaths: [number, string, ...string[]] = [
  400,
  'Bad Request',
  '/users/id',
  '/users/:userId',
  '/users/:uuid',
];
```

## enum 타입

> enum 타입은 열거형이라고도 부르는데 타입스크립트에서 지원하는 특수한 타입이다. enum은 일종의 구조체를 만드는 타입 시스템이다. enum을 사용해 열거형을 정의할 수 있는데 열겨형은 각각의 멤버를 가진다. 이것은 자바스크립트 객체와 닮았다. 다만 타입스크립트는 명명한 각 멤버의 값을 스스로 추론한다. 기본적인 추론 방식은 숫자 0부터 1씩 늘려가며 값을 할당하는 것이다.
> 

```tsx
enum ProgrammingLanguage {
  Typescript, // 0
  Javascript, // 1
  Java, // 2
  Python, // 3
  Kotlin, // 4
  Rust, // 5
  Go, // 6
}

// 각 멤버에게 접근하는 방식은 자바스크립트에서 객체의 속성에 접근하는 방식과 동일
ProgrammingLanguage.Typescript; // 0
ProgrammingLanguage.['Rust']; // 5

// 역방향 접근 또한 가능하다.
ProgrammingLanguage[2]; // 'Java'
```

### enum 타입은 주로 문자열 상수를 생성하는데 사용된다.

이를 통해 응집력있는 집합 구조체를 만들 수 있다.

```tsx
enum ItemStatusType {
  DELIVERY_HOLD = 'DELIVERY_HOLD', // 배송 보류
  DELIVERY_READY = 'DELIVERY_READY', // 배송 준비 중
  DELIVERING = 'DELIVERING', // 배송 중
  DELIVERED = 'DELIVERED', // 배송 완료
}

const checkItemAvailable = (itemStatus: ItemStatusType) => {
  switch(itemStatus) {
    case ItemStatusType.DELIVERY_HOLD:
    case ItemStatusType.DELIVERY_READY:
    case ItemStatusType.DELIVERING:
      return false;
    case ItemStatusType.DELIVERED:
    default:
      return true;
  }
}
```

### 함수 인자를 열거형 타입으로 가지는 위 예시의 장점

```
1. 타입 안정성: ItemStatusType에 명시되지 않은 다른 문자열은 인자로 받을 수 없다. 따라수 타입 안정성이 우수.

2. 명확한 의미 전달과 높은 응집력: ItemStatusType 타입이 다루는 값이 무엇인지 명확하다. 아이템 상태에 대한 값을 모아놓은 것으로 응집력이 뛰어남.

3. 가독성: 응집도가 높기 때문에 말하고자 하는 바가 더욱 명확하다. 따라서 열거형 멤버를 통해 어떤 상태를 나타내는지 쉽게 이해할 수 있다.
```

### 다만, 열거형에서 주의할 점이 있다!

> 숫자로만 이루어져 있거나 타입스크립트가 자동으로 추론한 열거형은 안전하지 않은 결과를 낳을 수 있다. 역방향으로 접근하는 경우 문제가 발생하는 예시를 보자.
> 

```tsx
ProgrammingLanguage[200]; // undefined를 출력하지만 별다른 에러를 발생시키지 않는다.
```

### 이 때, const enum을 사용하여 문제를 방지할 수 있다.

```tsx
const enum ProgrammingLanguage {
  // ...
}
```

### 그러나,

> const enum으로 열거형을 선언하더라도 숫자 상수로 관리되는 열거형은 선언 값 이외의 값을 할당하거나 접근할 때 이를 방지하지 못한다. 그런데 문자열 상수 방식으로 선언한 열거형은 미리 선언하지 않은 멤버로 접근을 방지한다.
> 

### 즉, 문자열 상수 방식으로 열거형을 사용하면 숫자 상수보다 더 안전하며 의도치 않은 값의 할당이나 접근을 방지하는 데 도움이 된다.

```tsx
const enum NUMBER {
  ONE: 1,
  TWO: 2,
}

const myNumber: NUMBER = 100; // NUMBER enum에서 100을 관리하고 있지 않지만 에러를 발생시키지 않는다

const enum STRING_NUMBER {
  ONE: 'ONE',
  TWO: 'TWO',
}

const myStringNumber: STRING_NUMBER = 'THREE'; // Error
```

### But! 열거형의 가장 큰 문제...

> 열거형은 타입스크립트 코드가 자바스크립트로 변횐될 때 즉시 실행 함수(IIFE) 형식으로 변환되는 것을 볼 수 있다.
> 
> 
> 이때 일부 번들러에서 트리쉐이킹 과정 중 즉시 실행 함수로 변환된 값을 사용하지 않는 코드로 인식하지 못하는 경우가 발생할 수 있다.
> 
> 따라서 불필요한 코드의 크기가 증가하는 결과를 초래할 수 있다. 이러한 문제를 해결하기 위해 앞서 언급한 const enum 또는 as const assertion을 사용해서 유니온 타입으로 열거형과 동일한 효과를 얻는 방법이 있다.
> 

# 3.2 타입 조합

## 1) 교차 타입(Intersection)

> 교차 타입을 사용하여 여러 가지 타입을 결합하여 하나의 단일 타입으로 만들 수 있다. 즉, 기존에 존재하는 다른 타입들을 합쳐서 해당 타입의 모든 멤버를 가지는 새로운 타입을 생성하는 것
> 
> 
> 교차 타입은 **&**을 사용해서 표기한다.
> 

ex. C가 A와 B의 교차 타입 즉, A & B라면 타입 C는 타입 A와 B의 모든 멤버를 가지고 있는 타입이다.

```tsx
type ProductItem = {
  id: number;
  name: string;
  type: string;
  price: number;
  imageUrl: string;
  quantity: number;
};

type ProductItemWithDiscount = ProductItem & { discountAmount: number };
```

## 2) 유니온 타입(Union)

> 교차 타입(A & B)이 타입 A와 타입 B를 모두 만족하는 경우라면, 유니온 타입은 타입 A 또는 타입 B 중 하나가 될 수 있는 타입을 말하며 A | B같이 표기한다.
> 

```tsx
type ProductItem = {
  id: number;
  name: string;
  type: string;
  price: number;
  imageUrl: string;
  quantity: number;
};

type CardItem = {
  id: number;
  name: string;
  type: string;
  price: number;
  imageUrl: string;
};

type PromotionEventItem = ProductItem | CardItem;

const printPromotionItem = (item: PromotionEventItem) => {
  console.log(item.name); // 0

  console.log(item.quantity); // 컴파일 에러
}
```

## 3) 인덱스 시그니처(Index Signatures)

> 인덱스 시그니처는 특정 타입의 속성 이름은 알 수 없지만 속성값의 타입을 알고 있을 때 사용하는 문법이다. 인터페이스 내부에 [Key: T] 꼴로 타입을 명시해주면 되는데 이는 해당 타입의 속성 키는 모두 K 타입이어야 하고 속성값은 모두 T 타입을 가져야 한다는 의미다.
> 

```tsx
interface IndexSignaturesEx {
  [key: string]: number;
}

interface IndexSignaturesEx2 {
  [key: string]: number | boolean;
  length: number;
  isValid: boolean;
  name: string; // 에러 발생!
}
```

## 4) 인덱스드 엑세스 타입(Indexed Access Types)

> 인덱스드 엑세스 타입은 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용된다. 아래 예시는 Example 타입의 a 속성이 가지는 타입을 조회하기 위한 인덱스드 엑세스 타입이다. 인덱스에 사용되는 타입 또한 그 자체로 타입이기 때문에 유니온 타입, keyof, 타입 별칭 등의 표현을 사용할 수 있다.
> 

```tsx
type Example = {
  a: number;
  b: string;
  c: boolean;
}

type IndexedAccess = Example["a"]; // number
type IndexedAccess2 = Example["a" | "b"]; // number | string
type IndexedAccess3 = Example[keyof Example]; // number | string | boolean

type ExAlias = "b" | "c";
type IndexedAccess4 = Example[ExAlias]; // string | boolean
```

### 💡 또한, 배열의 요소 타입을 조회하기 위해 인덱스드 엑세스 타입을 사용하는 경우가 있다.

### number로 인덱싱하여 배열 요소를 얻은 후 typeof 연산자를 부텨주면 해당 배열 요소의 타입을 가져올 수 있다.

```tsx
const ProductionList = [
  { type: "product", name: "chicken" },
  { type: "product", name: "pizza" },
  { type: "card", name: "cheer-up" },
]

type ElementOf<T> = typeof T[number];
type PromotionItemType = ElementOf<ProductionList>;
```

## 5) 맵드 타입(Mapped Types)

> 자바스크립트의 map 메서드를 생각했을 떄 배열 A를 기반으로 새로운 배열 B를 만들어내는 배열 메서드이다. 맵드 타입은 이와 마찬가지로 다른 타입을 기반으로 한 타입을 선언할 때 사용하는 문법인데, 인덱스 시그니처 문법을 사용해 반복적인 타입 선언을 효과적으로 줄일 수 있다.
> 

```tsx
type Example = {
  a: number;
  b: string;
  c: boolean;
};

type Subset<T> = {
  [K in keyof T] = T[K];
};

const aExample: Subset<Example> = { a: 3 };
const bExample: Subset<Example> = { b: "hello" };
const cExample: Subset<Example> = { a: 4, c: true };
```

- 맵드 타입 실 사용 예시바텀시트 코드를 생성할 때 각각 resolver, isOpened 등의 상태를 관리하는 스토어가 필요한데 이 스토어의 타입(BottomSheetStore)을 선언해줘야 한다.이때 BottomSheetMap에 존재하는 모든 키에 대해 일일이 스토어를 만들어줄 수도 있지만 불필요한 반복이 발생하게 된다. 이럴 때는 인덱스드 시그니처 문법을 사용해서 BottomSheetMap을 기반으로 각 키에 해당하는 스토어를 선언할 수 있다.

```tsx
const BottomSheetMap = {
  RECENT_CONTACTS: RecentContactBottomSheet,
  CARD_SELECT: CardSelectBottomSheet,
  SOFT_FILTER: SoftFilterBottomSheet,
  PRODUCT_SELECT: ProductSelectBottomSheet,
  REPLAY_CARD_SELECT: ReplayCardSelectBottomSheet,
  RECEND: RecendBottomSheet,
  STICKER: StickerBottomSheet,
  BASE: null,
};

export type BOTTOM_SHEET_ID = keyof typeof BottomSheetMap;

// 불필요한 반복이 발생된다.
type BottomSheetStore = {
  RECENT_CONTACTS: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
  CARD_SELECT: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
  SOFT_FILTER: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
  // ...
};

// Mapped Types를 통해 효율적으로 타입 선언을 할 수 있다.
type BottomSheetStore = {
  [index in BOTTOM_SHEET_ID]: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
};
```

+) 덧붙여 맵드 타입에서는 as 키워드를 사용하여 키를 재지정할 수 있다.

```tsx
type BottomSheetStore = {
  [index in BOTTOM_SHEET_ID as `${index}_BOTTOM_SHEET`]: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
};
```

## 6) 템플릿 리터럴 타입(Temeplate Literal Types)

> 자바스크립트의 템플릿 리터럴 문자를 사용하여 문자열 리터럴 타입을 선언할 수 있는 문법이다.
> 

```tsx
type Stage =
| "init"
| "select-image"
| "edit-image"
| "decorate-card"
| "capture-image";

type StageName = `${Stage}-stage`;
// 'init-stage' | 'select-image-stage' ...
```

- > Stage 타입의 모든 유니온 멤버 뒤에 -stage를 붙여서 새로운 유니온 타입을 만들었다.

## 7) 제네릭(Generic)

> 제네릭은 C나 자바 같은 정적 언어에서 다양한 타입 간에 재사용성을 높이기 위해 사용하는 문법이다. 타입스크립트도 정적 타입을 가지는 언어이기 때문에 제네릭 문법을 지원하고 있다.
> 

> 제네릭을 한마디로 말하자면 일반화된 데이터 타입 이라고 할 수 있는데 이를 좀 더 타입스크립트 개념으로 풀어본다면 함수, 타입 클래스 등에서 내부적으로 사용할 타입을 미리 정해두지 않고 타입 변수를 사용해서 해당 위치를 비워 둔 다음에, 실제로 그 값을 사용할 때 외부에서 타입 변수 자리에 타입을 지정하여 사용하는 방식이다.
> 

> 이렇게 하면 함수, 타입, 클래스 등 여러 타입에 대해 하나하나 따로 정의하지 않아도 되기 때문에 재사용성이 크게 향상된다. 타입 변수는 일반적으로 <T> 와 같이 꺽쇠괄호 내부에 정의 되며, 사용할 때는 함수에 매개변수를 넣는 것과 유사하게 원하는 타입을 넣어주면 된다.
> 
> 
> 보통 타입 변수명으로 T(Type), E(Element), K(Key), V(Value) 등 한 글자로 된 이름을 많이 사용한다.
> 

```tsx
type ExampleArrayType<T> = T[];

const array1: ExampleArrayType<string> = ["치킨", "피자", "우동"];
```

### 💡 앞서, 제네릭이 일반화된 데이터 타입을 말한다고 했는데, 이 표현만 보면 any의 쓰임과 혼동할 수 있을 것이다.

### 하지만, 둘은 명확히 다르다!

> any를 사용할 때는 타입 검사를 하지 않고 모든 타입이 허용되는 타입으로 취급된다. 반면 제네릭은 any처럼 아무 타입이나 무분별하게 받는 게 아니라, 배열 생성 시점에 원하는 타입으로 특정할 수 있다.
> 

### 🌟 다시 말해 제네릭을 사용하면 배열 요소가 전부 동일한 타입이라고 보장할 수 있다!

```tsx
type ExampleArrayType2 = any[];

const array2: ExampleArrayType2 = [
  "치킨",
  {
    id: 0,
    name: "치킨",
    price: 20000,
    quantity: 1,
  },
  99,
  true,
];
```

> 참고로 제네릭 함수를 호출할 때 반드시 꺽쇠괄호 (<>) 안에 타입을 명시해야 하는 것은 아니다. 타입을 명시하는 부분을 생략하면 컴파일러가 인수를 보고 타입을 추론해준다. 따라서 타입 추론이 가능한 경우에는 타입 명시 생략이 가능하다.
> 

```tsx
function exampleFunc<T>(arg: T): T[] {
  return new Array(3).fill(arg);
};

exampleFunc('hello'); // T는 string으로 추론된다
```

> 또한 특정 요소 타입을 알 수 없을 때는 제네릭 타입에 기본값을 추가할 수 있다.
> 

```tsx
interface SubmitEvent<T = HTMLElement> extends SyntheticEvent<T> { submitter: T; }
```

> 제네릭은 함수나 클래스 등 내부에서 제네릭을 사용할 떄 어떤 타입이든 될 수 있다는 개념을 잊어선 안된다. 특정한 타입에서만 존재하는 멤버를 참조하려고 하면 에러가 발생한다.
> 

```tsx
function exampleFunc2<T>(arg: T): number {
  return arg.length; // 에러 발생: Property 'length' does not exist on type 'T'
};

// 배열에만 존재하는 length 속성을 제네릭에서 참조하려고 하면 당연히 에러가 발생한다. 컴파일러는 어떤 타입이 제네릭에 전달될지 알 수 없기 때문에 모든 타입이 length 속성을 사용할 수는 없다고 알려주는 것이다.
```

### 그럼 이럴 때는 어떻게 해결할까?

> 제네릭 꺽쇠괄호 내부에 'length 속성을 가진 타입만 받는다'라는 제약을 걸어줌으로써 length 속성을 사용할 수 있게끔 만들 수 있다.
> 

```tsx
interface TypeWithLength {
  length: number;
}

function exampleFunc3<T extends TypeWithLength>(arg: T): number {
  return arg.length;
}
```

### 💡 제네릭을 사용할 때 주의할 점!

> 파일 확장자가 tsx일 때 화살표 함수에 제네릭을 사용하면 에러가 발생한다. tsx는 타입스크립트 + JSX 이므로 제네릭의 꺽쇠괄호와 태그의 꺽쇠괄호를 혼동하여 문제가 생기는 것이다.
> 

> 이러한 상황을 피하기 위해서는 제네릭 부분에 extends 키워드를 사용하여 컴파일러에게 특정 타입의 하위 타입만 올 수 있음을 확실히 알려주면 된다. 보통 제네릭을 사용할 때는 function 키워드로 선언하는 경우가 많다.
> 

```tsx
// 에러 발생: JSX Element 'T' has no corresponding closing tag
const arrowExampleFunc = <T>(arg: T): T[] => {
  return new Array(3).fill(arg);
};

// 에러 발생 x
const arrowExampleFunc2 = <T extends {}>(arg: T): T[] => {
  return new Array(3).fill(arg);
};
```

## 🧐 but,

> extends {} 를 사용했을 경우 원시 타입을 포함한(string, number 등) 거의 모든 값을 제네릭으로 받을 수 있으나, null과 undefined 는 제네릭으로 사용할 수 없다.
> 
> 
> 타입스크립트의 모든 타입을 포함하는 탑 타입인 unknown 을 사용하면 `<T>(arg: T): T` 와 동일하게 사용할 수 있다.
> 
> 출처: [[TypeScript] tsx 화살표 함수에 제네릭 사용하기](https://velog.io/@kimbangul/TypeScript-tsx-%ED%99%94%EC%82%B4%ED%91%9C-%ED%95%A8%EC%88%98%EC%97%90-%EC%A0%9C%EB%84%A4%EB%A6%AD-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
> 

```tsx
const arrowExampleFunc3 = <T extends unknown>(arg: T): T[] => {
  return new Array(3).fill(arg);
}

const returnResult = arrowExampleFunc3<string | undefined | null>(null);
```

> +) 타입 매개변수를 두 개 이상 사용할 때는 컴파일러가 <>를 제네릭으로 정상적으로 인식한다. 타입 매개변수가 하나밖에 필요없을 때에도 뒤에 , 를 붙여 주면 컴파일러가 정상적으로 제네릭을 인식한다.
> 

```tsx
const arrowExampleFunc4 = <T, >(arg: T): T[] => {
  return new Array(3).fill(arg);
}

const returnResult = arrowExampleFunc4<string | undefined | null>(null);
```

# 3.3 제네릭 사용법

## 1) 함수의 제네릭

> 어떤 함수의 매개변수나 반환 값에 다양한 타입을 넣고 싶을 때 제네릭을 사용할 수 있다.
> 

```tsx
function ReadOnlyRepository<T>(target: ObjectType<T> | EntitySchema<T> | string): Repository<T> {
  return getConnection("ro").getRepository(target);
};
```

## 2) 호출 시그니처의 제네릭

> 호출 시그니처는 타입스크립트의 함수 타입 문법으로 함수의 매개변수와 반환 타입을 미리 선언하는 것을 말한다. 호출 시그니처를 사용함으로써 개발자는 함수 호출 시 필요한 타입을 별도로 지정할 수 있게 된다. 제네릭을 어디에 위치시키는지에 따라 타입의 범위와 제네릭 타입을 언제 구체 타입으로 한정할지를 결정할 수 있다.
> 

```tsx
interface useSelectPaginationProps<T> {
  categoryAtom: RecoilState<number>;
  fileAtom: RecoilState<string[]>;
  sortAtom: RecoilState<SortType>;
  fetcherFunc: (props: CommonListRequest) => Promise<DefaultResponse<ContentListResponse<T>>>;
}
```

## 3. 제네릭 클래스

> 외부에서 입력된 타입을 클래스 내부에 적용할 수 있는 클래스이다.
> 

## 4. 제한된 제네릭

> 제한된 제네릭은 타입 매개변수에 대한 제약 조건을 설정하는 기능을 말한다. 타입 매개변수 T 타입을 제약하는 방법을 알아보자.
> 

예를 들어, string 타입으로 제약하려면 타입 매개변수는 특정 타입을 상속(extends)해야 한다.

```tsx
type ErrorRecord<Key extends string> = Exclude<Key, ErrorCodeType> extends never
? Partial<Record<Key, boolean>>
| never;
```

## 5. 확장된 제네릭

> 제네릭 타입은 여러 타입을 상속받을 수 있으며 타입 매개변수를 여러 개 둘 수도 있다.
> 

```tsx
<Key extends string>
```

> 타입을 이런 식으로 제약해버리면 제네릭의 유연성을 잃어버린다. 제네릭의 유연성을 잃지 않으면서 타입을 제약해야 할 때는 타입 매개변수에 유니온 타입을 상속해서 선언하면 된다.
> 

```tsx
<Key extends string | number>
```

## 6. 제네릭 예시

> 제네릭의 장점은 다양한 타입을 받게 함으로써 코드를 효율적으로 재사용할 수 있는 것이다. 실제 현업에서 API 응답 값의 타입을 지정할 때 많이 사용한다.
> 

```tsx
export interface MobileApiResponse<Data> {
  data: Data;
  statusCode: string;
  statusMessage: string;
}
```

위 코드를 살펴보면 API 응답 값에 따라 달라지는 data를 제네릭 타입 Data로 선언하고 있다. 이렇게 만든 MobileApiResponse는 실제 API 응답 값의 타입을 지정할 때 아래와 같이 사용된다.

```tsx
export const fetchPriceInfo = (): Promise<MobileApiResponse<PriceInfo>> => {
  const priceUrl = "http: ~~~"; // url 주소

  return request({
    method: "GET",
    url: priceUrl,
  });
};

export const fetchOrderInfo = (): Promise<MobileApiResponse<Order>> => {
  const orderUrl = "http: ~~~"; // url 주소

  return request({
    method: "GET",
    url: orderUrl,
  });
};
```

위와 같이 다양한 API 응답 값의 타입에 MobileApiResponse을 활용해서 코드를 효율적으로 재사용할 수 있다.

이런 식으로 제네릭을 필요한 곳에 사용하면 가독성을 높이고 코드를 효율적으로 작성할 수 있다. 하지만 굳이 필요하지 않은 곳에 제네릭을 사용하면 오히려 독이 되어 코드를 복잡하게 만든다.

## 제네릭을 굳이 사용하지 않아도 되는 타입

> 제네릭이 필요하지 않을 때도 사용하면 코드 길이만 늘어나고 가독성을 해칠 수 있다
> 

```tsx
type GType<T> = T;
type RequirementType = 'USE' | 'UN_USE' | 'NON_SELECT';
interface Order {
  getRequirement(): GType<RequirementType>;
}
```

GType이 다른 곳에서는 사용되지 않고 getRequirement 함수의 반환 값 타입으로만 사용되고 있다고 가정해보자.

GType이라는 이름이 현재 사용되고 있는 목적의 의미를 정확히 담고 있지도 않을뿐더러 굳이 제네릭을 사용하지 않고 매개변수를 그대로 선언하는 것과 같은 기능을 하고 있다.

```tsx
type RequirementType = 'USE' | 'UN_USE' | 'NON_SELECT';
interface Order {
  getRequirement(): RequirementType;
}
```