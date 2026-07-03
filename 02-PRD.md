# PRD — Product Requirements Document
## Sistema de Gestão Jurídica 2.0 — Menin & Deroma Advogados

**Versão:** 1.0
**Status:** Rascunho para validação
**Documentos relacionados:** `01-Brainstorm-Discovery.md` (base funcional completa), Design System (a elaborar), Documento de Arquitetura (a elaborar)

---

## 1. Visão do Produto

### 1.1 Problema
O escritório Menin & Deroma opera hoje sobre um sistema com módulos essenciais funcionando (Prazos, Pastas, Clientes, Ficha-Tempo, Financeiro básico), mas com lacunas relevantes: Financeiro raso (só Faturas e Despesas), ausência de Portal do Cliente robusto, ausência de workflow de revisão de documentos por senioridade, cadastro de clientes com campos livres sujeitos a erro, e nenhuma automação de captura de andamentos processuais.

### 1.2 Objetivo
Substituir o sistema atual por uma plataforma nova, construída em Clean Architecture, que:
- Centralize 100% da operação do escritório (contencioso, consultivo, financeiro, comunicação, compliance).
- Elimine erros operacionais estruturais (prazo perdido, documento no lugar errado, cobrança incorreta) através de automação e validação de dados.
- Ofereça um Portal do Cliente completo, com comunicação direta e bidirecional com o time interno.

### 1.3 Métricas de Sucesso (propostas — validar com o escritório)
- Zero prazos fatais perdidos por falha de sistema (meta operacional, não só técnica).
- Redução do tempo de coleta documental no onboarding (hoje manual via planilha) em X% (a definir baseline).
- Adoção do Portal do Cliente por Y% dos clientes ativos em até 90 dias do lançamento.
- Redução do tempo entre pagamento e emissão de NFS-e para dentro do SLA de 5 dias úteis em 100% dos casos.

### 1.4 Fora de Escopo (explicitamente não incluído nesta versão)
- Assinatura eletrônica com validade jurídica plena (ICP-Brasil) — MVP usa aceite simples com log de IP/timestamp.
- Consulta automatizada a listas sancionatórias (COAF/OFAC/ONU) — permanece registro manual no MVP.
- Onboarding de colaborador (RH) — módulo futuro, não desenhado nesta versão.
- Mapa visual de estrutura societária (organograma) — considerado diferencial de fase futura.
- Integração com sistema de contabilidade externo — financeiro é nativo (por trás dos panos, sem sistema externo).
- Design System detalhado (paleta, componentes, Login) — documento próprio, subsequente a este PRD.

---

## 2. Personas

| Persona | Perfil | Necessidade central |
|---|---|---|
| Sócio | Gestor + operador, responde por tudo | Visão consolidada, controle financeiro, aprovação de documentos e casos de risco |
| Advogado Sênior | Executa + revisa | Gerenciar carteira própria e revisar trabalho de Júnior/Estagiário |
| Advogado Pleno/Júnior | Executor de caso | Executar pastas atribuídas, cadastrar prazos, produzir documentos |
| Estagiário | Apoio, sob supervisão | Rascunhar, organizar, sem autonomia de publicação |
| Assistente | Apoio jurídico + geral | Suporte operacional a onboarding, agenda, organização |
| Administrativo/Financeiro | Retaguarda | Faturamento, cobrança, NFS-e |
| Cliente (Portal) | Contratante dos serviços | Acompanhar caso, se comunicar, pagar, sem jargão jurídico |

---

## 3. Escopo Funcional — Sistema Interno

> Cada módulo abaixo tem seu detalhamento funcional completo em `01-Brainstorm-Discovery.md`, seção 4. Este PRD resume o requisito e define prioridade.

