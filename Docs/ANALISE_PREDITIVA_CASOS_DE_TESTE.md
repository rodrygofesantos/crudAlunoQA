# Análise preditiva baseada em risco dos casos de teste funcionais

## 1. Escopo, método e limitações

Esta análise considera os 20 casos identificados diretamente pelas seções
`CT001` a `CT020` de `Docs/CASOS_DE_TESTE_FUNCIONAIS.md` e as evidências
disponíveis no código-fonte do frontend, controller, DTO, serviço, repositório,
entidade, tratamento global de exceções e configuração do H2.

> Esta é uma estimativa heurística de risco, e não uma probabilidade estatística calculada a partir de dados históricos.

Não foram fornecidos resultados anteriores de execução, histórico de defeitos,
cobertura, complexidade ciclomática, alterações funcionais recentes ou
incidentes. Por isso:

- `Probabilidade` representa a chance qualitativa de o caso revelar um defeito
  do produto, de 1 a 5;
- `Impacto` representa a consequência funcional, técnica e operacional do
  defeito hipotético, de 1 a 5;
- `Risco` é calculado por `Probabilidade × Impacto`;
- a prioridade declarada no documento foi preservada como dado de origem e não
  foi usada para forçar a pontuação;
- a confiança indica a força das evidências estáticas, não a certeza de que
  existe um defeito;
- empates foram ordenados por impacto, probabilidade, centralidade no fluxo e
  dependência para os testes seguintes.

As inferências sobre comportamento em execução foram identificadas como tais.
O código não foi executado e não há testes automatizados no repositório.

## 2. Resumo executivo

### 2.1 Cinco casos com maior risco

| Posição | Caso | Risco | Motivo dominante |
|---:|---|---:|---|
| 1 | CT004 — cadastro válido completo | 25 — Crítico | Concentra 12 requisitos e atravessa frontend, API, validação, serviço e banco |
| 2 | CT011 — atualização válida | 25 — Crítico | Modifica quase todo o estado persistido e combina unicidade, normalização e preservação de identidade |
| 3 | CT009 — CPF duplicado no cadastro | 20 — Crítico | Protege uma chave de negócio central em serviço e banco |
| 4 | CT010 — e-mail duplicado sem diferenciar caixa | 20 — Crítico | Combina normalização, consulta case-insensitive e restrição única |
| 5 | CT014 — exclusão confirmada pela interface | 20 — Crítico | Executa remoção física irreversível por todas as camadas |

O corte dos cinco primeiros não significa que CT013, CT012, CT001 e CT019
sejam dispensáveis. Eles permanecem de risco alto e protegem, respectivamente,
unicidade na atualização, autoexclusão das verificações de duplicidade,
disponibilidade da listagem e contrato da exclusão.

### 2.2 Funcionalidades mais propensas a revelar defeitos

1. **Atualização:** maior concentração de probabilidade. O fluxo carrega uma
   entidade, normaliza CPF e e-mail, ignora o próprio ID nas consultas de
   unicidade, altera seis campos e preserva condicionalmente outros dois.
2. **Cadastro e validações:** alta combinação de regras, limites, padrões,
   normalização, duplicidade e persistência. A validação é duplicada entre
   JavaScript e Bean Validation.
3. **Consulta/listagem no frontend:** a listagem é assíncrona e alimenta
   indicadores, pesquisa, filtros, edição e exclusão a partir de um estado local.

### 2.3 Defeitos de maior impacto

- exclusão do aluno errado ou exclusão sem uma confirmação efetiva;
- aceitação de CPF ou e-mail já usados por outro aluno;
- atualização parcial, no registro errado ou com perda de valores anteriores;
- falha de transação que deixe a interface e o banco em estados divergentes;
- perda inesperada de registros ou interpretação incorreta da persistência
  temporária do H2;
- criação de aluno com padrões incorretos de matrícula ou situação.

### 2.4 Principais riscos para a integridade dos dados

- **Unicidade:** as verificações preventivas do serviço e as restrições do banco
  precisam produzir o mesmo resultado, inclusive sob concorrência.
- **Normalização:** o frontend remove máscaras, enquanto o serviço apenas aplica
  `trim` ao CPF e converte o e-mail para minúsculas. Consumidores diretos da API
  podem se comportar de forma diferente da interface.
- **Atualização:** valores nulos de `dataMatricula` e `ativo` preservam o estado
  anterior, enquanto os demais campos são sobrescritos.
- **Exclusão física:** não existe recuperação no domínio depois do `DELETE`.
- **Persistência efêmera:** `jdbc:h2:mem` com `create-drop` elimina os dados ao
  final do processo; qualquer expectativa de retenção entre reinícios é
  incompatível com a configuração.

