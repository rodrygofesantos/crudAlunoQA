# Prompt para análise preditiva de casos de teste

Atue como cientista de dados especializado em Engenharia de Qualidade de Software, análise de riscos e testes orientados por dados.

Analise o arquivo anexado `CASOS_DE_TESTE_FUNCIONAIS.md` e, caso esteja disponível, consulte também o código-fonte citado nas evidências de cada caso de teste.

## Objetivo

Realizar uma análise preditiva baseada em risco para identificar:

1. Quais casos de teste apresentam maior probabilidade de revelar defeitos.
2. Quais defeitos teriam maior impacto funcional, técnico e operacional.
3. Quais funcionalidades representam os principais *hotspots* de risco.
4. Qual deve ser a ordem recomendada para execução dos testes.
5. Quais lacunas de teste podem ocultar defeitos relevantes.

## Regra sobre os dados

Antes da análise, verifique a qualidade e a consistência do documento:

- reconte os casos diretamente nas seções `CT001`, `CT002` etc.;
- não confie automaticamente no resumo de cobertura;
- confronte as prioridades declaradas com os dados de cada caso;
- valide a matriz de rastreabilidade;
- identifique requisitos ausentes, associações incorretas, duplicidades e inconsistências;
- diferencie claramente a prioridade informada no documento do risco calculado pela análise.

Se não forem fornecidos resultados anteriores de execução, defeitos históricos, cobertura, complexidade, alterações de código ou incidentes, informe:

> Esta é uma estimativa heurística de risco, e não uma probabilidade estatística calculada a partir de dados históricos.

Nesse cenário, não apresente percentuais como “80% de probabilidade”. Utilize níveis qualitativos e pontuações justificadas.

## Critérios de probabilidade de defeito

Atribua a cada caso uma pontuação de 1 a 5:

- **1 — Muito baixa**
- **2 — Baixa**
- **3 — Moderada**
- **4 — Alta**
- **5 — Muito alta**

Considere:

- quantidade de regras e condições envolvidas;
- fluxo positivo, negativo ou de valor-limite;
- integração entre *frontend*, API, serviço, repositório e banco;
- validações duplicadas entre *frontend* e *backend*;
- tratamento de exceções;
- normalização e transformação de dados;
- verificação de unicidade de CPF e e-mail;
- alteração de estado e persistência;
- possibilidade de inconsistência entre camadas;
- ambiguidades ou lacunas na especificação;
- quantidade de requisitos cobertos por um único caso;
- evidências encontradas no código-fonte.

**Definição:** probabilidade significa a chance de o caso de teste revelar um defeito no produto, e não a chance de falhar por problema de ambiente.

## Critérios de impacto

Atribua uma pontuação de impacto de 1 a 5:

- **1 — Insignificante**
- **2 — Baixo**
- **3 — Moderado**
- **4 — Alto**
- **5 — Crítico**

Considere:

- indisponibilidade de uma função principal;
- criação de dados incorretos;
- duplicidade ou perda de registros;
- alteração ou exclusão indevida;
- comprometimento da integridade dos dados;
- propagação do erro para outras funcionalidades;
- quantidade de usuários ou fluxos afetados;
- dificuldade de recuperação;
- impacto na confiança do usuário;
- violação de uma regra central do sistema.

## Cálculo do risco

Calcule:

```text
Risco = Probabilidade × Impacto
```

Classifique o resultado:

- **1 a 4 — Baixo**
- **5 a 9 — Moderado**
- **10 a 16 — Alto**
- **17 a 25 — Crítico**

Não altere a pontuação apenas para fazê-la coincidir com a prioridade já informada no documento.

## Resultado esperado

Gere dois artefatos complementares:

1. um relatório técnico em Markdown, que funcione como fonte completa e
   auditável da análise;
2. um arquivo HTML autônomo no formato de **dashboard executivo interativo**,
   pronto para ser aberto localmente no navegador, sem servidor e sem
   dependências externas.

O HTML é a entrega principal para consulta. Não apresente o conteúdo como uma
simples conversão linear do Markdown. Organize os dados para leitura visual,
comparação e tomada de decisão.

### Diretrizes obrigatórias para o dashboard

- mostrar no primeiro bloco os indicadores: total de casos, casos críticos,
  casos críticos + altos, lacunas propostas e nível de confiança predominante;
- destacar os cinco casos de maior risco em componentes visuais compactos;
- apresentar a distribuição dos casos por classificação de risco;
- apresentar uma matriz 5 × 5 de probabilidade e impacto, posicionando cada caso
  na célula correspondente;
- apresentar os *hotspots* em gráfico de barras comparável;
- apresentar a ordem P0–P3 em raias ou colunas, preservando dependências;
- disponibilizar ranking completo com pesquisa e filtros por classificação;
- permitir consultar a justificativa detalhada dos casos críticos e altos por
  expansão, painel ou diálogo, sem exibir todo o texto simultaneamente;
