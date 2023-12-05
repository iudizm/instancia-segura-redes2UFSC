# A fazer
(16/11)

* [x] Criar usuarios no sistema diferentes para cada usuario conectar-se via SSH.
* [X] restringir porta 80 na AWS para os IPs solicitados pelo professor.
* [x] Configurar o NTP para sincronizar o relógio da instância com o horário de Brasília.
* [x] Ativar o SELINUX - Sistema de acesso mandatorio na instância.
* [x] Escolher 2 tecnicas de seguranca para implementar, sgundo a documentacao do CIS Benchmark
* [x] Extrair os logs do servidor conforme solicitado.
* [ ] Analisar o resultado dos logs


* Para melhorar a certificacao SSL:
  * [x] Definir um IP elástico para a instância.
  * [x] Criar um domínio para o IP elástico. (FREE TIERS) ex: _.tech_
  * [x] Configurar o domínio para apontar para o IP elástico.
  * [x] Gerar um certificado no LetsEncrypt. Com o _certbot_
  * [x] Revisar o Firewall para liberar apenas as portas necessárias.
