# Arquitetura — ERP Consert Máquinas

## 1. Visão geral

Sistema web de gestão para a Consert Máquinas, cobrindo:

- Cadastro de clientes
- Controle de estoque (produtos/peças)
- Entradas e saídas de estoque
- Ordens de manutenção (OS)
- Emissão de Notas Fiscais (NF-e / NFS-e)

## 2. Stack tecnológica

| Camada | Tecnologia | Motivo |
|---|---|---|
| Frontend + Backend | **Next.js** (React) | Um único projeto cobre telas e API (rotas serverless), fácil de hospedar na Vercel |
| Hospedagem | **Vercel** | Deploy automático a cada push no GitHub, HTTPS grátis, escalável |
| Versionamento | **GitHub** | Histórico de código, controle de mudanças, integração direta com a Vercel |
| Banco de dados | **PostgreSQL** (Neon ou Vercel Postgres) | Relacional, robusto para estoque/financeiro, com backups automáticos |
| Autenticação | **Google Workspace (OAuth 2.0 / SSO)** | Login com a conta @consertmaquinas (ou domínio da empresa), sem senhas soltas, permite exigir 2FA já configurado no Google Admin |
| Armazenamento de arquivos | **Google Drive (API)** | Guardar XML/DANFE das notas fiscais, fotos de equipamentos em manutenção, backups |
| Automação/planilhas | **Google Apps Script** | Rotinas auxiliares: exportar relatórios para Google Sheets, enviar e-mails automáticos (ex.: aviso de estoque baixo, OS concluída), gerar backups periódicos |
| Emissão fiscal | **Provedor de NF-e via API** (Focus NFe / PlugNotas) | Comunicação com a SEFAZ já homologada; o sistema só envia os dados e recebe o XML/DANFE pronto |

### Por que não usar só Google Sheets como banco de dados?
Sheets é ótimo para relatórios e é usado aqui como **complemento** (via Apps Script), mas não aguenta bem operações concorrentes (várias pessoas lançando estoque ao mesmo tempo), não tem boas travas de integridade (ex.: impedir estoque negativo) e fica lento com muitos dados históricos (notas fiscais, movimentações). Por isso o dado "oficial" fica no PostgreSQL, e o Sheets recebe cópias/relatórios quando for útil para a equipe.

## 3. Segurança

- **Login único via Google Workspace**: ninguém cria senha própria no sistema; usa a conta corporativa Google, com 2FA controlado pelo Google Admin.
- **Perfis de acesso (roles)**: admin, financeiro, estoque, técnico — cada um vê só o que precisa.
- **HTTPS obrigatório** (padrão da Vercel).
- **Backups automáticos**: banco de dados com backup diário; documentos fiscais também salvos no Google Drive como segunda cópia.
- **Log de auditoria**: toda alteração sensível (estoque, nota fiscal, cadastro de cliente) fica registrada com usuário e data/hora.
- **Segredos e chaves de API** (Google, provedor de NF-e) nunca ficam no código — vão em variáveis de ambiente da Vercel.

## 4. Módulos do sistema

1. **Clientes** — cadastro (nome/razão social, CPF/CNPJ, endereço, contato, histórico de OS e notas).
2. **Estoque/Produtos** — peças e produtos, código, preço de custo/venda, estoque mínimo, alertas automáticos.
3. **Entradas e Saídas** — movimentações de estoque (compra, uso em OS, ajuste, devolução).
4. **Ordens de Manutenção (OS)** — abertura, equipamento, diagnóstico, peças usadas (baixa automática no estoque), técnico responsável, status (aberta/em andamento/concluída), valor do serviço.
5. **Notas Fiscais** — geração a partir de uma venda ou de uma OS concluída, envio ao provedor de NF-e, guarda do XML/DANFE, consulta de status (autorizada/cancelada).
6. **Relatórios/Dashboard** — visão geral de vendas, estoque baixo, OS em aberto, faturamento.

## 5. Fases de implementação sugeridas

1. **Fase 1** — Estrutura base do projeto (Next.js + banco de dados + login Google) ✅ próximo passo
2. **Fase 2** — Cadastro de clientes e estoque (CRUD completo)
3. **Fase 3** — Entradas/saídas de estoque e ordens de manutenção
4. **Fase 4** — Integração com provedor de NF-e (depende do certificado digital e contratação do provedor)
5. **Fase 5** — Relatórios, dashboard e automações via Apps Script

## 6. Próximos passos concretos

- [ ] Você providenciar (com o contador): certificado digital e-CNPJ, confirmação de Inscrição Estadual/Municipal
- [ ] Escolher o provedor de NF-e (recomendo começar pelo Focus NFe, tem ambiente de testes grátis)
- [ ] Criar conta no GitHub (repositório do projeto) e na Vercel (pode logar direto com GitHub)
- [ ] Eu monto o projeto Next.js inicial com banco de dados e tela de login Google, pronto para deploy