| # | Módulo | Prioridade | Resumo do requisito |
|---|---|---|---|
| RF-01 | Autenticação e RBAC | MVP | Login, MFA, controle de acesso por Cargo × Setor × Escopo de Dados (Equipe da Pasta) |
| RF-02 | Dashboard | MVP | Visão inicial personalizada por Cargo, conforme especificado na seção 4.1 do Discovery |
| RF-03 | Clientes | MVP | Cadastro PF/PJ, KYC, Sociedades, Imóveis, Onboarding com checklist de 10 etapas e SLA |
| RF-04 | Pastas | MVP | Contencioso/Consultivo, Equipe da Pasta, Modalidade de Cobrança, encerramento com motivação |
| RF-05 | Prazos | MVP | Judicial/Consultivo/Interno, 3 calendários, alertas D-10/D-3/D-1, confirmação humana obrigatória |
| RF-06 | Andamentos automatizados | MVP | Integração com TJMG, Justiça Federal e Trabalhista via job assíncrono |
| RF-07 | Documentos + Workflow de Revisão | MVP | Numeração automática, fluxo Rascunho→Revisão→Publicado por Cargo |
| RF-08 | Financeiro | MVP | Propostas, Contratos, Faturas por Modalidade de Cobrança, NFS-e, Inadimplência |
| RF-09 | Ficha-Tempo | MVP | Lançamento com cronômetro/manual, vínculo a Pasta, base de faturamento |
| RF-10 | Agenda | MVP | Compromissos pessoais/equipe, combinável com Prazos |
| RF-11 | Mensagens | MVP | Chat time-time (grupos/privado) e time-cliente (escopado por vínculo), via Realtime |
| RF-12 | Arquivo | MVP | Árvore documental auto-gerada por cliente, upload manual + híbrido via SCRIPTUM |
| RF-13 | VAULT (Biblioteca) | MVP | Modelos e repositório institucional, permissão por Cargo |
| RF-14 | Relatórios | MVP | Dashboards internos + Relatório do Cliente dinâmico, publicado no Portal |
| RF-15 | Portal do Cliente completo | MVP | Todos os 8 módulos especificados na seção 5 do Discovery |
| RF-16 | Notificações multicanal | MVP | Push + E-mail + WhatsApp Business API |
| RF-17 | Compliance/LGPD | MVP | Solicitações de titulares, incidentes, retenção de dados |
| RF-17b | Arquitetura de Navegação (Sidebar) | MVP | Menu agrupado (Principal/Institucional/Ferramentas/Sistema) com visibilidade por Cargo/Setor — árvore completa em `01-Brainstorm-Discovery.md`, seção 4.17 |
| RF-18 | Calculadoras Jurídicas | Fase 2 | 8 calculadoras, standalone |
| RF-19 | Institucional embutido | Fase 2 | BrandBook e Políticas Internas dentro do sistema |
| RF-20 | Onboarding de Colaborador | Fase 2 | Não desenhado — futuro |
| RF-21 | Mapa societário visual | Fase 2 | Organograma Pessoa↔Empresas |
| RF-22 | Assinatura eletrônica plena | Fase 2 | Evolução do aceite simples de proposta |

**Critério de priorização usado:** um item é MVP se (a) está diretamente ligado à operação diária que hoje já existe e não pode regredir, ou (b) resolve uma lacuna crítica identificada no sistema atual (Financeiro raso, Portal ausente, workflow de revisão ausente). Fase 2 = valor real, mas não bloqueia a operação no dia 1.

---

## 4. Escopo Funcional — Portal do Cliente

| # | Módulo | Prioridade |
|---|---|---|
| RF-P01 | Login + Onboarding (múltiplos usuários por Cliente) | MVP |
| RF-P02 | Dashboard do Cliente | MVP |
| RF-P03 | Minhas Pastas (timeline traduzida) | MVP |
| RF-P04 | Documentos (visibilidade seletiva) | MVP |
| RF-P05 | Financeiro (propostas, faturas, pagamento) | MVP |
| RF-P06 | Mensagens (escopadas por vínculo) | MVP |
| RF-P07 | Relatórios (histórico dinâmico) | MVP |
| RF-P08 | Checklist de documentos com upload | MVP |
| RF-P09 | Perfil + Meus Dados (exercício de direitos LGPD) | MVP |

---

## 5. Requisitos Não-Funcionais

| Categoria | Requisito |
|---|---|
| Confiabilidade | Zero perda de prazo por falha de sistema; toda ação crítica (confirmar prazo, publicar documento, gerar fatura) deve ter confirmação explícita e auditoria (quem fez, quando) |
| Segurança | MFA obrigatório no Interno; senha mínima 12 caracteres com troca a cada 90 dias (conforme política vigente); RLS no Supabase como camada extra de defesa; dados sensíveis nunca em URL/query string |
| Privacidade/LGPD | Consentimento registrado com timestamp; resposta a titular em até 15 dias úteis; retenção de dados por 5 anos após encerramento de pasta, com expurgo controlado |
| Performance | PWA com funcionamento offline básico (cache de assets, consulta a dados já carregados); tempo de resposta de listagens < 2s em condições normais |
| Disponibilidade | Sistema é operação crítica do escritório — meta de disponibilidade a definir com o time de infraestrutura (SLA de uptime) |
| Auditabilidade | Toda mudança de status (Pasta, Prazo, Documento, Fatura) registrada em histórico visível (padrão já usado em Pastas, a generalizar) |
| Escalabilidade de arquitetura | Clean Architecture obrigatória — regras de negócio isoladas de infraestrutura (Supabase), permitindo troca de provedor sem reescrever domínio |
| Usabilidade | Cadastro rápido de prazo acessível globalmente (1-2 cliques de qualquer tela); grid como padrão de visualização em módulos de conteúdo visual |
| Compatibilidade | PWA instalável em desktop e mobile, dois domínios distintos (subdomínios dedicados via Vercel) |

