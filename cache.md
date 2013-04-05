# Cache

- [Configuração](#configuration)
- [Modo de usar](#cache-usage)
- [Incrementos & Decrementos](#increments-and-decrements)
- [Database Cache](#database-cache)

<a name="configuration"></a>
## Configuração

Laravel oferece uma API unificada para vários sistemas de cache diferentes. A configuração de cache está localizada em `app/config/cache.php`. Nesse arquivo você pode escificar qual driver você quer usar por padrão na sua aplicação. Laravel também suporta os populares sistemas de cache como o [Memcached](http://memcached.org) e [Redis](http://redis.io).

The cache configuration file also contains various other options, which are documented within the file, so make sure to read over these options. By default, Laravel is configured to use the `file` cache driver, which stores the serialized, cached objects in the filesystem. For larger applications, it is recommended that you use an in-memory cache such as Memcached or APC.

O arquivo de configuração do cache também contém várias outras opções, que são documentadas no próprio arquivo, por isso certifique-se de ler sobre essas opções. Por padrão, o Laravel está configurado para usar o driver `file` de cache, que armazena os objetos serializados em cache no sistema de arquivos. Para aplicações maiores, recomenda-se o uso de um cache em memória, tais como Memcached ou APC.

<a name="cache-usage"></a>
## Modo de usar

**Guardando um item em cache**

	Cache::put('key', 'value', $minutes);

**Guardando um item em cache se ele não existe**

	Cache::add('key', 'value', $minutes);

**Buscando um item da Cache**

	$value = Cache::get('key');

**Buscando um item da cache, ou retornando um valor padrão caso ele não exista**

	$value = Cache::get('key', 'default');

	$value = Cache::get('key', function() { return 'default'; });

**Guardando um item em cache permanentemente**

	Cache::forever('key', 'value');

Pode ser que você queira buscar um item da cache, mas também, salvar um valor padrão caso o item procurado não exista.
Você pode fazer isso usando o método `Cache::remember`:

	$value = Cache::remember('users', $minutes, function()
	{
		return DB::table('users')->get();
	});

Você também pode combinar os métodos `remember` e `forever`:

	$value = Cache::rememberForever('users', function()
	{
		return DB::table('users')->get();
	});

Note que todos os itens guardados em cache são serializados, isso significa que você pode serializar todo o tipo de dado.

**Removendo um item da Cache**

	Cache::forget('key');

<a name="increments-and-decrements"></a>

Todos os drivers com excessão de `file` e `database` suportam as operações `increment` and `decrement`:

**Incrementando um valor**

	Cache::increment('key');

	Cache::increment('key', $amount);

**Decrementando um valor**

	Cache::decrement('key');

	Cache::decrement('key', $amount);

<a name="database-cache"></a>
## Database Cache

Quando estiver usando driver `database`, você precisará configurar uma tabela que receberá os itens. Abaixo um exemplo de um `Schema` para a tabela:

	Schema::create('cache', function($t)
	{
		$t->string('key')->unique();
		$t->text('value');
		$t->integer('expiration');
	});
