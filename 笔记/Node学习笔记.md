# Node学习笔记

## 1.文件读写

### 1.1文件读取

```js
//加载文件读写模块
let fs=require("fs");
//文件不存在时报错
fs.readFile('hello1.txt', {encoding:"utf-8",flag:"r"},(err, data) => {
    if (err) throw err;
    console.log("这是hello1.txt的文件内容:"+data);  //没有设置编码返回 buffer
});
```

### 1.2文件写入

```js
let fs=require("fs")
//异步写入
// a=append 追加写入  没有文件时自动创建  w=write 会覆盖掉原来内容
//文件不存在时会创建
fs.writeFile("write.txt","今天吃饭了吗\n",{flag:"w"},err=>{
    if(err){
        console.log(err)
    }else{
        console.log("写入成功！")
    }
})
```

### 1.3文件删除

```js
// fs.unlink
//注意！！！删除文件后无法在回收站找回
let fs=require("fs")
//先自动创建3秒后删除
fs.writeFile("delete.txt","测试删除\n",{flag:"w"},err=>{
    if(err){
        console.log(err)
    }else{
        console.log("写入成功！")
        //删除文件 三秒后删除
        setTimeout(()=>{
            fs.unlink('delete.txt', (err) => {
                if (err) throw err;
                console.log('已成功地删除文件 delete.txt');
            });
        },3000)     
    }
})
```

## 2.Buffer

> 目前阶段基本不会用到

```js
// 1 数据不能进行二进制操作
// 2 js数组不像其他语言 java Python 效率高
// 3 buffer内存空间开辟出固定大小的内存

// Buffer.from(array)
// 使用 0 – 255 范围内的字节数组 array 来分配一个新的 Buffer。 超出该范围的数组条目会被截断以适合它。
var str="hello world"
let buf=Buffer.from(str)
//打印出buffer中的内容
console.log(buf.toString())

//开辟一个空的buffer(缓冲区)
// Buffer.alloc(size[, fill[, encoding]])
// size <integer> 新 Buffer 的期望长度。
// fill <string> | <Buffer> | <Uint8Array> | <integer> 用于预填充新 Buffer 的值。默认值: 0。
// encoding <string> 如果 fill 是一个字符串，则这是它的字符编码。默认值: 'utf8'。

//开辟10字节的空间  连续空间
let buf2=Buffer.alloc(10)
buf2[0]=255   //设置第一个字节的内容 最大255
console.log(buf2)

//不安全的开辟  但是效率大于 上面的普通开辟方法
//不安全是因为它直接覆盖已有数据(可能会提取关键数据) 而上面的普通方法会全部清除
let buf3=Buffer.allocUnsafe(10)
buf2[0]=255   //设置第一个字节的内容 最大255
console.log(buf3)

//buffer在学习过程中不会用到 js写神经网络会用到
```

## 3.文件夹操作

### 封装模块

**myModule.js**

```js
// 我的模块
let fs=require("fs");

//读取文件内容函数
function fsRead(path){
    return new Promise((resolve,reject)=>{
        if(!fs.existsSync(path)){   //判断是否存在指定文件
            fs.writeFileSync(path,"")  //写一个空文件
        }
        fs.readFile(path, {encoding:"utf-8",flag:"r"},(err, data) => {
            if (err) reject(err);
            resolve(data);
        });
    })
}
//文件写入函数
function fsWrite(path,content){
    return new Promise((resolve,reject)=>{
        //此处注意 回调函数只有一个参数
        fs.writeFile(path,content,{flag:"a"},err=>{
            if(err){
                reject(err)
            }else{
                resolve(err)
                console.log("写入成功！")
            }
        })
    })
}
//导出函数
module.exports={
    fsRead,
    fsWrite
}
```



### 3.1读取文件夹内容

```js
// fs.readdir(path[, options], callback)

//导入外部模块
let {fsRead,fsWrite}=require("./myModule") 
let fs=require("fs")

//封装读取文件夹方法
function fsReadDir(path){
    return new Promise((resolve,reject)=>{
        fs.readdir(path,(err,file)=>{
            if(err){
                reject(err)
            }else{             
                resolve(file)               
            }
        })
    })
}

//读取文件夹中所有内容并写到一个新文件中
async function myF(){
    let files=await fsReadDir("../node1")
    console.log(files)  //打印文件夹内容数组
    files.forEach(async function(item) {
        let content=await fsRead("../node1/"+item)
        await fsWrite("allText.txt",content)
    });
}
myF()
```

