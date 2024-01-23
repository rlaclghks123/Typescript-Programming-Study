# 고급 타입

### 서브타입과 슈퍼타입

- 서브 타입 : `두 개의 타입 A,B가 있고, B가 A의 서브타입이면 A가 필요한 곳에는 어디든 B를 안전하게 사용할 수 있다.`
- 슈퍼 타입 : `두 개의 타입 A,B가 있고, B가 A의 슈퍼타입이면 B가 필요한 곳에는 어디든 A를 안전하게 사용할 수 있다.`

```
// 슈퍼타입
type Animal = {
  name: string;
  age: number;
};

// 서브타입
type Dog = Animal & {
  breed: string;
};

// animal은 슈퍼타입
let animal: Animal = {
  name: 'Buddy',
  age: 2,
};

// myDog는 서브타입
let myDog: Dog = {
  name: 'Buddy',
  age: 2,
  breed: 'Golden Retriever',
};

// 슈퍼타입에 서브타입은 담을 수 있다.
animal = myDog;

// 서브타입에 슈퍼타입을 담을 순 없다.
myDog = animal;
```

### 가변성

- 일반적으로는 A라는 타입이 B라는 타입의 서브타입인지 아닌지 쉽게 판단할 수 있다.
- 하지만 매개변수화된 타입(제네릭)등 복합타입에서는 이 문제가 복잡해진다.

  - Array<A>는 어떤 상황에서 Array<B>의 서브타입이 될까?
  - 형태 A는 어떤 상황에서 다른 형태 B의 서브타입이 될까?
  - 함수 : (a:A) => B는 어떤 상황에서 다른 함수 (c:C) => D의 서브타입이 될까?

- 아래의 꼼수는 타입스크립트 문법은 아니고 이해하기 쉽게 도와주는 `꼼수 표현`이라고 하겠습니다.
  - A <: B 는 A는 B와 같거나 B의 서브타입이라는 의미 => B가 슈퍼타입, A가 서브타입
  - A >: B 는 A는 B와 같거나 B의 슈퍼타입이라는 의미 => A가 슈퍼타입, B가 서브타입

### 형태와 배열 가변성

- 안정성, 유연성(실용성) 중 어디에 중점적으로 할지에 따라 달라지기 때문에 언어마다 복합 타입의 서브타입 규칙은 언어마다 다르다.
- 타입스크립트의 가변성

  1. 불변 : 정확히 T를 원함
  2. 공변 : `<:T`를 원함
  3. 반변 : `>:T`를 원함
  4. 양변 : `<:T 또는 >:T`를 원함

- 타입스크립트에서 `모든 복합 타입의 멤버(객체, 클래스, 배열, 함수, 반환타입)은 공변`
- `함수 매개변수 타입만 예외적으로 반변`

### 할당성

- 할당성이란 `A라는 타입을 다른 B라는 타입이 필요한 곳에 사용할 수 있는지를 결정하는 타입스크립트 규칙`
- 열거형이 아닌 타입은 아래의 2가지 조건을 만족해야함
  1. A <: B => 서브타입은 슈퍼타입에 할당 가능
  2. A는 any => any타입은 어디든 할당 가능

```
// 슈퍼타입
type Animal = {
  name: string;
  age: number;
};

// 서브타입
type Dog = Animal & {
  breed: string;
};

// animal은 슈퍼타입
let animal: Animal = {
  name: 'Buddy',
  age: 2,
};

// myDog는 서브타입
let myDog: Dog = {
  name: 'Buddy',
  age: 2,
  breed: 'Golden Retriever',
};

// 슈퍼타입에 서브타입은 담을 수 있다.
animal = myDog;

// 어느타입이든 any타입은 담을 수 있다.
myDog = {
  name: 'Buddy',
  age: 2,
} as any;
```

- 열거형인 타입
  1.  A는 열거형 B의 멤버다.
  2.  B는 number 타입의 멤버를 최소 한 개 이상 가지고 있으며 A는 number이다.
- 열거형은 규칙2로 인해 안정성이 떨어지기 때문에 권장하지 않는다고 한다.

