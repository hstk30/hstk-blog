Django ORM
============

order_by
---------

后一个 order_by 将覆盖前一个 order_by ，想要直接使前面的 order_by 失效（比如说，某个models的meta中有ordering属性）那么 
在最后加 order_by() ，这将使得生产的 sql 语句的 order_by 部分变成 **order by null**

annotate and values
---------------------

和使用 filter() 子句一样，作用于某个查询的annotate() 和  values() 子句的使用顺序是非常重要的。如果values() 子句在  annotate() 之前，就会根据 values()  子句产生的分组来计算注解。

但是，如果  annotate()  子句在 values()子句之前，就会根据整个查询集生成注解。在这种情况下，values() 子句只能限制输出的字段范围。

::

    -- City.objects.values('population').annotate(Count('id'))
    SELECT `blog_city`.`population`, COUNT(`blog_city`.`id`) AS `id__count` 
    FROM `blog_city` 
    GROUP BY `blog_city`.`population`, `blog_city`.`name` 
    ORDER BY `blog_city`.`name` ASC  LIMIT 21

    -- City.objects.values('population').annotate(Count('id')).order_by()
    SELECT `blog_city`.`population`, COUNT(`blog_city`.`id`) AS `id__count` 
    FROM `blog_city` 
    GROUP BY `blog_city`.`population` 
    ORDER BY NULL  LIMIT 21

    -- City.objects.values('id', 'population').annotate(Count('id'))
    SELECT `blog_city`.`id`, `blog_city`.`population`, COUNT(`blog_city`.`id`) AS `id__count` 
    FROM `blog_city` 
    GROUP BY `blog_city`.`id`
    ORDER BY `blog_city`.`name` ASC  LIMIT 21

    -- City.objects.values( 'population', 'name').annotate(Count('id'))
    SELECT `blog_city`.`population`, `blog_city`.`name`, COUNT(`blog_city`.`id`) AS `id__count` 
    FROM `blog_city` 
    GROUP BY `blog_city`.`population`, `blog_city`.`name` 
    ORDER BY `blog_city`.`name` ASC  
    LIMIT 21

    -- City.objects.annotate(Count('id'))
    SELECT `blog_city`.`id`, `blog_city`.`name`, `blog_city`.`country_id`, `blog_city`.`population`, COUNT(`blog_city`.`id`) AS `id__count` 
    FROM `blog_city` 
    GROUP BY `blog_city`.`id` 
    ORDER BY `blog_city`.`name` ASC  
    LIMIT 21

    -- City.objects.annotate(city_id=F('id'))
    SELECT `blog_city`.`id`, `blog_city`.`name`, `blog_city`.`country_id`, `blog_city`.`population`, `blog_city`.`id` AS `city_id` 
    FROM `blog_city` 
    ORDER BY `blog_city`.`name` ASC  
    LIMIT 21

