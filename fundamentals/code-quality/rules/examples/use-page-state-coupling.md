# 책임을 하나씩 관리하기 (Svelte 5/SvelteKit + TypeScript 기준)

<div style="margin-top: 16px">
<Badge type="info" text="결합도" />
</div>

쿼리 파라미터, 상태, API 호출 등 여러 책임을 한 곳에 몰아넣지 않는 것이 좋습니다. SvelteKit에서는 page store, $state, $derived 등으로 상태와 URL 파라미터를 명확하게 분리해서 관리하는 것이 유지보수와 가독성 모두에 유리합니다.

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

::: info

이 내용은 [가독성](./use-page-state-readability.md) 관점으로도 볼 수 있습니다.

:::

## 👃 코드 냄새 맡아보기 (Svelte 5 관점)

### 결합도

이렇게 모든 쿼리 파라미터를 한 곳에서 다루면, 페이지 내 다른 컴포넌트나 함수들이 이 값에 다 엮여버립니다. 수정할 때 영향 범위가 넓어지고, 유지보수도 어려워집니다.

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

이렇게 하면 각 파라미터별로 책임이 분리되어, 수정해도 영향 범위가 좁아집니다.

Svelte 5/SvelteKit에서는 page store, $state, $derived 등으로 상태와 URL 파라미터를 명확하게 쪼개서 관리하는 것이 정석입니다.
