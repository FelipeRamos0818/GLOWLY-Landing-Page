# 🌸 Glowly — Documentação do Projeto

> Leia este arquivo no início de cada nova conversa para retomar o contexto exato de onde paramos.

---

## 🧭 O que é a Glowly

Plataforma SaaS de agendamentos inteligentes para profissionais de beleza.  
Combina agenda online + assistente humanizada no WhatsApp que confirma horários, responde clientes e reduz faltas automaticamente.

**Stack:**
- Frontend/plataforma: Lovable (React + Tailwind + shadcn/ui)
- Banco de dados: Supabase (projeto: `emxbjuahedbnanzkgpyc`)
- Automação: n8n (self-hosted em VPS Hostinger)
- WhatsApp: Evolution API
- IA: Claude API (Anthropic)
- Landing page: HTML estático hospedado no Netlify

---

## 👤 Contexto do desenvolvedor

- Felipe — empreendedor, não técnico, usa Claude como parceiro de desenvolvimento
- Ferramentas principais: Lovable (plataforma), Cursor + Claude Code (código), n8n (automações)
- Projeto de teste chamado **"Bellas"** — simula um salão real para validar os fluxos

---

## ✅ O que já está feito

### Plataforma (Lovable)
- [x] Autenticação admin
- [x] Agenda com visualização por dia e semana
- [x] Cadastro de serviços com duração
- [x] Cadastro de horários de atendimento
- [x] Bloqueio de horários
- [x] Gestão de clientes
- [x] Página pública de agendamento (link para clientes)
- [x] Status de agendamento: `pending_confirmation`, `confirmed`, `cancelled`, `completed`
- [x] Edge Function `update-appointment-status` no Supabase (resolve problema de RLS)

### Banco de dados (Supabase)
- [x] Tabela `appointments` com campos: `id`, `status`, `client_whatsapp`, `appointment_date`, `start_time`, `service_id`
- [x] Edge Function deployada: `update-appointment-status`
  - URL: `https://emxbjuahedbnanzkgpyc.supabase.co/functions/v1/update-appointment-status`
  - Secret: `bellas_n8n_secret_2026`
  - Aceita: `confirmed`, `cancelled`, `pending_confirmation`

### n8n — Fluxo 1: Lembrete diário (08h)
- [x] Trigger: todo dia às 08h
- [x] Busca agendamentos do dia no Supabase
- [x] Agrupa por cliente e envia lembrete via WhatsApp (Evolution API)
- [x] Nó "Code in JavaScript" separa IDs individualmente
- [x] HTTP Request marca status como `pending_confirmation` via edge function
  - **Correção aplicada:** filtro `appointment_date=eq.{{ new Date().toISOString().slice(0,10) }}` para não pegar agendamentos antigos

### n8n — Fluxo 2: Confirmação via WhatsApp
- [x] Trigger: Webhook Evolution API (recebe mensagens)
- [x] Verifica se Pamela (humano) assumiu o atendimento via Redis
- [x] Normaliza o status da mensagem (SIM/NÃO)
- [x] Busca agendamento pendente do cliente no Supabase
- [x] Nó "SIM ou NÃO?" roteia para confirmar ou cancelar
- [x] "Confirmar Agendamento": POST para edge function com `status: confirmed`
- [x] "Cancelar Agendamento": POST para edge function com `status: cancelled`
- [x] Responde confirmação/cancelamento para o cliente no WhatsApp
- **Correção aplicada:** body usa `Using Fields Below` com `appointment_id: {{ $('Buscar Agendamento Pendente').first().json.id }}`

### Assistente WhatsApp (Luna)
- [x] Prompt configurado no n8n (nó de IA)
- [x] Nome: Luna
- [x] Tom: humanizado, feminino, próximo
- [x] Envia link de agendamento sem formatação Markdown
- [x] Não revela que é IA a menos que necessário
- **Última versão do prompt:** inclui regra explícita de nunca usar `[texto](link)` e link sempre em linha separada

### Landing page
- [x] Design futurista e feminino (fundo escuro, gradientes rosa/violeta)
- [x] Fontes: Outfit + Space Grotesk
- [x] Seções: Hero, Pain point (ancoragem), Como funciona, Funcionalidades, Demo WhatsApp, Para quem, Salões, Preços, CTA, Footer
- [x] Planos: R$149/mês (Agenda Inteligente) e R$299/mês (Agenda + IA Completa)
- [x] Bloco de salão mostra fluxo de escolha de profissional
- Nome atual na landing: **Glowly** (nome de trabalho — ainda não definitivo)
- Arquivo: `landing-futurista.html`

---

## 🔧 Configurações técnicas importantes

### Evolution API (WhatsApp)
- VPS: Hostinger
- Instância conectada ao número da Pamela (dona do salão teste)

### n8n — detalhes críticos
- Redis usado para controle de atendimento humano
  - Conta correta: `Formacao_2.0 1` (AWS sa-east-1)
  - Chaves no formato `@lid_status`
- Variável de referência do nó de busca: `$('Buscar Agendamento Pendente').first().json.id`
- Campo correto de data: `appointment_date` (não `start_time`)

### Supabase — edge function
```
POST https://emxbjuahedbnanzkgpyc.supabase.co/functions/v1/update-appointment-status
Headers:
  Content-Type: application/json
  x-webhook-secret: bellas_n8n_secret_2026
Body:
  { "appointment_id": "<uuid>", "status": "confirmed" | "cancelled" | "pending_confirmation" }
```

---

## 🗺️ Caminho a percorrer

### Imediato (próximas sessões)
- [ ] Publicar landing page no Netlify via GitHub
- [ ] Redesenhar visual da plataforma Lovable com a paleta rosa/futurista
- [ ] Adicionar modo escuro/claro na plataforma
- [ ] Implementar fluxo de salão na plataforma:
  - Admin cadastra profissionais
  - Cada profissional gerencia seus serviços e horários
  - Página pública do salão lista profissionais
  - Cliente escolhe profissional → vê agenda dela → agenda

### Médio prazo
- [ ] Definir nome final (candidatos: Glowly, Beauly, Agendê, Lumis, Slotly)
- [ ] Registrar domínio (ex: glowly.com.br)
- [ ] Primeiros 3 clientes reais (gratuito para ter prova social)
- [ ] Coletar depoimentos e prints de resultado
- [ ] Criar perfil no Instagram da marca (conteúdo sem aparecer)
- [ ] Estratégia de grupos de WhatsApp/Facebook de profissionais de beleza

### Longo prazo
- [ ] Tráfego pago (apenas após prova social orgânica)
- [ ] Plano para salões com múltiplas profissionais
- [ ] Relatórios e métricas para o admin
- [ ] Avisos em massa para base de clientes

---

## 💡 Decisões tomadas

| Decisão | Motivo |
|---|---|
| Usar edge function em vez de PATCH direto | RLS do Supabase bloqueia UPDATE com anon key |
| Não migrar do Lovable agora | Supabase já é externo, risco alto, sem ganho real agora |
| Netlify em vez de Vercel | Mais simples para HTML estático |
| Hospedagem via GitHub no Netlify | Felipe mexe com frequência na landing |
| IA descrita como "assistente humanizada" na landing | Palavra "IA" remete a robô engessado |
| Salão tem 1 link único, cliente escolhe profissional | Mais simples para o cliente, sem URL por funcionária |

---

## 📁 Arquivos do projeto

| Arquivo | Descrição |
|---|---|
| `landing-futurista.html` | Landing page atual — subir no Netlify via GitHub |
| `GLOWLY_PROJETO.md` | Este arquivo — documentação do projeto |

---

*Última atualização: 29/04/2026*
