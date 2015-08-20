# wordpress-rautu


Repositório para documentar a instalação, configuração e manutenção
do [Wordpress][] no servidor do [LEG][].

## Configuração inicial do servidor

O servidor é Ubuntu 14.04 e já existe um pacote `wordpress` disponível
para esse sistema.

Ao instalar o servidor, algumas configurações de rede já foram
definidas, como o IP da máquina e a forma de conexão. Essa configuração
já funcionou sem precisar fazer alterações posteriores. Para referência
veja [Network configuration][].

Antes de instalar o Wordpress é necessário instalar e configurar um
servidor LAMP (**L**inux, **A**pache, **M**ySQL, **P**HP), e o servidor
SSH. Todos essas pacotes foram instalados durante a instalação so Ubuntu
server, selecionando os pacotes `OpenSSH` e `LAMP` no momento da
instalação. Algumas configurações iniciais do Apache foram realizadas
ainda durante essa instalação. Depois disso, também não foram
necessárias mais configurações adicionais para o funcionamento básico
destas ferramentas. Para referências, veja [OpenSSH server][],
[ApacheMySQLPHP][], e para ajuda específica com Apache, veja
[HTTPD - Apache2 Web Server][].

Após a instalação é possível testar o funcionamento do Apache e do
PHP. Para testar se o Apache está funcionando, basta digitar o endereço
do servidor no navegador e deve aparecer uma página padrão do Apache.

> NOTA: para achar o IP público do servidor use:
```{sh}
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```
> ou usando um método alternativo:
```{sh}
curl http://icanhazip.com
```

Para testar o funcionamento do PHP, crie um arquivo `index.php` em
`/var/www/html` com o conteúdo

```{sh}
sudo echo "<?php phpinfo(); ?>" > /var/www/html/index.php
```
e no navegador, agora digite `http://<IP>/index.php`. Uma página com
informações da instalação do PHP deve aparecer se estiver tudo funcionando.

## Instalando Wordpress no servidor

Para instalar o Wordpress, utilizamos o pacote já disponível no Ubuntu
server

```{sh}
sudo apt-get install wordpress
```

Os arquivos de instalação ficam em `/usr/share/wordpress` e alguns
arquivos de configuração em `/etc/wordpress`.

A instalação de fato do Wordpress só é finalizada após as configurações
abaixo.

## Configurando Wordpress no servidor

Após a instalação algumas configurações básicas precisam ser feitas. A
página de referância para instalação e configuração no Ubuntu server é
[WordPress][].

O primeiro passo é configurar o Apache para localizar a instalação do
Wordpress, e setar algumas configurações básicas. No diretório
`/etc/apache2/sites-available`, foi criado um arquivo `wordpress.conf`
com o seguinte conteúdo:

```
<VirtualHost *:80>
  ServerName blog.leg.ufpr.br
  ServerAlias *.blog.leg.ufpr.br
  DocumentRoot /usr/share/wordpress

  <Directory /usr/share/wordpress>
    Options FollowSymLinks
    AllowOverride Limit Options FileInfo
    DirectoryIndex index.php
    Order allow,deny
    Allow from all
  </Directory>

  <Directory /usr/share/wordpress/wp-content>
    Options FollowSymLinks
    Order allow,deny
    Allow from all
  </Directory>

</VirtualHost>
```

que é um *virtual host* do Apache. Depois de salvo, habilite o site com

```{sh}
sudo a2ensite wordpress
```

e reinicie o Apache com

```{sh}
sudo service apache2 restart
```

> NOTA: lembrando que o `ServerName` deve ser configurado antes no
> servidor DNS. No arquivo de configuração do BIND (na zona do LEG), foi
> inserida a seguinte linha
```
blog    IN    A    <IP>
```

Isso ainda não garante o funcionamento do Wordpress. É necessário ainda 
criar um arquivo de configuração em PHP para que o Wordpress use uma
base de dados MySQL. Esse arquivo deve ficar em `/etc/wordpress` e deve
ter o mesmo nome do `ServerName` do *virtual host*. Portanto, aqui
criamos o arquivo `/etc/wordpress/config-blog.leg.ufpr.br.php` com o
seguinte conteúdo:

```
<?php
define('DB_NAME', 'wordpress');
define('DB_USER', 'wordpress');
define('DB_PASSWORD', '<senha>');
define('DB_HOST', 'localhost');
define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
?>
```

