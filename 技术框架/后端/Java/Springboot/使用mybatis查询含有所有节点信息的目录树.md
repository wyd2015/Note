## 利用mybatis查询含有所有节点信息的目录树

### 1、xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.***.dao.SysDepartmentCustomDAO">
<resultMap id="BaseResultMap" type="com.***.model.vo.organ.VSysOrgan">
	    <id column="organ_id" jdbcType="INTEGER" property="organId" />
	    <result column="organ_name" jdbcType="VARCHAR" property="organName" />
	    <result column="parent_id" jdbcType="INTEGER" property="parentId" />
	    <result column="dep_level" jdbcType="TINYINT" property="depLevel" />
	    <result column="dep_code" jdbcType="VARCHAR" property="depCode" />
	    <result column="create_time" jdbcType="TIMESTAMP" property="createTime" />
	    <result column="update_time" jdbcType="TIMESTAMP" property="updateTime" />
	    <result column="user_num" jdbcType="INTEGER" property="userNum" />
	    <result column="operate_user_id" jdbcType="INTEGER" property="operateUserId" />
	    
        <!-- 此处column为主键字段，select对应的键值是根据parent_id去查节点信息对应的select的id（方法名），此两项不可缺 -->
	    <collection property="children" ofType="com.trs.yq.model.po.generator.SysOrgan" 
	    column="organ_id" select="selectOrgList"></collection>

	  </resultMap>
	  
	<!-- 超管查询所有机构的部门信息（包含各机构），此方法是基础，不可少 -->
	<select id="selectOrgList" resultMap="BaseResultMap">
		select * from sys_organ where parent_id = #{organId}
	</select>
	
	<!-- 机构管理员查询本机构下的部门目录（包含机构） -->
	<select id="selectDepList" parameterType="int" resultMap="BaseResultMap">
		select * from sys_organ where organ_id = #{organId}
	</select>
</mapper>
```

### 2、接收对象
```java
@Data
@NoArgsConstructor
@JsonIgnoreProperties(value={"handler"})//此注解不可少
public class VSysOrgan implements Serializable{

    private Integer organId;
    private String organName;
    private Integer parentId;
    private Integer depLevel;
    private String depCode;
    private Date createTime;
    private Date updateTime;
    private Integer userNum;
    private Integer operateUserId;

    private List<SysOrgan> children;    //用于接收该节点的所有子节点

}
```

### 3、DAO
```java
public interface SysDepartmentCustomDAO {
	/*
	 * 查询所有机构下部门信息
	 * 此处的机构表使用sys_organ表
	 */
	List<VSysOrgan> selectOrgList(int organId);
	
	/*
	 * 查询指定机构下部门信息(包含当前机构信息)
	 * 此处的机构表使用sys_organ表
	 */
	List<VSysOrgan> selectDepList(int organId);
	
}
```