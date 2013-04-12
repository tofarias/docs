# Artisan Development

- [Introdução](#introduction)
- [Criando um novo comando](#building-a-command)
- [Registering Commands](#registering-commands)
- [Calling Other Commands](#calling-other-commands)

<a name="introduction"></a>
## Introdução

Em complemento aos comandos padrões do Artisan, você pode criar seus próprios comandos para trabalhar na sua aplicação. Você pode salvar seus comandos na pasta `app/commands`. Entretanto, você é livre para escolher onde irá guardar desde que eles possam ser carregados baseados nas configurações do seu `composer.json`.

<a name="building-a-command"></a>
## Criando um novo comando

### Gerando uma classe

Para criar um novo comando, você deve usar o comando `command:make` do Artisan, que irá gerar um esboço do comando para ajuda-lo a começar.

**Gerando uma nova classe de comando**

	php artisan command:make FooCommand

Por padrão, os comandos criados serão salvos na pasta `app/commands`; No entanto, você pode especificar um caminho diferente ou um namespace:

	php artisan command:make FooCommand --path="app/classes" --namespace="Classes"

### Escrevendo um comando

Uma vez que seu comando foi criado, você deve preencher as propriedades `name` e `description` da ckassem que serão usados para descrever o seu comando na lista de comandos na tela.

O método `fire` será chamando quando seu comando for executado. Você deve clocar toda a lógica de comando nesse método.

### Argumentos e Opções

Os métodos `getArguments` e `getOptions` são onde você pode definir qualquer argumento ou opção que seu comando receberá. Ambos os métodos retornam uma array de comandos, que são descritos por uma lista de array de opções.

Ao definir os `arguments`,  os valores da array de definições são representados da seguinte forma:

	array($name, $mode, $description, $defaultValue)

O argumento `mode` pode ser qualquer um desses: `InputArgument::REQUIRED` ou `InputArgument::OPTIONAL`.

Ao definir `options`, os valores da array de definições são representados da seguinte forma:

	array($name, $shortcut, $mode, $description, $defaultValue)

Para as opções, o argumento `mode` pode ser: `InputOption::VALUE_REQUIRED`, `InputOption::VALUE_OPTIONAL`, `InputOption::VALUE_IS_ARRAY` ou `InputOption::VALUE_NONE`.

O modo `VALUE_IS_ARRAY` indica que a chave pode ser usada várias vezes quando o comando for chamado:

	php artisan foo --option=bar --option=baz

A opção `VALUE_NONE` indica que a opção é simplesmente usada como uma chave.

	php artisan foo --option

### Retrieving Input

While your command is executing, you will obviously need to access the values for the arguments and options accepted by your application. To do so, you may use the `argument` and `option` methods:

**Retrieving The Value Of A Command Argument**

	$value = $this->argument('name');

**Retrieving All Arguments**

	$arguments = $this->argument();

**Retrieving The Value Of A Command Option**

	$value = $this->option('name');

**Retrieving All Options**

	$options = $this->option();

### Writing Output

To send output to the console, you may use the `info`, `comment`, `question` and `error` methods. Each of these methods will use the appropriate ANSI colors for their purpose.

**Sending Information To The Console**

	$this->info('Display this on the screen');

**Sending An Error Message To The Console**

	$this->error('Something went wrong!');

### Asking Questions

You may also use the `ask` and `confirm` methods to prompt the user for input:

**Asking The User For Input**

	$name = $this->ask('What is your name?');

**Asking The User For Secret Input**

	$password = $this->secret('What is the password?');

**Asking The User For Confirmation**

	if ($this->confirm('Do you wish to continue? [yes|no]'))
	{
		//
	}

You may also specify a default value to the `confirm` method, which should be `true` or `false`:

	$this->confirm($question, true);

<a name="registering-commands"></a>
## Registering Commands

Once your command is finished, you need to register it with Artisan so it will be available for use. This is typically done in the `app/start/artisan.php` file. Within this file, you may use the `Artisan::add` method to register the command:

**Registering An Artisan Command**

	Artisan::add(new CustomCommand);

If your command is registered in the application [IoC container](/docs/ioc), you may use the `Artisan::resolve` method to make it available to Artisan:

**Registering A Command That Is In The IoC Container**

	Artisan::resolve('binding.name');

<a name="calling-other-commands"></a>
## Calling Other Commands

Sometimes you may wish to call other commands from your command. You may do so using the `call` method:

**Calling Another Command**

	$this->call('command.name', array('argument' => 'foo', '--option' => 'bar'));
