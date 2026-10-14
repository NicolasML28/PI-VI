# Plano geral: Radar de Reclamações PI6

## 1. O que muda

A crítica da professora foi: *"o projeto de vocês é basicamente pegar dados e
filtrar"*. A versão atual separa avaliações por assunto e conta. A nova versão
**produz conclusões que não estão em nenhum dado**:

| Hoje | Depois |
|---|---|
| "A demora subiu." | **Por quê?** Um agente investiga a causa e prova com estatística. |
| (nada) | **E se eu...?** Um simulador mostra o efeito e o custo de uma mudança antes de o dono gastar. |
| (nada) | **Funcionou mesmo?** Um experimento controlado testa a mudança na vida real e separa efeito de coincidência. |

O projeto é escrito **do zero**. As telas da versão anterior (painel, previsão,
alertas, acurácia e entrada) são refeitas com dados do banco no lugar dos
sorteados e com linguagem simples. O código antigo serve só para entender o tema.

## 2. A jornada do dono do negócio

1. **Painel:** "A demora vai subir na Pizzaria Bella Massa." Botão **Por que isso está acontecendo?**
2. **Investigação:** o sistema lista as causas que testou, da mais provável para a
   menos provável. Exemplo: *"Provável causa: a demora subiu nas sextas e sábados
   à noite depois de 3 de junho, quando entrou o entregador novo."*
   Botão **E se eu mudar algo?**
3. **Simulador "E se?":** o dono mexe na equipe do turno: *"+1 entregador sexta e sábado à noite"*.
   O sistema responde: *"As reclamações de demora devem cair de 57 para 38 a 45 por mês.
   Custo: R$ 1.280 por mês, ou R$ 80 por reclamação evitada."* Botão **Testar na prática**.
4. **Experimento:** o sistema sorteia em quais sextas e sábados o entregador extra
   trabalha, durante 6 a 8 semanas, e sela o plano para ninguém mudar depois. O
   dono marca o que fez em cada dia. No fim: *"Funcionou: 11 reclamações a menos
   por mês (entre 6 e 16)"*, ou *"Ainda não dá para saber"*.

## 3. As três funcionalidades

### 3.1 Agente investigador (Funcionalidade 1)

**Gatilho:** um alerta de alta prevista (já existe) ou o botão na tela.

**De onde vêm as hipóteses:**
- **Diário do negócio:** mudanças que o dono registra com data. Exemplos:
  "entregador novo", "entrei no iFood", "mudei o cardápio", "reforma".
- **Calendário:** feriados (biblioteca `holidays`) e datas comemorativas.
- **Clima:** chuva e temperatura por dia, da API Open-Meteo, que é gratuita e
  não pede chave.
- **Quando acontece:** concentração da reclamação por dia da semana e por turno.
- **Palavras novas:** termos que passaram a aparecer nas avaliações do período,
  como "entregador" ou "motoboy".
- **IA (opcional):** um modelo de linguagem lê uma amostra das avaliações, já
  mascaradas, e propõe hipóteses **escolhidas de uma lista fechada de tipos**.
  A IA só sugere. Quem decide é a estatística.

**Como cada hipótese é testada:** o agente usa ferramentas estatísticas
determinísticas.
- Mudança antes/depois de um evento: regressão de Poisson com tendência e
  degrau, ou teste de permutação.
- Concentração por dia ou turno: qui-quadrado contra o período de base.
- Efeito do clima e do feriado: regressão de Poisson com controle de tendência.
- Palavra nova: log-odds com prior informativo.
- Correção para muitas hipóteses: Benjamini-Hochberg.

**Resultado:** cada hipótese recebe um veredito. Na tela aparecem só as
palavras simples:

| Veredito interno | Na tela |
|---|---|
| sustentada | **Provável causa** |
| descartada | **Não parece ser isso** |
| inconclusiva | **Ainda não dá para dizer** |

A força da evidência aparece como forte, moderada ou fraca.

Cada passo do agente fica registrado: o que ele leu, que teste rodou e o que
concluiu. O agente não escreve diagnóstico em texto livre nem plano de ação. Ele
entrega as hipóteses testadas e leva ao simulador.

