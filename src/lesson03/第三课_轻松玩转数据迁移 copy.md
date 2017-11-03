补充上节课：配置虚拟主机
不用修改nginx的配置文件，减少学习成本，只要在homestead.yaml 和host文件两个文件中做很小的改动就行
D:\03www2018\homestead\Homestead.yaml

    map: myblog.app
    to: /home/vagrant/abcde/study/myblog/public
  C:\Windows\System32\drivers\etc\hosts
  
      192.168.10.10 www.myblog.app  
D:\03www2018\study\myblog\config\app.php
```
'timezone' => 'Asia/Shanghai',
```
D:\03www2018\study\myblog\.env.home
```
DB_PREFIX=study_
```
D:\03www2018\study\myblog\config\database.php
```
'migrations' => 'migrations2017',
'prefix' => env('DB_PREFIX', ''),
```
# 一：数据库配置
1. 新建连接和数据库
在navicat for mysql中新建连接，用户名`homestead`,密码`secret`，新建数据库myblog，字符集为`utf8mb4 -- UTF-8 Unicode`，规则排序为`utf8mb4_general_ci`
2. 新增加一个用户
>GRANT ALL PRIVILEGES ON myblog.* TO daqi@localhost IDENTIFIED BY 'daqi168' WITH GRANT OPTION;

3. 给应用指定开发环境
> 有5个地方可以指定开发环境[我们这里使用第1种方法或第2种方法]
> - 在homestead.yaml中
> - 在入口文件的最开始加上 `putenv("APP_ENV=home");`
> - 修改web服务器的配置文件，如nginx配置文件中加上 env APP_ENV=home; 
>  apache配置文件中加上SetEnv APP_ENV home
> - 在.htaccess中加上
> - php主配置文件php-fpm.conf来设置

