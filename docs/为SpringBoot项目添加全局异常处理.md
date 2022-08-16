# `SpringBoot`全局异常处理

## 原理

发生异常是不可避免的事，做好异常的处理工作是必不可少的。通过`SpringBoot`可以实现 将对异常的处理，由`AOP`切面织入的方法，完全交由我们控制。

## 实现

1. 新建一个全局异常处理类，用`@ControllerAdvice`注解修饰此类
2. 新建方法，用`@ExpectionHandler(异常类型.class)`修饰该方法
3. 可以使用`@ResponseStatus(code = HttpStatus.ACCEPTED)`的方式修改http状态码

> 下面使用一段代码样例展示缺少参数的全局异常处理模板

```java
//AOP切面织入advice通知
@ControllerAdvice
//使用RestController可以使得方法前不需要添加@ResponseBody注解
@RestController
public class GlobalExceptionHandler {
    
    @ExceptionHandler(MissingServletRequestParameterException.class)
    //@ResponseStatus指定http的状态码
    @ResponseStatus(code = HttpStatus.ACCEPTED)
    public CommonResponse<Object> missingParam(){
        return CommonResponse.createForError(
                ResponseCode.ILLEGAL_ARGUMENT.getCode(),
                ResponseCode.ILLEGAL_ARGUMENT.getDescription()
        );
    }
    
}
```