```
enum Language {
  English,
  Spanish,
  Russian,
  Korean,
}

console.log(Language[99]); // undefined
```

### 타입 넓히기

- 타입스크립트는 타입을 정밀하게 추론하기 보다는 일반적으로 추론한다.

```
let a = 'x'   // x가 아닌 string으로 추론
var b =  true   // true가 아닌 boolean 추론
const c = {x:3}  //  {x:3}이 아닌 {x:number} 로 추론
let d = null;   // null이 아닌 any로 추론
```

- 타입이 넓혀지지 않도록 해주는 const라는 특별 타입을 제공한다.
- 객체의 경우 as const를 활용할 수 있다.

```
const a = 'x'; // x로 추론
const b = true; // true로 추론
let c = { x: 3 } as const; //  {readonly x:3}으로 추론
const d = null; // null로 추론
```

### 초과 프로퍼티 확인(잉여 속성 검사)

- 객체 타입과 그 멤버들은 공변 관계이므로 추가 확인을 수행하지 않으면 문제가 발생할 수 있다.
- 뭔소리냐? 슈퍼타입 A와 서브타입 B가 있는 경우 슈퍼타입에 서브타입을 담으면 문제가 없다. => 서브타입을 담을 때 오타가 나도 문제가 발생하지 않는다.

```
type Options = {
  baseUrl: string;
  cacheSize?: number;
  tier?: 'prod' | 'dev';
};

class API {
  constructor(private options: Options) ...
}

new API({
  baseUrl: 'https',
  tierrqweqwe: 'prod',
});

아래의 예시와 같이 이론상 바로위에서 tierrqweqwe 속성이 추가되는 새로운 객체도 서브타입이기 때문에 동작 되어야 된다.

// 슈퍼타입
type Animal = {
  name: string;
  age: number;
};

// 서브타입
type Dog = Animal & {
  breed: string;
};

// animal은 슈퍼타입
let animal: Animal = {
  name: 'Buddy',
  age: 2,
};

// myDog는 서브타입
let myDog: Dog = {
  name: 'Buddy',
  age: 2,
  breed: 'Golden Retriever',
};

// 슈퍼타입에 서브타입은 담을 수 있다.
animal = myDog;


```

- 하지만 타입스크립트는 초과 프로퍼티 확인 기능을 통해 객체의 경우 이 문제를 막아준다.

- 초과 프로퍼티 확인 기능은 `신선한 객체 리터럴 타입 T를 다른 타입 U에 할당하려는 상황에서 T가 U에는 존재하지 않는 프로퍼티를 가지고 있다면 타입스크립트는 에러로 처리`한다.

- 신선한 객체 리터럴 타입이란 : 타입스크립트가 객체 리터럴로부터 추론한 타입

  - 그러나 아래의 경우 신선함이 사라지고 타입이 넓혀짐

  1. 객체 리터럴이 타입 어설션을 사용하는 경우
  2. 변수로 할당되는 경우

- 아래의 5가지 예시가 있다.
  1. 문제없이 동작 하는 경우
  2. 초과 프로퍼티 확인 하는 경우
  3. 초과 프로퍼티 확인 하지 않는 경우 : 어설션으로 인한 신선한 객체 X
  4. 초과 프로퍼티 확인 하지 않는 경우 : 변수할당으로 인한 신선한 객체 X
  5. 초과 프로퍼티 확인 하는 경우 : 원하는 타입을 명시한 경우

