Blink 不仅仅是一个 Web 框架，也是一个 Web 服务器，可以脱离 php-fpm、Apache 独立运行，其底层基于 Swoole 的 http server，性能有保障。

Blink 为构建 Web 应用程序提供简洁优雅的API，高可扩展性，允许开发者非常灵活自如的使用，为构建高性能 API 服务而生。

主要特性：

- 依赖注入与服务定位器
- 路由，良好的 Restful 支持
- 登录与认证机制
- Session 管理
- Request 与 Response 中间件
- 良好的测试支持

创建 Blink 项目：

```
composer create-project --prefer-dist blink/seed your-app
```

中文文档：[https://github.com/bixuehujin/blink/blob/master/docs/zh-CN/README.md](https://github.com/bixuehujin/blink/blob/master/docs/zh-CN/README.md) 