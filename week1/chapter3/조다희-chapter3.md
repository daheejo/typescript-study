# 3장 고급 타입

### 3.1 타입스크립트의 독자적인 타입 시스템

1. any
    - 자바스크립트에 존재하는 모든 값을 오류 없이 받아낼 수 있다
    - tsconfig에서 noImplicitAny 옵션을 활성화 시 any 타입에 대한 경고를 발생시킬 수 있다
    - any를 어쩔 수 없이 사용해야 하는 경우
        - 개발 단계에서 임시로 값을 지정해야 할 때
        - 어떤 값을 받거나 넘겨줄지 정할 수 없을 때
        - 값을 예측할 수 없을 때
2. unknown
    - any와 마찬가지로 모든 타입에 할당 가능하나 any 타입 외에 다른 타입으로 할당 불가능
    - 데이터 구조를 파악하기 힘들 때 any 대신 사용하기를 권장
3. void
4. never
    - 값을 반환할 수 없는 타입
    - 자바스크립트에서 값을 반환할 수 없는 경우
        - 에러를 던지는 경우
        - 무한 루프
    - never는 모든 타입의 하위 타입 → never 자신을 제외한 어떠한 타입도 never에 할당 불가
5. Array
    - 원소의 타입 관리 강제
    - 여러 타입의 원소가 필요하다면 유니온 사용
    - 
    
    ```tsx
    const a1 :Array<number|string> = [1,'1']
    const a2 :number[]|string[]=[1,'1']
    const a3:(number|string)[]=[1,'1'] 
    ```
    
    - 튜플 사용시 배열 길이에 제한을 둘 수 있다
    
        
        ```tsx
        let t : [number] = [1]
        t = [1,2] // impossible
        
        let t2 : [number, string, boolean] = [1,'1',true]
        ```
        
        - react의 useState는 튜플을 반환; [상탯값, 세터]
        - 스프레드 연산자를 사용하여 튜플과 배열의 성질을 혼합하여 사용할 수 있다
6. enum
    - 타입스크립트는 명명한 각 멤버의 값을 스스로 추론
    - 각 멤버에 명시적으로 값을 할당할 수 있으며, 일부 멤버만 할당해도 나머지 멤버에 이전 멤버의 값을 기준으로 1씩 늘려가며 할당
    - 관련이 높은 멤버를 모아 문자열 상수처럼 사용하고자할 때 유용
    - 타입스크립트가 자동으로 추론한 열거형은 안전하지 않을 수 있다는 단점
        - const enum으로 열거형 선언하여 역방향 접근을 막을 수 있지만 숫자 상수로 관리되는 열거형은 선언한 값 이외의 값을 할당하거나 접근할 때 이를 방지할 수 없다는 단점
        - 문자열 상수 방식의 열거형을 사용하는 것이 안전
    - 열거형은 값공간과 타입공간에서 모두 사용되어 자바스크립트 코드로 변환될 때 즉시실행함수로 변환됨
        - 트리쉐이킹시 사용하지 않는 값인 경우도 번들에 포함하여 코드 크기 중가
        - const enum / as const 단언 사용하여 이를 해결할 수 있다

### 3.2 타입 조합

1. 교차타입: &
2. 유니온타입: |
3. 인덱스 시그니쳐
    1. 
    
    ```tsx
    interface IdxSig {
    [key: string]: number
    }
    ```
    
4. 인덱스드 엑세스 타입
5. 맵드 타입: 다른타입을 기반으로 한 타입 선언 시 유용
    
    ```tsx
    type Example {
    a: string,
    b: number
    }
    
    type Subset<T> = {
     [K in keyof T]?: T[K]
    }
    
    const ex1: SubSet<Example> = {a: 'a'}
    const ex2: SubSet<Example> = {b: 3}
    ```
    
    - as 키워드로 키를 재지정 가능
6. 템플릿 리터럴
7. 제네릭
    - 타입추론이 가능한 경우 타입 명시를 생략할 수 있다
    
    
    ```tsx
    function exfunc<T>(arg:T):T[] {
    	return new Array(3).fill(arg)
    }
    
    exfunc('hello')//T는 string으로 추론
    ```
    
    - 파일 확장자가 tsx 인 경우 화살표 함수에 제네릭 사용시 에러
    
    
    ```tsx
    //error
    const arrfunc1 = <T>(arg:T):T[] =>{
    return new Array(3).fill(arg)
    }
    
    //no error
    const arrfunc2 = <T extends {}>(arg:T):T[] =>{
    return new Array(3).fill(arg)
    }
    ```
    

### 3.3 제네릭 사용법

함수의 제네릭

```tsx
function identity<T>(arg: T): T {
    return arg;
}

// 사용 예시
const result1 = identity<number>(10); // result1의 타입은 number
const result2 = identity<string>("hello"); // result2의 타입은 string
```

호출시그니처의 제네릭

```tsx
interface GenericFunction {
    <T>(arg: T): T;
}

const myIdentity: GenericFunction = <T>(arg: T): T => {
    return arg;
};

// 사용 예시
const result1 = myIdentity(123); // result1의 타입은 number
const result2 = myIdentity("abc"); // result2의 타입은 string
```

제네릭 클래스

```tsx
class GenericBox<T> {
    private contents: T;

    constructor(value: T) {
        this.contents = value;
    }

    getContents(): T {
        return this.contents;
    }
}

// 사용 예시
const numberBox = new GenericBox<number>(123);
console.log(numberBox.getContents()); // 출력: 123

const stringBox = new GenericBox<string>("Hello");
console.log(stringBox.getContents()); // 출력: Hello

```

제한된 제네릭

```tsx
interface Lengthwise {
    length: number;
}

function logLength<T extends Lengthwise>(item: T): T {
    console.log(item.length);
    return item;
}

// 사용 예시
logLength({ length: 10, value: "Hello" }); // 출력: 10
logLength([1, 2, 3]); // 출력: 3
// logLength(10); // 오류: 'number' 타입은 'length' 속성이 없습니다.

```

확장된 제네릭

```tsx
class Pair<T> {
    constructor(public first: T, public second: T) {}
}

class KeyValuePair<K, V> extends Pair<K> {
    constructor(public key: K, public value: V) {
        super(key, value);
    }
}

// 사용 예시
const kvPair = new KeyValuePair<string, number>("age", 25);
console.log(kvPair.key); // 출력: age
console.log(kvPair.value); // 출력: 25

```