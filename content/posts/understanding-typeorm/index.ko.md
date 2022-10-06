---
title: "TypeORM을 이해하고 사용하기"
subtitle: "공식 문서를 통해 TypeORM을 이해하고 사용해보자."
date: 2022-10-01T12:00:00+09:00
lastmod: 2020-01-01T16:45:40+08:00
draft: false
author: "EastAsh"
authorLink: "https://github.com/LouiMinister"
description: "TypeORM 사용 방법을 알아보자."
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["Node", "TypeORM"]
categories: ["Node BackEnd"]

hiddenFromHomePage: false
hiddenFromSearch: false
twemoji: false
lightgallery: true
ruby: true
fraction: true
fontawesome: true
linkToMarkdown: true
rssFullText: false
---
<!--more-->
공식문서 [typeorm.io](http://typeorm.io) 를 통해 글을 작성하였다.

## 0. 숲 그리기

---

처음 TypeORM을 사용할 때는 방대한 내용에 머리가 어지러울 수 있다. 숲을 그리고 나무를 살펴보자!

1. **시작하기**
   - TypeORM을 설치하고, 셋팅하는 간단한 방법을 알아보자.
2. **Table-Model 매핑하기**
   - ORM은 DB 테이블을 Application 측의 Model과 1대1로 매핑해서 사용하는 것이다.
   - TypeORM은 어떻게 Table과 Model을 매핑하는지 알아보자.
3. **DataSource를 생성하기**
   - TypeORM은 DB와 연관되므로, DB 관련 connection 설정과, TypeORM 관련 설정이 필요하다. TypeORM은 DataSource를 통해 관리하므로, 이를 알아보자.
4. **EntityManager vs Repository**
   - TypeORM을 사용하여 DB Operation(작업)을 하는 방법은 크게 두가지가 있다.
   - 어떤 차이가 있는지, 어떤 방법을 사용해서 Operation을 하는지 알아보자.
5. **Repository를 통해 CRUD 작업하기**
   - DB CRUD Operation 작업을 Repository를 통해 하는 방법을 알아보자.
6. **연관/관계 테이블 다루기**
   - DB를 다룰 때도 연관 테이블을 다루는 것이 큰 산이다. TypeORM에서는 연관/관계 테이블을 생성하고 Operation을 진행하는지 알아보자.
7. **QueryBuilder 사용하기**
   - 복잡한 Operation은 직접 쿼리문을 작성하여 사용하고 싶을 경우가 생긴다. 그 부분을 QueryBuilder을 통해 해소해보자.


## 1. 시작하기

---

CLI를 이용하여 새로운 프로젝트를 생성하려면 다음 명령어를 실행시킨다.

> 이미 존재하는 Node 프로젝트에서도 사용할 수 있지만, 어떤 파일들을 덮어쓰기 할 수 있으니 조심해야 한다.
>
>
> > You can generate an ESM project by running
> npx typeorm init --name MyProject --database postgres --module esm
> >
>
> > You can generate an even more advanced project with express installed by running
> npx typeorm init --name MyProject --database mysql --express
> >
>
> > You can generate a docker-compose file by running
> npx typeorm init --name MyProject --database postgres --docker
> >

```shell
npx typeorm init --name MyProject --database postgres
```

사용할 데이터 베이스 이름을 넣어준다.
> 사용 가능한 데이터베이스들
`mysql`, `mariadb`, `postgres`, `cockroachdb`, `sqlite`, `mssql`, `sap`, `spanner`, `oracle`, `mongodb`, `cordova`, `react-native`, `expo`, `nativescript`.


\
생성 시 다음과 같은 디렉터리 구조를 가진 프로젝트가 생성된다.

```
MyProject
├── src                   // 타입스크립트 코드들이 존재하는 폴더
│   ├── entity            // entity들이 존재하는 폴더
│   │   └── User.ts       // sample entity
│   ├── migration         // migration들이 존재하는 폴더
│   ├── data-source.ts    // data source 와 connection 설정 파일
│   └── index.ts          // applicatoin의 start point
├── .gitignore            // gitignore 파일
├── package.json          // node module dependencies
└── tsconfig.json         // TypeScript compiler options
```

npm i를 통해 의존 라이브러리들을 설치한다.

```shell
cd MyProject
npm install
```

data-source.ts 파일을 수정하여 database connection 설정을 한다.

```ts
exportconst AppDataSource =new DataSource({
type: "postgres",
    host: "localhost",
    port: 5432,
    username: "test",
    password: "test",
    database: "test",
    synchronize:true,
    logging:true,
    entities: [Post, Category],
    subscribers: [],
    migrations: [],
})
```

가장 중요한 옵션은 `host`, `username`, `password`, `database`, `port` 등이다.

설정을 마쳤다면, 다음 명령어를 통해 어플리케이션을 실행한다.

```
npm start
```

이제 프로젝트는 성공적으로 실행될 것이고, 새로운 user을 당신의 데이터 베이스에 넣을 것이다. 그리고 계속 프로젝트를 이어나가기 위해서는, 엔티티들을 더 생성하는것을 시작해야 한다.

## 2. Table - Model 매핑하기

---

### ✔️ Model 생성하기

> TypeORM을 이용하여 데이터베이스를 이용하기 위해서는 `model`을 통해 `database table`을 만들어야 한다.
>

예를 들어 Photo 모델이 있다고 하자

```tsx
export class Photo {
    id: number
    name: string
    description: string
    filename: string
    views: number
    isPublished: boolean
}
```

만약 Photo를 database에 저장하고 싶다면, `database table` 이 필요할 것이다. `database table`은 `model`로부터 생성할 수 있는데, 이러한 모델을 `Entity`라고 한다.

### ✔️ Entity 생성하기

`Entity` 는 `@Entity` 로 데코레이팅 된 모델이다. 데이터베이스 테이블들은 `Entity`를 통해 만들어진다.

`Entity`를 통해 `CRUD operation` 작업을 할 수 있다

```tsx
import { Entity } from "typeorm"

@Entity()
export class Photo {
    id: number
    name: string
    description: string
    filename: string
    views: number
    isPublished: boolean
}
```

이제 `database table`은 Photo `Entity`를 통해  생성될 것이다.

하지만 column이 없는 database table은 없으므로 이번에는 column을 만들어보자

### ✔️ Table Column 추가하기

`column`을 추가하기 위해서는, `entity`의 속성을 `@Column` 으로 데코레이팅 해주면 된다.

```tsx
import { Entity, Column } from "typeorm"

@Entity()
export class Photo {
    @Column()
    id: number

    @Column()
    name: string

    @Column()
    description: string

    @Column()
    filename: string

    @Column()
    views: number

    @Column()
    isPublished: boolean
}
```

이제 `Entity`의 속성들이 photo 테이블의 `coluimn`로 추가되었다.

`컬럼의 타입`은 Entity 각각의 `속성의 타입`으로 정해진다.
ex ) number → integer, string → varchar, boolean → bool

