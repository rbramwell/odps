# MaxCompute安全白皮书 {#concept_nqm_mft_ngb .concept}

## 法律声明 {#section_ivq_byb_pgb .section}

阿里云提醒您在阅读或使用本文档之前请仔细阅读、充分理解本法律声明各条款的内容。如果您阅读或使用本文档，您的阅读或使用行为将被视为对本声明全部内容的认可。

1.  您应当通过阿里云网站或阿里云提供的其他授权通道下载、获取本文档，且仅能用于自身的合法、合规的业务活动。本文档的内容视为阿里云的保密信息，您应当严格遵守保密义务；未经阿里云事先书面同意，您不得向任何第三方披露本手册内容或提供给任何第三方使用。
2.  未经阿里云事先书面许可，任何单位、公司或个人不得擅自摘抄、翻译、复制本文档内容的部分或全部内容，不得以任何方式或途径进行传播和宣传。
3.  由于产品版本升级、调整或其他原因，本文档内容有可能变更。阿里云保留在没有任何通知或者提示下对本文档的内容进行修改的权利，和在阿里云授权通道中不定时发布更新后的用户文档。您应当实时关注用户文档的版本变更并通过阿里云授权渠道下载、获取最新版的用户文档。
4.  本文档仅作为用户使用阿里云产品及服务的参考性指引，阿里云以产品及服务的**现状**、**有缺陷**和**当前功能**的状态提供本文档。阿里云在现有技术的基础上尽最大努力提供相应的介绍及操作指引，但阿里云在此明确声明对本文档内容的准确性、完整性、适用性、可靠性等不作任何明示或暗示的保证。任何单位、公司或个人因为下载、使用或信赖本文档而发生任何差错或经济损失的，阿里云不承担任何法律责任。在任何情况下，阿里云均不对任何间接性、后果性、惩戒性、偶然性、特殊性或刑罚性的损害，包括用户使用或信赖本文档而遭受的利润损失，承担责任\(即使阿里云已被告知该等损失的可能性\)。
5.  阿里云网站上所有内容，包括但不限于著作、产品、图片、档案、资讯、资料、网站架构、网站画面的安排、网页设计，均由阿里云和/或其关联公司依法拥有其知识产权，包括但不限于商标权、专利权、著作权、商业秘密等。非经阿里云和/或其关联公司书面同意，任何人不得擅自使用、修改、复制、公开传播、改变、散布、发行或公开发表阿里云网站、产品程序或内容。此外，未经阿里云事先书面同意，任何人不得为了任何营销、广告、促销或其他目的使用、公布或复制阿里云的名称\(包括但不限于单独为或以组合形式包含**阿里云**、**Aliyun**、**万网**等阿里云和/或其关联公司品牌，上述品牌的附属标志及图案或任何类似公司名称、商号、商标、产品或服务名称、域名、图案标示、标志、标识或通过特定描述使第三方能够识别阿里云和/或其关联公司。
6.  如若发现本文档存在任何错误，请与阿里云取得直接联系。

## 安全隔离 {#section_pmy_tyb_pgb .section}

MaxCompute支持多租户的使用场景，通过阿里云账号认证体系，即认证方式采用AccessKey对称密钥认证技术，同时对于用户的每一个HTTP请求都会进行签名认证，针对不同的用户数据进行数据存储隔离，用户数据被离散存储在分布式文件系统中。可以同时满足多用户协同、数据共享、数据保密和安全的需要，做到真正的多租户资源隔离。同时，MaxCompute中所有计算是在受限的沙箱中运行的，多层次的应用沙箱，从KVM级到Kernel级。系统沙箱配合鉴权管理机制，用来保证数据的安全，以避免出现内部人员恶意或粗心造成服务器故障。

沙箱保护如图：

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/116975/156462890538224_zh-CN.png)

## 权鉴认证 {#section_fx3_2zb_pgb .section}

