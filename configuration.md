# Configuração

- [Introdução](#introduction)
- [Environment Configuration](#environment-configuration)

<a name="introduction"></a>
## Introdução

Todos os arquivos de configuração do Laravel estão armazenados no diretório `app/config`. Cada opção está documentada em seu respectivo arquivo, fique a vontade para olhar os arquivos e se familiarizar com as opções disponíveis para você.

Em algum momento você pode precisar acessar os valores de configuração em tempo de execução. Você pode fazer isso usando a classe `Config`:

**Acessando um valor de configuração**

	Config::get('app.timezone');

Notice that "dot" style syntax may be used to access values in the various files. Você também pode definir valores de configuração em tempo de execução:

**Definindo um valor de configuração**

	Config::set('database.default', 'sqlite');

<a name="environment-configuration"></a>
## Configuração de ambiente

Muitas vezes é útil ter valores diferentes com base no ambiente onde o aplicativo é executado. Por exemplo, você pode querer usar um drive de cache em sua maquina de desenvolvimento local diferente do drive usado em seu servidor de produção. É fácil fazer isto usando configuração baseada no ambiente.

Simplismente crie uma pasta dentro do diretório `config` que correspondente ao seu nome de ambiente, por exemplo `local`. Em seguida, crie os arquivos de configuração que você deseja sobrescrever e especifique as opções para teste ambiente. Por exemplo, para sobrescrever o driver de cache para seu ambiente local, você poderia criar um arquivo `cache.php` em `app/config/local` com o seguinte conteúdo:

	<?php

	return array(

		'driver' => 'file',

	);

> **Note:** Não use 'testing' como um nome de ambiente. Esta é reservada para testes de unidade.

Notice that you do not have to specify _every_ option that is in the base configuration file, but only the options you wish to override. The environment configuration files will "cascade" over the base files.

Next, we need to instruct the framework how to determine which environment it is running in. The default environment is always `production`. However, you may setup other environments within the `bootstrap/start.php` file at the root of your installation. In this file you will find an `$app->detectEnvironment` call. The array passed to this method is used to determine the current environment. You may add other environments and machine names to the array as needed.

    <?php

    $env = $app->detectEnvironment(array(

        'local' => array('your-machine-name'),

    ));

You may also pass a `Closure` to the `detectEnvironment` method, allowing you to implement your own environment detection:

	$env = $app->detectEnvironmnet(function()
	{
		return $_SERVER['MY_LARAVEL_ENV'];
	});

You may access the current application environment via the `environment` method:

**Accessing The Current Application Environment**

	$environment = App::environment();
