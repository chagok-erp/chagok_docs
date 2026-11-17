# 차곡ERP — 2일 차 SAP 공식 구매·재고 문서 조사

> 버전: v0.2 (업로드 EDMX 6종 실제 구조 추가) | 조사일: **2026-11-16** | 조사 대상: 공개 **SAP Help Portal** 자료 + 사용자가 제공한 **EDMX 6개 원본**
> 용도: SAP API를 호출하기 위한 연동 명세가 아니라, **자체 ERP 업무 모델의 참조 자료**.
> 주의: 자료는 SAP S/4HANA Cloud Public Edition과 SAP S/4HANA On-Premise의 **서로 다른 문서/릴리스**를 포함한다. 단일 버전의 완전한 API 조합이라고 가정하지 않는다.

## 1. 요약: 무엇을 가져올 것인가

| 확인 대상 | 공식 문서에서 확인한 사실 | 차곡ERP P0 채택 범위 | 명시적 제외 |
|---|---|---|---|
| Purchase Requisition(PR) | 구매 요청 생성·수정·조회 API와 승인 워크플로 제공 | PR 헤더/품목, DRAFT·제출·헤더 단위 승인·반려·재제출 | 다단계/품목별 결재, 외부 SAP 연동 |
| Purchase Order(PO) | 공급업체·품목을 포함한 PO 생성·조회/변경 API 제공 | **내부 PO 생성·조회·브라우저 인쇄**와 PR 참조·중복 생성 검증 | 공급업체 주문 전송, 결제, 복잡한 납기/계약 |
| Goods Receipt(GR) / Material Document | PO 품목 참조로 입고 게시, 자재문서 생성, 취소 API 제공 | PO에 연결된 수동·부분 GR, 재고원장, 중복·초과 입고 방지 | GR 취소는 P1, 반품·품질/차단 재고 |
| Material Stock | 품목별·저장위치별 재고 조회 API 제공 | 품목·창고·저장위치 단위 현재고 조회 | SAP 재고 평가·회계 금액 |
| Product | 품목 기준정보·단위·다국어 설명 모델 제공 | 품목·기본단위·규격명 참조 | SAP 복합 플랜트·판매/회계 속성 |
| Business Partner | 공급업체 엔티티 및 주소·연락처 관계 제공 | 공급업체 코드·이름·전화·이메일 참조 | SAP BP 전사 역할·세무·은행 상세 |

## 2. SAP 공개 API 조사표

> 아래 URL은 **SAP 시스템에서 사용되는 예시 OData 경로**이며 차곡ERP에 그대로 사용할 REST 경로가 아니다. `https://api.sap.com/`은 API 탐색 시작점이다. 인증·통신 설정 없는 SAP URL을 호출해 데이터를 받을 수 있다는 의미도 아니다.

| 업무 | 서비스·기술 이름 | 공개 API 예시 | 참고할 모델·동작 |
|---|---|---|---|
| PR | **Purchase Requisition (OData V4)** `API_PURCHASEREQUISITION_2` | `POST /sap/opu/odata4/sap/api_purchaserequisition_2/srvd_a2x/sap/purchaserequisition/0001/PurchaseReqn` | PR 생성과 품목 모델, 응답 식별자 |
| PR 대안 | **Purchase Requisition (OData V2)** `API_PURCHASEREQ_PROCESS_SRV` | `POST /sap/opu/odata/sap/API_PURCHASEREQ_PROCESS_SRV/A_PurchaseRequisitionHeader` | 기존 V2와 V4 기술 이름 구분 |
| PO | **Purchase Order (OData V4)** `API_PURCHASEORDER_2` | `POST /sap/opu/odata4/sap/api_purchaseorder_2/srvd_a2x/sap/purchaseorder/0001/PurchaseOrder` | PO 헤더·품목·공급업체·가격 구조 |
| PO 구형 | **Purchase Order (OData V2)** `API_PURCHASEORDER_PROCESS_SRV` | (구형 서비스 명칭만 참고) | 일부 SAP 에디션의 문서에서 **Deprecated**로 표시, 신규 설계 참고는 V4 우선 |
| 재고 | **Material Stock (OData V2)** `API_MATERIAL_STOCK_SRV` | `GET /sap/opu/odata/sap/API_MATERIAL_STOCK_SRV/A_MaterialStock` | 재고 조회, 품목·저장위치별 수량 |
| 자재문서·입고 | **Material Document (OData V2)** `API_MATERIAL_DOCUMENT_SRV` | `POST /sap/opu/odata/sap/API_MATERIAL_DOCUMENT_SRV/A_MaterialDocumentHeader` | GR 품목·PO 참조, 이동 유형, 자재문서 생성 |
| 자재문서 조회 | 위와 동일 | `GET /sap/opu/odata/sap/API_MATERIAL_DOCUMENT_SRV/A_MaterialDocumentHeader` | 재고 이동 이력·문서 조회 |
| 자재문서 취소 | 위와 동일 | `POST /sap/opu/odata/sap/API_MATERIAL_DOCUMENT_SRV/Cancel?...` | 역처리 개념 참고, 차곡ERP에서는 P1 |
| 품목 | **Product (OData V2)** `API_PRODUCT_SRV` | `GET /sap/opu/odata/sap/API_PRODUCT_SRV/A_Product` | 품목·단위·설명·규격 참조 |
| 공급업체 | **Business Partner (OData V2)** `API_BUSINESS_PARTNER` | `GET /sap/opu/odata/sap/API_BUSINESS_PARTNER/A_Supplier` | 공급업체·BP 주소·전화·이메일 관계 참조 |

