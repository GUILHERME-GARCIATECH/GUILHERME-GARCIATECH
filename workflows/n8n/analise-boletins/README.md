# Workflow n8n — Análise diária de boletins de licitação com IA

## Objetivo

Ler e-mails de boletins de licitação, analisar cada oportunidade com IA, aplicar critérios pré-definidos, gerar score e enviar um resumo com as melhores licitações do dia.

Horários:
- 07:30
- 12:30

---

## Estrutura recomendada do workflow

### 1. Schedule Trigger

Node: `Schedule Trigger`

Configuração:
- Executar todo dia às 07:30
- Executar todo dia às 12:30
- Timezone: America/Cuiaba ou America/Sao_Paulo, conforme o n8n estiver configurado

O n8n possui node próprio de agendamento. :contentReference[oaicite:0]{index=0}

---

### 2. Buscar e-mails da caixa

Opções:

Se for Microsoft 365 / Outlook:
- Node: `Microsoft Outlook`
- Operação: buscar mensagens
- Filtro sugerido:
  - e-mails recebidos desde a última execução
  - remetentes dos boletins
  - assunto contendo “licitação”, “pregão”, “edital”, “dispensa”, “concorrência”

O n8n possui node do Microsoft Outlook e trigger do Outlook. :contentReference[oaicite:1]{index=1}

Se for Gmail:
- Node: `Gmail`
- Operação: listar ou buscar mensagens

O Gmail node permite buscar, ler, marcar e responder mensagens. :contentReference[oaicite:2]{index=2}

Se for caixa IMAP genérica:
- Node: `Email Trigger (IMAP)` ou integração IMAP

O n8n também suporta leitura por IMAP. :contentReference[oaicite:3]{index=3}

---

### 3. Filtrar e-mails relevantes

Node: `IF` ou `Code`

Critérios:
- assunto contém termos de licitação
- corpo contém palavras-chave relevantes
- ignorar e-mails duplicados
- ignorar licitações já analisadas
- ignorar boletins sem links/anexos úteis

Palavras-chave úteis:
- pregão eletrônico
- licitação
- edital
- ata de registro de preços
- contratação de serviços
- tecnologia da informação
- suporte técnico
- Microsoft
- servidores
- redes
- infraestrutura
- outsourcing
- help desk
- manutenção
- software
- hardware
- firewall
- backup
- nuvem

---

### 4. Extrair dados do e-mail

Node: `Code` ou `Set`

Extrair:
- assunto
- remetente
- data de recebimento
- corpo do e-mail
- links
- anexos
- órgão público
- modalidade
- cidade/estado
- prazo
- objeto
- valor estimado, se houver

---

### 5. Analisar com IA

Node: `OpenAI`, `AI Agent` ou LangChain Agent

O n8n possui recursos próprios para workflows com IA e agentes. :contentReference[oaicite:4]{index=4}

Prompt sugerido:

Você é um analista especialista em licitações públicas para uma empresa de tecnologia, suporte técnico, infraestrutura, servidores, Microsoft 365, redes, segurança, firewall, backup e serviços gerenciados.

Analise o boletim abaixo e identifique licitações relevantes.

Critérios de avaliação:

1. Aderência ao negócio da empresa
- Suporte técnico
- Infraestrutura de TI
- Microsoft 365
- Servidores
- Redes
- Firewall
- Backup
- Segurança
- Serviços gerenciados
- Hardware e software corporativo

2. Viabilidade operacional
- A empresa conseguiria entregar?
- Exige equipe local?
- Exige certificações específicas?
- Exige marcas ou autorizações difíceis?
- O prazo parece viável?

3. Potencial financeiro
- Valor estimado
- Recorrência
- Tamanho do contrato
- Possibilidade de margem

4. Risco
- Prazo curto
- Edital muito restritivo
- Objeto fora da especialidade
- Exigências jurídicas ou técnicas complexas
- Localidade distante

5. Urgência
- Prazo de abertura
- Prazo para impugnação
- Prazo para proposta

Gere uma resposta em JSON com este formato:

{
  "licitacoes": [
    {
      "titulo": "",
      "orgao": "",
      "cidade_estado": "",
      "modalidade": "",
      "objeto": "",
      "prazo": "",
      "valor_estimado": "",
      "link": "",
      "score": 0,
      "classificacao": "Alta prioridade | Média prioridade | Baixa prioridade | Ignorar",
      "motivos_positivos": [],
      "riscos": [],
      "recomendacao": ""
    }
  ]
}

Regra de score:
- 90 a 100: oportunidade excelente
- 75 a 89: boa oportunidade
- 60 a 74: analisar com cautela
- abaixo de 60: baixa prioridade ou ignorar

Texto do boletim:
{{$json["body"]}}

---

### 6. Ordenar por score

Node: `Code`

Ordenar:
- score maior primeiro
- remover itens classificados como “Ignorar”
- limitar, por exemplo, às 5 ou 10 melhores oportunidades

---

### 7. Gerar e-mail final

Node: `OpenAI` ou `Set`

Formato do e-mail:

Assunto:
Resumo de licitações recomendadas — {{$now}}

Corpo:

Bom dia,

Segue a análise automática das principais licitações identificadas no boletim de hoje.

Resumo:
- Total de oportunidades analisadas: X
- Oportunidades recomendadas: Y
- Alta prioridade: Z

Principais oportunidades:

1. [Título]
Órgão:
Local:
Objeto:
Prazo:
Valor estimado:
Score:
Classificação:
Por que vale olhar:
Riscos:
Recomendação:

2. [Título]
...

Observação:
Esta análise foi gerada automaticamente por IA e deve ser validada antes de qualquer decisão comercial ou jurídica.

---

### 8. Enviar e-mail

Node:
- `Send Email`
- `Gmail`
- ou `Microsoft Outlook`

O n8n possui node próprio para envio de e-mail. :contentReference[oaicite:5]{index=5}

Destinatários:
- comercial
- técnico responsável
- gestor
- você

---

## Melhorias importantes

### Evitar analisar o mesmo e-mail duas vezes

Salvar IDs dos e-mails já processados em:
- Google Sheets
- banco PostgreSQL
- SQLite
- Data Store do n8n
- Airtable

Salvar:
- message_id
- data
- assunto
- score máximo encontrado
- status: analisado

---

### Separar critérios por peso

Modelo de score sugerido:

Aderência técnica: 35 pontos
Viabilidade operacional: 25 pontos
Potencial financeiro: 20 pontos
Prazo e urgência: 10 pontos
Risco baixo: 10 pontos

Total: 100 pontos

---

### Classificação final

90-100:
Alta prioridade — analisar imediatamente

75-89:
Boa oportunidade — encaminhar para avaliação

60-74:
Média prioridade — avaliar se houver tempo

0-59:
Baixa prioridade — normalmente ignorar

---

## Fluxo visual resumido

Schedule Trigger
↓
Buscar e-mails
↓
Filtrar boletins relevantes
↓
Extrair texto, links e anexos
↓
IA analisa cada oportunidade
↓
Gerar score
↓
Ordenar melhores licitações
↓
Montar resumo HTML
↓
Enviar e-mail às 07:30 e 12:30
↓
Salvar histórico dos e-mails analisados
