# 2주 차 상세 결정사항 · A/B 교차 리뷰표

> v0.1. 사용자(A)는 D-01~11의 **방향**을 확정/수용했으나, 아래 물리 필드·API·화면·보안 세부사항은 A/B 공동 최종 확인 전이다.

| ID | 결정 내용 | 이번 초안 권장 | A | B | 확정 문서 |
|---|---|---|---|---|---|
| X-01 | SQL 수량 정밀도 | `NUMERIC(18,3)` 및 JSON Decimal 문자열 | ☐ | ☐ | DB-SPEC, openapi |
| X-02 | 기준단위 소수점 규칙 | EA·SET 정수, KG/M/L 최대 3자리 | ☐ | ☐ | DB-SPEC, SCREEN-SPEC |
| X-03 | 금액·반올림 | KRW 정수·line HALF_UP | ☐ | ☐ | DB-SPEC, openapi |
| X-04 | DRAFT 필요일 | DRAFT 선택, SUBMIT 필수 | ☐ | ☐ | SCREEN-SPEC, openapi |
| X-05 | 공급업체 연락처 | 전화 필수, 이메일 선택·입력 시 형식검사 | ☐ | ☐ | SCREEN-SPEC, openapi |
| X-06 | 자기 승인 | ADMIN 포함 작성자 본인 승인/반려 거부 | ☐ | ☐ | RBAC-MATRIX |
| X-07 | 복수 역할/PR 조회 | 본인 DRAFT 비공개, 승인자의 SUBMITTED 심사, 관리자 대리편집 정책 별도 | ☐ | ☐ | RBAC-MATRIX |
| X-08 | `PRItem` 중복 라인 | 같은 PR 내 동일 품목 반복 허용/차단 여부 정하기 | ☐ | ☐ | DB-SPEC, API |
| X-09 | PO 라인 유일 | 한 PO에서 동일 PR item은 1라인, 여러 PO 간 재참조 허용 | ☐ | ☐ | DB-SPEC |
| X-10 | GR 위치 정책 | 한 GR에 하나의 저장위치(초기), 필요 시 품목별 위치 확장 | ☐ | ☐ | DB-SPEC, openapi |
| X-11 | API 경로/DTO | `/api/v1`, `requestKey`, `expectedVersion` 및 통일된 에러 객체 | ☐ | ☐ | openapi |
| X-12 | 프론트 수정 | 창고 CRUD, 메일/필요일, 업로드 확장자, 신뢰도% 제거 | ☐ | ☐ | SCREEN-SPEC |
| X-13 | 인증 세부 | CSRF, 세션 만료, Rust AI 서버 사용자 신뢰 경계 | ☐ | ☐ | RBAC-MATRIX, API |
| X-14 | 동시성 에러 UX | PO/GR 409, 원본 조회·재시도 및 요청키 유지 | ☐ | ☐ | API, SCREEN-SPEC |
| X-15 | 팀 소유권/브랜치 | GitHub Free Private vs Public vs Team 및 공동 Owner 정책 | ☐ | ☐ | GITHUB-SETUP, TEAM-OPERATING-AGREEMENT |

팀 회의 날짜: __________ · 합의한 항목: __________ · 보류/담당/기한: __________

## 교차 리뷰 분담

- A 초안 책임: Material/Supplier/PR/PO, Agent/RAG, 공통 디자인. B가 PO↔GR 및 PR 초안 계약 리뷰.
- B 초안 책임: User/Role, Warehouse/GR/Stock, Document AI/ES. A가 GR·Stock 및 문서/PR 연계 리뷰.
- 공동 책임: CI/CD·배포·로깅·PostgreSQL 운영, ERD/API 합본, 전 구간 테스트.

## 작업 착수 전 완료 판정

- [ ] PR 품목 100 → PO 60/40, 경쟁 60+60: 검증 로직과 수량 잠금 위치 합의
- [ ] PO 입고 100 → GR 60/40: 단일 트랜잭션·멱등키·원장/현재고 계약 합의
- [ ] UI/Agent/Document의 PR DRAFT 공통 서비스 입력·결과/버전 계약 합의
- [ ] 401/403/404/409/422 처리와 로그인/CSRF 흐름 합의
- [ ] 로컬 DB/마이그레이션 파일 관리 방식 합의
- [ ] 실제 React 소스 기준 `SCREEN-SPEC` 동작 비교(현재 미검증)
