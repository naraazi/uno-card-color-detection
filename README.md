# UNO Card Color Detection

Projeto de visão computacional em Python para detectar, em tempo real, as cores principais associadas a cartas de UNO usando webcam. O repositório também inclui uma variação integrada com Arduino via serial, capaz de acionar LEDs e reproduzir áudio conforme a cor identificada.

## Visão geral

O projeto possui dois modos de operação:

- `main.py`: versão standalone, com processamento local em OpenCV e exibição da cor predominante na tela.
- `serial_version/main_serial.py`: versão estendida que envia eventos para um Arduino e dispara áudios para feedback multimodal.

As cores monitoradas atualmente são:

- vermelho
- amarelo
- verde
- azul

## Casos de uso

- prototipação de detector de cartas do UNO
- demonstrações educacionais de visão computacional
- integração simples entre Python, OpenCV e Arduino
- experimentos de acessibilidade com feedback visual e sonoro

## Arquitetura

### 1. Captura de vídeo

O sistema usa `cv2.VideoCapture(...)` para ler frames da câmera em tempo real.

### 2. Conversão de espaço de cor

Cada frame é convertido de BGR para HSV. Esse espaço é mais adequado para segmentação por faixa de cor do que RGB/BGR, especialmente em cenários com pequenas variações de iluminação.

### 3. Segmentação por limiar

Para cada cor configurada, o código:

1. cria uma máscara com `cv2.inRange(...)`
2. contabiliza a quantidade de pixels detectados
3. considera a cor presente quando o total ultrapassa um limiar mínimo

### 4. Decisão de saída

O comportamento atual é:

- se todas as cores forem detectadas simultaneamente, retorna `All the colors!`
- se uma ou mais cores forem detectadas, retorna a cor com maior quantidade de pixels
- se nenhuma faixa atingir o limiar, retorna `No colors!`

### 5. Feedback visual

O frame de saída é reconstruído sobre um fundo branco, preservando apenas as regiões classificadas e desenhando o status textual na janela principal.

### 6. Integração serial com Arduino

Na versão `serial_version`, a aplicação envia um caractere pela serial conforme a cor detectada:

- `r`: red
- `y`: yellow
- `g`: green
- `b`: blue
- `a`: all colors

O sketch Arduino lê esse caractere e acende o LED correspondente por um curto intervalo.

## Estrutura do repositório

```text
.
├── assets/                   # Áudios usados pela versão com Arduino
├── serial_version/
│   ├── main.ino              # Sketch Arduino
│   └── main_serial.py        # Versão Python com serial + áudio
├── main.py                   # Versão principal com OpenCV
├── requirements.txt          # Dependências Python
├── README.md
└── MIT-LICENSE.txt
```

## Requisitos

### Software

- Python 3.10+ recomendado
- webcam funcional
- sistema com suporte a janela gráfica para OpenCV

### Dependências Python

Instale com:

```bash
pip install -r requirements.txt
```

Pacotes utilizados:

- `opencv-python`
- `numpy`
- `pyserial`
- `playsound`

Observação: `pyserial` e `playsound` só são necessários para a versão com Arduino.

## Como executar

### Modo 1: Detecção local com webcam

Execute:

```bash
python main.py
```

Comportamento esperado:

- a webcam será aberta
- a janela `Window` exibirá apenas as regiões detectadas
- o texto no topo indicará a cor predominante, ausência de cor, ou detecção simultânea de todas
- pressione `q` para encerrar

### Modo 2: Python + Arduino + áudio

1. Grave o arquivo `serial_version/main.ino` no Arduino.
2. Conecte os LEDs aos pinos configurados no sketch.
3. Ajuste a porta serial em `serial_version/main_serial.py`.
4. Ajuste os caminhos dos arquivos de áudio em `audio_files`.
5. Execute:

```bash
python serial_version/main_serial.py
```

Quando uma cor for detectada:

- o Python envia um caractere pela serial
- o Arduino acende o LED correspondente
- a aplicação tenta reproduzir o áudio associado

## Configuração e ajustes

### Faixas HSV

As faixas de cor são definidas no dicionário `color_ranges` presente nos scripts Python. Exemplo:

```python
color_ranges = {
    "red": [(0, 100, 100), (10, 255, 255)],
    "yellow": [(20, 150, 150), (30, 255, 255)],
    "green": [(40, 100, 100), (80, 255, 255)],
    "blue": [(90, 100, 100), (130, 255, 255)]
}
```

Se o ambiente tiver iluminação diferente, brilho intenso ou webcam com balanço de branco agressivo, essas faixas provavelmente precisarão ser recalibradas.

### Limiar de detecção

A presença de uma cor depende do número mínimo de pixels detectados:

```python
if color_pixels > 500:
```

Aumente esse valor para reduzir ruído. Diminua para ganhar sensibilidade.

### Índice da câmera

O projeto usa índices fixos de câmera:

- `main.py`: `cv2.VideoCapture(0)`
- `serial_version/main_serial.py`: `cv2.VideoCapture(2)`

Se a câmera correta não abrir, ajuste esse índice conforme o dispositivo disponível no seu sistema.

## Limitações conhecidas

- A detecção depende fortemente de iluminação e qualidade da câmera.
- O vermelho está coberto por apenas uma faixa HSV; em alguns cenários pode ser necessário tratar a segunda faixa típica do vermelho no HSV.
- A versão serial usa caminhos absolutos para os arquivos de áudio, o que reduz a portabilidade entre máquinas.
- O `playsound` pode apresentar comportamento inconsistente dependendo do sistema operacional e da configuração local de áudio.
- Não há calibração automática, testes automatizados ou interface de configuração.

## Licença

Este projeto é distribuído sob a licença MIT. Veja [MIT-LICENSE.txt](./MIT-LICENSE.txt).
