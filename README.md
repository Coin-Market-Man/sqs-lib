# SQS-lib for CMM
### Forked from nestjs-sqs:
Tested with: [AWS SQS](https://aws.amazon.com/en/sqs/) and [ElasticMQ](https://github.com/softwaremill/elasticmq).

Nestjs-sqs is a project to make SQS easier to use and control some required flows with NestJS.
This module provides decorator-based message handling suited for simple use.

This library internally uses [bbc/sqs-producer](https://github.com/bbc/sqs-producer) and [bbc/sqs-consumer](https://github.com/bbc/sqs-consumer), and implements some more useful features on top of the basic functionality given by them.

## Installation

```shell script
$ npm i --save @coin-market-man/sqs-lib
```

## Quick Start

### Register module

Just register this module:

```ts
@Module({
  imports: [
    SqsModule.register({
      consumers: [],
      producers: [],
    }),
  ],
})
class AppModule {}
```

Quite often you might want to asynchronously pass module options instead of passing them beforehand.
In such case, use `registerAsync()` method like many other Nest.js libraries.

- Use factory with config service (recommended in CMM)

```ts
SqsModule.registerAsync({
  useFactory: async (configService: ConfigService) =>
    configService.createSQSOptions(),
  inject: [ConfigService],
});
```

- Use class

```ts
SqsModule.registerAsync({
  useClass: SqsConfigService,
});
```

- Use existing

```ts
SqsModule.registerAsync({
  imports: [ConfigModule],
  useExisting: ConfigService,
});
```

### Decorate methods

You need to decorate methods in your NestJS providers in order to have them be automatically attached as event handlers for incoming SQS messages:

```ts
import { Message } from '@aws-sdk/client-sqs';

@Injectable()
export class AppMessageHandler {
  @SqsMessageHandler(/** name: */ 'queueName', /** batch: */ false)
  public async handleMessage(message: Message) {
  }
  
  @SqsConsumerEventHandler(/** name: */ 'queueName', /** eventName: */ 'processing_error')
  public onProcessingError(error: Error, message: Message) {
    // report errors here
  }
}
```

### Produce messages

```ts
export class AppService {
  public constructor(
    private readonly sqsService: SqsService,
  ) { }
  
  public async dispatchSomething() {
    await this.sqsService.send(/** name: */ 'queueName', {
      id: 'id',
      body: { ... },
      groupId: 'groupId',
      deduplicationId: 'deduplicationId',
      messageAttributes: { ... },
      delaySeconds: 0,
    });
  }
}
```

### Configuration

See [here](https://github.com/Coin-Market-Man/sqs-lib/blob/master/lib/sqs.types.ts), and note that we have same configuration as [bbc/sqs-consumer's](https://github.com/bbc/sqs-producer). 
In most time you just need to specify both `name` and `queueUrl` at the minimum requirements.

## License

This project is licensed under the terms of the MIT license.
