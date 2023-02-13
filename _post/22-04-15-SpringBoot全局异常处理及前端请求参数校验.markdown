---
layout:  post
title:   SpringBoot全局异常处理及前端请求参数校验
date:   22-04-15 22:01:58 修改
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-Springboot
-java
-spring

---


## SpringBoot全局异常捕获处理及参数校验







在日常开发中，为了不抛出异常堆栈信息给前端页面，每次编写Controller层代码都要尽可能的catch住所有service层、dao层等异常，代码耦合性较高，参数校验逻辑业务逻辑还长，不利于后期维护。

为解决该问题，可以将Controller层异常信息统一封装处理，且能区分对待Controller层方法返回给前端。


一共有两种方法：

- Spring的AOP 
- @ControllerAdvice结合@ExceptionHandler

我们这里使用第二种方式

## 统一结果封装

首先后端应该返回给前端统一的数据格式，所以为了方便，我们新建两个类，一个用于统一的返回类型，另一个是所有的枚举类型，用于返回前端正确或者错的信息。

比如：


![img](https://img-blog.csdnimg.cn/img_convert/350c1d342f9fb63ddd7de0cc3cb750c0.png)

### 统一返回结果

```java
import java.util.Map;

/**
 * 自定义响应数据类型
 *
 * @Package com.imooc.utils
 * @Description: 自定义响应数据结构
 *
 */
public class GraceJSONResult {
   

    // 响应业务状态码
    private Integer status;

    // 响应消息
    private String msg;

    // 是否成功
    private Boolean success;

    // 响应数据，可以是Object，也可以是List或Map等
    private Object data;

    /**
     * 成功返回，带有数据的，直接往OK方法丢data数据即可
     * @param data
     * @return
     */
    public static GraceJSONResult ok(Object data) {
   
        return new GraceJSONResult(data);
    }
    /**
     * 成功返回，不带有数据的，直接调用ok方法，data无须传入（其实就是null）
     * @return
     */
    public static GraceJSONResult ok() {
   
        return new GraceJSONResult(ResponseStatusEnum.SUCCESS);
    }
    public GraceJSONResult(Object data) {
   
        this.status = ResponseStatusEnum.SUCCESS.status();
        this.msg = ResponseStatusEnum.SUCCESS.msg();
        this.success = ResponseStatusEnum.SUCCESS.success();
        this.data = data;
    }


    /**
     * 错误返回，直接调用error方法即可，当然也可以在ResponseStatusEnum中自定义错误后再返回也都可以
     * @return
     */
    public static GraceJSONResult error() {
   
        return new GraceJSONResult(ResponseStatusEnum.FAILED);
    }

    /**
     * 错误返回，map中包含了多条错误信息，可以用于表单验证，把错误统一的全部返回出去
     * @param map
     * @return
     */
    public static GraceJSONResult errorMap(Map map) {
   
        return new GraceJSONResult(ResponseStatusEnum.FAILED, map);
    }

    /**
     * 错误返回，直接返回错误的消息
     * @param msg
     * @return
     */
    public static GraceJSONResult errorMsg(String msg) {
   
        return new GraceJSONResult(ResponseStatusEnum.FAILED, msg);
    }

    /**
     * 错误返回，token异常，一些通用的可以在这里统一定义
     * @return
     */
    public static GraceJSONResult errorTicket() {
   
        return new GraceJSONResult(ResponseStatusEnum.TICKET_INVALID);
    }

    /**
     * 自定义错误范围，需要传入一个自定义的枚举，可以到[ResponseStatusEnum.java[中自定义后再传入
     * @param responseStatus
     * @return
     */
    public static GraceJSONResult errorCustom(ResponseStatusEnum responseStatus) {
   
        return new GraceJSONResult(responseStatus);
    }
    public static GraceJSONResult exception(ResponseStatusEnum responseStatus) {
   
        return new GraceJSONResult(responseStatus);
    }

    public GraceJSONResult(ResponseStatusEnum responseStatus) {
   
        this.status = responseStatus.status();
        this.msg = responseStatus.msg();
        this.success = responseStatus.success();
    }
    public GraceJSONResult(ResponseStatusEnum responseStatus, Object data) {
   
        this.status = responseStatus.status();
        this.msg = responseStatus.msg();
        this.success = responseStatus.success();
        this.data = data;
    }
    public GraceJSONResult(ResponseStatusEnum responseStatus, String msg) {
   
        this.status = responseStatus.status();
        this.msg = msg;
        this.success = responseStatus.success();
    }

    public GraceJSONResult() {
   
    }

    public Integer getStatus() {
   
        return status;
    }

    public void setStatus(Integer status) {
   
        this.status = status;
    }

    public String getMsg() {
   
        return msg;
    }

    public void setMsg(String msg) {
   
        this.msg = msg;
    }

    public Object getData() {
   
        return data;
    }

    public void setData(Object data) {
   
        this.data = data;
    }

    public Boolean getSuccess() {
   
        return success;
    }

    public void setSuccess(Boolean success) {
   
        this.success = success;
    }
}
```

### 枚举类

```java
/**
 * 响应结果枚举，用于提供给GraceJSONResult返回给前端的
 * 本枚举类中包含了很多的不同的状态码供使用，可以自定义
 * 便于更优雅的对状态码进行管理，一目了然
 */
public enum ResponseStatusEnum {
   

    SUCCESS(200, true, "操作成功！"),
    FAILED(500, false, "操作失败！"),

    // 50x
    UN_LOGIN(501,false,"请登录后再继续操作！"),
    TICKET_INVALID(502,false,"会话失效，请重新登录！"),
    NO_AUTH(503,false,"您的权限不足，无法继续操作！"),
    MOBILE_ERROR(504,false,"短信发送失败，请稍后重试！"),
    SMS_NEED_WAIT_ERROR(505,false,"短信发送太快啦~请稍后再试！"),
    SMS_CODE_ERROR(506,false,"验证码过期或不匹配，请稍后再试！"),
    USER_FROZEN(507,false,"用户已被冻结，请联系管理员！"),
    USER_UPDATE_ERROR(508,false,"用户信息更新失败，请联系管理员！"),
    USER_INACTIVE_ERROR(509,false,"请前往[账号设置]修改信息激活后再进行后续操作！"),
    USER_INFO_UPDATED_ERROR(5091,false,"用户信息修改失败！"),
    USER_INFO_UPDATED_NICKNAME_EXIST_ERROR(5092,false,"昵称已经存在！"),
    USER_INFO_UPDATED_IMOOCNUM_EXIST_ERROR(5092,false,"慕课号已经存在！"),
    USER_INFO_CANT_UPDATED_IMOOCNUM_ERROR(5092,false,"慕课号无法修改！"),
    FILE_UPLOAD_NULL_ERROR(510,false,"文件不能为空，请选择一个文件再上传！"),
    FILE_UPLOAD_FAILD(511,false,"文件上传失败！"),
    FILE_FORMATTER_FAILD(512,false,"文件图片格式不支持！"),
    FILE_MAX_SIZE_500KB_ERROR(5131,false,"仅支持500kb大小以下的图片上传！"),
    FILE_MAX_SIZE_2MB_ERROR(5132,false,"仅支持2MB大小以下的图片上传！"),
    FILE_NOT_EXIST_ERROR(514,false,"你所查看的文件不存在！"),
    USER_STATUS_ERROR(515,false,"用户状态参数出错！"),
    USER_NOT_EXIST_ERROR(516,false,"用户不存在！"),

    // 自定义系统级别异常 54x
    SYSTEM_INDEX_OUT_OF_BOUNDS(541, false, "系统错误，数组越界！"),
    SYSTEM_ARITHMETIC_BY_ZERO(542, false, "系统错误，无法除零！"),
    SYSTEM_NULL_POINTER(543, false, "系统错误，空指针！"),
    SYSTEM_NUMBER_FORMAT(544, false, "系统错误，数字转换异常！"),
    SYSTEM_PARSE(545, false, "系统错误，解析异常！"),
    SYSTEM_IO(546, false, "系统错误，IO输入输出异常！"),
    SYSTEM_FILE_NOT_FOUND(547, false, "系统错误，文件未找到！"),
    SYSTEM_CLASS_CAST(548, false, "系统错误，类型强制转换错误！"),
    SYSTEM_PARSER_ERROR(549, false, "系统错误，解析出错！"),
    SYSTEM_DATE_PARSER_ERROR(550, false, "系统错误，日期解析出错！"),

    // admin 管理系统 56x
    ADMIN_USERNAME_NULL_ERROR(561, false, "管理员登录名不能为空！"),
    ADMIN_USERNAME_EXIST_ERROR(562, false, "管理员登录名已存在！"),
    ADMIN_NAME_NULL_ERROR(563, false, "管理员负责人不能为空！"),
    ADMIN_PASSWORD_ERROR(564, false, "密码不能为空后者两次输入不一致！"),
    ADMIN_CREATE_ERROR(565, false, "添加管理员失败！"),
    ADMIN_PASSWORD_NULL_ERROR(566, false, "密码不能为空！"),
    ADMIN_NOT_EXIT_ERROR(567, false, "管理员不存在或密码错误！"),
    ADMIN_FACE_NULL_ERROR(568, false, "人脸信息不能为空！"),
    ADMIN_FACE_LOGIN_ERROR(569, false, "人脸识别失败，请重试！"),
    CATEGORY_EXIST_ERROR(570, false, "文章分类已存在，请换一个分类名！"),

    // 媒体中心 相关错误 58x
    ARTICLE_COVER_NOT_EXIST_ERROR(580, false, "文章封面不存在，请选择一个！"),
    ARTICLE_CATEGORY_NOT_EXIST_ERROR(581, false, "请选择正确的文章领域！"),
    ARTICLE_CREATE_ERROR(582, false, "创建文章失败，请重试或联系管理员！"),
    ARTICLE_QUERY_PARAMS_ERROR(583, false, "文章列表查询参数错误！"),
    ARTICLE_DELETE_ERROR(584, false, "文章删除失败！"),
    ARTICLE_WITHDRAW_ERROR(585, false, "文章撤回失败！"),
    ARTICLE_REVIEW_ERROR(585, false, "文章审核出错！"),
    ARTICLE_ALREADY_READ_ERROR(586, false, "文章重复阅读！"),

    // 人脸识别错误代码
    FACE_VERIFY_TYPE_ERROR(600, false, "人脸比对验证类型不正确！"),
    FACE_VERIFY_LOGIN_ERROR(601, false, "人脸登录失败！"),

    // 系统错误，未预期的错误 555
    SYSTEM_ERROR(555, false, "系统繁忙，请稍后再试！"),
    SYSTEM_OPERATION_ERROR(556, false, "操作失败，请重试或联系管理员"),
    SYSTEM_RESPONSE_NO_INFO(557, false, ""),
    SYSTEM_ERROR_GLOBAL(558, false, "全局降级：系统繁忙，请稍后再试！"),
    SYSTEM_ERROR_FEIGN(559, false, "客户端Feign降级：系统繁忙，请稍后再试！"),
    SYSTEM_ERROR_ZUUL(560, false, "请求系统过于繁忙，请稍后再试！");


    // 响应业务状态
    private Integer status;
    // 调用是否成功
    private Boolean success;
    // 响应消息，可以为成功或者失败的消息
    private String msg;

    ResponseStatusEnum(Integer status, Boolean success, String msg) {
   
        this.status = status;
        this.success = success;
        this.msg = msg;
    }

    public Integer status() {
   
        return status;
    }
    public Boolean success() {
   
        return success;
    }
    public String msg() {
   
        return msg;
    }
}
```

## 使用验证框架

定义完返回前端的统一对象之后，我们应该如何优雅的处理异常呢，我们使用hibernate验证框架，以及@ControllerAdvice+@ExceptionHandler全局异常处理。

当用户输入的参数不符合我们所设置的格式时，hibernate将报出异常信息，然后被@ControllerAdvice+@ExceptionHandler拦截。通过细粒度的拦截异常信息，可以精确判断返回异常信息。

### 添加hibernate验证框架坐标

```java
<!-- hibernate 验证框架 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 统一异常拦截处理

```java
/**
 * 统一异常拦截处理
 * 可以针对异常的类型进行捕获，然后返回json信息到前端
 */
@ControllerAdvice
public class GraceExceptionHandler {
   

    @ExceptionHandler(MyCustomException.class)
    @ResponseBody
    public GraceJSONResult returnMyException(MyCustomException e) {
   
        e.printStackTrace();
        return GraceJSONResult.exception(e.getResponseStatusEnum());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseBody
    public GraceJSONResult returnMethodArgumentNotValid(MethodArgumentNotValidException e) {
   
        BindingResult result = e.getBindingResult();
        Map<String, String> map = getErrors(result);
        return GraceJSONResult.errorMap(map);
    }

    //验证前端传过来的参数是否存在错误
    public Map<String,String> getErrors(BindingResult result){
   
        Map<String, String> map = new HashMap<>();
        List<FieldError> errorList = result.getFieldErrors();
        for (FieldError error : errorList) {
   
            map.put(error.getField(), error.getDefaultMessage());
        }
        return map;
    }
}
```

### 测试

使用RegistLoginBO接收前端的请求参数

```java
public class RegistLoginBO {
   

    @NotBlank(message="手机号不能为空")
    @Length(min=11,max=11,message="手机号长度必须为11位")
    private String mobile;
    @NotBlank(message="验证码不能为空")
    private String verifyCode;
	 //setter and getter
}
```

```java
@PostMapping("login")
public GraceJSONResult login(@Valid @RequestBody RegistLoginBO registLoginBO) throws Exception {
   
   
    return GraceJSONResult.ok();
}
```


正确传参![img](https://img-blog.csdnimg.cn/img_convert/b7a577e20665e2467e0d4576b6d85fb5.png)

错误传参


![img](https://img-blog.csdnimg.cn/img_convert/a9e32b78b602e7a1b021d6239bb0c2b5.png)

## 常用注解

```java
@Null 验证对象是否为null

@NotNull 验证对象是否不为null, 无法查检长度为0的字符串

@NotBlank 检查约束字符串是不是Null还有被Trim的长度是否大于0,只对字符串,且会去掉前后空格.

@NotEmpty 检查约束元素是否为NULL或者是EMPTY.


@AssertTrue 验证 Boolean 对象是否为 true

@AssertFalse 验证 Boolean 对象是否为 false
长度检查

@Size(min=, max=) 验证对象（Array,Collection,Map,String）长度是否在给定的范围之内

@Length(min=, max=) Validates that the annotated string is between min and max included.

日期检查

@Past 验证 Date 和 Calendar 对象是否在当前时间之前

@Future 验证 Date 和 Calendar 对象是否在当前时间之后

@Pattern 验证 String 对象是否符合正则表达式的规则

数值检查

建议使用在Stirng,Integer类型，不建议使用在int类型上，因为表单值为“”时无法转换为int，但可以转换为Stirng为"",Integer为null

@Min 验证 Number 和 String 对象是否大等于指定的值

@Max 验证 Number 和 String 对象是否小等于指定的值

@DecimalMax 被标注的值必须不大于约束中指定的最大值. 这个约束的参数是一个通过BigDecimal定义的最大值的字符串表示.小数存在精度

@DecimalMin 被标注的值必须不小于约束中指定的最小值. 这个约束的参数是一个通过BigDecimal定义的最小值的字符串表示.小数存在精度

@Digits 验证 Number 和 String 的构成是否合法

@Digits(integer=,fraction=) 验证字符串是否是符合指定格式的数字，interger指定整数精度，fraction指定小数精度。 @Range(min=, max=) Checks whether the annotated value lies between (inclusive) the specified minimum and maximum. @Range(min=10000,max=50000,message=“range.bean.wage”) private BigDecimal wage;

@Valid 递归的对关联对象进行校验, 如果关联对象是个集合或者数组,那么对其中的元素进行递归校验,如果是一个map,则对其中的值部分进行校验.(是否进行递归验证)

@CreditCardNumber信用卡验证

@Email 验证是否是邮件地址，如果为null,不进行验证，算通过验证。

@ScriptAssert(lang= ,script=, alias=)

@URL(protocol=,host=, port=,regexp=, flags=)
```

