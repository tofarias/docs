# Controladores

- [Controladores Básicos](#basic-controllers)
- [Filtros de Controladores](#controller-filters)
- [Controladores RESTful](#restful-controllers)
- [Controladores de Recursos](#resource-controllers)
- [Manipulação de Métodos Ausentes](#handling-missing-methods)

<a name="basic-controllers"></a>
## Controladores Básicos

Em vez de definir toda a sua lógica de rotas no arquivo de `routes.php`, você pode querer organizar este comportamento usando classes Controladores. Controladores podem agrupar a lógica relacionada a rota em uma classe, além de tirar vantagem de recursos mais avançados em Laravel, tais como [injeção de dependência](/docs/ioc) automática.

Os controladores são normalmente armazenados no diretório `app/controllers`, e este diretório já vem predefinidp na opção `classmap` de seu arquivo `composer.json`.

Abaixo segue um exemplo de uma classe de Controlador Básico:


	class UserController extends BaseController {

		/**
		 * Mostrar o perfil de um usuário.
		 */
		public function showProfile($id)
		{
			$user = User::find($id);

			return View::make('user.profile', array('user' => $user));
		}

	}

Todos controladores deve extender a classe `BaseController`. O `BaseController` é armazenado no diretório `app/controllers`, onde a lógica compartilhada em controladores pode ser colocada. A classe `BaseController` extende a classe `Controller` da framework. Agora podemos encaminhar a ação do controlador:

	Route::get('user/{id}', 'UserController@showProfile');

Se você escolher organizar seu controlador usando PHP namespaces, basta usar o nome da classe com o namespace correspondente para definir a rota:

	Route::get('foo', 'Namespace\FooController@method');

Você também pode especificar nomes em rotas de controlador:

	Route::get('foo', array('uses' => 'FooController@method',
											'as' => 'name'));

Para gerar uma URL para uma ação de controlador, você pode usar o método `URL::action`:

	$url = URL::action('FooController@method');

Você pode acessar o nome da ação do controlador que está sendo executado com o método `currentRouteAction`:

	$action = Route::currentRouteAction();

<a name="controller-filters"></a>
## Filtros de Controladores

Os [filtros](/docs/routing#route-filters) podem ser especificados em rotas controladoras semelhantes às rotas regulares:

	Route::get('profile', array('before' => 'auth',
				'uses' => 'UserController@showProfile'));

No entanto, você também pode especificar filtros de dentro de seu controlador:

	class UserController extends BaseController {

		/**
		 * Instanciar uma nova instância de UserController.
		 */
		public function __construct()
		{
			$this->beforeFilter('auth');

			$this->beforeFilter('csrf', array('on' => 'post'));

			$this->afterFilter('log', array('only' =>
								array('fooAction', 'barAction')));
		}

	}

Você também pode especificar filtros de controladorers usando funções anônimas (Closures):

	class UserController extends BaseController {

		/**
		 * Instanciar uma nova instância de UserController.
		 */
		public function __construct()
		{
			$this->beforeFilter(function()
			{
				//
			});
		}

	}

<a name="restful-controllers"></a>
## Controladores RESTful

Laravel permite que você facilmente defina uma única rota para lidar com cada ação em um controlador usando simples, convenções de nomenclatura de REST. Primeiro defina uma rota usando o método `Route::controller`:


**Definindo um Controlador RESTful**

	Route::controller('users', 'UserController');

O método `controller` aceita dois argumentos. O primeiro é a URI base que controlador manipula, enquanto o segundo é o nome da classe do controlador. Em seguida, basta adicionar métodos em seu controlador, prefixado com o verbo HTTP que eles respondem:

	class UserController extends BaseController {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

	}

O método `index` irá responder ao URI raiz implementado pelo controlador que, neste caso, é `users`.

If your controller action contains multiple words, you may access the action using "dash" syntax in the URI. For example, the following controller action on our `UserController` would respond to the `users/admin-profile` URI:
Se a ação do sue controlador contém múltiplas palavras, você pode acessar a ação usando traços na sintaxe na URI. Por exemplo, a ação de controlador no nosso `UserController` responderia a URI `users/admin-profile` da seguinte forma:

	public function getAdminProfile() {}

<a name="resource-controllers"></a>
## Resource Controllers

Resource controllers make it easier to build RESTful controllers around resources. For example, you may wish to create a controller that manages "photos" stored by your application. Using the `controller:make` command via the Artisan CLI and the `Route::resource` method, we can quickly create such a controller.

To create the controller via the command line, execute the following command:

	php artisan controller:make PhotoController

Now we can register a resourceful route to the controller:

	Route::resource('photo', 'PhotoController');

This single route declaration creates multiple routes to handle a variety of RESTful actions on the photo resource. Likewise, the generated controller will already have stubbed methods for each of these actions with notes informing you which URIs and verbs they handle.

**Actions Handled By Resource Controller**

Verb      | Path                  | Action
----------|-----------------------|--------------
GET       | /resource             | index
GET       | /resource/create      | create
POST      | /resource             | store
GET       | /resource/{id}        | show
GET       | /resource/{id}/edit   | edit
PUT/PATCH | /resource/{id}        | update
DELETE    | /resource/{id}        | destroy

Sometimes you may only need to handle a subset of the resource actions:

	php artisan controller:make PhotoController --only=index,show

	php artisan controller:make PhotoController --except=index

And, you may also specify a subset of actions to handle on the route:

	Route::resource('photo', 'PhotoController',
					array('only' => array('index', 'show')));

<a name="handling-missing-methods"></a>
## Handling Missing Methods

A catch-all method may be defined which will be called when no other matching method is found on a given controller. The method should be named `missingMethod`, and receives the parameter array for the request as its only argument:

**Defining A Catch-All Method**

	public function missingMethod($parameters)
	{
		//
	}
