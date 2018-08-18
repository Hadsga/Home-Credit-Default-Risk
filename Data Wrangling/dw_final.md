#### Packages

    library(dplyr)

### 1 Reading data

    load("../Data Wrangling/data_bureau.Rda")
    load("../Data Wrangling/data_pos.Rda")
    load("../Data Wrangling/data_prev.Rda")
    load("../Data Wrangling/data_card.Rda")
    load("../Data Wrangling/data_inst.Rda")
    load("../Data Wrangling/data_train_test.Rda")

### 2 Data Wrangling

-   Merging data frames.
-   Removing NaN´s and Inf´s from the data.
-   Converting traget variable to a factor.

<!-- -->

    temp1 <- merge(dat_card, dat_bureau, by = "SK_ID_CURR", all = T)
    temp2 <- merge(temp1, dat_pos, by = "SK_ID_CURR", all = T)
    temp3 <- merge(temp2, dat_prev, by = "SK_ID_CURR", all = T)
    temp4 <- merge(temp3, dat_inst, by = "SK_ID_CURR", all = T)
    total <- full_join(full, temp4, by = "SK_ID_CURR")

    total[total=="NaN" & total!="NA"] <- 0
    total[total=="Inf" & total!="NA"] <- 0
    total[total=="-Inf" & total!="NA"] <- 0

    total <- total %>% mutate(TARGET = as.factor(TARGET))

-   Creating original train and test partitionitian.

<!-- -->

    train <- total[1:307511,]
    test <- total[307512:356255,]

### 3 Saving data

    save(total,file="full_final.Rda")
    save(train,file="train_final.Rda")
    save(test,file="test_final.Rda")
