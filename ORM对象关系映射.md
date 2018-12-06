##ORM对象关系映射<br>实现了数据模型与数据库的解耦,数据模型的设计不需要依赖于特定的数据库，通过简单的配置就可以轻松更换数据库

###功能<br>根据对象类型生成表。将对SQL语句的操作，转换成对象操作。
<br>
-- 

####Django支持常用数据库
* sqlite
* MySQL
* Oracle
* POstgreSQL

####Django改用MySQL数据库(修改settings.py文件中的DATABASE字典)
######先创建MySQL数据库，然后修改settings.py文件
		DATABASE = {
		'default':{
			"ENGINE":'django.db.backends.mysql',
			"NAME":"数据库名字",
			"USER"："账号"，
			"PASSWORD":"密码",
			"HOST":"数据库地址",
			"PORT":"端口号"
}
}
		迁移数据
		python manage.py migrate


####增加数据
		Model.py#添加一个类对象
		class Category(models.Model):
			name = models.CharFidle(max_length = 100)
			def __unicode__:
				return u"%s"%self.name

		views.py
		cate = Category(name="张三")
		cate.save()

####查看数据
		Category.objects.all()

####查看执行的SQL语句
		from django.db						 import connection		#connection指当前数据库连接
		connection.queries					#queries	指的是执行过的所有语句最后一条是刚刚执行的语句
		connection.queries[-1]["sql"]		#取最后一次执行的SQL语句，看不懂取出的内容的话，可以print打印一下就可以看懂了

		#定义一个show_sql方法
		def show_sql():
			from django.db.connection
			queries = connection.queries
			print queries[-1]['sql']

####根据已有的表生成model实体类
		修改settings.py文件中的默认数据库名称为要导入的数据库名称
		python manage.py insepectdb >appname/models.py	#insepectdb 针对已有的表数据生成model实体类
		Django与Author开头的类可以删掉

		
####Django 的ORM类库很有可能有惰性加载或者懒加载
* 对于返回数据多余一个的，会惰性查询，返回结构为1个的不会产生惰性查询。get、first、last，不会产生惰性
* 惰性查询是什么时候用什么时候查
* 查询多条数据的时候，不是全部查询，而是查询有限的数据条数（21条）。django version 1.11.6
 
####CRUD####

#####retrieve(检索)
* `category.objects.all()执行的语句是 select * from category limit 21`

* 查询一条记录(get()方法)
######等于
    `* category.objects.get(id = 1)执行的语句是 select * from category where category.id =1`
	`* category.objects.get(id__exact=1)执行的语句是 select * from category where category.id =1`
#####大于
git只能获取一条数据，如果查询的结果不存在或者多余1条会报错，如果查询结果多余1条使用filter
* `category.objects.filter(id__gt=1)执行的语句是 select * from category where category.id >1 limit 21`
#####小于
* `category.objects.filter(id__lt=1)执行的语句是 select * from category where category.id <1 limit 21`
#####大于等于
* `category.objects.filter(id__gte=1)执行的语句是 select * from category where category.id >=1 limit 21`
#####小于等于
* `category.objects.filter(id__lt=1)执行的语句是 select * from category where category.id <=1 limit 21`
#####在范围内（区间内）
* `category.objects.filter(id__range=(1,10))执行的语句是 select * from category where category.id between 1 and 10 limit 21`

#####包含关键字的
* `category.objects.filter(name__contains="战争")执行的语句是 select * from category where category.name like "%战争%" limit 21`
#####不包含关键字
* `category.objects.exclude(name__contains='战争')`
#####以关键字开头
* `category.objects.filter(name__startswith="战争")执行的语句是 select * from category where name like binary "战争%" limit 21`
#####以关键字结尾
* `category.objects.filter(name__endswith="战争")`
#####正则匹配
* `category.objects.filter(name__iregex="^\s^战争\w*$")`
####F函数(from django.db.models import F)
* F函数是使用字段的值的一个函数接收一个字段名称（应用场景统一的为某一批用户增加属性或权限）
* 例：修改一批用户的时间
*     `from datetime import timedelta		#专门针对时间进行加减操作`
*      `d = date(2018,11,4)`					
* `d+timedelta(days=1)					#正数为加，负数为减`
* `article.objects.update(update = F('update')+timedelta(days=1))  update是文章的发布时间F('update')是获得当前文章发布时间字段的值+timedelta(days=1) 是为update字段重新赋`
* 带条件更新数据
* `article.objects.filter(id = 2).update(update = F('update')+timedelta(days=1))`

####查询以什么开头以什么结尾
* `Category.objects.filter(name__startswith="战争",name_endswith="结束")`
####获得第一条数据
* `Category.objects.first()`
####获取最后一条数据
* `Category.objects.last()`
####倒叙查询（order_by 排序）
* `Category.objects.order_by('-id').all()[:10]`
* `Category.objects.order_by('-id')。order_by('name').all()[:10]`	#按照名字排序与ID排序
####使用values查询部分字段
* `Category.objects.values('field')`
####QuerySet（结果集）两大特点
* 惰性查询
* 会缓存数据
####聚合函数
#####针对一系列记录的

* django 常用五大聚合函数，count、max、min、Sum、AVG
* `from django.db.models import Count,Max,Min,Sum,Avg`
* 统计函数
* `Category.objects.aggregate(count = Count("*"))			#aggregate是使用聚合函数方法，使用的时候聚合函数需要别称`
* 使用聚合查询最大ID
* `Category.objects.aggregate(max = Max("id"))`
* 分组(group_by)聚合一旦和分组使用就会统计分组下的记录。
* Django 分组 annotate()
* `Category.objects.values("name").annotate(count = Count("*"))		#先使用values函数得到所有的name值，然后使用annotate进行分组annotate可以使用聚合函数分析`
###Django使用原生SQL语句
* `Category.objects.raw('sql语句')`
* 使用connection链接对象增删查改数据
* `from django.db import connection	#导入django自带的数据库连接对象`
* `cursor = connection().cursor()	#根据连接对象获得游标`
* `cursor.execute('sql语句')			#根据游标执行SQL语句`
* `cursor.close()`


