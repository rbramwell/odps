# 步骤二：添加角色并通过ACL授权 {#concept_kjy_2sz_vdb .concept}

本文向您介绍如何添加项目角色并通过ACL授权。

## 场景描述 {#section_xck_fie_wc9 .section}

Jack是项目空间prj1的管理员，有三个新加入的项目组成员：Alice、 Bob、Charlie，他们的角色是数据审查员。他们想申请如下权限：查看Table列表、提交作业、读取表userprofile。对于这个场景的授权，项目空间管理员可以使用基于对象的[ACL授权](cn.zh-CN/管理/安全功能详解/用户及授权管理/授权.md#) 机制来完成。

## 操作方法 {#section_1kz_l3e_1z2 .section}

由项目管理员输入下列命令行：

```
    use prj1;
    add user aliyun$alice@aliyun.com; --添加用户。
    add user aliyun$bob@aliyun.com;
    add user aliyun$charlie@aliyun.com;
    create role tableviewer; --创建角色。
    grant List, CreateInstance on project prj1 to role tableviewer; --对角色赋权。
    grant Describe, Select on table userprofile to role tableviewer;
    grant tableviewer to aliyun$alice@aliyun.com; --对用户赋予角色tableviewer。
    grant tableviewer to aliyun$bob@aliyun.com;
    grant tableviewer to aliyun$charlie@aliyun.com;
```

