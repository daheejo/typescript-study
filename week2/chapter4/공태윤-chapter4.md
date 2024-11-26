# 04장. 타입 확장하기 좁히기
### ✨ 키워드

- extends
- 타입가드
- typeof
- instanceof
- is
- 식별할 수 있는 유니온
- Exhaustiveness Checking

## 4.1 타입 확장하기

### 타입 확장이란 ?

> 기존 타입을 사용해서 새로운 타입을 정의하는 것
> 

- [interface / type]으로 타입 정의 ⇒ [extends / 교차 타입 / 유니온 타입]으로 타입 좁히기 / 확장하기

### 타입 확장의 장점

```tsx
interface Item {
	name: string
	price: number
	imgUrl: string
}

// 장바구니
interface Cart extends Item {
	quantity: number
}
```

- 타입 확장은 중복된 코드를 줄일 수 있도록 돕는다.
- 또한, 명시적으로 코드를 작성할 수 있다.
- 확장성 → 유지 보수에 효율적이다.

example )

```tsx
type Item = {
	name: string
	price: number
	imgUrl: string
}

// 장바구니
type Cart = {
	quantity: number
} & Item
```

- type은 &을 통해 상속을 구현할 수 있다.

### 유니온 타입

```tsx
type UnionType = AType | BType
```

example )

```tsx
type AType = {
	a: number
	c: string
}

type BType = {
	b: number
	c: string
}

type UnionType = AType | BType

function f(param: UnionType) {
	param.a // ERROR
	param.c // OK (합집합)
}
```

유니온은 집합의 합집합으로 해석될 수 있다.

따라서, 사용할 때 공통 속성만 사용해야 한다.

즉, UnionType은 AType 또는 BType 일 뿐이지, AType과 동시에 BType은 아니다.

### 교차 타입

반대로 교차 타입은 집합의 교집합이다.

```tsx
type AType = {
	a: number
	c: string
}

type BType = {
	b: number
	c: string
}

type InterSectionType = AType & BType
```

- InterSectionType은 { c: string }이 된다.

### extends와 교차 타입

> 위에서 살펴봤듯이 extends를 이용하면 interface를 확장할 수 있으며, 교차 타입을 이용하면 type을 확장할 수 있다.
> 

둘의 차이점을 간단하게 알아보자

```tsx
interface A {
	a: number
}

interface B extends A {
	a: string
}
```

- Error를 발생시킨다.

```tsx
type A = {
	a: number
}

type B = {
	a: string
} & A
```

- a는 never가 된다.

<aside>
💡

중복 코드 (중복 타입 선언)이 발생할 때 무분별하게 속성을 추가하기 보다는 명시적으로 타입을 확장하여 사용하는 것이 좋다.

</aside>

## 4.2 타입 좁히기 - 타입 가드

> 변수 또는 표현식의 타입 범위를 더 작은 범위로 좁혀나가는 과정
> 

- 명시적인 타입 추론 가능
- 복잡한 타입을 작은 범위로 축소하여 타입 안전성을 높일 수 있다.

### 타입 가드에 따라 분기 처리하기

> 조건문과 타입 가드를 활용하여 변수나 표현식의 타입 범위를 좁혀 다양한 상황에 따라 다른 동작을 수행하는 것을 말한다.
> 

example )

```tsx
type arrayOrNumber = number[] | number

const add = (x: arrayOrNumber, y: arrayOrNumber): arrayOrNumber | null => {
    if (typeof x === 'number' && typeof y === 'number') {
        return x + y
    } else if (typeof x === 'object' && typeof y === 'object') {
        return [...x, ...y]
    }

    return null
}

console.log(add(1, 2)) // 3
console.log(add([1, 2, 3], [4, 5, 6])) // [1, 2, 3, 4, 5, 6]
console.log(add([1, 2, 3], 4)) // null
```

> 타입 가드는 ‘런타임’에 조건문을 사용하여 타입을 검사하고 타입 범위를 좁혀주는 기능을 말한다.
> 

```tsx
type UnionType = A | B
```

다음과 같은 타입이 존재하고, 함수의 인자가 UnionType이라고 생각해보자

