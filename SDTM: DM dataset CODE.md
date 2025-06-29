
# Clinical Project: SDTM Domain Creation (DM)

This document outlines the SAS code used to process and transform raw clinical data from the CDM library into the SDTM DM domain format.

---

## 📁 Library Assignments
```sas
libname CDM "/home/u63603412/clinical_project_part_1/CDM";
libname SDTM "/home/u63603412/clinical_project_part_1/SDTM";
```

---

## 🧱 Step 1: Load and Format Demographic Data
```sas
data A;
SET CDM.dm;
STUDYID = "XYZ";
DOMAIN = "DM";
usubjid= cat(STUDYID,"/",SUBJCTID,"/", SITEID);
run;
```

---

## 📅 Step 2: Derive RFSTDTC from SPCPKB1
```sas
DATA B (DROP= IPFD1DAT IPFD1TIM);
SET CDM.SPCPKB1;
IF NOT MISSING(IPFD1DAT) and PSCHDAY = 1 AND PART = "A" THEN RFSTDTC= CAT(IPFD1DAT,"T", IPFD1TIM);
else if missing (RFSTDTC) then delete;
RUN;
```

---

## 🔗 Step 3: Merge All Required Datasets
```sas
Data DM1 ;
Merge A B CDM.EX CDM.DS CDM.DEATH CDM.IE;
By SUBJECT;
Run;
```

---

## 📌 Step 4: Derive RFENDTC and Other Variables
```sas
DATA DM2;
SET DM1 ;
if not missing(EXENDAT) then RFENDTC= CAT(IPFD1DAT,"T", IPFD1TIM);
else if Missing(EXENDAT) then RFENDTC= EXSTDAT;

SITEID = CENTRE;
BRTHDTC = BRTHDAT;
AGE=AGE;
DMDTC = VIS_DAT;
CENTRE = CENTRE;
PART = PART;
RACEOTH = upcase(RACEOTH);
VISITDTC = VIS_DAT;

If AGEU = 'C29848' then AGEU = "YEARS";
ELSE IF SEX='C20197' then SEX= 'M'; 
ELSE IF SEX='C16576' then SEX= 'F'; 
ELSE SEX='U';

If RACE = 'C41260' then RACE = 'ASIAN';
ELSE If RACE = 'C41261' then RACE = 'WHITE';
ELSE If ETHNIC = 'C41222' then ETHNIC = "NOT HISPANIC OR LATINO";

RFXSTDTC = RFSTDTC;  
RFXENDTC = RFENDTC; 
RFPENDTC = DSSTDAT;
DTHDTC   = DTH_DAT;
BRTHDTC  = BRTHDAT;

IF NOT MISSING(DTHDTC) then DTHFL = "Y";
RUN;
```

---

## 🌐 Step 5: Assign ARMCD, ARM, and COUNTRY
```sas
data arm;
set DM2;

IF NOT MISSING(RFSTDTC) then ARMCD = "A01-A02-A03";
else if IEYN = "0" and RFSTDTC="null" then ARMCD = "SCRNFAIL";
else ARMCD = "NOTASSGN";

if ARMCD = "SCRNFAIL" then ARM = "Screen Failure";
else if ARMCD = "NOTASSGN" then ARM = "Not Assigned";

ACTARMCD=ARMCD;
ACTARM = ARM;
CEN= put(CENTRE,6.);

IF substr(strip(CEN),1,2) in (23,24) then country="FRA";
else IF substr(strip(CEN),1,2)= 60 then country="KOR";
else IF substr(strip(CEN),1,2) in (70,71) then country="ESP";
run;
```

---

## 📝 Final Output: Generate Report
```sas
%let var = STUDYID DOMAIN USUBJID SUBJID RFSTDTC RFENDTC RFXSTDTC RFXENDTC RFPENDTC DTHDTC DTHFL
SITEID BRTHDTC AGE AGEU SEX RACE ETHNIC ARMCD ARM ACTARMCD ACTARM COUNTRY DMDTC
CENTRE PART RACEOTH VISITDTC ;

ods pdf file="/home/u63603412/clinical_project_part_1/SDTM.pdf";
proc report data=dm2;
column &var;
run;
ods pdf close;
```

---

This code creates the SDTM DM dataset by deriving key variables from raw clinical data sources, performing transformations, and generating a summary report in PDF format.
