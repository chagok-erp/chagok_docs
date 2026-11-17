# 차곡ERP REST API 구현 책임·리뷰 매핑 (API-OWNERSHIP)

> **v0.1 · 2026-11-17 · A/B 검토용**
> 기준: 최신 통합 기획서 v3.3, `../planning/FUNCTIONAL-SPEC.md` v0.2, `../planning/03-ROLE-AGREEMENT.md` v0.3, `openapi.yaml` v0.1.0-draft, `../security/RBAC-MATRIX.md` v0.1.
> **의미:** 기능/도메인 담당(A/B)은 기존 합의에 따르며, 이 문서의 **API별 리뷰 지정·DTO 검토 항목은 제안**이다. 실제 구현 완료나 팀 최종 승인 상태가 아니다.

## 1. 목적과 집계

- REST API별 **기능 ID → API operationId → 서버 구현 담당 → 교차 리뷰 담당 → 화면/AI 소비자** 연결을 제공한다.
- 기존 OpenAPI를 변경하지 않는다. 아래 표는 실제 `openapi.yaml`에 존재하는 Operation만 열거한다.
- 본인=**A(구매 ERP·Agent·공통 디자인)**, 팀원=**B(인증·입고/재고·Document AI)**. **공동 인프라·통합 테스트는 A+B**이다.
- 공통 인프라와 도메인 구현을 혼동하지 않는다: `SEC-01` 기반은 B, `SEC-02`의 각 도메인 권한 검증은 A/B 각 소유자. `AUD-01` 기록은 도메인별 A/B, 조회 집계 API는 공동 검토.

| 항목 | 정확한 집계 |
|---|---:|
| OpenAPI 경로 (`paths`) | **29개** |
| HTTP 동작 (`operationId`) | **37개** |
| A 주 구현 | **20개** |
| B 주 구현 | **16개** |
| A+B 공통 조회·통합 API | **1개** |
| 변경된 기존 `openapi.yaml` | **없음** |

**주의:** `PO-03`(A의 PO 입고 가능량 검증·누적량 갱신)은 별도 REST Endpoint가 아니라 **B의 `POST /goods-receipts` 내부에서 같은 Spring Boot/PostgreSQL 트랜잭션으로 호출하는 Java 서비스**다. 이를 새 HTTP 호출로 분리하지 않는다.

## 2. 전 37개 REST API 소유권 매핑

**표의 경로는 `openapi.yaml` `servers: /api/v1`를 제외한 상대 경로**다. `기능 ID` 중 `AUD-01` 등은 처리 결과로 발생하는 부가 기능을 포함한다. `요청/성공 응답`은 OpenAPI의 현재 정의를 읽은 것으로 팀 합의 전에 바뀔 수 있다.

### 공통 인증·사용자 (B 구현 / A 리뷰)

| # | Method · 경로 | operationId | 관련 기능 ID | 구현 / 리뷰 | 요청 → 성공 응답 | 핵심 계약 및 주의점 |
|---:|---|---|---|---|---|---|
| 01 | `POST /auth/login` | `post_auth_login` | `SEC-01`, `UI-04` | **B** / A | `LoginRequest` → `200` `User` | 공통 Spring Security 세션; CSRF 정책 검토 |
| 02 | `POST /auth/logout` | `post_auth_logout` | `SEC-01`, `UI-04` | **B** / A | — → `204` — | 세션 무효화·쿠키 처리 |
| 03 | `GET /auth/me` | `get_auth_me` | `SEC-01`, `SEC-02` | **B** / A | — → `200` `User` | 서버 인증 주체가 유일한 신원 근거 |

### 품목·공급업체 기준정보 (A 구현 / B 리뷰)

