# Servidor Minecraft Crossplay (Java & Bedrock / PE) para Celular e PC

Este repositório contém a configuração necessária para rodar um servidor de Minecraft **Java Edition** utilizando **Paper MC**, configurado com **GeyserMC** e **Floodgate** para permitir que jogadores de **celular (Minecraft PE), consoles e PC** joguem juntos no mesmo mundo.

Esta estrutura foi desenhada para ser implantada facilmente em uma VPS através do **Dokploy** (ou qualquer PaaS baseado em Docker/Docker Compose).

---

## 🛠️ Como Funciona esta Arquitetura?

1.  **Paper MC**: O servidor core é um servidor Java super otimizado. Ele é o responsável por rodar o mundo física, lógica e suportar os tradicionais **Plugins Java (.jar)**.
2.  **GeyserMC**: Um plugin de tradução instalado no Paper. Quando um jogador de celular/Bedrock tenta se conectar na porta `19132 UDP`, o Geyser traduz seus pacotes para a porta `25565 TCP` do Java.
3.  **Floodgate**: Permite que jogadores de celular façam login e entrem no servidor Java usando apenas a sua conta da Xbox Live (celular), sem precisar possuir uma conta paga do Minecraft de computador (Java).

---

## 🚀 Como Rodar Localmente (para Testes)

### Pré-requisitos
*   [Docker](https://docs.docker.com/get-docker/) e [Docker Compose](https://docs.docker.com/compose/install/) instalados.

### Passos
1.  Inicie o servidor em segundo plano:
    ```bash
    docker compose up -d
    ```
    *Nota: Na primeira execução, o Docker irá baixar automaticamente o Paper MC, o GeyserMC e o Floodgate. Isso pode levar alguns minutos dependendo da sua conexão.*
2.  Acompanhe a inicialização pelos logs:
    ```bash
    docker compose logs -f
    ```
3.  Para desligar o servidor:
    ```bash
    docker compose down
    ```

---

## 🔌 Como Instalar Plugins e Mods de Servidor

Como o servidor é baseado no Paper MC, você pode customizar o jogo instalando plugins (como sistemas de login, economia, proteção de blocos, teleportes, VIPs, etc.):

1.  Inicie o servidor pelo menos uma vez para que a estrutura de pastas seja criada.
2.  Uma pasta chamada `data` será criada no diretório do projeto.
3.  Baixe o arquivo `.jar` do plugin desejado (do SpigotMC, PaperMC ou Modrinth).
4.  Coloque o arquivo `.jar` na pasta [`/data/plugins/`](file:///Users/matheusbritto/tuf/mine/data/plugins/).
5.  Reinicie o servidor para carregar o plugin:
    ```bash
    docker compose up -d --force-recreate
    ```

---

## ⚙️ Configurações do docker-compose.yml

Você pode editar o bloco `environment` no arquivo [`docker-compose.yml`](file:///Users/matheusbritto/tuf/mine/docker-compose.yml):

| Variável | Descrição | Padrão |
| :--- | :--- | :--- |
| `MOTD` | Mensagem que aparece na lista de servidores | `Servidor Crossplay Java & PE` |
| `DIFFICULTY` | Dificuldade do jogo (`peaceful`, `easy`, `normal`, `hard`) | `normal` |
| `ONLINE_MODE`| `true` exige conta original para PC. Mude para `false` para permitir jogadores de PC pirata | `true` |
| `OPS` | Usernames de jogadores Java que receberão administrador (separados por vírgula) | *Vazio* |

---

## ☁️ Implantação na VPS usando Dokploy

### Passo 1: Configurar no Dokploy
1.  Acesse o painel do seu **Dokploy**.
2.  Crie um novo **Projeto** e, dentro dele, crie um serviço do tipo **Compose**.
3.  Configure a origem do código apontando para a URL deste repositório Git (ex: no GitHub).
4.  Defina a branch de deploy como `main`.

### Passo 2: Configurar o Firewall da VPS (CRÍTICO)
Para que os jogadores de computador e de celular consigam se conectar, você precisa expor e liberar **duas portas diferentes** no firewall da sua VPS:

1.  **Minecraft Java (PC)**: Porta **25565 TCP**
2.  **Minecraft Bedrock (Celular/PE/Consoles)**: Porta **19132 UDP**

*   Se estiver usando **UFW** no Ubuntu/Debian, execute na VPS:
    ```bash
    sudo ufw allow 25565/tcp
    sudo ufw allow 19132/udp
    ```
*   Se estiver usando um provedor de nuvem (AWS, Oracle Cloud, DigitalOcean, Google Cloud), abra as portas **25565 TCP** e **19132 UDP** nas regras de rede/firewall (Security Groups) no console do provedor.

### Passo 3: Como os jogadores se conectam

*   **Pelo PC (Java Edition)**:
    *   **IP/Endereço**: IP da sua VPS
    *   **Porta**: `25565` (Padrão)
*   **Pelo Celular/Console (Bedrock/PE)**:
    *   **IP/Endereço**: IP da sua VPS
    *   **Porta**: `19132` (Padrão)

---

## ⚡ Otimização de Desempenho (Redução de Lag)

Para garantir que o servidor rode liso na sua VPS sem travamentos ou picos de CPU, siga estas otimizações recomendadas:

### 1. Ajuste de Memória RAM (Aikar's Flags)
Já ativamos no [`docker-compose.yml`](file:///Users/matheusbritto/tuf/mine/docker-compose.yml) a variável `USE_AIKAR_FLAGS=true` e definimos `MEMORY: "3G"`.
*   **Dica**: Se sua VPS tiver mais RAM (ex: 8GB), altere o `MEMORY` para `7G` no `docker-compose.yml`. Mantenha sempre 1GB livre para o sistema operacional e para o Dokploy.

### 2. Otimização de Configurações do Paper (Mais Impacto)
O arquivo de configurações do mundo do Paper (gerado após a primeira inicialização em `/data/config/paper-world-defaults.yml` ou em `spigot.yml`) contém valores que afetam muito o desempenho. Recomendamos editar os seguintes valores:

*   **`view-distance` (Distância de Visão)**: Altere de `10` para `6` ou `8`. Isso controla quantos blocos (chunks) o servidor envia para o jogador. Valores menores poupam muita CPU e RAM.
*   **`simulation-distance` (Distância de Simulação)**: Altere para `4` ou `5`. Controla até onde os monstros se movem e plantas crescem.
*   **`entity-activation-range` (em spigot.yml)**: Reduza a ativação de entidades (mobs) para que eles não consumam CPU quando estiverem longe dos jogadores.

### 3. Pré-gerar o Mundo (Essencial para tirar Lag de Exploração)
Quando um jogador corre pelo mapa e entra em áreas novas, a CPU da VPS precisa calcular e gerar os blocos do zero em tempo real, causando travamentos temporários no servidor (lag spikes).

Para evitar isso, pré-gere os blocos usando o plugin **Chunky**:
1. Baixe o plugin [Chunky no Modrinth](https://modrinth.com/plugin/chunky) e coloque-o na pasta `/data/plugins/`.
2. Reinicie o servidor.
3. No console do jogo (ou executando comandos no contêiner), execute os seguintes comandos para gerar uma área de 5000 blocos ao redor do spawn:
   ```bash
   # Define o centro da geração no spawn do mundo
   /chunky spawn
   
   # Define o raio da geração (5000 blocos)
   /chunky radius 5000
   
   # Inicia a pré-geração
   /chunky start
   ```
4. Aguarde o processo terminar. Uma vez concluído, o lag de exploração desaparecerá por completo!