**기술명 혼동 방지:** `API_PURCHASEORDER_PROCESS_SRV`(PO V2)와 `API_PURCHASEORDER_2`(PO V4), `API_PURCHASEREQ_PROCESS_SRV`(PR V2)와 `API_PURCHASEREQUISITION_2`(PR V4)는 별도 서비스다. PO V2의 Deprecated 표시는 해당 SAP 공식 페이지의 **적용 에디션 조건에 따라** 해석해야 한다.

## 2.1 확보한 EDMX 명세 6종 — 실제 파일 분석 결과

**검증 방법:** 로컬에서 XML 파싱 후 EDMX 버전, EntityType의 Key/Property/NavigationProperty를 확인했다. 아래 이름은 업로드 파일에서 확인한 실제 메타데이터이며, 차곡ERP의 Java 엔티티 이름은 아니다. SAP 서비스 실행이나 요청·응답 테스트는 실시하지 않았다.

| 업로드 파일 | 형식 | 확인한 EntityType | 주요 Key 및 필드 |
|---|---|---|---|
| `CE_PURCHASEREQUISITION_0001.edmx` | OData V4 | `PurchaseReqnType`, `PurchaseReqnItemType` | `PurchaseRequisition`, `PurchaseRequisitionItem`, `Material`, `RequestedQuantity`, `BaseUnit`, `StorageLocation` |
| `CE_PURCHASEORDER_0001.edmx` | OData V4 | `PurchaseOrder_Type`, `PurchaseOrderItem_Type` | `PurchaseOrder`, `PurchaseOrderItem`, `Supplier`, `Material`, `OrderQuantity`, `PurchaseRequisition`, `PurchaseRequisitionItem` |
| `API_MATERIAL_DOCUMENT_SRV.edmx` | OData V2 | `A_MaterialDocumentHeaderType`, `A_MaterialDocumentItemType` | `MaterialDocumentYear`, `MaterialDocument`, `MaterialDocumentItem`, `PurchaseOrder`, `PurchaseOrderItem`, `GoodsMovementType`, `QuantityInBaseUnit` |
| `API_MATERIAL_STOCK_SRV.edmx` | OData V2 | `A_MaterialStockType`, `A_MatlStkInAcctModType` | `Material`, `Plant`, `StorageLocation`, `InventoryStockType`, `MatlWrhsStkQtyInMatlBaseUnit` |
| `API_PRODUCT_SRV.edmx` | OData V2 | `A_ProductType`, `A_ProductDescriptionType` | `Product`, `BaseUnit`, `PurchaseOrderQuantityUnit`, `ProductDescription` |
| `API_BUSINESS_PARTNER.edmx` | OData V2 | `A_SupplierType`, `A_BusinessPartnerType`, `A_BusinessPartnerAddressType`, `A_AddressPhoneNumberType`, `A_AddressEmailAddressType` | `Supplier`, `BusinessPartner`, `AddressID`, `PhoneNumber`, `EmailAddress` |

### 정확한 모델 해석과 자체 ERP 매핑

**PR 헤더/품목**