### 3.2删除一个文件夹

```js
// fs.rmdir(path[, options], callback)
//删除文件夹    注意！！！回收站没有！
let fs=require("fs")
//判断文件夹是否存在
if(fs.existsSync("./test")){
    console.log("test文件夹已经存在！")
}else{
    console.log("test文件夹不存在！")
    console.log("正在创建test文件夹！")
    fs.mkdirSync("./test")
    console.log("创建成功!")
}
console.log("3S后删除test文件夹")
setTimeout(()=>{
    fs.rmdir("./test",err=>{
        if(err){
            console.log("删除错误！"+err)
        }else{
            console.log("删除成功！")
        }
    })
},3000)
```

## 4.Node控制台输入输出

```js
//在终端输入内容
//导入readline 模块
let readline = require("readline")
//导入自定义模块
const { fsWrite, fsRead } = require("./myModule")

//实现接口
const rl1 = readline.createInterface({
    input: process.stdin,
    output: process.stdout
})
function setQuestions(question) {
    return new Promise(resolve => {
        //设置自定义问题
        rl1.question(question, answer => {
            resolve(answer)
        })
    })
}
//判断数据是否为json
function isJsonString(str) {
    try {
        if (typeof JSON.parse(str) == "object") {
            return true;
        }
    } catch (e) {
    }
    return false;
}
//操作方法
async function myF() {
    let name = await setQuestions("你的名字是？")
    let age = await setQuestions("你的年龄是？")
    let sex = await setQuestions("你的性别是？")
    //存放得到的数据
    let cont = {name,age,sex}
    let info = await fsRead("./info.json")
    //判断是否为json 空文件不为json
    if (isJsonString(info)) {
        info=JSON.parse(info)
        cont=[...info,cont]
    }else if(info===""){
        //把存放的数据放到数组里
        cont=[cont]
    }
    console.log("已存在的内容：\n")
    console.log(info)
    //序列化对象
    content = JSON.stringify(cont)
    //写入json文件
    await fsWrite("./info.json", content)
    //关闭终端
    rl1.close()
}
//执行函数
myF()

//可以监听close()事件
rl1.on("close", () => {
    console.log("程序结束！")
    //结束程序
    process.exit(0)
})
```

## 5.Stream流的简单操作

### 5.1读取写入流

```js
//文件输入输出流
//适用于较大的文件操作
// fs.createReadStream(path[, options])
let fs=require("fs")
//绝对路径注意转义符\的使用
const rs=fs.createReadStream("E:\\QQ音乐\\MV\\BLACKPINK - PLAYING WITH FIRE.avi.mp4")
//创建写入流 不存在时自动生成
const ws=fs.createWriteStream("./b.mp4")

//监听读取流读取的每部分数据
rs.on("data",function(chunk){
    console.log(chunk)
    //把读取的数据写入指定文件中
    ws.write(chunk,()=>{
        console.log("一批数据写入完成！")
    })
})

//读取流监听函数
rs.on("open",function(){
    console.log("文件读取流开启")
})
rs.on("ready",function(){
    console.log("文件读取流准备")
})
rs.on("end",function(){
    console.log("文件读取流读取完毕")
})
rs.on("close",function(){
    console.log("文件读取流关闭")
    //写入流结束
    ws.end()  //触发写入流的finish监听事件
})

//写入流监听
ws.on('finish', () => {
    console.log('写入已完成');
});
ws.on("close",function(){
    console.log("文件写入流关闭")
})
```

### 5.2使用pipe管道流简化上面的代码

```js
//使用管道流 简化流程

let fs=require("fs")
//绝对路径注意转义符\的使用
const rs=fs.createReadStream("E:\\QQ音乐\\MV\\BLACKPINK - PLAYING WITH FIRE.avi.mp4")
//创建写入流
const ws=fs.createWriteStream("./c.mp4")

//读取流的管道加入 写入流
rs.pipe(ws)   //一行代码即可完成读取写入    注意调用位置！！！

//读取流监听函数
rs.on("close",function(){
    console.log("文件读取流关闭")
     //写入流结束
    ws.end()
})

//写入流监听
ws.on('finish', () => {
    console.log('写入已完成');
});
ws.on("close",function(){
    console.log("文件写入流关闭")
})
```

## 6.Node中的events

