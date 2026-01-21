# 네트워크 엔지니어를 위한 자동화: Python과 Boto3로 AWS VPC 구축하기

### 서론
10년 이상의 경력을 가진 시니어 네트워크 엔지니어(CCIE)로서, 제 커리어의 대부분은 하드웨어 라우터의 CLI를 설정하는 데 쓰였습니다. 클라우드로 환경이 변해도 네트워크의 기본 원리는 변하지 않지만, 이를 배포하는 방식은 확실히 달라졌습니다. 오늘은 **Boto3** 라이브러리를 사용하여 표준 VPC 인프라 생성을 자동화하는 간단한 Python 스크립트를 공유하고자 합니다.

### 시나리오
이 스크립트는 `/16` CIDR 블록을 가진 VPC, 퍼블릭 서브넷, 그리고 외부 통신을 위한 인터넷 게이트웨이(IGW)를 생성합니다. 이는 점프 호스트나 웹 서버를 위해 사용되는 가장 기본적인 '퍼블릭 서브넷' 아키텍처를 모델로 합니다.

### Python 코드
```python
import boto3

# EC2 리소스 초기화
ec2 = boto3.resource('ec2', region_name='us-west-2')

# 1. VPC 생성
vpc = ec2.create_vpc(CidrBlock='10.0.0.0/16')
vpc.create_tags(Tags=[{"Key": "Name", "Value": "CommunityBuilder-VPC"}])
vpc.wait_until_available()
print(f"VPC 생성 완료: {vpc.id}")

# 2. 인터넷 게이트웨이 생성 및 연결
igw = ec2.create_internet_gateway()
vpc.attach_internet_gateway(InternetGatewayId=igw.id)
print(f"IGW 생성 및 연결 완료: {igw.id}")

# 3. 서브넷 생성
subnet = vpc.create_subnet(CidrBlock='10.0.1.0/24')
print(f"서브넷 생성 완료: {subnet.id}")

# 4. 라우팅 테이블 생성 및 퍼블릭 라우트 설정
route_table = vpc.create_route_table()
route = route_table.create_route(
    DestinationCidrBlock='0.0.0.0/0',
    GatewayId=igw.id
)
route_table.associate_with_subnet(SubnetId=subnet.id)
print(f"라우팅 테이블 생성 및 IGW 연결 완료")
