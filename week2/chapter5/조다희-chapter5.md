# 4장 타입 확장하기, 좁히기

### 4.1 타입 확장하기

- 타입스크립트에서는 interface 와 type으로 타입을 정의 후, extends, 교차타입, 유니온타입으로 타입을 확장한다.
1. 타입 확장의 장점

```tsx
//1. 코드 중복을 줄일 수 있다.
interface BaseMenuItem {
	itemName: string|null
	itemImage: string|null
	itemDiscountAmt: number
	stock: number|null
}

// 1-1
interface BaseCartItem extends BaseMenuItem {
	quantity: number
}

//1-2
type BaseCartItem = {
	quantity: number
} & BaseMenuItem

```

1. 유니온타입; 합집합

```tsx
type MyUnion = A|B

interface CookingStep {
	orderId: string
	price: number
}

interface DeliveryStep {
	orderId: string
	time: number
	distance: string
}

//유니온타입으로 선언된 값은 유니온타입에 포함된 모든 타입의 공통된 속성에만 접근 가능
function getDeliveryDistance (step: CookingStep | DeliveryStep) {
	return step.distance
	//error
}
```

1. 교차타입; 교집합

```tsx
type MyIntersection = A&B

interface CookingStep {
	orderId: string
	price: number
}

interface DeliveryStep {
	orderId: string
	time: number
	distance: string
}

type BaedalProgress = CookingStep & DeliveryStep

function logBaedalInfo (progress: BaedalProgress) {
	console.log(progress.price)
	console.log(progress.distance)
}
```

```tsx
interface DeliveryTip {
	tip: string
}

interface StarRating {
	rate: number
}

type Filter = DeliveryTip & StarRating

const filter: Filter = {
		tip: '1,000원 이하'
		rate: 4
}
```

```tsx
type IdType = string | number
type Numeric = number | boolean

type Universal = IdType & Numberic // number 이면서 number인 경우 => number 만 성립
```

1. extends와 교차타입

```tsx
interface BaseMenuItem {
	itemName: string|null
	itemImage: string|null
	itemDiscountAmt: number
	stock: number|null
}
//유니온타입과 교차타입을 사용한 새로운 타입은 type 키워드로만 선언 가능
type BaseCartItem = {
	quantity: number
} & BaseMenuItem

const baseCartItem: BaseCartItem = {
	itemName: '지은이네 떡볶이',
	itemImage: 'https://abc.com/image/232',
	itemDiscountAmt: 2000,
	stock: 200,
	quantity: 20
}
```

```tsx
interface DeliveryTip {
	tip: number
}
//#1 error
interface Filter extends DeliveryTip {
	tip: string
}

//#2 ok
type Filter = {
	tip: string
} & DeliveryType
// tip 속성의 타입: never ; 서로 호환되는 속성이 없으므로
// type 키워드는 교차타입으로 선언되었을 때 새로 추가되는 속성에 대해 미리 알 수 없기 때문에
// 선언시 에러가 발생하지 않는다 
```

### 4.2 타입 좁히기 - 타입가드

1. 타입가드에 따라 분기처리하기

조건문과 타입가드를 활용하여 변수나 표현식의 타입 범위를 좁혀 다양한 상황에 다른 동작 수행

- 타입가드: 런타임에 조건문을 사용하여 타입을 검사하고 타입 범위를 좁혀주는 기능
    - 자바스크립트 연산자를 이용한 타입가드
        - 원시타입 추론 시; typeof
        - 인스턴스화된 객체 타입을 판별 시; instanceof
        - 객체의 속성이 있는지 없는지 판별; in
        
        ```tsx
        interface BasicNoticeDialogProps {
        	title: string
        	body: string
        }
        
        interface NoticeDialogWithCookieProps extends BasicNoticeDialogProps {
        	cookieKey: string
        	noForADay?: boolean
        	neverAgain?: boolean
        }
        
        export type NoticeDialogProps = BasicNoticeDialogProps 
        | NoticeDialogWithCookieProps
        
        const NoticeDialog: React.FC<NoticeDialogProps> = (props) => {
        	if ("cookieKey" in props) return <NoticeDialogWithCookey {...props} />
        	return <NoticeDialogBase {...props} />
        }
        ```
        
    
    - 사용자 정의 타입가드
        - is 연산자로 사용자정의 타입가드 만들기
        
        ```tsx
        const isDestinationCode = (x: string): x is DestinationCode =>
        destinationCodeList.includes(x)
        
        const getAvailableDestinationNameList = async():Promise<DestinationName[]> => {
        	const data = await AxiosRequest<string[]>("get", ".../destinations")
        	const destinationNames: DestinationName[] = []
        	data?.forEach((str)=>{
        		if(isDestinationCode(str)){
        			destinationNames.push(DestinationNameSet[str])
        		}
        	})
        	return destinationNames
        }
        ```
        

### 4.3 타입 좁히기  - 식별할 수 있는 유니온(Tagged/Discriminated Union)

1. 에러 정의하기
    
    ```tsx
    type TextError = {
    	errorType: 'TEXT' //tagged union
    	errorCode: string
    	errorMessage: string
    }
    
    type ToastError = {
    	errorType: 'TOAST'
    	errorCode: string
    	errorMessage: string
    	showDuration: number
    }
    
    type AlertError = {
    	errorType: 'ALERT'
    	errorCode: string
    	errorMessage: string
    	onConfirm: ()=> void
    }
    
    type ErrorFeedbackType = TextError | ToastError | AlertError
    const errorArr: ErrorFeedbackType[] = [
    	errorType: 'TEXT'
    	errorCode: 999
    	errorMessage: '잘못된 에러'
    	showDuration: 500
    	onConfirm: ()=>{} //tagged union 없다면 error 발생 x
    ]
    ```
    
2. 식별할 수 있는 유니온 사용시 주의사항
    1. 유닛타입으로 선언되어야 한다. - null, undefined, 리터럴타입, true, 1 …

### 4. Exhaustivenss Checking으로 정확한 타입분기 유지하기

```tsx
type ProductPrice = "10000" | "20000" | "5000"

const getProductName = (productPrice: ProductPrice) =>{
	if(productPrice === '10000') return '배민상품권 1만원'
	if(productPrice === '20000') return '배민상품권 2만원'
	//if(productPrice === '5000') return '배민상품권 5천원'
	else{
		exhaustiveCheck(productPrice)
		return '배민상품권'
	}
}

const exhaustiveCheck = (param: never) =>{
	throw new Error('type error')
}
```