### 6.1设计模式  事件订阅的简单原理实现

```js
//js 设计模式 事件订阅的原理
let fs=require("fs")

fs.readFile("../info.json",{encoding:"utf-8"},(err,data)=>{
    if(err){
        console.log(err)
        return
    }
    // 1. 做第一件事
    // 2. 做第二件事
    // 3. 做第三件事
    //这里触发事件  类似于vue中的子向父传值
    myEvent.emit("fileSuccess",data)
})

let myEvent={
    //创建事件队列,存放监听的事件
    eventList:{
        //fileSuccess:[fn,fn,fn]
    },
    on:function(eventName,fn){
        //如果事件队列中有这个事件名
        if(this.eventList[eventName]){
            //把回调函数加入事件队列
            this.eventList[eventName].push(fn)
        }else{
            //如果没有就创建一个新的事件名key 并加入传过来的回调函数
            this.eventList[eventName]=[fn]
        }
    },
    //触发函数
    emit:function(eventName,eventMsg){
        //存在这个事件名就进行循环调用里面的方法
        if(this.eventList[eventName]){
            this.eventList[eventName].forEach(itemFn => {
                itemFn(eventMsg)
            });
        }
    }
}
//监听事件
myEvent.on("fileSuccess",()=>{
    console.log("做第一件事")
})
myEvent.on("fileSuccess",()=>{
    console.log("做第二件事")
})
myEvent.on("fileSuccess",(msg)=>{
    console.log("做第三件事,并显示内容如下：\n")
    console.log(msg)
})
//最终上面3个监听事件中的内容按顺序打印
```

### 6.2使用Node的自身模块简化上面代码

```js
//使用node 的 event 设计自定义事件

//引入事件模块
let Events=require("events")
//引入文件系统模块
let fs=require("fs")

//创建事件对象
const emitEvent=new Events.EventEmitter()

//这里监听事件
emitEvent.on("fileSuccess",data=>{
    console.log(data)
})

fs.readFile("../info.json",{encoding:"utf-8"},(err,data)=>{
    if(err){
        console.log(err)
        return
    }
    // 1. 做第一件事
    // 2. 做第二件事
    // 3. 做第三件事
    //这里触发事件
    emitEvent.emit("fileSuccess",data)
})
```

## 7.Node服务器

### 7.1自己封装服务器代码