---

## 6. Fluxos Críticos (a detalhar com critérios de aceite no backlog)

1. **Onboarding de Cliente completo** — do cadastro inicial à liberação do Portal, passando pelas 10 etapas com SLA.
2. **Ciclo de vida de um Prazo** — identificação (automática ou manual) → confirmação humana → alertas → cumprimento/perda registrados.
3. **Ciclo de vida de um Documento** — criação (SCRIPTUM ou upload) → revisão (se aplicável ao Cargo) → publicação → visibilidade no Portal (se marcado).
4. **Ciclo de faturamento** — Ficha-Tempo/evento de êxito/recorrência → geração de fatura → pagamento → emissão de NFS-e → registro no Arquivo.
5. **Fluxo de inadimplência** — vencimento → alerta interno (15 dias) → comunicação formal ao cliente → acompanhamento até quitação ou decisão do Sócio.

---

## 7. Dependências e Riscos

| Risco | Impacto | Mitigação proposta |
|---|---|---|
| Integração com tribunais (TJMG/JF/Trabalhista) pode ter instabilidade ou mudança de API sem aviso | Alto — afeta captura de andamentos e prazos | Isolar via Adapter (Clean Architecture); monitoramento de falha de sincronização com alerta ao T.I |
| Emissão de NFS-e depende de certificado digital A1 e webservice municipal | Alto — afeta compliance fiscal | Fila com retry e alerta se emissão falhar antes do prazo de 5 dias úteis |
| WhatsApp Business API exige aprovação de templates pela Meta | Médio — pode atrasar lançamento do canal | Validar aprovação de templates antes do go-live; ter Push+E-mail como fallback |
| Volume real de jobs assíncronos ainda não dimensionado (BullMQ vs. Edge Functions) | Médio — decisão técnica pendente | Resolver no Documento de Arquitetura, com teste de carga estimado |
| Aceite eletrônico simples (MVP) pode não ter força jurídica equivalente a assinatura ICP-Brasil | Médio — jurídico, não técnico | Validar com o próprio escritório se aceite simples é suficiente para os contratos de honorários |
| Rendering de HTML institucional (BrandBook/Políticas) exige isolamento (iframe sandbox) para evitar XSS | Médio — segurança | Nunca injetar HTML de upload diretamente no DOM; servir via Storage + iframe sandbox, decisão a formalizar na Arquitetura |

---

## 8. Roadmap Proposto (alto nível)

**Fase 1 — MVP:** todos os itens marcados MVP nas seções 3 e 4, incluindo Portal do Cliente completo. Este é o "sistema mínimo que substitui integralmente o atual sem perda de funcionalidade e com os ganhos essenciais (Financeiro robusto, Portal, workflow de revisão)".

**Fase 2:** Calculadoras Jurídicas, Institucional embutido, Onboarding de Colaborador, Mapa societário, Assinatura eletrônica plena, permissões granulares do VAULT por categoria, consulta automatizada a listas sancionatórias.

**Fase 3 (não escopada ainda):** integrações adicionais conforme necessidade (contabilidade externa, novos tribunais/estados se o escritório expandir atuação geográfica).

---

## 8.5 Decisões Visuais Já Travadas (input obrigatório para o Design System)

Embora este PRD não defina o Design System, os seguintes itens visuais já foram aprovados pelo cliente e **não devem ser reabertos** quando elaborarmos aquele documento:
- Linha divisória entre grupos do menu lateral: gradiente horizontal navy→dourado→navy, fina e sutil.
- Paleta base: navy escuro + dourado (heranças do BrandBook institucional).
- Grid como visualização padrão em módulos de conteúdo visual (Arquivo, VAULT, Documentos).

---

## 9. Próximos Passos

1. Validação deste PRD com os sócios do escritório (priorização MVP vs. Fase 2 pode mudar).
2. Elaboração do **Design System** (paleta, tipografia, componentes, telas-chave incluindo Login).
3. Elaboração do **Documento de Arquitetura Técnica** (modelagem de dados, diagramas de Clean Architecture, decisão final sobre fila de jobs, detalhamento de RLS vs. Use Cases).
4. Quebra do PRD em backlog detalhado (épicos → user stories → critérios de aceite) para entrada em desenvolvimento.