修改homestead.yaml加下
```
	variables:
	   - key: 'APP_ENV'
	     value: 'home'
	   - key: 'APP_DEBUG'
	     value: 'true'
```
4.编辑应用配置文件.env.home
先生成应用的key
> vagrant@homestead:~/abcde/study/myblog$ php artisan key:generate
> Application key [base64:s6Rhb/LhTYIpjXi3x+tN9Yon/iBavjnxO4bjXmrYq1g=] set successfully.
> 将生成的key放在配置文件.env.home中APP_KEY
> 有关base64的详细了解参考: [阮一峰的base64笔记](http://www.ruanyifeng.com/blog/2008/06/base64.html)
每个laravel必须得生成一个key，它是在加密模块中必须要用到的，

```
APP_NAME=我的博客
APP_ENV=local
APP_KEY= base64:s6Rhb/LhTYIpjXi3x+tN9Yon/iBavjnxO4bjXmrYq1g=
APP_DEBUG=true
APP_LOG_LEVEL=debug
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=myblog
DB_USERNAME=daqi
DB_PASSWORD=daqi168
DB_PREFIX=study_ //自己加

```

D:\03www2018\study\myblog\config\database.php中改
```
'prefix' => env('DB_PREFIX', ''),
'migrations' => 'migrations2017',
```


# 安装调试文件(可选)
	vagrant@homestead:~/abcde/study/myblog$ composer require advance100/helper

# 二：生成第一个迁移文件
[官方文档参考](https://laravel.com/docs/5.5/migrations)
先了解一下有关migrate的所有命令
> make:migration 新建一个迁移文件
> migrate 运行数据库迁移，这个是安装软件时执行的
> migrate:fresh        会删除所有已经生成的表，并重新运行所有的迁移
> migrate:install      新建一个迁移仓库，会在homestead数据库中生成一个表migrations
> migrate:refresh      Reset and re-run all migrations
> migrate:reset        Rollback all database migrations
> migrate:rollback     只回滚最后一条迁移
> migrate:status       显示每条迁移的状态

###make:migration
参考: \Illuminate\Database\Console\Migrations\MigrateMakeCommand
文件名格式为: date('Y_m_d_His')的值
文件的保存路径：默认为基本路径/database/migrations/下面，如果指定了--path参数就是基本路径/+指定路径
问题来了？如果我想放在其它路径，如vendor下面的某个模块下面如做操作?
答：这要用到 `php artisan vendor:publish`

以新建一个用户权限管理为例，建以下几个表
admins
roles
permissons
role_user
permisson_user

生成建表admins的迁移文件
```
vagrant@homestead:~/abcde/study/myblog$ php artisan make:migration create_admins_table
Created Migration: 2017_10_31_231348_create_admins_table
```
> 注意make:migration有**1**个必写的参数name, **3**个可选的选项 --create,--tabel,--path

> 1.  --path是指定迁移文件生成的位置，默认是放在 应用根目录/database/migrations下面，如果指定了--path=x/y/x，就会在 应用根目录/x/y/z下面生成迁移文件，注意的是得保证该目录已存在，否则会报错
> 2. 新建表时，name的写法可以是 create_表名_table，这种写法，后面的create可以省略，否则不能省，如make:migration wang --create=members
> 3. 选项前有两个减号--，新手易写掉一个
> 4. --create=表名，新建一个表，表名一般全为小写字母，复数形式
> 5. --table=表名，修改现成的表
> 6. --create与--tabel是2选1
> 7. 在命令行自动生成的迁移文件只能是在当前项目下，如果想在项目外如vendor/包/模块中生成迁移文件，得手工编写，或自动生成后拷贝到指定地方，但要注意的是模块中的服务提供商中得写publish方法
> 8. 生成的迁移文件名格式是 yyyy_mm_dd_His_name.php，文件中类名是将name写在大驼峰的格式，如name为create_admins_table变成`class CreateAdminsTable extends Migration`
> 9. 生成的文件名是可以修改的，用来调整顺序，系统的排序是`asort(名字名组成的数组,SORT_REGULAR );` 以升序的方式排列的，所以改名时要注意
> 10. 没有运行migrate之前，迁移文件是可以手工删除和改名，修改内容的

按上面生成另外几个迁移文件
```
php artisan make:migration create_roles_table
php artisan make:migration create_permissions_table
php artisan make:migration create_role_user_table
php artisan make:migration create_permission_role_table
```

# 三：编辑迁移文件
>**up**是migrate时执行的方法
>**down**是migrate:rollback,migrate:reset,migrate:fresh,migrate:refresh时会执行的方法
> **Facade**（有人翻译成门面,我觉得翻译成表面比较合适些） **Schema::方法**实际上是执行 `\Illuminate\Database\Schema\MySqlBuilder`的方法，分下面几种
>>判断的有 hasTable,hasColumn,hasColumns
>>读取的有 getAllTables,getColumnListing,getColumnType,getConnection
>>设置的有 setConnection,blueprintResolver
>>删除的有 dropAllTables
>>操作的有 rename,enableForeignKeyConstraints,disableForeignKeyConstraints
 *** 迁移常用有 create,table,drop,dropIfExists ***

> **Blueprint**(翻译成蓝图)，其实就是代表一个数据库中的表，它的方法主要有2种，一是针对命令commands的方法，另是针对列columns的方法，命令和列都是一个流Fluent对象，处理命令的方法不多，主要熟悉的是处理列的方法，下面是针对Mysql数据的一些列方法和修饰器

```
<?php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;
class CreateAdminsTable extends Migration{
    public function up(){
        Schema::create('admins', function (Blueprint $table) {       
            $table->charset='utf8mb4';
            $table->collation='utf8mb4_unicode_ci';
            $table->engine='innodb';
            //$table->temporary();
            // 上面四条是演示可以单独给某个表指定以上属性
            $table->increments('id');
            $table->string('name')->comment("用户名");
            $table->string('email')->unique()->comment("邮箱必须唯一"); //unique()为索引命令,comment为修饰
            $table->string('password',60)->comment("密码最长为60个字节");
            $table->rememberToken()->comment("口令");
            $table->text('description')->nullable()->comment("管理员说明");
            $table->timestamps();
        });
    }

    public function down(){
        Schema::dropIfExists('admins');
    }
}
```
###3.1 列Columns
- getColumns()获取蓝图`Blueprint`的所有列,每一列也是一个流`Fluent`对象,流对象中有type和name两个必须指定
- addColumn($type, $name, array $parameters = []) **重点讲**
- removeColumn($name)
- getAddedColumns() 获取所有新加的列
- getChangedColumns() 获取所有改动了的列

#### 3.1.1 字符串类型
| 分类 | 方法 | mysql类型 |使用说明 |
|--------|:--------|:--------|:-|
|字符串01  |  char(列名, 长度 = null) | char |定长字符串|
|字符串02 | string(列名, 长度 = null) | varchar|长度可变字符串|
|字符串03 | text(列名)|text| 支持2^16-1个字节，相当于2.1万个汉字|
|字符串04 | mediumText(列名) |  mediutext|支持2^24-1个字节，相当于560万个汉字
|字符串05 | longText(列名) |  longtext|支持2^32-1个字节，相当于14亿个汉字
|实现01: |  rememberToken()|varchar(100)|string('remember_token', 100)->nullable()
|实现02: |  uuid(列名)  | char(36)||
|实现03: |  ipAddress(列名) |  varchar(45)||
|实现04: |  macAddress(列名)  |varchar(17)||

> 说明1: string长度指的是字符长度,不是字节,最大支持2^8-1=255个字符,1个数字，字母，汉字都是算一个字符,string('name',25)表示可以储存25个汉字
> 说明2: text长度指的是字节，
> 说明3: char适合等长密码，邮编等,默认为...Builder::$defaultStringLength，这个值是可以在服务提供商中修改的
> 说明4: text与string有些不一样，在utf8编码下，一个汉字相当于3个字节，最大支持21845个汉字,多一个字就会报错

#### 3.1.2 整数类型
| 分类 | 方法 | mysql类型 |使用说明 |
|--------|:--------|:--------|:--------|
|整数01 | *参数(列名, 自增长 = false, 无符号 = false)*<br>tinyIntegert 和 unsignedTinyInteger |    tinyint| 占1个字节，长度2^8|
|整数02 |        integer 和 unsignedInteger|int|占2个字节，长度2^16|
|整数03 |   smallInteger 和 unsignedSmallInteger |smallint|占3个字节，长度2^24|
|整数04 |  mediumInteger 和 unsignedMediumInteger |mediumint| 占4个字节，长度 2^32|
|整数05 |     bigInteger 和 unsignedBigInteger |bigint|   占8个字节，长度2^63|
|自增长列01 |  increments(列名) unsignedInteger(列名, true)|
|自增长列02 |  tinyIncrements(列名) unsignedTinyInteger(列名, true)|
|自增长列03 |  smallIncrements(列名) unsignedSmallInteger(列名, true)|
|自增长列04 |  mediumIncrements(列名) unsignedMediumInteger(列名, true)|
|自增长列05 |  bigIncrements(列名)  unsignedBigInteger(列名, true)|
|真假| boolean($column) | tinyint(1)|
> 说明1: 整数类型都指定不了长度，也指定不了zerofill，不过这没有关系，因为整数实际存储的长度与指定的长度没有关系，只是显示的长度而已

#### 3.1.3 小数类型
| 分类 | 方法 | mysql类型 |使用说明 |
|--------|:--------|:--------|:--------|
|单精度浮点数| float($column, $total = 8, $places = 2)| 弃用|单精度必须要指定M和D|
|双精度浮点数|double($column, $total = null, $places = null)| double(M,D) 或double||
|正负定点数| decimal($column, $total = 8, $places = 2)<br>unsignedDecimal | decimal(M,D)|精度和标度必须指定，默认为(8,2)|
>说明1: float是单精度浮点数，double是双精度浮点数，decimal是定点数
说明2: float占4个字节，double占8个字节
说明3: 这3个类型也可以理解为小数
说明4: 浮点数存在误差问题，定点数精确，对货币等对精度敏感的数据，应该用定点数表示或存储；   
说明5: 编程中，如果用到浮点数，要特别注意误差问题，并尽量避免做浮点数比较；   
说明6: FLOAT(M,D)，DOUBLE(M,D)。表示共M位数，小数点前面为M-D位，小数点后面为D位
说明7: decimal在mysql内存是以字符串存储的

#### 3.1.4 日期时间
| 分类 | 方法 | mysql类型 |使用说明 |
|--------|:--------|:--------|:--------|
|日期 | date($column) | date||
|时间 | time($column) | time | |
|时间 | timeTz($column) | time | |
|日期时间 | dateTime($column, $precision = 0) |   datetime(精度)<br>datetime | |
|日期时间 | dateTimeTz($column, $precision = 0)  | datetime | |
|时间戳 | timestamp($column, $precision = 0)  | timestamp| 用使用useCurrent使用当前时间 |
|时间戳 | timestampTz($column, $precision = 0) |  timestamp | |
|时间戳实现 | timestamps($precision = 0){...}||timestamp('created_at', $precision)->nullable();<br>timestamp('updated_at', $precision)->nullable();|
|时间戳实现 |  timestampsTz($precision = 0){...} |  | timestampTz('created_at', $precision)->nullable();<br>timestampTz('updated_at', $precision)->nullable();|
|时间戳实现 | nullableTimestamps($precision = 0)    |  |  是timestamps的别名 | 
|时间戳实现 | softDeletes(列名 = 'deleted_at',精度 = 0) |  |  | 
|时间戳实现 | softDeletesTz($precision = 0) |  |  | |


> 说明01: datetime不支持取默认值now()，版本5.6之后的可以默认值CURRENT_TIMESTAMP,它会取当前时间，并转为2017-11-01 23:25:45类的格式，但这种情况不能指定精度，另处写法是new \Illuminate\Database\Query\Expression('CURRENT_TIMESTAMP')才能使用Mysql中的常量
>说明02: timestamp('列名',精度)，如果指定了精度就不能使用修饰->useCurrent()，框架源代码此处是一个bug，等修正， 只有不指定精度时才可能使用useCurrent
>说明03: datetime(3)意思是保留3为毫秒数，timestamp('login',3);的结果如2017-11-02 02:08:34.864，也是保留3位毫秒
>说明04: TZ是系统时区的意思，但实际使用中使用非Tz格式就行
>说明05: 未提供year的方法，可能是year的范围太小的原因舍弃了，可以使用date获取

|日期类型    |    存储空间     |  日期格式        |         日期范围 |
|--------|:--------|:--------|:--------|
|datetime    |   8 bytes  | YYYY-MM-DD HH:MM:SS  | 1000-01-01 00:00:00 至 9999-12-31 23:59:59 
|timestamp  |    4 bytes  | YYYY-MM-DD HH:MM:SS  | 1970-01-01 00:00:01 至 2038 
|date      |     3 bytes |  YYYY-MM-DD           | 1000-01-01          至 9999-12-31 
|year      |     1 bytes  | YYYY                 | 1901                至 2155


#### 3.1.5 其它
| 分类 | 方法 | mysql类型 |使用说明 |
|--------|:--------|:--------|:--------|
|枚举| enum($column, ['市场部','设计部','总裁办'])|生成enum('市场部','设计部','总裁办')||
||json($column) |json|这是5.7.7后才有的功能|
||jsonb($column) |json|这是5.7.7后才有的功能|
|二进制|binary($column) |blob|||
> 说明：json很好用，但要看mysql版本支不支持
> 说明: 没有mediumblob和longblob，这样储存图片声音等文件没有办法，解决办法是在

```
Schema::create("member", function($table) {
    // 这里可以放SchemaBuilder支持的各种方法
});
DB::statement("ALTER TABLE member ADD imagedata MEDIUMBLOB");
```
#### 3.1.6 空间
MySQL 5.7 GIS特性

| 方法 | mysql类型 |使用说明 |
|:--------|:--------|:--------|
|geometry($column)  | geometry||
|point($column)  | point||
|lineString($column)|   linestring||
|polygon($column)  | polygon||
|geometryCollection($column) |  geometrycollection||
|multiPoint($column)  | multipoint||
|multiLineString($column)  | multilinestring||
|multiPolygon($column) |  multipolygon||


#### 3.1.7 综合实现: 生成索引
```
morphs($name, $indexName = null){
    $this->unsignedInteger("{$name}_id");
    $this->string("{$name}_type");
    $this->index(["{$name}_id", "{$name}_type"], $indexName);
}

nullableMorphs($name, $indexName = null){
    $this->unsignedInteger("{$name}_id")->nullable();
    $this->string("{$name}_type")->nullable();
    $this->index(["{$name}_id", "{$name}_type"], $indexName);
}
```


###3.2:  Blueprint中的列的修饰符
Nullable,Default,Comment,Unsigned,VirtualAs,StoredAs,Charset,Collate,Increment,After,First

###3.3  索引
###### 添加索引的命令是
indexCommand($type, $columns, $index, $algorithm = null)

###### 删除索引的命令是
dropIndexCommand($command, $type, $index)

5种索引的单独命令

|命令|实际|
|--------------------|---------------------------|
|primary| indexCommand('primary', $columns, $name, $algorithm)|
|unique|  indexCommand('unique', $columns, $name, $algorithm)|
|index| indexCommand('index', $columns, $name, $algorithm)|
|spatialIndex|indexCommand('spatialIndex', $columns, $name)|
|foreign| indexCommand('foreign', $columns, $name)|
|dropIndex| dropIndexCommand('dropIndex', 'index', $index)|
|dropPrimary|  dropIndexCommand('dropPrimary', 'primary', $index)|
|dropUnique|  dropIndexCommand('dropUnique', 'unique', $index)|
|dropForeign| dropIndexCommand('dropForeign', 'foreign', $index)|

> 说明：如果将某单列定义为某种索引，可以直接按修饰命令的方式定义，如 $table->string('name')->unique();但这种方式不适合外键

###3.4  命令
每一个命令都是一个流(Flent)对象,流对象中name必须指定,比如 create()就是在commands增加一个name为create的流对象
处理命令的有: getCommands获取所有的命令,addCommand,createCommand
判断是否有create命令creating
对表create,drop,dropIfExists,rename
对表中的列,dropColumn,renameColumn

#四： 熟悉迁移命令

1. 生成迁移文件
`php artisan make:migration create_wang04_table --path=database/migrations2`
`php artisan make:migration create_wang05_table --path=database/migrations2`
2. 运行迁移，凡在指定路径下有的文件名，没有出现在仓库表的migration字段中的，就会执行
 `php artisan migrate --database=mysql2 --path=database/migrations2`
3. 撤销上一步迁移，只一步啊，上一次migrate有多少变动，如生成5张表，这一次会全部撤销
`php artisan migrate:rollback --database=mysql2 --path=database/migrations2`
4. 撤销全部，清空仓库表,除了仓库表外，所有的表都删除
`php artisan migrate:reset --database=mysql2 --path=database/migrations2`
5. 删除所有的表，再重新运行一个迁移，它的效率比6要高
`php artisan migrate:fresh --database=mysql2 --path=database/migrations2`
6. 全部撤销并重新运行迁移，与fresh的差别是它是一步一步的撤销，原来假设有78步，那么就一步一步撤，再一步一步运行,它俩有一个共同点是，最后只算一批，也就是所有的迁移是一批，仓库表中的batch值全部为1
`php artisan migrate:refresh --database=mysql2 --path=database/migrations2`
7. 显示当前迁移的状态
`php artisan migrate:status --database=mysql2 --path=database/migrations2`

|id|migration|batch|
|----------|-----------|
|6|	2017_11_02_091523_wang01|	1|
|7|	2017_11_02_103519_create_json_table|	1|
|8|	2017_11_02_142016_create_wang01_table|	1|
|9	|2017_11_02_142040_create_wang02_table|	1|
|10|	2017_11_02_142103_create_wang03_table|	1|
|11|	2017_11_02_142928_create_wang04_table|	2|
|12	|2017_11_02_143043_create_wang05_table|	3|
|13|2017_11_02_143104_create_wang06_table	|3|

> migate命令会将没有运行过的迁移文件运行，有几个执行几个
> migrate:rollback 只会撤销最后一批，如上面会撤销第3批生成的两个表
> migrate:install 用来生成仓库表，不用执行，系统会自动执行，且只会执行一次，该表不会被撤销
> migrate:status 显示哪些迁移文件还未运行,N表示还未运行
> migrate:reset        Rollback all database migrations
> migrate:fresh        会删除所有已经生成的表，并重新运行所有的迁移
> migrate:refresh      Reset and re-run all migrations



| Ran? | Migration                             |
|------|---------------------------------------|
| Y    | 2017_11_02_091523_wang01              |
| Y    | 2017_11_02_103519_create_json_table   |
| Y    | 2017_11_02_142016_create_wang01_table |
| Y    | 2017_11_02_142040_create_wang02_table |
| Y    | 2017_11_02_142103_create_wang03_table |
| Y    | 2017_11_02_142928_create_wang04_table |
| N    | 2017_11_02_143043_create_wang05_table |
| N    | 2017_11_02_143104_create_wang06_table |

#五: 迁移文件发布
数据迁移文件一般与模块放在一起，比如你开发了一个模块，有自己的数据表，当别人使用你的模块时，得先将你的数据迁移文件拷贝到应用的指定文件夹下。这样很不方便，所以得在模块的服务提供商文件中写好发布方法
```
<?php
namespace Wang\Providers;
use Illuminate\Support\ServiceProvider;
class ModuleServiceProvider extends ServiceProvider{
    protected $defer = false;
    public function boot(){
          $this->publishMigrations(); 
    }
    private function publishMigrations()    {
        $this->publishes([__DIR__ . '/../../migrations/' => base_path('database/migrations2')], 'migrations');
    }
    //意思就是在
   
}
```
在使用的应用中，只要使用 php artisan vendor:publish 就会将模块中的迁移文件拷贝到应用下相应目录中

### 如何给应用外模块生成迁移文件？
答：一般的做法是，先在当前应用中生成，经测试没问题后，可直接拷贝到模块中


#六: 多数据库版
###6.1 新建数据库laravel_study
>字符集utf8 -- UTF-8 Unicode,排序规则:utf8_general_ci

###6.2 配置第二个数据库 
D:\03www2018\study\myblog\config\database.php增加如下
```
 //第2个数据库连接
        'mysql2' => [
            'driver'    => 'mysql',
            'host'      => '127.0.0.1',
            'database'  => 'laravel_study',
            'username'  => 'zhangxueyou',
            'password'  => 'liudehua',
            'charset'   => 'utf8',
            'collation' => 'utf8_unicode_ci',
            'prefix'    => '',
        ],
```
###6.3 新增加一个用户
```
GRANT ALL PRIVILEGES ON study_laravel.* TO zhangxueyou@localhost IDENTIFIED BY 'liudehua' WITH GRANT OPTION;
```
###6.4 重启mysql
`vagrant@homestead:~/abcde/study/myblog$ sudo service mysql restart`

###6.5 新建迁移文件
> 先建好文件夹migrations2
vagrant@homestead:~/abcde/study/myblog$ php artisan make:migration wang01 --create=study01 --path=database/migrations2

###6.6 编辑迁移文件
```
<?php
use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;
class Wang01 extends Migration{

    public function up(){
        Schema::connection('mysql2')->create('study01', function (Blueprint $table) {
            $table->increments('id');
            $table->string('username',25);
            $table->dateTime('mydatetime')->default(new \Illuminate\Database\Query\Expression('CURRENT_TIMESTAMP'));
            $table->timestamp('register_time')->useCurrent();
            //$table->timestamp('register_time',3);
            $table->timestamps();
        });
    }
    public function down() {
        Schema::connection('mysql2')->dropIfExists('study01');
    }
}

```
> 上面要注意默认值为CURRENT_TIMESTAMP的写法，如果不这样写，系统为了安全会加上单引号，另外该msyql常量只在mysql版本高于5.6才有

###6.7 迁移和回滚
```
vagrant@homestead:~/abcde/study/myblog$ php artisan migrate --database=mysql2 --path=database/migrations2
vagrant@homestead:~/abcde/study/myblog$ php artisan migrate:rollback --database=mysql2 --path=database/migrations2

```


