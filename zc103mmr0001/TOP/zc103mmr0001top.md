```abap
*&---------------------------------------------------------------------*
*& Include ZC103MMR0001TOP                          - Report ZC103MMR0001
*&---------------------------------------------------------------------*
REPORT zc103mmr0001 MESSAGE-ID zmsgc103.

CLASS lcl_event_handler DEFINITION DEFERRED.

**********************************************************************
* Tables
**********************************************************************
TABLES : zc103mmt0007, zc103mmt0008.

**********************************************************************
* Class instance
**********************************************************************
DATA : go_main_cont TYPE REF TO cl_gui_custom_container, " 메인 컨테이너
       go_alv_grid  TYPE REF TO cl_gui_alv_grid,         " 메인 alv
       go_pop_cont  TYPE REF TO cl_gui_custom_container, " 팝업 컨테이너
       go_pop_grid  TYPE REF TO cl_gui_alv_grid.

*-- For TOP-OF-PAGE
DATA : go_topofp_cont TYPE REF TO cl_gui_docking_container, " TOP-OF-PAGE 컨테이너
       go_dyndoc_id   TYPE REF TO cl_dd_document,           " 텍스트 문서 구성
       go_html_cntrl  TYPE REF TO cl_gui_html_viewer.       " html 표시

**********************************************************************
* Internal table and Work area
**********************************************************************
*-- 구매요청 데이텨
DATA : BEGIN OF gs_body,
         icon            TYPE icon_d,                    " 아이콤
         prid            TYPE zc103mmt0007-prid,         " 구매요청번호
         scheduleid      TYPE zc103mmt0007-scheduleid,   " 항공스케줄아이디
         plnid           TYPE zc103mmt0007-plnid,        " 플랜트번호
         pname           TYPE zc103mmt0003-name,         " 플랜트이름
         strid           TYPE zc103mmt0007-strid,        " 스토리지로케이션번호
         sname           TYPE zc103mmt0004-name,         " 스토리지로케이션이름
         mrpid           TYPE zc103mmt0007-mrpid,        " mrp 번호
         pr_status       TYPE zc103mmt0007-pr_status,    " 구매요청 상태
         estkz           TYPE zc103mmt0007-estkz,        " 생성지시자
         estkz_text(32),                                 " 생성지시자 텍스트값
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
         pritemid        TYPE zc103mmt0008-pritemid.
         INCLUDE STRUCTURE zc103mms0001.              " 결재정보
         INCLUDE STRUCTURE ZC103PPS01.                " 타입스탬프
DATA :   status_text(16),                             " 구매요청 상태 텍스트
         approve(16),                                 " 결재 버튼
         update(16),                                  " 수정 버튼
       END OF gs_body,
       gt_body LIKE TABLE OF gs_body,
       gt_back LIKE TABLE OF gs_body.

*-- 구매요청헤더 테이블
DATA : gt_prhdt TYPE TABLE OF zc103mmt0007.

*-- 자재, 플랜트, 스토리지 마스터
DATA : BEGIN OF gs_text.
         INCLUDE STRUCTURE zc103mmt0001.
DATA :   pname TYPE zc103mmt0003-name,
         sname TYPE zc103mmt0004-name,
       END OF gs_text,
       gt_text LIKE TABLE OF gs_text,
       gs_stlo TYPE zc103mmt0004,
       gt_stlo TYPE TABLE OF zc103mmt0004.

*-- 직원정보
DATA : gt_emp TYPE TABLE OF zc103fit0011,
       gs_emp TYPE zc103fit0011.

*-- 구매요청 조회 필드
DATA : gv_mrpid LIKE gs_body-mrpid,
       gv_prid  LIKE gs_body-prid,
       gv_matid LIKE gs_body-matid.

*-- 구매요청 ALV 데이터
DATA : gs_fcat TYPE lvc_s_fcat,
       gt_fcat TYPE lvc_t_fcat,
       gs_layo TYPE lvc_s_layo,
       gt_sort TYPE lvc_t_sort,
       gs_vari TYPE disvariant.

*-- 구매요청 생성 ALV 헤더, 아이템
DATA : BEGIN OF gs_prcr.
         INCLUDE STRUCTURE gs_body.
DATA :   cell_tab   TYPE lvc_t_styl,
         modi_yn(1).
DATA : END OF gs_prcr,
       gt_prcr LIKE TABLE OF gs_prcr,
       gs_prhd TYPE zc103mmt0007,
       gt_prhd TYPE zc103mmt0007.

*-- 구매요청 생성 ALV
DATA : gt_pfcat TYPE lvc_t_fcat,
       gs_playo TYPE lvc_s_layo,
       gs_pvari TYPE disvariant,
       gt_psort TYPE lvc_t_sort.

*-- ALV 툴바
DATA : gt_ui_functions TYPE ui_functions.

*-- ALV 툴바 버튼
DATA : gs_button TYPE stb_button.

*-- 자재번호 서치헬프
DATA : BEGIN OF gs_sh_matid,
         matid   TYPE zc103mmt0001-matid,
         matname TYPE zc103mmt0001-matname,
       END OF gs_sh_matid,
       gt_sh_matid LIKE TABLE OF gs_sh_matid.

*-- ALV에서 구매요청 삭제
DATA : gt_prdelt LIke table of gs_prcr.

*-- 사원번호 search help
DATA : BEGIN OF gs_emp_value,
         empno   TYPE zc103fit0011-empno,
         empname TYPE zc103fit0011-empname,
         dptcode TYPE zc103fit0011-dptcode,
       END OF gs_emp_value,
       gt_emp_value LIKE TABLE OF gs_emp_value.

**********************************************************************
* Common Variable
**********************************************************************
DATA : gv_okcode  TYPE sy-ucomm,
       gv_mode(1) VALUE 'E'.

*-- 사원정보
DATA : gv_empno   TYPE zc103fit0011-empno,
       gv_empname TYPE zc103fit0011-empname,
       gv_dptcode TYPE zc103fit0011-dptcode.

*-- 구매요청개수
DATA : gv_totpr TYPE i, " 전체
       gv_aprdp type i, " 승인 대기
       gv_aprvp type i. " 승인 완료

*-- 팝업창 모드
DATA : gv_pmode VALUE 'C'. " C : 생성, U : 수정

----------------------------------------------------------------------------------
Extracted by Direct Download Enterprise version 1.3.1 - E.G.Mellodew. 1998-2005 UK. Sap Release 758
```
