```abap
*&---------------------------------------------------------------------*
*& Include SAPMZC103MM0002TOP                       - Module Pool      SAPMZC103MM0002
*&---------------------------------------------------------------------*
PROGRAM sapmzc103mm0002 MESSAGE-ID zmsgc103.



**********************************************************************
* Class Instance
**********************************************************************
DATA : go_cont TYPE REF TO cl_gui_custom_container,
       go_grid TYPE REF TO cl_gui_alv_grid.

DATA : go_item_cont TYPE REF TO cl_gui_custom_container,
       go_item_grid TYPE REF TO cl_gui_alv_grid.

**********************************************************************
* Internal table and Work area
**********************************************************************
" 구매요청 헤더
DATA : BEGIN OF gs_pohd.
         INCLUDE STRUCTURE zc103mmt0011.
DATA :   icon            TYPE icon_d,
         status_text(32),
         pname           TYPE zc103mmt0003-name,
         sname           TYPE zc103mmt0004-name,
         creator_name    TYPE ZC103fit0011-empname,
         approval_name    TYPE ZC103fit0011-empname,
         celltab         TYPE lvc_t_styl,
       END OF gs_pohd,
       gt_pohdback LIKE TABLE OF gs_pohd,
       gt_pohd     LIKE TABLE OF gs_pohd.

" 구매요청아이템
DATA : BEGIN OF gs_poit.
         INCLUDE STRUCTURE zc103mmt0012.
DATA : matname TYPE ZC103MMT0001-matname,
       bpname  TYPE ZC103MMT0002-name,
       END OF gs_poit,
gt_poit LIKE TABLE OF gs_poit.

" 마스터데이터
DATA : gs_vend TYPE zc103mmt0002,
       gt_vend TYPE TABLE OF zc103mmt0002,
       gs_mara TYPE zc103mmt0001,
       gt_mara TYPE TABLE OF zc103mmt0001,
       gs_emp  TYPE zc103fit0011,
       gt_emp  TYPE TABLE OF zc103fit0011,
       gs_plnt TYPE ZC103MMT0003,
       gt_plnt TYPE TABLE OF ZC103MMT0003,
       gs_stlo TYPE ZC103MMT0004,
       gt_stlo TYPE TABLE OF ZC103MMT0004.

" For alv grid
DATA : gt_fcat TYPE lvc_t_fcat,
       gs_layo TYPE lvc_s_layo,
       gs_vari TYPE disvariant.

" 구매오더아이템 fcat
DATA : gt_ifcat TYPE lvc_t_fcat,
       gs_ilayo TYPE lvc_s_layo,
       gs_ivari TYPE disvariant.

**********************************************************************
* Common variable
**********************************************************************
RANGES : gr_order_date   FOR zc103mmt0011-order_date,
         gr_receive_date FOR zc103mmt0011-receive_date,
         gr_poid         FOR zc103mmt0011-poid.

DATA : gv_today        TYPE dats, " 오늘 날짜
       gv_current_time TYPE tims. " 조회 시간

DATA : gv_total_cnt   TYPE i,   " 전체 건수
       gv_warning_cnt TYPE i, " 검증 필요 건수
       gv_ok_cnt      TYPE i.      " 검증 완료 건수

DATA : gs_sel_pohd LIKE gs_pohd. " 핫스팟 선택된 구매오더

DATA : gv_pname TYPE ZC103MMT0003-name,
       gv_sname TYPE ZC103MMT0004-name,
       gv_ordtext TYPE dd07v-ddtext.
```