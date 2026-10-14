# Gabriel: o motor

> **Como usar:** abra o Claude Code dentro do repositório e mande:
> `Leia CLAUDE.md, plano/00-visao-geral.md e plano/01-gabriel.md. Faça a Fase 0.`
> A base do site já está pronta: pode começar.
> Uma fase por vez. Ao terminar cada item, rode os testes e abra um PR.

## Quem você é neste projeto

Você é o Gabriel Lopes Londe Rodrigues. Cuida do motor: o programa em Python, o
banco de dados e tudo o que calcula um número. Se a tela mostra um valor errado,
o problema é seu. Se mostra o valor certo de um jeito feio, é do Nicolas.

Ferramentas que a disciplina espera ver no seu trabalho: Python, FastAPI,
Jinja2, PostgreSQL, SQLAlchemy, Alembic, pandas, scikit-learn, TF-IDF,
validação com origem móvel, erro médio absoluto e faixa P10–P90. Para as
funcionalidades novas, você também vai usar statsmodels, SciPy, SimPy,
`holidays` e a API Open-Meteo.

## Fase 0: banco de dados (semana 1)

A base do site já existe: `app/contratos.py`, `app/falsos.py` e
`app/servicos.py`, com as telas funcionando em cima dos dados de mentira. O
banco ainda não existe. Ele é a sua Fase 0, e a entrada do Matheus depende da
tabela `usuario`.

**0.1 Crie o banco** em `app/banco/`, com SQLAlchemy 2.0 e Alembic:
- `DATABASE_URL` funciona com SQLite local e com PostgreSQL na CI (acrescente o
  serviço `postgres` no `.github/workflows/ci.yml`);
- `alembic upgrade head` cria o banco do zero;
- existem as tabelas `empresa`, `usuario`, `avaliacao`, `previsao`, `alerta`,
  `evento_diario`, `investigacao`, `hipotese`, `turno`, `simulacao`,
  `experimento`, `dia_experimento`, `resultado_experimento`, `auditoria` e
  `gabarito`;
- **toda tabela tem `empresa_id`**, menos `empresa`;
- `avaliacao` tem `id_externo` com chave única junto com `empresa_id`, o texto já
  mascarado e o campo `removida`;
- nenhuma rota lê a tabela `gabarito`.

**0.2 Revise os contratos e os falsos.** `app/contratos.py` tem que estar igual
à seção 6 da visão geral. `app/falsos.py` tem que devolver esses tipos com
números plausíveis, porque o Nicolas desenha as telas em cima deles.

**0.3 Revise as dependências.** As versões precisam estar fixadas, e as que você
vai usar nas próximas fases precisam estar lá: pandas, numpy, scipy,
statsmodels, scikit-learn, joblib, simpy, holidays e pyyaml.

**Pronto quando:** o banco sobe do zero com `alembic upgrade head`, a CI roda com
PostgreSQL e um teste confirma que toda tabela, menos `empresa`, tem `empresa_id`.

## Fase 1: dados reais e mundo simulado (semanas 2–3)

**1.1 Mundo simulado diário** (`app/mundo/`), descrito na seção 4 da visão geral.

*Como gerar os negócios:*
- Cada negócio tem turnos: 7 dias × manhã/tarde/noite, cada um com equipe,
  movimento (Poisson com sazonalidade semanal) e tempo de atendimento.
- A fila gera a espera.
- A espera longa gera reclamação de demora, com probabilidade logística.
- Os outros assuntos têm mecanismo próprio:
  - atendimento depende de clientes por atendente;
  - preço depende de eventos de preço;
  - qualidade depende de eventos de produto ou fornecedor;
  - ambiente depende de lotação e de estrutura.

*Como gerar as avaliações:*
- A probabilidade de o cliente avaliar fica entre 3% e 8%.
- O cliente avalia de 0 a 3 dias depois da visita.
- O texto sai de frases por assunto, com termos ligados à causa ("entregador", "motoboy").
- Inclua elogios, avaliações só com estrela e frases vagas demais, como a versão anterior
  já fazia.

