# 가독성(Readability)

- 코드는 읽는 사람이 한 번에 이해할 수 있도록 맥락을 최소화해야 한다. (여러 역할이 한 컴포넌트에 섞여 있으면 분리해서 맥락을 줄인다)
- 코드는 위에서 아래로 자연스럽게 읽히도록 작성한다. (시점 이동 없이 한 곳에서 조건/상태/분기를 처리한다)
- 같이 실행되지 않는 코드는 분리한다. (권한별로 동작이 다를 때 컴포넌트를 분리해서 각자 역할만 담당하게 한다)
- 구현 상세는 추상화한다. (로그인 체크/리다이렉트 등은 Wrapper 컴포넌트로 분리해서 주요 로직만 남긴다)
- 로직 종류에 따라 합쳐진 함수는 쪼갠다. (여러 파라미터/상태를 한 곳에서 관리하지 말고 각각 분리해서 관리한다)
- 복잡한 조건에는 의미 있는 이름을 붙인다. (filter, some, && 등 복잡한 조건은 변수/함수로 분리해서 이름을 붙인다)
- 매직 넘버에는 이름을 붙인다. (300, 404 같은 숫자는 의미 있는 상수로 선언해서 쓴다)
- 시점 이동(여러 군데서 왔다갔다 하는 코드)은 줄인다. (조건, 상태, 분기 등을 한 곳에서 바로 처리해서 위에서 아래로 쭉 읽히게 한다)
- 삼항 연산자는 단순하게 쓴다. (중첩된 삼항 연산자 대신 if문 등으로 명확하게 분기한다)

## 예시

### 같이 실행되지 않는 코드 분리하기
(예시 파일: examples/submit-button.md)
동시에 실행되지 않는 코드가 한 컴포넌트에 섞여 있으면, 동작을 한눈에 파악하기 어렵다.

**개선 전:**
- 권한별로 버튼 동작이 한 컴포넌트에 분기 처리되어 있음.

```svelte
{#if user.role === 'viewer'}
  <button disabled>Submit</button>
{:else}
  <ButtonWithAnimation />
{/if}
```

**개선 후:**
- 권한별로 컴포넌트를 분리해서, 각 컴포넌트가 하나의 역할만 담당하도록 구조화.

```svelte
{#if user.role === 'viewer'}
  <ViewerSubmitButton />
{:else}
  <AdminSubmitButton />
{/if}
```

**핵심:**
한눈에 맥락 파악이 쉬워지고, 읽는 사람이 고려해야 할 맥락이 줄어듦.

### 구현 상세 추상화하기
(예시 파일: examples/login-start-page.md)
로그인 상태 확인, 리다이렉트 등 여러 로직이 한 곳에 섞여 있으면 읽기 어려움.

**개선 전:**
- 로그인 체크, 리다이렉트, UI 렌더링이 한 컴포넌트에 다 있음.

**개선 후:**
- 로그인 체크/리다이렉트 로직을 Wrapper 컴포넌트로 분리.

```svelte
<AuthGuard>
  <LoginStartPage />
</AuthGuard>
```

**핵심:**
주요 로직을 추상화해서, 읽는 사람이 한 번에 파악해야 할 맥락을 줄임.

### 로직 종류에 따라 합쳐진 함수 쪼개기
(예시 파일: examples/use-page-state-readability.md)
여러 파라미터/상태/로직을 한 곳에서 관리하면, 코드가 길어지고 복잡해짐.

**개선 전:**
- 여러 파라미터를 한 객체/함수에서 관리.

```svelte
const values = $derived(() => {
  // ...여러 파라미터 한 번에 관리
});
```

**개선 후:**
- 파라미터별로 $derived 등으로 분리해서 관리.

```svelte
let cardId = $derived(() => ...);
let dateFrom = $derived(() => ...);
```

**핵심:**
각 파라미터별 책임이 분리되어, 코드가 짧고 명확해짐.

### 복잡한 조건에 이름 붙이기
(예시 파일: examples/condition-name.md)
filter, some, && 등 복잡한 조건이 한 줄에 몰려 있으면 읽기 힘듦.

**개선 전:**
```js
const result = products.filter((product) =>
  product.categories.some(
    (category) =>
      category.id === targetCategory.id &&
      product.prices.some(
        (price) => price >= minPrice && price <= maxPrice
      )
  )
);
```

**개선 후:**
```js
const matchedProducts = products.filter((product) => {
  return product.categories.some((category) => {
    const isSameCategory = category.id === targetCategory.id;
    const isPriceInRange = product.prices.some(
      (price) => price >= minPrice && price <= maxPrice
    );
    return isSameCategory && isPriceInRange;
  });
});
```

**핵심:**
조건의 의미가 명확해지고, 가독성이 올라감.

### 매직 넘버에 이름 붙이기
(예시 파일: examples/magic-number-readability.md)
코드에 300, 404, 86400 같은 숫자가 바로 들어가 있으면 의미 파악이 어려움.

**개선 전:**
```js
await delay(300);
```

**개선 후:**
```js
const ANIMATION_DELAY_MS = 300;
await delay(ANIMATION_DELAY_MS);
```

**핵심:**
숫자의 의미가 명확해지고, 코드 리뷰/유지보수가 쉬워짐.

### 시점 이동 줄이기
(예시 파일: examples/user-policy.md)
여러 파일/함수/컴포넌트를 왔다갔다 하면서 읽어야 하면, 파악이 어려움.

**개선:**
조건, 상태, 분기 등을 한 곳에서 바로 처리해서, 위에서 아래로 쭉 읽히게 만듦.

```svelte
{#if user.role === 'admin'}
  <button>Invite</button>
  <button>View</button>
{:else if user.role === 'viewer'}
  <button disabled>Invite</button>
  <button>View</button>
{/if}
```

**핵심:**
읽는 사람이 여러 군데를 이동하지 않아도 한눈에 파악 가능.

### 삼항 연산자 단순하게 하기
(예시 파일: examples/ternary-operator.md)
중첩된 삼항 연산자, 복잡한 조건식은 가독성을 해침.

**개선 전:**
```