-   身份验证

    用户可以在阿里云控制台中自行创建AccessKey。AccessKey由AccessKey ID和AccessKey Secret组成，其中AccessKey ID是公开的，用于标识用户身份，AccessKey Secret是秘密的，用于用户身份的鉴别。当用户向MaxCompute发送请求时，首先需要将发送的请求按照MaxCompute指定的格式生成签名字符串，然后使用AccessKey Secret对签名字符串进行加密以生成请求签名。MaxCompute收到用户请求后，通过AccessKey ID找到对应的AccessKey Secret，以同样的方法提取签名字符串和验证码，如果计算出来的验证码和提供的一致即认为该请求是有效的；否则，MaxCompute将拒绝处理这次请求，并返回HTTP 403错误。

-   权限控制

    用户对MaxCompute资源访问分为两种，即用户主账号访问和用户子账号访问。主账号是阿里云的一个账号主体，主账号下可以包含不同的子账号以便用户可以灵活使用。MaxCompute支持主、子账号的权限访问策略：

    -   当用户使用主账号访问时，MaxCompute会校验该主账号是否为对应资源的所有者，只有对应资源的所有者才具备访问该资源的权限。
    -   当用户使用子账号访问时，此时会触发子账号授权策略。MaxCompute会校验该子账号是否被对应主账号授予了访问该资源的权限，同时也会校验该子账号对应的主账号是否具有该资源的所有者权限。
    MaxCompute目前主要支持ACL授权机制来完成对子账号的访问权限控制。

    ACL授权：ACL授权是一种基于对象的授权。通过ACL授权的权限数据\(即访问控制列表， Access Control List\)被看做是该对象的一种子资源，只有当对象已经存在时，才能进行ACL授权操作。当对象被删除时，通过ACL授权的权限数据会被自动删除。ACL授权支持的授权方法是采用类似SQL92定义的GRANT/REVOKE命令进行授权，通过对应的授权命令来完成对已存在的项目空间对象的授权或撤销授权。

    每次权限管理操作均是对效果\(授权、撤销\)、对象\(如表、资源等\)、主体\(用户或是角 色\)、操作\(读、写、删除等\)的组合描述，例如**允许用户zinan.tang读取表table1中的数据**。

    MaxCompute还支持更多的访问权限控制机制。

    -   跨项目空间的资源分享

        假设用户是项目空间的Owner或管理员\(admin角色\)，有人需要申请访问用户的项目空间资源。 如果申请人属于用户的项目团队，此时建议用户使用项目空间的用户授权管理功能。但是如果申请人并不属于用户的项目团队，此时用户可以使用基于Package的跨项目空间的资源分享功能。

        Package是一种跨项目空间共享数据及资源的机制，主要用于解决跨项目空间的用户授权问题。

        使用Package之后，A项目空间管理员可以对B项目空间需要使用的对象进行打包授权\(也就是创 建一Package\)，然后许可B项目空间安装这个Package。在B项目空间管理员安装Package之后，就可以自行管理Package是否需要进一步授权给自己Project下的用户。

        Package使用方法示例如下。

        -   Package创建者的操作示例如下。

            ``` {#codeblock_47q_agj_573}
            - 创建Package
            create package <pkgname>;
            --注意:
            --• 仅project的owner有权限进行该操作。
            --• 目前创建的package名称不能超过128个字符。
            ```

            ``` {#codeblock_c2g_6tx_fkb}
            -- 添加资源到Package
            add project_object to package package_name [with privileges privileges];
            remove project_object from package package_name;
            project_object ::= table table_name |
            instance inst_name |
            function func_name |
            resource res_name
            privileges ::= action_item1, action_item2, ...
            --说明:
            --• 目前支持的对象类型不包括Project类型，即不允许通过Package在其他Project中创建对象。 
            --• 添加到Package中的不仅仅是对象本身，还包括相应的操作权限。当没有通过[withprivileges privileges]来指定操作权限时，默认为只读权限， 即Read/Describe/Select。“对象及其权限”被看作一个整体，添加后不可被更新。若有需要，只能删除和重新添加。
            ```

            ``` {#codeblock_etc_ijc_qnk}
            -- 赋予其它项目空间使用权限
            allow project <prjname> to install package <pkgname> [using label <number>];
            ```

            ``` {#codeblock_yeg_pyv_f3u}
            -- 撤销其它项目空间使用权限
            disallow project <prjname> to install package <pkgname>;
            ```

            ``` {#codeblock_32g_fab_wk4}
            -- 删除Package
            delete package <pkgname>;
            ```

            ``` {#codeblock_mi9_8ec_2n3}
            -- 查看Package列表
            show packages;
            ```

            ``` {#codeblock_6vw_q59_frh}
            -- 查看Package详细信息
            describe package <pkgname>;
            ```

        -   Package使用者的操作示例如下。

            ``` {#codeblock_3t7_twf_ny3}
            -- 安装Package
            install package <pkgname>; 
            --说明:
            -- • 仅project的owner有权限进行该操作。
            --• 对于安装Package来说，要求pkgName的格式为:<projectName>.<packageName>。
            ```

            ``` {#codeblock_vyi_ab1_452}
            -- 卸载Package
            uninstall package <pkgname>; 
            -- 说明: 对于卸载Package来说，要求pkgName的格式为:<projectName>.<packageName>。
            ```

            ``` {#codeblock_jaz_4xu_fgq}
            -- 查看已创建和已安装的package列表
            show packages;
            ```

            ``` {#codeblock_set_psd_fca}
            -- 查看package详细信息
            describe package <pkgname>;
            ```

            被安装的Package是独立的MaxCompute对象类型，若要访问Package里的资源\(即其他项目空间分享给用户的资源\)，必须拥有对该Package的Read权限。若请求者无Read权限，则需向ProjectOwner或Admin申请，ProjectOwner或Admin可以通过ACL授权机制来完成授权。

            示例如下，仅供参考：通过ACL授权允许云账号odps\_test@aliyun.com访问Package里的资源。

            ``` {#codeblock_egq_q5h_ydv}
            use prj2;
            install package prj1.testpkg;
            grant read on package prj1.testpackage to user
            aliyun$odps_test@aliyun.com;
            ```

    -   列级别访问控制

        基于标签的安全\(LabelSecurity\)是项目空间级别的一种强制访问控制策略\(Mandatory Access Control，MAC\)，它的引入可以让项目空间管理员更加灵活地控制用户对列级别敏感数据的访问。

        LabelSecurity需要将数据和访问数据的人进行安全等级划分。一般来讲，会将数据的敏感度标记分为如下四类：

        -   0级\(不保密，Unclassified\)。
        -   1级\(秘密，Confidential\)。
        -   2级\(机密，Sensitive\)。
        -   3级\(高度机密，HighlySensitive\)。
        MaxCompute也遵循这一分类方法，ProjectOwner需要定义明确的数据敏感等级和访问许可等级划分标准。默认时所有用户的访问许可等级为0级，数据安全级别默认为0级。

        LabelSecurity对敏感数据的粒度可以支持列级别，管理员可以对表的任何列设置敏感度标记Label，一张表可以由不同敏感等级的数据列构成。而对于view，也支持和表相同的设置，即管理员可以对view设置label等级。

        View的等级和它对应的基表的label等级是独立的，在view创建时，默认的等级也是0级。

        在对数据和人分别设置安全等级标记之后，LabelSecurity的默认安全策略如下：

        -   No-ReadUp：不允许用户读取敏感等级高于用户等级的数据，除非显式授权。
        -   Trusted-User：允许用户写任意等级的数据，新建数据默认为0级\(不保密\)。
        **说明：** 

        -   在一些传统的强制访问控制系统中，为了防止数据在项目空间内部的任意分发，一般还支持更多复杂的安全策略，例如：不允许用户写敏感等级不高于用户等级的数据\(No-WriteDown\)。 但在MaxCompute平台中，考虑到项目空间管理员对数据敏感等级的管理成本，默认安全策略并不支持No-WriteDown，如果项目空间管理员有类似需求，可以通过修改项目空间安全配置`SetObjectCreatorHasGrantPermission=false`以达到控制目的。
        -   如果是为了控制数据在不同项目空间之间的流动，则可以将项目空间设置为受保护状态\(ProjectProtection\)。设置之后，只允许用户在项目空间内访问数据，这样可以有效防止数据流 出道项目空间之外。
        项目空间中的LabelSecurity安全机制默认是关闭的，ProjectOwner需要自行开启。需要注意， LabelSecurity安全机制一旦开启，上述的默认安全策略将被强制执行。此时，当用户访问数据表 时，除了必须拥有Select权限外，还必须获得读取敏感数据的相应许可等级。

        -   数据保护机制\(ProjectProtection\)

            同时在多个项目空间中拥有访问权限的用户，可以自由地使用任意支持跨Project的数据访问操作来 转移项目空间的数据。但是，如果项目空间中的数据非常敏感，绝对不允许流出到其他项目空间中去，此时管理员可以使用项目空间保护机制——设置ProjectProtection，明确要求项目空间中**数据只能本地循环，允许写入，不能读出**。

            具体设置如下：

            ``` {#codeblock_ztu_c6j_om1}
            set projectProtection=true;
            -- 设置ProjectProtection规则为:数据只能流入，不能流出。
            --需要注意，默认ProjectProtection不会被设置，默认值为false，即数据保护机制按需开启。
            ```

        -   开启数据保护机制后的数据流出方法

            设置TrustedProject。若当前项目空间处于受保护状态，如果将数据流出的目标空间设置为当前空间的TrustedProject，那么向目标项目空间的数据流向将不会被视为触犯ProjectProtection规则。如果多个项目空间之间两两互相设置为TrustedProject，那么这些项目空间就形成了一个TrustedProject Group，数据可以在这个Project Group内流动，但禁止流出到Project Group之外。

            管理TrustedProject的命令如下：

            ``` {#codeblock_bew_43v_z95}
            list trustedprojects;
            -- 查看当前project中的所有TrustedProjects。 
            add trustedproject <projectname>;
            -- 在当前project中添加一个TrustedProject。 
            remove trustedproject <projectname>;
            -- 在当前project中移除一个TrustedProject。
            ```

        -   资源分享与数据保护的关系

            在MaxCompute中，基于package的资源分享机制与ProjectProtection数据保护机制是正交的，但在功能上却是相互制约的。

            MaxCompute规定：资源分享优先于数据保护。即如果一个数据对象是通过资源分享方式授予其他项目空间用户访问，那么该数据对象将不受ProjectProtection规则的限制。

            -   防止数据从项目空间流出的更多检查

                如果要防止数据从项目空间的流出，在设置`ProjectProtection=true`之后，还需检查如下配置：

                -   确保没有添加trustedproject。如果有设置，则需要评估可能的风险。
                -   确保没有设置exception policy。如果有设置，则需要评估可能的风险，尤其要考虑TOC2TOU数据泄露风险。
                -   确保没有使用package数据分享。如果有设置，则需要确保package中没有敏感数据。
