# Download pricing\(Billing by quantity\) {#concept_2245131 .concept}

This topic introduces Maxcompute download pricing.

You can download data from the extranet through the MaxCompute Tunnel. The billing method for data downloads is Pay-As-You-Go. The calculation is as follows.

``` {#codeblock_31t_box_112}
Download Cost **from** Extranet/time = Downloaded Data Volume x Download Price
```

The price is as follows.

|item|price|
|:---|:----|
|Data download|0.1166 USD/GB|

**Note:** 

-   MaxCompute sends you messages to notify you of the size of your downloads, and to provide you with your download costs the next day.
-   Download data volume refers to the size of an HTTP body for one download request. The HTTP body that carries data uses protobuffer encoding, so it is generally smaller than the original data size, but larger than the data stored in MaxCompute after compression.
-   The different billing methods are applicable to different network environments, such as public networks, classic networks of Alibaba Cloud, or VPC networks. For more information about MaxCompute service connections, see [Configure endpoints](../../../../intl.en-US/Prepare/Configure endpoints.md#).

