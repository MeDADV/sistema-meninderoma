# Menin & Deroma — Sistema de Gestão Jurídica 2.0
## Documento de Brainstorm / Discovery

**Status:** Base consolidada para elaboração do PRD e Documento de Arquitetura
**Escopo:** Reescrita completa do sistema interno + criação do Portal do Cliente

---

## 1. Visão Geral e Princípios Norteadores

- Sistema construído do zero, substituindo o sistema atual, mantendo o que já funciona bem e evoluindo o que é fraco ou inexistente.
- Requisito não-funcional máximo: **erro-free e 100% funcional** — toda a operação do escritório passa a depender deste sistema.
- Código em **Clean Architecture**, tanto no backend quanto no frontend.
- Dois públicos, duas superfícies: **Sistema Interno** (equipe do escritório) e **Portal do Cliente**, com comunicação direta entre ambos.
- Baseado no sistema real em uso (Menin & Deroma Advogados) — todas as decisões abaixo foram validadas contra prints do sistema atual e documentos institucionais (Políticas Internas, BrandBook).

---

## 2. Arquitetura de Alto Nível

- **Dois PWAs separados**, mesma API/backend:
  - PWA Interno (equipe)
  - PWA Portal do Cliente
- **Backend:** NestJS + TypeScript, organizado em Clean Architecture (Entities / Use Cases / Interface Adapters / Infrastructure).
- **Banco de dados / Infra:** **Supabase** (Postgres gerenciado, Auth, Storage, Realtime).
  - NestJS orquestra toda regra de negócio complexa (motor de prazos, workflow de revisão, motor de faturamento, escopo de visibilidade, integrações externas).
  - Supabase Auth valida identidade; NestJS valida/usa o token.
  - Supabase Storage guarda documentos (Arquivo, VAULT, Documentos de Pasta).
  - Supabase Realtime usado diretamente pelo frontend **apenas** para o módulo de Mensagens/Chat (exceção deliberada de acesso direto frontend↔infra).
  - RLS do Supabase como camada extra de defesa (segurança em profundidade); a fonte da verdade da regra de negócio vive nos Use Cases do NestJS.
- **Frontend:** React + TypeScript + Vite + Workbox (Service Worker/PWA), também estruturado em Clean Architecture (telas não acessam API diretamente — passam por Use Cases/Repositories abstratos).
- **Jobs assíncronos:** fila (BullMQ/Redis ou equivalente compatível com Supabase) para: consulta a tribunais, envio de notificações (Push/E-mail/WhatsApp), emissão de NFS-e, geração de faturas recorrentes.
- **Integrações externas identificadas:**
  - Tribunais: TJMG (estadual) + Justiça Federal + Justiça do Trabalho (via DataJud/CNJ e sistemas próprios), consulta assíncrona.
  - NFS-e: webservice da prefeitura (certificado digital A1), por trás dos panos — usuário nunca sai do sistema.
  - Pagamento: gateway (Pix/boleto/cartão) por trás dos panos.
  - WhatsApp Business API (Meta Cloud API) como canal formal de notificação.
- **Design visual:** ainda não abordado (deliberadamente) — será tratado em documento próprio de Design System, após fechamento funcional. Referências já capturadas: paleta navy/dourado, tipografia serifada em títulos, padrão de card de métrica, grid como visualização padrão sempre que aplicável (Arquivo, VAULT, Calculadoras, Documentos).
- **Hospedagem/Domínio:** Vercel, dois projetos/deploys independentes como subdomínios de `meninderoma.com.br` (domínio raiz permanece com o site institucional). Ex.: `sistema.meninderoma.com.br` (Interno) e `portal.meninderoma.com.br` (Cliente) — nomes finais a definir. Cada PWA tem sua própria tela de Login (decisão que substitui a ideia anterior de Login único com link de desvio para o Portal).

---

## 3. Modelo de Permissões (RBAC + Escopo de Dados)

Três eixos independentes, combinados:

### 3.1 Cargo (define o que o usuário PODE FAZER)
- Sócio/Partner
- Advogado Sênior
- Advogado Pleno
- Advogado Júnior
- Estagiário
- Assistente
- Consultor (status: standby, permissões não definidas ainda)

### 3.2 Setor (define QUAIS MÓDULOS o usuário vê — relação N:N, usuário pode ter múltiplos setores)
- Administrativo
- T.I
- Recursos Humanos
- Atendimento ao Cliente
- Marketing
- Contencioso
- Consultivo

