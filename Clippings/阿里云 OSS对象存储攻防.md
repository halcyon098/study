---
title: "阿里云 OSS对象存储攻防"
source: "https://zone.huoxian.cn/d/918-oss"
author:
  - "[[火线 Zone-安全攻防社区]]"
published: 2022-02-24
created: 2025-03-18
description: "阿里云 OSS对象存储攻防 本文分为两个部分 第一部分介绍OSS对象存储攻防的方式 第二部分为真实漏洞案例 1、Bucket权限配置错误-公开访问 在创建Bucket桶时，默认是private的权限，如果在错误的配置下，给了listobject权限，就会导致可遍历存储桶 在此时如果选择公有读的话，会出现两..."
tags:
  - "clippings"
---
本文分为两个部分  
第一部分介绍OSS对象存储攻防的方式  
第二部分为真实漏洞案例

## 1、Bucket权限配置错误-公开访问

在创建Bucket桶时，默认是private的权限，如果在错误的配置下，给了listobject权限，就会导致可遍历存储桶

![image-20220211181113047](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9936f7cd7f2249ac7bc7428ea20070e3.png)

在此时如果选择公有读的话，会出现两种情况

1、在只配置读写权限设置为公有读或公共读写的情况下，无法列出对象

![image-20220212120109881](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/03696006bb7a054b8e6b75ca73f08734.png)

但是可以直接访问对应的KEY路径

![image-20220212130334977](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9c8b8b77370622ce50dfe044463da8e9.png)

![image-20220212130352857](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d7b822a12655570c30aff828215c72a4.png)

![image-20220212130410829](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/dbe01497dc470aa1edd4051b2f5c09a4.png)

2、如果想列出Object对象，只需要在Bucket授权策略中设置ListObject即可

![image-20220212130653830](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ce0e5b4744b776c4d07e39d1233ac98b.png)

![image-20220212130718351](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/7ef12883c328759b0e09fc90fa552f14.png)

这样再当我们访问存储桶域名的时候就会发现，已经把我们存储桶的东西列出来了

![image-20220212130802988](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/0a54cefbe4e592b01e685d32f02cd48e.png)

## 2、Bucket桶爆破

当不知道 Bucket 名称的时候，可以通过爆破获得 Bucket 名称，这有些类似于目录爆破，只不过目录爆破一般通过状态码判断，而这个通过页面的内容判断。

当对于阿里云OSS 不存在有两种返回情况，分别是 InvalidBucketName 和 NoSuchBucket

![image-20220212131516708](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/685c63084c14e6754b5c4d2f21f5dbd0.png)

**InvalidBucketName**：表示存储桶的名称不符合规范，属于无效的存储桶名称

![image-20220212131606965](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ee887ea80158b9e9ff834006a03cd4a3.png)

**NoSuchBucket**：表示没有这个存储桶

当存储桶存在时，则会返回以下两种情况

![image-20220212131741241](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/d380009af880c160ef72023d129f0bfa.png)

![image-20220212131857498](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ed1525484da1c7d72bd5bce782ecdc76.png)

这样通过返回内容的不同，就可以进行 Bucket 名称爆破了，知道 Bucket 名称后，Key 的爆破也就很容易了。

## 3、特定的Bucket策略配置

特定的策略配置的指的是，如果管理员设置了某些IP，UA才可以请求该存储桶的话，此时如果错误的配置了GetBucketPolicy，可导致攻击者获取策略配置

![image-20220214110621814](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5cd88e09de03ea9c1900205bee28a007.png)

可以看到我们此时是没有权限访问该存储桶的，我们尝试使用aliyun的cli获取policy

![image-20220214110649046](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/b5355d42491a64b9b84c049da4d97a5d.png)

我们可以看到，需要符合UserAgent为UzJu才可以访问

![image-20220214110725348](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/0fc7ef7ffff04652d3e26258a3e18d21.png)

## 4、Bucket Object遍历

![image-20220212133217024](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/dccde60262f7392cd98cb8f7f6ef061b.png)

如果设置了ListObject，这将会导致Bucket桶被遍历

![image-20220212133307081](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c73d65f7ed7b9d1fd93a8c92c948068f.png)

可通过访问Key，来下载该文件

![image-20220212133447070](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/dfbd38a0f3c1714983e309a6d95449c0.png)

## 5、任意文件上传与覆盖

如果在配置存储桶时，管理员错误的将存储桶权限，配置为可写，这将会导致攻击者可上传任意文件到存储桶中，或覆盖已经存在的文件

![image-20220212133628804](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/7f9ed3a6780b7a852dbd063721062468.png)

![image-20220212133814727](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c236b2e8498cbc0159011d6225a6705d.png)

![image-20220212133844941](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/bb7cf912f435cf2a893854b53534c6c2.png)

如果目标的对象存储支持 html 解析，那就可以利用任意文件上传进行 XSS 钓鱼、挂暗链、挂黑页、供应链投毒等操作。

## 6、AccessKeyId，SecretAccessKey泄露

如果目标的 AccessKeyId、SecretAccessKey 泄露，那么就能获取到目标对象存储的所有权限，一般可以通过以下几种方法进行收集：

**1、**通过GitHub等开源平台中的源代码可发现存在泄露的Key

![image-20220212134350154](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/fcd87ea2857029927695fbedc13f0a93.png)

**2、**通过反编译APK，找到敏感信息

**3、**在目标网站源代码中找到（Js等）

