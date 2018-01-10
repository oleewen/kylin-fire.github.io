# 方法调用处理探讨

<<<<<<< HEAD
[toc]

=======
>>>>>>> 2e5dd704434a0c113bc585ccbda4c346dc5a079a
## 前言

我们常在一个方法调用时，希望能对方法的调用做一些处理，例如异常、失败、调用时间过长等等，于是就有了这次探讨。
随着探讨深入，会发现一个调用的处理是有多方面需要考虑的，让我们先从一个简单的例子开始。

<<<<<<< HEAD
## 演进
### 远古时代
=======
## 远古时代
>>>>>>> 2e5dd704434a0c113bc585ccbda4c346dc5a079a
如果要根据一个商品id，查询商品，常见的调用代码是这样的：

list1：直接调用

```java
/** 商品服务 */
private ItemCall itemCall;

public Item acquireItemById(Long itemId) {
   // 根据商品id，调用商品服务查询商品
   ItemEntity itemEntity = itemCall.getItemById(itemId);
   // Entity转DomainObject
   return ItemFactory.asDomain(itemEntity);
}
```

<<<<<<< HEAD
### 石器时代
=======
## 石器时代
>>>>>>> 2e5dd704434a0c113bc585ccbda4c346dc5a079a
list1代码调用代码封装的不错，屏蔽了数据对象细节。

主要问题
- 没有捕捉异常，当调用异常时，虽有异常堆栈，但当前上下文（itemId）一定要记录下来
- 有些服务的声明了受检异常，会导致该异常在未捕捉的链路上都需要声明受检异常

解决办法

list2：捕捉异常

```java
/** 商品服务 */
private ItemCall itemCall;

public Item acquireItemById(Long itemId) {
   try {
      // 根据商品id，调用商品服务查询商品
      ItemEntity itemEntity = itemCall.getItemById(itemId);
      // Entity转DomainObject
      return ItemFactory.asDomain(itemEntity);
   } catch (Throwable e) {
       // 记录日志（含调用堆栈和上下文）
       logger.error(String.format("exception@itemId:%s", itemId), e);
       // 抛出非受检异常
       throw new RepositoryException(e);
   }
}
```

<<<<<<< HEAD
### 青铜时代
=======
## 青铜时代
>>>>>>> 2e5dd704434a0c113bc585ccbda4c346dc5a079a
list2处理了异常，出现异常时记录了当前调用堆栈和上下文，也对包装了异常，屏蔽了受检异常的繁琐。

主要问题
- 当ItemCall调用失败时，我们希望能对失败做记录上下文处理
- 同时，失败的原因，需要反馈给上层

解决办法

list3：处理失败

```java
/** 商品服务 */
private ItemCall itemCall;

public Result<Item> acquireItemById(Long itemId) {
   try {
      // 根据商品id，调用商品服务查询商品
      Result<ItemEntity> result = itemCall.getItemById(itemId);
      // 请求是否成功
      if(result == null || result.isSuccess()){
          // 记录错误日志
          logger.error(String.format("failure@itemId:%s;result:%s", itemId, JsonHelper.toJson(result)));
          // 返回
          return new Result(Status.ITEM_GET_FAILURE, itemId);
      }
      // Entity转DomainObject
      Item item = ItemFactory.asDomain(result.getData());
      return new Result(item);
   } catch (Throwable e) {
       // 记录日志（含调用堆栈和上下文）
       logger.error(String.format("exception@itemId:%s", itemId), e);
       // 抛出非受检异常
       throw new RepositoryException(e);
   }
}
```

<<<<<<< HEAD
### 铁器时代
=======
## 铁器时代
>>>>>>> 2e5dd704434a0c113bc585ccbda4c346dc5a079a
list3对成功失败都做了处理，但调用方需要对成功、失败、异常分别做处理，尤其是成功和失败，每次都需要通过result!=null && result.isSuccess()判定成功，无形增加了代码复杂度。

主要问题
- Result判断成功通过result!=null && result.isSuccess()，较复杂
- 整个链路都需要大量处理状态判断的代码

解决办法

list4：用异常封装状态

