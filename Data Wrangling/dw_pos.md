#### Packages

    library(dplyr)
    library(tidyr)

### 1 Data description

-   Monthly balance snapshots of previous POS (point of sales) and cash
    loans that the applicant had with Home Credit.
-   This table has one row for each month of history of every previous
    credit in Home Credit (consumer credit and cash loans) related to
    loans in our sample.

### 2 Reading the data

    POS_CASH_balance <- read.csv("../Data/POS_CASH_balance.csv")

    glimpse(POS_CASH_balance)

    ## Observations: 10,001,358
    ## Variables: 8
    ## $ SK_ID_PREV            <int> 1803195, 1715348, 1784872, 1903291, 2341...
    ## $ SK_ID_CURR            <int> 182943, 367990, 397406, 269225, 334279, ...
    ## $ MONTHS_BALANCE        <int> -31, -33, -32, -35, -35, -32, -38, -35, ...
    ## $ CNT_INSTALMENT        <dbl> 48, 36, 12, 48, 36, 12, 48, 36, 12, 24, ...
    ## $ CNT_INSTALMENT_FUTURE <dbl> 45, 35, 9, 42, 35, 12, 43, 36, 12, 16, 1...
    ## $ NAME_CONTRACT_STATUS  <fct> Active, Active, Active, Active, Active, ...
    ## $ SK_DPD                <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...
    ## $ SK_DPD_DEF            <int> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...

### 3 Data Wrangling

-   Because there are several loans per ID (i.e. duplicated IDs) the
    data have to be aggregated.

**SK\_ID\_CURR**

-   Creating the ID column.

<!-- -->

    dat_pos <- POS_CASH_balance %>% select(SK_ID_CURR) %>% unique()

**MONTHS\_BALANCE**

-   Calculating the mean of the monthly balance per ID.

<!-- -->

    pos_month_balance <- POS_CASH_balance %>% group_by(SK_ID_CURR) %>%
      summarise(pos_month_balance = mean(MONTHS_BALANCE, na.rm = T))

    dat_pos <- merge(dat_pos, pos_month_balance, by = "SK_ID_CURR")

**CNT\_INSTALMENT**

-   Calculating the mean of the term of the previous credits per ID.

<!-- -->

    pos_cnt_instalment <- POS_CASH_balance %>% group_by(SK_ID_CURR) %>%
      summarise(pos_cnt_instalment = mean(CNT_INSTALMENT, na.rm = T))

    dat_pos <- merge(dat_pos, pos_cnt_instalment, by = "SK_ID_CURR")

**CNT\_INSTALMENT\_FUTURE**

-   Calculating the sum of the remaining installments per ID.

<!-- -->

    pos_cnt_instalment_future <- POS_CASH_balance %>% group_by(SK_ID_CURR) %>%
      summarise(pos_cnt_instalment_future = sum(CNT_INSTALMENT_FUTURE, na.rm = T))

    dat_pos <- merge(dat_pos, pos_cnt_instalment_future, by = "SK_ID_CURR")

**NAME\_CONTRACT\_STATUS**

-   Calculating the relative frequency of the contract status per ID.

<!-- -->

    pos_name_contract_status <- POS_CASH_balance %>%
      select(SK_ID_CURR, NAME_CONTRACT_STATUS) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(NAME_CONTRACT_STATUS, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) %>%
      rename(xna.name.contract.status = XNA)

    dat_pos <- merge(dat_pos, pos_name_contract_status, by = "SK_ID_CURR")

**SK\_DPD**

-   Calculating the mean of days past due per ID.

<!-- -->

    pos_sk_dpd <- POS_CASH_balance %>% group_by(SK_ID_CURR) %>%
      summarise(pos_sk_dpd = mean(SK_DPD, na.rm = T))

    dat_pos <- merge(dat_pos,pos_sk_dpd, by = "SK_ID_CURR")

**SK\_DPD\_DEF**

-   Calculating the mean of days past due for low credits per ID.

<!-- -->

    pos_sk_dpd_def <- POS_CASH_balance %>% group_by(SK_ID_CURR) %>%
      summarise(pos_sk_dpd_def = mean(SK_DPD_DEF, na.rm = T))

    dat_pos <- merge(dat_pos, pos_sk_dpd_def, by = "SK_ID_CURR")

-   Making valid column names.

<!-- -->

    colnames(dat_pos) <- make.names(colnames(dat_pos))

### 4 Saving the data

    save(dat_pos, file="data_pos.Rda")
