# JSP学生信息管理系统笔记

## 1.css文件变化，而jsp页面无变化问题

**清除浏览器缓存，谷歌Ctrl+F5，或者直接设置中清除。**

## 2.使用cookie实现登录页面记住我功能

**servlet页面**

```java
				// 新建cookie
				//将用户名和密码加入cookie   cookie保存多个值的方法 不能放数组 只能String类型
				String data=account+","+passWord;
				Cookie cookie = new Cookie("remember",data);
				
				// 2 判断remeber 是否被选中
				if (isChecked != null && isChecked.equals("on")) {
					// "yes" 勾选了 ==> 设置有效时间为 一周
					cookie.setMaxAge(60 * 60 * 24 * 7);
				} else {
					// null 没勾选 ==> 设置cookie的有效时间为0
					cookie.setMaxAge(0);
				}
				// 3 将cookie添加到response
				response.addCookie(cookie);
				response.sendRedirect("success.jsp");
```

**jsp页面**

```jsp
    <%
        Cookie[] cookies = request.getCookies();
        Cookie remember = null;
		//存放字符分割后的数组
        String[] data=null;
        if (cookies != null && cookies.length > 0) {
            for (Cookie c : cookies) {
                if (c.getName().equals("remember")) {
                    remember = c;
                    //字符串分割
                    data=remember.getValue().split(",");
                   	//out.print(data[1]);
                }
            }
        }
    %>

<!-- 表单中 给输入框赋cookie中的值-->
  <input type="text" id="inputAccount" class="form-control" name="account" placeholder="用户名" required="" 
        autofocus="" value="<%=remember==null?"":data[0] %>">
```

## 3.登录界面验证码

**HTML表单**

```jsp
<input type="text" name="checkCode"  class="form-control" required="" id="inputCheckCode" style="width:50%;display:inline" placeholder="验证码">
				<img src="./tools/checkCode.jsp" id="checkCodeImage"/>
```

**将验证码图片单独做成一个jsp页面**

**图片链接为这个jsp页面**

```jsp
<%@ page contentType="image/jpeg" language="java" import="java.util.*,java.awt.*,java.awt.image.*,javax.imageio.*" pageEncoding="utf-8"%>

<%!
	Color getRandColor(int fc,int bc){
		Random random = new Random();
		if(fc > 255){
			fc = 255;
		}
		if(bc < 255){
			bc = 255;
		}
		int r = fc +random.nextInt(bc-fc);
		int g = fc +random.nextInt(bc-fc);
		int b = fc +random.nextInt(bc-fc);
		
		
		return new Color(r,g,b);
	}
%>

<% 
	//设置页面不缓存
	response.setHeader("Pragma","no-cache");
	response.setHeader("Cache-Control","no-catch");
	response.setDateHeader("Expires",0);
	
	//在内存中创建图象
	int width = 70;
	int height = 30;
	BufferedImage image = new BufferedImage(width,height,BufferedImage.TYPE_INT_RGB);
	
	//创建图象
	Graphics g = image.getGraphics();
	//生成随机对象
	Random random = new Random();
	//设置背景色
	g.setColor(getRandColor(200,250));
	g.fillRect(0,0,width,height);
	//设置字体
	g.setFont(new Font("Tines Nev Roman",Font.PLAIN,24));
	//随机产生干扰线
	g.setColor(getRandColor(160,200));
	for(int i = 0; i < 255; i++){
		int x = random.nextInt(width);
		int y = random.nextInt(height);
		int xl = random.nextInt(12);
		int yl = random.nextInt(12);
	}
	//随机产生认证码,4位数字
	String sRand = "";
	for(int i = 0; i < 4; i++){
		String rand = String.valueOf(random.nextInt(10));
		sRand  += rand;
		//将认证码显示到图象中
		g.setColor(new Color(20 + random.nextInt(110),20 + random.nextInt(110),20 + random.nextInt(110)));
		g.drawString(rand,13*i+6,25);
	}
	
	
	session.setAttribute("rCode",sRand);
	//图像生效
	g.dispose();
	//输出图像到页面
	ImageIO.write(image,"JPEG",response.getOutputStream());
	out.clear();
	out = pageContext.pushBody();
%>
```

**点击刷新功能**

```js
<script>
		//点击刷新验证码
		$("#checkCodeImage").click(function(){
			//console.log($(this).attr("src"));
			$(this).attr("src","./tools/checkCode.jsp?"+Math.random()); //重新设置img src路径
		});	
</script>
```

