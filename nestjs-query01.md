## nestjs-query

nestjs-query는 graphql의 crud를 자동으로 만들어주고 기타 기능들을 제공해주는 라이브러리다. 기존 graphql의 경우 손쉽게 서버를 구성하고 사용할 수 있었지만 querying, sorting, paging기능을 구현하고, dto와 같은 타입들을 모두 지정해줘야 하기 때문에 중복된 일을 하는 경우가 많았다. nestjs-query는 이런 문제들을 단순 설정을 통해 쉽게 세팅할 수 있게 해준다.


### 1. @ptc-org/nestjs-query

구글에 nestjs-query를 치면 https://doug-martin.github.io/nestjs-query/ 이 주소가 제일 상단에 나온다. 

해당 라이브러리는 0.3버전 이후 업데이트가 멈춰진 상태이기 때문에 https://tripss.github.io/nestjs-query/ 해당 주소로 접속하면 최근까지도 계속 유지보수하고 있는 nestjs-query를 볼 수 있다.





### 2. nestjs-query 기본 세팅
먼저 nest new project-name을 통해 기본 세팅완료 후 아래 순서대로 디펜던시들을 설치한다.

#### 2-1. 디펜던시 추가

```bash 
npm i @ptc-org/nestjs-query-core @nestjs/common class-transformer

npm i @ptc-org/nestjs-query-graphql @nestjs/common @nestjs/graphql graphql graphql-subscriptions class-transformer class-validator dataloader

npm i @ptc-org/nestjs-query-typeorm @nestjs/common @nestjs/typeorm class-transformer typeorm

npm i pg apollo-server-express
```


#### 2-2. app.module.ts

app.module.ts에 GraphqlModule을 주입해준다.
```typescript
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { DatabaseModule } from './database/database.module';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { ConfigModule } from '@nestjs/config';
import { BoardModule } from './domain/board/board.module';

@Module({
  imports: [
    ConfigModule.forRoot(),
    DatabaseModule.forRoot(),
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      // set to true to automatically generate schema
      autoSchemaFile: true,
    }),
    BoardModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

#### 2-3. board.module.ts

nest 명령어로 board.module.ts 생성 후 아래처럼 세팅해준다.

```typescript
// board.module.ts

import { Module } from '@nestjs/common';
import { BoardController } from './board.controller';
import { BoardService } from './board.service';
import {
  NestjsQueryGraphQLModule,
  PagingStrategies,
} from '@ptc-org/nestjs-query-graphql';
import { NestjsQueryTypeOrmModule } from '@ptc-org/nestjs-query-typeorm';
import { BoardEntity } from './board.entity';

import { BoardCreateDTO } from './dto/create-board.dto';
import { BoardUpdateDTO } from './dto/update-board.dto';

@Module({
  imports: [
    NestjsQueryGraphQLModule.forFeature({
      // import the NestjsQueryTypeOrmModule to register the entity with typeorm
      // and provide a QueryService
      imports: [NestjsQueryTypeOrmModule.forFeature([BoardEntity])],
      services: [BoardService],
      // describe the resolvers you want to expose
      resolvers: [
        {
          EntityClass: BoardEntity,
          DTOClass: BoardEntity,
          CreateDTOClass: BoardCreateDTO,
          UpdateDTOClass: BoardUpdateDTO,
          ServiceClass: BoardService,
          enableAggregate: true,
          pagingStrategy: PagingStrategies.OFFSET,
          read: {},
          create: {},
          update: { one: { disabled: true } },
          delete: { many: { disabled: true } },
          guards: [],
        },
      ],
    }),
  ],
  controllers: [BoardController],
  providers: [BoardService],
})
export class BoardModule {}

```

#### 2-4. board.service.ts

nestjs-query는 기본적인 CRUD를 제공하기 때문에 따로 resolver를 만들 필요는 없다. 

하지만 service는 따로 선언해서 모듈에 선언해줘야 한다. 아래 코드를 보면 TypeOrmQeuryService를 상속받아 세팅하게 된다. 

```typescript
// board.service.ts
import { Injectable } from '@nestjs/common';
import { BoardEntity } from './board.entity';
import { TypeOrmQueryService } from '@ptc-org/nestjs-query-typeorm';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { QueryService } from '@ptc-org/nestjs-query-core';

