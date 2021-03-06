# 타입 주변에 null 값 배치하기

strictNullChecks 설정을 하면 null이나 undefined 값 관련된 오류들이 나타나게 된다.  
이 때, 값이 전부 null이거나 전부 null이 아닌 경우로 분명히 구분된다면, 값이 섞여 있을 때보다 다루기 쉽다.

```typescript
function extent(nums: number[]) {
  let min, max;

  for (const num of nums) {
    if (!min) {
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num);
    }
  }
  return [min, max];
}
```

- 예를 들어, 이 코드에는 버그와 함께 설계적 결함이 존재한다.
- 최솟값이나 최댓값이 0인 경우, 값이 덧씌워져 버린다. 예를 들어, `extent([0, 1, 2])`의 결과는 `[1, 2]`가 된다.
- nums 배열이 비어 있다면 함수는 `[undefined, undefined]`를 반환한다.
- undefined를 포함하는 객체는 다루기 어렵고 권장하지 않는다. 코드를 살펴보면 min과 max가 동시에 undefined이거나 둘다 아니라는 것을 알 수 있지만, 이러한 정보는 타입 시스템에서 표현할 수 없다.
- strictNullChecks 설정을 하면 extent의 반환 타입이 `(number | undefined)[]`로 추론되어서 설계적 결함이 분명해진다.
- 이에 대한 적절한 해법은 min과 max를 한 객체 안에 넣고 null이거나 null이 아니게 하면 된다.

```typescript
function extent(nums: number[]) {
  let result: [number, number] | null = null;

  for (const num of nums) {
    if (!result) {
      result = [num, num];
    } else {
      result = [Math.min(result[0], num), Math.max(result[1], num)];
    }
  }
  return result;
}
```

- 이제는 반환 타입이 `[number, number] | null`이 되어 사용하기 수월해졌다.
- null 아님 단언을 사용하면 min과 max를 얻을 수 있다.
- extent 결과값으로 단일 객체를 사용함으로써 설계를 개선했고, 타입스크립트가 null 값 사이의 관계를 이해할 수 있도록 했으며 버그를 제거했다.

## 클래스 작성 시

클래스를 만들 때는 필요한 모든 값이 준비되었을 때 생성하여 null이 존재하지 않도록 하는 것이 좋다.

```typescript
class UserPosts {
  user: UserInfo;
  posts: Post[];

  constructor(user: UserInfo, posts: Post[]) {
    this.user = user;
    this.posts = posts;
  }

  static async init(userId: string): Promise<UserPosts> {
    const [user, posts] = await Promise.all([
      fetchUser(userId),
      fetchPostsForUser(userId),
    ]);

    return new UserPosts(user, posts);
  }
}
```

- 이와 같이 코드를 작성하면 클래스는 완전히 null이 아니게 되고, 메서드를 작성하기 쉬워진다.
- null인 경우가 필요한 속성은 프로미스로 바꾸면 안 된다. 코드가 매우 복잡해지며 모든 메서드가 비동기로 바뀌어야 한다.

# 유니온의 인터페이스보다는 인터페이스의 유니온 사용하기

유니온 타입의 속성을 가지는 인터페이스를 작성 중이라면, 인터페이스의 유니온 타입을 사용하는 게 더 알맞지 않을지 검토해 봐야 한다.

```typescript
interface Layer {
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

- layout 속성은 모양이 그려지는 방법과 위치를 제어하고, paint 속성은 스타일을 제어한다.
- layout이 LineLayout 타입이면서 paint 속성이 FillPaint 타입인 것은 무효한 상태이다.
- 더 나은 방법으로 모델링하려면 각각 타입의 계층을 분리된 인터페이스로 둬야 한다.

```typescript
interface FillLayer {
  layout: FillLayout;
  paint: FillPaint;
}

interface LineLayer {
  layout: LineLayout;
  paint: LinePaint;
}

interface PointLayer {
  layout: PointLayout;
  paint: PointPaint;
}

type Layer = FillLayer | LineLayer | PointLayer;
```

- 이런 형태로 Layer를 정의하면 layout과 paint 속성이 잘못된 조합으로 섞이는 경우를 방지할 수 있다.
- 이러한 패턴의 가장 일반적인 예시는 태그된 유니온이다.
- type 속성을 태그로 두어 태그를 참고하여 Layer의 타입 범위를 좁힐 수 있다.
- 어떤 데이터 타입을 태그된 유니온으로 표현할 수 있다면, 보통 그렇게 하는 것이 좋다.

## 여러 개의 선택적 필드가 동시에 값이 있거나, 없어야 하는 경우

```typescript
interface Person {
  name: string;
  // 다음은 둘 다 동시에 있거나 동시에 없다.
  placeOfBirth?: string;
  dateOfBirth?: Date;
}
```

- 타입 정보를 담고 있는 주석은 문제가 될 소지가 매우 높다.
- 두 선택적 필드는 실제로 관련되어 있지만, 타입 정보에는 어떤 관계도 표현되지 않았다.
- 두 개의 속성을 하나의 객체로 모으는 것이 더 좋은 설계이다.

```typescript
interface Person {
  name: string;
  birth?: {
    placeOfBirth: string;
    dateOfBirth: Date;
  };
}
```

- 이제 둘 중 하나만 있는 경우 오류가 발생한다.

## 타입의 구조를 손 댈 수 없는 상황 (ex - API 결과)

앞서 다룬 인터페이스의 유니온을 사용해서 속성 사이의 관계를 모델링 할 수 있다.

```typescript
interface Name {
  name: string;
}

