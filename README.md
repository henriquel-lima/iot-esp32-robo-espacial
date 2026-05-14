https://wokwi.com/projects/463933348212379649

Controle de Sistema com ESP32, Joystick e WhatsApp
Objetivo da etapa

Este projeto tem como objetivo desenvolver um sistema utilizando o ESP32 para:

Ler os movimentos de um joystick analógico;
Controlar o estado do sistema através de um botão;
Indicar o funcionamento utilizando LEDs;
Enviar notificações via WhatsApp usando a API do CallMeBot
;
Simular o circuito na plataforma Wokwi
.

O sistema permanece ativo enquanto o botão não for pressionado. Quando desligado, uma mensagem é enviada automaticamente para o WhatsApp configurado.

Componentes do circuito

Os componentes utilizados no projeto são:

1x ESP32
1x Joystick analógico
1x Push Button
1x LED verde
1x LED vermelho
2x Resistores para os LEDs (220Ω recomendados)
Jumpers para conexão
Rede Wi-Fi para comunicação HTTP
Funcionamento do sistema
Sistema ligado

Quando o sistema está ativo:

LED verde permanece aceso;
LED vermelho permanece apagado;
O joystick é monitorado continuamente;
Os comandos são exibidos no monitor serial:
ESQUERDA
DIREITA
FRENTE
TRÁS
Sistema desligado

Quando o botão é pressionado:

O estado do sistema é alternado;
LED verde apaga;
LED vermelho acende;
Uma mensagem é enviada via WhatsApp:
Comando: Desligar
Como rodar no Wokwi
1. Acesse o Wokwi

Abra a plataforma:

Wokwi Simulator

2. Crie um novo projeto ESP32
Clique em "New Project"
Escolha ESP32
3. Monte o circuito

Adicione os seguintes componentes:

ESP32
Joystick
Push Button
2 LEDs

Realize as conexões conforme os pinos definidos no código:

Componente	GPIO ESP32
Botão	GPIO 4
LED Verde	GPIO 9*
LED Vermelho	GPIO 10*
VRx do Joystick	GPIO 39
VRy do Joystick	GPIO 40

*Recomenda-se alterar GPIO 9 e 10 para GPIOs seguros no ESP32 real, como GPIO 2 e 5.

4. Cole o código

Copie o código .ino para o editor do Wokwi.

5. Configure o Wi-Fi

O Wokwi utiliza automaticamente:

const char* ssid = "Wokwi-GUEST";
const char* password = "";

Não é necessário alterar.

6. Configure o CallMeBot

Para receber mensagens no WhatsApp:

Acesse:

CallMeBot API Setup

Autorize o número desejado;
Copie sua API Key;
Substitua no código:
String phoneNumber = "SEU_NUMERO";
String apiKey = "SUA_APIKEY";

7. Execute a simulação
Clique em Start Simulation;
Abra o Serial Monitor;
Movimente o joystick para visualizar os comandos;
Pressione o botão para desligar o sistema e enviar a mensagem no WhatsApp.

Tecnologias utilizadas
Linguagem C++
ESP32
Wi-Fi
HTTPClient
API CallMeBot
Simulação Wokwi
