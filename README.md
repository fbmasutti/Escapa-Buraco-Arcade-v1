# Escapa Buraco  — Firmware v0.9

Guia para **gravar / atualizar o firmware** do jogo **Escapa Buraco** num
**Arduino Uno + CNC Shield v3**, e também para quem quer **montar o próprio
jogo em casa**.

- Já tem a máquina montada e só quer atualizar? Vá direto às seções **3 a 7**.
- Vai montar do zero? Comece pela seção **2 (hardware e ligações)**.

---

## 1. O que tem no zip

```
EscapaBuraco/
├── EscapeBuracoAnalog_v9/      ← pasta do sketch
│   ├── EscapeBuracoAnalog_v9.ino   (programa principal)
│   ├── tocarMusica.ino             (melodias / sinais sonoros)
│   ├── pitches.h                   (notas musicais)
│   └── pitches.cpp
├── libraries/
│   └── TM1637-avishorp.zip     ← biblioteca do display (instalar 1 vez)
└── LEIAME.md                   ← este arquivo
```

> **Importante:** o Arduino exige que o `.ino` principal esteja dentro de uma
> pasta com o **mesmo nome** dele. Por isso os 4 arquivos do sketch ficam juntos
> em `EscapeBuracoAnalog_v9/`. Ao descompactar, **não separe** esses arquivos
> nem renomeie a pasta.

---

## 2. Hardware e ligações (para montar em casa)

**Componentes principais**

| Item | Descrição |
|---|---|
| Arduino Uno | placa controladora |
| CNC Shield v3 | encaixa sobre o Uno; aloja os drivers |
| 2× drivers de passo | A4988 ou DRV8825 (eixos X e Y) |
| 2× motores de passo | NEMA 17 (uma para cada ponta da barra) |
| Display TM1637 | 4 dígitos, mostra o tempo |
| Buzzer | sinais sonoros |
| 4× chaves/joystick | controle das duas pontas (sobe/desce) |
| 2× fins de curso | referência de "zero" dos eixos X e Y |
| 3× chaves | início, fim de percurso e buraco 10 |
| Fonte externa | ~12 V para os motores (ver seção 9) |
| Cabo USB A–B | para gravar o firmware |

**Mapa de pinos** (já configurado no firmware — confira sua fiação por aqui):

| Função | Pino |
|---|---|
| STEP motor esquerdo (X) | D2 |
| STEP motor direito (Y) | D3 |
| DIR motor esquerdo (X) | D5 |
| DIR motor direito (Y) | D6 |
| ENABLE dos motores | D8 *(ativo em LOW)* |
| Display TM1637 — CLK | D4 |
| Display TM1637 — DIO | D7 |
| Joystick esquerdo sobe / desce | A0 / A1 |
| Joystick direito sobe / desce | A2 / A3 |
| Fim de curso X / Y | D9 / D10 |
| Buzzer | D11 |
| Botão de início | D12 |
| Switch de fim de percurso | D13 |
| Switch do buraco 10 (vitória) | A4 |

> Os pinos do **eixo Z** do shield (D4 e D7) são reaproveitados para o display,
> já que o jogo usa só 2 motores. Todos os botões/chaves usam o resistor de
> *pull-up* interno do Arduino: **pressionado = nível baixo (LOW)**.
> A pinagem completa também está comentada no topo do `EscapeBuracoAnalog_v9.ino`.

---

## 3. Antes de começar (pré-requisitos)

1. **Arduino IDE** instalada (2.x recomendada): https://www.arduino.cc/en/software
2. **Cabo USB** (A–B) ligando o Arduino Uno ao computador.
3. A **biblioteca do display TM1637** (passo 4).
4. (Para os motores girarem) a **fonte externa do CNC Shield** ligada — só o
   USB **não** alimenta os motores de passo. Veja a nota de segurança no fim.

---

## 4. Instalar a biblioteca do display (só uma vez por computador)

O firmware usa a biblioteca **TM1637 — por Avishay Orpaz (avishorp)**
(cabeçalho `TM1637Display.h`, classe `TM1637Display`).

> ⚠️ Existem várias bibliotecas chamadas "TM1637" com funções diferentes.
> Use **exatamente** a do Avishay Orpaz, senão o código não compila.

Escolha **uma** das opções:

