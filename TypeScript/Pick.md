# Pick

`Pick`은 **기존 타입에서 특정 프로퍼티만 골라** 새 타입을 만드는 유틸리티 타입이다.

 

> 이 타입에서 `a`, `b` 키**만** 남긴다

## 문법

```ts
Pick<Type, Keys>
```

- `Type`: 원본 객체 타입
- `Keys`: 남기고 싶은 프로퍼티 이름들의 유니온 (`'a' | 'b'`)

## 예시

```ts
type User = {
  id: number;
  name: string;
  email: string;
  password: string;
};

// 목록 셀에 최소 정보만
type UserListItem = Pick<User, 'id' | 'name'>;
// { id: number; name: string }

// 로그인 요청: 식별에 필요한 필드만
type LoginBody = Pick<User, 'email' | 'password'>;
// { email: string; password: string }
```

## 언제 쓰나

- 화면/API에서 **필요한 필드만** 노출할 때
- 큰 타입 중 **몇 개 키만** 재사용할 때
- `extends` 없이 **부분 타입**을 짧게 표현할 때

## Omit과의 관계

- `Pick`: **이것만** 남긴다 → 화이트리스트
- `Omit`: **이것만** 뺀다 → 블랙리스트

같은 결과도 두 방식으로 표현할 수 있다.

```ts
type T = { a: number; b: string; c: boolean };

// b만 빼기
type X = Omit<T, 'b'>;        // { a, c }
type Y = Pick<T, 'a' | 'c'>;  // { a, c } — 동일한 형태
```

남기는 키가 적으면 `Pick`, 빼는 키가 적으면 `Omit`이 읽기 편하다.

## 주의할 점

1. `Keys`는 반드시 `Type`에 **실제로 있는 키**의 부분집합이어야 한다. 없는 키를 넣으면 에러다.
2. `Pick`도 **얕게** 동작한다. 깊은 중첩 구조의 일부만 고르는 용도는 아니다.

## 정리

| 항목 | 내용 |
|------|------|
| 역할 | 원본 타입에서 지정한 프로퍼티만 유지 |
| 문법 | `Pick<T, 'k1' \| 'k2'>` |
| 반대 개념 | `Omit` (지정한 프로퍼티 제거) |
