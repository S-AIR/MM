```abap
*&---------------------------------------------------------------------*
*& Include SAPMZC103MM0001TOP                       - Module Pool      SAPMZC103MM0001
*&---------------------------------------------------------------------*
PROGRAM sapmzc103mm0001 MESSAGE-ID zmsgc103.

TABLES : zc103mmt0013.

**********************************************************************
* Class Instance
**********************************************************************
DATA : go_cont TYPE REF TO cl_gui_custom_container,
       go_grid TYPE REF TO cl_gui_alv_grid.

**********************************************************************
* Internal table and Work area
**********************************************************************
" 송장 헤더
DATA : BEGIN OF gs_grhd.
         INCLUDE STRUCTURE zc103mmt0013.
DATA :   color        TYPE lvc_t_scol,
         icon         TYPE icon_d,
         status_text(32),
         bpname       TYPE zc103mmt0002-name,
         checker_name TYPE zc103fit0011-empname,
         celltab      TYPE lvc_t_styl,
         btn1(16),
       END OF gs_grhd,
       gt_grhd      LIKE TABLE OF gs_grhd,
       gt_grhd_back LIKE TABLE OF gs_grhd.

" 마스터데이터
DATA : gs_vend TYPE zc103mmt0002,
       gt_vend TYPE TABLE OF zc103mmt0002,
       gs_mara TYPE zc103mmt0001,
       gt_mara TYPE TABLE OF zc103mmt0001,
       gs_emp  TYPE zc103fit0011,
       gt_emp  TYPE TABLE OF zc103fit0011.

" For alv grid
DATA : gt_fcat TYPE lvc_t_fcat,
       gs_layo TYPE lvc_s_layo,
       gs_vari TYPE disvariant.

**********************************************************************
* Common variable
**********************************************************************
RANGES : gr_receive_date FOR zc103mmt0013-receive_date, " 납품일자 검색용
         gr_grid         FOR zc103mmt0013-grid,
         gr_year         FOR zc103mmt0013-gjahr.

DATA : gv_today TYPE dats, " 오늘 날짜
       gv_current_time TYPE tims. " 조회 시간

DATA : gv_total_cnt TYPE i,   " 전체 건수
       gv_warning_cnt TYPE i, " 검증 필요 건수
       gv_ok_cnt TYPE i.      " 검증 완료 건수
```