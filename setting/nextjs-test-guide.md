# Next.js 테스트 환경 세팅 가이드 (Vitest + RTL + Playwright)

Alias 설치를 플러그인으로 하는 이유는 tsconfig.json의 경로(paths, alias)를 읽어오는 자체 내장(Native) 기능이 현재 버전 오류가 있기 때문에, 기존 Alias 설정으로 테스트를 실행할 경우 오류가 발생할 수 있습니다. 오류가 해결되면 추후 변경 예정입니다.

## 1. Playwright 설치 및 초기화 (E2E 테스트)

```bash
pnpm create playwright
```

> 프롬프트 선택:
>
> Where to put your end-to-end tests?: tests
>
> Add a GitHub Actions workflow? (Y/n): 필요에 따라 선택
>
> Install Playwright browsers (can be done manually via 'pnpm exec playwright install')? (Y/n) » true

## 2. Vitest & RTL 의존성 설치

```bash
pnpm add -D vitest @vitejs/plugin-react vite-tsconfig-paths jsdom @testing-library/react @testing-library/dom @testing-library/jest-dom
```

## 3. Vitest & RTL 세팅

### vitest.config.ts

컴포넌트 옆에 위치한 \*.test.tsx 파일들을 실행하되, Playwright용 tests 폴더는 제외하도록 설정합니다.

```typescript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  test: {
    environment: "jsdom",
    setupFiles: ["./vitest.setup.ts"],
    globals: true,
    exclude: ["node_modules", ".next", "tests"],
  },
});
```

### vitest.setup.ts

```typescript
import "@testing-library/jest-dom";
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "types": ["vitest/globals", "@testing-library/jest-dom"]
  }
}
```

## 4. Playwright 설정 마무리 (playwright.config.ts)

자동 생성된 `playwright.config.ts`에서 `webServer` 부분을 주석 해제하고 세팅합니다.

```typescript
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./tests", // 기본값 유지

  // ... (기본 병렬 처리, 재시도 설정 등 유지)

  webServer: {
    command: "pnpm dev",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
  },
});
```

## 5. 테스트 실행 명령어 추가

### package.json

```json
{
  "scripts": {
    "test": "vitest",
    "test:watch": "vitest --watch",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui"
  }
}
```

## 6. eslint.config.mjs 설정

Playwright 테스트 코드와 결과물 폴더가 린트 검사 대상에 포함되어 불필요한 에러(Expected an assignment or function call... 등)를 뿜어내는 것을 방지해야 합니다.

해당 앱의 eslint.config.mjs (또는 .js) 파일을 열고, ignores 처리하는 객체를 추가합니다.

```javascript
{
    // Playwright 관련 폴더 린트 검사 제외
    ignores: ['tests/**', 'playwright-report/**', 'test-results/**'],
  },
```

## 7. 디렉토리 구조 예시

단위/통합 테스트는 실제 컴포넌트 바로 옆에 위치시키고, E2E 테스트는 분리합니다.

```plaintext
├── src
│ └── components (또는 features 등)
│ └── Button
│ ├── Button.tsx
│ └── Button.test.tsx # 👈 컴포넌트와 테스트 코로케이션
├── tests # 👈 Playwright E2E 테스트 전용 폴더
│ └── example.spec.ts
├── vitest.config.ts
├── vitest.setup.ts
├── playwright.config.ts
└── tsconfig.json
```
