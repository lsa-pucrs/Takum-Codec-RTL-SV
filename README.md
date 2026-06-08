# Takum Codec RTL — SystemVerilog + VHDL

Repositório de referência: a conversão **SystemVerilog** do codec Takum lado a
lado com a implementação **VHDL original**, ambas como submódulos.

## Submódulos

| Submódulo | Diretório | Descrição |
|-----------|-----------|-----------|
| [Takum-Codec-RTL](https://github.com/takum-arithmetic/Takum-Codec-RTL) | `vhdl/` | Implementação original em VHDL (Laslo Hunhold, ISC, arXiv 2408.10594) |
| [Takum-SystemVerilog](https://github.com/lsa-pucrs/Takum-SystemVerilog) | `sv/` | Conversão para SystemVerilog, validada contra um oráculo independente |

## Clone com submódulos

```bash
git clone --recurse-submodules https://github.com/lsa-pucrs/Takum-Codec-RTL-SV.git
```

Se já clonou sem `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

## Estado da conversão SystemVerilog

A conversão SV foi **regenerada a partir do VHDL original** e corrigida. Dois
defeitos de conversão (presentes na primeira tentativa) foram corrigidos e cada
um tem um teste que falha se o bug reaparecer (verificado por controle
negativo):

1. **Predecoder, N < 12** — o segmento regime+característica precisa alinhar os
   bits do takum no topo (MSB) e preencher zeros nos LSBs, como no VHDL
   (`takum(n-3 downto 0) & (11-n downto 0 => '0')`). A conversão antiga
   preenchia o lado errado, zerando `regime_bits` e corrompendo quase todo
   decode em N=8.
2. **Postencoder, N ≥ 12** — o guarda de underflow deve comparar com `-255`,
   não com `9'sd255` (= +255, inalcançável). O bug deixava os valores de menor
   magnitude sem correção (positivo → `0x0000`, negativo → `0x8000`/NaR).

### Verificação

Toda a verificação roda contra um oráculo Python independente derivado da
especificação (não portado do RTL), com um segundo oráculo de codificação
baseado em valor para triangulação:

```bash
cd sv/
bash simulation/run_all.sh          # self-test + gates exaustivos N ∈ {8,12,16}
```

Cobertura: predecoder `OUTPUT_EXPONENT` 0 e 1, postencoder roundtrip +
fronteira de saturação + varredura completa `(sinal, característica, mantissa)`,
e os quatro wrappers de ponta a ponta. Detalhes — incluindo duas propriedades
do VHDL original reproduzidas fielmente (flush subnormal; o par linear não é
inverso para entradas positivas) — estão no `README.md` do submódulo `sv/`.

## Licença

ISC — mesma licença do [original](https://github.com/takum-arithmetic/Takum-Codec-RTL).
