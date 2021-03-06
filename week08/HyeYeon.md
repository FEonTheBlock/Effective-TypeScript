## 아이템 46 타입 선언과 관련된 세 가지 버전 이해하기

- 타입스크립트를 사용할 때 라이브러리의 버전, 타입 선언의 버전, 타입스크립트의 버전을 모두 고려하지 않으면 의존성 오류가 발생할 수 있음.
- 실제 라이브러리와 타입 정보의 버전이 별도로 관리되는 방식의 문제점
  1. 라이브러리를 업데이트했지만 타입 선언을 업데이트하지 않으면 타입 오류 발생
  - 타입 선언을 업데이트하거나 보강 기법을 활용하여 해결
  2. 라이브러리보다 타입 선언의 버전이 최신인 경우 오류 발생
  - 라이브러리 버전을 올리거나 타입 선언의 버전을 내려 버전을 맞추어 해결
  3. 프로젝트에서 사용하는 타입스크립트 버전보다 라이브러리에서 필요로 하는 타입스크립터 버전이 최신일 때 오류 발생
  - 라이브러리 타입 선언 버전을 내리거나 declare module 선언으로 라이브러리 타입 정보를 없애 해결
  - 라이브러리에서 typesVersions를 통해 타입스크립트 버전별로 다른 타입 선언을 제공
  4. @types 의존성이 중복될 경우 문제가 발생
  - 서로 버전이 호환되도록 업데이트하여 해결
- 타입스크립트에서 의존성을 잘 관리하면 라이브러리를 올바르게 사용하는 방법을 배우는 데 도움이 되고, 생산성이 향상됨.

## 아이템 47 공개 API에 등장하는 모든 타입을 익스포트하기

- 라이브러리를 제작할 때, 프로젝트 초기에 타입 익스포트부터 작성해야 함.
- 라이브러리 제작 시 사용자를 위해 명시적으로 타입을 익스포트하는 것이 좋음.

## 아이템 48 API 주석에 TSDoc 사용하기

- 타입스크립트 언어 서비스가 JSDoc 스타일을 지원하므로 적극적으로 활용하는 것이 좋음.
  - @param과 @returns같은 일반적 규칙을 사용
- TSDoc 주석은 마크다운 형식으로 꾸며짐.
- 주석을 장황하게 쓰지 않고 간단히 요점만 언급
- 타입스크립트에서는 타입 정보가 코드에 있으므로 TSDoc에서는 JSDoc과 다르게 타입 정보를 명시하지 않음.

## 아이템 49 콜백에서 this에 대한 타입 제공하기

- 콜백 함수에서 this 바인딩을 사용할 때, this 바인딩 문제를 생성자에서 메서드에 this를 바인딩하여 해결
- 핸들러를 화살표함수로 정의하여 this 바인딩 문제를 해결
- 콜백 함수에서 this를 사용해야 한다면 this는 API의 일부가 되는 것이므로 반드시 타입 정보를 명시

## 아이템 50 오버로딩 타입보다는 조건부 타입을 사용하기

- 함수에 타입 정보를 추가할 때, 함수 오버로딩을 사용하기보다 조건부 타입을 사용하는 것이 좋음.
- 조건부 타입을 사용하면 제너릭을 사용할 때보다 반환 타입이 더 정교함.
- 오버로딩 타입이 작성하기 쉽지만, 조건부 타입은 개별 타입의 유니온으로 일반화하므로 타입이 더 정확해짐.

## 아이템 51 의존성 분리를 위해 미러 타입 사용하기

- 서로 다른 그룹의 라이브러리 사용자에게 각자가 필요한 모듈만 사용할 수 있도록 구조적 타이핑을 적용할 수 있음.
- 작성 중인 라이브러리가 의존하는 라이브러리의 구현과 무관하게 타입에만 의존한다면 필요한 선언부만 추출하여 넣는 미러링을 활용할 수 있음.
- 다른 라이브러리 타입 선언의 대부분을 추출해야 한다면 명시적으로 @types 의존성을 추가하는 것이 좋음.
- 공개한 라이브러리를 사용하는 자바스크립트 사용자가 @types 의존성을 가지지 않게 해야 함.
