# [IndexDB](https://developer.mozilla.org/zh-CN/docs/Web/API/IndexedDB_API)
IndexedDB 不属于关系型数据库（不支持 SQL 查询语句），更接近 NoSQL 数据库。是一种底层 API，用于在客户端存储大量的结构化数据。
- 键值对存储
  - 所有类型的数据都可以直接存入
  - 主键必须是唯一的
- 异步
  - 不会卡死浏览器
- 支持事务
  - 操作失败整个事务都取消，不存在只改写一部分数据的情况
- 同源限制
- 存储空间大
  - 一般不小于250MB
- 支持二进制存储

## [API接口](https://wangdoc.com/javascript/bom/indexeddb.html#indexeddb-%E5%AF%B9%E8%B1%A1)
- 数据库：IDBDatabase 对象
  - IndexedDB 数据库有版本的概念。同一个时刻，只能有一个版本的数据库存在。如果要修改数据库结构（新增或删除表、索引或者主键），只能通过升级数据库版本完成。
- 对象仓库：IDBObjectStore 对象
  - 每个数据库包含若干个对象仓库（object store）。它类似于关系型数据库的表格。
  - 对象仓库保存的是数据记录。每条记录类似于关系型数据库的行，但是只有主键和数据体两部分。主键用来建立默认的索引，必须是不同的，否则会报错。主键可以是数据记录里面的一个属性，也可以指定为一个递增的整数编号。
- 索引： IDBIndex 对象
  - 为了加速数据的检索，可以在对象仓库里面，为不同的属性建立索引。
- 事务： IDBTransaction 对象
  - 数据记录的读写和删改，都要通过事务完成。事务对象提供error、abort和complete三个事件，用来监听操作结果。
- 操作请求：IDBRequest 对象
- 指针： IDBCursor 对象
- 主键集合：IDBKeyRange 对象

## IDBTransaction 对象
IDBTransaction 对象用来异步操作数据库事务，所有的读写操作都要通过这个对象进行。  
IDBDatabase.transaction()方法返回的就是一个IDBTransaction 对象。  
事务的执行顺序是按照创建的顺序，而不是发出请求的顺序。  
**IDBTransaction 对象有以下属性**
- IDBTransaction.db：返回当前事务所在的数据库对象 IDBDatabase。
- IDBTransaction.error：返回当前事务的错误。如果事务没有结束，或者事务成功结束，或者被手动终止，该方法返回null。
- IDBTransaction.mode：返回当前事务的模式，默认是readonly（只读），另一个值是readwrite。
- IDBTransaction.objectStoreNames：返回一个类似数组的对象 DOMStringList，成员是当前事务涉及的对象仓库的名字。
- IDBTransaction.onabort：指定abort事件（事务中断）的监听函数。
- IDBTransaction.oncomplete：指定complete事件（事务成功）的监听函数。
- IDBTransaction.onerror：指定error事件（事务失败）的监听函数。
**IDBTransaction 对象有以下方法**
- IDBTransaction.abort()：终止当前事务，回滚所有已经进行的变更。
- IDBTransaction.objectStore(name)：返回指定名称的对象仓库 IDBObjectStore。

## IDBIndex 对象
IDBIndex 对象代表数据库的索引，通过这个对象可以获取数据库里面的记录。数据记录的主键默认就是带有索引，IDBIndex 对象主要用于通过除主键以外的其他键，建立索引获取对象。  
IDBIndex 是持久性的键值对存储。只要插入、更新或删除数据记录，引用的对象库中的记录，索引就会自动更新。  
``IDBObjectStore.index()``方法可以获取 IDBIndex 对象。  
**IDBIndex 对象有以下属性**
- IDBIndex.name：字符串，索引的名称。
- IDBIndex.objectStore：索引所在的对象仓库。
- IDBIndex.keyPath：索引的主键。
- IDBIndex.multiEntry：布尔值，针对keyPath为数组的情况，如果设为true，创建数组时，每个数组成员都会有一个条目，否则每个数组都只有一个条目。
- IDBIndex.unique：布尔值，表示创建索引时是否允许相同的主键。
**IDBIndex 对象有以下方法，它们都是异步的，立即返回的都是一个 IDBRequest 对象。**
- IDBIndex.count()：用来获取记录的数量。它可以接受主键或 IDBKeyRange 对象作为参数，这时只返回符合主键的记录数量，否则返回所有记录的数量。
- IDBIndex.get(key)：用来获取符合指定主键的数据记录。
- IDBIndex.getKey(key)：用来获取指定的主键。
- IDBIndex.getAll()：用来获取所有的数据记录。它可以接受两个参数，都是可选的，第一个参数用来指定主键，第二个参数用来指定返回记录的数量。如果省略这两个参数，则返回所有记录。由于获取成功时，浏览器必须生成所有对象，所以对性能有影响。如果数据集比较大，建议使用 IDBCursor 对象。
- IDBIndex.getAllKeys()：该方法与IDBIndex.getAll()方法相似，区别是获取所有主键。
- IDBIndex.openCursor()：用来获取一个 IDBCursor 对象，用来遍历索引里面的所有条目。
- IDBIndex.openKeyCursor()：该方法与IDBIndex.openCursor()方法相似，区别是遍历所有条目的主键。

