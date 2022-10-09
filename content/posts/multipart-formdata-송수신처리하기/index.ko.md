---
title: "HTTP multipart/form-data 로 이미지 요청/응답 처리하기 (Feat. React, Express, Multer)"
subtitle: "Content-Type: multipart/form-data 을 이해하고, React, Express 환경에서 요청/응답을 처리해보자"
date: 2022-10-09T13:00:00+09:00
lastmod: 2020-10-09T13:00:00+09:00
draft: false
author: "EastAsh"
authorLink: "https://github.com/LouiMinister"
description: "Content-Type: multipart/form-data 을 이해하고, React, Express 환경에서 요청/응답을 처리해보자"
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["Node", "HTTP", "React", "Express", "Multer", "Image"]
categories: ["Node BackEnd", "Node FrontEnd"]

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
## 0. 흐름 잡기

---

{{< admonition type=tip title="HTTP란?" open=true >}}
인터넷 상에서 클라이언트와 서버가 자원을 주고받을 때 쓰는 통신 규약
{{< /admonition >}}

→ Client에서 Server로 이미지를 보낼 때도 HTTP 프로토콜을 이용해야 한다.

> **HTTP Header 중 Response Header에 Content-Type 항목이 존재한다**
>
>
>
> **At** [HTTP Protocol](https://www.notion.so/HTTP-Protocol-6fe9807f6ef64284b9f471513620741d)
>
> ### 📄 리소스 포맷
>
> - MIME (Multipurpose Internel Mail Extensions) Type으로 파일의 포맷을 분류한다.
    >     - MIME Type은 웹보다 상위 개념인 인터넷에서 사용되는 개념, Content-Type은 인터넷의 하위 집합인 웹에서 사용되는 개념
> - 원래는 전자 우편을 위한 표준이었는데, 웹에서도 활용하고 있다.
>
> ```
> Content-Type:
> 	text/plain
> 	text/html
> 	text/css
> 	image/jpeg
> 	image/png
> 	audio/mpeg
> 	audio/ogg
> 	audio/*
> 	video/mp4
> 	application/octet-stream
> 	multipart/mixed
> 	multipart/form-data
> ```
>

{{< admonition type=tip title="HTTP Request는 `Request Line`, `Header`, `Body` 로 이루어져 있다." open=false >}}

{{< /admonition >}}

파일이 존재하는 폼 전송은 `Request Header`의  `Content-Type` 을 `multipart/form-data` 로 지정하여 HTTP 요청을 구성한다.

그리고 `Request Body`에 문자로 변환한 이미지 정보를 담아 서버로 전송한다.

## 1. 그래서 multipart가 뭔데?

---

웹 클라이언트가 요청을 보낼 때, Http Request Body 부분에 `여러 종류의 데이터`를 `여러 부분`으로 나눠서 보낼 때의 `Content-Type` 이다.

{{< admonition type=question title="폼 전송은 여러 종류의 데이터를 한번에 보낼 수 있음" open=true >}}
-> 한 Body 안에 여러 종류의 데이터를 구분하여 넣어주는 방법도 필요해짐
{{< /admonition >}}

- 일반적인 form submit 을 할 때 `Content-type` 의 default 값은  `application/x-www-form-urlencoded` 이다.
- `multipart/form-data` 는 파일전송을 위해 파일 데이터 구분자가 붙기 때문에 일반적인 데이터 전송에는 비효율적이다.

## 2. 클라이언트에서 파일 업로드가 존재하는 폼 전송하기

---

`multipart/form-data` 를 사용하는 이유를 알아봤으니, 이제 직접 코드를 작성하여 서버에 `HTTP 요청`을 보내보자.

HTTP 요청은 Axios 라이브러리를 사용하였다.

```bash
$ npm i axios
```

{{< admonition type=warning title="이 예제는 여러개의 파일과, 여러개의 텍스트 데이터를 전송하는 것을 목표로 한다." open=true >}}
단일 파일을 보내는 경우에는 코드를 수정해야 한다.
{{< /admonition >}}

**Form.jsx**

```jsx
// 폼 컴포넌트
const ArticleForm = () => {
    const [imageList, setImageList] = useState([]);
    const [titleInput, setTitle] = useState('');
    const [contentInput, setContentInput] = useState('');

		// 임의의 버튼을 클릭하면 아래 함수를 실행하도록 한다.
    onClickSubmit(async () => {
				// 폼 데이터 생성
        const formData = new FormData();
        // 폼에 데이터를 첨부하기 위해서는 form.append('키값(필드)', 데이터) 를 이용한다.
        // 폼에 파일 첨부. 파일 첨부 같은 경우에는 반복문을 통해 append 해주어야 한다.
        imageList.forEach(image => {
            formData.append('files', image);
        });
        // 폼에 텍스트 정보 첨부.
				// 텍스트 그대로 전송되기 때문에, Object를 보내기 위해서는 JSON 형식으로 보낸다.
        formData.append(
            'data',
            JSON.stringify({
                title: titleInput,
                content: contentInput,
            }),
        );
        try {
            // axios를 이용한 post 요청. 헤더를 multipart/form-data 로 한다.
            await axios.post('/api/articles', formData, {
                headers: {'Content-Type': 'multipart/form-data', charset: 'utf-8'},
            });
            alert('게시글이 등록되었습니다');
        } catch (err) {
            alert(err.data.message || '게시글 등록에 실패했습니다');
        }
    });

    const onChangeImageInput = e => {
        setImageList([...imageList, ...e.target.files]);
    };

    return (
        <>
            <input type="file" accept="image/jpg,image/png,image/jpeg,image/gif" multiple onChange={onChangeImageInput} />
            <input placeholder={'글 제목'} defaultValue={titleInput} onChange={e => setTitle(e.target.value)} />
            <input placeholder={'게시글 내용을 작성해주세요.'} defaultValue={contentInput} onChange={e => setContentInput(e.target.value)} />
        </>
    );
};
```

- append 메서드를 이용해 FormData에 data들을 담고 post 요청을 한다.
- 파일을 여러개 보내기 위해서는, 1. 반복문을 이용해서 하나의 키값(필드)에 여러 파일을 첨부하는 방법과, 2. 여러 키값(필드)에 각각의 파일을 첨부하는 방법이 있다.
    - 해당 예제에서는 한 키값에 여러 파일들을 첨부하는 식으로 구현하였다. (서버에서 iteration을 편하게 하기 위함)
- 텍스트값은 텍스트 그대로 전송되기 때문에, Object를 보내기 위해서는 JSON 형식으로 보내주고, 서버측에서 JSON 파싱을 하도록 한다.

## 3.  Express 서버에서 multipart/form-data 형식의 요청 처리하기

---

### Multer 미들웨어 라이브러리 알아보기

해당 예제에서는 express 환경에서 multer 라이브러리를 이용하여 구현한다.

```bash
$ npm i multer
# 현 예제에서는 "^1.4.5-lts.1" 버전을 사용하였다.
```

[multer 라이브러리 github readme.ko](https://github.com/expressjs/multer/blob/master/doc/README-ko.md) 에서 공식 리드미를 확인해보자.

> Multer는 파일 업로드를 위해 사용되는 `multipart/form-data` 를 다루기 위한 node.js 의 미들웨어 입니다. 효율성을 최대화 하기 위해 [busboy](https://github.com/mscdex/busboy) 를 기반으로 하고 있습니다.
>
>
> **주**: Multer는 multipart (`multipart/form-data`)가 아닌 폼에서는 동작하지 않습니다.
>

정리해보자면 위에 설명하였듯이, `multipart/form-data` 요청은 여러 형식의 데이터들을 `구분자`로 나누어 전송한다.  따라서 서버에서는 해당 타입의 요청이 올 때, 여러 형식의 데이터들을 직접 파싱해주는 작업이 필요하다. `multer` 라이브러리는 이 작업을 편하게 할 수 있도록 도와주는 미들웨어 라이브러리 이다.

Multer 라이브러리에 대해 중점적으로 파악해야 할 부분은 두가지이다.

1. 폼 형식이 어떤지에 따라 어떤 multer 미들웨어를 라우터에 적용할지 선택한다.
    - file이 한개일 경우 → `upload.single(’키값(필드)’)` 적용
    - file이 두개일 경우 → `upload.array(’키값(필드)’, 최대 파일 갯수)` 적용
    - 텍스트 필드의 값을 포함하고, file이 한개 혹은 여러개일 경우 → `upload.fields([{ name: ‘키값(필드)’, maxCount: 쵀대 파일 갯수}, … , }]`
2. 요청으로 받은 파일을 어디에 저장할 지에 따라 multer 미들웨어 설정
    - 디스크에(폴더) 저장 → `DiskStorage` 이용
    - 메모리에 저장 (이후 S3 등 외부 스토리지등에 따로 파일 저장하는 경우) → `MemoryStorage` 이용

### Multer를 이용한 미들웨어 만들기

관심사의 분리를 위해 middleware를 다른 파일에 따로 구현하도록 한다.

**multer.middleware.js**

```jsx
import multer from 'multer';

// diskStorage를 사용할 경우
// const diskStorage = multer.diskStorage({
//     destination: __dirname + '/uploads/',
//     filename: (req, file, cb) => {
//         cb(null, Buffer.from(file.originalname, 'latin1').toString('utf8'));
//     },
// });

// memoryStorage를 사용할 경우
const multerStorage = multer.memoryStorage();
const upload = multer({storage: multerStorage});

// multer 라이브러리가 현재 텍스트를 latin1 으로 처리하여 한글로된 파일 이름이 깨지는 문제가 있다.
// Parser 함수를 따로 구현하여 사용하도록 한다.
export const fileNameParser = fileName => Buffer.from(fileName, 'latin1').toString('utf8');

// 해당 예제에서는 파일 데이터 여러개, JSON으로 된 텍스트 데이터 하나를 받는 폼을 가정한다.
export const articleFormDataHandler = upload.fields([
    {name: 'files', maxCount: 10},
    {name: 'data', maxCount: 1},
]);
```

### 미들웨어 라우터에 적용하기

구현한 미들웨어를 라우터에 적용하자.

**article.controller.ts (express router)**

```jsx
import express from 'express';
import {ArticleService} from '@/service/article.service';
import createError from 'http-errors';

import {articleFormDataHandler} from '@/middleware/multer.middleware';
import {uploadToS3} from '@/repository/s3.repository';
import * as ArticleDTO from '@/controller/dto/ArticleDTO';
import * as ArticleImageDTO from '@/controller/dto/ArticleImageDTO';
import * as CommonDTO from '@/controller/dto/CommonDTO';
import {User} from '@/entity/user';

export const router = express.Router();

// 위에 선언한 미들웨어 적용
router.post('/', articleFormDataHandler, async (req, res, next) => {
    try {
				// 미들웨어가 req.files로 파일 데이터를 담아줌. 그 중에서 필드값을 통해 첨부된 파일 참조
        // map 을 이용해 파일 이름 한국어 깨지지 않도록 함
        const files = req.files['files'].map(file => ({...file, originalname: fileNameParser(file.originalname)}));
				// 미들 웨어가 req.body로 텍스트 데이터를 담아줌. 그 중에서 필드값을 통해 첨부된 텍스트 값 참조
        const textData = JSON.parse(req.body.data);
			
				/* 파일들과 텍스트 데이터를 이용한 비즈니스 로직 */
    } catch (err) {
        next(err);
    }
});
```

- `upload.fields([{ name: ‘키값(필드)’, maxCount: 쵀대 파일 갯수}, … , }]` 로 multer 미들웨어를 구성하였을 경우, 파일은 **req.files[’설정한 필드값’]**,  텍스트는 **req.body[’설정한 필드값’]** 에 담기게 된다.
- 데이터 파싱은 마무리 하였으므로 사용자가 원하는 대로 비즈니스 로직을 처리하면 된다.
    - 저는 이미지를 스토리지 서버(S3같은) 에 따로 저장하고, 해당 URL을 RDB에서 값으로 관리함으로 써 이미지 파일들을 관리하였습니다.


\
\
**참고자료**

[https://velog.io/@shin6403/HTTP-multipartform-data-란](https://velog.io/@shin6403/HTTP-multipartform-data-%EB%9E%80)

[https://velog.io/@shin6403/React-Form-Data-전송](https://velog.io/@shin6403/React-Form-Data-%EC%A0%84%EC%86%A1)

[https://blogpack.tistory.com/1088](https://blogpack.tistory.com/1088)

[https://velog.io/@rand_guy/TIL-Input-타입-File로-이미지-업로드와-미리보기](https://velog.io/@rand_guy/TIL-Input-%ED%83%80%EC%9E%85-File%EB%A1%9C-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EC%97%85%EB%A1%9C%EB%93%9C%EC%99%80-%EB%AF%B8%EB%A6%AC%EB%B3%B4%EA%B8%B0)

[https://velog.io/@chloeelog/React-사진-업로더-만들기-0](https://velog.io/@chloeelog/React-%EC%82%AC%EC%A7%84-%EC%97%85%EB%A1%9C%EB%8D%94-%EB%A7%8C%EB%93%A4%EA%B8%B0-0)
