# 5장 타입 활용하기

### 5.1 조건부 타입

1. extends와 제네릭을 활용한 조건부 타입
    
    ```tsx
    interface PayMethodBaseFromRes {
    	financialCode: string
    	name: string
    }
    
    interface Bank extends PayMethodBaseFromRes {
    	fullName: string
    }
    
    interface Card extends PayMethodBaseFromRes {
    	appCardType?: string
    }
    
    type PayMethodInfo<T extends Bank | Card> = T & PayMethodInterface
    type PayMethodInterface = {
    	companyName: string
    	//...
    }
    
    type PayMethodType = PayMethodInfo<Card> | PayMethodInfo<Bank>
    
    export const useGetRegisteredList = (type: 'card'|'appcard'|'bank'):
    UseQueryResult<PayMethodType[]> => {
    	const url = 'pay/${type === 'appcard' ? card:type}
    	
    	const fetcher = fetcherFactory<PayMethodType[]>({
    		onSuccess:(res)=>{
    			const usablePocketList =
    				res?.filter(pocket:PocketInfo<Card>| PocketInfo<Bank>)=>
    				pocket?.useType === 'USE' ?? []
    			return usablePocketList
    		}
    	})
    	
    	const result = useCommonQuery<PayMethodType<T>[]>(url, undefined, fetcher)
    	
    	return reuslt
    }
    ```
    
2. infer로 타입추론
    
    extends, infer, 제네릭을 활용하면 타입을 조건에 따라 더 세밀하게 사용할 수 있다.
    
    ```tsx
    interface RouteBase {
    	name: stirng;
    	path: string;
    	component: ComponentType0
    }
    
    export interface RouteItem {
    	name: string;
    path: string;
    component?: ComponentType;
    pages: RouteBase[]
    }
    
    export const routes: RouteItem[] = [
    	{name: '기기 내역 관리'
    	path: '/device-history'
    component: DeviceHistoryPage
    }.
    {
    name:'헬멧 인증 관리'
    path:'/'helmet-certification'.
    component: HelmetCertificationPage
    }
    //...
    ]
    ```
    

```tsx
export interface SubMenu {
	name: string;
	path: string;
}

export interface MainMenu {
	name: string;
	path?: string;
	subMenus?:ReadonlyArray<SubMenu> //*
}

export type MenuItem = MainMenu|SubMenu

export const menuList: MenuList[]=[
	{
	name:'계정 관리'
	subMenus:[
	{
		name:'기기 내역 관리'
		pah:'/device-history'
	},
	{
		name:'헬멧 인증 관리'
		path:'/'helmet-certification'
}
	]
}
] as const //*

type PermissionNames = '기기 정보 관리'|'안전모 인증 관리'

//Route 관련 타입의 name은 menuList의 name에서 추출한 타입인 PermissionNames만 올 수 있도록 변경
type UnpackMenuNames<T extens ReadonlyArray<MenuItem>> 
= T extends ReadonlyArray<infer U>
		? U extends MainMenu
			? U['subMenus'] extends infer V
				? V extends ReadonlyArray<SubMenu>
					? UnpackMenuNames<V>
					: U['name']
				: never
			: U extends SubMenu
				? U['name']
				: never
			:never

```

### 5.3 커스텀 유틸리티 타입 활용

1. styled-components 의 중복 타입 선언 피하기

