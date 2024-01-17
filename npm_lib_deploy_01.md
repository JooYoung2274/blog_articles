## npm 라이브러리 배포

최근에 회사에서 사용하던 라이브러리가 지원 종료된다는 얘기를 듣고 해당 기능에 적합한 라이브러리르 직접 만들어 npm에 배포할 기회가 있었다. 아직 사용하고 있지는 않지만 택배조회 관련 라이브러리(링크)와 다이렉트샌드 문자발송 라이브러리(링크)를 만들어 배포해둔 상태다.

<br>

이후에 티스토리 open api가 서비스를 종료한다는 얘기를 듣고 어떻게 우회해서 cli 환경에서 업로드할수는 없을까 생각하다가 좋은 아이디어가 생각나 구현해 배포했다. 먼저 내가 했던 npm 배포 방식은 아래와 같다.

### 1. npm 배포

#### 1-2. npm 회원가입

먼저 npm 홈페이지(링크)에서 회원가입을 진행한 후 터미널에서 로그인 해준다.

```bash
$ npm login 
# or
$ npm adduser
```

#### 1-3. 배포할 코드 작성

다른 블로그들을 보면 추천하는 폴더구조가 나와있다. 물론 추천하는 폴더구조로 진행하면 통일성있게 작업이 가능하지만 어차피 나는 CLI 환경에서 구동되는 라이브러리를 만들 예정이어서 시작 파일 경로만 잘 설정하면 별 문제없이 배포 가능했다.


먼저 아래 명령어로 package.json을 생성해준다.

```bash
$ npm init
```

그리고 아래처럼 package.json을 작성해준다. 나는 타입스크립트를 사용했기 때문에 build용 명령어와 types를 추가해줬다. 

tsconfig.json, tsconfig.build.json도 당연히 추가해줬다.

```json
// package.json
{
  "dependencies": {
    // 필요한 dependency들 목록
  },
  "name": "", // 업로드할 라이브러리 이름,
  "version": "0.0.1", // 현재 버전
  "main": "./dist/app.js", 
  "types": "./dist/app.d.ts", // 타입스크립트 빌드 후 파일
  "scripts": {
    "start": "node dist/app.js",
    "build": "tsc -p ."
  }일
  "author": "",
  "license": "ISC", // 라이센스
  "description": "",
  "bin": { // CLI에서 실행할 것이기 때문에 bin으로 명령어를 추가해 시작 경로를 설정해준다.
    "joo-upload": "dist/app.js" 
  }
}


// tsconfig.json
{
  "compilerOptions": {
    "target": "es6",
    "module": "commonjs",
    "outDir": "dist",
    "rootDir": ".",
    "strict": true,
    "moduleResolution": "node",
    "esModuleInterop": true,
    "declaration": true
  }
}


// tsconfig.build.json
{
  "extends": "./tsconfig.json",
  "exclude": ["node_modules", "test", "dist", "**/*spec.ts"]
}
```

#### 1-4. 배포

배포는 프로젝트 루트 경로에서 아래 명령어로 진행한다.

```bash
$ npm publish --access=public
```

<br>


### 2. 버전 업데이트

배포가 완료되고 이후에 기능 추가나 버그 픽스로 인해 새롭게 업데이트해야 하는 경우에는 package.json에서 버전을 올려주고 다시 npm publish --access=public 해주면 된다.

만약 버전을 변경안할 경우 업데이트 배포 자체가 안되니 꼭 변경해줘야한다. 


<br>

### 3. nestjs에서 module로 npm 배포하기

nestjs에서 제공하는 라이브러리들을 보면 거의 module을 주입해 사용하게 되는데 nestjs에서 제공하는 DynamicModule 이용해 작성하면 된다.

```typescript
import { Module, DynamicModule } from '@nestjs/common';
import { DirectSendService } from './direct-send.service';
import { DIRECT_SEND_MODULE_OPTIONS } from './direct-send.config';

@Module({})
export class DirectSendModule {
  static register(options: DIRECT_SEND_MODULE_OPTIONS): DynamicModule {
    return {
      module: DirectSendModule,
      providers: [
        {
          provide: DirectSendService,
          useValue: new DirectSendService(options),
        },
      ],
      exports: [DirectSendService],
    };
  }
}

``` 

기타 서비스 로직은 따로 작성 후 providers에 적용해주면 된다.
