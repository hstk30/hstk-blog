mysql
======

INNER JOIN
-----------


COUNT() 的作用
---------------

- 统计某个列值的数量
- 统计行数

在统计列值时要求列值为非空的（不统计``NULL``）。如果要统计行数，直接使用``COUNT(*)``，性能也更好

如何在同一个查询中统计同一列的不同值得数量，以减少查询的数量
--------------------------------------------------

比如，需要找出所有待完成的事和已完成的事，或者总共的事::

    SELECT SUM(IF(is_complete=0, 1, 0)) AS pending_task_num, 
        SUM(IF(is_complete=1, 1, 0)) AS completed_task_num
    FROM tasks;

或者::
    
    SELECT COUNT(is_complete=0 OR NULL) AS pending_task_num,,
        COUNT(id_complete=1 OR NULL) AS completed_task_num
    FROM tasks;

在Django 中貌似没有找到IF 语句，但是可以使用``Case When``语句实现相同的功能::

    Tasks.objects.aggregate(
        pending_task_num=Sum(
            Case(
                When(is_complete=True, then=1), default=0, output_field=IntegerField()
            )
        ),
        completed_task_num=Sum(
            Case(
                When(is_complete=False, then=1), default=0, output_field=IntegerField()
            )
        )
    )

在Django 你可能想直接使用原生的sql进行查询，但是行不通。因为Django 中要求进行原生sql查询时一定要带有模型的主键。
