# 36. 해당 분야의 용어로 타입 이름 짓기

컴퓨터 과학에서 어려운 일은 단 두 가지뿐이다. 캐시 무효화와 이름 짓기.

라는 말이 있을 정도로 이름 짓기는 타입 설계에서 중요하면서도 어려운 부분이다.  
엄선된 타입, 속성, 변수의 이름은 의도를 명확히 하고 코드와 타입의 추상화 수준을 높여 준다.  
반면, 잘못 선택한 타입 이름은 코드의 의도를 왜곡하고 잘못된 개념을 심어주게 된다.

```typescript
interface Animal {
  name: string;
  endangered: boolean;
  habitat: string;
}

const leopard = {
  name: "Snow Leopard",
  endangered: false,
  habitat: "tundra",
};
```

위 코드에는 4가지 문제점이 있다.

- name은 매우 일반적인 용어이기에 정확히 무엇을 지칭하는지 알 수 없다.
- endangered 속성이 멸종 위기를 표현하기 위해 boolean 타입을 사용한 것이 이상하다. 해당 속성의 의도를 '멸종 위기 또는 멸종'으로 생각한 것일지도 모르고 정확하게 파악하기 힘들다.
- 서식지를 나타내는 habitat 속성은 너무 범위가 넓은 string 타입일 뿐만 아니라 서식지라는 뜻 자체도 불분명하기 때문에 다른 속성들보다 훨씬 모호하다.
- 객체의 변수명이 leopard이지만, name 속성의 값은 'Snow Leopard'이다. 객체의 이름과 속성의 name이 다른 의도로 사용된 것인지 불분명하다.

위의 문제를 해결하려면, 정보가 모호하기 때문에 해당 속성을 작성한 사람에게 의도를 물어봐야 한다. 이는 매우 비효율적이다.

```typescript
interface Animal {
  commonName: string;
  genus: string;
  species: string;
  status: ConservationStatus;
  climates: KoppenClimate[];
}
```

반면, 위의 코드 타입 선언은 의미가 분명하다.

- name은 commonName, genus, species 등 더 구체적인 용어로 대체했다.
- endangered는 동물 보호 등급에 대한 IUCN의 표준 분류 체계인 ConservationStatus 타입의 status로 변경되었다.
- habitat은 기후를 뜻하는 climates로 변경되었으며, 쾨펜 기후 분류를 사용한다.

위의 타입 선언은 데이터를 훨씬 명확하게 표현하여 정보를 찾기 위해 사람에 의존할 필요가 없다.  
코드로 표현하고자 하는 모든 분야에는 주제를 설명하기 위한 전문 용어들이 있다. 자체적으로 용어를 만들어 내려 하지 말고, 해당 분야에 이미 존재하는 용어를 사용하면 타입의 명확성을 올릴 수 있다.

## 타입, 속성, 변수에 이름을 붙일 때 명심해야 할 규칙

1. 동일한 의미를 나타낼 때는 같은 용어를 사용해야 한다. 정말로 의미적으로 구분이 되어야 하는 경우에만 다른 용어를 사용해야 한다.
2. data, info, item, object 같은 모호하고 의미 없는 이름은 피해야 한다. 만약 entity라는 용어가 해당 분야에서 특별한 의미를 가진다면 괜찮지만 귀찮다고 의미 없는 이름을 붙여서는 안 된다.
3. 이름을 지을 때는 포함된 내용이나 계산 방식이 아니라 데이터 자체가 무엇인지 고려해야 한다. 예를 들어, INodeList 보다 Directory가 더 의미있는 이름이다.

좋은 이름은 추상화의 수준을 높이고 의도치 않은 충돌의 위험성을 줄여 준다.

# 37. 공식 명칭에는 상표를 붙이기

구조적 타이핑의 특성 때문에 가끔 코드가 이상한 결과를 낼 수 있다.  
이를 해결하기 위해 공식 명칭을 사용할 수 있는데, 이는 타입이 아니라 값의 관점에서 말하는 것이다.  
공식 명칭 개념을 타입스크립트에서 흉내 내려면 '상표'를 붙이면 된다.

```typescript
interface Vector2D {
  _brand: "2d";
  x: number;
  y: number;
}

function vec2D(x: number, y: number): Vector2D {
  return { x, y, _brand: "2d" };
}

function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm(vec2D(3, 4));
const vec3D = { x: 3, y: 4, z: 1 };
calculateNorm(vec3D);
// '_brand' 속성이 '{ x: number; y: number; z: number; }' 형식에 없지만 'Vector2D' 형식에서 필수입니다.
```

