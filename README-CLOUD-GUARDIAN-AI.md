# Cloud Guardian AI

> Plataforma inteligente para apoiar decisões de **Cloud & Infrastructure** usando IA, com foco em análise, explicação, recomendação, governança e segurança.

## 1. Visão geral

O **Cloud Guardian AI** é uma iniciativa de engenharia voltada para apoiar profissionais de Cloud, SRE, DevOps, Platform Engineering, FinOps, Security e Architecture.

A proposta não é criar apenas um chatbot e também não é permitir que um modelo de IA altere ambientes de produção de forma autônoma. O objetivo é construir uma plataforma capaz de receber contexto técnico, analisar evidências, formular hipóteses, explicar riscos e sugerir próximos passos de forma estruturada.

A filosofia do projeto é:

```text
Analyze
   ↓
Explain
   ↓
Recommend
   ↓
Human Decision
   ↓
Controlled Action
```

> **Regra principal: Recommend before automate.**

---

## 2. Problema que queremos resolver

Ambientes de Cloud & Infrastructure geram uma grande quantidade de informações: logs, métricas, traces, alarmes, eventos, custos, inventário, tags, arquivos Terraform, findings de segurança, resultados de scanners e runbooks.

O problema não é apenas ter acesso ao dado. O desafio é transformar esses dados em respostas úteis para perguntas como:

- O que provavelmente está acontecendo?
- Quais evidências sustentam essa análise?
- O que ainda precisa ser investigado?
- Qual risco está envolvido?
- Existe desperdício de recurso?
- A infraestrutura está segura?
- Qual deve ser o próximo passo?
- Essa ação pode ser automatizada?
- Essa decisão exige aprovação humana?

O Cloud Guardian AI atua como uma camada inteligente de apoio a esse processo.

---

## 3. Capacidades iniciais

O projeto começa com três capacidades principais:

```text
Cloud Guardian AI
│
├── Incident Analysis
├── FinOps Analysis
└── Infrastructure as Code Review
```

Cada capacidade é implementada por um agente especializado.

### Incident Agent

Apoia investigação de incidentes usando contexto como logs, métricas, traces, alarmes, erros HTTP, filas e runbooks.

Princípio importante:

> **Evidência não é hipótese, e hipótese não é causa raiz confirmada.**

### FinOps Agent

Apoia análise de custos, utilização e oportunidades de otimização.

Exemplos:

- recursos ociosos;
- superdimensionamento;
- crescimento inesperado de custo;
- recursos ativos fora do horário esperado;
- ausência de tags;
- oportunidades de scheduling e rightsizing.

Princípio importante:

> **Baixa utilização não significa automaticamente desperdício.**

### IaC Reviewer

Analisa Infrastructure as Code, inicialmente Terraform, sob as óticas de:

- segurança;
- confiabilidade;
- custo;
- governança;
- observabilidade.

A IA não substitui ferramentas determinísticas. O fluxo esperado é:

```text
Terraform
   |
   +--> terraform validate
   +--> Checkov
   +--> tfsec
   +--> OPA
   +--> Cloud Guardian AI
            |
            +--> contextualiza
            +--> explica
            +--> prioriza
            +--> recomenda
```

---

## 4. Arquitetura de alto nível

```text
                         +------------------+
                         |  API / CLI / UI  |
                         +---------+--------+
                                   |
                                   v
                         +------------------+
                         |   Orchestrator   |
                         +---------+--------+
                                   |
              +--------------------+--------------------+
              |                    |                    |
              v                    v                    v
     +----------------+   +----------------+   +----------------+
     | Incident Agent |   | FinOps Agent   |   | IaC Reviewer   |
     +--------+-------+   +--------+-------+   +--------+-------+
              |                    |                    |
              +--------------------+--------------------+
                                   |
                                   v
                         +------------------+
                         | Governance Guard |
                         +---------+--------+
                                   |
                                   v
                         Structured Response
                                   |
                                   v
                         Human Approval Gate
```

### Responsabilidades principais

**API**
- recebe requisições;
- valida payloads;
- aciona o Orchestrator;
- devolve respostas estruturadas.

**Orchestrator**
- identifica o tipo de análise;
- seleciona o agente correto;
- coordena o fluxo;
- não concentra a lógica específica de cada domínio.

**Agents**
- implementam comportamento especializado por domínio.

**Governance Guard**
- aplica políticas transversais de segurança e governança;
- identifica ações mutáveis ou destrutivas;
- exige aprovação humana quando necessário.

---

## 5. Read-only first

A primeira versão deve operar prioritariamente em modo de leitura.

