#### Packages

    library(dplyr)
    library(tidyr)

1 Data description
------------------

-   All client's previous credits provided by other financial
    institutions that were reported to Credit Bureau (for clients who
    have a loan in our sample).
-   For every loan in our sample, there are as many rows as number of
    credits the client had in Credit Bureau before the application date.

2 Reading the data
------------------

    bureau <- read.csv("../Data/bureau.csv")

    glimpse(bureau)

    ## Observations: 1,716,428
    ## Variables: 17
    ## $ SK_ID_CURR             <int> 215354, 215354, 215354, 215354, 215354,...
    ## $ SK_ID_BUREAU           <int> 5714462, 5714463, 5714464, 5714465, 571...
    ## $ CREDIT_ACTIVE          <fct> Closed, Active, Active, Active, Active,...
    ## $ CREDIT_CURRENCY        <fct> currency 1, currency 1, currency 1, cur...
    ## $ DAYS_CREDIT            <int> -497, -208, -203, -203, -629, -273, -43...
    ## $ CREDIT_DAY_OVERDUE     <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...
    ## $ DAYS_CREDIT_ENDDATE    <dbl> -153, 1075, 528, NA, 1197, 27460, 79, -...
    ## $ DAYS_ENDDATE_FACT      <dbl> -153, NA, NA, NA, NA, NA, NA, -1710, -8...
    ## $ AMT_CREDIT_MAX_OVERDUE <dbl> NA, NA, NA, NA, 77674.5, 0.0, 0.0, 1498...
    ## $ CNT_CREDIT_PROLONG     <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...
    ## $ AMT_CREDIT_SUM         <dbl> 91323.00, 225000.00, 464323.50, 90000.0...
    ## $ AMT_CREDIT_SUM_DEBT    <dbl> 0.000, 171342.000, NA, NA, NA, 71017.38...
    ## $ AMT_CREDIT_SUM_LIMIT   <dbl> NA, NA, NA, NA, NA, 108982.620, 0.000, ...
    ## $ AMT_CREDIT_SUM_OVERDUE <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...
    ## $ CREDIT_TYPE            <fct> Consumer credit, Credit card, Consumer ...
    ## $ DAYS_CREDIT_UPDATE     <int> -131, -20, -16, -16, -21, -31, -22, -17...
    ## $ AMT_ANNUITY            <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,...

3 Data Warngling
----------------

-   Because there are several loans per ID (i.e. duplicated IDs) the
    data have to be aggregated.

**SK\_ID\_CURR**

-   Creating the ID column.

<!-- -->

    dat_bureau <- bureau %>% select(SK_ID_CURR) %>% unique()

**CREDIT\_ACTIVE**

-   Extracting the amount of current (active) credits per ID.

<!-- -->

    bureau_credit_active <- bureau %>% 
      mutate(CREDIT_ACTIVE = ifelse(CREDIT_ACTIVE == "Active", 1, 0)) %>%
      group_by(SK_ID_CURR) %>%
      summarise(bureau_credit_active = sum(CREDIT_ACTIVE)) 

    dat_bureau <- merge(dat_bureau, bureau_credit_active, by = "SK_ID_CURR")

**CREDIT\_CURRENCY**

-   Calculating the relative frequency of credit currency per ID.

<!-- -->

    bureau_credit_currency <- bureau %>%
      select(SK_ID_CURR, CREDIT_CURRENCY) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(CREDIT_CURRENCY, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean))

    dat_bureau <- merge(dat_bureau, bureau_credit_currency, by = "SK_ID_CURR")

**DAYS\_CREDIT**

-   Calculating the mean of past credit applications (in days) per ID.

<!-- -->

    bureau_days_credit <- bureau %>% 
      group_by(SK_ID_CURR) %>% 
      summarise(bureau_days_credit = mean(DAYS_CREDIT, na.rm = T))

    dat_bureau <- merge(dat_bureau, bureau_days_credit, by = "SK_ID_CURR")

**CREDIT\_DAY\_OVERDUE**

-   Calculating the mean of days credit overdue per ID.

<!-- -->

    bureau_credit_day_overdue <- bureau %>% group_by(SK_ID_CURR) %>% 
      summarise(bureau_credit_day_overdue = mean(CREDIT_DAY_OVERDUE, na.rm = T))

    dat_bureau <- merge(dat_bureau, bureau_credit_day_overdue, by = "SK_ID_CURR")

**DAYS\_CREDIT\_ENDDATE**

-   Calculating the sum regarding the remaining time of active credits
    per ID.

