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

#AWS Documentation > AWS Mobile SDK > Android Developer Guide > Amazon DynamoDB

**시작하기** 

이 섹션은 AWS Mobile SDK(Android)를 사용하여 DynamoDB를 시작하기 위한 가이드를 step-by-step으로 제공한다. 

코드 샘플 <br>
https://github.com/awslabs/aws-sdk-android-samples

AWS-SDK-Android<br>
https://github.com/aws/aws-sdk-android

###Include JAR Files in Your Project (내 프로젝트에 라이브러리 추가하기)

* **gradle 방식** 

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
| Auto Scaling | com.amazonaws:aws-android-sdk-autoscaling:2.+ |
| Amazon Cloud Watch | com.amazonaws:aws-android-sdk-cloudwatch:2.+ |
| Amazon Cognito Sync | com.amazonaws:aws-android-sdk-cognito:2.+ |
| Amazon DynamoDB | com.amazonaws:aws-android-sdk-ddb:2.+ |
| Amazon DynamoDB Object Mapper | com.amazonaws:aws-android-sdk-ddb-mapper:2.+ |
| Amazon EC2 | com.amazonaws:aws-android-sdk-ec2:2.+ |
| Elastic Load Balancing | com.amazonaws:aws-android-sdk-elb:2.+ |
| Amazon Kinesis | com.amazonaws:aws-android-sdk-kinesis:2.+ |
| Amazon Machine Learning | com.amazonaws:aws-android-sdk-machinelearning:2.+ |
| Amazon Mobile Analytics | com.amazonaws:aws-android-sdk-mobileanalytics:2.+ |
| Amazon S3 | com.amazonaws:aws-android-sdk-s3:2.+ |
| Amazon Simple DB | com.amazonaws:aws-android-sdk-sdb:2.+ |
| Amazon SES | com.amazonaws:aws-android-sdk-ses:2.+ |
| Amazon SNS | com.amazonaws:aws-android-sdk-sns:2.+ |
| Amazon SQS | com.amazonaws:aws-android-sdk-sqs:2.+ |

<br>
* **Maven 방식**
    ```
    <dependencies>
        <dependency>
            <groupid>com.amazonaws</groupid>
            <artifactid>aws-android-sdk-core</artifactid>
            <version>[2.2.0, 2.3)</version>
        </dependency>
        <dependency>
            <groupid>com.amazonaws</groupid>
            <artifactid>aws-android-sdk-cognito</artifactid>
            <version>[2.2.0, 2.3)</version>
        </dependency>
        <dependency>
            <groupid>com.amazonaws</groupid>
            <artifactid>aws-android-sdk-mobileanalytics</artifactid>
            <version>[2.2.0, 2.3)</version>
        </dependency>
    </dependencies>
    ```
Release버전은 여기서 확인 <br>
https://aws.amazon.com/releasenotes/Android

| Service/Feature | artifactID |
| -- | -- |
| AWS Mobile SDK Core [1] | aws-android-sdk-core |
| Auto Scaling | aws-android-sdk-autoscaling |
| Amazon Cloud Watch | aws-android-sdk-cloudwatch |
| Amazon Cognito Sync | aws-android-sdk-cognito |
| Amazon DynamoDB | aws-android-sdk-ddb |
| Amazon DynamoDB Object Mapper | aws-android-sdk-ddb-mapper |
| Amazon EC2 | aws-android-sdk-ec2 |
| Elastic Load Balancing | aws-android-sdk-elb |
| Amazon Kinesis | aws-android-sdk-kinesis |
| Amazon Machine Learning | aws-android-sdk-machinelearning |
| Amazon Mobile Analytics | aws-android-sdk-mobileanalytics |
| Amazon S3 | aws-android-sdk-s3 |
| Amazon Simple DB | aws-android-sdk-sdb |
| Amazon SES | aws-android-sdk-ses |
| Amazon SNS | aws-android-sdk-sns |
| Amazon SQS | aws-android-sdk-sqs |

<br>
* **jar파일 직접넣기** 
	
http://docs.aws.amazon.com/mobile/sdkforandroid/developerguide/Welcome.html<br>

여기서 **Download the AWS Mobile SDK for Android**부분에서 

'Download AWS Mobile SDK for Android (zip file)' 클릭하면 소스 전체를 다운받을 수 있는데

압축풀고 lib 폴더안에 jar파일들이 있다. (github에서 다운받아지는 거랑 다름)

이클립스를 사용중이라면 jar파일들을 내 프로젝트의 libs 폴더에 넣고, 안드로이드 스튜디오를 사용중이라면 apps/libs 폴더에 넣는다. 그러면 자동으로 빌드 패스에 포함시킬것이다.


### Add Import Statements 
내 메인 액티비티에 해당 클래스들을 import한다 
```
import com.amazonaws.auth.CognitoCachingCredentialsProvider;
import com.amazonaws.regions.Regions;
import com.amazonaws.services.dynamodbv2.*;
import com.amazonaws.mobileconnectors.dynamodbv2.dynamodbmapper.*;
import com.amazonaws.services.dynamodbv2.model.*;
```

### Set Permissions in Your Android Manifest
```
<uses-permission android:name="android.permission.INTERNET" />
```

### Create an Identity Pool
내 모바일 애플리케이션에 AWS Services를 사용하기 위해서는 내 credential provider로 Amazon Cognito Identity를 이용해서 AWS Credentials를 얻어야 한다. credentials provider를 사용하는 것은 내가 애플리케이션에서 pivate credentials를 내장하는거 없이 AWS Services에 접근을 허용한다. 이것은 또한 내 애플리케이션 사용자가 어떤 AWS Services를 접근하는지에 대한 권한을 통제 할수 있다. 

내 애플리케이션 사용자들의 identities는 identity pool에 의해 내 계정에 저장되고 관리된다. 모든 identity pool은 사용자가 접근할 수 있는 AWS resources를 지정하는 역할을 갖는다. 일반적으로 개발자는 한 애플리케이션에 한 identity pool을 사용한다. 

내 애플리케이션에 identity pool을 생성하기 위해서는 
1. https://console.aws.amazon.com/cognito/home 에 로그인해서 접속한다.
2. 'Create new identity pool' 버튼을 클릭한다.
![](identitypool0.PNG)
3. 'Identity pool name' 적고 'Unauthenticated identities'의 enable access to unauthenticated identities 체크박스를 체크하고 Create Pool 클릭한다. 
![](identitypool1.PNG)
4. Allow 누른다
![](identitypool2.PNG)

다음페이지는 credentials provider를 생성하는 코드를 보여준다. 그래서 너가 쉽게 안드로이드 애플리케이션에서 Cognito Identity를 통합 할 수 있다.
![](identitypool3.PNG)


###Create DynamoDB Table
이 튜토리얼을 위해 우리가 서점 앱을 만든다고 가정하자. Books table을 만들기 위해 </br>
1. https://console.aws.amazon.com/dynamodb/home 접속한다. (서버 지역 확인)
![](dbtable.PNG)
2. Create Table을 클릭한다.<br>

![](dbtable2.PNG)<br>
3. Table Name에 Books를 적는다.
4. Primary key type으로 Hash를 선택한다.
5. Hash Attribute Name에 ISBN적고 type은 String으로 한다. 그리고 continue 클릭

6. 인덱스는 hash와 range key 대체를 위한 데이터 구조이다.
7. 77
8. 88
9. 99
10. 110
11. 11
12. 12
13. 13
14. 14