-   RAM支持

    MaxCompute支持RAM鉴权。

    RAM\(Resource Access Management\)是阿里云提供的资源访问控制服务。通过RAM，主账号可以创建出子账号，子账号从属于主账号，所有资源都属于主账号，主账号可以将所属资源的访问权限授予给子账号。


## 数据安全 {#section_pky_w1c_pgb .section}

阿里云提供一个扁平的线性存储空间，并在内部对线性地址进行切片，一个分片称为一个Chunk。 对于每一个Chunk，都会复制出三个副本，并将这些副本按照一定的策略存放在集群中的不同节点 上，保证用户数据的可靠。

在数据存储系统中，有三类角色，分别称为Master、Chunk Server和Client。MaxCompute用户的每一个写操作经过层层转换，最终会交由Client来执行，执行过程如下：

1.  Client计算出这个写操作对应的Chunk。
2.  Client向Master查询该Chunk的三份副本的存放位置。
3.  Client根据Master返回的结果，向对应的三个Chunk Server发出写请求。
4.  如果三份副本都写成功，Client向用户返回成功；反之，Client向用户返回失败。

Master的分布策略会综合考虑集群中所有Chunk Server的磁盘使用情况、在不同交换机机架下的分 布情况、电源供电情况、及机器负载情况，尽量保证一个Chunk的三个副本分布在不同机架下的不同Chunk Server 上，从而有效防止由于一个Chunk Server或一个机架的故障导致的数据不可用。

