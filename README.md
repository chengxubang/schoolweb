jsp+servlet入党信息管理系统

如需源码请联系[程序帮](http://ll032.cn/HZ6vHa)：QQ1022287044


项目介绍
----
> 入党信息管理系统采用jsp+servlet搭建，mysql做数据支持，jsp做前端数据渲染，实现不用角色不同的条件查询，展示对应的页面和数据。本系统应用于院校的入党信息管理，目前实现了个人管理、系统管理、支部信息管理、入党积极分子信息、预备党员信息、正式党员信息、党费信息、公告管理、文件管理等功能，有时间在做进一步的完善。

项目适用人群
----
正在做毕设的学生，或者需要项目实战练习的Java学习者

开发环境：
-----
1. jdk 8
2. intellij idea
3. tomcat 8.5.40
4. mysql 5.7

所用技术：
-----
1. jsp+servlet
2. js+ajax
3. layUi
4. jdbc直连

项目访问地址
---
```
http://localhost:8090
```

项目结构
-----
![项目结构](https://github.com/chengxubang/schoolweb/blob/main/image/%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84.jpg)

项目截图
----

- 注册

![注册](https://github.com/chengxubang/schoolweb/blob/main/image/%E6%B3%A8%E5%86%8C.png)
- 系统管理

![系统管理](https://github.com/chengxubang/schoolweb/blob/main/image/%E7%B3%BB%E7%BB%9F%E7%AE%A1%E7%90%86.png)

- 新增/修改用户

![新增/修改用户](https://github.com/chengxubang/schoolweb/blob/main/image/%E6%96%B0%E5%A2%9E%E7%94%A8%E6%88%B7.png)

- 正式党员信息

![正式党员信息](https://github.com/chengxubang/schoolweb/blob/main/image/%E6%AD%A3%E5%BC%8F%E5%85%9A%E5%91%98%E4%BF%A1%E6%81%AF.png)

- 公告管理

![公告管理](https://github.com/chengxubang/schoolweb/blob/main/image/%E5%85%AC%E5%91%8A%E4%BF%A1%E6%81%AF.png)

- 文件管理

![文件管理](https://github.com/chengxubang/schoolweb/blob/main/image/%E6%96%87%E4%BB%B6%E6%9F%A5%E7%9C%8B.png)






关键代码:
-----
1.初始化工作
```diff
//数据库连接初始化
public Connection getConnection() {
    Connection conn = null;
    String sDBDriver = "com.mysql.jdbc.Driver";
    String url = "jdbc:mysql://localhost:3306/schooldb?useUnicode=true&characterEncoding=utf8";
    try {
        //加载数据库驱动
        Class.forName(sDBDriver);
        //获取数据库链接 -账号-密码
        conn = DriverManager.getConnection(url, "root", "root123");
        return conn;
    } catch (Exception e) {
        e.printStackTrace();
        System.err.println("数据库驱动注册错误信息： " + e.getMessage());
    }
    return conn;
}
```

2.党费信息Servlet
```diff
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    try {
        String mtype = request.getParameter("mtype");
        String url = "";
        DangfeiDao dangfeiDao = new DangfeiDao();
        /**
         * 添加党费信息
         */
        if ("add".equals(mtype)) {
            TDangfei dangfei = new TDangfei();
            dangfei.setTypeid(request.getParameter("typeid"));
            dangfei.setClassname(request.getParameter("classname"));
            dangfei.setDangfei(request.getParameter("dangfei"));
            dangfei.setDangtype(request.getParameter("dangtype"));
            dangfei.setShouyear(request.getParameter("shouyear"));
            dangfei.setTypename(request.getParameter("typename"));
            dangfei.setUsername(request.getParameter("username"));
            dangfeiDao.addDangfei(dangfei);
            url = "/DangfeiServlet?mtype=query&flag=success";
            /**
             * 初始化修改党费信息界面
             */
        } else if ("preupdate".equals(mtype)) {
            request.setAttribute("dangfei", dangfeiDao.getDangfei(request.getParameter("id")));
            url = "/files/dangfei/update.jsp";

            /**
             * 修改党费信息
             */
        } else if ("update".equals(mtype)) {
            TDangfei dangfei = new TDangfei();
            dangfei.setId(Integer.parseInt(request.getParameter("id")));
            dangfei.setTypeid(request.getParameter("typeid"));
            dangfei.setClassname(request.getParameter("classname"));
            dangfei.setDangfei(request.getParameter("dangfei"));
            dangfei.setDangtype(request.getParameter("dangtype"));
            dangfei.setShouyear(request.getParameter("shouyear"));
            dangfei.setTypename(request.getParameter("typename"));
            dangfei.setUsername(request.getParameter("username"));
            dangfeiDao.updateDangfei(dangfei);
            url = "/DangfeiServlet?mtype=query";
            /**
             * 遍历党费信息
             */
        } else if ("query".equals(mtype)) {
            if (request.getSession().getAttribute("querypageunit") == null) {
                request.getSession().setAttribute("querypageunit",
                        this.pageunit);
            }
            int curpage = this.getCurrentpage(request);
            int pageunit = this.getPageunit(request, "querypageunit");

            String urlpage = "DangfeiServlet?mtype=query";
            StringBuffer cond = new StringBuffer();
            if (null != request.getParameter("searchname") && "" != request.getParameter("searchname").trim()) {
                cond.append(" and a.typename like '%" + request.getParameter("searchname").trim() + "%' ");
            }
            if (null != request.getSession().getAttribute("usertype") && !"0".equals(((String) request.getSession().getAttribute("usertype")))) {
                cond.append(" and a.typeid like '%" + (String) request.getSession().getAttribute("typeid") + "%' ");
            }
            PageInfo pageInfo = dangfeiDao.queryTDangfei(curpage,
                    pageunit, request, urlpage, cond.toString());
            request.setAttribute("pageinfo", pageInfo);
            request.setAttribute("searchname", request.getParameter("searchname"));
            url = "/files/dangfei/list.jsp";
            /**
             * 删除党费信息
             */
        } else if ("delete".equals(mtype)) {
            dangfeiDao.delDangfei(request.getParameter("id"));
            url = "/DangfeiServlet?mtype=query";
        }
        //重定向到目标地址
        RequestDispatcher dispatcher = request.getRequestDispatcher(url);
        dispatcher.forward(request, response);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
 
3.文件下载
```
public void doGet(HttpServletRequest request, HttpServletResponse response)throws  IOException {
    //获得请求文件名  
    request.setCharacterEncoding("ISO-8859-1");
    String filename = request.getParameter("filename");
    filename = new String(filename.getBytes("ISO-8859-1"), "UTF-8");

    //设置文件MIME类型
    response.setContentType(getServletContext().getMimeType(filename));
    //设置Content-Disposition
    response.setHeader("Content-Disposition", "attachment;filename=" + new String(filename.getBytes("GBK"), "ISO-8859-1"));
    //读取目标文件，通过response将目标文件写到客户端
    //获取目标文件的绝对路径


    String fullFileName = getServletContext().getRealPath("/upload/" + filename);
    //读取文件
    InputStream in = new FileInputStream(fullFileName);
    OutputStream out = response.getOutputStream();

    //写文件
    int b;
    while ((b = in.read()) != -1) {
        out.write(b);
    }

    in.close();
    out.close();
}

``` 

项目后续
----
其他ssh，ssm，springboot版本后续迭代更新，持续关注