```
type Options = {
  baseUrl: string;
  cacheSize?: number;
  tier?: 'prod' | 'dev';
};

class API {
  constructor(private options: Options) {
    console.log(options);
  }
}

// 1. 문제 없음
new API({
  baseUrl: 'https',
  tier: 'prod',
});

// 2. 신선한 객체이므로 검사를 수행함 why? 타입스크립트가 추론(타입어설션 X), 변수에 할당하지 않았기 때문 => 따라서 에러 발생
new API({
  baseUrl: 'https',
  tiger: 'prod',
});

// 3. 신선한 객체가 아니므로 검사를 수행하지 않음 => 에러 X   Why? as Option을 통해 `타입 어설션 해줌`
new API({
  baseUrl: 'https',
  tiger: 'prod',
} as Options);

// 4. 신선한 객체가 아니므로 검사를 수행하지 않음 => 에러 X   Why? 객체를 변수에 할당했기 때문
let tiger = {
  baseUrl: 'https',
  tiger: 'prod',
};
new API(tiger);

// 5. 변수에 할당하더라도 Options 타입으로 명시하면 option에 할당된 객체는 신선한 객체로 취급 => 단 new API로 전달할떄가 아닌 tiger에 담을때 검사 수행

let tiger2: Options = {
  baseUrl: 'https',
  tiger: 'prod',
};

new API(tiger2);

let tiger = {
  baseUrl: 'https',
  tiger: 'prod',        /// 에러 발생
};
new API(tiger);


// 단 아래의 경우 에러가 발생하지 않음
let tmp = {
  baseUrl: 'https',
  tiger: 'prod',
};

// tmp를 변수에 담았기 떄문에
let tiger3: Options = tmp;

new API(tiger3);

```

### 차별화된 유니온 타입

- 태그된 유니온 : 타입에 태그를 추가하여 효율적으로 타입을 설계할 수 있는 방식
- 아래의 경우 event.target이 유니온 타입으로 추론된다. => 좀 더 정확하게 파악을 원함

```
type UserTextEvent = { value: string; target: HTMLInputElement };
type UserMouseEvnet = { value: [number, number]; target: HTMLElement };

type UserEvent = UserTextEvent | UserMouseEvnet;

function handle(event: UserEvent) {
  if (typeof event.value === 'string') {
    event.value; // string
    event.target; // HTMLInputElement | HTMLElement             ========>  event.target이 HTMLInputElement | HTMLElement로 제대로 정제되지 않음
    return;
  }
}

```

- 이떄 태그된 유니온을 사용하면 좋다.

```
// TextEvent라는 리터럴 타입 생성
type UserTextEvent = { type: 'TextEvent'; value: string; target: HTMLInputElement };

// MouseEvent라는 리터럴 타입 생성
type UserMouseEvnet = { type: 'MouseEvent'; value: [number, number]; target: HTMLElement };

type UserEvent = UserTextEvent | UserMouseEvnet;

function handle(event: UserEvent) {
  if (event.type === 'TextEvent') {
    event.value; // string
    event.target; // HTMLInputElement
    return;
  }
  event.value; // [number, number]
  event.target; // HTMLElement
}
```

# 고급 객체 타입

### 키인 연산자

- `[][key값] 형태` Ex) `type FriendList = APIResponse['user][friendList]`
- 특정 객체의 키 값
- 객체 타입에서 특정 속성의 타입을 동적으로 가져오기 위해 사용되는 기능
- 단 키인 프로퍼티 타입을 찾을 때 점표기법이 아닌 `대괄호 표기법을 사용`해야 한다.

- 아래의 경우 FriendList을 직접 다 쳐줄수도 있지만 너무 귀찮다.

  ```
  // 키인 연산자 사용 전
  type FriendList = {
    count: number;
    frineds: {
      firstName: string;
      lastName: string;
    }[];
  };

  type APIRes = {
    user: {
      userId: string;
      friendList: {
        count: number;
        frineds: FriendList;
      };
    };
  };

  ```

- 이때 키인 연산자를 통해 쉽게 구현

```
type APIRes = {
  user: {
    userId: string;
    friendList: {
      count: number;
      frineds: {
        firstName: string;
        lastName: string;
      }[];
    };
  };
};

type FriendList = APIRes['user']['friendList'];
type Friend = FriendList['frineds'][number];

```

### keyof 연산자

- keyOf를 이용하면 객체의 모든 키를 문자열 리터럴 타입 유니온으로 얻을 수 있다.
- 키인연산자와 혼합해서 사용하면 안전하게 타입을 만들 수 있다.