> ATENÇÃO: não deixe linhas em branco no final de arquivos `.php`!

Note que nesse arquivo é especificado um usuário `wordpress` com uma
senha `<senha>`, com acesso a um banco de dados MySQL. Para criar esse
usuário e senha no MySQL (e a própria base de dados), crie um arquivo
(em qualquer diretório) temporário `worpress.sql` com o seguinte
conteúdo: 

```
CREATE DATABASE wordpress;
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER
ON wordpress.*
TO wordpress@localhost
IDENTIFIED BY '<senha>';
FLUSH PRIVILEGES;
```

e execute o comando abaixo para de fato executar estes comandos

```{sh}
cat wordpress.sql | sudo mysql --defaults-extra-file=/etc/mysql/debian.cnf
```

Agora já deve ser possível acessar o Wordpress pelo navegador pelo
endereço `http://<IP-OU-ServerName/wp-admin/install.php>`.

Nessa página foi criado um usuário inicial `admin`, e a instalação é
concluída na sequência. A partir daqui, o Wordpress já está pronto para
uso.

## Configurações gerais

Algumas configurações adicionais foram necessárias para o funcionamento
correto do Wordpress.

### Mudando permissões de diretórios

Assuminado que o usuário `leg` é o principal mantenedor da instalação,
alteramos os donos dos diretórios `/var/www/html` e, principalmente,
`/usr/share/wordpress` para `leg:www-data`, com

```{sh}
sudo chown -R leg:www-data <dir>
```

Isso foi necessário para que pudessemos instalar plugins e temas no
Wordpress através da interface gráfica.

Para mais detalhes das permissões de diretórios do Wordpress, veja
[Changing file permissions][].

### Configurando o RSS

O RSS já deve estar configurado normalmente com a instalação padrão. O
endereço para as entradas RSS fica em `blog.leg.ufp.br/feed`.

No começo, tivemos um problema com o RSS que parecia estar
desconfigurado. Depois de muitas buscas, várias pessoas relataram o
mesmo problema, que no final das contas era causado por linhas em branco
ao final de arquivos `.php`. Foi feito uma busca nos principais arquivos
php e as linhas em branco foram removidas. No entanto, o principal
arquivo que estava com linha em branco era o
`/etc/wordpress/config-blog.leg.ufpr.br.php`, o primeiro a ser criado
durante a configuração do blog (conforme descrito acima). Após remover a
linha em branco no final desse arquivo, o RSS funcionou corretamente.

Um site para verificar a validade do RSS é o http://feedvalidator.org.

### Configurando servidor FTP e permitindo acesso pela interface

Para instalr plugins e temas do Wordpress facilmente pela interface web,
ainda foi necessário instalar um serviço de FTP e alterar uma
configuração para permitir o acesso pelo Wordpress.

Para instalar o serviço [FTP Server][]:

```{sh}
sudo apt-get install vsftpd
```

Para permitir o download anônimo por FTP, foi alterado o arquivo
`/etc/vsftpd.conf`, deixando a linha a seguir dessa forma

```
anonymous_enable=Yes
```
e reinicie o serviço com

```{sh}
sudo restart vsftpd
```

Ainda assim, o Wordpress continuava pedindo por usuário FTP e
senha. Para evitar isso, bastou adicionar a seguinte linha no arquivo
`/usr/share/wordpress/wp-config.php`

```
/* Sets up 'direct' method for wordpress, auto update without FTP */
define('FS_METHOD','direct');
```

como explicado nesse
[link](http://www.hongkiat.com/blog/update-wordpress-without-ftp/). Dessa
forma, o problema foi resolvido e conseguimos instalar plugins e temas
pela interface web do Wordpress.

<!-- Links -->

[Wordpress]: https://wordpress.com
[LEG]: http://www.leg.ufpr.br
[Network configuration]: https://help.ubuntu.com/lts/serverguide/network-configuration.html
[OpenSSH server]: https://help.ubuntu.com/lts/serverguide/openssh-server.html
[ApacheMySQLPHP]: https://help.ubuntu.com/community/ApacheMySQLPHP
[HTTPD - Apache2 Web Server]: https://help.ubuntu.com/lts/serverguide/httpd.html
[WordPress]: https://help.ubuntu.com/lts/serverguide/wordpress.html
[Changing file permissions]: http://codex.wordpress.org/Changing_File_Permissions
[FTP Server]: https://help.ubuntu.com/lts/serverguide/ftp-server.html
