# Regras de negócio

Resumo das regras implementadas em `index.html` (funções `lerCaixa`, `lerRede` e `conciliar`).

## Entradas

| Fonte | Formatos | Observações |
|---|---|---|
| Relatório Rede | `.xlsx`, `.xls` | Transações de cartão e PIX da adquirente |
| Relatório QVIS (Caixa) | `.xlsx`, `.xls`, `.pdf` | Aceita múltiplos arquivos |
| Histórico | `.json` | Gerado por **Exportar Histórico** (`tipo: conciliador-historico`) |

## Unidades disponíveis

CBV Matriz, Taguatinga, Águas Claras, Hospital Águas Claras (HAC) e Lago Norte. A lista fica no `<select id="inputUnidade">`.

## Filtragem dos lançamentos

- **Dinheiro e transferência** ficam fora da conciliação, mas entram no totalizador por pessoa.
- Lançamentos do usuário **TESOURARIA TRANSBORDO** são descartados do Caixa (`NOMES_CAIXA_EXCLUIDOS`).
- Status de venda rejeitados na Rede (negado, estornado, cancelado, recusado) não entram (`STATUS_REJEITADOS`).

## Chave de conciliação

- Cartão: `NSU|Autorização`, normalizados (sem acentos, maiúsculas, sem zeros à esquerda).
- PIX: chave própria com prefixo `PIX|`.
- Registros sem chave ficam à parte (`semChaveCaixa`, `semChaveRede`).
- Chaves duplicadas são separadas (`dupCaixa`, `dupRede`) e não entram no cruzamento.

## Cancelamento e relançamento

Se uma chave aparece no Caixa exatamente duas vezes, uma como RECEBIMENTO e outra como CANCELAMENTO/ESTORNO, o par é tratado como cancelado: o recebimento segue para o cruzamento marcado como cancelado.

Se uma **admissão** aparece na Conciliação e também em "Somente na Caixa QVIS", entende-se que o recebimento foi cancelado e relançado corretamente. Essa admissão é removida de "Somente na Caixa QVIS".

## Divergências verificadas (cartão)

Valor, bandeira, parcelas (vazio no Caixa equivale a 1 na Rede), NSU e autorização. Qualquer diferença marca a linha como **NÃO CONFERE** e descreve o motivo. Para PIX, só o valor é comparado.

## Categorias de resultado

| Categoria | Significado |
|---|---|
| Conciliação | Presente nos dois lados (Confere / Não confere) |
| Somente PIX | Todos os PIX do Caixa, conciliados ou não, em tópico próprio |
| Somente na Caixa QVIS | Está no Caixa e não na Rede |
| Somente na Rede | Está na Rede e não no Caixa; exibe o setor pelo código da maquininha |

## Setor por maquininha

O mapa `SETOR_POR_MAQUININHA` associa o código da maquininha ao setor (Caixa Principal, Lentes, Oftalmo, Caixa EV, Analista EV, Link Pagamento). É usado só para exibição na tabela "Somente na Rede" e não afeta a conciliação. Códigos novos aparecem com o próprio código até serem adicionados ao mapa.

## Totais e ajustes manuais

- **Total Relatório Rede**: crédito + débito + PIX aprovados.
- **Total Relatório Caixa**: soma de todos os lançamentos válidos.
- **Diferença (Caixa − Rede)**: considera os ajustes manuais, que não alteram os totais dos relatórios.

## Capa

O botão **Baixar Capa** gera um PDF de uma página com o resumo, e só é habilitado depois de concluída a conciliação.
