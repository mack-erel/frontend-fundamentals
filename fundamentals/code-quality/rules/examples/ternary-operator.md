# 삼항 연산자 단순하게 하기 (Svelte 5 + TypeScript 기준)

<div style="margin-top: 16px">
<Badge type="info" text="가독성" />
</div>

삼항 연산자를 복잡하게 사용하면 조건의 구조가 명확하게 보이지 않아서 코드를 읽기 어려울 수 있습니다.

## 📝 코드 예시 (Svelte 5 + TypeScript)

다음 코드는 `A조건`과 `B조건`에 따라서 "BOTH", "A", "B" 또는 "NONE" 중 하나를 `status`에 지정하는 코드입니다.

```svelte
<script lang="ts">
  let A조건 = $state(false);
  let B조건 = $state(false);

  // 복잡한 삼항 연산자 예시
  let status = $derived(() =>
    (A조건 && B조건) ? "BOTH" : (A조건 || B조건) ? (A조건 ? "A" : "B") : "NONE"
  );
</script>
```

## 👃 코드 냄새 맡아보기

### 가독성

이 코드는 여러 삼항 연산자가 중첩되어 사용되어서, 정확하게 어떤 조건으로 값이 계산되는지 한눈에 파악하기 어렵습니다.

## ✏️ 개선해보기 (Svelte 5 + TypeScript)

다음과 같이 조건을 `if` 문으로 풀어서 사용하면 보다 명확하고 간단하게 조건을 드러낼 수 있습니다.

```svelte
<script lang="ts">
  let A조건 = $state(false);
  let B조건 = $state(false);

  let status = $derived(() => {
    if (A조건 && B조건) return "BOTH";
    if (A조건) return "A";
    if (B조건) return "B";
    return "NONE";
  });
</script>
```
