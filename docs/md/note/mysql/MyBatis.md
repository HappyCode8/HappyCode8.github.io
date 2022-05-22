# Mybatis

## 建表示例

```sql
CREATE TABLE `ai_sco_day_analyse` (
   `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '自增主键',
   `template_id` varchar(64) NOT NULL DEFAULT '' COMMENT '机器人id',
   `call_date` date NOT NULL DEFAULT '0000-00-00' COMMENT '日期',
   `template_version` varchar(16) NOT NULL DEFAULT '' COMMENT '机器人版本',
   `total_count` int(11) NOT NULL DEFAULT '0' COMMENT '通话总数',
   `through_count` int(11) NOT NULL DEFAULT '0' COMMENT '接通总数',
   `first_round_not_hangup_count` int(11) NOT NULL DEFAULT '0' COMMENT '首轮未挂断总数',
   `cooperate_count` int(11) NOT NULL DEFAULT '0' COMMENT '配合数',
   `success_count` int(11) NOT NULL DEFAULT '0' COMMENT '成功数',
   `see_through_count` int(11) NOT NULL DEFAULT '0' COMMENT '识破数',
   `nlu_distinguish_count` int(11) NOT NULL DEFAULT '0' COMMENT 'nlu实际识别的query数',
   `nlu_eff_count` int(11) NOT NULL DEFAULT '0' COMMENT 'nlu可识别query数',
   `antipathy_count` int(11) NOT NULL DEFAULT '0' COMMENT '反感数',
   `talking_time_len` int(11) NOT NULL DEFAULT '0' COMMENT '通话时长',
   `insert_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '本条记录创建时间',
   `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '本条记录修改时间',
   `is_visible` tinyint(1) NOT NULL DEFAULT '1',
   PRIMARY KEY (`id`),
   UNIQUE KEY `idx_call_date_tenant_id` (`call_date`,`template_id`,`template_version`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4

select bot_id,
       bot_name,
       case when call_count>=100000                      then '价值高'
            when call_count<100000 and call_count>=10000 then '价值中'
            else '价值低'
             end as bot_value,
       case when nlu_distinguish_count_detail/nlu_eff_count_detail>=0.9 and nlu_distinguish_count_mark/nlu_eff_count_mark>=0.9                          then 'B'
            when nlu_distinguish_count_detail/nlu_eff_count_detail>=0.9 and (nlu_distinguish_count_mark/nlu_eff_count_mark<0.9 or nlu_eff_count_mark=0) then 'C'
            else 'D'
             end as bot_rank
  from us.yoona_bot_level_stastics
 where create_date BETWEEN '$$begindate' and '$$enddate'
```

## 枚举值转换

项目结构

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import wyj.enums.BusinessValueType;
import wyj.mapper.TemplateMapper;

@RestController
public class Controller {
    private final TemplateMapper templateMapper;

    public Controller(TemplateMapper templateMapper) {
        this.templateMapper = templateMapper;
    }

    @GetMapping("hello")
    public Object hello(){
        return templateMapper.selectCalculateStatus("124");
    }

    @GetMapping("hello2")
    public int hello3(){
        return templateMapper.updateCalculateStatus("123", BusinessValueType.EFFECTIVECONFIG);
    }
}

```

```java
import lombok.Getter;

@Getter
public enum BusinessValueType {
    NOANYCONFIG(0),
    BUSINESSCONFIG(1),
    EFFECTIVECONFIG(2);

    private int code;

    public int getCode() {
        return code;
    }

    BusinessValueType(int code) {
        this.code=code;
    }

    public static BusinessValueType getBusinessValueTypeByCode(Integer code) {
        switch (code) {
            case 1:
                return BUSINESSCONFIG;
            case 2:
                return EFFECTIVECONFIG;
            default:
                return NOANYCONFIG;
        }
    }
}
```

```java
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;
import wyj.enums.BusinessValueType;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class BusinessValueHandler extends BaseTypeHandler<BusinessValueType> {
    @Override
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, BusinessValueType businessValueType, JdbcType jdbcType) throws SQLException {
        preparedStatement.setInt(i, businessValueType.getCode());
    }

    @Override
    public BusinessValueType getNullableResult(ResultSet resultSet, String s) throws SQLException {
        return BusinessValueType.getBusinessValueTypeByCode(resultSet.getInt(s));
    }

    @Override
    public BusinessValueType getNullableResult(ResultSet resultSet, int i) throws SQLException {
        return BusinessValueType.getBusinessValueTypeByCode(resultSet.getInt(i));
    }

    @Override
    public BusinessValueType getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        return BusinessValueType.getBusinessValueTypeByCode(callableStatement.getInt(i));
    }
}
```

```java
import org.apache.ibatis.annotations.Mapper;
import wyj.enums.BusinessValueType;

@Mapper
public interface TemplateMapper {
    int updateCalculateStatus(String template, BusinessValueType businessValueType);

    BusinessValueType selectCalculateStatus(String template);
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="wyj.mapper.TemplateMapper">

    <insert id="updateCalculateStatus">
        insert into template_business_value_type(template_id, business_type) values(
            #{template},
            #{businessValueType, typeHandler=wyj.handler.BusinessValueHandler}
        ) on duplicate key update business_type=(
            #{businessValueType, typeHandler=wyj.handler.BusinessValueHandler}
        )
    </insert>

    <select id="selectCalculateStatus" resultType="wyj.enums.BusinessValueType">
        select business_type
        from template_business_value_type
        where template_id=#{
            template
        }
    </select>
</mapper>
```

```yml
mybatis:
  mapper-locations: classpath*:mappers/*Mapper.xml
  config-location: classpath:mybatis-config.xml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      initial-size: 1
      min-idle: 5
      max-active: 20
      max-wait: 60000
      time-between-eviction-runs-millis: 60000
      min-evictable-idle-time-millis: 300000
      validation-query: SELECT 'x'
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      pool-prepared-statements: false
      max-open-prepared-statements: 20
      stat-view-servlet:
        enabled: true
      url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
      username: root
      password: 20131983
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeHandlers>
        <typeHandler handler="wyj.handler.BusinessValueHandler" javaType="wyj.enums.BusinessValueType" />
    </typeHandlers>
</configuration>
```



## 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--对应的接口mapper-->
<mapper namespace="com.meituan.ai.sco.pallas.dal.mapper.CityAnalyseMapper">
<!--对应的结果类-->
  <resultMap id="ResultMap" type="com.meituan.ai.sco.pallas.model.stastics.AggregateCityResult">
      <result column="city" jdbcType="VARCHAR" property="city" />
      <result column="call_count" jdbcType="INTEGER" property="callCount" />
      <result column="through_rate" jdbcType="FLOAT" property="throughRate" />
      <result column="first_round_not_hangup_rate" jdbcType="FLOAT" property="firstRoundNotHangupRate" />
      <result column="cooperate_rate" jdbcType="FLOAT" property="cooperateRate" />
      <result column="success_rate" jdbcType="FLOAT" property="successRate" />
      <result column="success_after_cooperate_rate" jdbcType="FLOAT" property="successAfterCooperateRate" />
      <result column="antipathy_rate" jdbcType="FLOAT" property="antipathyRate" />
      <result column="talking_time_len" jdbcType="FLOAT" property="talkingTimeLen" />
  </resultMap>
    
    <select id="selectAllByCallDateAndTemplate" resultMap="ResultMap">
        <include refid="cityData"/>
    </select>
    
    <select id="selectByCallDateAndTemplate" resultMap="ResultMap">
        <include refid="cityData"/>
        <if test="sortClause != null">
            order by ${sortClause}
        </if>
    </select>
    
    <sql id="cityData">
        select city,
        sum(total_count) as call_count,
        sum(through_count)/sum(total_count) as through_rate,
        sum(first_round_not_hangup_count)/sum(through_count) as first_round_not_hangup_rate,
        sum(cooperate_count)/sum(through_count) as cooperate_rate,
        sum(success_count)/sum(through_count) as success_rate,
        sum(success_count)/sum(cooperate_count) as success_after_cooperate_rate,
        sum(antipathy_count)/sum(through_count) as antipathy_rate,
        sum(talking_time_len)/1000 as talking_time_len
        from ai_sco_day_city_analyse
        <where>
            <trim prefixOverrides="and">
                <if test="startDate != null">
                    and call_date &gt;= #{startDate}
                </if>
                <if test="endDate != null">
                    and call_date &lt;= #{endDate}
                </if>
                <if test="templateId != null">
                    and template_id = #{templateId}
                </if>
                <if test="templateVersion != null">
                    and template_version = #{templateVersion}
                </if>
            </trim>
        </where>
        group by city
    </sql>

    <foreach collection="list" item="item" index="index" separator=",">
        (#{item.taskId},#{item.callDetail})
    </foreach>
</mapper>
```

id与代码接口中的函数名对应起来，parameterType是查询的参数，resultMap是查询的结果

``` xml
<select id="baseStatisticsAggregateSearch" parameterType="com.meituan.ai.sco.rhea.dal.criterion.BaseStatisticsCriteria"
            resultMap="BaseAggregate">
        select *
        from
        (
        select
        <trim suffixOverrides=","><!--trim用于去除或者拼接字符，suffixOverrides=”,“去除sql语句后面的逗号-->
            <if test="templateColumn"><!--if test用于测试是否满足某种条件，直接写相当于！=null-->
                template_value,
                template_name,
                template_type,
            </if>
        </trim>
        from
        ai_sco_base_call_statistics
        where
        <trim prefixOverrides="and"><!--去除最前边的and-->
            <if test="whereStartDate != null">
                and call_date >= #{whereStartDate}
            </if>
            <if test="whereTenantIds != null">
                and tenant_id in
                <!--foreach用于循环列表，将列表用逗号分隔前后加括号拼接,collection中的要么是参数名，要么用@Param注解好-->
                <foreach collection="whereTenantIds" open="(" close=")" separator="," item="listItem">
                    <!--#{listItem} 预编译阶段会生成?类似的，但是如果${groupByClause}，就会做纯替换-->
                </foreach>
            </if>
            and is_visible = 1
        </trim>
    </select>
```

## 分页

```java
public PageInfo<KnowledgeVO> selectByDateAndTemplate() {
        PageHelper.startPage(currentPage,pageSize);
        List<AggregateKnowledgeResult> res = knowledgeMapper.selectByDateAndTemplate();
        PageInfo pageAggregateKnowledgeResults = new PageInfo(res);
        final List<KnowledgeVO> knowledgeVOS = new ArrayList<>();
        res.forEach(data->{
            KnowledgeVO knowledgeVO=new KnowledgeVO();
            //VO匹配
            knowledgeVOS.add(knowledgeVO);
        });
  			//重设list
        pageAggregateKnowledgeResults.setList(knowledgeVOS);
        return pageAggregateKnowledgeResults;
    }
```

## 与$符的区别

```
（1）
　　1）#{} 为参数占位符 ?，即sql 预编译
　　2）${} 为字符串替换，即 sql 拼接
（2）
　　1）#{}：动态解析 -> 预编译 -> 执行
　　2）${}：动态解析 -> 编译 -> 执行
（3）
　　1）#{} 的变量替换是在DBMS 中
　　2）${} 的变量替换是在 DBMS 外
（4）
　　1）变量替换后，#{} 对应的变量自动加上单引号 ''
　　2）变量替换后，${} 对应的变量不会加上单引号 ''
（5）
　　1）#{} 能防止sql 注入
　　2）${} 不能防止sql 注入

#{} 和 ${} 的实例：假设传入参数为 1
（1）开始
　　1）#{}：select * from t_user where uid=#{uid}
　　2）${}：select * from t_user where uid= '${uid}'
（2）然后
	 1）#{}：select * from t_user where uid= ?
	 2）${}：select * from t_user where uid= '1'
（3）最后
　　1）#{}：select * from t_user where uid= '1'
　　2）${}：select * from t_user where uid= '1'

#{} 和 ${} 在使用中的技巧和建议
（1）不论是单个参数，还是多个参数，一律都建议使用注解@Param("")
（2）能用 #{} 的地方就用 #{}，不用或少用 ${}
（3）表名作参数时，必须用 ${}。如：select * from ${tableName}
（4）order by 时，必须用 ${}。如：select * from t_user order by ${columnName}
		 3,4因为预编译加了单引号，会导致不起作用
（5）使用 ${} 时，要注意何时加或不加单引号，即 ${} 和 '${}'
```