### 2.5 Direcionamento

A equipe deve concentrar primeiro os testes no encadeamento
**cadastro → unicidade → atualização → exclusão**, usando pelo menos dois
registros. Em paralelo à execução manual pela interface, deve validar
diretamente a API nos pontos que o frontend normaliza ou preenche
automaticamente. Em seguida, deve testar tratamento de erros, limites e
persistência. Pesquisa, estados vazios e consultas simples podem entrar na
regressão posterior.

## 3. Auditoria da qualidade e consistência dos dados

### 3.1 Recontagem independente

| Dimensão | Resultado contado nas seções CT001–CT020 | Valor do resumo | Situação |
|---|---:|---:|---|
| Total | 20 | 20 | Consistente |
| Prioridade crítica | 8 | 8 | Consistente |
| Prioridade alta | 10 | 8 | Inconsistente |
| Prioridade média | 2 | 4 | Inconsistente |
| Tipo positivo | 11 | 10 | Inconsistente |
| Tipo negativo | 9 | 9 | Consistente |

As prioridades corretas, conforme os próprios casos, são:

- críticas: CT001, CT004, CT007, CT009, CT010, CT011, CT013 e CT014;
- altas: CT002, CT003, CT005, CT006, CT008, CT012, CT015, CT017, CT018 e CT019;
- médias: CT016 e CT020.

Os 11 casos positivos são CT001, CT002, CT004, CT005, CT011, CT012, CT014,
CT016, CT017, CT019 e CT020.

### 3.2 Inconsistências encontradas

| Inconsistência encontrada | Evidência | Consequência para a análise | Correção recomendada |
|---|---|---|---|
| Distribuição de prioridades incorreta | Seções individuais totalizam 8 críticas, 10 altas e 2 médias; o resumo informa 8/8/4 | Distorce capacidade, ordem e cobertura por severidade declarada | Alterar o resumo para 8 críticas, 10 altas e 2 médias |
| Quantidade de positivos incorreta | Há 11 casos com `Tipo = Positivo`; o resumo informa 10 | Compromete análises por partição positiva/negativa | Alterar o resumo para 11 positivos |
| Resumo por funcionalidade não usa as categorias declaradas nos casos e soma 23 ocorrências | Pelos campos `Funcionalidade`: Inicialização 2, Listagem 1, Cadastro 7, Atualização 3, Exclusão 3, Pesquisa e filtros 1, Consulta por ID 2 e Persistência 1 | Impede reconciliação e mistura funcionalidade, canal e nível | Recalcular pelas oito categorias declaradas ou marcar explicitamente categorias sobrepostas |
| `API005` associado ao caso errado na matriz | A matriz aponta CT015; CT015 apenas cancela o diálogo e cobre UI001. CT014 e CT019 declaram API005 e executam `DELETE` | Produz falsa cobertura do contrato 204 e omite os testes que o verificam | Substituir CT015 por CT014 e CT019 |
| Requisitos UI usados, mas ausentes da matriz | CT002/CT003 declaram UI004; CT016 declara UI005; não há linhas UI004 ou UI005 | Requisitos ficam sem definição e sem rastreabilidade verificável | Adicionar UI004 e UI005 com descrição, casos e prioridade |
| Matriz diverge dos requisitos declarados nos casos | Exemplos: RF001 omite CT020; RF002 inclui CT003 sem o RF002 estar no caso e omite CT020; RF004 associa CT012/CT013 sem esses IDs constarem nos casos e omite CT020; RF005 associa CT015 e omite CT019/CT020 | Contagens de cobertura e impacto por requisito tornam-se não confiáveis | Gerar a matriz a partir de uma fonte única e revisar cada relação bidirecional |
| Diversas regras são sub-rastreadas | RN001 omite CT004/CT013; RN002 omite CT004; RN003 omite CT004/CT005/CT011; RN005 omite CT004/CT011; RN007–RN009 omitem CT011; RN010 omite CT004 | Pode ocultar regressões em fluxos que exercitam a mesma regra | Incluir todos os casos que afirmam cobrir cada regra ou remover a afirmação do caso |
| `API001` e `API004` não são coerentes entre casos e matriz | API001 inclui CT005 apenas na matriz; API004 inclui CT011/CT012 apenas na matriz, embora não apareça em seus campos de requisitos | A cobertura do contrato de criação/atualização depende de interpretação externa ao caso | Harmonizar os IDs nos casos e na matriz |
| CT018 espera nomes de campos inexistentes no contrato real | O caso espera `status` e `path`; `ErroResposta` serializa `codigoHttp` e `caminho`, além de `dataHora` e `errosValidacao` | Pode gerar falso defeito mesmo com o produto conforme o código e o README | Corrigir o resultado esperado para `dataHora`, `codigoHttp`, `erro`, `mensagem`, `caminho` e `errosValidacao` |
| CT005 não consegue omitir os padrões pela interface | `abrirCadastro()` preenche a data atual e marca `ativo`; `obterDadosFormulario()` sempre envia `dataMatricula` e `ativo` | O caso não revela defeitos nos ramos de padrão do serviço, embora declare esse objetivo | Dividir em teste de interface para opcionais e teste direto de API omitindo as duas propriedades |
| DT001 é chamado de “CPF válido”, mas a validade é ambígua | `12345678901` tem 11 dígitos, porém não satisfaz a validação semântica usual; o código valida apenas `\d{11}` | Pode confundir validade sintática com validade de negócio | Renomear para “CPF sintaticamente válido” e decidir se dígitos verificadores fazem parte do requisito |
| Dados de fronteira catalogados não viraram casos | DT007 (nome com 101), DT010 (CPF com letras), DT014 (nascimento hoje) e DT015 (matrícula futura) não são usados em casos específicos | O catálogo dá aparência de cobertura que não existe | Criar casos dedicados e ligar cada dado à rastreabilidade |
| Metadado do commit não corresponde ao HEAD atual | Documento declara `25dfa5d`; a revisão atual é `87a4272` | A reprodução do conjunto completo pode usar uma revisão diferente | Registrar separadamente commit do código e commit do documento; o diff mostra que os fontes funcionais não mudaram desde `25dfa5d` |
| Sobreposição parcial não está identificada | CT014 valida exclusão pela UI e CT019 valida o contrato 204 da mesma operação | Pode ser interpretada como duplicidade ou causar execução redundante | Manter ambos, mas explicitar níveis e reutilizar a mesma preparação de dados |

