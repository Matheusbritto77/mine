# Servidor Minecraft Bedrock (Pocket Edition - PE) para Celular e Consoles

Este repositório contém a configuração necessária para rodar e implantar um servidor dedicado de Minecraft Bedrock Edition (PE) usando Docker Compose. Ele foi estruturado para ser implantado facilmente em uma VPS através do **Dokploy** (ou qualquer PaaS baseado em Docker/Docker Compose).

## 🚀 Como Rodar Localmente (para Testes)

Antes de implantar na sua VPS, você pode testar o servidor localmente na sua máquina:

### Pré-requisitos
*   [Docker](https://docs.docker.com/get-docker/) e [Docker Compose](https://docs.docker.com/compose/install/) instalados.

### Passos
1.  Inicie o servidor em segundo plano:
    ```bash
    docker compose up -d
    ```
2.  Para acompanhar os logs de inicialização e do servidor:
    ```bash
    docker compose logs -f
    ```
3.  Para parar o servidor:
    ```bash
    docker compose down
    ```

---

## ⚙️ Configuração Personalizada

Você pode personalizar o comportamento do servidor editando o bloco `environment` no arquivo [`docker-compose.yml`](file:///Users/matheusbritto/tuf/mine/docker-compose.yml):

| Variável | Descrição | Valor Padrão |
| :--- | :--- | :--- |
| `SERVER_NAME` | Nome que aparecerá na aba "Servidores" no jogo | `Servidor Minecraft PE (Bedrock)` |
| `GAMEMODE` | Modo de jogo padrão (`survival`, `creative`, `adventure`) | `survival` |
| `DIFFICULTY` | Dificuldade (`peaceful`, `easy`, `normal`, `hard`) | `normal` |
| `MAX_PLAYERS`| Limite máximo de jogadores simultâneos | `10` |
| `ONLINE_MODE`| `true` exige login na Xbox Live. Mude para `false` para permitir piratas/offline | `true` |
| `LEVEL_SEED` | Seed para geração do mundo (deixe vazio para aleatório) | *Vazio* |
| `OPS` | Usernames (separados por vírgula) que receberão Admin automaticamente | *Vazio* |

Após fazer qualquer alteração no `docker-compose.yml`, recrie o contêiner rodando:
```bash
docker compose up -d --force-recreate
```

---

## ☁️ Implantação na VPS usando Dokploy

O **Dokploy** suporta a implantação de aplicações que utilizam arquivos Docker Compose.

### Passo 1: Configurar no Dokploy
1.  Acesse o painel do seu Dokploy.
2.  Crie um novo **Projeto** e dentro dele crie um serviço do tipo **Compose**.
3.  Configure a origem como seu repositório Git (onde este código está hospedado).
4.  Certifique-se de preencher a URL do repositório, branch (ex: `main`) e credenciais de acesso se o repositório for privado.

### Passo 2: Configurar Redirecionamento de Portas e Firewall da VPS
Diferente de aplicações web tradicionais que usam HTTP/HTTPS (TCP), o Minecraft Bedrock utiliza o protocolo **UDP** na porta **19132**.

1.  **Portas no Dokploy**: Como especificamos a porta diretamente no `docker-compose.yml` (`19132:19132/udp`), o contêiner se ligará diretamente à porta da VPS. Dokploy reconhece e aplica a configuração de rede do Docker Compose.
2.  **Firewall da VPS (Muito Importante)**: Você precisará abrir a porta UDP `19132` no firewall da VPS para que os jogadores consigam se conectar.
    *   Se estiver usando **UFW** (Ubuntu/Debian):
        ```bash
        sudo ufw allow 19132/udp
        ```
    *   Se estiver usando provedores de nuvem (AWS, Oracle Cloud, DigitalOcean, Azure, Google Cloud), lembre-se de liberar a porta **19132 UDP** no painel de controle (Security Groups / Networking / Firewall) do provedor.

### Passo 3: Conectar no Celular / Console
1.  Abra o Minecraft no seu celular ou console.
2.  Vá em **Jogar** -> **Servidores** -> role até o fim e clique em **Adicionar Servidor**.
3.  Preencha as informações:
    *   **Nome do Servidor**: Qualquer nome de sua escolha.
    *   **Endereço do Servidor**: O IP público da sua VPS.
    *   **Porta**: `19132` (Padrão).
4.  Clique em **Salvar** ou **Jogar**!

---

## 📁 Estrutura do Repositório
*   [`docker-compose.yml`](file:///Users/matheusbritto/tuf/mine/docker-compose.yml): Configuração principal do contêiner Docker.
*   [`.gitignore`](file:///Users/matheusbritto/tuf/mine/.gitignore): Ignora a pasta `data/`, garantindo que os mundos e configurações geradas em tempo de execução fiquem salvos apenas localmente/na VPS e não sejam commitados no Git.
*   `/data/` (Gerado após rodar): Pasta local contendo mundos, logs e configurações persistentes do servidor.