interface PersonWithBirth extends Name {
  placeOfBirth: string;
  dateOfBirth: Date;
}

type Person = Name | PersonWithBirth;
```

- 유니온 타입의 속성을 여러 개 가지는 인터페이스에서는 속성 간의 관계가 분명하지 않기 때문에 실수가 자주 발생하므로 주의해야 한다.
- 유니온의 인터페이스보다 인터페이스의 유니온이 더 정확하고 타입스크립트가 이해하기도 좋다.
- 태그된 유니온은 타입스크립트와 매우 잘 맞기 때문에 자주 볼 수 있는 패턴이다.

# string 타입보다 더 구체적인 타입 사용하기

string 타입의 범위는 매우 넓기에 string 타입으로 변수를 선언하려 한다면, 그보다 더 좁은 타입이 적절하지는 않을지 검토해 보어야 한다.

## 값을 몇 개의 고정값으로만 받을 경우

예를 들어 Album 인터페이스에서 recordingType을 'live'와 'studio' 두 개의 값으로 유니온 타입을 정의해야 한다고 가정해보자.  
이때 enum을 사용할 수도 있지만 일반적으로 추천하지 않는다고 한다.

```typescript
type RecordingType = "studio" | "live";

interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordingType;
}
```

- 이와 같이 코드를 작성하면 타입스크립트는 오류를 더 세밀하게 체크한다.
- 이러한 방식에는 세 가지 장점이 더 있다.

1. 타입을 명시적으로 정의함으로써 다른 곳으로 값이 전달되어도 타입 정보가 유지된다.
2. 타입을 명시적으로 정의하고 해당 타입의 의미를 설명하는 주석을 붙여 넣을 수 있다.
3. keyof 연산자로 더욱 세밀하게 객체의 속성 체크가 가능해진다.

```typescript
/** 주석 내용 */
type RecordingType = "studio" | "live";
```

## 객체의 속성 이름을 함수 매개변수로 받을 때

어떤 배열에서 한 필드의 값만 추출하는 함수를 작성한다고 가정해보자.

```typescript
function pluck<T>(records: T[], key: keyof T): T[keyof T][] {
  return records.map((r) => r[key]);
}
```

- 제너릭과 keyof를 활용하여 위와 같이 코드를 작성했다.
- 그런데 key의 값으로 하나의 문자열을 넣게 되면, 그 범위가 너무 넓어서 적절한 타입이라고 보기 어렵다.
- 예를들어 releaseDate의 타입은 `(string | Date)[]`가 아니라 `Date[]`이어야 한다. 이는 string에 비해 훨씬 범위가 좁기는 하지만 그래도 여전히 넓다.
- 범위를 더 좁히기 위해 keyof T의 부분집합으로 두 번째 제너릭 매개변수를 도입해야 한다.

```typescript
function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
  return records.map((r) => r[key]);
}
```

이제 타입 시그니처가 완벽해지고, 매개변수 타입이 정밀해진 덕에 언어 서비스는 키에 자동 완성 기능을 제공할 수 있게 해 준다.

# 부정확한 타입보다는 미완성 타입을 사용하기

일반적으로는 타입이 구체적일수록 버그를 더 많이 잡고 타입스크립트가 제공하는 도구를 활용할 수 있게 된다.  
그러나 타입 선언의 정밀도를 높이는 일에는 주의를 기울여야 한다. 실수가 발생하기 쉽고 잘못된 타입은 차라리 없는 것보다 못할 수 있다.  
타입 선언을 세밀하게 만들 때 너무 과하게 하는 경우 오히려 타입이 부정확해져서 사용자들에게 불편함을 줄 수 있다.

일반적으로 복잡한 코드는 더 많은 테스트가 필요하고 타입의 관점에서도 마찬가지이다.  
타입을 정제할 때 any 같은 매우 추상적인 타입은 정제하는 것이 좋다. 그러나 타입이 구체적으로 정제된다고 해서 정확도가 무조건 올라가지는 않는다.

- 정확하게 타입을 모델링할 수 없다면, 부정확하게 모델링하지 말아야 한다. 또한 any와 unknown을 구별해서 사용해야 한다.
  - any는 타입 검사를 느슨하게 하므로 추후에 개발 후 예기치 못한 문제가 발생할 가능성이 매우 높다.
  - unknown은 any 타입과는 다르게 프로퍼티 또는 연산을 하는 경우 컴파일러가 체크하여 문제 되는 코드를 미리 예방할 수 있다.
- 타입 정보를 구체적으로 만들수록 오류 메시지와 자동 완성 기능에 주의를 기울여야 한다. 정확도뿐만 아니라 개발 경험과도 관련된다.

# 데이터가 아닌, API와 명세를 보고 타입 만들기

- 코드의 구석 구석까지 타입 안정성을 얻기 위해 API 또는 데이터 형식에 대한 타입 생성을 고려해야 한다.
- 데이터에 드러나지 않는 예외적인 경우들이 문제가 될 수 있기 때문에 데이터보다는 명세로부터 코드를 생성하는 것이 좋다.
- 명세를 기반으로 타입을 작성한다면 현재까지 경험한 데이터뿐만 아니라 사용 가능한 모든 값에 대해서 작동한다는 확신을 가질 수 있다.
- API의 명세로부터 타입을 생성할 수 있다면 그렇게 하는 것이 좋다. 특히 GraphQL처럼 자체적으로 타입이 정의된 API에서 잘 동작한다.
