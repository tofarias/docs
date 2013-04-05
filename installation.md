# Instalação

- [Instalando o Composer](#install-composer)
- [Instalando o Laravel](#install-laravel)
- [Requisitos do Servidor](#server-requirements)
- [Configuração](#configuration)
- [URLs amigáveis](#pretty-urls)

<a name="install-composer"></a>
## Instalando o Composer

Laravel utiliza o [Composer](http://getcomposer.org) para gerenciar suas dependências. Primeiro, baixe uma cópia do `composer.phar`. Uma vez que você tenha o arquivo PHAR, você mantê-lo na pasta do seu projeto ou movê-lo para `usr/local/bin` para usa-lo globalmente em seu sistema. No Windows, você pode usar o [Instalador do Windows](https://getcomposer.org/Composer-Setup.exe).

<a name="install-laravel"></a>
## Instalando o Laravel

Once Composer is installed, download the [latest version](https://github.com/laravel/laravel/archive/develop.zip) of the Laravel framework and extract its contents into a directory on your server. Next, in the root of your Laravel application, run the `php composer.phar install` command to install all of the framework's dependencies. This process requires Git to be installed on the server to successfully complete the installation.

Com o Composer instalado, baixe a [última versão](https://github.com/laravel/laravel/archive/develop.zip) do Laravel e extraia seu conteúdo em uma pasta no seu servidor. A seguirt, na raíz da sua aplicação, 

<a name="server-requirements"></a>
## Requisitos do Servidor

The Laravel framework has a few system requirements:

- PHP >= 5.3.7
- MCrypt PHP Extension

<a name="configuration"></a>
## Configuração

Laravel needs almost no configuration out of the box. You are free to get started developing! However, you may wish to review the `app/config/app.php` file and its documentation. It contains several options such as `timezone` and `locale` that you may wish to change according to your application.

> **Note:** One configuration option you should be sure to set is the `key` option within `app/config/app.php`. This value should be set to a 32 character, random string. This key is used when encrypting values, and encrypted values will not be safe until it is properly set. You can set this value quickly by using the following artisan command `php artisan key:generate`.

<a name="permissions"></a>
### Permissions
Laravel requires one set of permissions to be configured - folders within app/storage require write access by the web server.

<a name="paths"></a>
### Paths

Several of the framework directory paths are configurable. To change the location of these directories, check out the `bootstrap/paths.php` file.

<a name="pretty-urls"></a>
## URLs amigáveis

The framework ships with a `public/.htaccess` file that is used to allow URLs without `index.php`. If you use Apache to serve your Laravel application, be sure to enable the `mod_rewrite` module.

If the `.htaccess` file that ships with Laravel does not work with your Apache installation, try this one:

	Options +FollowSymLinks
	RewriteEngine on

	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d

	RewriteRule . index.php [L]