어떤 타입이 들어오는지에 따라 분기하여 결과를 처리하고 싶다.

하지만, 타입스크립트는 컴파일 시 타입 정보는 모두 제거되어 런타임에 존재하지 않기 때문에 타입을 사용하여 존건을 만들 수 없다.

이때, 타입 가드를 사용하면 되는데, 타입 가드는 크게 자바스크립트 연산자를 사용한 타입 가드와 사용자 정의 타입 가드로 구분할 수 있다.

### 타입 가드

> typeof, instanceof, in과 같은 연산자를 사용 (런타임에 유효한 타입 가드를 만들기 위해서)
> 

1. typeof - 원시 타입을 추론할 때

```tsx
const convertType: (p: string | number) => string | number = (p) => {
	if (typeof p === 'string') {
		return Number(p)
	}
	
	return p.toString()
}
```

⇒ 원시 타입을 좁히는 용도로는 typeof 연산자를 사용한다.

1. instanceof - 인스턴스화된 객체 타입을 판별할 때

```tsx
class Dog {
  bark() {
    console.log("멍멍!")
  }
}

class Cat {
  meow() {
    console.log("야옹!")
  }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark()
  } else if (animal instanceof Cat) {
    animal.meow()
  }
}

```

⇒ 인스턴스화된 객체 타입을 판별하는 타입 가드로 사용할 수 있다.

instanceof는 프로토타입 체인에 생성자가 존재하는지를 검사해서 존재한다면 true 그렇지 않다면 false를 반환한다.

```tsx
const onKeyDown = (event: React.KeyboardEvent) => {
	if (event.target instanceof HTMLInputElement && event.key === 'Enter') {
		event.target.blur()
		onCTAButtonClick(event)
	}
}
```

1. in - 객체의 속성이 있는지 없는지

```tsx
class Dog {
  bark() {
    console.log("멍멍!")
  }
}

class Cat {
  meow() {
    console.log("야옹!")
  }
}

function makeSound(animal: Dog | Cat) {
  if ('bark' in animal) {
    animal.bark()
  } else {
    animal.meow()
  }
}

```

- 객체에 속성이 있는지 확인한 다음에 true 또는 false를 반환한다.

프로토타입 체인으로 접근할수 있는 속성이면 전부 true를 반환한다.

하지만, undefined가 할당되었다고 해도 속성은 존재하기 때문에 true를 반환한다.

1. is - 사용자 정의 타입 가드 만들어 활용

```tsx
A is B
```

⇒ 다음과 같은 형식으로 작성하면 된다. 이때, A는 매개변수, B는 타입이다.

⇒ 타입 명제라고 부름

```tsx
const isDestinationCode = (x: string): x is DestinationCode =>
	destinationCodeList.includes(x)
	
function exampel(x) {
	if(isDestinationCode(x)) {
		// do destinationcode true code
	} else {
		// do destinationcode false code
	}
}
```

⇒ is 연산자를 사용하면 분기처리에서 타입스크립트도 다음이 destinationcode임을 알 수 있다.

⇒ is 연산자를 사용하지 않으면, 에러를 일으킨다.

## 4.3 타입 좁히기 - 식별할 수 있는 유니온

```tsx
type TextError = {
	errorCode: number
	errorMsg: string
}

type ToastError = {
	errorCode: number
	errorMsg: string
	toastShowDuration: number // ToastError 만의 프로퍼티
}

type AlertError = {
	errorCode: number
	errorMsg: string
	onConfirm: () => void // AlertError 만의 프로퍼티
}
```

⇒ 다음과 같이 세 종류의 Error 타입이 존재한다고 하자.

```tsx
type ErrorType = TextError | ToastError | AlertError
const errorArray: ErrorType[] = [
	//... { errorCode: 100, errorMsg: '잘못된 접근' }
]
```

하지만

```tsx
{ errorCode: 300, errorMsg: '잘못된 접근', 
toastShowDuration: 500, onConfirm: () => { alert('alert') }}
```

⇒ 이건 세 에러 타입 중 어떤 타입일까 ?

