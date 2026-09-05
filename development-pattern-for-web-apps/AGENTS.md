# Instruções para agentes de IA — este repositório

Este repositório **é** a skill *Development Pattern for Web Apps*. Ele não contém código de
aplicação: só Markdown, templates e dois scripts de shell.

## Antes de editar qualquer coisa aqui

Leia [`skills/development-pattern-for-web-apps/SKILL.md`](skills/development-pattern-for-web-apps/SKILL.md).
As quatorze regras que ele define valem também para este repositório.

## Regras da casa

- **R1 — nenhum segredo, nenhum dado real.** Todo exemplo usa placeholder obviamente falso e
  obviamente acionável — `SEU-USUARIO`, `SUA-SENHA`, `seu-dominio.com.br`, `usuario@example.com`.
  Nunca um valor que pareça funcionar: o leitor precisa ser incapaz de confundi-lo com algo real.
  Rode os scanners antes de todo commit:

  ```bash
  bash skills/development-pattern-for-web-apps/assets/scripts/scan-secrets.sh
  git diff --cached | bash skills/development-pattern-for-web-apps/assets/scripts/scan-secrets.sh --stdin
  bash skills/development-pattern-for-web-apps/assets/scripts/scan-sql-injection.sh
  ```

- **R4 — nada inventado, especialmente aqui.** A credibilidade desta skill depende de cada
  afirmação ser verificável. Antes de mudar uma função, uma constante do PDO, uma classe do
  Tailwind ou uma opção de configuração em qualquer referência, confirme na fonte e anote a
  versão conferida:

  ```bash
  php -v && php -m
  php -r 'var_dump(function_exists("str_contains"));'
  npm view next version && npm view tailwindcss version
  ```

  Inventários são **fotografias, não contratos**. Mantenha a linha "verificado em <versão>,
  <data>" ao lado de cada um.

- **R5 — o exemplo errado é sempre rotulado.** Todo trecho vulnerável neste repositório vem
  precedido de `❌` e acompanhado da forma correta com `✅`. Nunca deixe um exemplo inseguro
  sem rótulo: alguém vai copiá-lo.

- **R13 — README e página andam juntos.** `README.md` e `docs/index.html` afirmam as mesmas
  coisas em profundidades diferentes. Quando um muda, o outro muda no mesmo commit — mesmo
  aviso, mesma contagem de regras, mesmos números. Acrescentar ou renumerar uma regra exige
  editar **os dois**, mais as estatísticas da página, mais o `CHANGELOG.md`, mais a `version`
  em `.claude-plugin/plugin.json`.

- **Português (pt-BR) em todo o conteúdo.** Nomes de arquivo em minúsculas com hífen. Os
  identificadores dos templates (`RF-001`, `CT-SEC-009`, `R5`) são estáveis: renomear um
  quebra referências cruzadas espalhadas por vários documentos.

## Layout do repositório

```
skills/development-pattern-for-web-apps/
├── SKILL.md              # as 14 regras, o pipeline de 8 fases, o roteamento
├── references/           # 14 documentos, carregados sob demanda
└── assets/
    ├── templates/        # 18 arquivos prontos para copiar em um projeto
    └── scripts/          # scan-secrets.sh, scan-sql-injection.sh
docs/                     # fonte da página do GitHub Pages
specs/                    # a spec desta própria skill (R3 aplicada a ela mesma)
.github/workflows/        # publica docs/ no ramo gh-pages, gerado
.claude-plugin/           # manifestos de plugin e marketplace do Claude Code
```

`gh-pages` é **gerado e sobrescrito** pelo CI. Nunca edite aquele ramo — edite `docs/`.

## Convenções de escrita

- Cada referência abre com a regra que ela impõe e por que a falha importa, depois dá
  comandos, depois dá o checklist. Concreto antes de abstrato.
- Toda afirmação absoluta carrega a fonte: caminho de arquivo, URL ou versão de pacote.
- Templates usam marcadores `<PLACEHOLDER>` ou `SEU-`/`SUA-`, de modo que um campo não
  preenchido salte aos olhos.
- Tabelas antes de listas quando há mais de duas dimensões.

## Verificando uma mudança

Não há build. Antes do push:

```bash
# scanners limpos
bash skills/development-pattern-for-web-apps/assets/scripts/scan-secrets.sh
bash skills/development-pattern-for-web-apps/assets/scripts/scan-sql-injection.sh

# sintaxe dos scripts e dos templates executáveis
bash -n skills/development-pattern-for-web-apps/assets/scripts/scan-secrets.sh
bash -n skills/development-pattern-for-web-apps/assets/scripts/scan-sql-injection.sh
php  -l skills/development-pattern-for-web-apps/assets/templates/install-wizard.php.template
php  -l skills/development-pattern-for-web-apps/assets/templates/updater.php.template
php  -l skills/development-pattern-for-web-apps/assets/templates/Database.php.template

# arquivos estruturados
python3 -c "import yaml;yaml.safe_load(open('.github/workflows/pages.yml'))"
python3 -c "import json;json.load(open('.claude-plugin/plugin.json'))"
python3 -c "import json;json.load(open('skills/development-pattern-for-web-apps/assets/templates/update-manifest.json.template'))"

# abra docs/index.html no navegador e confira o layout em 1440px e 375px
```

Conferência de consistência — estes números aparecem em vários lugares e desalinham fácil:

```bash
grep -c '^| \*\*R' skills/development-pattern-for-web-apps/SKILL.md    # 14 regras
ls skills/development-pattern-for-web-apps/references/*.md | wc -l      # 14 referências
ls skills/development-pattern-for-web-apps/assets/templates/* | wc -l   # 18 templates
grep -rn "quatorze\|catorze\|treze" README.md AGENTS.md docs/index.html skills/**/SKILL.md
```
