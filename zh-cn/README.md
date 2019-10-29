
# NestCenter

基于 Consul 的 NodeJS 微服务解决方案，各组件使用 Typescript 语言 和 NestJS 框架编写。

## 安装

```bash
npm install --save @nestcenter/core @nestcenter/common @nestcenter/boot @nestcenter/consul @nestcenter/consul-service @nestcenter/consul-config @nestcenter/consul-loadbalance @nestcenter/feign @nestcenter/logger @nestcenter/schedule
```


## 主要组件

### [Boot](zh-cn/boot.md)

在应用启动的时候读取应用本地配置以及系统环境变量。


### [Consul](zh-cn/consul.md)

Consul 模块


### [Consul-Config](zh-cn/consul-config.md)

从配置中心读取和监听配置。


### [Consul-Service](zh-cn/consul-service.md)

服务注册和服务发现。


### [Consul-Loadbalance](zh-cn/consul-loadbalance.md)

软件实现的负载均衡，主要为 http 调用提供服务。


### [Feign](zh-cn/feign.md)

Http 客户端，支持负载均衡并且支持用 Decorator 编写。


### [Grpc](zh-cn/grpc.md)

提供 Grpc，并且支持负载均衡。


### [Memcached](zh-cn/memcached.md)

Memcached 模块。


### [Schedule](zh-cn/schedule.md)

支持分布式运行的定时任务库，支持 Decorator 编写。


### [Logger](zh-cn/logger.md)

基于 winston@2.x 实现的日志模块。

### [Validations](zh-cn/validations.md)

Http 请求参数验证模块。

### [Rbac](zh-cn/rbac.md)

基于角色的动态权限访问控制

## 快速开始

main.ts

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { NestLogger } from '@nestcenter/logger';
import { NestCenter } from '@nestcenter/core';

async function bootstrap() {
    const app = NestCenter.create(await NestFactory.create(AppModule, {
        logger: new NestLogger({
            path: __dirname,
            filename: `config.yaml`
        }),
    }));

    await app.listen(NestCenter.global.boot.get('consul.service.port', 3000));
}

bootstrap();
```

app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { NEST_BOOT, NEST_CONSUL_LOADBALANCE, NEST_CONSUL_CONFIG } from '@nestcenter/common';
import { BootModule } from '@nestcenter/boot';
import { ConsulModule } from '@nestcenter/consul';
import { ConsulConfigModule } from '@nestcenter/consul-config';
import { ConsulServiceModule } from '@nestcenter/consul-service';
import { LoadbalanceModule } from '@nestcenter/consul-loadbalance';
import { FeignModule } from '@nestcenter/feign';
import { LoggerModule } from '@nestcenter/logger';
import { TerminusModule } from '@nestjs/terminus';

@Module({
    imports: [
        LoggerModule.register(),
        BootModule.register(__dirname, `config.yaml`),
        ConsulModule.register({ dependencies: [NEST_BOOT] }),
        ConsulConfigModule.register({ dependencies: [NEST_BOOT] }),
        ConsulServiceModule.register({ dependencies: [NEST_BOOT] }),
        LoadbalanceModule.register({ dependencies: [NEST_BOOT] }),
        FeignModule.register({ dependencies: [NEST_BOOT, NEST_CONSUL_LOADBALANCE] }),
        TerminusModule.forRootAsync({
            useFactory: () => ({ endpoints: [{ url: '/health', healthIndicators: [] }] }),
        }),
    ]
})
export class AppModule {
}
```


## Starter

使用 [NestCenter-Starter](https://github.com/nest-center/nestcenter-starter) 快速开始创建你的项目.


## 例子

[Samples](https://github.com/nest-center/nestcenter/samples)

## 谁在使用 NestCenter

![cxl](https://nestcenter.org/_media/who-used/cxl.svg)


## 保持联系

- Author - [miaowing](https://github.com/miaowing)

## License

  NestCenter is [MIT licensed](https://github.com/nest-center/nestcenter/LICENSE).
