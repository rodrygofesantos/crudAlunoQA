# Análise e estratégia de testes funcionais — Academia Alunos Web

## 1. Identificação

| Campo | Valor |
|---|---|
| Sistema | academia-alunos-web (pom.xml, application.properties). O diretório e o repositório local chamam-se crudAlunoQA; a divergência é apenas de nome |
| Versão do sistema / do documento | 1.0.0 / 1.0 |
| Data | 24/11/2026 |
| Branch / commit | main / 5e03a989b1984d834472431ecd7e64136933ade2 |
| Alterações locais no início | Docs/prompt2.md não rastreado, preexistente. O commit, portanto, não representa sozinho todo o conteúdo do diretório |
| Responsável | A definir |
| Elaboração | Codex (catálogo, riscos e casos iniciais) e Claude Code (conclusão, revisão e exploração controlada); revisão humana pendente |
| Status | Especificação elaborada e não executada |
| Documento complementar | [Casos de teste funcionais](CASOS_DE_TESTE_FUNCIONAIS.md), versão 2.0 |

## 2. Objetivo e contexto

Este documento fundamenta e organiza os testes funcionais manuais do Academia Alunos Web, aplicação didática de cadastro de alunos de academia. Atende a uma pós-graduação em Quality Assurance: cada decisão de teste é ligada a uma evidência do repositório e, quando pertinente, à teoria que a sustenta.

O contexto informado foi confirmado: Java 21 e Spring Boot 3.5.13, API REST em `/api/alunos`, Spring Data JPA, H2 em memória, interface em HTML, CSS e JavaScript, operações de cadastro, consulta, atualização e exclusão, validações, pesquisa, filtros e decisões de interface associadas às heurísticas de Nielsen (README §16). Não há testes automatizados nem diretório `src/test`.

Três resultados merecem atenção antes da execução:

