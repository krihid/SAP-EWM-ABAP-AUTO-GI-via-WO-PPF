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
 

 
