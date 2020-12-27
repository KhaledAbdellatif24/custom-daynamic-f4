# custom-daynamic-f4

  TYPES: BEGIN OF studclass,
       classid TYPE zclass0-classid,
       studentid TYPE zstudent0-studentid,
       studentname TYPE zstudent0-STUDENT_NAME,
       END OF studclass.

DATA: wa      TYPE studclass,
      it       TYPE TABLE OF studclass.
DATA: gt_return      TYPE TABLE OF ddshretval,
      gwa_return     TYPE ddshretval.
DATA: gwa_dynpfields TYPE dynpread,
      gt_dynpfields  TYPE TABLE OF dynpread.
DATA: classid1       TYPE zclass0-classid.

PARAMETERS: p_id TYPE zclass0-classid OBLIGATORY.
PARAMETERS: p_studid TYPE zstudent0-studentid.


AT SELECTION-SCREEN ON VALUE-REQUEST FOR p_studid.

  REFRESH gt_dynpfields.

  gwa_dynpfields-fieldname = 'P_ID'.
  APPEND gwa_dynpfields TO gt_dynpfields.

*Get plant value on the selection screen
  CALL FUNCTION 'DYNP_VALUES_READ'
    EXPORTING
      dyname               = sy-repid
      dynumb               = sy-dynnr
    TABLES
      dynpfields           = gt_dynpfields
    EXCEPTIONS
      invalid_abapworkarea = 1
      invalid_dynprofield  = 2
      invalid_dynproname   = 3
      invalid_dynpronummer = 4
      invalid_request      = 5
      no_fielddescription  = 6
      invalid_parameter    = 7
      undefind_error       = 8
      double_conversion    = 9
      stepl_not_found      = 10
      OTHERS               = 11.

  READ TABLE gt_dynpfields INTO gwa_dynpfields
         WITH KEY fieldname = 'P_ID'.
  IF sy-subrc = 0.
    classid1 = gwa_dynpfields-fieldvalue.
  ENDIF.


  SELECT a~classid
         b~studentid
         b~student_name
         UP TO 10 ROWS
         INTO TABLE it
         FROM zclass0 AS a
         INNER JOIN zstudent0 AS b
         ON a~classid = b~classid
         WHERE a~classid = classid1.

CALL FUNCTION 'F4IF_INT_TABLE_VALUE_REQUEST'
    EXPORTING
      retfield        = 'STUDENTID'
      value_org       = 'S'
    TABLES
      value_tab       = it
      return_tab      = gt_return
    EXCEPTIONS
      parameter_error = 1
      no_values_found = 2
      OTHERS          = 3.

  READ TABLE gt_return INTO gwa_return INDEX 1.
  IF sy-subrc = 0.
    p_studid = gwa_return-fieldval.
  ENDIF. 