덕 타이핑에 따른 예상치 못한 오류가 발생한다.

따라서 식별할 수 있는 유니온을 사용한다.

### 식별할 수 있는 유니온

> 서로 포함 관계를 가지지 않도록 정의하는 방법이다
> 

```tsx
type TextError = {
	errorType: 'TEXT' // 판별자(discriminant)라고 부름
	errorCode: number
	errorMsg: string
}

type ToastError = {
	errorType: 'TOAST'
	errorCode: number
	errorMsg: string
	toastShowDuration: number // ToastError 만의 프로퍼티
}

type AlertError = {
	errorType: 'ALERT'
	errorCode: number
	errorMsg: string
	onConfirm: () => void // AlertError 만의 프로퍼티
}
```

⇒ 다음과 같이 식별할 수 있는 유니온을 사용하면 타입스크립트에서 오류를 발생시킨다.

다음과 같이 errorType을 유니온 식별자라고 부르는데, 유니온 식별자는 유닛 타입으로 선언되어야 정상적으로 동작한다.

유닛 타입

> 하나의 정확한 값을 가지는 타입
> 
- null
- undefined
- true
- 1

## 4.4 Exhaustiveness Checking으로 정확한 타입 분기 유지하기

Exhaustiveness는 사전적으로 철저함, 완전함을 의미한다.

모든 케이스에 대해 철저하게 타입을 검사하는 것을 말하며 타입 좁히기에 사용되는 프로그래밍 패러다임이다.

다음과 같은 상품권이 있다고 생각해보자.

```tsx
type ProductPrice = '10000' | '20000'
```

근데 ‘30000’원 상품권이 생겼다고 생각해보자.

```tsx
type ProductPrice = '10000' | '20000' | '30000'
```

```tsx
const getProductPrice = (price: ProductPrice): number => {
	if (price === '10000') return 10000
	if (price === '20000') return 20000
	
	return 0
}
```

⇒ 다음과 같이 사용해도 문제가 발생하진 않을 것이다. 하지만, ‘30000’의 분기처리를 해주지 않았다.

이처럼 모든 타입에 대한 검사를 강제하고 싶을 때 다음과 같이 코드를 작성하면 된다.

```tsx
const getProductPrice = (price: ProductPrice): number => {
	if (price === '10000') return 10000
	if (price === '20000') return 20000
	else {
		// 이미 분기처리가 다 되었다면 price는 어떠한 경우도 발생할 수 없음 => never
		exhaustiveCheck(price) // error
		return 0
	}
}

const exhaustiveCheck = (param: never) => {
	throw new Error('Type Error')
}
```

- param은 never를 인자로 받는다. 분기 처리에서 모든 타입에 대한 처리가 이루어졌다면, price는 never일 것이다. 하지만, 모든 분기처리가 되지 않았다면, never가 아닌 ‘30000’이 있을 것이다.
    
    ⇒ 타입스크립트 컴파일 타임에 에러를 일으킬 수 있다.
    

<aside>
💡

하지만 다음과 같이 프로덕션 코드에 어서션을 추가하면, 번들 사이즈에 영향이 가지 않을까 ? 걱정이 될 수 있지만, 어서션 자체가 성능에 크기 영향을 주지 않으니 DX 측면에서 좋을 수 있다.

</aside>

### 🚀 느낀점

- 프로젝트를 하면 Exhaustiveness Checking 함수를 util 함수로 만들어서 타입 가드 시 좀 더 안전하게 분기 처리를 하면 좋을거 같다.
- 타입가드를 알지만 타입가드를 적극적으로 사용한 경험이 부족한거 같다. 어떻게 하면 타입가드를 더욱 적극적으로 사용할 수 있을지 궁금하고, 다른 사람들의 타입가드 개발 경험이 궁금하다.

### 📒 면접 질문 리스트

- extends와 교차 타입의 차이점에 대해 알려주세요.
- 어떤 상황일 때 타입 가드를 사용하면 좋을까요 ?
- 식별할 수 있는 유니온에 대해 설명해주세요.
- Exhaustiveness Checking에 대해 설명해주세요.