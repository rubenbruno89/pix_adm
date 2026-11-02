# Cobrança Pix

Gerador de cobranças via Pix para enviar pelo WhatsApp. Informe cliente, descrição, valor e a chave Pix; o app gera o código **Pix copia e cola**, o **QR Code** e um **link público** da cobrança, pronto para compartilhar.

## Funcionalidades

- Criação de cobranças com cliente, WhatsApp (opcional), descrição, valor e chave Pix informada na hora (não há chave fixa)
- Código Pix copia e cola no padrão BR Code (EMV) com CRC16, gerado no navegador
- QR Code gerado no navegador, sem serviços externos
- Página pública de cada cobrança (`/c/<hash>`), sem login, com QR Code e botão de copiar
- Botão que abre o WhatsApp com a mensagem e o link da cobrança
- Histórico com busca, filtro (todas, pendentes, pagas) e rolagem infinita
- Status pago/pendente marcado manualmente
- Painel com valor a receber, valor recebido e total de cobranças
- Multiusuário: cada pessoa entra com a própria conta e vê apenas as suas cobranças

## Tecnologias

- React 19, TypeScript, Vite e Tailwind CSS 4
- react-router-dom e lucide-react
- Banco PostgreSQL e Server Functions SQL
- Login via Google ou Microsoft

## Estrutura

```
frontend/
  src/
    App.tsx                    rotas e controle de sessão
    pages/
      LoginPage.tsx            tela de entrada
      Painel.tsx               painel, histórico e nova cobrança
      CobrancaPublica.tsx      página pública da cobrança
    components/
      CobrancaDetalhe.tsx      QR Code, copia e cola e compartilhamento
      QrCode.tsx               renderização do QR Code em SVG
      ui/                      componentes de interface
    lib/
      pix.ts                   geração do código Pix (BR Code) e formatação
      qr.ts                    gerador de QR Code
      api.ts                   chamadas às Server Functions
      sf-ids.ts                identificadores das Server Functions
      mitra-auth.ts            autenticação
```

## Banco de dados

Tabela `cobrancas`:

| Coluna | Descrição |
|---|---|
| `id` | identificador sequencial |
| `hash` | identificador aleatório usado no link público |
| `usuario_id` | dono da cobrança |
| `cliente_nome`, `cliente_telefone` | dados do cliente |
| `descricao`, `valor` | o que está sendo cobrado |
| `chave_pix`, `recebedor_nome`, `recebedor_cidade` | dados do recebedor |
| `codigo_pix` | código copia e cola gerado |
| `status`, `pago_em` | `pendente` ou `pago` e a data do pagamento |
| `criado_em` | data de criação |

## Server Functions

- `cobrancas_listar`: lista paginada, com busca e filtro de status
- `cobrancas_resumo`: totais e valores para o painel
- `cobrancas_criar`: cria a cobrança
- `cobrancas_status`: marca como paga ou pendente
- `cobrancas_excluir`: exclui uma cobrança
- `cobranca_publica`: lê uma cobrança pelo `hash` (acesso público)

Todas, exceto `cobranca_publica`, filtram pelo usuário logado.

## Como funciona o Pix

O código segue o padrão BR Code do Banco Central: chave Pix, valor, nome e cidade do recebedor, com validação por CRC16. O pagamento é feito direto para a chave informada; o app não intermedeia o dinheiro e não confirma pagamentos, por isso o status é marcado manualmente.

## Desenvolvimento

```bash
cd frontend
npm install
npx tsc -b --noEmit   # checagem de tipos
```

As variáveis `VITE_MITRA_APP_ID` e `VITE_MITRA_API_URL` são fornecidas pela plataforma onde o app roda.