*Causas, iscas e gabarito:*
- As causas vêm de `dados/catalogo_causas.yaml`, com data de início, assunto e intensidade.
- Até o Nicolas entregar o catálogo, use um provisório com 6 causas.
- Inclua iscas (evento registrado que não causa nada) e cenários sem causa.
- O gabarito fica separado, na tabela `gabarito`.

*Os 3 negócios de demonstração ganham uma história:*
- Bella Massa: entregador novo nas sextas e sábados à noite, a partir de junho.
- Sorriso Claro: recepcionista nova.
- Patinhas: aumento de preço.

*Semear o banco:*

```bash
python -m app.mundo.semear --negocios 3 --semente 11
```

**Pronto quando:**
- Mesma semente gera os mesmos dados (teste).
- Os totais mensais ficam na mesma ordem de grandeza da versão anterior (30 a 70
  reclamações por assunto e por mês).
- Nenhum contrato expõe o gabarito.

**1.2 Classificador real** (`app/modelos/classificador.py`).

*Amostra rotulada:* cerca de 800 avaliações.
- Reviews do dataset público Olist (Kaggle, licença CC BY-NC-SA 4.0, uso acadêmico).
- Avaliações dos parceiros copiadas com autorização e sem o nome do autor. O
  Nicolas consegue a autorização.

*Rotulagem:*
- Escreva o guia de rotulagem em `docs/guia-rotulagem.md`.
- Você e o Nicolas rotulam 100 avaliações em comum e calculam o kappa de Cohen.
  A meta é ≥ 0,6.

*Modelo:*
- `TfidfVectorizer` com unigramas e bigramas, `sublinear_tf` e stopwords em português.
- `LogisticRegression(class_weight="balanced")` em `OneVsRestClassifier`, porque
  uma avaliação pode ter dois assuntos.
- Limiar de confiança ajustado na validação. Abaixo dele, a avaliação fica sem assunto.

*Validação e métricas:*
- Validação cruzada estratificada com 5 partes.
- F1 por assunto e macro.
- Régua: responder sempre o assunto mais comum.
- Um teste quebra a build se o macro do modelo não superar o da régua.

*Persistência:* salve o modelo com `joblib`, junto com a versão e o hash dos
dados de treino. Grave as métricas no banco para a tela de acurácia.

**Nunca meça o classificador no texto do mundo simulado.** Ele é montado a
partir de frases prontas, e a nota sairia inflada.

**1.3 Previsão** (`app/modelos/previsao.py`).
- Escreva do zero, seguindo a ideia da versão anterior, que ganhava da régua:
  - o ponto de partida é o último mês fechado;
  - a tendência dos últimos 6 meses só entra quando é maior que 1,8 vez o próprio
    erro padrão;
  - a faixa P10–P90 carrega o erro do ajuste e abre com o horizonte.
- Lê as contagens mensais do banco e grava as previsões.
- Validação com origem móvel: em cada mês, o modelo só enxerga o passado.
- Testes: erro médio absoluto menor que o da régua (repetir o mês anterior),
  cobertura da faixa entre 70% e 90% e P10 ≤ P50 ≤ P90.
- Não use `hash()` de string como semente, porque ele muda a cada execução do
  Python.

**1.4 Alertas** saem do banco.
- Há alerta quando a previsão passa o último mês fechado em pelo menos 1,5
  reclamação e 4%.
- O alerta vira "Vai piorar" quando nem o mínimo da faixa (P10) volta ao nível atual.

**1.5 Importação por planilha** (`app/coleta/planilha.py`).
- Colunas: `id_externo`, `data`, `estrelas` e `texto`.
- Remove duplicadas por `(empresa_id, id_externo)`.
- Marca como removida a avaliação que sumiu da origem.
- Chama `protecao.mascarar()` do Matheus **antes** de gravar o texto.

## Fase 2: agente investigador (semanas 3–5)

**2.1 Ferramentas estatísticas** (`app/investigacao/ferramentas.py`).

Funções puras: recebem séries diárias (pandas) e devolvem
`ResultadoTeste(efeito, minimo, maximo, p_valor, n)`.

