# Takum Codec RTL — SystemVerilog + VHDL

Repositório de referência com a implementação **SystemVerilog** do codec Takum, lado a lado com a implementação **VHDL original**.

## Submódulos

| Submódulo | Diretório | Descrição |
|-----------|-----------|-----------|
| [Takum-Codec-RTL](https://github.com/takum-arithmetic/Takum-Codec-RTL) | `vhdl/` | Implementação original em VHDL pelo autor (Laslo Hunhold) |
| [Takum-SystemVerilog](https://github.com/CCcassiusdjs/Takum-SystemVerilog) | `sv/` | Conversão para SystemVerilog, validada por simulação exaustiva |

## Clone com submódulos

```bash
git clone --recurse-submodules https://github.com/CCcassiusdjs/Takum-Codec-RTL-SV.git
```

Se já clonou sem `--recurse-submodules`:

```bash
git submodule init
git submodule update
```

## Simulação rápida (SystemVerilog)

Requer **Icarus Verilog** (`iverilog`) ou **Verilator** para lint.

### Predecoder — teste exaustivo (65536 valores para N=16)

```bash
cd sv/
iverilog -g2012 -o predecoder_tb \
  rtl/decoder/predecoder.sv \
  simulation/decoder/predecoder_tb.sv
./predecoder_tb
# Saída esperada: PASS: All 65536 values tested successfully.
```

### Postencoder — roundtrip test (decode → encode → compara)

```bash
iverilog -g2012 -o postencoder_tb \
  rtl/decoder/predecoder.sv \
  rtl/encoder/postencoder.sv \
  simulation/encoder/postencoder_tb.sv
./postencoder_tb
# Saída esperada: All 16 values roundtrip-tested successfully.
```

### Verilator lint

```bash
verilator --lint-only rtl/decoder/predecoder.sv
verilator --lint-only rtl/encoder/postencoder.sv
# etc.
```

## Módulos SystemVerilog

| Módulo | Arquivo | Descrição |
|--------|---------|-----------|
| `predecoder` | `rtl/decoder/predecoder.sv` | Decodificador principal: takum → (sign, characteristic, mantissa, precision, is_zero, is_nar) |
| `decoder_linear` | `rtl/decoder/decoder_linear.sv` | Wrapper linear (saída characteristic) |
| `decoder_logarithmic` | `rtl/decoder/decoder_logarithmic.sv` | Wrapper logarítmico (saída exponent) |
| `postencoder` | `rtl/encoder/postencoder.sv` | Encodificador principal: (sign, characteristic, mantissa, is_zero, is_nar) → takum |
| `encoder_linear` | `rtl/encoder/encoder_linear.sv` | Wrapper linear (entrada characteristic) |
| `encoder_logarithmic` | `rtl/encoder/encoder_logarithmic.sv` | Wrapper logarítmico (entrada exponent) |
| `takum_pkg` | `rtl/takum_pkg.sv` | Tipos e constantes compartilhados |

Parâmetro principal: `N` (bit width, padrão 16, faixa 2–254).

## Notas de conversão VHDL → SV

A lógica RTL é **idêntica** à original em VHDL. Mudanças foram apenas sintáticas ou de compatibilidade:

1. **Shift aritmético**: VHDL `shift_right(signed(...))` → SV `logic signed` + `>>>`
2. **Padding**: VHDL `(6 downto 0 => '0')` = 7 zeros (não 6). Bug corrigido na conversão.
3. **Arrays localparam**: iverilog não suporta → funções lookup
4. **`generate`**: → `always_comb` com part-select (`+:`)
5. **`numeric_std`**: → casts `$signed()`, `$unsigned()`, slicing

## Licença

ISC — mesma licença do [original](https://github.com/takum-arithmetic/Takum-Codec-RTL).