<!--
Sync Impact Report
- Version change: N/A → 1.0.0
- Modified principles: Template placeholders replaced with concrete principles I–V
- Added sections: "Scope, Non-Goals, and Constraints"; "Development Workflow & Quality Gates"
- Removed sections: None (only template example comments removed)
- Templates requiring updates:
  - .specify/templates/plan-template.md: ✅ aligned (Constitution Check derives gates from principles)
  - .specify/templates/spec-template.md: ✅ aligned (no constitution-specific constraints required)
  - .specify/templates/tasks-template.md: ✅ aligned (tasks organized by user story as required)
  - .specify/templates/agent-file-template.md: ✅ aligned (no agent-specific references)
  - .specify/templates/checklist-template.md: ✅ aligned (generic checklist structure)
  - .specify/templates/commands/*: ⚠ pending (directory not present; verify when commands are added)
- Follow-up TODOs: None
-->

# GPU Schedule Bot Constitution

## Core Principles

### I. Slack-First Reservation Experience

- 모든 핵심 사용자 플로우(보유 GPU 목록 조회, 사용 가능 시간 확인, 예약 생성·수정·취소, 내 예약 확인)는 Slack 안에서만 완결되어야 한다 (MUST).
- Slash 명령어·버튼·스레드 흐름은 일관된 패턴을 사용하고, 한 번의 상호작용에서 필요한 정보만 요구해야 한다 (MUST).
- 새로운 기능의 스펙(spec.md)에는 최소 한 개 이상의 Slack 대화 예시(정상/에러 케이스)가 포함되어야 한다 (MUST).
- 근거: 사용자는 별도 웹 UI 없이 Slack만으로 GPU 스케줄을 관리할 수 있어야 하고, 상호작용이 길어지면 실제 예약율이 떨어진다.

### II. 단일 소스 기반 GPU 스케줄

- GPU 예약·취소·변경에 대한 단일 소스(예: 데이터베이스 또는 외부 API 어댑터)가 존재해야 하며, Slack 메시지는 항상 이 소스를 조회한 결과만을 보여줘야 한다 (MUST).
- 예약 상태를 추론하거나 캐싱할 때는 원본 데이터와의 동기화 전략(만료 시간, 강제 새로고침 명령 등)을 명시해야 한다 (MUST).
- 수동 조정(예: 관리자가 DB에서 직접 수정)이 필요한 경우, 그 사실과 이유를 남길 수 있는 로그·노트 필드를 제공해야 한다 (SHOULD, 구현 비용이 과도한 경우 예외 사유를 기록).
- 근거: 여러 소스에 스케줄 정보가 흩어지면 이중 예약·누락이 발생하고, 사용자가 어떤 정보를 믿어야 할지 알 수 없다.

### III. 공정성·투명성 (NON-NEGOTIABLE)

- 모든 사용자는 현재·향후 GPU 예약 상황을 한두 번의 Slack 명령으로 조회할 수 있어야 한다 (MUST).
- 팀 차원의 공정성 설정(예: 사용자별/팀별 최대 동시 예약 수, 최대 선예약 기간, 밤샘 예약 정책 등)은 구성 가능해야 하며, 실제 적용 중인 규칙은 언제든 확인 가능해야 한다 (MUST).
- 예약 생성·변경·취소에는 최소한 "누가, 언제, 어떤 GPU/슬롯에 대해" 변경했는지가 기록되어야 한다 (SHOULD, 로그 또는 감사용 메타데이터 형태).
- 새로운 기능이 특정 사용자·팀에 유리하게 동작할 수 있는 경우, 스펙에 해당 편향·규칙을 명시하고 수용 가능한지 논의해야 한다 (MUST).
- 근거: GPU는 비용이 크고 희소한 자원이므로, 사용자는 본인이 공정하게 취급되고 있다는 사실을 이해할 수 있어야 한다.

### IV. 단순성·구성 가능성

- 기본 동작은 가능한 한 단순한 규칙(예: "선착순", "캘린더와 동일한 타임 슬롯 단위")으로 유지하고, 특별 규칙은 명시적 구성 옵션으로만 추가한다 (MUST).
- 설정은 소수의 핵심 개념(예: GPU 그룹, 예약 단위 시간, 공정성 정책, 근무 시간대)으로 표현하며, 중복되거나 서로 상충하는 설정을 만들지 않는다 (MUST).
- 새로운 설정을 추가할 때는 기본값·적용 범위·비활성화 방법을 스펙에 함께 정의해야 한다 (MUST).
- 근거: 과도한 설정·예외 규칙은 팀이 유지하기 어렵고, 실제로는 사용되지 않는 옵션이 증가해 혼란만 만든다.

### V. 신뢰성·관측 가능성

- 예약 관련 명령은 네트워크·외부 API 문제를 고려하여 재시도·타임아웃·사용자에게 보이는 실패 메시지를 명확히 정의해야 한다 (MUST).
- 같은 명령이 반복 실행되더라도 예약 상태가 꼬이지 않도록, 중요한 작업(예약·취소·변경)은 멱등성을 갖거나 중복 요청을 안전하게 처리해야 한다 (MUST).
- 최소한 "언제 어떤 명령이 어떤 파라미터로 호출되었고, 어떤 결과(성공/실패·에러 코드)를 가졌는지"를 추적 가능한 로그 또는 이벤트 형태로 남겨야 한다 (MUST).
- 근거: 예약 실패·중복 처리·외부 시스템 장애를 빠르게 파악·복구하지 못하면 사용자가 금방 신뢰를 잃고 수동 조정에 의존하게 된다.

## Scope, Non-Goals, and Constraints

- 이 봇은 OS·드라이버·컨테이너 레벨에서 GPU 사용을 강제 제한하거나, 특정 사용자만 GPU를 실제로 쓰게 막는 기능을 제공하지 않는다 (NON-GOAL, MUST NOT).
- 이 봇의 역할은 "사람 간 GPU 사용 약속과 스케줄 관리"에 한정되며, 실제 자원 제어는 클러스터·컨테이너·잡 스케줄러 등의 외부 시스템에 맡긴다.
- 모든 사용자-facing 설명(README, 온보딩 메시지, 주요 명령 도움말 등)에는 이 비강제적 특성(스케줄 조율용 봇이라는 점)을 명시해야 한다 (MUST).
- 초기 목표 스케일은 대략 "GPU 1–10대, 사용자 5–50명 수준의 팀"으로 가정하며, 이를 넘어서는 사용량(수백 대의 GPU, 수백 명의 사용자)이 예상될 경우에는 스펙 단계에서 확장성 요구사항을 명시적으로 정의해야 한다 (MUST).
- 성능 목표가 명시되지 않은 경우 기본 전제는 "일반적인 명령(조회·예약·취소) 응답 시간 p95 < 2초, Slack 레이트 리밋을 초과하지 않는 범위"로 한다 (DEFAULT CONSTRAINT).

## Development Workflow & Quality Gates

- 새로운 기능은 항상 `spec.md → plan.md → tasks.md → 구현` 순으로 진행하며, 각 단계에서 이 헌장의 원칙 위반 여부를 검토해야 한다 (MUST).
- 모든 스펙(spec.md)의 사용자 스토리에는 해당 스토리가 Slack 상에서 어떻게 표현·실행되는지(명령 예시, 인터랙티브 메시지 흐름 포함)를 명시해야 한다 (MUST).
- 스펙에는 최소한 다음을 포함해야 한다 (MUST):
  - 예약·취소·변경의 기본 흐름과 실패 케이스(예: 중복 예약, 이미 시작된 예약, 정책 위반).
  - 공정성 정책과의 관계(예: 선착순 vs 우선순위, 팀별 쿼터, 야간·주말 정책).
  - 단일 소스(데이터 저장소 또는 외부 API)와의 상호 작용 방식 및 실패 시 동작.
- 플랜(plan.md)의 "Constitution Check" 섹션에는 각 핵심 원칙(I–V)에 대해 이 기능이 어떻게 이를 만족하거나, 불가피하게 위반하는지와 그 완화책을 간단히 기록해야 한다 (MUST).
- 테스트 자동화가 과도한 비용을 요구하는 경우라도, 최소한 스펙의 수용 기준(acceptance scenarios)에 따라 수동 테스트 절차를 정의하고, 주요 시나리오(행복 경로 + 1–2개의 대표 실패 케이스)는 재현 가능해야 한다 (MUST).

## Governance

- 이 헌장은 GPU Schedule Bot 프로젝트에서 GPU 예약·스케줄 관련 기능을 설계·구현·운영할 때 다른 모든 관행보다 우선한다.
- 새로운 기능·아키텍처 변경·외부 시스템 통합이 이 헌장과 충돌할 경우, 먼저 헌장을 개정(또는 예외를 명시)한 뒤 구현을 진행해야 한다 (MUST).
- 헌장 개정은 항상 이 파일에 대한 PR로 이루어지며, 다음 버전 규칙을 따른다 (MUST):
  - MAJOR: 원칙의 제거·근본적인 의미 변경, 범위·비목표(Scope/Non-Goals) 재정의와 같이 기존 설계 가정을 깨는 변경.
  - MINOR: 새로운 원칙·섹션 추가, 기존 원칙의 실질적인 확대 적용.
  - PATCH: 표현 개선, 오탈자 수정, 의미를 바꾸지 않는 예시·설명 보강.
- Ratified 날짜는 최초 채택 일자를, Last Amended 날짜는 마지막으로 PR이 머지된 일자를 의미하며, 버전 변경 시 항상 함께 갱신해야 한다 (MUST).
- 코드 리뷰 시에는 최소한 다음을 확인해야 한다 (MUST):
  - 변경된 기능이 Slack-First UX, 단일 소스, 공정성·투명성, 단순성, 신뢰성·관측 가능성 원칙 중 어느 것에 영향을 주는지.
  - 스펙·플랜·태스크 문서가 이 헌장에 맞게 업데이트되었는지.
  - 위반이 필요한 경우, 위반 사유와 추후 개선 계획이 기록되었는지.
- 외부 요구사항(예: 특정 연구팀의 특수한 예약 규칙, 기업 보안 정책 등)이 이 헌장과 상충하는 경우, 임시 예외를 허용할 수 있으나, 해당 예외는 별도 문서로 기록하고 다음 MAJOR 또는 MINOR 버전 개정 시 통합 여부를 검토해야 한다.

**Version**: 1.0.0 | **Ratified**: 2025-11-22 | **Last Amended**: 2025-11-22
