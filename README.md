# Method as below
![image](https://github.com/liuhaibohyde/SAP/blob/master/Untitled%20picture.png)

  METHOD upload_xml_file.

    "a specific data with type IV_DDICTYPE to receive EV_DATA is required
    DATA: lo_data TYPE REF TO data.
    FIELD-SYMBOLS <fs_data> TYPE any.

    CREATE DATA lo_data TYPE (iv_ddictype).
    ASSIGN lo_data->* TO <fs_data>.

    DATA: lv_xml      TYPE x LENGTH 1000,
          lt_xml      LIKE STANDARD TABLE OF lv_xml,
          lt_filepath TYPE filetable,
          lv_filenum  TYPE i.

    IF iv_filename IS INITIAL.
      CALL METHOD cl_gui_frontend_services=>file_open_dialog
        CHANGING
          file_table = lt_filepath
          rc         = lv_filenum.
    ELSE.
      APPEND iv_filename TO lt_filepath.
      lv_filenum = 1.
    ENDIF.
    IF lv_filenum = 1.
      CALL METHOD cl_gui_frontend_services=>gui_upload
        EXPORTING
          filename = CONV #( lt_filepath[ 1 ]-filename )
          filetype = 'BIN'
        CHANGING
          data_tab = lt_xml.
      DATA(lo_xml_reader) = cl_sxml_table_reader=>create( lt_xml ).
      cl_proxy_xml_transform=>xml_to_abap(
        EXPORTING
          ddic_type = iv_ddictype
          xml_reader = lo_xml_reader
        IMPORTING
          abap_data = <fs_data> ).

      ev_data = <fs_data>.

    ENDIF.

  ENDMETHOD.