| # | Method · 경로 | operationId | 관련 기능 ID | 구현 / 리뷰 | 요청 → 성공 응답 | 핵심 계약 및 주의점 |
|---:|---|---|---|---|---|---|
| 04 | `GET /materials` | `get_materials` | `MAT-01` | **A** / B | — → `200` `Material[]` | B의 문서 매칭·색인에서도 소비 |
| 05 | `POST /materials` | `post_materials` | `MAT-01`, `AUD-01` | **A** / B | `MaterialCreate` → `201` `Material` | 중복 품목코드·기준단위·검색 색인 재처리 |
| 06 | `PATCH /materials/{id}` | `patch_materials_by_id` | `MAT-01` | **A** / B | `object(name, specification, referenceUnitPrice, safetyStockQty)` → `200` `Material` | 이미 참조 중인 단위 변경 금지/미지원 |
| 07 | `PATCH /materials/{id}/active` | `patch_materials_by_id_active` | `MAT-01` | **A** / B | `object(active)` → `200` `Material` | 신규 PR·PO·매칭 선택 가능 여부와 연동 |
| 08 | `GET /suppliers` | `get_suppliers` | `SUP-01` | **A** / B | — → `200` `Supplier[]` | PO 생성용 활성 공급업체 조회 |
| 09 | `POST /suppliers` | `post_suppliers` | `SUP-01` | **A** / B | `SupplierCreate` → `201` `Supplier` | 전화번호 필수·이메일 선택은 X-05 검토 |
| 10 | `PATCH /suppliers/{id}` | `patch_suppliers_by_id` | `SUP-01` | **A** / B | `object(name, contactName, phone, email, active)` → `200` `Supplier` | 비활성화·기존 발주 이력 보존 |

### 창고·저장위치 기준정보 (B 구현 / A 리뷰)

| # | Method · 경로 | operationId | 관련 기능 ID | 구현 / 리뷰 | 요청 → 성공 응답 | 핵심 계약 및 주의점 |
|---:|---|---|---|---|---|---|
| 11 | `GET /warehouses` | `get_warehouses` | `WH-01` | **B** / A | — → `200` `Warehouse[]` | A의 공통 UI와 조회 계약 연동 |
| 12 | `POST /warehouses` | `post_warehouses` | `WH-01` | **B** / A | `WarehouseCreate` → `201` `Warehouse` | 창고 CRUD는 P0 기능 범위 |
| 13 | `PATCH /warehouses/{id}` | `patch_warehouses_by_id` | `WH-01` | **B** / A | `object(name, active)` → `200` `Warehouse` | 활성 상태 변경 시 기존 재고 참조 보존 |
| 14 | `GET /storage-locations` | `get_storage_locations` | `WH-01` | **B** / A | — → `200` `StorageLocation[]` | 창고별 소속 관계 필터 확인 |
| 15 | `POST /storage-locations` | `post_storage_locations` | `WH-01` | **B** / A | `StorageLocationCreate` → `201` `StorageLocation` | 창고 소속·코드 중복 검증 |
| 16 | `PATCH /storage-locations/{id}` | `patch_storage_locations_by_id` | `WH-01` | **B** / A | `object(name, active)` → `200` `StorageLocation` | 비활성 위치에는 신규 입고 금지 |

### 구매요청·일괄 승인 (A 구현 / B 리뷰)

| # | Method · 경로 | operationId | 관련 기능 ID | 구현 / 리뷰 | 요청 → 성공 응답 | 핵심 계약 및 주의점 |
|---:|---|---|---|---|---|---|
| 17 | `GET /purchase-requisitions` | `get_purchase_requisitions` | `PR-01`, `PR-02`, `SEC-02` | **A** / B | — → `200` `PRDraft[]` | 본인 DRAFT·승인 대상 조회 범위 X-07 |
| 18 | `POST /purchase-requisitions` | `post_purchase_requisitions` | `PR-01`, `DOC-04`, `AG-04` | **A** / B | `PRCreate` → `201` `PRDraft` | UI·Agent·문서 AI가 동일 PR 저장 서비스 소비 |
| 19 | `GET /purchase-requisitions/{id}` | `get_purchase_requisitions_by_id` | `PR-01`, `PR-02`, `PO-01` | **A** / B | — → `200` `PRDraft` | PR 품목별 미발주 잔량, 소유권/가시성 |
| 20 | `PATCH /purchase-requisitions/{id}` | `patch_purchase_requisitions_by_id` | `PR-01`, `AG-05` | **A** / B | `PRPatch` → `200` `PRDraft` | DRAFT만, requestKey+expectedVersion |
| 21 | `POST /purchase-requisitions/{id}/submit` | `post_purchase_requisitions_by_id_submit` | `PR-02` | **A** / B | — → `200` `PRDraft` | DRAFT→SUBMITTED, 필요일 X-04, 버전 입력 검토 |
| 22 | `POST /purchase-requisitions/{id}/rework` | `post_purchase_requisitions_by_id_rework` | `PR-02` | **A** / B | — → `200` `PRDraft` | REJECTED→DRAFT, 이력 유지, 중복 전이 검토 |
| 23 | `POST /purchase-requisitions/{id}/approve` | `post_purchase_requisitions_by_id_approve` | `APR-01` | **A** / B | `DecisionRequest` → `200` `PRDraft` | 헤더 일괄 승인·자기 승인 금지, 경합 검증 |
| 24 | `POST /purchase-requisitions/{id}/reject` | `post_purchase_requisitions_by_id_reject` | `APR-01` | **A** / B | `RejectRequest` → `200` `PRDraft` | 반려 사유 필수·자기 반려 정책 X-06 |

