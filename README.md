# my-skill

자주 쓰는 Codex/AI 에이전트용 지침과 설정 가이드를 모아둔 저장소입니다.

## 구성

```text
setting/
  AGENTS_guide.md
  AGENTS_guide_next16_fsd.md
  nextjs-test-guide.md

skills/
  algo_blog_maker.md
```

## 사용법

### `setting/AGENTS_guide.md`

프로젝트별로 채워 넣어 쓰는 범용 `AGENTS.md` 템플릿입니다.

Next.js 버전, 아키텍처, 캐싱 정책이 프로젝트마다 다를 때 기본 뼈대로 사용합니다.

### `setting/AGENTS_guide_next16_fsd.md`

Next.js 16, Cache Components, Feature-Sliced Design 프로젝트에서 AI 에이전트가 지켜야 할 작업 규칙입니다.

프로젝트의 `AGENTS.md` 초안으로 쓰거나, AI에게 작업 전에 참고시키면 됩니다.

### `setting/nextjs-test-guide.md`

Next.js 프로젝트에 Vitest, React Testing Library, Playwright 테스트 환경을 세팅할 때 보는 가이드입니다.

테스트 도구 설치, 설정 파일, 실행 스크립트 기준을 잡을 때 사용합니다.

### `skills/algo_blog_maker.md`

알고리즘 풀이 코드를 블로그용 Markdown과 Supabase 삽입용 SQL로 변환하는 에이전트 지침입니다.

프로그래머스나 LeetCode 풀이를 포스팅 형태로 빠르게 정리할 때 사용합니다.

## 참고

이 저장소는 실행 코드가 아니라 작업 지침 모음입니다.  
실제 프로젝트에 적용할 때는 현재 프로젝트의 설정과 스크립트에 맞게 조금씩 조정해서 사용하면 됩니다.
