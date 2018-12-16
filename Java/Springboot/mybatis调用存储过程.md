由于项目需求变更，数据库表需要根据机构进行分表，每次在新建/删除机构时，需要动态的新建/删除对应的机构分表，设计使用存储过程进行表的动态创建和删除。

## mybatis调用存储过程大致需要以下部分：
1. xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.trs.yq.dao.ProcedureDAO">
	<!-- 调用存储过程创建表 -->
	<select id="generateTable" parameterType="com.trs.yq.model.vo.ProcedureParam" useCache="false" statementType="CALLABLE">  
        call proc_generate_table(
	        #{id, mode=IN, jdbcType=INTEGER},
	        #{tFinalName, mode=IN, jdbcType=VARCHAR},
	        #{tSourceName, mode=IN, jdbcType=VARCHAR}
        )
    </select>
    
    <!-- 调用存储过程删除表 -->
	<select id="dropTable" parameterType="com.trs.yq.model.vo.ProcedureParam" useCache="false" statementType="CALLABLE">  
        call proc_drop_table(
	        #{id, mode=IN, jdbcType=INTEGER},
	        #{tFinalName, mode=IN, jdbcType=VARCHAR}
        )
    </select>
```

2. DAO文件
```java
public interface ProcedureDAO {
	/**
	 * 调用存储过程生成表
	 * @param param		对象的id, dbName, tFinalName, tSourceName四个属性作为参数
	 * @return:void
	 */
	void generateTable(ProcedureParam param);
	
	/**
	 * 调用存储过程删除表
	 * @param param		对象的id, dbName, tFinalName三个属性作为参数
	 * @return:void
	 */
	void dropTable(ProcedureParam param);
}
```

3. 参数对象
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class ProcedureParam {
	
	private int id;	//区分表的后缀id
	private String tFinalName;	//需要生成的表名
	private String tSourceName;	//源表名称
	
	public ProcedureParam(int id, String tFinalName) {
		super();
		this.id = id;
		this.tFinalName = tFinalName;
	}
	
}
```

4. 存储过程
```sql
-- 创建新表
CREATE DEFINER=`djg`@`%` PROCEDURE `proc_generate_table`(IN `id` int(11),IN `tFinalName` varchar(50),IN `tSourceName` varchar(50))
BEGIN
    -- 声明最终表名
    DECLARE t_name VARCHAR(100) DEFAULT '';
    -- 定义将要创建的表名
    SET t_name = CONCAT(tFinalName, '_', id);
    -- 建表语句拼接
    SET @sql_generate_table = CONCAT('CREATE TABLE IF NOT EXISTS ', t_name, ' LIKE `', tSourceName, '`');
    -- 预编译建表sql
    PREPARE sql_generate_table FROM @sql_generate_table;
    -- 执行建表sql
    EXECUTE sql_generate_table;
END

-- 删除分表
CREATE DEFINER=`root`@`localhost` PROCEDURE `proc_drop_table`(IN `id` int(11),IN `tFinalName` varchar(50))
BEGIN
	DECLARE t_name VARCHAR(50) DEFAULT '';
	SET t_name = concat(tFinalName, '_', id);
  SET @sql_drop_table = CONCAT('DROP TABLE IF EXISTS ', t_name);
	SELECT @sql_drop_table;
	PREPARE sql_drop_table FROM @sql_drop_table;
  EXECUTE sql_drop_table;
END
```