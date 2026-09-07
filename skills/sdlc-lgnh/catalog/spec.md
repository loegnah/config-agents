# Stage 2: Technical Specification (spec.md)

Translate `intent.md` into technical requirements, architecture design, and policy specifications in `spec.md`.

## Language

Respond in the same language the user is using (e.g., Korean if the user communicates in Korean). Match the user's language for artifact content and discussions.

## Arguments

- `target` (optional): Path to target story directory (e.g. `docs/story/2026-09-06-feature/`).

## Constraints

- Translate business language from `intent.md` into concrete interface specifications and system architecture.
- Actively apply project and organizational policies (security, authentication, error handling, UX, performance).
- Explicitly surface technical risks or conflicting policies under the `Areas of Concern` section.

## Execution Steps

1. **Locate Target Directory & `intent.md`**
   - If argument provided, resolve `<target_dir>`.
   - If omitted:
     - Search `docs/story/` for the most recently modified directory containing `intent.md`.
   - Read `<target_dir>/intent.md`. If missing, report error and instruct user to run `/sdlc-lgnh intent` first.

2. **Explore Codebase Context & Policies**
   - Inspect existing codebase architecture, directory layout, types, and relevant APIs.
   - Review project rules (`CLAUDE.md`, `AGENTS.md`, configuration files).
   - Address open questions and constraints identified in `intent.md`.

3. **Write `spec.md`**
   - Save to `<target_dir>/spec.md` using the standard template:
     ```markdown
     # 명세서: [기능 명칭] (기반: intent.md YYYY-MM-DD)

     - 작성/검토자: [작성자 / 검토자]
     - 상태: [초안(draft) | 승인됨(approved)]

     ## 1. 아키텍처 개요 (Architecture Overview)

     - 기존 시스템과 신규 기능의 연동 구조 (데이터 흐름, 컴포넌트 관계)

     ## 2. 기능 요구사항 (Functional Requirements)

     - 세부 기능 1: 입력값, 유효성 검증 규칙, 기대 결과
     - 세부 기능 2: 처리 조건 및 예외 케이스 정의

     ## 3. 인터페이스 & 데이터 계약 (API & Data Contracts)

     - API 엔드포인트 명세 (HTTP 메서드, 경로, 요청/응답 JSON 스키마)
     - DB 스키마 변경 사항 또는 이벤트 메시지/타입 정의

     ## 4. 준수 정책 및 표준 (Policies Applied)

     - 보안: 인증/인가, 권한 스코프, 민감정보 처리
     - 성능: 캐싱 전략, 외부 API 타임아웃, 리소스 제한
     - UX / 에러 처리: 오류 발생 시 사용자 피드백 가이드

     ## 5. 우려 영역 및 정책 충돌 (Areas of Concern)

     - 잠재적 정책 충돌, 시스템 한계, 또는 기술적 리스크
     - intent.md 미결 질문에 대한 검토 결과 및 결정 사항
     ```

4. **Report & Guide Next Step**
   - Summarize generated `spec.md`, explicitly highlighting any entries under `Areas of Concern`.
   - Guide next step: Inform the user to review/approve the specification, then run `/sdlc-lgnh plan <target_dir>` (or `/sdlc-lgnh plan`) to formulate the Stage 3 implementation plan.
