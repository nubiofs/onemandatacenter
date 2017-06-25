# Systemctl

* **Resumo**
 * Saem: service, chckonfig, logs
 * Entram: systemctl, journalctl

## Básico

A primeira mudança: o CentOS 7 foi tomado por um negócio chamado **systemd**.

**Tudo**, praticamente **tudo** agora é systemd.

Isso quer dizer que, para levantar e parar serviços, ou marcar serviços para inicialização, deve-se usar o comando **systemctl** em vez dos antigos comandos *service* e *chkconfig*.

No CentOS 6:
```
# # Ativa o serviço Apache:
# service httpd start

# # Marca o serviço Apache para inicialização:
# chkconfig httpd on
```

No CentOS 7:
```
# systemctl start httpd

# systemctl enable httpd
```

Agora o comando *status* é muito mais completo:

```
systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
   Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enab
   Active: active (running) since Dom 2017-03-19 17:10:00 BRT; 7h ago
  Process: 17668 ExecReload=/bin/kill -HUP $MAINPID (code=exited, status=0/SUCCE
 Main PID: 1297 (sshd)
    Tasks: 1
   Memory: 1.5M
      CPU: 119ms
   CGroup: /system.slice/ssh.service
           └─1297 /usr/sbin/sshd -D

Mar 20 00:02:01 heaven systemd[1]: Reloading OpenBSD Secure Shell server.
Mar 20 00:02:01 heaven sshd[1297]: Received SIGHUP; restarting.
Mar 20 00:02:01 heaven systemd[1]: Reloaded OpenBSD Secure Shell server.
Mar 20 00:02:01 heaven sshd[1297]: Server listening on 0.0.0.0 port 22.
Mar 20 00:02:01 heaven sshd[1297]: Server listening on :: port 22.
Mar 20 00:02:01 heaven systemd[1]: Reloading OpenBSD Secure Shell server.
Mar 20 00:02:01 heaven sshd[1297]: Received SIGHUP; restarting.
Mar 20 00:02:01 heaven systemd[1]: Reloaded OpenBSD Secure Shell server.
Mar 20 00:02:01 heaven sshd[1297]: Server listening on 0.0.0.0 port 22.
Mar 20 00:02:01 heaven sshd[1297]: Server listening on :: port 22.

```

Mas a melhor maneira de acompanhar a saída dos daemons ainda são os logs. Os logs continuam a mesma coisa, não é? 

A resposta é: **não**. Os logs foram dominados pelo systemd. Você deve dominar o programa **journalctl** para extrair as informações que você precisa.

A melhor maneira de ler os logs do serviço sshd mencionado acima é usar o journalctl da forma abaixo:

```
journalctl -fu sshd
```

Você deve explorar também outros parâmetros desse comando, como *-e* e *-x*.

# SELinux

semanage port -a -t dns_port_t -p tcp 8053

# Firewalld

O Firewall padrão do CentOS 7 continua sendo o **iptables**. Para *facilitar* seu uso, foi implementado um daemon chamado **Firewalld** que permite gerenciar o uso de maneira mais intuitiva.

Mais intuitiva se você entender seus conceitos, é verdade, que parecem bem confusos. 

De qualquer forma, a ideia é que deve-se evitar chamar os comandos iptables diretamente "na mão", buscando as implementações Firewalld correspondentes.


# Acesso SSH

O acesso SSH a servidores agora é restrito a usuários com chaves válidas. Não existe mais qualquer tipo de acesso com senhas. Entretanto, o conceito de *chave válida* vai além do tradicional uso de chaves por meio da publicação de chave pública em um arquivo authorized_keys: é necessário possuir uma chave assinada pela **Autoridade Certificadora de Chaves** criadas exclusivamente para esta finalidade.

A ideia é simples: você gera um par de chaves para seu uso pessoal na sua máquina, devidamente protegendo sua private key com uma passphrase difícil. Você envia a sua chave pública para o administrador da CA SSH, que fará uma avaliação da sua solicitação e oferecerá sua chave pública de volta, agora assinada, na forma de um certificado, oferecendo os acessos aos servidores que você tem direito.

Todo o processo é transparente e você automaticamente acessará os servidores quando estiver de posse de tal certificado.

 