```js
const http = require("http")
const path = require("path")
const fs = require("fs")

//静态资源路径
const staticUrl = "./static/"

//自己封装服务器类
class myServer {
    constructor() {
        //实例化服务器对象
        this.server = http.createServer()
        //放请求路径对应的执行方法
        this.reqEvent = {
            // "/":function(){},
            // "/news":function(){}
        }
        //监听请求事件
        this.server.on("request", (req, res) => {
            //解析路径
            console.log(req.url)
            let mPath = path.parse(req.url)
            console.log(mPath)
            //遍历对象
            if (req.url in this.reqEvent) {           
                res.render=this.render
                res.replaceArr=this.replaceArr
                res.replaceVar=this.replaceVar
                req.path=path
                //如果存在key就执行value(为方法)
                // fn() 调用函数
                this.reqEvent[req.url](req, res)
            }else if(mPath.dir in this.reqEvent){
                res.render=this.render
                res.replaceArr=this.replaceArr
                res.replaceVar=this.replaceVar
                req.path=path
                this.reqEvent[mPath.dir](req, res)
            } else if (mPath.dir == "/abc") {   //静态文件访问专用路径
                //设置请求头
                res.setHeader("content-type", this.getContentType(mPath.ext))
                //访问静态目录的内容
                if(fs.existsSync(staticUrl+mPath.base)){
                    let rs = fs.createReadStream(staticUrl + mPath.base)
                    //返回数据流
                    rs.pipe(res)
                }else{
                    res.setHeader("Content-Type", "text/html;charset=utf-8")
                    res.end("<h1>404 not found</h1>")
                }          
            } else {
                //没有匹配到的路由
                //设置响应头
                res.setHeader("Content-Type", "text/html;charset=utf-8")
                res.end("<h1>404 not found</h1>")
            }
        })
    }
    //自定义on事件
    on(url, fn) {
        this.reqEvent[url] = fn
    }
    //自定义运行函数
    run(port, callback) {
        this.server.listen(port, callback)
    }
    //根据请求的内容设置响应类型
    getContentType(ext) {
        switch (ext) {
            case ".html":
                return "text/html;charset=utf-8"
            case ".jpg":
                return "image/jpeg"
            case ".png":
                return "image/png"
            case ".gif":
                return "image/gif"
            case ".js":
                return "text/javascript;charset=utf-8"
            case ".css":
                return "text/css"
            case ".json":
                return "text/json;charset=utf-8"
            case ".mp4":
                return "video/mp4" 
            default:
                return "text/html;charset=utf-8"   
        }
    }
    //自定义渲染函数
    render(option,path) {
        //读取需要请求的静态网页文件
        fs.readFile(path,{encoding:"utf-8",flag:"r"},(error,data)=>{
            if(error){
                console.log(error)
            }else{
                //替换循环项      
                data=this.replaceArr(data,option)

                //进行普通变量替换
                data=this.replaceVar(data,option)

                //替换完成返回数据
                //箭头函数this指向外面的作用域 this==res
                this.end(data)
            }
        })
    }
    //替换循环项 模板替换方法     
    replaceArr(data,option){
        let reg=/\{\%for\{(.*?)\}\%\}(.*?)\{\%endfor\%\}/igs
        let result
        while(result=reg.exec(data)){
            //console.log(result2[1])
            let arrKey=result[1].trim()
            //获取被循环的内容
            let reg2=/\{\%(.*)\%\}/igs
            //获取类型数组
            let arrValue=option[arrKey]
            let item=result[2]   //<li>{%item%}</li>
            let item2=item
            //console.log(item)
            let total=""  //把替换的内容合并
            let result2=reg2.exec(item)
            //result3[0]          //{%item%}
            arrValue.forEach((element,index) => {
                //循环 type中的内容 进行替换 存放到临时变量中
                total+=item2.replace(result2[0],element)
            });
            //type中的内容循环完成  整体替换
            data=data.replace(result[0],total)
        }
        return data
    }
    //替换普通变量 模板替换方法
    replaceVar(data,option){
        let reg=/\{\{(.*?)\}\}/igs
        let result
        while(result=reg.exec(data)){
            //result[0]=={{title}} result[1]==title
            //读取{{}}中的内容并去除两边的空白
            let strKey=result[1].trim()
            //获取json中的value
            let strValue=option[strKey]
            //替换掉 {{}} 和里面包裹的内容
            data=data.replace(result[0],strValue)
        }
        return data
    }
}

//抛出类
module.exports = myServer
```

### 7.2服务器入口文件

```js
const myServer=require("./myServer") //导入自己封装的代码
const fs=require("fs")
const ms=new myServer()

//静态资源路径
const staticUrl="./static"

ms.on("/",(req,res)=>{
    res.setHeader("content-type","text/html;charset=utf-8")
    res.end("<h1>这是首页</h1>")
})

ms.on("/news",(req,res)=>{
    res.setHeader("content-type","text/html;charset=utf-8")
    res.end("<h1>这是新闻页面</h1>")
})

//不在这写
// //测试读取静态页面
// ms.on("/abc/index.html",(req,res)=>{
//     //res.setHeader("content-type","text/html;charset=utf-8")
// })
ms.on("/movies",(req,res)=>{
    console.log(req.path.parse(req.url))
    //读取请求路径后的id
    let index=req.path.parse(req.url).base
    //读取Json数据
    let moviesJson=fs.readFileSync("./static/movie.json")
    //转成js对象
    moviesJson=JSON.parse(moviesJson)
    //调用渲染函数
    res.render(moviesJson[index],"./static/template.html")
})

ms.run(80,()=>{
    console.log("服务器启动成功！")
})
```

### 7.3自定义模板代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>动态渲染数据</title>
</head>
<body>
    <!--使用正则进行替换操作-->
    <h1>电影名:{{title}}</h1>
    <h2>英文名:{{enName}}</h2>
    <h2>评分:{{score}}</h2>
    <h2>类型:
        <ul>
            {%for{type}%}
                <li>{%item%}</li>
            {%endfor%}
        </ul>
    </h2>
    <h2>电影片长:{{movieLength}}</h2>
    <h2>上映时间:{{timeAndAdress}}</h2>
    <h2>剧情简介:{{synopsis}}</h2>
    <img src="{{bigImg_Url}}" alt="大图">
