# 开源后端项目功能对比

| 功能                   | RuoYi | Guns | 备注                                                         |
| ---------------------- | ----- | ---- | ------------------------------------------------------------ |
| 日志异步记录           | Y     | Y    | **日志获取**：通过注解和切面实现；**异步记录**：RuoYi通过Spring ThreadPoolTaskExecutor实现，Guns通过JDK ScheduledThreadPoolExecutor实现 |
| Wrapper 封装返回结果   | N     | Y    | 例如：将“1”和“2”这种魔术返回值封装成“男”和“女”               |
| XssFilter 过滤非法字符 | Y     | Y    | 通过Filter和正则表达式过滤                                   |
| DataScope 数据范围控制 | Y     | Y    | RuoYi通过注解和切面实现，Guns通过MyBatis拦截器实现(DataScope作为mapper的入参) |
| Shiro 鉴权             | Y     | N    |                                                              |
| Security + JWT 鉴权    | N     | Y    | 登录时请求令牌(jwt token)，后续请求带上令牌，然后由Token过滤器去校验token是否有效。 |
| 验证码生成             | Y     | Y    | 通过Kaptcha创建验证码，并保存到session中，最后通过response.getOutputStream()将验证码写到前端页面 |
| 登录密码请求加密       | N     | N    |                                                              |
| 密码保存               | Y     | Y    | MD5带盐值加密保存                                            |
| ExcelUtil，Excel 注解  | Y     | Y    | RuoYi在Apache POI基础上封装Excel注解、ExcelUtil实现；Guns通过EasyPOI实现 |

