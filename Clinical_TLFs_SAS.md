
# TLF Code and Outputs (SAS)

## 1. Listing: Medical History (Listing 16.2.4.2)

```sas
libname adam '/home/u63603412/study/sdtm/sdtm data';

data adm1 ;
set adam.admh ;
keep studyid usubjid subdisp mhstdtc mhendtc cohortn dose mhdecod mhterm astdt aendt mhclsig mhenrf mhseq ;
run ;

data adm2 ;
length col4B $ 200.;
set adm1;
col1=subdisp;
col2=cat("Cohort"," ",strip(put(cohortn, best.)),"/",strip(dose));
col3 = mhdecod ;
if not missing(astdt) then col4A =cat("S : ",strip(put(astdt,date9.))," E : ",strip(put(aendt,date9.))) ;
col4AL= length(col4A);
if length(col4A)=19 then col4=substr(col4A,1,14);
else col4=substr(col4A,1,27);
if mhenrf = "ONGOING" then col5 = "Yes" ;
else col5 = "No" ;
if mhclsig = "Y" then col6 = "Yes" ;
else if mhclsig = "N" then col6 = "No" ; 
keep usubjid mhseq col1-col6 ;
run ;

proc sort data=adm2;
by usubjid mhseq ;
run;

data adm22;
set adm2;
by usubjid mhseq ;
retain rown pgn 0;
rown + 1;
if rown>6 then do;
pgn=pgn+1;
rown=1;
end;
run;

data L16_2_4_2;
set adm22;
label
col1="Subject/Id/Age/Gender/Race"
col2="Cohort/Dose"
col3="Medical Condition/Description"
col4="S: Start Date E: Stop Date"
col5="Ongoing?"
col6="Clinically Significant?";
keep col: ;
run;

ods rtf file="/home/u63603412/study/listing.rtf" style=journal;
options orientation=landscape nodate nonumber papertype=A4  leftmargin=1in rightmargin=1in topmargin=1in bottommargin=1in;
ods escapechar="~";
title j=l font ='Courier New' height = 9pt "Study Code: XXX-YYY-103" j=right "Private and Confidential";
title2 j=l font ='Courier New' height = 9pt "Statistical Analysis Plan " j=right "Date: &sysdate ";
title3 j=l font ='Courier New' height = 9pt "Final Version 1.0" j=right 'page ~{pageof} ';
title4 font ='Courier New' height = 9pt j=center "Listing 16.2.4.2";
title5 font ='Courier New' height = 9pt j=center "Listing of Medical History";
title6 font ='Courier New' height = 9pt j=center "Safety Population";

proc report data=adm22 split="*"
style(report)={cellwidth=100% cellpadding=0.5% cellspacing = 5 } 
style(header)={vjust=t fontweight=bold FONT=("COURIER NEW") FONTSIZE=9PT} 
style(column)={vjust=t paddingbottom=4mm FONT=("COURIER NEW") FONTSIZE=9PT};
define col1 /order "Subject ID/*Age/*Gender/*Race" style(header)={just=l asis=on} style(column)={just=l asis=on cellwidth=12%};
define col2 /order "Cohort/*Dose" style(header)={just=l asis=on} style(column)={just=l asis=on cellwidth=13%};
define col3 /order=Internal "Medical Condition/Description" style(header)={just=l asis=on} style(column)={just=l asis=on cellwidth=13%};
define col4 / order=Internal "S: Start Date*E: Stop Date" style(header)={just=l asis=on} style(column)={just=l asis=on cellwidth=8%};
define col5 / order=Internal "Ongoing?" style(header)={just=l asis=on} style(column)={just=l asis=on cellwidth=8%};
define col6 / display "Clinically*Significant?" style(header)={just=l asis=on} style(column)={just=l asis=on cellwidth=12%};
footnote1 j=l font ='Courier New' height = 9pt "Note: M = Male; F = Female. Cohort 1: Caucasian; Cohort 2: Japanese.";
footnote2 j=l font ='Courier New' height = 9pt "Generated on &sysdate at &systime";
run;
ods rtf close;
```

## 2. Table: Summary of Students (Using `sashelp.class`)

```sas
data one1;
set sashelp.class;
OUTPUT;
SEX = 'TOTAL';
OUTPUT;
RUN;

proc sort data= one1;
by sex;
run;

proc means data= one1 noprint ;
var age height weight;
output out =dmg1
n= age_n hgt_n wgt_n
mean= age_mean hgt_mean wgt_mean
std= age_SD hgt_SD wgt_SD
min=age_min hgt_min wgt_min
max=age_max hgt_max wgt_max
median=age_median hgt_median wgt_median;
by sex;
run;

proc transpose data = dmg1 (DROP= _TYPE_ _FREQ_) out=dmg2;
by sex;
run;

data means2  (drop=_name_) ;
set dmg2;
name1=upcase(_name_);
if scan(name1, 2, "_") = 'N' then value = put(COL1, 7.0);
else if scan(name1, 2, "_") = 'MEAN' then value = put(COL1, 7.1);
else if scan(name1, 2, "_") = 'SD' then value = put(COL1, 7.2);
else if scan(name1, 2, "_") = 'MIN' then value = put(COL1, 7.0);
else if scan(name1, 2, "_") = 'MEDIAN' then value = put(COL1, 7.0);
else if scan(name1, 2, "_") = 'MAX' then value = put(COL1, 7.0);
run;

proc sort data=means2 out= means21;
by name1;
run;

proc transpose data= means21 out=ds4;
by name1;
id sex;
var value;
run;

data scan (drop= stat _name_);
length name $5 value1 $6;
set ds4;
name= scan(_name_ ,1,'_');
value1= right(scan(_name_ ,2,'_'));
run;

data ds3;
length SEX $10;
set scan;
if sex= 'F' then sex= 'female';
if sex= 'M' then sex= 'male';
if sex= 'T' then sex= 'total';
run;

proc sort data = ds3 out=sort;
by name  value1;
run;

ods rtf file='/home/u63347089/CODES/demo.rtf' style=minimal;
proc report data=DS4 headline headskip nowd style=[frame=hsides rules=groups  font_face="times new roman" font_size=8 pt] ;
column  name value1 female male total;
define name/'parameter' order  WIDTH=8;
define value1/'stat' display;
define  female/'female' display;
define male/'male' display;
define total/'total' display;
title 'Summary Report of Students';
footnote 'Generated by Vinit';
run;
ods rtf close;
```

---
> ✅ This markdown includes the SAS code for generating a demographic listing and a summary table from clinical data.
