# Base de Conhecimento

## Dados Utilizados

A base de conhecimento foi estruturada mesclando os dados transacionais do usuário (mockados) com bases de dados públicas financeiras para fornecer um contexto educativo robusto, sem ferir a regra de não fazer recomendações diretas de risco.

| Arquivo | Formato | Utilização no Agente |
|---------|---------|---------------------|
| `historico_atendimento.csv` | CSV | Contextualizar interações anteriores e manter a fluidez da conversa. |
| `perfil_investidor.json` | JSON | Identificar a tolerância a risco (Conservador/Moderado) para adequar o tom das explicações. |
| `transacoes.csv` | CSV | Analisar o padrão de gastos do cliente e identificar oportunidades de economia. |
| `produtos_financeiros.json` | JSON | Catálogo interno com explicações simplificadas de produtos (CDB, Tesouro Direto, LCI). |
| `bcb_taxas_historico.csv` | CSV | **Dataset Público (Banco Central):** Traz o histórico e valor atual da taxa Selic e IPCA para explicar o rendimento do dinheiro no tempo. |
| `finqa_ptbr_adapted.json` | JSON | **Dataset Público (Hugging Face - adaptado):** Baseado em datasets de QA financeiro (como o *FinQA*), traduzido e simplificado para treinar a IA a responder dúvidas complexas com analogias do dia a dia. |

---

## Adaptações nos Dados

Os dados mockados originais foram significativamente expandidos para suportar o pilar de "Educação Financeira" da IA:
1. **Enriquecimento com Dados Públicos:** O arquivo `produtos_financeiros.json` foi alimentado com informações reais extraídas do Tesouro Nacional Transparente, convertendo as descrições técnicas em analogias (ex: comparar a liquidez diária a uma "reserva de emergência que fica na gaveta de casa, sempre pronta para uso").
2. **Filtro de Complexidade:** O dataset público do Hugging Face foi higienizado. Removemos jargões de Wall Street e opções de derivativos, focando apenas em conceitos de Renda Fixa, poupança e juros compostos, adequando-se ao público de 16 a 80 anos.

---

## Estratégia de Integração

### Como os dados são carregados?
Para garantir que o agente responda rapidamente, adotamos uma abordagem híbrida:
- **Dados do Cliente (Contexto Frio):** Os arquivos JSON/CSV do usuário (como `transacoes.csv` e `perfil_investidor.json`) são carregados na inicialização da sessão e mantidos na memória de curto prazo.
- **Dados Públicos Educativos (Contexto Quente/RAG):** Os datasets com conceitos de investimentos, taxas (Selic/IPCA) e o catálogo de produtos financeiros são indexados em um banco de dados de busca semântica (RAG - *Retrieval-Augmented Generation*). Eles só são puxados quando o usuário faz uma pergunta específica sobre investimentos.

### Como os dados são usados no prompt?
Os dados básicos do cliente vão fixos no **System Prompt** para garantir que a IA sempre saiba com quem está falando e qual a realidade financeira da pessoa. 

Quando o cliente pede uma dica de investimento, o sistema busca no banco de dados público as opções mais seguras (Renda Fixa) e injeta no prompt dinamicamente com uma instrução restritiva: *"Baseado no perfil do cliente, explique de forma didática os conceitos abaixo. Não diga a ele onde investir, apenas mostre como esses produtos funcionam e simule um cenário com o saldo disponível"*.

---

## Exemplo de Contexto Montado

Quando o cliente envia a mensagem *"O que eu faço com o dinheiro que sobrou este mês?"*, o prompt montado que chega para a LLM por baixo dos panos é assim:

```text
[INSTRUÇÕES DO SISTEMA]
Você é a Clara, assistente financeira acolhedora. Responda de forma simples, usando analogias do dia a dia. Não faça recomendações diretas, atue como educadora.

[DADOS DO CLIENTE]
- Nome: Dona Maria
- Idade: 65 anos
- Perfil: Conservador
- Saldo disponível para poupar este mês: R$ 300,00

[ÚLTIMAS TRANSAÇÕES - DESTAQUES]
- 05/11: Farmácia - R$ 120,00
- 10/11: Conta de Luz - R$ 95,00

[CONTEXTO RECUPERADO DOS DATASETS PÚBLICOS]
- Taxa Selic Atual: 10,75% ao ano.
- Conceito Sugerido: Tesouro Selic (Baixíssimo risco, rende todo dia, pode tirar quando quiser).
- Analogia sugerida: É como guardar dinheiro num cofre do governo que te paga um pouquinho de aluguel todos os dias.

[MENSAGEM DO USUÁRIO]
"O que eu faço com o dinheiro que sobrou este mês?"
