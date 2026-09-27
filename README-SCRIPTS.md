# Cloud Guardian AI — Guia para criar o projeto

Este guia foi feito para que qualquer pessoa da equipe consiga criar o projeto **Cloud Guardian AI** sem precisar montar a estrutura manualmente.

Existem dois scripts:

- `create-cloud-guardian-ai.ps1` — Windows / PowerShell
- `create-cloud-guardian-ai.sh` — Linux, macOS e Git Bash

Os dois scripts criam automaticamente:

- estrutura de diretórios;
- código inicial em Go;
- agentes `Incident`, `FinOps` e `IaC`;
- `Governance Guard`;
- testes;
- prompts;
- documentação;
- exemplos;
- pipeline GitHub Actions;
- Makefile;
- arquivos de configuração;
- README do projeto;
- ZIP do projeto, quando suportado.

---

# 1. Escolha seu sistema operacional

## Windows

Use:

```text
create-cloud-guardian-ai.ps1
```

## Linux / macOS / Git Bash

Use:

```text
create-cloud-guardian-ai.sh
```

---

# 2. Pré-requisitos

Antes de executar o script, instale:

- Git
- Go 1.23 ou superior

Verifique:

```bash
git --version
go version
```

Exemplo:

```text
git version 2.x
go version go1.23.x
```

O script funciona mesmo sem Go instalado, mas nesse caso ele não consegue validar automaticamente o projeto criado.

---

# 3. Windows — PowerShell

Abra o PowerShell na pasta onde estão os arquivos.

Exemplo:

```powershell
cd C:\projetos\cloud-guardian
```

Confirme que o script existe:

```powershell
Get-ChildItem
```

Você deverá ver:

```text
create-cloud-guardian-ai.ps1
```

---

# 4. Liberar execução do script no Windows

Em algumas máquinas o PowerShell bloqueia scripts baixados da internet.

Você pode liberar apenas para a sessão atual:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Isso vale somente para aquela janela do PowerShell.

Depois execute:

```powershell
.\create-cloud-guardian-ai.ps1
```

---

# 5. Execução padrão no Windows

```powershell
.\create-cloud-guardian-ai.ps1
```

O script criará:

```text
cloud-guardian-ai\
```

e também:

```text
cloud-guardian-ai.zip
```

na pasta atual.

---

# 6. Escolher outro nome no Windows

```powershell
.\create-cloud-guardian-ai.ps1 -ProjectName "meu-cloud-guardian"
```

Resultado:

```text
meu-cloud-guardian\
meu-cloud-guardian.zip
```

---

# 7. Escolher outra pasta no Windows

Exemplo:

```powershell
.\create-cloud-guardian-ai.ps1 `
    -ProjectName "cloud-guardian-ai" `
    -OutputDir "C:\workspace"
```

O projeto será criado em:

```text
C:\workspace\cloud-guardian-ai
```

---

# 8. Recriar um projeto existente no Windows

Por segurança, o script não apaga uma pasta existente automaticamente.

Se aparecer:

```text
A pasta já existe
```

e você realmente quiser recriá-la:

```powershell
.\create-cloud-guardian-ai.ps1 -Force
```

Também pode combinar:

```powershell
.\create-cloud-guardian-ai.ps1 `
    -ProjectName "cloud-guardian-ai" `
    -OutputDir "C:\workspace" `
    -Force
```

> Atenção: `-Force` remove a pasta do projeto antes de recriá-la.

---

# 9. Criar sem ZIP no Windows

Se não quiser gerar o ZIP:

```powershell
.\create-cloud-guardian-ai.ps1 -NoZip
```

---

# 10. Linux / macOS / Git Bash

Abra um terminal na pasta onde está:

```text
create-cloud-guardian-ai.sh
```

Dê permissão de execução:

```bash
chmod +x create-cloud-guardian-ai.sh
```

Depois:

```bash
./create-cloud-guardian-ai.sh
```

---

# 11. Execução padrão no Shell

```bash
./create-cloud-guardian-ai.sh
```

O projeto será criado em:

```text
./cloud-guardian-ai
```

---

# 12. Escolher outro nome no Shell

O primeiro argumento é o nome do projeto:

```bash
./create-cloud-guardian-ai.sh meu-cloud-guardian
```

Resultado:

```text
./meu-cloud-guardian
```

---

# 13. Escolher outra pasta no Shell

O segundo argumento é a pasta onde o projeto será criado:

```bash
./create-cloud-guardian-ai.sh cloud-guardian-ai ~/workspace
```

Resultado:

```text
~/workspace/cloud-guardian-ai
```

---

# 14. Recriar um projeto existente no Shell

Por padrão o script não apaga uma pasta existente.

Para forçar:

```bash
FORCE=true ./create-cloud-guardian-ai.sh
```

Com nome e pasta:

```bash
FORCE=true ./create-cloud-guardian-ai.sh cloud-guardian-ai ~/workspace
```

> Atenção: `FORCE=true` remove a pasta existente antes de recriar.

---

# 15. Criar sem ZIP no Shell

```bash
NO_ZIP=true ./create-cloud-guardian-ai.sh
```

---

# 16. O que acontece durante a execução

O script cria a estrutura aproximadamente assim:

```text
cloud-guardian-ai/
│
├── cmd/
│   └── api/
│
├── internal/
│   ├── agents/
│   │   ├── incident/
│   │   ├── finops/
│   │   └── iac/
│   │
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
│   └── adr/
│
├── examples/
├── configs/
├── infra/
├── .github/
│   └── workflows/
│
├── README.md
├── SECURITY.md
├── Makefile
└── go.mod
```

---

# 17. Validação automática

Se o comando `go` estiver disponível, o script executa automaticamente:

```bash
go fmt ./...
go test ./...
```

Se os testes passarem, o projeto inicial está consistente.

---

# 18. Depois que o script terminar

Entre na pasta criada.

Windows:

```powershell
cd cloud-guardian-ai
```

Linux/macOS:

```bash
cd cloud-guardian-ai
```

Execute:

```bash
go test ./...
```

Depois:

```bash
go run ./cmd/api
```

Você deverá ver algo parecido com:

```text
Cloud Guardian AI listening on :8080
```

---

# 19. Testar a aplicação

Abra outro terminal.

Health check:

```bash
curl http://localhost:8080/health
```

Resposta esperada:

```json
{
  "service": "cloud-guardian-ai",
  "status": "ok"
}
```

---

# 20. Teste rápido do Incident Agent

Linux / macOS / Git Bash:

```bash
curl -X POST http://localhost:8080/v1/analyze \
  -H "Content-Type: application/json" \
  -d '{
    "kind": "incident",
    "content": "checkout-api retornando 504; queue_depth=1200"
  }'