| Função | O que testa | Como |
|---|---|---|
| `antes_depois(serie, data_evento, dias_semana=None, periodo=None)` | Mudou depois do evento? Opcionalmente só em certos dias ou turnos | GLM de Poisson com tendência e degrau |
| `concentracao(serie_alta, serie_base, por)` | A alta se concentra num dia ou turno? | Qui-quadrado mais o resíduo por célula |
| `clima(serie, clima)` | Chuva ou calor mexem na reclamação? | GLM de Poisson com controle de tendência |
| `feriado(serie, feriados)` | Feriado mexe? | GLM de Poisson |
| `termos_novos(textos_alta, textos_base)` | Que palavras passaram a aparecer? | Log-odds com prior informativo de Dirichlet |

Testes de cada ferramenta:
- numa série sintética **com** efeito conhecido, a ferramenta detecta;
- numa série **sem** efeito, a ferramenta erra em no máximo cerca de 5% das
  repetições (teste com 200 sementes).

**2.2 Motor** (`app/investigacao/motor.py`), com a função `investigar(...)` do contrato.

1. **Períodos:** a alta são as últimas 6 semanas e a base são as 8 semanas
   anteriores. Os dois valores são configuráveis.
2. **Candidatas:**
   - eventos do diário dentro da janela;
   - feriados e clima;
   - concentração por dia e turno;
   - termos novos;
   - propostas da IA, chamando `ia.propor_hipoteses()` do Matheus, só se
     `usar_ia=True` e houver chave da API.
3. **Hipóteses compostas:** cruze evento com o dia ou turno mais concentrado.
   Exemplo: "depois do entregador novo, só sexta e sábado à noite".
4. **Teste e correção:** roda a ferramenta certa para cada candidata e corrige
   os p-valores por Benjamini-Hochberg.
5. **Veredito:**
   - `sustentada`: q < 0,05 e o limite inferior do efeito ≥ 1 reclamação por semana;
   - `descartada`: o limite superior < 1 reclamação por semana;
   - `inconclusiva`: o resto.
   - A força vem do tamanho do efeito e da largura do intervalo.
6. **Ordem:** primeiro as sustentadas, pela força do efeito; depois as
   inconclusivas; por último as descartadas.
7. **Frase e exemplos:** cada hipótese ganha uma `frase` em português simples,
   sem número técnico. Exemplo: *"A demora aumentou nas sextas e sábados à
   noite depois de 3 de junho, quando entrou o entregador novo."* Guarde até 3
   avaliações de exemplo.
8. **Auditoria:** cada passo vira um `PassoInvestigacao` e é gravado com
   `auditoria.registrar()`.
9. **Dados insuficientes:** com menos de 6 semanas de dados ou menos de 30
   reclamações no assunto, devolva `estado="dados_insuficientes"` com o motivo.

**A IA nunca decide veredito.** Ela só entra como candidata, e toda candidata
passa pela estatística.

**2.3 Avaliador** (`app/investigacao/avaliador.py`).

Roda o motor em 200 cenários do mundo simulado e mede:
- acerto da causa principal: a primeira da lista é a do gabarito;
- acerto entre as três primeiras;
- falso alarme: nos cenários sem causa, quantas vezes aparece alguma "sustentada";
- **régua:** culpar o evento do diário mais recente antes da alta;
- **com IA e sem IA.** Na CI, use respostas gravadas da IA para não gastar.

Grave o resultado em `docs/evidencias/laboratorio-agente.json`. Combine com o
Nicolas um tipo `ResultadoLaboratorio` em `contratos.py` para a tela
`/laboratorio`.

**Pronto quando:** um teste com 40 cenários garante que o acerto do agente supera
o da régua e que o falso alarme fica em no máximo 10%.

## Fase 3: simulador "E se?" e experimento (semanas 5–7)

**3.1 Filas** (`app/decisao/filas.py`).
- Erlang C (M/M/c): recebe chegadas por hora, minutos de atendimento e equipe.
- Devolve a probabilidade de esperar, a espera média e a fração de clientes que
  espera mais que *t* minutos.