```tsx
//HrComponent.tsx
export type Props = {
	height?: string;
	color?: keyof typeof colors;
	isFull?:boolean
	className?: string
}

export const Hr: VFC<Props> = ({height, color, isFull, className})=>{
//,,,
return <HrComponent height={height} color={color} isFull={isFull} className={className} />

}

//style.ts
import {Props} from '...'
type styledProps = Pick<Props,'height'|'color'|'isFull'

const HrComponent = styled.hr<StyledProps>`
	height: ${({height})=> height || '10px'}
	${({isFull}) => isFull && css`margin: 0 '15px`

`
```

1. PickOne 유틸리티 함수

```tsx
type Card = {
	type:'card' //식별할 수 있는 유니온 추가하여 타입을 정확히 추론할 수 있도록 함
		//but 귀찮음..
	card:'string'
}

type Account = {
	type: 'account'
	account: 'string'
}

function withdraw(type: Card|Account){
//..
}
withdraw({account:'hana', card:'hana'}) //식별할 수 있는 유니온 없을 경우 타입에러 안남
```

- 하나의 속성이 들어왔을 때 다른 타입을 옵셔널 + undefined로 지정하면 사용자가 의도적으로 undefined 값을 넣지 않는 이상 원치않는 속성에 값을 넣었을 때 타입 에러가 발생할 것이다.

```tsx
type PayMethod = {account: string; card?: undefined} 
| {card: string; account?: undefined}

type PickOne<T> = {
	[P in keyof T]: Record<P, T[P]> & Partial<Record<Exclude<keyof T,P>, undefined>>
}[keyof T]
```

- PickOne 살펴보기

```tsx
type One<T> = { [P in keyof T]: Record<P, P[T]> }[keyof T]

type Card = {card:string}
const one:One<Card> = {card:'hana'}

type ExcludeOne<T> = { [P in keyof T]: Partial<Record<Exclude<key of T,P>, undefined>>
}[key of T]

type PickOne<T> = One<T> & ExcludeOne<T>
```

- PickOne 타입 적용

```tsx
type Card = {
	card:'string'
}

type Account = {
	account: 'string'
}

type CardOrAccount = PickOne<Card & Account>
function withdraw(type: CardOrAccount){}

withdraw({account:'hana', card:'hana'}) //error
```

1. NonNullable 타입 검사 함수로 타입가드하기
- NonNullable 타입
    - 제네릭으로 받는 T가 null 또는 undefined일 때 never 또는 T를 반환하는 타입
    - NonNullable을 사용하면 null이나 undefined가 아닌 경우를 제외할 수 있다
    
    ```tsx
    type NonNullable = T extends null | undefined ? never:T
    ```
    
- null, undefined를 검사해주는 NonNullable 함수

```tsx
function NonNullable<T>(value:T): value is NonNullable<T>{
	return value !== null && value !== undefined
}
```

- Promise.all 사용시 적용하기

```tsx
const shopList = [
 {no: 100, category:'pizza'}
//...
]

const shopAdCampaignList = await Promise.all(shopList.map
(s=>AdCampaignAPI.operating(s.no)))

const shopAd = shopAdCampaignList.filter(Nonnullable)
```

### 5.4 불변 객체 타입으로 활용하기

```tsx
const colors = {
	black: '#000000',
//..
}

const theme = {
	colors:{
		default:colors.black,
		...colors
	},
	backgroundColor:{
		default: colors.white,
		gray: colors.gray
		//...
	},
	fontSize: {
		sm: '12px',
		lg: '16px'
		//...
	}
}

type colorType = typeof keyof theme.colors
type backgroundColorType = typeof keyof theme.backgroundColor
type fontSizeType = typeof keyof theme.fontSize

interface Props {
	color?: colorType
	backgroundColor?: backgroundColorType
	fontSize?: fontSizeType
	onClick: (event: React.MouseEvent<HTMLButtonElement)=>void | Promise<void>
}

const Button: FC<Props> = ({color, backgroundColor, fontSize})=>{
	return 
		<ButtonWrap 
			color={color}
			backgroundColor={backgroundColor}
			fontSize={fontSzie}
		>
		{children}
		</ButtonWrap>
}

const ButtonWrap = styled.button<Omit<Props, onClick>>`
	color: ${({color})=> theme.color[color ?? 'default']}
//....
`
```

### 5.5 Record 원시 키 타입 개선하기

- Record 타입은 무한한 키 집합을 가지므로 객체 안에 없는 키 값을 사용해도 타입 에러가 안남

```tsx
type PartialRecord<K extends string, T> = Partial<Record<K,T>>

type Category = string

interface Food {
	name: string
	//..
}

const foodByCategory: PartialRecord<Category, Food[]> = {
	한식: [{name: '비빔밥'}, ...]
	일식: [{name:'초밥'},...]
}

foodByCategory['양식'] //Food[] | undefined
foodByCategory['양식'].map(food=> console.log(food.name)) //Object is possibly undefined
foodByCategory['양식']?.map(food=> console.log(food.name)) //ok
```