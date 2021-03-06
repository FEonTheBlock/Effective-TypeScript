# 11. 잉여 속성 체크의 한계 인지하기

타입이 명시된 변수에 객체 리터럴을 할당하거나 함수에 매개변수로 전달할 때, 타입스크립트는 해당 타입의 속성이 있는지, 그리고 그 외의 속성은 없는지를 확인한다.

```typescript
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}

const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
  // 오류 - 'Room' 형식에 'elephant'가 없습니다.
};
```

- 구조적 타이핑 관점으로 생각해보면 오류가 발생하지 않아야 한다.
- 임시 변수 obj를 선언하고 Room 타입에 할당하는 것은 가능하다.

```typescript
const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};

const r: Room = obj;
// 정상
```

- obj 타입은 Room 타입의 부분 집합을 포함하므로, Room에 할당 가능하며 타입 체커도 통과한다.
- 첫 번째 예제에서는, 구조적 타입 시스템에서 발생할 수 있는 중요한 종류의 오류를 잡을 수 있도록 '잉여 속성 체크'라는 과정이 수행되었다.
- 그러나 잉여 속성 체크 역시 조건에 따라 동작하지 않는다는 한계가 있고, 통상적인 할당 가능 검사와 함께 쓰이면 구조적 타이핑이 무엇인지 혼란스러워질 수 있다.
- 잉여 속성 체크가 할당 가능 검사와는 별도의 과정이라는 것을 알아야 타입스크립트 타입 시스템에 대한 개념을 정확히 잡을 수 있다.

## 의도와 다르게 작성된 코드

```typescript
interface Options {
  title: string;
  darkMode?: boolean;
}

function createWindow(options: Options) {
  if (options.darkMode) {
    setDarkMode();
  }
}

createWindow({
  title: "Spider Solitaire",
  darkmode: true,
  // 'Options' 형식에 'darkmode'가 없습니다.
});
```

위와 같은 코드를 실행하면 런타임에 어떠한 종류의 오류도 발생하지 않는다.  
그러나 타입스크립트가 알려주는 오류 메시지처럼 의도한 대로 동작하지 않을 수 있다.  
Options 타입은 범위가 매우 넓기 때문에, 순수한 구조적 타입 체커는 이런 종류의 오류를 찾아내지 못한다.  
왜냐하면 타입스크립트 타입 시스템은 구조적 타이핑을 따르기에 string 타입인 title 속성과 '또 다른 어떤 속성'을 가지는 모든 객체는 Options 타입의 범위에 속하기 때문이다.

### 잉여 속성 체크

잉여 속성 체크를 이용하면 기본적으로 타입 시스템의 구조적 본질을 해치지 않으면서도 객체 리터럴에 알 수 없는 속성을 허용하지 않음으로써, 문제점을 방지할 수 있다.

```typescript
const intermediate = { darkmode: true, title: "Ski Free" };
const o: Options = intermediate; // 정상
```

다만 위의 예시에서 두 번째 줄은 객체 리터럴이 아니기에 잉여 속성 체크가 적용되지 않고 오류는 사라진다.

잉여 속성 체크는 타입 단언문을 사용할 때도 적용되지 않는다.  
단언문보다 선언문을 사용해야 하는 이유 중 하나이다.

잉여 속성 체크를 원치 않는다면, 인덱스 시그니처를 사용해서 타입스크립트가 추가적인 속성을 예상하도록 할 수 있다.

```typescript
interface Options {
  darkMode?: boolean;
  [otherOptions: string]: unknown;
}
const o: Options = { darkmode: true }; // 정상
```

선택적 속성만 가지는 약한 타입에도 비슷한 체크가 동작한다.

```typescript
interface LineChartOptions {
  logscale?: boolean;
  invertedYAxis?: boolean;
  areaChart?: boolean;
}

const opts = { logScale: true };
const o: LineChartOptions = opts;
// '{ logScale: boolean; }' 유형에 'LineChartOptions' 유형과 공통적인 속성이 없습니다.
```

