# Design System — Sistema de Gestão Jurídica 2.0
## Menin & Deroma Advogados

**Versão:** 1.0 — Rascunho para validação
**Natureza deste documento:** sistematização e evolução da identidade visual já existente (BrandBook institucional + sistema atual), não uma criação do zero. Onde o sistema atual já define um padrão aprovado, esse padrão é herdado; onde há lacuna (estados vazios, componentes ausentes, telas novas como o Portal do Cliente), este documento propõe extensões consistentes.

---

## 1. Princípios

1. **Sóbrio, não genérico.** O escritório é boutique, técnico, de alto padrão — o sistema deve parecer feito sob medida para advocacia societária/cível de elite, não um SaaS jurídico de prateleira.
2. **Densidade com respiro.** Advogados trabalham com muito dado (pastas, prazos, valores) — a interface precisa caber informação sem parecer apertada. Cards e tabelas com espaçamento generoso, nunca coladas.
3. **Hierarquia clara entre Interno e Portal.** Mesma linguagem visual (é a mesma marca), mas o Portal é mais acolhedor/simples (cliente leigo em Direito) e o Interno é mais denso/operacional (equipe treinada).
4. **Consistência antes de originalidade.** Um componente (card de métrica, tabela, badge de status) se comporta igual em todo lugar onde aparece. Isso é requisito direto do "erro-free" do PRD — inconsistência visual gera erro de leitura de dado.

---

## 2. Tokens de Cor

**Paleta oficial (obrigatória — fornecida pelo cliente, não estimada):**

| Token | Hex | Nome | Uso |
|---|---|---|---|
| `--navy` | `#0D1F3C` | Azul-marinho (Principal) | Fundo da Sidebar, header, elementos de marca primários, texto de alto contraste sobre fundo claro |
| `--gold` | `#A8832A` | Ouro (Acento) | **Uso obrigatório deste HEX exato sempre que "dourado/ouro" for aplicado no sistema.** Reservado para acentos, ícones, bordas, linha divisória, labels curtas, links — **nunca para texto de corpo/parágrafo** (ver nota de contraste abaixo) |
| `--cream` | `#F7F3EE` | Creme (Fundo) | Fundo principal das telas (área de conteúdo), metade da tela de Login |
| `--gray-medium` | `#C8C2B8` | Cinza médio (Neutro) | Bordas, divisores, fundos neutros — **não usar como cor de texto** (contraste insuficiente) |
| `--gray-dark` | `#4A4540` | Cinza escuro (Texto) | Texto de corpo secundário sobre fundo claro |
| `--white` | `#FFFFFF` | — | Fundo de cards quando destacados do fundo creme |
| `--success` | a definir (verde discreto, coerente com a paleta) | — | Status "OK", "Ativa", "Paga" |
| `--warning` | a definir (tom âmbar, distinto do `--gold` para não confundir estado com marca) | — | Status "Pendente", "Em Andamento" |
| `--danger` | a definir (terracota/vermelho discreto) | — | Status "Vencida", "Perdido" |

### 2.1 Nota crítica de contraste (acessibilidade)
- `--gold` (`#A8832A`) sobre `--cream`/`--white` tem contraste ~2.8:1 — **abaixo do mínimo WCAG AA (4.5:1) para texto corrido**. Uso permitido apenas em: texto caixa-alta curto e espaçado (labels, eyebrows), ícones, bordas, linha divisória, botões/links onde o peso visual compensa. **Proibido** em parágrafos, valores de dado, texto de corpo.
- `--gray-medium` (`#C8C2B8`) nunca como cor de texto — só fundo/borda.
- `--navy` e `--gray-dark` como texto sobre `--cream`/`--white`: contraste excelente, uso livre.

### 2.2 Gradiente-assinatura
- Linha divisória de seções de menu: gradiente horizontal `navy → gold (#A8832A) → navy`, opacidade baixa, 1px de altura. Elemento de assinatura visual do produto — uso exclusivo para dividir grupos estruturais (menu lateral).

---

## 3. Tipografia

| Papel | Família | Uso |
|---|---|---|
| Display/Título | **Georgia** (fonte oficial do BrandBook) — testar em contexto; se não performar bem digitalmente (renderização, peso, presença), substituir por **Playfair Display** como fallback aprovado | Títulos de página (H1), nome de cards de destaque (ex: número da Pasta "MD-2026/001") |
| Corpo | Sans-serif neutra e legível (ex: Inter, Söhne, ou similar — confirmar se BrandBook define uma) | Textos de corpo, labels de formulário, navegação |
| Dados/Números | Mesma sans-serif do corpo, com tabular-nums habilitado | Valores monetários, datas, contadores — alinhamento vertical correto em tabelas |

> 📌 **Ação prévia à implementação:** testar Georgia nos tamanhos reais de H1/H2 do sistema (32px/24px) antes de decidir — Georgia foi desenhada para leitura em tela em corpo de texto pequeno, pode não ter a mesma presença/elegância de uma display face em títulos grandes. Se o teste não convencer, migrar para Playfair Display sem revisitar esta decisão.