Não foi encontrada duplicidade exata. CT014 e CT019 possuem interseção legítima,
mas objetivos e canais distintos.

### 3.3 Matriz de rastreabilidade corrigida em nível mínimo

Esta correção reflete as associações declaradas nas próprias seções dos casos;
não cria requisitos novos.

| Requisito | Casos declarados nas seções individuais |
|---|---|
| RF001 | CT004, CT005, CT020 |
| RF002 | CT001, CT002, CT020 |
| RF003 | CT017, CT018 |
| RF004 | CT011, CT020 |
| RF005 | CT014, CT019, CT020 |
| RF006 | CT016 |
| RN001 | CT004, CT005, CT009, CT012, CT013 |
| RN002 | CT004, CT005, CT010, CT012 |
| RN003 | CT004, CT006, CT011 |
| RN004 | CT004, CT007 |
| RN005 | CT004, CT008, CT011 |
| RN006 | CT004, CT005 |
| RN007 | CT004, CT005, CT011 |
| RN008 | CT004, CT005, CT011 |
| RN009 | CT004, CT005, CT011 |
| RN010 | CT004, CT005 |
| UI001 | CT014, CT015 |
| UI004 | CT002, CT003 |
| UI005 | CT016 |
| API001 | CT004 |
| API002 | CT001, CT003 |
| API003 | CT017, CT018 |
| API004 | Nenhum caso declara explicitamente |
| API005 | CT014, CT019 |
| API006 | CT006, CT007, CT008 |
| API007 | CT009, CT010, CT013 |
| API008 | CT018 |

Inferência: funcionalmente, CT011 e CT012 exercitam `PUT` e poderiam cobrir
API004, mas essa associação deve ser corrigida no catálogo antes de ser tratada
como dado. Da mesma forma, CT003 se relaciona ao endpoint de listagem, mas não
declara RF002. A tabela acima preserva a fonte em vez de completar relações por
suposição.

## 4. Ranking preditivo de todos os casos