- 구조적 관점에서 LineChartOptions 타입은 모든 속성이 선택적이므로 모든 객체를 포함할 수 있다.
- 이런 약한 타입에 대해서 타입스크립트는 값 타입과 선언 타입에 공통된 속성이 있는지 확인하는 별도의 체크를 수행한다.
- 그러나 잉여 속성 체크와 다르게 약한 타입과 관련된 할당문마다 수행되어, 임시 변수를 제거하더라도 공통 속성 체크는 여전히 동작한다.
- 잉여 속성 체크는 선택적 필드를 포함하는 Options 같은 타입에 특히 유용한 반면, 적용 범위도 매우 제한적이며 오직 객체 리터럴에만 적용된다.

# 12. 함수 표현식에 타입 적용

타입스크립트에서는 함수 표현식을 사용하는 것이 좋다. 함수 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있다는 장점이 있기 때문이다.

## 함수 타입의 장점

함수 타입의 선언은 불필요한 코드의 반복을 줄인다.

예를 들어, 사칙연산을 하는 함수를 작성할 때 반복되는 함수 시그니처를 하나의 함수 타입으로 통합할 수도 있다.

```typescript
type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a, b) => a + b;
...
```

- 만약 라이브러리를 직접 만드는 경우가 있다면, 공통 콜백 함수를 위한 타입 선언을 제공하는 것이 좋다.

시그니처가 일치하는 다른 함수가 있을 때도 함수 표현식에 타입을 적용해 볼 만하다.  
예를 들어, fetch 함수는 특정 리소스에 HTTP 요청을 보내고, response.json()를 사용해 응답의 데이터를 추출한다.  
fetch 함수의 경우 '404 Not Found'가 발생했을 때 reject 하지 않기 때문에 상태 체크를 수행해 줄 checkedFetch 함수를 선언문과 표현식 두 가지 방식으로 작성해보자

```typescript
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
  const response = await fetch(input, init);

  if (!response.ok) {
    throw new Error("Request failed: " + response.status);
  }
  return response;
}

const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init);

  if (!response.ok) {
    throw new Error("Request failed: " + response.status);
  }
  return response;
};
```

- 함수 표현식과 기존의 타입을 typeof로 적용한 경우 타입스크립트가 매개변수 타입을 추론할 수 있게 해 준다.
- 표현식 타입 구문은 checkedFetch의 반환 타입을 보장하기에 throw 대신 return을 사용했다면, 타입스크립트는 그 실수를 잡아낸다.
- 선언문으로 작성한 경우에도 return을 사용하면 오류가 발생하지만, 구현체가 아닌 함수를 호출한 위치에서 오류가 발생한다.
- 함수의 매개변수에 타입 선언을 하는 것보다 함수 표현식 전체 타입을 정의하는 것이 코드도 간결하고 안전하다.
- 다른 함수의 시그니처를 참조하려면 typeof fn을 사용하면 된다.

# 13. 타입과 인터페이스의 차이점 알기

타입스크립트에서 타입을 정의하는 방법은 타입과 인터페이스 두 가지 방법이 존재한다.  
대부분의 경우에는 타입을 사용해도 되고, 인터페이스를 사용해도 된다.  
그러나 타입과 인터페이스 사이에 존재하는 차이를 분명하게 알고, 같은 상황에서는 동일한 방법으로 명명된 타입을 정의해 일관성을 유지해야 한다.

## 비슷한 점

- 명명된 타입은 인터페이스로 정의하든 타입으로 정의하는 상태에는 차이가 없다.
  - 예를 들어, 객체 리터럴로 추가 속성과 함께 할당한다면 동일한 오류가 발생한다.
- 인덱스 시그니처는 인터페이스와 타입에서 모두 사용 가능하다.
- 함수 타입도 인터페이스와 타입에서 모두 사용 가능하다.
- 타입 별칭과 인터페이스는 모두 제너릭이 가능하다.
- 인터페이스는 타입을 확장할 수 있고, 타입은 인터페이스를 확장할 수 있다.

