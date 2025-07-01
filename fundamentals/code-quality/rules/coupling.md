# 결합도(Coupling)

- 코드를 수정했을 때 영향범위가 최소화되도록 작성한다. (여러 책임/로직이 한 곳에 몰려 있으면 분리해서 영향범위를 줄인다)
- 책임은 하나씩 관리한다. (쿼리 파라미터, 상태, API 호출 등은 각각 분리해서 관리한다)
- 중복 코드를 허용해서 결합도를 낮춘다. (공통화가 오히려 결합도를 높이면, 중복을 허용해서 영향범위를 줄인다)
- Props Drilling(깊은 컴포넌트 트리로 props 전달)은 지운다. (조합 패턴, Context API 등으로 불필요한 props 전달을 줄인다)

## 예시

### 책임을 하나씩 관리하기
(예시 파일: examples/use-page-state-coupling.md)
여러 책임(쿼리 파라미터, 상태, API 호출 등)을 한 곳에 몰아넣지 않고, 각각 분리해서 관리한다.

**개선 전:**
- 여러 쿼리 파라미터를 한 객체/함수에서 한 번에 관리해서, 수정 시 영향범위가 넓어짐.

```svelte
const values = $derived(() => {
  // ...여러 파라미터 한 번에 관리
});
```

**개선 후:**
- 각 파라미터별로 $derived 등으로 분리해서 관리.

```svelte
let cardId = $derived(() => ...);
let dateFrom = $derived(() => ...);
```

**핵심:**
각 책임이 분리되어, 수정해도 영향 범위가 좁아진다.

### 중복 코드 허용하기
(예시 파일: examples/use-bottom-sheet.md)
공통화가 오히려 결합도를 높이면, 중복을 허용해서 영향범위를 줄인다.

**개선 전:**
- 여러 페이지에서 반복되는 로직을 공통 함수로 분리해서, 수정 시 모든 페이지에 영향이 감.

```ts
export function useOpenMaintenanceBottomSheet() {
  // ...공통화된 로직
}
```

**개선 후:**
- 페이지마다 동작이 다를 여지가 있으면, 중복 코드를 허용해서 각자 관리.

**핵심:**
공통화가 무조건 좋은 게 아니라, 결합도를 낮추는 게 더 중요할 때는 중복을 허용한다.

### Props Drilling 지우기
(예시 파일: examples/item-edit-modal.md)
부모-자식-손자 컴포넌트로 props를 계속 전달하는 구조(Props Drilling)는 결합도를 높인다.

**개선 전:**
- 여러 컴포넌트가 동일한 props를 계속 전달받아야 해서, prop이 바뀌면 모든 컴포넌트를 수정해야 함.

```svelte
<ItemEditBody
  {items}
  {keyword}
  onKeywordChange={setKeyword}
  {recommendedItems}
  {onConfirm}
  {onClose}
/>
```

**개선 후:**
- 조합 패턴, Context API 등으로 불필요한 props 전달을 줄임.

```svelte
<ItemEditBody {onClose}>
  <ItemEditList
    {keyword}
    {items}
    {recommendedItems}
    {onConfirm}
  />
</ItemEditBody>
```

또는 Context API 활용:

```svelte
setContext('itemEdit', { items, recommendedItems });
```

**핵심:**
불필요한 props 전달을 줄이면, prop 변경 시 영향범위가 줄어들고, 컴포넌트 결합도가 낮아진다. 