<!-- -->

    bureau_day_credit_endate <- bureau %>%
      group_by(SK_ID_CURR) %>%
      summarise(bureau_day_credit_endate = sum(DAYS_CREDIT_ENDDATE[DAYS_CREDIT_ENDDATE >= 0])) 

    dat_bureau <- merge(dat_bureau, bureau_day_credit_endate, by = "SK_ID_CURR")

**AMT\_CREDIT\_MAX\_OVERDUE**

-   Calculating the mean of max overdue per ID.

<!-- -->

    bureau_amt_credit_max_overdue <- bureau %>% 
      group_by(SK_ID_CURR) %>% 
      summarise(bureau_amt_credit_max_overdue = mean(AMT_CREDIT_MAX_OVERDUE, na.rm = T))

    dat_bureau <- merge(dat_bureau, bureau_amt_credit_max_overdue, by = "SK_ID_CURR")

**CNT\_CREDIT\_PROLONG**

-   Calculating the mean of times the credits prolonged per ID.

<!-- -->

    bureau_cnt_credit_prolong <- bureau %>% group_by(SK_ID_CURR) %>% 
      summarise(bureau_cnt_credit_prolong = mean(CNT_CREDIT_PROLONG, na.rm = T))

    dat_bureau <- merge(dat_bureau, bureau_cnt_credit_prolong, by = "SK_ID_CURR")

**AMT\_CREDIT\_SUM**

-   Calculating the mean of the credit sums per ID.

<!-- -->

    bureau_amt_credit_sum <- bureau %>% group_by(SK_ID_CURR) %>% 
      summarise(bureau_amt_credit_sum = mean(AMT_CREDIT_SUM, na.rm = T))

    dat_bureau <- merge(dat_bureau, bureau_amt_credit_sum, by = "SK_ID_CURR")

**AMT\_CREDIT\_SUM\_DEBT**

-   Calculating the sum of current credit dept´s per ID.

<!-- -->

    bureau_amt_credit_sum_dept <- bureau %>% group_by(SK_ID_CURR) %>% 
      summarise(bureau_amt_credit_sum_dept = sum(AMT_CREDIT_SUM_DEBT))

    dat_bureau <- merge(dat_bureau, bureau_amt_credit_sum_dept, by = "SK_ID_CURR")

**AMT\_CREDIT\_SUM\_LIMIT**

-   Extracting the current credit card limit.

<!-- -->

    bureau_amt_credit_sum_limit <- bureau %>% 
      group_by(SK_ID_CURR) %>% 
      summarise(bureau_amt_credit_sum_limit = max(AMT_CREDIT_SUM_LIMIT))

    dat_bureau <- merge(dat_bureau, bureau_amt_credit_sum_limit, by = "SK_ID_CURR")

**AMT\_CREDIT\_SUM\_OVERDUE**

-   Calculating the sum of credit overdue per dept per ID.

<!-- -->

    bureau_amt_credit_sum_overdue <- bureau %>% group_by(SK_ID_CURR) %>% 
      summarise(bureau_amt_credit_sum_overdue = sum(AMT_CREDIT_SUM_OVERDUE))

    dat_bureau <- merge(dat_bureau, bureau_amt_credit_sum_overdue, by = "SK_ID_CURR")

**CREDIT\_TYPE**

-   Calculating the percentage for credit types per ID.

<!-- -->

    bureau_credit_type <- bureau %>%
      select(SK_ID_CURR, CREDIT_TYPE) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(CREDIT_TYPE, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean))

    dat_bureau <- merge(dat_bureau, bureau_credit_type, by = "SK_ID_CURR")

**DAYS\_CREDIT\_UPDATE**

-   Calculating the mean of days credit update.

<!-- -->

    bureau_days_credit_update <- bureau %>% 
      group_by(SK_ID_CURR) %>% 
      summarise(bureau_days_credit_update = mean(DAYS_CREDIT_UPDATE, na.rm = T))


    dat_bureau <- merge(dat_bureau, bureau_days_credit_update, by = "SK_ID_CURR")

**AMT\_ANNUITY**

-   Calculating the sum of annuity per ID.

<!-- -->

    bureau_amt_annuity <- bureau %>% 
      group_by(SK_ID_CURR) %>%
      summarise(bureau_amt_annuity = sum(AMT_ANNUITY, na.rm = T))

    dat_bureau <- merge(dat_bureau, bureau_amt_annuity, by = "SK_ID_CURR")

-   Making valid column names.

<!-- -->

    colnames(dat_bureau) <- make.names(colnames(dat_bureau))

4 Saving the Data
-----------------

    save(dat_bureau, file="data_bureau.Rda")
