---
status: aceita
---

# O agente escreve por operações incrementais e lê por consultas compactas

O agente altera o mapa enviando lotes de operações (criar ou atualizar bloco, criar ou atualizar relação, remover, marcar como não investigado). Cada lote é aplicado de forma atômica e devolve os erros de validação. Reenviar o documento inteiro custaria tokens e apagaria correções feitas em paralelo.

Para ler o mapa, o agente usa consultas que retornam pouco: um resumo do mapa, um bloco com suas relações e âncoras, e a vizinhança de um bloco. Quando o usuário seleciona algo, o aplicativo envia apenas os IDs e a pergunta, e o agente busca o resto sob demanda.