1. Há divergências observadas entre documentação e comportamento: nome abaixo do mínimo gravado pela API ([LAC001](#lac001)), plano aceito por número ([LAC016](#lac016)), foco perdido após salvar edição ([LAC002](#lac002)) e falha de carregamento apresentada como banco vazio após pesquisa ([LAC015](#lac015)).
2. 12 dos 116 cenários dependem de definição, principalmente sobre erros de protocolo HTTP ([LAC006](#lac006)).
3. As análises preditivas existentes usam os identificadores da versão 1.0 e não devem ser lidas com os atuais ([LAC013](#lac013)).

As divergências são possíveis defeitos, sujeitos à confirmação na execução formal.

## 3. Fontes de evidência e limitações da análise

### 3.1 Fontes

| Fonte | Uso |
|---|---|
| README.md | Referência do comportamento esperado: regras (§9), API (§10), interface (§12), persistência (§13), responsividade (§15), heurísticas (§16) e acessibilidade (§17) |
| Código Java: controller, dto, entity, exception, repository, service | Evidência do comportamento implementado |
| index.html, alunos.js | Evidência da interface; lidos integralmente |
| estilos.css | Inspeção das regras responsivas (media queries); a aparência está fora do escopo |
| application.properties, pom.xml | Configuração, persistência e versões |
| Docs/CASOS_DE_TESTE_FUNCIONAIS.md, versão 1.0 | Especificação anterior, com 20 casos; conteúdo relevante preservado e ampliado |
| Docs/ANALISE_PREDITIVA_CASOS_DE_TESTE.md | Contexto histórico; leitura parcial, não usada como fonte de requisitos |

Não há requisitos formais aprovados além do README, nem testes existentes. Regras que aparecem somente no código são identificadas como inferidas e não equivalem a requisito aprovado.

### 3.2 Exploração controlada

Realizada em 24/11/2026 (America/Sao_Paulo), em cópia do projeto fora do repositório, na porta 18080, com banco H2 em memória exclusivo e dados sintéticos. A API foi acionada por cliente HTTP e a interface por Google Chrome 158 em modo headless, com perfil descartável. Nenhum processo preexistente foi encerrado. Roteiros auxiliares foram mantidos em área temporária e não integram a entrega.

| Condição explorada | Observação | Casos relacionados | Lacuna |
|---|---|---|---|
| Nome " An " em POST e PUT | 201 / 200; persistido "An" | [CT106](CASOS_DE_TESTE_FUNCIONAIS.md#ct106) | [LAC001](#lac001) |
| Nome, CPF e e-mail vazios ou com espaços, 30 envios por entrada | 400; mensagem por campo varia entre envios idênticos | [CT082](CASOS_DE_TESTE_FUNCIONAIS.md#ct082) | [LAC004](#lac004) |
| PUT com id no corpo; POST com campo extra | 200 / 201; propriedade ignorada | [CT100](CASOS_DE_TESTE_FUNCIONAIS.md#ct100) | [LAC005](#lac005) |
| ativo "true" e 1; CPF numérico; data com hora | 201; valores convertidos | [CT105](CASOS_DE_TESTE_FUNCIONAIS.md#ct105) | [LAC005](#lac005) |
| plano 0 (número) | 201; gravado MENSAL | [CT086](CASOS_DE_TESTE_FUNCIONAIS.md#ct086) | [LAC016](#lac016) |
| ID abc, 1.5 e acima de Long; PATCH; text/plain; rotas inexistentes; /favicon.ico | 500 com mensagem genérica | [CT101](CASOS_DE_TESTE_FUNCIONAIS.md#ct101), [CT102](CASOS_DE_TESTE_FUNCIONAIS.md#ct102), [CT103](CASOS_DE_TESTE_FUNCIONAIS.md#ct103), [CT104](CASOS_DE_TESTE_FUNCIONAIS.md#ct104) | [LAC006](#lac006) |
| PUT em ID inexistente com corpo inválido | 400 (validação antes da existência) | [CT097](CASOS_DE_TESTE_FUNCIONAIS.md#ct097) | — |
| E-mail com 253, 254 e 255 caracteres | 201, 201 e 400 | [CT090](CASOS_DE_TESTE_FUNCIONAIS.md#ct090), [CT091](CASOS_DE_TESTE_FUNCIONAIS.md#ct091) | — |
| 60 disputas entre dois POST sincronizados | Sempre 201 + 409; um registro | [CT109](CASOS_DE_TESTE_FUNCIONAIS.md#ct109) | [LAC009](#lac009) |
| Falha de carregamento seguida de digitação na pesquisa | Painel de falha substituído por "Nenhum aluno cadastrado" | [CT004](CASOS_DE_TESTE_FUNCIONAIS.md#ct004) | [LAC015](#lac015) |
| Foco ao abrir e fechar com Esc os quatro diálogos | Conforme documentado | [CT055](CASOS_DE_TESTE_FUNCIONAIS.md#ct055), [CT056](CASOS_DE_TESTE_FUNCIONAIS.md#ct056) | — |
| Foco após salvar edição e após excluir | BODY | [CT063](CASOS_DE_TESTE_FUNCIONAIS.md#ct063), [CT064](CASOS_DE_TESTE_FUNCIONAIS.md#ct064) | [LAC002](#lac002), [LAC003](#lac003) |
| Esc durante POST lento e reabertura do formulário | POST concluído uma vez; a resposta fechou o segundo diálogo | [CT066](CASOS_DE_TESTE_FUNCIONAIS.md#ct066) | [LAC010](#lac010) |
| E-mails limítrofes na interface e na API | Aceitação divergente entre canais | [CT020](CASOS_DE_TESTE_FUNCIONAIS.md#ct020) | [LAC014](#lac014) |
| Colagem de textos acima do limite e de CPF com letras | Truncamento e máscara conforme o código | [CT015](CASOS_DE_TESTE_FUNCIONAIS.md#ct015), [CT017](CASOS_DE_TESTE_FUNCIONAIS.md#ct017) | — |
| Larguras de 320, 680 e 681 px | Cartão até 680 px; tabela em 681 px | [CT062](CASOS_DE_TESTE_FUNCIONAIS.md#ct062) | — |
| Nome com marcação HTML | Exibido como texto literal | [CT027](CASOS_DE_TESTE_FUNCIONAIS.md#ct027) | — |

A exploração não equivale à execução dos casos. Não atribui aprovação ou reprovação.

### 3.3 Limitações

- Um único navegador, em modo headless; leitor de tela e dispositivos físicos não foram usados.
- Execução em Java 25.0.2; o comportamento em JDK 21 não foi verificado.
- Virada de dia, fusos distintos, perda de resposta após gravação e reinício do processo não foram explorados.
- Frequência de uso não está documentada; os riscos usam estimativas.

## 4. Escopo e exclusões

Incluem-se comportamentos observáveis da interface, da API, da integração entre ambas e da persistência: fluxos positivos, negativos e alternativos, limites, mensagens, códigos HTTP, estados, cancelamento, recuperação, teclado, foco e diálogos.

Excluem-se desempenho, carga, estresse, segurança ofensiva, testes unitários, cobertura de código, mutação, implantação, estética e auditoria completa de acessibilidade. Não se cria automação nem se altera código.

## 5. Fundamentação teórica aplicada

### 5.1 Teste funcional e caixa-preta

O teste funcional avalia o que o sistema deve fazer, verificando completude, correção e adequação funcional (ISTQB, 2024, p. 29). As técnicas de caixa-preta derivam os casos do comportamento especificado, sem referência à estrutura interna, e por isso continuam úteis quando a implementação muda e o comportamento exigido permanece (ISTQB, 2024, p. 39).

Essa propriedade orientou a política de resultados esperados. O código foi lido para localizar regras não documentadas e explicar divergências, mas o resultado esperado vem do README sempre que ele define a regra. Em [CT106](CASOS_DE_TESTE_FUNCIONAIS.md#ct106), por exemplo, o esperado é a rejeição, embora a implementação observada grave o nome com dois caracteres. Como parte das regras só existe no código, a especificação não é puramente de caixa-preta; por isso cada metadado declara a origem.

### 5.2 Técnicas utilizadas

| Técnica | Fundamento | Aplicação | Cenários |
|---|---|---|---|
| Particionamento de equivalência | Divide dados em partições tratadas do mesmo modo; um teste por partição é suficiente (ISTQB, 2024, p. 39) | Formatos, planos, presença ou ausência de campos, canais | 61 |
| Análise de valor-limite | Exercita as fronteiras de partições ordenadas (ISTQB, 2024, p. 40) | Nome, e-mail, telefone, datas e larguras. Adotada a versão de dois valores, acrescida de valor nominal | 16 |
| Tabela de decisão | Registra combinações de condições com resultados distintos (ISTQB, 2024, p. 41) | Pesquisa × filtro, precedência de conflitos e de validação | 4 |
| Transição de estados | Modela estados, eventos, condições e ações (ISTQB, 2024, p. 41) | Diálogos, situação do aluno, gravação e persistência | 25 |
| Error guessing fundamentado | Antecipa erros a partir do conhecimento sobre a aplicação (ISTQB, 2024, p. 43-44) | Falhas de rede, registro removido por outro cliente, respostas tardias | 10 |

A ISO/IEC/IEEE 29119-4 define técnicas de projeto usadas no processo da parte 2 da mesma série (ISO; IEC; IEEE, 2021). Somente o resumo de catálogo foi acessado; nenhuma afirmação deste documento depende do texto integral, e não se declara conformidade.

Teste negativo é classificação de cenário, e contrato da API é foco de verificação; nenhum dos dois é tratado como técnica.

### 5.3 Abordagem baseada em risco

O nível de risco combina probabilidade e impacto e, na abordagem quantitativa, resulta da multiplicação dos dois (ISTQB, 2024, p. 52-53). A seção 7 aplica esse produto em escala ordinal de 1 a 4. O resultado ordena prioridades; não é probabilidade estatística, pois não há histórico de defeitos (README §§18 e 21).

### 5.4 Rastreabilidade

A rastreabilidade entre base de teste, riscos e casos apoia a avaliação da cobertura e permite verificar se os requisitos estão cobertos (ISTQB, 2024, p. 20-21). Ela é mantida nos dois sentidos: da regra para os cenários (seção 10.1) e do cenário para regras, risco e evidência (seção 10.2 e metadados).

### 5.5 Gherkin e BDD

North (2006) propôs expressar critérios de aceitação como cenários com contexto, evento e resultados. No Gherkin, os passos iniciais colocam o sistema em estado conhecido, o resultado deve recair sobre saída observável e o esquema é executado uma vez para cada linha de exemplos, exceto o cabeçalho (CUCUMBER, 2026a). Essa última regra fundamenta a contagem de instâncias. A recomendação de usar o idioma dos especialistas do domínio justifica o dialeto português.

Escrever em Gherkin não equivale a adotar BDD. O BDD é uma forma de trabalho colaborativa, com descoberta, formulação e automação, cuja documentação é verificada automaticamente contra o sistema (CUCUMBER, 2026b). O syllabus também prevê a tradução dos casos em testes executáveis (ISTQB, 2024, p. 26). Nesta entrega houve apenas a formulação de cenários, sem oficina de descoberta e sem automação: usa-se a notação, não o processo.

### 5.6 Heurísticas de Nielsen

As heurísticas são princípios gerais para avaliação de interfaces (NIELSEN, 1994). Aqui não serviram para julgar usabilidade, mas para localizar compromissos verificáveis que o README §16 associa a elas: visibilidade do status ([CT002](CASOS_DE_TESTE_FUNCIONAIS.md#ct002), [CT023](CASOS_DE_TESTE_FUNCIONAIS.md#ct023)), controle e liberdade ([CT024](CASOS_DE_TESTE_FUNCIONAIS.md#ct024), [CT043](CASOS_DE_TESTE_FUNCIONAIS.md#ct043), [CT066](CASOS_DE_TESTE_FUNCIONAIS.md#ct066)), prevenção de erros ([CT010](CASOS_DE_TESTE_FUNCIONAIS.md#ct010), [CT042](CASOS_DE_TESTE_FUNCIONAIS.md#ct042)) e recuperação de erros ([CT022](CASOS_DE_TESTE_FUNCIONAIS.md#ct022), [CT004](CASOS_DE_TESTE_FUNCIONAIS.md#ct004)). [LAC010](#lac010) e [LAC015](#lac015) mostram que a presença do recurso não garante o compromisso em situações de falha.

## 6. Catálogo de requisitos, regras e validações

Situação: **Definida**, quando o resultado é determinável; **Definida com ressalva**, quando há lacuna ou divergência associada; **Pendente**, quando falta decisão essencial.

| ID | Descrição verificável | Origem | Evidência | Conflitos e lacunas | Situação |
|---|---|---|---|---|---|
| RF001 | Cadastrar aluno e disponibilizar o registro para consulta | Documentado | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar | [LAC011](#lac011) | Definida com ressalva |
| RF002 | Listar alunos em ordem crescente de ID | Documentado | README.md: §9.12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: listarTodos<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: findAllByOrderByIdAsc | Nenhum identificado | Definida |
| RF003 | Consultar aluno por ID e informar inexistência | Documentado | README.md: §§9.14,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: buscarPorId | Nenhum identificado | Definida |
| RF004 | Atualizar campos editáveis preservando o ID | Documentado | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar | [LAC002](#lac002) | Definida com ressalva |
| RF005 | Excluir fisicamente aluno mediante confirmação na interface | Documentado | README.md: §§9.13,12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: excluir<br>src/main/resources/static/js/alunos.js: abrirExclusao | [LAC003](#lac003) | Definida com ressalva |
| RF006 | Pesquisar nome, CPF e e-mail e combinar com situação | Documentado | README.md: §§5,12<br>src/main/resources/static/js/alunos.js: renderizarListagem | Nenhum identificado | Definida |
| RN001 | CPF único, desconsiderando o próprio aluno na edição | Documentado | README.md: §9.2,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: verificarDuplicidadeNoCadastro/verificarDuplicidadeNaAtualizacao | [LAC009](#lac009) | Definida com ressalva |
| RN002 | E-mail único sem diferenciar caixa, ignorando o próprio ID | Documentado | README.md: §9.4,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarEmail<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: existsByEmailIgnoreCaseAndIdNot | [LAC009](#lac009) | Definida com ressalva |
| RN003 | Matrícula omitida ou nula no cadastro assume hoje; situação assume true | Documentado / extensão inferida para null | README.md: §9.8–9<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar | [LAC012](#lac012) | Definida com ressalva |
| RN004 | Na edição, matrícula e situação nulas/omitidas preservam os valores; telefone ausente é removido | Inferido da implementação | src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar/normalizarTelefone | Nenhum identificado | Definida |
| RN005 | Remover espaços externos do nome e converter o e-mail para minúsculas na gravação; telefone vazio vira null. Espaços externos no CPF e no e-mail enviados pela API são rejeitados antes da normalização | Inferido da implementação | src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar/atualizar/normalizarEmail/normalizarTelefone | [LAC001](#lac001) | Definida com ressalva |
| RN006 | Gerar ID e não permitir troca pela atualização | Documentado | README.md: §9.10<br>src/main/java/br/com/academia/alunos/entity/Aluno.java: id<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar | [LAC005](#lac005) | Definida com ressalva |
| RN007 | Dados duram durante o processo e desaparecem após reinício | Documentado | README.md: §13<br>src/main/resources/application.properties: datasource.url/ddl-auto | Nenhum identificado | Definida |
| VAL001 | Nome obrigatório, entre 3 e 100 caracteres | Documentado | README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario | [LAC001](#lac001), [LAC004](#lac004), [LAC007](#lac007) | Definida com ressalva |
| VAL002 | CPF obrigatório com exatamente 11 dígitos; sem validação de dígitos verificadores | Documentado | README.md: §9.2–3<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: cpf<br>src/main/resources/static/js/alunos.js: validarFormulario | [LAC004](#lac004) | Definida com ressalva |
| VAL003 | E-mail obrigatório e válido; máximo 254 caracteres | Documentado / limite inferido | README.md: §9.4<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: email<br>src/main/resources/static/index.html: email | [LAC004](#lac004), [LAC014](#lac014) | Definida com ressalva |
| VAL004 | Telefone opcional; vazio/null aceito; quando informado, 10 ou 11 dígitos | Documentado / null inferido | README.md: §9.5<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: telefone<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarTelefone | Nenhum identificado | Definida |
| VAL005 | Nascimento obrigatório e estritamente anterior a hoje | Documentado | README.md: §9.6<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataNascimento<br>src/main/resources/static/js/alunos.js: validarFormulario | [LAC012](#lac012) | Definida com ressalva |
| VAL006 | Plano obrigatório: MENSAL, TRIMESTRAL ou ANUAL | Documentado | README.md: §9.7<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: plano<br>src/main/java/br/com/academia/alunos/entity/PlanoAcademia.java | [LAC016](#lac016) | Definida com ressalva |
| VAL007 | Matrícula opcional, não futura | Documentado | README.md: §9.8<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataMatricula<br>src/main/resources/static/js/alunos.js: validarFormulario | [LAC012](#lac012) | Definida com ressalva |
| UI001 | Exibir confirmação nominal antes de excluir; cancelar não altera dados | Documentado | README.md: §§12,16<br>src/main/resources/static/js/alunos.js: abrirExclusao/tratarCliqueDelegado | Nenhum identificado | Definida |
| UI002 | Aplicar máscaras a CPF/telefone e enviar somente dígitos | Documentado | README.md: §9<br>src/main/resources/static/js/alunos.js: aplicarMascaraCpf/aplicarMascaraTelefone/obterDadosFormulario | Nenhum identificado | Definida |
| UI003 | Exibir progresso, bloquear reenvio por botão e notificar resultados | Documentado | README.md: §16<br>src/main/resources/static/js/alunos.js: salvarAluno/excluirAluno/alterarEstadoBotao | [LAC010](#lac010) | Definida com ressalva |
| UI004 | Distinguir carregamento, vazio, falha e ausência de resultados; permitir recuperação | Documentado | README.md: §16<br>src/main/resources/static/js/alunos.js: mostrarEstadoCarregando/mostrarEstadoErro/mostrarEstadoVazio/mostrarEstadoSemResultados | [LAC015](#lac015) | Definida com ressalva |
| UI005 | Atualizar cartões totais e contador de resultados filtrados | Documentado / detalhes inferidos | README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador | Nenhum identificado | Definida |
| UI006 | Preencher cadastro com hoje/ativo e edição com os valores atuais | Documentado / detalhes inferidos | README.md: §§12,16<br>src/main/resources/static/js/alunos.js: abrirCadastro/abrirEdicao | Nenhum identificado | Definida |
| UI007 | Fechar diálogos por controles/Esc e devolver foco ao acionador | Documentado | README.md: §12<br>src/main/resources/static/js/alunos.js: abrirDialogo/devolverFoco | [LAC002](#lac002), [LAC003](#lac003), [LAC010](#lac010) | Definida com ressalva |
| UI008 | Apresentar erros por campo, preservar dados e focar primeiro erro | Documentado / detalhes inferidos | README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro | Nenhum identificado | Definida |
| UI009 | Oferecer ajuda e navegação por teclado | Documentado | README.md: §12<br>src/main/resources/static/index.html: dialogo-ajuda/pular-conteudo | Nenhum identificado | Definida |
| UI010 | Normalizar caixa, acentos e pontuação na pesquisa local | Inferido da implementação | src/main/resources/static/js/alunos.js: normalizarBusca/renderizarListagem | Nenhum identificado | Definida |
| UI011 | Notificar em região acessível; sucesso desaparece e erro persiste até fechamento | Documentado / temporização inferida | README.md: §16<br>src/main/resources/static/js/alunos.js: notificar<br>src/main/resources/static/index.html: notificacoes | Nenhum identificado | Definida |
| UI012 | Preservar dados e ações na apresentação responsiva | Documentado | README.md: §15<br>src/main/resources/static/css/estilos.css: media max-width 680px | Nenhum identificado | Definida |
| UI013 | Apresentar os dados do aluno como texto literal, sem interpretar marcação HTML | Inferido da implementação | src/main/resources/static/js/alunos.js: criarCelula/criarLinhaAluno (textContent) | Nenhum identificado | Definida |
| API001 | POST /api/alunos retorna 201 e objeto com ID | Documentado | README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/dto/AlunoResponse.java | [LAC011](#lac011) | Definida com ressalva |
| API002 | GET /api/alunos retorna 200 e array ordenado | Documentado | README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: listarTodos | Nenhum identificado | Definida |
| API003 | GET /api/alunos/{id} retorna 200 ou 404 | Documentado | README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: buscarPorId<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarAlunoNaoEncontrado | Nenhum identificado | Definida |
| API004 | PUT /api/alunos/{id} retorna 200 com dados atualizados | Documentado | README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar | [LAC005](#lac005) | Definida com ressalva |
| API005 | DELETE /api/alunos/{id} retorna 204 sem corpo ou 404 se ausente | Documentado / 404 inferido | README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: excluir<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: buscarEntidade | Nenhum identificado | Definida |
| API006 | Validação retorna 400 com mapa por campo | Documentado | README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao | [LAC004](#lac004) | Definida com ressalva |
| API007 | Duplicidade retorna 409 sem efetivar operação | Documentado | README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarRecursoDuplicado/tratarIntegridade | [LAC009](#lac009), [LAC011](#lac011) | Definida com ressalva |
| API008 | Erro usa dataHora, codigoHttp, erro, mensagem, caminho e errosValidacao | Documentado | README.md: §10<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro | [LAC006](#lac006), [LAC008](#lac008) | Definida com ressalva |
| API009 | JSON ilegível retorna 400 e orientação sobre datas/opções | Inferido da implementação | src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarJsonInvalido | Nenhum identificado | Definida |
| API010 | Falha imprevista retorna 500 e mensagem genérica | Documentado | README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarErroInterno | [LAC008](#lac008) | Definida com ressalva |
| API011 | Contrato para ID não numérico, rota inexistente, método e tipo de mídia não suportados requer decisão | Pendente de definição | src/main/java/br/com/academia/alunos/controller/AlunoController.java: @PathVariable Long<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarErroInterno<br>README.md: §10 | [LAC006](#lac006) | Pendente |
| API012 | Na atualização, a validação do corpo precede a verificação de existência do ID | Inferido da implementação | src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar (@Valid antes da chamada ao serviço)<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar/buscarEntidade | Nenhum identificado | Definida |

### 6.1 Checklist de cobertura funcional

| Área | Item | Requisitos | Cenários | Situação |
|---|---|---|---|---|
| Inicialização | Página inicial e recursos | RF002, UI004 | [CT001](CASOS_DE_TESTE_FUNCIONAIS.md#ct001) | Coberto |
| Inicialização | Comunicação inicial com a API | RF002 | [CT001](CASOS_DE_TESTE_FUNCIONAIS.md#ct001) | Coberto |
| Inicialização | Carregamento | UI004 | [CT002](CASOS_DE_TESTE_FUNCIONAIS.md#ct002) | Coberto |
| Inicialização | Banco vazio | UI004 | [CT025](CASOS_DE_TESTE_FUNCIONAIS.md#ct025) | Coberto |
| Inicialização | Falha e recuperação | UI004 | [CT003](CASOS_DE_TESTE_FUNCIONAIS.md#ct003), [CT004](CASOS_DE_TESTE_FUNCIONAIS.md#ct004) | Coberto |
| Cadastro | Dados válidos completos | RF001 | [CT006](CASOS_DE_TESTE_FUNCIONAIS.md#ct006), [CT067](CASOS_DE_TESTE_FUNCIONAIS.md#ct067) | Coberto |
| Cadastro | Ausência de opcionais | RN003, VAL004 | [CT007](CASOS_DE_TESTE_FUNCIONAIS.md#ct007), [CT068](CASOS_DE_TESTE_FUNCIONAIS.md#ct068) | Coberto |
| Cadastro | Valores padrão | RN003, UI006 | [CT005](CASOS_DE_TESTE_FUNCIONAIS.md#ct005), [CT068](CASOS_DE_TESTE_FUNCIONAIS.md#ct068) | Coberto |
| Cadastro | Campos obrigatórios | VAL001–VAL003, VAL005, VAL006 | [CT010](CASOS_DE_TESTE_FUNCIONAIS.md#ct010), [CT081](CASOS_DE_TESTE_FUNCIONAIS.md#ct081), [CT082](CASOS_DE_TESTE_FUNCIONAIS.md#ct082) | Coberto |
| Cadastro | Tamanho e espaços no nome | VAL001, RN005 | [CT013](CASOS_DE_TESTE_FUNCIONAIS.md#ct013), [CT014](CASOS_DE_TESTE_FUNCIONAIS.md#ct014), [CT074](CASOS_DE_TESTE_FUNCIONAIS.md#ct074), [CT075](CASOS_DE_TESTE_FUNCIONAIS.md#ct075), [CT106](CASOS_DE_TESTE_FUNCIONAIS.md#ct106) | Coberto |
| Cadastro | CPF: formato, tamanho, máscara e duplicidade | VAL002, UI002, RN001 | [CT011](CASOS_DE_TESTE_FUNCIONAIS.md#ct011), [CT016](CASOS_DE_TESTE_FUNCIONAIS.md#ct016), [CT017](CASOS_DE_TESTE_FUNCIONAIS.md#ct017), [CT018](CASOS_DE_TESTE_FUNCIONAIS.md#ct018), [CT073](CASOS_DE_TESTE_FUNCIONAIS.md#ct073), [CT076](CASOS_DE_TESTE_FUNCIONAIS.md#ct076), [CT092](CASOS_DE_TESTE_FUNCIONAIS.md#ct092) | Coberto |
| Cadastro | E-mail: formato e duplicidade | VAL003, RN002 | [CT011](CASOS_DE_TESTE_FUNCIONAIS.md#ct011), [CT020](CASOS_DE_TESTE_FUNCIONAIS.md#ct020), [CT019](CASOS_DE_TESTE_FUNCIONAIS.md#ct019), [CT077](CASOS_DE_TESTE_FUNCIONAIS.md#ct077), [CT090](CASOS_DE_TESTE_FUNCIONAIS.md#ct090), [CT091](CASOS_DE_TESTE_FUNCIONAIS.md#ct091), [CT093](CASOS_DE_TESTE_FUNCIONAIS.md#ct093) | Coberto |
| Cadastro | Telefone | VAL004 | [CT007](CASOS_DE_TESTE_FUNCIONAIS.md#ct007), [CT011](CASOS_DE_TESTE_FUNCIONAIS.md#ct011), [CT078](CASOS_DE_TESTE_FUNCIONAIS.md#ct078), [CT083](CASOS_DE_TESTE_FUNCIONAIS.md#ct083) | Coberto |
| Cadastro | Datas e limites | VAL005, VAL007 | [CT012](CASOS_DE_TESTE_FUNCIONAIS.md#ct012), [CT079](CASOS_DE_TESTE_FUNCIONAIS.md#ct079), [CT080](CASOS_DE_TESTE_FUNCIONAIS.md#ct080), [CT084](CASOS_DE_TESTE_FUNCIONAIS.md#ct084) | Coberto; fuso pendente (LAC012) |
| Cadastro | Planos | VAL006 | [CT008](CASOS_DE_TESTE_FUNCIONAIS.md#ct008), [CT010](CASOS_DE_TESTE_FUNCIONAIS.md#ct010), [CT085](CASOS_DE_TESTE_FUNCIONAIS.md#ct085), [CT088](CASOS_DE_TESTE_FUNCIONAIS.md#ct088), [CT086](CASOS_DE_TESTE_FUNCIONAIS.md#ct086) | Coberto |
| Cadastro | Situação omitida ou informada | RN003 | [CT009](CASOS_DE_TESTE_FUNCIONAIS.md#ct009), [CT068](CASOS_DE_TESTE_FUNCIONAIS.md#ct068), [CT087](CASOS_DE_TESTE_FUNCIONAIS.md#ct087) | Coberto |
| Cadastro | Múltiplos envios | UI003 | [CT023](CASOS_DE_TESTE_FUNCIONAIS.md#ct023), [CT109](CASOS_DE_TESTE_FUNCIONAIS.md#ct109) | Coberto; disputa concorrente pendente (LAC009) |
| Cadastro | Mensagens e efeitos persistidos | API001, API006 | [CT067](CASOS_DE_TESTE_FUNCIONAIS.md#ct067), [CT089](CASOS_DE_TESTE_FUNCIONAIS.md#ct089) | Coberto |
| Listagem e consulta | Nenhum, um e vários | RF002 | [CT025](CASOS_DE_TESTE_FUNCIONAIS.md#ct025), [CT026](CASOS_DE_TESTE_FUNCIONAIS.md#ct026), [CT028](CASOS_DE_TESTE_FUNCIONAIS.md#ct028), [CT069](CASOS_DE_TESTE_FUNCIONAIS.md#ct069) | Coberto |
| Listagem e consulta | Ordenação | RF002 | [CT028](CASOS_DE_TESTE_FUNCIONAIS.md#ct028), [CT069](CASOS_DE_TESTE_FUNCIONAIS.md#ct069) | Coberto |
| Listagem e consulta | Campos, formatos e situação | RF002, UI013 | [CT026](CASOS_DE_TESTE_FUNCIONAIS.md#ct026), [CT027](CASOS_DE_TESTE_FUNCIONAIS.md#ct027) | Coberto |
| Listagem e consulta | Cartões e contadores | UI005 | [CT025](CASOS_DE_TESTE_FUNCIONAIS.md#ct025), [CT026](CASOS_DE_TESTE_FUNCIONAIS.md#ct026), [CT028](CASOS_DE_TESTE_FUNCIONAIS.md#ct028) | Coberto |
| Listagem e consulta | Consulta por ID existente, inexistente e inválido | RF003, API003, API011 | [CT029](CASOS_DE_TESTE_FUNCIONAIS.md#ct029), [CT030](CASOS_DE_TESTE_FUNCIONAIS.md#ct030), [CT101](CASOS_DE_TESTE_FUNCIONAIS.md#ct101) | Coberto; ID inválido pendente (LAC006) |
| Listagem e consulta | Estrutura da resposta | API001–API003 | [CT029](CASOS_DE_TESTE_FUNCIONAIS.md#ct029), [CT067](CASOS_DE_TESTE_FUNCIONAIS.md#ct067), [CT069](CASOS_DE_TESTE_FUNCIONAIS.md#ct069) | Coberto |
| Atualização | Abertura com dados existentes | UI006 | [CT031](CASOS_DE_TESTE_FUNCIONAIS.md#ct031) | Coberto |
| Atualização | Campos editáveis | RF004 | [CT032](CASOS_DE_TESTE_FUNCIONAIS.md#ct032), [CT033](CASOS_DE_TESTE_FUNCIONAIS.md#ct033), [CT034](CASOS_DE_TESTE_FUNCIONAIS.md#ct034), [CT070](CASOS_DE_TESTE_FUNCIONAIS.md#ct070) | Coberto |
| Atualização | Manutenção do identificador | RN006 | [CT070](CASOS_DE_TESTE_FUNCIONAIS.md#ct070), [CT100](CASOS_DE_TESTE_FUNCIONAIS.md#ct100) | Coberto |
| Atualização | Alteração da situação | RF004 | [CT032](CASOS_DE_TESTE_FUNCIONAIS.md#ct032), [CT035](CASOS_DE_TESTE_FUNCIONAIS.md#ct035), [CT087](CASOS_DE_TESTE_FUNCIONAIS.md#ct087) | Coberto |
| Atualização | Próprio CPF e e-mail | RN001, RN002 | [CT036](CASOS_DE_TESTE_FUNCIONAIS.md#ct036), [CT094](CASOS_DE_TESTE_FUNCIONAIS.md#ct094) | Coberto |
| Atualização | Conflito com outro aluno | RN001, RN002 | [CT037](CASOS_DE_TESTE_FUNCIONAIS.md#ct037), [CT038](CASOS_DE_TESTE_FUNCIONAIS.md#ct038), [CT092](CASOS_DE_TESTE_FUNCIONAIS.md#ct092), [CT093](CASOS_DE_TESTE_FUNCIONAIS.md#ct093) | Coberto |
| Atualização | Reaplicação das validações | VAL001–VAL007 | [CT039](CASOS_DE_TESTE_FUNCIONAIS.md#ct039), [CT075](CASOS_DE_TESTE_FUNCIONAIS.md#ct075), [CT081](CASOS_DE_TESTE_FUNCIONAIS.md#ct081) | Coberto |
| Atualização | Aluno inexistente | RF004 | [CT041](CASOS_DE_TESTE_FUNCIONAIS.md#ct041), [CT096](CASOS_DE_TESTE_FUNCIONAIS.md#ct096), [CT097](CASOS_DE_TESTE_FUNCIONAIS.md#ct097) | Coberto |
| Atualização | Cancelamento e fechamento | UI007 | [CT040](CASOS_DE_TESTE_FUNCIONAIS.md#ct040) | Coberto |
| Atualização | Atualização visual e persistência | UI005 | [CT032](CASOS_DE_TESTE_FUNCIONAIS.md#ct032), [CT054](CASOS_DE_TESTE_FUNCIONAIS.md#ct054), [CT113](CASOS_DE_TESTE_FUNCIONAIS.md#ct113) | Coberto |
| Exclusão | Confirmação e identificação | UI001 | [CT042](CASOS_DE_TESTE_FUNCIONAIS.md#ct042) | Coberto |
| Exclusão | Cancelamento e fechamento | UI001 | [CT043](CASOS_DE_TESTE_FUNCIONAIS.md#ct043) | Coberto |
| Exclusão | Exclusão confirmada | RF005 | [CT044](CASOS_DE_TESTE_FUNCIONAIS.md#ct044), [CT098](CASOS_DE_TESTE_FUNCIONAIS.md#ct098) | Coberto |
| Exclusão | Remoção conforme persistência | RF005 | [CT114](CASOS_DE_TESTE_FUNCIONAIS.md#ct114) | Coberto |
| Exclusão | Interface e contadores | UI005 | [CT044](CASOS_DE_TESTE_FUNCIONAIS.md#ct044), [CT045](CASOS_DE_TESTE_FUNCIONAIS.md#ct045) | Coberto |
| Exclusão | ID inexistente | API005 | [CT048](CASOS_DE_TESTE_FUNCIONAIS.md#ct048), [CT099](CASOS_DE_TESTE_FUNCIONAIS.md#ct099) | Coberto |
| Exclusão | Falha e recuperação | UI003 | [CT047](CASOS_DE_TESTE_FUNCIONAIS.md#ct047) | Coberto |
| Pesquisa e filtros | Critérios suportados; correspondência completa e parcial | RF006 | [CT049](CASOS_DE_TESTE_FUNCIONAIS.md#ct049) | Coberto |
| Pesquisa e filtros | Maiúsculas e minúsculas | UI010 | [CT049](CASOS_DE_TESTE_FUNCIONAIS.md#ct049) | Coberto |
| Pesquisa e filtros | Ausência de resultados | UI004 | [CT051](CASOS_DE_TESTE_FUNCIONAIS.md#ct051) | Coberto |
| Pesquisa e filtros | Limpeza e retorno ao estado inicial | RF006 | [CT052](CASOS_DE_TESTE_FUNCIONAIS.md#ct052), [CT053](CASOS_DE_TESTE_FUNCIONAIS.md#ct053) | Coberto |
| Pesquisa e filtros | Filtros e combinação | RF006 | [CT050](CASOS_DE_TESTE_FUNCIONAIS.md#ct050), [CT054](CASOS_DE_TESTE_FUNCIONAIS.md#ct054) | Coberto |
| Pesquisa e filtros | Contadores | UI005 | [CT049](CASOS_DE_TESTE_FUNCIONAIS.md#ct049), [CT050](CASOS_DE_TESTE_FUNCIONAIS.md#ct050) | Coberto |
| Interface | Abertura e fechamento; Cancelar e Esc | UI007 | [CT024](CASOS_DE_TESTE_FUNCIONAIS.md#ct024), [CT040](CASOS_DE_TESTE_FUNCIONAIS.md#ct040), [CT043](CASOS_DE_TESTE_FUNCIONAIS.md#ct043), [CT056](CASOS_DE_TESTE_FUNCIONAIS.md#ct056) | Coberto |
| Interface | Foco inicial e retorno | UI007 | [CT055](CASOS_DE_TESTE_FUNCIONAIS.md#ct055), [CT056](CASOS_DE_TESTE_FUNCIONAIS.md#ct056), [CT063](CASOS_DE_TESTE_FUNCIONAIS.md#ct063), [CT064](CASOS_DE_TESTE_FUNCIONAIS.md#ct064) | Coberto; destino após exclusão pendente (LAC003) |
| Interface | Navegação por teclado | UI009 | [CT057](CASOS_DE_TESTE_FUNCIONAIS.md#ct057), [CT058](CASOS_DE_TESTE_FUNCIONAIS.md#ct058) | Coberto |
| Interface | Bloqueio durante processamento | UI003 | [CT023](CASOS_DE_TESTE_FUNCIONAIS.md#ct023), [CT046](CASOS_DE_TESTE_FUNCIONAIS.md#ct046), [CT066](CASOS_DE_TESTE_FUNCIONAIS.md#ct066) | Coberto; fechamento durante envio pendente (LAC010) |
| Interface | Mensagens e erros por campo | UI008, UI011 | [CT060](CASOS_DE_TESTE_FUNCIONAIS.md#ct060), [CT061](CASOS_DE_TESTE_FUNCIONAIS.md#ct061), [CT020](CASOS_DE_TESTE_FUNCIONAIS.md#ct020) | Coberto |
| Interface | Preservação de dados após erro | UI008 | [CT018](CASOS_DE_TESTE_FUNCIONAIS.md#ct018), [CT022](CASOS_DE_TESTE_FUNCIONAIS.md#ct022), [CT060](CASOS_DE_TESTE_FUNCIONAIS.md#ct060) | Coberto |
| Interface | Estado vazio e sem resultados | UI004 | [CT025](CASOS_DE_TESTE_FUNCIONAIS.md#ct025), [CT051](CASOS_DE_TESTE_FUNCIONAIS.md#ct051) | Coberto |
| Interface | Ajuda | UI009 | [CT059](CASOS_DE_TESTE_FUNCIONAIS.md#ct059) | Coberto |
| API REST | Métodos, URLs, cabeçalhos e Content-Type | API001–API005, API011 | [CT067](CASOS_DE_TESTE_FUNCIONAIS.md#ct067), [CT102](CASOS_DE_TESTE_FUNCIONAIS.md#ct102), [CT103](CASOS_DE_TESTE_FUNCIONAIS.md#ct103) | Coberto; protocolo inválido pendente (LAC006) |
| API REST | Campos ausentes, nulos, vazios e inválidos | API006 | [CT081](CASOS_DE_TESTE_FUNCIONAIS.md#ct081), [CT082](CASOS_DE_TESTE_FUNCIONAIS.md#ct082), [CT075](CASOS_DE_TESTE_FUNCIONAIS.md#ct075) | Coberto |
| API REST | JSON malformado e tipos | API009 | [CT088](CASOS_DE_TESTE_FUNCIONAIS.md#ct088), [CT105](CASOS_DE_TESTE_FUNCIONAIS.md#ct105) | Coberto; conversões pendentes (LAC005) |
| API REST | Códigos HTTP | API001–API010 | [CT067](CASOS_DE_TESTE_FUNCIONAIS.md#ct067), [CT070](CASOS_DE_TESTE_FUNCIONAIS.md#ct070), [CT098](CASOS_DE_TESTE_FUNCIONAIS.md#ct098), [CT108](CASOS_DE_TESTE_FUNCIONAIS.md#ct108) | Coberto; disparo do 500 pendente (LAC008) |
| API REST | Estrutura dos erros e validações por campo | API006, API008 | [CT089](CASOS_DE_TESTE_FUNCIONAIS.md#ct089), [CT075](CASOS_DE_TESTE_FUNCIONAIS.md#ct075) | Coberto |
| API REST | Ausência de corpo | API005 | [CT098](CASOS_DE_TESTE_FUNCIONAIS.md#ct098) | Coberto |
| API REST | Consistência com consultas posteriores | API001, API004 | [CT067](CASOS_DE_TESTE_FUNCIONAIS.md#ct067), [CT070](CASOS_DE_TESTE_FUNCIONAIS.md#ct070) | Coberto |
| API REST | Ausência de alterações após falhas | API006, API007 | [CT092](CASOS_DE_TESTE_FUNCIONAIS.md#ct092), [CT116](CASOS_DE_TESTE_FUNCIONAIS.md#ct116) | Coberto |
| API REST | Paginação e filtros no servidor | — | — | Sem implementação e sem definição: não aplicável |
| API REST | Autenticação e autorização | — | — | Não aplicável: README §§6 e 21 declaram a ausência |
| Persistência | Recuperação após cadastro | RN007 | [CT112](CASOS_DE_TESTE_FUNCIONAIS.md#ct112), [CT113](CASOS_DE_TESTE_FUNCIONAIS.md#ct113) | Coberto |
| Persistência | Atualização persistida | RF004 | [CT113](CASOS_DE_TESTE_FUNCIONAIS.md#ct113) | Coberto |
| Persistência | Exclusão | RF005 | [CT114](CASOS_DE_TESTE_FUNCIONAIS.md#ct114) | Coberto |
| Persistência | Duração e reinicialização | RN007 | [CT115](CASOS_DE_TESTE_FUNCIONAIS.md#ct115) | Coberto |
| Persistência | Integridade após falha | RF004 | [CT116](CASOS_DE_TESTE_FUNCIONAIS.md#ct116) | Coberto; resposta perdida pendente em CT110 (LAC011) |

## 7. Análise de riscos e prioridades

Impacto: 1 (pequeno e recuperável) a 4 (perda de dados ou impedimento de operação essencial). Probabilidade: 1 (baixa exposição e complexidade) a 4 (alta exposição, múltiplas condições e dependências). Faixas: 12–16 Crítica; 8–11 Alta; 4–7 Média; 1–3 Baixa.

| ID | Risco | Impacto | Probabilidade | Nível | Prioridade | Justificativa | Cenários |
|---|---|---|---|---|---|---|---|
| R01 | Cadastro inválido ou bloqueio de matrícula | 3 | 3 | 9 | Alta | Validação duplicada e transformação de dados em UI/DTO/serviço. | 41 |
| R02 | Duplicidade de identificadores | 4 | 3 | 12 | Crítica | Consultas, normalização e restrições únicas precisam concordar. | 10 |
| R03 | Corrupção de atualização ou exclusão indevida | 4 | 3 | 12 | Crítica | Alterações persistentes, preenchimento prévio e confirmação. | 19 |
| R04 | Consulta/listagem indisponível ou incorreta | 3 | 2 | 6 | Média | Fluxo central com leitura e renderização; uso frequente estimado. | 8 |
| R05 | Pesquisa/contadores induzirem interpretação incorreta | 2 | 3 | 6 | Média | Normalização, interseção de filtros e estado local. | 6 |
| R06 | Falha sem recuperação ou reenvio involuntário | 3 | 3 | 9 | Alta | Fetch assíncrono, falhas de rede e estado dos controles. | 13 |
| R07 | Interação por teclado ou apresentação impedir operação | 2 | 2 | 4 | Média | Diálogos nativos e recriação dos elementos afetam foco. | 7 |
| R08 | Ajuda e notificações auxiliares inconsistentes | 1 | 2 | 2 | Baixa | Fluxos simples e impacto recuperável. | 2 |
| R09 | Contrato HTTP ambíguo ou erro mal classificado | 3 | 3 | 9 | Alta | Captura genérica de exceções e serialização. | 7 |
| R10 | Perda ou retenção inesperada de dados | 4 | 2 | 8 | Alta | H2 em memória e create-drop; reinício encerra os dados. | 3 |

Cada cenário declara um risco principal, cuja prioridade assume. As observações da exploração não alteraram as notas, para que a priorização não dependa de resultados parciais. Cadastros válidos têm prioridade Alta, e não Crítica: a falha é percebida e recuperável, ao contrário de duplicidade, corrupção ou exclusão indevida.

## 8. Estratégia de projeto

### 8.1 Interface, API e persistência

A interface é verificada quanto a interação, mensagens, foco e atualização visual. A API é verificada diretamente quanto a contrato e efeitos nos dados, porque a interface impede várias entradas e sempre envia `ativo`, `telefone` e `dataMatricula`. A persistência é verificada por consultas HTTP posteriores e, em dois casos, por SELECT no console H2. A validação da interface nunca é considerada proteção da API.

### 8.2 Seleção de variações

| Dado | Inválido inferior | Limite inferior | Nominal | Limite superior | Inválido superior | Outras partições |
|---|---|---|---|---|---|---|
| Nome (caracteres) | 2 | 3 | 18 | 100 | 101 | vazio, espaços, ausente, null, 2 após trim |
| CPF (dígitos) | 10 | 11 | — | 11 | 12 | letras, pontuado, espaços externos, dígitos repetidos |
| E-mail (caracteres) | — | — | 21 | 254 (e 253) | 255 | sem domínio, sem arroba, maiúsculas, espaços externos |
| Telefone (dígitos) | 9 | 10 | — | 11 | 12 | vazio, null, pontuado, letras, espaços |
| Nascimento | — | — | 1995-05-20 | ontem | hoje e amanhã | ausente, formato inválido, data inexistente |
| Matrícula | — | — | ontem | hoje | amanhã | ausente, null |

Reduções: campos inválidos variam um por vez, com um único caso de múltiplos erros por canal, para que a mensagem observada seja atribuível ao campo. POST e PUT repetem as partições porque compartilham o DTO, mas passam por métodos de serviço distintos. Combinações entre planos, situações e datas não foram cruzadas, pois nenhuma regra as relaciona. Limites de idade ou de antiguidade da matrícula não existem na documentação e não foram criados.

### 8.3 Tabelas de decisão

Unicidade na gravação:

| CPF de outro aluno | E-mail de outro aluno | Resultado | Cenários |
|---|---|---|---|
| Não | Não | Aceita | [CT067](CASOS_DE_TESTE_FUNCIONAIS.md#ct067), [CT070](CASOS_DE_TESTE_FUNCIONAIS.md#ct070) |
| Sim | Não | 409, mensagem de CPF | [CT092](CASOS_DE_TESTE_FUNCIONAIS.md#ct092) |
| Não | Sim | 409, mensagem de e-mail | [CT093](CASOS_DE_TESTE_FUNCIONAIS.md#ct093) |
| Sim | Sim | 409, mensagem de CPF (precedência inferida) | [CT095](CASOS_DE_TESTE_FUNCIONAIS.md#ct095) |
| Próprio aluno | Próprio aluno | Aceita | [CT094](CASOS_DE_TESTE_FUNCIONAIS.md#ct094) |

Opcionais ausentes ou nulos:

| Campo | Cadastro | Atualização | Cenários |
|---|---|---|---|
| telefone | null | null (remove) | [CT068](CASOS_DE_TESTE_FUNCIONAIS.md#ct068), [CT071](CASOS_DE_TESTE_FUNCIONAIS.md#ct071), [CT034](CASOS_DE_TESTE_FUNCIONAIS.md#ct034) |
| dataMatricula | data atual | preserva | [CT068](CASOS_DE_TESTE_FUNCIONAIS.md#ct068), [CT071](CASOS_DE_TESTE_FUNCIONAIS.md#ct071), [CT033](CASOS_DE_TESTE_FUNCIONAIS.md#ct033) |
| ativo | true | preserva | [CT068](CASOS_DE_TESTE_FUNCIONAIS.md#ct068), [CT071](CASOS_DE_TESTE_FUNCIONAIS.md#ct071) |

Precedência na atualização:

| Corpo válido | ID existe | Resultado | Cenário |
|---|---|---|---|
| Sim | Sim | 200 | [CT070](CASOS_DE_TESTE_FUNCIONAIS.md#ct070) |
| Sim | Não | 404 | [CT096](CASOS_DE_TESTE_FUNCIONAIS.md#ct096) |
| Não | Sim | 400 | [CT075](CASOS_DE_TESTE_FUNCIONAIS.md#ct075) |
| Não | Não | 400 (inferido) | [CT097](CASOS_DE_TESTE_FUNCIONAIS.md#ct097) |

### 8.4 Modelos de estados

| Objeto | Estado atual | Evento | Próximo estado | Cenários |
|---|---|---|---|---|
| Aluno | Inexistente | Cadastrar ativo / inativo | Ativo / Inativo | [CT006](CASOS_DE_TESTE_FUNCIONAIS.md#ct006), [CT009](CASOS_DE_TESTE_FUNCIONAIS.md#ct009) |
| Aluno | Ativo | Desmarcar situação | Inativo | [CT032](CASOS_DE_TESTE_FUNCIONAIS.md#ct032) |
| Aluno | Inativo | Marcar situação | Ativo | [CT035](CASOS_DE_TESTE_FUNCIONAIS.md#ct035) |
| Aluno | Ativo ou inativo | Excluir confirmado | Inexistente | [CT044](CASOS_DE_TESTE_FUNCIONAIS.md#ct044), [CT098](CASOS_DE_TESTE_FUNCIONAIS.md#ct098) |
| Aluno | Inexistente | Excluir | Inexistente, 404 | [CT099](CASOS_DE_TESTE_FUNCIONAIS.md#ct099) |
| Listagem | Carregando | Sucesso | Tabela ou vazio | [CT001](CASOS_DE_TESTE_FUNCIONAIS.md#ct001), [CT025](CASOS_DE_TESTE_FUNCIONAIS.md#ct025) |
| Listagem | Carregando | Falha | Erro | [CT003](CASOS_DE_TESTE_FUNCIONAIS.md#ct003) |
| Listagem | Erro | Tentar novamente | Carregando | [CT003](CASOS_DE_TESTE_FUNCIONAIS.md#ct003) |
| Listagem | Erro | Pesquisar | Erro | [CT004](CASOS_DE_TESTE_FUNCIONAIS.md#ct004) |
| Listagem | Tabela | Filtro sem correspondência | Sem resultados | [CT051](CASOS_DE_TESTE_FUNCIONAIS.md#ct051) |
| Formulário | Editando | Salvar inválido | Editando com erros | [CT010](CASOS_DE_TESTE_FUNCIONAIS.md#ct010), [CT060](CASOS_DE_TESTE_FUNCIONAIS.md#ct060) |
| Formulário | Editando | Salvar válido | Enviando | [CT023](CASOS_DE_TESTE_FUNCIONAIS.md#ct023) |
| Formulário | Enviando | Sucesso / falha | Fechado / Editando | [CT006](CASOS_DE_TESTE_FUNCIONAIS.md#ct006), [CT022](CASOS_DE_TESTE_FUNCIONAIS.md#ct022) |
| Formulário | Editando | Cancelar ou Esc | Fechado | [CT024](CASOS_DE_TESTE_FUNCIONAIS.md#ct024), [CT040](CASOS_DE_TESTE_FUNCIONAIS.md#ct040) |
| Formulário | Enviando | Esc | A definir | [CT066](CASOS_DE_TESTE_FUNCIONAIS.md#ct066) |

### 8.5 Tratamento das divergências

Com requisito documentado claro, o resultado esperado é o requisito, o cenário permanece especificado e a divergência é registrada. Sem definição essencial, o cenário recebe `@pendente_definicao`. O comportamento observado é informado, mas não convertido em requisito.

## 9. Ambiente, premissas e dependências

- JDK compatível com Java 21, Maven 3.9 ou superior, navegador com suporte a `dialog` e Fetch, cliente HTTP e DevTools.
- Instância local exclusiva; dados H2 existem somente durante o processo (`jdbc:h2:mem:academiadb`, `create-drop`).
- Navegador e processo Java no mesmo fuso; não atravessar a meia-noite durante uma instância.
- IDs são capturados, nunca presumidos.
- A compilação depende de dependências Maven já disponíveis ou de acesso à rede.

## 10. Matriz de rastreabilidade

### 10.1 Requisito para cenários

| Requisito | Riscos | Cenários | Instâncias | Maior prioridade | Lacunas | Ausência de cobertura |
|---|---|---|---|---|---|---|
| RF001 | R01, R06, R07, R10 | [CT005](CASOS_DE_TESTE_FUNCIONAIS.md#ct005), [CT006](CASOS_DE_TESTE_FUNCIONAIS.md#ct006), [CT007](CASOS_DE_TESTE_FUNCIONAIS.md#ct007), [CT008](CASOS_DE_TESTE_FUNCIONAIS.md#ct008), [CT009](CASOS_DE_TESTE_FUNCIONAIS.md#ct009), [CT021](CASOS_DE_TESTE_FUNCIONAIS.md#ct021), [CT022](CASOS_DE_TESTE_FUNCIONAIS.md#ct022), [CT024](CASOS_DE_TESTE_FUNCIONAIS.md#ct024), [CT057](CASOS_DE_TESTE_FUNCIONAIS.md#ct057), [CT065](CASOS_DE_TESTE_FUNCIONAIS.md#ct065), [CT067](CASOS_DE_TESTE_FUNCIONAIS.md#ct067), [CT068](CASOS_DE_TESTE_FUNCIONAIS.md#ct068), [CT110](CASOS_DE_TESTE_FUNCIONAIS.md#ct110)*, [CT112](CASOS_DE_TESTE_FUNCIONAIS.md#ct112), [CT113](CASOS_DE_TESTE_FUNCIONAIS.md#ct113) | 21 | Alta | [LAC011](#lac011) | Não há |
| RF002 | R04, R06, R10 | [CT001](CASOS_DE_TESTE_FUNCIONAIS.md#ct001), [CT003](CASOS_DE_TESTE_FUNCIONAIS.md#ct003), [CT025](CASOS_DE_TESTE_FUNCIONAIS.md#ct025), [CT026](CASOS_DE_TESTE_FUNCIONAIS.md#ct026), [CT027](CASOS_DE_TESTE_FUNCIONAIS.md#ct027), [CT028](CASOS_DE_TESTE_FUNCIONAIS.md#ct028), [CT069](CASOS_DE_TESTE_FUNCIONAIS.md#ct069), [CT112](CASOS_DE_TESTE_FUNCIONAIS.md#ct112) | 10 | Alta | — | Não há |
| RF003 | R04 | [CT029](CASOS_DE_TESTE_FUNCIONAIS.md#ct029), [CT030](CASOS_DE_TESTE_FUNCIONAIS.md#ct030) | 4 | Média | — | Não há |
| RF004 | R01, R02, R03, R05, R06, R07, R10 | [CT031](CASOS_DE_TESTE_FUNCIONAIS.md#ct031), [CT032](CASOS_DE_TESTE_FUNCIONAIS.md#ct032), [CT033](CASOS_DE_TESTE_FUNCIONAIS.md#ct033), [CT034](CASOS_DE_TESTE_FUNCIONAIS.md#ct034), [CT035](CASOS_DE_TESTE_FUNCIONAIS.md#ct035), [CT036](CASOS_DE_TESTE_FUNCIONAIS.md#ct036), [CT037](CASOS_DE_TESTE_FUNCIONAIS.md#ct037), [CT038](CASOS_DE_TESTE_FUNCIONAIS.md#ct038), [CT039](CASOS_DE_TESTE_FUNCIONAIS.md#ct039), [CT040](CASOS_DE_TESTE_FUNCIONAIS.md#ct040), [CT041](CASOS_DE_TESTE_FUNCIONAIS.md#ct041), [CT054](CASOS_DE_TESTE_FUNCIONAIS.md#ct054), [CT063](CASOS_DE_TESTE_FUNCIONAIS.md#ct063), [CT070](CASOS_DE_TESTE_FUNCIONAIS.md#ct070), [CT071](CASOS_DE_TESTE_FUNCIONAIS.md#ct071), [CT087](CASOS_DE_TESTE_FUNCIONAIS.md#ct087), [CT096](CASOS_DE_TESTE_FUNCIONAIS.md#ct096), [CT097](CASOS_DE_TESTE_FUNCIONAIS.md#ct097), [CT113](CASOS_DE_TESTE_FUNCIONAIS.md#ct113), [CT116](CASOS_DE_TESTE_FUNCIONAIS.md#ct116) | 39 | Crítica | [LAC002](#lac002) | Não há |
| RF005 | R03, R06, R07 | [CT042](CASOS_DE_TESTE_FUNCIONAIS.md#ct042), [CT043](CASOS_DE_TESTE_FUNCIONAIS.md#ct043), [CT044](CASOS_DE_TESTE_FUNCIONAIS.md#ct044), [CT045](CASOS_DE_TESTE_FUNCIONAIS.md#ct045), [CT046](CASOS_DE_TESTE_FUNCIONAIS.md#ct046), [CT047](CASOS_DE_TESTE_FUNCIONAIS.md#ct047), [CT048](CASOS_DE_TESTE_FUNCIONAIS.md#ct048), [CT064](CASOS_DE_TESTE_FUNCIONAIS.md#ct064)*, [CT098](CASOS_DE_TESTE_FUNCIONAIS.md#ct098), [CT099](CASOS_DE_TESTE_FUNCIONAIS.md#ct099), [CT114](CASOS_DE_TESTE_FUNCIONAIS.md#ct114) | 13 | Crítica | [LAC003](#lac003) | Não há |
| RF006 | R05 | [CT049](CASOS_DE_TESTE_FUNCIONAIS.md#ct049), [CT050](CASOS_DE_TESTE_FUNCIONAIS.md#ct050), [CT051](CASOS_DE_TESTE_FUNCIONAIS.md#ct051), [CT052](CASOS_DE_TESTE_FUNCIONAIS.md#ct052), [CT053](CASOS_DE_TESTE_FUNCIONAIS.md#ct053), [CT054](CASOS_DE_TESTE_FUNCIONAIS.md#ct054) | 19 | Média | — | Não há |
| RN001 | R02, R03 | [CT018](CASOS_DE_TESTE_FUNCIONAIS.md#ct018), [CT036](CASOS_DE_TESTE_FUNCIONAIS.md#ct036), [CT037](CASOS_DE_TESTE_FUNCIONAIS.md#ct037), [CT092](CASOS_DE_TESTE_FUNCIONAIS.md#ct092), [CT094](CASOS_DE_TESTE_FUNCIONAIS.md#ct094), [CT095](CASOS_DE_TESTE_FUNCIONAIS.md#ct095), [CT109](CASOS_DE_TESTE_FUNCIONAIS.md#ct109)*, [CT116](CASOS_DE_TESTE_FUNCIONAIS.md#ct116) | 9 | Crítica | [LAC009](#lac009) | Não há |
| RN002 | R02, R03 | [CT019](CASOS_DE_TESTE_FUNCIONAIS.md#ct019), [CT036](CASOS_DE_TESTE_FUNCIONAIS.md#ct036), [CT038](CASOS_DE_TESTE_FUNCIONAIS.md#ct038), [CT093](CASOS_DE_TESTE_FUNCIONAIS.md#ct093), [CT094](CASOS_DE_TESTE_FUNCIONAIS.md#ct094), [CT095](CASOS_DE_TESTE_FUNCIONAIS.md#ct095), [CT109](CASOS_DE_TESTE_FUNCIONAIS.md#ct109)*, [CT116](CASOS_DE_TESTE_FUNCIONAIS.md#ct116) | 11 | Crítica | [LAC009](#lac009) | Não há |
| RN003 | R01 | [CT007](CASOS_DE_TESTE_FUNCIONAIS.md#ct007), [CT068](CASOS_DE_TESTE_FUNCIONAIS.md#ct068), [CT087](CASOS_DE_TESTE_FUNCIONAIS.md#ct087), [CT111](CASOS_DE_TESTE_FUNCIONAIS.md#ct111)* | 9 | Alta | [LAC012](#lac012) | Não há |
| RN004 | R03 | [CT033](CASOS_DE_TESTE_FUNCIONAIS.md#ct033), [CT034](CASOS_DE_TESTE_FUNCIONAIS.md#ct034), [CT071](CASOS_DE_TESTE_FUNCIONAIS.md#ct071) | 4 | Crítica | — | Não há |
| RN005 | R01, R03 | [CT014](CASOS_DE_TESTE_FUNCIONAIS.md#ct014), [CT034](CASOS_DE_TESTE_FUNCIONAIS.md#ct034), [CT072](CASOS_DE_TESTE_FUNCIONAIS.md#ct072), [CT106](CASOS_DE_TESTE_FUNCIONAIS.md#ct106) | 7 | Crítica | [LAC001](#lac001) | Não há |
| RN006 | R01, R03 | [CT067](CASOS_DE_TESTE_FUNCIONAIS.md#ct067), [CT070](CASOS_DE_TESTE_FUNCIONAIS.md#ct070), [CT100](CASOS_DE_TESTE_FUNCIONAIS.md#ct100) | 3 | Crítica | [LAC005](#lac005) | Não há |
| RN007 | R03, R10 | [CT112](CASOS_DE_TESTE_FUNCIONAIS.md#ct112), [CT113](CASOS_DE_TESTE_FUNCIONAIS.md#ct113), [CT114](CASOS_DE_TESTE_FUNCIONAIS.md#ct114), [CT115](CASOS_DE_TESTE_FUNCIONAIS.md#ct115) | 4 | Crítica | — | Não há |
| VAL001 | R01 | [CT010](CASOS_DE_TESTE_FUNCIONAIS.md#ct010), [CT013](CASOS_DE_TESTE_FUNCIONAIS.md#ct013), [CT014](CASOS_DE_TESTE_FUNCIONAIS.md#ct014), [CT015](CASOS_DE_TESTE_FUNCIONAIS.md#ct015), [CT039](CASOS_DE_TESTE_FUNCIONAIS.md#ct039), [CT060](CASOS_DE_TESTE_FUNCIONAIS.md#ct060), [CT074](CASOS_DE_TESTE_FUNCIONAIS.md#ct074), [CT075](CASOS_DE_TESTE_FUNCIONAIS.md#ct075), [CT081](CASOS_DE_TESTE_FUNCIONAIS.md#ct081), [CT082](CASOS_DE_TESTE_FUNCIONAIS.md#ct082), [CT089](CASOS_DE_TESTE_FUNCIONAIS.md#ct089), [CT106](CASOS_DE_TESTE_FUNCIONAIS.md#ct106), [CT107](CASOS_DE_TESTE_FUNCIONAIS.md#ct107)* | 69 | Alta | [LAC001](#lac001), [LAC004](#lac004), [LAC007](#lac007) | Não há |
| VAL002 | R01 | [CT010](CASOS_DE_TESTE_FUNCIONAIS.md#ct010), [CT011](CASOS_DE_TESTE_FUNCIONAIS.md#ct011), [CT017](CASOS_DE_TESTE_FUNCIONAIS.md#ct017), [CT039](CASOS_DE_TESTE_FUNCIONAIS.md#ct039), [CT060](CASOS_DE_TESTE_FUNCIONAIS.md#ct060), [CT073](CASOS_DE_TESTE_FUNCIONAIS.md#ct073), [CT076](CASOS_DE_TESTE_FUNCIONAIS.md#ct076), [CT081](CASOS_DE_TESTE_FUNCIONAIS.md#ct081), [CT082](CASOS_DE_TESTE_FUNCIONAIS.md#ct082), [CT089](CASOS_DE_TESTE_FUNCIONAIS.md#ct089) | 63 | Alta | [LAC004](#lac004) | Não há |
| VAL003 | R01 | [CT010](CASOS_DE_TESTE_FUNCIONAIS.md#ct010), [CT011](CASOS_DE_TESTE_FUNCIONAIS.md#ct011), [CT015](CASOS_DE_TESTE_FUNCIONAIS.md#ct015), [CT020](CASOS_DE_TESTE_FUNCIONAIS.md#ct020), [CT039](CASOS_DE_TESTE_FUNCIONAIS.md#ct039), [CT077](CASOS_DE_TESTE_FUNCIONAIS.md#ct077), [CT081](CASOS_DE_TESTE_FUNCIONAIS.md#ct081), [CT082](CASOS_DE_TESTE_FUNCIONAIS.md#ct082), [CT090](CASOS_DE_TESTE_FUNCIONAIS.md#ct090), [CT091](CASOS_DE_TESTE_FUNCIONAIS.md#ct091) | 65 | Alta | [LAC004](#lac004), [LAC014](#lac014) | Não há |
| VAL004 | R01, R03 | [CT007](CASOS_DE_TESTE_FUNCIONAIS.md#ct007), [CT011](CASOS_DE_TESTE_FUNCIONAIS.md#ct011), [CT034](CASOS_DE_TESTE_FUNCIONAIS.md#ct034), [CT039](CASOS_DE_TESTE_FUNCIONAIS.md#ct039), [CT068](CASOS_DE_TESTE_FUNCIONAIS.md#ct068), [CT078](CASOS_DE_TESTE_FUNCIONAIS.md#ct078), [CT083](CASOS_DE_TESTE_FUNCIONAIS.md#ct083) | 33 | Crítica | — | Não há |
| VAL005 | R01 | [CT010](CASOS_DE_TESTE_FUNCIONAIS.md#ct010), [CT012](CASOS_DE_TESTE_FUNCIONAIS.md#ct012), [CT039](CASOS_DE_TESTE_FUNCIONAIS.md#ct039), [CT079](CASOS_DE_TESTE_FUNCIONAIS.md#ct079), [CT081](CASOS_DE_TESTE_FUNCIONAIS.md#ct081), [CT084](CASOS_DE_TESTE_FUNCIONAIS.md#ct084), [CT089](CASOS_DE_TESTE_FUNCIONAIS.md#ct089), [CT111](CASOS_DE_TESTE_FUNCIONAIS.md#ct111)* | 50 | Alta | [LAC012](#lac012) | Não há |
| VAL006 | R01, R09 | [CT008](CASOS_DE_TESTE_FUNCIONAIS.md#ct008), [CT010](CASOS_DE_TESTE_FUNCIONAIS.md#ct010), [CT039](CASOS_DE_TESTE_FUNCIONAIS.md#ct039), [CT081](CASOS_DE_TESTE_FUNCIONAIS.md#ct081), [CT085](CASOS_DE_TESTE_FUNCIONAIS.md#ct085), [CT086](CASOS_DE_TESTE_FUNCIONAIS.md#ct086), [CT088](CASOS_DE_TESTE_FUNCIONAIS.md#ct088) | 68 | Alta | [LAC016](#lac016) | Não há |
| VAL007 | R01 | [CT007](CASOS_DE_TESTE_FUNCIONAIS.md#ct007), [CT012](CASOS_DE_TESTE_FUNCIONAIS.md#ct012), [CT039](CASOS_DE_TESTE_FUNCIONAIS.md#ct039), [CT080](CASOS_DE_TESTE_FUNCIONAIS.md#ct080), [CT084](CASOS_DE_TESTE_FUNCIONAIS.md#ct084), [CT111](CASOS_DE_TESTE_FUNCIONAIS.md#ct111)* | 22 | Alta | [LAC012](#lac012) | Não há |
| UI001 | R03 | [CT042](CASOS_DE_TESTE_FUNCIONAIS.md#ct042), [CT043](CASOS_DE_TESTE_FUNCIONAIS.md#ct043), [CT044](CASOS_DE_TESTE_FUNCIONAIS.md#ct044) | 5 | Crítica | — | Não há |
| UI002 | R01 | [CT006](CASOS_DE_TESTE_FUNCIONAIS.md#ct006), [CT015](CASOS_DE_TESTE_FUNCIONAIS.md#ct015), [CT016](CASOS_DE_TESTE_FUNCIONAIS.md#ct016), [CT017](CASOS_DE_TESTE_FUNCIONAIS.md#ct017) | 9 | Alta | — | Não há |
| UI003 | R01, R03, R06 | [CT006](CASOS_DE_TESTE_FUNCIONAIS.md#ct006), [CT022](CASOS_DE_TESTE_FUNCIONAIS.md#ct022), [CT023](CASOS_DE_TESTE_FUNCIONAIS.md#ct023), [CT032](CASOS_DE_TESTE_FUNCIONAIS.md#ct032), [CT044](CASOS_DE_TESTE_FUNCIONAIS.md#ct044), [CT046](CASOS_DE_TESTE_FUNCIONAIS.md#ct046), [CT047](CASOS_DE_TESTE_FUNCIONAIS.md#ct047), [CT065](CASOS_DE_TESTE_FUNCIONAIS.md#ct065), [CT066](CASOS_DE_TESTE_FUNCIONAIS.md#ct066)* | 16 | Crítica | [LAC010](#lac010) | Não há |
| UI004 | R03, R04, R05, R06 | [CT001](CASOS_DE_TESTE_FUNCIONAIS.md#ct001), [CT002](CASOS_DE_TESTE_FUNCIONAIS.md#ct002), [CT003](CASOS_DE_TESTE_FUNCIONAIS.md#ct003), [CT004](CASOS_DE_TESTE_FUNCIONAIS.md#ct004), [CT025](CASOS_DE_TESTE_FUNCIONAIS.md#ct025), [CT045](CASOS_DE_TESTE_FUNCIONAIS.md#ct045), [CT051](CASOS_DE_TESTE_FUNCIONAIS.md#ct051), [CT065](CASOS_DE_TESTE_FUNCIONAIS.md#ct065) | 10 | Crítica | [LAC015](#lac015) | Não há |
| UI005 | R01, R03, R04, R05 | [CT006](CASOS_DE_TESTE_FUNCIONAIS.md#ct006), [CT009](CASOS_DE_TESTE_FUNCIONAIS.md#ct009), [CT025](CASOS_DE_TESTE_FUNCIONAIS.md#ct025), [CT026](CASOS_DE_TESTE_FUNCIONAIS.md#ct026), [CT028](CASOS_DE_TESTE_FUNCIONAIS.md#ct028), [CT032](CASOS_DE_TESTE_FUNCIONAIS.md#ct032), [CT035](CASOS_DE_TESTE_FUNCIONAIS.md#ct035), [CT044](CASOS_DE_TESTE_FUNCIONAIS.md#ct044), [CT045](CASOS_DE_TESTE_FUNCIONAIS.md#ct045), [CT049](CASOS_DE_TESTE_FUNCIONAIS.md#ct049), [CT050](CASOS_DE_TESTE_FUNCIONAIS.md#ct050), [CT051](CASOS_DE_TESTE_FUNCIONAIS.md#ct051), [CT052](CASOS_DE_TESTE_FUNCIONAIS.md#ct052), [CT054](CASOS_DE_TESTE_FUNCIONAIS.md#ct054) | 34 | Crítica | — | Não há |
| UI006 | R01, R03 | [CT005](CASOS_DE_TESTE_FUNCIONAIS.md#ct005), [CT009](CASOS_DE_TESTE_FUNCIONAIS.md#ct009), [CT031](CASOS_DE_TESTE_FUNCIONAIS.md#ct031), [CT035](CASOS_DE_TESTE_FUNCIONAIS.md#ct035) | 4 | Crítica | — | Não há |
| UI007 | R01, R03, R06, R07 | [CT024](CASOS_DE_TESTE_FUNCIONAIS.md#ct024), [CT040](CASOS_DE_TESTE_FUNCIONAIS.md#ct040), [CT043](CASOS_DE_TESTE_FUNCIONAIS.md#ct043), [CT055](CASOS_DE_TESTE_FUNCIONAIS.md#ct055), [CT056](CASOS_DE_TESTE_FUNCIONAIS.md#ct056), [CT063](CASOS_DE_TESTE_FUNCIONAIS.md#ct063), [CT064](CASOS_DE_TESTE_FUNCIONAIS.md#ct064)*, [CT066](CASOS_DE_TESTE_FUNCIONAIS.md#ct066)* | 20 | Crítica | [LAC002](#lac002), [LAC003](#lac003), [LAC010](#lac010) | Não há |
| UI008 | R01, R02, R06 | [CT010](CASOS_DE_TESTE_FUNCIONAIS.md#ct010), [CT011](CASOS_DE_TESTE_FUNCIONAIS.md#ct011), [CT012](CASOS_DE_TESTE_FUNCIONAIS.md#ct012), [CT014](CASOS_DE_TESTE_FUNCIONAIS.md#ct014), [CT018](CASOS_DE_TESTE_FUNCIONAIS.md#ct018), [CT019](CASOS_DE_TESTE_FUNCIONAIS.md#ct019), [CT020](CASOS_DE_TESTE_FUNCIONAIS.md#ct020), [CT021](CASOS_DE_TESTE_FUNCIONAIS.md#ct021), [CT022](CASOS_DE_TESTE_FUNCIONAIS.md#ct022), [CT037](CASOS_DE_TESTE_FUNCIONAIS.md#ct037), [CT038](CASOS_DE_TESTE_FUNCIONAIS.md#ct038), [CT039](CASOS_DE_TESTE_FUNCIONAIS.md#ct039), [CT041](CASOS_DE_TESTE_FUNCIONAIS.md#ct041), [CT060](CASOS_DE_TESTE_FUNCIONAIS.md#ct060) | 30 | Crítica | — | Não há |
| UI009 | R07, R08 | [CT055](CASOS_DE_TESTE_FUNCIONAIS.md#ct055), [CT057](CASOS_DE_TESTE_FUNCIONAIS.md#ct057), [CT058](CASOS_DE_TESTE_FUNCIONAIS.md#ct058), [CT059](CASOS_DE_TESTE_FUNCIONAIS.md#ct059) | 7 | Média | — | Não há |
| UI010 | R05, R06 | [CT004](CASOS_DE_TESTE_FUNCIONAIS.md#ct004), [CT049](CASOS_DE_TESTE_FUNCIONAIS.md#ct049), [CT050](CASOS_DE_TESTE_FUNCIONAIS.md#ct050), [CT052](CASOS_DE_TESTE_FUNCIONAIS.md#ct052) | 15 | Alta | — | Não há |
| UI011 | R08 | [CT061](CASOS_DE_TESTE_FUNCIONAIS.md#ct061) | 2 | Baixa | — | Não há |
| UI012 | R07 | [CT062](CASOS_DE_TESTE_FUNCIONAIS.md#ct062) | 3 | Média | — | Não há |
| UI013 | R04 | [CT027](CASOS_DE_TESTE_FUNCIONAIS.md#ct027) | 1 | Média | — | Não há |
| API001 | R01, R06 | [CT067](CASOS_DE_TESTE_FUNCIONAIS.md#ct067), [CT068](CASOS_DE_TESTE_FUNCIONAIS.md#ct068), [CT072](CASOS_DE_TESTE_FUNCIONAIS.md#ct072), [CT073](CASOS_DE_TESTE_FUNCIONAIS.md#ct073), [CT074](CASOS_DE_TESTE_FUNCIONAIS.md#ct074), [CT083](CASOS_DE_TESTE_FUNCIONAIS.md#ct083), [CT084](CASOS_DE_TESTE_FUNCIONAIS.md#ct084), [CT085](CASOS_DE_TESTE_FUNCIONAIS.md#ct085), [CT087](CASOS_DE_TESTE_FUNCIONAIS.md#ct087), [CT090](CASOS_DE_TESTE_FUNCIONAIS.md#ct090), [CT110](CASOS_DE_TESTE_FUNCIONAIS.md#ct110)* | 45 | Alta | [LAC011](#lac011) | Não há |
| API002 | R04 | [CT069](CASOS_DE_TESTE_FUNCIONAIS.md#ct069) | 3 | Média | — | Não há |
| API003 | R04 | [CT029](CASOS_DE_TESTE_FUNCIONAIS.md#ct029), [CT030](CASOS_DE_TESTE_FUNCIONAIS.md#ct030) | 4 | Média | — | Não há |
| API004 | R01, R02, R03 | [CT070](CASOS_DE_TESTE_FUNCIONAIS.md#ct070), [CT071](CASOS_DE_TESTE_FUNCIONAIS.md#ct071), [CT072](CASOS_DE_TESTE_FUNCIONAIS.md#ct072), [CT073](CASOS_DE_TESTE_FUNCIONAIS.md#ct073), [CT074](CASOS_DE_TESTE_FUNCIONAIS.md#ct074), [CT083](CASOS_DE_TESTE_FUNCIONAIS.md#ct083), [CT084](CASOS_DE_TESTE_FUNCIONAIS.md#ct084), [CT085](CASOS_DE_TESTE_FUNCIONAIS.md#ct085), [CT087](CASOS_DE_TESTE_FUNCIONAIS.md#ct087), [CT090](CASOS_DE_TESTE_FUNCIONAIS.md#ct090), [CT094](CASOS_DE_TESTE_FUNCIONAIS.md#ct094), [CT100](CASOS_DE_TESTE_FUNCIONAIS.md#ct100) | 45 | Crítica | [LAC005](#lac005) | Não há |
| API005 | R03, R06 | [CT048](CASOS_DE_TESTE_FUNCIONAIS.md#ct048), [CT098](CASOS_DE_TESTE_FUNCIONAIS.md#ct098), [CT099](CASOS_DE_TESTE_FUNCIONAIS.md#ct099) | 3 | Crítica | — | Não há |
| API006 | R01, R03 | [CT020](CASOS_DE_TESTE_FUNCIONAIS.md#ct020), [CT075](CASOS_DE_TESTE_FUNCIONAIS.md#ct075), [CT076](CASOS_DE_TESTE_FUNCIONAIS.md#ct076), [CT077](CASOS_DE_TESTE_FUNCIONAIS.md#ct077), [CT078](CASOS_DE_TESTE_FUNCIONAIS.md#ct078), [CT079](CASOS_DE_TESTE_FUNCIONAIS.md#ct079), [CT080](CASOS_DE_TESTE_FUNCIONAIS.md#ct080), [CT081](CASOS_DE_TESTE_FUNCIONAIS.md#ct081), [CT082](CASOS_DE_TESTE_FUNCIONAIS.md#ct082), [CT089](CASOS_DE_TESTE_FUNCIONAIS.md#ct089), [CT091](CASOS_DE_TESTE_FUNCIONAIS.md#ct091), [CT097](CASOS_DE_TESTE_FUNCIONAIS.md#ct097), [CT106](CASOS_DE_TESTE_FUNCIONAIS.md#ct106) | 75 | Crítica | [LAC004](#lac004) | Não há |
| API007 | R02, R06 | [CT092](CASOS_DE_TESTE_FUNCIONAIS.md#ct092), [CT093](CASOS_DE_TESTE_FUNCIONAIS.md#ct093), [CT095](CASOS_DE_TESTE_FUNCIONAIS.md#ct095), [CT109](CASOS_DE_TESTE_FUNCIONAIS.md#ct109)*, [CT110](CASOS_DE_TESTE_FUNCIONAIS.md#ct110)* | 9 | Crítica | [LAC009](#lac009), [LAC011](#lac011) | Não há |
| API008 | R01, R02, R03, R04, R06, R09 | [CT029](CASOS_DE_TESTE_FUNCIONAIS.md#ct029), [CT030](CASOS_DE_TESTE_FUNCIONAIS.md#ct030), [CT041](CASOS_DE_TESTE_FUNCIONAIS.md#ct041), [CT075](CASOS_DE_TESTE_FUNCIONAIS.md#ct075), [CT076](CASOS_DE_TESTE_FUNCIONAIS.md#ct076), [CT077](CASOS_DE_TESTE_FUNCIONAIS.md#ct077), [CT078](CASOS_DE_TESTE_FUNCIONAIS.md#ct078), [CT079](CASOS_DE_TESTE_FUNCIONAIS.md#ct079), [CT080](CASOS_DE_TESTE_FUNCIONAIS.md#ct080), [CT081](CASOS_DE_TESTE_FUNCIONAIS.md#ct081), [CT086](CASOS_DE_TESTE_FUNCIONAIS.md#ct086), [CT088](CASOS_DE_TESTE_FUNCIONAIS.md#ct088), [CT089](CASOS_DE_TESTE_FUNCIONAIS.md#ct089), [CT092](CASOS_DE_TESTE_FUNCIONAIS.md#ct092), [CT093](CASOS_DE_TESTE_FUNCIONAIS.md#ct093), [CT096](CASOS_DE_TESTE_FUNCIONAIS.md#ct096), [CT099](CASOS_DE_TESTE_FUNCIONAIS.md#ct099), [CT101](CASOS_DE_TESTE_FUNCIONAIS.md#ct101)*, [CT102](CASOS_DE_TESTE_FUNCIONAIS.md#ct102)*, [CT103](CASOS_DE_TESTE_FUNCIONAIS.md#ct103)*, [CT104](CASOS_DE_TESTE_FUNCIONAIS.md#ct104)*, [CT105](CASOS_DE_TESTE_FUNCIONAIS.md#ct105)*, [CT108](CASOS_DE_TESTE_FUNCIONAIS.md#ct108)* | 108 | Crítica | [LAC006](#lac006), [LAC008](#lac008) | Não há |
| API009 | R09 | [CT088](CASOS_DE_TESTE_FUNCIONAIS.md#ct088), [CT105](CASOS_DE_TESTE_FUNCIONAIS.md#ct105)* | 26 | Alta | — | Não há |
| API010 | R09 | [CT108](CASOS_DE_TESTE_FUNCIONAIS.md#ct108)* | 1 | Alta | [LAC008](#lac008) | Todos os cenários pendentes |
| API011 | R09 | [CT101](CASOS_DE_TESTE_FUNCIONAIS.md#ct101)*, [CT102](CASOS_DE_TESTE_FUNCIONAIS.md#ct102)*, [CT103](CASOS_DE_TESTE_FUNCIONAIS.md#ct103)*, [CT104](CASOS_DE_TESTE_FUNCIONAIS.md#ct104)* | 7 | Alta | [LAC006](#lac006) | Todos os cenários pendentes |
| API012 | R03 | [CT097](CASOS_DE_TESTE_FUNCIONAIS.md#ct097) | 1 | Crítica | — | Não há |

\* Cenário pendente de definição. Os exemplos de um esquema são identificados por CTxxx-EXnn. Um cenário pode aparecer em várias linhas; a coluna de instâncias não deve ser somada.

### 10.2 Cenário para requisitos

| Cenário | Requisitos | Risco | Prioridade | Instâncias | Situação |
|---|---|---|---|---|---|
| [CT001](CASOS_DE_TESTE_FUNCIONAIS.md#ct001) | RF002, UI004 | R04 | Média | 1 | Especificado |
| [CT002](CASOS_DE_TESTE_FUNCIONAIS.md#ct002) | UI004 | R06 | Alta | 1 | Especificado |
| [CT003](CASOS_DE_TESTE_FUNCIONAIS.md#ct003) | RF002, UI004 | R06 | Alta | 1 | Especificado |
| [CT004](CASOS_DE_TESTE_FUNCIONAIS.md#ct004) | UI004, UI010 | R06 | Alta | 1 | Especificado |
| [CT005](CASOS_DE_TESTE_FUNCIONAIS.md#ct005) | RF001, UI006 | R01 | Alta | 1 | Especificado |
| [CT006](CASOS_DE_TESTE_FUNCIONAIS.md#ct006) | RF001, UI002, UI003, UI005 | R01 | Alta | 1 | Especificado |
| [CT007](CASOS_DE_TESTE_FUNCIONAIS.md#ct007) | RF001, RN003, VAL004, VAL007 | R01 | Alta | 1 | Especificado |
| [CT008](CASOS_DE_TESTE_FUNCIONAIS.md#ct008) | RF001, VAL006 | R01 | Alta | 3 | Especificado |
| [CT009](CASOS_DE_TESTE_FUNCIONAIS.md#ct009) | RF001, UI005, UI006 | R01 | Alta | 1 | Especificado |
| [CT010](CASOS_DE_TESTE_FUNCIONAIS.md#ct010) | VAL001, VAL002, VAL003, VAL005, VAL006, UI008 | R01 | Alta | 6 | Especificado |
| [CT011](CASOS_DE_TESTE_FUNCIONAIS.md#ct011) | VAL002, VAL003, VAL004, UI008 | R01 | Alta | 3 | Especificado |
| [CT012](CASOS_DE_TESTE_FUNCIONAIS.md#ct012) | VAL005, VAL007, UI008 | R01 | Alta | 3 | Especificado |
| [CT013](CASOS_DE_TESTE_FUNCIONAIS.md#ct013) | VAL001 | R01 | Alta | 3 | Especificado |
| [CT014](CASOS_DE_TESTE_FUNCIONAIS.md#ct014) | VAL001, RN005, UI008 | R01 | Alta | 2 | Especificado |
| [CT015](CASOS_DE_TESTE_FUNCIONAIS.md#ct015) | VAL001, VAL003, UI002 | R01 | Alta | 4 | Especificado |
| [CT016](CASOS_DE_TESTE_FUNCIONAIS.md#ct016) | UI002 | R01 | Alta | 3 | Especificado |
| [CT017](CASOS_DE_TESTE_FUNCIONAIS.md#ct017) | UI002, VAL002 | R01 | Alta | 1 | Especificado |
| [CT018](CASOS_DE_TESTE_FUNCIONAIS.md#ct018) | RN001, UI008 | R02 | Crítica | 1 | Especificado |
| [CT019](CASOS_DE_TESTE_FUNCIONAIS.md#ct019) | RN002, UI008 | R02 | Crítica | 1 | Especificado |
| [CT020](CASOS_DE_TESTE_FUNCIONAIS.md#ct020) | UI008, VAL003, API006 | R01 | Alta | 1 | Especificado |
| [CT021](CASOS_DE_TESTE_FUNCIONAIS.md#ct021) | RF001, UI008 | R06 | Alta | 1 | Especificado |
| [CT022](CASOS_DE_TESTE_FUNCIONAIS.md#ct022) | RF001, UI003, UI008 | R06 | Alta | 1 | Especificado |
| [CT023](CASOS_DE_TESTE_FUNCIONAIS.md#ct023) | UI003 | R06 | Alta | 1 | Especificado |
| [CT024](CASOS_DE_TESTE_FUNCIONAIS.md#ct024) | RF001, UI007 | R01 | Alta | 3 | Especificado |
| [CT025](CASOS_DE_TESTE_FUNCIONAIS.md#ct025) | RF002, UI004, UI005 | R04 | Média | 1 | Especificado |
| [CT026](CASOS_DE_TESTE_FUNCIONAIS.md#ct026) | RF002, UI005 | R04 | Média | 1 | Especificado |
| [CT027](CASOS_DE_TESTE_FUNCIONAIS.md#ct027) | RF002, UI013 | R04 | Média | 1 | Especificado |
| [CT028](CASOS_DE_TESTE_FUNCIONAIS.md#ct028) | RF002, UI005 | R04 | Média | 1 | Especificado |
| [CT029](CASOS_DE_TESTE_FUNCIONAIS.md#ct029) | RF003, API003, API008 | R04 | Média | 1 | Especificado |
| [CT030](CASOS_DE_TESTE_FUNCIONAIS.md#ct030) | RF003, API003, API008 | R04 | Média | 3 | Especificado |
| [CT031](CASOS_DE_TESTE_FUNCIONAIS.md#ct031) | RF004, UI006 | R03 | Crítica | 1 | Especificado |
| [CT032](CASOS_DE_TESTE_FUNCIONAIS.md#ct032) | RF004, UI003, UI005 | R03 | Crítica | 8 | Especificado |
| [CT033](CASOS_DE_TESTE_FUNCIONAIS.md#ct033) | RF004, RN004 | R03 | Crítica | 1 | Especificado |
| [CT034](CASOS_DE_TESTE_FUNCIONAIS.md#ct034) | RF004, RN004, RN005, VAL004 | R03 | Crítica | 1 | Especificado |
| [CT035](CASOS_DE_TESTE_FUNCIONAIS.md#ct035) | RF004, UI005, UI006 | R03 | Crítica | 1 | Especificado |
| [CT036](CASOS_DE_TESTE_FUNCIONAIS.md#ct036) | RF004, RN001, RN002 | R02 | Crítica | 1 | Especificado |
| [CT037](CASOS_DE_TESTE_FUNCIONAIS.md#ct037) | RF004, RN001, UI008 | R02 | Crítica | 1 | Especificado |
| [CT038](CASOS_DE_TESTE_FUNCIONAIS.md#ct038) | RF004, RN002, UI008 | R02 | Crítica | 1 | Especificado |
| [CT039](CASOS_DE_TESTE_FUNCIONAIS.md#ct039) | RF004, VAL001, VAL002, VAL003, VAL004, VAL005, VAL006, VAL007, UI008 | R01 | Alta | 7 | Especificado |
| [CT040](CASOS_DE_TESTE_FUNCIONAIS.md#ct040) | RF004, UI007 | R03 | Crítica | 3 | Especificado |
| [CT041](CASOS_DE_TESTE_FUNCIONAIS.md#ct041) | RF004, UI008, API008 | R06 | Alta | 1 | Especificado |
| [CT042](CASOS_DE_TESTE_FUNCIONAIS.md#ct042) | RF005, UI001 | R03 | Crítica | 1 | Especificado |
| [CT043](CASOS_DE_TESTE_FUNCIONAIS.md#ct043) | RF005, UI001, UI007 | R03 | Crítica | 3 | Especificado |
| [CT044](CASOS_DE_TESTE_FUNCIONAIS.md#ct044) | RF005, UI001, UI003, UI005 | R03 | Crítica | 1 | Especificado |
| [CT045](CASOS_DE_TESTE_FUNCIONAIS.md#ct045) | RF005, UI004, UI005 | R03 | Crítica | 1 | Especificado |
| [CT046](CASOS_DE_TESTE_FUNCIONAIS.md#ct046) | UI003, RF005 | R06 | Alta | 1 | Especificado |
| [CT047](CASOS_DE_TESTE_FUNCIONAIS.md#ct047) | RF005, UI003 | R06 | Alta | 1 | Especificado |
| [CT048](CASOS_DE_TESTE_FUNCIONAIS.md#ct048) | RF005, API005 | R06 | Alta | 1 | Especificado |
| [CT049](CASOS_DE_TESTE_FUNCIONAIS.md#ct049) | RF006, UI010, UI005 | R05 | Média | 8 | Especificado |
| [CT050](CASOS_DE_TESTE_FUNCIONAIS.md#ct050) | RF006, UI005, UI010 | R05 | Média | 5 | Especificado |
| [CT051](CASOS_DE_TESTE_FUNCIONAIS.md#ct051) | RF006, UI004, UI005 | R05 | Média | 3 | Especificado |
| [CT052](CASOS_DE_TESTE_FUNCIONAIS.md#ct052) | RF006, UI005, UI010 | R05 | Média | 1 | Especificado |
| [CT053](CASOS_DE_TESTE_FUNCIONAIS.md#ct053) | RF006 | R05 | Média | 1 | Especificado |
| [CT054](CASOS_DE_TESTE_FUNCIONAIS.md#ct054) | RF004, RF006, UI005 | R05 | Média | 1 | Especificado |
| [CT055](CASOS_DE_TESTE_FUNCIONAIS.md#ct055) | UI007, UI009 | R07 | Média | 4 | Especificado |
| [CT056](CASOS_DE_TESTE_FUNCIONAIS.md#ct056) | UI007 | R07 | Média | 4 | Especificado |
| [CT057](CASOS_DE_TESTE_FUNCIONAIS.md#ct057) | RF001, UI009 | R07 | Média | 1 | Especificado |
| [CT058](CASOS_DE_TESTE_FUNCIONAIS.md#ct058) | UI009 | R07 | Média | 1 | Especificado |
| [CT059](CASOS_DE_TESTE_FUNCIONAIS.md#ct059) | UI009 | R08 | Baixa | 1 | Especificado |
| [CT060](CASOS_DE_TESTE_FUNCIONAIS.md#ct060) | UI008, VAL001, VAL002 | R01 | Alta | 1 | Especificado |
| [CT061](CASOS_DE_TESTE_FUNCIONAIS.md#ct061) | UI011 | R08 | Baixa | 2 | Especificado |
| [CT062](CASOS_DE_TESTE_FUNCIONAIS.md#ct062) | UI012 | R07 | Média | 3 | Especificado |
| [CT063](CASOS_DE_TESTE_FUNCIONAIS.md#ct063) | UI007, RF004 | R07 | Média | 1 | Especificado |
| [CT064](CASOS_DE_TESTE_FUNCIONAIS.md#ct064) | UI007, RF005 | R07 | Média | 1 | Pendente (LAC003) |
| [CT065](CASOS_DE_TESTE_FUNCIONAIS.md#ct065) | RF001, UI003, UI004 | R06 | Alta | 1 | Especificado |
| [CT066](CASOS_DE_TESTE_FUNCIONAIS.md#ct066) | UI007, UI003 | R06 | Alta | 1 | Pendente (LAC010) |
| [CT067](CASOS_DE_TESTE_FUNCIONAIS.md#ct067) | RF001, RN006, API001 | R01 | Alta | 1 | Especificado |
| [CT068](CASOS_DE_TESTE_FUNCIONAIS.md#ct068) | RF001, RN003, VAL004, API001 | R01 | Alta | 3 | Especificado |
| [CT069](CASOS_DE_TESTE_FUNCIONAIS.md#ct069) | RF002, API002 | R04 | Média | 3 | Especificado |
| [CT070](CASOS_DE_TESTE_FUNCIONAIS.md#ct070) | RF004, RN006, API004 | R03 | Crítica | 1 | Especificado |
| [CT071](CASOS_DE_TESTE_FUNCIONAIS.md#ct071) | RF004, RN004, API004 | R03 | Crítica | 2 | Especificado |
| [CT072](CASOS_DE_TESTE_FUNCIONAIS.md#ct072) | RN005, API001, API004 | R01 | Alta | 2 | Especificado |
| [CT073](CASOS_DE_TESTE_FUNCIONAIS.md#ct073) | VAL002, API001, API004 | R01 | Alta | 2 | Especificado |
| [CT074](CASOS_DE_TESTE_FUNCIONAIS.md#ct074) | VAL001, API001, API004 | R01 | Alta | 6 | Especificado |
| [CT075](CASOS_DE_TESTE_FUNCIONAIS.md#ct075) | VAL001, API006, API008 | R01 | Alta | 4 | Especificado |
| [CT076](CASOS_DE_TESTE_FUNCIONAIS.md#ct076) | VAL002, API006, API008 | R01 | Alta | 10 | Especificado |
| [CT077](CASOS_DE_TESTE_FUNCIONAIS.md#ct077) | VAL003, API006, API008 | R01 | Alta | 6 | Especificado |
| [CT078](CASOS_DE_TESTE_FUNCIONAIS.md#ct078) | VAL004, API006, API008 | R01 | Alta | 10 | Especificado |
| [CT079](CASOS_DE_TESTE_FUNCIONAIS.md#ct079) | VAL005, API006, API008 | R01 | Alta | 4 | Especificado |
| [CT080](CASOS_DE_TESTE_FUNCIONAIS.md#ct080) | VAL007, API006, API008 | R01 | Alta | 2 | Especificado |
| [CT081](CASOS_DE_TESTE_FUNCIONAIS.md#ct081) | VAL001, VAL002, VAL003, VAL005, VAL006, API006, API008 | R01 | Alta | 20 | Especificado |
| [CT082](CASOS_DE_TESTE_FUNCIONAIS.md#ct082) | VAL001, VAL002, VAL003, API006 | R01 | Alta | 12 | Especificado |
| [CT083](CASOS_DE_TESTE_FUNCIONAIS.md#ct083) | VAL004, API001, API004 | R01 | Alta | 8 | Especificado |
| [CT084](CASOS_DE_TESTE_FUNCIONAIS.md#ct084) | VAL005, VAL007, API001, API004 | R01 | Alta | 8 | Especificado |
| [CT085](CASOS_DE_TESTE_FUNCIONAIS.md#ct085) | VAL006, API001, API004 | R01 | Alta | 6 | Especificado |
| [CT086](CASOS_DE_TESTE_FUNCIONAIS.md#ct086) | VAL006, API008 | R01 | Alta | 4 | Especificado |
| [CT087](CASOS_DE_TESTE_FUNCIONAIS.md#ct087) | RN003, RF004, API001, API004 | R01 | Alta | 4 | Especificado |
| [CT088](CASOS_DE_TESTE_FUNCIONAIS.md#ct088) | VAL006, API009, API008 | R09 | Alta | 22 | Especificado |
| [CT089](CASOS_DE_TESTE_FUNCIONAIS.md#ct089) | API006, API008, VAL001, VAL002, VAL005 | R01 | Alta | 1 | Especificado |
| [CT090](CASOS_DE_TESTE_FUNCIONAIS.md#ct090) | VAL003, API001, API004 | R01 | Alta | 4 | Especificado |
| [CT091](CASOS_DE_TESTE_FUNCIONAIS.md#ct091) | VAL003, API006 | R01 | Alta | 2 | Especificado |
| [CT092](CASOS_DE_TESTE_FUNCIONAIS.md#ct092) | RN001, API007, API008 | R02 | Crítica | 2 | Especificado |
| [CT093](CASOS_DE_TESTE_FUNCIONAIS.md#ct093) | RN002, API007, API008 | R02 | Crítica | 4 | Especificado |
| [CT094](CASOS_DE_TESTE_FUNCIONAIS.md#ct094) | RN001, RN002, API004 | R02 | Crítica | 1 | Especificado |
| [CT095](CASOS_DE_TESTE_FUNCIONAIS.md#ct095) | RN001, RN002, API007 | R02 | Crítica | 1 | Especificado |
| [CT096](CASOS_DE_TESTE_FUNCIONAIS.md#ct096) | RF004, API008 | R03 | Crítica | 1 | Especificado |
| [CT097](CASOS_DE_TESTE_FUNCIONAIS.md#ct097) | RF004, API006, API012 | R03 | Crítica | 1 | Especificado |
| [CT098](CASOS_DE_TESTE_FUNCIONAIS.md#ct098) | RF005, API005 | R03 | Crítica | 1 | Especificado |
| [CT099](CASOS_DE_TESTE_FUNCIONAIS.md#ct099) | RF005, API005, API008 | R03 | Crítica | 1 | Especificado |
| [CT100](CASOS_DE_TESTE_FUNCIONAIS.md#ct100) | RN006, API004 | R03 | Crítica | 1 | Especificado |
| [CT101](CASOS_DE_TESTE_FUNCIONAIS.md#ct101) | API011, API008 | R09 | Alta | 3 | Pendente (LAC006) |
| [CT102](CASOS_DE_TESTE_FUNCIONAIS.md#ct102) | API011, API008 | R09 | Alta | 1 | Pendente (LAC006) |
| [CT103](CASOS_DE_TESTE_FUNCIONAIS.md#ct103) | API011, API008 | R09 | Alta | 1 | Pendente (LAC006) |
| [CT104](CASOS_DE_TESTE_FUNCIONAIS.md#ct104) | API011, API008 | R09 | Alta | 2 | Pendente (LAC006) |
| [CT105](CASOS_DE_TESTE_FUNCIONAIS.md#ct105) | API009, API008 | R09 | Alta | 4 | Pendente (LAC005) |
| [CT106](CASOS_DE_TESTE_FUNCIONAIS.md#ct106) | VAL001, RN005, API006 | R01 | Alta | 2 | Especificado |
| [CT107](CASOS_DE_TESTE_FUNCIONAIS.md#ct107) | VAL001 | R01 | Alta | 1 | Pendente (LAC007) |
| [CT108](CASOS_DE_TESTE_FUNCIONAIS.md#ct108) | API010, API008 | R09 | Alta | 1 | Pendente (LAC008) |
| [CT109](CASOS_DE_TESTE_FUNCIONAIS.md#ct109) | RN001, RN002, API007 | R02 | Crítica | 1 | Pendente (LAC009) |
| [CT110](CASOS_DE_TESTE_FUNCIONAIS.md#ct110) | RF001, API001, API007 | R06 | Alta | 1 | Pendente (LAC011) |
| [CT111](CASOS_DE_TESTE_FUNCIONAIS.md#ct111) | VAL005, VAL007, RN003 | R01 | Alta | 1 | Pendente (LAC012) |
| [CT112](CASOS_DE_TESTE_FUNCIONAIS.md#ct112) | RN007, RF001, RF002 | R10 | Alta | 1 | Especificado |
| [CT113](CASOS_DE_TESTE_FUNCIONAIS.md#ct113) | RF001, RF004, RN007 | R10 | Alta | 1 | Especificado |
| [CT114](CASOS_DE_TESTE_FUNCIONAIS.md#ct114) | RF005, RN007 | R03 | Crítica | 1 | Especificado |
| [CT115](CASOS_DE_TESTE_FUNCIONAIS.md#ct115) | RN007 | R10 | Alta | 1 | Especificado |
| [CT116](CASOS_DE_TESTE_FUNCIONAIS.md#ct116) | RF004, RN001, RN002 | R03 | Crítica | 1 | Especificado |

### 10.3 Migração da versão 1.0

| Versão 1.0 | Versão 2.0 |
|---|---|
| CT001 | [CT001](CASOS_DE_TESTE_FUNCIONAIS.md#ct001), [CT028](CASOS_DE_TESTE_FUNCIONAIS.md#ct028) |
| CT002 | [CT025](CASOS_DE_TESTE_FUNCIONAIS.md#ct025) |
| CT003 | [CT003](CASOS_DE_TESTE_FUNCIONAIS.md#ct003) |
| CT004 | [CT006](CASOS_DE_TESTE_FUNCIONAIS.md#ct006), [CT067](CASOS_DE_TESTE_FUNCIONAIS.md#ct067) |
| CT005 | [CT007](CASOS_DE_TESTE_FUNCIONAIS.md#ct007), [CT068](CASOS_DE_TESTE_FUNCIONAIS.md#ct068) |
| CT006 | [CT014](CASOS_DE_TESTE_FUNCIONAIS.md#ct014), [CT075](CASOS_DE_TESTE_FUNCIONAIS.md#ct075) |
| CT007 | [CT011](CASOS_DE_TESTE_FUNCIONAIS.md#ct011), [CT017](CASOS_DE_TESTE_FUNCIONAIS.md#ct017), [CT076](CASOS_DE_TESTE_FUNCIONAIS.md#ct076) |
| CT008 | [CT011](CASOS_DE_TESTE_FUNCIONAIS.md#ct011), [CT077](CASOS_DE_TESTE_FUNCIONAIS.md#ct077) |
| CT009 | [CT018](CASOS_DE_TESTE_FUNCIONAIS.md#ct018), [CT092](CASOS_DE_TESTE_FUNCIONAIS.md#ct092) |
| CT010 | [CT019](CASOS_DE_TESTE_FUNCIONAIS.md#ct019), [CT093](CASOS_DE_TESTE_FUNCIONAIS.md#ct093) |
| CT011 | [CT032](CASOS_DE_TESTE_FUNCIONAIS.md#ct032), [CT070](CASOS_DE_TESTE_FUNCIONAIS.md#ct070) |
| CT012 | [CT036](CASOS_DE_TESTE_FUNCIONAIS.md#ct036), [CT094](CASOS_DE_TESTE_FUNCIONAIS.md#ct094) |
| CT013 | [CT037](CASOS_DE_TESTE_FUNCIONAIS.md#ct037), [CT092](CASOS_DE_TESTE_FUNCIONAIS.md#ct092) |
| CT014 | [CT042](CASOS_DE_TESTE_FUNCIONAIS.md#ct042), [CT044](CASOS_DE_TESTE_FUNCIONAIS.md#ct044) |
| CT015 | [CT043](CASOS_DE_TESTE_FUNCIONAIS.md#ct043) |
| CT016 | [CT049](CASOS_DE_TESTE_FUNCIONAIS.md#ct049), [CT050](CASOS_DE_TESTE_FUNCIONAIS.md#ct050), [CT051](CASOS_DE_TESTE_FUNCIONAIS.md#ct051), [CT052](CASOS_DE_TESTE_FUNCIONAIS.md#ct052) |
| CT017 | [CT029](CASOS_DE_TESTE_FUNCIONAIS.md#ct029) |
| CT018 | [CT030](CASOS_DE_TESTE_FUNCIONAIS.md#ct030) |
| CT019 | [CT098](CASOS_DE_TESTE_FUNCIONAIS.md#ct098) |
| CT020 | [CT112](CASOS_DE_TESTE_FUNCIONAIS.md#ct112), [CT113](CASOS_DE_TESTE_FUNCIONAIS.md#ct113), [CT115](CASOS_DE_TESTE_FUNCIONAIS.md#ct115) |

## 11. Resumo da cobertura planejada

Refere-se à cobertura da especificação; não é cobertura de código nem resultado de execução.

| Medida | Total | Especificados | Pendentes |
|---|---|---|---|
| Cenários e esquemas escritos | 116 | 104 | 12 |
| — dos quais Cenário | 67 | 58 | 9 |
| — dos quais Esquema do Cenário | 49 | 46 | 3 |
| Instâncias de teste | 311 | 293 | 18 |

| Funcionalidade | Cenários | Instâncias | Pendentes (cenários) |
|---|---|---|---|
| Inicialização | 4 | 4 | 0 |
| Cadastro | 20 | 41 | 0 |
| Listagem | 4 | 4 | 0 |
| Consulta | 2 | 4 | 0 |
| Atualização | 11 | 26 | 0 |
| Exclusão | 7 | 9 | 0 |
| Pesquisa e filtros | 6 | 19 | 0 |
| Interface e interação | 12 | 21 | 2 |
| API REST | 45 | 178 | 10 |
| Persistência | 5 | 5 | 0 |
| **Total** | 116 | 311 | 12 |

| Prioridade | Cenários | Instâncias | Pendentes (cenários) |
|---|---|---|---|
| Crítica | 29 | 45 | 1 |
| Alta | 64 | 217 | 10 |
| Média | 21 | 46 | 1 |
| Baixa | 2 | 3 | 0 |
| **Total** | 116 | 311 | 12 |

| Classificação principal | Cenários | Instâncias | Pendentes (cenários) |
|---|---|---|---|
| Positivo | 33 | 70 | 0 |
| Negativo | 32 | 127 | 6 |
| Limite | 16 | 66 | 1 |
| Exceção | 10 | 10 | 3 |
| Transição | 25 | 38 | 2 |
| **Total** | 116 | 311 | 12 |

| Canal | Cenários | Instâncias | Pendentes (cenários) |
|---|---|---|---|
| Interface gráfica | 65 | 125 | 2 |
| API REST | 49 | 184 | 10 |
| API REST + console H2 | 2 | 2 | 0 |
| **Total** | 116 | 311 | 12 |

Requisitos sem cenário: nenhum dos 45 itens do catálogo. API010 e API011 possuem somente cenários pendentes.

## 12. Lacunas, ambiguidades, possíveis inconsistências e riscos

<a id="lac001"></a>

### LAC001 — Nome é validado antes de trim

| Campo | Informação |
|---|---|
| Fontes e evidências | README §9.1; AlunoRequest.nomeCompleto; AlunoService.cadastrar/atualizar |
| Descrição e impacto | Observado: POST e PUT com " An " gravaram "An", com dois caracteres. Na direção oposta, 100 caracteres com um espaço externo são recusados pela API, embora a interface remova o espaço. |
| Requisitos afetados | VAL001, RN005 |
| Cenários afetados | [CT014](CASOS_DE_TESTE_FUNCIONAIS.md#ct014), [CT106](CASOS_DE_TESTE_FUNCIONAIS.md#ct106) |
| Cenários pendentes por esta lacuna | Nenhum |
| Pergunta objetiva | Aplicar o limite antes ou depois da normalização? Qual mensagem deve ser usada após trim? |

<a id="lac002"></a>

### LAC002 — Retorno de foco antes da reconstrução da tabela

| Campo | Informação |
|---|---|
| Fontes e evidências | README §12; alunos.js: salvarAluno/devolverFoco/renderizarListagem |
| Descrição e impacto | Observado em Chrome headless: após salvar a edição, o foco ficou em BODY. Diverge do retorno documentado. |
| Requisitos afetados | UI007, RF004 |
| Cenários afetados | [CT063](CASOS_DE_TESTE_FUNCIONAIS.md#ct063) |
| Cenários pendentes por esta lacuna | Nenhum |
| Pergunta objetiva | Como preservar o foco no controle equivalente após replaceChildren? |

<a id="lac003"></a>

### LAC003 — Acionador desaparece depois da exclusão

| Campo | Informação |
|---|---|
| Fontes e evidências | README §12; alunos.js: excluirAluno/devolverFoco |
| Descrição e impacto | Observado: foco em BODY após a exclusão. Não existe destino alternativo definido; cenário pendente. |
| Requisitos afetados | UI007, RF005 |
| Cenários afetados | [CT064](CASOS_DE_TESTE_FUNCIONAIS.md#ct064) |
| Cenários pendentes por esta lacuna | [CT064](CASOS_DE_TESTE_FUNCIONAIS.md#ct064) |
| Pergunta objetiva | Após excluir a linha, qual controle deve receber foco? |

<a id="lac004"></a>

### LAC004 — Mais de uma restrição falha para o mesmo campo

| Campo | Informação |
|---|---|
| Fontes e evidências | AlunoRequest; GlobalExceptionHandler.tratarValidacao: putIfAbsent |
| Descrição e impacto | Observado: mensagens diferentes para envios idênticos repetidos. Os testes usam conjunto explícito de mensagens. |
| Requisitos afetados | VAL001, VAL002, VAL003, API006 |
| Cenários afetados | [CT082](CASOS_DE_TESTE_FUNCIONAIS.md#ct082) |
| Cenários pendentes por esta lacuna | Nenhum |
| Pergunta objetiva | Qual mensagem tem precedência por campo quando duas restrições falham? |

<a id="lac005"></a>

### LAC005 — Propriedades JSON desconhecidas e coerções

| Campo | Informação |
|---|---|
| Fontes e evidências | AlunoRequest não contém id; nenhuma configuração Jackson explícita |
| Descrição e impacto | Observado: propriedades extras são ignoradas; texto, número e data com hora são convertidos sem erro. Não há contrato de produto. |
| Requisitos afetados | RN006, API004 |
| Cenários afetados | [CT100](CASOS_DE_TESTE_FUNCIONAIS.md#ct100), [CT105](CASOS_DE_TESTE_FUNCIONAIS.md#ct105) |
| Cenários pendentes por esta lacuna | [CT105](CASOS_DE_TESTE_FUNCIONAIS.md#ct105) |
| Pergunta objetiva | Campos extras devem ser ignorados ou rejeitados? Quais coerções escalares são permitidas? |

<a id="lac006"></a>

### LAC006 — Erros de protocolo capturados como falha genérica

| Campo | Informação |
|---|---|
| Fontes e evidências | AlunoController: Long; GlobalExceptionHandler.tratarErroInterno |
| Descrição e impacto | Observado: 500 com mensagem genérica para ID não numérico, método e mídia não suportados, PUT na coleção, rotas inexistentes e /favicon.ico. O código adequado não está definido na documentação. |
| Requisitos afetados | API011, API008 |
| Cenários afetados | [CT101](CASOS_DE_TESTE_FUNCIONAIS.md#ct101), [CT102](CASOS_DE_TESTE_FUNCIONAIS.md#ct102), [CT103](CASOS_DE_TESTE_FUNCIONAIS.md#ct103), [CT104](CASOS_DE_TESTE_FUNCIONAIS.md#ct104) |
| Cenários pendentes por esta lacuna | [CT101](CASOS_DE_TESTE_FUNCIONAIS.md#ct101), [CT102](CASOS_DE_TESTE_FUNCIONAIS.md#ct102), [CT103](CASOS_DE_TESTE_FUNCIONAIS.md#ct103), [CT104](CASOS_DE_TESTE_FUNCIONAIS.md#ct104) |
| Pergunta objetiva | Quais códigos, cabeçalhos e mensagens devem valer para cada entrada de protocolo inválida? |

<a id="lac007"></a>

### LAC007 — Ajuda pede nome e sobrenome; regra objetiva exige tamanho

| Campo | Informação |
|---|---|
| Fontes e evidências | index.html: ajuda-nome; README §9.1 |
| Descrição e impacto | Casos de tamanho seguem README; exigência adicional de sobrenome não foi resolvida. |
| Requisitos afetados | VAL001 |
| Cenários afetados | [CT013](CASOS_DE_TESTE_FUNCIONAIS.md#ct013), [CT074](CASOS_DE_TESTE_FUNCIONAIS.md#ct074), [CT107](CASOS_DE_TESTE_FUNCIONAIS.md#ct107) |
| Cenários pendentes por esta lacuna | [CT107](CASOS_DE_TESTE_FUNCIONAIS.md#ct107) |
| Pergunta objetiva | Um nome único com três caracteres é aceito? Ajustar regra ou ajuda? |

<a id="lac008"></a>

### LAC008 — Disparo manual de falha interna não definido

| Campo | Informação |
|---|---|
| Fontes e evidências | GlobalExceptionHandler.tratarErroInterno |
| Descrição e impacto | Contrato 500 conhecido, sem disparo aprovado. GET /api/alunos/abc o reproduz hoje, mas depende de LAC006. |
| Requisitos afetados | API010, API008 |
| Cenários afetados | [CT108](CASOS_DE_TESTE_FUNCIONAIS.md#ct108) |
| Cenários pendentes por esta lacuna | [CT108](CASOS_DE_TESTE_FUNCIONAIS.md#ct108) |
| Pergunta objetiva | Qual condição local reproduzível pode gerar falha interna sem alterar produção? |

<a id="lac009"></a>

### LAC009 — Disputa concorrente não reproduzível com cliques sequenciais

| Campo | Informação |
|---|---|
| Fontes e evidências | Aluno: unique; AlunoService: exists; GlobalExceptionHandler.tratarIntegridade |
| Descrição e impacto | Observado com cliente sincronizado: 60 disputas, sempre 201 + 409 e um registro. Falta procedimento manual autorizado. |
| Requisitos afetados | RN001, RN002, API007 |
| Cenários afetados | [CT109](CASOS_DE_TESTE_FUNCIONAIS.md#ct109) |
| Cenários pendentes por esta lacuna | [CT109](CASOS_DE_TESTE_FUNCIONAIS.md#ct109) |
| Pergunta objetiva | Qual mecanismo autorizado sincronizará duas gravações para verificar a disputa? |

<a id="lac010"></a>

### LAC010 — Fechamento do diálogo durante envio

| Campo | Informação |
|---|---|
| Fontes e evidências | alunos.js: tratarCliqueDelegado/salvarAluno/excluirAluno |
| Descrição e impacto | Observado: fechar não cancela o envio; a resposta tardia fechou um segundo diálogo aberto depois, com risco de descartar o que foi digitado nele. |
| Requisitos afetados | UI003, UI007 |
| Cenários afetados | [CT066](CASOS_DE_TESTE_FUNCIONAIS.md#ct066) |
| Cenários pendentes por esta lacuna | [CT066](CASOS_DE_TESTE_FUNCIONAIS.md#ct066) |
| Pergunta objetiva | Fechamento deve ser bloqueado, cancelar envio ou apenas ocultar o diálogo? |

<a id="lac011"></a>

### LAC011 — Resposta perdida depois da gravação

| Campo | Informação |
|---|---|
| Fontes e evidências | alunos.js: requisitarJson; AlunoService.cadastrar |
| Descrição e impacto | Falha anterior ao envio é coberta. Falha após commit e reconciliação não têm preparação nem contrato; reenvio pode retornar 409. |
| Requisitos afetados | RF001, API001, API007 |
| Cenários afetados | [CT022](CASOS_DE_TESTE_FUNCIONAIS.md#ct022), [CT110](CASOS_DE_TESTE_FUNCIONAIS.md#ct110) |
| Cenários pendentes por esta lacuna | [CT110](CASOS_DE_TESTE_FUNCIONAIS.md#ct110) |
| Pergunta objetiva | Como o usuário deve identificar que o primeiro envio foi persistido e evitar repetição? |

<a id="lac012"></a>

### LAC012 — Data de referência e mudança de dia

| Campo | Informação |
|---|---|
| Fontes e evidências | LocalDate.now no serviço; dataLocalIso/limitarDatas no navegador |
| Descrição e impacto | Fuso não fixado; max dos campos é definido somente no carregamento. Cenário entre fusos pendente. |
| Requisitos afetados | VAL005, VAL007, RN003 |
| Cenários afetados | [CT012](CASOS_DE_TESTE_FUNCIONAIS.md#ct012), [CT079](CASOS_DE_TESTE_FUNCIONAIS.md#ct079), [CT080](CASOS_DE_TESTE_FUNCIONAIS.md#ct080), [CT084](CASOS_DE_TESTE_FUNCIONAIS.md#ct084), [CT111](CASOS_DE_TESTE_FUNCIONAIS.md#ct111) |
| Cenários pendentes por esta lacuna | [CT111](CASOS_DE_TESTE_FUNCIONAIS.md#ct111) |
| Pergunta objetiva | Qual fuso prevalece e como renovar os limites após a virada de dia? |

<a id="lac013"></a>

### LAC013 — Documentos preditivos referenciam IDs da versão anterior

| Campo | Informação |
|---|---|
| Fontes e evidências | Docs/ANALISE_PREDITIVA_CASOS_DE_TESTE.md: §1; versão anterior de casos |
| Descrição e impacto | Análises MD/HTML anteriores permanecem históricas; os CT atuais têm novo significado. Mapa de migração fornecido. |
| Requisitos afetados | RF001, RF002, RF003, RF004, RF005, RF006 |
| Cenários afetados | Todos os identificadores da versão 1.0; ver seção 10.3 |
| Cenários pendentes por esta lacuna | Nenhum |
| Pergunta objetiva | Em uma atividade futura, atualizar a análise preditiva com a versão 2.0? |

<a id="lac014"></a>

### LAC014 — Diferenças entre validação de e-mail do navegador e Jakarta

| Campo | Informação |
|---|---|
| Fontes e evidências | index.html type=email; alunos.js validarFormulario; AlunoRequest @Email |
| Descrição e impacto | Observado no Chrome: ana..b@example.com passa na interface e recebe 400 da API; josé@example.com e ana@exa_mple.com são barrados na interface e aceitos (201) pela API; ana@localhost é aceito por ambas. |
| Requisitos afetados | VAL003 |
| Cenários afetados | [CT020](CASOS_DE_TESTE_FUNCIONAIS.md#ct020), [CT011](CASOS_DE_TESTE_FUNCIONAIS.md#ct011), [CT077](CASOS_DE_TESTE_FUNCIONAIS.md#ct077), [CT090](CASOS_DE_TESTE_FUNCIONAIS.md#ct090), [CT091](CASOS_DE_TESTE_FUNCIONAIS.md#ct091) |
| Cenários pendentes por esta lacuna | Nenhum |
| Pergunta objetiva | Quais formatos limítrofes devem ser aceitos igualmente em ambos os canais? |

<a id="lac015"></a>

### LAC015 — Pesquisa após falha de carregamento apresenta banco vazio

| Campo | Informação |
|---|---|
| Fontes e evidências | README §16; alunos.js: carregarAlunos/renderizarListagem |
| Descrição e impacto | Observado: digitar na pesquisa troca o painel de falha por "Nenhum aluno cadastrado". Cartões e contador também permanecem em zero durante a falha. |
| Requisitos afetados | UI004 |
| Cenários afetados | [CT004](CASOS_DE_TESTE_FUNCIONAIS.md#ct004) |
| Cenários pendentes por esta lacuna | Nenhum |
| Pergunta objetiva | Como pesquisa, cartões e contador devem se comportar enquanto a lista não foi carregada? |

<a id="lac016"></a>

### LAC016 — Plano aceito por posição numérica

| Campo | Informação |
|---|---|
| Fontes e evidências | README §9.7; AlunoRequest.plano; PlanoAcademia |
| Descrição e impacto | Observado: plano 0 gravou MENSAL. Diverge da lista de valores documentada. |
| Requisitos afetados | VAL006 |
| Cenários afetados | [CT086](CASOS_DE_TESTE_FUNCIONAIS.md#ct086) |
| Cenários pendentes por esta lacuna | Nenhum |
| Pergunta objetiva | Confirmar a rejeição de números e a mensagem correspondente? |

Riscos da própria especificação: os casos foram produzidos com apoio de IA e precisam de auditoria humana, como o README §4.4 determina; as estimativas de risco são heurísticas; e casos que usam DevTools dependem da habilidade de quem executa.

## 13. Referências bibliográficas

Citadas somente as fontes efetivamente consultadas. Os números de página do syllabus são os impressos na versão em PDF.

CUCUMBER. **Behaviour-Driven Development**. [S. l.]: The Cucumber Open Source Project, 2026b. Disponível em: https://cucumber.io/docs/bdd/. Acesso em: 24 nov. 2026.

CUCUMBER. **Gherkin Reference**. [S. l.]: The Cucumber Open Source Project, 2026a. Disponível em: https://cucumber.io/docs/gherkin/reference/. Acesso em: 24 nov. 2026.

INTERNATIONAL SOFTWARE TESTING QUALIFICATIONS BOARD (ISTQB). **Certified Tester Foundation Level Syllabus**: v4.0.1. [S. l.]: ISTQB, 2024. 78 p. Disponível em: https://istqb.org/certifications/certified-tester-foundation-level-ctfl-v4-0/. Acesso em: 24 nov. 2026.

INTERNATIONAL ORGANIZATION FOR STANDARDIZATION (ISO); INTERNATIONAL ELECTROTECHNICAL COMMISSION (IEC); INSTITUTE OF ELECTRICAL AND ELECTRONICS ENGINEERS (IEEE). **ISO/IEC/IEEE 29119-4:2021**: software and systems engineering: software testing: part 4: test techniques. 2021. Resumo de catálogo. Disponível em: https://webstore.iec.ch/en/publication/71836. Acesso em: 24 nov. 2026. (Texto integral não consultado.)

NIELSEN, Jakob. **10 usability heuristics for user interface design**. [S. l.]: Nielsen Norman Group, 24 abr. 1994. Atualizado em 20 ago. 2026. Disponível em: https://www.nngroup.com/articles/ten-usability-heuristics/. Acesso em: 24 nov. 2026.

NORTH, Dan. **Introducing BDD**. Publicado originalmente em Better Software, mar. 2006. Disponível em: https://dannorth.net/blog/introducing-bdd/. Acesso em: 24 nov. 2026.

Limitações de consulta: as páginas da ISO e da IEEE recusaram o acesso automatizado, e o glossário do ISTQB não pôde ser lido; este último não foi citado.
