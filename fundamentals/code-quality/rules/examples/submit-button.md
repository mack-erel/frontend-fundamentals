# 같이 실행되지 않는 코드 분리하기 (Svelte 5 + TypeScript 기준)

<div style="margin-top: 16px">
<Badge type="info" text="가독성" />
</div>

동시에 실행되지 않는 코드가 하나의 함수 또는 컴포넌트에 있으면, 동작을 한눈에 파악하기 어려울 수 있습니다.
구현 부분에 많은 숫자의 분기가 들어가면, 어떤 역할을 하는지 이해하기 어렵기도 합니다.

## 📝 코드 예시 (Svelte 5 + TypeScript)

다음 `<SubmitButton />` 컴포넌트는 사용자의 권한에 따라서 다르게 동작합니다.

- 사용자의 권한이 보기 전용(`"viewer"`)이면, 제출 버튼은 비활성화되어 있고, 애니메이션도 재생하지 않습니다.
- 사용자가 일반 사용자이면, 제출 버튼을 사용할 수 있고, 애니메이션도 재생합니다.

```svelte
<script lang="ts">
  type UserRole = 'viewer' | 'user';
  let user = $state<{ role: UserRole }>({ role: 'user' });

  function showButtonAnimation() {
    // 애니메이션 로직 (구현 생략)
  }
</script>

{#if user.role === 'viewer'}
  <button disabled>Submit</button>
{:else}
  <ButtonWithAnimation />
{/if}

<!-- ButtonWithAnimation.svelte -->
<script lang="ts">
  $effect(() => {
    showButtonAnimation();
  });
</script>

<button type="submit">Submit</button>
```

## 👃 코드 냄새 맡아보기

### 가독성

`<SubmitButton />` 컴포넌트에서는 사용자가 가질 수 있는 2가지의 권한 상태를 하나의 컴포넌트 안에서 한 번에 처리하고 있습니다.
그래서 코드를 읽는 사람이 한 번에 고려해야 하는 맥락이 많아집니다.

동시에 실행되지 않는 코드가 교차되어서 나타나면 코드를 이해할 때 부담을 줄 수 있습니다.

![](../../images/examples/submit-button.png){.light-only}
![](../../images/examples/submit-button-dark.png){.dark-only}

## ✏️ 개선해보기 (Svelte 5 + TypeScript)

다음 코드는 사용자가 보기 전용 권한을 가질 때와 일반 사용자일 때를 완전히 나누어서 관리하도록 하는 코드입니다.

```svelte
<!-- SubmitButton.svelte -->
<script lang="ts">
  type UserRole = 'viewer' | 'user';
  let user = $state<{ role: UserRole }>({ role: 'user' });
</script>

{#if user.role === 'viewer'}
  <ViewerSubmitButton />
{:else}
  <AdminSubmitButton />
{/if}

<!-- ViewerSubmitButton.svelte -->
<script lang="ts">
</script>
<button disabled>Submit</button>

<!-- AdminSubmitButton.svelte -->
<script lang="ts">
  function showAnimation() {
    // 애니메이션 로직 (구현 생략)
  }
  $effect(() => {
    showAnimation();
  });
</script>
<button type="submit">Submit</button>
```

- `<SubmitButton />` 코드 곳곳에 있던 분기가 단 하나로 합쳐지면서, 분기가 줄어듭니다.
- `<ViewerSubmitButton />`과 `<AdminSubmitButton />` 에서는 하나의 분기만 관리하기 때문에, 코드를 읽는 사람이 한 번에 고려해야 할 맥락이 적어집니다.