**Opção A — pelo arquivo do zip (funciona offline)**
1. Abra a Arduino IDE.
2. **Sketch → Incluir Biblioteca → Adicionar biblioteca .ZIP…**
3. Selecione `libraries/TM1637-avishorp.zip` desta pasta.
4. Aguarde a confirmação.

**Opção B — pelo Gerenciador de Bibliotecas (precisa de internet)**
1. **Sketch → Incluir Biblioteca → Gerenciar Bibliotecas…**
2. Pesquise por `tm1637`.
3. Instale a opção **"TM1637" de Avishay Orpaz**.

---

## 5. Abrir o sketch

1. Entre na pasta `EscapeBuracoAnalog_v9/`.
2. Abra **`EscapeBuracoAnalog_v9.ino`** (duplo clique ou **Arquivo → Abrir…**).
3. A IDE abre o programa com **abas** no topo: `EscapeBuracoAnalog_v9`,
   `tocarMusica`, `pitches.h` e `pitches.cpp`. Se as 4 abas aparecerem, ok.

---

## 6. Selecionar a placa e a porta

1. Ligue o Arduino Uno ao computador pelo cabo USB.
2. **Ferramentas → Placa → Arduino AVR Boards → Arduino Uno**.
3. **Ferramentas → Porta** e selecione a porta que apareceu ao plugar:
   - Windows: `COM3`, `COM4`…
   - macOS/Linux: `/dev/cu.usbmodem…` ou `/dev/ttyACM0` / `ttyUSB0`.

> Na IDE 2.x dá para escolher placa e porta no seletor no topo da janela.

---

## 7. Compilar e enviar (upload)

1. Clique em **Verificar** (✔) para compilar — deve aparecer
   "Compilação concluída", sem erros em vermelho.
2. Clique em **Carregar / Upload** (→). A IDE compila de novo e grava no Arduino.
3. Aguarde **"Carregado" / "Done uploading"**. Durante o envio, os LEDs
   **TX/RX** do Arduino piscam.

**Como saber que deu certo:** ao reiniciar, o display mostra o tempo inicial
(`1:00`) com os dois pontos `:` piscando. Ao apertar o botão de início, o jogo
faz o *homing* dos motores e toca a musiquinha de abertura.

---

## 8. Problemas comuns (e soluções)

| Sintoma | Causa provável | Solução |
|---|---|---|
| `TM1637Display.h: No such file or directory` | Biblioteca não instalada ou errada | Refaça o passo 4 com a do **avishorp** |
| Erro em `showNumberDecEx`/`setSegments` | Instalada uma TM1637 **diferente** | Remova a outra e instale a do avishorp |
| `avrdude: ... not in sync` / porta não responde | Porta errada, cabo ou placa | Confira **Ferramentas → Porta**; troque o cabo; reconecte |
| Nenhuma porta aparece | Cabo "só carga" ou driver | Use cabo de dados; em clones, instale o driver CH340 |
| Compila e envia, mas **motores não giram** | Falta a fonte externa do shield | Ligue a fonte de força do CNC Shield (seção 9) |
| Motor gira para o **lado errado** | Fiação/sentido invertido | Inverta o conector do motor no driver, ou ajuste o `sentido` no código |
| Display apaga/trava em movimento | Alimentação instável | Verifique fonte e **GND comum** entre Arduino e shield |

---

## 9. ⚠️ Nota de segurança (alimentação dos motores)

- O **USB sozinho não alimenta os motores de passo**. O CNC Shield precisa da
  **fonte externa** (tipicamente 12 V) para movê-los.
- Ligue/desligue a fonte **sem** as mãos na mecânica — a barra pode se mover
  sozinha durante o *homing*.
- Não conecte/desconecte os drivers (A4988/DRV8825) com a placa energizada.
- Ajuste a **corrente dos drivers** (potenciômetro) conforme seus motores antes
  de usar por períodos longos, para evitar superaquecimento.

---

## 10. Atualizar o firmware no futuro

Mesmo processo: substitua os arquivos da pasta do sketch pela nova versão, abra
o `.ino` principal e repita as seções **6 e 7** (porta → upload). A biblioteca
só precisa ser instalada uma vez por computador.

---

### Créditos das bibliotecas
- Display: **TM1637** — Avishay Orpaz (LGPL 2.1) — github.com/avishorp/TM1637
- Notas musicais (`pitches.h`): HiBit — hibit.dev
- Rotina de músicas: baseada no *arduino-songs* de Robson Couto