```
type ResponseKeys = keyof APIRes; // user
type UserKeys = keyof APIRes['user']; // userId | friendList
type FriendListKeys = keyof APIRes['user']['friendList'] // count | frineds
```

- 제네릭을 사용할 경우

```
// 객체인 O받고, K는 O 객체의 키(keyof O)들로 구성되어야 한다.
function get<O extends object, K extends keyof O>(o: O, k: K): O[K] {
  return o[k];
}

type ActivityLog = {
  lastEvent: Date;
  events: {
    id: string;
    timestamp: Date;
    type: 'Read' | 'Write';
  }[];
};

const activityLog: ActivityLog = {
  lastEvent: new Date(),
  events: [
    {
      id: '123',
      timestamp: new Date(),
      type: 'Read',
    },
  ],
};

const test = get(activityLog, 'lastEvent');  // Date

// 위에서 get함수는 객체인 O받고, K는 O 객체의 키(keyof O)들로 구성되어야 한다고 했습니다.

// 1. test를 예로들면 객체인 activityLog를 받고, 'lastEvent'는 activityLog객체의 키들로 구성되어야 한다.
// => activityLog의 키는 lastEvent, events로 되어있기 때문에 문제가 없습니다.

// 2. get함수는 o[k]를 반환합니다.
// => test를 예로들면 객체인 activityLog[lastEvent]를 반환합니다 => Date

```

### Record 타입

- 무엇인가를 매핑하는 용도로 객체를 활용하는데 사용합니다.
- 사용법 : `Record<키타입, 값타입>` Ex) Reocrd<string, number> === [key:string]:number
- 인덱스 시그니처와 비슷하다. 인덱스 시그니처 : `[key:string] : number`
- 인덱스 시그니처와 차이점
  - 인덱스 시그니처에서는 객체의 값(value)의 타입은 제한할 수 있지만, 키는 반드시 일반 string, number, symbol이어야 한다. => 객체의 키값은 제한됨
  - Record는 `객체의 키 타입도 string, number의 서브타입으로 제한`할 수 있다.

```

type Weekday = 'Mon' | 'Tue' | 'Wed' | 'Thu' | 'Fri';
type Day = Weekday | 'Sat' | 'Sun';

// 인덱스 시그니처 사용
const nextDay: { [key: string]: Day } = {
  Mon: 'Tue',
  Tue: 'Wed',
  Wed: 'Thu',
  Thu: 'Fri',
  SUN: 'Mon', // 원하는 값과 다르지만 에러 X => 키값을 평일로 제한하고 싶은데, 인덱스 시그니처의 경우 방법이 없음
};

// 인덱스 시그니처 사용
type Weekday = 'Mon' | 'Tue' | 'Wed' | 'Thu' | 'Fri';
type Day = Weekday | 'Sat' | 'Sun';

const nextDay2: { [key: Weekday]: Day } = { // 에러발생 : 인덱스 시그니처 매개변수 형식은 리터럴 이나 제네릭 형식일 수 없다
  Mon: 'Tue',
  Tue: 'Wed',
  Wed: 'Thu',
  Thu: 'Fri',
  SUN: 'Mon', // 원하는 값과 다르지만 에러 X
};

// 키값을 string 서브타입인 Weekday 타입으로 제한할 수 있음. 유니언 타입을 통해 키값을 제한
type Weekday = 'Mon' | 'Tue' | 'Wed' | 'Thu' | 'Fri';
type Day = Weekday | 'Sat' | 'Sun';

const nextDay3: Record<Weekday, Day> = {
  Mon: 'Tue',
  Tue: 'Wed',
  Wed: 'Thu',
  Thu: 'Fri',
  Fri: 'Sat',
  SUN: 'Mon', // 에러발생 > SUM이 없다.
};


```

### 매핑된 타입

- 사용법 : `[key in 객체타입] : value타입 형태`
- 한 객체당 최대 한 개의 매핑된 타입을 가질 수 있다.

