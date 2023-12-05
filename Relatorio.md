# Relatório do Projeto de Instância Segura em Nuvem

**UFSC - Universidade Federeal de Santa Catarina**
*Centro de Ciências, Tecnologias e Saúde* - *Departamento de Engenharia de Computação*

***DEC7128 -  Redes de Computadores II***

Professor: [*Gerson Luiz Camillo*](https://github.com/glcamillo)

Alunos:

*Iudi Zurba Melgarejo*

*Michele*

## Sumário

## Introdução

É de conhecimento comum que as práticas de segurança são indispensáveis para montagem de qualquer sistema computacional em rede.

Nesse trabalho, abordamos práticas mínimas de segurança, para montagem de um servidor web na nuvem, provendo serviços HTTP, HTTPS e SSH.

Esse é um relatório descritivo da experiência de montagem e configuração do servidor, e as práticas de segurança adotadas.

## 1. Provedor de Nuvem

Inicialmente, tentamos utilizar a Azure para realizar o trabalho, entretanto, enfrentamos dificuldades com a conta da microsoft na Azure, que estava deslogando automaticamente na criação da instância Linux.

Fomos então tentar usar o serviço da AWS e tudo ocorreu bem na criação da conta e da instância, portanto foi a escolhida para o trabalho, utilizando o *Free Tier*.

Iudi ficou como o responsável pela conta na AWS.(*root*)

## 2. MFA

*Responsável: Iudi*

Usuário root ativou a autenticação multifatorial, utilizando o Google Authenticator.

## 3. Grupos IAM

*Responsável: Iudi*

O usuário root criou dois grupos, um para os gerentes da EC2 e outro para o usuário "cebola".

1. **Grupo de Gerentes**

    O grupo de gerentes foi criado para que os usuários que o compõem possam realizar ações administrativas na instância, como instalar pacotes, criar usuários, etc.

    Política de permissões do grupo: `AmazonEC2FullAccess`

2. **Grupo Cebola**

    O grupo cebola foi criado para que o usuário "cebola", possa ver e ler as informações na instância, como ler arquivos de log, por exemplo.

    Política de permissões do grupo: `AmazonEC2ReadOnlyAccess`

## 4. Instância EC2

*Responsável: Iudi*

Tipo de instância: EC2 t2.micro

Imagem de SO escolhida: Amazon Linux 2023

Nome da instância: "Curupira"

Criamos a instância sem problemas, utilizamos as opções padrões do Free Tier para as configurações de armazenamento.

Foi gerada o par de chaves padrão da criação da instância.

## 5. SSH - Primeiras conexoes

*Responsável: Iudi*

Com o par de chaves criado automaticamente pelo AWS, foi feita a conexão com o servidor:

![](imagens/ssh_chave_padrao.png)

Foram adicionadas as chaves públicas dos integrantes no arquivo `authorized_keys` do servidor.

Feito isso, conexão com a chave ssh própria (iudi):

![](imagens/ssh_iudi.png)

## 6. Usuários na instância

*Responsável: Iudi*

A equipe estava se conectando na instância com o usuário "ec2-user", que é o usuário padrão da instância.

É possível criar um usuário no sistema para cada integrante, e relacionar a respectiva chave pública em cada usuário criado.
Porém, optamos por manter um usuário único ec2-user, para manter a simplicidade no acesso e permissões no sistema.

## 7. Apache & HTTP

*Responsável: Iudi*

Montagem do servidor web com Apache e HTTP.

Instalação do Apache:

Seguindo o [Guia para instalação no Amazon Linux 2023](), instaladas as dependências e o Apache, bem como configuradas as permissões de acesso aos diretórios de `/var/www/html`.

* Serviço `httpd` iniciado e habilitado para iniciar com o sistema:

![httpd_status](imagens/httpd_status.png)
![httpd_ola_mundo](imagens/httpd_ola_mundo.png)

* PHP instalado e funcionando:

![php_info](imagens/php_info.png)

## 8. Restrições de portas na AWS

*Responsável: Iudi*

Inicialmente, deixamos as portas 80, 443 e 22 abertas para qualquer IP.

Posteriormente, a porta 80 foi restrita aos IPs da Rede UFSC.

## 9. Certificato SSL & HTTPS

*Responsável: Iudi*

* Geramos um certificado SSL auto assinado (nível mais básico de certificao)
![https_cert_auto](imagens/https_cert_auto.png)

## 9. Configuracoes de Firewall

*Responsável: Iudi*

* Permissão para a porta 80 liberada no firewall:

![httpd_firewall](imagens/httpd_firewall.png)

## 11. Instalando Honeypot

*Responsável: Iudi*

Instalamos a ferramenta Cowrie para realizar o serviço de honeypot na instância, escutando na porta 22 e 2222 por possiveis tentativas de acesso SSH.

O Cowrie simula um servidor SSH, e registra as tentativas de acesso, bem como as ações realizadas pelos atacantes. O conteudo e salvo em logs no usuario cowrie.

# 12. Reconfiguração da porta SSH

*Responsável: Iudi*

A porta para o servico SSH real foi alterada para uma porta especifica nao convencional, definidada pelo grupo.

* **Problema:** Inicialmente, cometi o deslize de nao alterar a porta do servico SSH antes de definir o encaminhamento da porta 22 para a 2222(onde o Honeypot esta escutando), isso acarretou com que o acesso por SSH a instancia fosse impossibilitado.

* **Solucao:** Apos certa pesquisa, consegui utilizar a funcionalidade da AWS chamada System Session Manager(SSM), que permite acesso a instancia sem a necessidade de uma porta aberta para SSH, e sem a necessidade de uma chave privada. Apenas com o usuario e senha do usuario ec2-user, e com o acesso ao console da AWS, foi possivel retomar acesso a instancia e corrigir a porta SSH.


## Configurar o NTP: sincronizar o relógio da instância com o horário de Brasília.

*Responsável: Michele*

Para configurar o NTP (Network Time Protocol) em uma instância da AWS com sistema CentOS/RHEL para sincronizar o relógio com o horário de Brasília usamos os seguintes comandos:
![image](https://github.com/iudizm/instancia-segura-redes2UFSC/assets/93664169/b1938c94-ec09-4c8c-81f1-27ab42396802)

Como mostrado na imagem o pacote npn não foi reconhecido ou não está disponível nos repositórios de pacotes padrão da instância CentOS.
O que nos levou a pegar uma alternativa. A partir do CentOS 8, o NTP foi substituído pelo Chrony como o cliente NTP padrão. Portanto, utilizamos o Chrony para configurar a sincronização de tempo em vez do NTP. 

![image](https://github.com/iudizm/instancia-segura-redes2UFSC/assets/93664169/7e2a4019-c956-4fc2-9b77-ef25cb21971e)

Como podemos ver na imagem abaixo, o sistema esta configurado para sincronizar o relógio da sua instância CentOS com o horário de Brasília usando servidores NTP.

![image](https://github.com/iudizm/instancia-segura-redes2UFSC/assets/93664169/42df45c8-a9d8-4753-a10f-87c384ccbfac)



## Acesso Mandatório (SELINUX): estrutura de segurança

*Responsável: Michele*

O SELinux, ou Security-Enhanced Linux, é uma estrutura de segurança desenvolvida para sistemas Linux que adiciona uma camada adicional de controle de acesso obrigatório (MAC - Mandatory Access Control) ao sistema operacional. Essa camada de segurança complementa o controle de acesso baseado em permissões tradicional, conhecido como controle de acesso discricionário (DAC - Discretionary Access Control), que é controlado pelos proprietários de arquivos e administradores do sistema.

- Enforcing (Impositivo): Neste modo, o SELinux está ativo e aplicará rigorosamente todas as políticas de segurança definidas, o que significa que ele restringirá o acesso a recursos do sistema de acordo com as políticas de controle de acesso obrigatório (MAC) configuradas. Se um processo tentar realizar uma ação que não esteja de acordo com as políticas, o SELinux negará essa ação.
  
O SELinux estará ativado e funcionando no modo "enforcing", o que aumentará a segurança do sistema, aplicando políticas de controle de acesso obrigatório. o SELinux pode ser rigoroso e pode exigir configurações adicionais para permitir que aplicativos específicos funcionem corretamente. Você pode consultar a documentação do SELinux e as políticas de segurança do CentOS ou RHEL para ajustar as configurações conforme necessário para as necessidades do seu sistema.

![image](https://github.com/iudizm/instancia-segura-redes2UFSC/assets/93664169/492c4ea1-feb0-411a-bc02-456d5ba83121)



## Tecnicas de seguranca: CIS Benchmark

- **Desabilitar Portas Não Utilizadas**: Isso pode ajudar a prevenir acessos não autorizados a serviços que não são necessários para a operação da sua instância.

- **Configurar uma Política de Senhas Forte no IAM**: Isso inclui a aplicação de uma política de senhas complexas, uso de autenticação multifator (MFA), desabilitação do usuário root e rotação regular das chaves de acesso​​​​. https://aws.amazon.com/pt/what-is/cis-benchmarks/

## certificacao SSL: LetsEncrypt

![image](https://github.com/iudizm/instancia-segura-redes2UFSC/assets/93664169/2fbcb49b-81c8-4713-9052-3dab87d38513)


![image](https://github.com/iudizm/instancia-segura-redes2UFSC/assets/93664169/bc9d65ca-7017-45c5-b67e-0ca6b719b42c)


![image](https://github.com/iudizm/instancia-segura-redes2UFSC/assets/93664169/f8186d0d-a8da-4994-a4a7-8df74a9a662b)


## Logs
*Responsável: Michele e iudi*

#### SHH porta oculta destinada e Logs Cowrie honeypot :22 e :2222
acesso a ip limitado no aws security group

[logs.txt](https://github.com/iudizm/instancia-segura-redes2UFSC/files/13573221/logs.txt)
[cowrie@ip-172-31-45-128 cowrie]$ cat var/log/cowrie/cowrie.log
2023-11-10T00:12:51.715324Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.160.94.238:26443 (172.31.45.128:2222) [session: 145f8fef94bb]
2023-11-10T00:12:51.716606Z [HoneyPotSSHTransport,0,45.160.94.238] Remote SSH version: SSH-2.0-OpenSSH_8.8
2023-11-10T00:12:51.736134Z [HoneyPotSSHTransport,0,45.160.94.238] SSH client hassh fingerprint: b50f8371fb780ca7060e53c0b2cf6172
2023-11-10T00:12:51.737293Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T00:12:51.737400Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:12:51.737490Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:12:55.355271Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T00:12:55.374290Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-userauth'
2023-11-10T00:12:55.394082Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'none'
2023-11-10T00:12:57.808573Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:12:57.809184Z [HoneyPotSSHTransport,0,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:12:57.809602Z [HoneyPotSSHTransport,0,45.160.94.238] login attempt [b'ec2-user'/b''] failed
2023-11-10T00:12:58.811070Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:12:58.811296Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:13:05.639672Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:13:05.640055Z [HoneyPotSSHTransport,0,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:13:05.640239Z [HoneyPotSSHTransport,0,45.160.94.238] login attempt [b'ec2-user'/b'pasword'] failed
2023-11-10T00:13:06.642040Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:13:06.642271Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:13:09.095211Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:13:09.095584Z [HoneyPotSSHTransport,0,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:13:09.095770Z [HoneyPotSSHTransport,0,45.160.94.238] login attempt [b'ec2-user'/b'clear'] failed
2023-11-10T00:13:10.097683Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:13:10.097922Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:13:10.119355Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T00:13:10.119529Z [HoneyPotSSHTransport,0,45.160.94.238] Connection lost after 18 seconds
2023-11-10T00:13:12.334128Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.160.94.238:27004 (172.31.45.128:2222) [session: dea3b5c9b00e]
2023-11-10T00:13:12.334897Z [HoneyPotSSHTransport,1,45.160.94.238] Remote SSH version: SSH-2.0-OpenSSH_8.8
2023-11-10T00:13:12.354976Z [HoneyPotSSHTransport,1,45.160.94.238] SSH client hassh fingerprint: b50f8371fb780ca7060e53c0b2cf6172
2023-11-10T00:13:12.356083Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T00:13:12.356191Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:13:12.356280Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:13:12.448771Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T00:13:12.467780Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-userauth'
2023-11-10T00:13:12.487838Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'none'
2023-11-10T00:13:16.025245Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:13:16.025632Z [HoneyPotSSHTransport,1,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:13:16.025806Z [HoneyPotSSHTransport,1,45.160.94.238] login attempt [b'ec2-user'/b'chiquita'] failed
2023-11-10T00:13:17.027612Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:13:17.027844Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:13:23.543982Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:13:23.544351Z [HoneyPotSSHTransport,1,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:13:23.544519Z [HoneyPotSSHTransport,1,45.160.94.238] login attempt [b'ec2-user'/b'root'] failed
2023-11-10T00:13:24.546415Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:13:24.546636Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:13:27.504964Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:13:27.505323Z [HoneyPotSSHTransport,1,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:13:27.505487Z [HoneyPotSSHTransport,1,45.160.94.238] login attempt [b'ec2-user'/b'ec2-user'] failed
2023-11-10T00:13:28.507287Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:13:28.507517Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:13:28.527991Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T00:13:28.528181Z [HoneyPotSSHTransport,1,45.160.94.238] Connection lost after 16 seconds
2023-11-10T00:16:55.814828Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.160.94.238:26095 (172.31.45.128:2222) [session: 1c9f74ee165a]
2023-11-10T00:16:55.815660Z [HoneyPotSSHTransport,2,45.160.94.238] Remote SSH version: SSH-2.0-OpenSSH_8.8
2023-11-10T00:16:55.835156Z [HoneyPotSSHTransport,2,45.160.94.238] SSH client hassh fingerprint: b50f8371fb780ca7060e53c0b2cf6172
2023-11-10T00:16:55.836407Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T00:16:55.836545Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:16:55.836635Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:16:55.923104Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T00:16:55.942603Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-userauth'
2023-11-10T00:16:55.960393Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'none'
2023-11-10T00:16:57.457472Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:16:57.458105Z [HoneyPotSSHTransport,2,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:16:57.458400Z [HoneyPotSSHTransport,2,45.160.94.238] login attempt [b'ec2-user'/b''] failed
2023-11-10T00:16:58.460081Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:16:58.460335Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:16:59.192332Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:16:59.192749Z [HoneyPotSSHTransport,2,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:16:59.192918Z [HoneyPotSSHTransport,2,45.160.94.238] login attempt [b'ec2-user'/b''] failed
2023-11-10T00:17:00.194919Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:17:00.195143Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:17:00.889274Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T00:17:00.889472Z [HoneyPotSSHTransport,2,45.160.94.238] Connection lost after 5 seconds
2023-11-10T00:19:08.160080Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.160.94.238:25287 (172.31.45.128:2222) [session: 19b5eb6547af]
2023-11-10T00:19:08.160812Z [HoneyPotSSHTransport,3,45.160.94.238] Remote SSH version: SSH-2.0-OpenSSH_8.8
2023-11-10T00:19:08.179893Z [HoneyPotSSHTransport,3,45.160.94.238] SSH client hassh fingerprint: b50f8371fb780ca7060e53c0b2cf6172
2023-11-10T00:19:08.181059Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T00:19:08.181165Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:19:08.181254Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:19:08.273880Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T00:19:08.294397Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-userauth'
2023-11-10T00:19:08.313974Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'none'
2023-11-10T00:19:10.090244Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:19:10.090597Z [HoneyPotSSHTransport,3,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:19:10.090761Z [HoneyPotSSHTransport,3,45.160.94.238] login attempt [b'ec2-user'/b''] failed
2023-11-10T00:19:11.092625Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:19:11.092881Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:19:12.048981Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:19:12.049341Z [HoneyPotSSHTransport,3,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:19:12.049511Z [HoneyPotSSHTransport,3,45.160.94.238] login attempt [b'ec2-user'/b''] failed
2023-11-10T00:19:13.051373Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:19:13.051598Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:21:08.261149Z [-] Timeout reached in HoneyPotSSHTransport
2023-11-10T00:21:08.261932Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T00:21:08.262043Z [HoneyPotSSHTransport,3,45.160.94.238] Connection lost after 120 seconds
2023-11-10T00:23:14.934832Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.160.94.238:26247 (172.31.45.128:2222) [session: 739d3f99da80]
2023-11-10T00:23:14.935613Z [HoneyPotSSHTransport,4,45.160.94.238] Remote SSH version: SSH-2.0-OpenSSH_8.8
2023-11-10T00:23:14.956856Z [HoneyPotSSHTransport,4,45.160.94.238] SSH client hassh fingerprint: b50f8371fb780ca7060e53c0b2cf6172
2023-11-10T00:23:14.957971Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T00:23:14.958075Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:23:14.958171Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:23:15.040047Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T00:23:15.040246Z [HoneyPotSSHTransport,4,45.160.94.238] Connection lost after 0 seconds
2023-11-10T00:23:32.122644Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.160.94.238:26720 (172.31.45.128:2222) [session: 18abe3d9aed0]
2023-11-10T00:23:32.123415Z [HoneyPotSSHTransport,5,45.160.94.238] Remote SSH version: SSH-2.0-OpenSSH_8.8
2023-11-10T00:23:32.142334Z [HoneyPotSSHTransport,5,45.160.94.238] SSH client hassh fingerprint: b50f8371fb780ca7060e53c0b2cf6172
2023-11-10T00:23:32.143473Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T00:23:32.143591Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:23:32.143682Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:23:32.227650Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T00:23:32.227842Z [HoneyPotSSHTransport,5,45.160.94.238] Connection lost after 0 seconds
2023-11-10T00:24:33.720391Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.160.94.238:26093 (172.31.45.128:2222) [session: a110e6a528f5]
2023-11-10T00:24:33.721184Z [HoneyPotSSHTransport,6,45.160.94.238] Remote SSH version: SSH-2.0-OpenSSH_8.8
2023-11-10T00:24:33.740873Z [HoneyPotSSHTransport,6,45.160.94.238] SSH client hassh fingerprint: b50f8371fb780ca7060e53c0b2cf6172
2023-11-10T00:24:33.742060Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T00:24:33.742170Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:24:33.742267Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:24:35.701820Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T00:24:35.721911Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-userauth'
2023-11-10T00:24:35.739407Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'none'
2023-11-10T00:24:37.634884Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:24:37.635236Z [HoneyPotSSHTransport,6,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:24:37.635401Z [HoneyPotSSHTransport,6,45.160.94.238] login attempt [b'ec2-user'/b''] failed
2023-11-10T00:24:38.637397Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:24:38.637623Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:24:39.363934Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:24:39.364283Z [HoneyPotSSHTransport,6,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:24:39.364444Z [HoneyPotSSHTransport,6,45.160.94.238] login attempt [b'ec2-user'/b''] failed
2023-11-10T00:24:40.366278Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:24:40.366509Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:26:33.750021Z [-] Timeout reached in HoneyPotSSHTransport
2023-11-10T00:26:33.750498Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T00:26:33.750608Z [HoneyPotSSHTransport,6,45.160.94.238] Connection lost after 120 seconds
2023-11-10T00:27:04.714459Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.160.94.238:25292 (172.31.45.128:2222) [session: a283c2c0729f]
2023-11-10T00:27:04.715186Z [HoneyPotSSHTransport,7,45.160.94.238] Remote SSH version: SSH-2.0-OpenSSH_8.8
2023-11-10T00:27:04.734091Z [HoneyPotSSHTransport,7,45.160.94.238] SSH client hassh fingerprint: b50f8371fb780ca7060e53c0b2cf6172
2023-11-10T00:27:04.735201Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T00:27:04.735304Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:27:04.735393Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:27:04.827030Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T00:27:04.845145Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-userauth'
2023-11-10T00:27:04.864088Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'none'
2023-11-10T00:27:08.028302Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:27:08.028655Z [HoneyPotSSHTransport,7,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:27:08.028818Z [HoneyPotSSHTransport,7,45.160.94.238] login attempt [b'ec2-user'/b''] failed
2023-11-10T00:27:09.030641Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:27:09.030867Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:27:10.332463Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:27:10.332810Z [HoneyPotSSHTransport,7,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:27:10.332993Z [HoneyPotSSHTransport,7,45.160.94.238] login attempt [b'ec2-user'/b''] failed
2023-11-10T00:27:11.334785Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:27:11.335035Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:27:12.660361Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:27:12.660732Z [HoneyPotSSHTransport,7,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:27:12.660899Z [HoneyPotSSHTransport,7,45.160.94.238] login attempt [b'ec2-user'/b'chiquita'] failed
2023-11-10T00:27:13.662705Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:27:13.662935Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:27:13.680990Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T00:27:13.681153Z [HoneyPotSSHTransport,7,45.160.94.238] Connection lost after 8 seconds
2023-11-10T00:27:23.645629Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.160.94.238:25762 (172.31.45.128:2222) [session: 2c2dd53aff03]
2023-11-10T00:27:23.646453Z [HoneyPotSSHTransport,8,45.160.94.238] Remote SSH version: SSH-2.0-OpenSSH_8.8
2023-11-10T00:27:23.664236Z [HoneyPotSSHTransport,8,45.160.94.238] SSH client hassh fingerprint: b50f8371fb780ca7060e53c0b2cf6172
2023-11-10T00:27:23.665352Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T00:27:23.665459Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:27:23.665549Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:27:23.755571Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T00:27:23.772605Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-userauth'
2023-11-10T00:27:23.791179Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'none'
2023-11-10T00:27:28.164277Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T00:27:28.164634Z [HoneyPotSSHTransport,8,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:27:28.164800Z [HoneyPotSSHTransport,8,45.160.94.238] login attempt [b'ec2-user'/b''] failed
2023-11-10T00:27:29.166621Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T00:27:29.166868Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:29:23.650014Z [-] Timeout reached in HoneyPotSSHTransport
2023-11-10T00:29:23.650493Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T00:29:23.650600Z [HoneyPotSSHTransport,8,45.160.94.238] Connection lost after 120 seconds
2023-11-10T00:32:47.795506Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.160.94.238:25382 (172.31.45.128:2222) [session: 295a3c01e8fe]
2023-11-10T00:32:47.796223Z [HoneyPotSSHTransport,9,45.160.94.238] Remote SSH version: SSH-2.0-OpenSSH_8.8
2023-11-10T00:32:47.815597Z [HoneyPotSSHTransport,9,45.160.94.238] SSH client hassh fingerprint: b50f8371fb780ca7060e53c0b2cf6172
2023-11-10T00:32:47.816878Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T00:32:47.816995Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:32:47.817081Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T00:32:47.905203Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T00:32:47.924153Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-userauth'
2023-11-10T00:32:47.944591Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'cowrie' trying auth b'none'
2023-11-10T00:32:49.109317Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'cowrie' trying auth b'password'
2023-11-10T00:32:49.109669Z [HoneyPotSSHTransport,9,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:32:49.109836Z [HoneyPotSSHTransport,9,45.160.94.238] login attempt [b'cowrie'/b''] failed
2023-11-10T00:32:50.111715Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'cowrie' failed auth b'password'
2023-11-10T00:32:50.111941Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:32:55.589317Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'cowrie' trying auth b'password'
2023-11-10T00:32:55.589684Z [HoneyPotSSHTransport,9,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:32:55.589896Z [HoneyPotSSHTransport,9,45.160.94.238] login attempt [b'cowrie'/b'cowrie'] failed
2023-11-10T00:32:56.591770Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'cowrie' failed auth b'password'
2023-11-10T00:32:56.592011Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:33:09.085353Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'cowrie' trying auth b'password'
2023-11-10T00:33:09.085709Z [HoneyPotSSHTransport,9,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T00:33:09.085878Z [HoneyPotSSHTransport,9,45.160.94.238] login attempt [b'cowrie'/b''] failed
2023-11-10T00:33:10.087711Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'cowrie' failed auth b'password'
2023-11-10T00:33:10.087944Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T00:33:10.106167Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T00:33:10.106317Z [HoneyPotSSHTransport,9,45.160.94.238] Connection lost after 22 seconds
2023-11-10T00:57:31.730843Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 198.235.24.3:56220 (172.31.45.128:2222) [session: a46e78d87e30]
2023-11-10T00:57:32.180630Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T00:57:32.180836Z [HoneyPotSSHTransport,10,198.235.24.3] Connection lost after 0 seconds
2023-11-10T01:45:47.272389Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 172.105.128.13:30060 (172.31.45.128:2222) [session: af54032a3fcc]
2023-11-10T01:45:47.273128Z [HoneyPotSSHTransport,11,172.105.128.13] Remote SSH version: SSH-2.0-Go
2023-11-10T01:45:47.406012Z [HoneyPotSSHTransport,11,172.105.128.13] SSH client hassh fingerprint: 4e066189c3bbeec38c99b1855113733a
2023-11-10T01:45:47.407039Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-rsa'
2023-11-10T01:45:47.407149Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T01:45:47.407240Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T01:45:47.542721Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T01:45:47.543378Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T01:45:47.543486Z [HoneyPotSSHTransport,11,172.105.128.13] Connection lost after 0 seconds
2023-11-10T01:45:47.676646Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 172.105.128.13:30074 (172.31.45.128:2222) [session: 754e45cba847]
2023-11-10T01:45:47.677393Z [HoneyPotSSHTransport,12,172.105.128.13] Remote SSH version: SSH-2.0-Go
2023-11-10T01:45:47.809986Z [HoneyPotSSHTransport,12,172.105.128.13] SSH client hassh fingerprint: 4e066189c3bbeec38c99b1855113733a
2023-11-10T01:45:47.811053Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ecdsa-sha2-nistp256'
2023-11-10T01:45:47.811160Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T01:45:47.811250Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T01:45:47.944981Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T01:45:47.945484Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T01:45:47.945608Z [HoneyPotSSHTransport,12,172.105.128.13] Connection lost after 0 seconds
2023-11-10T01:45:48.077052Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 172.105.128.13:30078 (172.31.45.128:2222) [session: 60897e87e2eb]
2023-11-10T01:45:48.077809Z [HoneyPotSSHTransport,13,172.105.128.13] Remote SSH version: SSH-2.0-Go
2023-11-10T01:45:48.220469Z [HoneyPotSSHTransport,13,172.105.128.13] SSH client hassh fingerprint: 4e066189c3bbeec38c99b1855113733a
2023-11-10T01:45:48.221490Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T01:45:48.221652Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T01:45:48.221758Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T01:45:48.355089Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T01:45:48.355622Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T01:45:48.355729Z [HoneyPotSSHTransport,13,172.105.128.13] Connection lost after 0 seconds
2023-11-10T02:01:11.197684Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 35.203.211.33:62186 (172.31.45.128:2222) [session: 67f551bd6e40]
2023-11-10T02:01:11.243444Z [HoneyPotSSHTransport,14,35.203.211.33] Remote SSH version: \xee\xea\xea_\x9dWr\xb2\x89\xf9_8<\x81\x8d\x8f\xadpzR\x89j\xfc\xb4\x9apȡ,\xcc[H \xc0}ԉ]m'\xa1\xa1\xc2u\x95br\xdeF\xfd1a+U=\xf7᪝\xce\xd53\xb0&\xc0+\xc0/\xc0,\xc00̨̩\xc0     \xc0\xc0
2023-11-10T02:01:11.244005Z [HoneyPotSSHTransport,14,35.203.211.33] Bad protocol version identification: b"\x16\x03\x01\x00\xee\x01\x00\x00\xea\x03\x03\xea_\x9d\x03Wr\xb2\x89\xf9_8<\x81\x8d\x8f\xadpzR\x1d\x89j\xfc\xb4\x9ap\xc8\xa1,\xcc[H \x17\xc0}\xd4\x89]m'\xa1\xa1\xc2u\x95br\xdeF\xfd1a+U\x0f=\xf7\xe1\xaa\x9d\xce\xd53\xb0\x00&\xc0+\xc0/\xc0,\xc00\xcc\xa9\xcc\xa8\xc0\t\xc0\x13\xc0"
2023-11-10T02:01:11.244331Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T02:01:11.244448Z [HoneyPotSSHTransport,14,35.203.211.33] Connection lost after 0 seconds
2023-11-10T02:01:11.622085Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 35.203.211.33:62190 (172.31.45.128:2222) [session: 4b78b0bea75f]
2023-11-10T02:01:11.637505Z [HoneyPotSSHTransport,15,35.203.211.33] Remote SSH version: \xca\xc6\xcb
2023-11-10T02:01:11.637991Z [HoneyPotSSHTransport,15,35.203.211.33] Bad protocol version identification: b'\x16\x03\x01\x00\xca\x01\x00\x00\xc6\x03\x03\xcb'
2023-11-10T02:01:11.638264Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T02:01:11.638369Z [HoneyPotSSHTransport,15,35.203.211.33] Connection lost after 0 seconds
2023-11-10T02:06:12.703333Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.33.87.154:40944 (172.31.45.128:2222) [session: 0a463e178514]
2023-11-10T02:06:12.995773Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T02:06:12.995965Z [HoneyPotSSHTransport,16,45.33.87.154] Connection lost after 0 seconds
2023-11-10T03:08:45.526307Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 104.128.67.17:53220 (172.31.45.128:2222) [session: a54e3192ca57]
2023-11-10T03:08:45.927789Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T03:08:45.928005Z [HoneyPotSSHTransport,17,104.128.67.17] Connection lost after 0 seconds
2023-11-10T03:58:35.531891Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 64.62.197.25:20245 (172.31.45.128:2222) [session: d81267d89b43]
2023-11-10T03:58:35.532635Z [HoneyPotSSHTransport,18,64.62.197.25] Remote SSH version: SSH-2.0-Go
2023-11-10T03:58:35.714954Z [HoneyPotSSHTransport,18,64.62.197.25] SSH client hassh fingerprint: 7216c7c473918b4f83d1139b3c70dbf9
2023-11-10T03:58:35.716276Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256@libssh.org' key alg=b'ecdsa-sha2-nistp256'
2023-11-10T03:58:35.716383Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T03:58:35.716473Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T03:58:35.897867Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T03:58:35.898239Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-userauth'
2023-11-10T03:58:36.078672Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'' trying auth b'none'
2023-11-10T03:58:39.532280Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T03:58:39.532486Z [HoneyPotSSHTransport,18,64.62.197.25] Connection lost after 3 seconds
2023-11-10T03:59:35.181965Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 162.142.125.221:42148 (172.31.45.128:2222) [session: cf530a5da44c]
2023-11-10T03:59:35.342223Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T03:59:35.342422Z [HoneyPotSSHTransport,19,162.142.125.221] Connection lost after 0 seconds
2023-11-10T03:59:35.501977Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 162.142.125.221:58598 (172.31.45.128:2222) [session: d7013eb48c74]
2023-11-10T03:59:35.502781Z [HoneyPotSSHTransport,20,162.142.125.221] Remote SSH version: SSH-2.0-Go
2023-11-10T03:59:35.685695Z [HoneyPotSSHTransport,20,162.142.125.221] SSH client hassh fingerprint: 873a5fb5fedc2d4f8638ebde4abc6cfc
2023-11-10T03:59:35.687015Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256@libssh.org' key alg=b'ecdsa-sha2-nistp256'
2023-11-10T03:59:35.687125Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T03:59:35.687217Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T03:59:35.899818Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T03:59:50.503555Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T03:59:50.503759Z [HoneyPotSSHTransport,20,162.142.125.221] Connection lost after 15 seconds
2023-11-10T04:15:09.987097Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 172.104.11.46:6782 (172.31.45.128:2222) [session: 2701864083cc]
2023-11-10T04:15:09.989430Z [HoneyPotSSHTransport,21,172.104.11.46] Remote SSH version: \x85\x81_\xa1i8\xbb\xc8\xfb\xc6]ѭG\xf5{\xdc\xd3u Z\xa8\x86G\xfcqy\x99\x9aY5\xecA\xee \xc0/\xc00\xc0+\xc0,̨̩\xc0\xc0 \xc0\xc0
2023-11-10T04:15:09.989944Z [HoneyPotSSHTransport,21,172.104.11.46] Bad protocol version identification: b'\x16\x03\x01\x00\x85\x01\x00\x00\x81\x03\x03_\xa1i8\xbb\xc8\xfb\xc6]\xd1\xadG\xf5{\xdc\xd3u Z\xa8\x86G\xfcqy\x99\x9aY5\xecA\xee\x00\x00 \xc0/\xc00\xc0+\xc0,\xcc\xa8\xcc\xa9\xc0\x13\xc0\t\xc0\x14\xc0'
2023-11-10T04:15:09.990230Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T04:15:09.990344Z [HoneyPotSSHTransport,21,172.104.11.46] Connection lost after 0 seconds
2023-11-10T06:38:46.736862Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 207.90.244.3:51572 (172.31.45.128:2222) [session: fe82e542597c]
2023-11-10T06:38:46.739626Z [HoneyPotSSHTransport,22,207.90.244.3] Remote SSH version: SSH-2.0-paramiko_2.11.0
2023-11-10T06:38:46.883531Z [HoneyPotSSHTransport,22,207.90.244.3] SSH client hassh fingerprint: a704be057881f0b1d623cd263e477a8b
2023-11-10T06:38:46.884830Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256@libssh.org' key alg=b'ssh-rsa'
2023-11-10T06:38:46.884986Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T06:38:46.885078Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T06:38:47.169132Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T06:38:47.313639Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 207.90.244.3:51574 (172.31.45.128:2222) [session: 1c5adc79ed10]
2023-11-10T06:38:47.314404Z [HoneyPotSSHTransport,23,207.90.244.3] Remote SSH version: SSH-2.0-paramiko_2.11.0
2023-11-10T06:38:47.455223Z [HoneyPotSSHTransport,23,207.90.244.3] SSH client hassh fingerprint: a704be057881f0b1d623cd263e477a8b
2023-11-10T06:38:47.456436Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256@libssh.org' key alg=b'ssh-ed25519'
2023-11-10T06:38:47.456548Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T06:38:47.456646Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T06:38:48.261333Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T06:38:48.261549Z [HoneyPotSSHTransport,22,207.90.244.3] Connection lost after 1 seconds
2023-11-10T06:38:51.079576Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T06:38:51.079777Z [HoneyPotSSHTransport,23,207.90.244.3] Connection lost after 3 seconds
2023-11-10T07:10:24.690078Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 172.104.11.51:28754 (172.31.45.128:2222) [session: 85c0ce759c79]
2023-11-10T07:10:24.690856Z [HoneyPotSSHTransport,24,172.104.11.51] Remote SSH version: SSH-2.0-Go
2023-11-10T07:10:24.822353Z [HoneyPotSSHTransport,24,172.104.11.51] SSH client hassh fingerprint: 4e066189c3bbeec38c99b1855113733a
2023-11-10T07:10:24.823377Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-rsa'
2023-11-10T07:10:24.823486Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T07:10:24.823584Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T07:10:24.956410Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T07:10:24.956920Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T07:10:24.957026Z [HoneyPotSSHTransport,24,172.104.11.51] Connection lost after 0 seconds
2023-11-10T07:10:25.089528Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 172.104.11.51:28764 (172.31.45.128:2222) [session: dcf29d328357]
2023-11-10T07:10:25.090282Z [HoneyPotSSHTransport,25,172.104.11.51] Remote SSH version: SSH-2.0-Go
2023-11-10T07:10:25.222667Z [HoneyPotSSHTransport,25,172.104.11.51] SSH client hassh fingerprint: 4e066189c3bbeec38c99b1855113733a
2023-11-10T07:10:25.223726Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ecdsa-sha2-nistp256'
2023-11-10T07:10:25.223832Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T07:10:25.223921Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T07:10:25.357239Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T07:10:25.357741Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T07:10:25.357847Z [HoneyPotSSHTransport,25,172.104.11.51] Connection lost after 0 seconds
2023-11-10T07:10:25.488276Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 172.104.11.51:28780 (172.31.45.128:2222) [session: 66e61f837861]
2023-11-10T07:10:25.489046Z [HoneyPotSSHTransport,26,172.104.11.51] Remote SSH version: SSH-2.0-Go
2023-11-10T07:10:25.620443Z [HoneyPotSSHTransport,26,172.104.11.51] SSH client hassh fingerprint: 4e066189c3bbeec38c99b1855113733a
2023-11-10T07:10:25.621459Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T07:10:25.621564Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T07:10:25.621652Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T07:10:25.753797Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T07:10:25.754293Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T07:10:25.754400Z [HoneyPotSSHTransport,26,172.104.11.51] Connection lost after 0 seconds
2023-11-10T08:50:52.358814Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 192.155.90.118:10980 (172.31.45.128:2222) [session: b34f9d402fd2]
2023-11-10T08:50:52.359711Z [HoneyPotSSHTransport,27,192.155.90.118] Remote SSH version: \x85\x81\xf4\xbc73\xeb\xeb\xec\xec*4\xe6-\xb2Ή\x94vxf40\xb6o\x85\xfd\x89`\xd6?\x9b\xd9 \xc0/\xc00\xc0+\xc0,̨̩\xc0\xc0      \xc0\xc0
2023-11-10T08:50:52.360200Z [HoneyPotSSHTransport,27,192.155.90.118] Bad protocol version identification: b'\x16\x03\x01\x00\x85\x01\x00\x00\x81\x03\x03\xf4\xbc73\xeb\xeb\xec\xec*4\x18\xe6-\xb2\xce\x89\x94v\x1b\xf40\xb6\to\x85\xfd\x89`\xd6?\x9b\xd9\x00\x00 \xc0/\xc00\xc0+\xc0,\xcc\xa8\xcc\xa9\xc0\x13\xc0\t\xc0\x14\xc0'
2023-11-10T08:50:52.360458Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T08:50:52.360561Z [HoneyPotSSHTransport,27,192.155.90.118] Connection lost after 0 seconds
2023-11-10T10:03:19.095795Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 167.248.133.124:58016 (172.31.45.128:2222) [session: b7f93520d88f]
2023-11-10T10:03:19.230104Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T10:03:19.230311Z [HoneyPotSSHTransport,28,167.248.133.124] Connection lost after 0 seconds
2023-11-10T10:03:19.364481Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 167.248.133.124:56550 (172.31.45.128:2222) [session: 5a43f2e52c6b]
2023-11-10T10:03:19.365208Z [HoneyPotSSHTransport,29,167.248.133.124] Remote SSH version: SSH-2.0-Go
2023-11-10T10:03:19.579371Z [HoneyPotSSHTransport,29,167.248.133.124] SSH client hassh fingerprint: 873a5fb5fedc2d4f8638ebde4abc6cfc
2023-11-10T10:03:19.580685Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256@libssh.org' key alg=b'ecdsa-sha2-nistp256'
2023-11-10T10:03:19.580794Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T10:03:19.580892Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T10:03:19.715924Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T10:03:34.377105Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T10:03:34.377315Z [HoneyPotSSHTransport,29,167.248.133.124] Connection lost after 15 seconds
2023-11-10T13:20:00.364802Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.79.172.21:55448 (172.31.45.128:2222) [session: 4258107cef30]
2023-11-10T13:20:00.442022Z [HoneyPotSSHTransport,30,45.79.172.21] Remote SSH version: \x85\x81=S\xc8!\xc3&䌈VB[d\x81\xb3cI
2023-11-10T13:20:00.442593Z [HoneyPotSSHTransport,30,45.79.172.21] Bad protocol version identification: b'\x16\x03\x01\x00\x85\x01\x00\x00\x81\x03\x03=\x1cS\xc8!\xc3&\xe4\x8c\x88VB[d\x19\x81\xb3cI'
2023-11-10T13:20:00.442898Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T13:20:00.442996Z [HoneyPotSSHTransport,30,45.79.172.21] Connection lost after 0 seconds
2023-11-10T13:20:01.082727Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.79.181.223:57766 (172.31.45.128:2222) [session: 0708e031db0f]
2023-11-10T13:20:01.083760Z [HoneyPotSSHTransport,31,45.79.181.223] Remote SSH version: SSH-2.0-Go
2023-11-10T13:20:01.216650Z [HoneyPotSSHTransport,31,45.79.181.223] SSH client hassh fingerprint: 4e066189c3bbeec38c99b1855113733a
2023-11-10T13:20:01.217688Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-rsa'
2023-11-10T13:20:01.217797Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T13:20:01.217887Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T13:20:01.352402Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T13:20:01.352916Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T13:20:01.353023Z [HoneyPotSSHTransport,31,45.79.181.223] Connection lost after 0 seconds
2023-11-10T13:20:01.483624Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.79.181.223:57770 (172.31.45.128:2222) [session: 5210576bb4e0]
2023-11-10T13:20:01.484389Z [HoneyPotSSHTransport,32,45.79.181.223] Remote SSH version: SSH-2.0-Go
2023-11-10T13:20:01.616882Z [HoneyPotSSHTransport,32,45.79.181.223] SSH client hassh fingerprint: 4e066189c3bbeec38c99b1855113733a
2023-11-10T13:20:01.617960Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ecdsa-sha2-nistp256'
2023-11-10T13:20:01.618074Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T13:20:01.618166Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T13:20:01.751375Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T13:20:01.751906Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T13:20:01.752011Z [HoneyPotSSHTransport,32,45.79.181.223] Connection lost after 0 seconds
2023-11-10T13:20:01.882498Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.79.181.223:57772 (172.31.45.128:2222) [session: 0dcf4226abdf]
2023-11-10T13:20:01.883219Z [HoneyPotSSHTransport,33,45.79.181.223] Remote SSH version: SSH-2.0-Go
2023-11-10T13:20:02.015185Z [HoneyPotSSHTransport,33,45.79.181.223] SSH client hassh fingerprint: 4e066189c3bbeec38c99b1855113733a
2023-11-10T13:20:02.016217Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T13:20:02.016325Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T13:20:02.016415Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes128-ctr' b'hmac-sha2-256' b'none'
2023-11-10T13:20:02.148755Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T13:20:02.149260Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T13:20:02.149368Z [HoneyPotSSHTransport,33,45.79.181.223] Connection lost after 0 seconds
2023-11-10T15:29:35.884665Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 104.152.52.139:50150 (172.31.45.128:2222) [session: 56f289b8d8ed]
2023-11-10T15:29:36.020454Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T15:29:36.020657Z [HoneyPotSSHTransport,34,104.152.52.139] Connection lost after 0 seconds
2023-11-10T16:47:08.995809Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 123.57.88.237:35072 (172.31.45.128:2222) [session: 87d8c27eb135]
2023-11-10T16:47:14.379335Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T16:47:14.379539Z [HoneyPotSSHTransport,35,123.57.88.237] Connection lost after 5 seconds
2023-11-10T16:47:21.192215Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 123.57.88.237:34786 (172.31.45.128:2222) [session: 9ef0230ed01d]
2023-11-10T16:47:21.937374Z [HoneyPotSSHTransport,36,123.57.88.237] Remote SSH version: SSH-2.0-libssh_0.7.4
2023-11-10T16:47:22.577796Z [HoneyPotSSHTransport,36,123.57.88.237] SSH client hassh fingerprint: e37f354a101aff5871ba233aa82b84ec
2023-11-10T16:47:22.578805Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256@libssh.org' key alg=b'ssh-ed25519'
2023-11-10T16:47:22.578909Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T16:47:22.579023Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T16:47:23.921126Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T16:47:24.299280Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-userauth'
2023-11-10T16:47:24.861755Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'NL5xUDpV2xRa' trying auth b'publickey'
2023-11-10T16:47:24.862076Z [HoneyPotSSHTransport,36,123.57.88.237] Unhandled Error
        Traceback (most recent call last):
          File "/home/cowrie/cowrie/cowrie-env/lib64/python3.9/site-packages/twisted/python/log.py", line 96, in callWithLogger
            return callWithContext({"system": lp}, func, *args, **kw)
          File "/home/cowrie/cowrie/cowrie-env/lib64/python3.9/site-packages/twisted/python/log.py", line 80, in callWithContext
            return context.call({ILogContext: newCtx}, func, *args, **kw)
          File "/home/cowrie/cowrie/cowrie-env/lib64/python3.9/site-packages/twisted/python/context.py", line 117, in callWithContext
            return self.currentContext().callWithContext(ctx, func, *args, **kw)
          File "/home/cowrie/cowrie/cowrie-env/lib64/python3.9/site-packages/twisted/python/context.py", line 82, in callWithContext
            return func(*args, **kw)
        --- <exception caught here> ---
          File "/home/cowrie/cowrie/cowrie-env/lib64/python3.9/site-packages/twisted/internet/posixbase.py", line 482, in _doReadOrWrite
            why = selectable.doRead()
          File "/home/cowrie/cowrie/cowrie-env/lib64/python3.9/site-packages/twisted/internet/tcp.py", line 248, in doRead
            return self._dataReceived(data)
          File "/home/cowrie/cowrie/cowrie-env/lib64/python3.9/site-packages/twisted/internet/tcp.py", line 253, in _dataReceived
            rval = self.protocol.dataReceived(data)
          File "/home/cowrie/cowrie/src/cowrie/ssh/transport.py", line 144, in dataReceived
            self.dispatchMessage(messageNum, packet[1:])
          File "/home/cowrie/cowrie/src/cowrie/ssh/transport.py", line 148, in dispatchMessage
            transport.SSHServerTransport.dispatchMessage(self, messageNum, payload)
          File "/home/cowrie/cowrie/cowrie-env/lib64/python3.9/site-packages/twisted/conch/ssh/transport.py", line 800, in dispatchMessage
            self.service.packetReceived(messageNum, payload)
          File "/home/cowrie/cowrie/cowrie-env/lib64/python3.9/site-packages/twisted/conch/ssh/service.py", line 50, in packetReceived
            return f(packet)
          File "/home/cowrie/cowrie/src/cowrie/ssh/userauth.py", line 72, in ssh_USERAUTH_REQUEST
            return userauth.SSHUserAuthServer.ssh_USERAUTH_REQUEST(self, packet)
          File "/home/cowrie/cowrie/cowrie-env/lib64/python3.9/site-packages/twisted/conch/ssh/userauth.py", line 173, in ssh_USERAUTH_REQUEST
            d = self.tryAuth(method, user, rest)
          File "/home/cowrie/cowrie/cowrie-env/lib64/python3.9/site-packages/twisted/conch/ssh/userauth.py", line 148, in tryAuth
            ret = f(data)
          File "/home/cowrie/cowrie/cowrie-env/lib64/python3.9/site-packages/twisted/conch/ssh/userauth.py", line 263, in auth_publickey
            algName, blob, rest = getNS(packet[1:], 2)
          File "/home/cowrie/cowrie/cowrie-env/lib64/python3.9/site-packages/twisted/conch/ssh/common.py", line 38, in getNS
            (l,) = struct.unpack("!L", s[c : c + 4])
        struct.error: unpack requires a buffer of 4 bytes

2023-11-10T16:47:24.863836Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T16:47:24.863936Z [HoneyPotSSHTransport,36,123.57.88.237] Connection lost after 3 seconds
2023-11-10T18:27:26.676657Z [cowrie.ssh.factory.CowrieSSHFactory] New connection: 45.160.94.238:25554 (172.31.45.128:2222) [session: e7764dcb8111]
2023-11-10T18:27:26.677407Z [HoneyPotSSHTransport,37,45.160.94.238] Remote SSH version: SSH-2.0-OpenSSH_8.8
2023-11-10T18:27:26.696440Z [HoneyPotSSHTransport,37,45.160.94.238] SSH client hassh fingerprint: b50f8371fb780ca7060e53c0b2cf6172
2023-11-10T18:27:26.697551Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] kex alg=b'curve25519-sha256' key alg=b'ssh-ed25519'
2023-11-10T18:27:26.697665Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] outgoing: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T18:27:26.697758Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] incoming: b'aes256-ctr' b'hmac-sha2-256' b'none'
2023-11-10T18:27:26.784360Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] NEW KEYS
2023-11-10T18:27:26.801277Z [cowrie.ssh.transport.HoneyPotSSHTransport#debug] starting service b'ssh-userauth'
2023-11-10T18:27:26.819605Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'none'
2023-11-10T18:27:28.592561Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' trying auth b'password'
2023-11-10T18:27:28.592911Z [HoneyPotSSHTransport,37,45.160.94.238] Could not read etc/userdb.txt, default database activated
2023-11-10T18:27:28.593077Z [HoneyPotSSHTransport,37,45.160.94.238] login attempt [b'ec2-user'/b'adsa'] failed
2023-11-10T18:27:29.594860Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] b'ec2-user' failed auth b'password'
2023-11-10T18:27:29.595082Z [cowrie.ssh.userauth.HoneyPotSSHUserAuthServer#debug] unauthorized login: ()
2023-11-10T18:27:30.377593Z [cowrie.ssh.transport.HoneyPotSSHTransport#info] connection lost
2023-11-10T18:27:30.377795Z [HoneyPotSSHTransport,37,45.160.94.238] Connection lost after 3 seconds
2023-11-10T18:39:28.298593Z [-] Received SIGTERM, shutting down.
2023-11-10T18:39:28.299105Z [-] (TCP Port 2222 Closed)
2023-11-10T18:39:28.299526Z [cowrie.ssh.factory.CowrieSSHFactory#info] Stopping factory <cowrie.ssh.factory.CowrieSSHFactory object at 0x7fccaf0c5310>
2023-11-10T18:39:28.299766Z [-] Main loop terminated.
2023-11-10T18:39:28.300239Z [twisted.scripts._twistd_unix.UnixAppLogger#info] Server Shut Down.


### LogsApache

[error_logs.txt](https://github.com/iudizm/instancia-segura-redes2UFSC/files/13573234/error_logs.txt)[Sun Nov 26 00:00:05.536877 2023] [ssl:warn] [pid 48616:tid 48616] AH01909: ip-172-31-45-128.sa-east-1.compute.internal:443:0 server certificate does NOT include an ID which matches the server name
[Sun Nov 26 00:00:05.537096 2023] [lbmethod_heartbeat:notice] [pid 48616:tid 48616] AH02282: No slotmem from mod_heartmonitor
[Sun Nov 26 00:00:05.538151 2023] [systemd:notice] [pid 48616:tid 48616] SELinux policy enabled; httpd running as context system_u:system_r:httpd_t:s0
[Sun Nov 26 00:00:05.539464 2023] [mpm_event:notice] [pid 48616:tid 48616] AH00489: Apache/2.4.56 (Amazon Linux) OpenSSL/3.0.8 configured -- resuming normal operations
[Sun Nov 26 00:00:05.539478 2023] [core:notice] [pid 48616:tid 48616] AH00094: Command line: '/usr/sbin/httpd -D FOREGROUND'
[Sun Nov 26 01:06:51.603882 2023] [proxy_fcgi:error] [pid 272949:tid 272993] [client 149.102.233.234:12813] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 01:36:35.326729 2023] [proxy_fcgi:error] [pid 272949:tid 272983] [client 20.70.235.244:50069] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 03:28:45.606803 2023] [proxy_fcgi:error] [pid 272949:tid 272990] [client 178.236.247.198:55850] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 05:38:10.522442 2023] [proxy_fcgi:error] [pid 272949:tid 272977] [client 156.146.35.172:24518] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 05:51:57.628227 2023] [proxy_fcgi:error] [pid 272949:tid 272983] [client 51.158.231.161:37856] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 06:40:18.909094 2023] [proxy_fcgi:error] [pid 272949:tid 272994] [client 20.235.243.146:63841] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 09:58:05.986415 2023] [proxy_fcgi:error] [pid 272949:tid 272991] [client 178.128.84.228:56031] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 11:27:47.712503 2023] [proxy_fcgi:error] [pid 272949:tid 272996] [client 92.223.85.235:10004] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 11:44:41.420823 2023] [proxy_fcgi:error] [pid 272949:tid 272996] [client 5.101.156.211:25267] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 14:02:29.030848 2023] [proxy_fcgi:error] [pid 272949:tid 272983] [client 178.128.84.228:57010] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 14:04:09.801010 2023] [proxy_fcgi:error] [pid 272949:tid 272981] [client 134.209.101.210:53564] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 14:06:13.887684 2023] [proxy_fcgi:error] [pid 272949:tid 272977] [client 40.74.69.181:63000] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 14:38:41.208897 2023] [proxy_fcgi:error] [pid 272949:tid 273001] [client 156.146.35.167:24251] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 16:44:18.437318 2023] [proxy_fcgi:error] [pid 272949:tid 272986] [client 34.32.44.189:51082] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 21:28:51.426381 2023] [proxy_fcgi:error] [pid 272949:tid 272989] [client 157.230.251.48:57507] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 21:36:29.902356 2023] [proxy_fcgi:error] [pid 272949:tid 272994] [client 95.142.121.53:2439] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 22:33:08.988551 2023] [proxy_fcgi:error] [pid 272949:tid 272982] [client 181.214.164.62:30220] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 22:58:44.265938 2023] [proxy_fcgi:error] [pid 272949:tid 272996] [client 156.146.35.179:7335] AH01071: Got error 'Primary script unknown'
[Sun Nov 26 23:15:53.857725 2023] [proxy_fcgi:error] [pid 272949:tid 272995] [client 209.126.81.117:58498] AH01071: Got error 'Primary script unknown'
[Mon Nov 27 00:00:49.833161 2023] [proxy_fcgi:error] [pid 272949:tid 272995] [client 156.146.35.174:59332] AH01071: Got error 'Primary script unknown'
[Mon Nov 27 02:58:52.465262 2023] [proxy_fcgi:error] [pid 272949:tid 272988] [client 169.150.220.130:4340] AH01071: Got error 'Primary script unknown'
[Mon Nov 27 05:19:53.263029 2023] [proxy_fcgi:error] [pid 272949:tid 272984] [client 20.235.243.146:59569] AH01071: Got error 'Primary script unknown'
[Mon Nov 27 06:55:19.575991 2023] [proxy_fcgi:error] [pid 272949:tid 272992] [client 212.102.51.228:26300] AH01071: Got error 'Primary script unknown'
[Mon Nov 27 07:53:24.955892 2023] [proxy_fcgi:error] [pid 272949:tid 272977] [client 185.180.12.115:25774] AH01071: Got error 'Primary script unknown'
[Mon Nov 27 17:36:08.373074 2023] [proxy_fcgi:error] [pid 272949:tid 272992] [client 20.82.151.237:52132] AH01071: Got error 'Primary script unknown'
[Mon Nov 27 22:37:29.993120 2023] [proxy_fcgi:error] [pid 269929:tid 270090] [client 160.153.250.108:37854] AH01071: Got error 'Primary script unknown'
[Mon Nov 27 23:46:30.707841 2023] [proxy_fcgi:error] [pid 272949:tid 272983] [client 20.70.235.244:63024] AH01071: Got error 'Primary script unknown'
[Tue Nov 28 00:33:04.613713 2023] [proxy_fcgi:error] [pid 272949:tid 272988] [client 92.223.86.29:51628] AH01071: Got error 'Primary script unknown'
[Tue Nov 28 03:53:42.140328 2023] [proxy_fcgi:error] [pid 272949:tid 272979] [client 92.223.85.153:16627] AH01071: Got error 'Primary script unknown'
[Tue Nov 28 05:23:52.567834 2023] [proxy_fcgi:error] [pid 272949:tid 272983] [client 45.141.215.138:62374] AH01071: Got error 'Primary script unknown'
[Tue Nov 28 05:24:05.737888 2023] [proxy_fcgi:error] [pid 272949:tid 272985] [client 45.141.215.138:63647] AH01071: Got error 'Primary script unknown'
[Tue Nov 28 06:22:15.950580 2023] [proxy_fcgi:error] [pid 272949:tid 272996] [client 64.227.139.153:57022] AH01071: Got error 'Primary script unknown'
[Tue Nov 28 06:22:19.838002 2023] [proxy_fcgi:error] [pid 269929:tid 270086] [client 64.227.139.153:56216] AH01071: Got error 'Primary script unknown'
[Tue Nov 28 06:22:22.663506 2023] [proxy_fcgi:error] [pid 272949:tid 272980] [client 64.227.139.153:56242] AH01071: Got error 'Primary script unknown'
[Tue Nov 28 06:22:23.363982 2023] [proxy_fcgi:error] [pid 269929:tid 270091] [client 64.227.139.153:56254] AH01071: Got error 'Primary script unknown'
[Tue Nov 28 06:32:49.288340 2023] [proxy_fcgi:error] [pid 272949:tid 272986] [client 156.146.35.166:27846] AH01071: Got error 'Primary script unknown'
[Tue Nov 28 08:29:24.116112 2023] [proxy_fcgi:error] [pid 272949:tid 272995] [client 178.128.84.228:56132] AH01071: Got error 'Primary script unknown'


