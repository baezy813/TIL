# Interface vs Type

둘 다 TypeScript에서 값의 모양(구조)을 적는 방법이다. 객체 프로퍼티, 함수 시그니처 등을 설명할 때 쓴다. 문법만 다르고 겹치는 부분이 많다.

차이는 어디까지 표현할 수 있나와  같은 이름으로 여러 번 선언할 수 있나에 있다!

---

## 한 줄 요약

| | `interface` | `type` |
|---|-------------|--------|
| 주 용도 | **객체 형태**를 이름 붙여 정의 | **모든 종류의 타입** 별칭 (객체, 유니온, 튜플 등) |
| 같은 이름 재선언 | **가능** (선언이 합쳐짐) | **불가능** (이름 충돌) |
| `extends` | 다른 `interface` / `class` 확장 | 교차 타입 `&` 등으로 조합 |

---

## 공통으로 할 수 있는 것

둘 다 객체 모양을 이렇게 쓸 수 있다.

```ts
interface User {
  id: number;
  name: string;
}

type User2 = {
  id: number;
  name: string;
};
```

둘 다 `implements`에 쓸 수 있고 제네릭도 비슷하게 붙인다.

---

## `interface`만의 특징

### 1. 선언 병합(Declaration Merging)

같은 이름의 `interface`를 **여러 파일/여러 번** 선언하면 **하나로 합쳐진다**.

```ts
interface Window {
  myApp: { version: string };
}
// 다른 파일에서 또 선언해도 Window에 myApp이 합쳐짐
```

라이브러리 타입을 **조금씩 덧붙일 때** 유용하다. 반면 `type`은 같은 이름을 두 번 쓰면 에러다.

### 2. 객체 중심의 확장

`extends`로 다른 인터페이스를 이어붙이기 읽기 쉽다.

```ts
interface Animal {
  name: string;
}
interface Dog extends Animal {
  bark(): void;
}
```

---

## `type`만 잘 하는 것 (또는 여기서 자연스러운 것)

### 1. 유니온·프리미티브·튜플

한 이름이 **여러 모양 중 하나**일 때는 `type`이 자연스럽다.

```ts
type Id = string | number;
type Point = [number, number];
type Status = 'idle' | 'loading' | 'error';
```

`interface`는 객체 한 덩어리에 맞춰져 있어서 위처럼 **문자열 리터럴 유니온**만 담는 식은 `type`이 더 흔하다.

### 2. 조건부 타입·매핑 등 고급 문법

`type` 별칭 쪽에서 조건부 타입, `infer`, 매핑된 타입 등을 많이 쓴다. (인터페이스로는 표현이 어렵거나 안 쓰는 패턴이다.)

### 3. 교차 타입 `&`

객체를 합칠 때 `type A = B & C` 패턴이 많다. `interface`는 `extends`로 비슷하게 할 수 있지만, **유니온에 `&`를 붙이는** 식은 `type`이 필요하다.

```ts
type A = { a: number };
type B = { b: string };
type AB = A & B;
```

---

## 타입 범위 차이

### `interface`

- 주로 객체 형태(객체 리터럴, 클래스 인터페이스)에 이름을 붙이는 데 사용

```ts
interface User {
  id: number;
  name: string;
}
```

- `number`, `string`, 유니온, 튜플 같은 원시/조합 타입 이름을 만들기에는 불편

### `type`

- **모든 타입에 이름을 줄 수 있다.**

```ts
type Status = "pending" | "success" | "failed";
type Coordinates = [number, number];
type ApiResponse<T> = { data: T; status: number };
```

- 객체 타입도 가능하지만, 객체 스키마를 정의하는 용도라면 `interface`를 더 선호

---

## 확장 방식 차이

### `interface`

- 같은 이름으로 여러 번 정의해서 자동 병합(declaration merging).

```ts
interface User {
  id: number;
}
interface User {
  name: string;
}
// User = { id: number; name: string }
```

- 라이브러리 확장, `Window` 같은 전역 타입에 속성 추가 시 유용

### `type`

- 같은 이름으로 다시 선언하면 에러

```ts
type User = { id: number };
type User = { name: string }; // ❌ Duplicate identifier
```

- 대신 `&`로 합성해서 확장

```ts
type User = { id: number } & { name: string };
```

---

##  계산된 속성 / 고급 타입 옵션

### `type`

- `mapped types`, `keyof`, `in`, `extends` 같은 것을 조합해서 동적으로 속성 이름을 만드는 타입을 만들 수 있다.

```ts
type Keys = "a" | "b";
type Obj = { [K in Keys]: string };
```

- 이런 패턴은 `interface`에서는불가능

### `interface`

- 단순한 객체 모양을 정의하는 데 최적화됨
  - 속성 충돌을 잘 감지 
  - 성능·읽기 측면에서 더 깔끔하게 합성 

---

 ## 사용 예시

### `interface`

- API 응답, DTO, 컴포넌트 props, 서비스 인터페이스 등  
  → **객체 스키마/모양을 정의할 때**

```ts
interface UserDto {
  id: number;
  name: string;
}
```

### `type`

- 유니온 타입, 튜플, `const Status = "a" | "b" | "c"`, 복잡한 타입 헬퍼 등  
  → **타입 계산/조합/원시형 이름 붙일 때**

```ts
type Status = "success" | "error";
type ApiResponse<T> = {
  data: T;
  status: Status;
};
```

---

## 정리

- 객체의 모양(스키마) 이름을 짓는 것이면 `interface`
- 모든 종류 타입(특히 유니온/튜플/고급 타입) 이름을 짓는 것이면 `type`  
 



---

작년에 학교에서 기본적인 Typescript 문법에 대해 공부했던 적이 있었다. 근데 그 이후에 뭔가 깊게 파 본 적은 없기도 하고, 요즘같이 바이브 코딩 시대에서 뭔가 생각없이 일 하고 있다는 생각이 계속 들었다..
물론 클로드가 있어서 그걸 이용하면 되지만 어느정도 유지보수하고 코드를 이해하는 주최자는 나여야 된다고 생각한다. 클로드가 다 해 줘도 어느정도는 이 코드에 대해 이해해야지 내가 추후에 설계를 다시 하거나 리팩토링을 할 수 있을 것이다. 그래서 지금부터 TypeScript 문법에 대해 공부해 보고자 한다!

오늘은 interface와 type의 차이와 정의에 대해 한번 다시 봐봤다. 작년에 배웠던 기억이 떠올랏기도 해서 왠지 더 반가운(?)..
이렇게라고 다시 한번 보면서 어느정도 생각하고 이해해 보고 싶다.  물론 속도도 중요하지만 아직 계속해서 배워나가는 단계이니 계속해서 머릿속에 물음표를 달고 살아야겠다 