## IDBCursor 对象 
IDBCursor 对象代表指针对象，用来遍历数据仓库（IDBObjectStore）或索引（IDBIndex）的记录。  
DBCursor 对象一般通过``IDBObjectStore.openCursor()``方法获得。
**IDBCursor 对象的属性**
- IDBCursor.source：返回正在遍历的对象仓库或索引。
- IDBCursor.direction：字符串，表示指针遍历的方向。共有四个可能的值：next（从头开始向后遍历）、nextunique（从头开始向后遍历，重复的值只遍历一次）、prev（从尾部开始向前遍历）、prevunique（从尾部开始向前遍历，重复的值只遍历一次）。该属性通过IDBObjectStore.openCursor()方法的第二个参数指定，一旦指定就不能改变了。
- IDBCursor.key：返回当前记录的主键。
- IDBCursor.value：返回当前记录的数据值。
- IDBCursor.primaryKey：返回当前记录的主键。对于数据仓库（objectStore）来说，这个属性等同于 IDBCursor.key；对于索引，IDBCursor.key 返回索引的位置值，该属性返回数据记录的主键。
**IDBCursor 对象有如下方法**
- IDBCursor.advance(n)：指针向前移动 n 个位置。
- IDBCursor.continue()：指针向前移动一个位置。它可以接受一个主键作为参数，这时会跳转到这个主键。
- IDBCursor.continuePrimaryKey()：该方法需要两个参数，第一个是key，第二个是primaryKey，将指针移到符合这两个参数的位置。
- IDBCursor.delete()：用来删除当前位置的记录，返回一个 IDBRequest 对象。该方法不会改变指针的位置。
- IDBCursor.update()：用来更新当前位置的记录，返回一个 IDBRequest 对象。它的参数是要写入数据库的新的值。

## IDBKeyRange 对象
IDBKeyRange 对象代表数据仓库（object store）里面的一组主键。根据这组主键，可以获取数据仓库或索引里面的一组记录。  
IDBKeyRange 可以只包含一个值，也可以指定上限和下限。它有四个静态方法，用来指定主键的范围。
- ``IDBKeyRange.lowerBound()``：指定下限。
- ``IDBKeyRange.upperBound()``：指定上限。
- ``IDBKeyRange.bound()``：同时指定上下限。
- ``IDBKeyRange.only()``：指定只包含一个值。
```
// All keys ≤ x
var r1 = IDBKeyRange.upperBound(x);

// All keys < x
var r2 = IDBKeyRange.upperBound(x, true);

// All keys ≥ y
var r3 = IDBKeyRange.lowerBound(y);

// All keys > y
var r4 = IDBKeyRange.lowerBound(y, true);

// All keys ≥ x && ≤ y
var r5 = IDBKeyRange.bound(x, y);

// All keys > x &&< y
var r6 = IDBKeyRange.bound(x, y, true, true);

// All keys > x && ≤ y
var r7 = IDBKeyRange.bound(x, y, true, false);

// All keys ≥ x &&< y
var r8 = IDBKeyRange.bound(x, y, false, true);

// The key = z
var r9 = IDBKeyRange.only(z);
```
``IDBKeyRange.lowerBound()、IDBKeyRange.upperBound()、IDBKeyRange.bound()``这三个方法默认包括端点值，可以传入一个布尔值，修改这个属性。  
**与之对应，IDBKeyRange 对象有四个只读属性**
- IDBKeyRange.lower：返回下限
- IDBKeyRange.lowerOpen：布尔值，表示下限是否为开区间（即下限是否排除在范围之外）
- IDBKeyRange.upper：返回上限
- IDBKeyRange.upperOpen：布尔值，表示上限是否为开区间（即上限是否排除在范围之外）
IDBKeyRange 实例对象生成以后，将它作为参数输入 IDBObjectStore 或 IDBIndex 对象的openCursor()方法，就可以在所设定的范围内读取数据。
```
var t = db.transaction(['people'], 'readonly');
var store = t.objectStore('people');
var index = store.index('name');

var range = IDBKeyRange.bound('B', 'D');

index.openCursor(range).onsuccess = function (e) {
  var cursor = e.target.result;
  if (cursor) {
    console.log(cursor.key + ':');

    for (var field in cursor.value) {
      console.log(cursor.value[field]);
    }
    cursor.continue();
  }
}
```
IDBKeyRange 有一个实例方法``includes(key)``，返回一个布尔值，表示某个主键是否包含在当前这个主键组之内。
```
var keyRangeValue = IDBKeyRange.bound('A', 'K', false, false);

keyRangeValue.includes('F') // true
keyRangeValue.includes('W') // false
```

