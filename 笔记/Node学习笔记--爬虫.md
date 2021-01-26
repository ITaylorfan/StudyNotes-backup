# Node学习笔记--爬虫

## 1.简单爬取电影数据

> 选择的网站是1905.com

![1905电影网](https://img.imgdb.cn/item/601030f23ffa7d37b3b3d683.png)

### **重要：正则表达式**

```js
//匹配符合条件的a标签
//注意特殊字符的转义  . : () /
//igs 表示忽略大小写 全局匹配 不略过空格 换行符
/<a href="javascript\:void\(0\);" onclick="location\.href='(.*?)';return false;".*?>(.*?)<\/a>/igs  
```

### 代码：

```js
//引入第三方库axios
let axios = require("axios")
let fs=require("fs")
let requestUrl = "https://www.1905.com/vod/list/n_1/o4p1.html"
//分类数据
let category = []
//所有电影
let allMovie=[]
axios.get(requestUrl).then(success => {
    //设置正则 
    // 爬取分类信息
    // <a href="javascript:void(0);" onclick="location.href='https://www.1905.com/vod/list/n_1_t_1/o4p1.html';return false;" >
    let re = /<a href="javascript\:void\(0\);" onclick="location\.href='(.*?)';return false;".*?>(.*?)<\/a>/igs   //i 表示忽略大小写 g 表示全局搜索 s 匹配换行符表示不忽略空格？
    category=getCategoryData(re,success.data)
    console.log(category)

    //提取当前页面的所有电影
    // <div class="grid-2x grid-3x-md grid-6x-sm">
    // <a class="pic-pack-outer" target="_blank" href="https://www.1905.com/vod/play/682663.shtml" title="霸王别姬"><img alt="霸王别姬" src="https://image11.m1905.cn/uploadfile/2013/0816/thumb_1_150_203_20130816013533696.jpg"><h3>霸王别姬</h3><i class="score"><b>9</b>.6</i><p>张国荣主演经典作</p></a>
    // </div>
    
    //<a class="pic-pack-outer" target="_blank" href="https://www.1905.com/vod/play/85676.shtml" title="举起手来"><img alt="举起手来" src="https://image11.m1905.cn/uploadfile/2009/1106/thumb_1_150_203_20091106112357202.jpg"><h3>举起手来</h3><i class="score"><b>9</b>.4</i><p>潘长江爆笑歼鬼子</p></a>

    let re2=/<a class="pic-pack-outer" target="\_blank" href="(.*?)" title="(.*?)"><img alt=".*?" src="(.*?)"><h3>.*?<\/h3><i class="score">(.*?)<\/i><p>(.*?)<\/p><\/a>/igs
    allMovie=getAllMovieData(re2,success.data)
    console.log(allMovie)

    //根据创建的内容生成目录
    if(!fs.existsSync("./movies")){
        fs.mkdirSync("./movies")
    }
    allMovie.forEach((item,index)=>{
        fs.writeFileSync("./movies/"+item.title+".json",JSON.stringify(item))
    })   
}, error => {
})

//封装方法 获取分类数据
function getCategoryData(RE,data) {
    //临时存放数据
    let myData=[]
    //执行正则
    //需要进行循环获取数据
    while (RE.exec(data)) {
        let Data = RE.exec(data)
        //console.log(data)
        //console.log(data[1],data[2])     //每个(*.?)的内容都可以提取
        let obj = {
            categoryName: Data[2],
            url: Data[1]
        }
        myData.push(obj)
    }
    //返回数据
    return myData
}

//封装方法获取当前页面所有电影
function getAllMovieData(RE,data) {
    //临时存放数据
    let myData=[]
    //执行正则
    //需要进行循环获取数据
    while (RE.exec(data)) {
        let Data = RE.exec(data)
        //console.log(data)
        //console.log(data[1],data[2])     //每个(*.?)的内容都可以提取
        let obj = {
            title: Data[2],
            url: Data[1],
            img_url:Data[3],
            score:Data[4].replace(/[^0-9]/ig,""),
            info:Data[5]
        }
        myData.push(obj)
    }
    //返回数据
    return myData
}
```

**爬取到的内容会自动生成json文件保存到movies文件夹中**

![爬取数据截图](https://img.imgdb.cn/item/601030ee3ffa7d37b3b3d445.png)

## 2.爬取表情包图片

> 选择的网站是 https://www.doutula.com/

![斗图啦](https://img.imgdb.cn/item/601031b43ffa7d37b3b433e4.jpg)

**这里使用cheerio 包代替复杂的正则表达式**

```bash
npm install cheerio
```

### **代码：**

```js
//导入第三方 cheerio包 代替正则表达式简化爬取步骤
//cheerio 可以使用JQuery方法来提取爬取到的网页
let cheerio = require("cheerio")
let axios = require("axios")
let fs = require("fs")
let path = require("path")

//首页链接
let RequestUrl = "https://www.doutula.com/article/list/?page=1"
//存放所有详情链接
let DetailUrl = []
//存放所有图片链接
let ImgUrl = []

//页码
let index = 1
//最大页数
let maxPage = 20
//主要函数
function mainF() {
    //爬取20页数据
    //使用延迟函数 减少压力
    setInterval(() => {
        axios.get("https://www.doutula.com/article/list/?page=" + index).then(success => {
            //console.log(success.data)
            //存放得到的页面详情数据
            DetailUrl = [...DetailUrl, ...getPageDetailUrl(success.data)]
            console.log(DetailUrl)
            //调用创建文件夹方法
            mkDir(DetailUrl)

            //根据得到详情链接获取详情页面中的数据
            //得到所有图片链接
            //ImgUrl=[...ImgUrl,...getDetailPageIamge(DetailUrl)]
            //console.log(ImgUrl)
            getDetailPageIamge(DetailUrl)
            if (index <= maxPage) index++
        }, error => {

        })
    }, 2000)
}

//获取页面中的详情链接
function getPageDetailUrl(data) {
    let DetailUrl = []
    //加载页面内容
    let $ = cheerio.load(data)
    //获取页面中需要的 a标签 
    $("#home .row .col-sm-9>a").each((index, item) => {
        //item 需要转换成JQuery对象才能使用 attr()方法
        let obj = {
            title: $(item).find(".random_title").text(),
            detailUrl: $(item).attr("href")
        }
        DetailUrl.push(obj)
        //console.log($(item).attr("href"))
    })
    return DetailUrl
}

//获取页面详情图片
function getDetailPageIamge(Url) {
    let imgUrl = []
    //对每个链接进行循环 获取详情页面的所有图片
    Url.forEach(async function (item) {
        //封装每个详情页面
        let OBJ = {}
        let res = await axios.get(item.detailUrl)
        let $ = cheerio.load(res.data)
        //设置选择器
        let Dom3 = $(".container_ .container .row .col-sm-9 .list-group-item .pic-title>div>.glyphicon")
        let Dom1 = $(".container_ .container .row .col-sm-9 .list-group-item .pic-title>h1>a")
        let Dom2 = $(".container_ .container .row .col-sm-9 .list-group-item .pic-content>.artile_des>table>tbody>tr>td>a>img")
        OBJ.title = $(Dom1).text() + $(Dom3).text()
        OBJ.img = []
        Dom2.each((index, item) => {
            let obj = {
                img_Url: $(item).attr("src")
            }
            OBJ.img.push(obj)
            //console.log($(item).attr("src"))
            //console.log(obj)
        })
        downloadImage(OBJ)
        imgUrl.push(OBJ)
        //console.log(imgUrl)       
    });
}

//创建文件夹
function mkDir(data) {
    let i = 0
    console.log("正在创建文件夹...")
    if (!fs.existsSync("./img")) {
        fs.mkdirSync("./img")
    }
    data.forEach(async function(item, index){
        if (!fs.existsSync("./img/" + item.title)) {
            let error=await fs.mkdir("./img/" + item.title)
            if (error) throw err
            i++
        }
    })
    console.log("创建所有文件夹成功,创建了" + i + "个文件夹")
}

//下载图片
function downloadImage(data) {
    //console.log(data.title)
    //遍历每张图片
    data.img.forEach((item, index) => {

        //请求图片需要设置响应类型为 流
        axios.get(item.img_Url, { responseType: "stream" }).then(success => {
            let extN = path.extname(item.img_Url)
            //创建写入流
            let ws = fs.createWriteStream("./img/" + data.title + "/" + data.title + "-" + index + extN)
            //创建读取管道
            success.data.pipe(ws)
            //console.log(success.data)
            console.log("成功写入一张图片: " + "./img/" + data.title + "/" + data.title + "-" + index + extN)
            //关闭写入流
            success.data.on("close", () => {
                ws.close()
            })
        }, error => {

        })
    })
}
//调用主函数
mainF()
```

![爬取到的表情包数据](https://img.imgdb.cn/item/601032f33ffa7d37b3b4b723.jpg)

### Bug：

**当创建文件夹路径出错时会终止程序**

![报错](https://img.imgdb.cn/item/6010341a3ffa7d37b3b53be6.jpg)

