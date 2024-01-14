## nestjs-query_02



### 1. BeforeCreateOneHook, BeforeUpdateOneHook

nestjs-query에서는 몇 가지 훅들을 제공한다. 그 중 가장 대표적으로 BeforeCreateOneHook과 BeforeUpdateOneHook이다. 해당 훅들은 module에서 자동으로 만들어 주는 CreateOne, UpdateOne 전에 실행되게 사용할 수 있는 기능이다. resolver에 도달하기 전에 실행되기 때문에 nestjs의 interceptor라고 봐도 된다.





### 2. 사용법


#### 2-1. BeforeCreateOneHook 사용법

아래 보이는 것처럼 BeforeCreateOneHook<ENTITY>를 사용해 구현할 수 있다.
<br>

instance는 자동 생성되는 createOne의 input query를 받아오고 context는 GqlContext를 받아온다.

```typescript
import { Injectable } from '@nestjs/common';
import { BoardEntity } from '../board.entity';
import {
  BeforeCreateOneHook,
  CreateOneInputType,
} from '@ptc-org/nestjs-query-graphql';

export type GqlContext = {
  req: {
    headers: Record<string, string>;
    connection?: any;
    ip?: string;
  };
};

@Injectable()
export class CreatedByHook implements BeforeCreateOneHook<BoardEntity> {
  async run(
    instance: CreateOneInputType<BoardEntity>,
    context: GqlContext,
  ): Promise<any> {
    const input = instance.input;
    const ip = context.req.headers['x-user-id'] || context.req.ip;
    return { input, ip };
  }
}

```


#### 2-2. BeforeUpdateOneHook 사용법

BeforeUpdateOneHook도 BeforeCreateOneHook과 동일한 방식으로 사용 가능하다.
```typescript
import { Injectable } from '@nestjs/common';
import { BoardEntity } from '../board.entity';
import {
  BeforeUpdateOneHook,
  UpdateOneInputType,
} from '@ptc-org/nestjs-query-graphql';

export type GqlContext = {
  req: {
    headers: Record<string, string>;
    connection?: any;
    ip?: string;
  };
};

@Injectable()
export class UpdatedByHook implements BeforeUpdateOneHook<BoardEntity> {
  async run(
    instance: UpdateOneInputType<BoardEntity>,
    context: GqlContext,
  ): Promise<any> {
    const update = instance.update;
    const ip = context.req.headers['x-user-id'] || context.req.ip;
    return { update, ip };
  }
}
```

### 3. Hook 종류들

이 두개 외에도 아래와 같은 Hook을 제공한다. 

@BeforeFindOne - invoked before any findOne query.
@BeforeQueryMany - invoked before any queryMany query.
@BeforeCreateOne - invoked before any createOne mutation.
@BeforeCreateMany - invoked before any createMany mutation.
@BeforeUpdateOne - invoked before any updateOne mutation.
@BeforeUpdateMany - invoked before any updateMany mutation.
@BeforeDeleteOne - invoked before any deleteOne mutation.
@BeforeDeleteMany - invoked before any deleteMany mutation.


### 4. Authorize

nestjs에서는 인증에 대한 기능도 제공한다. nestjs-query01(링크)에서 설정한 module에서 guards 옵션을 통해서 설정할수도 있고, entity에 @Authorize()를 통해 자동생성되는 CRUD에 대한 인증을 설정해줄수도 있다

#### 4-1. entity에 Authorize 적용하는 방법

```typescript
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
// 아래처럼 @Authorize()를 추가해 자동생성 되는 모든 CRUD에 인증 관련 기을 추가해줄 수 있다.
@Authorize({ authorize: (context: UserContext) => ({ userId: { eq: context.req.user.id } }) })
// BoardEntity와 Relation에 UserDTO가 있을 때
@FilterableRelation('owner', () => UserDTO)
@ObjectType('Board')
export class BoardEntity {
  @PrimaryGeneratedColumn()
  @IDField(() => ID) // 기존 graphql 처럼 하나의 엔티티에서 DTO도 선능언 가능함
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
```