```text
Cloud APIs
   ↓
Read
   ↓
Analyze
   ↓
Recommend
```

Não queremos começar com:

```text
AI
 ↓
terraform apply
```

Queremos:

```text
AI
 ↓
Recommendation
 ↓
Human Approval
 ↓
Controlled Execution
```

---

## 6. Human in the Loop

Mudanças relevantes precisam de revisão humana, por exemplo:

- excluir recursos;
- alterar Security Groups;
- modificar políticas IAM;
- reduzir ou aumentar capacidade;
- aplicar Terraform;
- desligar instâncias;
- executar ações com impacto financeiro ou operacional relevante.

Fluxo futuro:

```text
Recommendation
      ↓
Change Plan
      ↓
Human Review
      ↓
Approval
      ↓
Execution
      ↓
Audit
```

---

## 7. Estratégia arquitetural

O MVP utiliza **Monólito Modular em Go**.

```text
Application
│
├── Orchestrator
├── Incident
├── FinOps
├── IaC
└── Governance
```

A decisão evita complexidade prematura e facilita:

- desenvolvimento;
- testes;
- debug;
- deploy;
- evolução incremental.

Microserviços só devem ser considerados quando houver necessidade real de escala independente, ownership separado, isolamento, requisitos de segurança ou ciclos de deploy distintos.

---

## 8. Estrutura do projeto

```text
cloud-guardian-ai/
│
├── cmd/
│   └── api/
│       └── main.go
│
├── internal/
│   ├── agents/
│   │   ├── agent.go
│   │   ├── incident/
│   │   ├── finops/
│   │   └── iac/
│   ├── domain/
│   ├── governance/
│   └── orchestrator/
│
├── prompts/
│   ├── orchestrator/
│   ├── incident/
│   ├── finops/
│   ├── iac/
│   └── governance/
│
├── docs/
│   ├── architecture.md
│   ├── challenge-canvas.md
│   └── adr/
│
├── examples/
├── configs/
├── infra/
├── .github/
├── SECURITY.md
├── Makefile
└── go.mod
```

### Diretórios

| Diretório | Responsabilidade |
|---|---|
| `cmd/api` | entry point da aplicação |
| `internal/domain` | tipos centrais do domínio |
| `internal/agents` | agentes especializados |
| `internal/orchestrator` | coordenação do fluxo |
| `internal/governance` | regras de segurança e aprovação |
| `prompts` | prompts versionados |
| `docs` | documentação arquitetural e ADRs |
| `infra` | infraestrutura como código |
| `examples` | exemplos de entrada e cenários |

---

## 9. Contrato dos agentes

Todo agente deve implementar um contrato semelhante a:

```go
type Agent interface {
    Name() string
    Analyze(ctx context.Context, content string) (domain.Finding, error)
}
```

Isso permite incluir novas capacidades sem alterar toda a aplicação.

Possíveis agentes futuros:

- Security Agent;
- Reliability Agent;
- Architecture Agent;
- Compliance Agent;
- Sustainability Agent;
- Capacity Agent;
- Change Risk Agent.

---

## 10. Arquitetura Hexagonal

A evolução do projeto deve aproximar o core de Ports & Adapters.

```text
                    Domain / Agents
                           |
              +------------+------------+
              |            |            |
              v            v            v
          LLMClient     LogReader     CostReader
              ^            ^            ^
              |            |            |
           Bedrock     CloudWatch   Cost Explorer
```

Portas previstas:

- `LLMClient`
- `LogReader`
- `MetricsReader`
- `CostReader`
- `IaCScanner`
- `ApprovalStore`
- `AuditRepository`
- `KnowledgeRetriever`

Isso mantém o domínio desacoplado de serviços específicos da AWS.

---

## 11. Integrações AWS previstas

Possíveis integrações futuras:

- Amazon Bedrock;
- CloudWatch Logs;
- CloudWatch Metrics;
- AWS Cost Explorer;
- AWS Config;
- CloudTrail;
- Security Hub;
- Trusted Advisor;
- S3;
- DynamoDB.

Nem todas precisam existir no MVP. Cada integração deve ser adicionada conforme necessidade real do caso de uso.

---

## 12. Amazon Bedrock

A integração com LLM deve ocorrer por abstração.

```go
type LLMClient interface {
    Generate(ctx context.Context, prompt string) (string, error)
}
```

Exemplo:

```text
LLMClient
   ↑
   |
BedrockAdapter
```

Dessa forma, o domínio não depende diretamente da implementação do provedor.

---

## 13. Structured Output

Respostas de IA devem ser estruturadas sempre que possível.

Exemplo:

