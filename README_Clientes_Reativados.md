# Relatório de Clientes Reativados

## 📋 Contexto de Negócio

Em ambientes de vendas B2B e varejo, identificar clientes inativos que voltaram a comprar é fundamental para:
- Medir efetividade de campanhas de reativação
- Entender padrões de sazonalidade no comportamento de compra
- Identificar oportunidades de fidelização
- Distinguir entre clientes reativados e clientes verdadeiramente novos

Este relatório foi desenvolvido para fornecer à gestão comercial visibilidade sobre movimentações de clientes entre períodos, permitindo análises estratégicas baseadas em dados reais de vendas.

---

## 🎯 Problema

A empresa precisava responder perguntas como:
- **Quais clientes pararam de comprar no período X mas voltaram no período Y?**
- **Quem são os clientes verdadeiramente novos (primeira compra)?**
- **Qual o valor gerado por clientes reativados vs novos clientes?**

Sistemas ERP tradicionais não ofereciam essa análise de forma flexível, exigindo consultas manuais complexas ou relatórios estáticos que não permitiam exploração por diferentes intervalos de tempo.

---

## ✅ Solução Técnica

Desenvolvi um **relatório paginado no Power BI Report Builder** conectado a SQL Server (local e Fabric em nuvem) que permite:

### Funcionalidades Principais:
1. **Parâmetros flexíveis de data** - usuário define:
   - Período sem vendas (Data Início Passado → Data Fim Passado)
   - Período com vendas (Data Início Atual → Data Fim Atual)

2. **Dois cenários de análise**:
   - **Clientes Reativados**: quem não comprou em janeiro mas voltou em fevereiro
   - **Clientes Novos**: quem nunca comprou antes (filtrando "sem vendas" desde 1900)

3. **Agrupamento com totalizadores**:
   - Total de clientes identificados
   - Soma de valores gerados no período de retorno

---

## 🔧 Recursos Utilizados

### Power BI Report Builder
- **Parâmetros de entrada** (4 parâmetros de data)
- **Agrupamentos e subtotais** (count de clientes + sum de vendas)
- **Conexão híbrida** (SQL Server local + Fabric Cloud para demonstração online)

### SQL Server
- Lógica `EXISTS` / `NOT EXISTS` para comparação temporal
- Joins entre tabelas de clientes, vendas, cidades e filiais
- Filtragem dinâmica via parâmetros

---

## 💡 Como Funciona

### Exemplo 1: Clientes Reativados
```
Parâmetros:
- Data Sem Venda: 01/01/2026 a 31/01/2026
- Data Com Venda: 01/02/2026 a 28/02/2026

Resultado: Clientes que NÃO compraram em janeiro mas COMPRARAM em fevereiro
```

### Exemplo 2: Clientes Novos
```
Parâmetros:
- Data Sem Venda: 01/01/1900 a 31/01/2026
- Data Com Venda: 01/02/2026 a 28/02/2026

Resultado: Clientes com primeira compra em fevereiro de 2026
```

---

## 📊 Query SQL (Simplificada e Comentada)

```sql
SELECT 
    c.cliente_id,
    'CLIENTE ' + RIGHT('00000' + CAST(c.cliente_id AS VARCHAR), 5) AS CodigoCliente,
    cd.descricao AS cidade,
    c.uf,
    m.pedido_id,
    m.nf,
    m.[data],
    m.total,
    'FILIAL ' + RIGHT('00' + CAST(f.filial_id AS VARCHAR), 2) AS NomeFilial
FROM 
    clientes_silver c
    LEFT JOIN cidades_silver cd ON cd.cidade_id = c.cidade_id
    LEFT JOIN vendas_silver m ON c.cliente_id = m.cliente_id
    JOIN filiais_silver f ON f.filial_id = m.filial_id
WHERE 
    -- Cliente COMPROU no período atual (fevereiro)
    EXISTS (
        SELECT 1 
        FROM vendas_silver v 
        WHERE v.cliente_id = c.cliente_id 
          AND v.data >= @DataIniAtual 
          AND v.data <= @DataFimAtual
    )
    -- Cliente NÃO COMPROU no período passado (janeiro)
    AND NOT EXISTS (
        SELECT 1 
        FROM vendas_silver v 
        WHERE v.cliente_id = c.cliente_id 
          AND v.data >= @DataIniPassado 
          AND v.data <= @DataFimPassado
    )
```

### Lógica da Query:
- **EXISTS**: verifica se há vendas no período atual (garantindo que o cliente comprou)
- **NOT EXISTS**: verifica se NÃO há vendas no período passado (garantindo inatividade anterior)
- A combinação dessas condições identifica clientes que "voltaram" ou são novos

---

## 🛠️ Tecnologias

- **SQL Server** (local e Fabric Cloud)
- **Power BI Report Builder** (relatórios paginados)
- **T-SQL** (lógica de comparação temporal)

---

## 📸 Demonstração

> **[Adicionar screenshots aqui]**
> - Tela de parâmetros do relatório
> - Exemplo de saída (clientes reativados)
> - Exemplo de saída (clientes novos)
> - Totalizadores finais

---

## 🎓 Aprendizados

- **Modelagem temporal**: uso de EXISTS/NOT EXISTS para comparações eficientes entre períodos
- **Flexibilidade via parâmetros**: mesmo relatório serve múltiplos cenários de análise
- **Arquitetura híbrida**: SQL Server local + Fabric permite demonstração online
- **Clareza de negócio**: transformar pergunta gerencial em query SQL defensável

---

## 📝 Observações

Este relatório foi desenvolvido em contexto real de trabalho com dados de vendas. A estrutura de dados reflete um ambiente de produção com tabelas dimensionais (clientes, cidades, filiais) e tabela fato (vendas).

A escolha de relatório paginado (Report Builder) ao invés de dashboard interativo se justifica pela necessidade de:
- Exportação em PDF para distribuição
- Listagem detalhada de clientes (não apenas métricas agregadas)
- Processamento server-side de volumes maiores de dados
