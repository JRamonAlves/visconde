---
status: aceita
---

# O aplicativo decide o layout

O aplicativo posiciona os blocos com layout automático. O agente não envia coordenadas, só os blocos e as relações. O aplicativo deduz os agrupamentos de `contém` e a ordem de `precede` (ADR-0021). Coordenadas vindas do modelo custam tokens e mudam a cada revalidação.
