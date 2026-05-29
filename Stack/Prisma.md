# Prisma

> 데이터베이스를 자바스크립트 객체처럼 다룰 수 있게 해주는 ORM 도구.   
Node.js 생태계에서 널리 쓰임.

---


데이터베이스가 거대한 엑셀 파일이라고 떠올려보자. 원래는 **SQL**이라는 언어로 "이 시트에서 이 행 가져와", "이 셀 수정해"라고 말해야 한다.

**기존 SQL 방식**

```sql
SELECT * FROM users WHERE age > 20;
```

**Prisma 방식 (JS/TS)**

```ts
const users = await prisma.user.findMany({
  where: { age: { gt: 20 } }
});
```

SQL이랑 비교하면 훨씬 읽기 쉽다

---

## Prisma가 해주는 3가지

### 1️) `스키마 정의`

`schema.prisma` 파일 하나에 "User 테이블에는 이름, 이메일, 나이가 있어"라고 적으면 Prisma가 알아서 DB에 테이블을 만들어준다.

```prisma
model User {
  id    Int    @id @default(autoincrement())
  name  String
  email String @unique
  age   Int
}
```

### 2) `타입 자동 생성`

TypeScript를 쓰면 `prisma.user.create()`를 칠 때 어떤 필드가 있는지 자동완성으로 다 뜬다.

→ 오타와 실수가 거의 안 난다.

### 3) `마이그레이션`

나중에 User에 전화번호 컬럼 추가 하고자 한다면 스키마 파일을 수정하고 아래 명령어 한 번이면 끝이다.

```bash
npx prisma migrate dev
```

DB가 알아서 업데이트된다.

---

## 왜 쓸까?

| 장점 | 설명 |
|------|------|
| **편리함** | SQL 직접 안 써도 됨 |
|  **타입 안전성** | 런타임 에러가 컴파일 단계에서 잡힘 |
|  **DB 호환성** | PostgreSQL, MySQL, SQLite, MongoDB 등 지원 — DB 바꿔도 코드는 거의 그대로 |

---



> 마지막으로 한 줄로 요약해 보자면 Prisma는 **SQL 대신 코드로 안전하게 깔끔하게** 데이터베이스를 다루게 해주는 도구다!