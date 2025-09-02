# Project: EssentialsLite (Paper/Spigot)
# Java 17 • Paper API 1.21.1 • Gradle

# ──────────────────────────────────────────────────────────────────────────────
# settings.gradle
# ──────────────────────────────────────────────────────────────────────────────
rootProject.name = 'EssentialsLite'

# ──────────────────────────────────────────────────────────────────────────────
# build.gradle (Groovy)
# ──────────────────────────────────────────────────────────────────────────────
plugins {
    id 'java'
}

group = 'com.codecopilot.essentials'
version = '1.0.0'

def javaVersion = 17

repositories {
    mavenCentral()
    maven { url = 'https://repo.papermc.io/repository/maven-public/' }
}

dependencies {
    compileOnly 'io.papermc.paper:paper-api:1.21.1-R0.1-SNAPSHOT'
}

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(javaVersion)
    }
}

tasks.withType(JavaCompile).configureEach {
    options.encoding = 'UTF-8'
}

tasks.register('printJarPath') {
    doLast {
        def jarFile = tasks.jar.archiveFile.get().asFile
        println "Built JAR: ${jarFile.absolutePath}"
    }
}

# ──────────────────────────────────────────────────────────────────────────────
# src/main/resources/plugin.yml
# ──────────────────────────────────────────────────────────────────────────────
name: EssentialsLite
main: com.codecopilot.essentials.EssentialsLitePlugin
version: 1.0.0
api-version: '1.20'
authors: [Code Copilot]
description: "Lightweight essentials: heal, feed, spawn, setspawn, welcome message"
website: https://example.com

commands:
  heal:
    description: Heal yourself or another player
    usage: "/heal [player]"
    permission: essentialslite.heal
    aliases: [h]
  feed:
    description: Feed yourself or another player
    usage: "/feed [player]"
    permission: essentialslite.feed
    aliases: [f]
  spawn:
    description: Teleport to server spawn
    usage: "/spawn"
    permission: essentialslite.spawn
  setspawn:
    description: Set server spawn to your current position
    usage: "/setspawn"
    permission: essentialslite.setspawn

permissions:
  essentialslite.heal:
    description: Use /heal on self
    default: op
  essentialslite.heal.others:
    description: Use /heal on others
    default: op
  essentialslite.feed:
    description: Use /feed on self
    default: op
  essentialslite.feed.others:
    description: Use /feed on others
    default: op
  essentialslite.spawn:
    description: Use /spawn
    default: true
  essentialslite.setspawn:
    description: Use /setspawn
    default: op

# ──────────────────────────────────────────────────────────────────────────────
# src/main/resources/config.yml
# ──────────────────────────────────────────────────────────────────────────────
messages:
  prefix: "&7[&aEssentialsLite&7]&r "
  no-permission: "&cYou don't have permission to do that."
  player-only: "&cOnly players can use this command."
  player-not-found: "&cPlayer not found: &f{target}"
  healed-self: "&aYou have been healed."
  healed-other: "&aYou healed &f{target}&a."
  fed-self: "&aYou are no longer hungry."
  fed-other: "&aYou fed &f{target}&a."
  teleporting-to-spawn: "&aTeleporting to spawn..."
  spawn-not-set: "&eSpawn is not set, using world spawn."
  spawn-set: "&aServer spawn set to your current location."
  join: "&aWelcome, &f{player}&a!"

# Optional predefined spawn. If unset, plugin will fall back to world spawn.
spawn:
  world: ""
  x: 0.0
  y: 0.0
  z: 0.0
  yaw: 0.0
  pitch: 0.0

# ──────────────────────────────────────────────────────────────────────────────
# src/main/java/com/codecopilot/essentials/EssentialsLitePlugin.java
# ──────────────────────────────────────────────────────────────────────────────
package com.codecopilot.essentials;

import com.codecopilot.essentials.commands.FeedCommand;
import com.codecopilot.essentials.commands.HealCommand;
import com.codecopilot.essentials.commands.SetSpawnCommand;
import com.codecopilot.essentials.commands.SpawnCommand;
import com.codecopilot.essentials.listeners.JoinListener;
import org.bukkit.*;
import org.bukkit.configuration.file.FileConfiguration;
import org.bukkit.configuration.file.YamlConfiguration;
import org.bukkit.entity.Player;
import org.bukkit.location.Location;
import org.bukkit.plugin.java.JavaPlugin;

