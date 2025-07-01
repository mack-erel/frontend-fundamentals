# 시점 이동 줄이기 (Svelte 5 + TypeScript 기준)

<div style="margin-top: 16px">
<Badge type="info" text="가독성" />
</div>

코드를 읽을 때 코드의 위아래를 왔다갔다 하면서 읽거나, 여러 파일이나 함수, 변수를 넘나들면서 읽는 것을 **시점 이동**이라고 합니다.
시점이 여러 번 이동할수록 코드를 파악하는 데에 시간이 더 걸리고, 맥락을 파악하는 데에 어려움이 있을 수 있습니다.

Svelte 5에서는 명시적 리액티브(룬 문법)로 상태와 조건을 한눈에 드러내는 것이 훨씬 쉽습니다. 복잡한 추상화 없이, 컴포넌트 안에서 바로 조건을 드러내면 읽는 사람 입장에서 훨씬 직관적입니다.

## 📝 Svelte 5 코드 예시 (TypeScript)

다음 코드에서는 사용자의 권한에 따라서 버튼을 다르게 보여줍니다.

- 사용자의 권한이 관리자(Admin)라면, `Invite`와 `View` 버튼을 보여줍니다.
- 사용자의 권한이 보기 전용(Viewer)이라면, `Invite` 버튼은 비활성화하고, `View` 버튼을 보여줍니다.

```svelte
<script lang="ts">
  type UserRole = 'admin' | 'viewer';
  let user = $state<{ role: UserRole }>({ role: 'admin' });
</script>

{#if user.role === 'admin'}
  <button>Invite</button>
  <button>View</button>
{:else if user.role === 'viewer'}
  <button disabled>Invite</button>
  <button>View</button>
{/if}
```

## 👃 코드 냄새 맡아보기 (Svelte 5 관점)

### 가독성

옛날처럼 추상화된 policy 객체, 함수, 상수 이런 거 굳이 안 쓰고, Svelte 5에선 상태랑 조건을 바로 코드에 드러내면 됩니다.

- `user.role`만 보면 바로 어떤 버튼이 활성/비활성인지 알 수 있습니다.
- 시점 이동 없음. 위에서 아래로 쭉 읽으면 끝입니다.
- 복잡한 권한 체계 아니면 굳이 policy 객체, 함수 만들 필요 없습니다.

## ✏️ 개선 예시 (Svelte 5 + TypeScript 스타일)

### 1. 조건을 코드에 바로 드러내기

```svelte
<script lang="ts">
  type UserRole = 'admin' | 'viewer';
  let user = $state<{ role: UserRole }>({ role: 'viewer' });
</script>

{#if user.role === 'admin'}
  <button>Invite</button>
  <button>View</button>
{:else if user.role === 'viewer'}
  <button disabled>Invite</button>
  <button>View</button>
{/if}
```

### 2. 조건을 한눈에 볼 수 있는 객체로 관리 (그래도 단순하게)

```svelte
<script lang="ts">
  type UserRole = 'admin' | 'viewer';
  let user = $state<{ role: UserRole }>({ role: 'admin' });
  const policy: Record<UserRole, { canInvite: boolean; canView: boolean }> = {
    admin: { canInvite: true, canView: true },
    viewer: { canInvite: false, canView: true }
  };
</script>

{#if policy[user.role]}
  <button disabled={!policy[user.role].canInvite}>Invite</button>
  <button disabled={!policy[user.role].canView}>View</button>
{/if}
```

---

Svelte 5에서는 굳이 추상화된 함수, 상수, policy 객체를 밖에 빼서 시점 이동 유발하지 말고, 컴포넌트 안에서 바로 조건 드러내는 것이 훨씬 낫습니다.

룬 문법($state)로 상태 선언하고, 조건문({#if ...})으로 바로 분기 처리하면 끝입니다.

이게 Svelte 5에서 가독성 챙기는 정석입니다.
