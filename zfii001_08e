*&---------------------------------------------------------------------*
*& Report zfii001_08e
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT zfii001_##.

TYPES: BEGIN OF ty_raw,
         bukrs      TYPE bkpf-bukrs,
         gjahr      TYPE bkpf-gjahr,
         belnr      TYPE bkpf-belnr,
         budat      TYPE bkpf-budat,
         waers      TYPE bkpf-waers,
         buzei      TYPE bseg-buzei,
         hkont      TYPE bseg-hkont,
         chave_lanc TYPE bseg-bschl,
         deb_cred   TYPE bseg-shkzg,
         valor      TYPE bseg-dmbtr,
       END OF ty_raw.

TYPES: BEGIN OF ty_output,
         empresa         TYPE bkpf-bukrs,
         ano             TYPE bkpf-gjahr,
         nrdocumento     TYPE bkpf-belnr,
         datalancamento  TYPE string,
         moeda           TYPE bkpf-waers,
         nr_item         TYPE bseg-buzei,
         contacontabil   TYPE bseg-hkont,
         chave_lanc      TYPE string,
         deb_cred        TYPE char1,
         valor           TYPE string,
       END OF ty_output.

DATA: lt_raw_data TYPE TABLE OF ty_raw,
      lt_output   TYPE TABLE OF ty_output,
      ls_output   TYPE ty_output.

SELECTION-SCREEN BEGIN OF BLOCK b1 WITH FRAME TITLE text-001.
  DATA: lv_bukrs TYPE bkpf-bukrs,
        lv_belnr TYPE bkpf-belnr.
  SELECT-OPTIONS: s_bukrs FOR lv_bukrs,
                  s_belnr FOR lv_belnr.
  PARAMETERS:     p_gjahr TYPE gjahr OBLIGATORY.
SELECTION-SCREEN END OF BLOCK b1.

AT SELECTION-SCREEN ON BLOCK b1.
  IF p_gjahr < '1900' OR p_gjahr > '2100'.
    MESSAGE text-002 TYPE 'E'.
  ENDIF.

  SELECT b~bukrs, b~gjahr, b~belnr, b~budat, b~waers,
         s~buzei, s~hkont, s~bschl, s~shkzg, s~dmbtr
    FROM bkpf AS b
    INNER JOIN bseg AS s ON b~bukrs = s~bukrs
                        AND b~gjahr = s~gjahr
                        AND b~belnr = s~belnr
    INTO TABLE @lt_raw_data
    WHERE b~bukrs IN @s_bukrs
      AND b~gjahr = @p_gjahr
      AND b~belnr IN @s_belnr
      AND b~blart = 'SA'.

  IF sy-subrc <> 0 OR lt_raw_data IS INITIAL.
    MESSAGE text-003 TYPE 'E'.
  ENDIF.

START-OF-SELECTION.

  LOOP AT lt_raw_data ASSIGNING FIELD-SYMBOL(<fs_raw>).
    CLEAR ls_output.
    ls_output-empresa        = <fs_raw>-bukrs.
    ls_output-ano            = <fs_raw>-gjahr.
    ls_output-nrdocumento    = <fs_raw>-belnr.
    ls_output-datalancamento = |{ <fs_raw>-budat+0(4) }{ <fs_raw>-budat+4(2) }{ <fs_raw>-budat+6(2) }|.
    ls_output-moeda          = <fs_raw>-waers.
    ls_output-nr_item        = <fs_raw>-buzei.
    ls_output-contacontabil  = <fs_raw>-hkont.
    ls_output-chave_lanc     = |{ <fs_raw>-chave_lanc ALPHA = OUT }|.

    IF <fs_raw>-deb_cred = 'S'.
      ls_output-deb_cred = 'D'.
    ELSEIF <fs_raw>-deb_cred = 'H'.
      ls_output-deb_cred = 'C'.
    ENDIF.

    DATA(lv_value_char) = |{ <fs_raw>-valor NUMBER = USER }|.
    REPLACE ALL OCCURRENCES OF '.' IN lv_value_char WITH ''.
    REPLACE ALL OCCURRENCES OF ',' IN lv_value_char WITH '.'.
    CONDENSE lv_value_char NO-GAPS.
    ls_output-valor = lv_value_char.

    APPEND ls_output TO lt_output.
  ENDLOOP.

  WRITE: / text-004, /.
  WRITE: / text-005.


  LOOP AT lt_output INTO ls_output.
    WRITE: / ls_output-empresa, ';',
             ls_output-ano, ';',
             ls_output-nrdocumento, ';',
             ls_output-datalancamento, ';',
             ls_output-moeda, ';',
             ls_output-nr_item, ';',
             ls_output-contacontabil, ';',
             ls_output-chave_lanc, ';',
             ls_output-deb_cred, ';',
             ls_output-valor.
  ENDLOOP.

  DATA: lv_action    TYPE i,
        lv_filename  TYPE string,
        lv_path      TYPE string,
        lv_fullpath  TYPE string,
        lt_csv_lines TYPE truxs_t_text_data.

  cl_gui_frontend_services=>file_save_dialog(
    EXPORTING
      window_title      = |{ text-006 }|
      default_extension = 'csv'
      default_file_name = 'lancamentos_contabeis.csv'
      file_filter       = 'Arquivos CSV (*.csv)|*.csv'
    CHANGING
      filename          = lv_filename
      path              = lv_path
      fullpath          = lv_fullpath
      user_action       = lv_action
    EXCEPTIONS
      OTHERS            = 1 ).

  IF lv_action = cl_gui_frontend_services=>action_cancel.
    EXIT.
  ENDIF.

  CALL FUNCTION 'SAP_CONVERT_TO_CSV_FORMAT'
    EXPORTING
      i_field_seperator    = ';'
    TABLES
      i_tab_sap_data       = lt_output
    CHANGING
      i_tab_converted_data = lt_csv_lines
    EXCEPTIONS
      OTHERS               = 1.

  DATA: lt_final_csv TYPE truxs_t_text_data.
  APPEND text-005 TO lt_final_csv.
  APPEND LINES OF lt_csv_lines TO lt_final_csv.

  CALL FUNCTION 'GUI_DOWNLOAD'
    EXPORTING
      filename                = lv_fullpath
      filetype                = 'ASC'
    TABLES
      data_tab                = lt_final_csv
    EXCEPTIONS
      OTHERS                  = 1.

  IF sy-subrc = 0.
    SKIP.
    WRITE: / '---------------------------------------------------------------------------------------------------------'.
    WRITE: / text-007, lv_fullpath.
  ENDIF.