```typescript
interface IStateWithPop extends TState {
  population: number;
}
type TStateWithPop = IState & { population: number };
```

    * IStateWithPop과 TStateWithPop는 동일하다.
    * 여기서 주의할 점은 인터페이스는 유니온 타입 같은 복잡한 타입을 확장하지는 못한다는 것이다.
    * 복잡한 타입을 확장하고 싶다면 타입과 &를 사용해야 한다.

- 클래스를 구현(implements)할 때는, 타입과 인터페이스를 둘 다 사용할 수 있다.

## 다른 점

- 인터페이스는 타입을 확장할 수 있지만, 유니온은 할 수 없다.
- 튜플과 배열 타입도 type 키워드를 이용해 더 간결하게 표현할 수 있다.

```typescript
type Pair = [number, number];
type StringList = string[];

// 인터페이스로 구현 시
interface Tuple {
  0: number;
  1: number;
  length: 2;
}
```

- 그러나 인터페이스로 튜플과 비슷하게 구현하면 튜플에서 사용할 수 있는 concat 같은 메서드들을 사용할 수 없기에 type 키워드로 구현하는 것이 낫다.
- 인터페이스는 타입에 없는 기능이 있는데, 그 중 하나는 보강(augment)이 가능하다는 것이다.

```typescript
interface IState {
  name: string;
  capital: string;
}

interface IState {
  population: number;
}

const wyoming: IState = {
  name: "Wyoming",
  capital: "Cheyenne",
  population: 500,
};
```

- 이처럼 속성을 확장하는 것을 선언 병합이라고 한다. 선언 병합은 주로 타입 선언 파일에서 사용된다.
- 타입스크립트는 여러 버전의 자바스크립트 표준 라이브러리에서 여러 타입을 모아 병합한다.
  - 예를 들어, Array 인터페이스는 `lib.es5.d.ts`에 정의되어 있고 기본적으로 이에 선언된 인터페이스가 사용된다.
  - 그러나 `tsconfig.json`의 lib 목록에 ES2015를 추가하면 타입스크립트는 이에 선언된 인터페이스를 병합한다.
  - 결과적으로 각 선언이 병합되어 전체 메서드를 가지는 하나의 Array 타입을 얻게 된다.

## 결론

- 복잡한 타입이라면 타입 별칭을 사용
- 간단한 객체 타입이라면 일관성과 보강의 관점에서 고려해 볼 것
  - 일관되게 코드베이스에서 작업 중인 문법을 사용
  - 아직 스타일이 확립되지 않은 프로젝트라면 향후 보강 가능성을 고려해 볼 것
  - 어떤 API에 대한 타입 선언을 작성해야 한다면 인터페이스를 사용하는 것이 좋음
  - API가 변경될 때 사용자가 인터페이스를 통해 새로운 필드를 병합 할 수 있어 유용하기 때문
- 프로젝트 내부적으로 사용되는 타입에 선언 병합이 발생하는 것은 잘못된 설계

# 14. 타입 연산과 제너릭 사용으로 반복 줄이기

**DRY 원칙**
don't repeat yourself - 같은 코드를 반복하지 말라는 원칙

타입 중복은 코드 중복만큼 많은 문제를 발생시킨다.  
타입 간에 매핑하는 방법을 익히면, 타입 정의에서도 DRY의 장점을 적용할 수 있다.

- 반복을 줄이는 가장 간단한 방법으로는 타입에 이름을 붙이는 것이다.
- extends를 사용해서 인터페이스의 반복을 피할 수 있다.

## 중복 제거 예제

전체 애플리케이션의 상태를 표현하는 State 타입과 단지 부분만 표현하는 TopNavState가 있는 경우

```typescript
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

interface TopNavState {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
}
```

TopNavState를 확장하여 State를 구성하기보다, State의 부분 집합으로 TopNavState를 정의하는 것이 바람직하다.

### State 인덱싱

State를 인덱싱 하여 속성의 타입에서 중복을 제거할 수 있다.

```typescript
interface TopNavState {
  userId: State["userId"];
  pageTitle: State["pageTitle"];
  recentFiles: State["recentFiles"];
}
```

