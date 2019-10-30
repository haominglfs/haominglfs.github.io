---
title: mongodb使用
date: 2018-12-20 10:26:13
tags:
---

```shell
db.students.find({age:20,name:'sss'})

db.students.find({age:20})
db.students.findOne({age:20}).name

db.students.find()

db.students.insertOne({name:"mmm",age:20})

db.students.find({age:20}).count()


db.students.find({age:20}).length()

db.students.update({name:"hm"},{age:28})

//默认情况只会修改一个
db.students.update({"_id" : ObjectId("5c17c5836ff3da5ffa6da9f7")},{$set:{name:'hm',gender:"男",address:"石家庄"}})


db.students.update({"_id" : ObjectId("5c17c5836ff3da5ffa6da9f7")},{$unset:{address:"石家庄"}})


db.students.remove({age:20})

db.students.remove({})

db.students.drop()



for(var i=1;i<10;i++){
    db.logs.insert({name:i})
}

//var list = db.logs.find();
//printjson(list)

db.logs.find().sort({$natural:-1})

db.logs.insert({name:undefined})

//db.logs.find({name:undefined})

db.logs.find({name:{$type:6}})

db.logs.find({name:{$type:"undefined"}})
```

#### mongoTemplate使用(kotlin)

```kotlin
//根据id查询
mongoTemplate.findById(oid, Place::class.java)
//保存文档对象
mongoTemplate.save(place)
//构造查询对象
val pattern= Pattern.compile("^http", Pattern.CASE_INSENSITIVE) //匹配正则表达式
val query = Query(Criteria.where("coverPicture").regex(pattern))
query.with(PageRequest.of(page, 30)) //分页处理
val items = mongoTemplate.find(query, Place::class.java)
```

