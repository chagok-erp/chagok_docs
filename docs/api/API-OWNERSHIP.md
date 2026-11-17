# 차곡ERP REST API 구현 책임 및 교차 리뷰 (API-OWNERSHIP)

> v0.2 · 2026-11-18 · B 피드백 수용에 따른 설계 갱신본. Spring Boot 구현·팀 최종 승인과는 별개.
> `openapi.yaml`의 실제 `paths`·`operationId`를 기준으로 생성. 별도 API 엔드포인트로 PO↔GR 내부 서비스를 분리하지 않는다.

## 1. 범위와 책임

- **A 구현/B 리뷰:** 품목, 공급업체, PR, 승인, PO, 대화형 Agent/RAG.
- **B 구현/A 리뷰:** 사용자 인증, 창고/저장위치, GR DRAFT/POSTED, 현재고/원장, Document AI.
- **공동:** 감사 조회, 인프라/CI/CD 및 PO↔GR 내부 서비스 통합 검토.
- A의 `PurchaseOrderReceivingService`는 B의 `GoodsReceiptApplicationService` 내부에서 같은 PostgreSQL 트랜잭션으로 호출한다. 별도 REST API가 아니다.
- 전제: API 경로 공통 prefix `/api/v1` (경로 표에는 생략).

## 2. REST API 전체 소유권 매핑

| # | HTTP | 경로 | operationId | 담당 / 리뷰 | 관련 기능 |
|---:|---|---|---|---|---|
| 01 | `POST` | `/auth/login` | `post_auth_login` | **B** / A | SEC-01/02 |
| 02 | `POST` | `/auth/logout` | `post_auth_logout` | **B** / A | SEC-01/02 |
| 03 | `GET` | `/auth/me` | `get_auth_me` | **B** / A | SEC-01/02 |
| 04 | `GET` | `/materials` | `get_materials` | **A** / B | MAT-01 |
| 05 | `POST` | `/materials` | `post_materials` | **A** / B | MAT-01 |
| 06 | `PATCH` | `/materials/{id}` | `patch_materials_by_id` | **A** / B | MAT-01 |
| 07 | `PATCH` | `/materials/{id}/active` | `patch_materials_by_id_active` | **A** / B | MAT-01 |
| 08 | `GET` | `/suppliers` | `get_suppliers` | **A** / B | SUP-01 |
| 09 | `POST` | `/suppliers` | `post_suppliers` | **A** / B | SUP-01 |
| 10 | `PATCH` | `/suppliers/{id}` | `patch_suppliers_by_id` | **A** / B | SUP-01 |
| 11 | `GET` | `/warehouses` | `get_warehouses` | **B** / A | WH-01 |
| 12 | `POST` | `/warehouses` | `post_warehouses` | **B** / A | WH-01 |
| 13 | `PATCH` | `/warehouses/{id}` | `patch_warehouses_by_id` | **B** / A | WH-01 |
| 14 | `GET` | `/storage-locations` | `get_storage_locations` | **B** / A | WH-01 |
| 15 | `POST` | `/storage-locations` | `post_storage_locations` | **B** / A | WH-01 |
| 16 | `PATCH` | `/storage-locations/{id}` | `patch_storage_locations_by_id` | **B** / A | WH-01 |
| 17 | `GET` | `/purchase-requisitions` | `get_purchase_requisitions` | **A** / B | PR-01/02 또는 APR-01 |
| 18 | `POST` | `/purchase-requisitions` | `post_purchase_requisitions` | **A** / B | PR-01/02 또는 APR-01 |
| 19 | `GET` | `/purchase-requisitions/{id}` | `get_purchase_requisitions_by_id` | **A** / B | PR-01/02 또는 APR-01 |
| 20 | `PATCH` | `/purchase-requisitions/{id}` | `patch_purchase_requisitions_by_id` | **A** / B | PR-01/02 또는 APR-01 |
| 21 | `POST` | `/purchase-requisitions/{id}/submit` | `post_purchase_requisitions_by_id_submit` | **A** / B | PR-01/02 또는 APR-01 |
| 22 | `POST` | `/purchase-requisitions/{id}/rework` | `post_purchase_requisitions_by_id_rework` | **A** / B | PR-01/02 또는 APR-01 |
| 23 | `POST` | `/purchase-requisitions/{id}/approve` | `post_purchase_requisitions_by_id_approve` | **A** / B | PR-01/02 또는 APR-01 |
| 24 | `POST` | `/purchase-requisitions/{id}/reject` | `post_purchase_requisitions_by_id_reject` | **A** / B | PR-01/02 또는 APR-01 |
| 25 | `GET` | `/purchase-orders` | `get_purchase_orders` | **A** / B | PO-01/02 |
| 26 | `POST` | `/purchase-orders` | `post_purchase_orders` | **A** / B | PO-01/02 |
| 27 | `GET` | `/purchase-orders/{id}` | `get_purchase_orders_by_id` | **A** / B | PO-01/02 |
| 28 | `GET` | `/goods-receipts` | `get_goods_receipts` | **B** / A | GR-01/02 + PO-03 내부계약 |
| 29 | `POST` | `/goods-receipts` | `post_goods_receipts` | **B** / A | GR-01/02 + PO-03 내부계약 |
| 30 | `GET` | `/goods-receipts/{id}` | `get_goods_receipts_by_id` | **B** / A | GR-01/02 + PO-03 내부계약 |
| 31 | `PATCH` | `/goods-receipts/{id}` | `patch_goods_receipts_by_id` | **B** / A | GR-01/02 + PO-03 내부계약 |
| 32 | `GET` | `/stock-balances` | `get_stock_balances` | **B** / A | ST-01 |
| 33 | `GET` | `/stock-movements` | `get_stock_movements` | **B** / A | ST-01 |
| 34 | `GET` | `/audit-logs` | `get_audit_logs` | **A+B** / A+B | AUD-01 |
| 35 | `POST` | `/assistant/chat` | `post_assistant_chat` | **A** / B | AG-01~07 |
| 36 | `GET` | `/assistant/sessions/{id}` | `get_assistant_sessions_by_id` | **A** / B | AG-01~07 |
| 37 | `GET` | `/documents/analysis/{id}` | `get_documents_analysis_by_id` | **B** / A | DOC-01~05/MATCH-01~03 |
| 38 | `POST` | `/documents/analyze` | `post_documents_analyze` | **B** / A | DOC-01~05/MATCH-01~03 |
| 39 | `POST` | `/goods-receipts/{id}/post` | `post_goods_receipts_by_id_post` | **B** / A | GR-01/02 + PO-03 내부계약 |

