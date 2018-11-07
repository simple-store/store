### Idea新建Spring boot项目

> 最近有一个需求写Java后台，所有就重新拾回我那一丁点的Java基础

- 打开Idea新建项目

  > 选择Spring Initializr，jdk选择1.8

![image-20180627101150660](/Users/wuyunhe/Desktop/image-20180627101150660.png)

点击下一步

- 编写包名以及编写者

  > Artifact: 编写者、group:包名

![image-20180627101332108](/var/folders/08/f1jkxhcn0gsbdb58551kzws80000gn/T/abnerworks.Typora/image-20180627101332108.png)



- 选择需要的第三方

  > 这里采用了mybatis + mysql，web处勾选web

![image-20180627101723895](/var/folders/08/f1jkxhcn0gsbdb58551kzws80000gn/T/abnerworks.Typora/image-20180627101723895.png)

- 输入项目名称

![image-20180627103113641](/var/folders/08/f1jkxhcn0gsbdb58551kzws80000gn/T/abnerworks.Typora/image-20180627103113641.png)

到这里你就成功创建了项目了

- 导入相关依赖在pom.xml里操作
  - 添加resources(在build里面)

```xml
	<resources>
			<resource>
				<directory>src/main/java</directory>
				<includes>
					<include>**/*.xml</include>
				</includes>
				<filtering>true</filtering>
			</resource>
		</resources>
```

- 将resources下的application.prop…删除，添加application.yaml文件

  > 星号部分需替换

  ```yaml
  
  spring:
    datasource:
      url: jdbc:mysql://localhost:3306/*yourdatabase*?useSSL=false
      username: *yourusername*
      password: *yourpassword*
      driver-class-name: com.mysql.jdbc.Driver
  mybatis:
    mapperLocations: classpath:mapper/*Mapper.xml
  server:
    port: 9008
    
  ```

- 在main下创建entity/controller/services/mapper四个文件夹

  - entity: 和数据库表一一对应字段
  - controller: 暴露给外部的api接口
  - services: 对数据进行一些处理的地方
  - mapper: 对应mybatis mapper下的xml文件

- 在resources文件夹下添加mapper文件夹

  > 这里是mabatis的一些对数据库表映射的xml

  - a.xml

    > 需要注意的是，namespace和每个单独的inserct或者select的parameterType/resultType要和类名一一对应
    >
    > ID要和mapper下的类方法名一一对应

    ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.market.frank.mapper.AdminMapper">
      <!--<insert id="insertSelective" parameterType="com.market.frank.entity.Person">-->
          <!--INSERT INTO person-->
          <!--<trim prefix="(" suffix=")" suffixOverrides=",">-->
              <!--<if test="age != null">-->
                  <!--age,-->
              <!--</if>-->
              <!--<if test="name != null and name != ''">-->
                  <!--name,-->
              <!--</if>-->
          <!--</trim>-->
          <!--<trim prefix="VALUES (" suffix=")" suffixOverrides=",">-->
              <!--<if test="age != null">-->
                  <!--#{age},-->
              <!--</if>-->
              <!--<if test="name != null and name != ''">-->
                  <!--#{name},-->
              <!--</if>-->
          <!--</trim>-->
      <!--</insert>-->
      <insert id="insertNormal"  parameterType="com.market.frank.entity.Admin" >
          INSERT INTO admin
          <trim prefix="(" suffix=")" suffixOverrides=",">
          phone,password,
          <if test="name != null">
          name,
          </if>
          <if test="state != null">
          state,
          </if>
          </trim>
          <trim prefix="VALUES (" suffix=")" suffixOverrides=",">
          #{phone},#{password},
          <if test="name != null">
          #{name},
          </if>
          <if test="state != null">
          #{state}
          </if>
          </trim>
      </insert>
  
      <select id="isExist" resultType="com.market.frank.entity.Admin" parameterType="com.market.frank.entity.Admin">
          SELECT *
          FROM admin
          <trim prefix="WHERE">
              <if test="phone != null">
                 phone = #{phone}
              </if>
              <if test="password != null and password.length()>0">
                AND password = #{password}
              </if>
          </trim>
      </select>
      <update id="update" parameterType="com.market.frank.entity.Admin">
          UPDATE admin
          <trim prefix="SET" suffixOverrides=",">
            <if test="phone != null and phone.length() == 11">
              phone = #{phone}
            </if>
            <if test="password != null and password.length() >0">
                password = #{password}
            </if>
            <if test="name != null and name.length() > 0">
                name = #{name}
            </if>
  
          </trim>
  
      </update>
      <select id="selectAll" resultType="com.market.frank.entity.Admin">
          SELECT *
          FROM admin
          WHERE state != 404
      </select>
  
     <!--<select id="selectAllNonM" resultType="com.market.frank.entity.Admin">-->
         <!---->
     <!--</select>-->
  </mapper>
    ```

  

- 编写mapper下的文件

  > 其实就是和resources下的xml一一对应

  ```java
  
  
  import java.util.List;
  
  public interface AdminMapper {
      public List<Admin> selectAll();
      public Admin isExist(Admin admin);
      public int insertNormal(Admin admin);
      public int updateNormal(Admin admin);
  }
  
  ```

  

- 编写entity的实体类

  > 字段必须和数据库里的表字段一一对应

      ```java
public class Admin {

    private long id;
    private String account;
    private String phone;
    private String password;
    

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getAccount() {
        return account;
    }

    public void setAccount(String account) {
        this.account = account;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
      ```

- 编写service下的文件

  - adminService(service/AdminService)

  ```java
  import java.util.List;
  
  public interface AdminService {
      public Admin login(Admin admin);
      public int insertNormal(Admin admin);
      public int updateNormal(Admin admin);
      public List<Admin> selectAll();
  }
  
  ```

  - AdminServiceImpl(service/impl/AdminServiceImpl)

    ```java
    import org.springframework.stereotype.Service;
    
    import javax.annotation.Resource;
    import java.util.List;
    
    @Service
    public class AdminServiceImpl implements AdminService {
        @Resource
        private AdminMapper adminMapper;
    
        @Override
        public Admin login(Admin admin) {
            return adminMapper.isExist(admin);
        }
    
        @Override
        public int insertNormal(Admin admin) {
            return adminMapper.insertNormal(admin);
        }
    
        @Override
        public int updateNormal(Admin admin) {
            return adminMapper.updateNormal(admin);
        }
    
        @Override
        public List<Admin> selectAll() {
            return adminMapper.selectAll();
        }
    }
    
    ```

  

  

   编写controller下的文件

  - AdminController

   ```java
  
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RequestMethod;
  import org.springframework.web.bind.annotation.ResponseBody;
  import org.springframework.web.bind.annotation.RestController;
  
  import javax.annotation.Resource;
  import java.util.List;
  
  @RestController
  @RequestMapping("/api/admin/")
  public class AdminController {
  
      @Resource
      private AdminService adminService;
  
      /*
      * @function
      * @api
      * @params null
      * @return admin list
      * */
      @RequestMapping(value = "all", method = RequestMethod.GET)
      public @ResponseBody Object onAll() {
          List list = adminService.selectAll();
          return list;
      }
  }
  
   ```

  

  #### 接下来就可以点击右上角的绿色小三角运行项目拉。

  输入localhost:9008/api/admin/all你就可以得到数据库里的数据啦

  到这里你就已经成功运行起来了一个mybatis + spring boot + mysql集成的Java项目

     

