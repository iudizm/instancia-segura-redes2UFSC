# Relatório do Projeto de Instância Segura em Nuvem

**UFSC 2023/2**

Redes de Computadores II    
Professor [Gerson Luiz Camillo](https://github.com/glcamillo)

Grupo: Iudi e Michele

## Sumário

1. [Provedor](#provedor)
2. [MFA](#mfa)
3. [Grupos](#grupos)
4. [Instancia](#instancia)
5. [Chaves pública/privada](#chaves-públicaprivada)
7. [Apache2](#apache2)
8. [CA](#ca)
9. [Firewall](#firewall)
10. [FirewallInst](#firewallinst)
11. [Restrições](#restrições)
12. [MAC](#mac)
13. [Tecnicas](#tecnicas)
14. [Logs](#logs)
15. [LogsApache](#logsapache)
16. [NTP](#ntp)

## Provedor

Inicialmente, tentamos utilizar a Azure para realizar o trabalho, entretanto, enfrentamos dificuldades com a conta da microsoft na Azure, que estava deslogando automaticamente na criação da instância Linux.

Fomos então tentar usar o serviço da AWS e tudo ocorreu bem na criação da conta e da instância, portanto foi a escolhida para o trabalho, utilizando o *Free Tier*.

## MFA
_Responsável: Iudi_

Usuário root ativou a autenticação multifatorial, utilizando o Google Authenticator.

## Grupos IAM
_Responsável: Iudi_

O usuário root criou dois grupos, um para os gerentes da EC2 e outro para o usuário "cebola".

1. **Grupo de Gerentes**

    O grupo de gerentes foi criado para que os usuários que o compõem possam realizar ações administrativas na instância, como instalar pacotes, criar usuários, etc.

    Política de permissões do grupo: `AmazonEC2FullAccess`

2. **Grupo Cebola**

    O grupo cebola foi criado para que o usuário "cebola", possa ver e ler as informações na instância, como ler arquivos de log, por exemplo.

    Política de permissões do grupo: `AmazonEC2ReadOnlyAccess`

## Instância EC2
_Responsável: Iudi_

Especificações:
EC2 t2.micro | Amazon Linux 2023

Nome da instância: Curupira

## Chaves pública/privada
_Responsável: Iudi_

Com o par de chaves criado automaticamente pelo AWS, foi feita a conexão com o servidor:

![](imagens/ssh_chave_padrao.png)

Adicionadas as chaves públicas no `authorized_keys`, conexão com a chave ssh própria:

![](imagens/ssh_iudi.png)

## Apache & HTTP

## Firewall

## CA & HTTPS

## Restrições de Acesso

Inicialmente acesso liberado para as portas 80 e 443.   
Acesso restrito a um IP específico para a porta 22.

Posteriormente, restrita a porta 22 para rede da UFSC.

## MAC

## Tecnicas

## Logs

## LogsApache

## NTP