- apresentar auditoria e lacunas em tabelas responsivas, com destaque visual
  para severidade e ação recomendada;
- usar rótulos, valores e padrões além de cor para comunicar risco;
- manter legenda consistente: baixo, moderado, alto e crítico;
- aplicar hierarquia visual clara, espaçamento adequado e contraste acessível;
- adaptar o layout a telas grandes e celulares, sem cortar conteúdo;
- oferecer estilos de impressão para exportação em PDF;
- usar HTML semântico, controles acessíveis por teclado e textos em português
  do Brasil;
- manter CSS, JavaScript e dados no próprio HTML;
- não usar bibliotecas, fontes, APIs ou arquivos externos;
- não usar `fetch`, servidor local ou conexão com a internet;
- incluir aviso visível de que o resultado é uma estimativa heurística;
- não inventar métricas para preencher cartões ou gráficos: todos os números
  devem ser derivados e reconciliáveis com o ranking.

Evite:

- páginas formadas principalmente por parágrafos longos;
- tabelas extensas como primeiro elemento da página;
- cartões decorativos sem função analítica;
- gráficos redundantes que comuniquem a mesma relação;
- animações excessivas;
- percentuais de probabilidade sem dados históricos.

### 1. Resumo executivo

Apresente:

- os cinco casos com maior risco;
- as funcionalidades com maior probabilidade de defeitos;
- os defeitos com maior impacto;
- os principais riscos para a integridade dos dados;
- uma conclusão sobre onde a equipe deve concentrar os testes.

### 2. Auditoria da qualidade dos dados

Apresente uma tabela:

| Inconsistência encontrada | Evidência | Consequência para a análise | Correção recomendada |
|---|---|---|---|
|  |  |  |  |

### 3. Ranking preditivo dos casos

Apresente todos os casos, ordenados do maior para o menor risco:

| Posição | Caso | Funcionalidade | Prioridade original | Probabilidade | Impacto | Risco | Classificação | Confiança |
|---:|---|---|---|---:|---:|---:|---|---|
|  |  |  |  |  |  |  |  |  |

Use confiança **Alta**, **Média** ou **Baixa**, de acordo com a quantidade e a qualidade das evidências disponíveis.

### 4. Justificativa dos casos mais críticos

Para cada caso classificado como risco crítico ou alto, apresente:

- hipótese de defeito;
- evidência utilizada;
- motivo da probabilidade;
- consequência para o sistema;
- camadas afetadas;
- ação preventiva recomendada.

Não use justificativas genéricas.

### 5. *Hotspots* por funcionalidade

Agrupe os riscos em:

- cadastro;
- atualização;
- exclusão;
- consulta e listagem;
- validações;
- API REST;
- persistência;
- tratamento de erros.

Informe qual grupo concentra maior probabilidade e qual concentra maior impacto.

### 6. Ordem recomendada de execução

Organize os casos em:

- **P0 — Executar imediatamente**
- **P1 — Executar antes da liberação**
- **P2 — Executar na regressão**
- **P3 — Executar quando houver tempo disponível**

Considere as dependências entre os casos e explique resumidamente a ordem.

### 7. Lacunas preditivas

Identifique comportamentos com risco relevante que não estejam adequadamente cobertos, tais como:

- validações de fronteira ausentes;
- CPF sintaticamente correto, mas semanticamente inválido;
- datas inválidas ou futuras;
- telefone inválido;
- plano inexistente;
- JSON malformado;
- exclusão de aluno inexistente;
- divergência entre validação do *frontend* e do *backend*;
- erros de persistência e recuperação.

Para cada lacuna, proponha um novo caso de teste com identificador, objetivo, probabilidade, impacto e justificativa.

## Restrições

- Utilize somente evidências encontradas no documento e no código-fonte.
- Não invente histórico de defeitos, execuções ou métricas.
- Não confunda prioridade, probabilidade, impacto e risco.
- Não use percentuais sem dados históricos suficientes.
- Declare explicitamente todas as inferências.
- Mantenha a análise dentro do escopo funcional definido no documento.
- Escreva em português do Brasil, com linguagem técnica, objetiva e adequada a uma disciplina de pós-graduação.

## Observação sobre o documento analisado

Antes de executar o prompt, verifique as divergências já identificadas no arquivo:

- os casos individuais resultam em **8 críticos, 10 altos e 2 médios**, enquanto o resumo informa **8 críticos, 8 altos e 4 médios**;
- foram identificados **11 casos positivos**, e não 10.

Essas divergências devem ser corrigidas antes de o resultado ser tratado como uma análise orientada por dados.
