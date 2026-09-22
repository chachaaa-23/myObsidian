###### *REST란*
(Representational State Transfer), 자원을 이름으로 구분하여 해당 자원의 상태를 주고받는 모든 것

1. HTTP URI(Uniform Resource Identifier)를 통해 자원(Resource)을 명시하고,
2. HTTP Method(POST, GET, PUT, DELETE, PATCH 등)를 통해
3. 해당 자원(URI)에 대한 CRUD Operation을 적용하는 것을 의미합니다.

*API 란*
- 정의 및 프로토콜 집합을 사용하여 두 소프트웨어 구성 요소가 서로 통신할 수 있게 하는 메커니즘
Application Programming Interface

---
## NestJS
- **NestJS는 필수적인 라이브러리 및 편의 기능을 기본으로 포함하고 있습니다.** HTTP, 웹 소켓, 미들웨어, 가드, 예외 필터, 로깅 등 서버 동작에 필수적인 기능을 사전에 포함하고 있습니다.

비동기 방식으로 신호 처리하는 node.js
- 각 단계마다 처리해야 하는 콜백함수를 담기 위한 큐를 가짐.
- setTimeout ...
- poll 단계에서 콜백함수 수행

#### <기초 사용 단계>
1. `npm i -g @nestjs/cli`
2. `$ nest new project-name`
3. 필요한 패키지 설치 & npm run start

---
#### Flow
app.module - app.controller - app.service
-> app.module 의 `@module` 데코레이터를 통해 class 의 정보를 명시함. 
-> app.service 에서 실제 화면에 띄울 정보를 올림

 !데코레이터와 class 간의 연결이 중요.
 AppModule 이라는 root module 에서 여러 개의 모듈이 포함되있음

#### Controller
> url을 매핑, request 받아 Query, Body, Param 등을 넘김

- url 가져와서 함수를 실행. (like router)
- 각각의 명령에 맞게 decorator 등록하기
- url 을 실행하고 함수를 가져오는 것.
- ex. instgram 속의 picture Controller, DM Controller, text Controller , ...

- `$nest generate controller (nest g co)`
--> 컨트롤러를 자동으로 생성해줌. 

- @get 을 사용해서 api 라우터 생성하기
- url에 있는 id 라는 파라미터를 get 하기 원함 -> @param 사용
- id 라는 파라미터를 id 라는 argument 에 저장하기 원함

[Route 경로와 id value 참조 팁]

| 사용법              | 의미                     | 설명                                | 예제                                 |
| ---------------- | ---------------------- | --------------------------------- | ---------------------------------- |
| `/:id`           | **라우트 경로**             | URL에서 `id` 값을 가져오기 위해 사용          | `@Delete('/:id')`                  |
| `@Param('id')`   | **데코레이터 내부에서 id 값 참조** | `id` 값을 가져올 때 사용                  | `@Param('id') musicId: string`     |
| `@Param('/:id')` | ❌ **잘못된 사용법**          | `id`를 찾지 못해 `undefined`가 될 가능성 있음 | ❌ `@Param('/:id') musicId: string` |
-> `@Param('/:id')`라고 하면, **NestJS는 `/:id`라는 이름을 가진 변수를 찾게 되는데 존재하지 않음** → 오류 발생 가능.

---
#### 정보 전달 받기 (Param, Body, Query)
값을 받기 위해 요청해야 함. 이때 요청 대상이 param, Body, Query. 

##### Body
- @POST, @PUT, @PATCH 등의 HTTP 메서드에서 사용  
- 대량의 데이터 전송에 적합  
- JSON, 파일 업로드, Form 데이터 등을 포함  
example)
```typescript
@Post()
create(@Body() musicData: any) {
...
}
```

##### Param
-  URL의 일부로서, URL Path 내에 포함되는 데이터  
	예: `/movies/123`  
