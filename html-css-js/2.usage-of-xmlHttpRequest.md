# xmlHttpRequest
## POST
#### client
```html
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title>xmlHttpRequestDemo</title>
</head>
<body>
    <label id="labName">姓名：</label>
    <input id="txtName" />
    <br /><br />
    <label id="labMess">备注：</label>
    <input id="txtMess" />
    <br /><br />
    <input type="button" id="btnGet" value="Get" onclick="btnGetClick()" />
    <hr />
    <div id="result"></div>
    <script type="text/javascript">
        function getValueById(ElemId) {
            return document.getElementById(ElemId).value;
        }
        function setInnerHTMLToId(Html, ElemId) {
            document.getElementById(ElemId).innerHTML = Html;
        }
        function cheakNull(ElemId) {//检测Valse是否为空，空返回false，非空返回true
            var val = getValueById(ElemId);
            if (val === null || val === "")
                return false;
            return true;
        }
        function btnGetClick() {
            if (cheakNull("txtName") && cheakNull("txtMess")) { //姓名和备注都不为空

                //创建XMLHttpRequest对象
                var XMLHttp = null;
                if (window.ActiveXObject) {//ie浏览器
                    XMLHttp = new ActiveXObject("Microsoft.XMLHTTP");
                }
                else if (window.XMLHttpRequest) {//其他浏览器
                    XMLHttp = new XMLHttpRequest();
                }

                //配置XMLHttpRequest对象
                XMLHttp.open("post", "xmlHttpRequestDemo.aspx?rand=" + Math.random(), true);

                //设置回调函数
                XMLHttp.onreadystatechange = function () {
                    if (XMLHttp.readyState == 4 && XMLHttp.status == 200) {//收到完整的HTTP响应并且HTTP状态代码为200
                        setInnerHTMLToId(XMLHttp.responseText, "result");
                    }
                };

                //获取值
                var name = getValueById("txtName");
                var mess = getValueById("txtMess");

                //设置参数
                var params = "call=getMess&name=" + name + "&mess=" + mess;

                //设置头部
                XMLHttp.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
                XMLHttp.setRequestHeader("Content-length", params.length);

                //发送请求 
                XMLHttp.send(params);
            }
            return false; //结束点击响应事件。
        }
    </script>
</body>
</html>
```
#### server
```CSharp
using System;
using System.Collections.Generic;
using System.Linq;

namespace AjaxTest
{
    public partial class xmlHttpRequest : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {
            String method = Request.Form["call"];
            if (method == "getMess") {
                Response.Clear();
                Response.Write(getMess());
            }
            if (method != null)
                Response.End();
        }
        protected String getMess() {
            String name = Request.Form["name"];
            String mess = Request.Form["mess"];
            return String.Format("姓名：{0}<br/><br/>备注：{1}<br/><br/>当前时间：{2}", name, mess, DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss"));
        }
    }
}
```
## GET
#### client
```html
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title>xmlHttpRequestDemo2</title>
</head>
<body style="text-align:center;">
    <div><hr /></div>
    <div>产生时间：<%=DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss") %></div>
    <div id="Grade">
        <input type="button" id="G1" value="Grade1" onclick="_click(this)" />
        <input type="button" id="G2" value="Grade2" onclick="_click(this)" />
        <input type="button" id="G3" value="Grade3" onclick="_click(this)" />
    </div>
    <div><hr /></div>
    <div id="result"></div>
    <div><hr /></div>
    <script type="text/javascript">
        function setInnerHTMLToId(Html, ElemId) {
            document.getElementById(ElemId).innerHTML = Html;
        }
        function _creatXmlHttpRequestObj() {
            var xmlrequest = null;
            if (window.XMLHttpRequest) {
                //DOM 2浏览器  
                xmlrequest = new XMLHttpRequest();
            }
            else if (window.ActiveXObject) {
                // IE浏览器  
                try {
                    xmlrequest = new ActiveXObject("Msxml2.XMLHTTP");
                }
                catch (e) {
                    xmlrequest = new ActiveXObject("Microsoft.XMLHTTP");
                }
            }
            return xmlrequest;
        }
        function _click(obj) {
            //创建XmlHttpRequest对象
            var XMLRequest = _creatXmlHttpRequestObj();
            //设置请求响应的URL
            var url = "xmlHttpRequestDemo2.aspx?rand=" + Math.random() + "&grade=" + obj.value;
            XMLRequest.open("get", url, true);
            //设置处理响应的回调函数
            XMLRequest.onreadystatechange = function () {
                //响应完成且响应正常  
                if (XMLRequest.readyState == 4) {
                    if (XMLRequest.status == 200) {
                        //页面正常
                        setInnerHTMLToId(XMLRequest.responseText, "result");
                    }
                    else {
                        //页面不正常  
                        window.alert("您所请求的页面有异常。");
                    }
                }
            };
            //发送请求  
            XMLRequest.send(null);
        }
        _click(document.getElementById("G1"));
    </script>
</body>
</html>
```
#### server
```CSharp
using System;
using System.Collections.Generic;
using System.Linq;

namespace AjaxTest
{
    public partial class xmlHttpRequestDemo2 : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {
            String grade = Request.QueryString["grade"];
            if (grade != null && grade != "") {
                Response.Clear();
                Response.Write(String.Format("<div>产生时间：{0}</div><div>{1}-Class1</div><div>{1}-Class2</div><div>{1}-Class3</div>", DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss"), grade));
                Response.End();
            }
        }
    }
}
```
