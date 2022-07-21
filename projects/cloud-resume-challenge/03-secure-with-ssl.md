# Secure with SSL

이번 장에선 배포 중인 정적 웹 이력서 페이지를 HTTPS 연결로 암호화 시켜 보안성을 향상시키고 신뢰를 높히는 방안을 알아보겠습니다.

사용 기술: AWS Route 53, AWS CloudFront, AWS Certificate Manager

## SSL 적용하기

HTTPS는 HTTP 연결을 SSL로 암호화하여 사용자와의 암호화된 연결을 보장하면서, 통신 주체를 명확히 하므로 보안성을 높히는데 큰 역할을 합니다. 웹 사이트에 HTTPS를 적용하기 위해선 일반적으로 별도의 인증서가 필요하고, 인증서를 구매 및 적용하는 추가 절차가 필요합니다. 그러나 AWS 리소스를 사용하면 매우 간단하게 HTTPS를 적용할 수 있으며, 사용자(호스팅 중인 서비스를 이용하는 사용자)에게 서비스를 더 원활한 서비스를 제공할 수 있습니다.

이를 위해선 다음과 같은 요소들이 필요합니다.

* DNS: AWS Route 53, 후이즈, 가비아, Hosting.kr 등
* CDN 서비스: AWS CloudFront
* 인증서 발급 서비스: AWS Certificate Manager

DNS의 경우 Route 53을 사용하지 않더라도 크게 문제가 발생한 사항은 없었습니다. 다만, Route 53 서비스를 이용하면 다른 리소스들이 AWS의 리소스를 사용하므로, 관련 서비스들과의 설정 등을 수행할 때, 약간은 더 편리한 장점이 있습니다. 따라서 본 문서에선 Route 53을 사용하겠습니다.

주의할 점은 Route 53 서비스를 이용하면 프리티어 여부와 상관없이 25개의 Hosted zone 당 0.5$/월의 요금이 부과됩니다. 다만, 테스트 혹은 학습 목적으로 사용 후 12시간 이내에 리소스를 삭제하면 요금은 발생하지 않습니다.

## DNS