### 3.3 Área de Atuação (tag de especialidade — aplicada a Pasta e a Advogado, sem efeito em permissão)
- Cível Empresarial
- Societário e M&A
- (Observação em aberto: sinais de Direito de Família/Sucessório e Imobiliário aparecem em ferramentas do sistema atual — revisitar se cabe 3ª área)

### 3.4 Regras de Permissão por Cargo

| Cargo | Publica documento sem revisão? | Outras regras |
|---|---|---|
| Sócio/Partner | Sim | Acesso total, aprova tudo, financeiro total, gerencia usuários |
| Advogado Sênior | Sim | Revisa/aprova peças de Júnior e Estagiário |
| Advogado Pleno | Sim | Gerencia pastas próprias, sem poder de revisão sobre outros |
| Advogado Júnior | Não — precisa de revisão de Sênior/Sócio | Fluxo de aprovação obrigatório |
| Estagiário | Não — nunca publica | Nunca confirma prazo fatal |
| Assistente | N/A | Apoio jurídico + geral (transita entre setores) |
| Consultor | A definir | Standby |

### 3.5 Escopo de Dados — Visibilidade de Pastas (dimensão adicional, independente de Cargo/Setor)

| Cargo | Vê quais Pastas? |
|---|---|
| Sócio/Partner | Todas, sem restrição |
| Advogado Sênior | Todas as pastas do(s) Setor(es) a que pertence |
| Advogado Pleno | Só pastas em que está na `EquipeDaPasta` |
| Advogado Júnior | Só pastas em que está na `EquipeDaPasta` |
| Estagiário | Só pastas em que está na `EquipeDaPasta` |
| Assistente | Todas as pastas do(s) Setor(es) a que pertence |
| Consultor | A definir |

- **Cliente** só aparece na listagem se houver ≥1 Pasta visível daquele cliente para o usuário logado (visibilidade derivada).
- Regra vive como `PastaVisibilityPolicy` central na camada de Application — reutilizada em toda query de Pasta/Cliente/Relatório.

### 3.6 Exceção de Governança — Gestão de Usuários
- **Sócio** e **Setor T.I** têm controle total e equivalente sobre usuários (criar, editar, excluir, alterar Cargo/Setor), independente do Cargo individual da pessoa em T.I.

### 3.7 Permissão especial — VAULT (Biblioteca)
- Podem editar/publicar na aba Gestão: **Sócio, Sênior, Pleno e Administrativo**.
- Júnior/Estagiário: somente consulta (aba Biblioteca).

---

## 4. Módulos do Sistema Interno

### 4.1 Dashboard (varia por Cargo)

Template estrutural comum: 3 cards de métrica no topo → 2 blocos lado a lado → bloco de largura total → Comunicados Recentes no rodapé (presente para todos os cargos).

**Sócio/Partner** (o mais rico — organizado em seções: Operação / Financeiro / Gestão):
- Operação: Pastas Ativas, Pastas por Tipo (Contencioso × Consultivo), Prazos (7 dias), Prazos Críticos (≤5 dias)
- Financeiro: A Receber, Em Atraso/Inadimplência, Faturamento do Mês, NFS-e Pendentes
- Gestão: Fila de Revisão de Documentos pendentes, Onboardings de Cliente em andamento (com SLA), Solicitações LGPD pendentes
- Blocos: Prazos Críticos + Agenda de Hoje (lado a lado), Últimos Andamentos, **Pastas com Atenção Necessária** (combina prazo crítico + documento aguardando revisão + fatura vencida, priorizado), Comunicados

**Advogado Sênior:**
- Cards: Horas Lançadas Hoje, Mensagens Não Lidas, Próximo Compromisso
- Fila de Revisão (peças de Júnior/Estagiário aguardando aprovação dele) — destaque
- Pastas do Setor com prazo crítico
- Minhas Pastas Ativas, Meus Prazos da Semana, Agenda do Dia, Widget Ficha-Tempo (com cronômetro), Comunicados

**Advogado Pleno:** igual estrutura do Sênior, sem Fila de Revisão de terceiros.

**Advogado Júnior:** igual ao Pleno + card "Aguardando Revisão" (status dos documentos enviados: Em revisão/Aprovado/Devolvido).

**Estagiário:** igual estrutura base + card "Tarefas/Rascunhos pendentes de finalizar".