`@Column` 데코레이터에 직접 명시해줌으로써 `컬럼의 타입`을 정할 수도 있다.

#### ✔️ Primary Column 생성하기

모든 `Entity`에는 최소한 하나의 `Primary Key`가 존재햐아 한다.

`@PrimaryColumn` 데코레이터로 `primary key` column을 만들 수 있다.

```tsx
import { Entity, Column, PrimaryColumn } from "typeorm"

@Entity()
export class Photo {
    @PrimaryColumn()
    id: number
	...
}
```

#### ✔️ Auto-generated column 생성하기

`auto-generated` 컬럼 (auto-increment / sequence / serial / generated identity column) 을 생성하고 싶다면, `@PrimaryColumn` 데코레이터를 사용한다.

```tsx
import { Entity, Column, PrimaryColumn } from "typeorm"

@Entity()
export class Photo {
    @PrimaryGeneratedColumn()
    id: number
	...
}
```

#### ✔️ Column의 데이터 타입 명시하기

`@Column` 데코레이터의 인자를 통해 `데이터 타입`을 명시할 수 있다.

```tsx
import { Entity, Column, PrimaryGeneratedColumn } from "typeorm"

@Entity()
export class Photo {
    @PrimaryGeneratedColumn()
    id: number

    @Column({
        length: 100,
    })
    name: string

    @Column("text")
    description: string

    @Column()
    filename: string

    @Column("double")
    views: number

    @Column()
    isPublished: boolean
}
```

Default로, string → varchar(255), number → integer-like 타입으로 변환이 된다.