**Como provamos que funciona:** rodamos o agente em 200 negócios do mundo
simulado (seção 4), onde a causa verdadeira é conhecida, e medimos:
- **acerto da causa principal:** a primeira da lista é a verdadeira;
- **acerto entre as três primeiras;**
- **falso alarme:** nos cenários sem causa, o agente tem que dizer "ainda não dá para dizer";
- **régua:** "culpar o evento registrado mais recente antes da alta". O agente precisa ganhar dela;
- **com IA e sem IA:** mostra se o modelo de linguagem ajuda ou só enfeita.

### 3.2 Simulador "E se?" (Funcionalidade 2)

**O que o dono informa:** por dia da semana e turno, a equipe que atende ou
entrega, o movimento médio, o tempo médio de atendimento e o custo por hora.
São poucos números digitados à mão. Ninguém precisa abrir o caixa.

**O que o sistema faz:**
1. **Fila do turno:** usa a fórmula de Erlang C (M/M/c) ou uma simulação de
   eventos com SimPy. Calcula a espera e a fração de clientes que esperam mais
   que o aceitável.
2. **Ligação entre espera e reclamação:** uma regressão de Poisson calibrada com
   o histórico do negócio diz quantas reclamações de demora vêm de cada espera.
   O mesmo vale para atendimento: quanto mais clientes por atendente, mais
   reclamação.
3. **Cenário:** o dono altera turnos. Uma simulação de Monte Carlo dá a
   estimativa, o mínimo, o máximo e o custo extra por mês. Também calcula o
   **custo por reclamação evitada**.
4. **Honestidade:** se faltar dado para um turno, o simulador avisa
   (`confiavel=False` e o motivo) em vez de inventar.

**Como provamos que funciona:** o mundo simulado gera cada negócio a partir de
uma fila de verdade, então o efeito real de "+1 entregador" é conhecido. O
simulador precisa acertar esse efeito dentro da faixa em pelo menos 80% dos
cenários. A régua é "a reclamação cai na mesma proporção do aumento da equipe".

### 3.3 Experimento controlado (Funcionalidade 4)

**Desenho:**
- A unidade é o dia, ou o turno.
- O sorteio de quais dias recebem a ação é **balanceado por dia da semana**:
  metade das sextas com a ação, metade sem.
- A duração vem de uma análise de poder feita por simulação. A tela diz: *"Com
  6 semanas dá para perceber uma queda de 25% ou mais. Quedas menores podem
  passar despercebidas."*

**Selo:**
- Antes de começar, o plano vira um JSON, recebe um hash SHA-256 e fica gravado
  com data e hora.
- Qualquer alteração depois do início é detectada.
- Só o dono pode iniciar ou cancelar, e tudo vai para a auditoria.

**Durante o experimento:** o dono marca, dia a dia, se seguiu o sorteio. A
adesão é medida.

**Análise:**
- Regressão de Poisson com o indicador de ação e efeito fixo de dia da semana.
- A análise segue o sorteio original ("intenção de tratar") e o intervalo vem
  de teste de permutação.
- Veredito: **Funcionou**, **Não funcionou** ou **Ainda não dá para saber**.

**Cuidado com o atraso da avaliação:** o cliente escreve a avaliação dias depois
da visita. Quando a fonte informar a data da visita, use a data da visita. Se não
informar, atribua a avaliação a uma janela de 1 a 2 dias antes da data dela, e
documente essa escolha.

**Como provamos que funciona:**
- **Testes A/A:** a ação não faz nada. O experimento deve dizer "funcionou" em
  no máximo cerca de 5% dos casos.
- **Curva de poder:** com um efeito conhecido, medimos quantas vezes o
  experimento o detecta.

## 4. O mundo simulado (o laboratório)

O coração da prova. O gerador de dados fictícios do projeto original vira um
**mundo simulado com causas escondidas**:

- Cada negócio tem turnos e uma fila de verdade: equipe, movimento e tempo de
  atendimento.
- A espera gera reclamação de demora. O número de clientes por atendente gera
  reclamação de atendimento. E assim por diante.
