# wordpress-rautu


Repositório para documentar a instalação, configuração e manutenção
do [Wordpress][] no servidor do [LEG][].

## Instalando gitlab no servidor

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

Para testar o funcionamento do PHP

<!-- Links -->

[Wordpress]: https://wordpress.com
[LEG]: http://www.leg.ufpr.br
[Network configuration]: https://help.ubuntu.com/lts/serverguide/network-configuration.html
[OpenSSH server]: https://help.ubuntu.com/lts/serverguide/openssh-server.html
[ApacheMySQLPHP]: https://help.ubuntu.com/community/ApacheMySQLPHP
[HTTPD - Apache2 Web Server]: https://help.ubuntu.com/lts/serverguide/httpd.html
