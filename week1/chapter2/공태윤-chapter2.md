# 02. 타입
### ✨ 키워드

- 정적 타입 / 동적 타입
- 강타입 / 약타입
- 타입 애너테이션
- 구조적 타이핑
- 구조적 서브타이핑
- 구조적 타이핑 / 덕 타이핑
- 자바스크립트의 수퍼셋 타입스크립트
- 점진적 타입
- 값과 타입
- typeof
- 원시 타입
- 객체 타입
- type / interface

## 2.1 타입

### 자료형으로서의 타입

- 변수: 값을 저장할 수 있는 공간 (메모리)
    
    ⇒ 변수를 읽기 위해서는 메모리에서 얼마만큼의 메모리를 차지하고 있는지를 알아야 함
    
- 타입을 통해 메모리 공간을 알 수 있으며, 자바스크립트는 7개의 타입을 가지고 있음
    - undefined
    - null
    - boolean
    - string
    - symbol
    - numeric (number, bigInt)
    - object
- 위와 같은 타입을 통해 값의 종류를 명시하고, 메모리를 효율적으로 사용할 수 있음.

### 집합으로서의 타입

```jsx
const var1: string = 12
const var2: boolean = true

function func(a: string | number) {
	// do something
}

func(var1) // OK
func(var2) // ERROR
```

⇒ 수학의 집합과 같이 var1은 a의 범위 안에 들어있기 때문에 OK, var2는 아니기 때문에 Error

### 정적 타입과 동적 타입

- 정적 타입
    - 컴파일 타임에 잡을 수 있어 예상치 못한 오류를 빠르게 찾을 수 있음
    - C, Java, TypeScript
- 동적 타입
    - 타입이 런타임에 정해지기 때문에 언제 오류가 생길지 모름
    - Python, JavaScript

### 강타입과 약타입

- 반드시 타입을 명시해줄 필요가 없는 언어라도 모든 언어는 타입이 존재한다

```jsx
const a = +'12' // (암묵적 타입 변환)
```

⇒ 런타임에 타입이 자동으로 변경되는 것

✅ c++

```cpp
#include <iostream>
using namespace std;
int main() {
	cout << '2' - 1 << endl;// '2'를 아스키코드 50으로 변환하고 1을 빼서 49를 출력함
}
```

✅ java

```java
class Main {
	public static void main(String[] args) {
		System.out.println('2' - 1) // C++과 마찬가지로 아스키 코드로 변환해서 실행
	}
}
```

⛔️ python

```python
print('2' - 1) # 타입 에러
```

✅ javascript

```jsx
console.log('2' - 1) // 1
```

⛔️ typescript

```jsx
console.log('2' - 1)

// The left-hand side of an arithmetic operation must be of type 
// 'any', 'number', 'bigint' or an enum type.
```

강타입 : 암묵적 타입 변환이 불가함

- 파이썬
- 루비
- 타입스크립트

약타입 : 암묵적 타입 변환 가능

- C++
- 자바
- 자바스크립트

> 타입 에러를 발생시키는 것이 더 안전
> 

⇒ 결론적으로 자바스크립트는 약타입, 타입스크립트는 강타입이다.

### 컴파일 방식

다른 컴파일러와 다르게 고수준 언어 → 바이너리 코드로 변환하는 것이 아닌,

타입스크립트 - (컴파일) → 자바스크립트

⇒ 타입스크립트는 템플릿 언어, 확장 언어로 해석하는 견해도 있음

## 2.2 타입 시스템

### 타입 애너테이션

```jsx
const a: number = 10 (O)
const b = 10 (O)
```

`: type`을 붙여 타입 애너테이션을 사용할 수 있으며, 제거해도 정상 동작한다.

<aside>
🤔

왜 타입 애너테이션을 제거해도 정상 동작할까 ?

⇒ 타입스크립트는 점진적 타입 확인을 하기 때문이다. (자세한건 뒤에서)

</aside>

### 구조적 타이핑

> 타입스크립트는 구조로 타입을 결정한다.
> 

반대로, 이름으로 타입을 구분하는 것은 `명목적으로 구체화한 타입 시스템`이라고 부른다.

```tsx
interface A {
	val: number
}

interface B {
	val: number
}

let a: A = { val: 2 }
let b: B = { val: 3 }

a = b (OK)
```

⇒ A와 B는 같은 구조를 가졌기 때문에 이름은 다르지만, 같은 타입이다.

### 구조적 서브타이핑

> 타입스크립트의 타입은 값의 집합으로 생각할 수 있다.
> 

