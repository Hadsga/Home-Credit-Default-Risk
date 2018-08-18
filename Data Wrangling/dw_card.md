#### Packages

    library(dplyr)
    library(tidyr)

1 Data description
------------------

-   Monthly balance snapshots of previous credit cards that the
    applicant has with Home Credit.
-   This table has one row for each month of history of every previous
    credit in Home Credit (consumer credit and cash loans) related to
    loans in our sample.

2 Reading the data
------------------

    credit_card_balance <- read.csv("../Data/credit_card_balance.csv")

    glimpse(credit_card_balance)

    ## Observations: 3,840,312
    ## Variables: 23
    ## $ SK_ID_PREV                 <int> 2562384, 2582071, 1740877, 1389973,...
    ## $ SK_ID_CURR                 <int> 378907, 363914, 371185, 337855, 126...
    ## $ MONTHS_BALANCE             <int> -6, -1, -7, -4, -1, -7, -6, -7, -4,...
    ## $ AMT_BALANCE                <dbl> 56.970, 63975.555, 31815.225, 23657...
    ## $ AMT_CREDIT_LIMIT_ACTUAL    <int> 135000, 45000, 450000, 225000, 4500...
    ## $ AMT_DRAWINGS_ATM_CURRENT   <dbl> 0, 2250, 0, 2250, 0, 0, 67500, 4500...
    ## $ AMT_DRAWINGS_CURRENT       <dbl> 877.50, 2250.00, 0.00, 2250.00, 115...
    ## $ AMT_DRAWINGS_OTHER_CURRENT <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,...
    ## $ AMT_DRAWINGS_POS_CURRENT   <dbl> 877.50, 0.00, 0.00, 0.00, 11547.00,...
    ## $ AMT_INST_MIN_REGULARITY    <dbl> 1700.325, 2250.000, 2250.000, 11795...
    ## $ AMT_PAYMENT_CURRENT        <dbl> 1800.000, 2250.000, 2250.000, 11925...
    ## $ AMT_PAYMENT_TOTAL_CURRENT  <dbl> 1800.000, 2250.000, 2250.000, 11925...
    ## $ AMT_RECEIVABLE_PRINCIPAL   <dbl> 0.00, 60175.08, 26926.42, 224949.29...
    ## $ AMT_RECIVABLE              <dbl> 0.00, 64875.56, 31460.08, 233048.97...
    ## $ AMT_TOTAL_RECEIVABLE       <dbl> 0.00, 64875.56, 31460.08, 233048.97...
    ## $ CNT_DRAWINGS_ATM_CURRENT   <dbl> 0, 1, 0, 1, 0, 0, 1, 1, 3, 3, 5, 2,...
    ## $ CNT_DRAWINGS_CURRENT       <int> 1, 1, 0, 1, 1, 0, 1, 1, 8, 9, 5, 2,...
    ## $ CNT_DRAWINGS_OTHER_CURRENT <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,...
    ## $ CNT_DRAWINGS_POS_CURRENT   <dbl> 1, 0, 0, 0, 1, 0, 0, 0, 5, 6, 0, 0,...
    ## $ CNT_INSTALMENT_MATURE_CUM  <dbl> 35, 69, 30, 10, 101, 2, 6, 51, 3, 3...
    ## $ NAME_CONTRACT_STATUS       <fct> Active, Active, Active, Active, Act...
    ## $ SK_DPD                     <int> 0, 0, 0, 0, 0, 7, 0, 0, 0, 0, 0, 0,...
    ## $ SK_DPD_DEF                 <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,...

3 Data Wrangling
----------------

**SK\_ID\_CURR**

-   Building the ID column.

<!-- -->

    dat_card <- credit_card_balance %>% select(SK_ID_CURR) %>% unique()

**MONTHS\_BALANCE**

-   Calculating the mean of the monthly credit card balance regarding
    the credit application date per ID.

