```abap
FUNCTION-POOL zc1fg03mm0001.                "MESSAGE-ID ..

* INCLUDE LZC1FG03MM0001D...                 " Local class definition

**********************************************************************
* Class instance
**********************************************************************
" ALV
DATA : go_cont TYPE REF TO cl_gui_custom_container,
       go_grid TYPE REF TO cl_gui_alv_grid.

**********************************************************************
* Internal table and work area
**********************************************************************
" 송장 헤더, 아이템
DATA : gt_grhd TYPE TABLE OF ZC103mmt0013,
       gs_grhd TYPE ZC103mmt0013.

DATA : BEGIN OF gs_grit.
         INCLUDE STRUCTURE zc103mmt0014.
DATA :   matname TYPE ZC103mmt0001-matname,
       END OF gs_grit,
       gt_grit LIKE TABLE OF gs_grit.

" 시리얼
DATA : BEGIN OF gs_serial.
  INCLUDE TYPE zc103mmt0020.
DATA : pname TYPE zc103mmt0003-name,
       sname TYPE zc103mmt0004-name,
       END OF gs_serial,
       gt_serial LIKE TABLE OF gs_serial.


" 마스터 데이터
DATA : gt_vend TYPE ZC103MMtt0003, " 벤더
       gs_vend TYPE zc103mms0004,
       gt_mara TYPE zc103mmtt0004, " 자재
       gt_emp  TYPE zc103fitt0001, " 직원
       gt_matq TYPE TABLE OF zc103mmt0006, " 재고수량
       gs_plnt TYPE zc103mmt0003, " 플랜트
       gt_plnt TYPE TABLE OF zc103mmt0003,
       gs_stlo TYPE zc103mmt0004, " 저장위치
       gt_stlo TYPE TABLE OF zc103mmt0004.

**********************************************************************
* Common variable
**********************************************************************
DATA : gv_bpname  TYPE ZC103mmt0002-name, " 벤더명
       gv_empname TYPE ZC103fit0011-empname, " 사원명
       gv_dptcode TYPE ZC103fit0011-dptcode. " 부서코드

" 3번 펑션모듈용
DATA : gv_selection_mode TYPE xfeld. " 선택시 시리얼 선택 가능
DATA : gt_sel_serial     TYPE zc103mmtt0009. " 20번 테이블
DATA : gt_sel_serial_roid TYPE lvc_t_roid.  " 선택된 시리얼 저장용 (개수 세기)
DATA : gv_sel_serial_cnt TYPE i VALUE 0.
DATA : gv_serial_mode_text(16), " 조회모드 출력
       gv_serial_cnt TYPE i. " 전체 시리얼 수

INCLUDE lzc1fg03mm0001d01.

----------------------------------------------------------------------------------
Extracted by Direct Download Enterprise version 1.3.1 - E.G.Mellodew. 1998-2005 UK. Sap Release 758
```
