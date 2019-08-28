## 微信公众号及微信小程序的设计与实现

1. 准备工作

    在做微信公众号及微信小程序开发前，需要做一些资料的准备

    1. 准备

        1. APPID(小程序ID)和APPSECRET（微信公众号开发者密码）

            1. 访问https://developers.weixin.qq.com，注册微信小程序开发账号

            2. 微信开发者工具：https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html

        2. MCHID(商户ID)和KEY(商户密钥)

            **涉及到支付的功能，因此才需要申请。**

            1. 访问https://pay.weixin.qq.com/static/applyment_guide/applyment_detail_public.shtml

        3. 域名

            1. 访问http://service.oray.com，注册一个账号，并购买58元的https域名，比如https://sangbill.oicp.vip

            2. 花生壳5客户端:https://hsk.oray.com/

    2. 环境搭建所需的服务

        1. maven

        2. mysql

2. 系统设计

    整个系统按照前后端分离设计，且只为做出一个通用的架构。

    1. 模块分析

        除非特别说明只实现一种，否则要实现公众号方式和小程序两种方式

        1. 用户接入

            通过公众号或小程序接入，得到用户信息(openId,nickName,address,gender等)，后面的核心业务都要围绕用户展开

        2. 商品(Demo)

        3. 订单(Demo)

        4. 支付

            支付方式多种

            1. 微信公众号支付
            2. 微信H5支付
            3. 微信扫码支付
            4. 微信小程序支付
            5. 支付宝

            系统将实现覆盖所有方式。

    2. 角色权限

        1. 管理员

            1. 登录

            2. 用户信息管理

            3. 商品管理

            4. 订单管理

        2. 用户

            1. 允许公众号或小程序接入

            2. 购买商品

                浏览->下单->支付

    3. 代码模块规划

        模块|后台|前端-PC|前端-公众号|前端-小程序
        --|--|--|--|--
        登录|√|√|||
        用户|√|√|||
        商品|√|√|√|√|√
        订单|√|√|√|√|√
        支付|√||√|√