<!-- -->

    card_months_balance <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_months_balance = mean(MONTHS_BALANCE, na.rm = T))

    dat_card <- merge(dat_card, card_months_balance, by = "SK_ID_CURR")

**AMT\_BALANCE**

-   Calculating the mean of the monthly credit card balance per ID.

<!-- -->

    card_amt_balance <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_amt_balance = mean(AMT_BALANCE, na.rm = T))

    dat_card <- merge(dat_card, card_amt_balance, by = "SK_ID_CURR")

**AMT\_CREDIT\_LIMIT\_ACTUAL**

-   Calculating the mean of the previous credit card limits per ID.

<!-- -->

    card_amt_credit_limit_actual <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_amt_credit_limit_actual = mean(AMT_CREDIT_LIMIT_ACTUAL, na.rm = T))

    dat_card <- merge(dat_card, card_amt_credit_limit_actual, by = "SK_ID_CURR")

**AMT\_DRAWINGS\_ATM\_CURRENT**

-   Calculating the mean of the previous credit card limits per ID.

<!-- -->

    card_amt_drawings_atm_current <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_amt_drawings_atm_current = mean(AMT_DRAWINGS_ATM_CURRENT, na.rm = T))

    dat_card <- merge(dat_card, card_amt_drawings_atm_current, by = "SK_ID_CURR")

**AMT\_DRAWINGS\_CURRENT**

-   Calculating the mean of the AMT drawings per ID.

<!-- -->

    card_amt_drawings_current <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_amt_drawings_current = mean(AMT_DRAWINGS_CURRENT, na.rm = T))

    dat_card <- merge(dat_card, card_amt_drawings_current , by = "SK_ID_CURR")

**AMT\_DRAWINGS\_OTHER\_CURRENT**

-   Calculating the mean of other drawings per ID.

<!-- -->

    card_amt_drawings_other_current <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_amt_drawings_other_current = mean(AMT_DRAWINGS_OTHER_CURRENT, na.rm = T))

    dat_card <- merge(dat_card, card_amt_drawings_other_current, by = "SK_ID_CURR")

**AMT\_DRAWINGS\_POS\_CURRENT**

-   Calculating the mean of POS drawings per ID.

<!-- -->

    card_amt_drawings_pos_current <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_amt_drawings_pos_current = mean(AMT_DRAWINGS_POS_CURRENT, na.rm = T))

    dat_card <- merge(dat_card, card_amt_drawings_pos_current, by = "SK_ID_CURR")

**AMT\_INST\_MIN\_REGULARITY**

-   Calculating the mean of min minimal installment per ID.

<!-- -->

    card_amt_inst_min_regularity <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_amt_inst_min_regularity = mean(AMT_INST_MIN_REGULARITY, na.rm = T))

    dat_card <- merge(dat_card, card_amt_inst_min_regularity, by = "SK_ID_CURR")

**AMT\_PAYMENT\_CURRENT**

-   Calculating the mean of the payments for previous credits per ID.

<!-- -->

    card_amt_payment_current <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_amt_payment_current = mean(AMT_PAYMENT_CURRENT, na.rm = T))

    dat_card <- merge(dat_card, card_amt_payment_current, by = "SK_ID_CURR")

**AMT\_PAYMENT\_TOTAL\_CURRENT**

-   Calculating the mean of the total payments for previous credits per
    ID.

<!-- -->

    card_amt_payment_total_current <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_amt_payment_total_current = mean(AMT_PAYMENT_TOTAL_CURRENT, na.rm = T))

    dat_card <- merge(dat_card, card_amt_payment_total_current, by = "SK_ID_CURR")

**AMT\_RECEIVABLE\_PRINCIPAL**

-   Calculating the mean regarding the total amount receivable for
    principal on the previous credit per ID.

<!-- -->

    card_amt_receivable_principal <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_amt_receivable_principal = sum(AMT_RECEIVABLE_PRINCIPAL, na.rm = T))

    dat_card <- merge(dat_card, card_amt_receivable_principal, by = "SK_ID_CURR")

**AMT\_RECIVABLE**

