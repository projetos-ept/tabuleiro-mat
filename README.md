# 🎲 Jornada Matemática - Gerador de Tabuleiros PRO

## 📌 Visão Geral
O **Jornada Matemática** é uma aplicação web *single-file* (um único arquivo HTML contendo todo o CSS e JavaScript) desenhada para professores, educadores e pais. A ferramenta permite a geração procedural de tabuleiros de jogo de percurso (estilo "cobras e escadas") focados em desafios educativos. 

O sistema gera uma pista contínua e sinuosa em formato SVG vetorial, otimizada para impressão em papel A4 (paisagem), com estética *line-art* (estilo livro de colorir). O projeto não requer instalação, não usa dependências externas complexas (apenas Google Fonts) e roda inteiramente no navegador local.

---

## ✨ Principais Recursos

1. **Motor Geométrico de Pista:** Em vez de dispor círculos soltos, a aplicação calcula vetorialmente uma pista contínua em zigue-zague (curvas senoidais ligadas por semicírculos) e subdivide a malha para criar "casas" perfeitamente encaixadas.
2. **Design para Impressão (Print-Ready):** CSS meticulosamente ajustado com `@media print` para garantir que o tabuleiro preencha perfeitamente uma folha A4 e gere páginas adicionais contendo as Cartas de Desafio (8 por página) e o Gabarito de respostas.
3. **Distribuição Inteligente:** Distribuição automática de eventos (Avança, Volta, Troca e Pergunta) baseada em frequências, garantindo que não haja sobreposição e poupando o trabalho manual do usuário.
4. **Gerenciador de Baralho (Cards Modal):** Sistema completo de criação, edição, exclusão e importação via arquivo `.json` de perguntas para alimentar o jogo.
5. **Decoração Processual Automática:** O gerador injeta ilustrações poligonais suaves (árvores, grama, pedras) nos espaços vazios do tabuleiro sem colidir com a pista ou com os textos informativos.
6. **Portabilidade:** Capacidade de baixar o projeto atual (`.json`) e carregá-lo novamente no futuro para continuar editando.

---

## 🛠️ Guia do Usuário (Como usar)

A interface é dividida em um **Painel Lateral** (controles) e uma **Área de Pré-visualização** (resultado em tempo real).

### 1. Geometria e Textos
- **Título e Subtítulo:** Textos que aparecerão em destaque no topo direito do tabuleiro.
- **Total de Casas:** Quantidade de etapas da pista (entre 20 e 80).
- **Largura da Pista & Fileiras:** Controlam a espessura do caminho e quantas "voltas" o zigue-zague fará na folha.
- **Modo Colorido:** Uma chave condicional que pinta o fundo de azul pastel e colore as casas com eventos. Ideal para jogar digitalmente ou enviar para gráficas. Deixe desmarcado para imprimir P&B para as crianças colorirem.

### 2. Distribuição Automática
Define a frequência dos eventos. Exemplo: se "Pergunta" estiver `5`, o sistema colocará o ícone de pergunta a cada 5 casas (Casa 5, 10, 15...).
- Clique em **"🚀 Gerar Regras"** para aplicar. 
- *Nota:* O sistema protege a primeira e a última casa para evitar conflitos de lógica de jogo.

### 3. Edição Manual
Caso o usuário queira refinar a pista manualmente:
1. Clique diretamente em uma casa na pré-visualização. Ela ficará contornada de azul.
2. No painel, clique no evento que deseja atribuir (ou "Vazia" para limpar).

### 4. Gerenciamento de Cartas
Ao clicar em **"📝 Gerenciar Cartas de Pergunta"**, uma janela (Modal) se abre. O sistema garante que existam pelo menos tantas cartas quanto o número de casas de pergunta no tabuleiro.
- **Importar JSON:** Você pode criar seu baralho em bloco de notas ou ChatGPT e importar direto. O formato exigido é:
```json
[
  {
    "pergunta": "Qual o valor de 5 x 8?",
    "resposta": "40",
    "regra": "Acertou: avança 1 | Errou: volta 1"
  }
]
```

