# MySQL的JSON类型操作
    
    mysql自5.7.8版本开始，支持了json结构的数据存储和查询

## 参考文档：  https://www.jianshu.com/p/6a9ca839c5b5

## 建表中字段为json中的值 GENERATED ALWAYS
    CREATE TABLE `ccc_halls` (
      `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
      `cinema_code` bigint(20) unsigned DEFAULT NULL,
      `name` varchar(50) NOT NULL COMMENT '影厅名称',
      `seats` int(11) DEFAULT NULL COMMENT '座位数',
      `hall_attr` json DEFAULT NULL COMMENT '{''effect_4d'':1,''effect_screen'':2,''frame_type'':3,''image_type'':4,''sound_type'':5}',
      `server` json DEFAULT NULL COMMENT '{''video_1'': {''manufacture'':'''', ''model'':'''', ''sn'':'''', ''cert'':[namespace,cert_sn]},''video_2'': {...},''sound_1'': {...}}',
      `effect_4d` int(11) GENERATED ALWAYS AS (json_extract(`hall_attr`,'$.effect_4d')) VIRTUAL COMMENT '4D特效',
      `effect_screen` int(11) GENERATED ALWAYS AS (json_extract(`hall_attr`,'$.effect_screen')) VIRTUAL COMMENT '银幕特效',
      `frame_type` int(11) GENERATED ALWAYS AS (json_extract(`hall_attr`,'$.frame_type')) VIRTUAL COMMENT '画面制式 ',
      `image_type` int(11) GENERATED ALWAYS AS (json_extract(`hall_attr`,'$.image_type')) VIRTUAL COMMENT '图像制式',
      `sound_type` int(11) GENERATED ALWAYS AS (json_extract(`hall_attr`,'$.sound_type')) VIRTUAL COMMENT '声音制式',
      `server_sn` json GENERATED ALWAYS AS (json_extract(`server`,'$.*.sn')) VIRTUAL,
      `cert_sn` json GENERATED ALWAYS AS (json_extract(`server`,'$.*.cert[1]')) VIRTUAL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB  DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='影厅表'
    
## 示例数据 
   
    mysql> select id, hall_attr, cert_sn from ccc_halls_test  where  id = 80880;
    +-------+----------------------------------------------------------+----------------------+
    | id    | hall_attr                                                | cert_sn              |
    +-------+----------------------------------------------------------+----------------------+
    | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |
    +-------+----------------------------------------------------------+----------------------+

## 插如JSON数据

    json_object 创建json对象
    
    json_array 创建json数组
    
    1、插入 json 格式的字符串，可以是对象的形式，也可以是数组的形式;
        INSERT INTO ccc_halls_test (hall_attr, cert_sn) values ('{"frame_type": 2, "image_type": 2, 
        "resolution_type": 2}', '["A63784", "A63984"]')
    
    2、可以使用JSON_OBJECT、JSON_ARRAY函数生成
        INSERT INTO ccc_halls_test (hall_attr, cert_sn) values (jJSON_OBJECT("frame_type", 2, "image_type", 2
        , "resolution_type", 2),JSON_ARRAY("A63784", "A63984"))
    
## 查询JSON
    
    json_extract        提取json值  如果查询没有的key,那么是可以查询,不过返回的是NULL
    
    column->path        json_extract的简洁写法，MySQL 5.7.9开始支持
                        其中对象类型path的表示方式 $.path，数组类型的表示方式 $[index]
                        
    json_unquote            去除json字符串的引号，将值转成string类型
    
    CAST                    将字符串转成JSON的形式
                        
    json_contains           判断是否包含某个json值
    
    json_contains_path      判断某个路径下是否包json值
    
### 取值 column->path
    
    mysql> select id, hall_attr -> '$.frame_type' as frame_type, cert_sn -> '$[0]' as cert1 from ccc_halls_test where id = 80880;
    +-------+------------+----------+
    | id    | frame_type | cert1    |
    +-------+------------+----------+
    | 80880 | 2          | "A63784" |
    +-------+------------+----------+
    
    取不存在的属性时返回NULL
    
    mysql> select id, hall_attr -> '$.frame_type_1' as frame_type, cert_sn -> '$[2]' as cert_2 from ccc_halls_test where id = 80880;
    +-------+------------+--------+
    | id    | frame_type | cert_2 |
    +-------+------------+--------+
    | 80880 | NULL       | NULL   |
    +-------+------------+--------+


### json_unquote
    
    查询结果中字符串类型还包含有双引号，可以使用JSON_UNQUOTE函数将双引号去掉，从MySQL 5.7.13开始也可以使用操作符 ->>
    
    mysql>  select id,  cert_sn -> '$[0]' as cert1,JSON_UNQUOTE(cert_sn -> '$[0]') as cert_un, cert_sn ->> '$[0]' 
    as cert_str from ccc_halls_test where id = 80880;
    +-------+----------+---------+----------+
    | id    | cert1    | cert_un | cert_str |
    +-------+----------+---------+----------+
    | 80880 | "A63784" | A63784  | A63784   |
    +-------+----------+---------+----------+
 
### CAST  JSON 作为条件搜索
    
    1、JSON不同于字符串，如果直接和JSON字段比较，不会查询到结果
        mysql>  select  id, hall_attr, cert_sn from  ccc_halls_test where cert_sn = '["A63784", "A63984"]' and  id = 808880;
        Empty set (0.00 sec)
        
    2. 使用CAST将字符串转成JSON的形式：
        mysql>  select  id, hall_attr, cert_sn from  ccc_halls_test where cert_sn = cast('["A63784", "A63984"]' as JSON);
        +-------+----------------------------------------------------------+----------------------+
        | id    | hall_attr                                                | cert_sn              |
        +-------+----------------------------------------------------------+----------------------+
        | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |
        +-------+----------------------------------------------------------+----------------------+
    
    mysql>  select  id, hall_attr, cert_sn from  ccc_halls_test where cert_sn -> '$[0]' = "A63784";
    +-------+----------------------------------------------------------+----------------------+
    | id    | hall_attr                                                | cert_sn              |
    +-------+----------------------------------------------------------+----------------------+
    | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |
    +-------+----------------------------------------------------------+----------------------+
    
### -> 和 ->> 的区别：
    
    JSON中的元素搜索是严格区分变量类型的，比如整型和字符串时严格区分的（->的形式），但是：->>类型是不区分的：
    
    -> 搜索：
        mysql>  select  id, hall_attr, cert_sn from  ccc_halls_test where hall_attr -> '$.frame_type' = 2 and id = 80880;
        +-------+----------------------------------------------------------+----------------------+
        | id    | hall_attr                                                | cert_sn              |
        +-------+----------------------------------------------------------+----------------------+
        | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |
        +-------+----------------------------------------------------------+----------------------+
        
        mysql>  select  id, hall_attr, cert_sn from  ccc_halls_test where hall_attr -> '$.frame_type' = '2' and id = 808880;
        Empty set (0.00 sec)
    
    ->> 搜索：
        mysql>  select  id, hall_attr, cert_sn from  ccc_halls_test where hall_attr ->> '$.frame_type' = 2 and id = 808880;
        +-------+----------------------------------------------------------+----------------------+
        | id    | hall_attr                                                | cert_sn              |
        +-------+----------------------------------------------------------+----------------------+
        | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |
        +-------+----------------------------------------------------------+----------------------+
        
        mysql>  select  id, hall_attr, cert_sn from  ccc_halls_test where hall_attr ->> '$.frame_type' = '2' and id = 800880;
        +-------+----------------------------------------------------------+----------------------+
        | id    | hall_attr                                                | cert_sn              |
        +-------+----------------------------------------------------------+----------------------+
        | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |
        +-------+----------------------------------------------------------+----------------------+

### json_contains  返回值为 1 和 0 
    
    vlaue值判定, 可用于数量统计
    
    该函数的第二个参数不接受整型，无论JSON元素时整型还是字符串(第二个参数只能是字符)
    用单引号包着 包含时值为1 不包含时值为0:
    
    Array:
        
        mysql>  select  id, hall_attr, cert_sn, JSON_CONTAINS(cert_sn,'"A63784"', '$[0]' ) as has_cert from  ccc_halls_test  where id = 80880;
        +-------+----------------------------------------------------------+----------------------+----------+
        | id    | hall_attr                                                | cert_sn              | has_cert |
        +-------+----------------------------------------------------------+----------------------+----------+
        | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |        1 |
        +-------+----------------------------------------------------------+----------------------+----------+
        
        mysql>  select  id, hall_attr, cert_sn, JSON_CONTAINS(cert_sn,'"A63784"') as has_cert from  ccc_halls_test  where id = 80880;
        +-------+----------------------------------------------------------+----------------------+----------+
        | id    | hall_attr                                                | cert_sn              | has_cert |
        +-------+----------------------------------------------------------+----------------------+----------+
        | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |        1 |
        +-------+----------------------------------------------------------+----------------------+----------+
    
        
        mysql>  select  id, hall_attr, cert_sn, JSON_CONTAINS(cert_sn,'A63784', '$[0]' ) as has_cert from  ccc_halls_test  where id = 80880;
        ERROR 3141 (22032): Invalid JSON text in argument 2 to function json_contains: "Invalid value." at position 0
        
        mysql>  select  id, hall_attr, cert_sn, JSON_CONTAINS(cert_sn,'"A63789"', '$[0]' ) as has_cert from  ccc_halls_ttest  where id = 80880,
        +-------+----------------------------------------------------------+----------------------+----------+
        | id    | hall_attr                                                | cert_sn              | has_cert |
        +-------+----------------------------------------------------------+----------------------+----------+
        | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |        0 |
        +-------+----------------------------------------------------------+----------------------+----------+

    Object:
        mysql>  select  id, hall_attr, cert_sn, JSON_CONTAINS(hall_attr, '{"image_type":2}') as has_image from  ccc_hallss_test  where id = 80880;
        +-------+----------------------------------------------------------+----------------------+----------+
        | id    | hall_attr                                                | cert_sn              | has_image |
        +-------+----------------------------------------------------------+----------------------+----------+
        | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |        1 |
        +-------+----------------------------------------------------------+----------------------+----------+
        
        mysql>  select  id, hall_attr, cert_sn, JSON_CONTAINS(hall_attr,'2','$.image_type') as has_cert from  ccc_halls_test  where id = 80880;
        +-------+----------------------------------------------------------+----------------------+----------+
        | id    | hall_attr                                                | cert_sn              | has_cert |
        +-------+----------------------------------------------------------+----------------------+----------+
        | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |        1 |
        +-------+----------------------------------------------------------+----------------------+----------+
        
        mysql>  select  id, hall_attr, cert_sn, JSON_CONTAINS(hall_attr,'2','$.image_type1') as has_cert from  ccc_hallss_test  where id = 80880;
        +-------+----------------------------------------------------------+----------------------+----------+
        | id    | hall_attr                                                | cert_sn              | has_cert |
        +-------+----------------------------------------------------------+----------------------+----------+
        | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |     NULL |
        +-------+----------------------------------------------------------+----------------------+----------+
        
    Python中的查询：
        sql = """
            select  id, hall_attr, cert_sn, JSON_CONTAINS(cert_sn, '%s','$[0]' ) as has_cert from  ccc_halls_test  where id = 80880
        """ % json.dumps("A63784")

### JSON_CONTAINS_PATH
    
    判断object的Key值是否存在, 可用于数量统计
    
    JSON_CONTAINS_PATH(json_doc, one_or_all, path[, path] ...)
    
    one：只要存在一个
        mysql>  select  id, hall_attr, cert_sn, JSON_CONTAINS_PATH(hall_attr,'one','$.image_type1','$.image_type') as has_image from  ccc_halls_test  where id = 80880;
        +-------+----------------------------------------------------------+----------------------+----------+
        | id    | hall_attr                                                | cert_sn              | has_image |
        +-------+----------------------------------------------------------+----------------------+----------+
        | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |        1 |
        +-------+----------------------------------------------------------+----------------------+----------+
    
    all：必须全部存在
    
        mysql>  select  id, hall_attr, cert_sn, JSON_CONTAINS_PATH(hall_attr,'all','$.image_type1','$.image_type') as has_image from  ccc_halls_test  where id = 80880;
        +-------+----------------------------------------------------------+----------------------+----------+
        | id    | hall_attr                                                | cert_sn              | has_image |
        +-------+----------------------------------------------------------+----------------------+----------+
        | 80880 | {"frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |        0 |
        +-------+----------------------------------------------------------+----------------------+----------+
    
    统计具有某些属性的数量
        select
                sum(JSON_CONTAINS_PATH(sp_effects, 'all', '$.CINITY')) as CINITY,
                sum(JSON_CONTAINS_PATH(sp_effects, 'all', '$.DMAX')) as DMAX,
                sum(JSON_CONTAINS_PATH(sp_effects, 'all', '$.IMAX')) as IMAX,
                sum(JSON_CONTAINS_PATH(sp_effects, 'all', '$.DOLBY_VISION')) as DOLBY_VISION
            from ccc_halls_test where sp_effects is not null;

        +--------+------+------+--------------+
        | CINITY | DMAX | IMAX | DOLBY_VISION |
        +--------+------+------+--------------+
        |      2 |    0 |    0 |            0 |
        +--------+------+------+--------------+

## 更新JSON Object

### JSON_INSERT()函数
    
    插入新值，但不会覆盖已存在的值
    
    mysql> update ccc_halls_test set hall_attr = JSON_INSERT(hall_attr, '$.effect_4d', 1, '$.frame_type', 3) WHERE  id = 80880;
    
    mysql> select id, hall_attr, cert_sn from ccc_halls_test  where  id = 80880;                                   
    +-------+--------------------------------------------------------------------------+----------------------+
    | id    | hall_attr                                                                | cert_sn              |
    +-------+--------------------------------------------------------------------------+----------------------+
    | 80880 | {"effect_4d": 1, "frame_type": 2, "image_type": 2, "resolution_type": 2} | ["A63784", "A63984"] |
    +-------+--------------------------------------------------------------------------+----------------------+

### JSON_SET()函数
    
    插入新值，并覆盖已存在的值
    
    mysql> update ccc_halls_test set hall_attr = JSON_SET(hall_attr, '$.sound_type', 1, '$.frame_type', 3) WHERE  idd = 80880;
    
    mysql> select id, hall_attr  from ccc_halls_test  where  id = 80880;
    +-------+-------------------------------------------------------------------------------------------+
    | id    | hall_attr                                                                                 |
    +-------+-------------------------------------------------------------------------------------------+
    | 80880 | {"effect_4d": 1, "frame_type": 3, "image_type": 2, "sound_type": 1, "resolution_type": 2} |
    +-------+-------------------------------------------------------------------------------------------+
    
### JSON_REPLACE()函数
    
    只替换已存在的值
    
    mysql> update ccc_halls_test set hall_attr = JSON_REPLACE(hall_attr, '$.sound_type', 2, '$.hall_type', 3) WHERE   id = 80880;
    
    mysql> select id, hall_attr  from ccc_halls_test  where  id = 80880;                                           
    +-------+-------------------------------------------------------------------------------------------+
    | id    | hall_attr                                                                                 |
    +-------+-------------------------------------------------------------------------------------------+
    | 80880 | {"effect_4d": 1, "frame_type": 3, "image_type": 2, "sound_type": 2, "resolution_type": 2} |
    +-------+-------------------------------------------------------------------------------------------+

### JSON_REMOVE()函数
    
    删除JSON元素
    
    mysql> update ccc_halls_test set hall_attr = JSON_REMOVE(hall_attr, '$.sound_type', '$.effect_4d') WHERE  id = 80880;
    
    mysql> select id, hall_attr  from ccc_halls_test  where  id = 80880;                                           
    +-------+----------------------------------------------------------+
    | id    | hall_attr                                                |
    +-------+----------------------------------------------------------+
    | 80880 | {"frame_type": 3, "image_type": 2, "resolution_type": 2} |
    +-------+----------------------------------------------------------+

## 更新JSON Array

### JSON_ARRAY_APPEND()

    数组中添加元素
    
    mysql> update ccc_halls_test set cert_sn = JSON_ARRAY_APPEND(cert_sn, '$[0]', 'A63785') WHERE  id = 80880;
    
    mysql> select id, cert_sn  from ccc_halls_test  where  id = 80880;
    +-------+---------------------------------------+
    | id    | cert_sn                               |
    +-------+---------------------------------------+
    | 80880 | [["A63784","A63785"], "A63984"]       |
    +-------+---------------------------------------+


### JSON_ARRAY_INSERT()

    mysql> update ccc_halls_test set cert_sn = JSON_ARRAY_INSERT(cert_sn, '$[0]', 'A6000') WHERE  id = 80880;
    
    mysql> select id, cert_sn  from ccc_halls_test  where  id = 80880;
    +-------+-------------------------------------------------+
    | id    | cert_sn                                         |
    +-------+-------------------------------------------------+
    | 80880 | ["A6000", ["A63784","A63785"], "A63984"]        |
    +-------+-------------------------------------------------+
    
### JSON_REMOVE
    
    mysql> update ccc_halls_test set cert_sn = JSON_ARRAY_REMOVE(cert_sn, '$[1]', 'A6000') WHERE  id = 80880;
    
    mysql> select id, cert_sn  from ccc_halls_test  where  id = 80880;
    +-------+----------------------------+
    | id    | cert_sn                    |
    +-------+----------------------------+
    | 80880 | ["A6000", "A63984"]        |
    +-------+----------------------------+

## 创建索引

    MySQL的JSON格式数据不能直接创建索引，但是可以变通一下，把要搜索的数据单独拎出来，单独一个数据列，然后在这个字段上创建一个索引，官方的例子：
    
    CREATE TABLE muscleape_office(
      c JSON,
      g INT GENERATED ALWAYS AS (c->"$.id"),
      INDEX i (g)
    );


# MySQL常用函数

## 1.  concat 字符串连接

        返回结果为连接参数产生的字符串。如有任何一个参数为NULL ，则返回值为 NULL。可以有一个或多个参数。

        mysql> select id, file_name from cubor_chain_report where id = 122;
        +-----+--------------+
        | id  | file_name    |
        +-----+--------------+
        | 122 | 片商报表     |
        +-----+--------------+

        mysql> select id, file_name, concat(file_name, '-测试') as concat_name from cubor_chain_report where id = 122; 
        +-----+--------------+---------------------+
        | id  | file_name    | concat_name         |
        +-----+--------------+---------------------+
        | 122 | 片商报表     | 片商报表-测试       |
        +-----+--------------+---------------------+

        mysql> select id, file_name, concat(file_name, '-测试', '-1') as concat_name from cubor_chain_report where id = 122;
        +-----+--------------+-----------------------+
        | id  | file_name    | concat_name           |
        +-----+--------------+-----------------------+
        | 122 | 片商报表     | 片商报表-测试-1       |
        +-----+--------------+-----------------------+

## 2. CONCAT_WS 字符串连接 指定参数之间的分隔符

        分隔符可以是一个字符串，也可以是其它参数。如果分隔符为 NULL，则结果为 NULL。函数会忽略任何分隔符参数后的 NULL 值。
        但是CONCAT_WS()不会忽略任何空字符串。 (然而会忽略所有的 NULL）。

        mysql> select id, file_name, CONCAT_WS("-", file_name, "测试") as concat_name  from cubor_chain_report where idd = 122;
        +-----+--------------+---------------------+
        | id  | file_name    | concat_name         |
        +-----+--------------+---------------------+
        | 122 | 片商报表     | 片商报表-测试       |
        +-----+--------------+---------------------+

        mysql> select id, file_name, CONCAT_WS("-", file_name, "测试", NULL, 'LAST') as concat_name  from cubor_chain_rreport where id = 122;
        +-----+--------------+--------------------------+
        | id  | file_name    | concat_name              |
        +-----+--------------+--------------------------+
        | 122 | 片商报表     | 片商报表-测试-LAST       |
        +-----+--------------+--------------------------+

        mysql> select id, file_name, CONCAT_WS("-", file_name, "测试", '', 'LAST') as concat_name  from cubor_chain_repport where id = 122;
        +-----+--------------+---------------------------+
        | id  | file_name    | concat_name               |
        +-----+--------------+---------------------------+
        | 122 | 片商报表     | 片商报表-测试--LAST       |
        +-----+--------------+---------------------------+

        mysql> select id, file_name, CONCAT_WS("-", file_name, "测试", ' ', 'LAST') as concat_name  from cubor_chain_reeport where id = 122;
        +-----+--------------+----------------------------+
        | id  | file_name    | concat_name                |
        +-----+--------------+----------------------------+
        | 122 | 片商报表     | 片商报表-测试- -LAST       |
        +-----+--------------+----------------------------+

## GROUP_CONCAT（）函数

    返回一个字符串结果，该结果由分组中的值连接组合而成

    mysql> select cr.id as report_id, cd.id, cd.file_name from cubor_settle_report as cr inner join 
    cubor_settle_report_detail as cd  on  cr.id = cd.report_id;

    +-----------+----+-----------+
    | report_id | id | file_name |
    +-----------+----+-----------+
    |         2 |  1 | 1.csv     |
    |         2 |  2 | 2.csv     |
    |         3 |  3 | 3.csv     |
    |         3 |  4 | 4.csv     |
    |         4 |  5 | 5.csv     |
    |         4 |  6 | 6.csv     |
    |         5 |  7 | 7.csv     |
    |         5 |  8 | 8.csv     |
    |         6 |  9 | 9.csv     |
    |         6 | 10 | 10.csv    |
    |         7 | 11 | 11.csv    |
    |         7 | 12 | 12.csv    |
    +-----------+----+-----------+

### 系统默认的分隔符是逗号
    mysql> select cr.id as report_id, GROUP_CONCAT(cd.id) as sub_ids  from cubor_settle_report as cr inner join cubor_settle_report_detail as cd 
    on  cr.id = cd.report_id group by cr.id;

    +-----------+---------+
    | report_id | sub_ids |
    +-----------+---------+
    |         2 | 1,2     |
    |         3 | 4,3     |
    |         4 | 5,6     |
    |         5 | 7,8     |
    |         6 | 9,10    |
    |         7 | 11,12   |
    +-----------+---------+

### 修改分隔符:
    mysql> select cr.id as report_id, GROUP_CONCAT( cd.id  SEPARATOR '_') as sub_ids  from cubor_settle_report as cr inner join cubor_settle_report_detail as cd 
    on  cr.id = cd.report_id group by cr.id;

    +-----------+---------+
    | report_id | sub_ids |
    +-----------+---------+
    |         2 | 1_2     |
    |         3 | 4_3     |
    |         4 | 5_6     |
    |         5 | 7_8     |
    |         6 | 9_10    |
    |         7 | 11_12   |
    +-----------+---------+

### 排序:
    mysql> select cr.id as report_id, GROUP_CONCAT(cd.id ORDER BY cd.id DESC SEPARATOR '_') as sub_ids  from cubor_settle_report as cr inner join cubor_settle_report_detail as cd 
    on  cr.id = cd.report_id group by cr.id;

    +-----------+---------+
    | report_id | sub_ids |
    +-----------+---------+
    |         2 | 2_1     |
    |         3 | 4_3     |
    |         4 | 6_5     |
    |         5 | 8_7     |
    |         6 | 10_9    |
    |         7 | 12_11   |
    +-----------+---------+

### GROUP_CONCAT 的最大长度：
     
     GROUP_CONCAT将某一字段的值按指定的字符进行累加，系统默认的分隔符是逗号，可以累加的字符长度为1024字节。

    修改默认字符大小：
        1. 在MySQL配置文件中加上
            group_concat_max_len = 102400 #你要的最大长度

        或者：
         mysql> SET GLOBAL group_concat_max_len=102400;

## 4. date_format：将日期转换成字符

    mysql> select id, create_time, DATE_FORMAT(create_time, '%Y-%m-%d') as create_date from cubor_chain_report wherre id = 122;
    +-----+---------------------+-------------+
    | id  | create_time         | create_date |
    +-----+---------------------+-------------+
    | 122 | 2020-05-01 00:00:00 | 2020-05-01  |
    +-----+---------------------+-------------+

    mysql> select id, create_time, DATE_FORMAT(create_time, '%Y年%m月') as create_month from cubor_chain_report wheere id = 122;
    +-----+---------------------+--------------+
    | id  | create_time         | create_month |
    +-----+---------------------+--------------+
    | 122 | 2020-05-01 00:00:00 | 2020年05月   |
    +-----+---------------------+--------------+

## 5. str_to_date：将字符通过指定的格式转换成日期
    字符串必须为标准时间格式

    mysql> select id, create_time, str_to_date("2020-5-15", '%Y-%m-%d') as create_date from cubor_chain_report where id = 122;
    +-----+---------------------+-------------+
    | id  | create_time         | create_date |
    +-----+---------------------+-------------+
    | 122 | 2020-05-01 00:00:00 | 2020-05-15  |
    +-----+---------------------+-------------+

    mysql> select id, create_time, str_to_date("2020-05", '%Y-%m') as create_date from cubor_chain_report where id  = 122;
    +-----+---------------------+-------------+
    | id  | create_time         | create_date |
    +-----+---------------------+-------------+
    | 122 | 2020-05-01 00:00:00 | NULL        |
    +-----+---------------------+-------------+

    某个字段是str不是标准时间格式,可通过拼接完成

    mysql> select id, file_name, str_to_date(file_name, '%Y-%m-%d') as create_date from cubor_chain_report where idd = 122;
    +-----+-----------+-------------+
    | id  | file_name | create_date |
    +-----+-----------+-------------+
    | 122 | 2020-05   | NULL        |
    +-----+-----------+-------------+

    mysql> select id, file_name, str_to_date(concat(file_name,"-12"), '%Y-%m-%d') as create_date from cubor_chain_rreport where id = 122;
    +-----+-----------+-------------+
    | id  | file_name | create_date |
    +-----+-----------+-------------+
    | 122 | 2020-05   | 2020-05-12  |
    +-----+-----------+-------------+
