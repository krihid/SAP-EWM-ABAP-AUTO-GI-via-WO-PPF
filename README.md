# AUTO-GI-via-WO-PPF
AUTO GI via WO PPF


# How To Run
 
1.  Create Action Definition within Action Profile /SCWM/WO
2.  Implement a classic BaDI call for GI POST in Z class
3.  Create WO condition records

# Introduction
This document describes the automatic GI posting as implemented in the Pfeiffer Vacuum project.
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
 

<img width="929" height="338" alt="image" src="https://github.com/user-attachments/assets/0330b71c-f97d-483d-94b4-eb20d9c45e35" />

 


 
(2)    The defined action uses a Z-method
 

(3)    The defined promotion uses the scheduling condition /SCWM/WOSINGLE
 

Workbench
The following logic is used for posting goods issue:
 

The steps for the goods issue posting are outlined below
(1)    Determination of runtime data from the PPF object
(2)    Determination of the document reference (docid)
(3)    Check
(4)    GO posting
Determination of runtime data

Translated with DeepL.com (free version)
