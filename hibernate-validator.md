## hibernate-validator



### 注解

* @NotNull：表示属性不能为空。
* @NotEmpty：表示属性不能为空字符串或长度为零的集合。
* @Size：表示属性的长度必须在指定的范围内。
* @Min：表示属性的值必须大于等于指定的最小值。
* @Max：表示属性的值必须小于等于指定的最大值。
* @DecimalMin：表示属性的值必须大于等于指定的最小小数值。
* @DecimalMax：表示属性的值必须小于等于指定的最大小数值。
* @Digits：表示属性的值必须符合指定的小数位数。
* @Pattern：表示属性的值必须符合指定的正则表达式模式。
* @Email：表示属性的值必须是一个有效的电子邮件地址。
* @Future：表示属性的值必须是未来的日期。
* @Past：表示属性的值必须是过去的日期。
* @FutureOrPresent：表示属性的值必须是未来的日期或现在的日期。
* @PastOrPresent：表示属性的值必须是过去的日期或现在的日期。
* @CreditCardNumber：表示属性的值必须是一个有效的信用卡号码。
* @IBAN：表示属性的值必须是一个有效的国际银行帐户号码（IBAN）。
* @URL：表示属性的值必须是一个有效的 URL。
* @File：表示属性的值必须是一个有效的文件路径。
* @Length：表示属性的长度必须在指定的范围内。
* @Range：表示属性的值必须在指定的范围内。
* @Valid：表示属性的值必须通过所有关联的验证注解的验证。
