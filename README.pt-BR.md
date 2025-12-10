# Claude Secure Plugins

[![Security Audited](https://img.shields.io/badge/security-audited-green.svg)](docs/SECURITY.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

**Marketplace de plugins Claude Code com segurança por padrão.**

[🇺🇸 Read in English](README.md)

## Por que este marketplace?

Após um incidente de segurança onde configurações Docker inseguras geradas por IA resultaram em comprometimento de servidor, criamos este marketplace com foco em:

1. **Segurança por Padrão** - Todas as configurações seguem princípios de mínimo privilégio
2. **Código Auditado** - Cada plugin é revisado manualmente antes da publicação
3. **Práticas Documentadas** - Explicamos o "porquê" de cada decisão de segurança

## Princípios de Segurança

### Docker & Containers
- ✅ Portas SEMPRE vinculadas a `127.0.0.1` por padrão
- ✅ `security_opt: [no-new-privileges:true]` obrigatório
- ✅ Usuário não-root quando possível
- ✅ Read-only filesystem quando aplicável
- ✅ Sem `privileged: true` a menos que explicitamente necessário

### Infraestrutura
- ✅ Firewall restritivo por padrão (deny all, allow specific)
- ✅ Secrets nunca em arquivos de configuração
- ✅ TLS/HTTPS obrigatório para serviços expostos
- ✅ Rate limiting em APIs

### Código
- ✅ Input validation em todos os pontos de entrada
- ✅ Sem execução de comandos shell com input do usuário
- ✅ Dependências verificadas e atualizadas

## Instalação

```bash
# Adicionar o marketplace
claude plugin marketplace add cassao29/claude-secure-plugins

# Instalar um plugin específico
claude plugin install docker-compose-secure@claude-secure-plugins
```

## Plugins Disponíveis

### DevOps
| Plugin | Descrição | Status |
|--------|-----------|--------|
| `docker-compose-secure` | Gerador de Docker Compose com segurança por padrão | ✅ Auditado |
| `kubernetes-secure` | Manifests K8s com PodSecurityContext | ✅ Auditado |

### Segurança
| Plugin | Descrição | Status |
|--------|-----------|--------|
| `security-scanner` | Scanner de vulnerabilidades em código e configs | ✅ Auditado |
| `secrets-validator` | Validador de secrets e credenciais expostas | ✅ Auditado |

## Contribuindo

Leia [CONTRIBUTING.md](docs/CONTRIBUTING.md) antes de submeter plugins.

**Requisitos para contribuição:**
1. Código fonte completo (sem ofuscação)
2. Documentação de segurança
3. Testes automatizados
4. Passar na auditoria de segurança

## Auditoria de Segurança

Cada plugin passa por:
1. Revisão manual de código
2. Verificação de dependências (npm audit, pip audit)
3. Scan com ferramentas SAST
4. Teste de comportamento em ambiente isolado

Veja [docs/SECURITY.md](docs/SECURITY.md) para detalhes.

## Licença

MIT - Veja [LICENSE](LICENSE)

## Autores

- **Cássio Santos** - Criador e mantenedor

---

> ⚠️ **Aviso**: Este marketplace foi criado após um incidente real de segurança.
> Plugins de terceiros sem auditoria podem expor seu servidor a ataques.
> [Leia o relatório completo](docs/INCIDENT_REPORT.md)