**Assistente:** Checklist de Onboarding de Clientes em andamento (etapas "Equipe"), Agenda do Dia, Registro de Atividades (Ficha-Tempo não-faturável por padrão), Comunicados.

**Administrativo/Financeiro:** Faturamento do mês, A Receber, Em Atraso, NFS-e pendentes, Propostas de Honorários aguardando aceite, Fila de cobrança (vencidas 15+ dias).

**T.I:** Usuários ativos/inativos, convites de Portal pendentes, Log de Auditoria recente, status de integrações externas.

**RH:** Aniversariantes do mês, pendências de onboarding de colaborador (módulo futuro, não detalhado), distribuição da equipe por Cargo/Setor.

**Atendimento ao Cliente:** Mensagens não lidas (fila unificada, com indicador de SLA em risco — WhatsApp 4h / e-mail 1 dia útil), convites de Portal pendentes de ativação.

**Marketing:** Indicadores de Uso (acessos ao Portal, engajamento), Indicadores de Negócio (captação por origem, conversão, faturamento por Área de Atuação), Comunicados publicados recentemente + criar novo.

**Comunicados internos:** entidade única `Comunicado` (autor, título, corpo, público-alvo por Cargo/Setor, data). Sócio e Marketing publicam. Mesmo mecanismo alimenta o mural do Portal do Cliente quando marcado como `visivelNoPortal`.

---

### 4.2 Clientes

**Lista de Clientes** (escopada pela regra de visibilidade): Cliente, CPF/CNPJ, Contato, Sócio Responsável, Status, Onboarding (barra de progresso X/10), Origem (Captação/Indicação), nº Pastas, Status do Portal, Data de cadastro. Filtros por Área de Atuação, Setor, Status.

**Detalhe do Cliente** — abas:
- Identificação (nome, CPF/CNPJ, tipo PF/PJ, nacionalidade, estado civil, regime de bens, profissão, cônjuge)
- Endereço
- Contatos (e-mail principal/secundário, telefone, pessoa de contato se PJ, canal de comunicação preferido)
- Documentos (checklist dinâmico — ver 4.2.3)
- Membros/Sócios (qualificação de sócios/membros vinculados)
- Sociedades (participações societárias — Razão Social, CNPJ, NIRE, sócios/participações, capital social, última ACS, balanço/DRE, status docs)
- Imóveis (matrícula, CRI/Cartório, município/UF, descrição, proprietário, gravame, valor IR, certidão, status docs)
- KYC (conflito de interesses, PEP, listas sancionatórias + data, risco Baixo/Médio/Alto — informativo, sem bloqueio, LGPD assinada)
- Contrato (proposta enviada/aceita, valor, forma de pagamento, contrato assinado, procuração)
- Checklist de Onboarding
- Financeiro consolidado
- Usuários do Portal vinculados (múltiplos por Cliente — convidar/revogar)
- Histórico de Relatórios do Cliente enviados
- Observações internas (campo sigiloso, visível apenas à equipe)

**Cadastro de Cliente:** Tipo (PF/PJ), Origem (Captação/Indicação + indicado por), dados básicos, Sócio Responsável, Status, Observações internas. Ao salvar, dispara: criação automática da árvore no Arquivo + criação do `OnboardingChecklist`.

#### 4.2.1 Onboarding de Cliente — Checklist de 10 etapas (com SLA)

| # | Etapa | Responsável | Prazo |
|---|---|---|---|
| 1 | Verificação de conflito de interesses | Sócio | D+0 |
| 2 | Identificação e qualificação do cliente | Equipe | D+1 |
| 3 | KYC/Compliance (LGPD + PEP + listas) | Sócio | D+1 |
| 4 | Coleta de documentos | Equipe + Cliente | D+3 |
| 5 | Proposta de honorários enviada | Sócio | D+1 |
| 6 | Contrato de honorários assinado | Ambos | D+3 |
| 7 | Abertura da pasta no sistema | Equipe | D+3 |
| 8 | Acesso ao Portal do Cliente criado | Equipe | D+3 |
| 9 | Reunião de alinhamento inicial | Sócio + Cliente | D+5 |
| 10 | Cadastro de prazos iniciais | Equipe | D+5 |

- Classificação de Risco (Baixo/Médio/Alto): apenas informativa, não bloqueia o fluxo.
- Coleta de documentos: híbrida — Portal do Cliente tem checklist interativo de upload (cliente sobe o que falta) + Equipe também pode fazer upload manual pelo Sistema Interno.

