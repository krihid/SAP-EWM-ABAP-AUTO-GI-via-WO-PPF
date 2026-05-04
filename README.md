# AUTO-GI-via-WO-PPF
AUTO GI via WO PPF


# How To Run
 
1.  Create Action Definition within Action Profile /SCWM/WO
2.  Implement a classic BaDI call for GI POST in Z class
3.  Create WO condition records

# Introduction
This document describes the automatic GI posting as implemented in various customer projects.
The automatic posting of WA documents is to be controlled via a warehouse order PPF, since the DELIVERY PPF does not provide the necessary conditions to determine the exact time for the WA posting.
A prerequisite for implementation is a basic understanding of how background processes via PPF work and are implemented.

# Brief Description
The steps for implementation are outlined below:
</> Markdown
* Create Z-action definition in action profile /SCWM/WO
* Implementation of the goods issue posting
* Creation of condition records  

# Customizing
</> Markdown
Transaktion	Beschreibung
SPPFCADM	Bereitstellung PPF Anwendung
| Transaction | Description                |
|-------------|----------------------------|
| SPPFCADM    | Providing PPF application  |

 
(1)    Creation of an action definition in the action profile /SCWM/WO
<img width="929" height="338" alt="image" src="https://github.com/user-attachments/assets/0330b71c-f97d-483d-94b4-eb20d9c45e35" />



(2)    The defined action uses a Z-method
<img width="945" height="331" alt="image" src="https://github.com/user-attachments/assets/0a48a80d-7249-4b39-be24-82d83759b82e" />


 
(3)    The defined promotion uses the scheduling condition /SCWM/WOSINGLE
<img width="635" height="369" alt="image" src="https://github.com/user-attachments/assets/25928917-aaa5-4e29-bb5a-a5a0250a39a4" />


 
# Workbench
The following logic is used for posting goods issue:
<img width="804" height="401" alt="image" src="https://github.com/user-attachments/assets/3b4eb53b-8bce-428d-87bb-ba67caf4903d" />
 
The steps for the goods issue posting are outlined below
(1)    Determination of runtime data from the PPF object
(2)    Determination of the document reference (docid)
(3)    Check
(4)    Posting


</> abap

# Determination runtime data
TRY.
    DATA(lv_lgnum) = lo_wo_ppf->get_lgnum( ).
    DATA(lv_who)   = lo_wo_ppf->get_who( ).
    DATA(lv_tanum) = lo_wo_ppf->get_tanum( ).

  CATCH cx_os_object_not_found.
    RETURN.
ENDTRY.



# Determination document reference / document
</> abap
SELECT vlenr, qdoccat, qdocid, qitmid,
       rdoccat, rdocid, ritmid
  FROM /scwm/ordim_c
  UP TO 1 ROWS
  INTO CORRESPONDING FIELDS OF TABLE @lt_ordim_c_pick
 WHERE lgnum   = @mv_lgnum
   AND nlenr   = @ls_ordim_c-vlenr
   AND flghuto  = @abap_false
 ORDER BY confirmed_at DESCENDING.

DATA(ls_ordim_c_pick) = VALUE #( lt_ordim_c_pick[ 1 ] OPTIONAL ).

# Reading document item
</> abap
SELECT SINGLE docid, itemid, doccat
  FROM /scdl/db_proci_o
  INTO CORRESPONDING FIELDS OF @ls_docid
 WHERE docid  = @ls_ordim_c_pick-qdocid
   AND itemid = @ls_ordim_c_pick-qitmid
   AND doccat = @ls_ordim_c_pick-qdoccat.

IF ls_docid IS INITIAL.
  SELECT SINGLE docid, itemid, doccat
    FROM /scdl/db_proci_o
    INTO CORRESPONDING FIELDS OF @ls_docid
   WHERE docid  = @ls_ordim_c_pick-rdocid
     AND itemid = @ls_ordim_c_pick-ritmid
     AND doccat = @ls_ordim_c_pick-rdoccat.
ENDIF.

# Warehouse Request
</> abap
CALL FUNCTION '/SCWM/WHR_QUERY'
  EXPORTING
    it_docid          = lt_docid
    iv_doccat_whr     = ls_docid-doccat
    is_read_options   = ls_read_options
    is_include_data   = ls_include_data
  IMPORTING
    et_whr_headers    = lt_whr_headers
    et_whr_items      = lt_whr_items
    et_huhdr          = lt_huhdr
  EXCEPTIONS
    OTHERS            = 1.

# Status Check
</> abap
CASE ls_docid-doccat.

  WHEN /scdl/if_dl_doc_c=>sc_doccat_out_prd.
    lv_status = /scdl/if_dl_status_c=>sc_t_goods_issue.

  WHEN /scdl/if_dl_doc_c=>sc_doccat_inb_prd.
    lv_status = /scdl/if_dl_status_c=>sc_t_goods_receipt.

  WHEN OTHERS.
    ASSERT 1 = 2.
ENDCASE.

# HU cross check
</> abap

LOOP AT lt_huhdr ASSIGNING <hu>.

  /scwm/cl_dlv_prd2hum=>check_cross_hu(
    EXPORTING iv_guid_hu = <hu>-guid_hu
    IMPORTING ev_cross_hu = lv_xhu ).

  IF lv_xhu = abap_true.
    MESSAGE e018(/scwm/goods_movement) WITH <hu>-huident.
    rp_status = sppf_status_error.
    RETURN.
  ENDIF.

ENDLOOP.

# Posting in Update Task
</> abap
CALL FUNCTION '/SCWM/PPF_GM_POST_UPD'
  IN UPDATE TASK
  EXPORTING
    is_docid  = ms_docid
    ip_action = ip_action
    iv_qname  = lv_qname
    iv_guid   = lv_guid.

MESSAGE s428(/scwm/delivery) WITH lv_qname INTO lv_msg_txt.

cl_log_ppf=>add_message(
  ip_problemclass = sppf_pclass_5
  ip_handle       = ip_application_log ).

rp_status = sppf_status_processed.
    
