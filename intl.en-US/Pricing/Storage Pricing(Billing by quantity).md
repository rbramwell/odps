# Storage Pricing\(Billing by quantity\) {#concept_2245135 .concept}

The data that is stored in MaxCompute, including tables and resources, is billed according to the storage used. The billing cycle is **one day** .This topic introduces MaxCompute Storage Pricing.

MaxCompute records the storage used for each project on an hourly basis. The average storage over the day is then calculated for each project space at the end of each day.

The daily MaxCompute fee is calculated by applying the tiered unit prices in the table following to the average storage used. Up to 1 GB of storage is free each day, while storage used between 1 GB and 100 GB costs 0.0028 USD for each gigabyte and so on. If you require more than 1 PB of storage per day, you can open a ticket to get a quote for the price.

|Less than 1 GB USD/GB/Day|1 GB to 100 GB USD/GB/Day|100 GB to 1 TB USD/GB/Day|1 TB to 10 TB USD/GB/Day|10 TB to 100 TB USD/GB/Day|100 TB to 1 PB USD/GB/Day|More than 1 PB USD/GB/Day|
|-------------------------|-------------------------|-------------------------|------------------------|--------------------------|-------------------------|-------------------------|
|Free|0.0028|0.0014|0.0013|0.0011|0.0009|Please contact us|

For example, if you store 50 TB data in MaxCompute, the bill is calculated as follows.

``` {#codeblock_t3h_nwf_rc9}
(100 GB - 1) × 0.0028 USD/GB/day
+ (1024 - 100) GB × 0.0014 USD/GB/day
+ (10240 - 1024) GB × 0.0013 USD/GB/day
+ (50 x 1024 - 10240) GB × 0.0011 USD/GB/day
= 58.61 USD/day
```

**Note:** 

-   Because MaxCompute compresses and stores user data, the bill is based on the capacity size of the data after compresssion. This means the size of the stored data is different from the size of the data as measured by you before storage. The compression ratio is generally about 5.
-   Generally, MaxCompute fees are deducted no more than 6 hours after the daily fee calculation is completed, and are automatically deducted from the corresponding account balance.
-   On the MaxCompute console, you can view your consumption details under **Bill Management** .