#### 4.2.2 Fluxo de Convite ao Portal
1. Sócio/Advogado cadastra cliente → cria pasta no Arquivo + dispara convite.
2. Convite por e-mail, link com token único, expira em X dias.
3. Cliente define senha, aceita Termos + Política de Privacidade (consentimento LGPD registrado com timestamp).
4. Cliente pode ter **múltiplos usuários vinculados** (ex: diretoria de uma PJ) — `Cliente` (relação contratual) é diferente de `UsuarioPortal` (quem loga), relação 1:N.
- Pendência em aberto: hierarquia entre múltiplos usuários de um mesmo Cliente (todos veem tudo igual, ou existe usuário principal com mais acesso — ex: Financeiro/aceite de honorários)?

#### 4.2.3 Documentos do Cliente — Checklist dinâmico (baseado em planilha real do escritório)

Gerado condicionalmente conforme características do cliente:

**Sempre — Pessoa Física** (8 itens): RG/CNH, CPF, comprovante de endereço (90 dias), certidão nascimento/casamento, última DIRPF, recibo entrega IRPF, extrato bancário (3 meses, se necessário), procuração (se representado).

**Sempre — Pessoa Jurídica** (10 itens): contrato social consolidado + alterações, ata de eleição de administradores, cartão CNPJ, balanço patrimonial + DRE, balancete atualizado, QSA, comprovante de endereço da sede, certidão simplificada Junta Comercial, certidões de regularidade fiscal (federal/estadual/municipal).

**+ Societário/M&A** (se aplicável, 5 itens): acordo de sócios, laudos de avaliação, deliberações/atas relevantes, certidões de protesto, due diligence anterior.

**+ Imóveis** (se aplicável, por imóvel, 5 itens): certidão de matrícula (30 dias), guia IPTU, escritura de aquisição, certidão de ônus reais, declaração de valor.

**+ Processual** (se aplicável, 5 itens): procuração ad judicia, substabelecimento, petição inicial/última peça, documentos que instruem a inicial, contratos/acordos objeto da ação.

Cada `Sociedade` e cada `Imóvel` cadastrado gera seu próprio sub-checklist de status de documentos.

---

### 4.3 Pastas

**Tipo:** Contencioso | Consultivo (campo raiz, determina campos obrigatórios subsequentes)
**Área de Atuação:** Cível Empresarial | Societário e M&A
**Numeração:** `MD-2026/001` (sequencial anual)

**Cadastro (Nova/Editar Pasta):**
- Tipo, Área, Cliente, Empresa/Entidade (select vinculado às Sociedades já cadastradas do Cliente — não texto livre)
- Responsável (Advogado)
- Se Contencioso: Tribunal + Número do Processo (formato CNJ)
- Se Consultivo: Órgão (JUCEMG, extensível — entidade `Órgão` própria, com calendário de dias úteis próprio) + Número de Protocolo
- Valor do Contrato (valor da causa)
- Modalidade de Cobrança (ver 4.3.2)
- Descrição

**Detalhe da Pasta — abas:**
Resumo · Andamentos · Prazos · Documentos · Financeiro · Agenda · Equipe · **Ficha-Tempo** · **Mensagens**

- **Resumo:** informações gerais + Histórico de Status (timeline com quem/quando mudou status — padrão de auditoria a generalizar para outras entidades).
- **Equipe (`EquipeDaPasta`):** lista explícita de membros (Sócio responsável, Advogado(s), Estagiário(s)) — base da regra de escopo de visibilidade (seção 3.5).
- **Ficha-Tempo:** horas lançadas especificamente nesta pasta.
- **Mensagens:** conversa com o cliente escopada a esta pasta (ver seção 4.6).

**Encerramento de Pasta:** fluxo com motivação obrigatória (conforme política interna), muda status para Encerrada, dispara devolução/arquivamento de documentos originais e oferece geração de relatório final ao cliente.

#### 4.3.1 Andamentos (Contencioso — automatizado)
1. Job periódico consulta tribunais (TJMG, Justiça Federal, Justiça do Trabalho) via integração assíncrona.
2. Novo andamento detectado → sistema pode sugerir se gera prazo (regra + eventual apoio de IA revisável).
3. Prazo sugerido aparece para confirmação humana do advogado responsável (nunca criado como definitivo sem revisão).
4. Campo `descricaoParaCliente`: preenchido manualmente pelo advogado (100% manual, sem IA) — sem esse campo preenchido, o andamento não aparece no Portal (padrão restritivo).

