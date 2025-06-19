```abap
*&---------------------------------------------------------------------*
*& Include ZC103MMR0003TOP                          - Report ZC103MMR0003
*&---------------------------------------------------------------------*
REPORT zc103mmr0003 MESSAGE-ID zmsgc103.

**********************************************************************
* TAB Control
**********************************************************************
CONTROLS gc_tab TYPE TABSTRIP.     " 탭스트립

DATA : gv_subscreen TYPE sy-dynnr. " 서브스크린 번호

**********************************************************************
* Class instance
**********************************************************************
*-- 입고일자 탭
DATA : go_receipt_tree        TYPE REF TO cl_gui_alv_tree,         " 입고일자 기준 ALV 트리
       go_receipt_container   TYPE REF TO cl_gui_custom_container, " 입고일자 표시 컨테이너
       go_receipt_change_menu TYPE REF TO cl_ctmenu.               " 입고일자 트리 메뉴
*-- 저장위치 탭
DATA : go_stlo_tree        TYPE REF TO cl_gui_alv_tree,          " 저장위치 기준 ALV 트리
       go_stlo_cont        TYPE REF TO cl_gui_custom_container,  " 저장위치 표시 컨테이너
       go_stlo_change_menu TYPE REF TO cl_ctmenu.                " 저장위치 트리 메뉴
*-- 구매오더 alv용
DATA : go_main_cont TYPE REF TO cl_gui_custom_container, " 구매오더 컨테이너
       go_po_grid   TYPE REF TO cl_gui_alv_grid.         " 구매오더 alv
*-- 구매오더 아이템 alv
DATA : go_poid_cont TYPE REF TO cl_gui_custom_container, " 구매오더 아이템 컨테이너 (팝업창)
       go_poid_grid TYPE REF TO cl_gui_alv_grid.         " 구매오더 아이템 alv   (팝업창)
*-- 입고 alv
DATA : go_rece_cont TYPE REF TO cl_gui_custom_container, " 입고 컨테이너 (팝업창)
       go_rece_grid TYPE REF TO cl_gui_alv_grid.         " 입고 alv   (팝업창)
*-- 스플리터 컨테이너
DATA : go_split_cont  TYPE REF TO cl_gui_splitter_container,  " 우측 splitter container
       go_top_cont    TYPE REF TO cl_gui_container,           " 상단 컨테이너 ( 구매오더 )
       go_bottom_cont TYPE REF TO cl_gui_container.           " 하단 컨테이너 ( 송장헤더 )
*-- 송장 alv
DATA : go_gr_grid TYPE REF TO cl_gui_alv_grid.
*-- 송장 아이템 alv
DATA : go_gr_item_grid TYPE REF TO cl_gui_alv_grid,         " 송장 아이템 alv (팝업)
       go_gr_item_cont TYPE REF TO cl_gui_custom_container. " 송장 아이템 컨테이너 (팝업)
*-- 송장 검증 alv
DATA : go_checkgr_grid TYPE REF TO cl_gui_alv_grid,         " 송장 검증 alv (팝업)
       go_checkgr_cont TYPE REF TO cl_gui_custom_container. " 송장 검증 컨테이너 (팝업)


**********************************************************************
* Internal table and Work area
**********************************************************************
*-- 구매오더 헤더
DATA : BEGIN OF gs_pohd.
         INCLUDE STRUCTURE zc103mmt0011.
DATA :   pname                 TYPE zc103mmt0003-name,      " 플랜트명
         sname                 TYPE zc103mmt0004-name,      " 저장위치명
         creator_name          TYPE zc103fit0011-empname,   " 오더 생성자 이름
         approval_name         TYPE ZC103fit0011-empname,   " 오더 결재자 이름
         approval_dept         TYPE zc103fit0011-dptcode,   " 오더 결재자 부서코드
         order_status_text(32),                             " 오더 상테 도메인 텍스트
         icon                  TYPE icon_d,                 " 상태 아이콘
         celltab               TYPE lvc_t_styl,             " 편집용
         receive_status(32),
         receive_btn(16),
       END OF gs_pohd,
       gt_pohd     LIKE TABLE OF gs_pohd,                   " 구매오더 itab (필터링 후)
       gt_pohdback LIKE TABLE OF gs_pohd,                   " 구매오더 itab (필터링 전)
       gs_sel_pohd LIKE gs_pohd. " 송장 alv 출력을 위한 구매오더 선택

*-- 구매오더 아이템
DATA : BEGIN OF gs_poid.
         INCLUDE STRUCTURE ZC103MMt0012.                    " 구매오더 아이템 테이블
DATA :   matname TYPE zc103mmt0001-matname,                 " 자재명
         bpname  TYPE zc103mmt0002-name,                    " 공급업체명
       END OF gs_poid,
       gt_poid     LIKE TABLE OF gs_poid,                   " 구매오더 아이템 itab (필터링 전)
       gt_poidback LIKE TABLE OF gs_poid.                   " 구매오더 아이템 itab (필터링 후)

*-- 입고문서
DATA : BEGIN OF gs_rece.
         INCLUDE STRUCTURE ZC103MMt0015.           " 자재 입고 테이블
DATA :   bpname  TYPE ZC103MMt0002-name,           " 공급업쳄명
         matname TYPE zc103mmt0001-matname,        " 자재명
         celltab TYPE lvc_t_styl,
       END OF gs_rece,
       gt_rece     LIKE TABLE OF gs_rece,          " 입고 itab
       gt_receback LIKE TABLE OF gs_rece.
*-- 송장헤더
DATA : BEGIN OF gs_grhd.
         INCLUDE STRUCTURE ZC103MMt0013.           " 송장 헤더 테이블
DATA :   cell_tab         TYPE lvc_t_styl,
         bpname           TYPE zc103mmt0002-name,  " 공급업체명
         status_text(32),                          " 송장 상태 도메인 텍스트
         status           TYPE icon_d,             " 송장 상태 아이콘ㅇ
         check(32),                                " 검증자 사번
         checker_name(32),                         " 검증자 이름
         excel            TYPE icon_d,
       END OF gs_grhd,
       gt_grhd     LIKE TABLE OF gs_grhd,          " 송장헤더 itab (필터링 후)
       gt_grhdback LIKE TABLE OF gs_grhd.          " 송장헤더 itab (필터링 전)

*-- 구매오더 alv grid
DATA : gt_po_fcat TYPE lvc_t_fcat,  " 구매오더 alv 필드카탈로그
       gs_po_layo TYPE lvc_s_layo,  " 구매오더 alv 레이아웃
       gs_po_vari TYPE disvariant.  " 구매오더 alv variant

*-- 구매오더 아이템 alv grid
DATA : gt_poid_fcat TYPE lvc_t_fcat, " 구매오더 아이템 alv 필드카탈로그
       gs_poid_layo TYPE lvc_s_layo, " 구매오더 아이템alv 레이아웃
       gs_poid_vari TYPE disvariant. " 구매오더 아이템alv variant

*-- 입고 문서 alv grid
DATA : gt_rece_fcat TYPE lvc_t_fcat," 자재입고 alv 필드카탈로그
       gs_rece_layo TYPE lvc_s_layo," 자재입고 alv 레이아웃
       gs_rece_vari TYPE disvariant." 자재입고 alv variant

*-- 송장 alv grid
DATA : gt_gr_fcat TYPE lvc_t_fcat, " 송장헤더 필드카탈로그
       gs_gr_layo TYPE lvc_s_layo, " 송장헤더 레이아웃
       gs_gr_vari TYPE disvariant. " 송장헤더 베리언트

*-- 구매오더 헤더 트리용
DATA : BEGIN OF gs_pohdtree,
         month(4),                 " 월
         received     TYPE i,      " 입고수량
         not_received TYPE i,      " 미입고수량
       END OF gs_pohdtree,
       gt_pohdtree LIKE TABLE OF gs_pohdtree, " 구매오더헤더 트리
       gt_empohdtr LIKE TABLE OF gs_pohdtree. " 빈 트리 (채우기용)

*-- 입고일자 기준 ALV_트리
DATA : gs_receipt_hierhdr TYPE treev_hhdr, " 트리데이터
       gs_receipt_variant TYPE disvariant,
       gt_list_commentary TYPE slis_t_listheader.

*-- ALV_트리 이벤트
DATA : gt_receipt_events TYPE cntl_simple_events, "
       gs_receipt_events TYPE cntl_simple_event.

*-- ALV_트리 fcat
DATA : gt_receipt_fcat TYPE lvc_t_fcat,
       gs_receipt_fcat TYPE lvc_s_fcat.


*-- 저장위치 트리용
DATA : BEGIN OF gs_stlotree,
         plnid        TYPE zc103mmt0003-plnid,    " 플랜트번호
         pname        TYPE zc103mmt0003-name,     " 플랜트명
         address      TYPE zc103mmt0003-address,  " 플랜트주소
         strid        TYPE zc103mmt0004-strid,    " 저장위치번호
         sname        TYPE zc103mmt0004-name,     " 저장위치명
         received     TYPE i,                     " 입고건수
         not_received TYPE i,                     " 미입고건수
         nodekey      TYPE i,                     " 트리 키
       END OF gs_stlotree,
       gt_stlotree LIKE TABLE OF gs_stlotree,
       gt_sttrindx LIKE TABLE OF gs_stlotree, " 노드 인덱스 저장용
       gt_emsthdtr LIKE TABLE OF gs_stlotree.

*-- 저장위치 기준 ALV_트리
DATA : gs_stlo_hierhdr         TYPE treev_hhdr,
       gs_stlo_variant         TYPE disvariant,
       gt_stlo_list_commentary TYPE slis_t_listheader.

*-- ALV_트리 이벤트
DATA : gt_stlo_events TYPE cntl_simple_events, " 저장위치 트리 이벤트
       gs_stlo_events TYPE cntl_simple_event.

*-- ALV_트리 fcat
DATA : gt_stlo_fcat TYPE lvc_t_fcat,
       gs_stlo_fcat TYPE lvc_s_fcat.


*-- 마스터데이터
DATA : gt_mara  TYPE TABLE OF ZC103mmt0001, " 자재마스터 itab
       gs_mara  TYPE ZC103MMt0001,          " 자재마스터 스트럭쳐
       gt_plant TYPE TABLE OF ZC103MMt0003, " 플랜트마스터 itab
       gs_plant TYPE ZC103MMt0003,          " 플랜트마스터 스트럭쳐
       gt_stlo  TYPE TABLE OF ZC103MMt0004, " 저장위치마스터 itab
       gs_stlo  TYPE zc103mmt0004,          " 저장위치마스터 스트럭쳐
       gt_emp   TYPE TABLE OF zc103fit0011, " 직원마스터 itab
       gs_emp   TYPE zc103fit0011,          " 직원마스터 스트럭쳐
       gt_vend  TYPE TABLE OF ZC103MMt0002, " 공급업체마스터 itab
       gs_vend  TYPE ZC103MMt0002.          " 공급업체마스터 스트럭쳐

*-- 송장조회용 구매오더 선택 itab
DATA : gs_sel_gr           TYPE zc103mmt0013, " 핫스팟 클릭으로 선택된 구매오더
       gt_sel_item_gr_fcat TYPE lvc_t_fcat,   "
       gs_sel_item_gr_layo TYPE lvc_s_layo,
       gs_sel_item_gr_vari TYPE disvariant.

*-- 송장 아이템
DATA : BEGIN OF gs_sel_item_gr.
         INCLUDE STRUCTURE zc103mmt0014.
DATA :   matname TYPE zc103mmt0001-matname,
       END OF gs_sel_item_gr,
       gt_sel_item_gr LIKE TABLE OF gs_sel_item_gr.

*-- 송장 삭제용
DATA : gt_gr_delt LIKE gt_grhd. " 삭제용 행 저장.

*-- 송장 검증 itab
TYPES : BEGIN OF ts_checkgr,
          status      TYPE icon_d,
          grid        TYPE zc103mmt0013-grid,   " 송장번호
          ivid        TYPE zc103mmt0015-ivid,   " 입고번호
          poid        TYPE zc103mmt0011-poid,   " 구매오더번호
          matid       TYPE zc103mmt0001-matid,    " 자재번호
          matname     TYPE zc103mmt0001-matname,  " 자재이름
          po_quantity TYPE zc103mmt0012-quantity, " 구매오더 수량
          po_unit     TYPE zc103mmt0012-unit,     " 구매오더 단위
          iv_quantity TYPE zc103mmt0015-quantity, " 입고 수량
          iv_unit     TYPE zc103mmt0015-unit,     " 입고 단위
          gr_quantity TYPE zc103mmt0014-quantity, " 송장 수량
          gr_unit     TYPE zc103mmt0014-unit,     " 송장 단위
          po_price    TYPE zc103mmt0012-price,    " 구매오더 가격
          po_currency TYPE zc103mmt0012-currency, " 구매오더 통화
          iv_price    TYPE zc103mmt0015-price,    " 구매오더 가격
          iv_currency TYPE zc103mmt0015-currency, " 구매오더 통화
          gr_price    TYPE zc103mmt0014-total_price,    " 구매오더 가격
          gr_currency TYPE zc103mmt0014-currency, " 구매오더 통화
        END OF ts_checkgr,
        tt_checkgr TYPE TABLE OF ts_checkgr.

DATA : gs_checkgr TYPE ts_checkgr,
       gt_checkgr TYPE tt_checkgr.

*-- 송장검증 itab fcat
DATA : gt_checkgr_fcat TYPE lvc_t_fcat,
       gs_checkgr_fcat TYPE lvc_s_fcat,
       gs_checkgr_layo TYPE lvc_s_layo,
       gs_checkgr_vari TYPE disvariant.

*-- 사원번호 search help
DATA : BEGIN OF gs_emp_value,               " 직원 서치헬프 테이블
         empno   TYPE zc103fit0011-empno,   " 사원번호
         empname TYPE zc103fit0011-empname, " 이름
         dptcode TYPE zc103fit0011-dptcode, " 부서코드
       END OF gs_emp_value,
       gt_emp_value LIKE TABLE OF gs_emp_value.


**********************************************************************
* Common variable
**********************************************************************
DATA : gv_okcode TYPE sy-ucomm. " okcode

DATA : gv_logo TYPE sdydo_value. " 회사로고

* 입고
DATA : gv_re_aprv_name  TYPE zc103fit0011-empname, " 입고자 이름
       gv_re_aprv_dept  TYPE zc103fit0011-dptcode, " 입고자 부서코드
       gv_re_aprv_empno TYPE ZC103fit0011-empno,   " 입고저 사원번호
       gv_po_aprv_name  TYPE zc103fit0011-empname, " 구매오더 승인자 이름
       gv_po_aprv_dept  TYPE zc103fit0011-dptcode. " 구매오더 승인자 부서코드

" 입고 모드
DATA : gv_rece(1). " 입고 조회 모드, 'E' : 입고, 'V' : 조화

DATA : gv_sel_node_key TYPE lvc_nkey. " 선택노드저장

" 툴바 exclude
DATA : gt_ui_functions TYPE ui_functions.

" 송장 엑셀 파일 업로드
DATA : w_pickedfolder  TYPE string,
       w_initialfolder TYPE string,
       w_fullinfo      TYPE string,
       w_pfolder       TYPE rlgrap-filename. "MEMORY ID mfolder.

" 송장 저장 확인
DATA : gv_save_gr(1).

" 송장 ALV 모드 ( 검증완료시 송장업로드 비활성화 )
DATA : gv_checkgr_mode(1) VALUE 'V'. " E : 수정, V : 조회

" 검증자 정보
DATA : gv_checker         TYPE zc103fit0011-empno,   " 검증자 사원번호
       gv_checker_name    TYPE zc103fit0011-empname, " 검증자 이름
       gv_checker_dptcode TYPE zc103fit0011-dptcode. " 검증자 부서코드

**********************************************************************
* For Excel
**********************************************************************
DATA : objfile       TYPE REF TO cl_gui_frontend_services,
       pickedfolder  TYPE string,
       initialfolder TYPE string,
       fullinfo      TYPE string,
       pfolder       TYPE rlgrap-filename. "MEMORY ID mfolder.

DATA : gv_temp_filename     LIKE rlgrap-filename,
       gv_temp_filename_pdf LIKE rlgrap-filename,
       gv_form(40).

DATA: excel       TYPE ole2_object,
      workbook    TYPE ole2_object,
      books       TYPE ole2_object,
      book        TYPE ole2_object,
      sheets      TYPE ole2_object,
      sheet       TYPE ole2_object,
      activesheet TYPE ole2_object,
      application TYPE ole2_object,
      pagesetup   TYPE ole2_object,
      cells       TYPE ole2_object,
      cell        TYPE ole2_object,
      row         TYPE ole2_object,
      buffer      TYPE ole2_object,
      font        TYPE ole2_object,
      range       TYPE ole2_object,  " Range
      borders     TYPE ole2_object.

DATA: cell1 TYPE ole2_object,
      cell2 TYPE ole2_object.

" 선택 송장 정보
DATA : gs_sel_grhd LIKE gs_grhd,
       gt_sel_grit TYPE TABLE OF ZC103MMt0014,
       gs_sel_grit TYPE ZC103MMt0014.

*-- For Excel
DATA: gv_tot_page   LIKE sy-pagno,          " Total page
      gv_percent(3) TYPE n,                 " Reading percent
      gv_file       LIKE rlgrap-filename .  " File name

DATA: lt_excel_raw TYPE TABLE OF alsmex_tabline,  " 엑셀 원본 데이터를 저장할 인터널테이블
      ls_raw       TYPE alsmex_tabline,           " 엑셀 원본 데이터의 개별 행을 저장하는 구조
      lt_excel     LIKE gt_grhd,                  " 변환된 데이터를 저장할 인터널테이블
      ls_excel     LIKE gs_grhd,                   " 개별 행 데이터를 저장할 구조
      lv_file      TYPE rlgrap-filename,          " 업로드할 엑셀 파일의 경로를 저장할 변수
      lv_value     TYPE string.                   " 개별 셀 값을 저장할 변수

*-- For Refresh Tree
DATA: gv_refresh_tree_needed(1). " 트리 새로고침용


**********************************************************************
* E-Mail
**********************************************************************
DATA : gt_sel_grhd LIKE gt_grhd. " 메일로 전송할 송장 저장용

" 파일업로드
DATA : gt_files      TYPE filetable,  " 업로드 할 파일명 저장
       gs_file       TYPE file_table,
       gt_attachment TYPE solix_tab. " 업로드 파일 바이너리 테이블

" 메일
DATA: gv_subject      TYPE so_obj_des,
      gv_body_text    TYPE bcsy_text,
      gv_receiver     TYPE ad_smtpadr,
      gv_sender       TYPE REF TO cl_sapuser_bcs,
      go_send_request TYPE REF TO cl_bcs,
      go_document     TYPE REF TO cl_document_bcs,
      go_recipient    TYPE REF TO if_recipient_bcs.

----------------------------------------------------------------------------------
Extracted by Direct Download Enterprise version 1.3.1 - E.G.Mellodew. 1998-2005 UK. Sap Release 758
```
