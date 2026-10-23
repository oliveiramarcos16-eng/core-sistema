# core-sistema

RPG tático por turnos em um mundo pós-apocalíptico onde a tecnologia se perdeu e
virou relíquia.

## Documentação

| Documento | Conteúdo |
|---|---|
| [docs/GAME_DESIGN.md](docs/GAME_DESIGN.md) | Conceito, regras, mundo e MVP |
| [docs/TUTORIAL.md](docs/TUTORIAL.md) | Como usar OpenCode no VS Code e publicar no GitHub |
| [docs/fracasso.md](docs/fracasso.md) | Registro da sessão inicial de trabalho |

## Estrutura do repositório

```
.
├── README.md               # este arquivo
├── docs/                   # documentação e design
│   ├── GAME_DESIGN.md
│   ├── TUTORIAL.md
│   └── fracasso.md
├── .opencode/
│   └── commands/
│       └── publicar.md     # /publicar — commit + push
└── .gitignore
```

O código do jogo ainda não existe. Quando começar, a estrutura planejada é:

```
├── src/                    # código do jogo
│   ├── game/               # regras (combate, dados, turnos)
│   ├── ui/                 # interface
│   └── main.ts
├── assets/                 # pixel art, sons, fontes
└── tests/
```

## Status

Fase de design. Próximo passo: protótipo de **um combate jogável**.

## Regras em uma olhada

- Turnos táticos com **4 ações compartilhadas** pelo esquadrão.
- **Cronômetro** por turno: estourou, perde o turno.
- **D20** para acertar + rolagem de dano.
- Dupla fixa de irmãos veteranos + 1 recruta customizável.
- **Relíquias** com cargas limitadas.

## Como publicar

No OpenCode, rode:

```text
/publicar
```

Ou siga o [tutorial](docs/TUTORIAL.md).