- As causas são sorteadas do **catálogo de causas** (`dados/catalogo_causas.yaml`),
  escrito pelo Nicolas a partir das entrevistas com os parceiros. Exemplos:
  "funcionário novo mais lento às sextas", "entrou num aplicativo de entrega",
  "chuva aumenta pedidos", "mudança de fornecedor piora a qualidade".
- **Iscas:** eventos registrados no diário que não causam nada, para o agente não
  culpar sempre o último evento.
- **Cenários sem causa:** só ruído.
- **Ataques:** avaliações com tentativa de manipular a IA, escritas pelo Matheus.
- **Gabarito separado:** fica fora do alcance do agente e só o avaliador lê.
- **Granularidade diária com turno.** O simulador e o experimento precisam disso.

O mesmo mundo alimenta as telas de demonstração, prova o agente, calibra o
simulador e valida o experimento.

## 5. Estrutura de pastas

```
app/
  web.py                 cria o app e liga os roteadores
  contratos.py           os tipos que passam entre as partes (os três)
  conteudo.py            textos fixos e glossário (Nicolas)
  auditoria.py           trilha de quem fez o quê (Matheus)
  banco/                 SQLAlchemy, sessão, modelos (Gabriel)
  modelos/               classificador e previsão (Gabriel)
  mundo/                 mundo simulado, catálogo, gabarito (Gabriel)
  investigacao/
    ferramentas.py       testes estatísticos (Gabriel)
    motor.py             gera candidatas, testa, ordena (Gabriel)
    avaliador.py         mede o agente no mundo simulado (Gabriel)
    ia.py                chamada ao modelo de linguagem (Matheus)
    protecao.py          mascaramento, delimitação, validação de saída (Matheus)
  decisao/               filas, calibração, simulador "E se?" (Gabriel)
  experimento/
    desenho.py           sorteio e poder (Gabriel)
    analise.py           efeito e veredito (Gabriel)
    selo.py              hash e conferência do plano (Matheus)
  seguranca/             senha, sessão, CSRF, cabeçalhos, contas, permissões (Matheus)
  rotas/                 um roteador por área de tela (Nicolas; auth.py é do Matheus)
  graficos/              SVG desenhado no servidor (Nicolas)
  templates/  static/    telas (Nicolas)
alembic/                 migrações (Gabriel)
dados/catalogo_causas.yaml  (Nicolas escreve, Gabriel consome)
docs/                    relatório, entrevistas, personas, evidências (Nicolas; seguranca/ é do Matheus)
tests/                   unitários de cada um; tests/e2e/ com Playwright (Nicolas)
```

## 6. Contratos: `app/contratos.py`

> **Atualização:** `app/contratos.py` já existe e é a versão oficial. Em relação
> ao rascunho abaixo, ele acrescenta:
> - os assuntos com identificador sem acento (`preco`);
> - `antes_semana` e `depois_semana` em `Hipotese`, para o gráfico antes e depois;
> - `media_com` e `media_sem` em `ResultadoExperimento`;
> - os tipos `MarcacaoDia`, `PropostaIA` e `ResultadoLaboratorio`;
> - os tipos das telas de sempre: `SerieMensal`, `PrevisaoMes`, `ResumoComercio`,
>   `AvaliacaoTela`, `Alerta`, `AcuraciaClassificacao`, `AcuraciaPrevisao` e
>   `EtapaParceiro`.

O prompt de início cria este arquivo, exatamente com estes campos. Enquanto as
partes reais não ficam prontas, cada um trabalha com **versões falsas que
devolvem esses mesmos tipos** (`app/falsos.py`). Assim o Nicolas desenha a tela
da investigação sem esperar o motor.