이렇게 구현하면 State 내의 타입이 바뀌어도 TopNavState에도 반영된다.

### 맵드 타입

반복되는 코드를 더 줄이기 위해 맵드 타입을 사용하면 좀 더 나아진다.

```typescript
type TopNavState = {
  [k in "userId" | "pageTitle" | "recentFiles"]: State[k];
};
```

맵드 타입은 배열의 필드를 루프 도는 것과 같은 방식이다.

### Pick

이 패턴은 표준 라이브러리에서도 찾을 수 있으며, Pick이라고 한다.

```typescript
type TopNavState = Pick<State, "userId" | "pageTitle" | "recentFiles">;
```

제네릭 타입으로 함수에서 매개변수를 받아 결괏값을 반환하는 것처럼 T와 K 두 가지 타입을 받아서 결과 타입을 반환한다.

### 태그된 유니온

```typescript
interface SaveAction {
  type: "save";
}

interface LoadAction {
  type: "load";
}

type Action = SaveAction | LoadAction;
type ActionType = Action["type"];
```

Action 유니온을 인덱싱하면 타입 반복 없이 ActionType을 정의할 수 있다.

### keyof

생성하고 난 다음 업데이트가 되는 클래스를 정의한다면, update 메서드 매개변수의 타입은 생성자와 동일한 매개변수이면서, 타입 대부분이 선택적 필드가 된다.  
이때 매핑된 타입과 keyof를 사용하면 모든 타입을 선택적 필드로 만들 수 있다.

```typescript
type OptionsUpdate = {
  [k in keyof Options]?: Options[k];
};
```

표준 라이브러리에 Partial이라는 이름으로 포함되어 있기도 하다.

### typeof

값의 형태에 해당하는 타입을 정의하고 싶다면 typeof를 사용하면 된다.

```typescript
const INIT_OPTIONS = {
  width: 640,
  height: 480,
  color: "#00FF00",
  label: "VGA",
};

type Options = typeof INIT_OPTIONS;
```

- 자바스크립트의 런타임 연산자 typeof를 사용한 것처럼 보이지만, 실제로는 타입스크립트 단계에서 연산되며 훨씬 더 정확하게 타입을 표현한다.
- 값으로부터 타입을 만들어 낼 때는 선언의 순서에 주의해야 한다.
  - 타입 정의를 먼저하고 값이 그 타입에 할당 가능하다고 선언하는 것이 좋다.
  - 이렇게 해야 타입이 더 명확해지고, 예상하기 어려운 타입 변동을 방지할 수 있다.

### ReturnType

함수나 메서드의 반환 값에 명명된 타입을 만들고 싶은 경우 ReturnType 제너릭을 사용하면 된다.

```typescript
type UserInfo = ReturnType<typeof getUserInfo>;
```

함수의 타입을 가져와서 리턴 값의 타입을 반환한다.

### 제너릭 타입 매개변수 제한

제너릭 타입에서 매개변수를 제한할 수 있는 방법은 extends를 사용하는 것이다.  
extends를 이용하면 제너릭 매개변수가 특정 타입을 확장한다고 선언할 수 있다.

```typescript
interface Name {
  first: string;
  last: string;
}
type DancingDuo<T extends Name> = [T, T];

const couple1: DancingDuo<Name> = [
  { first: "Fred", last: "Astaire" },
  { first: "Jeremy", last: "Kim" },
]; // OK
```

- 현재 타입스크립트에서는 선언부에 항상 제너릭 매개변수를 작성하도록 되어 있다.
- 타입스크립트가 제너릭 매개변수의 타입을 추론하게 하기 위해, 함수를 작성할 때는 신중하게 타입을 고려해야 한다.

```typescript
const dancingDuo = <T extends Name>(x: DancingDuo<T>) => x;
```

### extends keyof

Pick의 정의는 extends를 사용해서 완성할 수 있다.

```typescript
type Pick<T, K extends keyof T> = {
  [k in K]: T[k];
};
```

