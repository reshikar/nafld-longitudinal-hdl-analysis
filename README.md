/* Longitudinal Analysis of HDL Cholesterol Levels in NAFLD */

/* Library */
libname cohort "/home/u63580297/Cohortstudies";

/* Import nafld1 */
proc import datafile="/home/u63580297/Cohortstudies/FinalProject/nafld1.csv"
    out=nafld1
    dbms=csv
    replace;
    guessingrows=max;
run;

/* Import nafld2 */
proc import datafile="/home/u63580297/Cohortstudies/FinalProject/nafld2.csv"
    out=nafld2
    dbms=csv
    replace;
    guessingrows=max;
run;

/* Prepare baseline data */
data base;
    set nafld1;
    if bmi ne "NA" then bmi_num = input(bmi, best12.);
    keep id age male bmi_num;
    if bmi_num ne .;
run;

/* Randomly sample 10% of subjects */
proc surveyselect data=base
    out=base_sample
    method=srs
    samprate=0.10
    seed=123;
run;

/* Keep only HDL measurements */
data hdl;
    set nafld2;
    where test = 'hdl';
    hdl = value;
    keep id days hdl;
    if hdl ne . and days ne .;
run;

/* Sort before merge */
proc sort data=base_sample;
    by id;
run;

proc sort data=hdl;
    by id;
run;

/* Merge baseline and longitudinal HDL data */
data merged;
    merge base_sample(in=a) hdl(in=b);
    by id;
    if a and b;
run;

/* Count HDL measurements per subject */
proc sql;
    create table counts as
    select id,
           count(*) as n_timepoints
    from merged
    group by id;
quit;

/* Keep subjects with at least 2 HDL measurements */
proc sql;
    create table final_hdl as
    select a.*, b.n_timepoints
    from merged as a
    inner join counts as b
    on a.id = b.id
    where b.n_timepoints >= 2;
quit;

/* Final sample size */
proc sql;
    select count(distinct id) as subjects,
           count(*) as observations
    from final_hdl;
quit;

/* Descriptive statistics */
proc means data=final_hdl mean std min max;
    var hdl age bmi_num days;
run;

proc freq data=final_hdl;
    tables male;
run;

/* Sort longitudinal data */
proc sort data=final_hdl;
    by id days;
run;

/* Create follow-up time groups */
data final_hdl_graph;
    set final_hdl;
    followup_year = days / 365.25;
    year_group = floor(followup_year);
run;

/* Figure 1: Mean HDL over follow-up time */
proc means data=final_hdl_graph noprint;
    class year_group;
    var hdl;
    output out=mean_hdl_by_year
        mean=mean_hdl
        stderr=se_hdl;
run;

data mean_hdl_by_year;
    set mean_hdl_by_year;
    where _TYPE_ = 1;
run;

proc sgplot data=mean_hdl_by_year;
    series x=year_group y=mean_hdl / markers;
    xaxis label="Follow-up Time (Years)";
    yaxis label="Mean HDL Cholesterol (mg/dL)";
    title "Figure 1. Mean HDL Cholesterol Levels Over Follow-up Time";
run;

/* Figure 2: Individual HDL trajectories */
proc surveyselect data=final_hdl
    out=sample_subjects
    method=srs
    sampsize=25
    seed=123;
    id id;
run;

proc sort data=sample_subjects nodupkey;
    by id;
run;

proc sql;
    create table spaghetti_data as
    select a.*
    from final_hdl as a
    inner join sample_subjects as b
    on a.id = b.id;
quit;

proc sgplot data=spaghetti_data;
    series x=days y=hdl / group=id;
    xaxis label="Follow-up Time (Days)";
    yaxis label="HDL Cholesterol (mg/dL)";
    title "Figure 2. Individual HDL Trajectories";
run;

/* Figure 3: Histogram of HDL levels */
proc sgplot data=final_hdl;
    histogram hdl;
    density hdl;
    xaxis label="HDL Cholesterol (mg/dL)";
    yaxis label="Frequency";
    title "Figure 3. Distribution of HDL Cholesterol Levels";
run;

/* Linear Mixed Model: Compound Symmetry */
proc mixed data=final_hdl method=reml;
    class id male;
    model hdl = days age male bmi_num / solution cl;
    repeated / subject=id type=cs;
    ods output FitStatistics=Fit_CS;
    title "LMM: Compound Symmetry";
run;

/* Linear Mixed Model: AR(1) */
proc mixed data=final_hdl method=reml;
    class id male;
    model hdl = days age male bmi_num / solution cl;
    repeated / subject=id type=ar(1);
    ods output FitStatistics=Fit_AR1;
    title "LMM: AR(1)";
run;

/* Linear Mixed Model: Unstructured */
proc mixed data=final_hdl method=reml;
    class id male;
    model hdl = days age male bmi_num / solution cl;
    repeated / subject=id type=un;
    ods output FitStatistics=Fit_UN;
    title "LMM: Unstructured";
run;

/* Final Linear Mixed Model using AR(1) */
proc mixed data=final_hdl method=reml;
    class id male;
    model hdl = days age male bmi_num / solution cl;
    repeated / subject=id type=ar(1);

    ods output
        SolutionF = Final_LMM_Estimates
        Type3 = Final_LMM_Type3
        FitStatistics = Final_LMM_Fit;

    title "Final Linear Mixed Model with AR(1) Covariance";
run;

/* GEE model with exchangeable correlation */
proc genmod data=final_hdl;
    class id male;

    model hdl = days age male bmi_num /
        dist=normal
        link=identity
        type3;

    repeated subject=id / type=exch corrw;

    ods output
        GEEEmpPEst = GEE_Estimates
        Type3 = GEE_Type3;

    title "GEE Model with Exchangeable Working Correlation";
run;