```python
from dataclasses import dataclass
from datetime import date, datetime
from typing import Literal

Veredito = Literal["sustentada", "descartada", "inconclusiva"]
Forca = Literal["forte", "moderada", "fraca"]
OrigemHipotese = Literal["diario", "calendario", "clima", "dia_turno", "termo_novo", "ia"]
TipoEvento = Literal["equipe", "produto", "canal_venda", "horario", "preco", "estrutura", "outro"]
Periodo = Literal["manha", "tarde", "noite"]


@dataclass(frozen=True)
class EventoDiario:
    id: str
    empresa_id: str
    data: date
    tipo: TipoEvento
    descricao: str


@dataclass(frozen=True)
class Hipotese:
    id: str
    origem: OrigemHipotese
    frase: str                      # português simples, pronto para a tela
    categoria: str
    teste: str                      # nome da ferramenta estatística usada
    efeito_semana: float            # reclamações a mais por semana
    efeito_min: float
    efeito_max: float
    q_valor: float                  # nunca aparece na tela do dono
    veredito: Veredito
    forca: Forca
    avaliacoes_exemplo: tuple[str, ...]   # ids de avaliações, texto já mascarado


@dataclass(frozen=True)
class PassoInvestigacao:
    ordem: int
    quando: datetime
    acao: str                       # "gerou candidatas", "rodou teste", "chamou IA"
    entrada_resumo: str
    saida_resumo: str


@dataclass(frozen=True)
class Investigacao:
    id: str
    empresa_id: str
    alerta_id: str | None
    categoria: str
    periodo_alta: tuple[date, date]
    periodo_base: tuple[date, date]
    hipoteses: tuple[Hipotese, ...]       # da mais provável para a menos
    passos: tuple[PassoInvestigacao, ...]
    usou_ia: bool
    estado: Literal["concluida", "dados_insuficientes"]
    motivo: str | None


@dataclass(frozen=True)
class Turno:
    dia_semana: int                 # 0 = segunda
    periodo: Periodo
    equipe: int
    clientes: float                 # média de clientes ou pedidos no turno
    minutos_atendimento: float
    custo_hora: float


@dataclass(frozen=True)
class Cenario:
    nome: str
    categoria: str
    mudancas: tuple[Turno, ...]     # só os turnos que mudam


@dataclass(frozen=True)
class ResultadoSimulacao:
    cenario: Cenario
    reclamacoes_atual: float        # por mês, na categoria
    reclamacoes_cenario: float
    minimo: float
    maximo: float
    espera_atual_min: float
    espera_cenario_min: float
    custo_extra_mes: float
    custo_por_reclamacao_evitada: float | None
    confiavel: bool
    motivo: str | None


@dataclass(frozen=True)
class PlanoExperimento:
    id: str
    empresa_id: str
    categoria: str
    acao: str
    inicio: date
    fim: date
    dias_com_acao: tuple[date, ...]
    dias_sem_acao: tuple[date, ...]
    menor_efeito_detectavel: float  # reclamações por mês
    selo: str                       # SHA-256 (Matheus)
    selado_em: datetime
    estado: Literal["rascunho", "em_andamento", "encerrado", "cancelado"]


@dataclass(frozen=True)
class ResultadoExperimento:
    plano_id: str
    dias_observados: int
    adesao: float                   # fração dos dias em que o sorteio foi seguido
    efeito_mes: float               # negativo = menos reclamação
    efeito_min: float
    efeito_max: float
    veredito: Literal["funcionou", "nao_funcionou", "ainda_nao_da_para_saber"]
    frase: str
```

**Funções de fronteira** (assinaturas fixas):

| Função | Dono | Quem chama |
|---|---|---|
| `investigacao.motor.investigar(sessao, empresa_id, categoria, alerta_id=None, usar_ia=True) -> Investigacao` | Gabriel | rotas (Nicolas) |
| `investigacao.ia.propor_hipoteses(avaliacoes_mascaradas, categoria, periodo) -> list[PropostaIA]` | Matheus | motor (Gabriel) |
| `investigacao.protecao.mascarar(texto) -> str` | Matheus | coleta e IA |
| `decisao.simulador.simular(sessao, empresa_id, cenario) -> ResultadoSimulacao` | Gabriel | rotas |
| `decisao.simulador.cenario_sugerido(investigacao) -> Cenario \| None` | Gabriel | rotas |
| `experimento.desenho.desenhar(sessao, empresa_id, categoria, acao, turnos, semanas) -> PlanoExperimento` | Gabriel | rotas |
| `experimento.selo.selar(plano) -> PlanoExperimento` e `conferir(plano) -> bool` | Matheus | rotas e análise |
| `experimento.analise.analisar(sessao, plano_id) -> ResultadoExperimento` | Gabriel | rotas |
| `seguranca.contas.usuario_atual(request)` e `empresa_do_usuario(request)` | Matheus | todas as rotas |
| `auditoria.registrar(sessao, usuario, acao, alvo, detalhe)` | Matheus | todos |

