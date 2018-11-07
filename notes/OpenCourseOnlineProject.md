### OpenCourseOnlineProject

- 数据库

  > 编号使用默认自增健（行为id）

  - **user (p0)**

    - user_name（p0）： *用户的昵称*

      

    - user_password（p0）：*用户密码*（存储md5加密后的base64字符）

      

    - user_phone（p0）： *用户的手机号*

    

    - user_balance（p0）: *用户的余额*

    

    - user_status（p0）:  *用户的状态*（删除为404,正常为1, etc）

    

    - user_bank_card_no（p1）: *用户银行卡账号*（消费时需绑定）

    

    - user_id_card_no（p1）: *用户身份证号*（消费时需绑定）

    

    - user_gmt_create_time: *用户创建时间<格林威治>*('20180607')

    

    - user_info: (region、gender、age、interest_subject_ids、etc)

    

    - extra_info: *额外的补充信息*（保留字段）

    

  - **manager (p0)**（教师/课程管理者）

    - manager_name： *管理者的昵称*

      

    - manager_password：*管理者密码*（存储md5加密后的base64字符）

      

    - manager_phone： *管理者的手机号*

    

    - manager_card_no（p1）: *管理者银行卡账号*（提现时需绑定）

    

    - manager_id_no（p1）: *管理者身份证号*（提现时需绑定）

    

    - manager_gmt_create_time:  *管理者创建时间<格林威治>*('20180607')

    

    - manager_profession_no（p1）: 管理者教师资格证（上传视频时需绑定）

    

    - manager_status:  *管理者的状态*（删除为404,正常为1, etc）

    

    - manager_info: (region、gender、age、interest_subject_ids、etc)

      

    - manager_balance:  *管理者的余额*

    

    - extra_info: *额外的补充信息*（保留字段）

  - **admin（p0）**（后台管理员/后台人员）

    - admin_account:  后台人员账号
    - admin_password: 后台人员密码（存储md5加密后的base64字符）
    - admin_phone：后台人员的手机（紧急通知发短信或拨号）
    - admin_gmt_create_time:  *后台人员创建时间<格林威治>*('20180607')

    - admin_employee_no: 后台人员工号
    - admin_info: (region、gender、age、interest_subject_ids、etc)

    - extra_info: *额外的补充信息*（保留字段）

  - **course (p0)**

    - course_name: *课程名*(具有唯一性)
    - course_description: 课程描叙
    - course_tag: 课程标签
    - course_upload_manager_id: 课程上传及维护者编号
    - course_gmt_create_time:  *课程创建时间<格林威治>*('20180607')
    - course_subject_id: *课程科目编号*
    - extra_info: *额外的补充信息*（保留字段）

    

    - **subject(p0)**
      - subject_name: 课程科目名（具有唯一性）
      - extra_info: *额外的补充信息*（保留字段）

  - **order (p0)**

    - order_amount: *订单总额*（分为单位例如，100元为10,000）
    - order_gmt_create_time:  *订单创建时间<格林威治>*('20180607')
    - order_timestamp_create_time: 订单创建时间<时间戳>(15012303)
    - order_cosumer_id: 消费者编号（与user_id相关）
    - order_eraner_id: 收益者编号(与manger_id相关)
    - order_course_id: 课程编号
    - order_status: 订单状态（删除为404,正常为1，etc）
    - order_progress: 订单进行状态（paid:200, not_paid:100, error: 500, canceld: 304, etc）
    - extra_info: *额外的补充信息*（保留字段）

  - **comment (p1)**

    - comment_gmt_create_time:  *评论创建时间<格林威治>*('20180607')
    - comment_commenter_id: 评论者者编号（与user_id相关）
    - comment_course_id: 评论课程
    - comment_content: 评价内容
    - comment_owner_id: 被评论者编号(与manger_id相关)
    - comment_status: 评论状态（删除为404,正常为1，etc）
    - comment_star_level: 评论星级（1-5, 5 is best)
    - extra_info: *额外的补充信息*（保留字段）

  - province(p3)

    - province_name: 省份名称

  - city(p3)

    - city_name: 城市名称
    - province_id: 省份编号

  - pemission(权限表)

    