#### Packages

    library(dplyr)

1 Data description
------------------

-   Repayment history for the previously disbursed credits in Home
    Credit related to the loans in our sample.
-   There is a) one row for every payment that was made plus b) one row
    each for missed payment.
-   One row is equivalent to one payment of one installment OR one
    installment corresponding to one payment of one previous Home Credit
    credit related to loans in our sample.

2 Reading the data
------------------

    installments_payments <- read.csv("../Data/installments_payments.csv")

    glimpse(installments_payments)

    ## Observations: 13,605,401
    ## Variables: 8
    ## $ SK_ID_PREV             <int> 1054186, 1330831, 2085231, 2452527, 271...
    ## $ SK_ID_CURR             <int> 161674, 151639, 193053, 199697, 167756,...
    ## $ NUM_INSTALMENT_VERSION <dbl> 1, 0, 2, 1, 1, 1, 4, 2, 0, 1, 1, 1, 0, ...
    ## $ NUM_INSTALMENT_NUMBER  <int> 6, 34, 1, 3, 2, 12, 11, 4, 14, 4, 3, 8,...
    ## $ DAYS_INSTALMENT        <dbl> -1180, -2156, -63, -2418, -1383, -1384,...
    ## $ DAYS_ENTRY_PAYMENT     <dbl> -1187, -2156, -63, -2426, -1366, -1417,...
    ## $ AMT_INSTALMENT         <dbl> 6948.360, 1716.525, 25425.000, 24350.13...
    ## $ AMT_PAYMENT            <dbl> 6948.360, 1716.525, 25425.000, 24350.13...

3 Data Warngling
----------------

Since there are several loans per ID (i.e. duplicated IDs) the data have
to be aggregated.

**SK\_ID\_CURR**

-   Building the ID column.

<!-- -->

    dat_inst <- installments_payments %>% select(SK_ID_CURR) %>% unique()

**NUM\_INSTALMENT\_NUMBER**

-   Calculating the normalized mean of the installment number.

<!-- -->

    inst_num_installment_number <- installments_payments %>% 
      group_by(SK_ID_CURR) %>%
      summarise(inst_num_installment_number = mean(NUM_INSTALMENT_NUMBER)) %>%
       mutate(inst_num_installment_number =
                (inst_num_installment_number-min(inst_num_installment_number))/
                (max(inst_num_installment_number)-min(inst_num_installment_number))) 


    dat_inst <- merge(dat_inst, inst_num_installment_number, by = "SK_ID_CURR")

**AMT\_PAYMENT/AMT\_INSTALMENT**

-   Calculating the relationship between prescribed and actual amount.

<!-- -->

    inst_diff_amt_installment_amt_payment <- installments_payments %>% 
      mutate(x = (AMT_PAYMENT/AMT_INSTALMENT)) %>%
      select(SK_ID_CURR,  x) %>% 
      group_by(SK_ID_CURR) %>% 
      summarise(inst_diff_amt_installment_amt_payment = mean(x, na.rm = T)) 

    dat_inst <- merge(dat_inst, inst_diff_amt_installment_amt_payment, by = "SK_ID_CURR")

**DAYS\_ENTRY\_PAYMENT/DAYS\_INSTALMENT**

-   Calculating the relationship between prescribed and actual date.

<!-- -->

    inst_diff_days_installment_days_entry_payment <- installments_payments %>% 
      mutate(x = (DAYS_ENTRY_PAYMENT/DAYS_INSTALMENT)) %>%
      select(SK_ID_CURR,  x) %>%
      group_by(SK_ID_CURR) %>%
      summarise(inst_diff_days_installment_days_entry_payment = mean(x, na.rm = T)) 

    dat_inst <- merge(dat_inst, inst_diff_days_installment_days_entry_payment, by = "SK_ID_CURR")

4 Saving the data
-----------------

    save(dat_inst, file="data_inst.Rda")
