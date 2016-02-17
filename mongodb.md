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
collection을 가져올땐 `getCollection()` 메서드에 collection 이름을 넣어라.
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
collection에 document를 삽입하기 위해서는 `insertOne()` 메서드를 사용해라. `collection.insertOne(doc);`
###Add Multiple Documents
다수의  document들을 추가하기 위해서 `insertMany()` 메서드를 사용한다. 

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
지금까지 101개의 document들을 삽입했다.(100개는 루프를 통해서, 아까 한개 추가한거까지) `count()` 메서드를 통해서 확인해볼 수 있다. 다음 예제 코드는 101이 나와야한다. 
`System.out.println(collection.count());`

###Query the Collection
`find()` 메서드로 collection을 쿼리한다.<br>
**Find the First Document in a Collection**<br>
collection에 첫 번째 document를 가져오기 위해서 `find()`명령의 `first()`메서드를 콜해라. `collection.find().first()`는 첫 번째 document나 null(커서보다)을 리턴한다. 이것은 단일 document 매치여부 쿼리나 첫 번째 document를 가져올때 유용하다. 

다음은 collection의 첫 번째 document를 찾아 출력하는 예제다. 
```
Document myDoc = collection.find().first();
System.out.println(myDoc.toJson());
```
이 예제는 다음의 document를 출력한다. 
```
{ "_id" : { "$oid" : "551582c558c7b4fbacf16735" },
  "name" : "MongoDB", "type" : "database", "count" : 1,
  "info" : { "x" : 203, "y" : 102 } }
```
NOTE : `_id` 요소는 MongoDB가 자동으로 추가해왔고 값은 보여지는 것과 다를 것이다. MongoDB는 내부적인 사용을 위해 "_"와 "$"로 시작하는 필드명을 남겨둔다. 

**Find All Documents in a Collection**<br>
collection의 모든 document들을 검색하기 위해서 `find()`메서드를 사용한다. `find()`메서드는 검색 작업의 체이닝이나 컨트롤링 인터페이스를 제공하는 FindIterable 인스턴스를 리턴한다. `iterator()` 메서드를 사용해서 document들의 iterator를 가져온다. 다음 코드는 collection의 모든 document들을 검색하고 출력한다. 
```
MongoCursor<Document> cursor = collection.find().iterator();
try {
    while (cursor.hasNext()) {
        System.out.println(cursor.next().toJson());
    }
} finally {
    cursor.close();
}
```
Although the following idiom is permissible, its use is discouraged as the application can leak a cursor if the loop terminates early:

