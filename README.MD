# Traefik

Projeto docker para rodar o traefik com gerenciamento DDNS e certificados pela Cloudflare.

## 🚀 Componentes da Stack
Fazem parte da solução de gerenciamento os seguintes serviços:

- **Cloudflared**: Cria um túnel seguro (Zero Trust) direto para a nuvem da Cloudflare, dispensando a abertura de portas no roteador local.

- **Traefik v3**: Proxy reverso que intercepta o tráfego do túnel e distribui para os containers corretos, gerenciando autenticação e redirecionamentos.

- **Crowdsec**: Sistema de Prevenção de Intrusão (IPS) colaborativo que analisa os logs do Traefik em tempo real e bloqueia ataques.

- **Error Pages**: Servidor super leve para exibir páginas de erro amigáveis caso um container esteja offline.

- **Homer**: Dashboard estático, rápido e de baixíssimo consumo para centralizar os links de acesso da sua infraestrutura local.

- **Netdata**: Agente de monitoramento em tempo real (APM) para análise profunda de saúde do hardware e dos containers.

# 📋 1. Preparação do ambiente 
Copie o arquivo de exemplo para criar o seu arquivo de variáveis de ambiente:

```bash
cp .env_example .env
```

Abra o arquivo `.env` e preencha as configurações do nó:

- NODE_NAME: Nome de identificação do servidor que será gerenciado (ex: `poa-rasp`).

- DOMAIN: Seu domínio principal gerenciado pela Cloudflare (ex: dominio.com).

SUBDOMAIN: O identificador deste nó (ex: spo, poa).

(Os URLs gerados seguirão o padrão servico-subdominio.dominio, ex: traefik-poa.dominio.com, para garantir a compatibilidade com o SSL Universal da Cloudflare).

# 🔐 2. Geração de Credenciais

## Senha do Dashboard (BASIC_AUTH)

O Traefik exige uma autenticação para os painéis gerenciais. Gere o hash da sua senha executando:

```
htpasswd -nb admin sua_senha
admin:$apr1$USMRR8qF$DHzFT0lhzmO46CkBh9DaS1
```
Importante: No resultado gerado, você deve dobrar todas as ocorrências do caractere `$`.
No exemplo retornado acima, o valor que deve ser colado no arquivo `.env` é:

`BASIC_AUTH=admin:$$apr1$$USMRR8qF$$DHzFT0lhzmO46CkBh9DaS1`

## Token do Cloudflare Tunnel

1. Acesse o [Dashboard da CloudFlare](https://one.dash.cloudflare.com) e acesse o menu `Networks >> Connector`.
2. Clique em `Create` e na guia `Overview` preencha um nome para o seu tunnel. Exemplo: `my-homelab`
3. Clique sobre o item `Docker` e na linha de comando que for exibida copie apenas o token, que é a sequência gigante de caracteres que fica depois do parametro `--token`   
4. Cole o valor no parametro `CF_TUNNEL_TOKEN` do arquivo `.env`.

## Token do Crowdsec (Bouncer)

Como o Traefik depende do Crowdsec para validar os acessos, precisamos gerar o token do bouncer antes de subir a stack completa.

Inicie apenas o container do Crowdsec:

```bash 
docker compose up -d crowdsec
```

Aguarde alguns segundos e gere o token:

```bash 
docker exec -t crowdsec cscli bouncers add traefik-bouncer
```

Copie o token retornado e cole na variável `TOKEN_BOUNCER` no arquivo `.env`.

# 📦 3. Execução

Com o .env devidamente preenchido e todos os tokens configurados, inicie toda a infraestrutura:

```bash
docker compose up -d
```

# 🌐 4. Roteamento na Cloudflare

No [Painel Zero Trust](https://one.dash.cloudflare.com/), vá até a configuração do seu túnel e acesse a aba Public Hostname. Adicione as rotas apontando para o serviço interno http://traefik:80.

Crie as rotas para os serviços nativos deste nó (substitua poa pelo seu SUBDOMAIN):

traefik-poa.strey.net.br ➔ http://traefik:80

home-poa.strey.net.br ➔ http://traefik:80

netdata-poa.strey.net.br ➔ http://traefik:80

Para qualquer novo sistema que você adicionar futuramente neste servidor e mapear no Traefik, basta criar uma nova rota com o respectivo subdomínio apontando para o mesmo http://traefik:80.

# 🖥️ 5. Configurando o Dashboard (Homer)

O Homer é um dashboard estático gerenciado via código (IaC). Ao rodar a stack pela primeira vez, uma pasta homer será criada no repositório local.

Para adicionar seus links, copie o arquivo `homer/config_example.yml` para `homer/config.yml` e edite as urls para o seu subdominio e dominio. 

As alterações no config.yml refletem instantaneamente no navegador ao atualizar a página, sem necessidade de reiniciar o container.

# 🛡️ 6. Segurança e Limpeza

## WAF da Cloudflare

Para aumentar a segurança global, acesse o [Dashboard da Cloudflare](https://dash.cloudflare.com/) e acesse o menu `Security > WAF`.
Crie uma regra "Block Bad Guys":

Field: Country

Operator: is not in

Value: Selecione apenas os países de onde você acessa sua infraestrutura (ex: Brazil).

Action: Managed Challenge (Apresenta um desafio invisível ou CAPTCHA para acessos estrangeiros).

## Limpeza

Agora você pode remover as rotas do tipo `A` (IPv4) e `AAAA` (IPv6). O túnel não usa o seu IP público. Manter esses registros é um risco de segurança desnecessário, pois revela o IP da sua casa para scanners na internet.

# 🚑 Troubleshooting (Resolução de Problemas)

- **Erro `ERR_SSL_VERSION_OR_CIPHER_MISMATCH` no navegador**:
Isso ocorre se você tentar usar uma URL de 4º nível (ex: `traefik.poa.dominio.com`). O SSL gratuito da Cloudflare só cobre até o 3º nível (`*.dominio.com`). Mantenha o padrão com hífen (`traefik-poa.dominio.com`).

- **O serviço não aparece no Traefik**:
Verifique se o container do serviço está conectado à rede externa frontend no seu respectivo docker-compose.yml. Caso seu container utilize mais de uma rede você deve especificar nas labels qual a rede que o traefik deve utilizar:

```yml 
      - "traefik.docker.network=frontend"
```
