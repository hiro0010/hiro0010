# Yuna Desktop

Base arquitetural da Yuna Desktop. A **Etapa 1** entregou os contratos de
modelo e permissões. A **Etapa 2A** adicionou estado local versionado,
checkpoints recuperáveis, aprovação vinculada a parâmetros e execução
exclusivamente simulada. A **Etapa 2C.1** adiciona um shell visual isolado em
PySide6/Qt Quick, ainda sem qualquer conexão com o Core. A **Etapa 2C.4.2**
adiciona um transporte HTTP isolado e testável para o OpenCode, bloqueado por
padrão até que o isolamento de ferramentas seja validado contra o CLI real.

A aplicação completa e a integração UI ↔ Core **não existem**.

## Escopo atual

- Catálogo de modelos com política **FREE_ONLY**.
- Política de permissões com **negação por padrão**.
- Aprovação humana vinculada à **ação e aos parâmetros**, com TTL, uso único e
  proteção contra replay.
- Store JSON atômico com checksum, geração, backup e quarentena.
- Contrato versionado de checkpoint (`yuna.checkpoint/v1`).
- Recuperação segura após interrupção, corrupção e schema inválido.
- Contratos de ferramentas e executor **somente simulado**.
- Diagnóstico local somente leitura.
- Shell visual **2C.1** com dados simulados, estados explícitos, cancelamento,
  layout responsivo (três colunas ou uma coluna com rolagem) e fronteira rígida
  com o Core.

## Fora de escopo

- Integração da UI com o Core, estados reais ou eventos reais (Etapa 2C.2).
- Voz, Gemini Live ou qualquer integração de áudio.
- Chamadas reais a modelos ou provedores pela UI.
- Execução real de ferramentas, shell, root ou operações privilegiadas.
- SQLite, migração automática de schema ou auditoria persistente.
- Distribuição do aplicativo ou decisão de licença final.

## Estrutura

```text
docs/adr/                    decisões arquiteturais
docs/contracts/              contratos versionados
docs/evidence/               evidências de validação local
scripts/doctor.py            diagnóstico somente leitura
src/yuna_desktop/
  model_router/catalog.py    catálogo FREE_ONLY
  permissions/policy.py      decisão de permissões
  permissions/approval.py    broker vinculado a ação + parâmetros
  state/store.py             store atômico e recuperação
  state/checkpoints.py       contrato de checkpoint v1
  execution/contracts.py     contratos de ferramentas
  execution/simulated.py     executor sem efeitos reais
  opencode/                  cliente, adaptador e transporte HTTP isolado
  ui/                        shell visual isolado (2C.1)
  ui/qml/                    componentes Qt Quick/QML
tests/                       suíte local; testes de UI exigem PySide6
```

## Executar

Testes do Core e da UI (a UI usa `offscreen` nos testes):

```bash
python3 -m unittest discover -s tests -t .
python3 scripts/doctor.py
```

O `doctor` não lê conteúdo de credenciais, não acessa a rede e não chama
modelos. Ele reporta apenas presença/ausência de arquivos e variáveis
sensíveis.

Shell visual simulado, sem instalar o pacote:

```bash
PYTHONPATH=src python3 -m yuna_desktop.ui
```

Em ambiente sem display, use `QT_QPA_PLATFORM=offscreen`. A janela abre com
dados de demonstração; nada é enviado ao Core. O botão **CANCELAR** retorna ao
estado `Idle`. O alvo de renderização é OpenGL; Vulkan não é exigido. Em janelas
estreitas (por exemplo, tileadas pelo i3), os painéis passam para uma coluna com
rolagem vertical e o seletor de estados vai para uma segunda linha.

## Segurança

- Modelos `PAID` e `UNKNOWN` são inelegíveis.
- `FREE` exige fonte e data de verificação.
- Ações desconhecidas são negadas.
- Ações privilegiadas exigem token de uso único, com TTL e vínculo à ação **e**
  aos parâmetros.
- Parâmetros usam JSON canônico com tipos restritos; `float`, `NaN` e
  `Infinity` são recusados.
- Uma string como `"yes"` não é autorização.
- Aprovações não são persistidas; reiniciar invalida tokens pendentes.
- Estado corrompido é colocado em quarentena; não há reset silencioso.
- Schema desconhecido não é migrado automaticamente.
- Chaves sensíveis conhecidas são recusadas no estado persistido.
- O executor não abre processos, sockets ou arquivos de ferramentas.
- A UI 2C.1 não importa o Core, não usa rede, subprocessos, modelos ou
  ferramentas, e não lê credenciais.