- `PurchaseReqnType` 키: `PurchaseRequisition`; 품목 연결: `_PurchaseRequisitionItem`.
- `PurchaseReqnItemType` 복합 키: `PurchaseRequisition` + `PurchaseRequisitionItem`; `RequestedQuantity`, `BaseUnit`, `Material`을 확인했다.
- SAP의 승인 상태/유연한 승인 워크플로 전체를 이 필드만으로 추론하지 않는다. `DRAFT/SUBMITTED/APPROVED/REJECTED`는 차곡ERP **자체 상태**다.

**PO 헤더/품목**

- `PurchaseOrder_Type`의 키는 `PurchaseOrder`, `Supplier` 등이 있다.
- `PurchaseOrderItem_Type`의 복합 키는 `PurchaseOrder` + `PurchaseOrderItem`이며 `PurchaseRequisition`·`PurchaseRequisitionItem` 참조 필드가 있어 **PR 품목 → PO 품목 연결**을 설계할 근거가 된다.
- `PurchaseOrderQuantityUnit`, `BaseUnit`, `OrderQuantity`, `NetPriceAmount`, `NetAmount`는 차곡ERP에 필요한 필드만 선별해 참고한다. SAP의 단위 환산·조건부 가격 계산 기능은 그대로 복제하지 않는다.

**GR·재고**

- `A_MaterialDocumentHeaderType`의 키는 `MaterialDocumentYear` + `MaterialDocument`; `A_MaterialDocumentItemType` 키는 여기에 `MaterialDocumentItem`이 추가된다.
- GR 품목에는 `PurchaseOrder`/`PurchaseOrderItem` 참조, `StorageLocation`, `QuantityInBaseUnit`, `MaterialBaseUnit`, `GoodsMovementType`가 있다.
- **재고 조회 수량은 `A_MaterialStockType` 자체가 아니라 관련 `A_MatlStkInAcctModType`의 `MatlWrhsStkQtyInMatlBaseUnit`에서 확인**한다. 후자에는 `Plant`, `StorageLocation`, `InventoryStockType` 등이 있다.
- 차곡ERP는 P0에서 복잡한 재고 유형·배치·특수재고를 제외하고 `Material + Warehouse + StorageLocation`으로 기본 StockBalance를 설계한다. SAP `Plant`는 차곡ERP `Warehouse`와 **동일 개념이 아니다**.

**품목·공급업체**

- `A_ProductType`: `Product`, `BaseUnit`, `PurchaseOrderQuantityUnit`, `SizeOrDimensionText`; `A_ProductDescriptionType`: `ProductDescription`(언어별).
- `A_SupplierType`: `Supplier`, `SupplierName`, `SupplierFullName`. **전화번호·이메일은 이 타입의 직접 필드가 아니다.** `A_BusinessPartnerType` ↔ `A_BusinessPartnerAddressType` ↔ `A_AddressPhoneNumberType`/`A_AddressEmailAddressType` 관계를 참고한다.
- 차곡ERP MVP에서는 SAP의 여러 엔티티를 그대로 분리하지 않고 `Supplier`에 기본 연락처를 직접 둘 수 있다. 단순화는 **자체 설계 선택**이며 SAP 구현과 일치한다고 표시하지 않는다.

### SAP 버전·자료 해석 주의

- PR/PO는 업로드한 **OData V4**, 재고·자재문서·품목·BP는 **OData V2**이다. 파일마다 선택한 SAP 제품·릴리스가 동일하다는 증거는 없으므로 단일 SAP 시스템 조합으로 간주하지 않는다.
- EDMX는 모델·타입·일부 동작을 명시하지만 SAP **실제 승인 정책, 허용오차 설정, 저장·취소 조건** 전부를 증명하지 않는다. 업무 정책은 공식 업무 문서와 별도로 대조한다.
- 신규 SAP API를 호출하지 않으므로 다운로드된 EDMX는 **설계 참고자료**이며 구현 대상 API 계약은 차곡ERP `../api/openapi.yaml`에서 별도로 정한다.

## 3. PR·승인 관련 확인 사항

