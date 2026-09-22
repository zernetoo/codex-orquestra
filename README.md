# Orquestra

Skill standalone do Codex para executar demandas no ciclo Astra → Luna → Astra:

- GPT-6 Astra entende a demanda, planeja, define critérios e revisa cada entrega.
- GPT-5.6 Luna implementa uma etapa por vez, executa verificações e devolve evidências.
- O ciclo termina somente quando Astra aprova todos os critérios ou declara um bloqueio real.

O modelo selecionado na conversa funciona apenas como despachante. Quando ele não é Astra, a skill cria um subagente `gpt-6-astra` com esforço `high`; a execução é delegada a `gpt-5.6-luna` com esforço `medium`.

## Requisitos

- Uma versão atual do Codex com subagentes habilitados.
- Acesso aos modelos `gpt-6-astra` e `gpt-5.6-luna`.
- Limite de pelo menos um subagente ativo por vez.

Não usa Router, API externa ou chave de API. A economia vem de concentrar o volume operacional no Luna; como todo fluxo multiagente, ele pode consumir mais tokens totais do que uma execução simples.

## Publicar no GitHub

Crie um repositório vazio e envie o conteúdo desta pasta para a branch `main`.

```bash
git init
git add .
git commit -m "feat: add Orquestra Astra-Luna skill"
git branch -M main
git remote add origin https://github.com/zernetoo/codex-orquestra.git
git push -u origin main
```

## Instalar globalmente a partir do GitHub

No Codex, peça ao instalador de skills:

```text
$skill-installer instale a skill da raiz de https://github.com/zernetoo/codex-orquestra
```

Ela ficará disponível para todos os seus projetos. Abra uma conversa nova após a instalação.

## Instalar somente em um projeto

Na raiz do projeto, adicione o repositório como submódulo:

```bash
mkdir -p .agents/skills
git submodule add https://github.com/zernetoo/codex-orquestra.git .agents/skills/orquestra
```

O Codex detecta skills em `.agents/skills` a partir da conversa seguinte.

## Usar

No compositor, digite `/`, procure **Orquestra Astra → Luna** e selecione a skill. Em qualquer cliente Codex, a invocação explícita canônica é:

```text
$orquestra implemente autenticação por passkey e cubra os fluxos críticos com testes
```

O atalho visual pode ser encontrado digitando `/orquestra` quando o cliente lista skills entre os comandos. Se o cliente não completar esse nome diretamente, use `$orquestra`, que é a sintaxe portátil.

## Estrutura

```text
SKILL.md
agents/openai.yaml
```

O gatilho foi deliberadamente limitado a `/orquestra` e `$orquestra`, para que tarefas comuns não ativem o fluxo por acidente.