```java
/** 商品服务 */
private ItemCall itemCall;

public Item acquireItemById(Long itemId) {
   try {
      // 根据商品id，调用商品服务查询商品
      Result<ItemEntity> result = itemCall.getItemById(itemId);
      // 请求是否成功
      if(result == null || result.isSuccess()){
          // 记录错误日志
          logger.error(String.format("failure@itemId:%s;result:%s", itemId, JsonHelper.toJson(result)));
          // 抛出非受检异常
          throw new RepositoryException(Status.ITEM_GET_FAILURE, itemId);
      }
      // Entity转DomainObject
      return ItemFactory.asDomain(result.getData());
   } catch (Throwable e) {
       // 记录日志（含调用堆栈和上下文）
       logger.error(String.format("exception@itemId:%s", itemId), e);
       // 抛出非受检异常
       throw new RepositoryException(e);
   }
}
```

<<<<<<< HEAD
### 工业革命1
=======
## 工业革命1
>>>>>>> 2e5dd704434a0c113bc585ccbda4c346dc5a079a
list4用异常来封装状态，上层调用再也不需要关注返回值的处理，有异常和失败的时候，仅对异常做捕捉处理。

问题也随之而来，从list1两行代码到list4十行代码，每次方法调用，都需要增加八行代码对异常和失败做处理。

主要问题
- 每个方法调用都需要增加8行异常和失败处理代码

解决办法

list5：使用AOP

```java
/** 商品服务 */
private ItemCall itemCall;

public Item acquireItemById(Long itemId) {
   // 根据商品id，调用商品服务查询商品
   Result<ItemEntity> result = itemCall.getItemById(itemId);
   // Entity转DomainObject
   return ItemFactory.asDomain(result.getData());
}
```

```java
@Aspect
public class CallAspect {
    Logger logger = LoggerFactory.getLogger(CallAround.class);

    @Pointcut("execution(* com.company.department.business.appname.call.*(..))")
    public void callPoint() {
    }

    @Around("callPoint()")
    public Object call(ProceedingJoinPoint joinPoint) throws Throwable {
        // 方法签名
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();

        long start = System.currentTimeMillis();

        Object result = null;
        try {
            // 执行方法
            result = joinPoint.proceed();

            long end = System.currentTimeMillis();

            // 打印日志
            logger(joinPoint, call, end - start, result);
        } catch (Throwable e) {
            long end = System.currentTimeMillis();
            // 打印日志
            error(joinPoint, call, end - start, e);
        }
        return result;
    }

    private void logger(ProceedingJoinPoint joinPoint, Call call, long elapsed, Object result) {
        Object[] args = joinPoint.getArgs();

        // 调用成功
        if (isSuccess(result)) {
            return;
        }

        // 调用失败，访问日志
        logger.warn(String.format("access@args:%s,elapsed:%d;duration:%d,success:false,result:%s", JsonHelper.toJson(args), call.elapsed(), elapsed, JsonHelper.toJson(result)));
    }

    /**
    * 判断Result是否成功，允许子类覆盖
    * @param result 调用结果
    */
    protected boolean isSuccess(Object result) {
        if (result != null) {
            if (result instanceof ResultSupport) {
                return ((ResultSupport) result).isSuccess();
            }
            return true;
        }
        return false;
    }

    private void error(ProceedingJoinPoint joinPoint, Call call, long elapsed, Throwable t) {
        // 记录异常日志
        logger.error(String.format("exception@args:%s,elapsed:%d;duration:%d", JsonHelper.toJson(joinPoint.getArgs()), call.elapsed(), elapsed), t);
        // 重新抛出异常
        throw new RepositoryException(e);
    }
}
```

<<<<<<< HEAD
### 工业革命2
=======
## 工业革命2
>>>>>>> 2e5dd704434a0c113bc585ccbda4c346dc5a079a
list5用到了AOP切面，解决了重复的异常和失败处理，更近一步，采用注解，可以携带注解信息

解决办法

list6：使用注解

```java
/** 商品服务 */
private ItemCall itemCall;

public Item acquireItemById(Long itemId) {
   // 根据商品id，调用商品服务查询商品
   Result<ItemEntity> result = itemCall.getItemById(itemId);
   // Entity转DomainObject
   return ItemFactory.asDomain(result.getData());
}
```