![download_image](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f3a86a8c837a8566a49195cdbf5f37d8.png)

## 7、Bucket接管

在阿里云下，当 Bucket 显示 NoSuchBucket 说明是可以接管的，如果显示 AccessDenied 则不行。

![image-20220212134841569](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/6dcb0f8038f4fee42803f5f3bcc23898.png)

假设有以下一种情况，管理员通过域名解析并绑定了一个存储桶，但是管理员将存储桶删除后，没有将域名解析的CNAME删除，这时会访问域名就会出现上面的情况，**NoSuchBucket**。

![image-20220212135014007](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4207cd7f9d13d0e48cd23d16b5c96985.png)

![image-20220212135053519](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/900fd98e2f2455b90e67e01910b84ab2.png)

![image-20220212135121070](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/f3624f634f8baf8e010e8c2cd0dbe92f.png)

现在我们将存储桶删除，就会出现如下情况

![image-20220212135244909](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/51c244153dd0beb50e84e3d0b50bb074.png)

现在我们再访问域名会出现如下情况

![image-20220212135316238](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/912005382e38ca3923a0e47402caf3a0.png)

现在阿里云加了限制，必须在传输管理中配置绑定域名即可。以下情况即可接管该存储桶

![image-20220212135948743](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c42e918bbbdf6fcb478766f23d691ec2.png)

当我们访问存储桶的域名时，提示我们NoSuchBucket，这个时候可以登录自己的阿里云账号，创建同样的名称即可

![image-20220212140112615](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/2d174f5f9b6171bd2e5be5fc6de0de12.png)

![image-20220212140129779](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e4cf59ad5aff2e6286712a6132314f49.png)

此时我们刷新

![image-20220212140150975](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9bfb142ac87fda296013cfceafa9b796.png)

已经成功接管了该存储桶，尝试上传文件后配置权限公开访问

![image-20220212140256729](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9edf3c36090304c69f6f351eff572196.png)

## 8、Bucket 策略配置可写

当我们访问存储桶的时候，会提示我们已经被policy拦截

![image-20220214111642789](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/4ea904a7dcd5ec3fe8f74bce06f02f43.png)

如果此时配置了存储桶的oss![😛](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/8830f1b24c0d0db0a26eb6e473354458.png)utBucketPolicy，就可以更改Deny为Allow即可访问

![image-20220214111722663](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c8b75a0927f7e51621319815b700727d.png)

我们可以看到Effect中设置为Deny，我们只需要将它更改为Allow即可

![image-20220214112003820](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/1044b1a7fe10e132e769be5a19e42799.png)

随后使用PUT方法上传

![image-20220214112030036](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/bd41911f09fc8131f8897084f9a2b9a4.png)

随后我们再使用GET获取

![image-20220214112059318](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/05856c0b0ab8eac98740c98fa40ebc6f.png)

此时我们可以正常看到存储桶中的对象了

![image-20220214112115270](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/a5dddf50bc71e78fd6cc667f13ba2eae.png)

## 9、修改策略导致网站瘫痪

当策略可写的时候，除了上面的将可原本不可访问的数据设置为可访问从而获得敏感数据外，如果目标网站引用了某个 s3 上的资源文件，而且我们可以对该策略进行读写的话，也可以将原本可访问的资源权限设置为不可访问，这样就会导致网站瘫痪了。

![image-20220214114352567](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/c2c11c9b6560893bf4357fb496284262.png)

此时我们如果可以修改策略，我们只需要将获取该对象的权限修改为Deny，该网站既无法在获取图片，JS等信息了

![image-20220214114934078](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/832902d4d08ca1ef7e12261a971205fc.png)

![image-20220214114949942](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/41056a9ad79cb2f072c5aeea86af809a.png)

## 10、实战案例

我们精心挑选了来自火线安全众测项目中，漏洞奖金较高的漏洞进行举例！

### 1、阿里云存储桶劫持

![image-20220224144542698](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/e13de6b75fdf0f41a426032dfeeee31d.png)

此时可以看到访问该域名显示NoSuchBucket，那么只需要去阿里云存储桶重新创建一个与HostID一样的存储桶名称即可

![image-20220224144606258](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/b592968656bab57800a820473fac553a.png)

![image-20220224144726823](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/08215e93a0546987c20d2349fc35b75e.png)

随后只需要上传文件，就可以让该域名显示我们上传的任意文件

![image-20220224144940049](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/8164bec2022130e2592195bc067e5cf9.png)

### 2、反编译小程序，APP找到泄露的Key

![image-20220224145010457](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/71c08eaeff00f30648b64c6badc71d45.png)

![image-20220224145017646](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/7dc3bf247dd211ae43fd665947dfebb9.png)

![image-20220224145025052](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/9c128d76fb8af846fe2232ba66f9b0bf.png)

### 3、在JS文件中找到存在泄露的AccessKey

![image-20220224145140498](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/ea0cfb33edae6427f5277f8a44f98de8.png)

![image-20220224145147967](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/5b8cf9291e1a9accf2eed9658fc9bd6d.png)

![image-20220224145154082](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/dc7d438b2e1905103b0afa1da45029da.png)

![image-20220224145200090](https://cdn.jsdelivr.net/gh/halcyon098/study-img/migrated/47d3ab9d19fedbaf7b51620e5bbb2aa9.png)

## Aws S3 对象存储攻防

- TeamsSix: [https://zone.huoxian.cn/d/907-aws-s3](https://zone.huoxian.cn/d/907-aws-s3)