# Renewal management {#concept_xpc_gxm_xdb .concept}

This topic describes how a Subscription instance can be manually or automatically renewed in the Renew center.

When your Subscription instance expires:

-   The Subscription projects associated with the instance are immediately locked by the system, and the instance enters the **Expiration** state. You can activate the instance only after you pay for it.
-   If the instance is not renewed within the specified time period or the renewal fails, the resources in the projects associated with the instance are released and cannot be recovered.

## Procedure for manual renewal {#section_kfm_lxm_xdb .section}

To renew a Subscription instance manually, follow these steps:

1.  Log on to the [MaxCompute console](https://renew-intl.console.aliyun.com/?spm=a2c95.c0e1020.app.9.df741127W29Zuz#/renew/odpsplus_intl?_k=gcoojz) by using your Alibaba Cloud account, choose **Billing Management** \> **Renew** in the top navigation bar, and in the left-side navigation pane of the displayed page click **MaxCompute**.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13800/156404878752765_en-US.jpg)

2.  On the **Manually Renew** tab, find the instance you want to renew, and in the **Actions** column click **Renew**.
3.  Drag the blue slider to specify a time period, and then click **Pay**.

## Procedure for automatic renewal {#section_rfm_lxm_xdb .section}

To enable automatic renewal, follow these steps:

1.  Log on to the [MaxCompute console](https://renew-intl.console.aliyun.com/?spm=a2c95.c0e1020.app.9.df741127W29Zuz#/renew/odpsplus_intl?_k=gcoojz)by using your Alibaba Cloud account, choose **Billing Management** \> **Renew** in the top navigation bar, and in the left-side navigation pane of the displayed page click **MaxCompute**.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13800/156404878752759_en-US.jpg)

2.  Find the instance you want to renew, and in the **Actions** column click **Enable Auto-Renew**.
3.  In the Enable Auto-Renew dialog box, select a time period from the **Auto-Renew Cycle** drop-down list and click **Enable Auto-Renew**.
4.  On the **Auto-Renew** tab, view information about the instance whose fees can be automatically paid.

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13800/156404878752761_en-US.jpg)

5.  Optional. Click **Modify Auto-Renew** to change the automatic renewal cycle, or click **Don't Renew** to cancel automatic renewal.

    **Note:** After you enable automatic renewal for the instance:

    -   After you enable automatic renewal for the instance, the system attempts to deduct fees from your account at 8:00 on the next day following the expiration date. If the deduction fails, the system attempts to deduct fees again on the sixth day. If the deduction still fails, the system attempts to deduct fees for the third time on the 14th day. If this third deduction fails, we recommend that you manually pay for the instance. If you do not pay for the instance, the instance stops on the 15th day.
    -   If you want to manually pay for the instance, make sure that you complete the payment before 8:00 on the day before the instance expires.
    -   If you select the **Don't Renew** option, the system does not send expiration reminders to you. However, three days before the instance expires, the system sends a reminder to you to verify that you do not want to renew the instance.

## Additional information {#section_xs1_cwr_opv .section}

When your Subscription instance expires, the Subscription projects associated with the instance are immediately locked by the system, and the instance enters the **Expiration** state. Therefore, you can only activate the instance after a payment is made to renew the instance.

**Note:** If the instance is not renewed within the specified time period or the renewal fails, the resources in the projects associated with the instance are released and cannot be recovered.

