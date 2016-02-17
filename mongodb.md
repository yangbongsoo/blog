# MongoDB Driver Quick Tour

###Make a Connection
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

###Get a Collection
collection을 가져올땐 getCollection() 메서드에 collection 이름을 넣어라.
`MongoCollection<Document> collection = database.getCollection("test");`

###Insert a Document
collection 객체를 가질때, collection에 documents를 삽입할 수 있다. 예를 들어, 다음 JSON document를 생각해봐라; 그 document는 `info` 필드로 또 하나의 document를 포함하고 있다.
```
{
   "name" : "MongoDB",
   "type" : "database",
   "count" : 1,
   "info" : {
               x : 203,
               y : 102
             }
}
```
자바 드라이버를 이용해 그 document를 생성하기 위해서는 Document 클래스를 사용해라. 이 클래스로 내장된 document 또한 생성할 수 있다. 

```
Document doc = new Document("name", "MongoDB")
               .append("type", "database")
               .append("count", 1)
               .append("info", new Document("x", 203).append("y", 102));
```
collection에 document를 삽입하기 위해서는 insertOne() 메서드를 사용해라. `collection.insertOne(doc);`
###Add Multiple Documents
다수의  document들을 추가하기 위해서 insertMany() 메서드를 사용한다. 

다음 예제는 다수의 document들을 추가한다. 
`{ "i" : value }`<br>
루프에서 document들을 생성한다. 
```
List<Document> documents = new ArrayList<Document>();
for (int i = 0; i < 100; i++) {
    documents.add(new Document("i", i));
}

collection.insertMany(documents);
```

###Count Documents in A Collection
지금까지 101개의 document들을 삽입했다.(100개는 루프를 통해서, 아까 한개 추가한거까지) count() 메서드를 통해서 확인해볼 수 있다. 다음 예제 코드는 101이 나와야한다. 
`System.out.println(collection.count());`

**Query the Collection**<br>
find() 메서드로 collection을 쿼리한다. 
