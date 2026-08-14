<p align="center">
  <img src="afrelefante.png" alt="Mascote AFRE Sefaz/RN" width="200"/>
</p>

<h1 align="center">remunerasefazrn</h1>

<p align="center">
  Simulador não-oficial da remuneração mensal (bruta e líquida) do cargo de<br/>
  <strong>Auditor Fiscal de Receitas Estaduais (AFRE)</strong> da Secretaria de Estado da Tributação do Rio Grande do Norte (Sefaz/RN).
</p>

<p align="center">
  🔗 <strong><a href="https://bacalhaunabrisa.github.io/remunerasefazrn/">bacalhaunabrisa.github.io/remunerasefazrn</a></strong>
</p>

---

## Sobre o projeto

Este repositório contém uma única página HTML autocontida (`index.html`) que simula, campo a campo, todas as verbas e descontos que compõem o contracheque de um AFRE da Sefaz/RN — do vencimento básico ao Imposto de Renda — e apresenta o resultado em tempo real, sem necessidade de servidor, backend ou build: basta abrir o arquivo no navegador (ou acessá-lo via GitHub Pages).

O projeto é uma ferramenta de estimativa pessoal, sem qualquer vínculo institucional com a Sefaz/RN. Os parâmetros normativos (valores, alíquotas, faixas) foram inseridos manualmente com base na legislação vigente à época da criação da página e podem ficar desatualizados caso a legislação mude.

## Como usar

1. Escolha o **padrão remuneratório** (AFRE-1 a AFRE-5).
2. Preencha as verbas adicionais (dias de Ajuda de Custo, ATS, horas noturnas).
3. Escolha o **teto remuneratório** vigente e o **regime previdenciário**.
4. Informe dependentes e outras deduções para o Imposto de Renda, e ative os toggles de descontos consignados (Sindifern/Asfarn), se aplicável.
5. O painel à direita ("Remuneração líquida estimada") recalcula automaticamente a cada alteração — não há botão de "calcular".

## Lógica de cálculo

Todo o cálculo roda **no navegador do usuário**, em JavaScript puro (nenhum dado é enviado a um servidor). A lógica segue, em ordem, os seguintes passos:

### 1. Proventos (formam a Remuneração Bruta)

| Verba | Regra |
|---|---|
| **Vencimento básico** | Valor fixo por padrão (AFRE-1 a AFRE-5), conforme tabela na própria página (que também exibe o valor monetário total das UPVs por padrão). |
| **UPVs** (Unidade de Parcela Variável) | `quantidade de UPVs do padrão × R$ 294,64`. Zerada se o padrão for AFRE-1 e o usuário ativar o toggle "em curso de formação" (Art. 46 da LOAT-RN). |
| **Adicional de periculosidade** | `30% do vencimento básico`, sempre devido em todos os padrões, independente das UPVs. |
| **Ajuda de Custo Operacional de Fiscalização** | Valor fixo por dia = `2,30% × (vencimento básico do AFRE-5 + UPVs do AFRE-5)`, multiplicado pelo número de dias informado (0 a 7). É indenizatória: não sofre desconto e não entra nas bases previdenciária/IR. |
| **ATS** (Adicional por Tempo de Serviço) | Percentual (0% a 35%, conforme faixa de tempo escolhida) `× vencimento básico`. |
| **Adicional noturno** | Calculado sobre `vencimento básico + UPVs + periculosidade + ATS`, dividido por 150 (horas mensais) para obter o valor-hora normal; esse valor-hora é multiplicado por 25% (acréscimo noturno) e pelo fator `60/52,5` (hora noturna reduzida de 52min30s), resultando no valor-hora noturno, que é então multiplicado pela quantidade de horas noturnas informada (0 a 56), conforme art. 82 da Lei Complementar nº 122/1994 (RJU-RN). É integralmente remuneratório. |

`Remuneração Bruta = vencimento básico + UPVs + periculosidade + ajuda de custo + ATS + adicional noturno`