```

No PowerShell:

```powershell
$body = @{
    kind = "incident"
    content = "checkout-api retornando 504; queue_depth=1200"
} | ConvertTo-Json

Invoke-RestMethod `
    -Method Post `
    -Uri "http://localhost:8080/v1/analyze" `
    -ContentType "application/json" `
    -Body $body
```

---

# 21. Testar FinOps

```json
{
  "kind": "finops",
  "content": "EC2 de sandbox executando 24x7 com baixa utilização"
}
```

---

# 22. Testar IaC

```json
{
  "kind": "iac",
  "content": "security group permitindo 0.0.0.0/0 na porta 22"
}
```

---

# 23. Fluxo completo para um novo desenvolvedor

## Windows

```powershell
git clone <URL_DO_REPOSITORIO_DOS_SCRIPTS>

cd <PASTA>

Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

.\create-cloud-guardian-ai.ps1

cd cloud-guardian-ai

go test ./...

go run ./cmd/api
```

---

## Linux / macOS

```bash
git clone <URL_DO_REPOSITORIO_DOS_SCRIPTS>

cd <PASTA>

chmod +x create-cloud-guardian-ai.sh

./create-cloud-guardian-ai.sh

cd cloud-guardian-ai

go test ./...

go run ./cmd/api
```

---

# 24. Fluxo recomendado para a equipe

A ideia é que o repositório inicial contenha apenas:

```text
bootstrap/
│
├── README-SCRIPTS.md
├── create-cloud-guardian-ai.ps1
└── create-cloud-guardian-ai.sh
```

O desenvolvedor:

```text
1. clona o repositório
       ↓
2. lê README-SCRIPTS.md
       ↓
3. escolhe PowerShell ou Shell
       ↓
4. executa o script
       ↓
5. projeto Go é criado
       ↓
6. testes são executados
       ↓
7. desenvolvedor começa a trabalhar
```

---

# 25. Erros comuns

## PowerShell: execução de scripts desabilitada

Erro parecido com:

```text
running scripts is disabled on this system
```

Execute:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Depois tente novamente.

---

## Shell: Permission denied

Execute:

```bash
chmod +x create-cloud-guardian-ai.sh
```

Depois:

```bash
./create-cloud-guardian-ai.sh
```

---

## `go: command not found`

Instale o Go e confirme:

```bash
go version
```

Depois execute novamente o script.

---

## Pasta já existe

Windows:

```powershell
.\create-cloud-guardian-ai.ps1 -Force
```

Shell:

```bash
FORCE=true ./create-cloud-guardian-ai.sh
```

---

## Porta 8080 ocupada

Linux/macOS:

```bash
HTTP_ADDR=:9090 go run ./cmd/api
```

PowerShell:

```powershell
$env:HTTP_ADDR=":9090"
go run ./cmd/api
```

---

# 26. O que o script NÃO faz

Nesta primeira versão ele não:

- cria recursos reais na AWS;
- executa `terraform apply`;
- cria access keys;
- configura credenciais automaticamente;
- habilita modelos no Bedrock;
- faz deploy em produção.

Isso é proposital.

O objetivo inicial é criar um ambiente de desenvolvimento seguro e reproduzível.

---

# 27. Segurança

Nunca coloque nos scripts ou no repositório:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
tokens
passwords
private keys
```

Quando AWS for integrada, use preferencialmente:

- AWS SSO para desenvolvimento;
- IAM Role para workloads;
- least privilege;
- read-only inicialmente.

---

# 28. Regra importante

Se o script terminou e:

```bash
go test ./...
```

passou, o ambiente básico está pronto.

O próximo passo é ler:

```text
cloud-guardian-ai/README.md
```

Esse segundo README explica como trabalhar dentro do projeto Cloud Guardian AI.

---

# 29. Resumo rápido

## Windows

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\create-cloud-guardian-ai.ps1
cd cloud-guardian-ai
go run ./cmd/api
```

## Linux / macOS

```bash
chmod +x create-cloud-guardian-ai.sh
./create-cloud-guardian-ai.sh
cd cloud-guardian-ai
go run ./cmd/api
```

Depois:

```text
http://localhost:8080/health
```

Pronto. O ambiente de desenvolvimento inicial do **Cloud Guardian AI** estará funcionando.
