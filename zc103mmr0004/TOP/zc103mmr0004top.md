```abap
*&---------------------------------------------------------------------*
*& Include ZC103MMR0004TOP                          - Report ZC103MMR0004
*&---------------------------------------------------------------------*
REPORT zc103mmr0004 MESSAGE-ID zmsgc103.

**********************************************************************
* Area for node
**********************************************************************
TYPES : node_table_type LIKE STANDARD TABLE OF mtreesnode WITH DEFAULT KEY.
DATA : node_table TYPE node_table_type.

**********************************************************************
* Class instance
**********************************************************************
DATA : go_main_cont  TYPE REF TO cl_gui_custom_container,
       go_split_cont TYPE REF TO cl_gui_splitter_container,
       go_left_cont  TYPE REF TO cl_gui_container,
       go_right_cont TYPE REF TO cl_gui_container,
       go_alv_grid   TYPE REF TO cl_gui_alv_grid,       " 재고 조회 alv
       go_tree       TYPE REF TO cl_gui_simple_tree.    " 플랜트, 스토리지로케이션 트리

**********************************************************************
* Events
**********************************************************************
DATA : events TYPE cntl_simple_events,
       event  TYPE cntl_simple_event.

**********************************************************************
* Internal table and work area
**********************************************************************
DATA : BEGIN OF gs_tr_plnt,
         plnid TYPE ZC103MMt0003-plnid,
         pname TYPE ZC103MMt0003-name,
         strid TYPE ZC103MMt0004-strid,
         sname TYPE zc103mmt0004-name,
       END OF gs_tr_plnt,
       gt_tr_plnt LIKE TABLE OF gs_tr_plnt.

" for fcat
DATA : gt_fcat TYPE lvc_t_fcat,
       gs_layo TYPE lvc_s_layo,
       gs_vari TYPE disvariant.

" 마스터데이터
DATA : gt_plnt TYPE TABLE OF zc103mmt0003,
       gs_plnt TYPE zc103mmt0003,
       gt_stlo TYPE TABLE OF zc103mmt0004,
       gs_stlo TYPE zc103mmt0004,
       gt_mara TYPE TABLE OF zc103mmt0001,
       gs_mara TYPE zc103mmt0001.

" 구매오더 테이블
DATA : BEGIN OF gs_po,
         matid        TYPE zc103mmt0012-matid,
         plnid        TYPE zc103mmt0011-plnid,
         strid        TYPE zc103mmt0011-strid,
         quantity     TYPE zc103mmt0012-quantity,
         order_status TYPE zc103mmt0011-order_status,
       END OF gs_po,
       gt_po LIKE TABLE OF gs_po.

" 재고 테이블
DATA : BEGIN OF gs_stock.
         INCLUDE TYPE zc103mmt0006.
DATA :   matname         TYPE zc103mmt0001-matname,
         matgrp          TYPE zc103mmt0001-matgrp,
         matgrp_text(32),
         rpoint          TYPE ZC103MMt0001-rpoint,
         is_serial       TYPE zc103mmt0001-is_serial,
         pname           TYPE zc103mmt0004-name,
         sname           TYPE zc103mmt0004-name,
         celltab         TYPE lvc_t_styl,
         color           TYPE lvc_t_scol,
         serial(32),
         status(1),  " A : 재고보충진행, B : 재고보충필요 (구매오더아이템 정보 읽어서 확인)
         po_quantity     TYPE zc103mmt0012-quantity, " 구매요청승인 상태인 재고 있는지 확인
         expected_stock  TYPE zc103mmt0006-available_stock, " 예상재고
         status_icon     TYPE icon_d,
         out_quantity    TYPE ZC103MMt0012-quantity, " 출고 예정 수량
       END OF gs_stock,
       gt_stock        LIKE TABLE OF gs_stock,
       gt_stock_backup LIKE TABLE OF gs_stock.

" 정비 BOM 테이블
DATA : gt_pm_bom TYPE TABLE OF zc103pmt0002,
       gs_pm_bom TYPE zc103pmt0002.

**********************************************************************
* Common Variable
**********************************************************************
DATA : gv_title(32).

" 건수
DATA : gv_total_cnt   TYPE i, " 전체 재고조회 건수
       gv_suppl_cnt   TYPE i, " 보충이 필요한 재고 건수
       gv_ok_cnt      TYPE i, " 보충이 필요하지 않은 재고 건수
       gv_warning_cnt TYPE i, " 가용재고와 예상가용재고수가 같은 재고 건수
       gv_error_cnt   TYPE i. " 보충이 필요한 재고 건수

" 조회정보
DATA : gv_today        TYPE sy-datum,
       gv_current_time TYPE sy-uzeit.

----------------------------------------------------------------------------------
Extracted by Direct Download Enterprise version 1.3.1 - E.G.Mellodew. 1998-2005 UK. Sap Release 758
```
