Django ORM
============

count 和 len 的区别
---------------------------

::
    from django.core.paginator import Paginator

    def p_false(data, page, count):
        p = Paginator(data, count)
        return p.num_pages, len(data), p.page(page).object_list

    def p_true(data, page, count):
        p = Paginator(data, count)
        return p.num_pages, p.count, p.page(page).object_list

    pg, cnt, pl = p_false(data, page, count)
    # 1. select count(*) from xx;
    # 2. select xx, xxx from aaa;
    assert type(pl) == list  # after len(data) the type(data) be the <list>

    pg, cnt, pl = p_true(data, page, count)
    # 1. select count(*) from xx;
    # 2. select xx, xxx from aaa LIMIT count OFFSET (page-1) * count



``f_false`` 是个假的分页，实际还是会取出所有的数据，因为``len`` 是内置的函数，对``data`` 会像对``iterator`` 一样直接迭代的计算长度，
所以``data`` 本来的懒加载被破坏，执行到``len(data)`` 时直接将所有数据都加载完了。


字段选项
----------------

null
~~~~~~~~~

官方推荐 ::
    字符串字段例如CharField 和TextField 要避免使用null，因为空字符串值将始终储存为空字符串而不是NULL。如果字符串字段的null=True，那意味着对于“无数据”有两个可能的值：NULL 和空字符串。在大多数情况下，对于“无数据”声明两个值是赘余的，Django 的惯例是使用空字符串而不是NULL。


order_by
------------

后一个 order_by 将覆盖前一个 order_by ，想要直接使前面的 order_by 失效（比如说，某个models的meta中有ordering属性）那么 
在最后加 order_by() ，这将使得生产的 sql 语句的 order_by 部分变成 **order by null**

annotate and values
------------------------

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

