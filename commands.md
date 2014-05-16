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

### Recuperando Entrada

Em quanto o comando esta sendo execultado, obviamente você vai precisar ter acesso aos valores de argumentos e opções aceitas pela sua aplicação. Para isso, você pode usar os metodos `argument` e `option`.

**Recuperando Os valores com Comando Argument**

	$value = $this->argument('name');

**Recuperando Todos valores do Argument**

	$arguments = $this->argument();

**Recuperando Os valores com Comando Option**

	$value = $this->option('name');

**Recuperando Todos valores do Option**

	$options = $this->option();

### Writing Output

Para enviat uma saida ao console, você pode usar os metodos `info`, `comment`, `question` e `error`. Cada um desses metodos vai usar usar as cores ANSI apropiadas para seu propósito.

**Enviando Informações para o Console**

	$this->info('Exibe isto na tela');

**Enviando uma mensagem de Erro para o console**

	$this->error('Algo deu errado!');

### Fazendo perguntas

Você também pode usar os metodos `ask` e `confirm` para solicitar uma entrada do usuario como:

**Perguntar algo ao usuário**

	$name = $this->ask('Qual o seu nome ?');

**Perguntaro a senha do usuário**

	$password = $this->secret('Qual a sua senha ?');

**Pedir confirmação ao usuário**

	if ($this->confirm('Você deseja continuar ? [yes|no]'))
	{
		//
	}

Você também pode especificar um valor padrão para o metodo `confirm`, 
You may also specify a default value to the `confirm` method, que deve ser `true` ou `false`:

	$this->confirm($question, true);

<a name="registering-commands"></a>
## Registrando comandos

Uma vez que seu comando esta finalizado, você precisa registrar com Artisan como o mesmo estara disponível para o usuário. Isto é geralmente feito no arquivo `app/start/artisan.php`. Dentro deste arquivo, você pode usar o metodo `Artisan::add` para registrar o comando:

**Registrando um comando Artisan**

	Artisan::add(novo comando personalizado);

Se seu comando esta registrado na aplicação [IoC container](/docs/ioc), você pode usar o metodos `Artisan::resolve` para torná-lo disponível no Artisan:

**Registrando um comando que esta no IoC Container**

	Artisan::resolve('binding.name');

<a name="calling-other-commands"></a>
## Chamando outros comandos

Em algum momento você pode querer chamar outros comando com o seu. Você pode fazer isso usando o metodo `call`:

**Chamando outho comando**

	$this->call('command.name', array('argument' => 'foo', '--option' => 'bar'));