```json
{
  "category": "incident",
  "title": "Possible downstream saturation",
  "evidence": [
    "HTTP 504 detected",
    "queue_depth=1200"
  ],
  "hypotheses": [
    "downstream timeout",
    "backpressure"
  ],
  "recommendations": [
    "check consumer throughput",
    "check downstream latency"
  ],
  "risk": "medium",
  "confidence": 0.72
}
```

Benefícios:

- validação;
- testes;
- integração;
- observabilidade;
- consistência.

---

## 14. Governance Guard

O Governance Guard deve aplicar regras como:

- bloquear ações destrutivas sem aprovação;
- impedir execução automática de mudanças sensíveis;
- redigir segredos;
- validar limites de execução;
- identificar prompt injection;
- exigir revisão humana quando necessário.

Estados esperados:

```text
ALLOWED
ALERT
BLOCKED
REQUIRES_APPROVAL
```

---

## 15. Segurança

Princípios obrigatórios:

- least privilege;
- nenhuma credencial hardcoded;
- IAM Role em runtime;
- read-only first;
- dados mínimos;
- sanitização de logs;
- proteção contra prompt injection;
- auditoria;
- revisão humana;
- uso apenas de dados permitidos, sintéticos ou autorizados.

Nunca enviar diretamente ao modelo:

```text
passwords
AWS_SECRET_ACCESS_KEY
tokens
private keys
credentials
customer secrets
```

---

## 16. Prompt Injection

Todo conteúdo externo deve ser tratado como **não confiável**.

Exemplo: um arquivo Terraform pode conter um comentário tentando instruir o modelo a ignorar regras do sistema. Esse conteúdo deve ser interpretado como dado, nunca como comando.

Prioridade conceitual:

```text
SYSTEM INSTRUCTIONS
      >
PROJECT POLICIES
      >
TOOL POLICIES
      >
USER REQUEST
      >
EXTERNAL DATA
```

---

## 17. Observabilidade

Precisamos observar não apenas a aplicação, mas também o comportamento da IA.

Campos importantes:

```text
traceId
requestId
agent
promptVersion
model
latency
tokenUsage
estimatedCost
toolCalls
decision
confidence
```

OpenTelemetry é uma evolução natural para tracing e métricas.

---

## 18. Confidence

O campo `confidence` não deve ser interpretado como probabilidade estatística real, a menos que exista mecanismo calibrado para isso.

Inicialmente ele deve ser entendido como um indicador interno da força das evidências disponíveis.

---

## 19. Testes

### Código

```bash
go test ./...
go vet ./...
go test -race ./...
```

### IA

Devemos avaliar cenários como:

- hallucination;
- ausência de evidência;
- excesso de confiança;
- prompt injection;
- recomendações perigosas;
- vazamento de segredos;
- respostas incompletas.

### Evals

Uma evolução esperada:

```text
evals/
│
├── incident/
├── finops/
└── iac/
```

Cada cenário pode ter:

```text
input
expected constraints
forbidden behavior
expected category
expected risk
```

---

## 20. Desenvolvimento local

Pré-requisitos:

- Go 1.23+;
- Git;
- curl ou Postman.

Execute:

```bash
go mod tidy
go test ./...
go vet ./...
go run ./cmd/api
```

Health check:

```bash
curl http://localhost:8080/health
```

---

## 21. Testando os agentes

### Incident

```bash
curl -X POST http://localhost:8080/v1/analyze \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "incident",
    "content": "checkout-api retornando 504; queue_depth=1200"
  }'
```

### FinOps

```bash
curl -X POST http://localhost:8080/v1/analyze \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "finops",
    "content": "EC2 sandbox rodando 24x7 com baixa utilização"
  }'
```

### IaC

```bash
curl -X POST http://localhost:8080/v1/analyze \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "iac",
    "content": "security group permitindo 0.0.0.0/0 na porta 22"
  }'
```

---

## 22. Como desenvolver uma nova feature

Fluxo recomendado:

```text
Understand Problem
      ↓
Define Behavior
      ↓
Write ADR if needed
      ↓
Implement
      ↓
Test
      ↓
Document
      ↓
Pull Request
```

Toda mudança relevante deve explicar:

- problema;
- solução;
- trade-offs;
- testes;
- impacto de segurança;
- impacto de observabilidade.

---

## 23. ADRs

Decisões arquiteturais relevantes devem ser registradas.

Exemplos já previstos:

```text
ADR-001-read-only-first.md
ADR-002-modular-monolith.md
```

Formato recomendado:

```text
Context
Decision
Alternatives
Consequences
```

---