### 3.1 Escala tipográfica (proposta, a validar)

| Nível | Tamanho | Peso | Uso |
|---|---|---|---|
| H1 | 32px | Regular (serifada) | Título de página ("Dashboard", "Clientes") |
| H2 | 24px | Regular (serifada) | Título de seção/card grande |
| H3 | 16px | Semibold (sans) | Subtítulo de card, label de aba ativa |
| Body | 14px | Regular (sans) | Texto padrão |
| Small/Caption | 12px | Regular/Medium, uppercase, letter-spacing | Labels de campo ("CLIENTE", "STATUS"), eyebrows |

---

## 4. Espaçamento e Grid

- Base de espaçamento: múltiplos de 4px (4, 8, 12, 16, 24, 32, 48).
- Cards com padding interno generoso (~24px), nunca abaixo de 16px.
- **Grid como padrão de visualização** (decisão já fechada no PRD) para: Arquivo, VAULT, Documentos, Calculadoras. Grid responsivo, 3-4 colunas em desktop, 1-2 em mobile.
- Tabelas/listas (dado primariamente tabular — comparação, ordenação) para: Faturas, Prazos, Ficha-Tempo, Log de Auditoria.

---

## 5. Componentes-Chave

### 5.1 Sidebar/NavBar
- Fundo `--navy`, item ativo com destaque em `--gold` (texto e/ou barra lateral).
- **Logo oficial centralizada no topo da Sidebar** (arquivo fornecido, versão sem fundo/transparente), com a **linha divisória gradiente logo abaixo dela** — mesma logo usada na tela de Login.
- Agrupamento em seções com header discreto (uppercase, cinza claro): PRINCIPAL, INSTITUCIONAL, FERRAMENTAS, SISTEMA.
- Linha divisória entre grupos: gradiente `navy → gold → navy` (seção 2.2).
- **Colapsável no desktop** (ícone/seta no topo) — estado colapsado mostra só ícones, comportamento já validado no sistema atual.
- Rodapé fixo: avatar + nome + Cargo do usuário logado, com ação de logout.
- Mobile: menu hambúrguer/drawer (modo exato a definir em protótipo).

### 5.2 Card de Métrica (padrão já em uso — Dashboard, Financeiro, Relatórios)
Estrutura: label uppercase pequeno (canto superior esquerdo) + ícone com fundo suave (canto superior direito) + valor grande (serifado ou sans bold) + linha de variação/contexto pequena abaixo (ex: "↗ 2 no total").
- **Bordas moderadamente arredondadas** (raio médio, ~12px — nem quadrado/corporativo demais, nem "app consumer" com raio muito alto).
- Estados de valor zerado devem ter tratamento (não é erro, é "ainda sem dados") — ver seção 6.

### 5.3 Card de Grid (VAULT, Arquivo, Calculadoras)
- Mesmo padrão de raio moderado (~12px) para consistência com Card de Métrica.
- Ícone/thumbnail no topo, título, metadado secundário (autor, data, tamanho ou contagem de itens), badge de tipo de arquivo no canto quando aplicável (PDF/DOCX/PNG com cor própria por tipo).

### 5.4 Badge de Status
Pill arredondado, cor de fundo suave + texto na cor correspondente (não preenchimento sólido saturado, mantendo sobriedade):
- Ativa/OK/Paga → `success`
- Pendente/Em Andamento → `warning`
- Vencida/Perdido/Inativo → `danger`
- Suspensa/Neutro → cinza

### 5.5 Tabela/Lista
- Header com labels uppercase pequenas, cinza.
- Linhas com hover sutil, densidade confortável (não comprimida).
- Ações secundárias em menu "..." (kebab), ação primária como botão visível.

### 5.6 Modal/Dialog
- Fundo `cream-100` ou branco, cabeçalho com título serifado + botão fechar (X), largura fixa confortável (não full-screen em desktop).
- Formulários com labels uppercase pequenas acima do campo (padrão já observado no cadastro de Cliente/Pasta).
- Campos condicionais (ex: Tribunal vs. Órgão conforme Tipo de Pasta) devem ter transição suave, não "pulo" abrupto de layout.

### 5.7 Botão
- Primário: fundo `navy-900`, texto branco.
- Secundário: borda `navy-900` ou cinza, fundo transparente/branco.
- Destrutivo: usar `danger` com moderação — ações como excluir usuário, cancelar fatura.

### 5.8 Widget de Ficha-Tempo (cronômetro)
- Alternância clara entre modo Cronômetro e Manual (toggle/tabs).
- Display do timer grande, monoespaçado, alto contraste — precisa ser lido rapidamente "de relance" enquanto se trabalha em outra coisa.

---

## 6. Estados Vazios, de Erro e de Carregamento

