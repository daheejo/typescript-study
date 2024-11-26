# 03. 고급 타입
### ✨ 키워드

- any
- unknown
- never
- void
- Array
- Tuple
- enum
- 유니온 타입
- 교차 타입
- 인덱스 시그니처
- 인덱스드 엑세스 타입
- 맵드 타입
- 템플릿 리터럴 타입
- 제네릭

## 3.1 타입스크립트만의 독자적 타입 시스템

타입스크립트의 타입 시스템은 모두 자바스크립트에서 기인했지만, `any`와 같이 타입스크립트에만 존재하는 독자적인 타입 시스템도 존재한다.

![image.png](./images/typescript_type.png)

- any
    - 자바스크립트의 기본 사용 방식, 타입을 명시하지 않은 것과 동일한 효과를 나타낸다
    - 하지만, 타입스크립트의 정적 타이핑을 무색하게 만들 수 있기 때문에 지양해야 한다.
    - 어쩔 수 없이 any를 사용해야 할 경우들이 존재한다.
        - 임의의 값을 정해야 한다. 변경될 가능성이 높기 때문에 타입을 세세하게 명시하기엔 개발 속도가 저하될 수 있다.
        - API 요청 및 응답 처리, 콜백 함수 전달, 타입 파악이 힘든 라이브러리 등과 같이 특정하기 힘들 때 any를 사용한다
- unknown
    - any와 유사하게 모든 타입의 값이 할당될 수 있다.
    
    ```tsx
    let unknownValue: unknown
    
    // unknown에 할당은 가능하지만
    unknownValue = 10
    unknownValue = 'str'
    
    // unknown을 할당하는 것은 any만 가능
    let anyValue: any = unknownValue (O)
    let strValue: string = unknownValue (X)
    let numValue: number = unknownValue (X)
    ```
    
    ⇒ 값을 가져오거나 내부 속성에 접근할 수 없음 ⇒ 엄격한 타입 검사 강제
    
    따라서, 데이터 구조를 파악하기 힘들 때 any → known을 권장
    
- void
    
    ```tsx
    function print(a: number): void {
    	console.log(a)
    }
    ```
    
    - 아무것도 리턴하지 않을 때 void를 사용한다
- never
    - 함수에서 값을 `반환할 수 없는 타입`
        - 에러를 던지는 경우
        - 무한히 함수가 실행되는 경우
- Array
    - `Object.prototype.toString.call(…)` 연산자를 사용하면 자바스크립트에서도 Array 키워드를 확인할 수 있다.
        - typeof와 다르게 객체 인스턴스까지 확인해서 알 수 있음
    
    ```tsx
    const arr: Array<number> = [1, 2, 3]
    const arr2: number[] = [1, 2, 3]
    ```
    
    - 타입을 강제시킴. 다른 타입은 들어갈 수 없음
    - 여러 타입에서는 유니온을 사용하면 됨
    
    ```tsx
    const arr: Array<number | string> = [1, 'str']
    const arr2: number[] | string[] = [1, 'str']
    const arr3: (number | string)[] = [1, 'str']
    ```
    
- 튜플
    
    <aside>
    🙋🏻‍♂️
    
    개인적인 생각
    
    튜플을 공부하며 그동안 배열을 많이 사용했는데 많은 부분들을 튜플로 바꿀 수 있을거 같다는 생각이 들었습니다. 추후에 배열보다는 튜플을 사용할 수 있을지 한 번 더 생각해보는 습관을 기르면 좋을거 같습니다.
    
    </aside>
    
    - 배열 타입은 배열의 길이까지는 제한할 수 없지만, 튜플은 길이 제한까지 가능하다
    
    ```tsx
    let tuple: [number, string] = [1, 'str']
    ```
    
    ```tsx
    const [value, setValue] = useState(true)
    ```
    
    ⇒ useState와 같이 잘 설계된 API는 튜플을 통해 유연성을 얻을 수 있다.
    
    ```tsx
    const { value: username, setValue: setUsername } = useState('')
    ```
    
    ⇒ 만약 객체로 한다면 유연성이 떨어질 수 있다.
    
    ```tsx
    const httpStatusFromPaths: [number, string, ...string[]] = [
    	400,
    	'Bad Request',
    	'/users/:id',
    	'/users/:userId',
    	'/users/:uuid',
    ]
    ```
    
    ⇒ 스프레드 연산자를 이용하여 명확한 타입으로 선언하고, 나머지는 배열처럼 사용할 수 있다.
    
    ```tsx
    const tuple: [number, string?] = [1]
    ```
    
    ⇒ 옵셔널 프로퍼티를 사용할 수도 있다.
    