| Posição | Caso | Funcionalidade | Prioridade original | Probabilidade | Impacto | Risco | Classificação | Confiança |
|---:|---|---|---|---:|---:|---:|---|---|
| 1 | CT004 | Cadastro | Crítica | 5 | 5 | 25 | Crítico | Alta |
| 2 | CT011 | Atualização | Crítica | 5 | 5 | 25 | Crítico | Alta |
| 3 | CT009 | Cadastro/CPF único | Crítica | 4 | 5 | 20 | Crítico | Alta |
| 4 | CT010 | Cadastro/e-mail único | Crítica | 4 | 5 | 20 | Crítico | Alta |
| 5 | CT014 | Exclusão pela interface | Crítica | 4 | 5 | 20 | Crítico | Alta |
| 6 | CT013 | Atualização/CPF único | Crítica | 4 | 4 | 16 | Alto | Alta |
| 7 | CT012 | Atualização sem falso conflito | Alta | 4 | 4 | 16 | Alto | Alta |
| 8 | CT001 | Inicialização/listagem | Crítica | 4 | 4 | 16 | Alto | Alta |
| 9 | CT019 | Exclusão pela API | Alta | 3 | 5 | 15 | Alto | Alta |
| 10 | CT007 | Validação de CPF | Crítica | 3 | 4 | 12 | Alto | Alta |
| 11 | CT005 | Cadastro com opcionais | Alta | 3 | 4 | 12 | Alto | Média |
| 12 | CT020 | Persistência H2 | Média | 3 | 4 | 12 | Alto | Média |
| 13 | CT016 | Pesquisa e filtros | Média | 4 | 3 | 12 | Alto | Alta |
| 14 | CT015 | Cancelamento de exclusão | Alta | 2 | 5 | 10 | Alto | Média |
| 15 | CT003 | Erro ao carregar | Alta | 3 | 3 | 9 | Moderado | Média |
| 16 | CT006 | Limite mínimo do nome | Alta | 3 | 3 | 9 | Moderado | Alta |
| 17 | CT008 | Formato de e-mail | Alta | 3 | 3 | 9 | Moderado | Alta |
| 18 | CT018 | Consulta de ID inexistente | Alta | 3 | 3 | 9 | Moderado | Alta |
| 19 | CT017 | Consulta de ID existente | Alta | 2 | 3 | 6 | Moderado | Alta |
| 20 | CT002 | Estado de lista vazia | Alta | 2 | 2 | 4 | Baixo | Alta |

### 4.1 Leitura do ranking

- A prioridade original superestima CT001 e CT007 em relação ao risco
  calculado e subestima CT012, CT019 e CT020.
- CT016, embora tenha prioridade média, recebe risco alto porque combina
  normalização, três campos pesquisáveis, filtro de situação, contador e três
  estados de renderização em um único caso.
- CT005 não recebeu probabilidade maior porque a interface sempre envia
  `dataMatricula` e `ativo`. Assim, o caso atual não alcança os ramos de omissão
  que pretende validar.
- CT018 tem confiança alta quanto à evidência de contrato, mas sua especificação
  precisa ser corrigida antes da execução para não registrar um falso positivo.

## 5. Justificativa dos casos críticos e altos

### CT004 — Cadastro completo — risco 25

- **Hipótese de defeito:** uma regra válida em uma camada pode ser transformada
  ou rejeitada em outra; o registro pode ser criado com campo incorreto ou a
  interface pode não refletir a persistência.
- **Evidência:** `salvarAluno()` remove máscaras e faz `POST`;
  `AlunoRequest` valida oito campos; `AlunoService.cadastrar()` normaliza,
  verifica duplicidade, aplica padrões e salva; `Aluno` impõe nulidade,
  comprimentos e unicidade.
- **Motivo da probabilidade:** é o caso mais amplo, cobre 12 requisitos e
  atravessa todas as camadas com várias transformações.
- **Consequência:** indisponibilidade do cadastro principal ou criação de dados
  inválidos/incompletos.
- **Camadas afetadas:** HTML/JavaScript, controller, DTO, serviço, repositório,
  JPA e H2.
- **Ação preventiva:** decompor o fluxo em asserções por contrato e confirmar o
  registro por `GET` após o `POST`, não apenas pela tabela recarregada.

### CT011 — Atualização válida — risco 25

- **Hipótese de defeito:** a edição pode sobrescrever campos errados, perder
  valores, alterar a situação incorretamente ou produzir falso conflito.
- **Evidência:** `abrirEdicao()` preenche oito controles; `salvarAluno()` usa
  `PUT`; `AlunoService.atualizar()` busca por ID, normaliza, verifica
  duplicidade, altera seis campos e atualiza condicionalmente matrícula e
  situação.
- **Motivo da probabilidade:** reúne leitura prévia, transformação no
  formulário, regras de unicidade e persistência de múltiplas alterações.
- **Consequência:** corrupção do cadastro existente e propagação para
  listagem, pesquisa, indicadores e exclusão.
- **Camadas afetadas:** frontend, API, validação, serviço, repositório e banco.
- **Ação preventiva:** comparar fotografia antes/depois, verificar ID imutável
  e confirmar individualmente campos alterados e preservados.

### CT009 — CPF duplicado no cadastro — risco 20

- **Hipótese de defeito:** dois alunos podem compartilhar CPF ou o conflito pode
  ser devolvido com código/mensagem inadequados.
- **Evidência:** `existsByCpf()` no repositório, verificação prévia no serviço,
  coluna `cpf` única e tratamento de `RecursoDuplicadoException`/restrição do
  banco como 409.
