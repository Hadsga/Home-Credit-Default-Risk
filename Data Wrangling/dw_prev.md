#### Packages

    library(dplyr)
    library(tidyr)

1 Data description
------------------

-   All previous applications for Home Credit loans of clients who have
    loans in our sample.
-   There is one row for each previous application related to loans in
    our data sample.

2 Reading the data
------------------

    previous_application <- read.csv("../Data/previous_application.csv")

    glimpse(previous_application)

    ## Observations: 1,670,214
    ## Variables: 37
    ## $ SK_ID_PREV                  <int> 2030495, 2802425, 2523466, 2819243...
    ## $ SK_ID_CURR                  <int> 271877, 108129, 122040, 176158, 20...
    ## $ NAME_CONTRACT_TYPE          <fct> Consumer loans, Cash loans, Cash l...
    ## $ AMT_ANNUITY                 <dbl> 1730.43, 25188.62, 15060.74, 47041...
    ## $ AMT_APPLICATION             <dbl> 17145.0, 607500.0, 112500.0, 45000...
    ## $ AMT_CREDIT                  <dbl> 17145.0, 679671.0, 136444.5, 47079...
    ## $ AMT_DOWN_PAYMENT            <dbl> 0.0, NA, NA, NA, NA, NA, NA, NA, N...
    ## $ AMT_GOODS_PRICE             <dbl> 17145.0, 607500.0, 112500.0, 45000...
    ## $ WEEKDAY_APPR_PROCESS_START  <fct> SATURDAY, THURSDAY, TUESDAY, MONDA...
    ## $ HOUR_APPR_PROCESS_START     <int> 15, 11, 11, 7, 9, 8, 11, 7, 15, 15...
    ## $ FLAG_LAST_APPL_PER_CONTRACT <fct> Y, Y, Y, Y, Y, Y, Y, Y, Y, Y, Y, Y...
    ## $ NFLAG_LAST_APPL_IN_DAY      <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...
    ## $ RATE_DOWN_PAYMENT           <dbl> 0.00000000, NA, NA, NA, NA, NA, NA...
    ## $ RATE_INTEREST_PRIMARY       <dbl> 0.1828318, NA, NA, NA, NA, NA, NA,...
    ## $ RATE_INTEREST_PRIVILEGED    <dbl> 0.8673362, NA, NA, NA, NA, NA, NA,...
    ## $ NAME_CASH_LOAN_PURPOSE      <fct> XAP, XNA, XNA, XNA, Repairs, Every...
    ## $ NAME_CONTRACT_STATUS        <fct> Approved, Approved, Approved, Appr...
    ## $ DAYS_DECISION               <int> -73, -164, -301, -512, -781, -684,...
    ## $ NAME_PAYMENT_TYPE           <fct> Cash through the bank, XNA, Cash t...
    ## $ CODE_REJECT_REASON          <fct> XAP, XAP, XAP, XAP, HC, XAP, XAP, ...
    ## $ NAME_TYPE_SUITE             <fct> , Unaccompanied, Spouse, partner, ...
    ## $ NAME_CLIENT_TYPE            <fct> Repeater, Repeater, Repeater, Repe...
    ## $ NAME_GOODS_CATEGORY         <fct> Mobile, XNA, XNA, XNA, XNA, XNA, X...
    ## $ NAME_PORTFOLIO              <fct> POS, Cash, Cash, Cash, Cash, Cash,...
    ## $ NAME_PRODUCT_TYPE           <fct> XNA, x-sell, x-sell, x-sell, walk-...
    ## $ CHANNEL_TYPE                <fct> Country-wide, Contact center, Cred...
    ## $ SELLERPLACE_AREA            <int> 35, -1, -1, -1, -1, -1, -1, -1, -1...
    ## $ NAME_SELLER_INDUSTRY        <fct> Connectivity, XNA, XNA, XNA, XNA, ...
    ## $ CNT_PAYMENT                 <dbl> 12, 36, 12, 12, 24, 18, NA, NA, NA...
    ## $ NAME_YIELD_GROUP            <fct> middle, low_action, high, middle, ...
    ## $ PRODUCT_COMBINATION         <fct> POS mobile with interest, Cash X-S...
    ## $ DAYS_FIRST_DRAWING          <dbl> 365243, 365243, 365243, 365243, NA...
    ## $ DAYS_FIRST_DUE              <dbl> -42, -134, -271, -482, NA, -654, N...
    ## $ DAYS_LAST_DUE_1ST_VERSION   <dbl> 300, 916, 59, -152, NA, -144, NA, ...
    ## $ DAYS_LAST_DUE               <dbl> -42, 365243, 365243, -182, NA, -14...
    ## $ DAYS_TERMINATION            <dbl> -37, 365243, 365243, -177, NA, -13...
    ## $ NFLAG_INSURED_ON_APPROVAL   <dbl> 0, 1, 1, 1, NA, 1, NA, NA, NA, NA,...

