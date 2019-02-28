# seckill

## 项目介绍

> 何为秒杀？

所谓“秒杀”，就是网络卖家发布一些超低价格的商品，所有买家在同一时间网上抢购的一种销售方式。由于商品价格低廉，往往一上架就被抢购一空，有时只用一秒钟。

> 为何选择Java高并发秒杀作为实战项目？

- 秒杀业务场景具有典型事务特性
- 秒杀/红包类需求越来越常见

> 为何使用SpringMVC+Spring+MyBatis框架

- 框架易于使用和轻量级
- 低代码侵入性
- 成熟的社区和用户群

> 能从该项目得到什么收获？

- 框架的使用和整合技巧
- 秒杀分析过程与优化思路

> How to play

- 将下载的源码解压后作为Maven项目导入到IDE工具中；或者将从GitHub克隆下来的项目作为Maven项目导入到IDE工具中
- 打开项目中的jdbc.properties文件，修改里边的url,username和password
- 将项目部署到Tomcat上并启动
  - 可以直接用IDE内嵌的Tomcat启动项目
  - 或者将本项目通过**mvn clean package**命令打成war包并丢到本地安装的Tomcat的webapps目录下，接着启动Tomcat即可
- 在浏览器上访问：`http://localhost:8080/seckill`

## 开发环境

- **操作系统**：Windows 8
- **IDE工具**：Eclipse
- **JDK**：JDK1.7
- **中间件**：Tomcat 7.0
- **数据库**：MySQL 5.0
- **构建工具**：Maven
- **框架**：SSM

## 项目总结

### 数据层技术

### 业务层技术

### WEB技术

### 并发优化

## 项目截图

#### 秒杀列表

![秒杀列表](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/list.jpg?Expires=1551363049&OSSAccessKeyId=TMP.AQFLeOJiqSWC2An98cjOxZfks3_hcNsf8aHkLiZd7K8efvee1A52qpIF51O2MC4CFQCJ1SCNRdBlMDTrMVGQwEDkXWHAMwIVAP6jjwPMRpUmxtDLY0diMyVsi-0r&Signature=Tu8tGLV3YuDGiQYY8IKvJPdN4As%3D)

#### 秒杀详情页

![秒杀详情页](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/detail.jpg?Expires=1551363098&OSSAccessKeyId=TMP.AQFLeOJiqSWC2An98cjOxZfks3_hcNsf8aHkLiZd7K8efvee1A52qpIF51O2MC4CFQCJ1SCNRdBlMDTrMVGQwEDkXWHAMwIVAP6jjwPMRpUmxtDLY0diMyVsi-0r&Signature=zehmk7iaDwpuQm3Bw1TAoQuB%2Bcg%3D)

#### 错误提示

![错误提示](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/error.jpg?Expires=1551363164&OSSAccessKeyId=TMP.AQFLeOJiqSWC2An98cjOxZfks3_hcNsf8aHkLiZd7K8efvee1A52qpIF51O2MC4CFQCJ1SCNRdBlMDTrMVGQwEDkXWHAMwIVAP6jjwPMRpUmxtDLY0diMyVsi-0r&Signature=WzVoQ0Aw8aVORpnu9f0os%2F%2B6JsU%3D)

#### 开始秒杀

![开始秒杀](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/seckill.jpg?Expires=1551363364&OSSAccessKeyId=TMP.AQFLeOJiqSWC2An98cjOxZfks3_hcNsf8aHkLiZd7K8efvee1A52qpIF51O2MC4CFQCJ1SCNRdBlMDTrMVGQwEDkXWHAMwIVAP6jjwPMRpUmxtDLY0diMyVsi-0r&Signature=f%2B%2F9BSg9tdMH7FUck8ue6avL8aA%3D)

#### 秒杀成功

![秒杀成功](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/seckill-success.jpg?Expires=1551363376&OSSAccessKeyId=TMP.AQFLeOJiqSWC2An98cjOxZfks3_hcNsf8aHkLiZd7K8efvee1A52qpIF51O2MC4CFQCJ1SCNRdBlMDTrMVGQwEDkXWHAMwIVAP6jjwPMRpUmxtDLY0diMyVsi-0r&Signature=GW4WhrMPBz2b9PgrMfPxmtlfToM%3D)

#### 重复秒杀

![重复秒杀](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/repeat-seckill.jpg?Expires=1551363411&OSSAccessKeyId=TMP.AQFLeOJiqSWC2An98cjOxZfks3_hcNsf8aHkLiZd7K8efvee1A52qpIF51O2MC4CFQCJ1SCNRdBlMDTrMVGQwEDkXWHAMwIVAP6jjwPMRpUmxtDLY0diMyVsi-0r&Signature=%2BEPwpEIBP672wCQtRisKCidpBIk%3D)

#### 秒杀倒计时

![秒杀倒计时](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/countdown.jpg?Expires=1551363436&OSSAccessKeyId=TMP.AQFLeOJiqSWC2An98cjOxZfks3_hcNsf8aHkLiZd7K8efvee1A52qpIF51O2MC4CFQCJ1SCNRdBlMDTrMVGQwEDkXWHAMwIVAP6jjwPMRpUmxtDLY0diMyVsi-0r&Signature=GIjVm0UGEh2r%2FepZJz9IlgbI8B4%3D)

#### 秒杀结束

![秒杀结束](https://blogs-image.oss-cn-beijing.aliyuncs.com/seckill/stoped.jpg?Expires=1551363459&OSSAccessKeyId=TMP.AQFLeOJiqSWC2An98cjOxZfks3_hcNsf8aHkLiZd7K8efvee1A52qpIF51O2MC4CFQCJ1SCNRdBlMDTrMVGQwEDkXWHAMwIVAP6jjwPMRpUmxtDLY0diMyVsi-0r&Signature=Yrsa9AR7%2FDRDVfBld%2BtOPp7xCMA%3D)