```
type Weekday = 'Mon' | 'Tue' | 'Wed' | 'Thu' | 'Fri';
type Day = Weekday | 'Sat' | 'Sun';

const nextDay: {[K in Weekday]:Day}= {
  Mon: 'Tue',
  Tue: 'Wed',
  Wed: 'Thu',
  Thu: 'Fri',
  Fri: 'Sat',
};


```

### 내장 매핑된 타입

1. Record<Keys, Values> : Keys 타입의 키와 Values 타입의 값을 갖는 객체

   ```
   type ValueTypes = {
     name: string;
     value: number;
    };

   type KeyTypes = 'event' | 'point';

   const recordPractice: Record<KeyTypes, ValueTypes> = {
     event: {
       name: 'event',
       value: 5,
    },
   point: {
     name: 'point',
     value: 3,
     },
   };
   ```

2. Partial<Object> : Object의 모든 필드는 선택형으로 표시

   ```
   interface Obj {
    id: string;
    title: string;
    isDone: boolean;
    startDate: string;
   }

   type Partialed = Partial<Obj>;

   const partialed1: Partialed = {
     id: '1'
   };

   const partialed2: Partialed = {
     id: '1',
     title: 'title',
   };

   const partialed3: Partialed = {
     id: '1',
     title: 'title',
     isDone: true,
   };

   const partialed4: Partialed = {
     id: '1',
     title: 'title',
     isDone: true,
     startDate: '13:00',
   };
   ```

3. Required<Object> : Object의 모든 필드는 필수형으로 표시

   ```

   interface Obj {
     id: string;
     title: string;
     isDone: boolean;
     startDate: string;
   }

   type Requireded = Required<Obj>;

   const requireded1: Requireded = { // 에러발생
     id: '1',
   };

   const requireded2: Requireded = {  // 에러발생
     id: '1',
     title: 'title',
   };

   const requireded3: Requireded = {  // 에러발생
     id: '1',
     title: 'title',
     isDone: true,
   };

   const requireded4: Requireded = {   // 문제없음
     id: '1',
     title: 'title',
     isDone: true,
     startDate: '13:00',
   };
   ```

4. Readonly<Object> : Object의 모든 필드는 읽기 전용으로 표시

   ```

   interface Obj {
     id: string;
     title: string;
     isDone: boolean;
     startDate: string;
   }


   let readonlyed1 = {
     id: '1',
     title: 'title',
     isDone: true,
     startDate: '13:00',
   };

   readonlyed1.id = '2';             // 수정 가능
   readonlyed1.title = 'title2';     // 수정 가능
   readonlyed1.isDone = false;       // 수정 가능
   readonlyed1.startDate = '00:00';  // 수정 가능


   type Readonlyed = Readonly<Obj>;
   let readonlyed2: Readonlyed = {
     id: '1',
     title: 'title',
     isDone: true,
     startDate: '13:00',
   };

   readonlyed2.id = '2';            // 에러발생 : 읽기 전용 이므로 할당할 수 없다
   readonlyed2.title = 'title2';     // 에러발생 : 읽기 전용 이므로 할당할 수 없다
   readonlyed2.isDone = false;       // 에러발생 : 읽기 전용 이므로 할당할 수 없다
   readonlyed2.startDate = '00:00'; // 에러발생 : 읽기 전용 이므로 할당할 수 없다

   ```

5. Pick<Object, Keys> : Object에서 Keys프로퍼티만 추출

   ```
   interface Obj {
    id: string;
    title: string;
    isDone: boolean;
    startDate: string;
   }

   type Picked = Pick<Obj, 'id' | 'title'>;

   const picked:Picked = {
     id:'1',
     title:'title',
     // isDone:true,       // 에러
     // startDate:'13:00'  // 에러
   }
   ```

6. Omit<Object, keys> : 특정 속성만 제거한 타입, Pick과 반대

   ```
   interface Obj {
    id: string;
    title: string;
    isDone: boolean;
    startDate: string;
   }

   type Omited = Omit<Obj, 'startDate'>;

   const omited: Omited = {
    id: '1',
    title: 'title',
    isDone:true,
    // startDate:'13:00'  // 에러
   };
   ```

### 컴패니언 객체 패턴

