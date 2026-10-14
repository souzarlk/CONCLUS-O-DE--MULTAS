# Registro oficial do mapeamento de campos — Conclusão de Multas

> **Fonte de referência do leitor.** Os exemplos abaixo indicam onde procurar cada informação; não são valores fixos. Em outra multa podem mudar empresa, placa, órgão, rodovia, KM, município, data, AIT, descrição e valores. O leitor deve buscar a mesma informação semântica a partir dos rótulos e da posição relativa no documento, sem reutilizar valores de arquivos anteriores.

## 1. Como o leitor de PDF e imagem funciona

### PDF
1. O navegador abre o documento com PDF.js.
2. Em cada página, os itens de texto são agrupados por coordenada vertical e ordenados da esquerda para a direita.
3. O parser lê rótulos e blocos descritos neste documento.
4. Se faltarem campos essenciais, o sistema renderiza as páginas em alta resolução e tenta OCR com Tesseract em português.
5. Se ainda houver campos sem leitura confiável, o sistema informa leitura parcial e indica os campos que precisam de conferência.

### Imagem JPEG/PNG
1. O arquivo é lido diretamente por Tesseract OCR em português.
2. O texto reconhecido segue para a mesma função parsePenaltyPdf usada pelo leitor de PDF.
3. Assim, as regras de mapeamento são compartilhadas por PDF e imagem; mudanças no mapeamento devem ser feitas no parser compartilhado.

### Princípios obrigatórios
- Os **valores mudam**, mas a origem semântica do campo é a mesma.
- Tolerar acentos, caixa alta/baixa, espaços, separadores e erros comuns de OCR sem inventar valores.
- Não confundir “Município” do veículo com “Município da Infração”; “Auto de Infração Originária” com o AIT principal; ou “Data da Postagem” com data da infração/vencimento.
- Preservar zeros à esquerda de KM, códigos e identificadores.
- Nunca considerar que houve leitura completa só porque algum texto foi reconhecido; validar os campos essenciais.
- Não hardcodar empresa, órgão, placa, AIT, cidade, datas, descrição de penalidade ou valores. Exceção: mapeamentos de negócio explicitamente confirmados, como Centro de Custo descrito abaixo.

## 2. Mapa completo dos campos do formulário

| Campo no site | Onde está a informação na multa | Regra para associar ao campo do site |
|---|---|---|
| **Empresa** | Bloco “PESSOA JURÍDICA PROPRIETÁRIA DO VEÍCULO NA DATA DA INFRAÇÃO ORIGINÁRIA”; valor do rótulo **Nome** (print 1) | Capturar a razão social associada a “Nome”, cortando antes dos rótulos seguintes. O nome da empresa varia por documento. |
| **Veículo** | Bloco de dados do veículo; campo **Placa** (print 2) | Ler a placa no bloco do veículo. Não usar o município deste bloco como local da infração. A interface descreve o campo como placa/modelo; não preencher modelo com outro campo por aproximação. |
| **Motorista** | Não extraído da multa pelo parser atual | Campo manual. |
| **Fornecedor** | Rótulo **Órgão Autuador** e o nome verdadeiro do órgão indicado no documento | Preencher com o nome do órgão, nunca com o próprio rótulo “Órgão Autuador”. Pode mudar entre DER e outros órgãos. |
| **Cod infração** | Não extraído pelo parser atual | Campo manual. Não confundir com o código da infração que aparece como coluna vizinha à penalidade, caso o operador precise de outro valor neste campo. |
| **N° multa** | Número principal do **Auto de Infração** (print 5) | Deve acompanhar o AIT principal. Não usar número de postagem nem AIT originário. |
| **Chegou apenas notificação** | Não extraído pelo parser atual | Campo manual. |
| **Local da multa/sinistro** | Bloco/tabela encabeçado por **Local / Rodovia** (print 3). Linha dos valores: Rodovia, KM, metros, sentido, **Município da Infração**, UF, código e data/hora | Montar RODOVIA + “KM” + número do KM + município da infração. Exemplo de formato: **SP 150 KM 043 CUBATÃO**. O exemplo serve apenas para demonstrar o formato: nunca fixar rodovia, KM ou município. Preservar zeros à esquerda (043). Procurar a cidade nesta linha/bloco e não no bloco do veículo (que pode indicar SANTOS ou outra cidade). |
| **Histórico multa** | Descrição logo abaixo da coluna/rótulo **Penalidade aplicada**, dentro de “TIPIFICAÇÃO DA PENALIDADE APLICADA” (print 8) | Capturar a descrição completa da penalidade. Parar antes das colunas vizinhas, como “Cód. Infração”, “Desdob.”, base legal, datas/horários. Não retornar o cabeçalho nem texto de rodapé. Normalizar erros óbvios de OCR sem fixar o texto de uma multa específica. |
| **Peças danificadas** | Não extraído pelo parser atual | Campo manual. |
| **Prejuízo valor** | Calculado a partir da tabela inferior | Valor líquido menos Des. valor; matematicamente equivale a Valor com Desconto. Não extrair como número independente. |
| **Des. valor** | Calculado usando **Valor Integral** e **Valor com Desconto** na tabela inferior | Valor Integral menos Valor com Desconto. É a diferença entre os dois valores do documento. |
| **Valor líquido** | Rótulo **Valor Integral** na tabela inferior | Copiar o Valor Integral do documento. O montante varia entre multas. |
| **Tipo ocorrência** | Regra fixa do sistema | Sempre “MULTA”; não extrair da multa. |
| **Data da infração** | Bloco da infração, rótulos **Data** e **Hora**, junto à área “Município da Infração” (print 4) | Combinar a data e o horário da ocorrência. Não confundir com postagem, vencimento, prazo de recurso ou data do arquivo. |
| **Data da chegada da cobrança** | Não extraído pelo parser atual | Campo manual. |
| **Pontos multa** | Não extraído pelo parser atual | Campo manual. |
| **N° AIT** | **Auto de Infração** principal, no bloco da infração (print 5) | Extrair o identificador completo com letras, números e sufixo se existir. Excluir o AIT originário; nunca escolher apenas a primeira sequência numérica encontrada no PDF. |
| **N° AIT originária** | Rótulo **Auto de Infração Originária** (print 6) | Capturar o identificador associado especificamente a esse rótulo, separado do AIT principal. |
| **Gastos com veículo** | Não extraído pelo parser atual | Campo manual. |
| **N° da fatura** | Sistema de destino, não faz parte da multa | Campo desabilitado/automático. Não preencher com AIT, código ou número de postagem. |
| **Data de vencimento** | Rótulo **Data de Vencimento** na tabela inferior, perto de **Valor Integral** e **Valor com Desconto** (print 7) | Capturar a data de vencimento da tabela. Não usar a data da infração, postagem nem prazo de recurso. |
| **Pago pelo motorista** | Não extraído pelo parser atual | Campo manual. |
| **Multa descontada** | Não extraído pelo parser atual | Campo manual. |
| **Data de desconto** | Não extraído pelo parser atual | Campo manual. |
| **Centro de custo** | Regra de associação baseada na empresa extraída | Mapeamento confirmado: **CRC DA COSTA TRANSPORTES LTDA → FROTA CRC**. Recalcular em cada leitura. Não atribuir o mesmo centro de custo a empresas diferentes sem uma regra confirmada. |

