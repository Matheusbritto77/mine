# Servidor Minecraft Crossplay (Java & Bedrock / PE) para Celular e PC

Este repositório contém a configuração necessária para rodar um servidor de Minecraft **Java Edition** utilizando **Purpur MC**, configurado com **GeyserMC** e **Floodgate** para permitir que jogadores de **celular (Minecraft PE), consoles e PC** joguem juntos no mesmo mundo.

Esta estrutura foi desenhada para ser implantada facilmente em uma VPS através do **Dokploy** (ou qualquer PaaS baseado em Docker/Docker Compose).

---

## 🛠️ Como Funciona esta Arquitetura?

1.  **Purpur MC**: O servidor core é um servidor Java focado em desempenho extremo e customização avançada. Ele é o responsável por rodar o mundo, física, lógica e suportar os tradicionais **Plugins Java (.jar)**. Ele é derivado do Paper, sendo ainda mais otimizado.
2.  **GeyserMC**: Um plugin de tradução instalado no Purpur. Quando um jogador de celular/Bedrock tenta se conectar na porta `19132 UDP`, o Geyser traduz seus pacotes para a porta `25565 TCP` do Java.
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

## 🔌 Plugins do Neon Syndicate

O servidor está configurado para baixar automaticamente a maioria dos plugins no boot. A infraestrutura completa do **Neon Syndicate** é dividida em camadas:

### 🏦 Economia & Corporações (Auto-instalados)
| Plugin | Função | Comando Principal |
| :--- | :--- | :--- |
| **`vault`** | API de economia unificada do servidor | — |
| **`essentialsx`** | Economia base, teleporte, spawn | `/bal`, `/pay`, `/tpa` |
| **`towny`** | Megacorporações (Cidades) e proteção de blocos | `/t new`, `/t invite` |
| **`luckperms`** | Cargos corporativos (CEO, Diretor, Freelancer) | `/lp user`, `/lp group` |
| **`fadah`** | Mercado de Ações e Leilões | `/ah` |

### 🏗️ Construção & Geração de Mundo (Auto-instalados)
| Plugin | Função | Comando Principal |
| :--- | :--- | :--- |
| **`fastasyncworldedit`** | Editor de mundo assíncrono (importar schematics de cidades cyberpunk) | `//paste`, `//schematic` |
| **`terra`** | Gerador de mundo customizado (terreno industrial/urbano) | Configurável via YAML |
| **`chunky`** | Pré-geração de mapa (otimização de desempenho) | `/chunky start` |

### 🎭 Gameplay & Imersão (Auto-instalados)
| Plugin | Função | Comando Principal |
| :--- | :--- | :--- |
| **`decentholograms`** | Hologramas neon flutuantes (propagandas, rankings) | `/dh create` |
| **`traincarts`** + **`bkcommonlib`** | Metrôs e trens automáticos entre corporações | Sinais de controle |
| **`beautyquests`** | Contratos e Missões via NPCs | `/quests create` |

### 🌐 Crossplay (Auto-instalados)
| Plugin | Função |
| :--- | :--- |
| **`geysermc`** + **`floodgate`** | Jogadores de celular (Bedrock) jogam com conta Xbox Live |

---

## 💼 Plugins de Instalação Manual

Os seguintes plugins **não estão no Modrinth** e precisam ser baixados e colocados manualmente na pasta `/data/plugins/` na VPS:

### 1. Jobs Reborn (Sistema de Profissões)
*   [Jobs Reborn no SpigotMC](https://www.spigotmc.org/resources/jobs-reborn.4216/)
*   [CMILib (Biblioteca obrigatória)](https://www.spigotmc.org/resources/cmilib.87610/)

### 2. Citizens2 (NPCs Executivos e Mercadores)
Permite criar NPCs com skins customizadas de executivos de corporações, ciborgues e mercadores.
*   [Citizens no SpigotMC](https://www.spigotmc.org/resources/citizens.13811/)
*   [Builds de desenvolvimento (CI)](https://ci.citizensnpcs.co/job/Citizens2/)

### 3. ItemsAdder ou Oraxen (Itens Customizados, Implantes e Cosméticos 3D)
Permite criar implantes cibernéticos, armas laser, jetpacks, keycards e cosméticos neon **sem que o jogador precise instalar mods** — o servidor envia um Resource Pack automaticamente.
*   **ItemsAdder** (premium): [SpigotMC](https://www.spigotmc.org/resources/itemsadder.73355/)
*   **Oraxen** (premium): [SpigotMC](https://www.spigotmc.org/resources/oraxen.72448/)

> **Nota:** Após baixar qualquer `.jar`, coloque-o em `/data/plugins/` e reinicie com `docker compose up -d --force-recreate`.

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

### 1. Coletor de Lixo e Otimização de RAM (ZGC & Deduplication)
Já ativamos no [`docker-compose.yml`](file:///Users/matheusbritto/tuf/mine/docker-compose.yml) as variáveis de otimização de memória RAM:
*   **ZGC (`-XX:+UseZGC`)**: Limpa a memória RAM em segundo plano com pausas menores que 1ms. No Java 25, o ZGC já é geracional (Generational ZGC) por padrão, por isso a antiga flag `-XX:+ZGenerational` foi descontinuada e removida.
*   **String Deduplication (`-XX:+UseStringDeduplication`)**: O Minecraft mantém milhões de textos repetidos na memória (nomes de blocos, coordenadas, logs de chat, UUIDs de entidades). Esta flag faz com que a máquina Java identifique strings duplicadas e aponte para um único endereço de memória, **liberando entre 15% e 25% de RAM**.
*   **Teto do Metaspace (`-XX:MaxMetaspaceSize=300M`)**: Impede que a JVM acumule dados de classes de metadados infinitamente, economizando RAM da VPS.
*   **Memória alocada**: Mantivemos 6GB de RAM alocados (`MEMORY: "6G"`), deixando 2GB livres na VPS de 8GB para que o Dokploy, banco de dados e o sistema operacional possam rodar sem causar o erro de falta de memória física (`Failed to commit memory`).

### 2. Otimização de Configurações da Engine (No-Tick Rendering & Chunk Unloading)
Os arquivos de configuração gerados após a primeira inicialização em `/data/config/paper-world-defaults.yml` (ou em `spigot.yml`) podem ser ajustados. As duas técnicas mais avançadas para economizar RAM e CPU são:

*   **Descarregamento Agressivo de Chunks (RAM)**:
    *   No arquivo `paper-world-defaults.yml`, procure por `chunk-loading.unload-delay`.
    *   O padrão é `600` ticks (o servidor segura o mapa na RAM por 30 segundos após o jogador sair da área).
    *   Altere para **`20`** ou **`40`** ticks (1 a 2 segundos). Isso faz com que a memória seja liberada imediatamente assim que o jogador se afastar ou deslogar.
*   **`view-distance` (Distância física/ticking)**: Defina para `4` ou `5`. Controla o raio de blocos onde as coisas acontecem (crescimento de plantações, movimentação de mobs).
*   **`no-tick-view-distance` (Distância visual)**: Defina para `8` ou `10`. O jogador continuará vendo os blocos de longe (evitando que o horizonte fique cortado), mas as áreas distantes não processam física. Isso poupa até 60% de CPU.
*   **`simulation-distance`**: Defina para `4` ou `5`.
*   **Limites de Entidades (Evitando Crash por Acúmulo)**:
    *   Em `paper-world-defaults.yml`, reduza os limites de entidades por chunk (`entities` -> `limit`). Isso previne que farms gigantes de mobs acumulem milhares de animais e travem a RAM do servidor por estouro de memória (Out Of Memory).

### 3. Banco de Dados Separado no Dokploy (Evitando travamento de disco)
Por padrão, plugins como **LuckPerms**, **EssentialsX** ou **CoreProtect** salvam suas informações em arquivos locais H2 ou SQLite (`.db`). Sempre que um jogador entra, compra algo ou quebra um bloco, o servidor bloqueia a thread de execução para escrever no disco da VPS, gerando picos de lag.

**Solução recomendada com o Dokploy**:
1. No painel do **Dokploy**, crie um banco de dados **MySQL** ou **MariaDB** (leva 1 clique).
2. O Dokploy fornecerá as credenciais (IP interno do banco, usuário, senha e porta).
3. Vá nas pastas de configuração dos plugins (ex: `/data/plugins/LuckPerms/config.yml`).
4. Altere o tipo de armazenamento de `H2` para `MYSQL` e preencha as credenciais.
5. Os plugins agora salvarão os dados em segundo plano (assíncrono) no banco de dados, eliminando totalmente qualquer lag de gravação de arquivos de dados!

### 4. Monitoramento em Tempo Real (Spark Profiler)
O plugin **Spark** já vem **pré-instalado nativamente** no Purpur/Paper (a partir da versão 1.21). Você não precisa baixar nada!
*   Caso o servidor apresente lentidão, digite no chat do jogo (se for OP) ou no console o comando:
    ```bash
    /spark profiler start
    ```
*   Deixe rodar por cerca de 2 a 3 minutos e execute:
    ```bash
    /spark profiler stop
    ```
*   O plugin gerará um link com gráficos detalhados mostrando o uso exato de CPU de cada plugin, entidade e evento do mundo, permitindo isolar a causa do lag imediatamente.

### 5. Pré-gerar o Mundo (Essencial para tirar Lag de Exploração)
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
