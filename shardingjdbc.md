数据库的扩展

	在平常的开发中，如果数据库单表数据特别多，那么查询起来的时候效率低下，单表查询压力会很大。

	关于数据库的扩展主要包括：业务拆分，主从复制，数据库分库分表

每一台机器无论配置多么多么好，总会有自身的物理上限，所以我们开始寻找别的方法提升系统性能，因此使用数据库的分库分表来提升系统性能。

数据库分库分表分为水平拆分与 垂直拆分

垂直拆分：如一个数据库中有用户表，订单表，XX表等等。将每张表放在不同的数据库中以达到分担数据库压力的效果，但是表压力并没有因此而减轻。

水平拆分：如用户表有20W数据，每5W就分一张表，分为四张表提供查询，就是水平拆分，每次查询都是从5W条数据中查询，比直接从20W数据中查询要快的多



shardingJDBC

既然决定了数据库需要进行分库，分表。就去找了一个已经开发好的开源项目，就是当当网的shardingJDBC。

shardingJDBC是轻量级的数据库分库分表中间件，使用jar包提供服务不需要额外的部署，可以理解为一个增强的jdbc驱动，可以与市面上的绝大多数orm框架结合。它可以与springboot结合使用，可以自定义分片规则，自身没有内置的分片算法，将分片场景抽象化，交由使用者自身去决定（实现某接口，再在application资源文件中指定文件路径）。



功能：分库分表、读写分离......（大多数分库分表工具都有）

shardingJDBC 的一些概念

数据源：数据库取名 （随意一个名称）

逻辑表：如 用户表 : user，源表

真实表：user_1,user_2,user_3,user_4...

数据节点：分片的最小单元由数据源和数据表组成如 

分片键：以什么标准来进行分片，如 以 user_id 来进行分片（分片依据）

分片算法：根据user_id 进行计算（取模等等），将这条数据分到某一个数据表中。

- 精确分片算法，用于处理单一分片键的 =  in 进行分片的场景
- 范围分片算法，用于处理单一分片键的bwteen and 进行分片的场景
- 复合分片算法，用于处理多分片键进行分片的场景
- hint分片算法，用于处理hint行分片的场景

分片策略：与分片算法结合使用，包含分片键和分片算法，真正可用于进行分片操作的是分片键+分片算法，也就是分片策略，目前提供5种策略

- 标准分片策略，提供对sql语句中的 = in 和 bwteen and 的分片支持。这个分片策略只支持单分片键。
- 复合分片策略，同上对= in beteen and 作分片支持，只不过这个分片策略支持多分片键，并且由于关系复杂，将分片键值组合以及操作符交给算法接口让开发者自行实现
- 行表达式分片策略，使用groovy的表达式，只支持单分片键，对于简单的分片算法可以通过简单的配置使用，避免繁琐的代码如：t_user${u_id % 8} 表示t_user表按照u_id按8取模分成8个表，表名称为t_user0到t_user7。 

分布式主键：

shardingjdbc 内置有分布式主键的生成，采用snowflake算法实现，生成的数据为64bit的长整型数据。 



使用方式

一：引入jar包依赖

    <dependency>
        <groupId>io.shardingsphere</groupId>
        <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
        <version>${sharding-sphere.version}</version>
    </dependency>

将原本的 spring-boot-starter依赖注释，因为这个依赖中已经引入了springboot-starter

二：配置shardingJDBC  (application.properties)

    sharding.jdbc.datasource.names=ds0  #数据源名称
    
    sharding.jdbc.datasource.ds0.type=org.apache.commons.dbcp2.BasicDataSource #数据库连接池类
    sharding.jdbc.datasource.ds0.driver-class-name=com.mysql.jdbc.Driver #数据库驱动
    sharding.jdbc.datasource.ds0.url=jdbc:mysql://localhost:3306/sharding_jdbc #数据库url
    sharding.jdbc.datasource.ds0.username=root #用户名
    sharding.jdbc.datasource.ds0.password=root #密码
    
    sharding.jdbc.config.sharding.tables.student<逻辑表名称>.actual-data-nodes=ds0.student_2018_$->{0..4} #真实表名称
    sharding.jdbc.config.sharding.tables.student<逻辑表名称>.table-strategy.standard.precise-algorithm-class-name=com.shardingjdbc.demo.shardingAlgorithm.StandardShardingStrategy  #某个实现了分片算法的类
    sharding.jdbc.config.sharding.tables.student<逻辑表名称>.table-strategy.standard.sharding-column=student_id #分片键
    sharding.jdbc.config.sharding.tables.student<逻辑表名称>.key-generator-column-name=id #分布式主键

实现了标准分片算法PreciseShardingAlgorithm 、复合分片算法接口.......，的类路径

以上全部配置完成后，可以配合各种orm框架进行使用如：

    public void addUser() throws ParseException {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String da = "2018-8-3 14:28:12";
        Date parse = simpleDateFormat.parse(da);
        Student student = new Student();
        student.setName("ggggr");
        student.setDate(new Date());
        Specification<Student> querySpecifi = (root, criteriaQuery, cb) -> {
            // and到一起的话所有条件就是且关系，or就是或关系
            return cb.equal(root.get("studentId"),231469524812038144l);
        };
        List all = studentRepository.findAll(querySpecifi);
    
    
        List<Student> all1 = studentRepository.findAll();
        Student one = studentRepository.findOne(231469524812038144L);
    }

以上代码完成了通过springdatajpa 构造查询条件进行查询的功能，目前我配置了标准分片算法，该算法对=或者in 操作符作操作，所以在构造 equal 条件时会走我自定义的逻辑，如果是查询条件没有进行分片配置那就执行全路由
