---
status: aceita
---

# Análise cancelada ou com falha preserva o que já foi feito

Se a análise for cancelada ou falhar, as operações já aplicadas continuam no mapa. O snapshot fica marcado como incompleto. O que nunca foi lido aparece como não investigado e, numa revalidação, os blocos que faltou conferir continuam desatualizados. A mesma regra vale quando acaba o teto de turnos (ADR-0040). Uma nova análise retoma desse estado, sem começar do zero.
