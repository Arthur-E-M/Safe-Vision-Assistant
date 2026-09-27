# Safe Vision Assistant

Sistema de fiscalização de EPIs (Equipamentos de Proteção Individual) com visão computacional.

Projeto vencedor do **1º lugar na InfoIT**.

---

## Finalidade

Em obras e indústrias, a falta de uso de EPIs, como capacete e colete, ainda causa muitos acidentes de trabalho.

Hoje, a fiscalização depende do técnico de segurança, que precisa:

- observar os funcionários o tempo todo;
- chamar a atenção de quem está sem EPI, o que gera atrito;
- juntar provas por conta própria quando precisa registrar uma irregularidade.

O **Safe Vision Assistant** automatiza essa fiscalização. O sistema monitora as câmeras em tempo real, emite o alerta quando identifica uma pessoa sem EPI e registra a ocorrência com foto, data, hora e câmera.

Com isso, o alerta vem do sistema, e não de uma pessoa, e as provas são geradas automaticamente.

---

## Principais vantagens

| Vantagem | O que significa na prática |
|---|---|
| Menos atrito | O técnico não precisa fazer a correção verbal. Quem avisa é o sistema. |
| Provas automáticas | Cada ocorrência fica salva com foto, data, hora e câmera. |
| Economia de tempo | O técnico não precisa juntar evidências manualmente. |
| Funciona offline | Tudo roda no próprio computador. Nenhuma imagem é enviada para a internet. |
| Baixo custo | Funciona com uma webcam comum ou com um celular usado como câmera (DroidCam). |
| Configurável | É possível escolher quais EPIs monitorar. |

---

## Como funciona

```
Câmera  →  Detector (YOLOv8 + OpenCV)  →  Tem alguém sem EPI?
                                              │
                         ┌────────────────────┴────────────────────┐
                        Não                                        Sim
                         │                                          │
               Contorno VERDE na pessoa            Contorno VERMELHO na pessoa
                                                   + alerta visual (LED da câmera pisca)
                                                   + registro no banco com foto
                                                              │
                                                   Painel web: monitoramento,
                                                   estatísticas e histórico
```

1. A câmera envia o vídeo para o sistema.
2. O modelo **YOLOv8** detecta as pessoas em cada quadro do vídeo.
3. Cada pessoa recebe um contorno:
   - **verde** = EPI ok;
   - **vermelho** = EPI ausente, com o nome do item que falta.
4. Quando há uma irregularidade:
   - o LED da câmera pisca como alerta visual (no máximo 1 vez a cada 8 segundos);
   - a ocorrência é salva no banco de dados com a foto (no máximo 1 registro a cada 30 segundos, para não repetir o mesmo caso).
5. O técnico acompanha tudo pelo painel web.

---

## Funcionalidades

- **Login** de usuário para acessar o sistema.
- **Monitoramento ao vivo** com o vídeo já marcado pelo detector.
- **Estatísticas** das ocorrências.
- **Últimas ocorrências** exibidas em tempo real.
- **Histórico** com paginação e filtros por:
  - câmera;
  - data de início e data de fim;
  - tipo de EPI.
- **Página de detalhe** de cada ocorrência, com a foto registrada.
- **Suporte a mais de uma câmera** (cada câmera tem o seu próprio identificador).
- **Liga/desliga do alerta** pelo painel.

---

## Tecnologias

| Tecnologia | Para que é usada |
|---|---|
| Python | Linguagem principal do projeto |
| OpenCV | Captura do vídeo e desenho dos contornos |
| YOLOv8 (Ultralytics) | Detecção de pessoas com inteligência artificial |
| Flask | Servidor web e painel de controle |
| APIs REST | Comunicação entre o painel e o sistema (JSON) |
| Banco de dados | Armazenamento das ocorrências e dos usuários |
| Multithreading | Processar o vídeo, os alertas e os registros ao mesmo tempo, sem travar |

---

