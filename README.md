# Puck Modding Guide

https://deepwiki.com/cryphor/Puck-Dev-Guide

A community-curated reference for creating mods and plugins for the game [**Puck**](https://store.steampowered.com/app/2994020/Puck/).

## Contributing / Reporting Issues

If you find something wrong or missing:

1. Open an issue at: **https://github.com/cryphor/Puck-Dev-Guide/issues**
2. Include:
   - Which file(s) have the problem
   - What the incorrect/outdated information is
   - What the correct information should be (with evidence if possible)

Pull requests that improve accuracy or add missing API surface are welcome.

## Website

Open `index.html` in a browser for a dark-themed, searchable site with syntax-highlighted code blocks. Deploy to Vercel for free by connecting this repo — everything works as a static site.

## Files

| # | File | What's inside |
|---|------|---------------|
| 01 | `01_Overview.txt` | Project setup, folder structure, plugin vs mod |
| 02 | `02_Plugin_API.txt` | IPuckPlugin, BasePlugin, ModManager, ServerConfig |
| 03 | `03_Event_System.txt` | EventManager API, all event names and message keys |
| 04 | `04_Player_Classes.txt` | Player, PlayerGameState, PlayerCustomizationState, PlayerInput |
| 05 | `05_Game_Objects.txt` | Puck, Stick, PlayerBody, PlayerPosition |
| 06 | `06_Managers.txt` | GameManager, PlayerManager, SettingsManager, InputManager, SaveManager, ItemManager, UIManager, all singleton managers |
| 07 | `07_UI_System.txt` | UIView, UIPopupManager, BasePopupContent, custom UI elements, UIMainMenu, UIPauseMenu, button injection, USS styling |
| 08 | `08_GameMode_System.txt` | BaseGameMode, configs, all phase hooks |
| 09 | `09_Utility_Classes.txt` | Utils, NetworkingUtils, BackendUtils, Constants |
| 10 | `10_Enums.txt` | All enums (Player, Game, Connection, UI, Item) |
| 11 | `11_Harmony_Patching.txt` | Harmony patching setup and usage |
| 12 | `12_Networking.txt` | NetworkBehaviour, RPCs, NetworkVariable |
| 13 | `13_Examples.txt` | 7 complete example mods (plugin, patch, gamemode, popup, proximity, stats, speed) |
| 14 | `14_Singletons_Lifecycle_Tips.txt` | Singleton base classes, lifecycle init order, general tips |

## Source

The API details were extracted from decompiling **Puck.dll** (`Puck`).

## License

Public domain / CC0 — free to use, share, and modify.