#### 4.3.2 Modalidade de Cobrança

| Modalidade | Comportamento no Financeiro |
|---|---|
| Fixo Mensal (Retainer/Partido) | Fatura recorrente automática; Ficha-Tempo é só controle interno |
| Fee por Projeto | Fatura única (ou parcelada) ao fechar contrato |
| Êxito | Fatura gerada quando evento de "resultado obtido" é registrado (% de êxito configurável) |
| Horas | Fatura a partir da Ficha-Tempo faturável do período, com revisão antes de converter |
| Fixo/Fee + Êxito | Híbrido: parte fixa + percentual condicional |

---

### 4.4 Prazos

**Tipos (categoria estrutural):**
- **Judicial (Contencioso):** vinculado a Processo, contagem por calendário forense (CPC, feriados forenses, suspensões).
- **Consultivo:** vinculado a Pasta Consultiva, majoritariamente Junta Comercial (JUCEMG) — contagem em dias úteis por calendário próprio do Órgão (entidade extensível).
- **Interno:** não vinculado a órgão externo — meta/entrega interna do escritório, prazo configurável (pode ser curtíssimo).

**Alertas automáticos:** D-10, D-3, D-1 (Push + E-mail + WhatsApp, conforme regra de notificação).

**Fluxo automático (Judicial):** andamento capturado → IA sugere prazo → advogado confirma (nunca automático).
**Fluxo manual:** botão de "cadastro rápido de prazo" acessível globalmente (requisito da política: cadastro imediato à identificação).

**Telas:** Dashboard de prazos (visão pessoal/equipe conforme Cargo), Kanban/Lista filtrável, Calendário (mensal/semanal, combinável com Agenda), Fila de confirmação de prazo sugerido por IA.

---

### 4.5 Documentos + Workflow de Revisão

Três repositórios distintos, não confundir:

| Módulo | Propósito |
|---|---|
| VAULT | Modelos/templates reutilizáveis do escritório |
| Arquivo | Documentos reais por Cliente/Empresa (árvore auto-gerada) |
| Documentos da Pasta | Peças vinculadas a um processo/consultoria específico, com numeração |

**Numeração automática por tipo:** Pareceres `PAR-001/2026`, Memorandos `MEM-001/2026`, Notas Técnicas `NT-001/2026`.

**Workflow:** Rascunho → Em Revisão → Publicado. Júnior/Estagiário sempre passam por revisão de Sênior/Sócio antes de publicar. Pleno/Sênior/Sócio publicam direto.

**Campo `visivelNoPortal`** (boolean, padrão `false`): controla o que aparece no Portal do Cliente — decisão explícita da equipe, nunca automática.

**Integração SCRIPTUM → Arquivo (fluxo híbrido):** documento gerado via SCRIPTUM sugere automaticamente destino no Arquivo (cliente/empresa + categoria), usuário confirma com 1 clique. Upload manual comum continua manual, como hoje.

---

### 4.6 Mensagens

**Time ↔ Time:**
- Grupos coletivos (por setor, pasta, ou ad-hoc)
- Chats privados 1:1

**Time ↔ Cliente:**
- Escopado por vínculo: usuário só vê mensagens de clientes com quem está diretamente envolvido (via Pasta/responsável designado).
- Sócio vê tudo (acesso total).
- Implementado via Supabase Realtime (acesso direto frontend↔infra, exceção arquitetural deliberada).

---

### 4.7 Ficha-Tempo

- Lançamento com **Cronômetro** (Iniciar/Resetar) ou **Manual**.
- Campos: Responsável, Data, Pasta/Processo, Cliente (auto-preenchido), Tipo de Atividade, Área, Faturável (Sim/Não), Descrição, Tempo.
- Visão pessoal ("Minha Ficha-Tempo") e consolidada (Sócio/Administrativo, por pasta/advogado/período).
- Alimenta diretamente o Financeiro nas modalidades Horas e Híbrida.
- Assistente: lança hora como controle de produtividade, não-faturável por padrão (ajustável).

---

### 4.8 Agenda

- Distinta do Calendário de Prazos (prazo = obrigação legal com contagem; Agenda = reuniões, audiências, compromissos).
- Agenda Pessoal e Agenda da Equipe (visão Sócio).
- Visão combinável com Calendário de Prazos (camadas ligáveis/desligáveis).

---

### 4.9 VAULT (Biblioteca)

