# Omit

`Omit`은 **기존 타입에서 특정 프로퍼티만 빼고** 새 타입을 만드는 유틸리티 타입이다.

 

> 이 타입에서 `a`, `b` 키는 제외한 나머지

## 문법

```ts
Omit<Type, Keys>
```

- `Type`: 원본 객체 타입
- `Keys`: 빼고 싶은 프로퍼티 이름들의 유니온 (`'a' | 'b'`)

## 예시

```ts
type User = {
  id: number;
  name: string;
  email: string;
  password: string;
};

// 회원가입 폼: password만 제외하고 받고 싶을 때
type SignUpInput = Omit<User, 'password'>;
// { id: number; name: string; email: string }

// 공개 프로필: 민감한 정보 제외
type PublicUser = Omit<User, 'password' | 'email'>;
// { id: number; name: string }
```

## 언제 쓰나

- API 응답/폼에서 **일부 필드만 숨기거나** 보내지 않을 때
- `password`, `token` 같이 **다른 타입과 공유하면 안 되는 필드**를 제외할 때
- 원본 타입은 그대로 두고 **부분만 쓰는 DTO**를 만들 때

## Pick과의 관계

- `Pick`: **이것만** 남긴다 → 화이트리스트
- `Omit`: **이것만** 뺀다 → 블랙리스트

남기는 키가 적으면 `Pick`, 빼는 키가 적으면 `Omit`을 쓰면 읽기 쉽다.

## 주의할 점

1. **존재하지 않는 키**를 `Keys`에 넣어도 컴파일 에러는 나지 않는다 (TypeScript 동작함) 의미 없는 키는 피하는 게 좋다 !
2. `Omit`은 **얕게(shallow)** 동작한다. 중첩 객체 안의 필드를 골라 빼는 용도는 아니다.

## 정리

| 항목 | 내용 |
|------|------|
| 역할 | 원본 타입에서 지정한 프로퍼티 제거 |
| 문법 | `Omit<T, 'k1' \| 'k2'>` |
| 반대 개념 | `Pick` (지정한 프로퍼티만 유지) |