## 4.使用JQuery中的Ajax请求servlet

**1.servlet**

```java
import com.alibaba.fastjson.JSON; //关键包 需要上github获取	com.alibaba.fastjson


protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
		
		//response.getWriter().append("Served at: ").append(request.getContextPath());
		
		
		// 从cookie中取值
		Cookie[] cookies = request.getCookies();
		Cookie remember = null;
		String[] data = null;
		if (cookies != null && cookies.length > 0) {
			for (Cookie c : cookies) {
				if (c.getName().equals("remember")) {
					remember = c;
					// 字符串分割
					data = URLDecoder.decode(remember.getValue(), "utf-8")
							.split(","); /* 使用utf-8读取 */
					// out.print(data[2]);
				}
			}
		}
		// System.out.println(data[3].substring(0,data[3].length()-2));
		
		

		if (data != null) {
			// 获取班号
			String class_id= data[3].substring(0,data[3].length()-2);
			
			String sql = "SELECT a.date,a.address,b.cname "
					+ "FROM course_chart a,course_info b "
					+ "WHERE a.cno=b.cno AND a.class_id="+class_id+";";
			//System.out.println(sql);
			ResultSet rSet = jdbc.query(sql);
					
			List<MyChart> list=new ArrayList<MyChart>();
			try {
				while (rSet.next()) {
					// 封装到javaBean中
					MyChart myChart = new MyChart(rSet.getString(1),rSet.getString(2),rSet.getString(3));
					// 把对象加入容器
					list.add(myChart);
				}
				// 把容器加入session
				HttpSession session = request.getSession();
				session.setAttribute("MyChart", list);
				
				
				//将list容器中的内容转换成json数据
				request.setCharacterEncoding("utf-8");  // 设置request字符编码
		        String searchText = request.getParameter("search"); // 获取传入的search字段的内容
		        response.setContentType("text/json; charset=utf-8");    // 设置response的编码及格式
		        PrintWriter out = response.getWriter();
		        String resJSON = JSON.toJSONString(list);     // 转换为json
		        //System.out.println(resJSON);
		        out.print(resJSON); // 返回数据
		        
		        //response.sendRedirect("control/MyCourseChart.jsp");
				
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		} else {
            //如果没有登录信息则返回登录页面
			response.sendRedirect("Login.jsp");
		}
	}
```

**2.JS文件**

```javascript
function getData(){
	//获取数据
	$.get("../MyCourseChart",function(data,status){
	    console.log("数据: " + JSON.stringify(data)+ "\n状态: " + status);
	    let DATA=JSON.stringify(data); //先要转换成json字符串 否则显示[object object]
	    console.log(JSON.parse(DATA))
	  });
}
getData();
```

## 5.servlet获取url地址栏参数乱码

```java
	request.setCharacterEncoding("utf-8");
	String q=request.getParameter("query"); //直接打印q出现乱码
	//解决url地址栏参数中文乱码
	String query = new String(q.getBytes("iso-8859-1"), "utf-8");  //可正常打印
```

## 6.配置404页面

**web.xml文件**

```jsp
  	<error-page>
        <error-code>404</error-code>
        <location>/tools/404NotFound.jsp</location>  <!-- 404页面路径 -->
	</error-page>
```

**404NotFound.jsp**

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<link rel="icon" href="../assets/images/favicon.jpeg">
<title>页面走丢了</title>
<style>
	.wrap{
		width:100%;
		height:100%;
		position:absolute;
		left:0;
		top:0;
		overflow:hidden;
		right:0;
		bottom:0;
		margin:auto;
		background-color:#F2F2F2;
	}
	.wrap img{		
		height:100%;
	}
	.wrap .img_box{
		text-align:center;
		height:100%;
		position:relative;
		margin:auto;
	}
	.wrap .text{
		position:absolute;
		left:10px;
		top:10px;
		z-index:100;
		
	}
	.wrap .text a{
		text-decoration:none;
	}
</style>
</head>
<body>

<section class="wrap">
	<div class="text">
		<span>请点此处</span><a href="JavaScript:history.go(-1)">返回</a>
	</div>
	<div class="img_box">
	<img src="<%=request.getContextPath()%>/assets/images/404.png">
	</div>
</section>

</body>
</html>
```