- enum
    
    ```tsx
    enum ProgrammingLanguage = {
    	TypeScript, // 0
    	JavaScript, // 1
    	Java, // 2
    	C, // 3
    	CSharp, // 4
    	CPlusPlus, // 5
    	Go, // 6
    }
    ```
    
    - 타입으로도 사용할 수 있으며, 가독성을 높이는 데 사용할 수 있다.
    - 하지만, 역방향으로도 추론이 가능하다 `ProgrammingLanguage[0]` 따라서, `const enum`으로 선언하면 역방향으로의 접근을 막을 수 있다.
    
    ```tsx
    const enum NUMBER {
    	ONE = 1,
    	TWO = 2,
    }
    
    const myNumber: NUMBER = 100 ✅
    
    const enum STRING {
    	ONE = 'ONE',
    	TWO = 'TWO',
    }
    
    const myString: STRING = 'THREE' ⛔️
    ```
    
    ⇒ 따라서, 문자열 상수 방식으로 열거형을 사용하는 것이 더 안전하다. 의도하지 않은 값에 대한 접근과 할당을 막을 수 있다.
    
    - enum은 즉시 실행 함수(IIFE)로 변환되므로 트리 쉐이킹이 안 되는 문제를 야기할 수 있다. 때문에 const enum 혹은 as const assertion을 사용해 불필요한 코드의 증가를 막을 수 있다.

## 3.2 타입 조합

- 교차 타입 (`&`)
    
    ```tsx
    type NumberOrStringOrBoolean = string | number | boolean
    type NumberType = number & NumberOrStringOrBoolean
    ```
    
- 유니온 타입 (`|`)
    
    ```tsx
    type NumberOrString = number | string
    ```
    
- 인덱스 시그니처
    
    ```tsx
    interface IndexSigatureEx {
    	[key: string]: number
    }
    ```
    
    ⇒ `[Key: K]`: T 다음과 같은 꼴로 정의한다.
    
    - 속성 이름은 알 수 없지만, 속성값의 타입을 알고 있을 때 사용할 수 있다.
    
    ```tsx
    interinterface IndexSigatureEx {
    	[key: string]: number
    	name: string ⛔️
    }
    ```
    
    ⇒ `name` key가 `string`인데, string key에서는 `number`만 올 수 있기 때문에 `name: string`은 에러를 발생시킨다.
    
- 인덱스드 엑세스 타입
    - 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용
    
    ```tsx
    type Person = {
    	name: string
    	age: number
    	gender: boolean
    }
    
    type IndexedAccess1 = Person['name'] // string
    type IndexedAccess2 = Person['name' | 'age'] // string | number
    type IndexedAccess3 = Person[keyof Person] // string | number | boolean
    
    type alias = 'name' | 'age'
    type IndexedAccess4 = Person[alias] // string | number
    ```
    
    ```tsx
    // 예제 따라쳤는데 에러 떠서 이렇게 했습니다.
    const todos = [
      { id: 1, todo: '밥 먹기' },
      { id: 2, todo: '공부하기' },
      { id: 3, todo: '운동가기' },
    ]
    
    type PromotionItemType = typeof todos[0]
    /*
    type PromotionItemType = {
        id: number;
        todo: string;
    }
    */
    ```
    
