```abap
*&---------------------------------------------------------------------*
*& Include ZC103MMR0002TOP                          - Report ZC103MMR0002
*&---------------------------------------------------------------------*
REPORT zc103mmr0002 MESSAGE-ID zmsgc103.

**********************************************************************
* TABLES
**********************************************************************
TABLES : zc103mmt0007, zc103mmt0008, zc103mmt0011, zc103mmt0012.

**********************************************************************
* Class Instance
**********************************************************************
" Container
DATA : go_container   TYPE REF TO cl_gui_docking_container,   " 메인
       go_split_cont  TYPE REF TO cl_gui_splitter_container,  " 상하 스플릿
       go_top_cont    TYPE REF TO cl_gui_container,           " 위  (구매요청)
       go_bottom_cont TYPE REF TO cl_gui_container,           " 아래 (구매오더)
       go_topof_cont  TYPE REF TO cl_gui_docking_container,    " top of page 컨테이너
       " ALV
       go_top_grid    TYPE REF TO cl_gui_alv_grid,            " 구매요청 정보
       go_bottom_grid TYPE REF TO cl_gui_alv_grid,            " 구매오더 정보
       " For Top-of-page
       go_topofp_cont TYPE REF TO cl_gui_docking_container,
       go_dyndoc_id   TYPE REF TO cl_dd_document,
       go_html_cntrl  TYPE REF TO cl_gui_html_viewer.

" Popup container
DATA : go_pop_prid_cont TYPE REF TO cl_gui_custom_container,  " 구매요청아이템
       go_pop_prid_grid TYPE REF TO cl_gui_alv_grid,          " 구매요청아이템
       go_pop_vs_cont   TYPE REF TO cl_gui_custom_container,  " 벤더선택
       go_pop_vs_grid   TYPE REF TO cl_gui_alv_grid.          " 벤더선택

**********************************************************************
* Internal table and work area
**********************************************************************
*-- 구매요청 데이텨
DATA : BEGIN OF gs_prbody,
         icon            TYPE icon_d,                    " 아이콤
         prid            TYPE zc103mmt0007-prid,         " 구매요청번호
         plnid           TYPE zc103mmt0007-plnid,        " 플랜트번호
         pname           TYPE zc103mmt0003-name,         " 플랜트이름
         strid           TYPE zc103mmt0007-strid,        " 스토리지로케이션번호
         sname           TYPE zc103mmt0004-name,         " 스토리지로케이션이름
         mrpid           TYPE zc103mmt0007-mrpid,        " mrp 번호
         pr_status       TYPE zc103mmt0007-pr_status,    " 구매요청 상태
         estkz           TYPE zc103mmt0007-estkz,        " 생성지시자
         request_date    TYPE zc103mmt0007-request_date, " 구매요청일
         receive_date    TYPE zc103mmt0007-receive_date, " 납품요청일
         matid           TYPE zc103mmt0008-matid,        " 자재번호
         matname         TYPE zc103mmt0001-matname,      " 자재이름
         quantity        TYPE zc103mmt0008-quantity,     " 자재수량
         unit            TYPE zc103mmt0008-unit,         " 수량단위
         price           TYPE zc103mmt0008-price,        " 개당요청가격
         tot_price       TYPE zc103mmt0008-price,        " 총요청가격
         currency        TYPE zc103mmt0008-currency,     " 통화키
         celltab         TYPE lvc_t_styl,                " 편집 스타일
         creator_name    TYPE zc103fit0011-empname,      " 요청자 이름
         approve_name    TYPE zc103fit0011-empname,      " 결재자 이름
         h_erdat         TYPE zc103mmt0007-erdat,        "
         h_erzet         TYPE zc103mmt0007-erzet,
         h_ernam         TYPE zc103mmt0007-ernam.
         INCLUDE STRUCTURE zc103mms0001.              " 결재정보
DATA :   status_text(16),                             " 구매요청 상태 텍스트
         make_order(16),                            " 오더 전환 버튼
       END OF gs_prbody,
       gt_prbody LIKE TABLE OF gs_prbody,
       gt_prback LIKE TABLE OF gs_prbody.

*-- 구매요청 ALV
DATA : gt_tfcat TYPE lvc_t_fcat,
       gs_tlayo TYPE lvc_s_layo,
       gs_tvari TYPE disvariant,
       gt_sort  TYPE lvc_t_sort.

*-- 구매오더 데이터
DATA : BEGIN OF gs_pobody,
         " 헤더
         icon            TYPE icon_d,                    " 아이콤
         poid            TYPE zc103mmt0011-poid,         " 구매오더번호
         prid            TYPE zc103mmt0011-prid,         " 무매요청번호
         plnid           TYPE zc103mmt0011-plnid,        " 플랜트 번호
         pname           TYPE zc103mmt0003-name,         " 플랜트이름
         strid           TYPE zc103mmt0011-strid,        " 저장위치번호
         sname           TYPE zc103mmt0004-name,         " 스토리지로케이션이름
         mrpid           TYPE zc103mmt0011-mrpid,        " mrp 번호
         order_date      TYPE zc103mmt0011-order_date,   " 주문일자
         receive_date    TYPE zc103mmt0011-receive_date, " 자재수령일자
         total_price     TYPE zc103mmt0011-total_price,  " 총 주문금액
         currency        TYPE zc103mmt0011-currency,     " 화폐
         order_status    TYPE zc103mmt0011-order_status, " 오더 상태
         creator         TYPE zc103mmt0011-creator,      " 생성자
         approval        TYPE zc103mmt0011-approval,     " 결재자
         " 아이템
         matid           TYPE zc103mmt0012-matid,        " 자재 번호
         matname         TYPE zc103mmt0001-matname,      " 자재 이름
         bpid            TYPE zc103mmt0012-bpid,         " 벤더 번호
         bpname          TYPE zc103mmt0003-name,         " 벤더 이름
         quantity        TYPE zc103mmt0012-quantity,     " 자재 수량
         unit            TYPE zc103mmt0012-unit,         " 자재 단위
         price           TYPE zc103mmt0012-price,        " 자재별 금액 총 합
         creator_name    TYPE zc103fit0011-empname,      " 요청자 이름
         approve_name    TYPE zc103fit0011-empname,      " 결재자 이름
         celltab         TYPE lvc_t_styl,                " 편집 스타일 속성
         status_text(16),                                " 구매오더 상태 텍스트
         approve(16),                                    " 결재 버튼
       END OF gs_pobody,
       gt_pobody LIKE TABLE OF gs_pobody,
       gt_poback LIKE TABLE OF gs_pobody.

*-- 구매오더 ALV
DATA : gt_bfcat TYPE lvc_t_fcat,
       gs_blayo TYPE lvc_s_layo,
       gs_bvari TYPE disvariant,
       gt_bsort TYPE lvc_t_sort.

*-- 팝업 구매요청 아이템 ALV
DATA : gt_ppfcat TYPE lvc_t_fcat,
       gs_pplayo TYPE lvc_s_layo,
       gs_ppvari TYPE disvariant.

*-- ALV 툴바
DATA : gt_ui_functions TYPE ui_functions.

*-- 직원 마스터 테이블
DATA : gt_emp TYPE TABLE OF zc103fit0011,
       gs_emp TYPE zc103fit0011.

*-- 플랜트, 스토리지로케이션, 자재, 벤더  마스터 테이블
DATA : gt_plant  TYPE TABLE OF zc103mmt0003,
       gs_plant  TYPE zc103mmt0003,
       gt_stlo   TYPE TABLE OF zc103mmt0004,
       gs_stlo   TYPE zc103mmt0004,
       gt_mara   TYPE TABLE OF zc103mmt0001,
       gs_mara   TYPE zc103mmt0001,
       gt_vendor TYPE TABLE OF zc103mmt0002,
       gs_vendor TYPE zc103mmt0002.

*-- 사용자가 선택한 구매요청 워크에이리어
DATA : gs_sprbody LIKE gs_prbody.

*-- 사용자가 선택한 구매요청 아이템 ITAB
DATA : BEGIN OF gs_sprit.
         INCLUDE STRUCTURE zc103mmt0008.
DATA :   matname       TYPE zc103mmt0001-matname,
         bpid          TYPE zc103mmt0002-bpid,
         bpname        TYPE zc103mmt0002-name,
         vs_complete   TYPE icon_d,
         vs_status(16),
       END OF gs_sprit,
       gt_sprit LIKE TABLE OF gs_sprit.

*-- 구매정보레코드 / 벤더평가
DATA : BEGIN OF gs_pir.
         INCLUDE STRUCTURE zc103mmt0005.            " 구매정보레코드 테이블
DATA :   matname    TYPE zc103mmt0001-matname,         " 자재이름
         matgrp     TYPE zc103mmt0001-matgrp,          " 자재타입
         mgrpnm(16),                                " 자재타입이름
         bpname     TYPE zc103mmt0002-name,            " 벤더타입
         qscore     TYPE zc103mmt0019-quality_score,   " 품질점수
         pscore     TYPE zc103mmt0019-price_score,     " 가격점수
         rscore     TYPE zc103mmt0019-receive_score,   " 납기점수
         celltab    TYPE lvc_t_styl,                   " 편집
         select(16),
       END OF gs_pir,
       gt_pir      LIKE TABLE OF gs_pir,
       gt_pir_back LIKE TABLE OF gs_pir.

*-- 벤더 선택 ALV
DATA : gt_pvfcat TYPE lvc_t_fcat,
       gs_pvlayo TYPE lvc_s_layo,
       gs_pvvari TYPE disvariant.

*-- 사원번호 search help
DATA : BEGIN OF gs_emp_value,
         empno   TYPE zc103fit0011-empno,
         empname TYPE zc103fit0011-empname,
         dptcode TYPE zc103fit0011-dptcode,
       END OF gs_emp_value,
       gt_emp_value LIKE TABLE OF gs_emp_value.

**********************************************************************
* Common variable
**********************************************************************
DATA : gv_okcode TYPE sy-ucomm.

" 팝업 표시용
DATA : gv_pr_status(16),
       gv_estkz(10).

" 구매오더/구매요청 건 수
DATA : gv_prcnt TYPE i,  " 미전환 구매요청 수
       gv_pocnt TYPE i,  " 미승인 구매오더 수
       gv_poacnt TYPE i. " 승인된 구매오더 수

" 툴바 모드
DATA : gv_toolbar_mode VALUE 'T'. " T : 전체, N : 승인대기, A : 승인

" 직원 부서
DATA : gv_dptcode      TYPE zc103fit0011-dptcode,
       gv_creator_name TYPE zc103fit0011-empname,
       gv_creator      TYPE zc103fit0011-empno.

----------------------------------------------------------------------------------
Extracted by Direct Download Enterprise version 1.3.1 - E.G.Mellodew. 1998-2005 UK. Sap Release 758
```
