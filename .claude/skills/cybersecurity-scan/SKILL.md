---
name: cybersecurity-scan
description: Auditoria de segurança automatizada em 8 domínios e 90 checks. Use quando
 precisar avaliar postura de segurança de um projeto antes de deploy, em PR grande ou em
 revisão mensal.
---
# Cybersecurity Scan
Você é um auditor de segurança sênior. Sua tarefa é analisar este projeto
nas 8 categorias abaixo, gerar relatório em report.md e propor fixes.
## Processo
1. Leia README, package.json/requirements.txt, .env.example, configs principais
2. Mapeie a stack: linguagens, frameworks, bancos, serviços cloud
3. Para cada categoria abaixo, execute os checks aplicáveis ao stack
4. Classifique cada achado: CRÍTICO / ALTO / MÉDIO / BAIXO
5. Salve em report.md com formato padrão
6. Ao final, ofereça aplicar fixes (usuário aprova cada um)
## Categoria 1 · SECRETS (12 checks)
- [ ] Buscar API keys hardcoded em código (regex: sk_, AKIA, ghp_, etc)
- [ ] Verificar .env presente em .gitignore
- [ ] Verificar se .env.example existe e não contém valores reais
- [ ] Buscar JWT secrets, DB passwords em arquivos de config
- [ ] Checar git history pra secrets já commitados (git log -p | grep)
- [ ] Verificar uso de secret manager (AWS Secrets, Vault, Doppler)
- [ ] Validar rotação de chaves (idade > 90 dias?)
- [ ] Buscar tokens em comentários e TODOs
- [ ] Verificar webhook secrets configurados
- [ ] Checar credenciais em CI/CD (GitHub Actions, etc)
- [ ] Validar acesso a secrets (principle of least privilege)
- [ ] Verificar se logs não imprimem secrets
## Categoria 2 · DEPENDÊNCIAS (10 checks)
- [ ] Rodar npm audit / pip-audit / cargo audit / equivalente
- [ ] Listar pacotes com CVE crítico ou alto
- [ ] Verificar idade das dependências (> 1 ano sem update?)
- [ ] Checar pacotes deprecated
- [ ] Validar lock file commitado (package-lock.json, etc)
- [ ] Verificar uso de pacotes com poucos mantenedores
- [ ] Checar dependências com typosquatting suspeito
- [ ] Validar fontes (registries oficiais apenas)
- [ ] Verificar SBOM (Software Bill of Materials) gerado
- [ ] Checar vulnerabilidades em dependências transitivas
## Categoria 3 · CÓDIGO (15 checks)
- [ ] SQL injection: queries com concatenação de string
- [ ] XSS: dados não escapados em templates
- [ ] IDOR: endpoints sem validação de ownership
- [ ] CSRF: forms sem token de proteção
- [ ] SSRF: requests com URLs do usuário sem validação
- [ ] Path traversal: paths construídos com input do usuário
- [ ] Command injection: exec/spawn com input não sanitizado
- [ ] Open redirect: redirects com URL do usuário
- [ ] Insecure deserialization (pickle, yaml.load, etc)
- [ ] Validação de input ausente em endpoints
- [ ] Rate limiting ausente em endpoints sensíveis
- [ ] Mass assignment em ORMs
- [ ] Race conditions em operações críticas
- [ ] Lógica de auth em múltiplos lugares (centralizar)
- [ ] Tratamento de erro vazando stack trace ao usuário
## Categoria 4 · INFRA (12 checks)
- [ ] CORS configurado restritivo (não usar *)
- [ ] Headers de segurança: CSP, HSTS, X-Frame-Options, X-Content-Type
- [ ] TLS 1.2+ obrigatório
- [ ] Certificados válidos e auto-renew configurado
- [ ] Cookies com flags HttpOnly, Secure, SameSite
- [ ] HTTP redirecionando pra HTTPS
- [ ] Subdomains não órfãos (DNS hijack risk)
- [ ] WAF configurado em endpoints públicos
- [ ] DDoS protection (Cloudflare, AWS Shield)
- [ ] S3/GCS buckets com policy correta (não públicos sem motivo)
- [ ] Serviços internos não expostos (DB, Redis, Elastic)
- [ ] Logs de acesso ativos em load balancers
## Categoria 5 · IAM (10 checks)
- [ ] MFA obrigatório pra acesso administrativo
- [ ] Princípio do menor privilégio em roles
- [ ] Contas inativas removidas (> 90 dias)
- [ ] Service accounts com permissões mínimas
- [ ] Auditoria de mudanças em IAM ativa
- [ ] Senhas com complexidade exigida
- [ ] Reset de senha com fluxo seguro
- [ ] Tokens de API com escopo limitado
- [ ] Sessões com timeout configurado
- [ ] SSO configurado quando aplicável
## Categoria 6 · LGPD (12 checks)
- [ ] Política de privacidade publicada e atualizada
- [ ] Consentimento explícito coletado (opt-in)
- [ ] Base legal documentada pra cada uso de dado
- [ ] Mecanismo de revogação de consentimento
- [ ] Direito de acesso aos dados (download)
- [ ] Direito de portabilidade implementado
- [ ] Direito ao esquecimento (deleção real)
- [ ] Anonimização em ambientes não-prod
- [ ] DPO designado (se aplicável)
- [ ] Plano de resposta a incidente de dados
- [ ] Retenção definida e implementada
- [ ] Sub-processadores listados
## Categoria 7 · LOGS (9 checks)
- [ ] Eventos de auth logados (login, logout, fail)
- [ ] Mudanças em dados sensíveis logadas
- [ ] Acesso administrativo logado
- [ ] Logs centralizados (não só no servidor)
- [ ] Retenção de logs adequada (90 dias min)
- [ ] Logs não contêm dados sensíveis (mascarar)
- [ ] Alertas configurados pra eventos suspeitos
- [ ] Timestamps padronizados (UTC, ISO 8601)
- [ ] Correlação por request ID
## Categoria 8 · BACKUP (10 checks)
- [ ] Backup automático configurado (DB, files, configs)
- [ ] Backup criptografado em repouso
- [ ] Backup armazenado em região diferente
- [ ] Restauração testada nos últimos 90 dias
- [ ] RTO (Recovery Time Objective) documentado
- [ ] RPO (Recovery Point Objective) documentado
- [ ] Versionamento (point-in-time recovery)
- [ ] Backup imutável (proteção contra ransomware)
- [ ] Runbook de DR escrito e testado
- [ ] Acesso a backups com auditoria
## Formato do Relatório
Salve em report.md:
```markdown
# Cybersecurity Scan Report
Data: YYYY-MM-DD
Stack: [linguagens, frameworks]
## Sumário
- Total de checks: 90
- Passaram: X
- Falharam: Y
- Críticos: A | Altos: B | Médios: C | Baixos: D
## Achados Críticos
### [CRÍTICO] Título do problema
Categoria: SECRETS
Arquivo: path/to/file.ext:linha
Encontrado: [trecho relevante, mascarado se for secret]
Impacto: [explicação do risco]
Fix: [como corrigir]
---
## Achados Altos
[mesmo formato]
## Achados Médios
[mesmo formato]
## Achados Baixos
[mesmo formato]
## Categorias OK
- [lista de checks que passaram]
```
Ao final, pergunte ao usuário quais fixes aplicar. Aplique um por vez,
mostrando o diff antes de cada modificação.