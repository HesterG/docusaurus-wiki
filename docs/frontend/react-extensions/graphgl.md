

# mongoose + mongoDB Cloud + graphql

学习graphql的过程中发现用mongoDB Cloud真是省事啊，不用本地装mongodb了



## MongoDB Cloud

在mongodb cloud创建数据库：

1. 在mongoDB Cloud新建一个免费的shared cluster
2.  在Network Access中设置可以访问改cluster的ip的白名单

    [设置之前先查看一下当前public ip](https://www.leiyouxi.com/y/82687.html)
3. 在Database -> Connect -> Connect yoour application 获取连接到node的地址



## GraphQL概述

GraphQL 是一种用于[应用编程接口（API）](https://www.redhat.com/zh/topics/api/what-are-application-programming-interfaces)的查询语言和服务器端运行时，它可以使客户端准确地获得所需的数据，没有任何冗余。



## GraphQL有什么用？

GraphQL 旨在让 API 变得快速、灵活并且为开发人员提供便利。它甚至可以部署在名为 [GraphiQL](https://github.com/graphql/graphiql) 的[集成开发环境（IDE）](https://www.redhat.com/zh/topics/middleware/what-is-ide)中。作为 [REST](https://www.redhat.com/zh/topics/integration/whats-the-difference-between-soap-rest) 的替代方案，GraphQL 允许开发人员构建相应的请求，从而通过单个 API 调用从多个数据源中提取数据。

此外，GraphQL 还可让 API 维护人员灵活地添加或弃用字段，而不会影响现有查询。开发人员可以使用自己喜欢的方法来构建 API，并且 GraphQL 规范将确保它们以可预测的方式在客户端发挥作用。



## 在node中使用

使用`express-graphql`和`graphql`构建api，这里发现Schema的写法简直跟ts一模一样..

`server.js`代码:

```js
const express = require("express")
const {buildSchema} = require("graphql")
const graphqlHttp = require("express-graphql")
//-------------链接数据库服务------------------------
var mongoose = require("mongoose")
mongoose.connect("mongodb+srv://<username>:<password>@clusteraddr/?ssl=true&replicaSet=Main-shard-0&authSource=admin&retryWrites=true&w=majority", { useNewUrlParser: true,useUnifiedTopology: true })
.then((res) => {
    // console.log('res', res)
    console.log('链接成功')
}).catch((e) => {
    console.log('error', e)
})
// 限制 数据库这个films（集合表） 只能存三个字段
var FilmModel = mongoose.model("film", new mongoose.Schema({
    name: String,
    poster: String,
    price: Number
}))

// FilmModel.create
// filmModel.find
// FilmModel.update
// FimlModel.delete
//-------------------------------------

var Scchema = buildSchema(`
   type Film{
       id:String,
       name:String,
       poster:String,
       price:Int
   }

   input FilmInput{
        name:String,
        poster:String,
        price:Int
   }

   type Query{
       getNowplayingList(id:String!):[Film]
    }

    type Mutation{
        createFilm(input: FilmInput):Film,
        updateFilm(id:String!,input:FilmInput):Film,
        deleteFilm(id:String!):Int
    }
`)
//处理器
const root = {
    getNowplayingList({id}){
        if(!id) return FilmModel.find()
        return FilmModel.find({_id:id})
    },

    createFilm({input}){
        /*
          1. 创建模型
          2. 操作数据库
        */
        return FilmModel.create({
            ...input
        })
    },
    updateFilm({id,input}){
        return FilmModel.updateOne({
            _id:id
        },{
            ...input
        }).then(res=>FilmModel.find({_id:id})).then(res=>res[0])
    },

    deleteFilm({id}){
        return FilmModel.deleteOne({_id:id}).then(res=>1)
    }
}

var app = express()
app.use("/graphql",graphqlHttp({
    schema:Scchema,
    rootValue:root,
    graphiql:true
}))

//配置静态资源目录
app.use(express.static("public"))

app.listen(3000)
```

`nodemon server.js`后在`localhost:3000/graphql`打开可视化工具，可以查询和修改的例子：

```js
query {
  getNowplayingList(id:0) {
    id
  }
}
mutation {
  createFilm(input:
    {name: "12345678", 
      poster: "http://111123", 
      price: 123}) {
    id
  }
  updateFilm(input:{name:"112233"}, id:5) {
    id
  }
  deleteFilm(id: 6)
}
```





# React中使用GraphQL

- [链接](https://github.com/HesterG/React_Study/tree/main/codes-qf/myapp/src/15-graphql)