- K의 범위를 설정하지 않으면 너무 넓고, 인덱스로 사용될 수 있는 `string | number | symbol`에 속해야 한다.
- K는 실제로는 T의 키의 부분집합이 되어야 한다.

# 15. 동적 데이터에 인덱스 시그니처 사용

타입스크립트에서는 타입에 인덱스 시그니처를 명시하여 유연하게 매핑을 표현할 수 있다.

```typescript
type Rocket = {
  [property: string]: string;
};

const rocket: Rocket = {
  name: "Falcon 9",
  variant: "v1.0",
  thrust: "4,940 kN",
};
```

- `[property: string]: string`이 인덱스 시그니처이며, 다음 세 가지 의미를 담고 있다.
- 키의 이름 - 키의 위치만 표시하는 용도로 타입 체커에서는 사용하지 않는다.
- 키의 타입 - string이나 number 또는 symbol의 조합이어야 하지만, 보통은 string을 사용한다.
- 값의 타입 - 어떤 것이든 될 수 있다.

## 인덱스 시그니처 타입 체크 단점

- 잘못된 키를 포함해 모든 키를 허용한다.
- 특정 키가 필요하지 않다. {}도 유효한 Rocket 타입이다.
- 키마다 다른 타입을 가질 수 없다.
- 키는 무엇이든 가능하기 때문에 자동 완성 기능 같은 언어 서비스를 받을 수 없다.

-> 인덱스 시그니처는 부정확하므로 더 나은 방법을 찾아야 한다. 즉, 가능하다면 인터페이스, Record, 매핑된 타입 같은 인덱스 시그니처보다 정확한 타입을 사용하고 런타임 때까지 객체의 속성을 알 수 없을 경우에만 인덱스 시그니처를 사용하도록 한다.

### 예시

CSV 파일처럼 헤더 행에 열 이름이 있고, 데이터 행을 열 이름과 같은 값으로 매핑하는 객체로 나타내고 싶은 경우

```typescript
function parseCSV(input: string): { [columnName: string]: string }[] {
  const lines = input.split("\n");
  const [header, ...rows] = lines;
  const headerColumns = header.split(",");
  return rows.map((rowStr) => {
    const row: { [columnName: string]: string } = {};
    rowStr.split(",").forEach((cell, i) => {
      row[headerColumns[i]] = cell;
    });
    return row;
  });
}
```

일반적인 상황에서 열 이름이 무엇인지 미리 알 방법은 없다.  
이럴 때는 인덱스 시그니처를 사용한다.

선언해 둔 열들이 런타임에 실제로 일치한다는 보장이 없기에 이를 안전하게 접근하기 위해서는 인덱스 시그니처 값 타입에 undefined를 추가하는 것을 고려해야 한다.

```typescript
function safeParseCSV(
  input: string
): { [columnName: string]: string | undefined }[] {
  return parseCSV(input);
}
```

체크를 추가해야 하기에 작업이 조금 번거로울 수 있다. undefined를 타입에 추가할지는 상황에 맞게 판단해야 한다.

타입에 가능한 필드가 제한되어 있는 경우에는 인덱스 시그니처로 모델링하지 말아야 한다.  
예를 들어 데이터에 A, B, C, D같은 키가 있지만, 얼마나 많이 있는 지 모른다면 선택적 필드 또는 유니온 타입으로 모델링하면 된다.  
유니온 타입은 정확하지만 사용하기 번거롭기에 선택적 필드로 모델링 하는 것이 최선이다.

## 인덱스 시그니처를 사용하기에 string 타입이 너무 광범위한 경우

### Record

키 타입에 유연성을 제공하는 제너릭 타입. string의 부분 집합을 사용할 수 있다.

```typescript
type Vec3D = Record<"x" | "y" | "z", number>;
// type Vec3D = {
//   x: number;
//   y: number;
//   z: number;
// };
```

### 맵드 타입

맵드 타입은 키마다 별도의 타입을 사용하게 해 준다.

```typescript
type ABC = { [k in "a" | "b" | "c"]: k extends "b" ? string : number };
// type ABC = {
//   a: number;
//   b: string;
//   c: number;
// };
```