当有数据节点损坏，或者某个数据节点上的部分硬盘发生故障时，集群中部分Chunk的有效副本数 会小于三。一旦发生这种情况，Master就会启动复制机制，在Chunk Server之间复制数据，保证集 群中所有Chunk的有效副本数达到三份。

综上所述，对MaxCompute上的数据而言，所有用户层面的操作都会同步到底层三份副本上，无论是新增、修改还是删除数据。通过这种机制，保障用户数据的可靠性和一致性。

另外，在用户进行删除操作后，释放的存储空间由飞天分布式文件系统回收，禁止任何用户访问，同时对内容进行擦除，最大限度保证用户的数据安全性。

## 传输加密 {#section_a51_z1c_pgb .section}

MaxCompute提供Restful的传输接口，其传输安全性由HTTPS保证。

## 日志审计 {#section_g5m_1bc_pgb .section}

MaxCompute会针对不同用户不同日志数据进行日志审计。在MaxCompute内部，MaxCompute提供元数据仓库进行日志数据存储，包括静态数据、运行记录及安全信息等内容。

-   静态数据：是指一旦产生就不会自动消失的数据。
-   运行记录：表示一个任务的运行过程，该记录只会出现在一个分区中。
-   安全信息：都来自TableStore，用于保存白名单、ACL列表等。

