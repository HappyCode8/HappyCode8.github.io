# Web开发

## 1. 异常处理

- 参数校验

  ```java
  @Data
  public class Account {
      @NotEmpty(message = "用户不允许为空")
      private String username;
  
      @NotEmpty(message = "用户姓名不允许为空")
      private String name;
  
      private String avatar;
  
      @Pattern(regexp = "1\\d{10}", message = "手机号格式不正确")
      private String telephone;
  
      @Email(message = "邮箱格式不正确")
      private String email;
  }
  
  @RestController
  @RequestMapping("/api/account")
  @Slf4j
  public class AccountController {
  
      @PostMapping("login")
      public Result<Account> test(@Valid @RequestBody Account account) {
          return Result.ok(account);
      }
  
  }
  ```

- 自定义校验

  ```java
  @Documented
  @Retention(RUNTIME)
  @Target({FIELD, METHOD, PARAMETER, TYPE})
  @Constraint(validatedBy = NotConflictAccount.NotConflictAccountValidator.class)
  public @interface NotConflictAccount {
      String message() default "用户名称不是123";
  
      Class<?>[] groups() default {};//校验分组
  
      Class<? extends Payload>[] payload() default {};
  
      class NotConflictAccountValidator implements ConstraintValidator<NotConflictAccount, Account> {
  
          @Override
          public boolean isValid(Account account, ConstraintValidatorContext constraintValidatorContext) {
              if (!account.getName().equals("123")) {
                  return false;
              }
              return true;
          }
      }
  }
  ```

- 业务代码校验

  ```java
  @RestController
  @RequestMapping("/api/exception")
  @Slf4j
  public class ExceptionController {
      @GetMapping("/meaningException")
      public void meaningfulException(){
          /**
           * 实际上会被WebExceptionHandler捕获处理
           * */
          throw new MeaningfulException(5001,"接收话术录音文件不合法");
      }
  
      @GetMapping("/Exception")
      public void exception() throws Exception {
          throw new Exception("dadsda");
      }
  }
  
  ```

- 建议：如果是简单参数校验直接使用参数校验，复杂且常用的使用自定义校验，更复杂的使用在业务代码中校验抛出异常然后全局统一捕获异常

## 2. 传参

- 前端传数字，后端直接转为枚举值，数据库入库时进行转换，取出时也进行转换，总结就是代码中用枚举，其余存储于给前端时都给数

  ```java
  public enum LanguageEnums{
      CHINESE(1,"中文"),
      ENGLISH(2, "英文");
  
      private Integer code;
      private String desc;
  
      LanguageEnums(Integer code, String desc) {
          this.code=code;
          this.desc=desc;
      }
  
      /**
       * 前端给的时候反序列化成枚举,注意的一点是如果不自己实现，spring也会默认转枚举，但是它是按照下标顺序取的
       * */
      @JsonCreator
      public static LanguageEnums getByCode(Integer code) {
          for (LanguageEnums value : LanguageEnums.values()) {
              if (code.equals(value.getCode())) {
                  return value;
              }
          }
          return null;
      }
  
      /**
      * 给前端时序列化成数字
      * */
      @JsonValue
      public Integer getCode() {
          return code;
      }
  
  }
  ```

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
      <typeHandlers>
          <typeHandler handler="com.wyj.handler.GenderEnumHandler" javaType="com.wyj.anno.GenderEnum" />
      </typeHandlers>
  </configuration>
  ```

  ```java
  public class GenderEnumHandler extends BaseTypeHandler<GenderEnum> {
  
      @Override
      public void setNonNullParameter(PreparedStatement preparedStatement, int i, GenderEnum genderEnum, JdbcType jdbcType) throws SQLException {
          preparedStatement.setInt(i, genderEnum.getCode());
      }
  
      @Override
      public GenderEnum getNullableResult(ResultSet resultSet, String s) throws SQLException {
          return GenderEnum.getGenderEnumByCode(resultSet.getInt(s));
      }
  
      @Override
      public GenderEnum getNullableResult(ResultSet resultSet, int i) throws SQLException {
          return GenderEnum.getGenderEnumByCode(resultSet.getInt(i));
      }
  
      @Override
      public GenderEnum getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
          return GenderEnum.getGenderEnumByCode(callableStatement.getInt(i));
      }
  }
  ```

  

