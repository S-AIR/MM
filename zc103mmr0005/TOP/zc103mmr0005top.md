```abap
*&---------------------------------------------------------------------*
*& Include ZC103MMR0005TOP                          - Report ZC103MMR0005
*&---------------------------------------------------------------------*
REPORT zc103mmr0005 MESSAGE-ID zmsgc103.

**********************************************************************
* Area for node
**********************************************************************
TYPES : node_table_type LIKE STANDARD TABLE OF mtreesnode WITH DEFAULT KEY.
DATA : node_table TYPE node_table_type.

**********************************************************************
* Class Instance
**********************************************************************
*-- splitter container
DATA : go_main_cont         TYPE REF TO cl_gui_custom_container,
       go_split_cont        TYPE REF TO cl_gui_splitter_container,
       go_rsplit_cont       TYPE REF TO cl_gui_splitter_container,
       go_left_cont         TYPE REF TO cl_gui_container,
       go_right_cont        TYPE REF TO cl_gui_container,
       go_right_top_cont    TYPE REF TO cl_gui_container,
       go_right_bottom_cont TYPE REF TO cl_gui_container,
       go_tree              TYPE REF TO cl_gui_simple_tree.

*-- Left t

*-- Right ALV
DATA : go_main_grid TYPE REF TO cl_gui_alv_grid, " 자재문서 헤더 그리드
       go_item_grid TYPE REF TO cl_gui_alv_grid. " 자재문서 아이템 그리드

**********************************************************************
* Events
**********************************************************************
DATA : events TYPE cntl_simple_events,
       event  TYPE cntl_simple_event.

**********************************************************************
* Internal table and work area
**********************************************************************
DATA : BEGIN OF gs_tr_move,
         movement_type          TYPE zc103mmt0016-movement_type,
         movement_type_text(32),
       END OF gs_tr_move,
       gt_tr_move LIKE TABLE OF gs_tr_move.

*-- ALV DATA
DATA : BEGIN OF gs_mdhd.
         INCLUDE STRUCTURE zc103mmt0016.
DATA :   celltab                TYPE lvc_t_styl,
         pname                  TYPE zc103mmt0003-name,
         sname                  TYPE zc103mmt0004-name,
         movement_type_text(32),
       END OF gs_mdhd,
       gt_mdhd      LIKE TABLE OF gs_mdhd,
       gt_mdhd_back LIKE TABLE OF gs_mdhd.

*-- For Material Document Grid
DATA : gt_fcat TYPE lvc_t_fcat,
       gs_layo TYPE lvc_s_layo,
       gs_vari TYPE disvariant.

*-- For Material Document Item grid
DATA : BEGIN OF gs_mdit.
         INCLUDE STRUCTURE zc103mmt0018.
DATA : matname TYPE zc103mmt0001-matname,
       matgrp  TYPE zc103mmt0001-matgrp,
       matgrp_text(32),
       matdocdate TYPE zc103mmt0016-matdocdate,
       is_serial  TYPE zc103mmt0001-is_serial,
       END OF gs_mdit,
       gt_mdit LIKE TABLE OF gs_mdit,
       gt_mdit_back LIKE TABLE OF gs_mdit.

*-- For Material Document Grid
DATA : gt_ifcat TYPE lvc_t_fcat,
       gs_ilayo TYPE lvc_s_layo,
       gs_ivari TYPE disvariant.

*-- Master data
DATA : gt_mara TYPE TABLE OF zc103mmt0001,
       gs_mara TYPE zc103mmt0001,
       gt_vend TYPE TABLE OF zc103mmt0002,
       gs_vend TYPE zc103mmt0002,
       gt_plnt TYPE TABLE OF zc103mmt0003,
       gs_plnt TYPE zc103mmt0003,
       gt_stlo TYPE TABLE OF zc103mmt0004,
       gs_stlo TYPE zc103mmt0004,
       gt_emp  TYPE TABLE OF zc103fit0011,
       gs_emp  TYPE zc103fit0011.

----------------------------------------------------------------------------------
Extracted by Direct Download Enterprise version 1.3.1 - E.G.Mellodew. 1998-2005 UK. Sap Release 758
```