- 타입스크립트에서는 타입과 값은 별도의 네임스페이스를 갖는다.
- 따라서 같은 영역에서 하나의 이름을 타입과 값 모두 연결하여 타입과 값을 그룹화 하는 패턴
- 아래의 예시와 같은 이름이지만 타입의 Currency, 값의 Currency 2가지로 사용 됨

```

type Currency = {
  unit: 'EUR' | 'KOR' | 'JPY' | 'USD';
  value: number;
};

// Currency는 any로 추론 : 어떤 Currency인지 혼란이 와서 any로 추론  => 에러는 안남
const Currency = {
  DEFAULT: 'USD',
  from(value: number, unit = Currency.DEFAULT) {
    return { unit, value };
  },
};

// 타입의 Currency 사용
const amountDue: Currency = {
  unit: 'JPY',
  value: 83123.1,
};

// 값의 Currency로 사용
const otherAmountDue = Currency.from(330, 'KOR');

```

# 조건부 타입

- 삼항연산자와 비슷하게 사용한다. `type isString<T> = T extends string ? true : false`
- 위의 식은 T는 string의 서브타입인가? 라는 의미
  - 서브타입이면 True => ? 다음값
  - 서브타입이 아니면 False => : 다음값

### 분배적 조건부

- 타입스크립트는 분배법칙을 통해 분배가 된다.

```
분배조건 일어나기 전                                                    분배조건 일어난 후
 string extends T ? A : B                          string extends T ? A : B
(string | number) extends T ? A : B               (string extends T ? A : B) |  (number extends T ? A : B)
(string | number | boolean) extends T ? A : B     (string extends T ? A : B) |  (number extends T ? A : B) | (boolean extends T ? A : B)
```

### 조건부타입과 분배적 조건부

- 조건부 타입과 분배적 조건부를 합쳐서 이해해보겠습니다.

```
type Without<T,U> = T extends U ? never : T

type A = Without<boolean | number | string, boolean>   -> A의 타입은  number | string 입니다.

1. 분배법칙이 일어난다.
// 1-1. boolean이 boolean 또는 boolean 서브타입이면 never 그렇지 않으면 boolean => never
boolean extends boolean ? never : boolean

// 1-2. number가 boolean 또는 boolean 서브타입이면 never 그렇지 않으면 number => number
number extends boolean ? never : number

// 1-3. string이 boolean 또는 boolean 서브타입이면 never 그렇지 않으면 string => string
string extends boolean ? never : boolean

2. 위의 분배법칙에 따라 never | number |string 된다.

3. 이를 단순화하여 type A = number | string이 된다.
```

### infer 키워드

- 타입을 스스로 추론할 수 있게 하는 것
- 인라인을 통해 선언하는 전용 문법

```
type Elements1<T> = T extends unknown[] ? T[number] : T;      => T[number]는 배열의 요소를 의미
type test1 = Elements1<number[]>; //number

type Elements2<T> = T extends (infer U)[] ? U : T;
type test2 = Elements2<number[]>; //number

type Elements3<T, U> = T extends U[] ? U : T;
type test3 = Elements<number[]>; // 에러발생 : 2개의 인수가 필요합니다.
```

### 내장 조건부 타입들

1. Exclude<T,U> : T에 속하지만, U에는 없는 타입을 구한다.

   ```
   type A = number | string | boolean;
   type B = number | boolean;

   type C = Exclude<A, B>; // string
   ```

2. Extract<T,U> : T 타입중 U에 할당할 수 있는 타입을 구한다.

   ```
   type A = number | string | { a: boolean };
   type B = { a: boolean };

   type C = Extract<A, B>; // {a:boolean}
   ```

3. NonNullable<T> : T에서 null과 undefined를 제외한 버전을 구한다.

   ```
   type A = { a: boolean } | null | undefined;
   type C = NonNullable<A>; // {a:boolean}
   ```

4. ReturnType<F> : 함수의 반환 타입을 구한다. 단 `제네릭과 오버로드된 함수에서는 동작하지 않음`

   ```
    const testFn = () => {
      name: 'chiman';
      age: 3;
      familly: true;
    };

    type A = ReturnType<A>;

    {
      name: string;
      age: number;
      familly: boolean;
    }
   ```

