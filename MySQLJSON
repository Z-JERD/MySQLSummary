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
    
    json_extract 提取json值
    
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