- **Motivo da probabilidade:** existem duas linhas de defesa com caminhos de
  exceção diferentes, sobretudo em gravações concorrentes.
- **Consequência:** quebra de identidade do aluno e ambiguidade em edição e
  exclusão.
- **Camadas afetadas:** serviço, repositório, banco, handler e frontend.
- **Ação preventiva:** validar conflito sequencial e complementar com cenário
  concorrente controlado, confirmando que somente um registro persiste.

### CT010 — E-mail duplicado case-insensitive — risco 20

- **Hipótese de defeito:** diferenças de caixa ou espaços podem contornar a
  unicidade ou gerar resposta inconsistente.
- **Evidência:** `normalizarEmail()` aplica `trim().toLowerCase()`;
  `existsByEmailIgnoreCase()` consulta sem diferenciar caixa; a coluna é única.
- **Motivo da probabilidade:** a regra depende da composição entre
  normalização da aplicação, consulta derivada do Spring Data e collation do
  banco.
- **Consequência:** contatos duplicados, associação ambígua e perda de confiança
  nos dados.
- **Camadas afetadas:** frontend/API, serviço, repositório, banco e handler.
- **Ação preventiva:** testar caixa, espaços externos e atualização, sempre
  consultando a quantidade final de registros.

### CT014 — Exclusão confirmada — risco 20

- **Hipótese de defeito:** o ID em estado local pode não corresponder ao nome
  mostrado, a exclusão pode ocorrer antes da confirmação ou a lista pode
  permanecer divergente do banco.
- **Evidência:** `abrirExclusao()` guarda o objeto em
  `estado.alunoParaExcluir`; `excluirAluno()` envia o ID guardado;
  `AlunoService.excluir()` carrega e remove fisicamente a entidade.
- **Motivo da probabilidade:** ação assíncrona e irreversível atravessa estado
  local, diálogo, API e persistência.
- **Consequência:** perda definitiva do registro errado ou falsa indicação de
  sucesso.
- **Camadas afetadas:** frontend, controller, serviço, repositório e banco.
- **Ação preventiva:** usar dois alunos visualmente distintos, conferir
  nome/ID no diálogo e validar por listagem e `GET` após a confirmação.

### CT013 — Atualização com CPF de outro aluno — risco 16

- **Hipótese de defeito:** a consulta que ignora o próprio ID pode ignorar o
  registro errado ou deixar passar o CPF de terceiro.
- **Evidência:** `existsByCpfAndIdNot(cpf, id)` e
  `verificarDuplicidadeNaAtualizacao()`.
- **Motivo da probabilidade:** predicado composto e dependência de dois
  registros são fontes comuns de inversão de condição.
- **Consequência:** se a defesa do banco funcionar, conflito genérico; se não,
  quebra de unicidade. O impacto foi limitado a 4 porque a coluna única fornece
  proteção complementar.
- **Camadas afetadas:** serviço, repositório, banco, handler e interface.
- **Ação preventiva:** verificar que nenhum campo do segundo aluno mudou depois
  do 409 e repetir o cenário para e-mail.

### CT012 — Atualização mantendo CPF e e-mail — risco 16

- **Hipótese de defeito:** o sistema pode comparar o registro consigo mesmo e
  bloquear qualquer atualização com falso 409.
- **Evidência:** os métodos `existsBy...AndIdNot` foram criados especificamente
  para excluir o ID atual.
- **Motivo da probabilidade:** é o complemento lógico de CT013; um erro no
  predicado pode fazer um dos dois cenários passar e o outro falhar.
- **Consequência:** indisponibilidade generalizada da edição, embora os dados
  existentes permaneçam íntegros.
- **Camadas afetadas:** serviço, repositório, handler e frontend.
- **Ação preventiva:** manter exatamente CPF/e-mail, alterar um único campo e
  validar que não houve normalização destrutiva.

### CT001 — Inicialização e listagem — risco 16

- **Hipótese de defeito:** a carga assíncrona pode não substituir o estado de
  carregamento, não atualizar os resumos ou falhar ao renderizar um item.
- **Evidência:** `DOMContentLoaded` chama `carregarAlunos()`, que atualiza
  estado, resumo, listagem e texto de atualização; a API mapeia `GET
  /api/alunos`.
- **Motivo da probabilidade:** a lista é a origem de pesquisa, edição,
  exclusão e três indicadores, ampliando a superfície do caso.
- **Consequência:** a aplicação aparenta indisponibilidade e bloqueia os fluxos
  dependentes mesmo com a API saudável.
- **Camadas afetadas:** frontend, controller, serviço, repositório e API.
- **Ação preventiva:** validar lista vazia, lista com múltiplos registros,
  ordenação por ID e consistência dos três indicadores.