- Se a ocupação for ≥ 1, a fila explode. Devolva um aviso ("a equipe não dá
  conta do movimento") em vez de um número.
- Teste: compare com uma simulação SimPy. A diferença deve ficar abaixo de 10%.

**3.2 Calibração** (`app/decisao/calibracao.py`).
- GLM de Poisson por negócio e assunto, com o movimento como offset:
  - demora em função da fração de clientes com espera longa;
  - atendimento em função de clientes por atendente.
- **Na vida real, a avaliação não diz o turno da visita.** Calibre pelo total do
  dia e distribua entre os turnos pelo peso de cada um. Documente essa escolha.

**3.3 Simulador** (`app/decisao/simulador.py`).
- `simular()`: faz 2.000 sorteios dos coeficientes, pela covariância do GLM,
  mais o ruído de Poisson. Calcula a mediana, o mínimo (P10) e o máximo (P90)
  por mês.
- Custo extra por mês: diferença de equipe × horas do turno × custo por hora ×
  4,33 semanas.
- Custo por reclamação evitada.
- Devolve `confiavel=False`, com o motivo, quando:
  - falta o turno;
  - há menos de 8 semanas de dados;
  - o coeficiente não é diferente de zero.
- `cenario_sugerido(investigacao)`: se a causa principal envolve dia, turno ou
  equipe, sugere +1 pessoa nesses turnos.

*Validação:* no mundo simulado, rode o mesmo negócio, com a mesma semente, com e
sem a mudança. Isso dá o efeito verdadeiro. O simulador deve acertar esse efeito
dentro da faixa em pelo menos 80% de 100 cenários e ganhar da régua
proporcional. Faça disso um teste.

**3.4 Desenho do experimento** (`app/experimento/desenho.py`).
- Sorteio balanceado por dia da semana, ou por turno, com a semente guardada.
- Durações possíveis: 4, 6 ou 8 semanas.
- Análise de poder por simulação: com 80% de poder e α = 5%, calcula o menor
  efeito detectável, em reclamações por mês.
- Entrega o plano para `selo.selar()` do Matheus.

**3.5 Análise do experimento** (`app/experimento/analise.py`).
1. Chama `selo.conferir()`. Se o selo não bater, recusa e registra na auditoria.
2. Modelo: GLM de Poisson com contagem em função do tratamento, com efeito fixo
   de dia da semana e tendência.
3. A análise segue o sorteio original ("intenção de tratar"). Mede também a adesão.
4. Intervalo: teste de permutação que sorteia de novo **dentro de cada bloco**,
   respeitando o desenho.
5. Veredito:
   - `funcionou`: o intervalo inteiro mostra queda;
   - `nao_funcionou`: o intervalo descarta uma queda do tamanho do menor efeito relevante;
   - `ainda_nao_da_para_saber`: o resto.

*Validação:*
- **Testes A/A:** 200 experimentos em que a ação não faz nada. O falso
  "funcionou" deve ficar em no máximo 8%.
- **Curva de poder:** grave em `docs/evidencias/poder-experimento.json`.

## Fase 4: prova e entrega (semanas 7–8)

- **Laboratório completo:** rode agente, simulador e experimento no mundo
  simulado e grave todos os resultados em `docs/evidencias/`.
- **`docs/modelos.md`:** para cada modelo, o método, as métricas, a régua, os
  resultados e as limitações. Deve estar pronto para o Nicolas colocar no
  relatório.
- **Revisão da tela de acurácia:** confira com o Nicolas que todo número da tela
  bate com o banco.

## Se der tempo

- Comparar o classificador TF-IDF com o BERTimbau ajustado (`neuralmind/bert-base-portuguese-cased`).
- Previsão com feriados (SARIMAX), comparada com o modelo atual e com a régua.
- Simulação de filas com tempo de atendimento não exponencial, em SimPy, no
  lugar de Erlang C.
- Coleta pela API do Perfil da Empresa no Google, junto com o Matheus.

## Não faça

- Não deixe a IA escolher veredito, número ou empresa.
- Não leia o banco sem `empresa_id`.
- Não mexa em `templates/`, `static/` ou `graficos/`: peça ao Nicolas.
- Não mexa em `seguranca/`, `ia.py` ou `protecao.py`: peça ao Matheus.
- Não mude `contratos.py` sem PR aprovado pelos três.
