# VixEconomias — Documentação

Plugin de economias infinitas para Bukkit/Spigot/Paper, com formatação configurável, top, tags, suporte a SQLite/MySQL e cache Redis opcional. Compatível com Minecraft 1.8.X a 1.21.X.

---

## Sumário

1. [Visão geral](#1-visão-geral)
2. [Instalação](#2-instalação)
3. [Arquivos de configuração](#3-arquivos-de-configuração)
4. [Comandos](#4-comandos)
5. [Permissões](#5-permissões)
6. [Placeholders (PlaceholderAPI)](#6-placeholders-placeholderapi)
7. [Banco de dados](#7-banco-de-dados)
8. [Redis (opcional)](#8-redis-opcional)
9. [Top jogadores](#9-top-jogadores)
10. [API para outros plugins](#10-api-para-outros-plugins)
11. [Eventos](#11-eventos)
12. [Compatibilidade entre versões](#12-compatibilidade-entre-versões)

---

## 1. Visão geral

VixEconomias permite criar **quantas economias quiser** (coins, gems, tokens, etc.), cada uma com seu próprio comando, formato de exibição, símbolo, saldo inicial e ranking. Cada economia vira um comando próprio no servidor (ex.: `/coins`, `/gems`, `/tokens`).

### Características principais

- Multi-economia (sem limite)
- 2 formatos: `DECIMAL` (`$1.234,56`) e `LETTER` (`1.2K`, `5.7M`, `9.1B`...)
- Top jogadores com modo `CHAT` ou `MENU` configurável
- Permissão dinâmica de bypass do top por economia
- Persistência em SQLite (default) ou MySQL
- Cache + sync entre servidores via Redis (opcional)
- API pública pra integração com outros plugins
- PlaceholderAPI integrado (`%vixeconomias_*%`)
- Auto-save periódico + save em quit/kick/disable
- Compatível 1.8 → 1.21 (MaterialAdapter resolve names automaticamente)

---

## 2. Instalação

A instalação é feita **via Vix Loader** com base na licença do cliente. Não é necessário fazer upload manual do `.jar` no servidor — o loader baixa o plugin sob demanda em memória.

### 2.1 Fluxo geral

```
┌───────────────────┐         ┌────────────────────┐         ┌─────────────────┐
│  Servidor Bukkit  │         │ www.vixplugins.com │         │   Sua licença   │
│                   │         │                    │         │                 │
│  + Vix-API        │ ───1──► │                    │         │                 │
│    plugin         │ ◄───2── │  Envia o plugin    │ ◄──3─── │  Valida quais   │
│                   │         │   de economias     │         │  plugins libera │
│  Carrega em RAM   │         │                    │         │                 │
└───────────────────┘         └────────────────────┘         └─────────────────┘

1. API autentica com a licença
2. Servidor recebe o plugin
```

### 2.2 Pré-requisitos no servidor

1. **Vix-API** instalado em `plugins/` (esse você baixa uma vez)
2. **Licença** configurada (adicionar o ip da sua maquina no painel de plugins)
3. **Java 8+** (recomendado Java 11/17 pra servidores modernos)
4. **PlaceholderAPI** (opcional, só se for usar os placeholders)
5. **Drivers JDBC** (caso o servidor não os forneça):
   - SQLite: maioria dos forks Bukkit já inclui
   - MySQL: `mysql-connector-java*.jar` em `plugins/` ou `lib/` do servidor

### 2.3 Ativar o VixEconomias na sua conta

1. Acesse o painel da Vix plugins
2. Faça login com sua conta licenciada
3. Localize o plugin **VixEconomias** na sua lista
4. Ative-o pro servidor/licença desejado

### 2.4 Primeira execução

Quando o servidor iniciar com o loader configurado:

1. O loader contata o website e verifica a licença
2. O`VixEconomias` é baixado em memória
3. O `onEnable()` do VixEconomias roda normalmente
4. Os arquivos de configuração são gerados em `plugins/VixEconomias/`:
   - `config.yml`
   - `economies.yml`
   - `commands.yml`
   - `messages.yml`
   - `bypass.yml` (gerado em runtime)
   - `database.db` (SQLite, se for o modo escolhido)
4. Console exibe: `[VixEconomias] Enabling VixEconomias v1.0.0`

### 2.5 Atualizar o plugin

Não é necessário substituir o jar manualmente. Quando uma nova versão é publicada no website:
- Reinicie o servidor (a API baixa a versão mais recente)
- Ou use o comando da API pra hot-reload (se suportado)

As configs em `plugins/VixEconomias/` **não** são sobrescritas em updates — só são adicionados campos novos.

### 2.6 Desinstalar

- No painel Vix, desative o VixEconomias pra essa licença
- Reinicie o servidor
- (Opcional) Apague `plugins/VixEconomias/` se não quiser manter dados/configs

### 2.7 Verificar que está rodando

Use `/vixeconomias help` no console ou in-game. Se exibir a lista de economias, o plugin foi carregado com sucesso.

Outra forma: `/plugins` mostra o `VixEconomias` na lista (em verde quando carregado).

### 2.8 Solução de problemas comuns na instalação

| Sintoma no console | Causa provável | Solução |
|---|---|---|
| `License invalid` ou `Unauthorized` | Licença expirada ou não tem acesso ao plugin | Cheque o painel da Vix, renove/ative |
| `Failed to download` | Sem conectividade ao website | Verifique firewall / DNS / proxy do servidor |
| `Driver SQLite não encontrado` | sqlite-jdbc ausente | Adicione `sqlite-jdbc-3.46.0.0.jar` em `plugins/` ou `lib/` do servidor |
| `Driver MySQL não encontrado` | mysql-connector ausente (quando `database.type: MYSQL`) | Adicione `mysql-connector-java-8.x.jar` em `plugins/` ou `lib/` |
| `Failed to load class org.slf4j.impl.StaticLoggerBinder` | Warning do Jedis, não afeta funcionamento | Pode ignorar |

---

## 3. Arquivos de configuração

### `config.yml`
Configurações gerais: prefixo de mensagens, intervalo de auto-save, banco de dados, Redis, formato decimal, lista de letras (K/M/B/T/...), config do menu de top, placeholders.

### `economies.yml`
Define cada economia. Estrutura:

```yaml
economies:
  coins:
    command: "coins"                    # Nome do comando criado
    aliases: ["coin", "money", "saldo"] # Aliases do comando
    display-name: "&aCoins"             # Exibido nas mensagens
    symbol: "$"                         # Símbolo monetário
    format: "DECIMAL"                   # DECIMAL ou LETTER
    starting-balance: 10.0              # Saldo inicial ao entrar 1ª vez
    min-balance: 0.0                    # Mínimo permitido
    max-balance: -1                     # -1 = ilimitado
    rich-tag:
      enabled: true
      tag: "&2[&a$ Top Coins&2] &r"     # Mostrado em %_rich% pro top #1
    top:
      mode: "MENU"                      # MENU ou CHAT
      amount: -1                        # -1 = todos
```

### `commands.yml`
Aliases customizáveis pra cada subcomando (`help`, `pay`, `top`, `give`, `remove`, `setar`, `reload`). Não confunda com os comandos das economias (esses estão em `economies.yml`).

### `messages.yml`
Todas as mensagens enviadas pro jogador. Suporta:
- Códigos de cor clássicos: `&a`, `&l`, `&r`, etc.
- Hex colors: `&#FF5733` (renderizado em Spigot/Paper 1.16+; clientes 1.8 não veem)
- Placeholders por mensagem (documentados em cada bloco)

### `bypass.yml`
Gerado automaticamente em runtime. Cache persistente do bypass do top — não mexa manualmente.

---

## 4. Comandos

### Comando admin

| Comando | Descrição | Permissão |
|---|---|---|
| `/vixeconomias` (`vixeco`, `veco`) | Comando admin/help | `vixeconomias.use` |
| `/vixeconomias help` | Lista economias + subcomandos | `vixeconomias.use` |
| `/vixeconomias reload` | Recarrega todas as configurações | `vixeconomias.reload` |

### Comandos por economia

Cada economia em `economies.yml` gera **seu próprio comando**. Default: `/coins`, `/gems`, `/tokens`.

| Comando | Descrição | Permissão |
|---|---|---|
| `/<economia>` | Mostra seu saldo | `vixeconomias.use` |
| `/<economia> <jogador>` | Mostra saldo de outro jogador | `vixeconomias.balance.others` |
| `/<economia> pay <jogador> <quantia>` | Paga outro jogador | `vixeconomias.pay` |
| `/<economia> top` | Abre o top (chat ou menu por config) | `vixeconomias.top` |
| `/<economia> give <jogador> <quantia>` | Adiciona saldo | `vixeconomias.give` |
| `/<economia> remove <jogador> <quantia>` | Remove saldo | `vixeconomias.remove` |
| `/<economia> setar <jogador> <quantia>` | Define saldo | `vixeconomias.setar` |

### Notação de quantia

O parâmetro `<quantia>` aceita:
- Números: `100`, `1500.50`, `1_000_000`
- Sufixos de letras (LETTER mode): `1K`, `2.5M`, `10B`
- Decimal com `,` ou `.`: `1,5K` ou `1.5K`

---

## 5. Permissões

### Permissões fixas

| Permissão | Default | Descrição |
|---|---|---|
| `vixeconomias.admin` | `op` | Acesso a todos os comandos administrativos |
| `vixeconomias.use` | `true` | Comandos básicos (ver próprio saldo, help) |
| `vixeconomias.reload` | `op` | Recarregar configurações |
| `vixeconomias.give` | `op` | Dar dinheiro a jogadores |
| `vixeconomias.remove` | `op` | Remover dinheiro de jogadores |
| `vixeconomias.setar` | `op` | Definir saldo de jogadores |
| `vixeconomias.balance.others` | `op` | Ver saldo de outros jogadores |
| `vixeconomias.pay` | `true` | Pagar outros jogadores |
| `vixeconomias.top` | `true` | Ver o top |

### Permissões dinâmicas (uma por economia)

| Permissão | Default | Descrição |
|---|---|---|
| `vixeconomias.<economyId>.top.bypass` | `false` | Oculta o jogador do top dessa economia específica |

Exemplos com as economias default:
- `vixeconomias.coins.top.bypass`
- `vixeconomias.gems.top.bypass`
- `vixeconomias.tokens.top.bypass`

Quem tiver a permissão não aparece no `/<economia> top` nem nos placeholders `%vixeconomias_<eco>_top_*%`.

---

## 6. Placeholders (PlaceholderAPI)

Todos começam com `%vixeconomias_...%`. Requer **PlaceholderAPI** instalado.

Substitua `<economia>` pelo **id** da economia em `economies.yml` (lowercase, ex.: `coins`, `gems`, `tokens`).

| Placeholder | Retorna |
|---|---|
| `%vixeconomias_<economia>_amount%` | Saldo formatado do jogador (ex.: `$1.234,56` ou `15K`) |
| `%vixeconomias_<economia>_rich%` | A `rich-tag` da economia se o jogador for #1 do top; senão `placeholders.rich-empty` da config |
| `%vixeconomias_<economia>_top_player_<N>%` | Nome do jogador na posição N do top |
| `%vixeconomias_<economia>_top_value_<N>%` | Valor formatado do jogador na posição N do top |

### Exemplos

```
%vixeconomias_coins_amount%        → $1.234,56
%vixeconomias_gems_amount%         → 15.2K
%vixeconomias_coins_rich%          → [$ Top Coins]    (só pro #1)
%vixeconomias_coins_top_player_1%  → Steve
%vixeconomias_coins_top_value_1%   → $999.999.999
%vixeconomias_gems_top_player_5%   → Alex
```

### Fallbacks (config.yml)

Quando uma posição do top está vazia ou o jogador não é o mais rico:

```yaml
placeholders:
  top-empty-player: "Ninguém"
  top-empty-value: "0"
  rich-empty: ""
```

---

## 7. Banco de dados

Configuração em `config.yml`:

```yaml
database:
  type: "SQLITE"   # ou "MYSQL"
  sqlite:
    file: "database.db"
  mysql:
    host: "localhost"
    port: 3306
    database: "vixeconomias"
    username: "root"
    password: ""
    use-ssl: false
    table-prefix: "vix_"
```

### Drivers JDBC

O plugin **não** bundla os drivers. Esperamos que o servidor forneça:
- **SQLite**: `sqlite-jdbc` (TacoSpigot/Paper costumam ter; se não, baixe em <https://repo.maven.apache.org/maven2/org/xerial/sqlite-jdbc/>)
- **MySQL**: `mysql-connector-java` (idem)

Sem driver presente, o plugin avisa no console e desabilita o tipo de banco correspondente.

### Estrutura das tabelas

```sql
CREATE TABLE players (
  uuid VARCHAR(36) NOT NULL PRIMARY KEY,
  name VARCHAR(32) NOT NULL,
  last_seen BIGINT NOT NULL
);

CREATE TABLE balances (
  uuid VARCHAR(36) NOT NULL,
  economy VARCHAR(64) NOT NULL,
  balance DOUBLE NOT NULL,
  PRIMARY KEY (uuid, economy)
);

-- Índices
CREATE INDEX idx_players_name ON players(name);
CREATE INDEX idx_balances_economy ON balances(economy, balance);
```

### Save garantido

Os dados do jogador são salvos automaticamente em:
- **Quit**: ao sair do servidor
- **Kick**: ao ser expulso
- **Plugin disable**: ao reiniciar/parar o servidor
- **Auto-save periódico**: a cada `auto-save-interval` segundos (default 300)
- **Comando admin em jogador offline**: imediato

Saves do mesmo jogador são serializados por lock por UUID — sem corrupção por concorrência.

---

## 8. Redis (opcional)

Cache compartilhado entre servidores. Habilite em `config.yml`:

```yaml
redis:
  enabled: true
  host: "localhost"
  port: 6379
  password: ""
  database: 0
  timeout: 2000
  channel: "vixeconomias:sync"
  key-prefix: "vixeconomias:"
  cache-ttl: 600
```

Funcionalidades quando ativado:
- Cache de saldos com TTL (reduz queries no banco)
- Pub/sub entre servidores: alteração em um servidor invalida cache nos outros
- Invalidação automática do top quando saldos mudam

O Jedis (cliente Redis) vem instalado no plugin. Não precisa instalar nada extra.

---

## 9. Top jogadores

### Configuração por economia (`economies.yml`)

```yaml
coins:
  top:
    mode: "MENU"   # MENU ou CHAT
    amount: -1     # quantos exibir (-1 = todos)
```

### Configuração global (`config.yml`)

```yaml
top:
  default-mode: "MENU"
  default-amount: 10
  chat-page-size: 10
  precompute: true        # roda refresh no startup
```

```yaml
settings:
  top-refresh-interval: 60  # atualiza cache a cada N segundos
```

### Como atualiza

- **Refresh periódico** a cada `top-refresh-interval` segundos (default 60)
- **Invalidação imediata** sempre que um saldo muda (give/take/set/pay/reload)
- **TTL lazy** de 5 minutos como segurança
- **Filtro de bypass**: jogadores com `vixeconomias.<eco>.top.bypass` somem do top

### Menu de top

Configurável em `config.yml > menu.top`:
- Tamanho, slots dos players, items de cada slot
- Botões: anterior/próximo/fechar (anterior e próxima aparecem condicionalmente — não aparecem na primeira página se ela não estiver lotada, e não aparece "próxima" na última página)
- Item de fundo (filler) e item vazio
- Suporta `PLAYER_HEAD` (1.13+) ou `SKULL_ITEM:3` (1.8-1.12) — adapter cuida disso

### Cabeça do viewer (`viewer-head`)

Item especial dentro do menu mostrando o jogador que abriu (não os do ranking).
Exibe o nome, saldo na economia atual e a posição dele no top.

```yaml
menu:
  top:
    viewer-head:
      enabled: true
      slot: 4
      material: "PLAYER_HEAD"
      name: "&e{player}"
      lore:
        - "&7Saldo: &a{symbol}{value}"
        - "&7Sua posição: &a#{position}"
```

Placeholders disponíveis: `{player}`, `{economy}`, `{symbol}`, `{value}`, `{position}`.

Quando o jogador não está no top (bypass ou saldo zero), `{position}` vira o
valor em `placeholders.viewer-no-rank` (default `—`).

---

## 10. API para outros plugins

### 10.1 Visão geral da API

A API expõe **6 tipos principais** no pacote `org.vix.economias.api`:

| Tipo | Responsabilidade |
|---|---|
| `VixEconomiasAPI` | Accessor estático — ponto de entrada |
| `VixEconomiasService` | Interface principal (todos os métodos) |
| `EconomyHandle` | Snapshot read-only de uma economia |
| `TopEntry` | Entrada do ranking |
| `TransactionResult` | Enum de retorno das transações |
| `PlayerBalanceChangeEvent` | Evento Bukkit cancelável |

A API **não tem implementação dentro do jar dela** — só assinaturas. A implementação real está no `VixEconomias` do servidor.

### 10.2 Setup do projeto consumidor

Tem duas maneiras de adicionar a API no seu projeto:

#### Opção A — Jar local (mais simples)

1. Copie `VixEconomias-API-1.0.0.jar` pra pasta `libs/` do seu projeto
2. No `build.gradle`:

```gradle
repositories {
    mavenCentral()
    maven { url = 'https://hub.spigotmc.org/nexus/content/repositories/snapshots/' }
    flatDir { dirs 'libs' }
}

dependencies {
    compileOnly 'org.spigotmc:spigot-api:1.8.8-R0.1-SNAPSHOT'
    compileOnly name: 'VixEconomias-API-1.0.0'
}
```

#### Opção B — Maven local (zero atrito recorrente)

No projeto do VixEconomias, rode uma vez:
```
gradlew :api:publishToMavenLocal
```

No `build.gradle` do consumidor:
```gradle
repositories {
    mavenLocal()
}
dependencies {
    compileOnly 'org.vix:VixEconomias-API:1.0.0'
}
```

### 10.3 Configurar o `plugin.yml`

```yaml
name: MeuPlugin
version: 1.0
main: com.exemplo.MeuPlugin
api-version: 1.13   # opcional, omita se for 1.8

depend: [VixEconomias]
# OU
softdepend: [VixEconomias]
```

| Modo | Comportamento |
|---|---|
| `depend` | Seu plugin **não carrega** se VixEconomias não estiver no servidor |
| `softdepend` | Seu plugin carrega mesmo sem VixEconomias; você precisa checar `isAvailable()` antes de usar |

### 10.4 Padrão de inicialização

```java
package com.exemplo;

import org.bukkit.plugin.java.JavaPlugin;
import org.vix.economias.api.VixEconomiasAPI;
import org.vix.economias.api.VixEconomiasService;

public class MeuPlugin extends JavaPlugin {

    private VixEconomiasService eco;

    @Override
    public void onEnable() {
        if (!VixEconomiasAPI.isAvailable()) {
            getLogger().severe("VixEconomias não está disponível — desabilitando MeuPlugin");
            getServer().getPluginManager().disablePlugin(this);
            return;
        }
        eco = VixEconomiasAPI.get();
        getLogger().info("Integrado ao VixEconomias com sucesso");

        // Registra seus listeners, comandos, etc...
    }

    public VixEconomiasService eco() {
        return eco;
    }
}
```

### 10.5 Métodos da API em detalhe

#### `getEconomies() : Collection<EconomyHandle>`

Retorna todas as economias registradas. Coleção imutável.

```java
for (EconomyHandle e : eco.getEconomies()) {
    getLogger().info(e.getId() + " (" + e.getDisplayName() + "§r)");
}
```

#### `getEconomy(String id) : EconomyHandle`

Retorna uma economia pelo id. **Retorna `null`** se não existir. Case-insensitive.

```java
EconomyHandle coins = eco.getEconomy("coins");
if (coins == null) {
    getLogger().warning("Economia 'coins' não está configurada");
    return;
}
String formatado = coins.format(1234.56);  // "$1.234,56"
```

#### `hasEconomy(String id) : boolean`

Atalho pra `getEconomy(id) != null`.

#### `getBalance(UUID player, String economyId) : CompletableFuture<Double>`

Retorna o saldo atual. Carrega do banco se o jogador estiver offline. Se a economia não existir ou o UUID for desconhecido, retorna `0.0`.

```java
eco.getBalance(player.getUniqueId(), "coins").thenAccept(saldo -> {
    player.sendMessage("§aSeu saldo: §f$" + saldo);
});
```

#### `deposit(UUID player, String economyId, double amount) : CompletableFuture<TransactionResult>`

Adiciona ao saldo. `amount` deve ser positivo (> 0). Dispara `PlayerBalanceChangeEvent` com `Cause.DEPOSIT` **antes** de persistir.

```java
eco.deposit(player.getUniqueId(), "coins", 500.0).thenAccept(result -> {
    switch (result) {
        case SUCCESS:     player.sendMessage("§a+§f500 coins"); break;
        case MAX_REACHED: player.sendMessage("§cVocê atingiu o saldo máximo"); break;
        case CANCELLED:   player.sendMessage("§cOperação cancelada"); break;
        default:          player.sendMessage("§cErro: " + result);
    }
});
```

#### `withdraw(UUID player, String economyId, double amount) : CompletableFuture<TransactionResult>`

Remove do saldo. `amount` deve ser positivo. Falha com `INSUFFICIENT` se saldo < amount (a menos que `allow-negative: true` na config). Cause: `WITHDRAW`.

```java
eco.withdraw(buyer.getUniqueId(), "coins", 100.0).thenAccept(result -> {
    if (result == TransactionResult.SUCCESS) {
        // Pagamento OK — entregue o item
        Bukkit.getScheduler().runTask(plugin, () -> {
            buyer.getInventory().addItem(new ItemStack(Material.DIAMOND));
            buyer.sendMessage("§aDiamante comprado por 100 coins");
        });
    } else if (result == TransactionResult.INSUFFICIENT) {
        buyer.sendMessage("§cVocê não tem 100 coins");
    }
});
```

#### `set(UUID player, String economyId, double amount) : CompletableFuture<TransactionResult>`

Define o saldo absoluto (clamp aplicado a `min-balance`/`max-balance`). Cause: `SET`.

```java
eco.set(player.getUniqueId(), "coins", 0).thenAccept(r -> {
    player.sendMessage("§cSeu saldo foi zerado");
});
```

#### `transfer(UUID from, UUID to, String economyId, double amount) : CompletableFuture<TransactionResult>`

Withdraw em `from` + deposit em `to`. Se o deposit falhar, faz **rollback** automático (devolve o valor pro `from`). Dispara 2 eventos: `Cause.PAY_SEND` no `from` e `Cause.PAY_RECEIVE` no `to`.

```java
eco.transfer(alice, bob, "coins", 250.0).thenAccept(result -> {
    if (result == TransactionResult.SUCCESS) {
        getLogger().info("Transfer OK");
    } else if (result == TransactionResult.INSUFFICIENT) {
        // Saldo da Alice insuficiente
    } else if (result == TransactionResult.MAX_REACHED) {
        // Bob ia estourar o max; saldo da Alice foi restaurado pelo rollback
    }
});
```

#### `getTop(String economyId, int limit) : CompletableFuture<List<TopEntry>>`

Retorna o top da economia. `limit = -1` = todos. Já filtra os jogadores com `vixeconomias.<eco>.top.bypass`.

```java
eco.getTop("coins", 10).thenAccept(top -> {
    Bukkit.getScheduler().runTask(plugin, () -> {
        int pos = 1;
        for (TopEntry entry : top) {
            Bukkit.broadcastMessage("§7#" + pos + " §f" + entry.getName() + " §8- §a$" + entry.getValue());
            pos++;
        }
    });
});
```

#### `EconomyHandle.format(double value) : String`

Formata um número segundo as regras dessa economia (DECIMAL ou LETTER). **Não** inclui o símbolo — concatene com `getSymbol()` se precisar.

```java
EconomyHandle coins = eco.getEconomy("coins");
String s1 = coins.format(1234.56);              // "1.234,56"
String s2 = coins.getSymbol() + s1;             // "$1.234,56"

EconomyHandle gems = eco.getEconomy("gems");    // formato LETTER
String s3 = gems.format(15_234);                // "15.23K"
String s4 = gems.format(1_500_000);             // "1.5M"
```

### 10.6 Cookbook — receitas práticas

#### Receita 1: Comprar um item da loja

```java
public void comprarItem(Player p, double preco, ItemStack item, String economyId) {
    eco.withdraw(p.getUniqueId(), economyId, preco).thenAccept(result -> {
        Bukkit.getScheduler().runTask(this, () -> {
            if (result == TransactionResult.SUCCESS) {
                p.getInventory().addItem(item);
                p.sendMessage("§aCompra realizada por §f" + preco);
            } else if (result == TransactionResult.INSUFFICIENT) {
                p.sendMessage("§cSaldo insuficiente.");
            } else {
                p.sendMessage("§cErro na compra: " + result);
            }
        });
    });
}
```

#### Receita 2: Recompensa por matar mob (sem await)

```java
@EventHandler
public void onMobKill(EntityDeathEvent event) {
    Player killer = event.getEntity().getKiller();
    if (killer == null) return;
    double reward = calcularRecompensa(event.getEntityType());
    if (reward <= 0) return;

    eco.deposit(killer.getUniqueId(), "coins", reward).thenAccept(result -> {
        if (result == TransactionResult.SUCCESS) {
            // Não precisa voltar pra main thread — sendMessage é thread-safe.
            killer.sendMessage("§a+§f" + reward + " coins por matar " + event.getEntityType());
        }
    });
}
```

#### Receita 3: Verificar saldo antes de abrir um menu

```java
public void abrirLojaSeTem500Coins(Player p) {
    eco.getBalance(p.getUniqueId(), "coins").thenAccept(saldo -> {
        Bukkit.getScheduler().runTask(this, () -> {
            if (saldo >= 500) {
                abrirLoja(p);
            } else {
                p.sendMessage("§cPrecisa de pelo menos 500 coins pra entrar na loja");
            }
        });
    });
}
```

#### Receita 4: Transferência entre players com confirmação

```java
public void transferir(Player from, Player to, double amount) {
    eco.transfer(from.getUniqueId(), to.getUniqueId(), "coins", amount)
       .thenAccept(result -> Bukkit.getScheduler().runTask(this, () -> {
           switch (result) {
               case SUCCESS:
                   from.sendMessage("§aEnviou §f" + amount + " §apara " + to.getName());
                   to.sendMessage("§aRecebeu §f" + amount + " §ade " + from.getName());
                   break;
               case INSUFFICIENT:
                   from.sendMessage("§cSaldo insuficiente");
                   break;
               case MAX_REACHED:
                   from.sendMessage("§cO destinatário está com saldo cheio");
                   break;
               default:
                   from.sendMessage("§cErro: " + result);
           }
       }));
}
```

#### Receita 5: Adicionar saldo a TODOS os jogadores online

```java
public void bonusGlobal(double valor) {
    for (Player p : Bukkit.getOnlinePlayers()) {
        eco.deposit(p.getUniqueId(), "coins", valor);
    }
    Bukkit.broadcastMessage("§aTodos receberam §f" + valor + " coins de bônus!");
}
```

#### Receita 6: Top 3 do servidor numa scoreboard

```java
public void atualizarScoreboardTop() {
    eco.getTop("coins", 3).thenAccept(top -> Bukkit.getScheduler().runTask(this, () -> {
        for (Player p : Bukkit.getOnlinePlayers()) {
            Scoreboard sb = p.getScoreboard();
            // ...preenche scoreboard com os 3 primeiros entries
            int pos = 1;
            for (TopEntry e : top) {
                String line = "§7#" + pos + " §f" + e.getName() + " §a" + e.getValue();
                // adiciona à scoreboard
                pos++;
            }
        }
    }));
}
```

#### Receita 7: Anti-cheat — bloquear depósitos suspeitos

```java
@EventHandler
public void onChange(PlayerBalanceChangeEvent event) {
    if (event.getCause() == PlayerBalanceChangeEvent.Cause.DEPOSIT
        && event.getDelta() > 10_000_000) {
        event.setCancelled(true);
        getLogger().warning("Depósito suspeito bloqueado: " +
            event.getPlayer() + " +" + event.getDelta() + " " + event.getEconomyId());
    }
}
```

#### Receita 8: Logging de transações importantes

```java
@EventHandler(priority = EventPriority.MONITOR, ignoreCancelled = true)
public void onTransaction(PlayerBalanceChangeEvent event) {
    if (event.getCause() == Cause.PAY_SEND) {
        FileLogger.log("[PAY] " + event.getPlayer() + " enviou " +
            Math.abs(event.getDelta()) + " " + event.getEconomyId());
    }
}
```

#### Receita 9: Integração com Vault (compat com plugins genéricos)

Pra plugins que falam só Vault (ChestShop, GriefPrevention, etc.), você pode criar um `Economy` adapter que delega pro VixEconomias:

```java
public class VixVaultAdapter implements net.milkbowl.vault.economy.Economy {
    private final VixEconomiasService eco;
    private final String economyId;  // ex: "coins"

    public VixVaultAdapter(VixEconomiasService eco, String economyId) {
        this.eco = eco;
        this.economyId = economyId;
    }

    @Override
    public double getBalance(OfflinePlayer player) {
        // ATENÇÃO: Vault espera retorno sync — use getNow ou cache
        try {
            return eco.getBalance(player.getUniqueId(), economyId).getNow(0.0);
        } catch (Throwable t) { return 0.0; }
    }

    @Override
    public EconomyResponse depositPlayer(OfflinePlayer player, double amount) {
        TransactionResult r = eco.deposit(player.getUniqueId(), economyId, amount).join();
        // ... mapear pra EconomyResponse
        return new EconomyResponse(amount, getBalance(player),
            r == TransactionResult.SUCCESS ? ResponseType.SUCCESS : ResponseType.FAILURE,
            r.name());
    }
    // ... outros métodos
}
```
(Cuidado: Vault tem API síncrona, o que não casa bem com a API async do VixEconomias. Use só se realmente precisar de compat.)

### 10.7 Pitfalls comuns

#### ❌ Bloquear a main thread

```java
// NUNCA faça isso:
double saldo = eco.getBalance(uuid, "coins").join();  // trava o servidor!
```

```java
// Sempre encadeie:
eco.getBalance(uuid, "coins").thenAccept(saldo -> {
    // ...
});
```

#### ❌ Mexer em Bukkit dentro de callback async

```java
// PROBLEMA: thenAccept executa em thread async
eco.getBalance(uuid, "coins").thenAccept(saldo -> {
    player.getInventory().addItem(item);  // ❌ chamada de inventário fora da main thread
});
```

```java
// CORRETO: agendar pra main thread
eco.getBalance(uuid, "coins").thenAccept(saldo -> {
    Bukkit.getScheduler().runTask(plugin, () -> {
        player.getInventory().addItem(item);  // ✓
    });
});
```

**Exceção**: `Player.sendMessage(String)` e leituras simples (getName, getUniqueId) são thread-safe em Spigot/Paper.

#### ❌ Cachear o `VixEconomiasService` permanentemente

```java
// Risco: se o VixEconomias der reload, esse handle pode ficar stale
public class MeuPlugin {
    private static final VixEconomiasService ECO = VixEconomiasAPI.get();
    // ...
}
```

```java
// MELHOR: pega quando precisa
VixEconomiasService eco = VixEconomiasAPI.get();
eco.deposit(...);
```

#### ❌ Ignorar `TransactionResult`

```java
// Você não sabe se a transação deu certo:
eco.deposit(uuid, "coins", 100.0);
```

```java
// Sempre cheque o resultado:
eco.deposit(uuid, "coins", 100.0).thenAccept(result -> {
    if (result != TransactionResult.SUCCESS) {
        getLogger().warning("Deposit falhou: " + result);
    }
});
```

#### ❌ Esquecer de checar `isAvailable()` em softdepend

```java
// Se o VixEconomias não estiver instalado, NPE:
VixEconomiasAPI.get().deposit(uuid, "coins", 100);
```

```java
// Sempre cheque (relevante só se for softdepend):
if (VixEconomiasAPI.isAvailable()) {
    VixEconomiasAPI.get().deposit(uuid, "coins", 100);
}
```

### 10.8 TransactionResult — tabela de referência

| Valor | Significado | Quando acontece |
|---|---|---|
| `SUCCESS` | Transação OK | Saldo foi modificado e persistido |
| `INSUFFICIENT` | Saldo insuficiente | Em withdraw/transfer, quando saldo < amount e `allow-negative: false` |
| `MAX_REACHED` | Limite máximo atingido | Em deposit, quando saldo + amount > `max-balance` |
| `INVALID_AMOUNT` | Quantia inválida | amount ≤ 0, NaN ou infinito |
| `UNKNOWN_PLAYER` | UUID desconhecido | Player nunca entrou no servidor / banco corrompido |
| `UNKNOWN_ECONOMY` | Economy id inválido | ID não está em `economies.yml` |
| `CANCELLED` | Cancelado por listener | Algum plugin chamou `PlayerBalanceChangeEvent.setCancelled(true)` |
| `ERROR` | Erro inesperado | Banco indisponível, exceção JDBC, etc. |

### 10.9 Thread-safety da API

| Componente | Thread-safety |
|---|---|
| `VixEconomiasAPI.get/isAvailable/set` | Sim (volatile) |
| `VixEconomiasService.*` (deposit/withdraw/etc.) | Sim — todos retornam `CompletableFuture` |
| `EconomyHandle.*` getters | Sim — imutável depois de construído |
| `TopEntry.*` getters | Sim — imutável |
| `PlayerBalanceChangeEvent` getters/setters | Não — listeners não devem modificar fora do thread do evento |
| Acessos concorrentes a saldos | Serializados internamente por UUID |

---

## 11. Eventos

### `PlayerBalanceChangeEvent`

Disparado **antes** de cada transação ser persistida. Cancelável.

```java
import org.vix.economias.api.event.PlayerBalanceChangeEvent;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;

public class MeuListener implements Listener {

    @EventHandler
    public void onBalance(PlayerBalanceChangeEvent event) {
        UUID player = event.getPlayer();
        String eco = event.getEconomyId();
        double from = event.getOldBalance();
        double to = event.getNewBalance();
        double delta = event.getDelta();
        Cause cause = event.getCause();  // DEPOSIT, WITHDRAW, SET, PAY_SEND, PAY_RECEIVE

        // Bloquear transação:
        if (delta > 1_000_000) event.setCancelled(true);
    }
}
```

### Causes

| Cause | Quando |
|---|---|
| `DEPOSIT` | `eco.deposit()` ou `/<economia> give` |
| `WITHDRAW` | `eco.withdraw()` ou `/<economia> remove` |
| `SET` | `eco.set()` ou `/<economia> setar` |
| `PAY_SEND` | Lado do remetente em `/<economia> pay` ou `transfer()` |
| `PAY_RECEIVE` | Lado do destinatário em `/<economia> pay` ou `transfer()` |

---

## 12. Compatibilidade entre versões

Plugin testado de **1.8.X** até **1.21.X**.

### O que se adapta automaticamente
- **Materials** no menu (`MaterialAdapter`): `PLAYER_HEAD` ↔ `SKULL_ITEM:3`, `BLACK_STAINED_GLASS_PANE` ↔ `STAINED_GLASS_PANE:15`, etc.
- **Hex colors** (`&#RRGGBB`): renderizados em 1.16+, removidos do output em 1.8-1.15 (pra não poluir o chat)
- **`SkullMeta.setOwningPlayer`** ↔ `setOwner(String)` (reflection)

### O que sempre funciona (todas as versões)
- Códigos de cor clássicos: `&a`, `&l`, `&r`, etc.
- Comandos, eventos, listeners, scheduler
- API JDBC (SQLite/MySQL)
- Redis (Jedis 5.x)
- PlaceholderAPI hook

### Limitações do cliente (não dão pra contornar no plugin)
- Cliente 1.8 não renderiza hex/gradient — limitação do protocolo
- Cliente 1.8 não tem hover/click rich text
- Custom model data: só 1.14+

---

## Suporte

Em caso de erro, verifique o `console` do servidor. As mensagens importantes começam com `[VixEconomias]`. Erros comuns:

- **Driver SQLite/MySQL não encontrado**: instale o jar do driver no servidor
- **Falha ao conectar Redis**: cheque `host/port/password` no `config.yml`
- **`PlaceholderAPI` não funciona**: confirme que o plugin está instalado e habilitado
- **Save falha repetidamente**: o `dirty` é mantido — próximo auto-save retenta
