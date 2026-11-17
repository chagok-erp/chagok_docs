# 차곡ERP 권한(RBAC) 및 인증·인가 규칙 매트릭스

> v0.3 / 2026-11-18 · **A 채택안, B 검토 전**. 기존 `SCREEN-SPEC.md`의 Mock 역할을 기반으로, 서버 권한을 구체화한 초안.  
> 각 요청마다 Spring Security의 인증 사용자 및 도메인 소유권·상태를 서버에서 다시 검증한다. 클라이언트 역할·사용자 ID·AI 생성 텍스트를 권한 근거로 삼지 않는다.

## 1. 역할과 범례

- `REQUESTER`: 구매요청 생성·수정·제출, 본인 AI 대화·문서 검토
- `APPROVER`: 제출된 PR 승인·반려(자기 승인 금지)
- `PURCHASER`: 기준정보 관리, 분할 발주, PR 작성
- `WAREHOUSE`: 창고·저장위치, 부분 입고, 현재고·수불원장
- `ADMIN`: 운영·예외 확인 권한. **다른 업무 담당자와 동일하게 자기 PR 승인 금지** 제안

`R` = 조회, `C` = 생성, `U` = 수정/상태전이, `–` = 접근 거부. `OWN`은 본인 소유에 한정, `ALL`은 권한 내 공용 조회. 한 사용자는 `user_roles`를 통해 **복수 역할을 가질 수 있음(A 채택안)**이며, 어떤 경우에도 소유권·SoD 제약은 별도로 적용한다.

## 2. 역할별 권한 초안

| 리소스/작업 | REQUESTER | APPROVER | PURCHASER | WAREHOUSE | ADMIN |
|---|---|---|---|---|---|
| 대시보드 | R(허용 범위) | R | R | R | R |
| 품목 | R | R | R/C/U | R | R/C/U |
| 공급업체 | R | R | R/C/U | – | R/C/U |
| 창고·저장위치 | – | – | R | R/C/U | R/C/U |
| PR 목록/상세 | R(OWN 및 제출 후 허용범위*) | R(심사 대상) | R(구매업무 범위) | – | R |
| PR DRAFT 생성 | C | – | C | – | C |
| PR DRAFT 수정/제출/반려 재작업 | U(OWN) | – | U(OWN) | – | U(OWN) |
| PR 승인/반려 | – | U(비자기) | – | – | U(비자기) |
| PO 목록/상세/인쇄 | – | R | R | R | R |
| PO 분할 발주 생성 | – | – | C | – | C |
| GR DRAFT 생성·본인 수정·확정·이력 | – | – | – | C/U(OWN)/R | C/U(OWN)/R |
| 현재고·수불원장 | R | – | R | R | R |
| Assistant 채팅·RAG·PR 초안 | OWN | – | OWN | – | OWN |
| 문서 업로드·검토·초안 저장 | OWN | – | OWN | – | OWN |
| 감사 로그 | 본인 업무 상세 | 승인 관련 | 구매 관련 | 입고 관련 | 전체(범위 검토) |
| 사용자·역할 관리 UI | – | – | – | – | P0 없음(인프라/시드) |

\* PR 조회 범위·관리자 대리편집은 상세 규칙 합의 필요. **최소 안전 기본값:** 본인 DRAFT만 조회/수정, 타인의 DRAFT는 서비스에서 거부. APPROVER가 SUBMITTED PR을 심사할 수 있어야 함. 구매 담당자는 승인/발주 가능한 범위를 조회 가능.

## 3. 업무 불변식과 보안 체크

1. **자기 승인:** `pr.requester_id == principal.userId`라면 APPROVER/ADMIN과 무관하게 승인·반려 금지(제안). `ADMIN` 긴급 예외를 추가하려면 별도 감사·승인 정책이 필요하므로 P0 미지원.
2. **타인 PR:** 자신의 DRAFT를 수정·제출할 수 있고, ADMIN의 타인 DRAFT 대리 수정은 P0에서 지원하지 않음. AI/Document 입력 경로는 동일 권한 적용.
3. **PO:** `PURCHASER`·`ADMIN`만 승인된 PR에서 발주 생성. 미발주 잔량 재검증, 공급업체 활성 검사, 요청키 멱등성 검증.
4. **GR:** `WAREHOUSE`·`ADMIN`만 DRAFT 생성·자신의 DRAFT 수정/확정 가능. POSTED는 조회만 가능. GR 하나는 PO·창고 각 하나를 참조하며 같은 POItem을 여러 저장위치에 분할 가능. 확정은 위치별 배분 SUM, 최신 PO 잔량, 버전·멱등/동시 요청 재검증. DRAFT 저장은 PO/재고 영향 없음.
5. **AI:** Agent에는 ERP 조회·RAG·PR DRAFT 생성/수정 Tool만 제공; PO/GR/승인/결제 Tool 제공 금지. 문서/모델 지시를 권한으로 승격하지 않음.
6. **세션:** Spring Security 서버 세션 + Secure·HttpOnly·적절한 SameSite 쿠키, CSRF 토큰 방식은 프론트와 함께 확정. 로그아웃과 세션 만료 시 401 상태 UX.
7. **외부 Rust PoC:** 브라우저가 Rust에 `userId`를 직접 전달해 권한을 결정하게 하지 않음. Spring ERP 인증 경유·내부 서비스 인증을 적용하고 사용자 권한은 Java 업무 서비스에서 재검증.
8. **로그:** `AUDIT_LOG`는 사용자·행위·시각·리소스 식별, 토큰·패스워드·민감문서 원문 제외.
9. **에러:** 미로그인 401, 권한 없음 403, 존재하지 않음 404(타인 리소스 존재 노출 방지 시 404 선택 가능), 상태/버전/잔량/멱등 충돌 409, 필드 검증 422 제안.