### 분할 발주·PO 조회 (A 구현 / B 리뷰)

| # | Method · 경로 | operationId | 관련 기능 ID | 구현 / 리뷰 | 요청 → 성공 응답 | 핵심 계약 및 주의점 |
|---:|---|---|---|---|---|---|
| 25 | `GET /purchase-orders` | `get_purchase_orders` | `PO-01` | **A** / B | — → `200` `PurchaseOrder[]` | 분할 발주 목록; GR의 PO 선택에도 사용 |
| 26 | `POST /purchase-orders` | `post_purchase_orders` | `PO-01` | **A** / B | `POCreate` → `201` `PurchaseOrder` | 동일 PR 여러 PO 허용; PR 품목 잠금·발주량·멱등키 |
| 27 | `GET /purchase-orders/{id}` | `get_purchase_orders_by_id` | `PO-01`, `PO-02`, `PO-03` | **A** / B | — → `200` `PurchaseOrder` | GR에서 미입고 잔량 조회, 단위별 계산 |

### 입고·재고 (B 구현 / A 리뷰; PO 내부 서비스는 A)

| # | Method · 경로 | operationId | 관련 기능 ID | 구현 / 리뷰 | 요청 → 성공 응답 | 핵심 계약 및 주의점 |
|---:|---|---|---|---|---|---|
| 28 | `GET /goods-receipts` | `get_goods_receipts` | `GR-01` | **B** / A | — → `200` `GoodsReceipt[]` | PO 기준 입고 이력; 화면은 B 소유 |
| 29 | `POST /goods-receipts` | `post_goods_receipts` | `GR-01`, `GR-02`, `PO-03`, `ST-01` | **B** / A | `GRCreate` → `201` `GoodsReceipt` | B 트랜잭션 오케스트레이터→A 내부 PO Service |
| 30 | `GET /goods-receipts/{id}` | `get_goods_receipts_by_id` | `GR-01` | **B** / A | — → `200` `GoodsReceipt` | 성공한 GR의 상세/원장 참조 |
| 31 | `GET /stock-balances` | `get_stock_balances` | `ST-01`, `AG-02` | **B** / A | — → `200` `StockBalance[]` | Agent 재고 조회 Tool이 B의 데이터 계약 소비 |
| 32 | `GET /stock-movements` | `get_stock_movements` | `ST-01` | **B** / A | — → `200` `StockMovement[]` | OPENING/GR_IN 불변 원장; 임의 수정 API 없음 |

### 감사 로그 (A+B 공동)

| # | Method · 경로 | operationId | 관련 기능 ID | 구현 / 리뷰 | 요청 → 성공 응답 | 핵심 계약 및 주의점 |
|---:|---|---|---|---|---|---|
| 33 | `GET /audit-logs` | `get_audit_logs` | `AUD-01` | **A+B** / A+B | — → `200` `AuditLog[]` | 도메인별 작성 A/B, 공통 조회 API 공동 검토·작업 이슈별 실행자 지정 |

### 대화형 Agent·RAG (A 구현 / B 리뷰)

| # | Method · 경로 | operationId | 관련 기능 ID | 구현 / 리뷰 | 요청 → 성공 응답 | 핵심 계약 및 주의점 |
|---:|---|---|---|---|---|---|
| 34 | `POST /assistant/chat` | `post_assistant_chat` | `AG-01`, `AG-02`, `AG-03`, `AG-04`, `AG-05`, `AG-06` | **A** / B | `AssistantChatRequest` → `200` `AssistantChatResponse` | AI 허용 Tool은 ERP 조회·RAG·PR DRAFT까지만 |
| 35 | `GET /assistant/sessions/{id}` | `get_assistant_sessions_by_id` | `AG-01`, `AG-05` | **A** / B | — → `200` `AssistantChatResponse` | 대화 소유권 및 후속 PR 참조 검증 |