## 操作
### 打开数据库
``indexedDB.open()``方法用于打开数据库。这是一个异步操作，但是会立刻返回一个 IDBOpenDBRequest 对象。  
```
var openRequest = window.indexedDB.open('test', 1);
```
上面代码表示，打开一个名为test、版本为1的数据库。如果该数据库不存在，则会新建该数据库。  
open()方法的第一个参数是数据库名称，格式为字符串，不可省略；第二个参数是数据库版本，是一个大于0的正整数（0将报错），如果该参数大于当前版本，会触发数据库升级。第二个参数可省略，如果数据库已存在，将打开当前版本的数据库；如果数据库不存在，将创建该版本的数据库，默认版本为1。  
``indexedDB.open()``方法返回一个 IDBRequest 对象。这个对象通过各种事件通知客户端。下面是有可能触发的4种事件。
- success：打开成功。
- error：打开失败。
- upgradeneeded：第一次打开该数据库，或者数据库版本发生变化。
- blocked：上一次的数据库连接还未关闭。
```
var openRequest = indexedDB.open('test', 1);
var db;

openRequest.onupgradeneeded = function (e) {
  console.log('Upgrading...');
  db = event.target.result;
  var objectStore;
  // 自动生成主键{ autoIncrement: true }
  if (!db.objectStoreNames.contains('person')) {
    objectStore = db.createObjectStore('person', { keyPath: 'id' });
    // 新建对象仓库以后，下一步可以新建索引。
    // createIndex()的三个参数分别为索引名称、索引所在的属性、配置对象（说明该属性是否包含重复的值）。
    objectStore.createIndex('name', 'name', { unique: false });
    objectStore.createIndex('email', 'email', { unique: true });
  }
}

openRequest.onsuccess = function (e) {
  console.log('Success!');
  db = openRequest.result;
}

openRequest.onerror = function (e) {
  console.log('Error');
  console.log(e);
}
```
### 新建数据库
新建数据库与打开数据库是同一个操作。如果指定的数据库不存在，就会新建。不同之处在于，后续的操作主要在upgradeneeded事件的监听函数里面完成，因为这时版本从无到有，所以会触发这个事件。

### 新增数据
``objectStore.add(value, key)``
该方法接受两个参数，第一个参数是键值，第二个参数是主键，该参数可选，如果省略默认为null。  
该方法只用于添加数据，如果主键相同会报错，因此更新数据必须使用put()方法。  
创建事务以后，就可以获取对象仓库，然后使用add()方法往里面添加数据了。
```
var db;
var DBOpenRequest = window.indexedDB.open('demo', 1);

DBOpenRequest.onsuccess = function (event) {
  db = DBOpenRequest.result;
  var transaction = db.transaction(['items'], 'readwrite');

  transaction.oncomplete = function (event) {
    console.log('transaction success');
  };

  transaction.onerror = function (event) {
    console.log('transaction error: ' + transaction.error);
  };

  var objectStore = transaction.objectStore('items');
  var objectStoreRequest = objectStore.add({ foo: 1 });

  objectStoreRequest.onsuccess = function (event) {
    console.log('add data success');
  };

};
```
### 更新数据
IDBObjectStore.put()方法用于更新某个主键对应的数据记录，如果对应的键值不存在，则插入一条新的记录。该方法返回一个 IDBRequest 对象。
``objectStore.put(item, key)``
该方法接受两个参数，第一个参数为新数据，第二个参数为主键，该参数可选，且只在自动递增时才有必要提供，因为那时主键不包含在数据值里面。
```
function update() {
  var request = db.transaction(['person'], 'readwrite')
    .objectStore('person')
    .put({ id: 1, name: '李四', age: 35, email: 'lisi@example.com' });

  request.onsuccess = function (event) {
    console.log('数据更新成功');
  };

  request.onerror = function (event) {
    console.log('数据更新失败');
  }
}

update();
```
上面代码中，put()方法自动更新了主键为1的记录。