import java.io.File;
import java.util.Objects;
import java.util.logging.Logger;

/**
 * EssentialsLite — minimal quality-of-life commands.
 *
 * Why: Keep a tiny, readable baseline plugin you can extend safely.
 */
public final class EssentialsLitePlugin extends JavaPlugin {
    public static final String PERM_HEAL = "essentialslite.heal";
    public static final String PERM_HEAL_OTHERS = "essentialslite.heal.others";
    public static final String PERM_FEED = "essentialslite.feed";
    public static final String PERM_FEED_OTHERS = "essentialslite.feed.others";
    public static final String PERM_SPAWN = "essentialslite.spawn";
    public static final String PERM_SETSPAWN = "essentialslite.setspawn";

    private Logger log;
    private Location spawnLocation; // nullable when not configured

    @Override
    public void onEnable() {
        this.log = getLogger();
        saveDefaultConfig();
        loadSpawnFromConfig();

        Objects.requireNonNull(getCommand("heal")).setExecutor(new HealCommand(this));
        Objects.requireNonNull(getCommand("feed")).setExecutor(new FeedCommand(this));
        Objects.requireNonNull(getCommand("spawn")).setExecutor(new SpawnCommand(this));
        Objects.requireNonNull(getCommand("setspawn")).setExecutor(new SetSpawnCommand(this));

        getServer().getPluginManager().registerEvents(new JoinListener(this), this);
        log.info("EssentialsLite enabled.");
    }

    @Override
    public void onDisable() {
        log.info("EssentialsLite disabled.");
    }

    public Logger logger() {
        return log;
    }

    public Location getConfiguredSpawnOrNull() {
        return spawnLocation;
    }

    public Location resolveSpawnFor(Player player) {
        if (spawnLocation != null) return spawnLocation;
        // Why: World spawn is a sane fallback if custom spawn isn't set.
        return player.getWorld().getSpawnLocation();
    }

    public void setAndSaveSpawn(Location loc) {
        this.spawnLocation = loc.clone();
        serializeSpawnToConfig(loc);
        saveConfig();
    }

    public String colorize(String input) {
        return ChatColor.translateAlternateColorCodes('&', input);
    }

    public String msg(String key) {
        String prefix = getConfig().getString("messages.prefix", "");
        String body = getConfig().getString("messages." + key, key);
        return colorize(prefix + body);
    }

    private void loadSpawnFromConfig() {
        FileConfiguration cfg = getConfig();
        String worldName = cfg.getString("spawn.world", "");
        if (worldName == null || worldName.isEmpty()) {
            spawnLocation = null;
            return;
        }
        World w = Bukkit.getWorld(worldName);
        if (w == null) {
            log.warning("Configured spawn world not found: " + worldName);
            spawnLocation = null;
            return;
        }
        double x = cfg.getDouble("spawn.x");
        double y = cfg.getDouble("spawn.y");
        double z = cfg.getDouble("spawn.z");
        float yaw = (float) cfg.getDouble("spawn.yaw");
        float pitch = (float) cfg.getDouble("spawn.pitch");
        spawnLocation = new Location(w, x, y, z, yaw, pitch);
    }

    private void serializeSpawnToConfig(Location loc) {
        FileConfiguration cfg = getConfig();
        cfg.set("spawn.world", loc.getWorld() != null ? loc.getWorld().getName() : null);
        cfg.set("spawn.x", loc.getX());
        cfg.set("spawn.y", loc.getY());
        cfg.set("spawn.z", loc.getZ());
        cfg.set("spawn.yaw", loc.getYaw());
        cfg.set("spawn.pitch", loc.getPitch());
    }
}

# ──────────────────────────────────────────────────────────────────────────────
# src/main/java/com/codecopilot/essentials/commands/HealCommand.java
# ──────────────────────────────────────────────────────────────────────────────
package com.codecopilot.essentials.commands;

import com.codecopilot.essentials.EssentialsLitePlugin;
import org.bukkit.Bukkit;
import org.bukkit.command.Command;
import org.bukkit.command.CommandExecutor;
import org.bukkit.command.CommandSender;
import org.bukkit.entity.Player;
import org.bukkit.potion.PotionEffect;