- 정렬이나 필터링 시 사용
- 특정 데이터를 식별하는 데 사용  
example)
```typescript
@Get('/:id')
getOne(@Param('id') musicId: string) {
...
}
```

##### Query
- 요청 주소에 포함되어있는 변수를 담음
- URL의 끝부분에 `?` 뒤에 위치하는 key-value 쌍으로 구성.  
	ex) `movies?name=tenet&director=nolan`  
	여기서 `name`과 `director`가 바로 Query key  
- 어떤 resource를 식별하고 싶을 때 사용
- 데이터를 필터링하거나 정렬하는 기준값을 전송할 때 사용
example)
```typescript
@Get('search')
search(@Query('year') searchingYear: string) {
...
}
```

---

#### Service
- 로직관리 해주는 것 파일들
- `nest generate service`
> 1. service 를 주고 받을 클래스(interface) type 을 export 하는 music.entity.ts 만들기
> 2. music.service.ts 라는 service 파일에서 musicList 들을 CRUD 할 수 있는 function들을 만들기.
> 3. music.controller.ts 에다가 @Post, @Get, @Patch, @Delete 하는 function을 가져다가 쓴다. 

[간단한 예외 처리 팁]
``` ts
getOne(id: string): Music | undefined {
	const music = this.musicList.find((music) => music.id === parseInt(id));
	if (!music) {
		throw new NotFoundException(`music with ID ${id} Not found`); //간단 예외 처리
	}
	return music;
}
```

---

#### DTO (Data Transfer Object)
- id 제외 사람들이 정보를 주고받을 때 사용할 데이터 타입

`main.ts` 에다가 `app.useGlobalPipes(new ValidationPipe()); `
> 유효성 검사하는 pipe 생성.

`npm i class-validator class-transformer`
-> class 유효성 검사 가능하도록 npm 설치.

다음과 같이 string인지, number 인지, ... 확인 가능
```ts
import { IsNumber, IsString } from 'class-validator';

export class CreateMusicDto {
	@IsString()
	readonly title: string;
	
	@IsNumber()
	readonly year: number;
	
	@IsOptional()
	@IsString({ each: true })
	readonly genre: string[];
}
```

--> POST 시 TypeScript로 실시간 코드 유효성 검사 함.

##### ValidationPipe Options ( whitelist, forbidNonWhitelisted, transform )
`new ValidationPipe({whitelist: true,forbidNonWhitelisted: true,transform: true,})`
1. whitelist : 아무 decorator 도 없는 어떠한 property 의 object 를 거름 (validation 에 도달하기도 전에 차단)
2. forbidNonWhitelisted: (request 자체를 막아버림)
3. transform: 유저에게서 받은 값 실제 코드상에서 필요로 하는 값으로 변경시킴. 

---

`npm i @nestjs/mapped-types`
- 타입을 변환시키고 사용할 수 있게 하는 package
- DTO 를 변환시키는데 사용됨 ( update 와 같은 동작들은 부분적으로 요소들을 사용할 수도 있기 때문. )

```ts
import { PartialType } from '@nestjs/mapped-types';
import { CreateMusicDto } from './create-music-dto';

export class UpdateMusicDto extends PartialType(CreateMusicDto) {}
```

---

- app.module 은 각각 하나의 AppController 와 AppService(provider) 만 지닌다.
	- App은 하위 개념으로써 여러 개의 module 을 지닌다. 
`nest generate module` 
--> app.module.ts
```ts
@Module({
	imports: [MusicListModule],
	controllers: [],
	providers: [],
})
```

> 다음과 같이 imports 에 모듈이 생긴다. 해당 모듈 안에 controller 와 provider 를 넣고, 그 모듈 자체를 import 한다!

music.service.ts 의 `@Injectable()` 을 통해 DI (Dependency Injection) 연결. 
-> music.controller가 music.service 를 필요로 함. 
-> musicList.module.ts 의 @module decorator 속의 controllers & providers 에 연결되어있음. 

