# ChargeGrid Intelligence - GoodWe EV Challenge (Sprint 3)

Chatbot de apoio ao **Operador Comercial** de uma rede de eletropostos, desenvolvido para o EV Challenge da GoodWe. Na Sprint 3 o núcleo conversacional das Sprints 1 e 2 foi refatorado para o **OpenAI Agents SDK**, com memória por sessão, guardrails de segurança, comparação entre modelos de linguagem e avaliação sistemática.

## Evolução do projeto

| | Sprints 1 e 2 | Sprint 3 |
|---|---|---|
| Orquestração | Loop manual sobre `chat.completions`, com `tool_calls` tratados à mão | `Agent` + `Runner` do OpenAI Agents SDK |
| Ferramenta de cálculo | Função e JSON schema escritos à mão | `@function_tool` (schema gerado pela assinatura) |
| Memória | Lista em RAM, resetada manualmente | `SQLiteSession` por `session_id`, persistida em `.db` |
| Segurança | Regras no system prompt | Guardrail de entrada (escopo e injeção) e de saída (conselho profissional) |
| Modelo | `gpt-4o-mini` fixo | Escolhido após comparar modelos com a mesma bateria de testes |
| Avaliação | 5 perguntas, checklist manual | 12 testes com critérios automáticos, tokens e latência por turno |

Repositórios anteriores: [Sprint 1](https://github.com/23diegob/GoodWe-EV-Challenge-Sprint1) e [Sprint 2](https://github.com/CrisRick367/GoodWe-EV-Challenge-Sprint2).

## Arquitetura

- **Agente principal** (`ChargeGrid Intelligence`): system prompt com as funcionalidades de negócio (Smart Surge Pricing, tickets de manutenção, relatórios cruzados, taxa de ociosidade, Peak Shaving e prioridade de frota), limites profissionais e regra de não inventar especificações técnicas.
- **Ferramenta** `calcular_expressao`: cálculos de faturamento e potência.
- **Guardrail de entrada:** um agente verificador com saída estruturada bloqueia pedidos fora do escopo e tentativas de Prompt Injection.
- **Guardrail de saída:** um segundo verificador bloqueia respostas que ajam como advogado, consultor financeiro ou eletricista, devolvendo a orientação de procurar um profissional habilitado.
- **Memória:** `SQLiteSession`, uma sessão por conversa, gravada em `chargegrid_sessions.db`.
- **Modelos:** o agente principal é configurável por `ModelSettings` (`temperature`, `top_p`, `max_tokens`). Os guardrails usam `gpt-4o-mini` em todas as configurações.

## Estrutura do repositório

```
.
├── README.md
├── .gitignore
├── .env.example
├── requirements.txt
├── integrantes.txt
├── sprint_3_prompt_ia.ipynb
├── casos_de_teste.md
├── relatorio_modelos.md
├── resultados_modelos.json
└── relatorio_evolucao.pdf
```

## Como executar

O notebook usa Secrets do Colab quando roda no Colab e o arquivo `.env` quando roda localmente.

### No Google Colab

1. Envie `sprint_3_prompt_ia.ipynb` para o Colab.
2. No ícone de chave (Secrets), crie `OPENAI_API_KEY` com a chave e ative **Acesso ao notebook**. Cole a chave sem espaços nem quebra de linha no final. 
3. Execute as células em ordem. A célula do menu interativo pede uma opção; digite `2` para seguir.
4. Os arquivos `relatorio_modelos.md`, `resultados_modelos.json`, são gerados no final e baixados automaticamente.

### Localmente

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
jupyter lab
```

## Comparação entre modelos

O notebook executa a mesma bateria em dois modelos e registra tudo em `relatorio_modelos.md`. O segundo modelo é na verdade o mesmo modelo com especificações diferentes.

## Testes

Os casos de teste estão documentados em [`casos_de_teste.md`](casos_de_teste.md): 5 funcionais, 1 de memória (3 turnos) e 6 de segurança, incluindo Prompt Injection, escopo, especificações inventadas e aconselhamento jurídico, financeiro e de segurança elétrica. Cada teste tem critérios automáticos e uma verificação manual.

## Decisões técnicas e problemas encontrados

- **Event loop:** `Runner.run_sync()` se recusa a rodar dentro de um event loop ativo, o que ocorre em células do Colab e nas chamadas aninhadas dos guardrails. A função `rodar()` usa `Runner.run` com `nest_asyncio` para resolver isso sem mudar o restante do código.
- **Acesso a modelos:** o projeto da OpenAI usado no desenvolvimento libera apenas o `gpt-4o-mini`.
- **Chave com espaço ou quebra de linha:** a biblioteca reporta apenas `Connection error`, sem apontar a chave como causa. O notebook remove espaços da chave e diagnostica o problema antes de rodar a bateria, para não gerar relatórios feitos só de erros.
- **Falsos positivos dos guardrails:** os primeiros testes mostraram o guardrail de saída bloqueando a resposta de Peak Shaving e o de entrada recusando perguntas jurídicas e financeiras como fora de escopo. As instruções dos verificadores foram ajustadas.

## Equipe

| Nome | RM | Responsabilidade |
|---|---|---|
| Lucca Bertolini | 569552 | Comparação entre ≥2 modelos + relatorio_modelos.md |
| Diego de Oliveira Brandão | 569773 | Empacotamento final: .env/.gitignore, integrantes.txt, organização do repo |
| Raphaello Caffettani | 572334 | Casos de teste de segurança (prompt injection, escopo, jurídico/financeiro/elétrico) |
| Cristhian Henrique Clementino | 574117 | Setup do projeto, refatoração do núcleo com o framework de agentes escolhido + memória por sessão |
| Fabio Pena Vieira | 570441 | Relatório de evolução em PDF (máx. 5 páginas) |
