# AWS CloudFormation

## 템플릿 기본 사항 알아보기

[AWS CloudFormation - 시작하기 - 템플릿 기본 사항 알아보기](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/gettingstarted.templatebasics.html)

* 템플릿
  * 스택을 구성하는 AWS 리소스를 선언한 것
* 리소스
  * 리소스 유형: `AWS::ProductIdentifier::ResourceType(AWS::S3::Bucket)`
  * 리소스 유형에 따라 필수 속성, 선택 속성이 나뉨[레퍼런스의 required 항목 확인]https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html
* 템플릿 및 CloudFormation의 가장 큰 장점
  * 함께 작동하는 리소스 세트를 만들어서 애플리케이션이나 솔루션을 생성할 수 있음
  * `Ref` 함수는 리소스의 식별 속성을 참조할 수 있음

```yaml
Resources:
  Ec2Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      SecurityGroups:
        - !Ref InstanceSecurityGroup
      KeyName: mykey
      ImageId: ''
  InstanceSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
```

* 리소스 생성을 위한 사전 조건이 요구될 수 있음
  * key pair 등
* 일부 리소스의 경우 템플릿에서 사용할 수 없는 값이 지정되는 추가 속성 존재
  * 이 경우 `Fn::GetAtt` 함수를 사용
* 파라미터
  * 파라미터는 템플릿의 Parameters 객체에서 선언
  * 해당 값
  * 구속 조건을 정의하는 속성 목록
  * `NoEcho` 속성을 사용하면 마스킹 됨
    * 예외
      * `Metadata` 템플릿 섹션
      * `Outputs` 템플릿 섹션
  * CF 외부에서 저장 및 관리되는 민감한 정보를 참조하려면 동적 파라미터 사용 [참고](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html#creds)

```yaml
Parameters:
  KeyName:
    Description: Name of an existing EC2 KeyPair to enable SSH access into the WordPress web server
    Type: AWS::EC2::KeyPair::KeyName
  WordPressUser:
    Default: admin
    NoEcho: true
    Description: The WordPress database admin account user name
    Type: String
    MinLength: 1
    MaxLength: 16
    AllowedPattern: "[a-zA-Z][a-zA-Z0-9]*"
  WebServerPort:
    Default: 8888
    Description: TCP/IP port for the WordPress web server
    Type: Number
    MinValue: 1
    MaxValue: 65535
```

* 파라미터는 조건부 입력을 받을 수 있음
  * Mappings 객체 `AWS::Region` 가상 파라미터
