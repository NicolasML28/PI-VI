# Nicolas: telas, textos, testes e parceiros

> **Como usar:** abra o Claude Code dentro do repositório e mande:
> `Leia CLAUDE.md, plano/00-visao-geral.md e plano/03-nicolas.md. Faça a Fase 0.`
> Uma fase por vez. Ao terminar cada item, rode os testes e abra um PR com prints
> em 390px e 1280px.

## Quem você é neste projeto

Você é o Nicolas Marques Linares. Cuida de tudo o que a pessoa vê e de tudo o
que a faculdade vai ler: telas, textos, gráficos, testes de ponta a ponta,
parceiros e relatório. Se o sistema calcula certo mas ninguém entende a tela, o
problema é seu. É o problema que mais reprova trabalho de conclusão.

Ferramentas que a disciplina espera ver no seu trabalho: HTML, CSS, Jinja2,
responsivo, acessibilidade, Playwright, Figma, persona, benchmarking, ABNT e a
ficha de extensão universitária de 62 horas.

**Meta de linguagem:** qualquer pessoa entende qualquer tela em 5 segundos,
mesmo quem nunca leu um gráfico na vida. Se precisar explicar, a explicação está
a um clique, num botão.

## Referência: o site atual

Os prints de todas as telas, em computador e celular, estão em
`referencia/site-atual/`. Foram tirados do mesmo código que está no ar em
<https://radar-de-reclamacoes-pi6.onrender.com>. Problemas encontrados neles:

| Tela | Print | Problema |
|---|---|---|
| Todas | `*-celular.png` | **No celular, os gráficos ficam ilegíveis.** O SVG de 760px é espremido em 340px e o texto cai para uns 5px. |
| Painel | `03-painel-celular.png` | A tabela de 8 colunas é cortada no celular. Somem justamente "o que deve crescer" e "hoje → mês que vem". |
| Painel | `03-painel-computador.png` | "Estado", "Tendência" (a linhazinha não tem escala) e "Passos 7/7" não dizem nada para o dono. |
| Comércio | `04-comercio-computador.png` | Dois gráficos dizem quase a mesma coisa. A mesma cor tem dois sentidos: o azul-petróleo é "o maior" no primeiro gráfico e "caiu" no segundo. |
| Comércio | `04-comercio-computador.png` | "Certeza 0,74" é número técnico. O dono não sabe se 0,74 é bom. |
| Previsão | `05-previsao-computador.png` | O traço de mínimo e máximo é minúsculo. Um parágrafo de 5 linhas tenta explicar o gráfico. |
| Alertas | `06-alertas-computador.png` | O alerta não diz o que fazer em seguida. "Confirmar" e "Descartar" não explicam para que servem. |
| Acurácia | `07-acuracia-computador.png` | F1, macro e régua na cara do cliente. Os rótulos "0,79" e "0,76" encostam na linha tracejada. |
| Etapas | `08-etapas-computador.png` | Página enorme que mistura o que é do dono (o que falta) com o que é da banca (tecnologias, autoria). |

Para ver as telas ao vivo, rode o projeto local e use o **Playwright MCP**, que
já está instalado no seu Claude Code:
*"Use o Playwright MCP para abrir http://localhost:8000, entrar com a conta de
demonstração e tirar print de cada tela em 390px e 1280px."*
Gabriel e Matheus instalam com:

```bash
claude mcp add --scope user playwright -- npx -y @playwright/mcp@0.0.83
```

No Windows, o comando precisa de `cmd /c` antes do `npx`.

## Regras de linguagem

Troque sempre:

| Não escreva | Escreva |
|---|---|
| F1, acurácia, macro | "acerta 8 de cada 10" |
| P10–P90, intervalo | "entre 55 e 59" |
| régua, palpite simples | "o chute mais simples: repetir o mês passado" |
| confiança 0,74 | "certeza alta", "média" ou "baixa" |
| variação +4 | "▲ 4 a mais que em julho" |
| hipótese | "possível causa" |
| sustentada, descartada, inconclusiva | "Provável causa", "Não parece ser isso", "Ainda não dá para dizer" |
| experimento, grupo controle | "teste na prática", "dias sem a mudança" |
| aleatorização | "o sistema sorteia os dias para a comparação ser justa" |
| significância, p-valor | nunca na tela do dono |
| Estado: Alta prevista, Atenção, Estável | "Vai piorar", "Fique de olho", "Tudo normal" |