## 3. Prints anotados que definem as origens

- **1 EMPRESA.jpeg** — Empresa / rótulo “Nome”.
- **2 VEICULO.jpeg** — bloco do veículo / placa.
- **3 LOCAL DO MULTA SINISTRO.jpeg** — “Local / Rodovia”, KM e Município da Infração.
- **4 DATA DA INFRAÇÃO.jpeg** — data e hora da infração.
- **5 N° AIT.jpeg** — AIT principal.
- **6 N° AIT ORIGINARIA.jpeg** — AIT originária.
- **7 DATA DE VENCIMENTO.jpeg** — vencimento na tabela inferior.
- **8 HISTORICO DA MULTA.jpg** — descrição abaixo de “Penalidade aplicada”.

Os prints marcam a posição dos campos, mas os valores impressos mudam a cada documento.

## 4. Checklist obrigatório antes de publicar alteração no parser

- [ ] Testar PDF digital com texto selecionável.
- [ ] Testar PDF que precise de OCR.
- [ ] Testar uma imagem JPEG/PNG.
- [ ] Empresa vem do rótulo “Nome” no bloco da pessoa jurídica proprietária.
- [ ] Fornecedor vem do nome real após/associado a “Órgão Autuador”.
- [ ] Local inclui rodovia, KM e Município da Infração, nunca só rodovia/KM se a cidade está no documento.
- [ ] A cidade do local não é copiada do bloco do veículo.
- [ ] Histórico mantém a descrição integral de “Penalidade aplicada” sem incorporar o código/colunas vizinhas.
- [ ] AIT principal e AIT originária permanecem separados.
- [ ] Data da infração, data da postagem e vencimento não são confundidos.
- [ ] Valor líquido = Valor Integral.
- [ ] Des. valor = Valor Integral − Valor com Desconto.
- [ ] Prejuízo valor = Valor líquido − Des. valor.
- [ ] Centro de custo CRC da Costa → FROTA CRC.
- [ ] Campos manuais continuam manuais e N° da fatura não é extraído da multa.
- [ ] Novo arquivo não reutiliza dados automáticos da multa anterior.

## 5. Manutenção do registro

Este arquivo é a referência durável do projeto. Atualizar este mapa junto com qualquer alteração relevante no parser. Se o layout da multa mudar, identificar primeiro o rótulo e o bloco de origem, modificar apenas a regra correspondente e revalidar os campos já corretos. Não substituir dados dinâmicos por exemplos fixos.
