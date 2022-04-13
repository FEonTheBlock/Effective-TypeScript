## 46 | 타입 선언과 관련된 세 가지 버전 이해하기

### 실제 라이브러리와 타입 정보의 버전이 별도로 관리되는 방식의 문제점

1. 라이브러리만 업데이트하고 타입 선언은 업데이트 하지않으면 타입 오류 발생
2. 라이브러리보다 타입 선언이 최신인 경우 TS에 명시된 메서드는 최신 메서드로 표시되지만 실제 런타임에 적용되는 메서드는 이전버전의 메서드가 되어 문제 발생
3. 프로젝트에 설치된 TS버전보다 라이브러리에서 사용된 TS버전이 더 최신인 경우 `@types` 선언 자체에서 타입 오류 발생
4. 라이브러리에서 의존되는 `@types`의 버전이 맞지 않는 경우 문제 발생

### 문제가 발생하지 않기 위해 해야하는 것

- 라이브러리를 업데이트할 때 `@types`도 업데이트 한다.

> 타입선언을 라이브러리에 포함하는 것과 `DefinitelyTyped`에 공개하는 것 사이의 장단점을 이해해야한다.
>
> TS로 작성된 라이브러리라면 타입 선언을 자체적으로 포함하고,
>
> JS로 작성도니 라이브러리라면 타입 선언을 `DefinitelyTyped`에 공개하는 것이 좋다.

<br>

---

## 47 | 공개 API에 등장하는 모든 타입을 익스포트하기

- 공개 메서드에 등장한 어떤 형태의 타입이든 익스포트하는 것이 좋다.
- 어차피 라이브러리 사용자가 추출할 수 있으므로, 익스포트하기 쉽게 만드는 것이 좋다.

<br>

---

## 48 | API주석에 TSDoc 사용하기

- 익스포트된 함수, 클래스, 타입에 주석을 달 때는 JSDoc/TSDoc을 사용하자
- `@params`, `@returns` 구문과 문서 서식을 위해 마크다운을 사용할 수 있다.
- 주석에 타입 정보를 포함하면 안된다.

<br>

---

## 49 | 콜백에서 this에 대한 타입 제공하기

- `this`는 동적 바인딩되는 자기 참조 변수이므로 코드를 예측하기 어렵게 한다.
- 콜백에서 `this`를 사용해야 한다면, 타입 정보를 명시해야한다.

<br>

---

## 50 | 오버로딩 타입보다는 조건부 타입을 사용하기

```ts
function double<T extends number | string>(
  x: T
): T extends string ? string : number;

function double(x: any) {
  return x + x;
}
```

- 오버로딩을 사용한다면 작성하기는 쉽겠지만 조건에 따라 오버로딩이 너무 많아지게 된다.
- 때문에 조건부 타입을 사용하여 개별 타입의 유니온으로 일반화하면 타입도 더 정확하게 할 수 있다.
- 개인적인 생각: 오버로딩을 할 경우가 그리 많지 않다면 오히려 오버로딩을 사용하는 것이 더 좋아보인다, 코드 가독성 측면에서 조건부 타입은 조금 읽기 어려운 감이 없잖아 있다.

<br>

---

## 51 | 의존성 분리를 위해 미러 타입 사용하기

> 미러링: 필요한 선언부만 추출하여 작성 중인 라이브러리에 넣는 것

- 의존성 타입같은 경우는 라이브러리 사용자가 타입을 추출해서 사용하지 않도록 미러링을 API에 적용하여 사용하는 것이 타입의 의존성을 안전하게 유지할 수 있다.

<br>

---

## 52 | 테스팅 타입의 함정에 주의하기

> 타입 선언을 테스트하는 것은 어렵다.

- 타입 선언이 예상한 타입으로 결과를 내는지 체크할 수 있는 방법은 함수를 호출하는 테스트 파일을 작성하는 것
- 타입을 테스트할 때는 함수 타입의 동일성과 할당 가능성의 차이를 알고 있어야 한다.
- 콜백 매개변수의 추론된 타입도 체크해야하며 이때 `this`가 사용된다면 당연히 `this`도 테스트해야 한다.
- `any`의 함정에 빠지기 쉬우므로 `dtslint`같은 도구를 사용하는 것이 좋다.

<br>

---

## 53 | 타입스크립트 기능보다는 ECMAScript 기능을 사용하기

- 타입스크립트 코드에서 모든 타입 정보를 제거하면 자바스크립트가 되지만 열거형(enum), 매개변수 속성, 트리플 슬래시 임포트, 데코레이터는 타입 정보를 제거한다고 자바스크립트가 되지는 않는다 -> 즉, pure한 JS기 때문에 런타임에 영향을 준다.
- TS의 역할을 명확하게 하려면 위 목록들은 사용하지 않는 것이 좋다.

<br>

---

## 54 | 객체를 순회하는 노하우

- 간만에 JS 안녕!
- 객체를 순회할 때 키가 어떤 타입인지 정확히 파악하고 있다면 `let k: keyof T`와 `for-in`루프를 사용하자 -> `for-in`문 보다는 `Object.keys`가 더 좋다.(프로토타입까지 순회하지 않으므로)

<br>

---

## 55 | DOM 계층 구조 이해하기

- Node, Element, HTMLElement, EventTarget 그리고 Event와 Mouse Event의 차이점을 알아야 한다.
- TS가 추론하기 유용하도록 DOM 엘리먼트와 이벤트에는 충분히 구체적인 타입 정보를 사용하자

<br>

---

## 56 | 정보를 감추는 목적으로 private 사용하지 않기

- JS에서는 클래스에서 property를 숨길 수 없다.
- TS에서 사용되는 `private`키워드는 타입 시스템서만 동작할 뿐이다.
- 정보를 감추려면 클로저를 사용하자

<br>

---

## 57 | 소스맵을 사용하여 타입스크립트 디버깅하기

- 소스맵 짱짱맨
- 소스맵에 원본 코드가 그대로 포함되도록 설정되어 있을 수 있는데 공개되지 않도록 설정을 확인하자

<br>

---

## 58 | 모던 자바스크립트로 작성하기

- JS공부를 잘하자(웅모강사님 짱짱맨)

## 59 | 타입스크립트 도입 전에 @ts-check와 JSDoc으로 시험해 보기

- 파일 상단에 `@ts-check`를 추가하면 JS에서도 타입 체크를 수행할 수 있다.
- JSDoc을 잘 활용하면 JS에서도 타입 단언과 타입 추론을 할 수 있다.

<br>

---

## 60 | allowJs로 타입스크립트와 자바스크립트 같이 사용하기

- 주로 JS에서 TS로 마이그레이션할때 사용되는 `tsconfig`설정

<br>

---

## 61 | 의존성 관계에 따라 모듈 단위로 전환하기

- 마이그레이션의 첫 단계는 서드파티 모듈과 외부 API 호출에 대한 `@types`를 추가하는 것
- 의존성 관계도의 아래에서부터 위로 올라가며 마이그레이션을 하면 된다.
  - 첫번째 모듈은 보통 유틸리티 모듈이다.
  - 의존성 관계도를 시각화하여 진행 과정을 추적하는 것이 좋다.
- 이상한 설계를 발견하더라도 리팩터링을 하면 안된다.
  - 마이그레이션할때는 마이그레이션만 하자
- 마이그레이션하며 발견하게 되는 일반적인 오류들은 기록해두자

<br>

---

## 62 | 마이그레이션의 완성을 위해 noImplicitAny 설정하기

- 마이그레이션의 마지막 단계는 `noImplicitAny`를 적극 활용하는 것이다.