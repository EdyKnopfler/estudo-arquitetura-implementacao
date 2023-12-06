# Estudo em arquitetura: implementação

Implementação do projeto planejado em https://github.com/EdyKnopfler/architecture-studies/. Para entendimento deste é indispensável (por enquanto) a leitura das provas de conceito documentadas ali, pois este é um estudo em andamento.

Com o tempo as informações de lá serão incorporadas a _esta_ documentação.

## Decisões de projeto

### Frontend de reservas

Embora desenvolvido usando o framework Next.js, o qual permite usar componentes React processados em servidor, foi feita a decisão de gerenciar o estado das sessões de usuário (quais pré-reservas estão feitas) a partir do cliente: este só vai ao servidor para solicitar a alteração da informação e caso a transação ocorra com sucesso o `localStorage` é alterado.

Dessa forma, a cada mudança de tela não é preciso recarregar um estado que o cliente tem condições de gerenciar por si.

Quanto às expirações de sessão (_timeouts_), inicialmente serão tratadas durante as chamadas ao servidor que falharem. O mecanismo de trava (_lock_, explicado nas provas de conceito) deve garantir que a alteração das sessões pelo mecanismo de expiração e pelas ações do usuário ocorram de forma ordenada, sem que uma provoque conflito com a outra. A tentativa de expirar uma sessão que está sendo confirmada ou vice-versa deve devolver um erro.

### Backend de reservas

Como experimentado nas provas de conceito (ver repositório linkado), o gerenciamento de sessões dos usuários será realizada por dois serviços, a [API da sessão](https://github.com/EdyKnopfler/estudo-arquitetura-reservas-backend) em si e o serviço de timeouts _(a implementar)_.

O motivo para esta escolha é poder escalar a API de forma independente e permitir ao serviço de timeouts ter alocados para si os recursos para realizar sua tarefa, que é uma tarefa a ser realizada periodicamente em lote: identificar sessões expiradas e realizar os cancelamentos de pré-reservas realizadas, sem competir por recursos com a API.

Também foram consideras (e descartadas as opções):

* _unir tudo no mesmo serviço:_ escalá-lo implicaria disparar mais ocorrências das tarefas em lote de forma desnecessária, se não for feito um gerenciamento de qual instância ficaria responsável por essa tarefa (complexidade acidental desnecessária)
* _o serviço de timeout fazer uma chamada para a API de sessão:_ a tarefa em lote pode não ter garantidos os recursos de forma imediata sempre que precise rodar

O código de manipulação das sessões usável pelos dois serviços será disponibilizado como bibilioteca _em projeto a implementar_. 

## Construir e rodar

### Desenvolvimento

Os projetos Java podem ser rodados diretamente do IDE. Sua configuração default acessam os bancos de dados MySQL e Redis e o broker RabbitMQ em `localhost` nas portas padrão `3306`, `6379` e `5672`, respectivamente. Esses bancos de dados podem ser iniciados a partir do `docker-compose.yml` na pasta raiz deste repositório:

```bash
# database: base de dados principal (MySQL)
# cache: base de dados de cache e gerenciamento de travas (Redis)
# broker: sistema de mensageria (RabbitMQ)
# -d (opcional): roda em background (daemon)
docker compose up database cache broker [-d]
```

### Imagens Docker

**(Em construção!)**

Imagem do backend das reservas:

```bash
./build-images.sh
cd ..
```

Subindo os contêineres:

```bash
# No diretório raiz
docker compose up [-d]
```