声明Call注解

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Call {
}
```

声明CallAspect切面，指定接入点为加了@Call注解的方法

```java
@Aspect
public class CallAspect {

    @Pointcut("execution(* com.company.department.business.appname.*.*(..)) && @annotation(org.springframework.ext.common.aspect.Call)")
    public void callPoint() {
    }

    @Around("callPoint()")
    public Object call(ProceedingJoinPoint joinPoint) throws Throwable {
      // 方法签名
      MethodSignature signature = (MethodSignature) joinPoint.getSignature();
      Method method = signature.getMethod();

      Call call = method.getAnnotation(Call.class);

      long start = System.currentTimeMillis();

      Object result = null;
      try {
          // 执行方法
          result = joinPoint.proceed();

          long end = System.currentTimeMillis();
          // 有@Logger注解
          if (call != null) {
              // 打印日志
              logger(joinPoint, call, end - start, result);
          }
      } catch (Throwable e) {
          long end = System.currentTimeMillis();
          // 打印日志
          error(joinPoint, call, end - start, e);
      }
      return result;
    }

    private void logger(ProceedingJoinPoint joinPoint, Call call, long elapsed, Object result) {
        Logger logger = getLogger(joinPoint, call);

        Object[] args = joinPoint.getArgs();

        // 调用成功
        if (isSuccess(result)) {
            return;
        }

        // 调用失败，访问日志
        logger.warn(String.format("access@args:%s,elapsed:%d;duration:%d,success:false,result:%s", JsonHelper.toJson(args), call.elapsed(), elapsed, JsonHelper.toJson(result)));
    }

    /**
    * 判断Result是否成功，允许子类覆盖
    * @param result 调用结果
    */
    protected boolean isSuccess(Object result) {
        if (result != null) {
            if (result instanceof ResultSupport) {
                return ((ResultSupport) result).isSuccess();
            }
            return true;
        }
        return false;
    }

    private void error(ProceedingJoinPoint joinPoint, Call call, long elapsed, Throwable t) {
        Logger logger = getLogger(joinPoint, call);

        // 记录异常日志
        logger.error(String.format("exception@args:%s,elapsed:%d;duration:%d", JsonHelper.toJson(joinPoint.getArgs()), call.elapsed(), elapsed), t);
        // 重新抛出异常
        throw new RepositoryException(e);
    }

    private Logger getLogger(ProceedingJoinPoint joinPoint, Call call) {
        return LoggerFactory.getLogger(StringUtils.defaultIfBlank(call.value(), joinPoint.getClass().getName()));
    }
}
```

<<<<<<< HEAD
### 工业革命3
=======
## 工业革命3
>>>>>>> 2e5dd704434a0c113bc585ccbda4c346dc5a079a
list6通过AOP+注解将调用异常、失败都处理了，但方法调用其实还有几种情况需要考虑：超时、采样、调试

主要问题
- 覆盖方法调用超时
- 进行采样日志打印
- 支持程序debug

解决办法

list7：支持超时、采用、调试

```java
/** 商品服务 */
private ItemCall itemCall;

public Item acquireItemById(Long itemId) {
   // 根据商品id，调用商品服务查询商品
   Result<ItemEntity> result = itemCall.getItemById(itemId);
   // Entity转DomainObject
   return ItemFactory.asDomain(result.getData());
}
```

声明Call注解

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Call {
    /** 日志名 */
    String value() default "";

    /** 耗时阀值 */
    int elapsed() default 1000;

    /** 采样机率:sample/10000 */
    int sample() default 1;

    /** 采样基数：默认10000 */
    int basic() default 10000;
}
```

声明CallAspect切面，指定接入点为加了@Call注解的方法

