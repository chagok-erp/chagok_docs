# 차곡ERP 문서 안내

문서를 용도별로 정리했습니다. 아래 경로는 이 문서를 기준으로 합니다.

| 폴더 | 문서 |
|---|---|
| planning | [01-PRD.md](planning/01-PRD.md) · [02-MVP-SCOPE.md](planning/02-MVP-SCOPE.md) · [03-ROLE-AGREEMENT.md](planning/03-ROLE-AGREEMENT.md) · [FUNCTIONAL-SPEC.md](planning/FUNCTIONAL-SPEC.md) · [FUNCTION-COVERAGE.md](planning/FUNCTION-COVERAGE.md) · [차곡ERP_통합기획서_v3.3.md](planning/차곡ERP_통합기획서_v3.3.md) |
| business | [04-ERP-WORKFLOW.md](business/04-ERP-WORKFLOW.md) · [06-ERP-BUSINESS-RULES.md](business/06-ERP-BUSINESS-RULES.md) · [procurement-flow.mmd](business/procurement-flow.mmd) |
| design | [07-SITEMAP-SCREEN-LIST.md](design/07-SITEMAP-SCREEN-LIST.md) · [08-USER-SCENARIOS.md](design/08-USER-SCENARIOS.md) · [SCREEN-SPEC.md](design/SCREEN-SPEC.md) |
| api | [openapi.yaml](api/openapi.yaml) · [API-OWNERSHIP.md](api/API-OWNERSHIP.md) · [09-PO-GR-CONTRACT.md](api/09-PO-GR-CONTRACT.md) · [10-AI-PR-DRAFT-CONTRACT.md](api/10-AI-PR-DRAFT-CONTRACT.md) |
| database | [DB-SPEC.md](database/DB-SPEC.md) · [ERD-core.mmd](database/ERD-core.mmd) |
| security | [RBAC-MATRIX.md](security/RBAC-MATRIX.md) |
| project-management | [11-WEEK1-REVIEW-DECISIONS.md](project-management/11-WEEK1-REVIEW-DECISIONS.md) · [OPEN-DECISIONS.md](project-management/OPEN-DECISIONS.md) · [GITHUB-SETUP.md](project-management/GITHUB-SETUP.md) · [VALIDATION-RESULTS.md](project-management/VALIDATION-RESULTS.md) |
| references | [05-SAP-REFERENCE.md](references/05-SAP-REFERENCE.md) |

저장소 작업 규칙은 [CONTRIBUTING.md](../CONTRIBUTING.md), PR 템플릿은 [pull_request_template.md](../.github/pull_request_template.md)를 참고하세요. 편집기 설정 `.editorconfig`와 Java 코드 스타일 `WootecoStyle.xml`은 저장소 루트에 있습니다.

## 기존 패키지 설명

아래는 기존 설계·개정 기록입니다. 작성 당시의 진행 상태와 검증 기록을 보존합니다.

# 차곡ERP — 2일 차 부분 개정 + 3일 차 기능 명세 패키지

**문서 기준일:** 2026-11-16  
**기준 제품 문서:** 통합 기획서 v3.2, PRD v0.2, MVP 범위표 v0.2, 역할 분담서 v0.2

## 문서 배치

현재 문서 배치는 상단 폴더별 안내를 참고하세요.

## 2일 차 변경 범위

- **유지:** PR → 승인/반려 → 내부 PO → 수동·부분 GR → 현재고/원장, 서버 권한 및 동시성·멱등성 원칙.
- **보완:** SAP EDMX 원본에서 직접 확인한 PR/PO/GR **품목별 키·참조**, Product 기본/구매 단위, Business Partner 연락처 관계, Material Stock의 실제 재고 수량 엔티티.
- **새 규칙 초안:** `BR-019~23`, 수용 테스트 `TC-17~20`. 팀 합의 전 상태·DB/DTO는 미확정.
- **ERD는 아직 작성 대상:** 현재 흐름도는 개념적 데이터 관계만 제안. 정식 ERD는 후속 `database/DB-SPEC.md`에서 작성.

## SAP 원본 파일

업로드된 다음 EDMX **6개를 직접 파싱**했다. 원본은 패키지에 중복 포함하지 않았다.

- `CE_PURCHASEREQUISITION_0001.edmx` (V4)
- `CE_PURCHASEORDER_0001.edmx` (V4)
- `API_MATERIAL_DOCUMENT_SRV.edmx` (V2)
- `API_MATERIAL_STOCK_SRV.edmx` (V2)
- `API_PRODUCT_SRV.edmx` (V2)
- `API_BUSINESS_PARTNER.edmx` (V2)