Mais regras:
- Frases de no máximo 20 palavras.
- Números inteiros. Meses por extenso.
- Termo técnico só dentro de `<details class="tecnico">` com o resumo "Detalhes
  técnicos". Esses blocos existem para a banca.
- Um teste de jargão percorre **todas** as telas do dono e falha se achar
  termo técnico fora desses blocos.
- Sugestão de menu em palavras do dono:
  - **Resumo**, **Avisos**, **Por quê?**, **E se?**, **Testes** e **Ajuda**;
  - e um "Mais" com **Dá para confiar?**, **O que falta** e **Sobre o sistema**.

## Regras dos gráficos (`app/graficos/svg.py` é seu)

1. **O título responde à pergunta.** Acima de cada gráfico vai uma frase que é a
   conclusão, com números. Exemplo: *"A demora é o que mais incomoda: 54
   reclamações em agosto, 4 a mais que em julho."* Monte a frase a partir dos
   dados do contrato, nunca à mão.
2. **Uma cor, um significado, no site inteiro:**
   - laranja (`#B4531F`): piorou ou precisa de atenção;
   - azul-petróleo (`#0F6E7B`): previsão e destaque neutro;
   - cinza: passado e normal.
   - Cor nunca vem sozinha: sempre com ▲ ▼ e a palavra, por causa de quem não
     enxerga cores.
3. **O rótulo fica em cima da coisa.** Escreva o valor em cada barra. Evite
   legenda separada.
4. **Incerteza escrita:** "entre 55 e 59" acima da coluna, com uma área clara.
   Nada de traço fino.
5. **Celular de verdade:**
   - gere uma versão estreita de cada gráfico (360px de largura, no máximo 6
     meses, barras horizontais quando couber);
   - alterne as versões com media query no CSS;
   - o texto nunca fica abaixo de 12px no tamanho final.
6. **Embaixo de todo gráfico, dois botões:** **Como ler este gráfico** e
   **Ver os números em tabela**. O segundo também serve à acessibilidade.

Como cada gráfico atual deve mudar:
- **Painel:** a tabela vira **um cartão por negócio**. Cada cartão tem:
  - um semáforo com palavra ("Vai piorar");
  - uma frase: *"A demora deve passar de 54 para cerca de 57 reclamações em
    outubro"*;
  - "o que mais incomoda";
  - três botões: **Por quê?**, **E se eu mudar algo?** e **Ver detalhes**.
  - A linhazinha vira 6 colunas pequenas com o primeiro e o último mês escritos,
    ou sai.
- **Comércio:** os dois gráficos viram **um**, com barras ordenadas e cada uma
  escrita "54 · ▲ 4 a mais que julho". "Certeza 0,74" vira ícone mais "alta".
  As estrelas viram ★★☆☆☆, com `aria-label`. As avaliações viram cartões no celular.
