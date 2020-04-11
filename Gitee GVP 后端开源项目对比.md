# 开源后端项目功能对比

| 功能                           | RuoYi | Guns | Pig    | SpringBlade | cloud-platform | 备注                                                         |
| ------------------------------ | ----- | ---- | ------ | ----------- | -------------- | ------------------------------------------------------------ |
|                                | 单体  | 单体 | 分布式 | 分布式      | 分布式         |                                                              |
| Shiro 鉴权                     | Y     | N    | N      |             |                |                                                              |
| Security + JWT 鉴权            | N     | Y    | N      |             |                | 登录时请求令牌(jwt token)，后续请求带上令牌，然后由Token过滤器去校验token是否有效。 |
| Security + OAuth2              | N     | N    | Y      |             |                | 通过OAuth2的密码模式登录                                     |
| 日志管理（业务日志和登录日志） | Y     | Y    | Y      |             |                | **日志获取**：通过注解和切面实现；**异步记录**：RuoYi通过Spring ThreadPoolTaskExecutor实现；Guns通过JDK ScheduledThreadPoolExecutor实现；Pig通过在切面中发送ApplicationEvent事件，再通过EventListener注解监听和Async注解异步处理 |
| Wrapper 封装返回结果           | N     | Y    | N      |             |                | 例如：将“1”和“2”这种魔术返回值封装成“男”和“女”               |
| XssFilter 过滤非法字符         | Y     | Y    | N      |             |                | 通过Filter和正则表达式过滤                                   |
| DataScope 数据范围控制         | Y     | Y    | Y      |             |                | RuoYi通过注解和切面实现，Guns和Pig通过MyBatis拦截器实现(DataScope作为mapper的入参) |
| 验证码生成                     | Y     | Y    | Y      |             |                | 通过[Kaptcha](https://code.google.com/archive/p/kaptcha/)创建验证码，并保存到session或Redis中，最后通过response.getOutputStream()将验证码写到前端页面。**如果是分布式环境，建议搭配SpringSession+Redis，将session保存到Redis中** |
| 登录密码请求加密               | N     | N    | Y      |             |                | Pig通过AES非对称加密，前端传入加密后的密码，后端解密，避免密码在登录时泄露 |
| 密码保存                       | Y     | Y    | Y      |             |                | RuoYi和Guns是MD5带盐值加密保存，**Pig是使用SpringSecurity的BCryptPasswordEncoder加密保存** |
| ExcelUtil，Excel 注解          | Y     | Y    | N      |             |                | RuoYi在Apache POI基础上封装Excel注解、ExcelUtil实现；**Guns通过[EasyPOI](https://gitee.com/lemur/easypoi)实现** |
|                                |       |      |        |             |                |                                                              |
|                                |       |      |        |             |                |                                                              |

