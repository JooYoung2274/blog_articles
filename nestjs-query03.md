## entity relation

이번에는 nestjs-query에서 지원하는 entity 관련 기능들에 대해서 알아보려고 한다.

일단 nestjs-query는 총 4개의 ORM을 지원한다. 

- TypeORM
- Sequelize
- Mongoose
- Typegoose

이중 TypeORM에 관련된 기능들을 알아볼것이다


relationName- 관계를 검색할 때 사용할 관계의 이름QueryService
nullable- true관계가 Null 허용인 경우로 설정됩니다.
complexity- 관계 복잡도를 지정하기 위해 설정합니다. 자세한 내용은 복잡성 문서를 참조하세요.
disableRead- true읽기 작업을 비활성화하려면 으로 설정합니다.
update
enabled- true업데이트 작업을 활성화하려면 으로 설정하세요.
description- 업데이트 작업에 대한 설명입니다.
complexity- 관계 복잡도를 지정하기 위해 설정합니다. 자세한 내용은 복잡성 문서를 참조하세요.
decorators=[]- 사용자 정의 배열 PropertyDecorator또는 MethodDecorators엔드포인트에 추가합니다.
remove
enabled- true제거 작업을 활성화하려면 으로 설정하세요.
description- 제거 작업에 대한 설명입니다.
complexity- 관계 복잡도를 지정하기 위해 설정합니다. 자세한 내용은 복잡성 문서를 참조하세요.
decorators=[]- 사용자 정의 배열 PropertyDecorator또는 MethodDecorators엔드포인트에 추가합니다.
allowFiltering- true관계에 대한 필터링을 허용하려면 로 설정하세요.
guards=[]- 엔드 포인트 에 추가할 경비원 배열 .updateremove
interceptors=[]- 엔드 포인트 에 추가할 인터셉터 의 배열입니다 .updateremove
pipes=[]- 엔드 포인트 에 추가할 파이프 배열입니다 .updateremove
filters=[]- 엔드 포인트 에 추가할 필터 배열입니다 .updateremove