## 7. Telas novas

| Endereço | O que mostra | Botão principal |
|---|---|---|
| `/investigacao/{empresa}/{categoria}` | Causas testadas, com veredito e força | **E se eu mudar algo?** |
| `/diario` | Mudanças que o dono registrou, com data | **Registrar mudança** |
| `/operacao` | Equipe e movimento por turno | **Salvar** |
| `/simulador/{empresa}` | Cenário atual comparado com o novo, e o custo | **Testar na prática** |
| `/experimentos` e `/experimentos/{id}` | Plano, calendário do sorteio, marcação diária e resultado | **Marcar o dia de hoje** |
| `/laboratorio` | O quanto o agente, o simulador e o experimento acertam no mundo simulado | (área da banca) |
| `/ajuda` | Glossário e "como ler cada gráfico" | |

## 8. Fases

Ajuste as semanas ao calendário da disciplina. Cada fase termina com tudo
integrado na `main` e publicado no Render.

| Fase | Semanas | Gabriel | Matheus | Nicolas |
|---|---|---|---|---|
| **0. Início do zero** | 1 | Cria o banco e revisa os contratos e os falsos | Escreve a entrada segura e liga a CSP | Base do site pronta (telas, gráficos, ajuda, testes); cria o git e marca as entrevistas |
| **1. Dados reais e telas novas** | 2–3 | Classificador real, previsão no banco, mundo simulado diário | Contas por empresa, isolamento, auditoria, CI de segurança | Entrevistas, catálogo de causas, protótipo no Figma, redesenho dos gráficos e botões de ajuda |
| **2. Agente** | 3–5 | Ferramentas estatísticas, motor, avaliador | IA segura, mascaramento, bateria de ataques | Telas de investigação e diário, com dados falsos e depois reais |
| **3. Simulador e experimento** | 5–7 | Filas, calibração, simulador, desenho e análise | Selo, permissões do experimento | Telas do simulador e do experimento |
| **4. Prova e entrega** | 7–8 | Resultados do laboratório | OWASP Top 10, evidências, modelo de ameaças | Playwright, teste de usabilidade, relatório ABNT, ensaio da banca |

**Dependências críticas:**
- O catálogo de causas (Nicolas) precisa sair até o fim da semana 2. O mundo
  simulado depende dele.
- `contratos.py` e `falsos.py` saem no prompt de início, na semana 1. As telas
  dependem deles.
- Mascaramento e isolamento (Matheus) precisam sair antes de qualquer avaliação
  real ou de qualquer chamada à IA.

## 9. Riscos e o que fazer

| Risco | Plano |
|---|---|
| Parceiro não assina o termo a tempo | Tudo roda no mundo simulado, com o aviso de demonstração. A coleta real vira extra. |
| Pouca avaliação por dia, e o experimento sem poder | A tela diz o menor efeito detectável antes de começar. Documentamos isso como limitação, não escondemos. |
| Custo ou queda da API de IA | O modo sem IA é o padrão nos testes. A IA tem limite de chamadas e de custo, e sempre há um caminho sem ela. |
| Escopo grande para 8 semanas | Cada arquivo de pessoa separa **Obrigatório** de **Se der tempo**. O obrigatório fecha a história inteira. |
| Banco gratuito do Render expira | Confirmar o limite do plano gratuito na semana 1. A alternativa é o Neon. |

## 10. O que fica de fora

- O sistema não escreve diagnóstico em texto livre nem plano de ação, e não
  acompanha sozinho se a ação resolveu. Quem mede isso é o experimento.
- Nada de coletar avaliação copiando a página do Google, o que viola os termos
  de uso. Coleta real só com autorização do dono, pela API do Perfil da Empresa
  no Google, ou por importação de planilha.
- Nada de publicar resposta em nome do dono.