### 5. Impressão
O painel oferece botões finais de ação:
- **Refazer Fundo:** Sorteia novamente as árvores e pedras no fundo.
- **Toggles Pré-Impressão:** Escolha se a resposta será impressa de ponta-cabeça na própria carta (para fácil conferência do mediador) e se o sistema deve gerar uma folha A4 final com um gabarito de todas as respostas.
- **Imprimir Tudo:** Abre o diálogo de impressão nativo do sistema operacional. As cartas e o tabuleiro já saem recortados no tamanho exato.

---

## 💻 Arquitetura Técnica (Para Desenvolvedores)

A aplicação foi construída em paradigma funcional clássico em JavaScript (sem frameworks como React ou Vue), operando num padrão de estado simples: `let estado = { casas: [], decoracoes: [], cartas: [] }`. Toda vez que o estado é alterado, a função `atualizarTabuleiro()` re-renderiza o DOM do SVG.

### O Motor Matemático (`calcularPontosDaPista()`)
A mágica da pista sem falhas está nesta função:
1. **Ondulação:** O algoritmo traça "fileiras" imaginárias na folha. Ele usa a função `Math.sin()` para gerar as curvas do caminho. Uma dupla senóide atua como envelope para garantir que o caminho comece e termine perfeitamente horizontal nas bordas da página.
2. **Arcos:** Nas extremidades laterais, semicírculos paramétricos são desenhados unindo a fileira de cima com a de baixo.
3. **Cálculo da Normal:** Após extrair centenas de pontos da linha central contínua, o algoritmo itera sobre eles calculando o **vetor normal** perpendicular (`nx, ny`) em cada ponto.
4. **Rasterização Vetorial:** Para cada "Casa" (segmento da pista), o sistema usa os vetores normais para dar o espaçamento (largura da pista), criando um polígono complexo através do `<path>` SVG (`d="M ... L ... Z"`). 

### Camada de Renderização (SVG)
- Utiliza a tag `<defs>` para declarar *symbols* (os ícones dos dados, árvore, perguntas, etc).
- Utiliza `<use>` instanciando e movendo os *symbols*, garantindo que a árvore do DOM permaneça leve e rápida, mesmo com dezenas de casas e decorações.
- As decorações possuem um sistema anti-colisão primitivo, mas eficaz, que checa o raio de distância em relação ao centro da pista calculada e às caixas delimitadoras (`bounding boxes`) do Título e da Legenda.

### Manipulação de CSS e Mídia de Impressão
O trunfo da folha de estilos é a secção `@media print`:
- Elementos como a `.sidebar` e o `#modal-cartas` recebem `display: none !important`.
- As margens da folha (`@page { margin: 0; }`) são limpas para que a impressora obedeça estritamente os milímetros passados via CSS (`width: 297mm; height: 210mm;`).
- Os containers das cartas são estruturados em um CSS Grid estrito (`grid-template-columns: repeat(4, 1fr); grid-template-rows: repeat(2, 1fr)`), forçando exatas 8 cartas por página.

---

## 🚀 Melhorias Futuras Possíveis (Roadmap)
Caso queira expandir este código no futuro, eis algumas funcionalidades interessantes a se implementar:

1. **Upload de Imagens nas Cartas:** Permitir que, via modal, o usuário adicione imagens (Base64) nas cartas de pergunta (útil para geometria).
2. **Temas Personalizados:** Trocar as decorações poligonais atuais (árvores/pedras) por pacotes temáticos (Tema Espaço, Tema Fundo do Mar, Tema Halloween).
3. **Pinos de Jogador (Papelcraft):** Adicionar uma rotina de impressão final que gere moldes de recorte (tokens de papel com abas para colar) de personagens, e um dado de papel montável, para que a pessoa tenha um jogo de tabuleiro 100% feito de papel em casa.
