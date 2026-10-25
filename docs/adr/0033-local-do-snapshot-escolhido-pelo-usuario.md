---
status: aceita
---

# O usuário escolhe onde o snapshot fica guardado

Para cada projeto, o usuário escolhe se o snapshot fica em `.visconde/` dentro do repositório analisado, onde pode ser commitado e compartilhado, ou no diretório de dados local do Visconde. Os dois casos atendem usos diferentes, e o aplicativo não decide pelo usuário.

## Consequências

Se o snapshot ficar dentro do repositório, a detecção de mudanças e a revalidação ignoram `.visconde/`.