SAP EDMX가 제공하는 것은 **API 모델·필드·관계**이고, 차곡ERP는 이를 참고한 **자체 구현**이다. 실제 SAP 서버 연동, 샌드박스 API 호출 및 SAP 동작 테스트는 하지 않았다.

## 3일 차 작성 결과

- `planning/FUNCTIONAL-SPEC.md`의 **모든 P0 40개 항목**에 담당·우선순위·입력·출력·예외·완료 기준 포함.
- MVP v0.2의 ID와 1:1 대응. **문서 누락은 0개**, 상세 설계 미결정 `D-01~11`은 존재.
- P1/P2는 착수 전 상세화할 후보 목록만 보존. 이번 5개월 MVP에 무리하게 추가하지 않음.

## 권장 다음 작업

1. A/B가 기능별 완료 기준·업무 정책 미결정표 검토.
2. 4일 차 사이트맵 및 화면 흐름/AI Studio 프로토타입 구성.
3. 이후 ERD·API 계약·권한 매트릭스·서비스 계약 구체화.

**주의:** 문서 작성 완료와 실제 서비스 구현 완료는 다르다. 현재는 팀 검토용 명세 초안이다.

# 차곡ERP — D-01~D-11 결정 반영 개정본 (2026-11-17)

## 이번에 채택한 정책

- **D-01:** PR 1개 → PO 여러 개, PO당 공급업체 한 곳. **동일 PR 품목을 여러 PO로 수량 분할**하고, 유효 PO 발주 합계는 해당 PR 품목 승인 수량 이내로 제한한다.
- **D-02:** PR **헤더 전체 일괄 승인/반려**. 품목별 승인·수량 조정 승인 제외.
- **D-03~D-11:** 이전 추천 방향을 개발자 A가 수용했다. 품목 기준단위/Decimal·KRW, OPENING 재고 원장, PO↔GR 잠금·멱등·단일 트랜잭션, 사용자 입력 임계값, 제한된 문서 양식, 버전 있는 RAG 규정, Spring Security 세션 우선, AI PoC 후 기술 선택·ES 색인 후처리·실패 복구, 계약 우선 점진 설계.

**확정의 범위:** 대화에서 **A가 선택한 정책**을 문서에 반영했다. **B의 문서 리뷰·서명, 실제 API/ERD/수량 정밀도·오류 코드·AI 프레임워크 결정은 남아 있다.** 실제 구현·테스트·SAP 서버 연동은 수행하지 않았다.

## 파일 목록 — 변경된 문서 13개

| 구분 | 파일 | 반영 요약 |
|---|---|---|
| 통합 | [기획서 v3.3](planning/차곡ERP_통합기획서_v3.3.md) | 분할 발주·일괄 승인 및 D-03~11 정책 |
| 1일 차 | [PRD v0.3](planning/01-PRD.md) | 분할 발주 사용자 시나리오·수용 조건 |
| 1일 차 | [MVP 범위표 v0.3](planning/02-MVP-SCOPE.md) | PO-01 완료 조건·테스트 |
| 1일 차 | [역할 분담서 v0.3](planning/03-ROLE-AGREEMENT.md) | A의 PR→PO 분할 발주 정합성 책임 |
| 2일 차 | [ERP 업무 흐름 v0.3](business/04-ERP-WORKFLOW.md) | 여러 PO 연결·완료 상태 오류 수정 |
| 2일 차 | [업무 규칙 v0.3](business/06-ERP-BUSINESS-RULES.md) | BR-005 수정·BR-024~27 및 TC-21~25 |
| 3일 차 | [기능 명세 v0.2](planning/FUNCTIONAL-SPEC.md) | PO-01·APR-01 및 결정 D-01~11 반영 |
| 3일 차 | [기능 커버리지 v0.2](planning/FUNCTION-COVERAGE.md) | 40개 기능 ID 유지, 미정 사항 분리 |
| 4일 차 | [화면 목록 v0.2](design/07-SITEMAP-SCREEN-LIST.md) | PR 미발주 vs PO 미입고 잔량 분리 |
| 4일 차 | [사용자 시나리오 v0.2](design/08-USER-SCENARIOS.md) | US-01A 분할 발주 추가 |
| 5일 차 | [PO↔GR 계약 v0.2](api/09-PO-GR-CONTRACT.md) | 분할 PO 각각 독립 GR 수령 |
| 5일 차 | [AI↔PR 계약 v0.2](api/10-AI-PR-DRAFT-CONTRACT.md) | Decimal·세션 인증 우선, 초안/분할발주 경계 |
| 5일 차 | [1주 차 결정 기록 v0.2](project-management/11-WEEK1-REVIEW-DECISIONS.md) | D-01~11 선택 사항과 2주 차 검증 |

