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

