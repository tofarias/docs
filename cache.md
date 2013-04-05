# Cache

- [Configuração](#configuration)
- [Modo de usar](#cache-usage)
- [Incrementos & Decrementos](#increments-and-decrements)
- [Database Cache](#database-cache)

<a name="configuration"></a>
## Configuração

Laravel oferece uma API unificada para vários sistemas de cache diferentes. A configuração de cache está localizada em `app/config/cache.php`. Nesse arquivo você pode escificar qual driver você quer usar por padrão na sua aplicação. Laravel também suporta os populares sistemas de cache como o [Memcached](http://memcached.org) e [Redis](http://redis.io).

The cache configuration file also contains various other options, which are documented within the file, so make sure to read over these options. By default, Laravel is configured to use the `file` cache driver, which stores the serialized, cached objects in the filesystem. For larger applications, it is recommended that you use an in-memory cache such as Memcached or APC.

<a name="cache-usage"></a>
## Cache Usage

**Storing An Item In The Cache**

	Cache::put('key', 'value', $minutes);

**Storing An Item In The Cache If It Doesn't Exist**

	Cache::add('key', 'value', $minutes);

**Retrieving An Item From The Cache**

	$value = Cache::get('key');

**Retrieving An Item Or Returning A Default Value**

	$value = Cache::get('key', 'default');

	$value = Cache::get('key', function() { return 'default'; });

**Storing An Item In The Cache Permanently**

	Cache::forever('key', 'value');

Sometimes you may wish to retrieve an item from the cache, but also store a default value if the requested item doesn't exist. You may do this using the `Cache::remember` method:

	$value = Cache::remember('users', $minutes, function()
	{
		return DB::table('users')->get();
	});

You may also combine the `remember` and `forever` methods:

	$value = Cache::rememberForever('users', function()
	{
		return DB::table('users')->get();
	});

Note that all items stored in the cache are serialized, so you are free to store any type of data.

**Removing An Item From The Cache**

	Cache::forget('key');

<a name="increments-and-decrements"></a>

All drivers except `file` and `database` support the `increment` and `decrement` operations:

**Incrementing A Value**

	Cache::increment('key');

	Cache::increment('key', $amount);

**Decrementing A Value**

	Cache::decrement('key');

	Cache::decrement('key', $amount);

<a name="database-cache"></a>
## Database Cache

When using the `database` cache driver, you will need to setup a table to contain the cache items. Below is an example `Schema` declaration for the table:

	Schema::create('cache', function($t)
	{
		$t->string('key')->unique();
		$t->text('value');
		$t->integer('expiration');
	});