# 16. number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용

자바스크립트에서 객체의 키에 숫자는 사용할 수 없다. 숫자를 사용하려고 하면, 자바스크립트 런타임은 문자열로 변환할 것이다.  
타입스크립트는 이러한 혼란을 바로잡기 위해 숫자 키를 허용하고, 문자열 키와 다른 것으로 인식한다.

- 런타임에는 문자열 키로 인식하므로 이는 가상이라고 할 수 있지만, 타입 체크 시점에 오류를 잡을 수 있어서 유용하다.
- 배열을 순회할 때 인덱스에 신경 쓰지 않는다면 for-of를, 인덱스 타입이 중요하다면 Array.prototype.forEach를 사용하면 된다.
- 루프 중간에 멈춰야 한다면 for 루프를 사용하는 것이 좋다.
- 인덱스 시그니처에 number를 사용하기보다 Array나 튜플, ArrayLike를 사용하는 것이 좋다.

# 17. 변경 관련된 오류 방지를 위해 readonly 사용하기

readonly 접근 제어자를 사용하면 변경 불가한 타입을 선언할 수 있다.  
readonly로 배열을 선언하면 일반 배열과 구분되는 몇 가지 특징이 있다.

- 배열의 요소를 읽을 수 있지만, 쓸 수는 없다.
- length를 읽을 수 있지만, 바꿀 수는 없다.
- 배열을 변경하는 pop을 비롯한 다른 메서드를 호출할 수 없다.

`number[]`는 `readonly number[]`보다 기능이 많기 때문에, `readonly number[]`의 서브타입이 된다. 따라서 변경 가능한 배열을 readonly 배열에 할당할 수 있다. 하지만 그 반대는 불가능하다.

## 매개변수에서의 readonly

매개변수를 readonly로 선언하면 다음과 같은 일이 생긴다.

- 타입스크립트는 매개변수가 함수 내에서 변경이 일어나는지 체크한다.
- 호출하는 쪽에서는 함수가 매개변수를 변경하지 않는다는 보장을 받게 된다.
- 호출하는 쪽에서 함수에 readonly 배열을 매개변수로 넣을 수도 있다.

만약 함수가 매개변수를 변경하지 않는다면, readonly로 선언해야 한다.  
더 넓은 타입으로 호출할 수 있고, 의도치 않은 변경은 방지될 것이다.  
다른 라이브러리에 있는 readonly 함수를 호출하는 경우라면, 타입 선언을 바꿀 수 없으므로 타입 단언문을 사용해야 한다.

## readonly를 사용하여 지역 변수 변경 오류 방지

소설의 연속된 행을 가져와서 빈 줄을 기준으로 구분되는 단락으로 나누는 기능을 하는 프로그램을 작성했다고 가정해보자

```typescript
function parseTaggedText(lines: string[]): string[][] {
  const paragraphs: string[][] = [];
  const currPara: string[] = [];

  const addParagraph = () => {
    if (currPara.length) {
      paragraphs.push(currPara);
      currPara.length = 0;
    }
  };

  for (const line of lines) {
    if (!line) {
      addParagraph();
    } else {
      currPara.push(line);
    }
  }
  addParagraph();
  return paragraphs;
}
// [ [], [], [] ] - 의도와 다른 출력
```

위와 같이 출력되는 이유는 currPara에 새 값을 채우거나 지운다면 동일한 객체를 참조하고 있는 paragraphs 요소에도 변경이 반영되기 때문이다.  
currPara를 readonly로 선언하면 이런 동작을 방지할 수 있다.  
선언을 바꾸는 즉시 코드 내에서 몇 가지 오류가 발생한다.

```typescript
function parseTaggedText(lines: string[]): string[][] {
  const paragraphs: string[][] = [];
  const currPara: string[] = [];

  const addParagraph = () => {
    if (currPara.length) {
      paragraphs.push(currPara);
      // 'readonly string[]' 형식의 인수는 'string[]' 형식의 매개 변수에 할당될 수 없습니다.
      currPara.length = 0;
      // 읽기 전용 속성이므로 'length'에 할당할 수 없습니다.
    }
  };

  for (const line of lines) {
    if (!line) {
      addParagraph();
    } else {
      currPara.push(line);
      // 'readonly string[]' 형식에 'push' 속성이 없습니다.
    }
  }
  addParagraph();
  return paragraphs;
}
```