- **Previsão:** 6 meses fechados, depois "setembro até hoje" (com o rótulo "mês
  pela metade"), depois os 2 meses previstos, com "entre X e Y" escrito. O
  parágrafo vai para dentro do botão "Como ler".
- **Acurácia ("Dá para confiar?"):**
  - pictograma de 10 bonecos: *"O sistema acerta o assunto de 8 em cada 10
    avaliações. O chute mais simples acerta 1 em cada 10."*;
  - por assunto: "acerta 89 de cada 100";
  - previsão: *"Em 84 de cada 100 vezes, o número real ficou dentro do que o
    sistema disse."*;
  - a linha de previsto contra realizado vira colunas lado a lado por mês, com
    a diferença escrita;
  - F1 e companhia vão para "Detalhes técnicos".

## Botões de entendimento (em todo o site)

Crie macros em `_macros.html`. Os textos ficam no `GLOSSARIO` de `conteudo.py`,
uma fonte só, para a mesma explicação não ser escrita duas vezes.

| Botão | Onde | Como |
|---|---|---|
| **?** "O que é isso?" | Ao lado de todo número de destaque, título de coluna e termo novo | `<details class="ajuda">`, com o `<summary>` "?" e um `aria-label`. Duas frases e um exemplo. Funciona sem JavaScript e pelo teclado. |
| **Como ler este gráfico** | Embaixo de todo gráfico | Três passos numerados |
| **Ver os números em tabela** | Embaixo de todo gráfico | Tabela com os mesmos dados |
| **Próximo passo** | No fim de toda tela | "Por que isso está acontecendo?", "E se eu mudar algo?", "Testar na prática", "Voltar ao resumo" |
| **Primeira vez aqui? Faça o tour** | No Resumo | Tour de 5 passos em `static/js/tour.js`, permitido pela CSP. Lembra que a pessoa já viu, com `localStorage` dentro de `try/catch`. |
| **Me mostre um exemplo** | Investigação, simulador e teste | Mostra a história da Bella Massa do começo ao fim |
| **Página de Ajuda** (`/ajuda`) | No menu | Glossário inteiro e perguntas frequentes. Exemplos: "Por que a previsão não é um número exato?", "Por que os dias do teste são sorteados?" |
| **Estado vazio com ação** | Toda lista que pode estar vazia | "Ainda não há mudanças registradas." e o botão **Registrar a primeira** |

Lembre que a CSP do site não permite `<script>` inline, `onclick=` nem `style=`.
Use `<details>`, que é nativo, e JavaScript em arquivo.

## Fase 0: prompt de início e parceiros (semana 1)

**0.1 Base do site: feita.** Já estão prontos:
- o esqueleto, a CI e a configuração do Render;
- os contratos, os dados de mentira e `app/servicos.py`;
- o visual, os gráficos com versão de celular e os botões de ajuda;
- as telas iniciais;
- os testes com Playwright.

Falta só o git. A pasta do seu usuário no Windows (`C:\Users\Gamer`) já é um
repositório git, provavelmente criado sem querer. Rode `git init -b main`
**dentro de `PI6`** antes do primeiro commit, envie para
<https://github.com/NicolasML28/PI-VI> e adicione o Gabriel e o Matheus como colaboradores.

**0.2 Revisão do grupo.** Peça ao Gabriel que revise os contratos e os falsos
antes de começar o banco. Peça ao Matheus o usuário dele no GitHub, para o
`CODEOWNERS`.

**0.3 Render.** Crie um serviço novo no Render a partir do `render.yaml`, mais o
banco PostgreSQL, ou aponte o serviço `radar-de-reclamacoes-pi6` para o
repositório novo. Confira se o banco gratuito expira.

**0.4 Parceiros.** Marque as entrevistas com os donos dos negócios parceiros.
Comece a ficha de extensão: data, horas, atividade, contato e evidência de cada
encontro.

## Fase 1: entrevistas, catálogo, protótipo e redesenho (semanas 2–3)

**1.1 Entrevistas** (`docs/entrevistas/`): 30 a 40 minutos por parceiro, gravadas
só com autorização. Roteiro mínimo:
1. Como acompanha hoje o que os clientes escrevem?
2. Na última vez que as reclamações aumentaram, o que tinha acontecido? E antes
   disso? *Daqui sai o catálogo de causas.*
3. Que mudanças costuma fazer, como equipe, cardápio, aplicativos de entrega ou
   horário? Anotaria a data num diário?
4. Saberia dizer quantas pessoas trabalham em cada turno, o movimento médio e
   quanto custa uma hora de funcionário?
5. **Toparia testar uma mudança só em dias sorteados, durante 6 semanas?** Essa
   resposta decide se o experimento é viável com parceiro real.
6. Mostre o protótipo: "O que esta tela está dizendo?" Anote as palavras que a
   pessoa usa.

**1.2 Catálogo de causas** (`dados/catalogo_causas.yaml`). Entregue até o fim da
semana 2: o mundo simulado do Gabriel depende dele. Pelo menos 10 causas, cada
uma com a fonte:

```yaml
causas:
  - id: entregador-novo-lento
    descricao: Funcionário novo mais lento em alguns turnos
    tipo_evento: equipe            # como o dono registraria no diário
    registrada_no_diario: provavel # sempre | provavel | raramente
    afeta: demora
    mecanismo: tempo_atendimento   # tempo_atendimento | movimento | equipe | preco | produto | estrutura | clima
    intensidade: [1.2, 1.6]        # quanto multiplica o mecanismo
    onde: {dias_semana: [4, 5], periodo: noite}
    termos: [entregador, motoboy, chegou fria]
    fonte: "Entrevista com a Pizzaria X em 14/10/2026"
```

**1.3 Personas e benchmarking.**
- **Personas** (`docs/personas.md`): 2 ou 3, tiradas das entrevistas, não inventadas.
- **Benchmarking** (`docs/benchmarking.md`): compare com o que já existe no
  mercado.
  - Exemplos: Reclame Aqui, os relatórios do Perfil da Empresa no Google e
    ferramentas pagas de gestão de avaliações.
  - Mostre o que nenhuma delas faz: investigar a causa, simular a decisão e
    provar com teste.

**1.4 Protótipo no Figma:**
- Resumo em cartões;
- investigação;
- simulador;
- teste na prática;
- o botão "?" e o "Como ler".

Desenhe em 390px e 1280px. Mostre aos parceiros antes de programar e guarde o
link e os prints em `docs/prototipo/`.

**1.5 Redesenho das telas existentes:** siga as regras de linguagem, as regras
dos gráficos e os botões de entendimento descritos acima, nesta ordem: Resumo,
Comércio, Previsão, Avisos e "Dá para confiar?". Separe "O que falta" (do dono)
de "Sobre o sistema" (da banca).

## Fase 2: investigação e diário (semanas 3–5)

Comece com `app/falsos.py` e troque pelas funções reais quando o Gabriel entregar.

**2.1 Tela "Por quê?"** (`/investigacao/{empresa}/{categoria}`).
- Título: *"Possíveis causas da alta na demora"*.
- Um cartão por possível causa:
  - etiqueta ("Provável causa", "Ainda não dá para dizer" ou "Não parece ser isso");
  - a `frase`;
  - a força em 3 barrinhas, com a palavra;
  - um mini gráfico antes e depois, em duas colunas: *"antes de 3 de junho: 8
    por semana"* e *"depois: 13 por semana"*;
  - o botão **Ver avaliações de exemplo**.
- Bloco **"O que o sistema verificou"**: os passos da investigação em linguagem
  simples, dentro de um `<details>`.
- Aviso com "?": *"A inteligência artificial só sugere ideias. Quem decide é a
  conta."*
- Estado `dados_insuficientes`: explica o que falta e quando deve ficar pronto.
- Botão principal: **E se eu mudar algo?**, já preenchido com o
  `cenario_sugerido`.

**2.2 Tela "Diário do negócio"** (`/diario`): lista do mais recente para o mais
antigo e um formulário com data, tipo (lista com ícones) e descrição. Só dono e
gerente veem o formulário. Cada tipo tem um "?" com exemplos.

## Fase 3: simulador e teste na prática (semanas 5–7)

**3.1 Tela "Operação"** (`/operacao`).
- No computador, uma grade de 7 dias × 3 turnos. No celular, uma sanfona por dia.
- Campos: pessoas, clientes ou pedidos médios, minutos por atendimento e custo
  por hora.
- Cada campo tem "?" e a opção **Não sei**. Turno sem dado aparece no simulador
  como "faltam dados".

**3.2 Tela "E se?"** (`/simulador/{empresa}`).
- **À esquerda:** a mudança, com botões − e + de pessoas por turno. Os turnos da
  causa principal aparecem primeiro.
- **À direita:** o resultado em duas colunas grandes:
  - *"Hoje: 57 por mês"*;
  - *"Com a mudança: entre 38 e 45 por mês"*.
- **Embaixo:**
  - *"Espera média: de 22 para 9 minutos"*;
  - *"Custo: R$ 1.280 por mês"*;
  - *"R$ 80 por reclamação evitada"*.
- Se `confiavel` for falso, mostre um aviso amarelo com o `motivo`.
- Funciona com formulário comum, sem JavaScript. O JavaScript só melhora.
- Botão principal: **Testar na prática**.

**3.3 Teste na prática** (`/experimentos`, `/experimentos/{id}`).
- **Assistente em 3 passos:**
  1. O que mudar.
  2. Por quanto tempo. Mostre a frase *"Com 6 semanas dá para perceber uma
     queda de 25% ou mais"*.
  3. Confirmar. Gera o **código do plano** (o selo curto do Matheus).
- **Calendário:** dias com a mudança e dias sem, em cor e com texto.
- **Hoje:** cartão *"Hoje é dia COM o entregador extra"* e os botões
  **Fiz** / **Não deu para fazer**.
- **Barra de progresso.**
- **Resultado:**
  - veredito grande ("Funcionou", "Não funcionou" ou "Ainda não dá para saber");
  - duas colunas: média nos dias com a mudança e nos dias sem;
  - a frase do contrato.
- Botão "?" **Por que sorteamos os dias?**

**3.4 Tela do laboratório** (`/laboratorio`, área da banca). Mostra em
gráficos simples:
- o acerto do agente contra o chute simples;
- o agente com IA e sem IA;
- o acerto do simulador;
- os falsos "funcionou" dos testes A/A.

Os números vêm de `ResultadoLaboratorio`, combinado com o Gabriel.

## Fase 4: testes, usabilidade e relatório (semanas 7–8)

**4.1 Testes de ponta a ponta** (`tests/e2e/`, Playwright em Python).
- **A jornada inteira:** entrar → Resumo → Por quê? → E se? → Testar na prática
  → marcar o dia. Roda em 390px e em 1280px.
- Print de cada passo em `docs/evidencias/telas/`.
- Nenhuma tela tem rolagem horizontal: `scrollWidth <= innerWidth`.
- Todo botão "?" e "Como ler" abre e fecha.
- A jornada inteira funciona **só com o teclado**.
- O axe não acha nenhuma violação séria ou crítica.
- Todo formulário funciona de verdade: clicar em Confirmar num aviso muda o estado.
- Os testes rodam na CI.

**4.2 Teste de usabilidade** (`docs/usabilidade/`).
- **Quem:** 5 pessoas sem formação técnica, incluindo 1 ou 2 donos parceiros.
- **Tarefas:**
  1. Qual problema está piorando?
  2. Por quê?
  3. Quanto custaria resolver?
  4. Comece um teste.
  5. O teste funcionou?
- **O que medir:**
  - sucesso, tempo e erros em cada tarefa;
  - questionário SUS, com nota de 0 a 100 e meta ≥ 70;
  - **teste dos 5 segundos** em cada gráfico ("o que esse gráfico diz?"), com
    meta de 4 em 5 acertos.
- **Rode duas vezes:** uma com o site da versão anterior, que continua no ar,
  e outra com o novo. A comparação
  prova que o redesenho melhorou. É a ideia de comparar com uma régua, aplicada
  às telas.

**4.3 Relatório ABNT** (`docs/relatorio/`).
- **Capítulos:**
  - introdução e problema;
  - objetivos;
  - fundamentação: classificação de texto, séries temporais, inferência causal,
    teoria de filas, experimentos controlados, segurança e LGPD;
  - trabalhos relacionados (o benchmarking);
  - metodologia;
  - requisitos;
  - arquitetura;
  - implementação;
  - avaliação: laboratório do Gabriel, segurança do Matheus e usabilidade;
  - extensão;
  - conclusão;
  - limitações.
- **Material dos outros:** os capítulos técnicos vêm de `docs/modelos.md`
  (Gabriel) e de `docs/seguranca/` (Matheus). Você junta, padroniza e revisa.

**4.4 Extensão e banca.**
- Feche a ficha de 62 horas com as evidências.
- Monte o roteiro da apresentação seguindo a história da Bella Massa: aviso →
  por quê → e se → teste → resultado → prova no laboratório.

## Se der tempo

- Modo escuro.
- Relatório mensal em PDF para o dono imprimir.
- Avisos por e-mail, com o texto escrito por você e o envio feito pelo Gabriel.

## Não faça

- Não calcule número no template. Todo número vem do contrato.
- Não coloque termo técnico fora de `<details class="tecnico">`.
- Não use `<script>` inline, `onclick=` ou `style=`, porque a CSP bloqueia.
- Não mexa em estatística, banco ou segurança: peça ao Gabriel ou ao Matheus.
- Não grave entrevista nem copie avaliação de parceiro sem autorização escrita.