-   Calculating the mean regarding the total amount on the previous
    credit per ID.

<!-- -->

    card_amt_recivable <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_amt_recivable = sum(AMT_RECIVABLE, na.rm = T))

    dat_card <- merge(dat_card, card_amt_recivable, by = "SK_ID_CURR")

**AMT\_TOTAL\_RECEIVABLE**

-   Calculating the mean regarding the total amount receivable for
    principal on the previous credit per ID.

<!-- -->

    card_amt_total_receivable <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_amt_total_receivable = sum(AMT_TOTAL_RECEIVABLE, na.rm = T))

    dat_card <- merge(dat_card, card_amt_total_receivable, by = "SK_ID_CURR")

**CNT\_DRAWINGS\_ATM\_CURRENT**

-   Calculating the mean for AMT drawings per ID.

<!-- -->

    card_cnt_drawings_current <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_cnt_drawings_current = mean(CNT_DRAWINGS_ATM_CURRENT, na.rm = T))

    dat_card <- merge(dat_card, card_cnt_drawings_current, by = "SK_ID_CURR")

**CNT\_DRAWINGS\_CURRENT**

-   Calculating the mean of general drawings per ID.

<!-- -->

    card_cnt_drawings_amt_current <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_cnt_drawings_amt_current = mean(CNT_DRAWINGS_CURRENT, na.rm = T))

    dat_card <- merge(dat_card, card_cnt_drawings_amt_current, by = "SK_ID_CURR")

**CNT\_DRAWINGS\_OTHER\_CURRENT**

-   Calculating the mean for other drawings per ID.

<!-- -->

    card_cnt_drawings_other_current <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_cnt_drawings_other_current = mean(CNT_DRAWINGS_OTHER_CURRENT, na.rm = T))

    dat_card <- merge(dat_card, card_cnt_drawings_other_current, by = "SK_ID_CURR")

**CNT\_DRAWINGS\_POS\_CURRENT**

-   Calculating the mean of drawings for goods per ID.

<!-- -->

    card_cnt_drawings_pos_current <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_cnt_drawings_pos_current = mean(CNT_DRAWINGS_POS_CURRENT, na.rm = T))

    dat_card <- merge(dat_card, card_cnt_drawings_pos_current, by = "SK_ID_CURR")

**CNT\_INSTALMENT\_MATURE\_CUM**

-   Calculating the mean of days past due per ID.

<!-- -->

    card_cnt_instalment_mature_cum <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_cnt_instalment_mature_cum = mean(CNT_INSTALMENT_MATURE_CUM, na.rm = T))

    dat_card <- merge(dat_card, card_cnt_instalment_mature_cum, by = "SK_ID_CURR")

**NAME\_CONTRACT\_STATUS**

-   Calculating relative frequency of the contract status.

<!-- -->

    card_name_contract_status <- credit_card_balance %>%
      select(SK_ID_CURR, NAME_CONTRACT_STATUS) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(NAME_CONTRACT_STATUS, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean))

    dat_card <- merge(dat_card, card_name_contract_status, by = "SK_ID_CURR")

**SK\_DPD**

-   Calculating the mean of days past due per ID.

<!-- -->

    card_sk_dpd <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_sk_dpd = mean(SK_DPD, na.rm = T))

    dat_card <- merge(dat_card, card_sk_dpd, by = "SK_ID_CURR")

**SK\_DPD\_DEF**

-   Calculating the mean of days past due with tolerance per ID.

<!-- -->

    card_sk_dpd_def <- credit_card_balance %>% group_by(SK_ID_CURR) %>%
      summarise(card_sk_dpd_def = mean(SK_DPD_DEF, na.rm = T))

    dat_card <- merge(dat_card, card_sk_dpd_def, by = "SK_ID_CURR")

-   Making valid column names.

<!-- -->

    colnames(dat_card) <- make.names(colnames(dat_card))

4 Saving the data
-----------------

    save(dat_card, file="data_card.Rda")
