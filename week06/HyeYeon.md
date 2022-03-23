## 아이템 36 해당 분야의 용어로 타입 이름 짓기

- 타입 이름을 잘못 선택하면 코드의 의도를 왜곡하므로 해당 분야의 용어를 사용하여 가독성을 높이고 추상화 수준을 높여야 함.
- 타입 이름을 구체적인 용어로 하기
- 데이터를 명확하게 표현할 수 있는 이름 사용하기
- 동일한 의미를 나타낼 때는 같은 용어 사용하기

## 아이템 37 공식 명칭에는 상표를 붙이기

- 값을 세밀하게 구분하여 공식 명칭을 사용하기 위해 타입스크립트에서 상표를 붙이기
- 상표 기법을 이용하면 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 잇음.

## 아이템 38 any 타입은 가능한 한 좁은 범위에서만 사용하기

- 타입스크립트가 함수의 반환 타입을 추론할 수 있는 경우 함수의 반환 타입을 명시
- 객체 전체를 any로 단언하면 다른 속성들도 타입 체크가 되지 않으므로 최소한의 범위에만 any를 사용
- any 대신 @ts-ignore를 사용하여 타입 오류를 제거

## 아이템 39 any를 구체적으로 변형해서 사용하기

- any보다 더 구체적으로 표현할 수 있는 타입을 찾아 타입 안전성을 높여야 함
- any[][]와 같은 형태보다 {[key: string]: any}와 같이 선언하여 더 정확하게 모델링할 수 있도록 해야 함

## 아이템 40 함수 안으로 타입 단언문 감추기

- 함수 내부 로직이 복잡하면 안전한 타입으로 구현하기 어려운 경우가 있음.
- 함수 내부에는 타입 단언을 사용, 함수 외부의 타입 정의를 정확히 명시하기