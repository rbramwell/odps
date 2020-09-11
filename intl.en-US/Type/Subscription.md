---
keyword: [subscription, CU]
---

# Subscription

This topic describes the basic resources, billing rules, usage notes, and the considerations for MaxCompute subscription.

## Basic resources

MaxCompute subscription provides the following basic resources:

-   Computing resources are allocated based on the number of CUs that you purchase, and are combined into a dedicated resource pool. For projects that use the subscription billing method, only resources that are dedicated to the projects are consumed to execute computing jobs, such as SQL statements, UDFs, MapReduce jobs, Spark jobs, and Graph jobs.
-   Storage resources are shared and automatically scaled. No limits are imposed on these resources. MaxCompute tables and resources consume storage resources.
-   Resources for data uploads and downloads are shared. You cannot specify resources for upload and download jobs. No limits are imposed on these resources. However, both upload and download jobs must compete for these resources. For example, these resources are consumed when you use the Tunnel Upload and Download commands.

## Billing rules

The resources of MaxCompute are charged based on two billing methods. CUs that are purchased when you activate MaxCompute are charged based on the subscription billing method. The resources are charged based on the following rules:

-   Computing resources: You are charged for the CUs that you purchase. You must purchase at least 10 CUs. For more information, see [Billing for Computing](/intl.en-US/Pricing/Computing pricing.md).
-   Storage resources: You are charged only for the resources that are used to store tables. MaxCompute compresses your data for storage. You are charged for the data volume after compression. Data is typically compressed to about 20% of the original data volume. For more information, see [Storage pricing \(pay-as-you-go\)](/intl.en-US/Pricing/Storage pricing (pay-as-you-go).md).
-   Resources for data uploads: You are not charged for the resources that are used to upload data to MaxCompute.
-   Resources for data downloads: You are charged only for the resources that are used to download data over the Internet. For more information, see [Download pricing\(Pay-As-You-Go\)](/intl.en-US/Pricing/Download pricing(Pay-As-You-Go).md).

## Usage notes

-   For more information about how to activate MaxCompute and purchase resources through subscription, see [Activate MaxCompute](/intl.en-US/Prepare/Activate MaxCompute.md). Resources purchased through subscription can be used by all projects that reside in the same region where the resources are purchased.

    For example, if you purchase 100 CUs in the China \(Hangzhou\) region through subscription, MaxCompute provides you with a dedicated resource pool that contains 100 CUs. If the projects in this region use resources that are charged based on the subscription billing method, the computing jobs of these projects can use only resources in this resource pool. If a computing job uses less than 100 CUs, the remaining CUs remain idle. If a computing job requires more than 100 CUs, the job uses the 100 available CUs. As a result, the execution duration is prolonged.

-   After you purchase subscription-based resources, you can use MaxCompute Management to manage resources, monitor the system status and jobs, and allocate quota groups. For more information, see [MaxCompute Management](/intl.en-US/Management/Resource and job management/MaxCompute Management.md).
-   Subscription and pay-as-you-go billing methods are independent of each other. If you want a project to use subscription-based resources, you must select the subscription billing method for the project. After you purchase a subscription resource package, you can create a project that can use subscription-based resources. If you want the existing projects to use subscription-based resources, you must switch the billing method of these projects to subscription. For more information, see [t11937.md\#](/intl.en-US/Pricing/Switch billing methods.md).
-   If a project uses a pay-as-you-go resource package and you have purchased a subscription-based resource package, you can switch the resource package of the project to the subscription-based resource package. If a project uses a subscription-based resource package and you have purchased a pay-as-you-go resource package, you can switch the resource package of the project to the pay-as-you-go resource package.
-   If the CUs that you purchase through subscription no longer meet your requirements, you can increase or decrease the number of CUs. For more information, see [Configuration upgrade and downgrade](/intl.en-US/Pricing/Configuration upgrade and downgrade.md).

