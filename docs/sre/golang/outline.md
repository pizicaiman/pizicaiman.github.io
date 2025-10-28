model类，定义user bookinfo等数据结构

config类，定义中间件的yaml文件，然后read yaml文件，生成server的配置信息

init类中创建数据库连接，创建redis连接，创建session，创建log 等 实例化
  加载驱动类，来初始化连接或者判断是否已经连接，如果有问题报错

router类，定义路由，然后调用controller类中的方法
  定义路由规则，统一不同分组或者业务逻辑，然后调用controller类中的方法

controller类，定义业务逻辑，调用service类中的方法

repository类，定义数据库操作，调用model类中的方法 dao类，定义数据库操作，调用model类中的方法

util类，定义工具类，如字符串处理，时间处理等

middleware类，定义中间件，如日志，权限验证等

exception类，定义异常类，如参数错误，权限错误等

service类，定义业务逻辑，调用model类中的方法

main函数中，调用router类中的方法，启动服务