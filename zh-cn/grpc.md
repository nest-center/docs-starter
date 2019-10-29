# GRPC

对 GRPC 提供负载均衡支持

## 安装

```bash
npm install @nestcenter/grpc
```

## 使用

```typescript
import { Controller, Get } from '@nestjs/common';
import { GrpcClient, LbClient, IClientConfig, Service } from '@nestcenter/grpc';
import { HeroService } from './interfaces/hero-service.interface';
import { join } from 'path';

const grpcOptions: IClientConfig = {
    service: 'rpc-server',
    package: 'hero',
    protoPath: join(__dirname, './hero.proto'),
};

@Controller()
export class HeroController {
    @LbClient(grpcOptions)
    private readonly client: GrpcClient;
    @Service('HeroService', grpcOptions)
    private readonly heroService: HeroService;

    @Get()
    async execute(): Promise<any> {
        return await this.heroService.get({ id: 1 }).toPromise();
    }
}
```

## 例子

请访问：https://github.com/nest-center/nestcenter-grpc-example
