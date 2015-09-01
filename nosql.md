빅데이터로 통하는 NoSQL은 다양한 데이터 구조를 갖는다. 그리고 Key/Value, Column Family, Graph, Document 등 다양한 형태의 데이터베이스로 분류된다. 

가장 기본이 되는 Key/Value Store <br>
Key/Value Store에서 컬럼 개념을 확장하여 생성된 Column Family <br>
문서 저장에 적합한 Document Store <br>
네트워크 구조인 Graph 

RDBMS는 row 별로 데이터가 보관되고 조회된다. 관계형 모델의 모든 기준은 row 단위이다. 

NoSQL에서 가장 기본이 되는 것은 Key/Value Store이다. 

Column Family 모델은 Key/Value 모델의 확장이다.

**RDB 모델 **<br>
데이터를 수평적으로 배열하는 구조

|  | 사번 | 성명 | 성별 | 최종학력 |
| -- | -- | -- | -- | -- |
| row1 | 1001 | 홍,길동 | 남 | 하버드대학교 |
| row2 | 1004 | 김,천사 | 여 | 옥스포드대학교 |

**Column Family 모델**<br>
데이터를 수직적으로 쌓아가는 구조

| row key(사번) | Columns(컬럼명) | Value(값) |
| -- | -- | -- |
| 1001 | 성명 | 홍,길동 |
| 1001 | 성별 | 남 |
| 1001 | 최종학교 | 하버드대학교 |
| 1004 | 성명 | 김,천사 |
| 1004 | 성별 | 여 |
| 1004 | 최종학교 | 옥스포드대학교 |

관계형 모델에서는 새로운 컬럼이 추가될 때는 칼럼을 추가해 테이블 구조를 바꾸어야 하고, 특수한 경우에는 데이터도 새로 적재해야 한다. 즉 기존 데이터의 새로 추가된 컬럼에 업데이트하는 과정이 필요하다. 그러나 Column Family 테이블에서는 구조 변경이 필요 없고 확보된 데이터를 칼럼에 기혼여부로 하여 입력하면 된다.

| row key(사번) | Columns(컬럼명) | Value(값) |
| -- | -- | -- |
| 1001 | 성명 | 홍,길동 |
| 1001 | 성별 | 남 |
| 1001 | 최종학교 | 하버드대학교 |
| 1004 | 성명 | 김,천사 |
| 1004 | 성별 | 여 |
| 1004 | 최종학교 | 옥스포드대학교 |
| 1001| 기혼여부 | 기혼 |
| 1004| 기혼여부 | 미혼 |

Column Family DB의 단점은 편한 대신 타 테이블과의 조인이 안 된다는 점이다. 그래서 NoSQL 데이터베이스에서는 조인이 필요한 데이터는 하나의 테이블에 중복으로 관리해 처리 속도를 향상시킨다. 동일한 데이터들이 여러 테이블에 존재하게 되는데 이것을 관리하는 것이 No
SQL 데이터베이스에서는 가장 어려운 점이고 가장 중요한 특징이다. 



---

대표적으로 사용하는 Key/Value Store는 아마존 DynamoDB 같은 NoSQL이다. 


**DynamoDB를 시작하려면** 
1. 테이블의 Key 와 Index 를 결정
2. Read/Write 처리량을 결정

**주로 사용되는 기**능 <br>
Get/Put/Update/Delete/BatchGet <br>
Scan (전체 테이블을 싹쓸이로 긁어옴) <br> 
Query(Hash + 범위 키만) <br>

**테이블 디자인을 위한 요소 (1)**<br>

Table 
* 기본 키로 "Hash key" or "Hash key & Range key"를 선택 <br>

기본 키 : Hash key
* Hash key 단체로 데이터를 고유하게 식별할 수 있는 경우 사용 

기본 키 : Hash key & Range key
* Hash key에 해당하는 여러 데이터에서 Range key로 검색 가능 

Local Secondary Indexes
* Range key 이외에 필터 검색을 위한 키를 가질 수 있음

**테이블 디자인을 위한 요소 (2)**<br>

Attributes
* 데이터의 내용. Hash key에 해당하는 Attributes 이외에는 미리 정의할 필요는 없다. 또한 레코드에서 Attributes가 불규칙하더라도 문제 없다. 

Attributes 형식
* String
* Number
* Binary
* Array of String
* Array of Number
* Array of Binary

**DynamoDB의 데이터 모델**
![](dynamodbtablemodel.PNG)

Hash key
* 간단한 키 값
* Hash 이므로 정렬이 필요 없음 

Hash key + Range key
* 복합 기본 키 
* Range key는 sort가 있음 

샘플(1) 상품 카탈로그 <br>

스키마 
* 테이블명 Products
* 상품 ID(ProductId)를 테이블의 Hash key로 사용 



---

###AWS Documentation > AWS Mobile SDK > Android Developer Guide > Amazon DynamoDB

**시작하기** 

이 섹션은 AWS Mobile SDK(Android)를 사용하여 DynamoDB를 시작하기 위한 가이드를 step-by-step으로 제공한다. 

**Include JAR Files in Your Project (내 프로젝트에 라이브러리 추가하기)**

1. gradle 방식 <br>

```
dependencies {
    compile 'com.amazonaws:aws-android-sdk-core:2.+'
    compile 'com.amazonaws:aws-android-sdk-cognito:2.+'
    compile 'com.amazonaws:aws-android-sdk-s3:2.+'
    compile 'com.amazonaws:aws-android-sdk-ddb:2.+'
}
```
dependencies 풀 리스트

| Dependency | Build.gradle Value |
| -- | -- |
| AWS Mobile SDK core | com.amazonaws:aws-android-sdk-core:2.+ |
| Auto Scaling | 1:3 |
| Amazon Cloud Watch | 1:4 |
| Amazon Cognito Sync | 1:5 |
| Amazon DynamoDB | 1:6 |
| Amazon DynamoDB Object Mapper | 1:7 |
| Amazon EC2 | 1:8 |
| Elastic Load Balancing | 1:9 |
| Amazon Kinesis | 1:10 |
| Amazon Machine Learning | 1:11 |
| Amazon Mobile Analytics | 1:12 |
| Amazon S3 | 1:13 |
| Amazon Simple DB | 1:14 |
| Amazon SES | 1:15 |
| Amazon SNS | 1:16 |
| Amazon SQS | 1:17 |


	
	
	com.amazonaws:aws-android-sdk-autoscaling:2.+
	com.amazonaws:aws-android-sdk-cloudwatch:2.+
	com.amazonaws:aws-android-sdk-cognito:2.+
	com.amazonaws:aws-android-sdk-ddb:2.+
	com.amazonaws:aws-android-sdk-ddb-mapper:2.+
	com.amazonaws:aws-android-sdk-ec2:2.+
	com.amazonaws:aws-android-sdk-elb:2.+
	com.amazonaws:aws-android-sdk-kinesis:2.+
	com.amazonaws:aws-android-sdk-machinelearning:2.+
	com.amazonaws:aws-android-sdk-mobileanalytics:2.+
	com.amazonaws:aws-android-sdk-s3:2.+
	com.amazonaws:aws-android-sdk-sdb:2.+
	com.amazonaws:aws-android-sdk-ses:2.+
	com.amazonaws:aws-android-sdk-sns:2.+
	com.amazonaws:aws-android-sdk-sqs:2.+