# Realismo do openttd-motor: carga perecível, combustível e oficina

Regras implementadas em cima do JGR Patchpack. Todas ficam em **Configurações → Veículos** e já vêm **ligadas** em jogo novo.
Saves antigos continuam abrindo (veículos começam com tanque cheio).

## 1. Carga perecível (`vehicle.cargo_spoilage_rate`)

- **O que é perecível:** toda carga com a classe *Refrigerated* (refrigerada) e as cargas com rótulo `LVST` (gado), `FOOD` (alimentos), `FRUT` (frutas), `FISH` (peixe) e `MILK` (leite). **Correio nunca estraga.**
- **Prazo:** cada carga já tem dois prazos de pagamento (`transit_periods[0]` e `transit_periods[1]`, os "days1/days2" do OpenTTD), contados em *períodos de envelhecimento* da carga (por padrão 185 ticks ≈ 2,5 dias; o mod do veículo pode mudar). O prazo de perda é a soma dos dois:
  - gado: 4 + 18 = 22 períodos (≈ 55 dias)
  - alimentos: 0 + 30 = 30 períodos (≈ 75 dias)
  - frutas: 0 + 15 = 15 períodos (≈ 37 dias)
- **Regra:** a cada período de envelhecimento em que um lote de carga dentro do veículo já passou do prazo, ele perde `teto(quantidade × taxa / 100)` unidades. Lote que zera some.
- **Taxa:** configuração `vehicle.cargo_spoilage_rate`, de 0 a 20 % por período, padrão **2 %**. **0 desliga** a regra.
- Só estraga com o veículo **em viagem**: enquanto ele está carregando/descarregando na estação a carga não perde nada (mas continua envelhecendo, como no jogo original).
- A carga perdida some de verdade (o veículo fica mais leve e entrega menos). O pagamento por atraso continua igual ao do jogo original.

## 2. Combustível (`vehicle.fuel_consumption`, `vehicle.fuel_tank_days_road`, `vehicle.fuel_tank_days_rail`)

Vale para **trens e automóveis** (ônibus, caminhões, bondes). Navios e aviões continuam com o custo antigo.

### 2.1 Esforço do motor (a cada tick)

| Situação | Carga do motor |
|---|---|
| Acelerando, ou subindo ladeira (velocidade limitada só pela tração, mesmo perdendo velocidade) | **100 %** |
| Mantendo a velocidade no limite (cruzeiro), ou automóvel seguindo o veículo da frente numa fila andando | **35 %** |
| Parado com motor ligado (carregando, sinal fechado, fila), freando ou desacelerando para um limite menor | **5 %** (marcha lenta) |
| Parado pelo jogador, quebrado/acidentado ou guardado no depósito | 0 % |

O consumo do tick é `carga do motor × fator de consumo / 100`, medido em "centésimos de tick em potência total de um diesel".

### 2.2 Fonte de energia

| Fonte | Fator de consumo | Preço por unidade | Tanque? | Custo relativo |
|---|---|---|---|---|
| Vapor (carvão) | 170 % | 60 % | sim | 102 % (carvão barato, mas gasta muito — tanque esvazia 1,7× mais rápido) |
| Diesel (trem e automóvel) | 100 % | 100 % | sim | 100 % |
| Elétrico, monotrilho, maglev, bonde | 100 % | 85 % | **não** (energia vem da linha) | 85 % |

Num trem com várias locomotivas, quem decide a fonte é a locomotiva da frente.

### 2.3 Quanto custa

- O custo fixo de operação (tripulação, seguro…) cai para **50 %** do custo de operação original do veículo. É isso que aparece como "Custo de operação" na janela do veículo.
- O combustível é cobrado uma vez por dia do veículo, como **custo de operação** (mesma linha das finanças: "Trens — operação" / "Automóveis — operação") e sai do lucro do veículo:

  `custo do combustível = custo de operação original anual × unidades gastas × preço% × 200% / (365 × 74 × 100)`

  Ou seja: um ano inteiro em potência total, a diesel, custa **2×** o custo de operação original. Como o custo de operação original já vem do preço-base do jogo × fator do veículo, o combustível acompanha a **inflação** e os multiplicadores de custo dos NewGRFs.