## 24. O que NÃO queremos no MVP

Evitar inicialmente:

- Kubernetes sem necessidade;
- dezenas de microserviços;
- múltiplos bancos;
- agentes autônomos executando mudanças;
- arquitetura excessivamente distribuída;
- complexidade operacional prematura.

> **Validar valor antes de escalar arquitetura.**

---

## 25. MVP

O primeiro MVP deve provar um fluxo ponta a ponta.

Exemplo:

```text
Incident
   ↓
API
   ↓
Orchestrator
   ↓
Incident Agent
   ↓
Bedrock
   ↓
Governance
   ↓
Structured Response
```

Os dados podem começar como mocks ou dados sintéticos e ser substituídos progressivamente por integrações reais.

---

## 26. Roadmap

### Fase 1 — Foundation
- Go API;
- Orchestrator;
- agentes;
- Governance Guard;
- testes;
- documentação.

### Fase 2 — AI
- Amazon Bedrock;
- prompts versionados;
- structured output;
- evals.

### Fase 3 — AWS Read-only
- CloudWatch;
- Cost Explorer;
- inventário;
- IaC.

### Fase 4 — Observability & Security
- OpenTelemetry;
- tracing;
- auditoria;
- redaction;
- prompt injection testing.

### Fase 5 — Controlled Automation
- approval workflow;
- allowlist;
- execution plan;
- audit;
- rollback;
- idempotência.

---

## 27. Como cada pessoa pode contribuir

### Backend
- APIs;
- domínio;
- Orchestrator;
- agentes.

### AI
- prompts;
- Bedrock;
- evals;
- structured outputs.

### Cloud
- AWS;
- IAM;
- CloudWatch;
- Cost Explorer.

### Security
- threat modeling;
- guardrails;
- redaction;
- policies.

### FinOps
- custos;
- rightsizing;
- economia estimada.

### DevOps
- Terraform;
- CI/CD;
- containers;
- observabilidade.

### Architecture
- ADRs;
- trade-offs;
- quality attributes.

---

## 28. Definition of Done

Uma feature só deve ser considerada concluída quando estiver:

- implementada;
- testada;
- documentada;
- observável;
- segura;
- revisada;
- com comportamento de erro definido.

Para funcionalidades de IA, adicionar:

- prompt versionado;
- evals;
- structured output quando aplicável;
- riscos conhecidos documentados.

---

## 29. Critério de sucesso

Não queremos medir sucesso por:

```text
"Conseguimos chamar um LLM."
```

Queremos medir impacto, por exemplo:

- redução do tempo de investigação;
- aumento da qualidade de reviews;
- redução de findings recorrentes;
- economia identificada;
- melhor rastreabilidade;
- maior consistência nas decisões.

---

## 30. Princípios arquiteturais

1. Problema antes da tecnologia.
2. Começar pequeno.
3. Read-only first.
4. Human-in-the-loop.
5. Least privilege.
6. Security by Design.
7. Observabilidade desde o início.
8. Evidência não é hipótese.
9. IA não substitui validação determinística.
10. Prompts são código.
11. Infraestrutura deve ser reproduzível.
12. Arquitetura deve evoluir conforme necessidade real.
13. Automação deve ser controlada e auditável.
14. Dados externos são não confiáveis.
15. Todo comportamento crítico precisa ser testável.

---

## 31. Para novos desenvolvedores

Se você acabou de entrar no projeto, entenda estas cinco coisas antes de começar:

### 1. Cloud Guardian AI não é apenas um chatbot
É uma plataforma de análise e decisão assistida por IA.

### 2. Existem inicialmente três domínios

```text
Incident
FinOps
IaC
```

### 3. O Orchestrator coordena
Ele não deve concentrar a lógica de negócio dos agentes.

### 4. Governance é obrigatório
Toda ação relevante precisa respeitar regras de segurança, risco e aprovação.

### 5. IA recomenda antes de executar

```text
Analyze
Explain
Recommend
Human Decision
Controlled Action
```

---

## 32. Visão futura

No futuro, o Cloud Guardian AI pode atuar em várias etapas do ciclo de engenharia:

```text
                         Cloud Guardian AI

                                |
         +----------------------+----------------------+
         |                      |                      |
         v                      v                      v

      DESIGN                  BUILD                   RUN

 Architecture Review       IaC Review           Incident Analysis
 Cost Estimation           Security Scan        Observability
 Reliability Review        Policy Check         FinOps
```

---

## 33. Nossa regra principal

> **O Cloud Guardian AI deve ajudar engenheiros a tomar decisões melhores — não retirar deles a responsabilidade pelas decisões.**