</body>
</html>
```

### 7.4目录截图


<div align=left><img  src="https://img.imgdb.cn/item/601e99fa3ffa7d37b389e323.jpg"/></div>



## 8.自己上传npm包

### 8.1自己封装的代码

```js
// 我的模块
let fs=require("fs");

//读取文件内容函数
function fsRead(path){
    return new Promise((resolve,reject)=>{
        if(!fs.existsSync(path)){   //判断是否存在指定文件
            fs.writeFileSync(path,"")  //写一个空文件
        }
        fs.readFile(path, {encoding:"utf-8",flag:"r"},(err, data) => {
            if (err) reject(err);
            resolve(data);
        });
    })
}

//文件写入函数
function fsWrite(path,content){
    return new Promise((resolve,reject)=>{
        //此处注意 回调函数只有一个参数
        fs.writeFile(path,content,{flag:"w"},err=>{
            if(err){
                reject(err)
            }else{
                resolve(err)
                console.log("写入成功！")
            }
        })
    })
}

//创建文件夹
function mkDir(path) {
    return new Promise((resolve,reject)=>{
        fs.mkdir(path,{recursive:true},err=>{
            if(err){
                reject(err)
            }else{
                console.log("创建文件夹成功！路径为："+path)
            }
        })
    })
}

//读取文件夹
function readDir(path) {
    return new Promise((resolve,reject)=>{
        fs.readdir(path,(err,files)=>{
            if(err){
                reject(err)
            }else{
                resolve(files)
                console.log("读取文件夹成功！")
            }
        })
    })
}

//导出函数
module.exports={
    fsRead,
    fsWrite,
    mkDir,
    readDir
}
```

### 8.2打包

**运行**

```bash
npm init
```

**然后弹出 包信息填写**

### 8.3包的相关信息填写

**填写完成后，会自动生成package.json文件**

```json
{
  "name": "fxc-promise-fs",
  "version": "0.1.0",
  "description": "fsRead,fsWrite,mkDir,readDir",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "node",
    "promise"
  ],
  "author": "iTaylorfan",
  "license": "ISC"
}

```

**本地打包完成**

### 8.4上传包到npm官网

**先本机登录**

```bash
npm login
```

**登录完成开始上传**

```bash
npm publish
```

**上传完成去官网查看**

![npm官网](https://img.imgdb.cn/item/601f5ee13ffa7d37b3dc36ad.jpg)

**完成！可以下载使用了**

## 9.Node连接数据库

### 9.1安装mysql包

```bash
npm install mysql
```

### 9.2代码

```js
const mysql = require("mysql")
const fs = require("fs")

//自定义配置项
let options = {
    host: "localhost",
    user: "root",
    password: "123456",
    database: "test"
}
//创建连接
const con = mysql.createConnection(options)
//进行连接
con.connect(err => {
    if (err) {
        console.log(err)
    } else {
        console.log("数据库连接成功！")
    }
})

function mainF() {
    //查询的sql语句
    let sql = "select * from user"
    let result = query(sql)
    console.log(result)
    //使用 ?占位符进行插入操作
    //  let insertSql = "insert into user(id,username,password)values(?,?,?)"
    let data = []
    //读取json
    data = fs.readFileSync("./movie.json")
    data = JSON.parse(data)

    //循环插入数据
    data.forEach((item, index) => {
        let type = ""
        item.type.forEach((element, index) => {
            type += element
            if (index != item.type.length - 1) {
                type += ","
            }
        });
        //需要插入的数据
        let insertData=[]
        insertData = [index+1, item.title, item.score, item.img_Url, item.enName, type, item.movieLength, item.timeAndAdress, item.synopsis, item.bigImg_Url]
        //插入数据的sql语句
        let insertSql = "insert into movies(id,title,score,img_Url,enName,type,movieLength,timeAndAdress,synopsis,bigImg_Url)values(?,?,?,?,?,?,?,?,?,?)"
        insert(insertSql, insertData)
    })
}
//查询方法
function query(sql) {
    con.query(sql, (err, result, feilds) => {
        if (err) {
            console.log(err)
        } else {
            //打印每个字段的详细信息
            // console.log(feilds)
            //返回结果集
            return result
        }
    })
}

//插入数据方法
//data为需要插入数据的数组
function insert(insertSql, data) {
    con.query(insertSql, data, (err, result) => {
        if(err){
           console.log(err)
            return
        }      
        console.log(result)
    })
}

//入口函数
mainF()
```