## Estrutura do projeto

```
safe-vision-assistant/
├── app.py              # Servidor web (Flask): rotas, login, painel e APIs
├── detector.py         # Detecção com YOLOv8, desenho dos contornos e alerta
├── yolov8n.pt          # Modelo de IA já baixado (permite rodar offline)
├── requirements.txt    # Bibliotecas necessárias
├── database/
│   └── db.py           # Acesso ao banco de dados (DatabaseManager)
└── templates/
    ├── login.html          # Tela de login
    ├── monitoramento.html  # Vídeo ao vivo e estatísticas
    ├── dados.html          # Histórico de ocorrências com filtros
    └── detalhe.html        # Detalhe de uma ocorrência
```

---

## Como instalar e rodar

**Pré-requisitos:** Python 3.11 e uma câmera (webcam ou celular com DroidCam).

1. Instale as bibliotecas (esta é a única etapa que precisa de internet):

   ```bash
   pip install -r requirements.txt
   ```

2. Inicie o sistema:

   ```bash
   python app.py
   ```

3. Abra no navegador:

   ```
   http://localhost:5000
   ```

4. Entre com o usuário padrão:

   - Usuário: `admin`
   - Senha: `admin123`

---

## Configuração

As configurações ficam no início do arquivo `detector.py`.

**Quais EPIs monitorar**

```python
EPIS_MONITORADOS = {"capacete", "colete"}   # ambos
# EPIS_MONITORADOS = {"capacete"}           # só capacete
# EPIS_MONITORADOS = {"colete"}             # só colete
```

**Qual câmera usar**

```python
CAMERA_SOURCE = 0                                    # webcam do notebook
# CAMERA_SOURCE = 1                                  # segunda câmera conectada
# CAMERA_SOURCE = "http://192.168.0.10:4747/video"   # celular com DroidCam (Wi-Fi)
```

Se a câmera escolhida não abrir, o sistema tenta usar a câmera `0` automaticamente.

**Alerta visual**

```python
FLASH_HABILITADO = True    # True = LED pisca | False = sem alerta visual
```

---

## Rotas do sistema

| Rota | Método | O que faz |
|---|---|---|
| `/login` | GET, POST | Tela de login |
| `/logout` | GET | Sai do sistema |
| `/monitoramento` | GET | Painel com vídeo ao vivo |
| `/stream/<camera_id>` | GET | Vídeo da câmera com as marcações |
| `/dados` | GET | Histórico de ocorrências com filtros |
| `/dados/<id>` | GET | Detalhe de uma ocorrência |
| `/api/estatisticas` | GET | Estatísticas em JSON |
| `/api/ocorrencias` | GET | Últimas 15 ocorrências em JSON |
| `/api/flash` | POST | Liga ou desliga o alerta visual |
| `/api/limpar` | POST | Apaga as ocorrências registradas |

Todas as rotas, exceto `/login`, exigem login.

---

## Estado atual e próximos passos

Este projeto é um **protótipo funcional**.

**O que já funciona hoje**

- Detecção real de pessoas em tempo real com YOLOv8.
- Painel web completo, com login, monitoramento, histórico, filtros e registro com foto.
- Funcionamento offline.

**O que ainda é simulado**

- A verificação de EPI (se a pessoa está ou não com capacete e colete) ainda é **simulada** no arquivo `detector.py`. O modelo atual (`yolov8n.pt`) reconhece pessoas, mas não foi treinado para reconhecer capacete e colete.

**Próximos passos**

1. Treinar um modelo YOLOv8 próprio com imagens de capacete e colete, para que a verificação de EPI seja real.
2. Adicionar um alerta sonoro, além do alerta visual.
3. Preparar o sistema para uso em produção:
   - trocar a chave secreta (`secret_key`) e a senha padrão;
   - desligar o modo de depuração (`debug=True`).

---

## Autor

**Arthur Monteiro**
Estudante de Sistemas de Informação.
