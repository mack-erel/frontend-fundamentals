# 응집도(Cohesion)

- 함께 수정되어야 할 코드는 항상 같이 수정될 수 있도록 구조화한다. (함께 변경되는 파일/로직은 같은 도메인/기능별 디렉토리(src/lib/domains/...)로 묶는다)
- 함께 수정되는 파일은 같은 디렉토리에 둔다. (SvelteKit 기준, src/lib/domains/ 아래에 도메인/기능별로 디렉토리를 나눠서 의존관계를 명확히 한다)
- 매직 넘버는 없앤다. (같이 수정되어야 할 상수/로직은 의미 있는 이름으로 묶어서 관리한다)
- 폼의 각 필드는 응집도를 고려해서 관리한다. (필드 단위/폼 전체 단위 등 상황에 맞게 응집도를 설계한다)
- (참고) 응집도를 높이기 위해 추상화가 필요할 때, 가독성보다 응집도를 우선한다. (함께 수정되지 않으면 오류가 나는 경우엔 공통화/추상화 우선)
- (참고) 위험성이 낮으면 가독성을 우선해서 코드 중복을 허용한다. (중복이 유지보수에 더 유리하면 중복 허용)

## 예시

### 함께 수정되는 파일을 같은 디렉토리에 두기
(예시 파일: examples/code-directory.md)
여러 파일이 서로 의존하거나 함께 변경된다면, SvelteKit 기준으로 src/lib/domains/ 아래에 도메인/기능별로 묶어서 관리한다.

**개선 전:**
- 파일을 종류별(components, hooks, utils 등)로만 분리해서, 의존관계가 명확하지 않음.

```text
└─ src
   ├─ components
   ├─ constants
   ├─ containers
   ├─ contexts
   ├─ remotes
   ├─ hooks
   ├─ utils
   └─ ...
```

**개선 후:**
- SvelteKit 기준, 도메인/기능별로 src/lib/domains/ 아래에 디렉토리를 나눠서, 함께 수정되는 파일을 한 곳에 모음.

```text
└─ src
   └─ lib
       └─ domains
           ├─ domain1
           │   ├─ components
           │   ├─ stores
           │   ├─ utils
           │   └─ ...
           └─ domain2
               ├─ components
               ├─ stores
               ├─ utils
               └─ ...
```

**핵심:**
함께 수정되는 파일을 한 곳에 두면, 의존관계 파악과 유지보수가 쉬워진다.

### 매직 넘버 없애기
(예시 파일: examples/magic-number-cohesion.md)
매직 넘버(의미 없는 숫자 상수)는 응집도를 해치고, 변경 시 오류를 유발할 수 있다.

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
같이 수정되어야 할 값(상수/로직)은 의미 있는 이름으로 묶어서 관리한다.

### 폼의 응집도 생각하기
(예시 파일: examples/form-fields.md)
폼의 각 필드/전체를 어떻게 묶어서 관리할지에 따라 응집도가 달라진다.

**필드 단위 응집:**
- 각 필드별로 상태/검증 로직을 독립적으로 관리.
- 재사용성과 독립성이 높음.

```svelte
let name = $state('');
let email = $state('');
function validateName(value: string): string { ... }
function validateEmail(value: string): string { ... }
```

**폼 전체 단위 응집:**
- 모든 필드의 상태/검증을 한 곳에서 관리.
- 폼 전체 흐름을 일관되게 유지할 수 있음.

```svelte
let form = $state({ name: '', email: '' });
let errors = $state<{ name?: string; email?: string }>({});
function validateForm(values: { name: string; email: string }) { ... }
```

**핵심:**
변경 단위(필드/폼 전체)에 따라 응집도를 설계하고, 상황에 맞게 구조를 선택한다. 