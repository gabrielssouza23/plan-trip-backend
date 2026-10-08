# plan-trip-backend

API de planejamento de viagens em grupo (**plann.er**). O organizador cria uma viagem, convida participantes por e-mail e o grupo monta junto a agenda de atividades e uma lista de links úteis (reservas, ingressos, etc.).

## Stack

- **Node.js** + **TypeScript**
- **Fastify 4** com **Zod** (`fastify-type-provider-zod`) para validação e tipagem das rotas
- **Prisma** ORM com **SQLite**
- **Nodemailer** para os e-mails de confirmação (usa contas de teste do [Ethereal](https://ethereal.email) em desenvolvimento)
- **Day.js** para datas

## Fluxo

1. `POST /trips` cria a viagem com o dono já confirmado e os convidados pendentes, e envia ao dono um e-mail com o link de confirmação.
2. Ao abrir `GET /trips/:tripId/confirm`, a viagem é confirmada e cada convidado recebe seu próprio link de confirmação.
3. `GET /participants/:participantId/confirm` confirma o participante.

Os links de confirmação redirecionam para o frontend (`WEB_BASE_URL`).

## Modelo de dados

```
Trip ─┬─< Participant   (name, email, is_confirmed, is_owner)
      ├─< Activity      (title, occurs_at)
      └─< Link          (title, url)
```

Schema em [`prisma/schema.prisma`](./prisma/schema.prisma), migrations em `prisma/migrations`.

## Endpoints

| Método | Rota | Descrição |
|---|---|---|
| POST | `/trips` | Cria viagem (`destination`, `starts_at`, `ends_at`, `owner_name`, `owner_email`, `emails_to_invite`) |
| GET | `/trips/:tripId` | Detalhes da viagem |
| PUT | `/trips/:tripId` | Atualiza destino e datas |
| GET | `/trips/:tripId/confirm` | Confirma a viagem e dispara os convites |
| POST | `/trips/:tripId/invites` | Convida um novo participante por e-mail |
| GET | `/trips/:tripId/participants` | Lista os participantes |
| GET | `/participants/:participantId` | Detalhes de um participante |
| GET | `/participants/:participantId/confirm` | Confirma presença do participante |
| POST | `/trips/:tripId/activities` | Cria atividade (`title`, `occurs_at`) |
| GET | `/trips/:tripId/activities` | Atividades agrupadas por dia da viagem |
| POST | `/trips/:tripId/links` | Adiciona link (`title`, `url`) |
| GET | `/trips/:tripId/links` | Lista os links |

### Regras e erros

- Datas validadas no servidor: a viagem não pode começar no passado nem terminar antes de começar, e atividades precisam cair dentro do período da viagem.
- Um error handler central converte erros do Zod em `400` com os campos inválidos e erros de regra de negócio (`ClientError`) em `400` com a mensagem.

## Rodando localmente

Pré-requisito: Node.js 20.6+ (o script `dev` usa `--env-file`).

```bash
npm install
cp .env.example .env
npx prisma migrate dev
npm run dev
```

A API sobe em `http://localhost:3333`. Os links dos e-mails enviados aparecem no Ethereal; para inspecionar o banco, use `npx prisma studio`.

### Variáveis de ambiente

| Variável | Exemplo | Uso |
|---|---|---|
| `DATABASE_URL` | `file:./dev.db` | Banco SQLite |
| `API_BASE_URL` | `http://localhost:3333` | Base dos links de confirmação nos e-mails |
| `WEB_BASE_URL` | `http://localhost:3000` | Para onde as confirmações redirecionam |
| `PORT` | `3333` | Porta da API |
