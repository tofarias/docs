# Eloquent ORM

- [Introdução](#introduction)
- [Utilização Básica](#basic-usage)
- [Mass Assignment](#mass-assignment)
- [Insert, Update, Delete](#insert-update-delete)
- [Timestamps](#timestamps)
- [Query Scopes](#query-scopes)
- [Relacionamentos](#relationships)
- [Eager Loading](#eager-loading)
- [Inserindo Models Relacionados](#inserting-related-models)
- [Touching Parent Timestamps](#touching-parent-timestamps)
- [Trabalhando com Tabelas Dinâmicas](#working-with-pivot-tables)
- [Coleções](#collections)
- [Accessors & Mutators](#accessors-and-mutators)
- [Eventos de Model](#model-events)
- [Convertendo Arrays / JSON](#converting-to-arrays-or-json)

<a name="introduction"></a>
## Introdução

O Eloquente ORM incluído com Laravel proporciona uma bela, simples implementação ActiveRecord para trabalhar com seu banco de dados. Cada tabela de banco de dados tem um "Model" correspondente, que é usado para interagir com suas respectivas tabelas.

Antes de começar, sera necessário configurar um banco de dados em `app/config/database.php`.

<a name="basic-usage"></a>
## Utilização Básica

Para começar, crie um Eloquent model. Models geralmente encontram - se no diretório `app/models`, mas você é livre para colocá-los em qualquer lugar que pode ser carregado automaticamente de acordo com seu arquivo `composer.json`.

**Definindo um Eloquent Model**

	class User extends Eloquent {}

Note que não disse ao Eloquent que tabela a ser usada em nosso model `User`. O nome da classe será usado como o nome da tabela em letras minúsculas, a menos que um outro nome seja especificado. Assim, neste caso, Eloquent assumirá o `User` como `users`. Você pode especificar uma tabela personalizada, defina uma propriedade `tabela` em seu model:

	class User extends Eloquent {

		protected $table = 'my_users';

	}

Eloquent também assumira que cada tabela tem uma coluna `primary key` de nome `id`. Você pode definir uma propriedade `primaryKey` para substituir esta convencional. Do mesmo modo, você pode definir uma propriedade `connection` para substituir o nome do banco de dados que deve ser utilizado quando se utilizar o model.

> **Note:** Eloquent também assume que cada tabela tem uma coluna primary key nomeada `id`. Você pode definir uma propriedade `primaryKey` para substituir esta convencional.

Uma vez com o model definido, você esta pronto para começar a recuperar e criar registros em sua tabela. Note que você precisará colocar as colunas `updated_at` e `created_at` em sua tabela por padrão. Se você não quiser ter estas colunas automaticas mantidas, defina a propriedade `$timestamps` em seu model como `false`.

**Recuperando Todos Models**

	$users = User::all();

**Recuperando Um Registro pela Chave Primária**

	$user = User::find(1);

	var_dump($user->name);

> **Note:** Todos metodos dsponíveis em [query builder](/docs/queries) estão tabém dsponíveis quando consultar Eloquent models.

**Recuperando um Model através da Chave Primária ou Lançar uma Exceção**

Algumas vezes você pode desejar lançar uma exceção se um model não for encontrado, permitindo você capturar as exceções usando um `App::error` manipular e exibir um página 404.

	$model = User::findOrFail(1);

Para registrar o manipulador de erro, chame `ModelNotFoundException`

	use Illuminate\Database\Eloquent\ModelNotFoundException;

	App::error(function(ModelNotFoundException $e)
	{
		return Response::make('Not Found', 404);
	});

**Consulta Usando Eloquent Models**

	$users = User::where('votes', '>', 100)->take(10)->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

Claro, você pode também usar a construção de consultas a funções aggregates.

**Eloquent Aggregates**

	$count = User::where('votes', '>', 100)->count();

<a name="mass-assignment"></a>
## Atribuição em massa

Quando criamos um novo model, you passa um array de atributos para o construtor do model. Estes atributos são atribuidos ao model via mass-assignment. Isto é conveniente; contudo, pode ser um **sério** problema de segurança quando simplesmente passamos a entrada de usuário diretamente no model. Se entrada de usuário é simplesmente diretamente em um model, o usuário se torna livre para modificar **qualquer** e **todos** atributos do model. Por esta razão, todos models Eloquent por padrão são protegidas contra mass-assignment (atribuição em massa).

Para começar, defina as propriedades `fillable` ou `guarded` em seus model.

A propriedade `fillable` especifica que os atributos devem ser mass-assignable (nesse caso aceitar atribuição em massa). Isso pode ser definido a nível de classe ou instacia.

**Definindo Atributos Fillable em Um Model**

	class User extends Eloquent {

		protected $fillable = array('first_name', 'last_name', 'email');

	}

Neste exemplo' apenas três atributos serão listados como mass-assignable.

O inverso de `fillable` é `guarded`, e serve como uma "lista-negra" em vez de um "lista-branca":

**Definindo Atributos Guarded em Um Model**

	class User extends Eloquent {

		protected $guarded = array('id', 'password');

	}

No exemplo acima, os atributos `id` e `password` não podem ser "atribuidos em massa". Todos outros (ou seja não expecificado com $guarded) poderão ser "atribuidos em massa". Você também pode bloquar todos os atributos de atribuição em massa usando o metodo guard:

**Bloqueando Todos Atributos de Atribuição em massa**

	protected $guarded = array('*');

<a name="insert-update-delete"></a>
## Insert, Update, Delete

Para criar um novo registro a partir de um model, basta criar uma nova instancia do model e chamar o metodo `save`.

**SSalvando Um Novo Model**

	$user = new User;

	$user->name = 'John';

	$user->save();

> **Note:** Geralmente, seu model Eloquent terá chaves auto incrementadas. Contudo, se você deseja especificar sua própria chave, defina a propriedade `incrementing` do seu model como `false`.

Você também pode usar o metodo `create` para salvar um model em uma única linha. O model inserido sera retornado para você a partir do metodo. Contudo, antes de fazer isso, você vai precisar expecíficar ou um atributo `fillable` ou `guarded` no model, como todos models Eloquent protegidos contra mass-assignment (atribuição em massa).

**Definindo O Atributo Guarded no model**

	class User extends Eloquent {

		protected $guarded = array('id', 'account_id');

	}

**Usando o Metodo Create no model**

	$user = User::create(array('name' => 'John'));

Para atualizar um model, você pode recuperá - lo, alterar um atributo, e usar o metodo `save`:

**Atualizando Um Model Recuperado**

	$user = User::find(1);

	$user->email = 'john@foo.com';

	$user->save();

Você também pode execultar atualizações como consultas em conjuntos do model:

	$affectedRows = User::where('votes', '>', 100)->update(array('status' => 2));

To delete a model, simply call the `delete` method on the instance:

**Deleting An Existing Model**

	$user = User::find(1);

	$user->delete();

**Deleting An Existing Model By Key**

	User::destroy(1);

	User::destroy(1, 2, 3);

Of course, you may also run a delete query on a set of models:

	$affectedRows = User::where('votes', '>', 100)->delete();

If you wish to simply update the timestamps on a model, you may use the `touch` method:

**Updating Only The Model's Timestamps**

	$user->touch();

<a name="timestamps"></a>
## Timestamps

By default, Eloquent will maintain the `created_at` and `updated_at` columns on your database table automatically. Simply add these `datetime` columns to your table and Eloquent will take care of the rest. If you do not wish for Eloquent to maintain these columns, add the following property to your model:

**Disabling Auto Timestamps**

	class User extends Eloquent {

		protected $table = 'users';

		public $timestamps = false;

	}

If you wish to customize the format of your timestamps, you may override the `freshTimestamp` method in your model:

**Providing A Custom Timestamp Format**

	class User extends Eloquent {

		public function freshTimestamp()
		{
			return time();
		}

	}

<a name="query-scopes"></a>
## Query Scopes

Scopes allow you to easily re-use query logic in your models. To define a scope, simply prefix a model method with `scope`:

**Defining A Query Scope**

	class User extends Eloquent {

		public function scopePopular($query)
		{
			return $query->where('votes', '>', 100);
		}

	}

**Utilizing A Query Scope**

	$users = User::popular()->orderBy('created_at')->get();

<a name="relationships"></a>
## Relationships

Of course, your database tables are probably related to one another. For example, a blog post may have many comments, or an order could be related to the user who placed it. Eloquent makes managing and working with these relationships easy. Laravel supports four types of relationships:

- [One To One](#one-to-one)
- [One To Many](#one-to-many)
- [Many To Many](#many-to-many)
- [Polymorphic Relations](#polymorphic-relations)

<a name="one-to-one"></a>
### One To One

A one-to-one relationship is a very basic relation. For example, a `User` model might have one `Phone`. We can define this relation in Eloquent:

**Defining A One To One Relation**

	class User extends Eloquent {

		public function phone()
		{
			return $this->hasOne('Phone');
		}

	}

The first argument passed to the `hasOne` method is the name of the related model. Once the relationship is defined, we may retrieve it using Eloquent's dynamic properties:

	$phone = User::find(1)->phone;

The SQL performed by this statement will be as follows:

	select * from users where id = 1

	select * from phones where user_id = 1

Take note that Eloquent assumes the foreign key of the relationship based on the model name. In this case, `Phone` model is assumed to use a `user_id` foreign key. If you wish to override this convention, you may pass a second argument to the `hasOne` method:

	return $this->hasOne('Phone', 'custom_key');

To define the inverse of the relationship on the `Phone` model, we use the `belongsTo` method:

**Defining The Inverse Of A Relation**

	class Phone extends Eloquent {

		public function user()
		{
			return $this->belongsTo('User');
		}

	}

<a name="one-to-many"></a>
### One To Many

An example of a one-to-many relation is a blog post that "has many" comments. We can model this relation like so:

	class Post extends Eloquent {

		public function comments()
		{
			return $this->hasMany('Comment');
		}

	}

Now we can access the post's comments through the dynamic property:

	$comments = Post::find(1)->comments;

If you need to add further constraints to which comments are retrieved, you may call the `comments` method and continue chaining conditions:

	$comments = Post::find(1)->comments()->where('title', '=', 'foo')->first();

Again, you may override the conventional foreign key by passing a second argument to the `hasMany` method:

	return $this->hasMany('Comment', 'custom_key');

To define the inverse of the relationship on the `Comment` model, we use the `belongsTo` method:

**Defining The Inverse Of A Relation**

	class Comment extends Eloquent {

		public function post()
		{
			return $this->belongsTo('Post');
		}

	}

<a name="many-to-many"></a>
### Many To Many

Many-to-many relations are a more complicated relationship type. An example of such a relationship is a user with many roles, where the roles are also shared by other users. For example, many users may have the role of "Admin". Three database tables are needed for this relationship: `users`, `roles`, and `role_user`. The `role_user` table is derived from the alphabetical order of the related model names, and should have `user_id` and `role_id` columns.

We can define a many-to-many relation using the `belongsToMany` method:

	class User extends Eloquent {

		public function roles()
		{
			return $this->belongsToMany('Role');
		}

	}

Now, we can retrieve the roles through the `User` model:

	$roles = User::find(1)->roles;

If you would like to use an unconventional table name for your pivot table, you may pass it as the second argument to the `belongsToMany` method:

	return $this->belongsToMany('Role', 'user_roles');

You may also override the conventional associated keys:

	return $this->belongsToMany('Role', 'user_roles', 'user_id', 'foo_id');

Of course, you may also define the inverse of the relationship on the `Role` model:

	class Role extends Eloquent {

		public function users()
		{
			return $this->belongsToMany('User');
		}

	}

<a name="polymorphic-relations"></a>
### Polymorphic Relations

Polymorphic relations allow a model to belong to more than one other model, on a single association. For example, you might have a photo model that belongs to either a staff model or an order model. We would define this relation like so:

	class Photo extends Eloquent {

		public function imageable()
		{
			return $this->morphTo();
		}

	}

	class Staff extends Eloquent {

		public function photos()
		{
			return $this->morphMany('Photo', 'imageable');
		}

	}

	class Order extends Eloquent {

		public function photos()
		{
			return $this->morphMany('Photo', 'imageable');
		}

	}

Now, we can retrieve the photos for either a staff member or an order:

**Retrieving A Polymorphic Relation**

	$staff = Staff::find(1);

	foreach ($staff->photos as $photo)
	{
		//
	}

However, the true "polymorphic" magic is when you access the staff or order from the `Photo` model:

**Retrieving The Owner Of A Polymorphic Relation**

	Photo::find(1);

	$imageable = $photo->imageable;

The `imageable` relation on the `Photo` model will return either a `Staff` or `Order` instance, depending on which type of model owns the photo.

To help understand how this works, let's explore the database structure for a polymorphic relation:

**Polymorphic Relation Table Structure**

	staff
		id - integer
		name - string

	orders
		id - integer
		price - integer

	photos
		id - integer
		path - string
		imageable_id - integer
		imageable_type - string

The key fields to notice here are the `imageable_id` and `imageable_type` on the `photos` table. The ID will contain the ID value of, in this example, the owning staff or order, while the type will contain the class name of the owning model. This is what allows the ORM to determine which type of owning model to return when accessing the `imageable` relation.

<a name="eager-loading"></a>
## Eager Loading

Eager loading exists to alleviate the N + 1 query problem. For example, consider a `Book` model that is related to `Author`. The relationship is defined like so:

	class Book extends Eloquent {

		public function author()
		{
			return $this->belongsTo('Author');
		}

	}

Now, consider the following code:

	foreach (Book::all() as $book)
	{
		echo $book->author->name;
	}

This loop will execute 1 query to retrieve all of the books on the table, then another query for each book to retrieve the author. So, if we have 25 books, this loop would run 26 queries.

Thankfully, we can use eager loading to drastically reduce the number of queries. The relationships that should be eager loaded may be specified via the `with` method:

	foreach (Book::with('author')->get() as $book)
	{
		echo $book->author->name;
	}

In the loop above, only two queries will be executed:

	select * from books

	select * from authors where id in (1, 2, 3, 4, 5, ...)

Wise use of eager loading can drastically increase the performance of your application.

Of course, you may eager load multiple relationships at one time:

	$books = Book::with('author', 'publisher')->get();

You may even eager load nested relationships:

	$books = Book::with('author.contacts')->get();

In the example above, the `author` relationship will be eager loaded, and the author's `contacts` relation will also be loaded.

### Eager Load Constraints

Sometimes you may wish to eager load a relationship, but also specify a condition for the eager load. Here's an example:

	$users = User::with(array('posts' => function($query)
	{
		$query->where('title', 'like', '%first%');
	}))->get();

In this example, we're eager loading the user's posts, but only if the post's title column contains the word "first".

### Lazy Eager Loading

It is also possible to eagerly load related models directly from an already existing model collection. This may be useful when dynamically deciding whether to load related models or not, or in combination with caching.

	$books = Book::all();

	$books->load('author', 'publisher');

<a name="inserting-related-models"></a>
## Inserting Related Models

You will often need to insert new related models. For example, you may wish to insert a new comment for a post. Instead of manually setting the `post_id` foreign key on the model, you may insert the new comment from its parent `Post` model directly:

**Attaching A Related Model**

	$comment = new Comment(array('message' => 'A new comment.'));

	$post = Post::find(1);

	$comment = $post->comments()->save($comment);

In this example, the `post_id` field will automatically be set on the inserted comment.

### Inserting Related Models (Many To Many)

You may also insert related models when working with many-to-many relations. Let's continue using our `User` and `Role` models as examples. We can easily attach new roles to a user using the `attach` method:

**Attaching Many To Many Models**

	$user = User::find(1);

	$user->roles()->attach(1);

You may also pass an array of attributes that should be stored on the pivot table for the relation:

	$user->roles()->attach(1, array('expires' => $expires));

Of course, the opposite of `attach` is `detach`:

	$user->roles()->detach(1);

You may also use the `sync` method to attach related models. The `sync` method accepts an array of IDs to place on the pivot table. After this operation is complete, only the IDs in the array will be on the intermediate table for the model:

**Using Sync To Attach Many To Many Models**

	$user->roles()->sync(array(1, 2, 3));

Sometimes you may wish to create a new related model and attach it in a single command. For this operation, you may use the `save` method:

	$role = new Role(array('name' => 'Editor'));

	User::find(1)->roles()->save($role);

In this example, the new `Role` model will be saved and attached to the user model. You may also pass an array of attributes to place on the joining table for this operation:

	User::find(1)->roles()->save($role, array('expires' => $expires));

<a name="touching-parent-timestamps"></a>
## Touching Parent Timestamps

When a model `belongsTo` another model, such as a `Comment` which belongs to a `Post`, it is often helpful to update the parent's timestamp when the child model is updated. For example, when a `Comment` model is updated, you may want to automatically touch the `updated_at` timestamp of the owning `Post`. Eloquent makes it easy. Just add a `touches` property containing the names of the relationships to the child model:

	class Comment extends Eloquent {

		protected $touches = array('post');

		public function post()
		{
			return $this->belongsTo('Post');
		}

	}

Now, when you update a `Comment`, the owning `Post` will have its `updated_at` column updated:

	$comment = Comment::find(1);

	$comment->text = 'Edit to this comment!';

	$comment->save();

<a name="working-with-pivot-tables"></a>
## Working With Pivot Tables

As you have already learned, working with many-to-many relations requires the presence of an intermediate table. Eloquent provides some very helpful ways of interacting with this table. For example, let's assume our `User` object has many `Role` objects that it is related to. After accessing this relationship, we may access the `pivot` table on the models:

	$user = User::find(1);

	foreach ($user->roles as $role)
	{
		echo $role->pivot->created_at;
	}

Notice that each `Role` model we retrieve is automatically assigned a `pivot` attribute. This attribute contains a model representing the intermediate table, and may be used as any other Eloquent model.

By default, only the keys will be present on the `pivot` object. If your pivot table contains extra attributes, you must specify them when defining the relationship:

	return $this->belongsToMany('Role')->withPivot('foo', 'bar');

Now the `foo` and `bar` attributes will be accessible on our `pivot` object for the `Role` model.

If you want your pivot table to have automatically maintained `created_at` and `updated_at` timestamps, use the `withTimestamps` method on the relationship definition:

	return $this->belongsToMany('Role')->withTimestamps();

To delete all records on the pivot table for a model, you may use the `delete` method:

**Deleting Records On A Pivot Table**

	User::find(1)->roles()->detach();

Note that this operation does not delete records from the `roles` table, but only from the pivot table.

<a name="collections"></a>
## Collections

All multi-result sets returned by Eloquent either via the `get` method or a relationship return an Eloquent `Collection` object. This objects implements the `IteratorAggregate` PHP interface so it can be iterated over like an array. However, this object also has a variety of other helpful methods for working with result sets.

For example, we may determine if a result set contains a given primary key using the `contains` method:

**Checking If A Collection Contains A Key**

	$roles = User::find(1)->roles;

	if ($roles->contains(2))
	{
		//
	}

Collections may also be converted to an array or JSON:

	$roles = User::find(1)->roles->toArray();

	$roles = User::find(1)->roles->toJson();

If a collection is cast to a string, it will be returned as JSON:

	$roles = (string) User::find(1)->roles;

Eloquent collections also contain a few helpful methods for looping and filtering the items they contain:

**Iterating & Filtering Collections**

	$roles = $user->roles->each(function($role)
	{

	});

	$roles = $user->roles->filter(function($role)
	{

	});

Sometimes, you may wish to return a custom Collection object with your own added methods. You may specify this on your Eloquent model by overriding the `newCollection` method:

**Returning A Custom Collection Type**

	class User extends Eloquent {

		public function newCollection(array $models = array())
		{
			return new CustomCollection($models);
		}

	}

**Applying callback to the objects in a collection**

	$roles = User::find(1)->roles;
	
	$roles->each(function($role)
	{
		//	
	});
	

<a name="accessors-and-mutators"></a>
## Accessors & Mutators

Eloquent provides a convenient way to transform your model attributes when getting or setting them. Simplify define a `getFooAttribute` method on your model to declare an accessor. Keep in mind that the methods should follow camel-casing, even though your database columns are snake-case:

**Defining An Accessor**

	class User extends Eloquent {

		public function getFirstNameAttribute($value)
		{
			return ucfirst($value);
		}

	}

In the example above, the `first_name` column has an accessor. Note that the value of the attribute is passed to the accessor.

Mutators are declared in a similar fashion:

**Defining A Mutator**

	class User extends Eloquent {

		public function setFirstNameAttribute($value)
		{
			$this->attributes['first_name'] = strtolower($value);
		}

	}

<a name="model-events"></a>
## Model Events

Eloquent models fire several events, allowing you to hook into various points in the model's lifecycle using the following methods: `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`. If `false` is returned from the `creating`, `updating`, or `saving` events, the action will be cancelled:

**Cancelling Save Operations Via Events**

	User::creating(function($user)
	{
		if ( ! $user->isValid()) return false;
	});

Eloquent models also contain a static `boot` method, which may provide a convenient place to register your event bindings.

**Setting A Model Boot Method**

	class User extends Eloquent {

		public static function boot()
		{
			parent::boot();

			// Setup event bindings...
		}

	}

<a name="converting-to-arrays-or-json"></a>
## Converting To Arrays / JSON

When building JSON APIs, you may often need to convert your models and relationships to arrays or JSON. So, Eloquent includes methods for doing so. To convert a model and its loaded relationship to an array, you may use the `toArray` method:

**Converting A Model To An Array**

	$user = User::with('roles')->first();

	return $user->toArray();

Note that entire collections of models may also be converted to arrays:

	return User::all()->toArray();

To convert a model to JSON, you may use the `toJson` method:

**Converting A Model To JSON**

	return User::find(1)->toJson();

Note that when a model or collection is cast to a string, it will be converted to JSON, meaning you can return Eloquent objects directly from your application's routes!

**Returning A Model From A Route**

	Route::get('users', function()
	{
		return User::all();
	});

Sometimes you may wish to limit the attributes that are included in your model's array or JSON form, such as passwords. To do so, add a `hidden` property definition to your model:

**Hiding Attributes From Array Or JSON Conversion**

	class User extends Eloquent {

		protected $hidden = array('password');

	}
