# Prompt de início: o projeto do zero

> **Já executado em parte.** A base do site foi feita: etapas 2, 3, 6, 7 e 8
> (esqueleto, contratos, falsos, visual, telas e testes). O banco (etapa 4)
> virou a Fase 0 do Gabriel, e a entrada segura (etapa 5), a Fase 0 do Matheus.
> As etapas 1 e 9 (git e PR) ficam com o Nicolas. Este arquivo fica como
> registro de como o projeto começou.

Quem roda: **Nicolas**, no Claude Code, dentro da pasta `C:\Users\Gamer\Desktop\PI6`.
Copie tudo o que está dentro do bloco abaixo e cole como primeira mensagem.

O trabalho vai em 9 etapas. No fim de cada uma, o Claude para, mostra o que fez
e espera você dizer **pode seguir**. Depois da etapa 9, cada um segue o próprio
arquivo de plano a partir da Fase 0.

```text
Eu sou o Nicolas Marques Linares. Vamos começar do zero o projeto Radar de
Reclamações PI6 (Projeto Integrador VI, Engenharia de Software, PUC-Campinas),
feito por mim, pelo Gabriel Lopes Londe Rodrigues e pelo Matheus Rocafa Moraes.

CONTEXTO
- O tema vem de um projeto anterior nosso:
  https://github.com/Lopesloro/radar-de-reclamacoes, que está no ar em
  https://radar-de-reclamacoes-pi6.onrender.com.
  Ele lê avaliações públicas de pequenos comércios, separa por assunto
  (atendimento, demora, preço, qualidade, ambiente), prevê o mês seguinte e
  compara todo modelo com um palpite simples.
- A professora achou o projeto básico ("pegar dados e filtrar"). Por isso a
  versão nova acrescenta três coisas: o agente investigador ("Por quê?"), o
  simulador ("E se?") e o experimento controlado ("Testar na prática").
- NÃO copie código do projeto anterior. Pode lê-lo só para entender o tema.
  Tudo aqui é escrito do zero.
- Antes de tudo, leia nesta pasta: CLAUDE.md, plano/00-visao-geral.md,
  plano/01-gabriel.md, plano/02-matheus.md e plano/03-nicolas.md. Abra também
  alguns prints de referencia/site-atual/. Eles mostram a versão anterior e os
  problemas que não podemos repetir, listados em plano/03-nicolas.md.

OBJETIVO DESTA CONVERSA
Criar a base do projeto do zero: repositório, esqueleto, contratos, dados
falsos, banco, entrada segura, base visual com os botões de ajuda, telas
iniciais e testes. As partes inteligentes (classificador real, mundo simulado,
agente, simulador e experimento de verdade) NÃO entram agora. Cada integrante
faz a sua depois, seguindo o próprio plano. Aqui, essas partes aparecem nas
telas alimentadas por app/falsos.py.

COMO TRABALHAR
- Uma etapa por vez. No fim de cada etapa:
  - rode os testes;
  - faça um commit com a minha identidade do git (não mude user.name nem
    user.email), com mensagem em português;
  - me mostre um resumo curto em linguagem simples: o que foi feito e o que
    devo conferir;
  - espere eu dizer "pode seguir".
- Antes de qualquer ação que mude algo no GitHub, me diga o que vai fazer e
  espere meu ok.
- Siga as regras do CLAUDE.md:
  - toda tabela e toda consulta com empresa_id;
  - avaliação é dado, nunca ordem;
  - nenhum termo técnico na tela do dono, fora de <details class="tecnico">;
  - CSP sem unsafe-inline: nada de <script> inline, onclick= ou style=;
  - nomes em português.
- Estou no Windows. O ambiente virtual fica em .venv e o Python em
  .venv\Scripts\python.exe.

ETAPA 1 · REPOSITÓRIO
1. A pasta C:\Users\Gamer é um repositório git criado sem querer. Não mexa nele.
   Rode "git init -b main" dentro de PI6 e confirme com
   "git rev-parse --show-toplevel" que a raiz agora é PI6.
2. Crie o .gitignore: Python, .venv, .env, .env.*, .pytest_cache, *.db,
   test-results, .claude/.
3. Primeiro commit na main só com CLAUDE.md, plano/, referencia/ e .gitignore.
4. Crie o repositório radar-de-reclamacoes-pi6 no GitHub. Pergunte se deve ser
   público ou privado. Recomende público: o site mostra o código como prova de
   autoria, e a proteção de branch é grátis em repositório público. Use o gh se
   estiver instalado e logado. Se não, me guie pelo site.
5. Envie a main. Adicione como colaboradores o Gabriel (Lopesloro) e o Matheus.
   Pergunte o usuário do GitHub do Matheus.
6. Se for público, proteja a main: só por pull request, com 1 aprovação.
7. Crie a branch nicolas/inicio. As etapas 2 a 8 acontecem nela.

ETAPA 2 · ESQUELETO
- requirements.txt com versões fixadas: fastapi, uvicorn, jinja2,
  python-multipart, itsdangerous, sqlalchemy, alembic, psycopg[binary], pandas,
  numpy, scipy, statsmodels, scikit-learn, joblib, simpy, holidays, pyyaml.
- requirements-dev.txt: pytest, httpx, ruff, pytest-playwright,
  axe-playwright-python.
- pyproject.toml com a configuração do pytest e do ruff.
- A estrutura de pastas da seção 5 de plano/00-visao-geral.md, com __init__.py.
- app/web.py:
  - cria o FastAPI, monta /static e os templates, e liga os roteadores de
    app/rotas/;
  - GET /saude devolve só estado e versão, nada sobre credenciais.
- .github/CODEOWNERS conforme a tabela do CLAUDE.md.
- .github/workflows/ci.yml: ruff e pytest, com um serviço PostgreSQL.
- render.yaml:
  - plano gratuito;
  - buildCommand instala o requirements.txt;
  - startCommand: uvicorn com --proxy-headers e --forwarded-allow-ips;
  - healthCheckPath em /saude;
  - variáveis RADAR_CHAVE (gerada), DATABASE_URL e RADAR_SENHA_DEMO, nenhuma
    com valor no arquivo.
- README.md curto: o que é e como rodar.

ETAPA 3 · CONTRATOS E FALSOS
- app/contratos.py exatamente como na seção 6 de plano/00-visao-geral.md.
  Acrescente:
  - PropostaIA: tipo da lista fechada, termo opcional, dias_semana, periodo,
    justificativa_curta;
  - ResultadoLaboratorio: acerto do agente, acerto da régua, com e sem IA,
    acerto do simulador, falsos positivos do teste A/A;
  - os tipos das telas que a versão anterior já tinha: ResumoComercio,
    SerieMensal (meses fechados, mês parcial, previsões com p10, p50 e p90),
    AvaliacaoTela, Alerta, AcuraciaClassificacao e AcuraciaPrevisao.
  - Cada tipo leva um comentário de uma linha dizendo para que serve.
- app/falsos.py:
  - mesmas assinaturas das funções de fronteira da seção 6;
  - números plausíveis e sempre iguais (semente fixa; nunca hash() de string);
  - as três histórias de demonstração:
    - Pizzaria Bella Massa: entregador novo nas sextas e sábados à noite a
      partir de junho, e a demora sobe;
    - Clínica Sorriso Claro: recepcionista nova, e o atendimento piora;
    - Pet Shop Patinhas: aumento de preço.
- app/servicos.py: a única porta que as rotas usam para pedir dados. Hoje
  chama os falsos. Amanhã cada função passa a chamar a versão real, sem mexer
  nas telas.
- Testes:
  - os falsos devolvem os tipos do contrato;
  - P10 ≤ P50 ≤ P90;
  - a mesma chamada devolve sempre o mesmo resultado.

ETAPA 4 · BANCO
- app/banco/: engine e sessão por DATABASE_URL, com padrão sqlite:///radar.db.
- Modelos SQLAlchemy 2.0 de todas as tabelas do item 0.1 de plano/01-gabriel.md,
  com os campos de usuario e empresa do item 0.1 de plano/02-matheus.md.
  - Toda tabela tem empresa_id, menos empresa.
  - avaliacao tem id_externo único junto com empresa_id, e o campo removida.
- Alembic com a migração inicial.
- python -m app.banco.semear cria:
  - as três empresas de demonstração (demonstracao=True);
  - um usuário de papel "demo", com a senha vinda de RADAR_SENHA_DEMO. Sem a
    variável, usa "demo" só em ambiente local, e a tela de entrada avisa.
- Testes:
  - alembic upgrade head funciona num SQLite temporário;
  - toda tabela, menos empresa, tem empresa_id.

ETAPA 5 · ENTRADA SEGURA (área do Matheus, que vai revisar)
Implemente do zero, com um teste para cada linha da tabela da Fase 0 de
plano/02-matheus.md:
- senha com scrypt e sal aleatório por usuário, no formato
  scrypt$n$r$p$sal$hash, comparada em tempo constante;
- sessão em cookie radar_sessao: HttpOnly, SameSite=Lax, Secure vindo de
  RADAR_COOKIE_SEGURO, validade de 8 horas, assinada com RADAR_CHAVE;
- token CSRF em todo POST, com um teste que abre todas as telas logado e confere
  todos os <form method="post">;
- destino_seguro() no redirecionamento depois da entrada, também para quem já
  está logado;
- limite de 8 tentativas em 5 minutos, por origem e por e-mail;
- cabeçalhos em toda resposta:
  - CSP sem unsafe-inline (fontes do Google permitidas);
  - HSTS em https;
  - X-Frame-Options DENY, nosniff, Referrer-Policy, Permissions-Policy e
    Cross-Origin-Opener-Policy;
- dependências usuario_atual, empresa_do_usuario e exige_papel(...). O
  identificador de empresa vindo da URL só vale se pertencer ao usuário. Se não
  pertencer, 403. O papel "demo" vê só empresas com demonstracao=True;
- teste de isolamento que percorre app.routes automaticamente: um usuário de
  outra empresa recebe 403 em toda rota com empresa;
- telas /entrar e /sair. A tela de entrada não tem jargão (nada de cookie, hash
  ou CSRF).

ETAPA 6 · BASE VISUAL, GRÁFICOS E BOTÕES DE AJUDA (área do Nicolas)
- CSS do zero em app/static/css/app.css, com tokens em :root.
  - Identidade inspirada na versão anterior (ver os prints): título em serifa
    (Playfair Display), texto em Inter.
  - Cores com um significado só:
    - azul-petróleo #0F6E7B: marca e previsão;
    - laranja #B4531F: piorou ou atenção;
    - cinza: passado e normal.
  - Contraste AA. Funciona em 390px sem rolagem horizontal.
- base.html:
  - menu em palavras do dono: Resumo, Avisos, Por quê?, E se?, Testes, Ajuda,
    e um "Mais" com Dá para confiar?, O que falta e Sobre o sistema;
  - faixa de "dados de demonstração";
  - rodapé.
- Macros em _macros.html:
  - ajuda(termo): <details class="ajuda"> com o summary "?" e aria-label, texto
    vindo do GLOSSARIO;
  - como_ler(passos);
  - numeros_em_tabela(...);
  - proximo_passo(botoes);
  - estado_vazio(texto, botao);
  - semaforo(estado), que mostra "Vai piorar", "Fique de olho" ou "Tudo
    normal", sempre com ícone e palavra;
  - etiqueta_veredito(veredito);
  - tecnico(conteudo).
- app/conteudo.py: GLOSSARIO com pelo menos 20 termos, cada um em duas frases
  simples com um exemplo, e os textos fixos do site.
- app/graficos/svg.py, do zero, seguindo as "Regras dos gráficos" de
  plano/03-nicolas.md:
  - título que responde à pergunta;
  - rótulo direto no lugar de legenda;
  - ▲ ▼ sempre junto da cor;
  - faixa escrita "entre X e Y";
  - duas versões de cada gráfico (720px e 360px) alternadas por media query, e
    texto nunca abaixo de 12px no tamanho final.
  - Gráficos:
    - colunas_mensais: 6 meses fechados, o mês parcial e 2 meses previstos;
    - barras_assuntos: contagem e variação escrita;
    - antes_depois: duas colunas;
    - comparacao_cenario: hoje contra a mudança;
    - pictograma: "8 de cada 10".
- app/static/js/: inicio.js marca que há JavaScript. tour.js tem um tour de 5
  passos no Resumo, que lembra se a pessoa já viu (localStorage dentro de
  try/catch).

ETAPA 7 · TELAS INICIAIS, COM OS FALSOS (área do Nicolas)
- Página pública "/": comercial, sem jargão, uma ação só ("Entrar"). Mensagem:
  "descobre por que seus clientes reclamam, mostra o que fazer antes de gastar
  e prova se funcionou".
- Telas internas, todas pedindo dados só por app/servicos.py:
  - /resumo: um cartão por negócio;
  - /comercio/{empresa};
  - /previsao/{empresa};
  - /avisos;
  - /investigacao/{empresa}/{categoria};
  - /simulador/{empresa}: formulário GET que funciona com os falsos;
  - /experimentos e /experimentos/{id}: só leitura por enquanto;
  - /confianca: a antiga acurácia, em linguagem simples;
  - /o-que-falta;
  - /ajuda: glossário e perguntas frequentes;
  - /sobre, a área da banca, com:
    - "Quem construiu": os três, com a frente de cada um, conforme os
      arquivos do plano;
    - as tecnologias;
    - os detalhes técnicos dentro de <details class="tecnico">.
- Em toda tela:
  - título que responde à pergunta;
  - botão "?" nos números de destaque;
  - "Como ler" e "Ver os números em tabela" em todo gráfico;
  - um próximo passo no fim.
- Teste de jargão: percorre todas as telas do dono e falha se achar F1, P10,
  P90, MAE, macro, régua, p-valor, q-valor, hash ou CSRF fora de
  <details class="tecnico">.

ETAPA 8 · TESTES DE PONTA A PONTA E CI
- tests/e2e/conftest.py:
  - sobe o app numa thread com uvicorn.Server, numa porta livre, com um SQLite
    temporário semeado;
  - fixture pagina_logada, que entra com o usuário demo;
  - roda em 390px e 1280px.
- Testes:
  - toda tela abre, sem erro no console e sem rolagem horizontal da página;
  - todo "?" abre e fecha;
  - o axe não acha violação séria ou crítica;
  - o botão principal de cada tela é alcançável só com Tab.
- Guarde um print de cada tela em docs/evidencias/telas/.
- Acrescente na CI um job de ponta a ponta que instala o Chromium do
  Playwright.
- Rode tudo localmente e conserte o que falhar.

ETAPA 9 · PR DE INÍCIO
- Envie a branch nicolas/inicio e abra um PR para a main. O PR deve ter:
  - o que foi feito, em linguagem simples;
  - os prints em 390px e 1280px;
  - um checklist de revisão:
    - Gabriel: banco, contratos e falsos;
    - Matheus: entrada segura;
    - Nicolas: telas.
- Me mostre a mensagem pronta para eu mandar no grupo. Ela avisa que o PR está
  aberto, o que cada um revisa e que, depois do merge, cada um segue a Fase 1
  do próprio arquivo em plano/.
- No fim, me explique em linguagem simples como o projeto ficou organizado, para
  eu conseguir apresentar na banca.

Comece pela Etapa 1.
```