- Calibração (medida no jogo): ônibus urbano (para-e-anda, muitas curvas) tem carga média de ~50 % → total (50 % fixo + combustível) ≈ **1,5×** o custo original; trem a vapor numa linha simples ficou em ~42 % de carga → ≈ **1,35×**; veículo de estrada que passa mais tempo em cruzeiro (~38 %) → ≈ **1,25×**. Fórmula do total: `0,5 + 2 × carga média × fator da fonte`.
- Nota: o pedido era "combustível ∝ potência". Como no jogo original o custo de operação já cresce com a potência (mas de forma muito irregular: de 6 a 42 hp por unidade de custo entre as locomotivas originais), usar a potência pura quebraria a calibração de 1,3–1,6×. Por isso o preço do combustível é ancorado no custo de operação original de cada veículo (e de cada locomotiva extra num trem).

### 2.4 Tanque e reabastecimento

- Tanque medido em **dias rodando em potência total**: automóveis `vehicle.fuel_tank_days_road` (padrão **40 dias**), trens `vehicle.fuel_tank_days_rail` (padrão **60 dias**), ambos de 1 a 120. Em cruzeiro (35 %) o tanque dura ~3× mais.
- O tanque esvazia com o mesmo consumo do item 2.1 (vapor 1,7× mais rápido).
- **Abaixo de 35 %**: o veículo passa a "precisar de revisão" e procura um depósito, mesmo com intervalo de revisão desligado ou com "sem revisão se quebras desligadas". Nessa busca ele aceita um depósito **4× mais fora do caminho** do que numa revisão normal (`maximum_go_to_depot_penalty × 4`).
- **Revisão no depósito = tanque cheio** (qualquer passagem pelo depósito que conte como revisão).
- **Tanque vazio**: o veículo não fica preso, só anda a no máximo **25 % da velocidade máxima** até reabastecer.
- Janela de detalhes do veículo mostra uma linha **"Combustível: 63%"** (laranja abaixo de 35 % "procurando depósito", vermelho "vazio").
- Save antigo carrega com tanque cheio e nada de combustível pendente.
- Calibragem (testado): com tanques de 8/12 dias o alcance ficou curto demais (um ônibus urbano esvaziava em ~10 dias e a maioria chegava ao depósito já vazia). Com 30/45 dias e reserva de 25 %, numa cidade congestionada ainda metade chegava vazia. Ficou assim: tanques de **40/60 dias** e reserva de **35 %**; um ônibus urbano roda uns dois meses entre abastecimentos.
- Para a IA (script) e na janela do veículo, "custo de operação" passa a ser só a parte fixa (50 %); a janela de compra continua mostrando o custo original cheio.

## 3. Oficina (`vehicle.service_maintenance_cost`)

- Toda vez que um veículo **entra no depósito** (e é revisado), paga a oficina:

  `custo = 0,5 % do valor atual × (1 + idade em anos / 10)` — somado para cada parte do veículo (locomotiva e vagões, cada um com sua idade e valor).

  Exemplo: ônibus valendo £10.000 com 5 anos → 0,5 % × 1,5 = **£75** por revisão.
- Vale para trens, automóveis, navios e aviões.
- Despesa lançada como **custo de operação** do tipo do veículo (não como "manutenção de propriedade", que no jogo é a infraestrutura) e descontada do lucro do veículo.

## Configurações (resumo)

| Nome | Padrão | Faixa | O que faz |
|---|---|---|---|
| `vehicle.cargo_spoilage_rate` | 2 | 0–20 (% por período; 0 = desligado) | Perda de carga perecível |
| `vehicle.fuel_consumption` | ligado | liga/desliga | Combustível por esforço + tanque + custo fixo pela metade |
| `vehicle.fuel_tank_days_road` | 40 | 1–120 dias | Tanque dos automóveis |
| `vehicle.fuel_tank_days_rail` | 60 | 1–120 dias | Tanque dos trens (vapor/diesel) |
| `vehicle.service_maintenance_cost` | ligado | liga/desliga | Custo da oficina na revisão |

## Para conferir no jogo (log)

Rodando com `-d misc=3` o jogo escreve no console:
- `Fuel: vehicle N (...) is low on fuel (34%), looking for a depot`
- `Fuel: vehicle N (...) refuelled in depot at X% ...`
- `Fuel: vehicle N (...) ran out of fuel, speed limited to 25%`
- `Maintenance: vehicle N (...) serviced in depot, workshop cost X`
- `Spoilage: vehicle N (...) lost X of LVST ...`

Com `-d misc=4` aparece também a cobrança diária de combustível de cada veículo.