### CT019 — DELETE 204 sem corpo — risco 15

- **Hipótese de defeito:** a API pode retornar corpo/código incorreto, responder
  antes da efetivação ou não remover o registro.
- **Evidência:** controller usa `noContent()`, serviço busca antes de excluir e
  `requisitarJson()` trata 204 sem tentar desserializar JSON.
- **Motivo da probabilidade:** contrato 204 tem tratamento específico no
  frontend e efeito persistente separado da resposta.
- **Consequência:** perda ou retenção silenciosa de dados e quebra de clientes
  da API.
- **Camadas afetadas:** controller, serviço, repositório, banco e cliente HTTP.
- **Ação preventiva:** verificar cabeçalhos, corpo de zero bytes e 404 em
  consulta posterior.

### CT007 — CPF com quantidade inválida — risco 12

- **Hipótese de defeito:** frontend e backend podem divergir depois da remoção
  de máscara, aceitando ou rejeitando comprimentos diferentes.
- **Evidência:** JavaScript testa comprimento 11; Bean Validation usa
  `\d{11}`; a entidade limita a coluna a 11.
- **Motivo da probabilidade:** três camadas expressam partes diferentes da
  regra; o frontend verifica comprimento, enquanto o backend também exige
  somente dígitos.
- **Consequência:** cadastro inválido ou comportamento diferente entre UI e API.
- **Camadas afetadas:** frontend, DTO, handler e banco.
- **Ação preventiva:** executar 10, 11 e 12 dígitos e complementar com letras
  diretamente na API.

### CT005 — Cadastro com campos opcionais — risco 12

- **Hipótese de defeito:** telefone vazio pode não virar `null`, ou padrões de
  matrícula/situação podem ser incorretos quando omitidos pela API.
- **Evidência:** serviço aplica `LocalDate.now()` e `Boolean.TRUE`;
  `normalizarTelefone()` converte vazio em `null`. Porém a UI preenche e envia
  matrícula e situação.
- **Motivo da probabilidade:** há ramos condicionais e transformação, mas o
  desenho atual não alcança toda a hipótese.
- **Consequência:** registros com situação ou data errada e inconsistência entre
  clientes da API.
- **Camadas afetadas:** frontend, serviço e persistência.
- **Ação preventiva:** separar o caso em UI e API; no caso de API, remover
  completamente as propriedades `dataMatricula` e `ativo` do JSON.

### CT020 — Persistência H2 e reinicialização — risco 12

- **Hipótese de defeito:** dados podem não estar visíveis durante a mesma
  execução ou podem sobreviver a um reinício contra a especificação.
- **Evidência:** URL `jdbc:h2:mem:academiadb` e
  `spring.jpa.hibernate.ddl-auto=create-drop`.
- **Motivo da probabilidade:** o resultado depende do ciclo real do processo,
  transações e conexão; não pode ser provado apenas pela configuração.
- **Consequência:** ambiente de teste contaminado ou perda/retensão de dados
  contrária ao contrato didático.
- **Camadas afetadas:** JPA, datasource, H2 e processo da aplicação.
- **Ação preventiva:** distinguir recarregar página, reiniciar contexto e
  encerrar processo; registrar PID e consultar o banco em cada etapa.

### CT016 — Pesquisa e filtros combinados — risco 12

- **Hipótese de defeito:** normalização, combinação booleana ou contador pode
  exibir alunos fora do filtro ou omitir correspondências.
- **Evidência:** `renderizarListagem()` combina nome/CPF/e-mail com ativo/inativo
  e decide entre tabela, vazio e sem resultados.
- **Motivo da probabilidade:** um único caso reúne múltiplas condições e estados
  do DOM.
- **Consequência:** consulta incorreta sem alterar os dados persistidos.
- **Camadas afetadas:** estado e renderização no frontend.
- **Ação preventiva:** usar tabela de decisão com termo vazio/preenchido,
  situação e correspondência por cada campo.

### CT015 — Cancelamento da exclusão — risco 10

- **Hipótese de defeito:** fechar por um dos mecanismos pode disparar ou deixar
  pronta uma exclusão indevida.
- **Evidência:** exclusão ocorre apenas no evento do botão confirmar; fechar,
  cancelar e `Esc` usam o comportamento do diálogo.
- **Motivo da probabilidade:** o caminho é simples, reduzindo a probabilidade,
  mas há três formas de fechar a serem verificadas.
- **Consequência:** uma falha causaria perda física de registro, justificando
  impacto 5.
- **Camadas afetadas:** eventos e estado do frontend.
- **Ação preventiva:** executar separadamente botão fechar, cancelar e `Esc`,
  monitorando ausência de `DELETE`.

