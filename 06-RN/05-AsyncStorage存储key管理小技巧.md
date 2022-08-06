## AsyncStorage存储key管理小技巧

### 场景

AsyncStorage是React Native推荐的数据存储方式。当我们需要根据条件从本地查询出多条记录时，你会想到来一个`select * from xx where xx`。但是很不幸的告诉你，AsyncStorage 是不支持sql的，因为AsyncStorage是Key-Value存储系统。

**那么如何才能快速的从众多记录中将符合条件的记录查询出来呢？** 请往下看…

### AsyncStorage key管理

为了方便查询多条符合规则的记录，我们可以在保存数据前，对这条数据进行分类，然后记录下这条记录的key。下次再查询该数据前，只需要先查询之前保存的key，然后通过 `static multiGet(keys, callback?) `API，将符合规则的数据一并查询出来。

### 用例

> 保存数据

**第一步：保存数据**

```react
  saveFavoriteItem(key,vaule,callback) {
    AsyncStorage.setItem(key,vaule,(error,result)=>{
      if (!error) {//更新Favorite的key
        this.updateFavoriteKeys(key,true);
      }
    });
  }
```

**第二步：更新key**

```react
/**
* 更新Favorite key集合
* @param isAdd true 添加,false 删除
* **/
  updateFavoriteKeys(key,isAdd){
    AsyncStorage.getItem(this.favoriteKey,(error,result)=>{
      if (!error) {
        var favoriteKeys=[];
        if (result) {
          favoriteKeys=JSON.parse(result);
        }
        var index=favoriteKeys.indexOf(key);
        if(isAdd){
          if (index===-1)favoriteKeys.push(key);
        }else {
          if (index!==-1)favoriteKeys.splice(index, 1);
        }
        AsyncStorage.setItem(this.favoriteKey,JSON.stringify(favoriteKeys));
      }
    });
  }
```

> 查询批量数据

**第一步：查询key**

```react
getFavoriteKeys(){//获取收藏的Respository对应的key
    return new Promise((resolve,reject)=>{
      AsyncStorage.getItem(this.favoriteKey,(error,result)=>{
        if (!error) {
          try {
            resolve(JSON.parse(result));
          } catch (e) {
            reject(error);
          }
        }else {
          reject(error);
        }
      });
    });
  }
```

**第二步：根据key查询数据**

```react
AsyncStorage.multiGet(keys, (err, stores) => {
    try {
      stores.map((result, i, store) => {
        // get at each store's key/value so you can work with it
        let key = store[i][0];
        let value = store[i][1];
        if (value)items.push(JSON.parse(value));
      });
      resolve(items);
    } catch (e) {
      reject(e);
    }
 });
```
