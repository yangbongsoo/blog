# MongoDB

**Make a Connection**<br>
다음 예제는 로컬 머신에 `mydb` DB를 연결하는 5가지 방법이다. 만약 DB가 없다면 MongoDB는 알아서 만들어준다.
```
// To directly connect to a single MongoDB server
// (this will not auto-discover the primary even if it's a member of a replica set)
MongoClient mongoClient = new MongoClient();

// or
MongoClient mongoClient = new MongoClient( "localhost" );

// or
MongoClient mongoClient = new MongoClient( "localhost" , 27017 );

// or, to connect to a replica set, with auto-discovery of the primary, supply a seed list of members
MongoClient mongoClient = new MongoClient(
  Arrays.asList(new ServerAddress("localhost", 27017),
                new ServerAddress("localhost", 27018),
                new ServerAddress("localhost", 27019)));

// or use a connection string
MongoClientURI connectionString = new MongoClientURI("mongodb://localhost:27017,localhost:27018,localhost:27019");
MongoClient mongoClient = new MongoClient(connectionString);

MongoDatabase database = mongoClient.getDatabase("mydb");
```
이 시점에서 database 객체는 MongoDB 서버에 연결이 된다.

**MongoClient**<br>
MongoClient 인스턴스는 사실 데이터베이스 연결의 풀을 대표한다. 그리고 다수의 스레드 환경에서도 MongoClient 인스턴스는 하나만 있으면 된다. 