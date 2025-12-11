# TranslateMC Plugin - Java Developer API Documentation

---

## Overview

The TranslateMC Plugin API enables developers to integrate translation functionality into their own plugins. Access translations and manage player languages through a simple, clean API.

---

## Maven Setup

### Add Repository

```xml
<repository>
    <id>steyon-repo</id>
    <url>https://repo.steyon.dev/</url>
</repository>
```

### Add Dependency

```xml
<dependency>
    <groupId>dev.steyon</groupId>
    <artifactId>TranslateMC-Plugin</artifactId>
    <version>1.0.0</version>
    <scope>provided</scope>
</dependency>
```

---

## Quick Start

```java
// Get plugin instance
TranslateMCPlugin plugin = TranslateMCPlugin.getInstance();
TranslationManager manager = plugin.getTranslationManager();

// Get translation for a player
String greeting = manager.getTranslation(player, "greeting");
player.sendMessage(greeting);

// Set player language
manager.setPlayerLanguage(player, "de");

// Get available languages
List<String> codes = manager.getAvailableLanguageCodes();
```

---

## Main API

### TranslateMCPlugin

Entry point to access the plugin and manager.

```java
public static TranslateMCPlugin getInstance()
```
Returns the singleton plugin instance.

```java
public TranslationManager getTranslationManager()
```
Returns the translation manager for all operations.

---

### TranslationManager

Core class for translations and player language management.

#### Get Translations

```java
public String getTranslation(Player player, String key)
```
**Gets translation in the player's language**
- Returns the translated text or the key if not found
- Automatically falls back to default language if translation missing

```java
String greeting = manager.getTranslation(player, "greeting");
player.sendMessage(greeting);
```

```java
public String getTranslation(String key, String languageCode)
```
**Gets translation for a specific language**
- `key`: The translation key (e.g., "greeting")
- `languageCode`: Language code (e.g., "de", "en", "fr")

```java
String germanGreeting = manager.getTranslation("greeting", "de");
```

#### Player Language Management

```java
public void setPlayerLanguage(Player player, String languageCode)
```
**Sets the player's language preference**

```java
manager.setPlayerLanguage(player, "de");
manager.setPlayerLanguage(player, "en");
```

```java
public String getPlayerLanguage(Player player)
```
**Gets the player's currently selected language**

```java
String playerLang = manager.getPlayerLanguage(player);
```

#### Available Languages

```java
public List<String> getAvailableLanguageCodes()
```
**Returns all available language codes**

```java
List<String> codes = manager.getAvailableLanguageCodes();
// Returns: ["en", "de", "fr", "es", ...]

for (String code : codes) {
    System.out.println(code);
}
```

```java
public List<TranslationAPI.Language> getAvailableLanguages()
```
**Returns all available languages with details**

```java
List<TranslationAPI.Language> languages = manager.getAvailableLanguages();
for (TranslationAPI.Language lang : languages) {
    System.out.println(lang.getName() + " (" + lang.getCode() + ")");
}
```

```java
public boolean isLanguageAvailable(String languageCode)
```
**Checks if a language is available**

```java
if (manager.isLanguageAvailable("de")) {
    System.out.println("German is available");
}
```

```java
public TranslationAPI.Language getLanguage(String languageCode)
```
**Gets language details by code**

```java
TranslationAPI.Language german = manager.getLanguage("de");
if (german != null) {
    System.out.println(german.getName()); // "Deutsch"
}
```

---

### TranslationAPI.Language

Represents a language with metadata.

```java
public String getCode()
```
The language code (e.g., "de", "en", "fr")

```java
public String getName()
```
The display name (e.g., "Deutsch", "English", "Français")

```java
public boolean isSource()
```
Returns true if this is the default/source language

```java
TranslationAPI.Language lang = manager.getLanguage("de");
if (lang != null) {
    System.out.println(lang.getCode());      // "de"
    System.out.println(lang.getName());      // "Deutsch"
    System.out.println(lang.isSource());     // false
}
```

---

## Practical Examples

### Example 1: Translate Command Handler

```java
import org.bukkit.command.Command;
import org.bukkit.command.CommandExecutor;
import org.bukkit.command.CommandSender;
import org.bukkit.entity.Player;
import dev.steyon.translateMCPlugin.TranslateMCPlugin;

public class GreetCommand implements CommandExecutor {
    
    @Override
    public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {
        if (!(sender instanceof Player)) return false;
        
        Player player = (Player) sender;
        TranslateMCPlugin plugin = TranslateMCPlugin.getInstance();
        
        // Get translation in player's language
        String greeting = plugin.getTranslationManager()
            .getTranslation(player, "greeting");
        
        player.sendMessage(greeting);
        return true;
    }
}
```

### Example 2: Language Selection Command

```java
public class LangCommand implements CommandExecutor {
    
    @Override
    public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {
        if (!(sender instanceof Player)) return false;
        if (args.length == 0) return false;
        
        Player player = (Player) sender;
        TranslateMCPlugin plugin = TranslateMCPlugin.getInstance();
        TranslationManager manager = plugin.getTranslationManager();
        
        String languageCode = args[0].toLowerCase();
        
        // Check if language exists
        if (!manager.isLanguageAvailable(languageCode)) {
            player.sendMessage("Language not found!");
            return true;
        }
        
        // Set language
        manager.setPlayerLanguage(player, languageCode);
        
        // Get language name for confirmation
        String langName = manager.getLanguage(languageCode).getName();
        String message = manager.getTranslation(player, "language.changed");
        message = message.replace("{language}", langName);
        
        player.sendMessage(message);
        return true;
    }
}
```

