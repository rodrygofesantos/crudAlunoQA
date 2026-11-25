# Casos de teste funcionais — Academia Alunos Web

## 1. Identificação

| Campo | Valor |
|---|---|
| Sistema | academia-alunos-web; diretório local crudAlunoQA |
| Versão | 2.0 — substitui a especificação tradicional de 20 casos |
| Data | 24/11/2026 — America/Sao_Paulo |
| Branch / commit | main / 5e03a989b1984d834472431ecd7e64136933ade2 |
| Responsável | A definir |
| Elaboração | Codex (catálogo e casos iniciais) e Claude Code (conclusão, revisão e exploração controlada); revisão humana pendente |
| Tipo e status | Testes funcionais manuais; especificação elaborada, não executada |
| Base analisada | Arquivos locais do commit e documentos de contexto; Docs/prompt2.md já estava não rastreado |
| Estratégia e rastreabilidade | [Análise e estratégia](ANALISE_E_ESTRATEGIA_DE_TESTES_FUNCIONAIS.md) |

A versão 1.0 foi lida e permanece no histórico Git. Seus comportamentos relevantes foram preservados e ampliados, com correção de contratos, dados e contagens. Os IDs foram reorganizados nesta versão; não interpretar as análises preditivas antigas como se usassem os IDs atuais. O mapa de migração está no documento de análise.

## 2. Convenções

Cada CT identifica uma definição, seja Cenário ou Esquema do Cenário. Uma definição simples equivale a uma instância; em esquemas, cada linha de Exemplos equivale a uma instância, identificada como CTxxx-EXnn. As colunas exemplo e particao identificam a linha e a partição ou limite que ela representa; são descritivas e não aparecem nos passos. Em células de Exemplos, o símbolo ␠ representa um espaço ASCII, pois o Gherkin descarta espaços nas bordas das células.

As classificações principais são exclusivas: Positivo, Negativo, Limite, Exceção e Transição. Limite prevalece quando a fronteira é o objetivo; Transição quando a mudança de estado é central. “Especificado” não significa executado ou aprovado. A tag @pendente_definicao exclui o cenário dos prontos para execução até resolução da lacuna.

Todos os blocos usam Gherkin em português e são especificações manuais: não existem step definitions ou automação. Os metadados e o catálogo são parte obrigatória de cada caso. Os resultados de passos Então e E descrevem consequências observáveis, sem exigir uma verificação artificial após cada ação.

Os níveis são Sistema, incluindo as camadas internas reais da aplicação. API é um canal, não um nível de teste isolado. As verificações HTTP têm foco em contrato; isso não é uma técnica de projeto.

## 3. Ambiente e preparação manual

Utilizar uma instância local exclusiva, sem dados reais. Não executar em produção. JDK capaz de Java 21, Maven 3.9+, navegador atual com dialog e Fetch e cliente HTTP manual são necessários. O pom fixa Spring Boot 3.5.13 e Java 21. Na exploração, o pacote foi compilado com Maven 3.9.16 (JDK 26.0.1) e executado com Java 25.0.2; a execução em JDK 21 não foi verificada.

Houve exploração controlada em 24/11/2026, em cópia isolada, com H2 próprio em memória e dados sintéticos. Seu objetivo foi esclarecer lacunas; ela não constitui execução dos casos, e nenhum resultado foi atribuído. As observações estão identificadas por “Exploração controlada” e resumidas no documento de análise, seção 3.

O executor pode iniciar a aplicação com o procedimento do README, por exemplo `mvn spring-boot:run`, em terminal próprio. Usar a porta configurada, normalmente `http://localhost:8080`; todos os caminhos abaixo são relativos a essa base. A abertura do console H2 usa a configuração local documentada no README §13.

Antes de cada instância:

1. Confirmar que a instância é exclusiva e que não contém dados de terceiros.
2. Preparar somente os registros indicados por POST manual, usando os JSON do catálogo. Registrar os IDs retornados; nunca presumir ID 1.
3. Confirmar a massa por GET da coleção. Para casos de alteração, registrar os oito campos e IDs antes da ação.
4. Quando o caso exige formulário aberto, carregar a página e acionar Novo aluno ou Editar do registro indicado. Preencher A quando o caso disser “formulário contém A”.
5. Restabelecer os dados originais entre linhas de Exemplos; cada linha é independente. Não depender de outro CT.
6. Em falha de rede, usar bloqueio de requisição ou perfil de rede do DevTools conforme o caso. Não derrubar o processo para simular rede, pois isso perderia a massa H2.
7. Observar requisições no painel Network sem alterar código. O console H2 só será usado em consultas SELECT nos casos explícitos.

Instâncias sem massa declarada utilizam banco vazio. Instâncias que mencionam A/B/C usam somente os registros expressamente necessários. A preparação não é um resultado do cenário, mas deve ser verificada antes de iniciá-lo.

### Convenção OP para POST e PUT

| Operação do exemplo | Preparação | Método e endpoint | Corpo |
|---|---|---|---|
| POST | Registro-base ainda ausente e sem conflito de CPF/e-mail | POST /api/alunos | JSON-base com as substituições da linha |
| PUT | Registro-base existente com ID capturado; demais registros somente se indicados | PUT /api/alunos/ID_A ou ID_B, conforme o registro-base | JSON-base completo com as substituições da linha |

Enviar `Accept: application/json` em consultas e escritas; enviar também `Content-Type: application/json` quando houver corpo, exceto nos casos explícitos de mídia inválida. GET e DELETE não têm corpo. Não há autenticação definida. Não exigir cabeçalhos não contratados.

“Substituir um campo” significa preservar literalmente todos os outros campos do JSON-base. “Omitir” significa remover a propriedade, não enviar string vazia nem null. Os contratos locais especificam qualquer exceção. Resposta de erro deve ser comparada com a fotografia anterior: POST não cria; PUT não altera; DELETE malsucedido não remove outros registros.

## 4. Catálogo de dados sintéticos

A, B e C são pessoas fictícias; e-mails usam domínios reservados. CPFs possuem somente validade sintática, pois o README §9.3 exclui verificação de dígitos verificadores.

D0 é a data local do processo Java no início da instância; D-1 e D+1 são o dia anterior e seguinte. Na execução comum, alinhar navegador e processo ao mesmo fuso, preferencialmente America/Sao_Paulo. Não atravessar meia-noite durante uma instância. Converter datas para ISO yyyy-MM-dd no JSON e selecionar o mesmo dia no controle local do navegador. Exemplo apenas de referência: em 24/11/2026, D0 = 2026-11-24. Não fixar essa data em execuções futuras.

| Símbolo | Significado |
|---|---|
| VAZIO | String de comprimento zero; limpar o campo na UI |
| ␠ | Um espaço ASCII; substituir pelo caractere antes de enviar. Ex.: ␠An␠ são quatro caracteres |
| null | Literal JSON null; nunca a string "null" |
| 3 espaços | String com exatamente três espaços ASCII |
| ID_A / ID_B / ID_C | Inteiro capturado no POST de preparação |
| N100 | Nome literal de 100 caracteres abaixo |
| N101 | Nome literal de 101 caracteres abaixo |
| E253 / E254 / E255 | E-mails literais abaixo, com 253/254/255 caracteres |
| sem seleção | Opção “Selecione um plano” na UI |

### A — cadastro completo válido

```json
{
  "nomeCompleto": "Ana Souza Ferreira",
  "cpf": "12345678901",
  "email": "ana.souza@example.com",
  "telefone": "31999998888",
  "dataNascimento": "1995-05-20",
  "plano": "MENSAL",
  "dataMatricula": "D0",
  "ativo": true
}
```

### B — registro distinto e inativo

```json
{
  "nomeCompleto": "Bruno Costa",
  "cpf": "98765432100",
  "email": "bruno.costa@example.com",
  "telefone": "1133224455",
  "dataNascimento": "1990-03-15",
  "plano": "ANUAL",
  "dataMatricula": "D-1",
  "ativo": false
}
```

### C — registro para pesquisa e combinação

```json
{
  "nomeCompleto": "João Souza",
  "cpf": "23456789012",
  "email": "joao.souza@example.com",
  "telefone": null,
  "dataNascimento": "1992-08-10",
  "plano": "TRIMESTRAL",
  "dataMatricula": "D-1",
  "ativo": true
}
```

Antes de enviar, substituir D0/D-1 por datas ISO concretas. Esses símbolos não são aceitos literalmente pela API. Na UI, telefone null significa campo vazio; true/false correspondem a marcado/desmarcado.

### Dados de fronteira para copiar

| Código | Comprimento | Valor literal |
|---|---:|---|
| N100 | 100 | `AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA` |
| N101 | 101 | `AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA` |
| E253 | 253 | `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb.ccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc.dddddddddddddddddddddddddddddddddddddddddddddddddddd.example` |
| E254 | 254 | `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb.ccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc.ddddddddddddddddddddddddddddddddddddddddddddddddddddd.example` |
| E255 | 255 | `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb.ccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc.dddddddddddddddddddddddddddddddddddddddddddddddddddddd.example` |

E253–E255 têm parte local de 64 caracteres e rótulos de domínio individuais de até 63 caracteres. A diferença isola o limite total. Não é necessária entrega de e-mail nem consulta DNS. Partições sem limite inferior documentado, como matrícula passada, não recebem limites etários inventados.

### Contrato SUCESSO

Resposta JSON com os nove campos: `id` inteiro gerado; `nomeCompleto`, `cpf`, `email`, `plano` strings; `telefone` string ou null; `dataNascimento` e `dataMatricula` datas ISO; `ativo` boolean. Comparar todos os valores com o corpo efetivamente enviado, após as normalizações e padrões explicitamente previstos no caso. GET/PUT retornam 200; POST 201. A coleção contém objetos dessa estrutura. Não impor ordem das propriedades JSON, sequência consecutiva de IDs nem valores fixos para IDs.

### Contrato ERRO

| Campo | Verificação |
|---|---|
| dataHora | Data/hora local serializada, interpretável como ISO, compatível com o intervalo da requisição no relógio do servidor; sem presumir UTC ou sufixo Z |
| codigoHttp | Inteiro igual ao status HTTP |
| erro | Bad Request para 400; Not Found para 404; Conflict para 409; Internal Server Error para 500 |
| mensagem | Literal especificado no caso |
| caminho | Caminho efetivo da requisição, com o ID substituído e sem host |
| errosValidacao | Objeto por campo em Bean Validation; objeto vazio nas exceções gerais aqui especificadas |

Content-Type deve incluir application/json; parâmetros adicionais são permitidos. Não substituir `codigoHttp` por `status` nem `caminho` por `path`, erro existente na versão anterior.

### Catálogo MSG de validações

| Código / campo | Mensagem |
|---|---|
| MSG-NOME | O nome completo deve ter entre 3 e 100 caracteres. |
| MSG-CPF | O CPF deve conter exatamente 11 números. |
| MSG-NASCIMENTO | A data de nascimento deve ser anterior à data atual. |
| nomeCompleto obrigatório | O nome completo é obrigatório. |
| cpf obrigatório | O CPF é obrigatório. |
| email obrigatório | O e-mail é obrigatório. |
| email formato | Informe um e-mail válido. |
| email tamanho | O e-mail deve ter no máximo 254 caracteres. |
| telefone | O telefone deve conter 10 ou 11 números. |
| nascimento obrigatório | A data de nascimento é obrigatória. |
| plano API | O plano é obrigatório. |
| plano UI | Selecione um plano. |
| matrícula | A data de matrícula não pode estar no futuro. |

Para CT082, conjuntos admissíveis: nome vazio → obrigatório ou MSG-NOME; nome com três espaços → obrigatório; CPF vazio/espaços → obrigatório ou MSG-CPF; e-mail vazio → obrigatório; e-mail com três espaços → obrigatório ou formato. A seleção de uma única mensagem quando há duas violações é LAC004; não aceitar texto arbitrário. Nos demais casos com uma violação isolada, exigir o literal indicado.

## 5. Casos de teste

### Índice

**Inicialização**

