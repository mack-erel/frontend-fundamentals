# 로직 종류에 따라 합쳐진 함수 쪼개기 (Svelte 5/SvelteKit + TypeScript 기준)

<div style="margin-top: 16px">
<Badge type="info" text="가독성" />
</div>

쿼리 파라미터, 상태, API 호출 등 여러 책임을 한 곳에 몰아넣지 않는 것이 좋습니다. SvelteKit에서는 page store, $state, $derived 등으로 상태와 URL 파라미터를 명확하게 분리해서 관리하는 것이 가독성과 성능 모두에 유리합니다.

## 📝 SvelteKit 코드 예시 (TypeScript)

다음은 SvelteKit에서 여러 쿼리 파라미터를 한 번에 관리하는 좋지 않은 예시입니다.

```svelte
<script lang="ts">
  import { page } from '$app/state';
  import dayjs from 'dayjs';

  // 기본값
  const defaultDateFrom = dayjs().subtract(3, 'month');
  const defaultDateTo = dayjs();

  // page store에서 쿼리 파라미터 추출 (룬 없이 구린 예시)
  // 아래는 Svelte 5 스타일이 아님. 개선 예시는 아래 참고
  const values = $derived(() => {
    const query = page.url.searchParams;
    return {
      cardId: query.get('cardId') ? Number(query.get('cardId')) : undefined,
      statementId: query.get('statementId') ? Number(query.get('statementId')) : undefined,
      dateFrom: query.get('dateFrom') ? dayjs(query.get('dateFrom')) : defaultDateFrom,
      dateTo: query.get('dateTo') ? dayjs(query.get('dateTo')) : defaultDateTo,
      statusList: query.getAll('statusList').length > 0 ? query.getAll('statusList') : undefined
    };
  });
</script>
```

## 👃 코드 냄새 맡아보기 (Svelte 5 관점)

### 가독성

이렇게 모든 쿼리 파라미터를 한 곳에서 다루면, 컴포넌트가 담당하는 책임이 무제한으로 늘어날 수 있습니다. 새로운 파라미터가 추가될 때마다 이 값에 다 엮여버려서, 코드가 점점 길어지고 무슨 역할인지 파악하기 힘들어집니다.

### 성능

이렇게 한 곳에서 다 관리하면, 불필요한 리렌더링도 많이 생깁니다. 예를 들어 cardId만 쓰는 컴포넌트도 dateFrom이 바뀌면 같이 리렌더링됩니다. Svelte 5/SvelteKit에서는 필요한 값만 따로따로 관리해서, 진짜 필요한 부분만 리렌더링되게 하는 것이 성능에 훨씬 좋습니다.

::: info

이 내용은 [결합도](./use-page-state-coupling.md) 관점으로도 볼 수 있습니다.

:::

---

## ✏️ 개선 예시 (Svelte 5/SvelteKit + TypeScript 룬 스타일)

각 쿼리 파라미터별로 Svelte 5 룬($derived)로 계산값을 분리하는 것이 훨씬 좋습니다.

```svelte
<script lang="ts">
  import { page } from '$app/state';
  import dayjs from 'dayjs';

  // cardId만 따로 관리 (룬 스타일)
  let cardId = $derived(() => {
    const v = page.url.searchParams.get('cardId');
    return v ? Number(v) : undefined;
  });

  // dateFrom만 따로 관리 (룬 스타일)
  let dateFrom = $derived(() => {
    const v = page.url.searchParams.get('dateFrom');
    return v ? dayjs(v) : dayjs().subtract(3, 'month');
  });
</script>
```

이렇게 하면 각 파라미터별로 책임이 분리되어, 코드도 짧고 명확해지고, 수정해도 영향 범위가 좁아집니다.

Svelte 5/SvelteKit에서는 page store, $state, $derived 등으로 상태와 URL 파라미터를 명확하게 쪼개서 관리하는 것이 정석입니다.