@Injectable()
@QueryService(BoardEntity)
export class BoardService extends TypeOrmQueryService<BoardEntity> {
  constructor(@InjectRepository(BoardEntity) repo: Repository<BoardEntity>) {
    super(repo, { useSoftDelete: true });
  }

  async getBoardList(): Promise<BoardEntity[]> {
    return await this.repo.find();
  }
}

```

#### 2-5. 기타 파일 들

```typescript
// board.entity.ts
import { Field, GraphQLISODateTime, ID, ObjectType } from '@nestjs/graphql';
import { FilterableField, IDField } from '@ptc-org/nestjs-query-graphql';
import {
  Column,
  CreateDateColumn,
  Entity,
  PrimaryGeneratedColumn,
  UpdateDateColumn,
} from 'typeorm';

@Entity()
@ObjectType('Board')
export class BoardEntity {
  @PrimaryGeneratedColumn()
  @IDField(() => ID) // 기존 graphql 처럼 하나의 엔티티에서 DTO도 선언 가능함
  id!: number;

  @Column()
  @FilterableField()
  title!: string;

  @Column()
  @FilterableField()
  description!: string;

  @CreateDateColumn()
  @Field(() => GraphQLISODateTime)
  created!: Date;

  @UpdateDateColumn()
  @Field(() => GraphQLISODateTime)
  updated!: Date;
}


// create-board.dto.ts
import { Field, InputType } from '@nestjs/graphql';
import { IsBoolean, IsString } from 'class-validator';

@InputType('CreateBoard')
export class BoardCreateDTO {
  @IsString()
  @Field()
  title!: string;

  @IsBoolean()
  @Field()
  description!: boolean;
}

// update-board-dto.ts
import { Field, InputType } from '@nestjs/graphql';
import { BeforeUpdateOne } from '@ptc-org/nestjs-query-graphql';
import { IsBoolean } from 'class-validator';
import { UpdatedByHook } from './nestjs-query-before-update-one.hook';

@InputType('UpdateBoard')
@BeforeUpdateOne(UpdatedByHook)
export class BoardUpdateDTO {
  @IsBoolean()
  @Field()
  description!: boolean;
}
```

여기 까지 마무리하면 기본적인 CRUD 기능들은 사용할 수 있게 된다. 


### 3. module 옵션 

nestjs-query는 단수, 복수의 세팅이나 guards, pagination, aggregate등의 세팅이 가능하다.

먼저 아래는 board.module.ts의 resolver[] 부분 이다.

```typescript
resolvers: [
        {
          EntityClass: BoardEntity,
          DTOClass: BoardEntity,
          CreateDTOClass: BoardCreateDTO,
          UpdateDTOClass: BoardUpdateDTO,
          ServiceClass: BoardService,
          enableAggregate: true,
          pagingStrategy: PagingStrategies.OFFSET,
          read: {},
          create: {},
          update: { one: { disabled: true } },
          delete: { many: { disabled: true } },
          guards: [],
        },
      ],
```
아래 read, create, update, delete 옵션을 볼 수 있다. 해당 옵션에서 한번에 아래 코드를 통해 단수, 복수 둘다 사용하지 못하게 설정할 수 있고, 

 *read: { disabled: true }* 
 
아래 코드를 통해 단수, 복수 둘 중 하나만 사용하지 못하게 설정할 수 있다.
 
 *read: { one : { disabled: true} }* 
 
 

또한 아래 코드를 통해 따로 코드에 선언해둔 guard기능들을 통채로 적용할 수도 있고,

*guards: [MYGUARD]*

아래 처럼 read, create, update, delete 내부에 guards를 선언해 특정 기능에만 추가할 수 도 있다. 

*read : { guards: [MYGUARD] }*



<br><br>
나머지 기능들에 대한 설명은 다음 글에서