### 문서 AI·품목 매칭 (B 구현 / A 리뷰)

| # | Method · 경로 | operationId | 관련 기능 ID | 구현 / 리뷰 | 요청 → 성공 응답 | 핵심 계약 및 주의점 |
|---:|---|---|---|---|---|---|
| 36 | `GET /documents/analysis/{id}` | `get_documents_analysis_by_id` | `DOC-02`, `DOC-03`, `MATCH-01`, `MATCH-02`, `MATCH-03` | **B** / A | — → `200` `DocumentAnalyzeResult` | 본인 분석 작업만 조회; 실제 조회 결과 형태 검토 |
| 37 | `POST /documents/analyze` | `post_documents_analyze` | `DOC-01`, `DOC-02`, `MATCH-01`, `MATCH-02`, `MATCH-03` | **B** / A | `multipart` `object(file)` → `200` `DocumentAnalyzeResult` | 제한된 XLSX·텍스트 PDF 파싱; 사용자 확정 전 PR 미저장 |

## 3. 인터페이스별 실제 구현 경계 — API 담당 ≠ 내부 데이터 소유권

| 연계 경계 | API 공개·최종 응답 책임 | 내부 구현/DB 책임 | 필수 소비자 검토 |
|---|---|---|---|
| 인증·보안 | B `Auth` Controller, 공통 필터 | B: User/Role·Spring Security / A·B: 자기 도메인 소유권 검증 | A: React 공통 Shell, Agent; B: 문서 AI·GR |
| Material → PR/품목 검색 | A `GET /materials`, `POST/PATCH /materials` | A: PostgreSQL Material 원본·값 검증 / B: ES BM25 파생 인덱스·동기화·복구 | B: 인덱스 반영 시점, 활성 품목·규격/단위 검증 |
| **PR 생성·수정 공통 서비스** | **A `POST/PATCH /purchase-requisitions`** | A: DRAFT 소유권, 수량·단위, `requestKey`, `expectedVersion`, 저장 | **B: 문서 검토 후 `materialId` 확정 결과를 공통 DTO로 전달** |
| PR → PO 분할 발주 | A `POST /purchase-orders` | A: PR Item 비관적 잠금, 품목별 승인 수량−유효 기발주량 검증, PO 멱등성 | B: PO Item/입고 가능량 조회 계약 |
| **PO → GR** | **B `POST /goods-receipts`** | **B: 트랜잭션·GR·재고 원장/현재고 / A: `PurchaseOrderReceivingService` PO 헤더/품목 잠금·수량 갱신** | **A+B: 같은 DB 트랜잭션과 실패 시 전체 롤백 테스트** |
| ERP 재고 조회 → Agent | B `GET /stock-balances` / A `POST /assistant/chat` | B: 창고·위치·현재고 정합성 / A: Tool Calling·문맥/응답 | A: 서버 인증 문맥 하에서 B API 조회, 임계값 사용자가 명시 |
| 문서 분석 → PR DRAFT | B `POST /documents/analyze`, `GET /documents/analysis/{id}` | B: Excel/PDF 추출·Rule/ES/LLM 후보·사용자 검토 패널 | A: 확정 Material ID·기준단위·버전·멱등성으로 PR 서비스 저장 |
| **감사 이력** | **A+B `GET /audit-logs` 공동 구현·통합 검토** | A: 구매/Agent 도메인 기록 / B: 인증·입고/재고·문서 도메인 기록 / 공통 로그 운영 A+B | 상세 `/audit-logs` 합본의 소유권·조회 범위는 A+B 공동 유지, PR 이슈별 구현 실행자는 별도 지정 |

### GR 처리에서 절대 나누면 안 되는 경계

```text
POST /api/v1/goods-receipts             # API/Controller: B
  GoodsReceiptApplicationService      # B, 트랜잭션 시작
    -> PurchaseOrderReceivingService  # A, 같은 Java 프로세스·트랜잭션
         PO 헤더/품목 잠금, 입고 가능 수량 검사, receivedQty 갱신
    -> GoodsReceipt/Item 저장         # B
    -> StockMovement 기록             # B
    -> StockBalance 갱신              # B
  COMMIT 또는 전체 ROLLBACK
```