- [CT001 — Carregar a página e seus recursos ao acessar o sistema](#ct001)
- [CT002 — Informar carregamento enquanto a consulta inicial está em andamento](#ct002)
- [CT003 — Recuperar a listagem após falha de comunicação](#ct003)
- [CT004 — Manter a indicação de falha ao pesquisar após carregamento malsucedido](#ct004)

**Cadastro**

- [CT005 — Abrir cadastro com valores padrão ao solicitar novo aluno](#ct005)
- [CT006 — Cadastrar aluno com sucesso ao informar todos os dados válidos](#ct006)
- [CT007 — Cadastrar aluno sem telefone e com matrícula em branco](#ct007)
- [CT008 — Cadastrar aluno com cada plano permitido](#ct008)
- [CT009 — Cadastrar aluno inativo ao desmarcar a situação](#ct009)
- [CT010 — Rejeitar campos obrigatórios não preenchidos antes de enviar o cadastro](#ct010)
- [CT011 — Rejeitar formato ou quantidade de dígitos inválida antes de enviar o cadastro](#ct011)
- [CT012 — Rejeitar datas fora dos limites permitidos antes de enviar o cadastro](#ct012)
- [CT013 — Aceitar o nome nos limites de tamanho pela interface](#ct013)
- [CT014 — Rejeitar nome abaixo do mínimo após remover espaços](#ct014)
- [CT015 — Limitar entradas de texto ao comprimento suportado pela interface](#ct015)
- [CT016 — Aplicar máscaras e remover caracteres não numéricos na entrada](#ct016)
- [CT017 — Impedir cadastro quando letras deixam CPF incompleto](#ct017)
- [CT018 — Rejeitar cadastro com CPF já utilizado](#ct018)
- [CT019 — Rejeitar cadastro com e-mail já utilizado](#ct019)
- [CT020 — Exibir junto ao campo o erro de validação devolvido pela API](#ct020)
- [CT021 — Permitir correção e reenvio após erro de validação](#ct021)
- [CT022 — Manter formulário recuperável após falha de rede no envio](#ct022)
- [CT023 — Bloquear novo clique de gravação durante processamento](#ct023)
- [CT024 — Descartar cadastro não enviado ao fechar o formulário](#ct024)

**Listagem**

- [CT025 — Exibir estado vazio quando não existem alunos](#ct025)
- [CT026 — Exibir os campos e contador singular quando existe um aluno](#ct026)
- [CT027 — Exibir literalmente caracteres de marcação presentes no nome](#ct027)
- [CT028 — Ordenar registros e atualizar indicadores quando existem vários alunos](#ct028)

**Consulta**

- [CT029 — Retornar os dados completos ao consultar ID existente](#ct029)
- [CT030 — Retornar ausência ao consultar ID numérico não cadastrado](#ct030)

**Atualização**

- [CT031 — Abrir edição com os dados atuais do aluno selecionado](#ct031)
- [CT032 — Persistir cada campo editável ao salvar a edição](#ct032)
- [CT033 — Preservar a matrícula anterior ao limpar a data na edição](#ct033)
- [CT034 — Remover o telefone ao limpar o campo na edição](#ct034)
- [CT035 — Reativar aluno inativo ao marcar a situação na edição](#ct035)
- [CT036 — Aceitar edição ao manter CPF e e-mail do próprio aluno](#ct036)
- [CT037 — Rejeitar edição ao utilizar CPF de outro aluno](#ct037)
- [CT038 — Rejeitar edição ao utilizar e-mail de outro aluno](#ct038)
- [CT039 — Reaplicar validações locais ao editar aluno](#ct039)
- [CT040 — Preservar registro ao cancelar uma edição não enviada](#ct040)
- [CT041 — Informar inexistência ao salvar aluno removido por outro cliente](#ct041)

**Exclusão**

- [CT042 — Identificar o aluno correto antes de confirmar a exclusão](#ct042)
- [CT043 — Preservar aluno ao fechar a confirmação sem excluir](#ct043)
- [CT044 — Remover aluno e atualizar resumo após confirmação](#ct044)
- [CT045 — Exibir estado vazio ao excluir o último aluno](#ct045)
- [CT046 — Bloquear cliques adicionais enquanto a exclusão é processada](#ct046)
- [CT047 — Permitir nova tentativa após falha de rede na exclusão](#ct047)
- [CT048 — Informar ausência ao excluir registro removido por outro cliente](#ct048)

**Pesquisa e filtros**

- [CT049 — Localizar alunos pelos critérios de pesquisa suportados](#ct049)
- [CT050 — Combinar termo e situação por interseção](#ct050)
- [CT051 — Distinguir ausência de correspondências de banco vazio](#ct051)
- [CT052 — Restaurar a lista ao limpar pesquisa e filtros](#ct052)
- [CT053 — Preservar filtro de situação ao apagar somente a pesquisa](#ct053)
- [CT054 — Reaplicar filtros ao atualizar dados de um aluno](#ct054)

**Interface e interação**

- [CT055 — Posicionar o foco inicial ao abrir cada diálogo](#ct055)
- [CT056 — Devolver foco ao acionador após fechar diálogo sem gravação](#ct056)
- [CT057 — Cadastrar aluno utilizando somente teclado](#ct057)
- [CT058 — Direcionar a navegação ao conteúdo principal pelo link de salto](#ct058)
- [CT059 — Apresentar instruções de uso ao abrir a ajuda](#ct059)
- [CT060 — Preservar o formulário e focar o primeiro erro quando há vários campos inválidos](#ct060)
- [CT061 — Anunciar e encerrar notificações conforme seu tipo](#ct061)
- [CT062 — Manter dados e ações acessíveis na apresentação estreita](#ct062)
- [CT063 — Restaurar foco após salvar uma edição e reconstruir a listagem](#ct063)
- [CT064 — Definir destino de foco após remover o acionador da exclusão](#ct064) *(pendente)*
- [CT065 — Informar falha de atualização da lista após gravação bem-sucedida](#ct065)
- [CT066 — Definir efeito de fechar diálogo durante gravação em andamento](#ct066) *(pendente)*

**API REST**

- [CT067 — Criar aluno e retornar representação completa com identificador](#ct067)
- [CT068 — Aplicar valores padrão ao cadastrar com opcionais ausentes ou nulos](#ct068)
- [CT069 — Listar alunos em array ordenado conforme os registros existentes](#ct069)
- [CT070 — Atualizar dados por PUT preservando a identidade do aluno](#ct070)
- [CT071 — Preservar matrícula e situação na edição com opcionais omitidos](#ct071)
- [CT072 — Normalizar dados válidos antes de persistir cadastro e atualização](#ct072)
- [CT073 — Aceitar CPF sintático sem exigir dígitos verificadores](#ct073)
- [CT074 — Aceitar valores de nome dentro dos limites na API](#ct074)
- [CT075 — Rejeitar nome fora dos limites no cadastro e na atualização](#ct075)
- [CT076 — Rejeitar CPF fora do formato exigido no cadastro e na atualização](#ct076)
- [CT077 — Rejeitar e-mail com formato inválido no cadastro e na atualização](#ct077)
- [CT078 — Rejeitar telefone fora das partições permitidas no cadastro e na atualização](#ct078)
- [CT079 — Rejeitar nascimento atual ou futuro no cadastro e na atualização](#ct079)
- [CT080 — Rejeitar matrícula futura no cadastro e na atualização](#ct080)
- [CT081 — Rejeitar campos obrigatórios nulos ou omitidos pela API](#ct081)
- [CT082 — Rejeitar textos obrigatórios vazios ou somente espaços](#ct082)
- [CT083 — Aceitar telefone nas partições opcionais e limites permitidos](#ct083)
- [CT084 — Aceitar datas válidas nos limites permitidos](#ct084)
- [CT085 — Aceitar cada plano permitido pela API](#ct085)
- [CT086 — Rejeitar plano informado por valor numérico](#ct086)
- [CT087 — Persistir situação explícita sem substituí-la pelo padrão](#ct087)
- [CT088 — Rejeitar conteúdo que não possa ser convertido em requisição válida](#ct088)
- [CT089 — Informar todos os campos inválidos em uma mesma resposta de validação](#ct089)
- [CT090 — Aceitar e-mail no limite máximo de tamanho](#ct090)
- [CT091 — Rejeitar e-mail acima do tamanho máximo](#ct091)
- [CT092 — Rejeitar duplicidade de CPF no cadastro e na atualização](#ct092)
- [CT093 — Rejeitar e-mail duplicado independentemente de caixa](#ct093)
- [CT094 — Aceitar atualização com os identificadores do próprio aluno](#ct094)
- [CT095 — Priorizar conflito de CPF quando CPF e e-mail pertencem a outro aluno](#ct095)
- [CT096 — Retornar ausência ao atualizar aluno inexistente](#ct096)
- [CT097 — Priorizar a validação do corpo ao atualizar aluno inexistente](#ct097)
- [CT098 — Excluir registro e retornar resposta sem corpo](#ct098)
- [CT099 — Retornar ausência ao repetir exclusão de registro removido](#ct099)
- [CT100 — Preservar o identificador quando o corpo de atualização tenta trocá-lo](#ct100)
- [CT101 — Definir resposta para identificador incompatível com Long](#ct101) *(pendente)*
- [CT102 — Definir resposta para método não suportado na coleção](#ct102) *(pendente)*
- [CT103 — Definir resposta para tipo de mídia incompatível](#ct103) *(pendente)*
- [CT104 — Definir resposta para rota inexistente](#ct104) *(pendente)*
- [CT105 — Definir tratamento de tipos escalares convertíveis](#ct105) *(pendente)*
- [CT106 — Rejeitar nome que fica abaixo do mínimo após normalização](#ct106)
- [CT107 — Definir contrato para entrada de nome sem sobrenome](#ct107) *(pendente)*
- [CT108 — Especificar retorno de falha interna sem expor detalhes técnicos](#ct108) *(pendente)*
- [CT109 — Preservar unicidade quando gravações concorrentes disputam os mesmos dados](#ct109) *(pendente)*
- [CT110 — Definir recuperação quando a resposta de cadastro se perde após persistência](#ct110) *(pendente)*
- [CT111 — Definir referência temporal quando navegador e servidor estão em dias distintos](#ct111) *(pendente)*

**Persistência**

- [CT112 — Recuperar dados após recarregar página sem reiniciar o processo](#ct112)
- [CT113 — Verificar armazenamento físico após cadastro e atualização](#ct113)
- [CT114 — Confirmar remoção física após exclusão pela API](#ct114)
- [CT115 — Reinicializar dados ao encerrar e iniciar a instância isolada](#ct115)
- [CT116 — Manter os registros intactos após falha de atualização](#ct116)

### 5.1 Inicialização

<a id="ct001"></a>

#### CT001 — Carregar a página e seus recursos ao acessar o sistema

| Campo | Informação |
|---|---|
| Identificador | CT001 |
| Funcionalidade | Inicialização |
| Objetivo | Carregar a página e seus recursos ao acessar o sistema. |
| Prioridade | Média |
| Risco associado | R04 — Consulta/listagem indisponível ou incorreta (6) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF002, UI004 |
| Origem da regra | RF002: Documentado; UI004: Documentado |
| Evidência no repositório | README.md: §9.12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: listarTodos<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: findAllByOrderByIdAsc<br>README.md: §16<br>src/main/resources/static/js/alunos.js: mostrarEstadoCarregando/mostrarEstadoErro/mostrarEstadoVazio/mostrarEstadoSemResultados |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Inicialização

  @CT001 @RF002 @UI004 @prioridade_media @positivo @interface
  Cenário: Carregar a página e seus recursos ao acessar o sistema
    Dado que a instância local está disponível e contém o aluno A
    Quando o usuário acessa a página inicial
    Então deve ver o título "Gestão de Alunos" e os controles "Novo aluno" e "Ajuda"
    E os recursos /css/estilos.css e /js/alunos.js devem ser carregados sem falha
    E a requisição GET /api/alunos deve alimentar a lista com A e o texto "Atualizado agora"
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct002"></a>

#### CT002 — Informar carregamento enquanto a consulta inicial está em andamento

| Campo | Informação |
|---|---|
| Identificador | CT002 |
| Funcionalidade | Inicialização |
| Objetivo | Informar carregamento enquanto a consulta inicial está em andamento. |
| Prioridade | Alta |
| Risco associado | R06 — Falha sem recuperação ou reenvio involuntário (9) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | UI004 |
| Origem da regra | UI004: Documentado |
| Evidência no repositório | README.md: §16<br>src/main/resources/static/js/alunos.js: mostrarEstadoCarregando/mostrarEstadoErro/mostrarEstadoVazio/mostrarEstadoSemResultados |
| Situação da especificação | Especificado |

**Preparação e dados:** Utilizar limitação de rede no DevTools apenas no navegador local; restaurar ao concluir.

```gherkin
# language: pt
Funcionalidade: Inicialização

  @CT002 @UI004 @prioridade_alta @transicao @interface
  Cenário: Informar carregamento enquanto a consulta inicial está em andamento
    Dado que a velocidade de rede do navegador está limitada e a instância está disponível
    Quando o usuário acessa a página
    Então deve ver "Carregando alunos..." e "Aguarde enquanto buscamos os dados mais recentes."
    E a tabela deve permanecer oculta até a conclusão da consulta
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct003"></a>

#### CT003 — Recuperar a listagem após falha de comunicação

| Campo | Informação |
|---|---|
| Identificador | CT003 |
| Funcionalidade | Inicialização |
| Objetivo | Recuperar a listagem após falha de comunicação. |
| Prioridade | Alta |
| Risco associado | R06 — Falha sem recuperação ou reenvio involuntário (9) |
| Classificação principal | Exceção |
| Características secundárias | Transição (recuperação) |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Error guessing fundamentado |
| Requisitos cobertos | RF002, UI004 |
| Origem da regra | RF002: Documentado; UI004: Documentado |
| Evidência no repositório | README.md: §9.12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: listarTodos<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: findAllByOrderByIdAsc<br>README.md: §16<br>src/main/resources/static/js/alunos.js: mostrarEstadoCarregando/mostrarEstadoErro/mostrarEstadoVazio/mostrarEstadoSemResultados |
| Situação da especificação | Especificado |

**Preparação e dados:** Bloquear a requisição da API, não todo o host; não parar o processo nem perder o H2. A notificação persistente anterior pode precisar de fechamento manual.

```gherkin
# language: pt
Funcionalidade: Inicialização

  @CT003 @RF002 @UI004 @prioridade_alta @excecao @interface
  Cenário: Recuperar a listagem após falha de comunicação
    Dado que somente a URL /api/alunos está bloqueada no DevTools e a página pode carregar
    Quando o usuário abre a página
    Então deve ver "Não foi possível carregar os alunos" e "Tentar novamente"
    E deve receber a notificação "Não foi possível carregar os alunos."
    Quando o bloqueio é removido e o usuário seleciona "Tentar novamente"
    Então uma nova consulta deve carregar os alunos e retirar o painel de falha
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct004"></a>

#### CT004 — Manter a indicação de falha ao pesquisar após carregamento malsucedido

| Campo | Informação |
|---|---|
| Identificador | CT004 |
| Funcionalidade | Inicialização |
| Objetivo | Manter a indicação de falha ao pesquisar após carregamento malsucedido. |
| Prioridade | Alta |
| Risco associado | R06 — Falha sem recuperação ou reenvio involuntário (9) |
| Classificação principal | Exceção |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Error guessing fundamentado |
| Requisitos cobertos | UI004, UI010 |
| Origem da regra | UI004: Documentado; UI010: Inferido da implementação |
| Evidência no repositório | README.md: §16<br>src/main/resources/static/js/alunos.js: mostrarEstadoCarregando/mostrarEstadoErro/mostrarEstadoVazio/mostrarEstadoSemResultados<br>src/main/resources/static/js/alunos.js: normalizarBusca/renderizarListagem |
| Situação da especificação | Especificado |

**Preparação e dados:** Bloquear /api/alunos no DevTools antes de abrir a página e manter o bloqueio durante a digitação.

```gherkin
# language: pt
Funcionalidade: Inicialização

  @CT004 @UI004 @UI010 @prioridade_alta @excecao @interface
  Cenário: Manter a indicação de falha ao pesquisar após carregamento malsucedido
    Dado que A existe e a consulta inicial /api/alunos falhou por bloqueio no navegador
    E o painel "Não foi possível carregar os alunos" está visível
    Quando o usuário digita "Ana" no campo de pesquisa
    Então não deve aparecer "Nenhum aluno cadastrado"
    E deve continuar disponível a ação "Tentar novamente"
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** LAC015. README §16 exige distinguir indisponibilidade de outras situações. Exploração controlada: a digitação substituiu o painel de falha por "Nenhum aluno cadastrado", retirando a ação de nova tentativa.


### 5.2 Cadastro

<a id="ct005"></a>

#### CT005 — Abrir cadastro com valores padrão ao solicitar novo aluno

| Campo | Informação |
|---|---|
| Identificador | CT005 |
| Funcionalidade | Cadastro |
| Objetivo | Abrir cadastro com valores padrão ao solicitar novo aluno. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF001, UI006 |
| Origem da regra | RF001: Documentado; UI006: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §§12,16<br>src/main/resources/static/js/alunos.js: abrirCadastro/abrirEdicao |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT005 @RF001 @UI006 @prioridade_alta @positivo @interface
  Cenário: Abrir cadastro com valores padrão ao solicitar novo aluno
    Dado que a listagem está carregada
    Quando o usuário seleciona "Novo aluno"
    Então deve abrir o diálogo "Cadastrar aluno" com os campos de identificação vazios
    E a matrícula deve conter D0, a situação deve estar marcada e o texto deve ser "Aluno ativo"
    E o plano deve apresentar "Selecione um plano"
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct006"></a>

#### CT006 — Cadastrar aluno com sucesso ao informar todos os dados válidos

| Campo | Informação |
|---|---|
| Identificador | CT006 |
| Funcionalidade | Cadastro |
| Objetivo | Cadastrar aluno com sucesso ao informar todos os dados válidos. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF001, UI002, UI003, UI005 |
| Origem da regra | RF001: Documentado; UI002: Documentado; UI003: Documentado; UI005: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §9<br>src/main/resources/static/js/alunos.js: aplicarMascaraCpf/aplicarMascaraTelefone/obterDadosFormulario<br>README.md: §16<br>src/main/resources/static/js/alunos.js: salvarAluno/excluirAluno/alterarEstadoBotao<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT006 @RF001 @UI002 @UI003 @UI005 @prioridade_alta @positivo @interface
  Cenário: Cadastrar aluno com sucesso ao informar todos os dados válidos
    Dado que o formulário de cadastro está aberto e A ainda não existe
    Quando o usuário preenche os dados de A e seleciona "Salvar aluno"
    Então deve receber "Aluno cadastrado com sucesso." e o diálogo deve fechar
    E a linha de A deve exibir CPF "123.456.789-01", plano "Mensal" e situação "Ativo"
    E o total e os ativos devem aumentar em um após a consulta de atualização
```

**Pós-condições e restauração:** Consultar A por ID para conferir os oito campos. Remover somente A após a instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct007"></a>

#### CT007 — Cadastrar aluno sem telefone e com matrícula em branco

| Campo | Informação |
|---|---|
| Identificador | CT007 |
| Funcionalidade | Cadastro |
| Objetivo | Cadastrar aluno sem telefone e com matrícula em branco. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF001, RN003, VAL004, VAL007 |
| Origem da regra | RF001: Documentado; RN003: Documentado / extensão inferida para null; VAL004: Documentado / null inferido; VAL007: Documentado |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §9.8–9<br>README.md: §9.5<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: telefone<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarTelefone<br>README.md: §9.8<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataMatricula<br>src/main/resources/static/js/alunos.js: validarFormulario |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT007 @RF001 @RN003 @VAL004 @VAL007 @prioridade_alta @positivo @interface
  Cenário: Cadastrar aluno sem telefone e com matrícula em branco
    Dado que o formulário está aberto e A ainda não existe
    Quando o usuário preenche A, limpa telefone e matrícula e salva
    Então o cadastro deve concluir com "Aluno cadastrado com sucesso."
    E a consulta posterior deve retornar telefone null, matrícula D0 e ativo true
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** A interface envia telefone vazio, matrícula null e ativo true. Omissão real de propriedades é coberta diretamente na API.

<a id="ct008"></a>

#### CT008 — Cadastrar aluno com cada plano permitido

| Campo | Informação |
|---|---|
| Identificador | CT008 |
| Funcionalidade | Cadastro |
| Objetivo | Cadastrar aluno com cada plano permitido. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF001, VAL006 |
| Origem da regra | RF001: Documentado; VAL006: Documentado |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §9.7<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: plano<br>src/main/java/br/com/academia/alunos/entity/PlanoAcademia.java |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT008 @RF001 @VAL006 @prioridade_alta @positivo @interface
  Esquema do Cenário: Cadastrar aluno com cada plano permitido
    Dado que o formulário está aberto e A ainda não existe
    Quando o usuário preenche A com o plano "<plano>" e salva
    Então a linha cadastrada deve apresentar o plano "<rotulo>"

    Exemplos:
      | exemplo    | particao         | plano      | rotulo     |
      | CT008-EX01 | plano Mensal     | MENSAL     | Mensal     |
      | CT008-EX02 | plano Trimestral | TRIMESTRAL | Trimestral |
      | CT008-EX03 | plano Anual      | ANUAL      | Anual      |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct009"></a>

#### CT009 — Cadastrar aluno inativo ao desmarcar a situação

| Campo | Informação |
|---|---|
| Identificador | CT009 |
| Funcionalidade | Cadastro |
| Objetivo | Cadastrar aluno inativo ao desmarcar a situação. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF001, UI005, UI006 |
| Origem da regra | RF001: Documentado; UI005: Documentado / detalhes inferidos; UI006: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador<br>README.md: §§12,16<br>src/main/resources/static/js/alunos.js: abrirCadastro/abrirEdicao |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT009 @RF001 @UI005 @UI006 @prioridade_alta @transicao @interface
  Cenário: Cadastrar aluno inativo ao desmarcar a situação
    Dado que o formulário está preenchido com A e o banco está vazio
    Quando o usuário desmarca a situação
    Então o texto deve mudar para "Aluno inativo"
    Quando o usuário salva
    Então a linha deve mostrar "Inativo" e os cartões devem indicar total 1, ativos 0 e inativos 1
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct010"></a>

#### CT010 — Rejeitar campos obrigatórios não preenchidos antes de enviar o cadastro

| Campo | Informação |
|---|---|
| Identificador | CT010 |
| Funcionalidade | Cadastro |
| Objetivo | Rejeitar campos obrigatórios não preenchidos antes de enviar o cadastro. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | VAL001, VAL002, VAL003, VAL005, VAL006, UI008 |
| Origem da regra | VAL001: Documentado; VAL002: Documentado; VAL003: Documentado / limite inferido; VAL005: Documentado; VAL006: Documentado; UI008: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §9.2–3<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: cpf<br>README.md: §9.4<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: email<br>src/main/resources/static/index.html: email<br>README.md: §9.6<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataNascimento<br>README.md: §9.7<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: plano<br>src/main/java/br/com/academia/alunos/entity/PlanoAcademia.java<br>README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT010 @VAL001 @VAL002 @VAL003 @VAL005 @VAL006 @UI008 @prioridade_alta @negativo @interface
  Esquema do Cenário: Rejeitar campos obrigatórios não preenchidos antes de enviar o cadastro
    Dado que o formulário de cadastro contém A ainda não cadastrado
    E somente o campo "<campo>" recebe o valor "<valor>"
    Quando o usuário seleciona "Salvar aluno"
    Então deve aparecer "<mensagem>" junto ao campo
    E deve aparecer "Corrija os campos destacados para continuar."
    E o campo inválido deve receber foco e aria-invalid true
    E nenhuma requisição POST deve ser enviada e os demais dados devem ser preservados

    Exemplos:
      | exemplo    | particao                 | campo          | valor       | mensagem                                           |
      | CT010-EX01 | nomeCompleto: ausente    | nomeCompleto   | VAZIO       | O nome completo deve ter entre 3 e 100 caracteres. |
      | CT010-EX02 | nomeCompleto: só espaços | nomeCompleto   | 3 espaços   | O nome completo deve ter entre 3 e 100 caracteres. |
      | CT010-EX03 | cpf: ausente             | cpf            | VAZIO       | O CPF deve conter exatamente 11 números.           |
      | CT010-EX04 | email: ausente           | email          | VAZIO       | Informe um e-mail válido.                          |
      | CT010-EX05 | dataNascimento: ausente  | dataNascimento | VAZIO       | A data de nascimento é obrigatória.                |
      | CT010-EX06 | plano: ausente           | plano          | sem seleção | Selecione um plano.                                |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct011"></a>

#### CT011 — Rejeitar formato ou quantidade de dígitos inválida antes de enviar o cadastro

| Campo | Informação |
|---|---|
| Identificador | CT011 |
| Funcionalidade | Cadastro |
| Objetivo | Rejeitar formato ou quantidade de dígitos inválida antes de enviar o cadastro. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Negativo |
| Características secundárias | Limite |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | VAL002, VAL003, VAL004, UI008 |
| Origem da regra | VAL002: Documentado; VAL003: Documentado / limite inferido; VAL004: Documentado / null inferido; UI008: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §9.2–3<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: cpf<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §9.4<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: email<br>src/main/resources/static/index.html: email<br>README.md: §9.5<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: telefone<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarTelefone<br>README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT011 @VAL002 @VAL003 @VAL004 @UI008 @prioridade_alta @negativo @interface
  Esquema do Cenário: Rejeitar formato ou quantidade de dígitos inválida antes de enviar o cadastro
    Dado que o formulário de cadastro contém A ainda não cadastrado
    E somente o campo "<campo>" recebe o valor "<valor>"
    Quando o usuário seleciona "Salvar aluno"
    Então deve aparecer "<mensagem>" junto ao campo
    E deve aparecer "Corrija os campos destacados para continuar."
    E o campo inválido deve receber foco e aria-invalid true
    E nenhuma requisição POST deve ser enviada e os demais dados devem ser preservados

    Exemplos:
      | exemplo    | particao                    | campo    | valor      | mensagem                                 |
      | CT011-EX01 | CPF: mínimo − 1 dígito      | cpf      | 1234567890 | O CPF deve conter exatamente 11 números. |
      | CT011-EX02 | e-mail: sem domínio         | email    | usuario@   | Informe um e-mail válido.                |
      | CT011-EX03 | telefone: mínimo − 1 dígito | telefone | 123456789  | O telefone deve conter 10 ou 11 números. |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct012"></a>

#### CT012 — Rejeitar datas fora dos limites permitidos antes de enviar o cadastro

| Campo | Informação |
|---|---|
| Identificador | CT012 |
| Funcionalidade | Cadastro |
| Objetivo | Rejeitar datas fora dos limites permitidos antes de enviar o cadastro. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL005, VAL007, UI008 |
| Origem da regra | VAL005: Documentado; VAL007: Documentado; UI008: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §9.6<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataNascimento<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §9.8<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataMatricula<br>README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Digitar as datas pelos segmentos do controle. O formulário usa novalidate, portanto o atributo max não bloqueia o envio por si só. Se o navegador impedir a entrada, registrar o bloqueio e usar os casos de API; não alterar o DOM.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT012 @VAL005 @VAL007 @UI008 @prioridade_alta @limite @interface
  Esquema do Cenário: Rejeitar datas fora dos limites permitidos antes de enviar o cadastro
    Dado que o formulário de cadastro contém A ainda não cadastrado
    E somente o campo "<campo>" recebe o valor "<valor>"
    Quando o usuário seleciona "Salvar aluno"
    Então deve aparecer "<mensagem>" junto ao campo
    E deve aparecer "Corrija os campos destacados para continuar."
    E o campo inválido deve receber foco e aria-invalid true
    E nenhuma requisição POST deve ser enviada e os demais dados devem ser preservados

    Exemplos:
      | exemplo    | particao                           | campo          | valor | mensagem                                             |
      | CT012-EX01 | nascimento: hoje (limite inválido) | dataNascimento | D0    | A data de nascimento deve ser anterior à data atual. |
      | CT012-EX02 | nascimento: futuro                 | dataNascimento | D+1   | A data de nascimento deve ser anterior à data atual. |
      | CT012-EX03 | matrícula: hoje + 1                | dataMatricula  | D+1   | A data de matrícula não pode estar no futuro.        |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct013"></a>

#### CT013 — Aceitar o nome nos limites de tamanho pela interface

| Campo | Informação |
|---|---|
| Identificador | CT013 |
| Funcionalidade | Cadastro |
| Objetivo | Aceitar o nome nos limites de tamanho pela interface. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Positivo |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL001 |
| Origem da regra | VAL001: Documentado |
| Evidência no repositório | README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT013 @VAL001 @prioridade_alta @limite @interface
  Esquema do Cenário: Aceitar o nome nos limites de tamanho pela interface
    Dado que o formulário contém A ainda não cadastrado
    Quando o usuário informa "<nome>" como nome e salva
    Então o aluno deve ser cadastrado com o nome de <tamanho> caracteres

    Exemplos:
      | exemplo    | particao | nome               | tamanho |
      | CT013-EX01 | mínimo   | Ana                | 3       |
      | CT013-EX02 | nominal  | Ana Souza Ferreira | 18      |
      | CT013-EX03 | máximo   | N100               | 100     |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct014"></a>

#### CT014 — Rejeitar nome abaixo do mínimo após remover espaços

| Campo | Informação |
|---|---|
| Identificador | CT014 |
| Funcionalidade | Cadastro |
| Objetivo | Rejeitar nome abaixo do mínimo após remover espaços. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL001, RN005, UI008 |
| Origem da regra | VAL001: Documentado; RN005: Inferido da implementação; UI008: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar/atualizar/normalizarEmail/normalizarTelefone<br>README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT014 @VAL001 @RN005 @UI008 @prioridade_alta @limite @interface
  Esquema do Cenário: Rejeitar nome abaixo do mínimo após remover espaços
    Dado que o formulário contém A ainda não cadastrado
    Quando o usuário informa o nome "<nome>" e salva
    Então deve ver "O nome completo deve ter entre 3 e 100 caracteres."
    E nenhuma requisição POST deve ser enviada

    Exemplos:
      | exemplo    | particao             | nome |
      | CT014-EX01 | mínimo − 1           | An   |
      | CT014-EX02 | mínimo − 1 após trim | ␠An␠ |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct015"></a>

#### CT015 — Limitar entradas de texto ao comprimento suportado pela interface

| Campo | Informação |
|---|---|
| Identificador | CT015 |
| Funcionalidade | Cadastro |
| Objetivo | Limitar entradas de texto ao comprimento suportado pela interface. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL001, VAL003, UI002 |
| Origem da regra | VAL001: Documentado; VAL003: Documentado / limite inferido; UI002: Documentado |
| Evidência no repositório | README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §9.4<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: email<br>src/main/resources/static/index.html: email<br>README.md: §9<br>src/main/resources/static/js/alunos.js: aplicarMascaraCpf/aplicarMascaraTelefone/obterDadosFormulario |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT015 @VAL001 @VAL003 @UI002 @prioridade_alta @limite @interface
  Esquema do Cenário: Limitar entradas de texto ao comprimento suportado pela interface
    Dado que o formulário está aberto
    Quando o usuário cola "<entrada>" no campo "<campo>"
    Então o campo deve conter "<saida>"
    E ainda não deve haver requisição de gravação

    Exemplos:
      | exemplo    | particao                    | campo        | entrada      | saida                            |
      | CT015-EX01 | nome: máximo + 1            | nomeCompleto | N101         | N100                             |
      | CT015-EX02 | e-mail: máximo + 1          | email        | E255         | primeiros 254 caracteres de E255 |
      | CT015-EX03 | CPF: máximo + 1 dígito      | cpf          | 123456789012 | 123.456.789-01                   |
      | CT015-EX04 | telefone: máximo + 1 dígito | telefone     | 119999988887 | (11) 99999-8888                  |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Verificar comprimento no próprio campo; não confundir truncamento de entrada com rejeição da API.

<a id="ct016"></a>

#### CT016 — Aplicar máscaras e remover caracteres não numéricos na entrada

| Campo | Informação |
|---|---|
| Identificador | CT016 |
| Funcionalidade | Cadastro |
| Objetivo | Aplicar máscaras e remover caracteres não numéricos na entrada. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | UI002 |
| Origem da regra | UI002: Documentado |
| Evidência no repositório | README.md: §9<br>src/main/resources/static/js/alunos.js: aplicarMascaraCpf/aplicarMascaraTelefone/obterDadosFormulario |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT016 @UI002 @prioridade_alta @positivo @interface
  Esquema do Cenário: Aplicar máscaras e remover caracteres não numéricos na entrada
    Dado que o formulário contém os demais dados de A
    Quando o usuário cola "<entrada>" no campo "<campo>"
    Então o controle deve mostrar "<mascara>"
    Quando o usuário salva
    Então a requisição deve conter "<numero>" nesse campo e o cadastro deve concluir

    Exemplos:
      | exemplo    | particao              | campo    | entrada        | mascara         | numero      |
      | CT016-EX01 | CPF já pontuado       | cpf      | 123.456.789-01 | 123.456.789-01  | 12345678901 |
      | CT016-EX02 | telefone fixo (10)    | telefone | 1133224455     | (11) 3322-4455  | 1133224455  |
      | CT016-EX03 | telefone celular (11) | telefone | 11999998888    | (11) 99999-8888 | 11999998888 |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Somente linhas com quantidade final válida podem ser salvas.

<a id="ct017"></a>

#### CT017 — Impedir cadastro quando letras deixam CPF incompleto

| Campo | Informação |
|---|---|
| Identificador | CT017 |
| Funcionalidade | Cadastro |
| Objetivo | Impedir cadastro quando letras deixam CPF incompleto. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | UI002, VAL002 |
| Origem da regra | UI002: Documentado; VAL002: Documentado |
| Evidência no repositório | README.md: §9<br>src/main/resources/static/js/alunos.js: aplicarMascaraCpf/aplicarMascaraTelefone/obterDadosFormulario<br>README.md: §9.2–3<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: cpf<br>src/main/resources/static/js/alunos.js: validarFormulario |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT017 @UI002 @VAL002 @prioridade_alta @negativo @interface
  Cenário: Impedir cadastro quando letras deixam CPF incompleto
    Dado que o formulário contém os demais dados de A
    Quando o usuário cola "12345abc901" no CPF e tenta salvar
    Então o CPF deve conter apenas os oito dígitos "12345901" com máscara progressiva
    E deve aparecer "O CPF deve conter exatamente 11 números." sem enviar POST
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct018"></a>

#### CT018 — Rejeitar cadastro com CPF já utilizado

| Campo | Informação |
|---|---|
| Identificador | CT018 |
| Funcionalidade | Cadastro |
| Objetivo | Rejeitar cadastro com CPF já utilizado. |
| Prioridade | Crítica |
| Risco associado | R02 — Duplicidade de identificadores (12) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RN001, UI008 |
| Origem da regra | RN001: Documentado; UI008: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §9.2,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: verificarDuplicidadeNoCadastro/verificarDuplicidadeNaAtualizacao<br>README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT018 @RN001 @UI008 @prioridade_critica @negativo @interface
  Cenário: Rejeitar cadastro com CPF já utilizado
    Dado que A existe e o formulário de novo aluno contém B
    Quando o usuário substitui cpf por "12345678901" e salva
    Então deve ver "Já existe um aluno com o CPF informado."
    E o diálogo e os valores preenchidos devem permanecer disponíveis
    E a consulta da lista deve continuar contendo somente A
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct019"></a>

#### CT019 — Rejeitar cadastro com e-mail já utilizado

| Campo | Informação |
|---|---|
| Identificador | CT019 |
| Funcionalidade | Cadastro |
| Objetivo | Rejeitar cadastro com e-mail já utilizado. |
| Prioridade | Crítica |
| Risco associado | R02 — Duplicidade de identificadores (12) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RN002, UI008 |
| Origem da regra | RN002: Documentado; UI008: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §9.4,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarEmail<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: existsByEmailIgnoreCaseAndIdNot<br>README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT019 @RN002 @UI008 @prioridade_critica @negativo @interface
  Cenário: Rejeitar cadastro com e-mail já utilizado
    Dado que A existe e o formulário de novo aluno contém B
    Quando o usuário substitui email por "ANA.SOUZA@EXAMPLE.COM" e salva
    Então deve ver "Já existe um aluno com o e-mail informado."
    E o diálogo e os valores preenchidos devem permanecer disponíveis
    E a consulta da lista deve continuar contendo somente A
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct020"></a>

#### CT020 — Exibir junto ao campo o erro de validação devolvido pela API

| Campo | Informação |
|---|---|
| Identificador | CT020 |
| Funcionalidade | Cadastro |
| Objetivo | Exibir junto ao campo o erro de validação devolvido pela API. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | UI008, VAL003, API006 |
| Origem da regra | UI008: Documentado / detalhes inferidos; VAL003: Documentado / limite inferido; API006: Documentado |
| Evidência no repositório | README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro<br>README.md: §9.4<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: email<br>src/main/resources/static/index.html: email<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao |
| Situação da especificação | Especificado |

**Preparação e dados:** O valor é aceito pelo controle type=email do Chrome e rejeitado pelo servidor; por isso exercita o retorno da API. Se o navegador alvo rejeitar o valor antes do envio, registrar o bloqueio.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT020 @UI008 @VAL003 @API006 @prioridade_alta @negativo @interface
  Cenário: Exibir junto ao campo o erro de validação devolvido pela API
    Dado que o formulário de cadastro contém A ainda não cadastrado, exceto o e-mail "ana..b@example.com"
    Quando o usuário seleciona "Salvar aluno"
    Então uma requisição POST deve ser enviada e respondida com 400
    E deve aparecer "Informe um e-mail válido." junto ao campo e-mail, que deve receber foco
    E deve aparecer a notificação "Corrija os campos destacados para continuar."
    E o formulário deve permanecer aberto com os dados preservados e nenhum aluno criado
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** LAC014. A diferença entre as gramáticas foi observada em exploração controlada (controle válido; API 400).

<a id="ct021"></a>

#### CT021 — Permitir correção e reenvio após erro de validação

| Campo | Informação |
|---|---|
| Identificador | CT021 |
| Funcionalidade | Cadastro |
| Objetivo | Permitir correção e reenvio após erro de validação. |
| Prioridade | Alta |
| Risco associado | R06 — Falha sem recuperação ou reenvio involuntário (9) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF001, UI008 |
| Origem da regra | RF001: Documentado; UI008: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT021 @RF001 @UI008 @prioridade_alta @transicao @interface
  Cenário: Permitir correção e reenvio após erro de validação
    Dado que A ainda não existe e o formulário teve o nome "An" rejeitado
    Quando o usuário corrige o nome para "Ana Souza Ferreira" e salva
    Então os indicadores de erro do campo devem ser removidos
    E A deve ser cadastrado uma única vez com a mensagem de sucesso
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct022"></a>

#### CT022 — Manter formulário recuperável após falha de rede no envio

| Campo | Informação |
|---|---|
| Identificador | CT022 |
| Funcionalidade | Cadastro |
| Objetivo | Manter formulário recuperável após falha de rede no envio. |
| Prioridade | Alta |
| Risco associado | R06 — Falha sem recuperação ou reenvio involuntário (9) |
| Classificação principal | Exceção |
| Características secundárias | Transição (recuperação) |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Error guessing fundamentado |
| Requisitos cobertos | RF001, UI003, UI008 |
| Origem da regra | RF001: Documentado; UI003: Documentado; UI008: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §16<br>src/main/resources/static/js/alunos.js: salvarAluno/excluirAluno/alterarEstadoBotao<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Bloqueio antes do envio assegura que a primeira tentativa não foi processada. Falha após commit é outra condição, registrada como lacuna.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT022 @RF001 @UI003 @UI008 @prioridade_alta @excecao @interface
  Cenário: Manter formulário recuperável após falha de rede no envio
    Dado que o formulário contém A e o DevTools bloqueia /api/alunos antes do envio
    Quando o usuário salva
    Então deve aparecer "Não foi possível salvar o aluno. Tente novamente."
    E o formulário deve conservar os dados e reabilitar "Salvar aluno"
    Quando o bloqueio é removido e o usuário salva novamente
    Então A deve ser criado uma única vez
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct023"></a>

#### CT023 — Bloquear novo clique de gravação durante processamento

| Campo | Informação |
|---|---|
| Identificador | CT023 |
| Funcionalidade | Cadastro |
| Objetivo | Bloquear novo clique de gravação durante processamento. |
| Prioridade | Alta |
| Risco associado | R06 — Falha sem recuperação ou reenvio involuntário (9) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | UI003 |
| Origem da regra | UI003: Documentado |
| Evidência no repositório | README.md: §16<br>src/main/resources/static/js/alunos.js: salvarAluno/excluirAluno/alterarEstadoBotao |
| Situação da especificação | Especificado |

**Preparação e dados:** Registrar requisições no painel Network. Não avaliar latência como requisito de desempenho.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT023 @UI003 @prioridade_alta @transicao @interface
  Cenário: Bloquear novo clique de gravação durante processamento
    Dado que o formulário contém A e o navegador usa rede lenta
    Quando o usuário seleciona "Salvar aluno"
    Então o botão deve ficar desabilitado com "Salvando..."
    E tentativas de novo clique no botão não devem gerar outro POST
    E após a resposta o botão deve voltar a "Salvar aluno" e ficar habilitado
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct024"></a>

#### CT024 — Descartar cadastro não enviado ao fechar o formulário

| Campo | Informação |
|---|---|
| Identificador | CT024 |
| Funcionalidade | Cadastro |
| Objetivo | Descartar cadastro não enviado ao fechar o formulário. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF001, UI007 |
| Origem da regra | RF001: Documentado; UI007: Documentado |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §12<br>src/main/resources/static/js/alunos.js: abrirDialogo/devolverFoco |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Cadastro

  @CT024 @RF001 @UI007 @prioridade_alta @transicao @interface
  Esquema do Cenário: Descartar cadastro não enviado ao fechar o formulário
    Dado que A ainda não existe e o formulário contém os dados de A sem envio em andamento
    Quando o usuário fecha por "<controle>"
    Então o diálogo deve fechar sem POST e A deve continuar ausente
    Quando o usuário seleciona "Novo aluno"
    Então deve receber um formulário limpo com os padrões de cadastro

    Exemplos:
      | exemplo    | particao                     | controle          |
      | CT024-EX01 | fechar por Cancelar          | Cancelar          |
      | CT024-EX02 | fechar por Fechar formulário | Fechar formulário |
      | CT024-EX03 | fechar por Esc               | Esc               |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.


### 5.3 Listagem

<a id="ct025"></a>

#### CT025 — Exibir estado vazio quando não existem alunos

| Campo | Informação |
|---|---|
| Identificador | CT025 |
| Funcionalidade | Listagem |
| Objetivo | Exibir estado vazio quando não existem alunos. |
| Prioridade | Média |
| Risco associado | R04 — Consulta/listagem indisponível ou incorreta (6) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF002, UI004, UI005 |
| Origem da regra | RF002: Documentado; UI004: Documentado; UI005: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §9.12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: listarTodos<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: findAllByOrderByIdAsc<br>README.md: §16<br>src/main/resources/static/js/alunos.js: mostrarEstadoCarregando/mostrarEstadoErro/mostrarEstadoVazio/mostrarEstadoSemResultados<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Listagem

  @CT025 @RF002 @UI004 @UI005 @prioridade_media @positivo @interface
  Cenário: Exibir estado vazio quando não existem alunos
    Dado que a instância está sem alunos
    Quando o usuário abre a página
    Então deve ver "Nenhum aluno cadastrado" e "Cadastrar primeiro aluno"
    E o contador deve indicar "0 resultados" e os três cartões devem indicar 0
    Quando o usuário seleciona "Cadastrar primeiro aluno"
    Então deve abrir o formulário "Cadastrar aluno"
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct026"></a>

#### CT026 — Exibir os campos e contador singular quando existe um aluno

| Campo | Informação |
|---|---|
| Identificador | CT026 |
| Funcionalidade | Listagem |
| Objetivo | Exibir os campos e contador singular quando existe um aluno. |
| Prioridade | Média |
| Risco associado | R04 — Consulta/listagem indisponível ou incorreta (6) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF002, UI005 |
| Origem da regra | RF002: Documentado; UI005: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §9.12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: listarTodos<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: findAllByOrderByIdAsc<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Listagem

  @CT026 @RF002 @UI005 @prioridade_media @positivo @interface
  Cenário: Exibir os campos e contador singular quando existe um aluno
    Dado que somente A existe
    Quando o usuário carrega a página
    Então deve ver uma linha com o ID capturado, Ana Souza Ferreira, 123.456.789-01, ana.souza@example.com, Mensal e Ativo
    E deve ver as ações Editar e Excluir na linha
    E o contador deve indicar "1 resultado" e os cartões total 1, ativos 1 e inativos 0
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct027"></a>

#### CT027 — Exibir literalmente caracteres de marcação presentes no nome

| Campo | Informação |
|---|---|
| Identificador | CT027 |
| Funcionalidade | Listagem |
| Objetivo | Exibir literalmente caracteres de marcação presentes no nome. |
| Prioridade | Média |
| Risco associado | R04 — Consulta/listagem indisponível ou incorreta (6) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF002, UI013 |
| Origem da regra | RF002: Documentado; UI013: Inferido da implementação |
| Evidência no repositório | README.md: §9.12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: listarTodos<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: findAllByOrderByIdAsc<br>src/main/resources/static/js/alunos.js: criarCelula/criarLinhaAluno (textContent) |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Listagem

  @CT027 @RF002 @UI013 @prioridade_media @positivo @interface
  Cenário: Exibir literalmente caracteres de marcação presentes no nome
    Dado que existe A com nomeCompleto "Ana <b>Souza</b> & Cia", criado pela API
    Quando o usuário carrega a listagem
    Então a célula de nome deve mostrar exatamente o texto "Ana <b>Souza</b> & Cia", sem negrito
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Verificação de apresentação; não constitui teste de segurança. Os sinais de menor e maior pertencem ao dado: este Cenário não é um Esquema e não possui parâmetros.

<a id="ct028"></a>

#### CT028 — Ordenar registros e atualizar indicadores quando existem vários alunos

| Campo | Informação |
|---|---|
| Identificador | CT028 |
| Funcionalidade | Listagem |
| Objetivo | Ordenar registros e atualizar indicadores quando existem vários alunos. |
| Prioridade | Média |
| Risco associado | R04 — Consulta/listagem indisponível ou incorreta (6) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF002, UI005 |
| Origem da regra | RF002: Documentado; UI005: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §9.12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: listarTodos<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: findAllByOrderByIdAsc<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Listagem

  @CT028 @RF002 @UI005 @prioridade_media @positivo @interface
  Cenário: Ordenar registros e atualizar indicadores quando existem vários alunos
    Dado que A, B e C foram criados nessa ordem
    Quando o usuário carrega a listagem
    Então os registros devem aparecer em ordem crescente dos IDs capturados
    E o contador deve indicar "3 resultados"
    E os cartões devem indicar total 3, ativos 2 e inativos 1
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.


### 5.4 Consulta

<a id="ct029"></a>

#### CT029 — Retornar os dados completos ao consultar ID existente

| Campo | Informação |
|---|---|
| Identificador | CT029 |
| Funcionalidade | Consulta |
| Objetivo | Retornar os dados completos ao consultar ID existente. |
| Prioridade | Média |
| Risco associado | R04 — Consulta/listagem indisponível ou incorreta (6) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF003, API003, API008 |
| Origem da regra | RF003: Documentado; API003: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §§9.14,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: buscarPorId<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarAlunoNaoEncontrado<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** GET /api/alunos/ID_A; sem corpo; resposta SUCESSO; nenhum efeito de escrita.

```gherkin
# language: pt
Funcionalidade: Consulta

  @CT029 @RF003 @API003 @API008 @prioridade_media @positivo @api
  Cenário: Retornar os dados completos ao consultar ID existente
    Dado que A existe com ID_A capturado na preparação
    Quando o cliente envia GET /api/alunos/ID_A sem corpo
    Então deve receber HTTP 200 e Content-Type application/json
    E o corpo deve satisfazer o contrato SUCESSO com os valores de A
    E uma nova consulta deve manter os mesmos dados
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct030"></a>

#### CT030 — Retornar ausência ao consultar ID numérico não cadastrado

| Campo | Informação |
|---|---|
| Identificador | CT030 |
| Funcionalidade | Consulta |
| Objetivo | Retornar ausência ao consultar ID numérico não cadastrado. |
| Prioridade | Média |
| Risco associado | R04 — Consulta/listagem indisponível ou incorreta (6) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF003, API003, API008 |
| Origem da regra | RF003: Documentado; API003: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §§9.14,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: buscarPorId<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarAlunoNaoEncontrado<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** GET /api/alunos/<id>; sem corpo; Content-Type de resposta application/json; erro Not Found.

```gherkin
# language: pt
Funcionalidade: Consulta

  @CT030 @RF003 @API003 @API008 @prioridade_media @negativo @api
  Esquema do Cenário: Retornar ausência ao consultar ID numérico não cadastrado
    Dado que o ID "<id>" não aparece na listagem
    Quando o cliente envia GET /api/alunos/<id>
    Então deve receber 404 e o contrato ERRO
    E mensagem deve ser "Não foi encontrado um aluno com o ID <id>."
    E errosValidacao deve ser vazio e a lista permanecer inalterada

    Exemplos:
      | exemplo    | particao       | id                  |
      | CT030-EX01 | zero           | 0                   |
      | CT030-EX02 | negativo       | -1                  |
      | CT030-EX03 | máximo de Long | 9223372036854775807 |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.


### 5.5 Atualização

<a id="ct031"></a>

#### CT031 — Abrir edição com os dados atuais do aluno selecionado

| Campo | Informação |
|---|---|
| Identificador | CT031 |
| Funcionalidade | Atualização |
| Objetivo | Abrir edição com os dados atuais do aluno selecionado. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF004, UI006 |
| Origem da regra | RF004: Documentado; UI006: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §§12,16<br>src/main/resources/static/js/alunos.js: abrirCadastro/abrirEdicao |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Atualização

  @CT031 @RF004 @UI006 @prioridade_critica @positivo @interface
  Cenário: Abrir edição com os dados atuais do aluno selecionado
    Dado que A e B existem
    Quando o usuário seleciona Editar na linha de B
    Então deve abrir "Editar aluno" com todos os oito campos de B
    E CPF e telefone devem estar formatados, plano Anual e situação desmarcada
    E o foco inicial deve estar no nome
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Dados de B não podem ser substituídos pelos de A. O ID permanece associado à URL de atualização.

<a id="ct032"></a>

#### CT032 — Persistir cada campo editável ao salvar a edição

| Campo | Informação |
|---|---|
| Identificador | CT032 |
| Funcionalidade | Atualização |
| Objetivo | Persistir cada campo editável ao salvar a edição. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF004, UI003, UI005 |
| Origem da regra | RF004: Documentado; UI003: Documentado; UI005: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §16<br>src/main/resources/static/js/alunos.js: salvarAluno/excluirAluno/alterarEstadoBotao<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador |
| Situação da especificação | Especificado |

**Preparação e dados:** Recriar A com os dados originais antes de cada linha; sem B/C ou registros conflitantes.

```gherkin
# language: pt
Funcionalidade: Atualização

  @CT032 @RF004 @UI003 @UI005 @prioridade_critica @positivo @interface
  Esquema do Cenário: Persistir cada campo editável ao salvar a edição
    Dado que A existe e seu formulário de edição está aberto
    Quando o usuário altera somente "<campo>" para "<valor>" e salva
    Então deve ver "Aluno atualizado com sucesso." e o diálogo deve fechar
    E a consulta por ID_A deve retornar o campo alterado com os demais campos e ID preservados
    E a listagem e os cartões devem refletir os dados atualizados

    Exemplos:
      | exemplo    | particao              | campo          | valor                    |
      | CT032-EX01 | altera nomeCompleto   | nomeCompleto   | Ana Souza Oliveira       |
      | CT032-EX02 | altera cpf            | cpf            | 11122233344              |
      | CT032-EX03 | altera email          | email          | ana.oliveira@example.com |
      | CT032-EX04 | altera telefone       | telefone       | 1133224455               |
      | CT032-EX05 | altera dataNascimento | dataNascimento | 1994-05-20               |
      | CT032-EX06 | altera plano          | plano          | TRIMESTRAL               |
      | CT032-EX07 | altera dataMatricula  | dataMatricula  | D-1                      |
      | CT032-EX08 | altera ativo          | ativo          | false                    |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct033"></a>

#### CT033 — Preservar a matrícula anterior ao limpar a data na edição

| Campo | Informação |
|---|---|
| Identificador | CT033 |
| Funcionalidade | Atualização |
| Objetivo | Preservar a matrícula anterior ao limpar a data na edição. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF004, RN004 |
| Origem da regra | RF004: Documentado; RN004: Inferido da implementação |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar/normalizarTelefone |
| Situação da especificação | Especificado |

**Preparação e dados:** Criar A substituindo dataMatricula por D-1, para distinguir a preservação da aplicação da data atual.

```gherkin
# language: pt
Funcionalidade: Atualização

  @CT033 @RF004 @RN004 @prioridade_critica @positivo @interface
  Cenário: Preservar a matrícula anterior ao limpar a data na edição
    Dado que A existe com dataMatricula D-1 e está aberto para edição
    Quando o usuário limpa a data de matrícula e salva
    Então deve ver "Aluno atualizado com sucesso."
    E a consulta por ID_A deve retornar dataMatricula D-1
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Regra inferida. A ajuda do campo só define o comportamento da data vazia no cadastro (index.html: ajuda-matricula).

<a id="ct034"></a>

#### CT034 — Remover o telefone ao limpar o campo na edição

| Campo | Informação |
|---|---|
| Identificador | CT034 |
| Funcionalidade | Atualização |
| Objetivo | Remover o telefone ao limpar o campo na edição. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF004, RN004, RN005, VAL004 |
| Origem da regra | RF004: Documentado; RN004: Inferido da implementação; RN005: Inferido da implementação; VAL004: Documentado / null inferido |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar/normalizarTelefone<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar/atualizar/normalizarEmail/normalizarTelefone<br>README.md: §9.5<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: telefone<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarTelefone |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Atualização

  @CT034 @RF004 @RN004 @RN005 @VAL004 @prioridade_critica @positivo @interface
  Cenário: Remover o telefone ao limpar o campo na edição
    Dado que A existe com telefone 31999998888 e está aberto para edição
    Quando o usuário limpa o telefone e salva
    Então deve ver "Aluno atualizado com sucesso."
    E a consulta por ID_A deve retornar telefone null e os demais campos inalterados
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct035"></a>

#### CT035 — Reativar aluno inativo ao marcar a situação na edição

| Campo | Informação |
|---|---|
| Identificador | CT035 |
| Funcionalidade | Atualização |
| Objetivo | Reativar aluno inativo ao marcar a situação na edição. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF004, UI005, UI006 |
| Origem da regra | RF004: Documentado; UI005: Documentado / detalhes inferidos; UI006: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador<br>README.md: §§12,16<br>src/main/resources/static/js/alunos.js: abrirCadastro/abrirEdicao |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Atualização

  @CT035 @RF004 @UI005 @UI006 @prioridade_critica @transicao @interface
  Cenário: Reativar aluno inativo ao marcar a situação na edição
    Dado que somente B existe, inativo, e está aberto para edição
    Quando o usuário marca a situação
    Então o texto deve mudar para "Aluno ativo"
    Quando o usuário salva
    Então a linha de B deve mostrar "Ativo" e os cartões devem indicar total 1, ativos 1 e inativos 0
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct036"></a>

#### CT036 — Aceitar edição ao manter CPF e e-mail do próprio aluno

| Campo | Informação |
|---|---|
| Identificador | CT036 |
| Funcionalidade | Atualização |
| Objetivo | Aceitar edição ao manter CPF e e-mail do próprio aluno. |
| Prioridade | Crítica |
| Risco associado | R02 — Duplicidade de identificadores (12) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF004, RN001, RN002 |
| Origem da regra | RF004: Documentado; RN001: Documentado; RN002: Documentado |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §9.2,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: verificarDuplicidadeNoCadastro/verificarDuplicidadeNaAtualizacao<br>README.md: §9.4,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarEmail<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: existsByEmailIgnoreCaseAndIdNot |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Atualização

  @CT036 @RF004 @RN001 @RN002 @prioridade_critica @positivo @interface
  Cenário: Aceitar edição ao manter CPF e e-mail do próprio aluno
    Dado que A existe e está aberto para edição
    Quando o usuário muda somente o nome para "Ana S. Ferreira" e salva
    Então a atualização deve concluir sem conflito de CPF ou e-mail
    E ID, CPF e e-mail devem permanecer os mesmos
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct037"></a>

#### CT037 — Rejeitar edição ao utilizar CPF de outro aluno

| Campo | Informação |
|---|---|
| Identificador | CT037 |
| Funcionalidade | Atualização |
| Objetivo | Rejeitar edição ao utilizar CPF de outro aluno. |
| Prioridade | Crítica |
| Risco associado | R02 — Duplicidade de identificadores (12) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF004, RN001, UI008 |
| Origem da regra | RF004: Documentado; RN001: Documentado; UI008: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §9.2,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: verificarDuplicidadeNoCadastro/verificarDuplicidadeNaAtualizacao<br>README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Atualização

  @CT037 @RF004 @RN001 @UI008 @prioridade_critica @negativo @interface
  Cenário: Rejeitar edição ao utilizar CPF de outro aluno
    Dado que A e B existem e A está aberto para edição
    Quando o usuário informa cpf "98765432100" e salva
    Então deve aparecer "Já existe um aluno com o CPF informado."
    E os dados digitados devem permanecer no formulário
    E as consultas posteriores de A e B devem manter seus valores anteriores
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct038"></a>

#### CT038 — Rejeitar edição ao utilizar e-mail de outro aluno

| Campo | Informação |
|---|---|
| Identificador | CT038 |
| Funcionalidade | Atualização |
| Objetivo | Rejeitar edição ao utilizar e-mail de outro aluno. |
| Prioridade | Crítica |
| Risco associado | R02 — Duplicidade de identificadores (12) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF004, RN002, UI008 |
| Origem da regra | RF004: Documentado; RN002: Documentado; UI008: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §9.4,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarEmail<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: existsByEmailIgnoreCaseAndIdNot<br>README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Atualização

  @CT038 @RF004 @RN002 @UI008 @prioridade_critica @negativo @interface
  Cenário: Rejeitar edição ao utilizar e-mail de outro aluno
    Dado que A e B existem e A está aberto para edição
    Quando o usuário informa email "BRUNO.COSTA@EXAMPLE.COM" e salva
    Então deve aparecer "Já existe um aluno com o e-mail informado."
    E os dados digitados devem permanecer no formulário
    E as consultas posteriores de A e B devem manter seus valores anteriores
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct039"></a>

#### CT039 — Reaplicar validações locais ao editar aluno

| Campo | Informação |
|---|---|
| Identificador | CT039 |
| Funcionalidade | Atualização |
| Objetivo | Reaplicar validações locais ao editar aluno. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF004, VAL001, VAL002, VAL003, VAL004, VAL005, VAL006, VAL007, UI008 |
| Origem da regra | RF004: Documentado; VAL001: Documentado; VAL002: Documentado; VAL003: Documentado / limite inferido; VAL004: Documentado / null inferido; VAL005: Documentado; VAL006: Documentado; VAL007: Documentado; UI008: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §9.2–3<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: cpf<br>README.md: §9.4<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: email<br>src/main/resources/static/index.html: email<br>README.md: §9.5<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: telefone<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarTelefone<br>README.md: §9.6<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataNascimento<br>README.md: §9.7<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: plano<br>src/main/java/br/com/academia/alunos/entity/PlanoAcademia.java<br>README.md: §9.8<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataMatricula<br>README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Atualização

  @CT039 @RF004 @VAL001 @VAL002 @VAL003 @VAL004 @VAL005 @VAL006 @VAL007 @UI008 @prioridade_alta @negativo @interface
  Esquema do Cenário: Reaplicar validações locais ao editar aluno
    Dado que A existe e está aberto para edição
    Quando o usuário substitui somente "<campo>" por "<valor>" e salva
    Então deve ver "<mensagem>" sem envio de PUT
    E os dados persistidos de A devem permanecer iguais

    Exemplos:
      | exemplo    | particao                | campo          | valor       | mensagem                                             |
      | CT039-EX01 | nomeCompleto inválido   | nomeCompleto   | An          | O nome completo deve ter entre 3 e 100 caracteres.   |
      | CT039-EX02 | cpf inválido            | cpf            | 123         | O CPF deve conter exatamente 11 números.             |
      | CT039-EX03 | email inválido          | email          | usuario@    | Informe um e-mail válido.                            |
      | CT039-EX04 | telefone inválido       | telefone       | 123         | O telefone deve conter 10 ou 11 números.             |
      | CT039-EX05 | dataNascimento inválido | dataNascimento | D0          | A data de nascimento deve ser anterior à data atual. |
      | CT039-EX06 | plano inválido          | plano          | sem seleção | Selecione um plano.                                  |
      | CT039-EX07 | dataMatricula inválido  | dataMatricula  | D+1         | A data de matrícula não pode estar no futuro.        |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct040"></a>

#### CT040 — Preservar registro ao cancelar uma edição não enviada

| Campo | Informação |
|---|---|
| Identificador | CT040 |
| Funcionalidade | Atualização |
| Objetivo | Preservar registro ao cancelar uma edição não enviada. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF004, UI007 |
| Origem da regra | RF004: Documentado; UI007: Documentado |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §12<br>src/main/resources/static/js/alunos.js: abrirDialogo/devolverFoco |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Atualização

  @CT040 @RF004 @UI007 @prioridade_critica @transicao @interface
  Esquema do Cenário: Preservar registro ao cancelar uma edição não enviada
    Dado que A está aberto para edição e o nome foi alterado para "Nome não salvo"
    E não existe envio em andamento
    Quando o usuário fecha por "<controle>"
    Então não deve ocorrer PUT e A deve manter o nome original
    Quando o usuário reabre a edição de A
    Então o nome deve ser "Ana Souza Ferreira"

    Exemplos:
      | exemplo    | particao                     | controle          |
      | CT040-EX01 | fechar por Cancelar          | Cancelar          |
      | CT040-EX02 | fechar por Fechar formulário | Fechar formulário |
      | CT040-EX03 | fechar por Esc               | Esc               |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct041"></a>

#### CT041 — Informar inexistência ao salvar aluno removido por outro cliente

| Campo | Informação |
|---|---|
| Identificador | CT041 |
| Funcionalidade | Atualização |
| Objetivo | Informar inexistência ao salvar aluno removido por outro cliente. |
| Prioridade | Alta |
| Risco associado | R06 — Falha sem recuperação ou reenvio involuntário (9) |
| Classificação principal | Exceção |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Error guessing fundamentado |
| Requisitos cobertos | RF004, UI008, API008 |
| Origem da regra | RF004: Documentado; UI008: Documentado / detalhes inferidos; API008: Documentado |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Remover somente o registro sintético A usando DELETE manual; não reiniciar o processo.

```gherkin
# language: pt
Funcionalidade: Atualização

  @CT041 @RF004 @UI008 @API008 @prioridade_alta @excecao @interface
  Cenário: Informar inexistência ao salvar aluno removido por outro cliente
    Dado que A está aberto para edição no navegador
    E um cliente da API removeu A após a abertura
    Quando o usuário altera o nome para "Ana Editada" e salva
    Então deve ver "Não foi encontrado um aluno com o ID ID_A." com o ID real substituído
    E o formulário deve preservar a edição e reabilitar o botão
    E nenhum novo aluno deve ser criado
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.


### 5.6 Exclusão

<a id="ct042"></a>

#### CT042 — Identificar o aluno correto antes de confirmar a exclusão

| Campo | Informação |
|---|---|
| Identificador | CT042 |
| Funcionalidade | Exclusão |
| Objetivo | Identificar o aluno correto antes de confirmar a exclusão. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF005, UI001 |
| Origem da regra | RF005: Documentado; UI001: Documentado |
| Evidência no repositório | README.md: §§9.13,12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: excluir<br>src/main/resources/static/js/alunos.js: abrirExclusao<br>README.md: §§12,16<br>src/main/resources/static/js/alunos.js: abrirExclusao/tratarCliqueDelegado |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Exclusão

  @CT042 @RF005 @UI001 @prioridade_critica @positivo @interface
  Cenário: Identificar o aluno correto antes de confirmar a exclusão
    Dado que A e B existem
    Quando o usuário seleciona Excluir na linha de B
    Então deve ver "Tem certeza de que deseja excluir o aluno Bruno Costa?"
    E deve ver o aviso de remoção permanente
    E nenhuma requisição DELETE deve ocorrer antes da confirmação
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct043"></a>

#### CT043 — Preservar aluno ao fechar a confirmação sem excluir

| Campo | Informação |
|---|---|
| Identificador | CT043 |
| Funcionalidade | Exclusão |
| Objetivo | Preservar aluno ao fechar a confirmação sem excluir. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF005, UI001, UI007 |
| Origem da regra | RF005: Documentado; UI001: Documentado; UI007: Documentado |
| Evidência no repositório | README.md: §§9.13,12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: excluir<br>src/main/resources/static/js/alunos.js: abrirExclusao<br>README.md: §§12,16<br>src/main/resources/static/js/alunos.js: abrirExclusao/tratarCliqueDelegado<br>README.md: §12<br>src/main/resources/static/js/alunos.js: abrirDialogo/devolverFoco |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Exclusão

  @CT043 @RF005 @UI001 @UI007 @prioridade_critica @transicao @interface
  Esquema do Cenário: Preservar aluno ao fechar a confirmação sem excluir
    Dado que a confirmação de exclusão de A está aberta e não há envio em andamento
    Quando o usuário fecha por "<controle>"
    Então o diálogo deve fechar sem DELETE
    E A deve continuar consultável com os mesmos dados

    Exemplos:
      | exemplo    | particao                      | controle           |
      | CT043-EX01 | fechar por Cancelar           | Cancelar           |
      | CT043-EX02 | fechar por Fechar confirmação | Fechar confirmação |
      | CT043-EX03 | fechar por Esc                | Esc                |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct044"></a>

#### CT044 — Remover aluno e atualizar resumo após confirmação

| Campo | Informação |
|---|---|
| Identificador | CT044 |
| Funcionalidade | Exclusão |
| Objetivo | Remover aluno e atualizar resumo após confirmação. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF005, UI001, UI003, UI005 |
| Origem da regra | RF005: Documentado; UI001: Documentado; UI003: Documentado; UI005: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§9.13,12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: excluir<br>src/main/resources/static/js/alunos.js: abrirExclusao<br>README.md: §§12,16<br>src/main/resources/static/js/alunos.js: abrirExclusao/tratarCliqueDelegado<br>README.md: §16<br>src/main/resources/static/js/alunos.js: salvarAluno/excluirAluno/alterarEstadoBotao<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Exclusão

  @CT044 @RF005 @UI001 @UI003 @UI005 @prioridade_critica @transicao @interface
  Cenário: Remover aluno e atualizar resumo após confirmação
    Dado que A e B existem e a confirmação de exclusão de A está aberta
    Quando o usuário seleciona "Excluir aluno"
    Então deve ver "Aluno excluído com sucesso." e o diálogo deve fechar
    E a tabela deve conter somente B com contador "1 resultado"
    E os cartões devem mostrar total 1, ativos 0 e inativos 1
    E GET de ID_A deve retornar 404
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct045"></a>

#### CT045 — Exibir estado vazio ao excluir o último aluno

| Campo | Informação |
|---|---|
| Identificador | CT045 |
| Funcionalidade | Exclusão |
| Objetivo | Exibir estado vazio ao excluir o último aluno. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF005, UI004, UI005 |
| Origem da regra | RF005: Documentado; UI004: Documentado; UI005: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§9.13,12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: excluir<br>src/main/resources/static/js/alunos.js: abrirExclusao<br>README.md: §16<br>src/main/resources/static/js/alunos.js: mostrarEstadoCarregando/mostrarEstadoErro/mostrarEstadoVazio/mostrarEstadoSemResultados<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Exclusão

  @CT045 @RF005 @UI004 @UI005 @prioridade_critica @transicao @interface
  Cenário: Exibir estado vazio ao excluir o último aluno
    Dado que somente A existe e sua confirmação está aberta
    Quando o usuário confirma a exclusão
    Então deve ver "Nenhum aluno cadastrado" e "0 resultados"
    E os cartões devem indicar 0 e a tabela não deve apresentar A
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct046"></a>

#### CT046 — Bloquear cliques adicionais enquanto a exclusão é processada

| Campo | Informação |
|---|---|
| Identificador | CT046 |
| Funcionalidade | Exclusão |
| Objetivo | Bloquear cliques adicionais enquanto a exclusão é processada. |
| Prioridade | Alta |
| Risco associado | R06 — Falha sem recuperação ou reenvio involuntário (9) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | UI003, RF005 |
| Origem da regra | UI003: Documentado; RF005: Documentado |
| Evidência no repositório | README.md: §16<br>src/main/resources/static/js/alunos.js: salvarAluno/excluirAluno/alterarEstadoBotao<br>README.md: §§9.13,12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: excluir<br>src/main/resources/static/js/alunos.js: abrirExclusao |
| Situação da especificação | Especificado |

**Preparação e dados:** Restaurar a rede após a resposta; verificar quantidade de requisições no Network.

```gherkin
# language: pt
Funcionalidade: Exclusão

  @CT046 @UI003 @RF005 @prioridade_alta @transicao @interface
  Cenário: Bloquear cliques adicionais enquanto a exclusão é processada
    Dado que A existe, sua confirmação está aberta e a rede do navegador está limitada
    Quando o usuário confirma a exclusão
    Então o botão deve ficar desabilitado com "Excluindo..."
    E cliques adicionais no botão não devem gerar outro DELETE
    E ao concluir o botão deve recuperar o texto "Excluir aluno"
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct047"></a>

#### CT047 — Permitir nova tentativa após falha de rede na exclusão

| Campo | Informação |
|---|---|
| Identificador | CT047 |
| Funcionalidade | Exclusão |
| Objetivo | Permitir nova tentativa após falha de rede na exclusão. |
| Prioridade | Alta |
| Risco associado | R06 — Falha sem recuperação ou reenvio involuntário (9) |
| Classificação principal | Exceção |
| Características secundárias | Transição (recuperação) |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Error guessing fundamentado |
| Requisitos cobertos | RF005, UI003 |
| Origem da regra | RF005: Documentado; UI003: Documentado |
| Evidência no repositório | README.md: §§9.13,12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: excluir<br>src/main/resources/static/js/alunos.js: abrirExclusao<br>README.md: §16<br>src/main/resources/static/js/alunos.js: salvarAluno/excluirAluno/alterarEstadoBotao |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Exclusão

  @CT047 @RF005 @UI003 @prioridade_alta @excecao @interface
  Cenário: Permitir nova tentativa após falha de rede na exclusão
    Dado que A existe e /api/alunos/ID_A foi bloqueado antes do DELETE
    Quando o usuário confirma a exclusão de A
    Então deve ver "Não foi possível excluir o aluno. Tente novamente."
    E o diálogo deve permanecer aberto e o botão deve ser reabilitado
    Quando o bloqueio é removido e o usuário confirma novamente
    Então A deve ser excluído com mensagem de sucesso
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct048"></a>

#### CT048 — Informar ausência ao excluir registro removido por outro cliente

| Campo | Informação |
|---|---|
| Identificador | CT048 |
| Funcionalidade | Exclusão |
| Objetivo | Informar ausência ao excluir registro removido por outro cliente. |
| Prioridade | Alta |
| Risco associado | R06 — Falha sem recuperação ou reenvio involuntário (9) |
| Classificação principal | Exceção |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Error guessing fundamentado |
| Requisitos cobertos | RF005, API005 |
| Origem da regra | RF005: Documentado; API005: Documentado / 404 inferido |
| Evidência no repositório | README.md: §§9.13,12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: excluir<br>src/main/resources/static/js/alunos.js: abrirExclusao<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: excluir<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: buscarEntidade |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Exclusão

  @CT048 @RF005 @API005 @prioridade_alta @excecao @interface
  Cenário: Informar ausência ao excluir registro removido por outro cliente
    Dado que a confirmação de A está aberta
    E outro cliente removeu A pela API
    Quando o usuário confirma a exclusão na interface
    Então deve ver a mensagem de ID_A não encontrado do contrato 404
    E o diálogo deve permanecer aberto com o botão habilitado
    E os demais registros devem permanecer intactos
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.


### 5.7 Pesquisa e filtros

<a id="ct049"></a>

#### CT049 — Localizar alunos pelos critérios de pesquisa suportados

| Campo | Informação |
|---|---|
| Identificador | CT049 |
| Funcionalidade | Pesquisa e filtros |
| Objetivo | Localizar alunos pelos critérios de pesquisa suportados. |
| Prioridade | Média |
| Risco associado | R05 — Pesquisa/contadores induzirem interpretação incorreta (6) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF006, UI010, UI005 |
| Origem da regra | RF006: Documentado; UI010: Inferido da implementação; UI005: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§5,12<br>src/main/resources/static/js/alunos.js: renderizarListagem<br>src/main/resources/static/js/alunos.js: normalizarBusca/renderizarListagem<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Pesquisa e filtros

  @CT049 @RF006 @UI010 @UI005 @prioridade_media @positivo @interface
  Esquema do Cenário: Localizar alunos pelos critérios de pesquisa suportados
    Dado que a listagem contém somente A, B e C
    Quando o usuário pesquisa "<termo>" com situação Todos
    Então deve ver somente "<alunos>" com contador "<contador>"
    E os cartões devem continuar indicando total 3, ativos 2 e inativos 1
    E a pesquisa não deve emitir nova requisição HTTP

    Exemplos:
      | exemplo    | particao                   | termo                 | alunos | contador          |
      | CT049-EX01 | nome completo              | Ana Souza Ferreira    | A      | 1 de 3 resultados |
      | CT049-EX02 | parte do nome, vários      | Souza                 | A e C  | 2 de 3 resultados |
      | CT049-EX03 | CPF só dígitos             | 12345678901           | A      | 1 de 3 resultados |
      | CT049-EX04 | CPF pontuado               | 123.456.789-01        | A      | 1 de 3 resultados |
      | CT049-EX05 | e-mail completo            | ana.souza@example.com | A      | 1 de 3 resultados |
      | CT049-EX06 | e-mail parcial, maiúsculas | ANA.SOUZA             | A      | 1 de 3 resultados |
      | CT049-EX07 | sem acento                 | joao                  | C      | 1 de 3 resultados |
      | CT049-EX08 | espaços externos           | ␠␠SOUZA␠␠             | A e C  | 2 de 3 resultados |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct050"></a>

#### CT050 — Combinar termo e situação por interseção

| Campo | Informação |
|---|---|
| Identificador | CT050 |
| Funcionalidade | Pesquisa e filtros |
| Objetivo | Combinar termo e situação por interseção. |
| Prioridade | Média |
| Risco associado | R05 — Pesquisa/contadores induzirem interpretação incorreta (6) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Tabela de decisão |
| Requisitos cobertos | RF006, UI005, UI010 |
| Origem da regra | RF006: Documentado; UI005: Documentado / detalhes inferidos; UI010: Inferido da implementação |
| Evidência no repositório | README.md: §§5,12<br>src/main/resources/static/js/alunos.js: renderizarListagem<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador<br>src/main/resources/static/js/alunos.js: normalizarBusca/renderizarListagem |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Pesquisa e filtros

  @CT050 @RF006 @UI005 @UI010 @prioridade_media @positivo @interface
  Esquema do Cenário: Combinar termo e situação por interseção
    Dado que somente A, B e C estão carregados
    Quando o usuário pesquisa "<termo>" e seleciona "<filtro>"
    Então deve ver "<alunos>" e o contador "<contador>"
    E o resumo deve permanecer total 3, ativos 2 e inativos 1

    Exemplos:
      | exemplo    | particao                        | termo | filtro   | alunos   | contador          |
      | CT050-EX01 | termo ausente; filtro Todos     | VAZIO | Todos    | A, B e C | 3 resultados      |
      | CT050-EX02 | termo ausente; filtro Ativos    | VAZIO | Ativos   | A e C    | 2 de 3 resultados |
      | CT050-EX03 | termo ausente; filtro Inativos  | VAZIO | Inativos | B        | 1 de 3 resultados |
      | CT050-EX04 | termo presente; filtro Ativos   | Souza | Ativos   | A e C    | 2 de 3 resultados |
      | CT050-EX05 | termo presente; filtro Inativos | Bruno | Inativos | B        | 1 de 3 resultados |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct051"></a>

#### CT051 — Distinguir ausência de correspondências de banco vazio

| Campo | Informação |
|---|---|
| Identificador | CT051 |
| Funcionalidade | Pesquisa e filtros |
| Objetivo | Distinguir ausência de correspondências de banco vazio. |
| Prioridade | Média |
| Risco associado | R05 — Pesquisa/contadores induzirem interpretação incorreta (6) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Tabela de decisão |
| Requisitos cobertos | RF006, UI004, UI005 |
| Origem da regra | RF006: Documentado; UI004: Documentado; UI005: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§5,12<br>src/main/resources/static/js/alunos.js: renderizarListagem<br>README.md: §16<br>src/main/resources/static/js/alunos.js: mostrarEstadoCarregando/mostrarEstadoErro/mostrarEstadoVazio/mostrarEstadoSemResultados<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Pesquisa e filtros

  @CT051 @RF006 @UI004 @UI005 @prioridade_media @negativo @interface
  Esquema do Cenário: Distinguir ausência de correspondências de banco vazio
    Dado que A, B e C estão carregados
    Quando o usuário pesquisa "<termo>" com filtro "<filtro>"
    Então deve ver "Nenhum aluno encontrado" e "0 de 3 resultados"
    E deve existir a ação "Limpar pesquisa e filtros"
    E não deve aparecer "Nenhum aluno cadastrado"

    Exemplos:
      | exemplo    | particao                  | termo           | filtro   |
      | CT051-EX01 | termo sem correspondência | Inexistente XYZ | Todos    |
      | CT051-EX02 | termo só em ativos        | Souza           | Inativos |
      | CT051-EX03 | termo só em inativos      | Bruno           | Ativos   |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct052"></a>

#### CT052 — Restaurar a lista ao limpar pesquisa e filtros

| Campo | Informação |
|---|---|
| Identificador | CT052 |
| Funcionalidade | Pesquisa e filtros |
| Objetivo | Restaurar a lista ao limpar pesquisa e filtros. |
| Prioridade | Média |
| Risco associado | R05 — Pesquisa/contadores induzirem interpretação incorreta (6) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF006, UI005, UI010 |
| Origem da regra | RF006: Documentado; UI005: Documentado / detalhes inferidos; UI010: Inferido da implementação |
| Evidência no repositório | README.md: §§5,12<br>src/main/resources/static/js/alunos.js: renderizarListagem<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador<br>src/main/resources/static/js/alunos.js: normalizarBusca/renderizarListagem |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Pesquisa e filtros

  @CT052 @RF006 @UI005 @UI010 @prioridade_media @transicao @interface
  Cenário: Restaurar a lista ao limpar pesquisa e filtros
    Dado que A, B e C estão carregados e a busca Souza com filtro Inativos não tem resultados
    Quando o usuário seleciona "Limpar pesquisa e filtros"
    Então a pesquisa deve ficar vazia e a situação deve voltar a Todos
    E A, B e C devem aparecer com "3 resultados"
    E o foco deve retornar ao campo de pesquisa
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct053"></a>

#### CT053 — Preservar filtro de situação ao apagar somente a pesquisa

| Campo | Informação |
|---|---|
| Identificador | CT053 |
| Funcionalidade | Pesquisa e filtros |
| Objetivo | Preservar filtro de situação ao apagar somente a pesquisa. |
| Prioridade | Média |
| Risco associado | R05 — Pesquisa/contadores induzirem interpretação incorreta (6) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF006 |
| Origem da regra | RF006: Documentado |
| Evidência no repositório | README.md: §§5,12<br>src/main/resources/static/js/alunos.js: renderizarListagem |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Pesquisa e filtros

  @CT053 @RF006 @prioridade_media @transicao @interface
  Cenário: Preservar filtro de situação ao apagar somente a pesquisa
    Dado que A, B e C estão carregados com pesquisa Ana e filtro Ativos
    Quando o usuário apaga todo o texto da pesquisa
    Então devem aparecer A e C com "2 de 3 resultados"
    E o filtro deve continuar em Ativos
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct054"></a>

#### CT054 — Reaplicar filtros ao atualizar dados de um aluno

| Campo | Informação |
|---|---|
| Identificador | CT054 |
| Funcionalidade | Pesquisa e filtros |
| Objetivo | Reaplicar filtros ao atualizar dados de um aluno. |
| Prioridade | Média |
| Risco associado | R05 — Pesquisa/contadores induzirem interpretação incorreta (6) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF004, RF006, UI005 |
| Origem da regra | RF004: Documentado; RF006: Documentado; UI005: Documentado / detalhes inferidos |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §§5,12<br>src/main/resources/static/js/alunos.js: renderizarListagem<br>README.md: §12<br>src/main/resources/static/js/alunos.js: atualizarResumo/atualizarContador |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Pesquisa e filtros

  @CT054 @RF004 @RF006 @UI005 @prioridade_media @transicao @interface
  Cenário: Reaplicar filtros ao atualizar dados de um aluno
    Dado que A, B e C estão carregados com pesquisa Ana e filtro Ativos
    Quando o usuário edita A, desmarca a situação e salva
    Então A não deve continuar na lista filtrada e o contador deve indicar "0 de 3 resultados"
    E os cartões devem indicar total 3, ativos 1 e inativos 2
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.


### 5.8 Interface e interação

<a id="ct055"></a>

#### CT055 — Posicionar o foco inicial ao abrir cada diálogo

| Campo | Informação |
|---|---|
| Identificador | CT055 |
| Funcionalidade | Interface e interação |
| Objetivo | Posicionar o foco inicial ao abrir cada diálogo. |
| Prioridade | Média |
| Risco associado | R07 — Interação por teclado ou apresentação impedir operação (4) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | UI007, UI009 |
| Origem da regra | UI007: Documentado; UI009: Documentado |
| Evidência no repositório | README.md: §12<br>src/main/resources/static/js/alunos.js: abrirDialogo/devolverFoco<br>src/main/resources/static/index.html: dialogo-ajuda/pular-conteudo |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Interface e interação

  @CT055 @UI007 @UI009 @prioridade_media @positivo @interface
  Esquema do Cenário: Posicionar o foco inicial ao abrir cada diálogo
    Dado que A está carregado e nenhum diálogo está aberto
    Quando o usuário aciona "<acao>" pelo teclado
    Então deve abrir o diálogo correspondente com foco em "<foco>"

    Exemplos:
      | exemplo    | particao                 | acao         | foco                      |
      | CT055-EX01 | diálogo por Novo aluno   | Novo aluno   | Nome completo             |
      | CT055-EX02 | diálogo por Editar de A  | Editar de A  | Nome completo             |
      | CT055-EX03 | diálogo por Excluir de A | Excluir de A | Excluir aluno             |
      | CT055-EX04 | diálogo por Ajuda        | Ajuda        | Fechar ajuda no cabeçalho |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct056"></a>

#### CT056 — Devolver foco ao acionador após fechar diálogo sem gravação

| Campo | Informação |
|---|---|
| Identificador | CT056 |
| Funcionalidade | Interface e interação |
| Objetivo | Devolver foco ao acionador após fechar diálogo sem gravação. |
| Prioridade | Média |
| Risco associado | R07 — Interação por teclado ou apresentação impedir operação (4) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | UI007 |
| Origem da regra | UI007: Documentado |
| Evidência no repositório | README.md: §12<br>src/main/resources/static/js/alunos.js: abrirDialogo/devolverFoco |
| Situação da especificação | Especificado |

**Preparação e dados:** A deve estar na lista para editar/excluir. Sem atualização concorrente da tabela.

```gherkin
# language: pt
Funcionalidade: Interface e interação

  @CT056 @UI007 @prioridade_media @transicao @interface
  Esquema do Cenário: Devolver foco ao acionador após fechar diálogo sem gravação
    Dado que o usuário abriu "<dialogo>" pelo controle "<origem>"
    Quando pressiona Esc sem requisição em andamento
    Então o diálogo deve fechar e o foco retornar a "<origem>"

    Exemplos:
      | exemplo    | particao            | dialogo  | origem       |
      | CT056-EX01 | diálogo de cadastro | cadastro | Novo aluno   |
      | CT056-EX02 | diálogo de edição   | edição   | Editar de A  |
      | CT056-EX03 | diálogo de exclusão | exclusão | Excluir de A |
      | CT056-EX04 | diálogo de ajuda    | ajuda    | Ajuda        |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct057"></a>

#### CT057 — Cadastrar aluno utilizando somente teclado

| Campo | Informação |
|---|---|
| Identificador | CT057 |
| Funcionalidade | Interface e interação |
| Objetivo | Cadastrar aluno utilizando somente teclado. |
| Prioridade | Média |
| Risco associado | R07 — Interação por teclado ou apresentação impedir operação (4) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF001, UI009 |
| Origem da regra | RF001: Documentado; UI009: Documentado |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §12<br>src/main/resources/static/index.html: dialogo-ajuda/pular-conteudo |
| Situação da especificação | Especificado |

**Preparação e dados:** Em macOS, habilitar navegação por teclado de todos os controles no navegador/SO; registrar navegador e versão.

```gherkin
# language: pt
Funcionalidade: Interface e interação

  @CT057 @RF001 @UI009 @prioridade_media @positivo @interface
  Cenário: Cadastrar aluno utilizando somente teclado
    Dado que A ainda não existe e a página está carregada
    Quando o usuário percorre os controles com Tab, abre Novo aluno com Enter e preenche A
    E envia o formulário com Enter a partir do campo Nome completo
    Então o cadastro deve concluir com mensagem de sucesso
    E o percurso deve apresentar foco visível e permitir alcançar todos os campos e botões
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct058"></a>

#### CT058 — Direcionar a navegação ao conteúdo principal pelo link de salto

| Campo | Informação |
|---|---|
| Identificador | CT058 |
| Funcionalidade | Interface e interação |
| Objetivo | Direcionar a navegação ao conteúdo principal pelo link de salto. |
| Prioridade | Média |
| Risco associado | R07 — Interação por teclado ou apresentação impedir operação (4) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | UI009 |
| Origem da regra | UI009: Documentado |
| Evidência no repositório | README.md: §12<br>src/main/resources/static/index.html: dialogo-ajuda/pular-conteudo |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Interface e interação

  @CT058 @UI009 @prioridade_media @positivo @interface
  Cenário: Direcionar a navegação ao conteúdo principal pelo link de salto
    Dado que a página acabou de carregar
    Quando o usuário pressiona Tab para alcançar "Pular para o conteúdo principal" e Enter
    Então o conteúdo principal deve receber o foco
    E a navegação seguinte deve prosseguir pelos controles do conteúdo
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct059"></a>

#### CT059 — Apresentar instruções de uso ao abrir a ajuda

| Campo | Informação |
|---|---|
| Identificador | CT059 |
| Funcionalidade | Interface e interação |
| Objetivo | Apresentar instruções de uso ao abrir a ajuda. |
| Prioridade | Baixa |
| Risco associado | R08 — Ajuda e notificações auxiliares inconsistentes (2) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | UI009 |
| Origem da regra | UI009: Documentado |
| Evidência no repositório | README.md: §12<br>src/main/resources/static/index.html: dialogo-ajuda/pular-conteudo |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Interface e interação

  @CT059 @UI009 @prioridade_baixa @positivo @interface
  Cenário: Apresentar instruções de uso ao abrir a ajuda
    Dado que a página está carregada
    Quando o usuário seleciona Ajuda
    Então deve ver "Como usar o sistema" e instruções para cadastrar, editar, excluir, encontrar e preencher obrigatórios
    Quando o usuário seleciona Fechar ajuda no rodapé
    Então a ajuda deve fechar sem alterar dados
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct060"></a>

#### CT060 — Preservar o formulário e focar o primeiro erro quando há vários campos inválidos

| Campo | Informação |
|---|---|
| Identificador | CT060 |
| Funcionalidade | Interface e interação |
| Objetivo | Preservar o formulário e focar o primeiro erro quando há vários campos inválidos. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | UI008, VAL001, VAL002 |
| Origem da regra | UI008: Documentado / detalhes inferidos; VAL001: Documentado; VAL002: Documentado |
| Evidência no repositório | README.md: §16<br>src/main/resources/static/js/alunos.js: tratarErroFormulario/definirErro/focarPrimeiroErro<br>README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §9.2–3<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: cpf |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Interface e interação

  @CT060 @UI008 @VAL001 @VAL002 @prioridade_alta @negativo @interface
  Cenário: Preservar o formulário e focar o primeiro erro quando há vários campos inválidos
    Dado que o formulário contém A com nome An e CPF 123
    Quando o usuário tenta salvar
    Então nome e CPF devem apresentar suas mensagens específicas e aria-invalid true
    E o foco deve estar no nome, primeiro campo inválido na ordem do formulário
    E e-mail, telefone, datas, plano e situação devem conservar os valores digitados
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct061"></a>

#### CT061 — Anunciar e encerrar notificações conforme seu tipo

| Campo | Informação |
|---|---|
| Identificador | CT061 |
| Funcionalidade | Interface e interação |
| Objetivo | Anunciar e encerrar notificações conforme seu tipo. |
| Prioridade | Baixa |
| Risco associado | R08 — Ajuda e notificações auxiliares inconsistentes (2) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | UI011 |
| Origem da regra | UI011: Documentado / temporização inferida |
| Evidência no repositório | README.md: §16<br>src/main/resources/static/js/alunos.js: notificar<br>src/main/resources/static/index.html: notificacoes |
| Situação da especificação | Especificado |

**Preparação e dados:** Inspecionar a árvore de acessibilidade; isto não constitui auditoria completa nem teste de leitor de tela.

```gherkin
# language: pt
Funcionalidade: Interface e interação

  @CT061 @UI011 @prioridade_baixa @transicao @interface
  Esquema do Cenário: Anunciar e encerrar notificações conforme seu tipo
    Dado que o usuário provocou "<evento>"
    Quando a notificação é apresentada
    Então ela deve estar na região "Mensagens do sistema" com aria-live polite e role "<role>"
    E deve ter o comportamento "<duracao>"
    E o botão Fechar mensagem deve permitir sua remoção manual

    Exemplos:
      | exemplo    | particao | evento                            | role   | duracao                                               |
      | CT061-EX01 | sucesso  | cadastro válido de A              | status | removida automaticamente após cerca de cinco segundos |
      | CT061-EX02 | erro     | tentativa de cadastro com nome An | alert  | permanece após cinco segundos até fechamento manual   |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct062"></a>

#### CT062 — Manter dados e ações acessíveis na apresentação estreita

| Campo | Informação |
|---|---|
| Identificador | CT062 |
| Funcionalidade | Interface e interação |
| Objetivo | Manter dados e ações acessíveis na apresentação estreita. |
| Prioridade | Média |
| Risco associado | R07 — Interação por teclado ou apresentação impedir operação (4) |
| Classificação principal | Limite |
| Características secundárias | Positivo |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | UI012 |
| Origem da regra | UI012: Documentado |
| Evidência no repositório | README.md: §15<br>src/main/resources/static/css/estilos.css: media max-width 680px |
| Situação da especificação | Especificado |

**Preparação e dados:** Usar modo responsivo do DevTools; verificar função e acesso, não estética subjetiva.

```gherkin
# language: pt
Funcionalidade: Interface e interação

  @CT062 @UI012 @prioridade_media @limite @interface
  Esquema do Cenário: Manter dados e ações acessíveis na apresentação estreita
    Dado que A está carregado e a largura da viewport é <largura> pixels
    Quando o usuário consulta a lista e abre a edição
    Então ID, nome, CPF, e-mail, plano e situação devem continuar disponíveis
    E Editar, Excluir, Salvar aluno e Cancelar devem permanecer alcançáveis
    E os rótulos não devem depender apenas do cabeçalho oculto em modo cartão

    Exemplos:
      | exemplo    | particao           | largura |
      | CT062-EX01 | cartão, nominal    | 320     |
      | CT062-EX02 | cartão, limite     | 680     |
      | CT062-EX03 | tabela, limite + 1 | 681     |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** README §15 documenta o modo cartão em telas estreitas, sem rolagem horizontal. O corte de 680 px é inferido do CSS. Exploração controlada: modo cartão em 320 e 680 px; em 681 px a tabela permaneceu tabular e o contêiner apresentou rolagem própria.

<a id="ct063"></a>

#### CT063 — Restaurar foco após salvar uma edição e reconstruir a listagem

| Campo | Informação |
|---|---|
| Identificador | CT063 |
| Funcionalidade | Interface e interação |
| Objetivo | Restaurar foco após salvar uma edição e reconstruir a listagem. |
| Prioridade | Média |
| Risco associado | R07 — Interação por teclado ou apresentação impedir operação (4) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | UI007, RF004 |
| Origem da regra | UI007: Documentado; RF004: Documentado |
| Evidência no repositório | README.md: §12<br>src/main/resources/static/js/alunos.js: abrirDialogo/devolverFoco<br>README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Interface e interação

  @CT063 @UI007 @RF004 @prioridade_media @transicao @interface
  Cenário: Restaurar foco após salvar uma edição e reconstruir a listagem
    Dado que A foi aberto por seu botão Editar
    Quando o usuário altera o nome para Ana Editada e salva
    Então após a atualização da tabela o foco deve retornar ao controle Editar de A conforme README §12
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** LAC002. Requisito documentado mantido como resultado esperado. Exploração controlada (Chrome headless): após salvar a edição, document.activeElement era BODY. A divergência deve ser confirmada por pessoa testadora no navegador alvo antes do registro de defeito.

<a id="ct064"></a>

#### CT064 — Definir destino de foco após remover o acionador da exclusão

| Campo | Informação |
|---|---|
| Identificador | CT064 |
| Funcionalidade | Interface e interação |
| Objetivo | Definir destino de foco após remover o acionador da exclusão. |
| Prioridade | Média |
| Risco associado | R07 — Interação por teclado ou apresentação impedir operação (4) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | UI007, RF005 |
| Origem da regra | UI007: Documentado; RF005: Documentado |
| Evidência no repositório | README.md: §12<br>src/main/resources/static/js/alunos.js: abrirDialogo/devolverFoco<br>README.md: §§9.13,12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: excluir<br>src/main/resources/static/js/alunos.js: abrirExclusao |
| Situação da especificação | Pendente de definição — [LAC003](ANALISE_E_ESTRATEGIA_DE_TESTES_FUNCIONAIS.md#lac003) |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Interface e interação

  @CT064 @UI007 @RF005 @prioridade_media @transicao @interface @pendente_definicao
  Cenário: Definir destino de foco após remover o acionador da exclusão
    Dado que A e B existem e a confirmação de A foi aberta pelo botão Excluir de A
    Quando o usuário confirma e a linha de A é removida
    Então o destino final do foco deve atender à decisão ainda pendente em LAC003
```

**Pós-condições e restauração:** Não executar como aceite antes da definição. Depois, aplicar a restauração da seção 6.

**Observações, dependências e lacunas:** LAC003. O botão original deixa de existir e não há destino alternativo especificado. Exploração controlada: o foco ficou em BODY. Não executar como critério de aceite até a definição.

<a id="ct065"></a>

#### CT065 — Informar falha de atualização da lista após gravação bem-sucedida

| Campo | Informação |
|---|---|
| Identificador | CT065 |
| Funcionalidade | Interface e interação |
| Objetivo | Informar falha de atualização da lista após gravação bem-sucedida. |
| Prioridade | Alta |
| Risco associado | R06 — Falha sem recuperação ou reenvio involuntário (9) |
| Classificação principal | Exceção |
| Características secundárias | Transição (recuperação) |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Error guessing fundamentado |
| Requisitos cobertos | RF001, UI003, UI004 |
| Origem da regra | RF001: Documentado; UI003: Documentado; UI004: Documentado |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §16<br>src/main/resources/static/js/alunos.js: salvarAluno/excluirAluno/alterarEstadoBotao<br>src/main/resources/static/js/alunos.js: mostrarEstadoCarregando/mostrarEstadoErro/mostrarEstadoVazio/mostrarEstadoSemResultados |
| Situação da especificação | Especificado |

**Preparação e dados:** No DevTools, pausar na linha de salvarAluno antes de await carregarAlunos(false), após retorno do POST; habilitar bloqueio de /api/alunos e continuar. Depois desativar bloqueio e breakpoint. Não alterar os arquivos.

```gherkin
# language: pt
Funcionalidade: Interface e interação

  @CT065 @RF001 @UI003 @UI004 @prioridade_alta @excecao @interface
  Cenário: Informar falha de atualização da lista após gravação bem-sucedida
    Dado que A ainda não existe e seu cadastro POST conclui com 201
    E a consulta GET de atualização da lista é bloqueada antes de ser enviada
    Quando a interface tenta atualizar a listagem
    Então deve apresentar o sucesso do cadastro e a falha "Não foi possível carregar os alunos."
    E deve oferecer "Tentar novamente"
    Quando o bloqueio é removido e a consulta é repetida
    Então A deve aparecer uma única vez sem novo POST
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Recurso de depuração manual necessário; o sucesso da gravação e a falha de leitura são fatos distintos.

<a id="ct066"></a>

#### CT066 — Definir efeito de fechar diálogo durante gravação em andamento

| Campo | Informação |
|---|---|
| Identificador | CT066 |
| Funcionalidade | Interface e interação |
| Objetivo | Definir efeito de fechar diálogo durante gravação em andamento. |
| Prioridade | Alta |
| Risco associado | R06 — Falha sem recuperação ou reenvio involuntário (9) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | UI007, UI003 |
| Origem da regra | UI007: Documentado; UI003: Documentado |
| Evidência no repositório | README.md: §12<br>src/main/resources/static/js/alunos.js: abrirDialogo/devolverFoco<br>README.md: §16<br>src/main/resources/static/js/alunos.js: salvarAluno/excluirAluno/alterarEstadoBotao |
| Situação da especificação | Pendente de definição — [LAC010](ANALISE_E_ESTRATEGIA_DE_TESTES_FUNCIONAIS.md#lac010) |

**Preparação e dados:** Rede lenta para observar o intervalo; não executar antes de definir o contrato.

```gherkin
# language: pt
Funcionalidade: Interface e interação

  @CT066 @UI007 @UI003 @prioridade_alta @transicao @interface @pendente_definicao
  Cenário: Definir efeito de fechar diálogo durante gravação em andamento
    Dado que o envio de A está em andamento e o botão Salvar está desabilitado
    Quando o usuário fecha o diálogo por Esc
    Então a possibilidade de cancelamento, continuidade e comunicação do resultado deve seguir LAC010
```

**Pós-condições e restauração:** Não executar como aceite antes da definição. Depois, aplicar a restauração da seção 6.

**Observações, dependências e lacunas:** LAC010. Exploração controlada: fechar por Esc não cancelou o POST, e o aluno foi gravado uma vez. Ao reabrir Novo aluno antes da resposta, o botão Salvar aluno estava desabilitado, e a resposta tardia fechou o segundo diálogo. Não assumir que fechar desfaz a gravação.


### 5.9 API REST

<a id="ct067"></a>

#### CT067 — Criar aluno e retornar representação completa com identificador

| Campo | Informação |
|---|---|
| Identificador | CT067 |
| Funcionalidade | API REST |
| Objetivo | Criar aluno e retornar representação completa com identificador. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF001, RN006, API001 |
| Origem da regra | RF001: Documentado; RN006: Documentado; API001: Documentado |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §9.10<br>src/main/java/br/com/academia/alunos/entity/Aluno.java: id<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §10<br>src/main/java/br/com/academia/alunos/dto/AlunoResponse.java |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** POST /api/alunos; corpo A completo; resposta SUCESSO; cria exatamente um registro. Não exigir Location, pois não é definido.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT067 @RF001 @RN006 @API001 @prioridade_alta @positivo @api
  Cenário: Criar aluno e retornar representação completa com identificador
    Dado que A ainda não existe
    Quando o cliente envia POST /api/alunos com o JSON A
    Então deve receber 201, Content-Type application/json e o contrato SUCESSO
    E o ID deve ser um inteiro gerado e os oito campos devem corresponder a A
    E GET do ID retornado deve apresentar os mesmos valores
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct068"></a>

#### CT068 — Aplicar valores padrão ao cadastrar com opcionais ausentes ou nulos

| Campo | Informação |
|---|---|
| Identificador | CT068 |
| Funcionalidade | API REST |
| Objetivo | Aplicar valores padrão ao cadastrar com opcionais ausentes ou nulos. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF001, RN003, VAL004, API001 |
| Origem da regra | RF001: Documentado; RN003: Documentado / extensão inferida para null; VAL004: Documentado / null inferido; API001: Documentado |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §9.8–9<br>README.md: §9.5<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: telefone<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarTelefone<br>README.md: §10<br>src/main/java/br/com/academia/alunos/dto/AlunoResponse.java |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Corpo A com a substituição literal da linha; Content-Type application/json; resposta SUCESSO.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT068 @RF001 @RN003 @VAL004 @API001 @prioridade_alta @positivo @api
  Esquema do Cenário: Aplicar valores padrão ao cadastrar com opcionais ausentes ou nulos
    Dado que A ainda não existe
    Quando o cliente envia POST /api/alunos com A e a alteração "<alteracao>"
    Então deve receber 201 com telefone null, dataMatricula D0 e ativo true
    E a consulta posterior deve confirmar os padrões

    Exemplos:
      | exemplo    | particao              | alteracao                                         |
      | CT068-EX01 | propriedades omitidas | omitir telefone, dataMatricula e ativo            |
      | CT068-EX02 | valores null          | definir telefone, dataMatricula e ativo como null |
      | CT068-EX03 | telefone vazio        | telefone vazio; dataMatricula e ativo omitidos    |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct069"></a>

#### CT069 — Listar alunos em array ordenado conforme os registros existentes

| Campo | Informação |
|---|---|
| Identificador | CT069 |
| Funcionalidade | API REST |
| Objetivo | Listar alunos em array ordenado conforme os registros existentes. |
| Prioridade | Média |
| Risco associado | R04 — Consulta/listagem indisponível ou incorreta (6) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF002, API002 |
| Origem da regra | RF002: Documentado; API002: Documentado |
| Evidência no repositório | README.md: §9.12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: listarTodos<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: findAllByOrderByIdAsc<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: listarTodos |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** GET /api/alunos; Accept application/json; sem corpo; Content-Type da resposta application/json. Banco vazio retorna [].

```gherkin
# language: pt
Funcionalidade: API REST

  @CT069 @RF002 @API002 @prioridade_media @positivo @api
  Esquema do Cenário: Listar alunos em array ordenado conforme os registros existentes
    Dado que a preparação contém "<massa>"
    Quando o cliente envia GET /api/alunos sem corpo
    Então deve receber 200 e um array JSON com <quantidade> objetos SUCESSO
    E os objetos devem estar em ordem crescente de ID e corresponder à massa
    E nenhuma alteração deve ocorrer nos dados

    Exemplos:
      | exemplo    | particao | massa       | quantidade |
      | CT069-EX01 | nenhum   | banco vazio | 0          |
      | CT069-EX02 | um       | somente A   | 1          |
      | CT069-EX03 | vários   | A, B e C    | 3          |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct070"></a>

#### CT070 — Atualizar dados por PUT preservando a identidade do aluno

| Campo | Informação |
|---|---|
| Identificador | CT070 |
| Funcionalidade | API REST |
| Objetivo | Atualizar dados por PUT preservando a identidade do aluno. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF004, RN006, API004 |
| Origem da regra | RF004: Documentado; RN006: Documentado; API004: Documentado |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §9.10<br>src/main/java/br/com/academia/alunos/entity/Aluno.java: id<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** PUT /api/alunos/ID_A; corpo B; resposta SUCESSO com ID_A; substitui os oito campos, sem criar novo registro.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT070 @RF004 @RN006 @API004 @prioridade_critica @positivo @api
  Cenário: Atualizar dados por PUT preservando a identidade do aluno
    Dado que A existe com ID_A e B está ausente
    Quando o cliente envia PUT /api/alunos/ID_A com o JSON B completo
    Então deve receber 200 com os campos de B e o mesmo ID_A
    E GET /api/alunos deve retornar um único registro
    E GET /api/alunos/ID_A deve confirmar os novos dados
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct071"></a>

#### CT071 — Preservar matrícula e situação na edição com opcionais omitidos

| Campo | Informação |
|---|---|
| Identificador | CT071 |
| Funcionalidade | API REST |
| Objetivo | Preservar matrícula e situação na edição com opcionais omitidos. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF004, RN004, API004 |
| Origem da regra | RF004: Documentado; RN004: Inferido da implementação; API004: Documentado |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar/normalizarTelefone<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** PUT /api/alunos/ID_B; base B; resposta SUCESSO; telefone não segue a preservação dos outros opcionais.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT071 @RF004 @RN004 @API004 @prioridade_critica @positivo @api
  Esquema do Cenário: Preservar matrícula e situação na edição com opcionais omitidos
    Dado que B existe com dataMatricula D-1 e ativo false
    Quando o cliente envia PUT /api/alunos/ID_B com B e a alteração "<alteracao>"
    Então deve receber 200 mantendo dataMatricula D-1 e ativo false
    E telefone deve ser null e os demais campos permanecer iguais a B
    E GET /api/alunos/ID_B deve confirmar os valores

    Exemplos:
      | exemplo    | particao | alteracao                                         |
      | CT071-EX01 | omitidos | omitir telefone, dataMatricula e ativo            |
      | CT071-EX02 | null     | definir telefone, dataMatricula e ativo como null |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct072"></a>

#### CT072 — Normalizar dados válidos antes de persistir cadastro e atualização

| Campo | Informação |
|---|---|
| Identificador | CT072 |
| Funcionalidade | API REST |
| Objetivo | Normalizar dados válidos antes de persistir cadastro e atualização. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RN005, API001, API004 |
| Origem da regra | RN005: Inferido da implementação; API001: Documentado; API004: Documentado |
| Evidência no repositório | src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar/atualizar/normalizarEmail/normalizarTelefone<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/dto/AlunoResponse.java<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** POST /api/alunos com A ausente; PUT /api/alunos/ID_A com A presente. Corpo A com três substituições explícitas; resposta SUCESSO.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT072 @RN005 @API001 @API004 @prioridade_alta @positivo @api
  Esquema do Cenário: Normalizar dados válidos antes de persistir cadastro e atualização
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia a requisição de <operacao> com nome "  Ana Souza Ferreira  ", e-mail "ANA.SOUZA@EXAMPLE.COM" e telefone vazio
    Então deve receber <codigo> com nome "Ana Souza Ferreira", e-mail "ana.souza@example.com" e telefone null
    E os demais campos devem corresponder a A e a consulta posterior confirmar a normalização

    Exemplos:
      | exemplo    | particao    | operacao | codigo |
      | CT072-EX01 | cadastro    | POST     | 201    |
      | CT072-EX02 | atualização | PUT      | 200    |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct073"></a>

#### CT073 — Aceitar CPF sintático sem exigir dígitos verificadores

| Campo | Informação |
|---|---|
| Identificador | CT073 |
| Funcionalidade | API REST |
| Objetivo | Aceitar CPF sintático sem exigir dígitos verificadores. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | VAL002, API001, API004 |
| Origem da regra | VAL002: Documentado; API001: Documentado; API004: Documentado |
| Evidência no repositório | README.md: §9.2–3<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: cpf<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/dto/AlunoResponse.java<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** POST /api/alunos ou PUT /api/alunos/ID_A; corpo A com CPF 11111111111. README §9.3 exclui verificação semântica.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT073 @VAL002 @API001 @API004 @prioridade_alta @positivo @api
  Esquema do Cenário: Aceitar CPF sintático sem exigir dígitos verificadores
    Dado que a preparação "<operacao>" foi realizada para A e o CPF 11111111111 não pertence a outro aluno
    Quando o cliente envia <operacao> com A alterando CPF para "11111111111"
    Então deve receber <codigo> e persistir esse CPF
    E não deve exigir dígitos verificadores válidos

    Exemplos:
      | exemplo    | particao                       | operacao | codigo |
      | CT073-EX01 | cadastro: dígitos repetidos    | POST     | 201    |
      | CT073-EX02 | atualização: dígitos repetidos | PUT      | 200    |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct074"></a>

#### CT074 — Aceitar valores de nome dentro dos limites na API

| Campo | Informação |
|---|---|
| Identificador | CT074 |
| Funcionalidade | API REST |
| Objetivo | Aceitar valores de nome dentro dos limites na API. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Positivo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL001, API001, API004 |
| Origem da regra | VAL001: Documentado; API001: Documentado; API004: Documentado |
| Evidência no repositório | README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/dto/AlunoResponse.java<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Endpoint de POST/PUT definido na convenção OP; corpo A com nome substituído; resposta SUCESSO.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT074 @VAL001 @API001 @API004 @prioridade_alta @limite @api
  Esquema do Cenário: Aceitar valores de nome dentro dos limites na API
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e nomeCompleto "<valor>"
    Então deve receber <codigo> e persistir o nome com <tamanho> caracteres
    E a consulta posterior deve retornar o mesmo nome

    Exemplos:
      | exemplo    | particao             | operacao | valor              | tamanho | codigo |
      | CT074-EX01 | cadastro: mínimo     | POST     | Ana                | 3       | 201    |
      | CT074-EX02 | cadastro: nominal    | POST     | Ana Souza Ferreira | 18      | 201    |
      | CT074-EX03 | cadastro: máximo     | POST     | N100               | 100     | 201    |
      | CT074-EX04 | atualização: mínimo  | PUT      | Ana                | 3       | 200    |
      | CT074-EX05 | atualização: nominal | PUT      | Ana Souza Ferreira | 18      | 200    |
      | CT074-EX06 | atualização: máximo  | PUT      | N100               | 100     | 200    |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct075"></a>

#### CT075 — Rejeitar nome fora dos limites no cadastro e na atualização

| Campo | Informação |
|---|---|
| Identificador | CT075 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar nome fora dos limites no cadastro e na atualização. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL001, API006, API008 |
| Origem da regra | VAL001: Documentado; API006: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; Content-Type application/json; corpo A com o campo indicado substituído. Resposta ERRO; POST não cria e PUT não altera.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT075 @VAL001 @API006 @API008 @prioridade_alta @limite @api
  Esquema do Cenário: Rejeitar nome fora dos limites no cadastro e na atualização
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e nomeCompleto "<valor>"
    Então deve receber 400, erro "Bad Request" e mensagem "Corrija os campos destacados para continuar."
    E errosValidacao.nomeCompleto deve ser "<mensagem>"
    E o contrato ERRO deve conter o caminho da requisição
    E a lista deve permanecer igual à fotografia anterior à tentativa

    Exemplos:
      | exemplo    | particao                | operacao | valor | mensagem                                           |
      | CT075-EX01 | cadastro: mínimo − 1    | POST     | An    | O nome completo deve ter entre 3 e 100 caracteres. |
      | CT075-EX02 | cadastro: máximo + 1    | POST     | N101  | O nome completo deve ter entre 3 e 100 caracteres. |
      | CT075-EX03 | atualização: mínimo − 1 | PUT      | An    | O nome completo deve ter entre 3 e 100 caracteres. |
      | CT075-EX04 | atualização: máximo + 1 | PUT      | N101  | O nome completo deve ter entre 3 e 100 caracteres. |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct076"></a>

#### CT076 — Rejeitar CPF fora do formato exigido no cadastro e na atualização

| Campo | Informação |
|---|---|
| Identificador | CT076 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar CPF fora do formato exigido no cadastro e na atualização. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Negativo |
| Características secundárias | Limite |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | VAL002, API006, API008 |
| Origem da regra | VAL002: Documentado; API006: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §9.2–3<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: cpf<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; Content-Type application/json; corpo A com o campo indicado substituído. Resposta ERRO; POST não cria e PUT não altera.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT076 @VAL002 @API006 @API008 @prioridade_alta @negativo @api
  Esquema do Cenário: Rejeitar CPF fora do formato exigido no cadastro e na atualização
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e cpf "<valor>"
    Então deve receber 400, erro "Bad Request" e mensagem "Corrija os campos destacados para continuar."
    E errosValidacao.cpf deve ser "<mensagem>"
    E o contrato ERRO deve conter o caminho da requisição
    E a lista deve permanecer igual à fotografia anterior à tentativa

    Exemplos:
      | exemplo    | particao                      | operacao | valor          | mensagem                                 |
      | CT076-EX01 | cadastro: 10 dígitos          | POST     | 1234567890     | O CPF deve conter exatamente 11 números. |
      | CT076-EX02 | cadastro: 12 dígitos          | POST     | 123456789012   | O CPF deve conter exatamente 11 números. |
      | CT076-EX03 | cadastro: letras              | POST     | 12345abc901    | O CPF deve conter exatamente 11 números. |
      | CT076-EX04 | cadastro: pontuado            | POST     | 123.456.789-01 | O CPF deve conter exatamente 11 números. |
      | CT076-EX05 | cadastro: espaços externos    | POST     | ␠12345678901␠  | O CPF deve conter exatamente 11 números. |
      | CT076-EX06 | atualização: 10 dígitos       | PUT      | 1234567890     | O CPF deve conter exatamente 11 números. |
      | CT076-EX07 | atualização: 12 dígitos       | PUT      | 123456789012   | O CPF deve conter exatamente 11 números. |
      | CT076-EX08 | atualização: letras           | PUT      | 12345abc901    | O CPF deve conter exatamente 11 números. |
      | CT076-EX09 | atualização: pontuado         | PUT      | 123.456.789-01 | O CPF deve conter exatamente 11 números. |
      | CT076-EX10 | atualização: espaços externos | PUT      | ␠12345678901␠  | O CPF deve conter exatamente 11 números. |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** A linha com espaços externos é regra inferida: o padrão de 11 dígitos é avaliado antes do trim do serviço.

<a id="ct077"></a>

#### CT077 — Rejeitar e-mail com formato inválido no cadastro e na atualização

| Campo | Informação |
|---|---|
| Identificador | CT077 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar e-mail com formato inválido no cadastro e na atualização. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | VAL003, API006, API008 |
| Origem da regra | VAL003: Documentado / limite inferido; API006: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §9.4<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: email<br>src/main/resources/static/index.html: email<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; Content-Type application/json; corpo A com o campo indicado substituído. Resposta ERRO; POST não cria e PUT não altera.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT077 @VAL003 @API006 @API008 @prioridade_alta @negativo @api
  Esquema do Cenário: Rejeitar e-mail com formato inválido no cadastro e na atualização
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e email "<valor>"
    Então deve receber 400, erro "Bad Request" e mensagem "Corrija os campos destacados para continuar."
    E errosValidacao.email deve ser "<mensagem>"
    E o contrato ERRO deve conter o caminho da requisição
    E a lista deve permanecer igual à fotografia anterior à tentativa

    Exemplos:
      | exemplo    | particao                      | operacao | valor                   | mensagem                  |
      | CT077-EX01 | cadastro: sem domínio         | POST     | usuario@                | Informe um e-mail válido. |
      | CT077-EX02 | cadastro: sem arroba          | POST     | sem-arroba              | Informe um e-mail válido. |
      | CT077-EX03 | cadastro: espaços externos    | POST     | ␠ana.souza@example.com␠ | Informe um e-mail válido. |
      | CT077-EX04 | atualização: sem domínio      | PUT      | usuario@                | Informe um e-mail válido. |
      | CT077-EX05 | atualização: sem arroba       | PUT      | sem-arroba              | Informe um e-mail válido. |
      | CT077-EX06 | atualização: espaços externos | PUT      | ␠ana.souza@example.com␠ | Informe um e-mail válido. |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** A linha com espaços externos é regra inferida: @Email é avaliado antes do trim do serviço, que por isso não tem efeito sobre e-mails recebidos pela API. Comportamento confirmado em exploração controlada (POST).

<a id="ct078"></a>

#### CT078 — Rejeitar telefone fora das partições permitidas no cadastro e na atualização

| Campo | Informação |
|---|---|
| Identificador | CT078 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar telefone fora das partições permitidas no cadastro e na atualização. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL004, API006, API008 |
| Origem da regra | VAL004: Documentado / null inferido; API006: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §9.5<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: telefone<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarTelefone<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; Content-Type application/json; corpo A com o campo indicado substituído. Resposta ERRO; POST não cria e PUT não altera.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT078 @VAL004 @API006 @API008 @prioridade_alta @limite @api
  Esquema do Cenário: Rejeitar telefone fora das partições permitidas no cadastro e na atualização
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e telefone "<valor>"
    Então deve receber 400, erro "Bad Request" e mensagem "Corrija os campos destacados para continuar."
    E errosValidacao.telefone deve ser "<mensagem>"
    E o contrato ERRO deve conter o caminho da requisição
    E a lista deve permanecer igual à fotografia anterior à tentativa

    Exemplos:
      | exemplo    | particao                             | operacao | valor           | mensagem                                 |
      | CT078-EX01 | cadastro: 9 dígitos (mínimo − 1)     | POST     | 123456789       | O telefone deve conter 10 ou 11 números. |
      | CT078-EX02 | cadastro: 12 dígitos (máximo + 1)    | POST     | 123456789012    | O telefone deve conter 10 ou 11 números. |
      | CT078-EX03 | cadastro: pontuado                   | POST     | (11) 99999-8888 | O telefone deve conter 10 ou 11 números. |
      | CT078-EX04 | cadastro: letras                     | POST     | abc1234567      | O telefone deve conter 10 ou 11 números. |
      | CT078-EX05 | cadastro: só espaços                 | POST     | 3 espaços       | O telefone deve conter 10 ou 11 números. |
      | CT078-EX06 | atualização: 9 dígitos (mínimo − 1)  | PUT      | 123456789       | O telefone deve conter 10 ou 11 números. |
      | CT078-EX07 | atualização: 12 dígitos (máximo + 1) | PUT      | 123456789012    | O telefone deve conter 10 ou 11 números. |
      | CT078-EX08 | atualização: pontuado                | PUT      | (11) 99999-8888 | O telefone deve conter 10 ou 11 números. |
      | CT078-EX09 | atualização: letras                  | PUT      | abc1234567      | O telefone deve conter 10 ou 11 números. |
      | CT078-EX10 | atualização: só espaços              | PUT      | 3 espaços       | O telefone deve conter 10 ou 11 números. |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct079"></a>

#### CT079 — Rejeitar nascimento atual ou futuro no cadastro e na atualização

| Campo | Informação |
|---|---|
| Identificador | CT079 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar nascimento atual ou futuro no cadastro e na atualização. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL005, API006, API008 |
| Origem da regra | VAL005: Documentado; API006: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §9.6<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataNascimento<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; Content-Type application/json; corpo A com o campo indicado substituído. Resposta ERRO; POST não cria e PUT não altera.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT079 @VAL005 @API006 @API008 @prioridade_alta @limite @api
  Esquema do Cenário: Rejeitar nascimento atual ou futuro no cadastro e na atualização
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e dataNascimento "<valor>"
    Então deve receber 400, erro "Bad Request" e mensagem "Corrija os campos destacados para continuar."
    E errosValidacao.dataNascimento deve ser "<mensagem>"
    E o contrato ERRO deve conter o caminho da requisição
    E a lista deve permanecer igual à fotografia anterior à tentativa

    Exemplos:
      | exemplo    | particao                            | operacao | valor | mensagem                                             |
      | CT079-EX01 | cadastro: hoje (limite inválido)    | POST     | D0    | A data de nascimento deve ser anterior à data atual. |
      | CT079-EX02 | cadastro: futuro                    | POST     | D+1   | A data de nascimento deve ser anterior à data atual. |
      | CT079-EX03 | atualização: hoje (limite inválido) | PUT      | D0    | A data de nascimento deve ser anterior à data atual. |
      | CT079-EX04 | atualização: futuro                 | PUT      | D+1   | A data de nascimento deve ser anterior à data atual. |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct080"></a>

#### CT080 — Rejeitar matrícula futura no cadastro e na atualização

| Campo | Informação |
|---|---|
| Identificador | CT080 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar matrícula futura no cadastro e na atualização. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL007, API006, API008 |
| Origem da regra | VAL007: Documentado; API006: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §9.8<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataMatricula<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; Content-Type application/json; corpo A com o campo indicado substituído. Resposta ERRO; POST não cria e PUT não altera.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT080 @VAL007 @API006 @API008 @prioridade_alta @limite @api
  Esquema do Cenário: Rejeitar matrícula futura no cadastro e na atualização
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e dataMatricula "<valor>"
    Então deve receber 400, erro "Bad Request" e mensagem "Corrija os campos destacados para continuar."
    E errosValidacao.dataMatricula deve ser "<mensagem>"
    E o contrato ERRO deve conter o caminho da requisição
    E a lista deve permanecer igual à fotografia anterior à tentativa

    Exemplos:
      | exemplo    | particao              | operacao | valor | mensagem                                      |
      | CT080-EX01 | cadastro: hoje + 1    | POST     | D+1   | A data de matrícula não pode estar no futuro. |
      | CT080-EX02 | atualização: hoje + 1 | PUT      | D+1   | A data de matrícula não pode estar no futuro. |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct081"></a>

#### CT081 — Rejeitar campos obrigatórios nulos ou omitidos pela API

| Campo | Informação |
|---|---|
| Identificador | CT081 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar campos obrigatórios nulos ou omitidos pela API. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | VAL001, VAL002, VAL003, VAL005, VAL006, API006, API008 |
| Origem da regra | VAL001: Documentado; VAL002: Documentado; VAL003: Documentado / limite inferido; VAL005: Documentado; VAL006: Documentado; API006: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §9.2–3<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: cpf<br>README.md: §9.4<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: email<br>src/main/resources/static/index.html: email<br>README.md: §9.6<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataNascimento<br>README.md: §9.7<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: plano<br>src/main/java/br/com/academia/alunos/entity/PlanoAcademia.java<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; corpo A retirando uma propriedade ou usando null JSON (não string); resposta ERRO de validação.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT081 @VAL001 @VAL002 @VAL003 @VAL005 @VAL006 @API006 @API008 @prioridade_alta @negativo @api
  Esquema do Cenário: Rejeitar campos obrigatórios nulos ou omitidos pela API
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e aplica "<representacao>" ao campo "<campo>"
    Então deve receber 400 no contrato ERRO com a mensagem geral de validação
    E errosValidacao deve conter "<mensagem>" na chave "<campo>"
    E nenhuma criação ou alteração deve ser persistida

    Exemplos:
      | exemplo    | particao                            | operacao | campo          | representacao      | mensagem                            |
      | CT081-EX01 | cadastro: nomeCompleto omitido      | POST     | nomeCompleto   | omitir propriedade | O nome completo é obrigatório.      |
      | CT081-EX02 | cadastro: nomeCompleto null         | POST     | nomeCompleto   | valor null         | O nome completo é obrigatório.      |
      | CT081-EX03 | cadastro: cpf omitido               | POST     | cpf            | omitir propriedade | O CPF é obrigatório.                |
      | CT081-EX04 | cadastro: cpf null                  | POST     | cpf            | valor null         | O CPF é obrigatório.                |
      | CT081-EX05 | cadastro: email omitido             | POST     | email          | omitir propriedade | O e-mail é obrigatório.             |
      | CT081-EX06 | cadastro: email null                | POST     | email          | valor null         | O e-mail é obrigatório.             |
      | CT081-EX07 | cadastro: dataNascimento omitido    | POST     | dataNascimento | omitir propriedade | A data de nascimento é obrigatória. |
      | CT081-EX08 | cadastro: dataNascimento null       | POST     | dataNascimento | valor null         | A data de nascimento é obrigatória. |
      | CT081-EX09 | cadastro: plano omitido             | POST     | plano          | omitir propriedade | O plano é obrigatório.              |
      | CT081-EX10 | cadastro: plano null                | POST     | plano          | valor null         | O plano é obrigatório.              |
      | CT081-EX11 | atualização: nomeCompleto omitido   | PUT      | nomeCompleto   | omitir propriedade | O nome completo é obrigatório.      |
      | CT081-EX12 | atualização: nomeCompleto null      | PUT      | nomeCompleto   | valor null         | O nome completo é obrigatório.      |
      | CT081-EX13 | atualização: cpf omitido            | PUT      | cpf            | omitir propriedade | O CPF é obrigatório.                |
      | CT081-EX14 | atualização: cpf null               | PUT      | cpf            | valor null         | O CPF é obrigatório.                |
      | CT081-EX15 | atualização: email omitido          | PUT      | email          | omitir propriedade | O e-mail é obrigatório.             |
      | CT081-EX16 | atualização: email null             | PUT      | email          | valor null         | O e-mail é obrigatório.             |
      | CT081-EX17 | atualização: dataNascimento omitido | PUT      | dataNascimento | omitir propriedade | A data de nascimento é obrigatória. |
      | CT081-EX18 | atualização: dataNascimento null    | PUT      | dataNascimento | valor null         | A data de nascimento é obrigatória. |
      | CT081-EX19 | atualização: plano omitido          | PUT      | plano          | omitir propriedade | O plano é obrigatório.              |
      | CT081-EX20 | atualização: plano null             | PUT      | plano          | valor null         | O plano é obrigatório.              |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct082"></a>

#### CT082 — Rejeitar textos obrigatórios vazios ou somente espaços

| Campo | Informação |
|---|---|
| Identificador | CT082 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar textos obrigatórios vazios ou somente espaços. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | VAL001, VAL002, VAL003, API006 |
| Origem da regra | VAL001: Documentado; VAL002: Documentado; VAL003: Documentado / limite inferido; API006: Documentado |
| Evidência no repositório | README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §9.2–3<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: cpf<br>README.md: §9.4<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: email<br>src/main/resources/static/index.html: email<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; corpo A com string vazia ou três espaços; ERRO de validação.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT082 @VAL001 @VAL002 @VAL003 @API006 @prioridade_alta @negativo @api
  Esquema do Cenário: Rejeitar textos obrigatórios vazios ou somente espaços
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e "<valor>" em "<campo>"
    Então deve receber 400 e o mapa errosValidacao deve conter a chave "<campo>"
    E a mensagem desse campo deve pertencer ao conjunto permitido no catálogo MSG
    E nenhum dado deve ser gravado

    Exemplos:
      | exemplo    | particao                             | operacao | campo        | valor     |
      | CT082-EX01 | cadastro: nomeCompleto vazio         | POST     | nomeCompleto | VAZIO     |
      | CT082-EX02 | cadastro: nomeCompleto só espaços    | POST     | nomeCompleto | 3 espaços |
      | CT082-EX03 | cadastro: cpf vazio                  | POST     | cpf          | VAZIO     |
      | CT082-EX04 | cadastro: cpf só espaços             | POST     | cpf          | 3 espaços |
      | CT082-EX05 | cadastro: email vazio                | POST     | email        | VAZIO     |
      | CT082-EX06 | cadastro: email só espaços           | POST     | email        | 3 espaços |
      | CT082-EX07 | atualização: nomeCompleto vazio      | PUT      | nomeCompleto | VAZIO     |
      | CT082-EX08 | atualização: nomeCompleto só espaços | PUT      | nomeCompleto | 3 espaços |
      | CT082-EX09 | atualização: cpf vazio               | PUT      | cpf          | VAZIO     |
      | CT082-EX10 | atualização: cpf só espaços          | PUT      | cpf          | 3 espaços |
      | CT082-EX11 | atualização: email vazio             | PUT      | email        | VAZIO     |
      | CT082-EX12 | atualização: email só espaços        | PUT      | email        | 3 espaços |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** LAC004. Exploração controlada: em 30 envios idênticos de cada entrada, as duas mensagens do conjunto ocorreram para nome vazio, CPF vazio, CPF com espaços e e-mail com espaços. A mensagem única não é determinística; exigir uma delas depende de decisão.

<a id="ct083"></a>

#### CT083 — Aceitar telefone nas partições opcionais e limites permitidos

| Campo | Informação |
|---|---|
| Identificador | CT083 |
| Funcionalidade | API REST |
| Objetivo | Aceitar telefone nas partições opcionais e limites permitidos. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Positivo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL004, API001, API004 |
| Origem da regra | VAL004: Documentado / null inferido; API001: Documentado; API004: Documentado |
| Evidência no repositório | README.md: §9.5<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: telefone<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarTelefone<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/dto/AlunoResponse.java<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; null representa literal JSON; resposta SUCESSO.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT083 @VAL004 @API001 @API004 @prioridade_alta @limite @api
  Esquema do Cenário: Aceitar telefone nas partições opcionais e limites permitidos
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e telefone "<valor>"
    Então deve receber <codigo> com telefone "<persistido>"
    E a consulta posterior deve confirmar esse valor

    Exemplos:
      | exemplo    | particao                         | operacao | valor       | persistido  | codigo |
      | CT083-EX01 | cadastro: vazio                  | POST     | VAZIO       | null        | 201    |
      | CT083-EX02 | cadastro: null                   | POST     | null        | null        | 201    |
      | CT083-EX03 | cadastro: 10 dígitos (mínimo)    | POST     | 1133224455  | 1133224455  | 201    |
      | CT083-EX04 | cadastro: 11 dígitos (máximo)    | POST     | 11999998888 | 11999998888 | 201    |
      | CT083-EX05 | atualização: vazio               | PUT      | VAZIO       | null        | 200    |
      | CT083-EX06 | atualização: null                | PUT      | null        | null        | 200    |
      | CT083-EX07 | atualização: 10 dígitos (mínimo) | PUT      | 1133224455  | 1133224455  | 200    |
      | CT083-EX08 | atualização: 11 dígitos (máximo) | PUT      | 11999998888 | 11999998888 | 200    |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct084"></a>

#### CT084 — Aceitar datas válidas nos limites permitidos

| Campo | Informação |
|---|---|
| Identificador | CT084 |
| Funcionalidade | API REST |
| Objetivo | Aceitar datas válidas nos limites permitidos. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Positivo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL005, VAL007, API001, API004 |
| Origem da regra | VAL005: Documentado; VAL007: Documentado; API001: Documentado; API004: Documentado |
| Evidência no repositório | README.md: §9.6<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataNascimento<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §9.8<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataMatricula<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/dto/AlunoResponse.java<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; substituir D por data ISO calculada no servidor; resposta SUCESSO.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT084 @VAL005 @VAL007 @API001 @API004 @prioridade_alta @limite @api
  Esquema do Cenário: Aceitar datas válidas nos limites permitidos
    Dado que a preparação "<operacao>" foi realizada para A no dia D0
    Quando o cliente envia <operacao> com A alterando "<campo>" para "<valor>"
    Então deve receber <codigo> e persistir a data informada
    E a consulta posterior deve confirmar a data

    Exemplos:
      | exemplo    | particao                                      | operacao | campo          | valor      | codigo |
      | CT084-EX01 | cadastro: nascimento ontem (limite válido)    | POST     | dataNascimento | D-1        | 201    |
      | CT084-EX02 | cadastro: nascimento nominal                  | POST     | dataNascimento | 1995-05-20 | 201    |
      | CT084-EX03 | cadastro: matrícula ontem                     | POST     | dataMatricula  | D-1        | 201    |
      | CT084-EX04 | cadastro: matrícula hoje (limite válido)      | POST     | dataMatricula  | D0         | 201    |
      | CT084-EX05 | atualização: nascimento ontem (limite válido) | PUT      | dataNascimento | D-1        | 200    |
      | CT084-EX06 | atualização: nascimento nominal               | PUT      | dataNascimento | 1995-05-20 | 200    |
      | CT084-EX07 | atualização: matrícula ontem                  | PUT      | dataMatricula  | D-1        | 200    |
      | CT084-EX08 | atualização: matrícula hoje (limite válido)   | PUT      | dataMatricula  | D0         | 200    |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct085"></a>

#### CT085 — Aceitar cada plano permitido pela API

| Campo | Informação |
|---|---|
| Identificador | CT085 |
| Funcionalidade | API REST |
| Objetivo | Aceitar cada plano permitido pela API. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | VAL006, API001, API004 |
| Origem da regra | VAL006: Documentado; API001: Documentado; API004: Documentado |
| Evidência no repositório | README.md: §9.7<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: plano<br>src/main/java/br/com/academia/alunos/entity/PlanoAcademia.java<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/dto/AlunoResponse.java<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; corpo A com plano da linha; resposta SUCESSO.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT085 @VAL006 @API001 @API004 @prioridade_alta @positivo @api
  Esquema do Cenário: Aceitar cada plano permitido pela API
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e plano "<plano>"
    Então deve receber <codigo> e o plano "<plano>" na resposta e na consulta posterior

    Exemplos:
      | exemplo    | particao                | operacao | plano      | codigo |
      | CT085-EX01 | cadastro: MENSAL        | POST     | MENSAL     | 201    |
      | CT085-EX02 | cadastro: TRIMESTRAL    | POST     | TRIMESTRAL | 201    |
      | CT085-EX03 | cadastro: ANUAL         | POST     | ANUAL      | 201    |
      | CT085-EX04 | atualização: MENSAL     | PUT      | MENSAL     | 200    |
      | CT085-EX05 | atualização: TRIMESTRAL | PUT      | TRIMESTRAL | 200    |
      | CT085-EX06 | atualização: ANUAL      | PUT      | ANUAL      | 200    |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct086"></a>

#### CT086 — Rejeitar plano informado por valor numérico

| Campo | Informação |
|---|---|
| Identificador | CT086 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar plano informado por valor numérico. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | VAL006, API008 |
| Origem da regra | VAL006: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §9.7<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: plano<br>src/main/java/br/com/academia/alunos/entity/PlanoAcademia.java<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; corpo A com plano numérico sem aspas. O README §9.7 aceita somente MENSAL, TRIMESTRAL e ANUAL. A redação da mensagem não é critério deste caso.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT086 @VAL006 @API008 @prioridade_alta @negativo @api
  Esquema do Cenário: Rejeitar plano informado por valor numérico
    Dado que a preparação "<operacao>" foi realizada para A com plano ANUAL
    Quando o cliente envia <operacao> com A e plano como número JSON <valor>
    Então deve receber 400 no contrato ERRO
    E nenhum dado deve ser criado ou alterado

    Exemplos:
      | exemplo    | particao                         | operacao | valor |
      | CT086-EX01 | cadastro: posição existente      | POST     | 0     |
      | CT086-EX02 | cadastro: posição inexistente    | POST     | 3     |
      | CT086-EX03 | atualização: posição existente   | PUT      | 0     |
      | CT086-EX04 | atualização: posição inexistente | PUT      | 3     |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** LAC016. Exploração controlada: POST com plano 0 retornou 201 e gravou MENSAL (desserialização por posição da enumeração). O valor 3 está fora das posições existentes e não foi explorado.

<a id="ct087"></a>

#### CT087 — Persistir situação explícita sem substituí-la pelo padrão

| Campo | Informação |
|---|---|
| Identificador | CT087 |
| Funcionalidade | API REST |
| Objetivo | Persistir situação explícita sem substituí-la pelo padrão. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RN003, RF004, API001, API004 |
| Origem da regra | RN003: Documentado / extensão inferida para null; RF004: Documentado; API001: Documentado; API004: Documentado |
| Evidência no repositório | README.md: §9.8–9<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/dto/AlunoResponse.java<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; boolean JSON, não string; resposta SUCESSO.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT087 @RN003 @RF004 @API001 @API004 @prioridade_alta @transicao @api
  Esquema do Cenário: Persistir situação explícita sem substituí-la pelo padrão
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e ativo <ativo>
    Então deve receber <codigo> e ativo <ativo>
    E a consulta posterior deve confirmar a situação

    Exemplos:
      | exemplo    | particao             | operacao | ativo | codigo |
      | CT087-EX01 | cadastro: ativo      | POST     | true  | 201    |
      | CT087-EX02 | cadastro: inativo    | POST     | false | 201    |
      | CT087-EX03 | atualização: ativo   | PUT      | true  | 200    |
      | CT087-EX04 | atualização: inativo | PUT      | false | 200    |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct088"></a>

#### CT088 — Rejeitar conteúdo que não possa ser convertido em requisição válida

| Campo | Informação |
|---|---|
| Identificador | CT088 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar conteúdo que não possa ser convertido em requisição válida. |
| Prioridade | Alta |
| Risco associado | R09 — Contrato HTTP ambíguo ou erro mal classificado (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | VAL006, API009, API008 |
| Origem da regra | VAL006: Documentado; API009: Inferido da implementação; API008: Documentado |
| Evidência no repositório | README.md: §9.7<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: plano<br>src/main/java/br/com/academia/alunos/entity/PlanoAcademia.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarJsonInvalido<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; application/json; usar corpo A com alteração indicada, exceto linhas que substituem o corpo inteiro. Resposta ERRO Bad Request.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT088 @VAL006 @API009 @API008 @prioridade_alta @negativo @api
  Esquema do Cenário: Rejeitar conteúdo que não possa ser convertido em requisição válida
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com a alteração "<alteracao>" no corpo
    Então deve receber 400 e mensagem "Não foi possível interpretar os dados. Confira datas e opções informadas."
    E o contrato ERRO deve conter errosValidacao vazio
    E nenhum dado deve ser criado ou alterado

    Exemplos:
      | exemplo    | particao                               | operacao | alteracao                               |
      | CT088-EX01 | cadastro: JSON malformado              | POST     | corpo literal {"nomeCompleto":          |
      | CT088-EX02 | cadastro: sem corpo                    | POST     | corpo ausente                           |
      | CT088-EX03 | cadastro: plano desconhecido           | POST     | plano como string "SEMANAL"             |
      | CT088-EX04 | cadastro: plano em minúsculas          | POST     | plano como string "mensal"              |
      | CT088-EX05 | cadastro: data fora de ISO             | POST     | dataNascimento como string "20/05/1995" |
      | CT088-EX06 | cadastro: data inexistente             | POST     | dataNascimento como string "2026-02-30" |
      | CT088-EX07 | cadastro: tipo incompatível (ativo)    | POST     | ativo como objeto {}                    |
      | CT088-EX08 | cadastro: tipo incompatível (cpf)      | POST     | cpf como objeto {}                      |
      | CT088-EX09 | cadastro: raiz em array                | POST     | corpo literal []                        |
      | CT088-EX10 | cadastro: plano vazio                  | POST     | plano como string vazia ""              |
      | CT088-EX11 | cadastro: corpo null                   | POST     | corpo literal null                      |
      | CT088-EX12 | atualização: JSON malformado           | PUT      | corpo literal {"nomeCompleto":          |
      | CT088-EX13 | atualização: sem corpo                 | PUT      | corpo ausente                           |
      | CT088-EX14 | atualização: plano desconhecido        | PUT      | plano como string "SEMANAL"             |
      | CT088-EX15 | atualização: plano em minúsculas       | PUT      | plano como string "mensal"              |
      | CT088-EX16 | atualização: data fora de ISO          | PUT      | dataNascimento como string "20/05/1995" |
      | CT088-EX17 | atualização: data inexistente          | PUT      | dataNascimento como string "2026-02-30" |
      | CT088-EX18 | atualização: tipo incompatível (ativo) | PUT      | ativo como objeto {}                    |
      | CT088-EX19 | atualização: tipo incompatível (cpf)   | PUT      | cpf como objeto {}                      |
      | CT088-EX20 | atualização: raiz em array             | PUT      | corpo literal []                        |
      | CT088-EX21 | atualização: plano vazio               | PUT      | plano como string vazia ""              |
      | CT088-EX22 | atualização: corpo null                | PUT      | corpo literal null                      |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct089"></a>

#### CT089 — Informar todos os campos inválidos em uma mesma resposta de validação

| Campo | Informação |
|---|---|
| Identificador | CT089 |
| Funcionalidade | API REST |
| Objetivo | Informar todos os campos inválidos em uma mesma resposta de validação. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | API006, API008, VAL001, VAL002, VAL005 |
| Origem da regra | API006: Documentado; API008: Documentado; VAL001: Documentado; VAL002: Documentado; VAL005: Documentado |
| Evidência no repositório | README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro<br>README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §9.2–3<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: cpf<br>README.md: §9.6<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataNascimento |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** POST /api/alunos; corpo A com três substituições; contrato ERRO. Ordem das propriedades JSON é irrelevante.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT089 @API006 @API008 @VAL001 @VAL002 @VAL005 @prioridade_alta @negativo @api
  Cenário: Informar todos os campos inválidos em uma mesma resposta de validação
    Dado que A ainda não existe
    Quando o cliente envia POST com A, nome An, CPF 123 e dataNascimento D0
    Então deve receber 400 com errosValidacao contendo nomeCompleto, cpf e dataNascimento
    E as mensagens devem ser respectivamente MSG-NOME, MSG-CPF e MSG-NASCIMENTO
    E nenhum aluno deve ser criado
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct090"></a>

#### CT090 — Aceitar e-mail no limite máximo de tamanho

| Campo | Informação |
|---|---|
| Identificador | CT090 |
| Funcionalidade | API REST |
| Objetivo | Aceitar e-mail no limite máximo de tamanho. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Positivo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL003, API001, API004 |
| Origem da regra | VAL003: Documentado / limite inferido; API001: Documentado; API004: Documentado |
| Evidência no repositório | README.md: §9.4<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: email<br>src/main/resources/static/index.html: email<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/dto/AlunoResponse.java<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; e-mails literais do catálogo; resposta SUCESSO.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT090 @VAL003 @API001 @API004 @prioridade_alta @limite @api
  Esquema do Cenário: Aceitar e-mail no limite máximo de tamanho
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e email "<email>"
    Então deve receber <codigo> com o e-mail completo de <tamanho> caracteres
    E a consulta posterior deve retornar o mesmo valor

    Exemplos:
      | exemplo    | particao                | operacao | email | tamanho | codigo |
      | CT090-EX01 | cadastro: máximo − 1    | POST     | E253  | 253     | 201    |
      | CT090-EX02 | cadastro: máximo        | POST     | E254  | 254     | 201    |
      | CT090-EX03 | atualização: máximo − 1 | PUT      | E253  | 253     | 200    |
      | CT090-EX04 | atualização: máximo     | PUT      | E254  | 254     | 200    |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct091"></a>

#### CT091 — Rejeitar e-mail acima do tamanho máximo

| Campo | Informação |
|---|---|
| Identificador | CT091 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar e-mail acima do tamanho máximo. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL003, API006 |
| Origem da regra | VAL003: Documentado / limite inferido; API006: Documentado |
| Evidência no repositório | README.md: §9.4<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: email<br>src/main/resources/static/index.html: email<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; corpo A com E255; ERRO de validação. E255 mantém local e rótulos DNS dentro dos limites individuais para isolar @Size.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT091 @VAL003 @API006 @prioridade_alta @limite @api
  Esquema do Cenário: Rejeitar e-mail acima do tamanho máximo
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e email E255
    Então deve receber 400 com errosValidacao.email "O e-mail deve ter no máximo 254 caracteres."
    E nenhum dado deve ser criado ou alterado

    Exemplos:
      | exemplo    | particao                | operacao |
      | CT091-EX01 | cadastro: máximo + 1    | POST     |
      | CT091-EX02 | atualização: máximo + 1 | PUT      |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct092"></a>

#### CT092 — Rejeitar duplicidade de CPF no cadastro e na atualização

| Campo | Informação |
|---|---|
| Identificador | CT092 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar duplicidade de CPF no cadastro e na atualização. |
| Prioridade | Crítica |
| Risco associado | R02 — Duplicidade de identificadores (12) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RN001, API007, API008 |
| Origem da regra | RN001: Documentado; API007: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §9.2,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: verificarDuplicidadeNoCadastro/verificarDuplicidadeNaAtualizacao<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarRecursoDuplicado/tratarIntegridade<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** POST /api/alunos: B ausente; PUT /api/alunos/ID_B: B existente. Corpo B com CPF de A; contrato ERRO.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT092 @RN001 @API007 @API008 @prioridade_critica @negativo @api
  Esquema do Cenário: Rejeitar duplicidade de CPF no cadastro e na atualização
    Dado que A existe e a preparação "<operacao>" foi realizada para B
    Quando o cliente envia <operacao> com B substituindo cpf por "12345678901"
    Então deve receber 409, erro "Conflict" e mensagem "Já existe um aluno com o CPF informado."
    E errosValidacao deve ser vazio
    E os registros devem permanecer iguais à fotografia anterior

    Exemplos:
      | exemplo    | particao    | operacao |
      | CT092-EX01 | cadastro    | POST     |
      | CT092-EX02 | atualização | PUT      |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct093"></a>

#### CT093 — Rejeitar e-mail duplicado independentemente de caixa

| Campo | Informação |
|---|---|
| Identificador | CT093 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar e-mail duplicado independentemente de caixa. |
| Prioridade | Crítica |
| Risco associado | R02 — Duplicidade de identificadores (12) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RN002, API007, API008 |
| Origem da regra | RN002: Documentado; API007: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §9.4,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarEmail<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: existsByEmailIgnoreCaseAndIdNot<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarRecursoDuplicado/tratarIntegridade<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Endpoint conforme OP para B; corpo B com email indicado; ERRO Conflict.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT093 @RN002 @API007 @API008 @prioridade_critica @negativo @api
  Esquema do Cenário: Rejeitar e-mail duplicado independentemente de caixa
    Dado que A existe e a preparação "<operacao>" foi realizada para B
    Quando o cliente envia <operacao> com B substituindo email por "<email>"
    Então deve receber 409 com mensagem "Já existe um aluno com o e-mail informado."
    E errosValidacao deve ser vazio e os dados persistidos não devem mudar

    Exemplos:
      | exemplo    | particao                | operacao | email                 |
      | CT093-EX01 | cadastro: igual         | POST     | ana.souza@example.com |
      | CT093-EX02 | cadastro: maiúsculas    | POST     | ANA.SOUZA@EXAMPLE.COM |
      | CT093-EX03 | atualização: igual      | PUT      | ana.souza@example.com |
      | CT093-EX04 | atualização: maiúsculas | PUT      | ANA.SOUZA@EXAMPLE.COM |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct094"></a>

#### CT094 — Aceitar atualização com os identificadores do próprio aluno

| Campo | Informação |
|---|---|
| Identificador | CT094 |
| Funcionalidade | API REST |
| Objetivo | Aceitar atualização com os identificadores do próprio aluno. |
| Prioridade | Crítica |
| Risco associado | R02 — Duplicidade de identificadores (12) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RN001, RN002, API004 |
| Origem da regra | RN001: Documentado; RN002: Documentado; API004: Documentado |
| Evidência no repositório | README.md: §9.2,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: verificarDuplicidadeNoCadastro/verificarDuplicidadeNaAtualizacao<br>README.md: §9.4,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarEmail<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: existsByEmailIgnoreCaseAndIdNot<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** PUT /api/alunos/ID_A; A com nome e email substituídos; SUCESSO.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT094 @RN001 @RN002 @API004 @prioridade_critica @positivo @api
  Cenário: Aceitar atualização com os identificadores do próprio aluno
    Dado que A e B existem
    Quando o cliente envia PUT /api/alunos/ID_A com A, nome "Ana Editada" e email "ANA.SOUZA@EXAMPLE.COM"
    Então deve receber 200 sem conflito
    E CPF e ID devem permanecer os de A, e-mail deve voltar a minúsculas e B deve permanecer intacto
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct095"></a>

#### CT095 — Priorizar conflito de CPF quando CPF e e-mail pertencem a outro aluno

| Campo | Informação |
|---|---|
| Identificador | CT095 |
| Funcionalidade | API REST |
| Objetivo | Priorizar conflito de CPF quando CPF e e-mail pertencem a outro aluno. |
| Prioridade | Crítica |
| Risco associado | R02 — Duplicidade de identificadores (12) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Tabela de decisão |
| Requisitos cobertos | RN001, RN002, API007 |
| Origem da regra | RN001: Documentado; RN002: Documentado; API007: Documentado |
| Evidência no repositório | README.md: §9.2,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: verificarDuplicidadeNoCadastro/verificarDuplicidadeNaAtualizacao<br>README.md: §9.4,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarEmail<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: existsByEmailIgnoreCaseAndIdNot<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarRecursoDuplicado/tratarIntegridade |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** POST /api/alunos; B com cpf 12345678901 e email ana.souza@example.com; ERRO Conflict.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT095 @RN001 @RN002 @API007 @prioridade_critica @negativo @api
  Cenário: Priorizar conflito de CPF quando CPF e e-mail pertencem a outro aluno
    Dado que A existe e B ainda não existe
    Quando o cliente envia POST /api/alunos com B usando CPF e e-mail de A
    Então deve receber 409 com mensagem "Já existe um aluno com o CPF informado."
    E nenhum novo registro deve existir
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Prioridade da mensagem inferida da ordem das verificações do serviço; a proibição de duplicidade é documentada.

<a id="ct096"></a>

#### CT096 — Retornar ausência ao atualizar aluno inexistente

| Campo | Informação |
|---|---|
| Identificador | CT096 |
| Funcionalidade | API REST |
| Objetivo | Retornar ausência ao atualizar aluno inexistente. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF004, API008 |
| Origem da regra | RF004: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** PUT /api/alunos/ID_A; corpo A; ERRO Not Found. Preparação pela API em instância isolada.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT096 @RF004 @API008 @prioridade_critica @negativo @api
  Cenário: Retornar ausência ao atualizar aluno inexistente
    Dado que ID_A foi capturado na criação de A e depois A foi removido
    Quando o cliente envia PUT /api/alunos/ID_A com A válido
    Então deve receber 404 com mensagem "Não foi encontrado um aluno com o ID ID_A." usando o ID real
    E errosValidacao deve ser vazio
    E nenhum registro deve ser criado
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct097"></a>

#### CT097 — Priorizar a validação do corpo ao atualizar aluno inexistente

| Campo | Informação |
|---|---|
| Identificador | CT097 |
| Funcionalidade | API REST |
| Objetivo | Priorizar a validação do corpo ao atualizar aluno inexistente. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Tabela de decisão |
| Requisitos cobertos | RF004, API006, API012 |
| Origem da regra | RF004: Documentado; API006: Documentado; API012: Inferido da implementação |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar (@Valid antes da chamada ao serviço)<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar/buscarEntidade |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** PUT /api/alunos/ID_A; corpo A com nome inválido; ERRO Bad Request.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT097 @RF004 @API006 @API012 @prioridade_critica @negativo @api
  Cenário: Priorizar a validação do corpo ao atualizar aluno inexistente
    Dado que ID_A foi capturado na criação de A e depois A foi removido
    Quando o cliente envia PUT /api/alunos/ID_A com A e nomeCompleto "An"
    Então deve receber 400 com errosValidacao.nomeCompleto "O nome completo deve ter entre 3 e 100 caracteres."
    E nenhum registro deve ser criado
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Precedência inferida da implementação e observada em exploração controlada.

<a id="ct098"></a>

#### CT098 — Excluir registro e retornar resposta sem corpo

| Campo | Informação |
|---|---|
| Identificador | CT098 |
| Funcionalidade | API REST |
| Objetivo | Excluir registro e retornar resposta sem corpo. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF005, API005 |
| Origem da regra | RF005: Documentado; API005: Documentado / 404 inferido |
| Evidência no repositório | README.md: §§9.13,12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: excluir<br>src/main/resources/static/js/alunos.js: abrirExclusao<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: excluir<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: buscarEntidade |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** DELETE /api/alunos/ID_A; Accept application/json; sem corpo; não exigir Content-Type na resposta 204.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT098 @RF005 @API005 @prioridade_critica @transicao @api
  Cenário: Excluir registro e retornar resposta sem corpo
    Dado que A e B existem
    Quando o cliente envia DELETE /api/alunos/ID_A sem corpo
    Então deve receber 204 com zero bytes de corpo
    E GET de ID_A deve retornar 404 enquanto GET de ID_B mantém B
    E a listagem deve conter somente B
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct099"></a>

#### CT099 — Retornar ausência ao repetir exclusão de registro removido

| Campo | Informação |
|---|---|
| Identificador | CT099 |
| Funcionalidade | API REST |
| Objetivo | Retornar ausência ao repetir exclusão de registro removido. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF005, API005, API008 |
| Origem da regra | RF005: Documentado; API005: Documentado / 404 inferido; API008: Documentado |
| Evidência no repositório | README.md: §§9.13,12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: excluir<br>src/main/resources/static/js/alunos.js: abrirExclusao<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: excluir<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: buscarEntidade<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** DELETE /api/alunos/ID_A; sem corpo; ERRO Not Found, não 204.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT099 @RF005 @API005 @API008 @prioridade_critica @negativo @api
  Cenário: Retornar ausência ao repetir exclusão de registro removido
    Dado que A foi criado e removido pela API e seu ID_A foi registrado
    Quando o cliente envia novamente DELETE /api/alunos/ID_A
    Então deve receber 404 com a mensagem de ID_A não encontrado
    E errosValidacao deve ser vazio e a lista não deve mudar
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct100"></a>

#### CT100 — Preservar o identificador quando o corpo de atualização tenta trocá-lo

| Campo | Informação |
|---|---|
| Identificador | CT100 |
| Funcionalidade | API REST |
| Objetivo | Preservar o identificador quando o corpo de atualização tenta trocá-lo. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Positivo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RN006, API004 |
| Origem da regra | RN006: Documentado; API004: Documentado |
| Evidência no repositório | README.md: §9.10<br>src/main/java/br/com/academia/alunos/entity/Aluno.java: id<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: atualizar |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** PUT /api/alunos/ID_A; corpo A + propriedade numérica id: ID_B. O código da resposta não é critério deste caso, pois o tratamento de propriedades desconhecidas não está definido (LAC005).

```gherkin
# language: pt
Funcionalidade: API REST

  @CT100 @RN006 @API004 @prioridade_critica @positivo @api
  Cenário: Preservar o identificador quando o corpo de atualização tenta trocá-lo
    Dado que A e B existem
    Quando o cliente envia PUT /api/alunos/ID_A com o corpo A acrescido da propriedade id igual a ID_B
    Então a listagem deve continuar contendo exatamente ID_A e ID_B
    E GET /api/alunos/ID_A deve apresentar os dados de A
    E GET /api/alunos/ID_B deve permanecer igual à fotografia anterior
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** A imutabilidade do ID é documentada (README §9.10). Exploração controlada: a propriedade extra foi ignorada e a resposta foi 200 com o ID da URL. Isso não torna a aceitação um requisito.

<a id="ct101"></a>

#### CT101 — Definir resposta para identificador incompatível com Long

| Campo | Informação |
|---|---|
| Identificador | CT101 |
| Funcionalidade | API REST |
| Objetivo | Definir resposta para identificador incompatível com Long. |
| Prioridade | Alta |
| Risco associado | R09 — Contrato HTTP ambíguo ou erro mal classificado (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | API011, API008 |
| Origem da regra | API011: Pendente de definição; API008: Documentado |
| Evidência no repositório | src/main/java/br/com/academia/alunos/controller/AlunoController.java: @PathVariable Long<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarErroInterno<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Pendente de definição — [LAC006](ANALISE_E_ESTRATEGIA_DE_TESTES_FUNCIONAIS.md#lac006) |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** GET /api/alunos/<id>; sem corpo; código e categoria pendentes em LAC006.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT101 @API011 @API008 @prioridade_alta @negativo @api @pendente_definicao
  Esquema do Cenário: Definir resposta para identificador incompatível com Long
    Dado que a API está disponível
    Quando o cliente envia GET /api/alunos/<id>
    Então o código e a categoria de erro devem seguir a decisão pendente em LAC006
    E nenhum dado deve ser alterado

    Exemplos:
      | exemplo    | particao                | id                  |
      | CT101-EX01 | texto                   | abc                 |
      | CT101-EX02 | decimal                 | 1.5                 |
      | CT101-EX03 | acima do máximo de Long | 9223372036854775808 |
```

**Pós-condições e restauração:** Não executar como aceite antes da definição. Depois, aplicar a restauração da seção 6.

**Observações, dependências e lacunas:** LAC006. Exploração controlada: a resposta foi 500 com a mensagem genérica de erro interno. O README reserva 500 a erro interno imprevisto e não define código para esta condição; o comportamento observado não foi adotado como resultado esperado.

<a id="ct102"></a>

#### CT102 — Definir resposta para método não suportado na coleção

| Campo | Informação |
|---|---|
| Identificador | CT102 |
| Funcionalidade | API REST |
| Objetivo | Definir resposta para método não suportado na coleção. |
| Prioridade | Alta |
| Risco associado | R09 — Contrato HTTP ambíguo ou erro mal classificado (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | API011, API008 |
| Origem da regra | API011: Pendente de definição; API008: Documentado |
| Evidência no repositório | src/main/java/br/com/academia/alunos/controller/AlunoController.java: @PathVariable Long<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarErroInterno<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Pendente de definição — [LAC006](ANALISE_E_ESTRATEGIA_DE_TESTES_FUNCIONAIS.md#lac006) |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** PATCH /api/alunos; corpo A; application/json; código, categoria e cabeçalhos pendentes em LAC006.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT102 @API011 @API008 @prioridade_alta @negativo @api @pendente_definicao
  Cenário: Definir resposta para método não suportado na coleção
    Dado que A existe
    Quando o cliente envia PATCH /api/alunos com o corpo A e application/json
    Então código, categoria e cabeçalhos devem seguir a decisão pendente em LAC006
    E A deve continuar inalterado
```

**Pós-condições e restauração:** Não executar como aceite antes da definição. Depois, aplicar a restauração da seção 6.

**Observações, dependências e lacunas:** LAC006. Exploração controlada: a resposta foi 500 com a mensagem genérica de erro interno. O README reserva 500 a erro interno imprevisto e não define código para esta condição; o comportamento observado não foi adotado como resultado esperado.

<a id="ct103"></a>

#### CT103 — Definir resposta para tipo de mídia incompatível

| Campo | Informação |
|---|---|
| Identificador | CT103 |
| Funcionalidade | API REST |
| Objetivo | Definir resposta para tipo de mídia incompatível. |
| Prioridade | Alta |
| Risco associado | R09 — Contrato HTTP ambíguo ou erro mal classificado (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | API011, API008 |
| Origem da regra | API011: Pendente de definição; API008: Documentado |
| Evidência no repositório | src/main/java/br/com/academia/alunos/controller/AlunoController.java: @PathVariable Long<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarErroInterno<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Pendente de definição — [LAC006](ANALISE_E_ESTRATEGIA_DE_TESTES_FUNCIONAIS.md#lac006) |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** POST /api/alunos; texto literal do JSON A; Content-Type text/plain; código e categoria pendentes em LAC006.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT103 @API011 @API008 @prioridade_alta @negativo @api @pendente_definicao
  Cenário: Definir resposta para tipo de mídia incompatível
    Dado que A ainda não existe
    Quando o cliente envia POST /api/alunos com o JSON A e Content-Type text/plain
    Então código e categoria devem seguir a decisão pendente em LAC006
    E nenhum aluno deve ser criado
```

**Pós-condições e restauração:** Não executar como aceite antes da definição. Depois, aplicar a restauração da seção 6.

**Observações, dependências e lacunas:** LAC006. Exploração controlada: a resposta foi 500 com a mensagem genérica de erro interno. O README reserva 500 a erro interno imprevisto e não define código para esta condição; o comportamento observado não foi adotado como resultado esperado.

<a id="ct104"></a>

#### CT104 — Definir resposta para rota inexistente

| Campo | Informação |
|---|---|
| Identificador | CT104 |
| Funcionalidade | API REST |
| Objetivo | Definir resposta para rota inexistente. |
| Prioridade | Alta |
| Risco associado | R09 — Contrato HTTP ambíguo ou erro mal classificado (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | API011, API008 |
| Origem da regra | API011: Pendente de definição; API008: Documentado |
| Evidência no repositório | src/main/java/br/com/academia/alunos/controller/AlunoController.java: @PathVariable Long<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarErroInterno<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Pendente de definição — [LAC006](ANALISE_E_ESTRATEGIA_DE_TESTES_FUNCIONAIS.md#lac006) |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** GET <caminho>; sem corpo; código pendente.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT104 @API011 @API008 @prioridade_alta @negativo @api @pendente_definicao
  Esquema do Cenário: Definir resposta para rota inexistente
    Dado que a aplicação está disponível
    Quando o cliente envia GET <caminho>
    Então código e categoria devem seguir a decisão pendente em LAC006

    Exemplos:
      | exemplo    | particao    | caminho         |
      | CT104-EX01 | sob a API   | /api/alunos/1/x |
      | CT104-EX02 | fora da API | /naoexiste      |
```

**Pós-condições e restauração:** Não executar como aceite antes da definição. Depois, aplicar a restauração da seção 6.

**Observações, dependências e lacunas:** LAC006. Exploração controlada: ambas retornaram 500; /favicon.ico também, requisição que navegadores fazem automaticamente.

<a id="ct105"></a>

#### CT105 — Definir tratamento de tipos escalares convertíveis

| Campo | Informação |
|---|---|
| Identificador | CT105 |
| Funcionalidade | API REST |
| Objetivo | Definir tratamento de tipos escalares convertíveis. |
| Prioridade | Alta |
| Risco associado | R09 — Contrato HTTP ambíguo ou erro mal classificado (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | API009, API008 |
| Origem da regra | API009: Inferido da implementação; API008: Documentado |
| Evidência no repositório | src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarJsonInvalido<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Pendente de definição — [LAC005](ANALISE_E_ESTRATEGIA_DE_TESTES_FUNCIONAIS.md#lac005) |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** POST /api/alunos; corpo A com o tipo indicado; código pendente.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT105 @API009 @API008 @prioridade_alta @negativo @api @pendente_definicao
  Esquema do Cenário: Definir tratamento de tipos escalares convertíveis
    Dado que A ainda não existe
    Quando o cliente envia POST /api/alunos com A e <campo> igual a <valor>
    Então a aceitação ou rejeição deve seguir a decisão pendente em LAC005

    Exemplos:
      | exemplo    | particao           | campo          | valor                        |
      | CT105-EX01 | texto → boolean    | ativo          | string "true"                |
      | CT105-EX02 | número → boolean   | ativo          | número 1                     |
      | CT105-EX03 | número → texto     | cpf            | número 12345678901           |
      | CT105-EX04 | data e hora → data | dataNascimento | string "1995-05-20T10:00:00" |
```

**Pós-condições e restauração:** Não executar como aceite antes da definição. Depois, aplicar a restauração da seção 6.

**Observações, dependências e lacunas:** LAC005. Exploração controlada: as quatro entradas foram aceitas com 201 e convertidas.

<a id="ct106"></a>

#### CT106 — Rejeitar nome que fica abaixo do mínimo após normalização

| Campo | Informação |
|---|---|
| Identificador | CT106 |
| Funcionalidade | API REST |
| Objetivo | Rejeitar nome que fica abaixo do mínimo após normalização. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL001, RN005, API006 |
| Origem da regra | VAL001: Documentado; RN005: Inferido da implementação; API006: Documentado |
| Evidência no repositório | README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar/atualizar/normalizarEmail/normalizarTelefone<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarValidacao |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Convenção OP; corpo A com nome de quatro caracteres incluindo espaços; expectativa normativa 400, sem persistência.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT106 @VAL001 @RN005 @API006 @prioridade_alta @limite @api
  Esquema do Cenário: Rejeitar nome que fica abaixo do mínimo após normalização
    Dado que a preparação "<operacao>" foi realizada para A
    Quando o cliente envia <operacao> com A e nomeCompleto " An "
    Então deve receber 400 no contrato ERRO
    E nenhum nome com menos de três caracteres deve ser persistido
    E a lista deve permanecer igual à fotografia anterior à tentativa

    Exemplos:
      | exemplo    | particao                 | operacao |
      | CT106-EX01 | cadastro: 2 após trim    | POST     |
      | CT106-EX02 | atualização: 2 após trim | PUT      |
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** LAC001. A regra documentada (mínimo de três caracteres) é o resultado esperado. Exploração controlada: POST retornou 201 e PUT retornou 200, ambos gravando "An". A mensagem específica para a rejeição pós-normalização não está definida e não é critério deste caso.

<a id="ct107"></a>

#### CT107 — Definir contrato para entrada de nome sem sobrenome

| Campo | Informação |
|---|---|
| Identificador | CT107 |
| Funcionalidade | API REST |
| Objetivo | Definir contrato para entrada de nome sem sobrenome. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | VAL001 |
| Origem da regra | VAL001: Documentado |
| Evidência no repositório | README.md: §9.1<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: nomeCompleto<br>src/main/resources/static/js/alunos.js: validarFormulario |
| Situação da especificação | Pendente de definição — [LAC007](ANALISE_E_ESTRATEGIA_DE_TESTES_FUNCIONAIS.md#lac007) |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** POST /api/alunos; A com nome Ana; README exige tamanho e obrigatoriedade, ajuda do HTML pede nome e sobrenome.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT107 @VAL001 @prioridade_alta @negativo @api @pendente_definicao
  Cenário: Definir contrato para entrada de nome sem sobrenome
    Dado que A ainda não existe
    Quando o cliente envia POST /api/alunos com A e nomeCompleto "Ana"
    Então a necessidade de sobrenome deve seguir a decisão pendente em LAC007
```

**Pós-condições e restauração:** Não executar como aceite antes da definição. Depois, aplicar a restauração da seção 6.

**Observações, dependências e lacunas:** LAC007. Os casos de limite seguem a regra objetiva de três caracteres do README; este cenário trata somente da obrigação adicional sugerida pela ajuda do formulário.

<a id="ct108"></a>

#### CT108 — Especificar retorno de falha interna sem expor detalhes técnicos

| Campo | Informação |
|---|---|
| Identificador | CT108 |
| Funcionalidade | API REST |
| Objetivo | Especificar retorno de falha interna sem expor detalhes técnicos. |
| Prioridade | Alta |
| Risco associado | R09 — Contrato HTTP ambíguo ou erro mal classificado (9) |
| Classificação principal | Exceção |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Error guessing fundamentado |
| Requisitos cobertos | API010, API008 |
| Origem da regra | API010: Documentado; API008: Documentado |
| Evidência no repositório | README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarErroInterno<br>src/main/java/br/com/academia/alunos/exception/ErroResposta.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: criarErro |
| Situação da especificação | Pendente de definição — [LAC008](ANALISE_E_ESTRATEGIA_DE_TESTES_FUNCIONAIS.md#lac008) |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Método, endpoint e massa de disparo dependem de LAC008; não executar injeção de falhas ou modificar produção.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT108 @API010 @API008 @prioridade_alta @excecao @api @pendente_definicao
  Cenário: Especificar retorno de falha interna sem expor detalhes técnicos
    Dado que existe uma forma aprovada e reproduzível de provocar exceção interna não tratada durante uma requisição
    Quando a exceção é provocada no ambiente isolado
    Então deve retornar 500 com erro "Internal Server Error"
    E mensagem deve ser "Ocorreu um problema interno. Tente novamente em alguns instantes."
    E o corpo deve seguir ERRO sem stack trace
```

**Pós-condições e restauração:** Não executar como aceite antes da definição. Depois, aplicar a restauração da seção 6.

**Observações, dependências e lacunas:** LAC008. O contrato do corpo é conhecido; falta um procedimento de disparo aprovado. Exploração controlada: GET /api/alunos/abc produziu exatamente esse corpo, mas usar essa entrada como disparo depende de LAC006, pois a decisão pode retirá-la do 500.

<a id="ct109"></a>

#### CT109 — Preservar unicidade quando gravações concorrentes disputam os mesmos dados

| Campo | Informação |
|---|---|
| Identificador | CT109 |
| Funcionalidade | API REST |
| Objetivo | Preservar unicidade quando gravações concorrentes disputam os mesmos dados. |
| Prioridade | Crítica |
| Risco associado | R02 — Duplicidade de identificadores (12) |
| Classificação principal | Exceção |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Error guessing fundamentado |
| Requisitos cobertos | RN001, RN002, API007 |
| Origem da regra | RN001: Documentado; RN002: Documentado; API007: Documentado |
| Evidência no repositório | README.md: §9.2,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: verificarDuplicidadeNoCadastro/verificarDuplicidadeNaAtualizacao<br>README.md: §9.4,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarEmail<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: existsByEmailIgnoreCaseAndIdNot<br>README.md: §10<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarRecursoDuplicado/tratarIntegridade |
| Situação da especificação | Pendente de definição — [LAC009](ANALISE_E_ESTRATEGIA_DE_TESTES_FUNCIONAIS.md#lac009) |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** Duas requisições POST application/json com A. Preparação para concorrência reprodutível ainda não definida.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT109 @RN001 @RN002 @API007 @prioridade_critica @excecao @api @pendente_definicao
  Cenário: Preservar unicidade quando gravações concorrentes disputam os mesmos dados
    Dado que A não existe e dois clientes preparam POST /api/alunos com A
    Quando as requisições disputam a mesma combinação CPF e e-mail
    Então somente um registro deve ser criado e uma requisição deve retornar 201
    E a outra deve retornar 409 com mensagem específica do serviço ou "Já existe um aluno com o CPF ou e-mail informado."
    E a listagem deve conter exatamente um registro
```

**Pós-condições e restauração:** Não executar como aceite antes da definição. Depois, aplicar a restauração da seção 6.

**Observações, dependências e lacunas:** LAC009. Não é teste de carga. Exploração controlada com cliente HTTP sincronizado: em 60 disputas entre duas requisições, o resultado foi sempre um 201 e um 409, com uma única linha ao final. As duas origens de mensagem ocorreram. O procedimento manual equivalente ainda não está definido.

<a id="ct110"></a>

#### CT110 — Definir recuperação quando a resposta de cadastro se perde após persistência

| Campo | Informação |
|---|---|
| Identificador | CT110 |
| Funcionalidade | API REST |
| Objetivo | Definir recuperação quando a resposta de cadastro se perde após persistência. |
| Prioridade | Alta |
| Risco associado | R06 — Falha sem recuperação ou reenvio involuntário (9) |
| Classificação principal | Exceção |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Error guessing fundamentado |
| Requisitos cobertos | RF001, API001, API007 |
| Origem da regra | RF001: Documentado; API001: Documentado; API007: Documentado |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §10<br>src/main/java/br/com/academia/alunos/dto/AlunoResponse.java<br>src/main/java/br/com/academia/alunos/exception/GlobalExceptionHandler.java: tratarRecursoDuplicado/tratarIntegridade |
| Situação da especificação | Pendente de definição — [LAC011](ANALISE_E_ESTRATEGIA_DE_TESTES_FUNCIONAIS.md#lac011) |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** POST /api/alunos; corpo A; o serviço atual tende a 409 no reenvio. Preparação para perda da resposta após commit não definida.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT110 @RF001 @API001 @API007 @prioridade_alta @excecao @api @pendente_definicao
  Cenário: Definir recuperação quando a resposta de cadastro se perde após persistência
    Dado que o cadastro de A é persistido mas sua resposta não chega ao cliente
    Quando o cliente repete POST /api/alunos com A
    Então a recuperação e identificação do registro criado devem seguir a decisão de LAC011
```

**Pós-condições e restauração:** Não executar como aceite antes da definição. Depois, aplicar a restauração da seção 6.

**Observações, dependências e lacunas:** LAC011. Não há contrato de idempotência; não atribuir automaticamente novo 201 ou perda de dados.

<a id="ct111"></a>

#### CT111 — Definir referência temporal quando navegador e servidor estão em dias distintos

| Campo | Informação |
|---|---|
| Identificador | CT111 |
| Funcionalidade | API REST |
| Objetivo | Definir referência temporal quando navegador e servidor estão em dias distintos. |
| Prioridade | Alta |
| Risco associado | R01 — Cadastro inválido ou bloqueio de matrícula (9) |
| Classificação principal | Limite |
| Características secundárias | Negativo |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Análise de valor-limite |
| Requisitos cobertos | VAL005, VAL007, RN003 |
| Origem da regra | VAL005: Documentado; VAL007: Documentado; RN003: Documentado / extensão inferida para null |
| Evidência no repositório | README.md: §9.6<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataNascimento<br>src/main/resources/static/js/alunos.js: validarFormulario<br>README.md: §9.8<br>src/main/java/br/com/academia/alunos/dto/AlunoRequest.java: dataMatricula<br>README.md: §9.8–9<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar |
| Situação da especificação | Pendente de definição — [LAC012](ANALISE_E_ESTRATEGIA_DE_TESTES_FUNCIONAIS.md#lac012) |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** POST /api/alunos; A com dataMatricula igual à data local do navegador; código depende da decisão.

```gherkin
# language: pt
Funcionalidade: API REST

  @CT111 @VAL005 @VAL007 @RN003 @prioridade_alta @limite @api @pendente_definicao
  Cenário: Definir referência temporal quando navegador e servidor estão em dias distintos
    Dado que a data local do navegador é diferente da data local do servidor
    Quando uma matrícula é enviada na data do navegador
    Então aceitação e aplicação do padrão devem seguir a referência temporal a definir em LAC012
```

**Pós-condições e restauração:** Não executar como aceite antes da definição. Depois, aplicar a restauração da seção 6.

**Observações, dependências e lacunas:** LAC012. Não alterar o relógio de sistema compartilhado. O servidor usa LocalDate.now e a interface usa Date local.


### 5.10 Persistência

<a id="ct112"></a>

#### CT112 — Recuperar dados após recarregar página sem reiniciar o processo

| Campo | Informação |
|---|---|
| Identificador | CT112 |
| Funcionalidade | Persistência |
| Objetivo | Recuperar dados após recarregar página sem reiniciar o processo. |
| Prioridade | Alta |
| Risco associado | R10 — Perda ou retenção inesperada de dados (8) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | Interface gráfica |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RN007, RF001, RF002 |
| Origem da regra | RN007: Documentado; RF001: Documentado; RF002: Documentado |
| Evidência no repositório | README.md: §13<br>src/main/resources/application.properties: datasource.url/ddl-auto<br>README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §9.12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: listarTodos<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: findAllByOrderByIdAsc |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

```gherkin
# language: pt
Funcionalidade: Persistência

  @CT112 @RN007 @RF001 @RF002 @prioridade_alta @transicao @interface @persistencia
  Cenário: Recuperar dados após recarregar página sem reiniciar o processo
    Dado que A foi cadastrado e o processo permanece em execução
    Quando o usuário fecha a aba e abre novamente a página
    Então A deve continuar disponível com os mesmos oito campos e ID
    E a consulta pela API deve concordar com a interface
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct113"></a>

#### CT113 — Verificar armazenamento físico após cadastro e atualização

| Campo | Informação |
|---|---|
| Identificador | CT113 |
| Funcionalidade | Persistência |
| Objetivo | Verificar armazenamento físico após cadastro e atualização. |
| Prioridade | Alta |
| Risco associado | R10 — Perda ou retenção inesperada de dados (8) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST + console H2 |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF001, RF004, RN007 |
| Origem da regra | RF001: Documentado; RF004: Documentado; RN007: Documentado |
| Evidência no repositório | README.md: §§5,9,10<br>src/main/java/br/com/academia/alunos/controller/AlunoController.java: cadastrar<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: cadastrar<br>README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §13<br>src/main/resources/application.properties: datasource.url/ddl-auto |
| Situação da especificação | Especificado |

**Preparação e dados:** Abrir /h2-console usando a configuração local do README §13. Executar somente SELECT * FROM ALUNOS WHERE ID = ID_A, substituindo o ID; não executar comandos de alteração.

**Requisição e contrato:** Preparação POST /api/alunos com A; alteração PUT /api/alunos/ID_A com A e telefone 1133224455; contratos SUCESSO 201/200.

```gherkin
# language: pt
Funcionalidade: Persistência

  @CT113 @RF001 @RF004 @RN007 @prioridade_alta @transicao @api @persistencia
  Cenário: Verificar armazenamento físico após cadastro e atualização
    Dado que A foi cadastrado pela API com ID_A
    Quando o responsável consulta a tabela ALUNOS no console H2
    Então deve existir exatamente uma linha para ID_A com os dados de A
    Quando A é atualizado pela API para telefone 1133224455
    Então nova consulta SQL deve mostrar o telefone alterado sem mudança de ID
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** A observação SQL é manual e explícita; não pressupõe acesso invisível a detalhes internos.

<a id="ct114"></a>

#### CT114 — Confirmar remoção física após exclusão pela API

| Campo | Informação |
|---|---|
| Identificador | CT114 |
| Funcionalidade | Persistência |
| Objetivo | Confirmar remoção física após exclusão pela API. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST + console H2 |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RF005, RN007 |
| Origem da regra | RF005: Documentado; RN007: Documentado |
| Evidência no repositório | README.md: §§9.13,12<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: excluir<br>src/main/resources/static/js/alunos.js: abrirExclusao<br>README.md: §13<br>src/main/resources/application.properties: datasource.url/ddl-auto |
| Situação da especificação | Especificado |

**Preparação e dados:** Utilizar somente SELECT no console. Substituir ID_A pelo número capturado.

**Requisição e contrato:** DELETE /api/alunos/ID_A sem corpo; GET subsequente 404 ERRO.

```gherkin
# language: pt
Funcionalidade: Persistência

  @CT114 @RF005 @RN007 @prioridade_critica @transicao @api @persistencia
  Cenário: Confirmar remoção física após exclusão pela API
    Dado que A existe e foi localizado por ID_A no console H2
    Quando o cliente envia DELETE /api/alunos/ID_A
    Então deve receber 204 sem corpo
    E SELECT * FROM ALUNOS WHERE ID = ID_A deve retornar zero linhas
    E a consulta HTTP de ID_A deve retornar 404
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct115"></a>

#### CT115 — Reinicializar dados ao encerrar e iniciar a instância isolada

| Campo | Informação |
|---|---|
| Identificador | CT115 |
| Funcionalidade | Persistência |
| Objetivo | Reinicializar dados ao encerrar e iniciar a instância isolada. |
| Prioridade | Alta |
| Risco associado | R10 — Perda ou retenção inesperada de dados (8) |
| Classificação principal | Transição |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Transição de estados |
| Requisitos cobertos | RN007 |
| Origem da regra | RN007: Documentado |
| Evidência no repositório | README.md: §13<br>src/main/resources/application.properties: datasource.url/ddl-auto |
| Situação da especificação | Especificado |

**Preparação e dados:** Executar por último e somente em instância exclusiva com dados descartáveis. Encerrar apenas o processo preparado pelo executor.

**Requisição e contrato:** GET /api/alunos; sem corpo; application/json; lista vazia. Não exigir sequência específica de ID após reinício.

```gherkin
# language: pt
Funcionalidade: Persistência

  @CT115 @RN007 @prioridade_alta @transicao @api @persistencia
  Cenário: Reinicializar dados ao encerrar e iniciar a instância isolada
    Dado que a instância local exclusiva contém A e a configuração original mem/create-drop está ativa
    Quando o responsável encerra esse processo e inicia uma nova instância com a mesma configuração
    Então GET /api/alunos deve retornar 200 com []
    E nenhum dos dados anteriores deve aparecer na nova instância
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

<a id="ct116"></a>

#### CT116 — Manter os registros intactos após falha de atualização

| Campo | Informação |
|---|---|
| Identificador | CT116 |
| Funcionalidade | Persistência |
| Objetivo | Manter os registros intactos após falha de atualização. |
| Prioridade | Crítica |
| Risco associado | R03 — Corrupção de atualização ou exclusão indevida (12) |
| Classificação principal | Negativo |
| Características secundárias | Nenhuma |
| Nível de teste | Sistema |
| Canal | API REST |
| Técnica de projeto | Particionamento de equivalência |
| Requisitos cobertos | RF004, RN001, RN002 |
| Origem da regra | RF004: Documentado; RN001: Documentado; RN002: Documentado |
| Evidência no repositório | README.md: §§9.10–11,10<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: atualizar<br>README.md: §9.2,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: verificarDuplicidadeNoCadastro/verificarDuplicidadeNaAtualizacao<br>README.md: §9.4,11<br>src/main/java/br/com/academia/alunos/service/AlunoService.java: normalizarEmail<br>src/main/java/br/com/academia/alunos/repository/AlunoRepository.java: existsByEmailIgnoreCaseAndIdNot |
| Situação da especificação | Especificado |

**Preparação e dados:** Aplicar a preparação manual da seção 3 e os dados explicitados nos passos e exemplos. Confirmar o estado inicial por consulta; recriar a massa entre instâncias.

**Requisição e contrato:** PUT /api/alunos/ID_A; A com nome Nome não persistido e cpf 98765432100; ERRO Conflict. GET de ambos para comparar todos os campos.

```gherkin
# language: pt
Funcionalidade: Persistência

  @CT116 @RF004 @RN001 @RN002 @prioridade_critica @negativo @api @persistencia
  Cenário: Manter os registros intactos após falha de atualização
    Dado que A e B existem e suas representações foram registradas
    Quando o cliente envia PUT /api/alunos/ID_A com A, nome "Nome não persistido" e CPF de B
    Então deve receber 409 com a mensagem específica de CPF duplicado
    E consultas posteriores devem retornar A e B idênticos às fotografias anteriores
```

**Pós-condições e restauração:** Aplicar a restauração da seção 6 antes da próxima instância.

**Observações, dependências e lacunas:** Sem dependência de outro caso.

## 6. Restauração e limites de execução

Após cada instância, remover via DELETE somente os IDs sintéticos criados durante sua preparação e ação. Se um ID já foi excluído, confirmar sua ausência em vez de interpretar o 404 de limpeza como falha do cenário. Não usar exclusão em massa nem reiniciar instância compartilhada.

Recriar A/B/C conforme a próxima preparação; mudanças em CPF/e-mail não alteram o ID capturado para limpeza. Desativar bloqueio de requisição, limitação de rede, breakpoint e modo responsivo utilizados. Fechar notificações persistentes antes da próxima instância quando interferirem na observação.

CT115 exige instância exclusiva e perde toda a massa por definição; executar por último. Os casos com console H2 exigem acesso local e apenas SELECT. A consulta HTTP comprova persistência observável, enquanto CT113/CT114 verificam explicitamente a tabela.

Cenários pendentes não possuem condições suficientes para aceite completo. Os demais estão especificados, mas continuam sujeitos aos pré-requisitos de ambiente e à revisão humana. Nenhum cenário foi executado formalmente nesta elaboração e nenhum resultado foi atribuído.

Os casos CT113 e CT114 consultam o banco de modo deliberado: a documentação do Cucumber recomenda resultados observáveis fora do sistema, e aqui a remoção física (README §9.13) é a própria regra verificada. Ambos mantêm também a verificação por HTTP.
