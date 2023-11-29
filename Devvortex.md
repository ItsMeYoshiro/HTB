# Devvortex

## Enumaração
Primeiramente, iremos iniciar com o NMAP com uma varredura simples.

![](https://i.imgur.com/Rcquds3.png)

Portas comuns, nada de especial.

Usando o endereço ip `10.10.11.242` somos direcionados para http://devvortex.htb. Um site bem estruturado e que possuí um forms de contato sem nenhuma aplicação útil.
![](https://i.imgur.com/F2MjClU.png)
![](https://i.imgur.com/IZZpusH.png)


Também usei o gobuster nessa página não encontrei nenhuma outra pista. 
![](https://i.imgur.com/2pWJqLM.png)


Então decidi procurar por mais domínios atrelados ao `devvortex.htb` usando o dnsmap.
![](https://i.imgur.com/kNgguFG.png)


Encontrei outro domínio sendo o http://dev.devvortex.htb. 
![](https://i.imgur.com/4GSKyQw.png)


Utilizando o gobuster nessa página foi possível encontrar uma página de administrador.
![](https://i.imgur.com/atQGy9J.png)


Indo até http://dev.devvortex.htb/administrator/ foi possível encontrar página de login do "Joomla!".
![](https://i.imgur.com/Y1UIobh.png)


O Joomla é usado como CMS, ele ajuda principalmente para gestão de conteúdo de páginas oferecendo ferramentas para criar, publicar e gerir conteúdos online. Vamos verificar qual versão está sendo rodada usando o JoomScan para procurar por vulnerabilidades.
![](https://i.imgur.com/0cMo2No.png)
![](https://i.imgur.com/SpbK97M.png)


Descobrimos então, que a versão que o Devvortex está rodando é a 4.2.6.

## Explorando a vulnerabilidade

Pesquisando, encontrei a vulnerabilidade CVE-2023-23752, um exploit que explora versões abaixo da 4.2.8. Então, provavelmente, esse sistema será afetado por esse exploit.

O exploit está disponível em:https://github.com/Acceis/exploit-CVE-2023-23752
Mais informações em: https://nvd.nist.gov/vuln/detail/CVE-2023-23752

Utilizando o exploit, conseguimos informações de login no sistema. E com essas informações, conseguimos adentrar no Joomla.
![](https://i.imgur.com/7SXXdkQ.jpg)
![](https://i.imgur.com/W4rTCq5.png)


Dentro do sistema, conseguimos o direito de edição dos templates. Seguindo o caminho System > Templates > Administrator Templates, encontrei algumas páginas php disponíveis dentro de um template.

Implementei um código para que, assim que essa página fosse chamada pelo navegador, a linha seria lida e um shell reverso feito.
![](https://i.imgur.com/aaorM0m.png)


Assim que eu visitei http://dev.devvortex.htb/administrator/templates/atum/login.php, o meu netcat foi ativado, conseguindo assim acesso ao shell.
![](https://i.imgur.com/Xpkxkgj.png)


## Escalonando para usuário

Verificando o sistema de dentro, foi póssivel identificar o usuário logan novamente, pois antes haviamos encontrado ele no scan em que fizemos utilizando o JoomScan. E como ele está registrado no Joomla, é bem provável que a sua senha esteja no banco de dados do sistema. 
![](https://i.imgur.com/Jb46ahV.png)


Antes de tentar acessar o banco de dados, é necessário iniciar um pseudo terminal de utilidades com python. Isso vai nos ajudar para que seja possível realizar operações dentro do banco de dados.
![](https://i.imgur.com/BUsbqiF.jpg)


Pesquisando no banco de dados, é possível encontrar a tabela com os usuários do sistema, e abrindo ela, é possível identificar hash's de senhas. Vamos pegar a hash do usuário logan e quebrar utilizando o john the ripper para que possamos efetuar o login no ssh.
![](https://i.imgur.com/9zD4n4X.png)
![](https://i.imgur.com/yVj92vL.png)
![](https://i.imgur.com/Y7h5ejZ.png)


Após essas etapas, é possível acessar o ssh com a senha que foi quebrada e conseguir a userflag.
![](https://i.imgur.com/xIxHZtc.png)


## Escalonando para root
Utilizei "sudo -l" para listar os privilégios de sudo que o usuário logan possuí. Identifiquei quais operações estão autorizadas para ele, assim, proporcionando uma visão clara dos privilégios disponíveis e possíveis brechas para escalonar o privilegio como root. Com acesso a essas informações, consegui detectar a versão disponível no apport-cli.
![](https://i.imgur.com/g6JOt0b.png)


Essa versão do apport-cli possível uma vulnerabilidade em que vamos carregar e visualizar os crash reports usando o apport-cli. O sistema linux possuí esses crash no /var/crash, utilizando o seguinte comando:
![](https://i.imgur.com/xoUVkf9.png)]


Após começar a carregar, digite V e a página de crash irá carregar. Dessa forma, poderemos digitar !sh conseguindo acesso root e a flag root.
![](https://i.imgur.com/56qfrw0.png)
![](https://i.imgur.com/OyQZ8Vm.jpg)





