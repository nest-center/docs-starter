# 日志

Logger 提供日志功能，基于 winston@2 实现。

## 安装

```bash
npm install winston@2 @nestcenter/logger --save
```

## 使用

```typescript
import { NestFactory } from '@nestjs/core'
import { NestLogger, Logger } from '@nestcenter/logger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
      logger: new NestLogger({path: __dirname, filename: 'logger.yml'})
  });
}
```

## 配置

```yaml
logger:
  level: info
  transports:
    - transport: console
      level: debug
      colorize: true
      datePattern: YYYY-MM-DD h:mm:ss
      label: user-service
    - transport: file
      name: info
      filename: info.log
      datePattern: YYYY-MM-DD h:mm:ss
      label: user-service
      # 100M
      maxSize: 104857600
      json: false
      maxFiles: 10
    - transport: dailyRotateFile
      filename: info.log
      datePattern: YYYY-MM-DD-HH
      zippedArchive: true
      maxSize: 20m
      maxFiles: 14d
```

## 注入 logger

```typescript
import { Module } from '@nestjs/common';
import { LoggerModule, Logger } from '@nestcenter/logger';

@Module({
  imports: [
      LoggerModule.register()
  ],
})
export class ApplicationModule {}
```

```typescript
import { Injectable } from '@nestjs/common';
import { InjectLogger, Logger } from '@nestcenter/logger';
import { ILogger } from '@nestcenter/common';

@Injectable()
export class TestService {
  constructor(@InjectLogger() private readonly logger: ILogger) {}

  log() {
      this.logger.info('The first log');
  }
}
```

## 自定义 Typeorm 日志

```typescript
import { Module } from '@nestjs/common';
import { TypeormLogger } from '@nestcenter/logger';
import { TypeOrmModule } from '@nestjs/typeorm';
import { NEST_TYPEORM_LOGGER_PROVIDER } from '@nestcenter/common';

@Module({
  imports: [
      TypeOrmModule.forRootAsync({
          useFactory: (logger: TypeormLogger) => ({
              /* ... */
              logger,
          }),
          inject: [NEST_TYPEORM_LOGGER_PROVIDER],
      })
  ],
})
export class ApplicationModule {}
```