- 맵드 타입
    - 자바스크립트 map 메서드처럼 다른 타입을 기반으로 한 타입을 선언할 때 사용하는 문법
    
    ```tsx
    type Example = {
    	a: number
    	b: string
    	c: boolean
    }
    
    type Subset<T> = {
    	[K in keyof T]?: T[K]
    }
    
    const aExample: Subset<Example> = { a: 3 },
    const bExample: Subset<Example> = { b: 'hello' },
    const cExample: Subset<Example> = { a: 3, c: true },
    ```
    
    ⇒ 맵드 타입에서 매핑할 때 `readonly`와 `?` 수식어를 사용할 수 있다.
    
    또한, 맵드 타입을 사용하면 기존 타입에 존재하던 `readonly`나 `?`를 `-`를 통해 제거한 타입을 선언할 수 있다.
    
    ```tsx
    type ReadOnlyEx = {
    	readonly a: number
    	readonly b: string
    }
    
    type CreateMutable<Type> = {
    	-readonly [Property in keyof Type]: Type[Property]
    }
    
    type ResultType = CreateMutation<ReadOnlyEx>
    ```
    
    ⇒ `readonly`를 제거한 타입을 생성
    
    ```tsx
    type OptionalEx = {
    	a?: number
    	b?: string
    }
    
    type Concrete<Type> = {
    	[Property in keyof Type]-?: Type[Property]
    }
    
    type ResultType = Concrete<OptionalEx>
    ```
    
    ⇒ `?`를 제거할 수 있음
    
    ```tsx
    // BottomSheetMap 객체: 바텀 시트 컴포넌트들을 관리
    const BottomSheetMap = {
      RECENT_CONTACTS: RecentContactsBottomSheet,
      CARD_SELECT: CardSelectBottomSheet,
      SORT_FILTER: SortFilterBottomSheet,
      PRODUCT_SELECT: ProductSelectBottomSheet,
      REPLY_CARD_SELECT: ReplyCardSelectBottomSheet,
      RESEND: ResendBottomSheet,
      STICKER: StickerBottomSheet,
      BASE: null,
    } as const;
    
    // BOTTOM_SHEET_ID 타입: BottomSheetMap의 키들을 타입으로 사용
    // BOTTOM_SHEET_ID는 "RECENT_CONTACTS" | "CARD_SELECT" | ...
    export type BOTTOM_SHEET_ID = keyof typeof BottomSheetMap;
    
    // 불필요한 중복
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
      SORT_FILTER: {
        resolver?: (payload: any) => void;
        args?: any;
        isOpened: boolean;
      };
      // ...
    };
    
    // Mapped Types를 활용하여 BottomSheetStore의 타입 선언
    type BottomSheetStore = {
      [key in BOTTOM_SHEET_ID]: {
        resolver?: (payload: any) => void;
        args?: any;
        isOpened: boolean;
      };
    };
    ```
    
- 템플릿 리터럴 타입
    
    ```tsx
    type Stage = 
        | 'init'
        | 'select-image'
        | 'edit-image'
        | 'decorate-card'
        | 'capture-image'
    type StateName = `${Stage}-stage`
    ```
    
