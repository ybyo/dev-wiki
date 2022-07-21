# Deploy Static Web

이번 장에선 정적 웹 이력서 배포 방안을 알아보겠습니다.

사용 기술: AWS S3

## AWS S3

> 어디서나 원하는 양의 데이터를 검색할 수 있도록 구축된 객체 스토리지

S3는 위와 같은 슬로건을 가진 AWS의 **스토리지 서비스**입니다. 이 외에도 AWS 스토리지 서비스는 상당히 많지만, 주목적인 정적 웹 이력서를 배포하기 위해서 S3를 활용하겠습니다.

[S3 개요](https://aws.amazon.com/ko/s3/) 페이지에 접속하면 가장 눈에 띄는 부분은 아무래도 '**99.999999999%**' 에 달한다는 안정성 소개 문구입니다. 이처럼 S3는 뛰어난 저장공간임과 함께 몇 가지 설정만으로 **정적 웹**을 배포하는 기능을 제공합니다. 이번 장에서 다룰 내용은 S3를 활용한 정적 웹 이력서 배포 과정입니다.

S3를 정적 웹 호스팅 용도로 사용하기 위해선 다음과 같은 방법들을 사용할 수 있습니다.

* Amazon S3 console(콘솔)
* REST API
* the AWS SDKs
* the AWS CLI or AWS CloudFormation

본 문서에선 Amazon S3 console을 활용한 웹 이력서 호스팅을 과정을 설명드린 후 추후에 AWS CLI 혹은 AWS CloudFormation을 활용한 호스팅을 설명드리겠습니다.

## 아키텍처

## 버킷(Bucket) 개요

Bucket은 S3의 핵심 개념이며 [S3 사용 설명서](https://docs.aws.amazon.com/ko\_kr/AmazonS3/latest/userguide/Welcome.html#BasicsBucket)에서 Bucket을 다음과 같이 설명하고 있습니다.

> Amazon S3에 저장된 객체에 대한 컨테이너

모든 객체는 특정 Bucket에 소속되며, Bucket의 이름은 파일 및 웹 페이지에 접근할 때 사용될 URL에 영향을 미칩니다. 예를 들어 `yibyeongyong.com`이라는 도메인을 가지고 있으며, 이력서 페이지의 주소를 `resume-en.yibyeongyong.com`으로 부여하고 싶다면, 버킷의 이름은 `resume-en.yibyeongyong.com`으로 설정해야 합니다([버킷 네이밍](https://docs.aws.amazon.com/en_es/AmazonS3/latest/userguide/bucketnamingrules.html)).

## 버킷 생성하기

버킷을 생성하려면 먼저 S3 콘솔로 이동해야 합니다. `Alt` + `S`를 누르면 커서가 검색창으로 이동합니다. 이를 통해 AWS 리소스 이름을 직접 검색하면 콘솔까지 빠르게 이동할 수 있습니다.
``
이미 한국어 이력서(resume-ko)를 배포 중이므로 다음 스크린숏 화면처럼 버킷 목록을 확인할 수 있습니다. 한글 이력서는 배포를 유지한 채, 이번엔 영문 이력서를 배포하기 위해 버킷을 생성하겠습니다. 화면에서 `Create bucket`을 눌러 버킷을 생성할 수 있습니다.

![S3 Bucket List](https://img.yibyeongyong.com/crc/02-deploy-static-web-on-aws/00-s3-bucket-list.png)

버킷 이름은 계정 이름을 생성하듯이 타 계정에서 생성된 기존의 버킷과 겹칠 수 없습니다. 또한, 웹 사이트를 호스팅 할 목적으로 버킷을 생성한다면 버킷의 이름은 **호스팅 할 웹 주소와 일치**시켜야 합니다.

한국어 이력서의 경우 서울 리전(`ap-northeast-2`)에 생성했지만, 영문 이력서는 버지니아 리전(`us-east-1`)에 생성하겠습니다. 리전의 차이가 실질적으로 웹 화면을 로딩하는 속도, AWS CloudFront 등의 리소스를 사용하는 경우 캐싱에 얼마나 영향을 끼칠지는 모르겠습니다. 이 점에 대해선 추후 성능 실험 자료 등을 찾거나 직접 실험을 수행하여 비교해 볼 예정입니다.

하단의 `Object Ownership`에서 권한 제어 리스트(Access Control Lists, ACLs)의 활성화 여부를 선택할 수 있습니다. 해당 버킷은 개인 이력서를 호스팅할 목적으로 사용할 예정이며, 누구나 접근할 수 있도록 공개할 예정입니다. 따라서, 권한 제어 리스트는 별도로 활성화하지 않겠습니다.

![버킷 이름과 리전 설정](https://img.yibyeongyong.com/crc/02-deploy-static-web-on-aws/01-set-bucket-name-and-region.png)

`Block all public access`의 체크를 해제하고 외부로부터의 모든 접근을 허용합니다. `Block all access`를 해제하면 하단에 경고 문구가 생성됩니다. 이는 불특정 다수의 외부 사용자가 접근할 수 있음을 경고합니다. 이 부분을 동의합니다.

![퍼블릭 접근 허용](https://img.yibyeongyong.com/crc/02-deploy-static-web-on-aws/02-enable-public-access.png)

이후의 설정 항목들은 필요에 따라 설정할 수 있으며 개인적으로는 이후의 옵션 중 `Tag`의 경우 이미 호스팅 중인 버킷인`resume-ko.yibyeongyong.com` 구분을 위해 따로 설정하였습니다. `Bucket Versioning`의 경우엔, 이력서를 포함한 프로젝트 관련 모든 코드를 Github에 업로드할 예정이므로 따로 사용할 필요를 느끼지 못했으며 마찬가지로 다른 옵션도 추가 설정은 하지 않았습니다. 설정은 버킷이 생성된 이후에도 변경이 가능하니 필요할 때 바꾸어주면 됩니다.

모든 설정을 마치면 하단의 `Create bucket`을 클릭하여 버킷 생성을 완료합니다.

## 웹 호스팅 활성화 하기

이후 버킷을 웹 호스팅 용도로 사용하기 위해선 추가 설정이 필요합니다. 생성된 버킷 리스트 중 방금 생성한 버킷을 누르고 탭에서 `Properties` -> 가장 하단의 `Static website hosting`에서 `Edit` 버튼을 누르면 다음과 같은 화면으로 이동합니다.

![웹 호스팅 활성화](https://img.yibyeongyong.com/crc/02-deploy-static-web-on-aws/03-enable-web-hosting.png)

위 화면의 `Static website hosting`을 `Enable`로 변경하고 `Hosting type`은 `Host a static website`로 변경합니다. 하단의 `Index document`는 이력서 페이지의 URL에 접근하면 가장 먼저 보여줄 페이지의 파일명을 입력하는 곳입니다. 관례를 따라 `index.html`로 설정하겠습니다. `Error document`나 `Redirection rules`는 따로 설정하지 않겠습니다. 필요한 설정을 마치면 `Save changes`를 눌러 이전 화면으로 돌아갑니다. 호스팅을 활성화하면 버킷에 웹으로 접근할 수 있는 엔드포인트가 생성됩니다.

![버킷 웹사이트 엔드포인트](https://img.yibyeongyong.com/crc/02-deploy-static-web-on-aws/04-bucket-website-endpoint.png)

탭의 `Objects`를 눌러 이력서 파일을 업로드합니다. 주의할 점은 방금 설정했던 `Index document`와 파일명을 일치시키는 것입니다. 테스트 목적으로 Github에서 다운로드 한 이력서 템플릿 레포지토리에서 파일명만 `index.html`로 수정하여 업로드해보겠습니다. 업로드는 브라우저로 드래그를 통해서도 가능합니다.

이 상태에서 파일 이름(`index.html`)을 클릭하면 첫 화면인 `Properties` 탭에서 `Object URL`을 확인할 수 있습니다. 현재 해당 URL을 통해 이력서 페이지에 접근할 수 있으며, 추후 DNS 서비스를 사용하면 자신만의 도메인으로 접근할 수 있습니다.

그러나 `Object URL`을 통해 접속해보면 기대와 달리 `AccessDenined`가 발생합니다.

```xml
This XML file does not appear to have any style information associated with it. The document tree is shown below.
<Error>
    <Code>AccessDenied</Code>
    <Message>Access Denied</Message>
    <RequestId>...</RequestId>
    <HostId>...</HostId>
</Error>
```

혹시나 내부 객체에 직접 접근하는 것만 불가능한 것이 아닐까 싶어 '버킷 설정 화면 -> `Properties` 탭 -> 하단 `Static website hosting`'의 `Bucket website endpoint`에 표시된 URL로 접근을 시도하더라도 `403 Forbidden`이 발생합니다.

이러한 오류가 발생하는 이유는 AWS 리소스마다 접근을 제어하는 정책(Policy)이 개별적으로 존재하기 때문입니다. S3 버킷의 경우 `Bucket policy`가 추가로 존재합니다. 웹 페이지 접근을 위한 관련 권한 설정 및 `Bucket policy` 설정에 관한 자세한 내용은 [이곳](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteAccessPermissionsReqd.html)에서 확인할 수 있습니다.

AWS 리소스의 정책은 Json 포맷으로 설정합니다. 버킷 설정 창에서 `Permissions`탭으로 이동 후 `Bucket policy`에서 `Edit`을 눌러 정책을 설정할 수 있습니다. 이후 화면에서 1) 정책을 직접 입력하거나, 2) 중앙의 `Policy generator`을 활용하거나, 3) 우측의 `Add new statement`에서 정책을 구성할 수 있습니다.

버킷의 객체에 읽기를 허용하는 정책을 생성하려면 다음과 같이 작성해야 합니다. 이 중 `Sid` 항목은 자유롭게 지을 수 있고 `Resource` 항목의 `Bucket-name` 부분만 개인적으로 사용하는 버킷 이름으로 변경하면 됩니다.

각 항목에 대한 설명은 [이곳](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference\_policies\_elements\_version.html)에서 상세히 다루고 있습니다.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::Bucket-Name/*"
      ]
    }
  ]
}
```

버킷 정책을 설정 후 `Object URL`과 `Bucket website endpoint`를 통해 접근해보면 두 방법 모두 정상적으로 이력서가 화면에 출력되는 것을 확인할 수 있습니다.

![이력서 접근 성공](https://img.yibyeongyong.com/crc/02-deploy-static-web-on-aws/05-resume-loading-success.png)

## 맺음말

지금까지 AWS S3의 버킷을 사용하여 정적 웹 이력서를 배포하는 방법에 대해서 알아보았습니다. 물론 이 자체로도 사용할 수 있지만 도메인 네임의 존재 이유가 무색하게 URL 자체가 외우기엔 길기도하고, 특히 주소창 좌측의 `주의 요함` 이라는 문구가 신경 쓰입니다.

이러한 문제들은 AWS의 DNS 서비스인 Route 53과 CDN 서비스인 CloudFront를 활용하여 해결할 수 있습니다. 이어지는 내용은 다음 장에서 다루겠습니다.

## 생각해볼 점

1. 버킷 정책이 설정되어있지 않아 버킷 오브젝트에 접근하지 못하는 상황에서 도대체 어느 부분이 문제인지 몰라 이를 고치는 데 많은 시간을 소비했습니다. 특히 AWS는 매우 다양한 리소스를 제공하므로 운용하는 클라우드의 규모가 클 수록 원인 파악이 더 어려워질 것 입니다. 문제의 원인을 빠르게 찾는 방법은 없을까요?
2. 현재까지의 작업을 잘 따라온 상태에서 눈썰미가 좋다면 이상한 부분을 발견했을 것 입니다. 왜 같은 `index.html`에 접근했음에도 S3 버킷 오브젝트에 직접 접근하면 HTTPS가 적용되어있고 버킷이 호스팅 중인 주소로 접근하면 HTTPS가 적용되어있지 않을까요? 이러한 차이는 왜 발생하는 걸까요? 사용자에게 데이터를 출력하기까지의 과정에서 무언가 차이점이 발생하는걸까요?

![같은 페이지임에도 SSL 적용에 차이가 있음](https://img.yibyeongyong.com/crc/02-deploy-static-web-on-aws/06-url-comparison.png)
