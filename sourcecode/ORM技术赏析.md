# ORM技术赏析

## 名词解析

名词 | 解析
--- | ----
DAL | 数据访问层(Data Access Layer)， 主要是对数据库数据的操作层，为业务逻辑层提供数据服务
DAO | 数据访问对象(Data Access Object), DAL的具体实现，提供对数据访问的一个封装
ORM | 对象关系映射(Object-relational mapping)， 与DAO一样提供对数据访问的封装，不同的是在于将你程序中的数据对象自动转化为关系型数据库中的表和列
ActiveRecord | ORM模式的一个实现，同时提供数据持久化和数据对象集合操作
Eloquent | ActiveRecord的一个具体实现，集合在Laravel框架中

## PHP数据库连接使用示例

PHP中一般数据库操作如下，其主要包括两种方式：mysql专用扩展以及PDO数据抽象层

	# mysqli

	$con = mysqli_connect("localhost", "waimai", "123456")   # connect
		or die("Could not connect： " . mysql_error());

	mysqli_select_db("mis");                                 # select db

	$sql = "select * from wfe_domain limit 1";               # build sql
	$result = mysqli_query($sql);                            # db query
	while ($row = mysqli_fetch_assoc($result)) {             # handle result
		# do something ...
	}
	mysqli_close($con);	

	# pdo

	$dsn = "mysql:host=localhost;dbname=mis";
	try {
		$pdo = new PDO($dsn, "waimai", "123456");            # connect
	} catch(PDOException $e) {
		echo "connect error: " . $e->getMessage() . "<br>";
	}
	
	$sql = "select * from wfe_domain where id>:id";          # build sql
	$pdoSta = $pdo->prepare($sql);                          
	$pdoSta->execute(["id" => 1]);                           # db query
	while($row = $pdoSta->fetch(PDO::FETCH_ASSOC)) {         # handle result
		# do something ...
	}

从上可以看出，数据操作包含以下步骤：

* 数据库管理系统(RDBMS)连接
* 数据库选择
* 构建SQL语句
* 查询数据
* 返回数据处理

## ORM源码分析
因此，构建一个能用的ORM，需要解决以下几个问题

* 数据库连接
* SQL语句的构建
* 数据对象的转化
* 关联关系的处理

### 连接

![](http://i.imgur.com/gVPC0Bv.png)

如图所示，`ORM`其实是对`PDO`的一个封装,通过实现简单工厂模式，来实现差异化连接生成。而`DabaseManager`类主要对连接的生命周期进行管理。

目前`Eloquent`支持的`RDBMS`有：`mysql, postgres, sqlite, sqlserver`四种

### 语句构建	

SQL语句的构建是委托给查询构造器（`Illuminate\Database\Eloquent\Builder`）来完成。

已`Dao_User::find(1)`为例子，来看下查询构造器是怎样生成`select * from user where id = 1`这样的语句并执行相关查询。

当调用静态方法`find`的时候, 由于实际上Model，因此实际调用模式方法`__callStatic`。

	public static function __callStatic($method, $parameters)  
	{
		return (new static)->$method(...$parameters);
	}

由此看来，此时先使用后期静态绑定（`new static`)的方法生成Dao_User类，然后再通过普通调用调用`find`方法，需要注意的是，涉及到查询相关的所有方法都被封装在查询构造器中，Model类只涉及数据库保存的相关方法。因此，可以看到此时会调用另外一个魔术方法`__call`

	public function __call($method, $parameters)
    {    
        if (in_array($method, ['increment', 'decrement'])) {
            return $this->$method(...$parameters);
        }    

        return $this->newQuery()->$method(...$parameters);
    } 

可以看到， 此时通过`newQuery`方法生成了查询构造器(Builder类)，然后将`find`方法委托给此类执行

    public function find($id, $columns = ['*'])
    {    
        return $this->whereKey($id)->first($columns);
    }

在此通过`wherekey`方法，添加了已主键`id`，值`$id`的条件数组，然后通过first的方法

	public function first($columns = ['*'])
    {
        return $this->take(1)->get($columns)->first();
    }