- 제네릭
    - 함수, 타입, 클래스 등에서 내부적으로 사용할 타입을 미리 정해두지 않고 타입 변수를 사용해서 동적으로 타입을 지정해서 사용하는 방식
        
        ⇒ 재사용성이 크게향상
        
        - T: Type
        - E: Element
        - K: Key
        - V: Value
    - 반드시 꺾쇠괄호(`<>`)를 사용하지 않아도 된다. 컴파일러가 추론 가능하면 알아서 추론해준다.
    
    ```tsx
    const printValue = <T extends {}>(arg: T): void => {
        console.log(arg)
    }
    
    printValue(1)
    printValue('name')
    ```
    
    - 기본값도 추가할 수 있다.
    
    ```tsx
    type MyType<T = number> = {
    	a: T
    }
    ```
    
    ```tsx
    function Exam<T>(arg: T): number {
    	return T.length // ⛔️
    }
    ```
    
    ⇒ T에 length라는 프로퍼티가 존재하는지 모르기 때문에 에러를 발생시킨다.
    
    ```tsx
    type LengthProperty = {
    	length: number
    }
    
    function Exam<T extends LengthProperty>(arg: T): number {
    	return T.length // ✅
    }
    ```
    
    <aside>
    💡
    
    .tsx 에서 화살표 함수에 제네릭을 사용할 수 없다.
    제네릭의 꺾쇠괄호와 태그의 꺾소괄호를 혼동하는 문제가 발생한다.
    이를 해결하기 위해서는
    
    ```tsx
    const arrowFunc = <T extends {}>(arg: T): T[] => {
    	return new Array(3).fill(arg)
    }
    ```
    
    `extends {}`로 사용할 수 있다. 하지만, 보통 제네릭에서는 `function` 키워드를 많이 사용한다.
    
    </aside>
    

## 3.3 제네릭 사용법

> 다양한 예시를 보며 제네릭이 어떻게 사용되는지 알아보자
> 

### 함수의 제네릭

```tsx
function printStudent<T>(target: StudentType<T> | string | number): StudentType<T> {
	// do something
}
```

### 호출 시그니처의 제네릭

```tsx
function sum(a: number, b: number): number {
	return a + b
}
```

⇒ 다음과 같은 함수가 존재할 때

```tsx
(a: number, b: number) => number
```

⇒ 위 함수의 호출 시그니처는 다음과 같다.

그러면 이 호출 시그니처에 어떻게 제네릭을 사용할 수 있을까 ?

```tsx
interface useSelect<T> {
	fetchSelect: (prop: Params) => Promise<Response<SelectContent<T>>>
}
```

⇒ 다음과 같이 호출 시그니처의 제네릭을 사용할 수 있다.

### 제네릭 클래스

> 클래스에서는 어떻게 제네릭을 사용하는 지에 대해 알아본다.
> 

```tsx
class Box<T> {
  private content: T

  constructor(content: T) {
    this.content = content
  }

  getContent(): T {
    return this.content
  }

  setContent(content: T): void {
    this.content = content
  }
}
```

### 제네릭의 extends

> 타입 매개변수에 대한 제약 조건을 설정하는 기능을 할 수 있다.
> 

```tsx
const Fruits = {
  apple: 20000,
  kiwi: 30000,
}

// apple | kiwi
type FruitKeyType = keyof typeof Fruits
type FruitsType<Key extends FruitKeyType> = Record<Key, number>
```

제네릭 Key는 FruitKeyType으로 제한됨

하지만, 제한된 제네릭을 사용할 경우, 제네릭의 유연성을 잃을 수 있기 때문에 유니온 타입을 상속해서 선언한다.

### 우아한 형제들의 제네릭

> API 응답 값의 타입을 지정할 때 주로 사용한다.
> 

```tsx
export const fetchUser = (id: number): Promise<ApiResponse<UserInfo>>=> {
	const userPath = `/user/${id}`
	
	return request({
		method: 'GET',
		path: userPath
	})
}
```

> 불필요한 제네릭은 피하자
> 

> any 사용하기
> 

```tsx
type ReturnType<T = any> = {
	// ...
}
```

⇒ 이럴거면 제네릭을 사용하는 이유가 사라진다.

> 과한 제네릭 사용은 지양하자
> 

### 📒 면접 질문 리스트

- any와 unknown의 차이는 무엇인가요 ?
- void와 never의 차이는 무엇인가요 ?
- const enum을 왜 사용하나요 ?
- 유니온 타입과 교차 타입의 차이점은 무엇인가요 ?
- 인덱스 시그니처에 대해 설명해주세요.
- 인덱스드 엑세스 타입에 대해 설명해주세요.
- 맵드 타입에 대해 설명해주세요.
- 제네릭에 대해 설명해주세요.