### 2. Deduções (formam a Remuneração Líquida)

| Dedução | Regra |
|---|---|
| **Redutor Art. 37/CF** | Se `(vencimento básico + UPVs + periculosidade + ATS + adicional noturno)` ultrapassar o teto remuneratório escolhido, o excedente é descontado integralmente. |
| **Desconto previdenciário oficial** | Aplicado sobre a *base previdenciária* (`vencimento básico + UPVs + ATS`, limitada ao teto escolhido; **não inclui** periculosidade nem adicional noturno), com alíquotas progressivas por faixa (11% a 18%), conforme Portaria nº 001/2026/CRH/PR. No regime **Previdência Complementar**, essa base só é tributada até a 2ª faixa (11% e 14%). |
| **Desconto previdenciário complementar** | Só existe no regime Previdência Complementar: alíquota escolhida pelo usuário (0%, 7,5%, 8,0% ou 8,5%, padrão 8,5%) aplicada sobre a parcela de `vencimento básico + UPVs + ATS + adicional noturno` (limitada ao teto) que excede R$ 8.475,55. |
| **Imposto de Renda** | Base de cálculo = `(vencimento básico + UPVs + periculosidade + ATS + adicional noturno, limitada ao teto) − desconto previdenciário oficial − desconto previdenciário complementar − (dependentes × R$ 189,59) − outras deduções`, nunca negativa. Sobre essa base aplica-se a tabela progressiva por faixas (0%, 7,5%, 15%, 22,5%, 27,5%). |
| **Descontos consignados em folha** | Sindifern (0,5%) e/ou Asfarn (0,4%) sobre `vencimento básico + UPVs`, limitado ao teto remuneratório escolhido (caso essa soma o ultrapasse), se os respectivos toggles estiverem ativados (ambos desligados por padrão). |

`Remuneração Líquida = Remuneração Bruta − Redutor Art. 37/CF − desconto previdenciário oficial − desconto previdenciário complementar − Imposto de Renda − descontos consignados`

### Regra de arredondamento

Em todo o simulador, qualquer valor monetário calculado com mais de duas casas decimais é **truncado** (nunca arredondado) para duas casas — os algarismos excedentes são simplesmente descartados. Essa regra está centralizada numa única função utilitária (`truncar()`), aplicada logo após cada cálculo intermediário, para que o resultado final reproduza fielmente essa lógica em cada etapa da conta.

## Tecnologias utilizadas

- **HTML5** — estrutura da página, em arquivo único (`index.html`).
- **CSS3** puro — sem framework; variáveis CSS (`:root`) para cores/tema, grid/flexbox para o layout responsivo (colunas empilham em telas estreitas) e fontes do Google Fonts (*Space Grotesk*, *Inter*, *JetBrains Mono* para os valores monetários).
- **JavaScript (ES5/ES6)** — toda a lógica de cálculo, validação de campos e atualização da interface.
- **jQuery 3.7** (via CDN) — manipulação do DOM e eventos (`change`/`input`) que disparam o recálculo automático.
- **GitHub Pages** — hospedagem estática, sem backend, sem build step, sem dependências instaladas: o repositório é publicado como está.

Não há framework de front-end (React, Vue etc.), bundler ou etapa de compilação — o projeto é intencionalmente simples para poder ser mantido e publicado direto pela interface do GitHub, do mesmo modo que o projeto irmão [`cebraspe`](https://github.com/BacalhauNaBrisa/cebraspe).

## Estrutura do repositório

```
remunerasefazrn/
├── index.html        # página única: HTML + CSS + JS embutidos
├── afrelefante.png   # logotipo/mascote do projeto
└── README.md         # este arquivo
```

## Aviso legal

Simulador independente e não-oficial, sem qualquer vínculo com a Sefaz/RN. Os valores, alíquotas e regras aqui reproduzidos têm caráter meramente estimativo e ilustrativo. Consulte sempre a legislação vigente e o órgão competente para fins oficiais antes de tomar qualquer decisão com base nos resultados desta página.