/** Why: Separate command classes keep the main plugin lean and testable. */
public class HealCommand implements CommandExecutor {
    private final EssentialsLitePlugin plugin;

    public HealCommand(EssentialsLitePlugin plugin) {
        this.plugin = plugin;
    }

    @Override
    public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {
        if (args.length == 0) {
            if (!(sender instanceof Player player)) {
                sender.sendMessage(plugin.msg("player-only"));
                return true;
            }
            if (!player.hasPermission(EssentialsLitePlugin.PERM_HEAL)) {
                player.sendMessage(plugin.msg("no-permission"));
                return true;
            }
            heal(player);
            player.sendMessage(plugin.msg("healed-self"));
            return true;
        }

        String targetName = args[0];
        Player target = Bukkit.getPlayerExact(targetName);
        if (target == null) {
            sender.sendMessage(plugin.msg("player-not-found").replace("{target}", targetName));
            return true;
        }
        if (!sender.hasPermission(EssentialsLitePlugin.PERM_HEAL_OTHERS)) {
            sender.sendMessage(plugin.msg("no-permission"));
            return true;
        }
        heal(target);
        sender.sendMessage(plugin.msg("healed-other").replace("{target}", target.getName()));
        if (sender != target) {
            target.sendMessage(plugin.msg("healed-self"));
        }
        return true;
    }

    private void heal(Player p) {
        p.setHealth(p.getMaxHealth());
        p.setFoodLevel(20);
        p.setSaturation(20f);
        p.setFireTicks(0);
        p.setRemainingAir(p.getMaximumAir());
        for (PotionEffect effect : p.getActivePotionEffects()) {
            p.removePotionEffect(effect.getType());
        }
    }
}

# ──────────────────────────────────────────────────────────────────────────────
# src/main/java/com/codecopilot/essentials/commands/FeedCommand.java
# ──────────────────────────────────────────────────────────────────────────────
package com.codecopilot.essentials.commands;

import com.codecopilot.essentials.EssentialsLitePlugin;
import org.bukkit.Bukkit;
import org.bukkit.command.Command;
import org.bukkit.command.CommandExecutor;
import org.bukkit.command.CommandSender;
import org.bukkit.entity.Player;

public class FeedCommand implements CommandExecutor {
    private final EssentialsLitePlugin plugin;

    public FeedCommand(EssentialsLitePlugin plugin) {
        this.plugin = plugin;
    }

    @Override
    public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {
        if (args.length == 0) {
            if (!(sender instanceof Player player)) {
                sender.sendMessage(plugin.msg("player-only"));
                return true;
            }
            if (!player.hasPermission(EssentialsLitePlugin.PERM_FEED)) {
                player.sendMessage(plugin.msg("no-permission"));
                return true;
            }
            feed(player);
            player.sendMessage(plugin.msg("fed-self"));
            return true;
        }

        String targetName = args[0];
        Player target = Bukkit.getPlayerExact(targetName);
        if (target == null) {
            sender.sendMessage(plugin.msg("player-not-found").replace("{target}", targetName));
            return true;
        }
        if (!sender.hasPermission(EssentialsLitePlugin.PERM_FEED_OTHERS)) {
            sender.sendMessage(plugin.msg("no-permission"));
            return true;
        }
        feed(target);
        sender.sendMessage(plugin.msg("fed-other").replace("{target}", target.getName()));
        if (sender != target) {
            target.sendMessage(plugin.msg("fed-self"));
        }
        return true;
    }

    private void feed(Player p) {
        p.setFoodLevel(20);
        p.setSaturation(20f);
        p.setExhaustion(0f);
    }
}

# ──────────────────────────────────────────────────────────────────────────────
# src/main/java/com/codecopilot/essentials/commands/SpawnCommand.java
# ──────────────────────────────────────────────────────────────────────────────
package com.codecopilot.essentials.commands;

import com.codecopilot.essentials.EssentialsLitePlugin;
import org.bukkit.command.Command;
import org.bukkit.command.CommandExecutor;
import org.bukkit.command.CommandSender;
import org.bukkit.entity.Player;

public class SpawnCommand implements CommandExecutor {
    private final EssentialsLitePlugin plugin;

    public SpawnCommand(EssentialsLitePlugin plugin) {
        this.plugin = plugin;
    }

