
在 Solon Mvc 里，@Mapping 注解一般是配合 @Controller 和 @Remoting，作请求路径映射用的。且，只支持加在 public 函数 或 类上。


### 1、注解属性




| 属性 | 说明 | 备注 |
| --- | --- | --- |
| value | 路径 | 与 path 互为别名 |
| path | 路径 | 与 value 互为别名 |
| method | 请求方式限定(def\=all) | 可用 `@Post`、`@Get` 等注解替代此属性 |
| consumes | 指定处理请求的提交内容类型 | 可用 `@Consumes` 注解替代此属性 |
| produces | 指定返回的内容类型 | 可用 `@Produces` 注解替代此属性 |
| multipart | 申明支持多分片请求(def\=false) | 如果为false，则自动识别 |


当 method\=all，即不限定请求方式


### 2、支持的路径映射表达式




| 符号 | 说明 | 示例 |
| --- | --- | --- |
| `**` | 任意字符、不限段数 | `**` 或 `/user/**` |
| `*` | 任意字符 | `/user/*` |
| `?` | 可有可无 | `/user/?` |
| `/` | 路径片段开始符和间隔符 | `/` 或 `/user` |
| `{name}` | 路径变量申明 | `/user/{name}` |


路径组合（控制器映射与动作映射）及应用示例：



```
import org.noear.solon.annotation.Controller;
import org.noear.solon.annotation.Mapping;

@Mapping("/user") //或 "user"，开头自动会补"/"
@Controller
public void DemoController{

    @Mapping("") //=/user
    public void home(){ }
    
    @Mapping("/") //=/user/，与上面是有区别的，注意下。
    public void home2(){ }
    
    @Mapping("/?") //=/user/ 或者 /user，与上面是有区别的，注意下。
    public void home3(){ }
    
    @Mapping("list") //=/user/list ，间隔自动会补"/"
    public void getList(){ }
    
    @Mapping("/{id}") //=/user/{id}
    public void getOne(long id){ }
    
    @Mapping("/ajax/**") //=/user/ajax/**
    public void ajax(){ }
}

```

提醒：一个 `@Mapping` 函数不支持多个路径的映射


### 3、参数注入


非请求参数的可注入对象：




| 类型 | 说明 |
| --- | --- |
| Context | 请求上下文（org.noear.solon.core.handle.Context） |
| Locale | 请求的地域信息，国际化时需要 |
| ModelAndView | 模型与视图对象（org.noear.solon.core.handle.ModelAndView） |


支持请求参数自动转换注入：


* 当变量名有对应的请求参数时（即有名字可对上的请求参数）
	+ 会直接尝试对请求参数值进行类型转换
* 当变量名没有对应的请求参数时
	+ 当变量为实体时：会尝试所有请求参数做为属性注入
	+ 否则注入 null


支持多种形式的请求参数直接注入：


* queryString
* form\-data
* x\-www\-form\-urlencoded
* path
* json body


其中 queryString, form\-data, x\-www\-form\-urlencoded, path 参数，支持 ctx.param() 接口统一获取。



```
import org.noear.solon.core.handle.Context;
import org.noear.solon.core.handle.ModelAndView;
import org.noear.solon.core.handle.UploadedFile;
import java.util.Locale;

@Mapping("/user") 
@Controller
public void DemoController{
    //非请求参数的可注入对象
    @Mapping("case1")
    public void case1(Context ctx, Locale locale , ModelAndView mv){ }
    
    //请求参数（可以是散装的；支持 queryString, form；json，或支持的其它序列化格式）
    @Mapping("case2")
    public void case2(String userName, String nickName, long[] ids, List names){ }
    
    //请求参数（可以是整装的结构体；支持 queryString, form；json，或支持的其它序列化格式Mapping
    @Mapping("case3")
    public void case3(UserModel user){ }
    
    //也可以是混搭的
    @Mapping("case4")
    public void case4(Context ctx, UserModel user, String userName){  }
    
    //文件上传    //注意与  名字对上
    @Mapping("case5")
    public void case5(UploadedFile file1, UploadedFile file2){ } 
    
    //同名多文件上传
    @Mapping("case6")
    public void case6(UploadedFile[] file){ }  
}

```