## 6. Hotspots por funcionalidade técnica

Os grupos abaixo são sobrepostos: um mesmo caso pode contribuir para mais de um
hotspot. A avaliação representa concentração qualitativa, não soma independente.

| Grupo | Casos mais representativos | Probabilidade concentrada | Impacto concentrado | Evidência de hotspot |
|---|---|---|---|---|
| Cadastro | CT004–CT010 | Alta | Crítico | Muitas regras, padrões, normalização, unicidade e criação persistente |
| Atualização | CT011–CT013 | Muito alta | Crítico | Predicados com exclusão do próprio ID e sobrescrita/preservação de estado |
| Exclusão | CT014, CT015, CT019 | Moderada | Crítico | Remoção física, confirmação, estado local e contrato 204 |
| Consulta e listagem | CT001–CT003, CT016–CT018 | Moderada | Alto | Estado assíncrono compartilhado por indicadores, filtros e ações |
| Validações | CT006–CT010, CT012, CT013 | Alta | Crítico | Regras duplicadas, limites e unicidade em camadas diferentes |
| API REST | CT004, CT009–CT014, CT017–CT019 | Alta | Crítico | Cinco métodos/contratos e corpos distintos de sucesso e erro |
| Persistência | CT004, CT005, CT009–CT014, CT019, CT020 | Alta | Crítico | Restrições únicas, transações, exclusão física e ciclo de vida efêmero |
| Tratamento de erros | CT003, CT006–CT010, CT013, CT018 | Alta | Alto | 400, 404, 409, 500 e erro de rede seguem caminhos distintos |

**Maior probabilidade:** atualização, seguida por cadastro/validações.

**Maior impacto:** persistência e exclusão, seguidas por unicidade no cadastro e
na atualização.

Inferência: atualização ocupa o primeiro hotspot de probabilidade porque cada
operação combina mais estados prévios e posteriores do que o cadastro. Essa
inferência é baseada na estrutura do código, não em frequência histórica.

## 7. Ordem recomendada de execução

### P0 — Executar imediatamente

1. **CT004** — cria o primeiro registro e valida o caminho principal.
2. **CT005 como preparação de segundo registro** — usar dados distintos; para
   validar os padrões de fato, complementar com chamada direta sem as
   propriedades opcionais.
3. **CT009** — tenta repetir o CPF do primeiro registro.
4. **CT010** — tenta repetir o e-mail alterando a caixa.
5. **CT011** — atualiza um registro e confirma o estado persistido.
6. **CT014** — exclui pela interface e valida lista/indicadores.

CT005 é antecipado por dependência de massa, não por superar o risco dos casos
seguintes.

### P1 — Executar antes da liberação

1. **CT001** — confirma que a listagem e os resumos suportam os demais fluxos.
2. **CT012** — garante que o próprio CPF/e-mail não bloqueiam a atualização.
3. **CT013** — usa os dois registros para validar CPF de terceiro.
4. **CT007** — cobre a regra estrutural do identificador.
5. **CT019** — valida o contrato de exclusão diretamente na API.
6. **CT015** — confirma ausência de efeito nos três cancelamentos.
7. **CT016** — valida pesquisa, filtro e contador sobre a massa remanescente.
8. **CT020** — deve ser o último do ciclo, pois reiniciar a aplicação destrói a
   massa de dados.

### P2 — Executar na regressão

1. **CT006** — limite mínimo do nome.
2. **CT008** — formato inválido de e-mail.
3. **CT018**, após corrigir os nomes dos campos esperados — contrato 404.
4. **CT003** — indisponibilidade e nova tentativa de listagem.
5. **CT017** — contrato positivo de consulta por ID.

### P3 — Executar quando houver tempo disponível

1. **CT002** — estado visual para banco vazio.

P3 não significa ausência de valor. O caso é simples, tem baixo impacto sobre
integridade e pode ser observado naturalmente após CT020.

## 8. Lacunas preditivas e novos casos recomendados

Os identificadores abaixo continuam a sequência existente. Eles são propostas e
não devem ser considerados cobertura atual.

