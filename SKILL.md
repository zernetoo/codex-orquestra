---
name: orquestra
description: "Orquestre uma demanda com Astra planejando e revisando e Luna implementando e testando. Use somente quando o usuário invocar /orquestra ou $orquestra."
---

# Orquestra Astra-Luna

Trate o conteúdo após a invocação como a demanda integral. O modelo selecionado na conversa é apenas o despachante; planejamento, decisões e aprovação pertencem ao GPT-6 Astra, enquanto edição e testes pertencem ao GPT-5.6 Luna.

## Contrato de execução

- Exija ferramentas de subagentes e disponibilidade de `gpt-6-astra` e `gpt-5.6-luna`. Se algum requisito faltar, pare como bloqueado e diga exatamente o que está indisponível.
- Ao criar um subagente com modelo explícito, use um fork sem histórico ou com histórico limitado e envie um prompt autocontido. Forks de histórico completo podem herdar o modelo do pai e rejeitar a substituição.
- Preserve as instruções, permissões e limites do projeto. A skill não amplia a autoridade concedida pelo usuário.
- Mantenha um único executor Luna ativo e uma única etapa em execução. Escritas paralelas são incompatíveis com este fluxo.
- O despachante transfere mensagens e acompanha o estado. Ele não substitui o julgamento do Astra nem aprova a entrega.

## Inicialização

1. Preserve a demanda do usuário, o contexto relevante da conversa e as instruções do projeto.
2. Quando o agente raiz for explicitamente identificado como `gpt-6-astra`, ele pode assumir o papel de coordenador. Caso contrário, ou quando houver dúvida, crie um subagente `astra_orquestrador` com `gpt-6-astra`, esforço `high`, e envie a demanda e o contexto preservados.
3. Instrua o Astra a atuar somente como planejador e revisor: compreender o projeto, definir critérios verificáveis, decidir a próxima ação e aprovar ou rejeitar evidências. O Astra pode fazer leituras dirigidas, mas delega exploração volumosa ao Luna como uma etapa de descoberta.
4. O Astra deve responder usando um dos estados abaixo e sempre fornecer o bloco necessário para a próxima ação.

```text
ORQUESTRA_STATUS: DELEGATE | COMPLETE | BLOCKED
PLAN_VERSION: <identificador>
CURRENT_STAGE: <etapa>
DELEGATION: <instrução autocontida para Luna, quando DELEGATE>
ACCEPTANCE: <critérios verificáveis da etapa>
DECISION: <motivo da decisão>
FINAL_EVIDENCE: <síntese, quando COMPLETE>
BLOCKER: <impedimento e informação necessária, quando BLOCKED>
```

## Laço sequencial

1. Quando o Astra retornar `DELEGATE`, crie ou reutilize um subagente `luna_builder` com `gpt-5.6-luna` e esforço `medium`. Encaminhe `DELEGATION` e `ACCEPTANCE` sem ampliar o escopo.
2. Instrua Luna a executar somente essa delegação, respeitar as instruções do repositório, preservar mudanças alheias e rodar as verificações relevantes. Luna não escolhe a próxima etapa nem aprova a própria entrega.
3. Exija de Luna este retorno:

```text
EXECUTION_STATUS: DONE | BLOCKED
RESULT: <resultado da etapa>
CHANGES: <arquivos e decisões relevantes>
VERIFICATION: <comandos e resultados>
RISKS: <lacunas restantes>
BLOCKER: <informação necessária, quando bloqueado>
```

4. Aguarde Luna terminar e encaminhe a resposta completa ao mesmo Astra. O Astra confronta mudanças reais e evidências com os critérios. Ele pode fazer uma verificação independente curta e então responde novamente no protocolo `ORQUESTRA_STATUS`.
5. Para `DELEGATE`, envie a correção, descoberta ou próxima etapa ao mesmo Luna e repita. Para `COMPLETE`, encerre. Para `BLOCKED`, peça ao usuário somente a decisão, credencial ou autoridade necessária.

## Critério de término

Continue até o Astra retornar `COMPLETE` com todos os critérios globais atendidos. Se a mesma falha ocorrer em três ciclos sem evidência nova, encaminhe-a ao Astra para classificação final como `BLOCKED`; nunca converta bloqueio em sucesso.

Na resposta final, apresente o plano cumprido, o que mudou, as verificações executadas e os riscos residuais aprovados pelo Astra. Não exponha o diálogo interno nem alegue que o modelo selecionado na conversa foi trocado.