Lacuna identificada: o sistema atual mostra "0" e "Nenhum X encontrado" de forma bem neutra — vale evoluir isso com leve orientação de próxima ação, sem exagerar (tom institucional, não descontraído):
- Vazio: texto neutro + ação sugerida quando fizer sentido (ex: "Nenhum prazo crítico" está ok como está; "Nenhum comunicado" poderia ganhar um botão discreto "Criar comunicado" para quem tem permissão).
- Erro: mensagem direta do que aconteceu e o que fazer — nunca genérica tipo "Algo deu errado" sem contexto (alinhado à diretriz de escrita: "erros não se desculpam, não são vagos").
- Carregamento: skeleton screens (não spinners genéricos) para listas/cards, respeitando o layout final já conhecido.

---

## 7. Tela de Login (Interno e Portal)

**Layout:** split-screen, sem alteração estrutural do padrão atual:
- **Metade esquerda:** fundo `--navy`, com a **logo oficial centralizada** (mesma logo da Sidebar), e o **ícone/monograma do escritório em marca d'água** posicionado no fundo (arquivo já entregue com opacidade embutida — não aplicar opacidade adicional via CSS, usar a imagem tal como fornecida).
- **Metade direita:** fundo `--cream` (não branco puro — ajuste em relação ao sistema atual, que usava branco nessa metade).

**Formulário:** título serifado, campos E-mail/Senha, "Esqueci minha senha", botão primário `--navy`.

**Saudação personalizada e dinâmica** (novo, não existia no sistema atual):
- Primeiro acesso do usuário: **"Bem-vindo, [Nome do Usuário]"**
- Acessos seguintes: **"Bem-vindo de volta, [Nome do Usuário]"**
- Implica: o sistema precisa reconhecer o e-mail digitado (ou já ter sessão anterior) para renderizar a saudação correta **antes** da autenticação completa — ou exibir a saudação genérica ("Acesso ao Sistema"/"Bem-vindo(a)") até o e-mail ser preenchido, e só personalizar com nome após identificar o usuário (ex: on blur do campo e-mail, uma chamada leve verifica se é primeiro acesso). Este comportamento exato (quando a saudação personaliza) é uma decisão de fluxo a validar no protótipo — abordagens possíveis: (a) personalizar após digitar e-mail e sair do campo, (b) personalizar somente na tela seguinte, pós-login bem-sucedido (ex: um passo intermediário "Bem-vindo(a), Pedro" antes de cair no Dashboard). **Recomendo a opção (b)** — mais simples de implementar, sem exigir consulta ao backend a cada tecla, e sem expor se um e-mail existe ou não no sistema antes da autenticação (boa prática de segurança, evita enumeração de usuários).

**MFA:** etapa própria pós-senha (tela/passo separado), não campo adicional na mesma tela.

**Diferenciação Interno × Portal:**
- Mesma estrutura visual (navy+creme, logo, ícone d'água) nos dois.
- Copy do Interno: mais operacional ("Acesso ao Sistema").
- Copy do Portal: mais acolhedor, adequado a cliente leigo em Direito — a saudação "Bem-vindo(a) de volta, [Nome]" funciona ainda melhor aqui, é o padrão a manter em ambos.
- **Sem link cruzado entre os dois logins** (removido) — cada domínio/subdomínio tem seu Login próprio e independente, conforme decisão de arquitetura (dois PWAs, dois subdomínios).

**Assets já recebidos e aprovados para uso:**
- Logo oficial (versão transparente, para Sidebar e Login).
- Ícone/monograma do escritório com opacidade já aplicada (uso exclusivo no fundo navy do Login).

---

## 8. Ícones

- Sistema atual usa estilo de ícone outline, peso leve, consistente com a sobriedade da marca (não ícones preenchidos/coloridos chamativos).
- Recomendação: biblioteca de ícones outline consistente (ex: Lucide, Phosphor) para garantir uniformidade em todo o sistema — evitar misturar bibliotecas.

---

## 9. Responsividade

- Desktop: layout com Sidebar fixa (conforme visto).
- Mobile: Sidebar vira drawer/hambúrguer; cards de métrica empilham em coluna única; tabelas com scroll horizontal ou colapsam para formato de card por linha.
- PWA instalável — ícone de app e splash screen devem seguir a paleta navy/dourado.

---

## 10. Itens Pendentes de Validação

1. Testar Georgia nos tamanhos reais de H1/H2 (32px/24px) — se não convencer, migrar direto para Playfair Display (fallback já pré-aprovado).
2. Confirmar se o BrandBook define uma família sans-serif oficial para corpo/dados, ou se cabe a mim propor uma.
3. Definir hex exatos de `--success`, `--warning`, `--danger` — coerentes com a paleta navy/creme/ouro, evitando saturação que destoe da sobriedade da marca.
4. Comportamento exato da Sidebar em mobile (drawer completo vs. bottom nav híbrido).
5. Fluxo exato de personalização da saudação de Login (recomendação registrada na seção 7 — validar se a opção (b) é aceita).
6. Paleta de cores por tipo de arquivo no VAULT/Arquivo (PDF, DOCX, PPTX, PNG).
