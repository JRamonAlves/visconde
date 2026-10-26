---
status: aceita
---

# Stack: React, electron-vite, React Flow com ELK, Shiki e TypeScript compiler API

A interface usa React com TypeScript sobre electron-vite. O mapa é renderizado com React Flow e posicionado pelo ELK, porque o layout é responsabilidade do aplicativo (ADR-0032). O leitor de código usa Shiki, mais leve que o Monaco e suficiente para um leitor sem edição. O índice de símbolos e a validação de âncoras usam o TypeScript compiler API no processo principal.