Column의 타입은 데이터베이스 명세에 따르는데, 더 많은 정보는 [이곳](https://typeorm.io/entities#column-types)을 참고한다.

## 3. DataSource 생성하기

---

Database에 관련된 설정들은 DataSource을 통해 할 수 있다.

```tsx
import "reflect-metadata"
import { DataSource } from "typeorm"
import { Photo } from "./entity/Photo"

const AppDataSource = new DataSource({
    type: "postgres",
    host: "localhost",
    port: 5432,
    username: "root",
    password: "admin",
    database: "test",
    entities: [Photo],
    synchronize: true,
    logging: false,
})

// to initialize initial connection with the database, register all entities
// and "synchronize" database schema, call "initialize()" method of a newly created database
// once in your application bootstrap
AppDataSource.initialize()
    .then(() => {
        // here you can start to work with your database
    })
    .catch((error) => console.log(error))
```

> 주의! synchronize 옵션을 걸어놓으면, application이 실행할 때 마다 entity을 통해 동기화된다! (테이블이 덮어씌워진다)
>

AppDataSource.initialize().then()에 callback으로 database를 이용하는 코드들을 작성한다.

#### ✔️ Express의 App.ts 경우

```tsx
import { AppDataSource } from "./data-source"

AppDataSource.initialize().then(async () => {
	...
    const app = express()
    app.listen(3000)
	...
}).catch(error => console.log(error))
```

application 시작에 AppDataSource.initialize()를 통해 데이터베이스와 연결한다.

## 4. `EntityManager` VS `Repository`

---

새로운 photo를 데이터베이스에 Insert하기

```tsx
import { Photo } from "./entity/Photo"
import { AppDataSource } from "./index"

const photo = new Photo()
photo.name = "Me and Bears"
photo.description = "I am near polar bears"
photo.filename = "photo-with-bears.jpg"
photo.views = 1
photo.isPublished = true

await AppDataSource.manager.save(photo)
console.log("Photo has been saved. Photo id is", photo.id)
```

entity를 save()하고 나면 id 값을 생성하여 entity 객체를 수정하고 데이터베이스에 저장한다.

save()가 반환하는 객체는 인자로 넘겨준 entity와 같은 객체이다. (id만 수정됨)

### ✔️ `EntityManager` 을 이용하기

위의 예제에서는 `EntityManager`을 이용하여 Entity를 다루었었다.

```tsx
import { Photo } from "./entity/Photo"
import { AppDataSource } from "./index"

const savedPhotos = await AppDataSource.manager.find(Photo)
console.log("All photos from the db: ", savedPhotos)
```

- `EntityManager` 을 이용하면, 아무 Entity나 CRUD 작업을 할 수 있다.
- `EntityManager`는 모든 entity의 repository 들의 집합처럼 여기면 된다.

### ✔️ `Repository` 를 이용하기

> Entity를 다루는 일이 많을 경우, `Repository`가 `EnityManager`보다 유리하다.
>

`EntityManager` 대신 `Repository`를 사용해보자.

```tsx
import { User } from "./entity/User"

const userRepository = dataSource.getRepository(User)
const user = await userRepository.findOneBy({
    id: 1,
})
user.name = "Umed"
await userRepository.save(user)
```

- `Repository` 는 Entity와 1대1로 대응되도록 만들어지고, Entity를 다루는 모든 operation을 `Repository`에서 한다.
- Repositry는 operation이 특정 entity에만 제한된 EntityManager와 같다.

## 5. Reopitory를 통해 CRUD 작업하기

---

### 1. 조회하기

- **`find`**
- `findOneBy`
- `findBy`
- `findAndCount`
- Ex Code

    ```tsx
    import { Photo } from "./entity/Photo"
    import { AppDataSource } from "./index"
    
    const photoRepository = AppDataSource.getRepository(Photo)
    const allPhotos = await photoRepository.find()
    console.log("All photos from the db: ", allPhotos)
    
    const firstPhoto = await photoRepository.findOneBy({
        id: 1,
    })
    console.log("First photo from the db: ", firstPhoto)
    
    const meAndBearsPhoto = await photoRepository.findOneBy({
        name: "Me and Bears",
    })
    console.log("Me and Bears photo from the db: ", meAndBearsPhoto)
    
    const allViewedPhotos = await photoRepository.findBy({ views: 1 })
    console.log("All viewed photos: ", allViewedPhotos)
    
    const allPublishedPhotos = await photoRepository.findBy({ isPublished: true })
    console.log("All published photos: ", allPublishedPhotos)
    
    const [photos, photosCount] = await photoRepository.findAndCount()
    console.log("All photos: ", photos)
    console.log("Photos count: ", photosCount)
    ```


### 2. 수정하기

**`조회하기 + 엔티티 수정하기 + 저장하기`**

```tsx
import { Photo } from "./entity/Photo"
import { AppDataSource } from "./index"

const photoRepository = AppDataSource.getRepository(Photo)
const photoToUpdate = await photoRepository.findOneBy({
    id: 1,
})
photoToUpdate.name = "Me, my friends and polar bears"
await photoRepository.save(photoToUpdate)
```

### 3. 삭제하기

- `remove`

```tsx
import { Photo } from "./entity/Photo"
import { AppDataSource } from "./index"

const photoRepository = AppDataSource.getRepository(Photo)
const photoToRemove = await photoRepository.findOneBy({
    id: 1,
})
await photoRepository.remove(photoToRemove)
```

### 4. 추가하기

- `save`

```tsx
const photo = new Photo()
photo.name = "Me and Bears"
photo.description = "I am near polar bears"
photo.filename = "photo-with-bears.jpg"
photo.views = 1
photo.isPublished = true

const photoRepository = AppDataSource.getRepository(Photo)

await photoRepository.save(photo)
```

## 6. 연관/관계 테이블 다루기

---

### ✔️ 1대1 관계 생성하기

```tsx
import {
    Entity,
    Column,
    PrimaryGeneratedColumn,
    OneToOne,
    JoinColumn,
} from "typeorm"
import { Photo } from "./Photo"

@Entity()
export class PhotoMetadata {
		...

    @OneToOne(() => Photo)
    @JoinColumn()
    photo: Photo
}
```

- 다른 `Entity`와 `1대1 관계`를 설정하기 위해서는 `@OneToOne` 데코레이터를 사용해야 한다.
  - `@OneToOne` 데코레이터의 인자로 `() ⇒ Photo`  를 줌으로써 관계를 맺을 Entity의 클래스를 설정한다.
  - ( 가독성 향상을 위해 `(type) ⇒ Photo` 로 명시할 수도 있다. )
  - 언어의 스펙으로 인해 Class 자체를 인자로 보내지 않고 Arrow Function을 인자로 보낸다.
- 관계를 소유하는 Entity를 설정하기 위해 `@JoinColumn` 데코레이터를 사용한다.
  - 관계는 양방향 또는 단방향이 될 수 있다.
  - 한쪽만 관계를 소유할 수 있다.

> 관계를 소유하는 쪽은 1 : n 일 때 n 쪽이 되어야 한다. (Foreign Key를 가진 쪽)
1이 되어버리면 하나의 레코드가 여러개의 상대 레코드와 조인이 된다.
>

### ✔️ 1대1 관계 저장하기

```tsx
import { Photo } from "./entity/Photo"
import { PhotoMetadata } from "./entity/PhotoMetadata"

// create a photo
const photo = new Photo()
photo.name = "Me and Bears"
photo.description = "I am near polar bears"
photo.filename = "photo-with-bears.jpg"
photo.views = 1
photo.isPublished = true

// create a photo metadata
const metadata = new PhotoMetadata()
metadata.height = 640
metadata.width = 480
metadata.compressed = true
metadata.comment = "cybershoot"
metadata.orientation = "portrait"
metadata.photo = photo // this way we connect them

// get entity repositories
const photoRepository = AppDataSource.getRepository(Photo)
const metadataRepository = AppDataSource.getRepository(PhotoMetadata)

// first we should save a photo
await photoRepository.save(photo)

// photo is saved. Now we need to save a photo metadata
await metadataRepository.save(metadata)

// done
console.log(
    "Metadata is saved, and the relation between metadata and photo is created in the database too",
)
```

관계를 소유한 쪽이 나중에 저장이 되어야 한다.

`관계를 소유한 쪽 = RK를 가지고 있는 쪽` 이므로 RK가 가르키는 대상이 먼저 저장이 되는게 당연히 올바른 순서이다.

### ✔️ 관계를 역전하기

관계는 양방향, 단방향이 될 수 있다. 위에서는 단방향으로 설정했는데, 관계를 소유한 PhotoMetaData는 Photo에 대한 정보를 알 수 있지만, Photo는 PhotoMetaData에 대해 알지 못한다. 이렇게 되면 Photo에서 PhotoMetaData에 접근하기 어려워지는데, 관계를 역전하여 양방향으로 만들어 이 문제를 해결할 수 있다.

```tsx
import {
    Entity,
    Column,
    PrimaryGeneratedColumn,
    OneToOne,
    JoinColumn,
} from "typeorm"
import { Photo } from "./Photo"

@Entity()
export class PhotoMetadata {
    /* ... other columns */

    @OneToOne(() => Photo, (photo) => photo.metadata)
    @JoinColumn()
    photo: Photo
}
```

```tsx
import { Entity, Column, PrimaryGeneratedColumn, OneToOne } from "typeorm"
import { PhotoMetadata } from "./PhotoMetadata"

@Entity()
export class Photo {
    /* ... other columns */

    @OneToOne(() => PhotoMetadata, (photoMetadata) => photoMetadata.photo)
    metadata: PhotoMetadata
}
```

반대편 관계에 대해 명시하기 위해 @OneToOne 데코레이터에 두번째 인자로  `photo => photo.metadata` 화살표 함수를 넣어주었다.

> `@JoinColumn` 데코레이터는 관계를 소유하는 한 쪽에만 달아주는 것을 명심하자!
>

### ⚠️ ES Module을 사용하는 경우

```tsx
...
@OneToOne(() => Photo, (photo) => photo.metadata)
@JoinColumn()
photo: Relation<Photo>

...
@OneToOne(() => PhotoMetadata, (photoMetadata) => photoMetadata.photo)
metadata: Relation<PhotoMetadata>
```

속성의 타입을 `Relation`으로 감싸 `circular dependency` 문제를 해결한다.

### ✔️ 연관된 테이블들을 같이 조회하기

연관된 테이블을 같이 조회하는 방법은 두가지가 있다.

1. `find` 메서드 이용하기
2. `QueryBuilder` 이용하기

#### ✔️ `find` 메서드를 이용하는 방법

```tsx
import { Photo } from "./entity/Photo"
import { PhotoMetadata } from "./entity/PhotoMetadata"
import { AppDataSource } from "./index"

const photoRepository = AppDataSource.getRepository(Photo)
const photos = await photoRepository.find({
    relations: {
        metadata: true,
    },
})
```

`find` 옵션을 이용하여 연관된 테이블을 같이 조회할 수 있다.
더 많은 find 옵션은 [여기](https://typeorm.io/find-options#)를 참조.

#### ✔️ `QueryBuilder` 을 이용하는 방법

`find option`을 이용하는 방법은 심플해서 유용하지만, 복잡한 쿼리를 사용하려 한다면 `QueryBuilder`가 유리하다.

```tsx
import { Photo } from "./entity/Photo"
import { PhotoMetadata } from "./entity/PhotoMetadata"
import { AppDataSource } from "./index"

const photos = await AppDataSource.getRepository(Photo)
    .createQueryBuilder("photo")
    .innerJoinAndSelect("photo.metadata", "metadata")
    .getMany()
```

> `QueryBuilder` 을 사용해야 할 때는 SQL query를 직접 작성할 필요가 있을 때다.
>

### ✔️ Casecade를 이용하여 관계된 Object를 자동으로 저장하기

`@OneToOne` 데코레이터에서 `cascase` 옵션을 설정해 줌으로써 cascade를 설정할 수 있다.

```tsx
export class Photo {
    /// ... other columns

    @OneToOne(() => PhotoMetadata, (metadata) => metadata.photo, {
        cascade: true,
    })
    metadata: PhotoMetadata
}
```

`cascade`를 허용하면 photo를 PhotoMetaData와 따로 저장할 수 없게 된다.

photo Entitiy Object에 metadata를 속성으로 할당하고, `save()`를 호출하면 cascade 옵션으로 인해 metadata도 자동으로 같이 저장된다.

```tsx
import { AppDataSource } from "./index"

// create photo object
const photo = new Photo()
photo.name = "Me and Bears"
photo.description = "I am near polar bears"
photo.filename = "photo-with-bears.jpg"
photo.isPublished = true

// create photo metadata object
const metadata = new PhotoMetadata()
metadata.height = 640
metadata.width = 480
metadata.compressed = true
metadata.comment = "cybershoot"
metadata.orientation = "portrait"

photo.metadata = metadata // this way we connect them

// get repository
const photoRepository = AppDataSource.getRepository(Photo)

// saving a photo also save the metadata
await photoRepository.save(photo)

console.log("Photo is saved, photo metadata is saved too.")
```

> cascade는 photo에만 걸었기 때문에 metadata 측에서 저장할 때는 cascade가 동작하지 않는다.
>

### ✔️ 1대다, 다대1 관계 생성하기

```tsx
import {Entity,Column,PrimaryGeneratedColumn,OneToMany,JoinColumn} from "typeorm"
import { Photo } from "./Photo"

@Entity()
export class Author {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @OneToMany(() => Photo, (photo) => photo.author) // note: we will create author property in the Photo class below
    photos: Photo[]
}
```

Auther 측에서 photo와 `1대다` 관계이므로 `@OneToMany` 데코레이터를 이용한다.

`OneToMany`는 반대편의 `ManyToOne` 없이 존재할 수 없다.

```tsx
import { Entity, Column, PrimaryGeneratedColumn, ManyToOne } from "typeorm"
import { PhotoMetadata } from "./PhotoMetadata"
import { Author } from "./Author"

@Entity()
export class Photo {
    /* ... other columns */

    @ManyToOne(() => Author, (author) => author.photos)
    author: Author
}
```

> 1대다, 다대1 관계에서는 항상 다대1 관계를 가지는 측이 주인이다.
다 쪽이 RK를 가지고 있고, 다쪽이 봤을 때 1은 항상 하나의 레코드이므로.
>

> ManyToOne() 데코레이터를 단 쪽이 항상 관계의 주인 이므로 JoinColumn() 데코레이터를 생략 가능하다.
컬럼의 이름을 변경하고 싶을 때는, JoinColumn({name: ‘column_name’}) 을 통해 컬럼 이름을 명시하자!

### ✔️ 다대다 관계 형성하기

다대다 관계를 형성하는 Photo와 Album Entity를 보자.

```tsx
import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    ManyToMany,
    JoinTable,
} from "typeorm"

@Entity()
export class Album {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @ManyToMany(() => Photo, (photo) => photo.albums)
    @JoinTable()
    photos: Photo[]
}
```

1대1 관계와 마찬가지로 @JoinTable 데코레이터를 이용해 소유하는 측을 설정하자.

```tsx
export class Photo {
    /// ... other columns

    @ManyToMany(() => Album, (album) => album.photos)
    albums: Album[]
}
```

application을 실행하면 ORM은 두 테이블의 피벗 테이블을 생성한다.

```tsx
+-------------+--------------+----------------------------+
|                album_photos_photo_albums                |
+-------------+--------------+----------------------------+
| album_id    | int(11)      | PRIMARY KEY FOREIGN KEY    |
| photo_id    | int(11)      | PRIMARY KEY FOREIGN KEY    |
+-------------+--------------+----------------------------+
```

데이터베이스에 album과 photo를 저장해보자

```tsx
import { AppDataSource } from "./index"

// create a few albums
const album1 = new Album()
album1.name = "Bears"
await AppDataSource.manager.save(album1)

const album2 = new Album()
album2.name = "Me"
await AppDataSource.manager.save(album2)

// create a few photos
const photo = new Photo()
photo.name = "Me and Bears"
photo.description = "I am near polar bears"
photo.filename = "photo-with-bears.jpg"
photo.views = 1
photo.isPublished = true
photo.albums = [album1, album2]
await AppDataSource.manager.save(photo)

// now our photo is saved and albums are attached to it
// now lets load them:
const loadedPhoto = await AppDataSource.getRepository(Photo).findOne({
    where: {
        id: 1,
    },
    relations: {
        albums: true,
    },
})
```

## 7. QueryBuilder 사용하기

---

복잡한 SQL을 만들려고 할 때, `QueryBuilder`을 사용할 수 있다.

```tsx
const photos = await AppDataSource.getRepository(Photo)
    .createQueryBuilder("photo") // first argument is an alias. Alias is what you are selecting - photos. You must specify it.
    .innerJoinAndSelect("photo.metadata", "metadata")
    .leftJoinAndSelect("photo.albums", "album")
    .where("photo.isPublished = true")
    .andWhere("(photo.name = :photoName OR photo.name = :bearName)")
    .orderBy("photo.id", "DESC")
    .skip(5)
    .take(10)
    .setParameters({ photoName: "My", bearName: "Mishka" })
    .getMany()
```

좀 더 자세한 내용은 [이곳](https://typeorm.io/select-query-builder#) 참조.