- Duas abas: **Biblioteca** (consulta geral) e **Gestão** (edição/publicação — permissão: Sócio, Sênior, Pleno, Administrativo).
- Categorias extensíveis (pastas): Imagens, Contratos, Políticas, Apresentações, Guias, Pareceres + criação de novas categorias.
- Duas responsabilidades internas:
  - `RepositorioInstitucional` (logos, apresentações, políticas — mais estático)
  - `ModeloDeDocumento` (contratos, minutas, pareceres, com schema/campos — alimenta o SCRIPTUM)
- Visualização em **grid** (padrão do sistema).

---

### 4.10 Calculadoras Jurídicas

- Taxonomia própria (não relacionada a Área de Atuação): Gestão, Geral, Societário, Processual, Família, Imobiliário.
- 8 calculadoras identificadas: Honorários Advocatícios, Atualização Monetária, Juros Simples e Compostos, Multa Contratual, Divisão de Quotas Societárias, Cálculo de Prazo Processual, Pensão Alimentícia, Aluguel.
- Standalone por ora (integração com módulo Prazos avaliada e descartada no MVP).

---

### 4.11 Arquivo (Gestão Documental)

- Estrutura: **Escritório** (documentos administrativos internos) + **Clientes** (1 pasta por cliente, auto-criada no cadastro).
- Subpastas padrão auto-geradas por Cliente (e replicadas por Empresa/Sociedade vinculada): Apresentações & Relatórios, Atos Registrados, Docs Bens, Docs Empresas, Docs Sócios, Guias e Formulários, Minutas.
- Upload: manual (padrão) + híbrido quando originado do SCRIPTUM (sugestão automática de destino, confirmação em 1 clique).
- Visualização em **grid** (padrão do sistema, com opção lista/grid conforme print observado).

---

### 4.12 Financeiro

**Dashboard:** Faturamento Total, A Receber, Em Atraso, Despesas, Resultado Líquido (cards padrão), gráfico Receitas×Despesas.

**Abas:**
1. **Faturas** — lista com filtro por status, cliente, período; mostra Pasta de origem, Modalidade, valor, vencimento, status NFS-e vinculada.
2. **Despesas** — lançamento por categoria, vinculável a Pasta quando repassável ao cliente.
3. **Propostas de Honorários** (novo) — Rascunho → Enviada → Aceita/Recusada. Aceite eletrônico: mecanismo formal ainda em aberto (MVP: log de IP/timestamp; evolução futura: assinatura eletrônica completa).
4. **Contratos de Honorários** (novo) — repositório com histórico de versões (alteração de escopo exige novo aceite).
5. **NFS-e** (novo) — fila de emissão, status, alerta se ultrapassar 5 dias úteis (SLA da política interna).
6. **Inadimplência** (novo) — régua automática (15+ dias → alerta ao Sócio + e-mail formal ao cliente via modelo VAULT/SCRIPTUM), painel com dias em atraso e última ação tomada.

**Fluxo ponta-a-ponta (exemplo Hora):** Ficha-Tempo faturável → revisão no fechamento do período → Fatura consolidada → Portal do Cliente + notificação (Push/E-mail/WhatsApp) → pagamento via gateway → webhook confirma → NFS-e emitida automaticamente → tudo registrado no Arquivo do cliente.

---

### 4.13 Relatórios

**Aba Escritório:** Financeiro, Pastas (distribuição por área/tipo, abertura ao longo do tempo), Prazos (cumpridos/perdidos/taxa), com exportação CSV/PDF.

**Aba Minhas Pastas:** mesma estrutura, escopada ao usuário.

**Aba Relatório do Cliente:** seleção Cliente + Período → Resumo Executivo e Próximos Passos (texto editorial) + dados puxados automaticamente (Pastas ativas, Andamentos, Prazos, Documentos compartilhados) → **Baixar PDF** ou **Enviar por e-mail**.
- **Dinâmico** (não é snapshot): reflete dados atualizados sempre que reaberto, tanto internamente quanto no Portal.
- **Publicado automaticamente no Portal do Cliente** ao gerar/enviar — histórico acessível, sempre com dados live. PDF exportado é o único artefato "congelado" (ponto-in-time).

---

