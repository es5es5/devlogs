# Uploaing Images on the open source WYSIWYG.
## "오픈소스 위지위그에 이미지 올리기" 개발일지.

> &nbsp;
> <img src="./images/base64.PNG">
>
> *"에디터에 사진을 추가했더니 게시글 데이터가 미친듯이 올라갔어요 !"*

## 1. Situation
[summernote](https://summernote.org/) 를 비롯한 대부분의 오픈소스 위지위그 에디터에 이미지를 첨부하면 **base64로 인코딩** 되면서 첨부된다.<br>
단순한 이미지 1개를 첨부한게 위 상황이다.

## 2. Problem
저렇게 엄청난 문자열 데이터를 **데이터베이스에 저장하는 것 자체도 문제**인데, http 통신을 할 때 패킷이 감당할 수 없는 데이터를 보내기 때문에 **통신 자체가 안되는 경우**까지도 생긴다.

## 3. Solution
해결법은 **AWS S3, MINIO 같은 스토리지 서버에 이미지를 저장**하고, 그 이미지를 API GET 하는 방법이다.

## 4. Process
간단한 프로세스는 이렇다.

1. 이미지 첨부
2. 스토리지 서버에 저장
3. **이미지 태그로 치환 (중요 포인트)**

### 4-1. support
"이미지 태그로 치환" 이 부분은 에디터단에서 지원해주는게 중요하다.<br>
만약 위 프로세스대로 진행하려는데, 에디터에서 해당 기능을 지원해주지 않는다면.. <s>매우 고생할 것이다.</s><br>  🤦

그렇기 때문에 시중에 있는 오픈소스 위지위그 에디터를 채택할 때 **해당 기능을 지원 해주는지 반드시 고려**해봐야한다.

### 4-2. then..

> &nbsp;
> <img src="./images/img_tag.png">
>
> *"이미지 태그 " &lt;img&gt; " 하나로 불러오니깐 깔끔하네 !"*

# HOW TO
이번 일지에선 두 가지 에디터에서 내가 개발한 방식을 소개한다.

## 1. Summernote

[썸머노트 공식 홈페이지](https://summernote.org/)에서 이미지 업로드를 해보자.<br>
base64로 인코딩해서 올라가는걸 바로 확인할 수 있다.<br><br>

썸머노트는 [onImageUpload 콜백 메소드](https://summernote.org/deep-dive/#onimageupload)로 이미지 업로드를 공식 지원해준다.

```js
// Override image upload handler(default: base64 dataURL on IMG tag). You can upload image to server or AWS

// onImageUpload callback
$('#summernote').summernote({
  callbacks: {
    onImageUpload: function(files) {
      // upload image to server and create imgNode...
      $summernote.summernote('insertNode', imgNode);
    }
  }
});

// summernote.image.upload
$('#summernote').on('summernote.image.upload', function(we, files) {
  // upload image to server and create imgNode...
  $summernote.summernote('insertNode', imgNode);
});
```
