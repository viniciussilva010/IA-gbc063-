# Evacuação em Incêndio Florestal com Drones

Projeto desenvolvido para a disciplina de **Inteligência Artificial** — ambiente visual Top-Down 2D em **Pygame** (`650 x 420` px, grade discreta `20 x 13` células de `32x32` px) inspirado na estética de *Stardew Valley*.

![Preview do Ambiente]("https://github.com/viniciussilva010/IA-gbc063-/blob/main/Fuga-do-Fogo-_650x420_-2026-10-28-21-19-34.gif?raw=true")

---

## Sumário do Problema (Desafio 1)

Um agente sobrevivente inicia no **Ponto de Spawn** (canto inferior esquerdo) acompanhado por **3 drones batedores** e precisa alcançar a **Zona Segura** (canto superior direito) enquanto **focos de incêndio se propagam dinamicamente** pelo mapa a partir de focos iniciais, desviando de obstáculos intransponíveis (lagos, rochas e bases de troncos de árvores).

---

## 1. Ambiente e Estado Markoviano ($S_t$)

O ambiente é modelado como um **Processo de Decisão de Markov (MDP)** sob uma grade discreta $G = 20 \times 13$ tiles (`32x32` pixels cada):

- **Propriedade Markoviana**: O estado atual $s_t \in \mathcal{S}$ concentra toda a informação necessária do ambiente no instante $t$, tornando o futuro condicionalmente independente do passado dado $s_t$ e $a_t$:
  $$P(S_{t+1} = s' \mid S_t = s_t, A_t = a_t, S_{t-1}, \dots, S_0) = P(S_{t+1} = s' \mid S_t = s_t, A_t = a_t)$$

### Informações disponíveis para o Agente no Estado $s_t$:
1. **Posição do Agente**: Coordenada discreta $p_t = (x_t, y_t) \in \{0,\dots,19\} \times \{0,\dots,12\}$.
2. **Posição da Zona Segura (Objetivo)**: Coordenada alvo $g = (x_{\text{meta}}, y_{\text{meta}})$ no canto superior direito e vetor distância $(\Delta x, \Delta y)$.
3. **Mapa/Matriz de Ocupação e Perigos (observado pelo agente + 3 drones batedores)**:
   - Cada célula $(i, j)$ possui uma codificação de estado:
     - `0` = **Livre** (grama, piso pavimentado ou copa de árvore — permite passagem por trás da copa).
     - `1` = **Obstáculo Estático** (lago, rocha ou base `1x1` do tronco de árvore — bloqueia passagem).
     - `2` = **Zona em Chamas Ativa** (foco de incêndio — estado terminal de falha/dano severo).
     - `3` = **Zona de Risco Iminente** (células adjacentes aos focos de incêndio mapeadas pelos 3 drones batedores, onde o fogo pode surgir no próximo ciclo).
     - `4` = **Zona Segura** (estado terminal de sucesso).

---

## 2. Espaço de Ações do Agente ($\mathcal{A}$)

Em cada passo de decisão discreto $t$, o agente escolhe uma ação $a_t \in \mathcal{A}$ com $|\mathcal{A}| = 5$:

| Código | Ação | Deslocamento $(\Delta x, \Delta y)$ | Descrição |
| :---: | :--- | :---: | :--- |
| $a_0$ | **Norte (Cima)** | $(0, -1)$ | Move 1 tile para cima (`y - 1`) |
| $a_1$ | **Sul (Baixo)** | $(0, +1)$ | Move 1 tile para baixo (`y + 1`) |
| $a_2$ | **Oeste (Esquerda)** | $(-1, 0)$ | Move 1 tile para a esquerda (`x - 1`) |
| $a_3$ | **Leste (Direita)** | $(+1, 0)$ | Move 1 tile para a direita (`x + 1`) |
| $a_4$ | **Aguardar (Wait)** | $(0, 0)$ | Permanece no tile atual aguardando atualização dos drones/fogo |

> **Regra de transição em obstáculos**: Caso a ação tente mover o agente para fora dos limites da janela (`650x420`) ou para um obstáculo intransponível (lago, rocha ou base de tronco), o agente permanece na célula atual $p_{t+1} = p_t$ e recebe penalidade de colisão. A copa das árvores (`Y-sorting`) permite passagem livre por trás.

---

## 3. Função de Recompensa ($R(s_t, a_t, s_{t+1})$)

A função de recompensa guia o agente a encontrar a rota mais segura e eficiente até o canto superior direito antes que os focos de incêndio fechem os corredores:

| Evento / Transição | Recompensa ($R$) | Justificativa no Aprendizado |
| :--- | :---: | :--- |
| **Alcançar a Zona Segura** (Estado Terminal) | **$+100.0$** | Objetivo principal concluído com sucesso |
| **Entrar ou ser atingido por Fogo** (Estado Terminal) | **$-100.0$** | Penalidade máxima — evita zonas em chamas a todo custo |
| **Aproximar-se de célula adjacente ao fogo** | **$-10.0$** | Penalidade de risco térmico (alerta dos 3 drones batedores) |
| **Tentar atravessar Lago / Rocha / Bordas** | **$-5.0$** | Desencoraja ações inválidas contra obstáculos intransponíveis |
| **Passo de movimentação normal (custo de tempo)** | **$-1.0$** | Incentiva a menor rota devido à propagação dinâmica do fogo |

---

## Como Executar o Projeto

```powershell
# 1. Criar ambiente virtual (caso ainda não exista) e instalar dependências
python -m venv .venv
.\.venv\Scripts\pip install -r requirements.txt

# 2. Executar o ambiente visual
.\.venv\Scripts\python.exe ambiente_stardew.py
```

- Pressione **`R`** com a janela aberta para gerar um novo mapa proceduralmente.
- Pressione **`ESC`** para encerrar.