```java
@Aspect
public class CallAspect {

    @Pointcut("execution(* com.company.department.business.appname.*.*(..)) && @annotation(org.springframework.ext.common.aspect.Call)")
    public void callPoint() {
    }

    @Around("callPoint()")
    public Object call(ProceedingJoinPoint joinPoint) throws Throwable {
      // 方法签名
      MethodSignature signature = (MethodSignature) joinPoint.getSignature();
      Method method = signature.getMethod();

      Call call = method.getAnnotation(Call.class);

      long start = System.currentTimeMillis();

      Object result = null;
      try {
          // 执行方法
          result = joinPoint.proceed();

          long end = System.currentTimeMillis();
          // 有@Logger注解
          if (call != null) {
              // 打印日志
              logger(joinPoint, call, end - start, result);
          }
      } catch (Throwable e) {
          long end = System.currentTimeMillis();
          // 打印日志
          error(joinPoint, call, end - start, e);
      }
      return result;
    }

    private void logger(ProceedingJoinPoint joinPoint, Call call, long elapsed, Object result) {
        Logger logger = getLogger(joinPoint, call);

        Object[] args = joinPoint.getArgs();

        // 调用成功
        if (isSuccess(result)) {
            // 超时，打印日志：默认1s超时
            if (elapsed > call.elapsed()) {
                logger.error(String.format("timeout@args:%s,elapsed:%d;duration:%d,result:%s", JsonHelper.toJson(args), call.elapsed(), elapsed, JsonHelper.toJson(result)));
            }
            // 符合采样频率条件：默认万分之一
            else if (random.nextInt(call.basic()) <= call.sample()) {
                logger.warn(String.format("sample@args:%s,elapsed:%d;duration:%d,result:%s", JsonHelper.toJson(args), call.elapsed(), elapsed, JsonHelper.toJson(result)));
            }
            // debug日志：线程ThreadLocal级debug
            else if (Context.debug()) {
                logger.warn(String.format("debug@args:%s,elapsed:%d;duration:%d,result:%s", JsonHelper.toJson(args), call.elapsed(), elapsed, JsonHelper.toJson(result)));
            }
            // 其他（未超时、未采用命中），不打印日志
            return;
        }

        // 调用失败，访问日志
        logger.warn(String.format("access@args:%s,elapsed:%d;duration:%d,success:false,result:%s", JsonHelper.toJson(args), call.elapsed(), elapsed, JsonHelper.toJson(result)));
    }

    /**
    * 判断Result是否成功，允许子类覆盖
    * @param result 调用结果
    */
    protected boolean isSuccess(Object result) {
        if (result != null) {
            if (result instanceof ResultSupport) {
                return ((ResultSupport) result).isSuccess();
            }
            return true;
        }
        return false;
    }

    private void error(ProceedingJoinPoint joinPoint, Call call, long elapsed, Throwable t) {
        Logger logger = getLogger(joinPoint, call);

        // 记录异常日志
        logger.error(String.format("exception@args:%s,elapsed:%d;duration:%d", JsonHelper.toJson(joinPoint.getArgs()), call.elapsed(), elapsed), t);
        // 重新抛出异常
        throw new RepositoryException(e);
    }

    private Logger getLogger(ProceedingJoinPoint joinPoint, Call call) {
        return LoggerFactory.getLogger(StringUtils.defaultIfBlank(call.value(), joinPoint.getClass().getName()));
    }
}
```

<<<<<<< HEAD
### 工业革命4
=======
## 工业革命4
>>>>>>> 2e5dd704434a0c113bc585ccbda4c346dc5a079a
list7对方法调用的多数情况做了处理，遇到额外的情况如何处理？CallAspect如何做到通用性？

主要问题
- 支持额外的情况扩展
- 通用的CallAspect切面

解决办法

list8：切面实例化延迟（子类实例化）

```java
/** 商品服务 */
private ItemCall itemCall;

public Item acquireItemById(Long itemId) {
   // 根据商品id，调用商品服务查询商品
   Result<ItemEntity> result = itemCall.getItemById(itemId);
   // Entity转DomainObject
   return ItemFactory.asDomain(result.getData());
}
```

声明Call注解

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Call {
    /** 日志名 */
    String value() default "";

    /** 耗时阀值 */
    int elapsed() default 1000;

    /** 采样机率:sample/10000 */
    int sample() default 1;

    /** 采样基数：默认10000 */
    int basic() default 10000;
}
```

声明CallAround切面，实现Around调用处理，支持调用是否成功判断、支持其他case扩展

```java
public class CallAround {
    @Around("callPoint()")
    public Object call(ProceedingJoinPoint joinPoint) throws Throwable {
      // 方法签名
      MethodSignature signature = (MethodSignature) joinPoint.getSignature();
      Method method = signature.getMethod();

      Call call = method.getAnnotation(Call.class);

      long start = System.currentTimeMillis();

      Object result = null;
      try {
          // 执行方法
          result = joinPoint.proceed();

          long end = System.currentTimeMillis();
          // 有@Logger注解
          if (call != null) {
              // 打印日志
              logger(joinPoint, call, end - start, result);
          }
      } catch (Throwable e) {
          long end = System.currentTimeMillis();
          // 打印日志
          error(joinPoint, call, end - start, e);
      }
      return result;
    }