DNS는 복잡한 URL 주소와 숫자로만 구성된 IP 등을 사람이 인식하고 외우기 편하게 바꾸어줍니다. 간단하게 자료구조의 맵(Map)과 같은 역할을 한다고 생각하면 됩니다. 웹 이력서에 DNS를 사용하는 이유는 HTTPS를 구성하기 위해선 인증서가 필요하고, 인증서를 발급받기 위해선 AWS Certificate Manager에서 [완전히 정규화된 도메인 이름](https://docs.aws.amazon.com/acm/latest/userguide/setup-domain.html)(Fully Qualified Domain Name, FQDN)을 요구하기 때문입니다.

## 도메인 구매하기

도메인은 다양한 업체에서 구매할 수 있습니다. 그리고 판매 업체, 도메인 명, 최상위 도메인에 따라서 가격, 그리고 기본으로 제공하는 서비스에 따라서 천차만별입니다. 국내 주요 업체의 도메인 종류 별 가격은 [KRNIC](https://xn--3e0bx5euxnjje69i70af08bea817g.xn--3e0b707e/jsp/popup/agencyFeePop.jsp)에서 확인할 수 있어 유용합니다. 본 문서 작성 시점에 사용한 도메인은 [호스팅케이알](https://www.hosting.kr/)을 통해 구매하였습니다.

주의하실 점은 업체에 따라서 도메인을 구매하고 별다른 설정을 하지 않으면, 해당 도메인을 후이즈에 검색했을 때 구매자 정보가 표시되는 경우가 있습니다. 따라서, 개인정보를 보호하기 위해선 별도의 서비스 신청 혹은 설정이 필요합니다. 다른 업체의 경우 도메인 구매 가격에 포함되어있는 경우도 있으나, 호스팅케이알의 경우 도메인 가격이 매우 저렴한 대신 'DOMAIN PRIVACY'라는 서비스를 별도로 결제해야합니다. 비용은 매우 저렴하므로 부담이 되진 않지만, 이러한 서비스가 별도로 신청이 필요하다는 것을 알지 못하면 그냥 넘어갈 수 있으니 주의해야합니다. 업체마다 해당 서비스의 이름은 다를 수 있지만, 이 서비스를 신청하면 후이즈를 통해 검색했을 때 구매자의 정보가 아닌 도메인 업체의 주소, 전화번호, 이메일 등으로 대체 표시가 됩니다.

## Route 53

> 최종 사용자를 인터넷 애플리케이션으로 라우팅하는 안정적이고 비용 효율적인 방법

AWS Route 53은 AWS에서 운영하는 클라우드 DNS입니다. 본 프로젝트에서 배포하고자하는 이력서는 Route 53없이 도메인 판매 업체의 네임 서버만을 활용해도 별다른 문제 없이 진행할 수 있습니다. 그러나 학습 목적으로 Route 53을 활용한 HTTPS 적용 및 웹 이력서를 배포해보겠습니다.

## 호스팅 영역 생성하기

Route 53 콘솔로 이동한 후 `Create hosted zone`을 눌러 호스팅 영역을 생성합니다. 다음 화면에서 호스팅 영역을 설정합니다. 만일 구매한 도메인이 `yibyeongyong.com`이라면 다음과 같이 서브 도메인 없이 작성 후, 외부에서도 접근할 수 있도록 하단의 `Type`을 `Public hosted zone`로 설정 후 생성합니다.

![호스팅 영역(Hosted Zone) 생성하기](https://img.yibyeongyong.com/crc/03-secure-with-ssl/00-create-hosted-zone.png)

## 인증서 요청하기

도메인을 구매했다면 인증서를 신청할 수 있습니다. 인증서는 AWS Certificate Manager에서 신청할 수 있습니다. 다만 본 프로젝트를 수행하는데 있어 필요한 인증서는 AWS CloudFront와 연결해서 사용할 예정이므로 **반드시 리전을 `us-east-1`로 변경한 뒤 신청**해야합니다. 이와 관련된 내용은 [이곳](https://aws.amazon.com/premiumsupport/knowledge-center/migrate-ssl-cert-us-east/?nc1=h_ls)에서 확인할 수 있습니다. AWS Certificate Manager 콘솔에서 리전이 `us-east-1`로 변경됐는지 확인 후 `Request a certificate`을 눌러 발급을 진행합니다.

신청할 인증서는 대부분의 브라우저와 운영체에서 신용할 수 있어야하므로 공개 인증서를 신청해야합니다. 따라서 `Certificate type`은 `Request a public certificate`으로 선택합니다.

도메인 이름은 모든 하위 도메인과 하위 도메인을 입력하지 않고 접속하는 모든 경우를 커버할 수 있도록 `*.yibyeongyong.com`과 `yibyeongyong.com` 두 가지를 추가합니다.

![인증서 요청하기](https://img.yibyeongyong.com/crc/03-secure-with-ssl/01-request-public-certificate.png)

이후 `Select validation method`에서 인증 방법을 선택해야합니다. 해당 도메인의 실제 소유 여부를 확인하는 절차입니다. 인증 방법에는 `DNS validation`과 `Email validation` 방식이 있습니다.

`Email validatoin`은 레코드에 접근 권한이 없는 경우 권장되는 방식입니다. 인증 방법은 후이즈 데이터베이스에 등록된 3개의 이메일과 관례적으로 생성하는 5개의 계정(`administrator`, `hostmaster`, `postmaster`, `webmaster`, `admin`)에 도메인을 붙인 주소로 5개의 이메일을 보내 총 8개의 이메일로 인증 메일을 보내어 인증 절차를 수행합니다. 이메일 인증에 대한 상세한 내용은 [이곳](https://docs.aws.amazon.com/acm/latest/userguide/email-validation.html)에서 확인할 수 있습니다. 다만, 앞서 개인정보 보호를 위해 후이즈 데이터베이스에 등록된 이메일을 호스팅 업체의 이메일로 변경해두었으므로, 본 프로젝트는 `DNS validation`으로 수행하겠습니다.

`DNS validation`인증 방식을 선택하면 AWS Certificate Manager는 하나의 CNAME 레코드를 발급합니다. 이 CNAME 레코드를 현재 이용 중인 네임 서버의 레코드에 등록해야 정상적으로 인증이 수행됩니다. 현재 Route 53을 사용하는 상태에서 인증을 진행하므로 Route 53 호스트 영역 생성시 부여된 네임 서버를 도메인 등록대행업체에 설정해야 합니다. 참고로 네임 서버는 안정적이 서비스 제공을 위해 최소 2개 이상 등록해야합니다. 예시 그림에선 호스팅 영역 생성시 부여받은 4개의 네임 서버를 등록대행자에 모두 설정했습니다.

![호스팅 영역 생성시 부여받은 네임 서버](https://img.yibyeongyong.com/crc/03-secure-with-ssl/02-set-up-name-server-aws.png)

![네임 서버 등록](https://img.yibyeongyong.com/crc/03-secure-with-ssl/03-set-up-name-server-for-third-party.png)

이후 Route 53에 인증 레코드를 등록해야 합니다. 이는 인증서 발급 신청 후 인증서 이름을 눌러 상세 정보 창으로 이동 후 `Create records in Route 53`을 클릭하면 Route 53에 자동으로 레코드가 등록됩니다. 물론 직접 등록하는 것도 가능합니다.

![DNS로 검증](https://img.yibyeongyong.com/crc/03-secure-with-ssl/04-validation-with-dns.png)

인증에는 약간의 시간이 필요합니다. 인증이 지나치게 지연되는 경우엔 네임 서버 등이 정상적으로 작동하는 지, 등록 시 오탈자 등은 없었는지 확인이 필요합니다. 레코드가 정상적으로 등록됐는지 확인하려면 Route 53의 호스트 정보에 있는 `Test record`, [Google Admin Toolbox Dig](https://toolbox.googleapps.com/apps/dig/), Linux 라면 [dig](https://linux.die.net/man/1/dig), Window 라면 [nslookup](https://docs.microsoft.com/ko-kr/windows-server/administration/windows-commands/nslookup) 등 네임 서버에 쿼리를 전송할 수 있는 도구를 사용하여 확인할 수 있습니다.

![Route 53에서 검증 상황 테스트.png](https://img.yibyeongyong.com/crc/03-secure-with-ssl/05-test-validation-status.png)

![인증서 발급 완료.png](https://img.yibyeongyong.com/crc/03-secure-with-ssl/06-issued-certificate.png)

도메인까지 인증을 받으면 이후엔 인증서를 사용할 수 있습니다. CloudFront에 인증서를 적용하고 몇 가지 옵션을 설정하면 CloudFront를 거치는 모든 통신은 HTTPS로 연결됩니다.

## CloudFront

> 낮은 대기 시간과 높은 전송 속도로 안전하게 콘텐츠 전송

[CloudFront](https://aws.amazon.com/cloudfront/?nc1=h_ls)는 AWS에서 콘텐츠 전송 네트워크(Content Delivery Network, CDN) 역할을 수행하는 리소스입니다. 주로 보안성 향상, 데이터 캐싱, 요금 절감을 목적으로 사용하며, 본 도전 과제에선 웹 이력서를 배포하고있는 버킷과 연결하여 HTTPS 적용 및 액세스 제어를 통한 보안성 향상을 목적으로 사용할 예정입니다.

CloudFront를 이용하기 위해선 `Distribution`을 생성해야합니다. CloudFront 콘솔에서 `Create Distribution`을 클릭해 생성을 시작합니다. 가장 먼저 `Origin`을 설정해야합니다. `Origin`은 배포를 수행할 AWS 리소스를 선택해야합니다. 웹 이력서는 S3 버킷을 통해 배포중이므로 `Origin domain`에서 배포할 이력서를 보관하고있는 S3 버킷의 `Bucket website endpoint`(S3 버킷의 호스팅 옵션을 활성화 했을 때 부여받은 주소)를 선택합니다.

![오리진 설정.png](https://img.yibyeongyong.com/crc/03-secure-with-ssl/07-set-origin.png)

이후 `Default cache behavior`의 `Viewer`에서 `Viewer protocol policy`을 `Redirect HTTP to HTTPS`로 설정합니다 이를 통해 HTTP로 접근하더라도 자동으로 HTTPS 연결로 리다렉트합니다. 다음 `Settings`의 `Alternate domain name (CNAME) - optional`에서 CNAME을 설정합니다. 지금 CNAME을 설정하지 않더라도 우선 CloudFront 생성시 부여된 `*.cloudfront.net` 형태의 주소로 접근할 수 있으며, 당장 등록하지 않더라도 추후 Route 53 혹은 외부 도메인 등록업체에 직접 등록할 수 있습니다.

다음 `Custom SSL certificate - optional`에선 Certificate Manager에서 생성한 인증서를 선택합니다. 이를 통해 HTTPS 연결을 활성화합니다. 마지막으로 `Default root object - optional`에선 S3 버킷에서 첫 페이지로 출력되도록 설정했던 `index.html`등의 파일 이름을 입력합니다.

![SSL 설정.png](https://img.yibyeongyong.com/crc/03-secure-with-ssl/08-set-ssl.png)

Distribution 배포는 약간의 시간이 걸립니다. CloudFront의 Distribution의 상세 정보 창에서 `Last modified`가 `Deploying`에서 마지막 수정 시간으로 바뀝니다.

![Distribution 배포 완료.png](https://img.yibyeongyong.com/crc/03-secure-with-ssl/09-deployed-distribution.png)

배포가 완료되면 CloudFront 자체에 부여된 주소로 접속을 시도합니다. 잘 접속된다면 Route 53에 CloudFront의 CNAME 레코드를 등록하여 혹은 레코드 타입을 A로 설정하고 `Alias` 옵션을 활성화 시켜 CloudFront 리소스를 직접 선택할 수 있습니다. CloudFront 생성 시 `Default root object`를 `index.html`로 설정했으므로 만일 CNAME으로 레코드 등록을 수행한다면 Route 53에 레코드를 등록할 땐 `index.html`을 붙이지 않습니다.

![CNAME 레코드 등록](https://img.yibyeongyong.com/crc/03-secure-with-ssl/10-create-cname-record.png)

이후 `Alternate domain name`으로 등록했던 주소에 접속해봅니다. 도메인도 정상적으로 적용된 것을 확인할 수 있으며 성공적으로 HTTPS 연결 또한 활성화 된 것을 확인할 수 있습니다.

![HTTPS 연결 확인](https://img.yibyeongyong.com/crc/03-secure-with-ssl/11-secured-with-ssl.png)

## 맺음말

지금까지 S3 버킷을 활용하여 호스팅 중인 웹 이력서에 Route 53을 활용하여 도메인 네임을 부여하고, CloudFront를 활용하여 HTTPS 연결을 활성화 했습니다. 다음 장에선 이렇게 배포 중인 이력서에 AWS Lambda를 활용하여 방문자 조회수 기록 기능을 추가할 예정입니다.

## 생각해볼 점

1. 본 장에서 HTTPS 연결을 구성했지만 어딘가 모르게 부족한 느낌이 듭니다. 혹시 놓친 부분이 있지 않을까요? 아무리 성문을 튼튼하게 쌓아올린다고 하더라도 우회로가 있다면 성은 쉽게 무너집니다. HTTPS라는 튼튼한 성벽을 쌓아올렸지만, 이력서까지 접근하는 우회로가 있진 않을까요?
2. Route 53은 AWS 서비스와의 상호 연결을 구성할 땐 `Alias`같은 편의 기능을 제공해 편리한 장점이 있습니다. 하지만, 프리티어 사용자임에도 호스트 영역을 생성하면 반드시 요금이 부과되는 문제가 있습니다. 이러한 요금을 줄일 수 있는 방안이 있지 않을까요?