### 4.14 Institucional (dentro do sistema)
- BrandBook
- Políticas Internas
- **Formato de conteúdo:** arquivos HTML autocontidos (é assim que o BrandBook e as Políticas Internas existem hoje — documentos ricos, com estilo próprio embutido).
- **Requisito de manutenção:** troca fácil do conteúdo quando o documento for atualizado (gera-se um novo arquivo HTML) — sem depender de deploy de código. Fluxo proposto:
  1. Sócio (ou Setor com permissão) acessa a tela de gestão do item Institucional (BrandBook / Políticas Internas).
  2. Faz upload do novo arquivo `.html`, que substitui o anterior (versão anterior mantida em histórico, não perdida — reaproveitando o padrão de auditoria já usado em outras entidades).
  3. Sistema armazena o arquivo no Supabase Storage; a tela apenas referencia o arquivo mais recente daquele item.
- **Renderização — decisão técnica que precisa ir para o Documento de Arquitetura:** HTML de terceiros/upload nunca deve ser injetado diretamente no DOM da aplicação (`dangerouslySetInnerHTML` ou equivalente), por risco de XSS caso o arquivo contenha script malicioso ou seja substituído por engano com conteúdo não confiável. **Recomendação:** renderizar em `<iframe sandbox>` apontando para o arquivo servido pelo Storage, isolando completamente o HTML do restante da aplicação. Isso também tem a vantagem de preservar 100% o estilo/layout original do arquivo HTML (que já vem com CSS próprio, como visto nos exemplos), sem conflito com o CSS do sistema.
- Cada versão trocada gera entrada no Histórico de Status (mesmo padrão usado em Pastas), com quem trocou e quando — relevante porque Políticas Internas tem peso normativo (funcionários "declaram ter lido e compreendido").

### 4.15 Compliance / LGPD
- Painel de Solicitações de Titulares (prazo 15 dias úteis)
- Registro de Incidentes de Segurança
- Painel de Retenção de Dados (pastas com 5 anos vencendo, elegíveis a expurgo)

### 4.16 Admin / Sistema
- Gestão de Usuários (Sócio + Setor T.I)
- Log de Auditoria
- Configurações
- Cadastro de Órgãos (JUCEMG e futuros, com calendário próprio)

---

## 4.17 Arquitetura de Navegação (Sidebar/NavBar)

### 4.17.1 Decisão visual travada (não-negociável, vale para o Design System)
- Linha divisória entre grupos de menu: gradiente horizontal navy→dourado→navy (fina, sutil), conforme referência aprovada. Aplicar entre todos os grupos da Sidebar do Sistema Interno.

### 4.17.2 Menu — Sistema Interno (MVP)

**Grupo PRINCIPAL**
Dashboard · Clientes · Pastas · Prazos · Agenda · Mensagens · Ficha-Tempo · Arquivo · Financeiro · Relatórios

**Grupo INSTITUCIONAL**
BrandBook · Políticas Internas

**Grupo FERRAMENTAS**
M&D | VAULT

**Grupo SISTEMA**
Admin de Usuários · Log de Auditoria · Cadastro de Órgãos · Compliance/LGPD · Configurações

- **M&D | SCRIPTUM** e **M&D | Links**: fora do menu no MVP (Fase 2).
- **M&DCALC (Calculadoras Jurídicas)**: Fase 2 (conforme priorização já definida no PRD) — quando entrar, provavelmente no grupo Ferramentas junto ao VAULT.

### 4.17.3 Menu — Portal do Cliente (MVP)

Dashboard · Minhas Pastas · Documentos · Financeiro · Mensagens · Relatórios · Checklist de Documentos · Perfil

### 4.17.4 Visibilidade de item de menu por Cargo/Setor (resumo — regra completa na seção 3)

| Item de menu | Regra de visibilidade |
|---|---|
| Dashboard, Agenda, Mensagens, Arquivo | Todos os Cargos |
| Clientes, Pastas, Prazos | Escopados conforme regra de visibilidade de dados (seção 3.5) — item aparece para todos, conteúdo é que é filtrado |
| Ficha-Tempo | Todos (uso varia: faturável para Advogados, não-faturável por padrão para Assistente) |
| Financeiro (módulo completo) | Sócio, Setor Administrativo |
| Relatórios | Todos veem "Minhas Pastas"; aba "Escritório" restrita a Sócio/Administrativo |
| VAULT — Biblioteca | Todos os Cargos (consulta) |
| VAULT — Gestão | Sócio, Sênior, Pleno, Administrativo |
| BrandBook, Políticas Internas | Todos os Cargos |
| Admin de Usuários | Sócio, Setor T.I (permissão equivalente, conforme seção 3.6) |
| Log de Auditoria, Cadastro de Órgãos | Sócio, Setor T.I |
| Compliance/LGPD | Sócio (demais acessos a definir caso a caso) |
| Configurações | Todos (escopo pessoal: perfil, senha, MFA) + Sócio/T.I (escopo do sistema) |