`REQUIRES_NEW`, 메시지 큐 또는 AI 서비스 HTTP 호출로 이 트랜잭션을 분리하지 않는다. 이 제약은 기존 역할 분담과 `09-PO-GR-CONTRACT.md`에서 명시된 내용이다.

## 4. DTO·HTTP 오류 검토표 — A/B 회의용 (아직 합의·구현 확인 전)

| 리뷰 묶음 | 현재 OpenAPI 계약 | 반드시 검토할 항목 | 1차 책임 / 상대 리뷰 | 관련 X 항목 |
|---|---|---|---|---|
| 인증 | `LoginRequest` → `User`, 200/401/422; 전역 `sessionCookie` | 세션 쿠키·CSRF 토큰 전달과 401/403, 실제 부서/복수 역할/로그아웃 쿠키 처리 | B / A | X-07, X-13 |
| Material/Supplier | `MaterialCreate`, `SupplierCreate`, `Material`, `Supplier` | 기준단위 변경 허용 범위, 중복 코드, 이메일 선택 여부, 비활성 품목의 기존 이력 보존 | A / B | X-01~03, X-05 |
| Warehouse/Location | `WarehouseCreate`, `StorageLocationCreate` 및 PATCH | 기존 재고가 있는 창고/저장위치 비활성 정책·창고별 위치 코드 유일성 | B / A | X-12 |
| PR 신규/수정 | `PRCreate`, `PRPatch`, `PRDraft` | `requestKey`, `expectedVersion`, Source, DRAFT 필요일, 실제 itemId 매핑, 중복 품목 행 | A / B | X-04, X-08, X-11 |
| PR 제출/재작업/승인 | `POST .../submit|rework|approve|reject` | 상태 전이 경쟁, 승인 권한/자기 승인, **제출·승인 API 버전 조건 표기 여부** | A / B | X-06, X-07, X-11 |
| PO 분할 발주 | `POCreate` / `POItemCreate` / `PurchaseOrder` | 한 PR item을 여러 PO에서 참조, 공급업체 1개/PO, 잔량 계산, 멱등 요청키 재시도 | A / B | X-09, X-11, X-14 |
| **GR REST vs 내부 Java 계약** | `GRCreate`에 `warehouseId`, `storageLocationId`는 헤더, items는 id+receivedQty | **`09-PO-GR-CONTRACT.md` 개념 DTO는 품목별 단위·위치 포함. GR REST가 서버에서 PO 기준단위를 조회·검증할지, `unit`을 외부 요청에 둘지 결정** | B / A | X-10, X-11 |
| 현재고·원장 | `StockBalance`, `StockMovement` | 개시 재고 원장, 현재고 유일성, 자료 조회 범위, 단위별 합산 금지 | B / A | X-01~02, X-14 |
| Agent 채팅/세션 | `AssistantChatRequest`, `AssistantChatResponse` | 응답이 대화 전체 이력인지 최신 메시지인지, Tool 호출 시 중복 DRAFT 방지·소유권, Rust↔Java 인증 | A / B | X-11, X-13 |
| Document AI 파일/결과 | `multipart/form-data` 파일 → `DocumentAnalyzeResult`(200), 분석 `jobId` 조회 | 최초 응답이 **동기 결과 200**인지 **비동기 접수 202**인지, 지원 파일/크기, 재시도 및 비율 표기 | B / A | X-11~13 |
| 감사 로그 | `GET /audit-logs` (현행 `AuditLog[]`) | A/B 감사 이벤트 필수 필드·권한에 따른 필터·민감정보/문서 원문 비노출 | A+B / A+B | X-07, X-11 |
| 공통 오류 포맷 | `ApiError {code,message,traceId,fieldErrors}` | 401/403/404/409/422(파일 413/415, AI/문서 502) 매핑. 실제 사용하지 않는 오류코드 남용 제거 | A+B / A+B | X-11, X-14 |

### 교차 대조에서 발견한 구체적 계약 차이 (자동 수정하지 않음)