最后执行`get`操作，而`get`最终完成语句构建，查询，结果处理等工作

    public function get($columns = ['*'])
    {

        $results = $this->processor->processSelect($this, $this->runSelect());

        return collect($results);
    }

	protected function runSelect()
    {
        return $this->connection->select(
            $this->toSql(), $this->getBindings(), ! $this->useWritePdo
        );
    }

	# 将builder中的sql语句段（数组）组合成sql语句
    public function toSql()
    {
        return $this->grammar->compileSelect($this);
    }
	

	# connection
	# 执行查询语句
	public function select($query, $bindings = [], $useReadPdo = true)
    {
        return $this->run($query, $bindings, function ($query, $bindings) use ($useReadPdo) {
            if ($this->pretending()) {
                return [];
            }

            $statement = $this->prepared($this->getPdoForSelect($useReadPdo)
                              ->prepare($query));

            $this->bindValues($statement, $this->prepareBindings($bindings));

            $statement->execute();

            return $statement->fetchAll();
        });
    }

从代码中可以看到，首先get方法中，通过`grammer`类对sql语句段进行相关的组合，然后，将组合的语句和绑定参数传递到`connection`中执行

### 数据对象转化

查询返回的结果是一个集合类，包含着一个以php内置类(stdClass)为元素的数组，因此为了实现`ActiveRecord`模式，需要将此数组转换为Model为元素的数组

具体只需调用这一个具体方法

	public function hydrate(array $items)
    {
        $instance = $this->newModelInstance();

        return $instance->newCollection(array_map(function ($item) use ($instance) {
            return $instance->newFromBuilder($item); 
        }, $items));
    }

    public function newFromBuilder($attributes = [], $connection = null)
    {   
        $model = $this->newInstance([], true);
        
        $model->setRawAttributes((array) $attributes, true);
        
        $model->setConnection($connection ?: $this->getConnectionName());

        return $model;
    }

如上所示，`hydrate`方法中对每一个数组元素构建`Model`类，其中数据表一行中的字段和值，都保存在`Model`的`attributes`属性中
  

### 关联关系处理

数据间的关联关系，归纳起来无非是一下几种情况：

* One-To-One
* One-To-Many
* Many-To-Many

以"User-Phone"这一个"One-To-One"的关系为例，来了解一下ORM怎样解决关联关系

	class User extends Model {	
	    public function phone() {
	        return $this->hasOne('phone', 'user_id');
	    }   
	}

	User::find(1)->phone;

如例子所示，类`User`定义了与类`Phone`之间的`One-To-One`关系，即“一个用户拥有一个手机号”，当如前所属调用`User::find(1)`来获取一个User的Model实例以后，然后用`->phone`获取User类中的**phone**属性。

当然， 该属性并未在Model类中存在定义。因此，调用魔术方法`__get`

	public function __get($key)
    {
        return $this->getAttribute($key);
    }

	public function getAttribute($key)
    {       
        if (array_key_exists($key, $this->attributes) ||
            $this->hasGetMutator($key)) {
            return $this->getAttributeValue($key);
        }

        return $this->getRelationValue($key);
    }

	public function getRelationValue($key)
    {
        if (method_exists($this, $key)) {
            return $this->getRelationshipFromMethod($key);
        }
    }

    protected function getRelationshipFromMethod($method)
    {       
        $relation = $this->$method();

        return tap($relation->getResults(), function ($results) use ($method) {
            $this->setRelation($method, $results);
        });
    }

`getAttribute`方法首先会判断给属性是否为`attribute`的key(即为User表的一个字段名)， 如果不是，这认为这是关联模型的一个属性，进而调用`getRelationValue`方法初始化Relation类。

Relation类是Eloquent中为了处理关联关系而使用的一组类，其类关系如下

![](http://i.imgur.com/7cFVena.png)

如图所示，每一种子关系都继承自Relation类，各自实现`getResult`来获取关联模型的数据。注意其中有些关系是成对出现（比如"一个用户拥有一个手机号"和"该手机号属于一个用户"）。

    public function getResults()
    {   
        return $this->query->first() ?: $this->getDefaultFor($this->parent);
    } 

在`getResults`方法中，其实就又重新执行"语句构建-执行"这一个过程，所以不同于一般的SQL关联语句，这其实是分开两次，对数据库进行查询，如下所示

	# 一般连表查询
	select * from user, phone where user.id = 1 and phone.user_id = user.id limit 1;
	
	# ORM连表查询
	select * from user where id = 1;
	// get the $userId
	select * from phone where user_id = $userId

因此，ORM这种懒加载的形式就能解决数据库查询的`N+1`问题，检索数据以最少的查询次数解决问题


## 性能测试
(待补充)

## 参考
* [orm 系列 之 Eloquent演化历程1](http://www.jianshu.com/p/6364958b138f "orm 系列 之 Eloquent演化历程1")
* [Laravel数据库：入门](http://d.laravel-china.org/docs/5.4/database "Laravel数据库：入门")