## 4. 화면 명세와의 차이 및 조치

| 항목 | 기존 AI Studio 초안 | 수정/보류 |
|---|---|---|
| `SCR-AUTH-001` 빠른 역할 전환 | 로그인 버튼으로 즉시 계정 전환 | **Mock에서만**. 실제 인증 서버에서는 별도 시드 계정 비밀번호로 로그인해야 함 |
| `SCR-APR-001` 자기 승인 | ADMIN 예외 | **ADMIN 포함 자기 승인 거부 제안** |
| `SCR-SUP-001` 이메일 | UI 필수 | 이메일 **선택** 제안, 입력된 경우 형식 검증 |
| `SCR-PR-002` 필요일 | DRAFT부터 필수·고정 기본값 | DRAFT 선택, SUBMIT 필수 제안(필요일은 사용자 선택) |
| `SCR-WH-001` | 창고 생성·수정 UI 누락 | MVP WH-01 확인 후 창고 관리 UI 추가(별도 화면 ID 불필요) |

## 5. 권한 테스트 초안

- `RBAC-01`: REQUESTER가 APPROVER/GR URL 직접 호출 → 403, DB 무변경
- `RBAC-02`: APPROVER가 본인 PR 승인 → 403 또는 409(외부 계약에 단일 매핑), 상태 불변
- `RBAC-03`: REQUESTER가 타인 DRAFT 수정/제출 → 403/404, 버전 불변
- `RBAC-04`: WAREHOUSE가 PO 생성 → 403, PO 미생성
- `RBAC-05`: AI가 타인 대화·문서 참조 → 403/404, 데이터 미노출
- `RBAC-06`: 쿠키 없는 인증 필요 API 호출 → 401
- `RBAC-07`: CSRF 보호 대상 세션 변경 요청에서 토큰 누락 → 403
- `RBAC-08`: ADMIN 본인 PR 승인 → 거부(제안)
- `RBAC-09`: WAREHOUSE/ADMIN이 타인 GR DRAFT를 수정·확정하려고 하면 403/404, 변동 없음
- `RBAC-10`: 본인 GR DRAFT를 POSTED 확정한 후 PATCH 재시도 → 409, 재고 변동 없음

## 6. 두 사람이 결정할 것

- [ ] 관리자 자기 승인 금지·복수 역할 정책 수용
- [ ] 타부서 PR 조회/승인 대상 범위와 관리자 대리편집 여부
- [ ] 공급업체 이메일 선택, PR DRAFT 필요일 선택·SUBMIT 필수
- [ ] 브라우저 배포 도메인과 Secure/SameSite/CSRF 적용 방식
- [ ] 각 API별 서버 권한 테스트 완료 기준


## v0.2 A 채택 정책 (B 리뷰 전)

- 복수 역할은 권한의 합집합으로 평가하되, `OWN`·작성자와 승인자 분리·문서 상태 제약은 역할보다 우선한다.
- ADMIN도 자기 PR 승인/반려 금지, 타인 DRAFT 대리 수정·제출 금지.
- 본인 DRAFT는 본인만 조회·수정. APPROVER는 심사 대상 SUBMITTED, PURCHASER는 승인·발주 대상 문서를 조회.
- Spring Security 세션 인증 + CSRF, HttpOnly/Secure 쿠키. 세션 만료는 401, CSRF 거부는 403. Rust PoC 도입 시 브라우저는 Spring Boot 경유.
- B의 교차 리뷰 전이므로 실제 보안 정책이 배포되었다는 의미가 아니다.

## v0.3 추가: GR DRAFT 권한 및 창고 비활성화

- B는 `WAREHOUSE/ADMIN` 모두 GR DRAFT를 만들 수 있게 하되 **DRAFT는 createdBy=principal.id 사용자 본인만 수정/확정**하도록 검사한다. 관리자 대리수정/대리확정은 별도 합의 없이는 허용하지 않는다.
- `GET /goods-receipts`의 DRAFT는 본인 것만 조회한다. POSTED는 권한 범위 내 조회하며 민감한 이력 접근은 서버에서 판단한다.
- `PATCH /warehouses/{id}` 및 `PATCH /storage-locations/{id}`의 활성화 변경은 WAREHOUSE/ADMIN에 한정한다. 거래 이력/재고 참조 리소스의 물리 삭제는 지원하지 않는다.
- 비활성 창고·위치는 과거 기록에서 계속 보여도 신규 GR POSTED 대상이 될 수 없다. DRAFT에는 옛 위치가 남을 수 있지만 확정 시 최신 활성 상태로 검증한다.
- B 최종 검토: 현재고>0 비활성화를 차단할지, GR DRAFT 사용자별 만료/보관 정책. 서버 권한 테스트로 결정과 일치하는지 확인한다.
