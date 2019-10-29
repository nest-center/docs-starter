# Role Based Access Control

## Description

提供基于角色的动态权限访问控制支持

## 安装

```bash
$ npm i --save @nestcenter/rbac
```

## 如何使用

```typescript
import { Module } from '@nestjs/common';
import { NEST_BOOT, NEST_CONSUL } from '@nestcenter/common';
import { BootModule } from '@nestcenter/boot';
import { ConsulModule } from '@nestcenter/consul';
import { Backend, ConsulValidator, RbacModule } from '@nestcenter/rbac';
import { HeroController } from './hero.controller';

@Module({
    imports: [
        BootModule.register(__dirname, `config.yaml`),
        ConsulModule.register({ dependencies: [NEST_BOOT] }),
        RbacModule.register({
            dependencies: [NEST_CONSUL, NEST_BOOT],
            backend: Backend.CONSUL,
            validator: ConsulValidator,
        })
    ],
    controllers: [HeroController],
})
export class AppModule {
}
```

### Boot 配置

```yaml
consul:
  host: localhost
  port: 8500
rbac:
  parameters:
    key: service-rbac
```

### RBAC 配置

RBAC 模块有三个类型：Account，Role，RoleBinding，请在 YAML 文件中 使用 '---' 分割它们，
然后在 Consul KV store 中创建一个名为 'service-rbac' 来存储 RBAC 配置。

```yaml
kind: Account
name: test

---

kind: Role
name: admin
rules:
  - resources: ["user"]
    verbs: ["get", "list"]

---

kind: RoleBinding
role: admin
accounts:
  - test
```

### 编写一个 Guard

在该 Guard 中为 request 对象赋值一个 user 对象，RbacGuard 需要该对象来做权限验证，例子如下：

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { IRbacAccount } from '@nestcenter/rbac';

@Injectable()
export class AccountGuard implements CanActivate {
    canActivate(context: ExecutionContext): boolean {
        const request = context.switchToHttp().getRequest();
        request.user = { name: request.query.user } as IRbacAccount;
        return true;
    }
}

```

### 在控制器中定义 Resource 和 Verb

注意，你编写的 Guard 必须在 RbacGuard 之前，否则会无效。

```typescript
import { Controller, Get, Param, UseGuards } from '@nestjs/common';
import { AccountGuard } from './account.guard';
import { RbacGuard, Resource, Verb, Verbs } from '@nestcenter/rbac';

@Controller('/users')
@Resource('user')
@UseGuards(AccountGuard, RbacGuard)
export class HeroController {
    @Get('/:userId')
    @Verb(Verbs.GET)
    async get(@Param('heroId') heroId: number): Promise<any> {
        return { user: 'Shadow hunter' };
    }

    @Get('/')
    @Verb(Verbs.LIST)
    async list(): Promise<any> {
        return { users: ['Shadow hunter', 'BladeMaster'] };
    }
}
```

## 如何使用自定义校验器

默认情况下，RBAC 使用 ConsulValidator 作为默认的校验器，使用 Consul KV 作为存储后端，
如果你不想使用 Consul 存储 Rbac 配置，可以通过实现 IRbacValidator 接口来自定义。例子如下：

```typescript
import { IRbacValidator } from "./interfaces/rbac-validator.interface";
import { IRbacAccount } from "./interfaces/rbac-account.interface";
import { IRbacRole } from "./interfaces/rbac-role.interface";
import { IRbacRoleBinding } from "./interfaces/rbac-role-binding.interface";
import { Store } from "./store";
import { IRbacConfig } from "./interfaces/rbac-config.interface";

export class CustomValidator implements IRbacValidator {
    private readonly store: Store = new Store();

    /**
    *
    * @param config
    * @param client
    * If set config.backend to Backend.CONSUl, the client will be consul instance;
    * if set config.backend to Backend.LOADBALANCE, the client will be loadbalance instance;
    * if set config.backend to Backend.CUSTOM or not set, the client will be null.
    */
    public async init(config: IRbacConfig, client?: any) {
        const roles: IRbacRole[] = [];
        const accounts: IRbacAccount[] = [];
        const roleBindings: IRbacRoleBinding[] = [];
        this.store.init(accounts, roles, roleBindings);
    }

    public validate(resource: string, verb: string, account: IRbacAccount): boolean {
        return this.store.validate(account.name, resource, verb);
    }
}

```

## 例子

请访问：https://github.com/nest-center/nestcenter-rbac-example