- SAP는 PR을 조달 요청 문서로 다루며, API로 생성·조회·수정할 수 있다.
- SAP의 **Flexible Workflow**에서는 조건에 따라 한 단계 또는 여러 단계 승인을 설계하고, **헤더 전체 승인**과 **품목별 승인**을 구분한다.
- **차곡ERP 선택:** 승인자 1명, PR 헤더 전체 승인. `DRAFT → SUBMITTED → APPROVED` 또는 `REJECTED → DRAFT(재작업)`을 제안한다. 이는 SAP 상태 코드를 복사한 것이 아니다.
- **차곡ERP 제약:** 승인되지 않은 PR은 내부 PO로 전환하지 않는다. 승인 이력을 남기고, 중복 승인·중복 PO를 거부한다.

## 4. PO 관련 확인 사항

- PO V4는 PO 헤더와 품목, 공급업체, 가격 등 구조를 제공하며 생성·읽기·수정에 관한 공개 API 문서를 제공한다.
- SAP의 실제 PO는 공급업체를 대상으로 하는 구매 문서이지만 **차곡ERP PO는 실제 발송을 수행하지 않는 내부 기록**이다.
- **차곡ERP 선택:** 공급업체 1개, 승인 PR 참조, 품목·수량·단가·총액, 조회 및 브라우저 인쇄를 구현한다. 외부 발주/PG 결제는 제외한다.
- 본 프로젝트의 PO 수량은 입고로 줄이지 않고 원발주량을 보존한다. 누적 입고량을 별도 필드로 관리한다.

## 5. GR·재고 관련 확인 사항

### PO 참조와 부분 입고

- SAP의 구매 문서 기반 GR 화면에서는 **PO를 선택하고 열린 수량(open quantity)을 확인**, 품목별 입고 수량과 저장위치를 입력한 후 자재문서를 생성한다.
- SAP 문서에는 부분 납품 및 후속 입고가 명시되어 있다. PO 참조를 통해 발주·입고를 추적할 수 있다.
- **차곡ERP 선택:** PO 품목의 잔량 이내로 여러 번 입고할 수 있다. 100개 발주에 60개와 40개로 입고한다.

### 이동 유형과 문서

- SAP 재고 이동에서 **101 = 구매발주에 대한 입고**, **102 = 101 입고의 취소** 개념을 확인했다.
- Material Document API의 `GoodsMovementCode = 01`은 PO 입고, 품목의 `GoodsMovementType = 101`은 PO 입고 이동 유형 예시다. 이 둘은 **서로 다른 필드**다.
- **차곡ERP 선택:** 입고 생성 시 양수 StockMovement와 GR 문서를 생성. 이동 유형 명칭은 유사하게 참조할 수 있으나 SAP와 완전히 동일한 코드 체계는 강제하지 않는다. 102 역처리는 P1.

### 초과 납품과 SAP 대비 차이

- SAP는 열린 발주 수량을 기준으로 부족/초과 납품을 판정하며, 설정에 따라 초과납품 허용치(overdelivery tolerance)를 둘 수 있다.
- **차곡ERP의 독립적인 P0 정책은 허용오차 0**이다. `입고 요청량 > 발주 잔량`이면 무조건 거부한다. SAP가 모든 설정에서 초과 납품을 금지한다는 뜻은 아니다.

### 재고 조회 vs 재고 이동

- Material Stock API는 **현재 재고 조회** 참고 자료다. 이 서비스와 입고 자재문서를 만드는 Material Document API의 목적은 다르다.
- **차곡ERP 선택:** `StockBalance`(현재고)와 `StockMovement`(이력)를 모두 관리하며, GR 처리와 동일한 DB 트랜잭션 안에서 동기화한다. 이는 자체 아키텍처 정책이다.

## 6. SAP의 전체 범위 중 이번 MVP에 포함하지 않는 것

- 자동 MRP 구매요청 생성, RFQ·공급업체 선정 프로세스, 복잡한 계약/납기 일정.
- 품목별/다단계 승인, 자동 승인 조건 분기.
- 입고 허용오차, 미입고 조기 완료(delivery completed) 고급 규칙, 차단·품질 검사 재고.
- GR 취소·반품(현재 P1), 재무회계 평가, 송장 검증, 실제 지급 및 실거래.
- 실 SAP API 인증/호출, 데이터 동기화, ABAP.

## 7. 공식 출처 — 직접 확인한 문서

1. **PR V4 API 생성 예시** — SAP Help Portal, *Create a Purchase Requisition* (S/4HANA On-Premise)
   https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/91af7f8d3acd47da90d33aaacfcd0d59/72e35630d8244849a842fe2bd079d06a.html
