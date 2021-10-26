---
layout: single
title: "[AWS] 2. S3 대용량파일 클라이언트단에서 다운로드하기(JAVA) "
categories: [AWS]
author_profile: true
excerpt: AWS 개념 중 클라이언트에서 직접 S3 대용량파일을 다운로드하는 방식에 대해 정리한다. 
toc: true
toc_sticky: true
---

## S3 대용량 파일 다운로드
- 진행중인 프로젝트 중에 S3에 업로드 되어있는 대용량 이미지를 **프론트 단**에서 바로 다운로드 할 수 있도록 하는 기능이 필요하여 작업중이다.
- 현재 프로젝트는 SpringBoot으로 구성되어있다.

<br>


### 기본 구성
--------------------
- 같이 작업하는 수석님께 프로젝트를 이어 받았을 때는 스프링시큐리티를 이용한 로그인, 회원가입 기능과, 프로젝트 기본 구성 (여러 page, logger, intercepter, exception, DB 연동 등), AWS 연동 등이 구현된 상태였다.
- S3에 업로드 되어있는 파일목록들이 Table에 출력되어서 확인이 가능한 상황이었다.
- 기본적인 파일 다운로드 프로세스 처럼 프론트단에서 서버단에 API를 호출한 뒤 서버단에서 파일다운로드를 처리하도록 구현되어있었다.

<br>

### 문제점
--------------------
- 프론트단에서 다운로드 버튼을 클릭했을 때 파일이 다운로드는 되지만 꽤 오랜 시간이 흐른뒤에 파일 저장/열기 화면이 출력되었다. 
- 아마도 **서버에서 파일 다운로드**를 완료한 후에 **프론트로 리턴**시키기 때문인것으로 보였다.
- 용량이 적은 파일이라면 이러한 방식으로 다운로드를 진행하여도 별 문제는 없을 것으로 보였지만 현재 프로젝트에서 관리할 파일들은 **대부분이 GB단위**인 이미지들이라 시간이 오래걸리는 문제가 발생할 수 밖에 없어 수정이 필요하다.

<br>

### 첫번째 방법 - 백엔드에서 처리
---------------------
- 프론트단에서 바로 이미지를 다운받는 것은 보안상 문제가 있을것이라는 막연한 생각때문에 백엔드에서 처리하는 방향으로 개발을 진행하였다.
- Multipart 다운로드나 스트림을 이용하여 프론트에서 병렬로 다운로드 하는 방식으로 구현하면 어떨까하는 생각에 열심히 검색 + 조사를 해봤지만 완벽하게 구현되지 않았다.


<br>

### 두번째 방법 - 프론트에서 처리 + presignedURL
---------------------
- 구글링을 통해 **pre-signedURL**이라는 AWS SDK지원 기능을 찾게되었다.
- PresignedURL이란? : AWS 자원에 접근권한을 제공하기 위해서 사용되는 URL이며, 사전에 적절한 권한을 가진 자격증명에 의하여 Signed된 URL을 의미한다.

=> **사전에 이미 AWS S3를 접근할 수 있는 권한을 가진 URL**

![deployment App](/assets/img/aws/2_s3_presignedurl_1.png)

- 일반적인 파일 업로드의 경우 [**클라이언트** -> **서버** -> **S3**] 순으로 업로드가 진행된다.
- PresiginedURL을 사용할 경우는 [**클라이언트** -> **S3**]로 직접 접근이 가능하다.

<br>

### 파일 다운로드 구현
------------
- 순서 :  Ajax REST API 호출 -> PresignedURL 발급 -> PresignedURL
리턴 -> PresignedURL로 접속하여 파일 다운로드
- API 호출을 할때 **Keyname**을 파라미터로 준다.
- JAVA용 AWS SDK를 사용하여 PresignedURL을 발급받는 코드는 아래 Link에 상세하게 나와있다.

[AWS Documentation - Java용 AWS SDK를 사용하여 미리 서명된 객체 URL 생성 >>](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/ShareObjectPreSignedURLJavaSDK.html)<br>

**해당코드**

```java

import com.amazonaws.AmazonServiceException;
import com.amazonaws.HttpMethod;
import com.amazonaws.SdkClientException;
import com.amazonaws.auth.profile.ProfileCredentialsProvider;
import com.amazonaws.regions.Regions;
import com.amazonaws.services.s3.AmazonS3;
import com.amazonaws.services.s3.AmazonS3ClientBuilder;
import com.amazonaws.services.s3.model.GeneratePresignedUrlRequest;

import java.io.IOException;
import java.net.URL;

public class GeneratePresignedURL {

    public static void main(String[] args) throws IOException {
        Regions clientRegion = Regions.DEFAULT_REGION;
        String bucketName = "*** Bucket name ***"; // ①
        String objectKey = "*** Object key ***";   // ②

        try {
            AmazonS3 s3Client = AmazonS3ClientBuilder.standard()
                    .withRegion(clientRegion)
                    .withCredentials(new ProfileCredentialsProvider())
                    .build();

            // Set the presigned URL to expire after one hour.
            java.util.Date expiration = new java.util.Date();
            long expTimeMillis = expiration.getTime();
            expTimeMillis += 1000 * 60 * 60;       //③
            expiration.setTime(expTimeMillis);

            // Generate the presigned URL.
            System.out.println("Generating pre-signed URL.");
            GeneratePresignedUrlRequest generatePresignedUrlRequest =
                    new GeneratePresignedUrlRequest(bucketName, objectKey)
                            .withMethod(HttpMethod.GET)
                            .withExpiration(expiration);
            URL url = s3Client.generatePresignedUrl(generatePresignedUrlRequest);

            System.out.println("Pre-Signed URL: " + url.toString());
        } catch (AmazonServiceException e) {
            // The call was transmitted successfully, but Amazon S3 couldn't process 
            // it, so it returned an error response.
            e.printStackTrace();
        } catch (SdkClientException e) {
            // Amazon S3 couldn't be contacted for a response, or the client
            // couldn't parse the response from Amazon S3.
            e.printStackTrace();
        }
    }
}
```

    - ① : application.yml에 aws 정보를 입력하였다면 그중 bucket name을 value로 사용하면된다.
    - ② : API호출시 던져주었던 keyname을 사용한다.
    - ③ : URL 만료시간을 입력한다. (나는 5초로 주었다.)

<br>
- URL을 리턴해줄수 있도록 함수를 수정하여 사용한다.
- 리턴받은 URL로 이동하도록 프론트단에서 작업처리한다.

<br>

### 추후에 수정이 필요하다고 생각되는 사항
-------------------------
1. 현재 프로젝트는 다운로드기능만 사용하기 때문에 S3접근 권한을 읽기 권한만 허용한 상태이다. 따라서 보안에 크게 문제가 될 것으로 보이지 않아 PresignedURL 만료시간을 짧게 주어 URL남용을 하지 않도록만 처리 해놓은 상태이다.

-> 추후에 쓰기권한도 필요하게 된다면 만료시간을 이용해서가 아닌 좀 더 확실한 방법으로 보안유지할 필요성이 있어 보인다.



<br>
<br>

------------------
**◎ 참고자료**
- [AWS Documentation - Java용 AWS SDK를 사용하여 미리 서명된 객체 URL 생성](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/ShareObjectPreSignedURLJavaSDK.html)

- [S3 pre-signed URL 한번만 사용하기 :: 마이구미](https://mygumi.tistory.com/380)