currPara를 let으로 선언하고 변환이 없는 메서드를 사용함으로써 두 개의 오류를 고칠 수 있다.

```typescript
let currPara: readonly string[] = [];

currPara = [];

currPara = currPara.concat([line]);
```

concat은 원본을 수정하지 않고 새 배열을 반환한다.

paragraphs 오류를 바로잡는 방법은 세 가지이다.

1. currPara의 복사본을 만든다.
   `paragraphs.push([...curPara]);` \* currPara는 readonly로 유지되지만, 복사본은 원하는 대로 변경이 가능하기 때문에 오류가 사라진다.
2. paragraphs(그리고 함수의 반환 타입)를 readonly string[]의 배열로 변경한다.
   `const paragraphs: (readonly string[])[] = [];`
   _ 여기서 괄호가 중요한데, `readonly string[][]`은 readonly 배열의 변경 가능한 배열이 아니라 변경 가능한 배열의 readonly 배열이기 때문이다.
   _ 이 코드는 동작하지만, readonly를 반환하기 때문에 함수가 반환한 값에 대해 영향을 끼치는 것이 맞는 방법인지 고민해 봐야 한다.
3. 배열의 readonly 속성을 제거하기 위해 단언문을 사용한다.
   `paragraphs.push(currPara as string[]);` \* 바로 다음 문장에서 currPara를 새 배열에 할당하므로, 매우 공격적인 단언문은 아닌 것으로 보인다.

## 얕게 동작하는 readonly

readonly는 얕게 동작한다는 것에 유의하며 사용해야 한다.  
객체에 사용되는 Readonly 제너릭도 마찬가지로 얕게 동작한다.  
현재 시점에는 deep readonly 타입이 기본적으로 지원되지 않지만, 제너릭을 만들면 사용할 수 있다.  
그러나 제너릭은 만들기 까다롭기 때문에 ts-essentials에 있는 DeepReadonly 제너릭과 같은 라이브러리를 사용하는 것이 낫다.

# 18. 매핑된 타입을 사용하여 값을 동기화하기

산점도(scatter plot)를 그리기 위한 UI 컴포넌트를 작성한다고 가정해보자.  
여기에는 디스플레이와 동작을 제어하기 위한 몇 가지 다른 타입의 속성이 포함된다.

```typescript
interface ScatterProps {
  xs: number[];
  ys: number[];

  xRange: [number, number];
  yRange: [number, number];
  color: string;

  onClick: (x: number, y: number, index: number) => void;
}
```

- 불필요한 작업을 피하기 위해, 필요한 때에만 차트를 다시 그릴 수 있다.
- 다른 속성이 변경되면 다시 그려야 하지만, 이벤트 핸들러가 변경되면 다시 그릴 필요가 없다.

```typescript
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;

  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== "onClick") return true;
    }
  }
  return false;
}
```

- 이와 같이 코드를 작성하면 interface에 다른 이벤트 핸들러가 추가된 경우 값이 변경될 때마다 차트를 다시 그린다.
- 이렇게 처리하는 것을 보수접 접근법이라고 한다. 또는 실패에 닫힌 접근법이라고 한다.
- 이 접근법을 이용하면 차트가 정확하지만 너무 자주 그려질 가능성이 있다.

```typescript
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  return (
    oldProps.xs !== newProps.xs ||
    oldProps.ys !== newProps.ys ||
    oldProps.xRange !== newProps.xRange ||
    oldProps.yRange !== newProps.yRange ||
    oldProps.color !== newProps.color
  );
}
```