元数据仓库：就是使用MaxCompute来分析MaxCompute自己的运行状况，将MaxCompute中的各种元信息整理汇总成MaxCompute中的表，方便用户查询和统计。

## 访问控制-IP白名单 {#section_ehy_bbc_pgb .section}

MaxCompute安全上的访问控制有多个层次：如项目空间的多租户及安全认证机制，只有获取了正确的经过授权的AccessKey ID及Access Secret才能通过鉴权，在已经赋予的权限范围内进行数据访问和计算。本章节主要介绍在以上访问认证基础上增强的一种以IP白名单的方式，进行访问控制的配置方法和策略，并指导用户完成相关配置。

获取需要配置的IP地址的方式如下：

-   如果使用MaxCompute Console\(odpscmd\)使用，用户可以直接获取机器的IP地址。
-   如果使用应用系统\(如DataWorks或者数据集成\)进行项目空间数据访问，需要配置DataWorks或者数据集成所在的部署server机器的IP地址（应用系统会将默认的机器ip统一添加白名单）。
-   如果使用了代理服务器或者经过了多跳代理服务器来访问MaxCompute服务实例，需要添加的IP地址为最后一跳代理服务器的IP地址。
-   如果是ECS机器中访问MaxCompute服务，获取到的IP地址为NAT IP。

IP地址配置的格式如下：

多个IP由`逗号,`分割，且支持三种IP格式：

1.  单独IP地址。
2.  IP地址段，由`-`连接。
3.  带有子网掩码的IP。

示例如下：

``` {#codeblock_eax_qrl_ltu}
10.32.180.8,10.32.180.9,10.32.180.10
-- 单独IP地址。 
10.32.180.8-10.32.180.12
-- IP地址段。 
10.32.180.0/23
-- 带子网掩码的IP地址。
```

下面将介绍project级别IP白名单所涉及的相关配置操作。

Project的owner通过console执行命令如下：

``` {#codeblock_n7w_3zo_u0x}
setproject odps.security.ip.whitelist=101.132.236.134,100.116.0.0/16,101.132.236.134-101.132.236.144;
```

**说明：** 

-   设置IP白名单后，只有白名单列表中的IP（console或者SDK所在的出口IP）能够访问这个Project。
-   设置IP白名单后，您需要等待五分钟后才会生效。
-   VPC环境下当前仅支持针对SLB整段地址进行放通。
-   如果您误操作，将自己屏蔽，请通过提工单向阿里云技术支持寻求帮助。

IP白名单清空后，MaxCompute就认为Project关闭了白名单功能。

``` {#codeblock_sqg_wei_gjy}
setproject odps.security.ip.whitelist=;
```

影响与效果

-   配置之前MaxCompute服务针对访问项目空间的机器IP地址没有限制。
-   配置之后，满足配置规则的IP地址及IP地址段才能访问该项目空间。在原有AccessKey ID及AccessKey Secret认证基础上叠加了IP规则的检查。

