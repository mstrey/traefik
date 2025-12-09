# Traefik

Projeto docker para rodar o traefik com gerenciamento DDNS pela Cloudflare e certificados assinados dinamicamente por LetsEncrypt.

Copie o arquivo `.env_example` renomeando para `.env`.  
Preencha a variável `DOMAIN` com o domninio gerenciado pela CloudFlare

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

# CloudFlare

## Token
Acesse o painel de [Dashboard de tunnel da CloudFlare](https://one.dash.cloudflare.com) e acesse o menu `Networks >> Connector`.

Clique em `Create` e na guia `Overview` preencha um nome para o seu tunnel. Exemplo: `my-homelab`

Clique sobre o item `Docker` e na linha de comando que for exibida copie apenas o token, que é a sequência gigante de caracteres que fica depois do parametro `--token`   
Cole o valor no parametro `CF_TUNNEL_TOKEN` do arquivo `.env`.

## Roteamento

Primeiro vamos criar a rota principal, que vai conectar o traefik. Na guia `Published application routes` clique em `Add a published application route` para criar uma nova rota e informe os seguintes valores:   

- Subdomain: traefik
- Domain: escolha o seu domínio
- Path: deixe em branco
- Type: HTTP
- URL: traefik:80 (Sim, é só o nome do container e a porta. o Docker vai resolver esse nome do container)

Agora vamos criar uma segunda rota para tratar as demais rotas, assim não será necessário criar uma rota para cada serviço:

- Subdomain: *
- Domain: escolha o seu domínio
- Path: deixe em branco
- Type: HTTP
- URL: traefik:80 (Sim, é só o nome do container e a porta. o Docker vai resolver esse nome do container)

## Limpeza

Agora você pode remover as rotas do tipo `A` (IPv4) e `AAAA` (IPv6). O túnel não usa o seu IP público. Manter esses registros é um risco de segurança desnecessário, pois revela o IP da sua casa para scanners na internet.

## Throwable shooting

Caso as rotas adicionais não estejam funcionando, reveja as rotas de DNS. Acesse o [dashboard da cloudflare](https://dash.cloudflare.com/), acesse o seu domínio e vá no menu `DNS > Records`.

Procure a rota `CNAME` do traefik e copie o valor de `content` para a rota `CNAME` de `*`.