**참고 파일(내용 수정 없음):** [SAP EDMX 6종 조사 v0.2](references/05-SAP-REFERENCE.md). API 사실관계는 업무 정책 선택으로 달라지지 않으므로 원문을 유지했다.

## 2주 차 우선 구현 검증

1. `RequisitionItem` **1:N** `PurchaseOrderItem` FK와 수량 합계 제약(애플리케이션 트랜잭션/잠금/요청키)을 ERD/API 명세에 반영한다.
2. PO 생성과 PO 기반 GR을 **각기 다른 트랜잭션**으로 구현한다. GR 등록의 원자성(PO 수령량+GR+원장+현재고)은 유지한다.
3. `100 EA PR → PO-A 60 + PO-B 40` 성공, `추가 PO 1` 거절, `동시 PO 60+60` 초과 방지, `GR 60+40` 및 롤백 테스트를 PostgreSQL에서 검증한다.
4. 통합 AI Assistant, 인증·디자인 및 공통 운영 역할은 변경하지 않는다.


# 차곡ERP — 2주 차 설계 및 GitHub 조직 협업 준비 패키지

> 2026-11-17 / 설계 초안. **실제 GitHub 조직/저장소 생성, API 서버·DB 적용, React 실행, 팀 B 승인/서명은 아직 수행하지 않음.**

## 파일 목록

| 경로 | 용도 |
|---|---|
| `database/DB-SPEC.md` | PostgreSQL 초기 데이터 사전, ERD, PK/FK/유니크, 트랜잭션/동시성 원칙, 테스트 기준 |
| `database/ERD-core.mmd` | Mermaid ERD 원문(렌더링 도구/Markdown에서 확인) |
| `api/openapi.yaml` | **OpenAPI 3.1** 초기 계약 초안. 29 경로/37 동작, JSON 수량 Decimal 문자열 제안 |
| `security/RBAC-MATRIX.md` | 역할별 R/C/U, 자기 승인, 소유권, 세션/CSRF/AI 보안 경계 |
| `design/SCREEN-SPEC.md` | 전달받은 `design/SCREEN-SPEC.md` v0.1을 최소 변경으로 개정한 **v0.2**; 실제 AI Studio 코드 변경 없음 |
| `project-management/OPEN-DECISIONS.md` | 개발자 A/B가 최종 선택·확인할 15개 세부 항목 |
| `project-management/GITHUB-SETUP.md` | GitHub 조직 생성, 초대, Team, 권한, 2FA, 브랜치 보호, PR·Actions 안내 |
| `github/TEAM-OPERATING-AGREEMENT.md` | 동등한 두 공동 개발자를 위한 운영 합의 초안(미서명) |
| `../CONTRIBUTING.md` | 제공된 원본 협업 규칙 복사본(변경 없음) |
| `../.github/pull_request_template.md` | 제공된 원본 PR 템플릿 복사본. 저장소에서는 `../.github/pull_request_template.md`로 이동 |
| `../.editorconfig` | 제공된 원본 편집기 설정 복사본(변경 없음) |

## 주요 설계 원칙

- **기확정 방향:** PR 헤더 일괄 승인, 1 PR에서 여러 PO, PO마다 공급업체 1곳, 부분 입고, AI는 DRAFT까지만.
- **제안 세부:** `NUMERIC(18,3)` 수량, KRW 정수 단가, PR DRAFT 필요일 선택/제출 필수, 공급업체 이메일 선택, ADMIN 자기 승인 금지, 기본 RBAC 매트릭스 등. 팀 B의 확인이 남아 있음.
- **불변식:** PR 품목 승인 수량 ≥ 그 품목에 배정된 유효 PO 발주 수량 총합, PO 품목 발주량 ≥ 누적 GR 입고량, 재고 현재고 = 개시재고 + 원장 이동 합계.
- **단위:** 품목별 기준단위만 사용, 서로 다른 단위의 수량을 합산하지 않음.
- **트랜잭션:** PO 생성에서 PR 품목 잠금. GR 확정에서 PO 헤더·품목 잠금과 GR/원장/현재고 동일 PostgreSQL 트랜잭션.
- **책임:** A 품목·공급업체·PR/PO·Agent·공통 UI 디자인, B 창고·GR·Stock·Document AI·인증, CI/CD·배포·모니터링·PostgreSQL 운영 공동.

