# 같은 종류의 함수는 반환 타입 통일하기

<div style="margin-top: 16px">
<Badge type="info" text="예측 가능성" />
</div>

SvelteKit에서 API 호출이나 상태 관리를 할 때, 같은 종류의 함수나 store가 서로 다른 반환 타입을 가지면 코드의 일관성이 떨어져서, 같이 일하는 동료들이 코드를 읽는 데에 혼란이 생길 수 있습니다.

## 📝 코드 예시 1: user와 serverTime 데이터 관리 (TypeScript)

아래 예시는 Svelte 5에서 전역 상태도 runes($state)로 타입을 명시해서 관리하는 방법입니다. 예전 React 환경에서는 Hook마다 반환 타입이 달라서 혼란이 생길 수 있었지만, Svelte 5에서는 전역 상태도 runes로 일관성 있게 관리할 수 있습니다.

```ts
// userStore.svelte.ts
let user = $state<{ name: string } | null>(null);
let serverTime = $state<string | null>(null);

export { user, serverTime };

export async function fetchUser(): Promise<void> {
  const res = await fetch('/api/user');
  user = await res.json();
}

export async function fetchServerTime(): Promise<void> {
  const res = await fetch('/api/server-time');
  serverTime = await res.json();
}
```

```svelte
<script lang="ts">
  import { user, serverTime, fetchUser, fetchServerTime } from './userStore.svelte.ts';
  import { onMount } from 'svelte';
  onMount(() => {
    fetchUser();
    fetchServerTime();
  });
</script>

{#if user}
  <p>유저: {user.name}</p>
{/if}
{#if serverTime}
  <p>서버 시간: {serverTime}</p>
{/if}
```

또는 SvelteKit의 load 함수에서 데이터를 받아서 props로 넘기는 방식도 사용할 수 있습니다.

```svelte
<script lang="ts">
  let { data }: { data: { user: { name: string }, serverTime: string } } = $props();
  // 실제로는 +page.ts/+page.server.ts에서 load 함수로 fetch
  // 여기선 예시로 data.user, data.serverTime 사용
</script>

{#if data.user}
  <p>유저: {data.user.name}</p>
{/if}
{#if data.serverTime}
  <p>서버 시간: {data.serverTime}</p>
{/if}
```

### 👃 코드 냄새 맡아보기

#### 예측 가능성

서버 API를 호출하거나 상태를 관리하는 store의 반환 타입이 서로 다르면, 동료들은 이런 store나 함수를 쓸 때마다 반환 타입이 무엇인지 확인해야 합니다. 일관된 방식으로 데이터를 관리하면, 코드를 읽고 사용하는 데 혼란이 줄어듭니다.

### ✏️ 개선해보기

Svelte 5에서는 전역 상태도 runes($state)로 타입을 명시해서 일관성 있게 관리하는 것이 좋습니다. 아래처럼 user와 serverTime 모두 $state로 관리하면, 반환 타입이 일관되어 예측 가능성이 높아집니다.

```ts
// userStore.svelte.ts
let user = $state<{ name: string } | null>(null);
let serverTime = $state<string | null>(null);

export { user, serverTime };

export async function fetchUser(): Promise<void> {
  const res = await fetch('/api/user');
  user = await res.json();
}

export async function fetchServerTime(): Promise<void> {
  const res = await fetch('/api/server-time');
  serverTime = await res.json();
}
```

---

## 📝 코드 예시 2: checkIsValid (TypeScript)

아래 예시는 이름과 나이의 유효성을 검사하는 함수입니다. Svelte 5 환경에서도 유효성 검사 함수의 반환 타입을 일관성 있게 맞추는 것이 중요합니다.

```ts
/** 사용자 이름은 20자 미만이어야 합니다. */
export function checkIsNameValid(name: string): boolean {
  return name.length > 0 && name.length < 20;
}

/** 사용자 나이는 18세 이상 99세 이하의 자연수여야 합니다. */
export function checkIsAgeValid(age: number): { ok: true } | { ok: false; reason: string } {
  if (!Number.isInteger(age)) {
    return {
      ok: false,
      reason: "나이는 정수여야 해요."
    };
  }

  if (age < 18) {
    return {
      ok: false,
      reason: "나이는 18세 이상이어야 해요."
    };
  }

  if (age > 99) {
    return {
      ok: false,
      reason: "나이는 99세 이하이어야 해요."
    };
  }

  return { ok: true };
}
```

### 👃 코드 냄새 맡아보기

#### 예측 가능성

유효성 검사 함수의 반환 값이 다르면, 동료들은 함수를 쓸 때마다 반환 타입을 확인해야 해서 혼란이 생길 수 있습니다. Svelte 5 환경에서도 일관된 반환 타입을 유지하는 것이 좋습니다.

특히 [엄격한 불리언 검증](https://typescript-eslint.io/rules/strict-boolean-expressions/)과 같은 기능을 사용하지 않는 경우, 코드의 오류가 생기는 원인이 될 수 있습니다.

```ts
// 이 코드는 이름이 규칙에 맞는지 올바르게 검증합니다.
if (checkIsNameValid(name)) {
  // ...
}

// 이 함수는 항상 객체 { ok, ... } 를 반환하기 때문에,
// `if` 문 안에 있는 코드가 항상 실행됩니다.
if (checkIsAgeValid(age)) {
  // ...
}
```

### ✏️ 개선해보기

아래처럼 유효성 검사 함수가 일관적으로 `{ ok, ... }` 타입의 객체를 반환하게 하면, 코드를 읽고 사용하는 사람이 예측하기 쉬워집니다.

```ts
/** 사용자 이름은 20자 미만이어야 합니다. */
export function checkIsNameValid(name: string): { ok: true } | { ok: false; reason: string } {
  if (name.length === 0) {
    return {
      ok: false,
      reason: "이름은 빈 값일 수 없어요."
    };
  } 
  
  if (name.length >= 20) {
    return {
      ok: false,
      reason: '이름은 20자 이상 입력할 수 없어요.'
    };
  }

  return { ok: true };
}

/** 사용자 나이는 18세 이상 99세 이하의 자연수여야 합니다. */
export function checkIsAgeValid(age: number): { ok: true } | { ok: false; reason: string } {
  if (!Number.isInteger(age)) {
    return {
      ok: false,
      reason: "나이는 정수여야 해요."
    };
  }

  if (age < 18) {
    return {
      ok: false,
      reason: "나이는 18세 이상이어야 해요."
    };
  }

  if (age > 99) {
    return {
      ok: false,
      reason: "나이는 99세 이하이어야 해요."
    };
  }

  return { ok: true };
}
```

::: tip

유효성 검사 함수의 반환 타입을 Discriminated Union으로 정의하면, `ok` 값에 따라 `reason`의 존재 유무를 확인할 수 있습니다.

```ts
type ValidationCheckReturnType = { ok: true } | { ok: false; reason: string };

export function checkIsAgeValid(age: number): ValidationCheckReturnType {
  if (!Number.isInteger(age)) {
    return {
      ok: false,
      reason: "나이는 정수여야 해요."
    };
  }
  // ...
}

const isAgeValid = checkIsAgeValid(1.1);

if (isAgeValid.ok) {
  isAgeValid.reason; // 타입 에러: { ok: true } 타입에는 reason 속성이 없습니다.
} else {
  isAgeValid.reason; // ok가 false일 때만 reason 속성에 접근할 수 있습니다.
}
```

이렇게 하면 컴파일러를 통해 불필요한 접근을 예방할 수 있습니다.

:::