```tsx
// ⛔️
const numberOrString: number | string = 2
const str: string = numberOrString

// ✅
const str: string = 'abc'
const numberOrString: string | number = str
```

⚠️ 위의 예시는 구조적 서브타이핑의 예시는 아님.

```tsx
interface Pet {
	name: string
}

interface Cat {
	name: string
	age: number
}

let pet: Pet
let cat: Cat = { name: 'meow', age: 12 }

pet = cat // ✅
```

집합의 관점으로 보자면 Pet이 Cat의 하위 집합이다.

따라서 Pet 변수는 Cat 변수를 할당받을 수 있다. 다르게 말하면 Cat 변수는 Pet 변수에게 할당할 수 있다.

Pet ⊂ Cat ⇒ Cat → Pet 할당 가능

```tsx
interface Pet {
	name: string
}

let cat = { name: 'zag', age: 2 }

function greet(pet: Pet) {
	console.log('hello', pet.name)
}

greet(cat) // ✅
```

<aside>
🤔

타입스크립트의 구조적 타이핑 (구조적 서브타이핑)의 위험

이러한 타입스크립트의 동작은 자바스크립트와의 호환성에는 유리하지만, 의도치않은 위험이 있을 수 있다고 생각합니다.

[TS Playground - An online editor for exploring TypeScript and JavaScript](https://www.typescriptlang.org/play/?#code/C4TwDgpgBAShDGwCGA7A5gG2gXigbwCgpioAPALihQFcBbAIwgCcAaIkkSmh5tgXwIF4AexQBnYFDF0AghgxRcACiYJk6LJTiJUmCAEpFAPnztiWSdNqKoABjNQAZsKZQlI8ZIBuSDNWjCjlAA8vQAVmoAdD5+EGIqarpY+oaEJOlSdFAA1Lgx-g4CDqrA1EwombQERR4SUKqIAIw2eGSUjbYsUJxQAEy2UAK1wliRGMJoSlZyGAlNKVAA9ItQAMz2QqJ1Db0tbVAdXT39XShItBCUAOS0IPVqV4Ob4iMQYxNTsvJzvQvLUAAida3e6IAEEIA)

```tsx
type Rectangle = {
    x: number,
    y: number,
}

const sumAll = (rectangle: Rectangle) => {
    let sum = 0
    for (const value of Object.values(rectangle)) {
        sum += value
    }

    return sum
}

const rect1 = { x: 10, y: 20 }
console.log(sumAll(rect1)) // 30

const rec2 = { x: 10, y: 20, name: 'my rect' }
console.log(sumAll(rec2)) // "30my rect"
```

- 타입가드 사용하기

[TS Playground - An online editor for exploring TypeScript and JavaScript](https://www.typescriptlang.org/play/?#code/C4TwDgpgBAShDGwCGA7A5gG2gXigbwCgpioAPALihQFcBbAIwgCcAaIkkSmh5tgXwIF4AexQBnYFACWYuIlSYcUABTD6AK0pzk6LAEpKa9dLGwEOxVGwA+fO2JMIwakxQr7JYgCJSX6W6MoADIgj08vED8pAI1g0M9PUEhhADMoIwA6UitsXC9uRiY-ELCSJIhU9I0MkBy8guZi+ITiAHkNcwyAawgQMVUNPQysdGAACzqoACYPPQIBEXFJMToAQQwMKxVHeV0ILXMFfStbQk8pNOUAQhltI4hlHYt9PTsW4nGmYQB3KghfgCiTC+TGUAHJABnjgG6uwA4LVBAB7jgAwhwCh44AJpsAJ00ZMFzTwCTweLDLOhbAAMHhSwiYKkWEigADckBhqNBKu11J0GUyIP0nvc9K8zu8VrQoABqXCc5keAQeRzOVxQYXzQTAJi1QXEGmSJ4ARi2eDIlB1JJYUE40xJUDxJBpwiww2EaGUwvWGEe5h1-KgAHpvVAAMxkjxaqA7Kb6w1QY2m81TE1UJC0fZQMG0WpPMFW4OiMR2iAOp0ujbu+BTL2+qBeQNp0PmLzzKDwJDAeATZTMEEC7PiPMF9vAylzPhAA)

- 타입선언을 통해 막기

[TS Playground - An online editor for exploring TypeScript and JavaScript](https://www.typescriptlang.org/play/?#code/C4TwDgpgBAShDGwCGA7A5gG2gXigbwCgpioAPALihQFcBbAIwgCcAaIkkSmh5tgXwIF4AexQBnYFDF0AghgxRcACiYJk6LJTiJUmCAEpFAPnztiWSdNqKoABjNQAZsKZQlI8ZIBuSDNWjCjlAA8vQAVmoAdD5+EGIqarpY+oaEJOlSdFAA1Lgx-g4CDqrA1EwombQERR4SUKqIAIxaiRo4+GSUjbYsUJxQAEy2UAK1wliRGMJoSlZyGAlNKVAA9CtQAMz2QqJ1DQMtOm02eJ1Q3b39Q70oSLQQlADktCD1ao8jO+LjEJPTs7J5IsBss1lBmEwXAQgA)

 ⭐️  다양한 방법이 있을거 같은데 공유해보면 좋을거 같아요!

</aside>

### 자바스크립트를 닮은 타입스크립트

> 자바스크립트는 덕 타이핑을 타입스크립트는 구조적 타이핑을 제공한다. 이는 타입스크립트는 자바스크립트의 슈퍼셋으로 자바스크립트와의 호환성의 이유가 존재한다.
> 

덕 타이핑과 구조적 타이핑은 유사하지만, 타임 검사 시점의 차이가 존재한다.

런타임 → 덕 타이핑 (동적)

컴파일타임 → 구조적 타이핑 (정적)

<aside>
💡

덕 타이핑 🦆

어떤 타입에 부합하는 변수와 메서드를 가질 경우 해당 타입에 속하는 것으로 간주하는 방식

“만약 어떤 새가 오리처럼 걷고, 헤엄치며 꽥꽥거리는 소리를 낸다면 나는 그 새를 오리라고 부를 것이다”

</aside>

### 타입스크립트의 점진적 타입 확인

> 타입스크립트는 점진적으로 타입을 확인하는 언어
> 

```tsx
function add(x, y) {
	return x + y
}
```

암묵적으로 x, y, 그리고 return 값들이 any로 타입 변환이 일어난다.

tsconfig.ts

```tsx
{
	noImplicitAny: true
}
```

⇒ 타입 애너테이션이 없을 때 변수가 any 타입으로 추론되는 것을 막을 수 있어, 옵션을 설정하는 것이 좋다.

### 값과 타입

```tsx
// 타입 공간 네임스페이스 => 컴파일 이후에 제거됨
type a = number
// 값 공간 네임스페이스
const a: a = 3 // OK
```

⇒ 타입스크립트는 자바스크립트의 슈퍼셋으로 type은 자바스크립트 런타임에 제거되기 때문에 값 공간과 타입 공간은 서로 충돌하지 않는다.

- 값
    - 할당 연산자 (`=`)
- 타입
    - 타입 선언 (`:`)
    - 단언 문 (`as`)

### 값과 타입 동시에 사용하는 enum과 class

클래스와 enum은 값과 타입이 동시에 존재한다.

```tsx
class Person {
	constructor(name) {
		this.name = name
	}
}

const me: Person = new Person('taeyun')
```

⇒ 타입스크립트에서 class는 타입이자 값(함수)이다.

: Person → 타입

Person('taeyun') → 값

```tsx
enum Direction {
  Up = 1,
  Down,
  Left,
  Right,
}

// 타입으로도 그리고 값으로도 사용할 수 있다.
const currentDirection: Direction = Direction.Up;
console.log(currentDirection); // 1

function move(direction: Direction) {
  switch (direction) {
    case Direction.Up:
      console.log('Moving Up');
      break;
    case Direction.Down:
      console.log('Moving Down');
      break;
    case Direction.Left:
      console.log('Moving Left');
      break;
    case Direction.Right:
      console.log('Moving Right');
      break;
    default:
      console.log('Invalid direction');
  }
}

move(Direction.Left); // Moving Left
```

⇒ enum도 마찬가지로 타입과 값 네임스페이스 모두에 등록되기 때문이다.

<aside>
ℹ️

enum은 값으로도 그리고 타입으로도 사용되기 때문에 어색함을 줄 수 있어 사용이 지양하는 사람도 존재한다. 그리고 만약 enum을 사용한다면 const enum을 사용하는 것이 좋다. 번들링 툴에 의해 트리 쉐이킹(사용하지 않는 코드를 삭제하는 방식)이 되기 때문이다.

</aside>

| 키워드 | 값 | 타입 |
| --- | --- | --- |
| class | Y | Y |
| const, let, var | Y | N |
| enum | Y | Y |
| function | Y | N |
| interface | N | Y |
| type | N | Y |
| namespace | Y | N |

### 타입을 확인하는 방법

타입스크립트에서 typeof 연산자는 값일 때와 타입일 때의 약할이 다르다.

```tsx
type Person = {
	first; string
	last: string
}

const person: Person = { first: 'zig', last: 'song' }
const email = (options: { persong: Person, subject: string, body: string }) => {}

// 값
const v1 = typeof person // 'object'
const v2 = typeof email // 'function'

// 타입
type T1 = typeof person // Person
type T2 = typeof email // (options) => void
```

```tsx
const person = {
  name: "John",
  age: 30,
}

// person 객체의 타입을 추론하여 NewPerson 타입 생성
type NewPerson = typeof person

const anotherPerson: NewPerson = {
  name: "Jane",
  age: 25,
}
```

⇒ 타입을 추론하는 데 도움을 줄 수 있다.

```tsx
class Developer {
	name: string
	
	constructor(name) {
		this.name = name
	}
}

const V = typeof Developer // funtion (값)
type T = typeof Developer // Developer 생성자 함수
```

### 타입 단언

```tsx
const txt: unknown

function proxy(text: string) {
	return 'hello' + text
}

proxy(txt as string)
```

txt는 unknown이기 때문에 proxy 함수의 인자로 들어갈 수 없지만, `as string`으로 타입을 단언함으로써 인자로 사용될 수 있다.

## 2.3 원시 타입

원시 타입은 number와 같이 소문자로 쓴다. Number와 같이 사용하는 것은 원시 래퍼 객체(자바스크립트 원시 래퍼 객체)이다. 따라서, 원시 타입과 원시 래퍼 객체를 구분하여 사용해야 한다.

- boolean
- undefined
- null
- number
- bigInt
- string
- symbol

## 2.4 객체 타입

7개의 원시 타입이 아닌 값은 모두 객체 타입으로 분류할 수 있다.

- object
    - 지양한다. any 타입과 유사하게 값을 유동적으로 할당할 수 있어 정적 타이핑의 의미가 크기 퇴색되기 때문이다
- `{}`
    
    ```tsx
    const print = ({title, subTitle}: {title: string, subTitle: string) => {
    	return console.log(`${title}: ${subTitle}`)
    }
    
    print({title: 'hello', subTitle: 'world'})
    ```
    
    ```tsx
    const obj = {}
    obj.name = 'kim' // ⛔️
    ```
    
    - 속성을 지정할 수 없음
- array
    - 자바스크립트와 달리 타입스크립트는 배열에 하나의 타입값만 가질 수 있다.
    
    ```tsx
    const arr: string[]
    const arr2: Array<number>
    ```
    
- 튜플
    
    ```tsx
    type Tuple = [string, number]
    const tuple: Tuple = ['a', 1]
    ```
    
- type과 interface
    - object 타입은 실무에서 사용하지 않고, 타입스크립트만의 독자적인 type 또는 interface 키워드를 사용한다
    
    ```tsx
    type NoticePopupType = {
    	title: string
    	description: string
    }
    
    interface INoticePopup {
    	title: string
    	description: string
    }
    ```
    
- function
    
    ```tsx
    type add = (a: number, b: number) => number
    ```
    
    - 호출 시그니처를 통해 타입을 정의한다

### 우아한 형제들의 type과 interface

<aside>
🙋🏻‍♂️

개인적인 의견

팀의 컨벤션에 맞게 사용하는 것이 좋아 보임. 내가 정할 일이 있다면 말 그대로 인터페이스의 역할은 interface를, 그냥 타입 정의는 type을 사용할 거 같음

</aside>

- 공식문서대로 전역은 interface, 작은 범위 한정은 type
- 값에 대한 정의 → type 확장할 수 있는 basis 정의 또는 object 구성 → interface
- 선언 병합 → interface, computed value → type
- type만 사용
- 상속할 때는 interface 사용
- 유니온 타입 혹은 교차 타입 등 type 정의에서만 쓸 수 있는 기능에서는 type 사용
- 속성 공유는 interface
- …

### 📒 면접 질문 리스트

- 자바스크립트의 타입의 종류에 대해 설명해주세요.
- 정적 타입과 동적 타입의 차이가 무엇인지 설명해주세요.
- 강타입과 약타입의 차이점은 무엇인가요?
- 암묵적 타입 변환에 대해 설명하고, 예시를 들어주세요.
- 구조적 타이핑과 덕 타이핑의 차이점에 대해 설명해주세요.
- 타입 애너테이션을 제거해도 타입스크립트가 정상적으로 동작하나요? 그렇다면 왜 그렇게 동작하는지 설명해주세요.
- 타입스크립트의 typeof 연산자에 대해 설명해주세요.
- 타입스크립트의 type과 interface의 차이점과 면접자분은 어떤 방식을 선호하는지 말씀해주세요.