## 전달받은 SCREEN-SPEC v0.1에서 수정·정리한 사항

1. 문서 버전 v0.2 및 개정 메모. **원래 프로토타입이 수정됐다고 주장하지 않음.**
2. 혼합 단위 품목의 발주 가능 여부는 `remainingToOrderTotal` 합산 대신 품목별 잔량 `>0` 존재 검사.
3. 수량 정책 `EA/SET` 정수 + `KG/M/L` 3자리 소수 **제안**.
4. `neededByDate` DRAFT 선택/제출 필수, 이메일 선택, ADMIN 자기 승인 거부 **제안** 표시.
5. 문서 입력 `.xlsx` + 텍스트 PDF P0 **제안**, CSV/XLS와 OCR 제외.
6. ES/LLM 품목 추천에 검증되지 않은 '신뢰도 %'를 사용하지 않도록 변경.
7. 기능 매핑 직접 화면 항목 '29' 오타를 '30'으로 교정.
8. 기존 API `/api/...`는 Mock 메모이고 새 OpenAPI `/api/v1`로 팀 합의 후 통일 필요.

## 현재 확인한 것과 미확인

**문서 검증:** YAML 파싱, 경로/메서드 개수·operationId 고유성·참조 스키마, Markdown 파일 존재, ZIP 무결성을 자동 점검할 계획.

**미실행:** 실제 GitHub 조직 생성, GitHub 팀원 초대/권한 변경, DB 마이그레이션/SQL 실행, Spring Boot 빌드·테스트, AI Studio React 프로토타입 코드 실행, 실제 동시성 및 인증 테스트.

## 사용 순서

1. `project-management/OPEN-DECISIONS.md`를 A/B 함께 검토해 세부값 확정.
2. `database/DB-SPEC.md`의 핵심 PK/FK·잠금/트랜잭션과 `api/openapi.yaml`의 필드/에러/권한을 교차 리뷰.
3. `design/SCREEN-SPEC.md`를 실제 React 프로토타입과 비교해 UI 수정.
4. `project-management/GITHUB-SETUP.md`를 따라 조직·저장소·보안·팀원 권한 설정(실제 관리자 계정 필요).
5. 팀 확정 후 첫 수직 기능(Material/PR DRAFT 조회·저장) 구현 및 DB 마이그레이션/테스트, 계약서 갱신.


# 차곡ERP — 2주 차 설계 및 GitHub 조직 협업 준비 패키지

> 2026-11-17 / 설계 초안. **실제 GitHub 조직/저장소 생성, API 서버·DB 적용, React 실행, 팀 B 승인/서명은 아직 수행하지 않음.**

## 파일 목록

| 경로 | 용도 |
|---|---|
| `docs/DB-SPEC.md` | PostgreSQL 초기 데이터 사전, ERD, PK/FK/유니크, 트랜잭션/동시성 원칙, 테스트 기준 |
| `diagrams/ERD-core.mmd` | Mermaid ERD 원문(렌더링 도구/Markdown에서 확인) |
| `docs/openapi.yaml` | **OpenAPI 3.1** 초기 계약 초안. 29 경로/37 동작, JSON 수량 Decimal 문자열 제안 |
| `docs/RBAC-MATRIX.md` | 역할별 R/C/U, 자기 승인, 소유권, 세션/CSRF/AI 보안 경계 |
| `docs/SCREEN-SPEC.md` | 전달받은 `SCREEN-SPEC.md` v0.1을 최소 변경으로 개정한 **v0.2**; 실제 AI Studio 코드 변경 없음 |
| `docs/OPEN-DECISIONS.md` | 개발자 A/B가 최종 선택·확인할 15개 세부 항목 |
| `github/GITHUB-SETUP.md` | GitHub 조직 생성, 초대, Team, 권한, 2FA, 브랜치 보호, PR·Actions 안내 |
| `github/TEAM-OPERATING-AGREEMENT.md` | 동등한 두 공동 개발자를 위한 운영 합의 초안(미서명) |
| `github/CONTRIBUTING.md` | 제공된 원본 협업 규칙 복사본(변경 없음) |
| `github/pull_request_template.md` | 제공된 원본 PR 템플릿 복사본. 저장소에서는 `.github/pull_request_template.md`로 이동 |
| `github/.editorconfig` | 제공된 원본 편집기 설정 복사본(변경 없음) |