    @Override
    public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {
        if (!(sender instanceof Player player)) {
            sender.sendMessage(plugin.msg("player-only"));
            return true;
        }
        if (!player.hasPermission(EssentialsLitePlugin.PERM_SPAWN)) {
            player.sendMessage(plugin.msg("no-permission"));
            return true;
        }
        var configured = plugin.getConfiguredSpawnOrNull();
        if (configured == null) {
            player.sendMessage(plugin.msg("spawn-not-set"));
        } else {
            player.sendMessage(plugin.msg("teleporting-to-spawn"));
        }
        player.teleport(plugin.resolveSpawnFor(player));
        return true;
    }
}

# ──────────────────────────────────────────────────────────────────────────────
# src/main/java/com/codecopilot/essentials/commands/SetSpawnCommand.java
# ──────────────────────────────────────────────────────────────────────────────
package com.codecopilot.essentials.commands;

import com.codecopilot.essentials.EssentialsLitePlugin;
import org.bukkit.command.Command;
import org.bukkit.command.CommandExecutor;
import org.bukkit.command.CommandSender;
import org.bukkit.entity.Player;

public class SetSpawnCommand implements CommandExecutor {
    private final EssentialsLitePlugin plugin;

    public SetSpawnCommand(EssentialsLitePlugin plugin) {
        this.plugin = plugin;
    }

    @Override
    public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {
        if (!(sender instanceof Player player)) {
            sender.sendMessage(plugin.msg("player-only"));
            return true;
        }
        if (!player.hasPermission(EssentialsLitePlugin.PERM_SETSPAWN)) {
            player.sendMessage(plugin.msg("no-permission"));
            return true;
        }
        plugin.setAndSaveSpawn(player.getLocation());
        player.sendMessage(plugin.msg("spawn-set"));
        return true;
    }
}

# ──────────────────────────────────────────────────────────────────────────────
# src/main/java/com/codecopilot/essentials/listeners/JoinListener.java
# ──────────────────────────────────────────────────────────────────────────────
package com.codecopilot.essentials.listeners;

import com.codecopilot.essentials.EssentialsLitePlugin;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.player.PlayerJoinEvent;

public class JoinListener implements Listener {
    private final EssentialsLitePlugin plugin;

    public JoinListener(EssentialsLitePlugin plugin) {
        this.plugin = plugin;
    }

    @EventHandler
    public void onJoin(PlayerJoinEvent e) {
        String raw = plugin.getConfig().getString("messages.join", "");
        if (raw == null || raw.isBlank()) return;
        String msg = plugin.colorize(raw.replace("{player}", e.getPlayer().getName()));
        e.getPlayer().sendMessage(msg);
    }
}

# ──────────────────────────────────────────────────────────────────────────────
# HOW TO BUILD & INSTALL
# ──────────────────────────────────────────────────────────────────────────────
# 1) Ensure Java 17 is installed.
# 2) From the project root:
#    Option A — Using local Gradle (if you have it):
#       gradle wrapper --gradle-version 8.8
#       ./gradlew build   (Windows: gradlew.bat build)
#    Option B — Using IntelliJ IDEA:
#       Open project → Import Gradle project → run Gradle task "build".
#    Option C — GitHub Actions (no local build needed):
#       Commit this project to a new GitHub repo and include the workflow below.
#       Push → go to Actions → latest run → download JAR artifact.
# 3) Copy build/libs/EssentialsLite-1.0.0.jar into your server's plugins/ folder.
# 4) Start server. Config will be generated at plugins/EssentialsLite/config.yml.
# 5) Edit messages/permissions as needed, then /reload confirm or restart.

# ──────────────────────────────────────────────────────────────────────────────
# .github/workflows/build.yml (GitHub Actions — upload JAR artifact)
# ──────────────────────────────────────────────────────────────────────────────
name: build
on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  jar:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Temurin JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - name: Cache Gradle
        uses: gradle/actions/setup-gradle@v3
      - name: Build with Gradle
        run: |
          gradle wrapper --gradle-version 8.8
          ./gradlew --no-daemon clean build -x test
      - name: Upload JAR artifact
        uses: actions/upload-artifact@v4
        with:
          name: EssentialsLite-jar
          path: build/libs/*.jar
