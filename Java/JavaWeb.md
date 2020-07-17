# Session&Cookie
## Cookie
1. 浏览器端保存少量数据的一种技术 
2. 它只能保存少量的数据，而且必须是纯文本；保存在当前网站的 cookie，每次访问这个网站都会携带
3. 默认不支持中文
4. 掌握，服务器如何给浏览器分发Cookie
```java
Cookie cookie = new Cookie("username", "zhangsan");
response.addCookie(cookie)
response.setHeader("Set-Cookie", "username=zhangsan");
```
5. 浏览器一旦保存 Cookie，访问这个网站都会带上这个cookie 
6. Cookie 的有效时间：
    * 默认会话期间有效，只要浏览器不管，cookie就会存在
    * cookie实际上存在于浏览器的进程中
    * setMaxAge(int expiry)：设置以秒为单位的生命周期
        * 正数表示超时时间
        * 负数表示Cookie就是会话Cookie，随浏览器一起消亡
        * 0 立即失效
7. 修改或删除 Cookie 只能进行同名覆盖，因为getCookie拿到的只是浏览器端数据的一个封装，并不影响浏览器端的值；想修改，只能进行同名覆盖；可以使用 setMaxAge(0)完成
8. 读取 Cookie： `Cookie[] cookies =request.getCookies()`

## Session
1. 服务器端保存大量数据的技术
2. Session 中保存的数据可以在同一个会话期间共享。
3. `session.setAttribute() / session.getAttribute()`
4. 会话控制原理
    * 为什么在别的地方给 session 保存的数据，在另外一个地方可以被获取？
        * 和服务器交互期间，可能需要保存一些数据，服务器专门为每一个会话创建一个 map，用这个 map 来保存数据，这个map 我们就叫他 session
        * 有多少个会话就有多少个 map，每次被创建 map 的时候 map 有唯一的一个 标识，JSESSIONID
5. 流程：
    * 当一个浏览器第一次访问服务器，会进行尝试获取 session，但是没有，服务器就会为这个浏览器分配一个 session，并且告诉浏览器这个session 的 id：set—Cookie:JSESSIONID=A
    * 第二次这个浏览器访问这台服务器，发送请求带有 Cookie 属性的 JID = A，就会去对应的 session 获取需要的值

6. 利用浏览器每次访问都会带上它的 Cookie，服务器只需要创建一块能保存数据的 map，给这个 map 一个唯一的标识 JID，创建好以后命令浏览器保存这个标识，以后浏览器访问就会带上这个 map 的标识；服务器按照这个 map 取出对应的值
7. 会话关闭，
    * 默认 Cookie 没了；通过 cookie 持久化技术继续找到之前的 session
    * session 失效；自动超时或者手动失效

## 令牌机制
防止表单重复提交： 访问页面的时候生成一个令牌，这个令牌分置两处；服务器端 `session.setAttribute("token", token)`;一处页面保存每次发请求带上
```
<form>
    <input name="token" value="<%=token %>" />
</form>
```
> <% String token = UUID.randomUUID().toString(); %>

Servlet 收到表单提交，拿到服务器端保存的令牌`String token1 = session.getAttribute("toke")`，再拿到页面带来的令牌`String token2 = session.getParameter("toke")`; 
```java
if(token1.equals(token2)) {
    //处理请求
    session.setAttribute("token", anotherT);
} else {
    //不处理请求
}
```
## 单点登录
[Solution 1](http://blog.csdn.net/qq_39089301/article/details/80615348)

[Solution 2](https://blog.csdn.net/cutesource/article/details/5838693?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)

[code](https://blog.csdn.net/zhangjingao/article/details/81735041?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)