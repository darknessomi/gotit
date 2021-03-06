# 正方教务系统查询工具


### 中文验证码

> 正方教务系统从原来的数字验证码到现在的中文验证码，尝试过验证码识别， 但是都不怎么理想，所以选择了让用户自己识别验证码，我们只模拟登陆。这样需要满足几个条件。  

> 首先，成功获取验证码并保存为图片，其中图片命名遇到了问题，起初选择了使用随机整型数字命名，由于程序中的随机都是伪随机， 重合率很高！后来选择了使用时间的MD5值，一直沿用到现在作为用户的唯一识别码，使`time.time()`获得的时间精确到0.001秒，以现在的用户访问量遇到在同一0.001秒同时访问几乎是不可能的，并且事实也是如此。

> 用户需要在页面上直接输入学号、密码和验证码， 这就需要我们在用户第一次访问时就为其抓取下验证码供其识别，用户填写后由后台使用用户的数据进行模拟登陆从而抓取下成绩内容或者其他信息。而此处的难点是：怎样保证用户识别的验证码和他应该提交的验证码是同一个。因为正方系统还为每一个页面状态提供了一个`VIEWSTATE`参数，每次post都需要提供该参数。我一开始的想法是能够解析出每次抓取时的COOKIE不过后来想通了，由于每次都是在服务端抓取， 每次的COOKIE都是一样的，而每次的VIEWSTATE却是不一样的，所以这个方法被否定了。

> [ma6174](http://ma6174.github.io)同学想出了另一个方法，就是：创建一个独立于`WEB.PY`之外的字典，通过键值对的方式，将每一次处理时的对象直接保存起来，此处对象中有两个关键方法，一个是`pre_login`另一个是`login`，前者用于抓取验证码、viewstate，并且将此处的对象内容以某一KEY保存在字典中，该KEY开始时直接使用的VIEWSTATE，不过在本学期末发现: VIEWSTATE在一段时间内不变了, 这就导致了字典中的某一值一直被重写，不过这种情况在本地测试是正常的，因为只有一个人访问。为了避免这个问题就再次使用了上面说的`time_md5`。

> 现在的做法就是：以time_md5为键，将由VIEWSTATE和对象组成的元组保存在全局字典中，验证码图片也是直接以time_md5命名，由于http的无状态，我们也懒得再弄一个cookie，所以直接将time_md5传到页面作为一个hidden input，之后再从post值中取出使用。

#### URL中的随机字符串

> 相对来说，这个问题就很小了，不过已有一些小纠结。第一次由于132的链接登陆后内容异常，只能使用133，而133的链接就多了这里的主角`随机字符串`, 当然这里随机是对我们来说的，正方教务系统后台肯定会有所控制，该字符串在一段时间内是不会发生改变的，我也只等了五分钟，五分钟已经足够完成正常的查询功能了。



### 实现方法
使用`urllib2`模拟登录,然后使用`BeaufitualSoup`解析出所需的表格内容. 登录难点是`url`中有一段随机字符串,需要先将其匹配出来获得`base_url`, 一开始时是每一次登录都匹配一次该随机字符串,也就是说查询一次成绩程序要访问两次正方教务系统, 现在使用了马伟伟同学写的`cache`模块,每500秒获取一次该字符串,提高了成绩查询速度.另一个就是模拟登录时的`post`内容中有一个`VIEWSTATE`值, 类似与`csrf_token`吧, 登录时需要先抓取该值然后`post`.  

### 该类中的方法  
+ `initlog`  初始化日志模块  
+ `__get_base_url()`  获取基础`url`,返回值类似`http://210.44.176.133/(mncumtmnxzhoz4550kr2qjnf)/`  
+ `cache_url()`  用来缓存`base_url`以提高成绩查询效率, 500秒更新一次`base_url`.
+ `__get_login_url` and `__get_url()`  用来获取需要查询信息的`url`
+ `login()` 实现登录功能,登录成功时返回`opener`,登录失败时返回`None`
+ `get_table()`  真正用来获取需要查询的信息.
