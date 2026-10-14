# MOTOR_DELAY - Atraso de partida com TON (Semana 7 / P07)

Disciplina: Sistemas Automatizados (UC) - Prof. Calvetti
Tema: Programação sequencial de CLP - temporização com TON

## Objetivo

Implementar em Ladder uma saída com atraso de partida: ao acionar `START`, o motor (`MOTOR_DELAY`) liga somente depois de 3 s. Se `STOP_OK` ou `EMG_OK` forem para 0, o temporizador zera e a saída desliga.

## Ambiente

- Simulador: rungs.dev Studio 1.5.5 (PLC Simulator Online, no navegador)
- Linguagem: Ladder (LD), dentro de uma AOI chamada `MOTOR_DELAY`
- Link do projeto: _colar aqui o link de compartilhamento (botão Share) e testar o acesso_

## Tags

| Tag | Tipo | Uso | Valor inicial | Descrição |
|---|---|---|---|---|
| START | BOOL | Local | 0 | Botão de partida |
| STOP_OK | BOOL | Local | 1 | Permissivo de parada (1 = normal, 0 = parada acionada) |
| EMG_OK | BOOL | Local | 1 | Permissivo de emergência (1 = normal, 0 = emergência) |
| MOTOR_DELAY | BOOL | Local | 0 | Saída do motor com atraso |
| T1 | TIMER | Local | PRE = 3000 | Temporizador TON (3 s) |

Campos do temporizador no simulador e correspondência com o slide:

| Slide | Simulador |
|---|---|
| IN | Entrada do TON (rung 0) |
| PT | T1.PRE (3000 ms) |
| ET | T1.ACC |
| Q | T1.DN |

## Lógica

```
Rung 0:  EMG_OK --] [-- STOP_OK --] [-- START --] [--[ TON T1, PRE = 3000 ms ]
Rung 1:  T1.DN --] [--------------------------------( MOTOR_DELAY )
```

- Rung 0: os três contatos normalmente abertos em série habilitam o TON. Se `EMG_OK` ou `STOP_OK` forem para 0, ou se `START` for solto, a continuidade some, `T1.ACC` volta a 0 e `T1.DN` vai a 0.
- Rung 1: a saída segue o bit de conclusão do timer (`T1.DN`).

## Decisões e prioridades

- `STOP_OK` e `EMG_OK` seguem a lógica de segurança (contato fechado = 1 = condição normal). Valor 0 significa parada ou emergência acionada.
- Os permissivos de segurança estão em série com `START`, então dominam a partida: qualquer um deles em 0 impede o timer e remove a saída.
- Não há selo (retenção) do `START`: o botão precisa ficar acionado durante os 3 s. Soltar `START` antes de PT cancela a temporização.
- Não há religamento automático ao liberar uma falha: a saída só volta se `START` estiver em 1 quando os permissivos voltarem. Por isso o T05 é feito com `START = 0`.
- Preset em milissegundos: 3 s = 3000.

## Testes

Procedimento: simulação em execução (Start), valores alterados no I/O Panel, com `START = 0`, `STOP_OK = 1` e `EMG_OK = 1` entre um teste e outro.

| ID | Ação | Esperado | Obtido | Situação |
|---|---|---|---|---|
| T01 | START por 2 s | Q=0 e MOTOR_DELAY=0 | ACC=0, DN=0, saída=0 | Aprovado |
| T02 | START por 3 s | Q=1 e MOTOR_DELAY=1 | ACC=3000, DN=1, saída=1 | Aprovado |
| T03 | STOP_OK=0 durante ET | ET zera e saída=0 | ACC=0, DN=0, saída=0 | Aprovado |
| T04 | EMG_OK=0 com saída ligada | saída=0 | ACC=0, DN=0, saída=0 | Aprovado |
| T05 | Liberar falha sem START | saída permanece 0 | saída=0 | Aprovado |

Os resultados completos estão em `casos_teste.csv`.

## Capturas de tela

- `T01_START_solto.png`
- `T02_START_3s.png`
- `T03_STOP_OK_0.png`
- `T04_EMG_OK_0.png`
- `T05_liberar_falha.png`

## Limitações

- A captura do T01 mostra o estado depois de soltar `START` (ACC=0, DN=0, saída=0). Ela não mostra o ACC subindo antes da interrupção, nem o tempo exato em que `START` ficou ligado.
- A captura do T03 também mostra apenas o estado final (timer zerado com `STOP_OK = 0` e `START = 1`).
- A captura do T02 mostra o valor final (ACC=3000), não a contagem em andamento.
- A simulação demonstra o comportamento lógico dentro do ambiente utilizado (rungs.dev Studio).

## Referências

- IEC 61131-3
- Documentação do rungs.dev
- Slides da Semana 7 (T07 / P07) - Prof. Calvetti