- vec3D 값에 `_brand: '2d'`라고 추가하는 악의적인 사용을 막을 수는 없지만 실수를 방지하기에는 충분하다.
- 상표 기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있다.
- 타입 시스템이기 때문에 런타임 오버헤드를 없앨 수 있고 추가 속성을 붙일 수 없는 string이나 number 같은 내장 타입도 상표화할 수 있다.

절대 경로를 사용해 파일 시스템에 접근하는 함수를 만들 때, 런타임에는 절대 경로로 시작하는지 체크하기 쉽지만, 타입 시스템에서는 절대 경로를 판단하기 어렵기 때문에 상표 기법을 사용하다.

```typescript
type AbsolutePath = string & { _brand: "abs" };
function listAbsolutePath(path: AbsolutePath) {
  //
}

function isAbsolutePath(path: string): path is AbsolutePath {
  return path.startsWith("/");
}

function f(path: string) {
  if (isAbsolutePath(path)) {
    listAbsolutePath(path);
  }
}
```

- string이면서 \_brand 속성을 가지는 객체를 만들 수는 없다. AbsolutePath는 온전히 타입 시스템의 영역이다.
- 만약 path 값이 절대 경로와 상대 경로 둘 다 될 수 있다면, 타입 가드와 함께 사용하면 이러한 오류를 방지할 수 있다.

# 38. any 타입은 가능한 한 좁은 범위에서만 사용하기

```typescript
function processBar(b: Bar) {
  // ...
}

function f() {
  const x = expressionReturningFoo();
  processBar(x);
  // 'Foo' 형식의 인수는 'Bar' 형식의 매개변수에 할당될 수 없습니다.
}
```

위의 오류를 제거하는 방법은 두 가지이다.

```typescript
function f1() {
  const x: any = expressionReturningFoo();
  processBar(x);
}

function f2() {
  const x = expressionReturningFoo();
  processBar(x as any);
}
```

- 두 가지 해결책 중 f2에 사용된 `x as any` 형태가 권장된다.
- 그 이유는 any 타입이 매개변수에서만 사용된 표현식이므로 다른 코드에는 영향을 미치지 않기 때문이다.
- f1에서는 함수의 마지막까지 x의 타입이 any이기에 만약 f1 함수가 x를 반환한다면 문제가 커진다.
- 이와 같이 의도치 않은 타입 안정성의 손실을 피하기 위해 any의 사용 범위를 최소한으로 좁혀야 한다.

## `@ts-ignore`

강제로 타입 오류를 제거하려면 any 대신 `@ts-ignore`를 사용하는 것이 좋다.

```typescript
function f1() {
  const x: any = expressionReturningFoo();
  // @ts-ignore
  processBar(x);
}
```

그러나 이 방법은 근본적 해결 방법이 아니기 때문에 다른 곳에서 더 큰 문제가 발생할 수도 있다.

## 큰 객체 안에서 한 개 속성이 타입 오류를 가지는 경우

```typescript
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value as any,
  },
};
```

- 객체 전채를 `as any`로 선언하면 오류를 제거할 수 있지만 다른 속성들 역시 타입 체크가 되지 않는다.
- 그러므로 최소한의 범위에만 any를 사용하는 것이 좋다.

# 39. any를 구체적으로 변형해서 사용하기

any 타입의 값을 그대로 정규식이나 함수에 넣는 것은 권장되지 않는다.

```typescript
function getLengthBad(array: any) {
  return array.length;
}

function getLength(array: any[]) {
  return array.length;
}
```

- getLengthBad보다 getLength는 세 가지 장점을 가지고 있다.
  - 함수 내의 array.length 타입이 체크된다.
  - 반환 타입이 any 대신 number로 추론된다.
  - 함수 호출될 때 매개변수가 배열인지 체크된다.

함수의 매개변수가 객체이긴 하지만 값을 알 수 없다면 `{[key: string]: any}`처럼 선언하면 된다.

## 함수의 타입 구체화

함수의 타입에도 단순히 any를 사용해서는 안 된다. 최소한으로나마 구체화할 수 있는 세 가지 방법이 있다.

```typescript
type Fn0 = () => any; // 매개변수 없이 호출 가능한 모든 함수
type Fn1 = (arg: any) => any; // 매개변수 1개
type FnN = (...args: any[]) => any; // 모든 개수의 매개 변수, Function 타입과 동일
```
