# Traefik

Projeto docker para rodar o traefik com gerenciamento DDNS pela Cloudflare e certificados assinados dinamicamente por LetsEncrypt.

Copie o arquivo `.env_example` renomeando para `.env`.  
Preencha a variável `DOMAIN` com o domninio gerenciado pela CloudFlare

# CloudFlare

Acesse o painel de [Dashboard de tunnel da CloudFlare](https://one.dash.cloudflare.com) e acesse o menu `Networks >> Connector`.

Na guia `Overview` preencha um nome para o seu tunnel e clique sobre o item `Docker`:

Na linha de comando que for exibida copie apenas o token, que é a sequência gigante de caracteres que fica depois do parametro `--token`   
Cole o valor do parametro `CF_TUNNEL_TOKEN` do arquivo `.env`.

# Senha Dashboard

Execute o comando abaixo no prompt para gerar o hash de sua senha que será utilizada para acessar o dashboard do Traefik. 
Esse resultado deve ser colado na variável `BASIC_AUTH` do arquivo .env.

No retorno do método altere a string dobrando as 3 ocorrências de `$`. Exemplo:

```
htpasswd -nb admin sua_senha
admin:$apr1$USMRR8qF$DHzFT0lhzmO46CkBh9DaS1
```

O valor que deve ser colado no arquivo `.env` é:

`BASIC_AUTH=admin:$$apr1$$USMRR8qF$$DHzFT0lhzmO46CkBh9DaS1`