## 주요 설계 원칙

- **기확정 방향:** PR 헤더 일괄 승인, 1 PR에서 여러 PO, PO마다 공급업체 1곳, 부분 입고, AI는 DRAFT까지만.
- **제안 세부:** `NUMERIC(18,3)` 수량, KRW 정수 단가, PR DRAFT 필요일 선택/제출 필수, 공급업체 이메일 선택, ADMIN 자기 승인 금지, 기본 RBAC 매트릭스 등. 팀 B의 확인이 남아 있음.
- **불변식:** PR 품목 승인 수량 ≥ 그 품목에 배정된 유효 PO 발주 수량 총합, PO 품목 발주량 ≥ 누적 GR 입고량, 재고 현재고 = 개시재고 + 원장 이동 합계.
- **단위:** 품목별 기준단위만 사용, 서로 다른 단위의 수량을 합산하지 않음.
- **트랜잭션:** PO 생성에서 PR 품목 잠금. GR 확정에서 PO 헤더·품목 잠금과 GR/원장/현재고 동일 PostgreSQL 트랜잭션.
- **책임:** A 품목·공급업체·PR/PO·Agent·공통 UI 디자인, B 창고·GR·Stock·Document AI·인증, CI/CD·배포·모니터링·PostgreSQL 운영 공동.

## 전달받은 SCREEN-SPEC v0.1에서 수정·정리한 사항

1. 문서 버전 v0.2 및 개정 메모. **원래 프로토타입이 수정됐다고 주장하지 않음.**
2. 혼합 단위 품목의 발주 가능 여부는 `remainingToOrderTotal` 합산 대신 품목별 잔량 `>0` 존재 검사.
3. 수량 정책 `EA/SET` 정수 + `KG/M/L` 3자리 소수 **제안**.
4. `neededByDate` DRAFT 선택/제출 필수, 이메일 선택, ADMIN 자기 승인 거부 **제안** 표시.
5. 문서 입력 `.xlsx` + 텍스트 PDF P0 **제안**, CSV/XLS와 OCR 제외.
6. ES/LLM 품목 추천에 검증되지 않은 '신뢰도 %'를 사용하지 않도록 변경.
7. 기능 매핑 직접 화면 항목 '29' 오타를 '30'으로 교정.
8. 기존 API `/api/...`는 Mock 메모이고 새 OpenAPI `/api/v1`로 팀 합의 후 통일 필요.

## 현재 확인한 것과 미확인

**문서 검증:** YAML 파싱, 경로/메서드 개수·operationId 고유성·참조 스키마, Markdown 파일 존재, ZIP 무결성을 자동 점검할 계획.

**미실행:** 실제 GitHub 조직 생성, GitHub 팀원 초대/권한 변경, DB 마이그레이션/SQL 실행, Spring Boot 빌드·테스트, AI Studio React 프로토타입 코드 실행, 실제 동시성 및 인증 테스트.

## 사용 순서

1. `docs/OPEN-DECISIONS.md`를 A/B 함께 검토해 세부값 확정.
2. `docs/DB-SPEC.md`의 핵심 PK/FK·잠금/트랜잭션과 `docs/openapi.yaml`의 필드/에러/권한을 교차 리뷰.
3. `docs/SCREEN-SPEC.md`를 실제 React 프로토타입과 비교해 UI 수정.
4. `github/GITHUB-SETUP.md`를 따라 조직·저장소·보안·팀원 권한 설정(실제 관리자 계정 필요).
5. 팀 확정 후 첫 수직 기능(Material/PR DRAFT 조회·저장) 구현 및 DB 마이그레이션/테스트, 계약서 갱신.


## 2026-11-17 A 채택안 덮어쓰기(v0.2)

기존 `docs/OPEN-DECISIONS.md`를 A 채택안으로 덮어쓰고 DB/RBAC/SCREEN/API-OWNERSHIP/OpenAPI를 함께 갱신했다. `openapi.yaml`의 GR 요청은 품목별 `storageLocationId`를 받도록 변경했다. 팀원 B의 최종 리뷰, DB 마이그레이션, 실제 API/React 테스트는 미완료다.
