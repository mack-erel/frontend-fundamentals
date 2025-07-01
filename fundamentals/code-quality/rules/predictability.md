# 예측 가능성(Predictability)

- 함수, 컴포넌트의 이름과 파라미터, 반환 값만 보고 동작을 예측할 수 있어야 한다. (숨은 로직 없이, 이름/파라미터/반환값만 봐도 동작이 예측되게 한다)
- 이름이 겹치지 않게 관리한다. (라이브러리 함수와 서비스 함수가 같은 이름을 쓰지 않게 구분해서 혼동을 막는다)
- 같은 종류의 함수는 반환 타입을 통일한다. (API, 상태 관리, 유효성 검사 등은 반환 타입을 일관되게 맞춘다)
- 숨겨진 로직은 드러내서 명확하게 한다. (함수 내부에서 암묵적으로 실행되는 부가 로직은 분리해서 명확하게 드러낸다)

## 예시

### 이름 겹치지 않게 관리하기
(예시 파일: examples/http.md)
동일한 이름을 가진 함수/변수가 서로 다른 동작을 하면 예측 가능성이 떨어진다.

**개선 전:**
- 서비스에서 만든 http 모듈과 라이브러리 http가 같은 이름을 사용해서 혼동을 유발.

```ts
export const http = {
  async get(url: string) {
    const token = await fetchToken();
    return httpLibrary.get(url, {
      headers: { Authorization: `Bearer ${token}` }
    });
  }
};
```

**개선 후:**
- 서비스 함수는 httpService, getWithAuth 등으로 명확하게 구분.

```ts
export const httpService = {
  async getWithAuth(url: string) {
    const token = await fetchToken();
    return httpLibrary.get(url, {
      headers: { Authorization: `Bearer ${token}` }
    });
  }
};
```

**핵심:**
함수 이름만 봐도 실제 동작을 예측할 수 있고, 혼동이 줄어든다.

### 같은 종류의 함수는 반환 타입 통일하기
(예시 파일: examples/use-user.md)
API, 상태 관리, 유효성 검사 등에서 반환 타입이 다르면 혼란이 생긴다.

**개선 전:**
- user, serverTime 등 상태 관리가 각각 다른 방식/타입으로 반환됨.
- 유효성 검사 함수가 boolean, 객체 등 서로 다른 타입을 반환함.

```ts
export function checkIsNameValid(name: string): boolean { ... }
export function checkIsAgeValid(age: number): { ok: true } | { ok: false; reason: string } { ... }
```

**개선 후:**
- 모든 유효성 검사 함수가 `{ ok, reason }` 형태의 객체를 반환하도록 통일.

```ts
export function checkIsNameValid(name: string): { ok: true } | { ok: false; reason: string } { ... }
export function checkIsAgeValid(age: number): { ok: true } | { ok: false; reason: string } { ... }
```

**핵심:**
함수/스토어/검증 함수의 반환 타입이 일관되면, 코드를 읽고 사용할 때 예측이 쉬워진다.

### 숨은 로직 드러내기
(예시 파일: examples/hidden-logic.md)
함수/컴포넌트의 이름, 파라미터, 반환 값에 드러나지 않는 숨은 로직이 있으면 예측이 어렵다.

**개선 전:**
- fetchBalance 함수 내부에서 암묵적으로 로깅이 실행됨.

```ts
async function fetchBalance(): Promise<number> {
  const balance = await http.get<number>("...");
  logging.log("balance_fetched");
  return balance;
}
```

**개선 후:**
- 로깅 등 부가 로직은 함수 밖에서 명시적으로 실행.

```ts
async function fetchBalance(): Promise<number> {
  const balance = await http.get<number>("...");
  return balance;
}

async function handleClick() {
  const b = await fetchBalance();
  logging.log("balance_fetched");
  await syncBalance(b);
}
```

**핵심:**
함수의 이름/파라미터/반환값만 봐도 동작이 예측되고, 숨은 부가 로직으로 인한 혼란이 사라진다. 