5. InstanceType<C> : 클래스 생성자의 인스턴스 타입을 구한다.

   ```
   class C {
    x = 0;
    y = 0;
   }

   type T0 = InstanceType<typeof C>; // C   // 인스턴스를 만들기 위해 typeof C로 제네릭안에 담아줌
   type T1 = InstanceType<any>; // any
   type T2 = InstanceType<never>; // never

   type T3 = InstanceType<C>; // 에러   // 인스턴스가 아닌 클래스라서 에러
   type T4 = InstanceType<string>; // 에러
   type T5 = InstanceType<Function>; // 에러
   ```

### 타입 어서션

- A <: B <: C를 만족하면 타입 검사기에 B는 실제로 A거나 C라고 타입 어서션을 할 수 있다.
- 단 어떤 하나의 타입은 자신의 슈퍼타입이나 서브타입으로만 어서션 할 수 있다.
- 어서션 문법 2가지
  1. as 문법 (as문법을 더 추천) : `format(input as string)`
  2. 꺾쇠 괄호 문법 : `format(<string>input)`

```
function format(input: string) {
  return input;
}
function getUserInput(): string | number {
  return '';
}

let input = getUserInput();

format(input); // 에러발생 string | number 형식은 string형식에 할당할 수 없다

format(input as string);
format(<string>input)

```

- 두 타입 사이에 연관성이 충분하지 않아서 한 타입을 다른 타입이라고 어서션 할 수 없을 떄도 있다 => any를 통해 우회가능. 단 되도록 피해야 한다.

### Nonnull어서션 (!)

- 대상이 null | undefined가 아님을 확신하는 경우라면 `Nonnull어서션 연산자(!)`을 활용할 수 있다.
- T | null | undefined로 정의된 타입은 T로, number | string | null로 정의된 타입은 number | string으로 바꾼다.
- `단 너무 많이 사용되고 있다면 코드를 리팩토링 해야하는 징후이다` => 정말 확실할 때 사용, 최대한 지양

```
const toUpperCase = (prop: string | null | undefined) => {
  return prop.toUpperCase(); //  에러발생 : prop은 null 또는 undefined일 수 있습니다.
};

const toUpperCase2 = (prop: string | null | undefined) => {
  return prop!.toUpperCase(); //  문제 없음
};

```

### 확실한 할당 어서션 (!)

- 사용방법 : 변수 선언할때 `!:` 형태로 사용. Ex) let a!: string
- 변수를 선언했지만 값을 할당하지 않은 경우 에러가 발생한다.
- 사용할 시점에는 변수에 값이 반드시 할당 되어있다고 타입스크립트에게 알릴 수 있는데 이때 nonnull 어서션을 통해 에러를 발생시키지 않도록 할 수 있다.
- `단 너무 많이 사용되고 있다면 코드를 리팩토링 해야하는 징후이다` => 정말 확실할 때 사용, 최대한 지양

```
// 선언은 했지만 할당은 하지 않음
let userId: string;
userId.toUpperCase(); // 에러 발생 : 할당되기 전에 사용됨

let userId2!: string;
userId2.toUpperCase();

```

### 타입 브랜딩

- 타입스크립트는 구조기반 방식이지만 이름기반으로 사용할 때 사용 하는 것이 타입 브랜딩이다.
- 사용방법 : type Brand<K, T> = K & { \_\_brand: T };
- 브랜드는 컴파일 타임에만 쓰이는 구조물이다.

```
type Brand<K, T> = K & { __brand: T };

type Km = Brand<number, 'km'>;
type Mile = Brand<number, 'mile'>;

function kmToMile(km: Km) {
  return (km * 0.62) as Mile;
}

const km = 3 as Km;
const mile = kmToMile(km);
const mile2 = 5 as Mile;
kmToMile(mile2);             // 에러발생 Mile 형식은 Km 형식에 할당할 수 없음

```
