# Swagger2常用注解及其说明

## 1. Api
  
用在Controller中，标记一个Controller作为swagger的文档资源。

|属性名称|说明|
|---|---|
|value|Controller的注解|
|description|对api资源的描述|
|hidden |配置为true将在文档中隐藏|
  
使用方法：

```
@Api(value = "登录服务",description = "用户登录相关接口")
@RestController("loginControllerLayui")
@RequestMapping("/login")
public class LoginController {}
```

## 2. ApiOperation
  
该注解用在Controller的方法中，用于注解接口。

|属性名称|说明|
|---|---|
|value|接口的名称|
|notes|接口的注释|
|response|接口的返回类型，比如说：response = String.class|
|hidden|配置为true 将在文档中隐藏|

使用方法：

```
@ApiOperation(value = "添加权限",notes = "插入权限",response = JsonData.class)
@PostMapping("/insertAcl.json")
public JsonData insertAcl(@ApiParam(name = "param",value = "实体类AclParam",required = true) AclParam param){}
```

## 3. ApiParam
  
该注解用在方法的参数中。

|属性名称|说明|
|---|---|
|name|参数名称|
|value|参数值|
|required|是否必须，默认false|
|defaultValue|参数默认值|
|type|参数类型|
|hidden|隐藏该参数|

使用方法：

```
@ApiOperation(value = "添加权限",notes = "插入权限",response = JsonData.class)
@PostMapping("/insertAcl.json")
public JsonData insertAcl(@ApiParam(name = "param",value = "实体类AclParam",required = true) AclParam param){}
```

## 4. ApiResponses/ApiResponse
  
该注解用在Controller的方法中，用于注解方法的返回状态。

|属性名称|说明|
|---|---|
|code|http的状态码|
|message|状态的描述信息|
|response|状态相应，默认响应类 Void|

使用方法：

```
@ApiOperation(value = "菜单",notes = "进入菜单界面",nickname = "菜单界面")
@ApiResponses({
        @ApiResponse(code = 200,message = "成功！"),
        @ApiResponse(code = 401,message = "未授权！"),
        @ApiResponse(code = 404,message = "页面未找到！"),
        @ApiResponse(code = 403,message = "出错了！")
})
@GetMapping("/aclModule.page")
public ModelAndView aclModule(Model model){}
```

## 5. ApiModel
  
该注解用在实体类中。

|属性名称|说明|
|---|---|
|value|实体类名称|
|description|实体类描述|
|parent|集成的父类，默认为Void.class|
|subTypes|子类，默认为{}|
|reference|依赖，默认为“”|

使用方法：

```
@ApiModel(value = "JsonData",description = "返回的数据类型")
public class JsonData {}
```

## 6. ApiModelProperty
  
该注解用在实体类的字段中。

|属性名称|说明|
|---|---|
|name|属性名称|
|value|属性值|
|notes|属性注释|
|dataType|数据类型，默认为“”|
|required|是否必须，默认为false|
|hidden|是否隐藏该字段，默认为false|
|readOnly|是否只读，默认false|
|reference|依赖，，默认“”|
|allowEmptyValue|是否允许空值，默认为false|
|allowableValues|允许值，默认为“”|

使用方法：

```
//返回状态信息
@ApiModelProperty(name = "code",value = "状态code",notes = "返回信息的状态")
private int code;
//返回携带的信息内容
@ApiModelProperty(name = "msg",value = "状态信息",notes = "返回信息的内容")
private String msg = "";
//返回信息的总条数
@ApiModelProperty(name = "count",value = "查询数量",notes = "返回信息的条数")
private int count;
//返回对象
@ApiModelProperty(name = "data",value = "查询数据",notes = "返回数据的内容")
private Object data;
```

## 7. ApiImplicitParams/ApiImplicitParam
  
该注解用在Controller的方法中，同ApiParam的作用相同，但是比较建议使用ApiParam。

|属性名称|说明|
|---|---|
|name|参数名称|
|value|参数值|
|defaultValue|参数默认值|
|required|是否必须|
|allowMultiple|是否允许重复|
|dataType|数据类型|
|paramType|参数类型|

使用方法：

```
@ApiOperation(value = "创建用户",notes = "根据User对象创建用户")
@ApiImplicitParam(name = "user",value = "用户详细实体user")
@RequestMapping(value="/", method=RequestMethod.POST)
public String postUser(@ModelAttribute User user){}
```