---

## 5. Portal do Cliente

Espelho seletivo e traduzido do Sistema Interno — nunca uma tela desconectada.

**Login:** possivelmente compartilhado com ponto de entrada do Sistema Interno (a decidir — ver seção Design System) ou PWA totalmente separado (decisão arquitetural já tomada: 2 PWAs).

**Módulos:**
1. **Dashboard** — pastas ativas, próximo prazo relevante (linguagem simples), pendências financeiras, mensagens não lidas, link ao Relatório mais recente.
2. **Minhas Pastas** — lista simplificada + timeline de andamentos traduzida (`descricaoParaCliente`, preenchida manualmente pelo advogado).
3. **Documentos** — somente itens com `visivelNoPortal = true`.
4. **Financeiro** — propostas pendentes de aceite, faturas, notas fiscais, histórico de pagamento (tom cuidadoso, nunca agressivo mesmo em inadimplência).
5. **Mensagens** — escopado por vínculo com Pasta/responsável.
6. **Relatórios** — histórico de Relatórios do Cliente, sempre dinâmico.
7. **Perfil** — dados cadastrais, gestão de usuários vinculados ao Cliente (múltiplos logins), área "Meus Dados" para exercício de direitos LGPD (conecta ao módulo de Compliance interno, SLA 15 dias úteis).
8. **Checklist de Documentos** — interativo, upload direto pelo cliente dos itens pendentes do onboarding.

---

## 6. Notificações

**Canais:** Push (PWA), E-mail, **WhatsApp Business API** (confirmado como canal formal).

| Evento | Interno | Portal Cliente |
|---|---|---|
| Alerta de prazo D-10/D-3/D-1 | Push+E-mail (responsável) | Se relevante ao cliente |
| Novo andamento relevante | Push (responsável) | Push/E-mail se `visivelNoPortal` |
| Nova mensagem | Push+E-mail (se não visto) | Push+E-mail |
| Documento aguardando revisão | Push (Sênior/Sócio) | — |
| Novo Relatório do Cliente publicado | — | Push+E-mail |
| Fatura próxima do vencimento | Financeiro interno | E-mail (+ push) |
| Proposta de honorários enviada | — | Push+E-mail (ação necessária) |

---

## 7. Pendências Registradas (não bloqueiam, retomar em fases seguintes)

1. Permissões exatas do Cargo Consultor (atualmente standby).
2. Hierarquia entre múltiplos usuários de um mesmo Cliente no Portal (todos veem tudo igual, ou existe usuário principal com mais acesso).
3. Mecanismo definitivo de aceite eletrônico de Proposta de Honorários (assinatura eletrônica vs. log simples).
4. Permissão granular por categoria dentro da aba Gestão do VAULT (ex: Marketing só gerencia Imagens/Apresentações).
5. Revisitar se Áreas de Atuação formais deveriam incluir Família/Sucessório e Imobiliário (sinais encontrados no VAULT e nas Calculadoras).
6. Módulo de Onboarding de Colaborador (RH) — mencionado, não desenhado.
7. Mapa visual de estrutura societária (organograma Pessoa↔Empresas) — considerado diferencial, não essencial ao MVP.
8. Design System completo (paleta, tipografia, tela de Login, componentes) — documento próprio, a iniciar após fechamento funcional.
9. Dimensionamento da fila de jobs assíncronos (BullMQ/Redis vs. Supabase Edge Functions + pg_cron) conforme volume real de integrações.

---

## 8. Referências de Fonte

- Políticas Internas do escritório (políticas de prazos, comunicação, honorários, LGPD, tecnologia).
- BrandBook / identidade visual.
- Prints do sistema atual: Dashboard (Sócio e Estagiário), Clientes (lista, detalhe, cadastro), Pastas (lista, cadastro, detalhe), Prazos, Arquivo, Relatórios, Financeiro, VAULT, M&DCALC, SCRIPTUM, Login.
- Planilha real de Onboarding de Novos Clientes (Painel de Status, Ficha Geral, Qualificação de Membros/Sócios, Documentos, Sociedades e Patrimônio).