### Example 3: Join Listener with Default Language

```java
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.player.PlayerJoinEvent;
import dev.steyon.translateMCPlugin.TranslateMCPlugin;

public class PlayerJoinListener implements Listener {
    
    @EventHandler
    public void onPlayerJoin(PlayerJoinEvent event) {
        Player player = event.getPlayer();
        TranslateMCPlugin plugin = TranslateMCPlugin.getInstance();
        TranslationManager manager = plugin.getTranslationManager();
        
        // Send welcome message in player's language
        String welcome = manager.getTranslation(player, "welcome.message");
        player.sendMessage(welcome);
    }
}
```

### Example 4: List Available Languages

```java
public class LanguageListCommand implements CommandExecutor {
    
    @Override
    public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {
        TranslateMCPlugin plugin = TranslateMCPlugin.getInstance();
        TranslationManager manager = plugin.getTranslationManager();
        
        sender.sendMessage("Available languages:");
        
        for (TranslationAPI.Language lang : manager.getAvailableLanguages()) {
            String marker = lang.isSource() ? " (default)" : "";
            sender.sendMessage("  " + lang.getCode() + " - " + lang.getName() + marker);
        }
        
        return true;
    }
}
```

### Example 5: Broadcast Message in All Languages

```java
import org.bukkit.Bukkit;
import org.bukkit.entity.Player;

public class BroadcastManager {
    
    public static void broadcastTranslated(String translationKey) {
        TranslateMCPlugin plugin = TranslateMCPlugin.getInstance();
        TranslationManager manager = plugin.getTranslationManager();
        
        for (Player player : Bukkit.getOnlinePlayers()) {
            String message = manager.getTranslation(player, translationKey);
            player.sendMessage(message);
        }
    }
}
```

---

## Best Practices

### 1. Always Check for Null Languages

```java
// ✅ Safe
TranslationAPI.Language lang = manager.getLanguage("de");
if (lang != null) {
    System.out.println(lang.getName());
}

// ❌ Risk of NullPointerException
// System.out.println(manager.getLanguage("de").getName());
```

### 2. Validate Language Codes

```java
// ✅ Good
String code = args[0].toLowerCase();
if (manager.isLanguageAvailable(code)) {
    manager.setPlayerLanguage(player, code);
} else {
    player.sendMessage("Invalid language code!");
}
```

### 3. Use Translations for Messages

```java
// ✅ Use translation system
String message = manager.getTranslation(player, "error.no.permission");
player.sendMessage(message);

// ❌ Don't hardcode messages
// player.sendMessage("You don't have permission!");
```

### 4. Fallback Gracefully

```java
String translation = manager.getTranslation(player, "my.key");
if (translation.equals("my.key")) {
    // Translation not found, use default
    player.sendMessage("Default message");
}
```

---

## Complete API Reference

| Method | Description |
|--------|-------------|
| `getInstance()` | Get plugin instance |
| `getTranslationManager()` | Get translation manager |
| `getTranslation(Player, String)` | Get translation in player's language |
| `getTranslation(String, String)` | Get translation for specific language |
| `setPlayerLanguage(Player, String)` | Set player's language |
| `getPlayerLanguage(Player)` | Get player's language |
| `getAvailableLanguageCodes()` | Get all language codes |
| `getAvailableLanguages()` | Get all languages with details |
| `isLanguageAvailable(String)` | Check if language exists |
| `getLanguage(String)` | Get language by code |
| `Language.getCode()` | Get language code |
| `Language.getName()` | Get language name |
| `Language.isSource()` | Check if default language |

---

## Common Use Cases

### Use Case 1: Translate Server Messages

```java
TranslationManager manager = TranslateMCPlugin.getInstance()
    .getTranslationManager();

for (Player player : Bukkit.getOnlinePlayers()) {
    String msg = manager.getTranslation(player, "server.shutdown");
    player.sendMessage(msg);
}
```

### Use Case 2: Player Selects Language

```java
String selectedLanguage = getUserInput(); // from GUI or command
TranslationManager manager = TranslateMCPlugin.getInstance()
    .getTranslationManager();

if (manager.isLanguageAvailable(selectedLanguage)) {
    manager.setPlayerLanguage(player, selectedLanguage);
    
    String confirmation = manager.getTranslation(player, "lang.changed");
    player.sendMessage(confirmation);
}
```

### Use Case 3: Get Language Name

```java
TranslationManager manager = TranslateMCPlugin.getInstance()
    .getTranslationManager();

String code = manager.getPlayerLanguage(player);
TranslationAPI.Language lang = manager.getLanguage(code);

if (lang != null) {
    player.sendMessage("Your language: " + lang.getName());
}
```

---

## Compatibility

- **Minecraft**: 1.21+
- **Paper**: 1.21+
- **Java**: 21+
- **Plugin Version**: 1.0.0

---

## License

MIT License - See LICENSE in the repository

---

## Support
- **GitHub**: https://github.com/steyondev/TranslateMC-Plugin

---

**Last Updated**: December 2025