### 读取数据
读取数据也是通过事务完成。
``IDBObjectStore.get()``
用于获取主键对应的数据记录。该方法返回一个 IDBRequest 对象。  
``IDBObjectStore.getAll()``
用于获取对象仓库的记录。该方法返回一个 IDBRequest 对象。  
```
// 获取所有记录
objectStore.getAll()

// 获取所有符合指定主键或 IDBKeyRange 的记录
objectStore.getAll(query)

// 指定获取记录的数量
objectStore.getAll(query, count)
```
### 删除数据
``IDBObjectStore.delete()``
删除指定主键的记录。该方法返回一个 IDBRequest 对象。
``IDBObjectStore.clear()``
删除当前对象仓库的所有记录。该方法返回一个 IDBRequest 对象。
### 遍历数据
``IDBObjectStore.openCursor()``
获取一个指针对象。可以用来遍历数据。该对象也是异步的，有自己的success和error事件，可以对它们指定监听函数。
```
var t = db.transaction(['test'], 'readonly');
var store = t.objectStore('test');

var cursor = store.openCursor();

cursor.onsuccess = function (event) {
  var res = event.target.result;
  if (res) {
    console.log('Key', res.key);
    console.dir('Data', res.value);
    res.continue();
  }
}
```
监听函数接受一个事件对象作为参数，该对象的target.result属性指向当前数据记录。该记录的key和value分别返回主键和键值（即实际存入的数据）。continue()方法将光标移到下一个数据对象，如果当前数据对象已经是最后一个数据了，则光标指向null。  
openCursor()方法的第一个参数是主键值，或者一个 IDBKeyRange 对象。如果指定该参数，将只处理包含指定主键的记录；如果省略，将处理所有的记录。该方法还可以接受第二个参数，表示遍历方向，默认值为next，其他可能的值为prev、nextunique和prevunique。后两个值表示如果遇到重复值，会自动跳过。
### 使用索引
假定新建表格的时候，对name字段建立了索引
``objectStore.createIndex('name', 'name', { unique: false });``
现在，就可以从name找到对应的数据记录了。
```
var transaction = db.transaction(['person'], 'readonly');
var store = transaction.objectStore('person');
var index = store.index('name');
var request = index.get('李四');

request.onsuccess = function (e) {
  var result = e.target.result;
  if (result) {
    // ...
  } else {
    // ...
  }
}
```
### count
``IDBObjectStore.count(key)``方法用于计算记录的数量。该方法返回一个 IDBRequest 对象。  
不带参数时，该方法返回当前对象仓库的所有记录数量。如果主键或 IDBKeyRange 对象作为参数，则返回对应的记录数量。
### key
``IDBObjectStore.getKey()``用于获取主键。该方法返回一个 IDBRequest 对象。  
``IDBObjectStore.getAllKeys()``用于获取所有符合条件的主键。该方法返回一个 IDBRequest 对象。
```
// 获取所有记录的主键
objectStore.getAllKeys()

// 获取所有符合条件的主键
objectStore.getAllKeys(query)

// 指定获取主键的数量
objectStore.getAllKeys(query, count)
```
### createIndex
``IDBObjectStore.createIndex()``方法用于新建当前数据库的一个索引。该方法只能在VersionChange监听函数里面调用。  
``objectStore.createIndex(indexName, keyPath, objectParameters)``
- indexName：索引名
- keyPath：主键
- objectParameters：配置对象（可选）
  - unique：如果设为true，将不允许重复的值
  - multiEntry：如果设为true，对于有多个值的主键数组，每个值将在索引里面新建一个条目，否则主键数组对应一个条目。

```
var person = {
  name: name,
  email: email,
  created: new Date()
};
// ...
var store = db.createObjectStore('people', { autoIncrement: true });

store.createIndex('name', 'name', { unique: false });
store.createIndex('email', 'email', { unique: true });
```
上面代码告诉索引对象，name属性不是唯一值，email属性是唯一值。
### deleteIndex
``IDBObjectStore.deleteIndex()``方法用于删除指定的索引。该方法只能在VersionChange监听函数里面调用。
### index
``IDBObjectStore.index()``方法返回指定名称的索引对象 IDBIndex。
```
var t = db.transaction(['people'], 'readonly');
var store = t.objectStore('people');
var index = store.index('name');

var request = index.get('foo');
```
上面代码打开对象仓库以后，先用index()方法指定获取name属性的索引，然后用get()方法读取某个name属性(foo)对应的数据。如果name属性不是对应唯一值，这时get()方法有可能取回多个数据对象。另外，get()是异步方法，读取成功以后，只能在success事件的监听函数中处理数据。