1. **PO↔GR 단위/위치:** 현재 `GRCreate`의 `items`는 `purchaseOrderItemId`·`receivedQty`만 요구하며 위치 정보는 헤더에 있다. 기존 `09-PO-GR-CONTRACT.md`의 개념적 명령은 품목별 `unit`·창고·위치를 담는다. **두 형태 중 어느 것을 외부 REST 계약으로 채택할지 팀 리뷰가 필요**하다. 내부 Java 서비스에서 단위/위치를 검증해야 한다는 원칙만 동일하다.
2. **PR 제출·승인과 버전:** `PRPatch`는 `expectedVersion`을 필수로 지정하지만, `submit`, `approve`, `reject`, `rework`는 현재 OpenAPI에서 같은 필드를 요구하지 않는다. DB 잠금/상태검증으로 경쟁을 막을 수도 있으므로, 낙관적 버전을 요청에 추가할지는 추후 결정한다.
3. **Document AI 동기성:** `POST /documents/analyze` 성공은 현재 `200 DocumentAnalyzeResult`이고 `GET /documents/analysis/{id}`도 있다. 지연 처리·Job 상태 모델과 모순이 없는지 확인하며, 메시지 큐 도입은 기본 요구사항이 아니다.
4. **Agent DRAFT 중복:** `AssistantChatRequest`는 `message`만 필수이고 `requestKey`는 없다. Agent가 PR 생성 Tool을 중복 호출/재시도할 경우 A의 PR 서비스가 실질적인 멱등키와 인증 주체를 확보하도록 설계해야 한다.
5. **HTTP 상태 코드:** 다수의 GET 목록 API에도 404·409·422가 공통 템플릿처럼 선언되어 있다. 구체적 발생 조건이 없으면 OpenAPI 검토에서 불필요한 응답을 정리한다. 409는 잔량·멱등·버전 충돌에 집중한다.
6. **PO 생성 입력:** `POCreate`의 품목 요청에는 `requisitionItemId`·`quantity`·`unitPrice`가 있으며 공급업체는 헤더 `supplierId` 하나다. 같은 PR item을 여러 PO에 분할하는 것은 허용하지만, PR 전체의 유일성 제약으로 PO를 제한하지 않는다.
7. **결정 문서 의존:** `X-01~X-14` 세부 스키마·인증·응답 코드는 A/B 검토 전이다. 이미 합의된 `D-01`(1:N PO)과 `D-02`(헤더 일괄 승인)는 다시 미확정으로 취급하지 않는다.

## 5. HTTP 오류 처리 기본 기준과 테스트 책임

| 상황 | 현재 API의 후보 HTTP 코드 | 서버에서 검증할 내용 | 테스트 담당 |
|---|---|---|---|
| 로그인 필요/만료 | `401` | 인증되지 않은 접근 시 상태 변경 금지 | B 기반 + 각 도메인 소유자 |
| 역할·소유권 위반 | `403` 또는 리소스 존재 보호를 위한 `404` | 다른 사용자 DRAFT·분석 결과 접근 금지 | A PR/Agent, B 문서/GR |
| 대상 ID 부재 | `404` | 존재하지 않는 Material/PR/PO/GR/Job | 각 도메인 소유자 |
| 상태·버전·중복 키·잔량 경쟁 | `409` | 승인이력 1회, PO/GR 중복 없음, 잔량 초과 전체 실패 | A: PR·PO / B: GR·재고 / 통합 A+B |
| 필수 필드·단위·수량 오류 | `422` | EA 등 정수 규칙, `qty > 0`, 필요일·반려사유 | 각 도메인 소유자 |
| 파일 크기·형식 | `413` / `415` | PDF 텍스트 여부, 파일 크기 및 허용 형식 | B |
| 외부 AI 서비스 장애 | `502` (초안) | 서비스 중단 시 허위 성공·중복 PR 생성 없음 | A Agent / B Document AI |

성공 코드는 현재 OpenAPI 그대로 표기했고, 특히 **재시도 때 기존 결과를 돌려주는 경우의 HTTP 코드(200/201 유지 여부)**와 오류 코드 일관성은 A+B가 확정해야 한다.

## 6. 기능 명세 40개와 REST 매핑 원칙

- **직접 REST 구현 도메인:** `MAT-01`, `SUP-01`, `WH-01`, `PR-01/02`, `APR-01`, `PO-01/02`, `GR-01/02`, `ST-01`, `SEC-01/02`, `AG-01~06`, `DOC-01~04`, `MATCH-01~03`, `AUD-01`. 하나의 기능이 여러 REST 동작과 관계될 수 있고 그 반대도 가능하다.
- **독립 REST Endpoint를 만들지 않는 기능:** `PO-03`(GR 내부 서비스), `AG-07`·`DOC-05`(오프라인 평가), `UX-01`·`UI-01~04`(프론트 업무 화면), `OPS-01~04`(공동 운영), `QA-01`(자동/통합 테스트). **독립 API가 없다는 이유로 P0에서 빠지는 것은 아니다.**
- **Document AI가 PR 초안 생성·수정:** `DOC-04`는 B가 검토 완료한 결과로 A 소유 `POST/PATCH /purchase-requisitions`를 소비한다. B가 별도 PR 저장 테이블/REST를 새로 만들지 않는다.
- **공통 개발 영역:** ERD·전체 API 계약 통합 리뷰, CI/CD·배포·로그·PostgreSQL 운영·E2E는 **A+B 공동**, 특정 API 소유자 수를 50:50으로 맞추기 위해 재배정하지 않는다.

## 7. 리뷰와 구현 인수인계 체크리스트

### A 담당 PR 제출 전

- [ ] 본인이 맡은 API의 `operationId`, 기능 ID, 요청·응답 스키마와 OpenAPI 일치
- [ ] 인증·도메인 소유권·상태 검증, 변경 기록 구현
- [ ] `PR 품목 100 → PO 60+40 가능`, `60+60 경쟁 발주 거부` PostgreSQL 통합 테스트
- [ ] B가 소비하는 PO 수량 서비스, Material 조회, 공통 PR DRAFT DTO 계약을 PR에 함께 명시
- [ ] B 리뷰에서 DTO·오류 코드 호환성 확인

### B 담당 PR 제출 전

- [ ] 공통 인증/세션·CSRF 설정과 A 도메인에서 사용 가능한 Principal 계약 확인
- [ ] GR 서비스가 A PO 서비스와 **하나의 트랜잭션**에서 동작하는지 통합 테스트
- [ ] GR 60/40, 중복 요청키, 동시 60+60, StockBalance/Movement 롤백 테스트
- [ ] Document AI 후보는 활성 Material ID로 확정·사용자 검토 후 A PR 서비스에 저장
- [ ] A 리뷰에서 ERP·AI 소비 계약 및 응답 상태 검토

### A+B 공동 리뷰 후

- [ ] `../project-management/OPEN-DECISIONS.md`의 X-01~X-14 관련 항목 합의 여부 기록; 1주 차 D-01/02 확정 유지
- [ ] OpenAPI 변경 시 operationId·기능 ID·담당자 매핑 갱신
- [ ] `../database/DB-SPEC.md`, `../security/RBAC-MATRIX.md`, `../design/SCREEN-SPEC.md`와 관련 변경 동시 리뷰
- [ ] E2E 세 경로(PR→PO→GR→Stock, Agent→PR, Document AI→PR) 통합 테스트 계획 작성
- [ ] 실제 검증 환경(테스트 컨테이너/PostgreSQL/CI), 실행 날짜·실패 내역을 PR에 기록

## 8. 관련 문서·유지 규칙

- [OpenAPI 계약 초안](openapi.yaml) — 경로/메서드/DTO의 단일 기준
- [권한 매트릭스](../security/RBAC-MATRIX.md) — 현재 제안된 RBAC 규칙
- [열린 설계 결정](../project-management/OPEN-DECISIONS.md) — X-01~X-15 승인 기록
- `../planning/FUNCTIONAL-SPEC.md` — P0 40개 기능 ID와 담당, 수용 조건
- `../planning/03-ROLE-AGREEMENT.md` — 팀 도메인·공통 인프라 역할 합의
- `09-PO-GR-CONTRACT.md` — B GR 오케스트레이터와 A PO 내부 서비스 계약

**업데이트 규칙:** OpenAPI의 경로·Method·operationId가 수정되면 이 매핑도 같은 PR에서 갱신한다. API 삭제/추가로 총 37개가 바뀌면 실제 OpenAPI 기준으로 수를 다시 센다. GitHub 이슈/PR에는 구현 담당과 리뷰 담당을 구분하고, 공통 계약 변경은 양측 승인이 필요한 변경으로 다룬다.
