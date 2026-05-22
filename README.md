# Robô Detector de Vida

Robô baseado em **ESP32** que monitora o ambiente com múltiplos sensores, calcula uma **probabilidade de vida** e envia alertas via **WhatsApp** e salva os dados no **Supabase**.

---

## Sumário

- [Como o código funciona](#como-o-código-funciona)
- [Como montar o robô](#como-montar-o-robô)
- [Como consultar os dados salvos](#como-consultar-os-dados-salvos)
- [Configuração inicial](#configuração-inicial)

---

## Como o código funciona

O projeto é dividido em três arquivos principais:

### `main.ino` — Núcleo do sistema

Contém o `setup()` e o `loop()` principais. A cada ciclo do loop, o ESP32:

1. **Verifica a conexão Wi-Fi** e tenta reconectar se necessário.
2. **Lê todos os sensores**: joystick (X e Y), temperatura/umidade (DHT22), luminosidade (LDR) e presença (PIR).
3. **Calcula a probabilidade de vida** com base nas leituras.
4. **Controla os LEDs**:
   - LED verde aceso: sistema ativo e probabilidade de vida abaixo de 75%.
   - LED vermelho aceso: probabilidade de vida alta ( 75%) ou sistema desativado.
5. **Envia dados ao Supabase** a cada 10 segundos (se conectado ao Wi-Fi).
6. **Dispara alerta no WhatsApp** se a probabilidade de vida ultrapassar 75%.

### `functions.ino` — Funções auxiliares

| Função | O que faz |
|---|---|
| `calcularProbabilidadeVida()` | Soma pontos com base nos sensores e gera um percentual de 0 a 100% |
| `alertaVida()` | Envia mensagem de WhatsApp uma única vez quando probabilidade > 75% |
| `enviarDados()` | Faz POST no Supabase com temperatura, umidade, luminosidade, presença e probabilidade |
| `enviarWhatsApp()` | Envia mensagem via API do CallMeBot |
| `botao()` | Liga/desliga o sistema com debounce de 20 ms |
| `monitorSerial()` | Exibe dados no Serial Monitor e move o servo conforme o joystick |
| `calibrarLdr()` | Tira 100 amostras do LDR para calcular a média de calibração |

### Lógica de probabilidade de vida

A pontuação é calculada somando blocos de pontos:

| Condição | Pontos |
|---|---|
| Temperatura entre 15C e 30C | +25 |
| Umidade entre 40% e 70% | +25 |
| Luminosidade (LDR) > 2000 | +20 |
| Presença detectada pelo PIR | +30 |
| **Total máximo** | **100%** |

### `conections.ino` — Gerenciamento de Wi-Fi

Gerencia a conexão Wi-Fi de forma não-bloqueante com timeout de **20 segundos**. Se não conseguir conectar, desconecta e tenta novamente automaticamente.

---

## Como montar o robô

### Lista de componentes

| Componente | Quantidade |
|---|---|
| ESP32 (DevKit ou similar) | 1 |
| Sensor DHT22 (temperatura e umidade) | 1 |
| Sensor PIR (presença/movimento) | 1 |
| LDR (resistor dependente de luz) | 1 |
| Servo motor | 1 |
| Joystick analógico (módulo KY-023) | 1 |
| LED vermelho | 1 |
| LED verde | 1 |
| Resistores 220 (para os LEDs) | 2 |
| Resistor 10k (pull-down para o LDR) | 1 |
| Botão (push button) | 1 |
| Protoboard e jumpers |  |

### Pinagem

| Componente | Pino no ESP32 |
|---|---|
| Joystick VRx | GPIO 36 |
| Joystick VRy | GPIO 39 |
| Servo motor | GPIO 19 |
| DHT22 | GPIO 23 |
| Botão | GPIO 25 |
| LED Vermelho | GPIO 26 |
| LED Verde | GPIO 27 |
| LDR | GPIO 34 |
| Sensor PIR | GPIO 13 |

### Diagrama de conexões

```
ESP32
 GPIO 36  VRx do Joystick
 GPIO 39  VRy do Joystick
 GPIO 19  Sinal do Servo Motor
 GPIO 23  Data do DHT22
 GPIO 25  Botão (outra perna no GND)
 GPIO 26  Resistor 220  LED Vermelho  GND
 GPIO 27  Resistor 220  LED Verde  GND
 GPIO 34  LDR em divisor de tensão (com resistor 10k para GND)
 GPIO 13  Sinal do Sensor PIR
```

> **Atenção:** O botão deve ser conectado entre o **GPIO 25** e o **GND**. O código já usa `INPUT_PULLUP` internamente.

---

## Como consultar os dados salvos

Os dados são enviados ao **Supabase** a cada 10 segundos. Para consultá-los:

### 1. Pelo painel do Supabase

1. Acesse [supabase.com](https://supabase.com) e faça login.
2. Abra o seu projeto.
3. No menu lateral, clique em **Table Editor**.
4. Selecione a tabela onde os dados estão sendo inseridos.
5. Os registros aparecem em ordem de inserção, com os campos:

| Campo | Descrição |
|---|---|
| `temperatura_c` | Temperatura em graus Celsius |
| `umidade_pct` | Umidade relativa em % |
| `luminosidade` | Valor analógico do LDR (04095) |
| `presenca` | `true` se movimento foi detectado, `false` caso contrário |
| `probabilidade_vida` | Percentual calculado (0100%) |

### 2. Pelo SQL Editor do Supabase

Ainda no painel, clique em **SQL Editor** e execute:

```sql
SELECT * FROM nome_da_tabela ORDER BY created_at DESC LIMIT 50;
```

Substitua `nome_da_tabela` pelo nome real da sua tabela.

### 3. Via API REST

Você pode consultar os dados de qualquer aplicação com uma requisição HTTP:

```bash
curl "https://SEU_PROJETO.supabase.co/rest/v1/nome_da_tabela?order=created_at.desc&limit=10" \
  -H "apikey: SUA_API_KEY" \
  -H "Authorization: Bearer SUA_API_KEY"
```

---

## Configuração inicial

Antes de carregar o código no ESP32, edite o arquivo `secrets.h`:

```cpp
#define WIFI_SSID     "nome_da_sua_rede"
#define WIFI_PASSWORD "senha_da_sua_rede"
#define PHONE_NUMBER  "+5511999999999"   // Com código do país
#define API_KEY       "sua_chave_callmebot"
```

E no arquivo `main.ino`, preencha as credenciais do Supabase:

```cpp
const char* supabase_url = "https://SEU_PROJETO.supabase.co/rest/v1/nome_da_tabela";
const char* api_key      = "SUA_API_KEY_DO_SUPABASE";
```

### Obter a chave do CallMeBot

1. Adicione o número **+34 644 61 25 65** nos seus contatos do WhatsApp.
2. Envie a mensagem: `I allow callmebot to send me messages`
3. Você receberá sua `API_KEY` em alguns instantes.

---

## Bibliotecas necessárias

Instale pelo **Gerenciador de Bibliotecas** da Arduino IDE:

- `DHTesp`
- `ESP32Servo`
- `ArduinoJson`
- `UrlEncode`
- `WiFi` *(já inclusa no pacote ESP32)*
- `HTTPClient` *(já inclusa no pacote ESP32)*
