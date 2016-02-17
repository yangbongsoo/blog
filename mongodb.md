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
<br>(중요)<br>
전형적으로 주어진 데이터베이스 클러스터를 위해 하나의 MongoClient 인스턴스를 생성하고 너의 애플리케이션에 사용한다. 다수의 인스턴스를 생성할때:<br>
- 모든 리소스 사용 한계(최대 연결수, 등등)는 MongoClient 인스턴스 당 적용한다. 
- 인스턴스를 처분할때, 너는 자원을 클린업하기 위해서 MongoClient.close()를 콜해야 한다.  

**Get a Collection**<br>
collection을 가져올땐 getCollection() 메서드에 collection 이름을 넣어라.
`MongoCollection<Document> collection = database.getCollection("test");`

**Insert a Document**<br>
Once you have the collection object, you can insert documents into the collection. 
collection 객체를 가질때, collection에 documents를 삽입할 수 있다. 예를 들어, 다음 JSON document를 생각해봐라; 그 document는 `info` 필드로 또 하나의 document를 포함하고 있다.
