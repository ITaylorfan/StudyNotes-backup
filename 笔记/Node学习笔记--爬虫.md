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

## 3.爬取歌曲

> 爬取网易云音乐

### 3.1右键选择查看框架源代码

![右键查看框架源代码](https://img.imgdb.cn/item/60117a9b3ffa7d37b342b152.png)

### 3.2**需要的数据都在这**

![需要的数据](https://img.imgdb.cn/item/60117b5a3ffa7d37b34302f6.png)

### 代码：

```js
let cheerio = require("cheerio")
let fs = require("fs")
let axios = require("axios")
let path = require("path")
//存放获取到的歌曲信息
let songs = []

//爬取网易云音乐热歌榜单内容  https://music.163.com/#/discover/toplist?id=3778678
//下面榜单数据id可能会更换、以官网为准
let discover =[
  { title: '飙升榜', id: 19723756 },
  { title: '新歌榜', id: 3779629 },
  { title: '原创榜', id: 2884035 },
  { title: '热歌榜', id: 3778678 }
]

async function mainF(query) {
    //爬取热歌榜
    //需要使用查看网页框架中的链接  去掉#符号
    let baseUrl = "https://music.163.com/discover/toplist?id=" + query[3].id
    //设置请求头
    //let Header={"User-Agent":"Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.25 Safari/537.36 Core/1.70.3741.400 QQBrowser/10.5.3863.400"}
    let res = await axios.get(baseUrl)
    let $ = cheerio.load(res.data)
    // console.log(res.data)
    //获取a标签
    let dom = $(".f-hide>li>a")
    dom.each((index, item) => {
        let obj = {
            title: $(item).text(),
            url: $(item).attr("href"),
            id: $(item).attr("href").replace(/[^\d]/g, "")   //使用正则去掉非数字
        }
        songs.push(obj)
        // console.log(obj)
    })
    getSongs(songs)
    //console.log(res.data)
}

//根据获取到的歌曲id去请求歌曲资源  然后下载到本地
function getSongs(data) {
    let num=0
    // http://music.163.com/song/media/outer/url?id=   在线播放音乐的链接
    let getSongsUrl = "http://music.163.com/song/media/outer/url?id="
    console.log("一共有："+data.length+"首歌曲")
    //方便调试 只爬10首
    for(let i=0;i<10;i++){
        axios.get(getSongsUrl + data[i].id,{responseType:"stream"}).then(success => {
            // 创建文件夹 如果没有的话
            if (!fs.existsSync("./songs")) {
                fs.mkdirSync("./songs")
            }
            //创建写入流
            let ws = fs.createWriteStream("./songs/" + data[i].title + ".mp3")
            //使用管道流生成文件
            success.data.pipe(ws)
            //读取流关闭时执行下面操作
            success.data.on("close", () => {
                num++
                console.log("写入成功"+num+"首,歌曲名为："+data[i].title+".mp3")
                ws.close()
            })
        }, error => {
            console.log("错误,歌曲名为："+data[i].title+".mp3")
        })
    }
}
//调用主函数
mainF(discover)
```

**目录结构：**

 <div align=left><img  src="https://img.imgdb.cn/item/60117c7b3ffa7d37b3438b67.png"/></div>

**终端：**

 <div align=left><img  src="https://img.imgdb.cn/item/60117cce3ffa7d37b343aae8.png"/></div>



## 4.使用前端神器puppeteer做爬虫

> Puppeteer 是一个控制 headless Chrome 的 Node.js API 。它是一个 Node.js 库，通过 [DevTools 协议](https://chromedevtools.github.io/devtools-protocol/)提供了一个高级的 API 来控制 [headless](https://developers.google.com/web/updates/2017/04/headless-chrome) Chrome。它还可以配置为使用完整的（非 headless）Chrome。
>
> 在浏览器中手动完成的大多数事情都可以通过使用 Puppeteer 完成。

**安装puppeteer库**

```bash
npm instal puppeteer
```

### 4.1爬取猫眼电影热映电影的数据

![正在热映](https://img.imgdb.cn/item/6017eda93ffa7d37b3f46005.png)

> 直接使用axios来爬取比较困难，此网站有滑块验证
>
> 使用puppeteer中的鼠标事件 可以模拟拖动以破解验证码(此网站验证码缺口出现位置不是随机的比较简单)

![滑块验证](https://img.imgdb.cn/item/6017ee1d3ffa7d37b3f4932d.png)

> 滑块的内容在iframe框架中，无法直接获取dom

![iframe](https://img.imgdb.cn/item/6017ee973ffa7d37b3f4be9d.png)

### 4.2代码

```js
//导入puppeteer库
const puppeteer = require("puppeteer")
//导入自定义写入模块
const { writeJson } = require("./write")

//爬取猫眼电影 正在热映的电影数据
const baseUrl = "https://maoyan.com"
async function mainF() {
    let option = {
        //是否开启无头浏览器模式
        headless: true,
        //设置可视区域大小
        defaultViewport: {
            width: 1200,
            height: 900
        },
        // 获取跨域iframe中的内容  关键
        args: [
            '--disable-web-security',
            '--disable-features=IsolateOrigins,site-per-process', // 很关键...
        ],
        //放慢速度
        //   slowMo:100
    }
    const bowser = await puppeteer.launch(option)
    let page = await bowser.newPage()
    await page.goto(baseUrl)
    //等待全部按钮加载完成
    await page.waitForSelector(".panel-header>.panel-more>a")
    //获取 全部 按钮 的dom
    //注意这里 的这个类被用到多次
    let btn = await page.$$(".panel-header>.panel-more")
    //跳转到正在热映的全部电影 2正在热映 3即将上映
    await btn[2].click()
    //等待路由跳转完成
    await page.waitForNavigation()
    //等待电影区块加载完成
    await page.waitForSelector(".movie-list>dd")
    await page.waitForSelector(".movie-list>dd>.movie-item.film-channel>a>.movie-poster>img:last-child")  
    //存放所有数据
    let totalData = await page.$$eval(".movie-list>dd", el => {
        //回调函数中无法直接使用外部变量
        let totalData = []
        el.forEach((item, index) => {
            //标题 
            let titleDom = item.querySelector(".channel-detail.movie-item-title")
            //评分
            let scoreDom = item.querySelector(".channel-detail.channel-detail-orange")
            //详情
            let detailDom = item.querySelector(".movie-item.film-channel>a")
            //图片链接
            let imgDom = item.querySelector(".movie-item.film-channel>a>.movie-poster>img:last-child")
            let obj = {
                title: titleDom.getAttribute("title"),
                score: scoreDom.innerHTML === "暂无评分" ? '暂无评分' : scoreDom.innerHTML.replace(/\D/g, ""),
                //图片属性 因为懒加载 所以链接属性会不同 src
                img_Url: imgDom.getAttribute("data-src"),
                detail_Url: detailDom.getAttribute("href")
            }
            if (obj.score !== "暂无评分") {
                obj.score = obj.score[0] + "." + obj.score[1]
            }
            totalData.push(obj)
        })
        return totalData
    })
    console.log(totalData.length)
    //关闭首页的浏览器
    bowser.close()
    
    //遍历获取到的所有内容 然后点击
    for (let i = 0; i < totalData.length; i++) {
        const bowser = await puppeteer.launch(option)
        let page = await bowser.newPage()
        //打开 详情页面 可能需要人机验证
        await page.goto(baseUrl + totalData[i].detail_Url)

        //处理滑块验证  验证区域加载的是iframe框架
        let url = page.url()
        //console.log(url)
        //表示跳转到了验证页面 破解滑块验证 只能破解有规律的 缺口出现位置差不多的 随机缺口未实现
        if (url.includes("https://verify.maoyan.com/")) {
            //等待框架
            await page.waitForSelector("#tcaptcha_transform iframe")
            //获取框架
            const elementHandle = await page.$('#tcaptcha_transform iframe');
            //获取框架内容
            const frame = await elementHandle.contentFrame();
            //等待框架跳转
            await frame.waitForNavigation()
            //获取滑块dom
            let thumb = await frame.$("#tcaptcha_drag_thumb")
            //获取滑块的位置信息
            let thumbPosInfo = await thumb.boundingBox()
            console.log(thumbPosInfo.x, thumbPosInfo.y, thumbPosInfo.width, thumbPosInfo.height)
            //鼠标移动到此位置
            await page.mouse.move(thumbPosInfo.x + 30, thumbPosInfo.y);
            // 按下鼠标
            await page.mouse.down();
            //获取进度条宽度
            let track = await frame.$(".tc-drag-track")
            let trackPosInfo = await track.boundingBox()
            console.log(trackPosInfo.x, trackPosInfo.y, trackPosInfo.width, trackPosInfo.height)
            // 一点点移动鼠标
            for (let i = 0; i < trackPosInfo.width - 54 - thumbPosInfo.width; i++) {
                await page.mouse.move(thumbPosInfo.x + 30 + i, thumbPosInfo.y);
            }
            //松开鼠标
            await page.mouse.up();
        }

        //等待需要的内容加载
        await page.waitForSelector(".movie-brief-container>.ename.ellipsis")
        //获取我们需要的内容
        let enName = await page.$eval(".movie-brief-container>.ename.ellipsis", el => el.innerHTML)
        let type = await page.$$eval(".movie-brief-container>ul>li>a", el => {
            let type = []
            el.forEach((item, index) => {
                type.push(item.innerHTML)
            })
            return type
        })
        let movieLength = await page.$eval(".movie-brief-container>ul>li:nth-child(2)", el => el.innerText)
        let timeAndAdress = await page.$eval(".movie-brief-container>ul>li:last-child", el => el.innerHTML)
        let synopsis = await page.$eval(".mod-content>.dra", el => el.innerHTML)
        let bigImg_Url = await page.$eval(".celeInfo-left>.avatar-shadow>img", el => el.getAttribute("src"))
        //封装
        let obj = {
            enName,
            type: type,
            movieLength,
            timeAndAdress,
            synopsis,
            bigImg_Url
        }
        //合并对象
        totalData[i] = { ...totalData[i], ...obj }
        //调用自定义写入模块 写入json
        writeJson(totalData[i])
        //完成一个页面就关闭浏览器
        bowser.close()
    }
}
//调用主函数
mainF()

```

**自定义json写入模块 write.js**

```js
//自定义 写入模块
const fs = require("fs")
function writeJson(myJsonData) {
    //如果存在就读取
    if (fs.existsSync("./movie.json")) {
        jsonData = fs.readFileSync("./movie.json")
        // console.log(jsonData)
        if (typeof JSON.parse(jsonData) === "object") {
            console.log("是json")
            jsonData = JSON.parse(jsonData)   //转成对象
            jsonData = [...jsonData,myJsonData]  //合并数据
        } else {  //为其他数据 或者空 直接覆盖
            jsonData =[myJsonData]
        }
    } else {    //不存在 就直接写入
        jsonData =[myJsonData]
    }
    let i=jsonData.length
    //序列化
    jsonData = JSON.stringify(jsonData)
    //获取一个完成
    fs.writeFileSync("./movie.json", jsonData)
    console.log("写入json数据成功！" + i + "个")
}
//抛出方法
module.exports={
    writeJson
}
```

### 4.3完成截图

![json截图](https://img.imgdb.cn/item/6017f05c3ffa7d37b3f572b2.jpg)

### 4.4存在问题

**1.只能获取全部数据中的一页数据，如果有多页获取后面的数据未能获取**

**2.json会写入重复数据，完成后需要去重**

**去重代码：**

```js
let fs=require("fs")
//直接使用 node 命令运行 进行去重
//对获取到的数据去重
function mainF(){
    if(fs.existsSync("./movie.json")){
        let jsonData=fs.readFileSync("./movie.json")
        jsonData=JSON.parse(jsonData)
        console.log("去重前有"+jsonData.length+"条数据")
        jsonData=quchong(jsonData)
        console.log("去重后有"+jsonData.length+"条数据")
        jsonData=JSON.stringify(jsonData)
        fs.writeFileSync("./movie.json",jsonData)
        console.log("去重完成！")
    }else{
        console.log("未找到源数据！")
    }
}
function quchong(arr) {
    let array = [];
    for(let i = 0; i < arr.length; i++) {
        let isChage = true;
        for(let z = 0; z < array.length; z++) {
            if(arr[i]['title'] == array[z]['title']) {
                isChage = false;
            }
        }
        if(isChage) {
            array.push(arr[i]);
        }
    }
    return array;
}
//调用主函数
mainF()
```