2. **PR V2 서비스 명칭** — SAP Help Portal, *Purchase Requisition - OData V2* (S/4HANA On-Premise)
   https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/91af7f8d3acd47da90d33aaacfcd0d59/43c43f584eff2160e10000000a44147b.html
3. **PR 승인/반려 개념** — SAP Help Portal, *Flexible Workflow for Purchase Requisitions*
   https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/af9ef57f504840d2b81be8667206d485/781017ca64e446909d6cd6b0ee0ce021.html
4. **PO V4 서비스와 데이터 모델** — SAP Help Portal, *Purchase Order (OData V4)*
   https://help.sap.com/docs/SAP_S4HANA_CLOUD/0e602d466b99490187fcbb30d1dc897c/c89eec80ec2043d980cb7b8c89e0a00a.html
5. **PO V4 실제 operation** — SAP Help Portal, *Operations for Purchase Orders*
   https://help.sap.com/docs/SAP_S4HANA_CLOUD/bb9f1469daf04bd894ab2167f8132a1a/649d14483a984b788f3753ac88b37810.html
6. **PO V2 폐기 표시와 조건** — SAP Help Portal, *Purchase Order (OData V2) - (Deprecated)*
   https://help.sap.com/docs/PRODUCT_ID/91af7f8d3acd47da90d33aaacfcd0d59/acd2da57df6cc525e10000000a4450e5.html
7. **Material Stock API 조회** — SAP Help Portal, *Operations for Material Stock API*
   https://help.sap.com/docs/SAP_S4HANA_CLOUD/3f57e7df4a114edabffe8b2d581a59ed/dfc5b3e292874297843ed6cfb08eb83a.html
8. **Material Document API 조회·생성·취소** — SAP Help Portal, *Operations for Material Document API*
   https://help.sap.com/docs/SAP_S4HANA_CLOUD/3f57e7df4a114edabffe8b2d581a59ed/1aef4e402acd4c8b8ec2ea2bfda7715b.html
9. **Material Document PO 입고 요청 구조** — SAP Help Portal, *Create Material Documents*
   https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/eb2a39dd0c124fed8252f684002d55e1/8bb0d08295044ee3af444b4f2a6e4457.html
10. **PO 참조 GR 화면 절차** — SAP Help Portal, *How to Post a Goods Receipt for a Purchasing Document*
    https://help.sap.com/docs/SAP_S4HANA_CLOUD/32da8359c8ee4e8b8e8c5e15cacba5aa/31661de4ee984e4ea7a3a984e6bcca4b.html
11. **부분·초과 납품과 허용오차** — SAP Help Portal, *Underdeliveries, Overdeliveries, and Final Deliveries*
    https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/9905622a5c1f49ba84e9076fc83a9c2c/af63bd534f22b44ce10000000a174cb4.html?version=2025.002
12. **이동 유형 101/102 참고** — SAP Help Portal, *Supported Movement Types*
    https://help.sap.com/docs/IRPA_S4HANA/18862e3cddb74ce7ac751f49e568c1e0/a99a1bcb969f4939957469ea7d251e78.html

13. **Product API** — SAP Help Portal, *Create Product Master Data* (`API_PRODUCT_SRV`)
    https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/18fe3fab96864826bfa0be0de4f65b85/882b275c352e487cab45abecc4d59549.html
14. **Business Partner API** — SAP Help Portal, *Business Partner (A2X) – OData API* (`API_BUSINESS_PARTNER`)
    https://help.sap.com/docs/SAP_S4HANA_CLOUD/3c916ef10fc240c9afc594b346ffaf77/85043858ea0f9244e10000000a4450e5.html
15. **재고 조회 엔티티·수량** — SAP Help Portal, *Material Stock* (`A_MatlStkInAcctMod`)
    https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/eb2a39dd0c124fed8252f684002d55e1/67952bad694c4f6ea81e325c6720b61e.html

**원문 근거와 자체 제안의 구별:** 이 문서의 `공식 문서에서 확인한 사실`은 SAP 공개 문서 기반이고, `차곡ERP 선택` 항목은 두 사람이 앞으로 구현할 설계 제안이다. SAP 실행 환경·샌드박스 검증이나 실 SAP API 호출은 수행하지 않았다.