3 Data warngling
----------------

Since there are several loans per ID (i.e. duplicated IDs) the data have
to be aggregated.

**SK\_ID\_CURR**

-   Creating the ID column.

<!-- -->

    dat_prev <- previous_application %>% select(SK_ID_CURR) %>% unique()

**NAME\_CONTRACT\_TYPE**

-   Calculating the relative frequency of contract types per ID.

<!-- -->

    prev_name_contract_type <- previous_application %>%
      select(SK_ID_CURR, NAME_CONTRACT_TYPE) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(NAME_CONTRACT_TYPE, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) %>%
      rename(xna.name.contract.type = XNA)


    dat_prev <- merge(dat_prev, prev_name_contract_type, by = "SK_ID_CURR")

**AMT\_ANNUITY**

-   Calculating the mean of the previous annuity per ID.

<!-- -->

    prev_amt_annuity <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_amt_annuity = mean(AMT_ANNUITY, na.rm = T))

    dat_prev <- merge(dat_prev, prev_amt_annuity, by = "SK_ID_CURR")

**AMT\_APPLICATION**

-   Calculating the mean of the requested credit amount per ID.

<!-- -->

    prev_amt_application <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_amt_application = mean(AMT_APPLICATION, na.rm = T))

    dat_prev <- merge(dat_prev, prev_amt_application, by = "SK_ID_CURR")

**AMT\_CREDIT**

-   Calculating the mean of the granted/final credit amount per ID.

<!-- -->

    prev_amt_credit <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_amt_credit = mean(AMT_CREDIT, na.rm = T))

    dat_prev <- merge(dat_prev, prev_amt_credit, by = "SK_ID_CURR")

**AMT\_DOWN\_PAYMENT**

-   Calculating the mean of the down payment amount per ID.

<!-- -->

    prev_amt_down_payment <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_amt_down_payment = mean(AMT_DOWN_PAYMENT, na.rm = T))

    dat_prev <- merge(dat_prev, prev_amt_down_payment, by = "SK_ID_CURR")

**AMT\_GOODS\_PRICE**

-   Calculating the mean of the goods price per ID.

<!-- -->

    prev_amt_goods_price <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_amt_goods_price = mean(AMT_GOODS_PRICE, na.rm = T))

    dat_prev <- merge(dat_prev, prev_amt_goods_price, by = "SK_ID_CURR")

**RATE\_DOWN\_PAYMENT**

-   Calculating the mean of the down payment rate per ID.

<!-- -->

    prev_rate_down_payment <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_rate_down_payment = mean(RATE_DOWN_PAYMENT, na.rm = T))

    dat_prev <- merge(dat_prev, prev_rate_down_payment, by = "SK_ID_CURR")

**RATE\_INTEREST\_PRIMARY**

-   Calculating the mean of the interest rate per ID.

<!-- -->

    prev_rate_interest_primary <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_rate_interest_primary = mean(RATE_INTEREST_PRIMARY, na.rm = T))

    dat_prev <- merge(dat_prev, prev_rate_interest_primary, by = "SK_ID_CURR")

**RATE\_INTEREST\_PRIVILEGED**

-   Calculating the mean of the interest rate (privileged) per ID.

<!-- -->

    prev_rate_interest_privileged <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_rate_interest_privileged = mean(RATE_INTEREST_PRIVILEGED, na.rm = T))

    dat_prev <- merge(dat_prev, prev_rate_interest_privileged, by = "SK_ID_CURR")

**NAME\_CASH\_LOAN\_PURPOSE**

-   Creating the relative frequency of the loan proposes per ID.

<!-- -->

    prev_name_cash_loan_prupose <- previous_application %>%
      select(SK_ID_CURR, NAME_CASH_LOAN_PURPOSE) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(NAME_CASH_LOAN_PURPOSE, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) %>%
      rename(xap.name.cash.loan.purpose = XAP) %>%
      rename(xnp.name.cash.loan.purpose = XNA)


    dat_prev <- merge(dat_prev, prev_name_cash_loan_prupose, by = "SK_ID_CURR")