- 속성이 추가되었을 때 차트를 불필요하게 다시 그리는 단점을 해결했다.
- 이렇게 처리하는 방법을 실패에 열린 접근법이라고 한다.
- 다만 속성이 추가되었을 때 값이 변경되면 실제로 차트를 다시 그려야 할 경우 누락되는 일이 생길 수 있다.
- 이러면 속성이 추가될 때마다 ScatterProps 인터페이스를 수정해야 하는데 이는 번거로운 일이다.

```typescript
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
};

function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;

  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      return true;
    }
  }
  return false;
}
```

- 타입 체커가 동작하도록 개선한 코드이다.
- `[k in keyof ScatterProps]: boolean`는 타입 체커에게 ScatterProps와 동일한 속성을 가져야 한다는 정보를 제공한다.
- 이와 같이 코드를 작성하면 나중에 ScatterProps에 새로운 속성을 추가하는 경우 REQUIRES_UPDATE에 해당 속성이 없는 것을 개발자에게 알려주어 예상치 못한 오류를 발생시킬 확률을 감소시킬 것이다.
- 이와 같이 매핑된 타입은 한 객체가 다른 객체와 정확히 같은 속성을 가지게 할 때 이상적이다.

# 19. 추론 가능한 타입을 사용해 장황한 코드 방지

코드의 모든 변수에 타입을 선언하는 것은 비생산적이며 좋지 않다.  
타입 추론이 된다면 명시적 타입 구문은 필요하지 않다. 오히려 방해가 되고, 이를 지양하는 것이 좋다.

- 비구조화 할당문은 모든 지역 변수의 타입이 추론되도록 한다.
- 함수 내에서 생성된 지역 변수에는 타입 구문을 넣지 않도록 한다. 타입 구문을 생략하여 코드를 읽는 사람이 구현 로직에 집중할 수 있게 하는 것이 좋다.
- 보통 타입 정보가 있는 라이브러리에서, 콜백 함수의 매개변수 타입은 자동으로 추론된다.

## 타입을 명시해야 하는 상황

1. 객체 리터럴을 정의할 때
   - 객체 리터럴 정의에 타입을 명시하면, 잉여 속성 체크가 동작한다.
   - 잉여 속성 체크는 변수가 사용되는 순간이 아닌 할당되는 시점에 오류를 표시해주고, 오류를 잡는 데 효과적이다.
2. 함수의 반환
   - 타입 추론이 가능할지라도 구현상의 오류가 함수를 호출한 곳까지 영향을 미치지 않도록 하기 위해 타입 구문을 명시하는 게 좋다.
   - 반환 타입을 명시하면, 구현상의 오류가 사용자 코드의 오류로 표시되지 않는다.
   - 또한 함수에 대해 더욱 명확하게 알 수 있고, 명명된 타입으로 더욱 직관적인 표현이 된다.

# 20. 다른 타입에는 다른 변수 사용하기

```typescript
let id = "12-34-56";
fetchProduct(id);
id = 123456;
// 'number' 형식은 'string' 형식에 할당할 수 없습니다.
```

자바스크립트에서는 서로 다른 타입의 값을 하나의 변수에 재할당해도 문제가 없다.  
하지만 타입스크립트에서는 처음에 변수의 타입을 string으로 추론하기 때문에 number 값을 재할당하면 문제가 발생한다.

-> 변수의 값은 바귈 수 있지만 그 타입은 보통 바뀌지 않는다.

타입을 바꿀 수 있는 한 가지 방법은 범위를 좁히는 것인데, 타입을 더 작게 제한하는 것이다.  
예를 들어, id 변수를 string과 number의 유니온 타입으로 확장하는 것인데, 이보다는 별도의 변수를 도입하는 것이 낫다.  
그 이유는,

- 서로 관련이 없는 두 개의 값을 분리한다.
- 변수명을 더 구체적으로 지을 수 있다.
- 타입 추론을 향상시키며, 타입 구문이 불필요해진다.
- 타입이 유니온보다 더 간결해진다.
- let 대신 const로 변수를 선언하게 된다.

-> 즉, 타입이 바뀌는 변수는 피해야 하며, 목적이 다른 곳에는 별도의 변수명을 사용해야 한다.
