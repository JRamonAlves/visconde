---
status: aceita
---

# Snapshot persistente, revalidado a partir do diff do Git

O snapshot guarda a versão do código analisada. Quando o código muda, o aplicativo marca o snapshot como desatualizado e oferece a revalidação. O diff entre a versão analisada e a atual define onde a revalidação começa, e o agente segue para os consumidores das partes alteradas, mesmo que não tenham mudado. Revalidar só as linhas do diff deixaria passar consequências.

A revalidação só começa por ação do usuário, para que um `git pull` não dispare trabalho do agente sozinho.

## Presumido

- Mudanças locais ainda sem commit também entram na comparação.