## 3. 변경된 핵심 GR 계약

| 경로 | 역할 | DB/업무 효과 |
|---|---|---|
| `POST /goods-receipts` | GR DRAFT 생성 | PO 수량·StockMovement·StockBalance 변경 **없음** |
| `GET /goods-receipts?status=DRAFT` | 초안 목록 | 본인 초안만 조회 |
| `GET /goods-receipts/{id}` | 초안·확정 상세 | DRAFT 본인 소유권 검증 |
| `PATCH /goods-receipts/{id}` | 초안 수정 | `expectedVersion` 확인, items 전체 교체 허용, 재고 미변경 |
| `POST /goods-receipts/{id}/post` | POSTED 확정 | B가 트랜잭션 관리, A가 POItem별 **합산 수량** 검증·누적, B가 **위치별** 원장·현재고 반영 |

**Breaking change:** 종전 `POST /goods-receipts`는 즉시 POSTED 생성이었지만 이제 DRAFT 생성이다. React Mock/SDK/테스트 및 B Controller를 함께 변경해야 한다.

**GR 배분:** 한 GR은 한 PO·한 창고. 같은 POItem이 다른 저장위치에 여러 번 등장할 수 있음. 한 POItem·동일 위치 반복은 **거부 권고(B 검토 전)**. 다른 창고/비활성 위치는 확정 불가.

**멱등:** 초안 생성/수정/확정의 requestKey는 연산별로 구분. 확정 동일 키/동일 내용 재전송 시 기존 성공 결과, 다른 내용은 409. 편집은 `expectedVersion` 충돌 409.

## 4. 계약 교차 리뷰 및 B 미결정

- **A가 B의 구현을 리뷰:** GR DRAFT 저장 최소 필드·소유권, 위치별 배분, POItem별 합계, 현재고 잠금·멱등, 문서/PR 초안 연계.
- **B가 A의 구현을 리뷰:** PR 동일 품목 중복 행과 `itemId/displayOrder` 보존, `pr_item_id`별 1:N 분할 발주 잔량, PO 내부 서비스 DTO 및 예외.
- **B 최종 검토:** 현재고>0 창고/위치 비활성화(차단 권고), 같은 POItem+위치 중복 배분 행(거부 권고), DRAFT 접근 및 만료/보관 방식.
- **A/B 공동:** PRItem별 필요일·용도 신규 필드 여부(미확정), GR 확정 API의 요청키·버전·잠금 순서, POSTED 트랜잭션 복구 및 UI 오류 응답.
- **원칙:** 실제 API 구현과 PR 승인·PO 생성·GR 확정은 모두 Spring Boot 업무 서비스에서 권한 검증한다. Agent/Document는 PR DRAFT 외 쓰기 Tool이 없다.

## 5. 정적 집계 (OpenAPI 참조)

- `paths`: **30**
- HTTP 동작: **39**
- A 구현: **20**, B 구현: **18**, 공동: **1**
- 실제 컴파일·통합 테스트/배포는 아직 수행하지 않았다.
