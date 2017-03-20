# Systemctl

A primeira mudança: o CentOS 7 foi tomado por um negócio chamado **systemd**.

**Tudo**, praticamente **tudo** agora é systemd.

Isso quer dizer que, para levantar e parar serviços, ou marcar serviços para inicialização, deve-se usar o comando **systemctl** em vez dos antigos comandos service e chkconfig.

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