- Todos os textos, mensagens, comandos e eventos da UI são explicitamente
  simulados; `coreConnected` permanece `false`.
- Nenhum caminho de root ou `sudo` faz parte desta etapa.

## Limitações conhecidas

- O lock do store usa `fcntl`, específico de Linux.
- A durabilidade depende do filesystem e do `fsync`.
- O diretório padrão de estado ainda não foi fixado; o store recebe um root
  explícito.
- A auditoria do executor é apenas em memória.
- O executor simulado não valida integração real com OpenCode ou ferramentas.
- A API real do `opencode serve` 1.18.35 ainda não foi validada.
- O transporte HTTP (`opencode/http_transport.py`) é opt-in e fica bloqueado por
  padrão (`HttpIntegrationBlocked`); falta validar que ferramentas permanecem
  desabilitadas no servidor real.
- A UI 2C.1 não está conectada ao Core e não prova estados reais.
- PySide6 é uma dependência opcional de UI; ela já está presente neste
  ambiente, mas não é instalada pelo projeto. Os testes de UI exigem PySide6.
- O alvo é OpenGL 4.6 com driver NVIDIA 470.256.02; Vulkan não foi verificado e
  não é requisito. Não há promessa de FPS.
- Não há teste de distribuição, empacotamento ou conformidade LGPL final.

## Licenciamento

A licença final **não foi decidida**. Consulte
[`LICENSE-DECISION.md`](LICENSE-DECISION.md) e
[`docs/adr/0005-reuse-and-license.md`](docs/adr/0005-reuse-and-license.md).
O shell visual usa PySide6 6.12.0 (LGPL-3.0/GPL) como dependência opcional.
Nenhum código do Mark-LV ou do Yuna-Orchestrator foi copiado.

## Projetos

- **netwave** — Marca principal: identidade, nomenclatura e visão dos 9 projetos.
- **axiom** — App de controle desktop: núcleo + módulos instaláveis.
- **netwave‑site** — Site oficial: vitrine, store, status e Netwave ID.
- **vitalis** — Resiliência digital: sistema imunológico da infraestrutura.
- **netwaveos** — SO Linux próprio sobre CachyOS/Arch.
- **axionengine** — Motor nativo C++20/N‑API: telemetria de hardware.
- **mochi‑ac** — Anti‑cheat Minecraft: detecção por simulação.
- **nyx‑client** — Mod PvP/performance Fabric: 38 módulos de otimização.
- **axiom‑bot** — Bot Discord: 64 comandos, RBAC e modlogs.

## Estado

- Etapa 1 — base arquitetural: concluída.
- Etapa 2A — persistência, checkpoints, autorização vinculada e execução
  simulada: implementada e testada localmente.
- Etapa 2B — cliente/adaptador OpenCode: presente e coberto por testes locais;
  integração real com `opencode serve` não validada.
- Etapa 2C.1 — shell visual isolado com dados simulados: implementada e
  testada localmente.
- Etapa 2C.2 — integração UI ↔ Core: **bloqueada** até autorização explícita.
- Etapa 2C.4.2 — transporte HTTP isolado com mocks: implementado e testado;
  integração real bloqueada por padrão. Ver `docs/adr/0011-opencode-http-transport.md`.

## Etapa 2C.4.5 – Resolução de Gates (27/10/2026)

- **Gate A – Bloqueado**: sem modelo FREE com `source` + `verified_at` no catálogo local; catálogo de produção vazio; `opencode/space-bunny-free` tem custo >0 e status deprecated.
- **Gate B – Bloqueado**: sem prova verificável de desabilitação de ferramentas. Código estático mostra que `{"*": false}` no payload de `tools` gera permissão `*` deny e o filtro `jd` remove todas as ferramentas; porém o transporte HTTP atual (`_validate_request`) rejeita qualquer `request.tools`. Uma correção mínima (adicionar `"tools": {"*": false}` ao JSON de mensagem) seria fundamentada pelo código estático, mas ainda depende da integração real do servidor; se não for aceita, o gate permanece bloqueado.

*(Detalhes completos estão no relatório estruturado gerado durante a etapa.)
