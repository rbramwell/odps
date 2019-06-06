# ACID语义 {#concept_287059 .concept}

本文为您介绍MaxCompute在作业并发情况下ACID的语义。

## 相关术语 {#section_hf5_rwi_nxq .section}

-   操作：指在MaxCompute上提交的单个作业。
-   数据对象：指持有实际数据的对象，例如非分区表、分区。
-   INTO类作业：`INSERT INTO`、`DYNAMIC INSERT INTO`等。
-   OVERWRITE类作业：`INSERT OVERWRITE`、`DYNAMIC INSERT OVERWRITE`等。
-   Tunnel数据上传：可以归结为INTO类或OVERWRITE类作业。

## ACID语义描述 {#section_fsk_n8m_1iu .section}

-   原子性（Atomicity）：一个操作或是全部完成，或是全部不完成，不会结束在中间某个环节。
-   一致性（Consistency）：从操作开始至结束的期间，数据对象的完整性没有被破坏。
-   隔离性（Isolation）：操作独立于其它并发操作完成。
-   持久性（Durability）：操作处理结束后，对数据的修改将永久有效，即使出现系统故障，该修改也不会丢失。

## MaxCompute的ACID具体情形 {#section_l0e_bvb_1d5 .section}

-   原子性（Atomicity）
    -   任何时候MaxCompute会保证在冲突时只会一个作业成功，其它冲突作业失败。
    -   对于单个表/分区的CREATE/OVERWRITE/DROP操作，可以保证其原子性。
    -   跨表操作时不支持原子性（例如MULTI-INSERT）。
    -   在极端情况下，以下操作可能不保证原子性：
        -   `DYNAMIC INSERT OVERWRITE`多于一万个分区，不支持原子性。
        -   INTO类操作：这类操作的失败的原因是事务回滚时数据清理失败，但不会造成原始数据丢失。
-   一致性（Consistency）
    -   OVERWRITE类作业可保证一致性。
    -   INTO类作业在冲突失败后可能存在失败作业的数据残留。
-   隔离性（Isolation）
    -   非INTO类操作保证读已提交。
    -   INTO类操作存在读未提交的场景。
-   持久性（Durability）

    MaxCompute保证数据的持久性。