提醒： `?user[name]=1&ip[0]=a&ip[1]=b&order.id=1` 风格的参数注入，需要引入插件：[solon\-serialization\-properties](https://github.com)


### 4、带注解的参数注入


注解：




| 注解 | 说明 |
| --- | --- |
| @Param | 注入请求参数（包括：query\-string、form）。起到指定名字、默认值等作用 |
| @Header | 注入请求 header |
| @Cookie | 注入请求 cookie |
| @Path | 注入请求 path 变量（因为框架会自动处理，所以这个只是标识下方便文档生成用） |
| @Body | 注入请求体（一般会自动处理。仅在主体的 String, InputSteam, Map 时才需要） |


注解相关属性:




| 属性 | 说明 | 适用注解 |
| --- | --- | --- |
| value | 参数名字 | `@Param, @Header, @Cookie, @Path` |
| name | 参数名字（与 value 互为别名） | `@Param, @Header, @Cookie, @Path` |
| required | 必须的 | `@Param, @Header, @Cookie, @Body` |
| defaultValue | 默认值 | `@Param, @Header, @Cookie, @Body` |



```
import org.noear.solon.annotation.Controller;
import org.noear.solon.annotation.Mapping;
import org.noear.solon.annotation.Header;
import org.noear.solon.annotation.Body;
import org.noear.solon.annotation.Path;

@Mapping("/user") 
@Controller
public void DemoController{
    @Mapping("case1")
    public void case1(@Body String bodyStr){   }
    
    @Mapping("case2")
    public void case2(@Body Map paramMap, @Header("Token") String token){ }
    
    @Mapping("case3")
    public void case3(@Body InputStream stream, @Cookie("Token") token){  }
    
    //这个用例加不加 @Body 效果一样
    @Mapping("case4")
    public void case4(@Body UserModel user){  } 
    
    @Mapping("case5/{id}")
    public void case5(String id){  }
    
    //如果名字不同，才有必要用 @Path //否则是自动处理（如上）
    @Mapping("case5_2/{id}")
    public void case5_2(@Path("id") String name){  } 
    
    @Mapping("case6")
    public void case6(String name){ }
    
    //如果名字不同，才有必要用 @Param //否则是自动处理（如上）
    @Mapping("case6_2")
    public void case6_2(@Param("id") String name){ } 
    
    //如果要默认值，才有必要用 @Param
    @Mapping("case6_3")
    public void case6_3(@Param(defaultValue="world") String name){ }
    
    @Mapping("case7")
    public void case7(@Header String token){ }
    
    @Mapping("case7_2")
    public void case7_2(@Header String[] user){ } //v2.4.0 后支持
}

```

### 5、请求方式限定


可以1个或多个加个 @Mppaing 注解上，用于限定请求方式（不限，则支持全部请求方式）




| 请求方式限定注解 | 说明 |
| --- | --- |
| @Get | 限定为 Http Get 请求方式 |
| @Post | 限定为 Http Post 请求方式 |
| @Put | 限定为 Http Put 请求方式 |
| @Delete | 限定为 Http Delete 请求方式 |
| @Patch | 限定为 Http Patch 请求方式 |
| @Head | 限定为 Http Head 请求方式 |
| @Options | 限定为 Http Options 请求方式 |
| @Trace | 限定为 Http Trace 请求方式 |
| @Http | 限定为 Http 所有请求方式 |
|  |  |
| @Message | 限定为 Message 请求方式 |
| @To | 标注转发目标 |
|  |  |
| @WebSokcet | 限定为 WebSokcet 请求方式 |
|  |  |
| @Sokcet | 限定为 Sokcet 请求方式 |
|  |  |
| @All | 允许所有请求方式（默认） |




| 其它限定注解 | 说明 |
| --- | --- |
| @Produces | 申明输出内容类型 |
| @Consumes | 申明输入内容类型（当输出内容类型未包函 @Consumes，则响应为 415 状态码） |
| @Multipart | 显式申明支持 Multipart 输入 |


例：



```
import org.noear.solon.boot.web.MimeType;

@Mapping("/user") 
@Controller
public void DemoController{
    @Get
    @Mapping("case1")
    public void case1(Context ctx, Locale locale , ModelAndView mv){ }
   
    //也可以直接使用 Mapping 的属性进行限定。。。但是没使用注解的好看
    @Mapping(path = "case1_2", method = MethodType.GET)
    public void case1_2(Context ctx, Locale locale , ModelAndView mv){ }
    
    @Put
    @Message
    @Mapping("case2")
    public void case2(String userName, String nickName){ }
    
    //如果没有输出申明，侧 string 输出默认为 "text/plain"
    @Produces(MimeType.APPLICATION_JSON_VALUE)
    @Mapping("case3")
    public String case3(){
        return "{code:1}";
    }
    
    ////也可以直接使用 Mapping 的属性进行限定。。。但是没使用注解的好看
    @Mapping(path= "case3_2", produces=MimeType.APPLICATION_JSON_VALUE))
    public String case3_2(){
        return "{code:1}";
    }
    
    //如果没有输出申明，侧 object 输出默认为 "application/json"
    @Mapping("case3_3")
    public User case3_3(){
        return new User();
    }
    
}

```

### 6、输出类型



```
@Mapping("/user") 
@Controller
public void DemoController{

    //输出视图与模型，经后端渲染后输出最终格式
    @Maping("case1")
    public ModelAndView case1(){
        ModelAndView mv = new ModelAndView();
        mv.put("name", "world");
        mv.view("hello.ftl");
        
        return mv;
    }
    
    //输出结构体，默认会采用josn格式渲染后输出
    @Maping("case2")
    public UserModel case2(){
        return new UserModel();
    }
    
    //输出下载文件
    @Maping("case3")
    public Object case3(){
        return new File(...); //或者 return new DownloadedFile(...);
    }
}

```

### 7、父类继承支持



```
@Mapping("user")
public void UserController extends CrudControllerBase{
           
}

public class CrudControllerBase{
    @Post
    @Mapping("add")
    public void add(T t){
        ...
    }

    @Delete
    @Mapping("remove")
    public void remove(T t){
        ...
    }
}

```

 本博客参考[楚门加速器](https://shexiangshi.org)。转载请注明出处！
