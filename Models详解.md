##Model详解##
###Queryset两大特性
* 惰性加载
* 缓存
	* 当获取所有数据之后，django会默认缓存数据。
	* 所有的切割[0][:]不会缓存数据，重新查数据。
	* 除了切割之外，所有的都会缓存数据。

##事务##
	from django.db import transaction
	@transaction.atomic()
	def addcategory():
		pass
* 多表操作(增删查改)
* 当多表操作，出现错误时，有的表添加了数据，有的没有添加成功数据，从而制造了脏数据，使用事务回滚的方法，以装饰器的形式引用可以解决这个问题


###models扩展功能
* 直接写自己的扩展方法。
* 定义一些models类的方法

###objects()管理器

* type(Article.objects)#<class 'django.db.models.manager.Manager'>
* manager 是所有的数据模型每个数据模型至少有一个manager
* objects是默认的管理器
* 管理器是用来增删改查(CRUD)
* 多对多添加，需要数据库中现有文章
* 什么时候自定义管理器
	* 默认的管理器没有你要用的方法。
	* 重写mannager已有的方法

			管理器自定义多对多添加

			class Category(models.Model):
			name = models.charfield(maxlength = 50)
	
		    class Tag(models.Model):
				name = models.Charfield(maxlength=50)
		
			class ArticleManager(models.Manage):
	
				'''自定义一个manager管理器(manager相当于objects)，self指Manage管理器，self.create相当于objects.create()'''
	
			    def createadd(self,title,cate,tags):
					cate = Category.objects.last()
				    article = self.create(title='zhangsan',category=cate)
					article.Tag.add(*tags)
					article.save()
	
	
			class Article(models.Model):
		        title = models.CharField(maxlength=50)
				tag = models.ManytoManyField(Tag)
				category = models.Foreignkey(Category)
				objects = ArticleManager()	这个objects是自定义的相当于ArticleManager，然后ArticleManager类里边有createadd方法。
				article = Article.create(
				title = 'XXX',
				cate = 	Category.objects.last(),
				tags = Tag.objects.last()

		)
	
