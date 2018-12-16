# 解决办法：
添加`columnOverride`标签，将对应字段的`jdbcType`设置成`VARCHAR`后重新生成即可！
```xml
<table tableName="sys_feedback_info">
  <generatedKey column="fb_id" sqlStatement="MySql" identity="true"/>
  <columnOverride column="fb_content" jdbcType="VARCHAR"></columnOverride>
</table>
```