**NAME\_CONTRACT\_STATUS**

-   Calculating the relative frequency of contract staus per ID.

<!-- -->

    prev_name_contract_status <- previous_application %>%
      select(SK_ID_CURR, NAME_CONTRACT_STATUS) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(NAME_CONTRACT_STATUS, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) 


    dat_prev <- merge(dat_prev, prev_name_contract_status, by = "SK_ID_CURR")

**DAYS\_DECISION**

-   Calculating the days decision per ID.

<!-- -->

    prev_days_decision <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_days_decision = mean(DAYS_DECISION, na.rm = T))

    dat_prev <- merge(dat_prev, prev_days_decision, by = "SK_ID_CURR")

**NAME\_PAYMENT\_TYPE**

-   Calculating the relative frequency of the payment method per ID.

<!-- -->

    prev_name_payment_type <- previous_application %>%
      select(SK_ID_CURR, NAME_PAYMENT_TYPE) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(NAME_PAYMENT_TYPE, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) %>%
      rename(xna.name.payment.type = XNA)

    dat_prev <- merge(dat_prev, prev_name_payment_type, by = "SK_ID_CURR")

**CODE\_REJECT\_REASON**

-   Calculating the relative frequency of the reject reasons per ID .

<!-- -->

    prev_code_reject_reason <- previous_application %>%
      select(SK_ID_CURR, CODE_REJECT_REASON) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(CODE_REJECT_REASON, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) %>%
      rename(xap.code.reject.reason = XAP) %>%
      rename(xna.code.reject.reason = XNA)

    dat_prev <- merge(dat_prev, prev_code_reject_reason, by = "SK_ID_CURR")

**NAME\_TYPE\_SUITE**

-   Calculating the relative frequency of the accompany per ID.

<!-- -->

    prev_name_type_suite <- previous_application %>%
      select(SK_ID_CURR, NAME_TYPE_SUITE) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(NAME_TYPE_SUITE, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean))

    dat_prev <- merge(dat_prev, prev_name_type_suite, by = "SK_ID_CURR")

**NAME\_GOODS\_CATEGORY**

-   Calculating the relative frequency of the goods category per ID.

<!-- -->

    prev_name_goods_category <- previous_application %>%
      select(SK_ID_CURR, NAME_GOODS_CATEGORY) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(NAME_GOODS_CATEGORY, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) %>%
      rename(xna.name.goods.category = XNA)


    dat_prev <- merge(dat_prev, prev_name_goods_category, by = "SK_ID_CURR")

**NAME\_PORTFOLIO**

-   Calculating the relative frequency of the portfolio per ID.

<!-- -->

    prev_name_portfolio <- previous_application %>%
      select(SK_ID_CURR, NAME_PORTFOLIO) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(NAME_PORTFOLIO, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) %>%
      rename(xna.name.portfolio = XNA)

    dat_prev <- merge(dat_prev, prev_name_portfolio, by = "SK_ID_CURR")

**NAME\_PRODUCT\_TYPE**

-   Calculating the relative frequency of the product type per ID.

<!-- -->

    prev_name_product_type <- previous_application %>%
      select(SK_ID_CURR, NAME_PRODUCT_TYPE) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(NAME_PRODUCT_TYPE, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) %>%
      rename(xna.name.product.type = XNA)


    dat_prev <- merge(dat_prev, prev_name_product_type, by = "SK_ID_CURR")

**CHANNEL\_TYPE**

-   Calculating the relative frequency of the channel type per ID.

<!-- -->

    prev_channel_type <- previous_application %>%
      select(SK_ID_CURR, CHANNEL_TYPE) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(CHANNEL_TYPE, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) 


    dat_prev <- merge(dat_prev, prev_channel_type, by = "SK_ID_CURR")

**SELLERPLACE\_AREA**

-   Calculating the mean of the sellerplace area per ID.

<!-- -->

    prev_sellerplace_area <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_sellerplace_area = mean(SELLERPLACE_AREA, na.rm = T))

    dat_prev <- merge(dat_prev, prev_sellerplace_area, by = "SK_ID_CURR")

**NAME\_SELLER\_INDUSTRY**