    private void logger(ProceedingJoinPoint joinPoint, Call call, long elapsed, Object result) {
        Logger logger = getLogger(joinPoint, call);

        Object[] args = joinPoint.getArgs();

        // 调用成功
        if (isSuccess(result)) {
            // 超时，打印日志：默认1s超时
            if (elapsed > call.elapsed()) {
                logger.error(String.format("timeout@args:%s,elapsed:%d;duration:%d,result:%s", JsonHelper.toJson(args), call.elapsed(), elapsed, JsonHelper.toJson(result)));
            }
            // 符合采样频率条件：默认万分之一
            else if (random.nextInt(call.basic()) <= call.sample()) {
                logger.warn(String.format("sample@args:%s,elapsed:%d;duration:%d,result:%s", JsonHelper.toJson(args), call.elapsed(), elapsed, JsonHelper.toJson(result)));
            }
            // debug日志：线程ThreadLocal级debug
            else if (Context.debug()) {
                logger.warn(String.format("debug@args:%s,elapsed:%d;duration:%d,result:%s", JsonHelper.toJson(args), call.elapsed(), elapsed, JsonHelper.toJson(result)));
            }
            // 其他（未超时、未采用命中），不打印日志
            else {
                otherCase(joinPoint, call, elapsed, result);
            }
            return;
        }

        // 调用失败，访问日志
        logger.warn(String.format("access@args:%s,elapsed:%d;duration:%d,success:false,result:%s", JsonHelper.toJson(args), call.elapsed(), elapsed, JsonHelper.toJson(result)));
    }

    /**
    * 判断Result是否成功，允许子类覆盖
    * @param result 调用结果
    */
    protected boolean isSuccess(Object result) {
        if (result != null) {
            if (result instanceof ResultSupport) {
                return ((ResultSupport) result).isSuccess();
            }
            return true;
        }
        return false;
    }

    protected void otherCase(ProceedingJoinPoint joinPoint, Call call, long elapsed, Object result) {
    }

    private void error(ProceedingJoinPoint joinPoint, Call call, long elapsed, Throwable t) {
        Logger logger = getLogger(joinPoint, call);

        // 记录异常日志
        logger.error(String.format("exception@args:%s,elapsed:%d;duration:%d", JsonHelper.toJson(joinPoint.getArgs()), call.elapsed(), elapsed), t);
        // 重新抛出异常
        throw new RepositoryException(e);
    }

    private Logger getLogger(ProceedingJoinPoint joinPoint, Call call) {
        return LoggerFactory.getLogger(StringUtils.defaultIfBlank(call.value(), joinPoint.getClass().getName()));
    }
}
```

实例化CallAspect切面，可定义任意切面；并可扩展调用结果是否成功判断，扩展非超时、非采样、非调试的其他case扩展
```java
public class CallAspect extends CallAround {
      @Pointcut("execution(* com.company.department.business.appname.*.*(..)) && @annotation(org.springframework.ext.common.aspect.Call)")
      public void callPoint() {
      }

      protected boolean isSuccess(Object result) {
          return super.isSuccess();
      }

      protected void otherCase(ProceedingJoinPoint joinPoint, Call call, long elapsed, Object result) {
          super.otherCase();
      }
}
```
<<<<<<< HEAD

## 小结
自此，我们回过头来看下方法处理的特性有了哪些：
- 方法调用处理的case包括了异常捕捉、失败日志、成功超时日志、成功采样日志、调试日志，并支持case扩展
- 可以使用注解来声明方法调用的处理
- 支持切面扩展
- 支持判断返回结算是否成功，进而可以扩展成功和失败的处理方式
=======
>>>>>>> 2e5dd704434a0c113bc585ccbda4c346dc5a079a