| Novo caso | Objetivo | Probabilidade | Impacto | Risco | Justificativa |
|---|---|---:|---:|---:|---|
| CT021 | Submeter CPF com 11 dígitos, mas semanticamente inválido | 5 | 4 | 20 — Crítico | O código aceita qualquer `\d{11}`; é necessária decisão explícita de requisito para evitar identidades fictícias |
| CT022 | Atualizar aluno usando e-mail de outro aluno, inclusive com caixa diferente | 4 | 5 | 20 — Crítico | O caso simétrico de CPF existe, mas a consulta `existsByEmailIgnoreCaseAndIdNot` não tem caso negativo dedicado |
| CT023 | Omitir `dataMatricula` e `ativo` em POST direto | 4 | 4 | 16 — Alto | CT005 não alcança esses ramos pela UI; o serviço deve aplicar hoje e `true` |
| CT024 | Enviar nascimento igual a hoje e nascimento futuro | 4 | 4 | 16 — Alto | `@Past` e a comparação do JavaScript usam fronteira estrita; DT014 existe, mas não é exercido |
| CT025 | Enviar matrícula futura e confirmar ausência de persistência | 4 | 4 | 16 — Alto | DT015 existe sem caso; frontend e `@PastOrPresent` precisam concordar |
| CT026 | Forçar erro de persistência durante cadastro/atualização e validar rollback | 3 | 5 | 15 — Alto | Transações são declaradas, mas não há evidência de recuperação nem consistência após falha |
| CT027 | Enviar o mesmo cadastro de CPF/e-mail concorrentemente | 3 | 5 | 15 — Alto | A checagem prévia tem janela de corrida; a restrição única e o handler devem garantir um único registro e 409 |
| CT028 | Chamar API diretamente com valores que o frontend normaliza | 4 | 4 | 16 — Alto | CPF com máscara não é removido pelo serviço; e-mail recebe `trim/lowercase`; há risco de divergência entre clientes |
| CT029 | Enviar telefone com 9, 10, 11, 12 dígitos e caracteres | 4 | 3 | 12 — Alto | RN006 só aparece incidentalmente em CT004/CT005 e não possui fronteiras próprias |
| CT030 | Enviar plano ausente e plano desconhecido | 3 | 4 | 12 — Alto | `null` segue Bean Validation; enum inválido segue `HttpMessageNotReadableException`, caminhos diferentes sem cobertura |
| CT031 | Enviar JSON malformado, data fora do ISO e tipos incompatíveis | 4 | 3 | 12 — Alto | O handler possui caminho específico ainda não testado e mensagem agregada para diferentes causas |
| CT032 | Excluir ID inexistente | 3 | 4 | 12 — Alto | Busca e exclusão compartilham a exceção 404, mas apenas GET inexistente está coberto |
| CT033 | Enviar nome com 3, 100 e 101 caracteres | 3 | 3 | 9 — Moderado | Só o limite inferior inválido é testado; DT007 já prevê 101 caracteres |
| CT034 | Enviar CPF com letras diretamente à API | 3 | 4 | 12 — Alto | O frontend remove caracteres, portanto o padrão `\d{11}` só é isolado por cliente direto; DT010 não é usado |
| CT035 | Simular 500 e resposta não JSON em salvar/excluir/listar | 3 | 4 | 12 — Alto | O frontend depende de `content-type` e de mensagens alternativas; o handler genérico 500 não tem caso |
| CT036 | Repetir edição/exclusão após a lista local ficar desatualizada | 3 | 5 | 15 — Alto | O frontend guarda objetos e IDs em memória; remoção concorrente pode gerar 404 após o diálogo abrir |

### 8.1 Observações sobre as lacunas

- **CPF semântico:** atualmente a ausência de validação é uma limitação
  declarada, portanto CT021 é primeiro um teste de caracterização e uma decisão
  de requisito. Só deve esperar rejeição depois que a regra for aprovada.
- **Plano inexistente, JSON malformado e datas inválidas:** estão dentro do
  contrato funcional de erro já implementado pelo handler; não exigem ampliar o
  escopo para segurança.
- **Falhas de persistência:** exigem ambiente controlável, mas continuam sendo
  testes funcionais de atomicidade e recuperação, não testes de desempenho.
- **Divergência frontend/backend:** os casos devem comparar respostas da UI e da
  API para a mesma classe de entrada, sem pressupor que a máscara será aceita
  pelo backend.

## 9. Conclusão

O conjunto atual cobre o caminho nominal do CRUD, mas a qualidade dos metadados
não permite usar o resumo e a matriz sem correção. O maior risco não está na
quantidade de casos declarados como críticos, e sim na combinação de:

1. atualização de estado existente;
2. unicidade e normalização de CPF/e-mail;
3. exclusão física;
4. persistência e transações;
5. diferenças entre o que a interface envia e o que a API aceita.

Antes de uma decisão de liberação, a equipe deve corrigir a rastreabilidade,
separar o teste de padrões do backend do CT005, ajustar o contrato do CT018 e
executar P0/P1 com confirmação direta no banco ou por consultas subsequentes.
Sem histórico de execução, qualquer evolução para “probabilidade estatística”
exigirá registrar, por caso e versão, resultados, defeitos encontrados,
severidade, área alterada e recorrência.