-   Calculating the relative frequency of the seller industry per ID.

<!-- -->

    prev_name_seller_industry <- previous_application %>%
      select(SK_ID_CURR, NAME_SELLER_INDUSTRY) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(NAME_SELLER_INDUSTRY, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) %>%
      rename(xna.name.seller.industry = XNA)

    dat_prev <- merge(dat_prev, prev_name_seller_industry, by = "SK_ID_CURR")

**CNT\_PAYMENT**

-   Calculating the mean of the term of the previous credit per ID.

<!-- -->

    prev_cnt_payment <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_cnt_payment = mean(CNT_PAYMENT, na.rm = T))

    dat_prev <- merge(dat_prev, prev_cnt_payment, by = "SK_ID_CURR")

**NAME\_YIELD\_GROUP**

-   Calculating the relative frequency of the yield group per ID.

<!-- -->

    prev_name_yield_group <- previous_application %>%
      select(SK_ID_CURR, NAME_YIELD_GROUP) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(NAME_YIELD_GROUP, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) %>%
      rename(xna.name.yield.group = XNA)

    dat_prev <- merge(dat_prev, prev_name_yield_group, by = "SK_ID_CURR")

**PRODUCT\_COMBINATION**

-   Calculating the relative frequency of the product combinations per
    ID.

<!-- -->

    prev_product_combination <- previous_application %>%
      select(SK_ID_CURR, PRODUCT_COMBINATION) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(PRODUCT_COMBINATION, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) 

    dat_prev <- merge(dat_prev, prev_product_combination, by = "SK_ID_CURR")

**DAYS\_FIRST\_DRAWING**

-   Calculating the mean of the relative application date per ID.

<!-- -->

    prev_days_first_drawing <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_days_first_drawing = mean(DAYS_FIRST_DRAWING , na.rm = T))

    dat_prev <- merge(dat_prev, prev_days_first_drawing, by = "SK_ID_CURR")

**DAYS\_FIRST\_DUE**

-   Calculating the mean of the first disbursement date per ID.

<!-- -->

    prev_days_first_due <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_days_first_due = mean(DAYS_FIRST_DUE , na.rm = T))

    dat_prev <- merge(dat_prev, prev_days_first_due, by = "SK_ID_CURR")

**DAYS\_LAST\_DUE\_1ST\_VERSION**

-   Calculating the mean of the relative last due date (first version)
    per ID.

<!-- -->

    prev_days_last_due_1ST_Version <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_days_last_due_1ST_Version = mean(DAYS_LAST_DUE_1ST_VERSION , na.rm = T))

    dat_prev <- merge(dat_prev, prev_days_last_due_1ST_Version, by = "SK_ID_CURR")

**DAYS\_LAST\_DUE**

-   Calculating the mean of the relative last due date per ID.

<!-- -->

    prev_days_last_due <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_days_last_due = mean(DAYS_LAST_DUE, na.rm = T))

    dat_prev <- merge(dat_prev, prev_days_last_due, by = "SK_ID_CURR")

**DAYS\_TERMINATION**

-   Calculating the mean of the relative termination date per ID.

<!-- -->

    prev_days_of_termination <- previous_application %>% group_by(SK_ID_CURR) %>%
      summarise(prev_days_of_termination = mean(DAYS_TERMINATION, na.rm = T))


    dat_prev <- merge(dat_prev, prev_days_of_termination, by = "SK_ID_CURR")

**NFLAG\_INSURED\_ON\_APPROVAL**

-   Calculating the relative frequency of the requested insurance per
    ID.

<!-- -->

    prev_nflag_insured_on_approval <- previous_application %>%
      select(SK_ID_CURR, NFLAG_INSURED_ON_APPROVAL) %>%
      mutate(ID = row_number(),
             yes = 1) %>%
      spread(NFLAG_INSURED_ON_APPROVAL, yes, fill = 0) %>%
      select(-ID) %>%
      group_by(SK_ID_CURR) %>%
      summarise_all(funs(mean)) %>%
      rename(xna.insured.on.approval = "<NA>")


    dat_prev <- merge(dat_prev, prev_nflag_insured_on_approval, by = "SK_ID_CURR")

-   Making valid column names.

<!-- -->

    colnames(dat_prev) <- make.names(colnames(dat_prev), unique = T)

4 Saving the Data
-----------------

    save(dat_prev,file="data_prev.Rda")
