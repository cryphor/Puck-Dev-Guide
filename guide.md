# Puck 2.0 Modding Guide

## Overview

Puck supports two types of plugins/mods:

- **Plugin** - A local plugin loaded from the `Plugins/` directory (one folder per plugin, each containing a DLL). These are standalone mods not distributed via Steam Workshop.
- **Mod** - A Steam Workshop item. Managed through Steam Workshop and installed to the workshop folder.

Both use the same underlying system: a DLL that implements `IPuckPlugin` with `OnEnable()` and `OnDisable()` returning `bool`.

---

## Getting Started

### Project Setup
1. Create a new C# Class Library project targeting .NET Standard 2.1
2. Reference `Puck.dll` and required Unity assemblies from the game's `Puck_Data/Managed/` directory
3. Reference `HarmonyLib.dll` for patching
4. Build your DLL

### Folder Structure
```
Puck/
└── Plugins/
    └── MyPlugin/           <-- Folder name = plugin ID
        └── MyPlugin.dll    <-- Your compiled DLL
```

For Steam Workshop mods, files go through the workshop system.

---

## Plugin/Mod API

### IPuckPlugin Interface
Every mod/plugin DLL must contain exactly **one** class implementing this interface:

```csharp
public interface IPuckPlugin
{
    bool OnEnable();   // Called when mod is enabled. Return true on success.
    bool OnDisable();  // Called when mod is disabled. Return true on success.
}
```

The game uses reflection to find the class: it looks for the first non-abstract class in the assembly that implements `IPuckPlugin`.

### BasePlugin\<T\>
The internal base class. Key properties:

| Property | Type | Description |
|----------|------|-------------|
| `State` | `T` | The plugin state |
| `Path` | `string` | Directory path of the plugin |
| `IsReady` | `bool` | Whether the plugin is ready |
| `IsEnabled` | `bool` | Whether the plugin is currently enabled |
| `HasAssembly` | `bool` | Whether the plugin has an associated DLL |

### ModManager
Static API for managing mods and plugins:

```csharp
// Lists
ModManager.Mods           // List<Mod> - all mods
ModManager.Plugins        // List<Plugin> - all plugins
ModManager.ReadyMods      // Mod[] - ready mods
ModManager.EnabledMods    // Mod[] - enabled mods
ModManager.ReadyPlugins   // Plugin[]
ModManager.EnabledPlugins // Plugin[]

// Lookup by ID
ModManager.GetModById(string id)
ModManager.GetPluginById(string id)

// Status
ModManager.GetModStatus(string id)  // bool
ModManager.SetModStatus(string id, bool isEnabled)
```

### ModConfig (for servers)
```csharp
public class ModConfig
{
    public string id { get; set; }
    public bool isEnabled { get; set; }
    public bool isClientRequired { get; set; }
}
```

### ServerConfig
```csharp
public class ServerConfig
{
    public ushort port;           // default: 30609
    public string name;           // default: "MY PUCK SERVER"
    public int maxPlayers;        // default: 12
    public string password;
    public int tickRate;          // default: 200
    public bool isPublic;         // default: true
    public bool useVoip;
    public bool useWhitelist;
    public ModConfig[] mods;
    public string gameMode;       // default: "public"
    public string level;          // default: "default"
    public string[] EnabledModIds;       // [JsonIgnore]
    public string[] ClientRequiredModIds; // [JsonIgnore]
}
```

---

## Event System

The game uses a centralized event system. You can listen for or trigger events.

### EventManager

```csharp
// Listen for events
EventManager.AddEventListener("EventName", CallbackMethod);

// Remove listener
EventManager.RemoveEventListener("EventName", CallbackMethod);

// Check if event has listeners
EventManager.HasEventListeners("EventName");

// Trigger events
EventManager.TriggerEvent("EventName", new Dictionary<string, object>
{
    { "key", value }
});
```

Callback signature: `void MethodName(Dictionary<string, object> message)`

### Event Naming Convention

| Prefix | Description |
|--------|-------------|
| `Event_Server_` | Server-side only event |
| `Event_Client_` | Client-side only event |
| `Event_Everyone_` | Fires on both server and client |
| `Event_On` | General event (local) |

### Key Events

#### Player Events
| Event Name | Message Keys | Notes |
|------------|-------------|-------|
| `Event_Everyone_OnPlayerSpawned` | `player` (Player) | Player network object created |
| `Event_Everyone_OnPlayerDespawned` | `player` (Player) | |
| `Event_Everyone_OnPlayerAdded` | `player` (Player) | Added to PlayerManager |
| `Event_Everyone_OnPlayerRemoved` | `player` (Player) | |
| `Event_Everyone_OnPlayerGameStateChanged` | `player`, `oldGameState`, `newGameState` | |
| `Event_Everyone_OnPlayerCustomizationStateChanged` | `player`, `oldCustomizationState`, `newCustomizationState` | |
| `Event_Everyone_OnPlayerHandednessChanged` | `player`, `oldHandedness`, `newHandedness` | |
| `Event_Everyone_OnPlayerSteamIdChanged` | `player`, `oldSteamId`, `newSteamId` | |
| `Event_Everyone_OnPlayerUsernameChanged` | `player`, `oldUsername`, `newUsername` | |
| `Event_Everyone_OnPlayerNumberChanged` | `player`, `oldNumber`, `newNumber` | |
| `Event_Everyone_OnPlayerPatreonLevelChanged` | `player`, `oldLevel`, `newLevel` | |
| `Event_Everyone_OnPlayerAdminLevelChanged` | `player`, `oldLevel`, `newLevel` | |
| `Event_Everyone_OnPlayerGoalsChanged` | `player`, `oldGoals`, `newGoals` | |
| `Event_Everyone_OnPlayerAssistsChanged` | `player`, `oldAssists`, `newAssists` | |
| `Event_Everyone_OnPlayerPingChanged` | `player`, `oldPing`, `newPing` | |
| `Event_Everyone_OnPlayerPositionChanged` | `player`, `oldPlayerPosition`, `newPlayerPosition` | |
| `Event_Everyone_OnPlayerIsMutedChanged` | `player`, `oldIsMuted`, `newIsMuted` | |
| `Event_Everyone_OnPlayerEnterZone` | - | |
| `Event_Everyone_OnPlayerExitZone` | - | |

#### Puck Events
| Event Name | Message Keys | Notes |
|------------|-------------|-------|
| `Event_Everyone_OnPuckSpawned` | `puck` (Puck) | |
| `Event_Everyone_OnPuckDespawned` | `puck` (Puck) | |
| `Event_Everyone_OnPuckIsReplayChanged` | `puck`, `oldIsReplay`, `newIsReplay` | |

#### Game Events
| Event Name | Message Keys | Notes |
|------------|-------------|-------|
| `Event_Everyone_OnGameStateChanged` | `oldGameState`, `newGameState` | Contains GameState struct |
| `Event_Everyone_OnGoalScored` | - | |
| `Event_Server_OnPuckEnterGoal` | - | |

#### Mod/Plugin Events
| Event Name | Message Keys | Notes |
|------------|-------------|-------|
| `Event_OnModAdded` | `mod` (Mod) | |
| `Event_OnModRemoved` | `mod` (Mod) | |
| `Event_OnPluginAdded` | `plugin` (Plugin) | |
| `Event_OnPluginRemoved` | `plugin` (Plugin) | |
| `Event_OnModStateChanged` | `mod`, `oldState`, `newState` | |
| `Event_OnPluginStateChanged` | `plugin`, `oldState`, `newState` | |
| `Event_OnModEnableFailed` | `mod` | |
| `Event_OnModDisableFailed` | `mod` | |
| `Event_OnPluginEnableFailed` | `plugin` | |
| `Event_OnPluginDisableFailed` | `plugin` | |
| `Event_OnModsPluginEnabled` | `plugin` | |
| `Event_OnModsPluginDisabled` | `plugin` | |
| `Event_OnModsModEnabled` | `mod` | |
| `Event_OnModsModDisabled` | `mod` | |

#### Connection Events
| Event Name | Message Keys | Notes |
|------------|-------------|-------|
| `Event_Server_OnServerStarted` | - | |
| `Event_Server_OnApprovedClientConnected` | `clientId`, `connectionApproval` | |
| `Event_OnConnectionStateChanged` | - | |
| `Event_OnReconnectionStateChanged` | - | |

#### UI / Menu Events
| Event Name | Notes |
|------------|-------|
| `Event_OnMainMenuShow` | Fired when main menu becomes visible |
| `Event_OnMainMenuHide` | Fired when main menu is hidden |
| `Event_OnMainMenuClickPlay` | Play button clicked |
| `Event_OnMainMenuClickPlayer` | Player button clicked |
| `Event_OnMainMenuClickSettings` | Settings button clicked |
| `Event_OnMainMenuClickMods` | Mods button clicked |
| `Event_OnMainMenuClickExitGame` | Exit game button clicked |
| `Event_OnPauseMenuClickReturnToGame` | |
| `Event_OnPauseMenuClickSelectTeam` | |
| `Event_OnPauseMenuClickSelectPosition` | |
| `Event_OnPauseMenuClickForfeit` | |
| `Event_OnPauseMenuClickServerBrowser` | |
| `Event_OnPauseMenuClickSettings` | |
| `Event_OnPauseMenuClickDisconnect` | |
| `Event_OnPauseMenuClickExitGame` | |
| `Event_OnPopupShow` | message has `name` |
| `Event_OnPopupHide` | message has `name` |
| `Event_OnPopupClickOk` | message has `popup` |
| `Event_OnPopupClickClose` | message has `popup` |
| `Event_OnSettingsClickClose` | |
| `Event_OnSettingsClickResetToDefault` | |

#### Settings/Input Events
| Event Name | Notes |
|------------|-------|
| `Event_OnFovChanged` | message has `value` (float) |
| `Event_OnKeyBindsLoaded` | message has `keyBinds` |
| `Event_OnKeyBindsSaved` | message has `keyBinds` |
| `Event_OnKeyBindRebindStart` | |
| `Event_OnKeyBindRebindComplete` | |
| `Event_OnKeyBindRebindCancel` | |

#### Other Events
| Event Name | Notes |
|------------|-------|
| `Event_Server_OnChatCommand` | Chat command received by server |
| `Event_OnBaseCameraStarted` | |
| `Event_OnBaseCameraDestroyed` | |
| `Event_OnBaseCameraEnabled` | |
| `Event_OnBaseCameraDisabled` | |
| `Event_OnModPreviewLinkClicked` | message has `id` |

---

## Important Classes

### Player (NetworkBehaviour)
Access via `PlayerManager.Instance.GetPlayers()`, event messages, or `PlayerManager.Instance.GetLocalPlayer()`.

```csharp
// NetworkVariables
player.GameState              // NetworkVariable<PlayerGameState> - Phase, Team, Role
player.CustomizationState     // NetworkVariable<PlayerCustomizationState>
player.Handedness             // NetworkVariable<PlayerHandedness>
player.SteamId                // NetworkVariable<FixedString32Bytes>
player.Username               // NetworkVariable<FixedString32Bytes>
player.Number                 // NetworkVariable<int>
player.PatreonLevel           // NetworkVariable<int>
player.AdminLevel             // NetworkVariable<int>
player.Goals                  // NetworkVariable<int>
player.Assists                // NetworkVariable<int>
player.Ping                   // NetworkVariable<ulong>
player.IsMuted                // NetworkVariable<bool>
player.IsReplay               // NetworkVariable<bool>

// Convenience properties
player.Phase                  // PlayerPhase (None, TeamSelect, PositionSelect, Play, Replay, Spectate)
player.Team                   // PlayerTeam (None, Blue, Red, Spectator)
player.Role                   // PlayerRole (None, Attacker, Goalie)
player.FlagID
player.HeadgearIDBlueAttacker
// ... plus all other customization IDs

// Component references
player.PlayerInput            // PlayerInput component
player.PlayerCamera           // PlayerCamera (BaseCamera subclass)
player.PlayerBody             // PlayerBody reference
player.StickPositioner        // StickPositioner reference
player.Stick                  // Stick reference
player.PlayerPosition         // PlayerPosition reference
player.SpectatorCamera        // SpectatorCamera reference
player.IsCharacterSpawned     // bool - has full character
player.IsLocalPlayer          // (from NetworkBehaviour)
player.OwnerClientId          // (from NetworkBehaviour)

// Methods
player.GetPlayerJerseyID()
player.GetPlayerHeadgearID()
player.GetPlayerStickSkinID()
player.GetPlayerStickShaftTapeID()
player.GetPlayerStickBladeTapeID()
```

### PlayerGameState
```csharp
public struct PlayerGameState
{
    public PlayerPhase Phase;
    public PlayerTeam Team;
    public PlayerRole Role;
}
```

### PlayerCustomizationState
```csharp
public struct PlayerCustomizationState
{
    public int FlagID;
    public int HeadgearIDBlueAttacker;
    public int HeadgearIDRedAttacker;
    public int HeadgearIDBlueGoalie;
    public int HeadgearIDRedGoalie;
    public int MustacheID;
    public int BeardID;
    public int JerseyIDBlueAttacker;
    // ... JerseyIDRedAttacker, JerseyIDBlueGoalie, JerseyIDRedGoalie
    // ... StickSkinID*, StickShaftTapeID*, StickBladeTapeID*
}
```

### PlayerInput (NetworkBehaviour)
```csharp
// Networked inputs
player.PlayerInput.MoveInput                 // NetworkedInput<Vector2>
player.PlayerInput.StickRaycastOriginAngleInput // NetworkedInput<Vector2>
player.PlayerInput.LookAngleInput            // NetworkedInput<Vector2>
player.PlayerInput.BladeAngleInput           // NetworkedInput<sbyte>
player.PlayerInput.SlideInput                // NetworkedInput<bool>
player.PlayerInput.SprintInput               // NetworkedInput<bool>
player.PlayerInput.TrackInput                // NetworkedInput<bool>
player.PlayerInput.LookInput                 // NetworkedInput<bool>
player.PlayerInput.JumpInput                 // NetworkedInput<byte>
player.PlayerInput.StopInput                 // NetworkedInput<bool>
player.PlayerInput.TwistLeftInput            // NetworkedInput<byte>
player.PlayerInput.TwistRightInput           // NetworkedInput<byte>
player.PlayerInput.DashLeftInput             // NetworkedInput<byte>
player.PlayerInput.DashRightInput            // NetworkedInput<byte>
player.PlayerInput.ExtendLeftInput           // NetworkedInput<bool>
player.PlayerInput.ExtendRightInput          // NetworkedInput<bool>
player.PlayerInput.LateralLeftInput          // NetworkedInput<bool>
player.PlayerInput.LateralRightInput         // NetworkedInput<bool>
player.PlayerInput.TalkInput                 // NetworkedInput<bool>

// Tick rate
player.PlayerInput.TickRate = 200;
```

### Puck (NetworkBehaviour)
```csharp
puck.Rigidbody                   // Rigidbody
puck.SynchronizedObject          // SynchronizedObject
puck.NetworkObjectCollisionRecorder
puck.ReplayTouchTracker
puck.CollisionRecorder
puck.IsReplay                    // NetworkVariable<bool>
puck.Speed                       // float (current speed)
puck.AngularSpeed                // float
puck.PredictedSpeed              // float
puck.PredictedAngularSpeed       // float
puck.ShotSpeed                   // float
puck.IsGrounded                  // bool
puck.IsTouchingStick             // bool
puck.TouchingStick               // Stick
puck.MaxSpeed                    // float (30f)
puck.MaxAngularSpeed             // float (30f)

// Methods
puck.Server_Freeze(RigidbodyConstraints)
puck.Server_Unfreeze()
puck.GetPlayerCollisions()       // List<KeyValuePair<Player, float>>
puck.GetPlayerCollisionsByTeam(PlayerTeam team)
puck.IsActivelyTouchedBy(Player player)
```

### Stick (NetworkBehaviour)
```csharp
stick.Player                     // Player (owner)
```

### PlayerBody (NetworkBehaviour)
```csharp
playerBody.Player                // Player (owner)
```

### PlayerPosition (NetworkBehaviour)
```csharp
playerPosition.transform.position // World position of the player
```

### GameManager (NetworkBehaviourSingleton\<GameManager\>)
```csharp
GameManager.Instance.GameState   // NetworkVariable<GameState>
GameManager.Instance.Phase       // GamePhase (None, Warmup, PreGame, FaceOff, Play, BlueScore, RedScore, Replay, Intermission, GameOver, PostGame)
GameManager.Instance.Tick        // int
GameManager.Instance.Period      // int
GameManager.Instance.BlueScore   // int
GameManager.Instance.RedScore    // int
GameManager.Instance.IsOvertime  // bool
```

### GameState
```csharp
public struct GameState
{
    public GamePhase Phase;
    public int Tick;
    public int Period;
    public int BlueScore;
    public int RedScore;
    public bool IsOvertime;
}
```

### PlayerManager (MonoBehaviourSingleton\<PlayerManager\>)
```csharp
PlayerManager.Instance.GetPlayers(bool includeReplay = false)
PlayerManager.Instance.GetPlayersByPhase(PlayerPhase phase)
PlayerManager.Instance.GetPlayersByTeam(PlayerTeam team)
PlayerManager.Instance.GetPlayersByTeams(PlayerTeam[] teams)
PlayerManager.Instance.GetPlayerByClientId(ulong clientId)
PlayerManager.Instance.GetPlayerByUsername(FixedString32Bytes username)
PlayerManager.Instance.GetPlayerByNumber(int number)
PlayerManager.Instance.GetPlayerBySteamId(FixedString32Bytes steamId)
PlayerManager.Instance.GetPlayerByNeedle(string needle)
PlayerManager.Instance.GetLocalPlayer()
PlayerManager.Instance.GetSpawnedPlayers(bool includeReplay = false)
PlayerManager.Instance.GetSpawnedPlayersByTeam(PlayerTeam team)
PlayerManager.Instance.GetReplayPlayers()
```

### PuckManager (MonoBehaviourSingleton\<PuckManager\>)
```csharp
PuckManager.Instance    // Singleton instance
```

### ChatManager (NetworkBehaviourSingleton\<ChatManager\>)
```csharp
ChatManager.Instance    // Singleton instance
```

### SettingsManager
All settings are static fields. When changed, trigger `Event_On*` events.

```csharp
// Gameplay
SettingsManager.CameraAngle             // float - default 30
SettingsManager.Handedness              // PlayerHandedness - default Right
SettingsManager.ShowPuckSilhouette      // bool - default true
SettingsManager.ShowPuckElevation       // bool - default true
SettingsManager.Units                   // Units (Metric/Imperial)
SettingsManager.GlobalStickSensitivity  // float - default 0.2
SettingsManager.HorizontalStickSensitivity
SettingsManager.VerticalStickSensitivity
SettingsManager.LookSensitivity         // float - default 0.2
SettingsManager.Fov                     // float - default 90
SettingsManager.Team                    // PlayerTeam - default Blue
SettingsManager.Role                    // PlayerRole - default Attacker

// Audio
SettingsManager.GlobalVolume
SettingsManager.AmbientVolume
SettingsManager.GameVolume
SettingsManager.VoiceVolume
SettingsManager.UIVolume

// Video
SettingsManager.FullScreenMode
SettingsManager.DisplayIndex
SettingsManager.ResolutionIndex
SettingsManager.VSync
SettingsManager.FpsLimit
SettingsManager.Quality
SettingsManager.ShadowQuality
SettingsManager.MotionBlur

// Customization IDs
SettingsManager.FlagID
SettingsManager.HeadgearIDBlueAttacker
SettingsManager.HeadgearIDRedAttacker
SettingsManager.HeadgearIDBlueGoalie
SettingsManager.HeadgearIDRedGoalie
SettingsManager.JerseyIDBlueAttacker
SettingsManager.JerseyIDRedAttacker
SettingsManager.JerseyIDBlueGoalie
SettingsManager.JerseyIDRedGoalie
SettingsManager.StickSkinIDBlueAttacker
SettingsManager.StickSkinIDRedAttacker
SettingsManager.StickSkinIDBlueGoalie
SettingsManager.StickSkinIDRedGoalie
SettingsManager.StickShaftTapeIDBlueAttacker
SettingsManager.StickShaftTapeIDRedAttacker
SettingsManager.StickShaftTapeIDBlueGoalie
SettingsManager.StickShaftTapeIDRedGoalie
SettingsManager.StickBladeTapeIDBlueAttacker
SettingsManager.StickBladeTapeIDRedAttacker
SettingsManager.StickBladeTapeIDBlueGoalie
SettingsManager.StickBladeTapeIDRedGoalie
```

### InputManager
All input actions are static fields:

```csharp
// Movement
InputManager.MoveForwardAction
InputManager.MoveBackwardAction
InputManager.TurnLeftAction
InputManager.TurnRightAction

// Stick
InputManager.StickAction           // Reads Vector2 value
InputManager.BladeAngleUpAction
InputManager.BladeAngleDownAction

// Actions
InputManager.SlideAction
InputManager.SprintAction
InputManager.TrackAction
InputManager.LookAction
InputManager.JumpAction
InputManager.StopAction
InputManager.TwistLeftAction
InputManager.TwistRightAction
InputManager.DashLeftAction
InputManager.DashRightAction
InputManager.ExtendLeftAction
InputManager.ExtendRightAction
InputManager.LateralLeftAction
InputManager.LateralRightAction
InputManager.TalkAction

// UI
InputManager.AllChatAction
InputManager.TeamChatAction
InputManager.PauseAction
InputManager.PositionSelectAction
InputManager.ScoreboardAction
InputManager.PointAction
InputManager.ClickAction

// Quick Chat
InputManager.QuickChat1Action ... QuickChat5Action

// Debug
InputManager.Debug1Action ... Debug4Action

// Key binding
InputManager.RebindableInputActions  // List<InputAction>
InputManager.RebindAction(string actionName, string modifierPath, string path)
InputManager.SaveKeyBinds()
InputManager.LoadKeyBinds()
```

### SaveManager
```csharp
SaveManager.SetBool(string key, bool value)
SaveManager.GetBool(string key, bool defaultValue)
SaveManager.SetInt(string key, int value)
SaveManager.GetInt(string key, int defaultValue)
SaveManager.SetFloat(string key, float value)
SaveManager.GetFloat(string key, float defaultValue)
SaveManager.SetString(string key, string value)
SaveManager.GetString(string key, string defaultValue)
SaveManager.SetEnum<T>(string key, T value)
SaveManager.GetEnum<T>(string key, T defaultValue)
```

### ItemManager
```csharp
ItemManager.Items           // List<Item>
ItemManager.GetItemById(int id)
ItemManager.GetItemsByCategories(string[] categories)
ItemManager.IsItemOwned(int itemId, PlayerData playerData)
```

### Item
```csharp
item.id
item.name
item.description
item.categories    // string[] - "flag", "headgear", "jersey", "stickSkin", etc.
item.price
item.code
item.IsFlag / IsHeadgear / IsJersey / IsMustache / IsBeard / IsStickSkin / IsStickShaftTape / IsStickBladeTape
item.IsAttackerItem
item.IsGoalieItem
item.IsOwned
item.IsPurchased
```

### GameMode System
The game uses a pluggable game mode system:

```csharp
abstract class BaseGameMode<TConfig> : IGameMode where TConfig : BaseGameModeConfig, new()
```

Available configurations:
- `BaseGameModeConfig` - base
- `StandardGameModeConfig` - standard mode config
- `CompetitiveGameModeConfig` - comp mode config
- `PublicGameModeConfig` - public mode config

Available game modes:
- `StandardGameMode<TConfig>` - standard mode
- `CompetitiveGameMode<TConfig>` - competitive mode
- `PublicGameMode<TConfig>` - public mode
- `MatchableGameMode<TConfig>` - matchable/base for comp

Key methods:
```csharp
// Lifecycle
Initialize(Level, ServerManager, GameManager, PlayerManager, PuckManager, ChatManager, ReplayManager, VoteManager)
Dispose()

// Scoring
ScoreGoal(PlayerTeam byTeam, Player goalPlayer, Player assistPlayer, Player secondAssistPlayer, Puck puck)

// Game phase hooks (all virtual)
OnGamePhaseStarted(GamePhase gamePhase)
OnGamePhaseEnded(GamePhase gamePhase)
OnGamePhaseTimedOut(GamePhase gamePhase)
OnWarmupStarted() / OnWarmupTimedOut()
OnPreGameStarted() / OnPreGameTimedOut()
OnFaceOffStarted() / OnFaceOffTimedOut()
OnPlayStarted() / OnPlayTimedOut()
OnBlueScoreStarted() / OnBlueScoreTimedOut()
OnRedScoreStarted() / OnRedScoreTimedOut()
OnReplayStarted() / OnReplayTimedOut()
OnIntermissionStarted() / OnIntermissionTimedOut()
OnGameOverStarted() / OnGameOverTimedOut()
OnPostGameStarted() / OnPostGameTimedOut()
```

### UIManager (MonoBehaviourSingleton\<UIManager\>)
```csharp
UIManager.Instance.RootVisualElement     // VisualElement (UI Toolkit root)
UIManager.Instance.PanelSettings         // PanelSettings
UIManager.Instance.AudioSource           // AudioSource for UI sounds

// UI Views
UIManager.Instance.MainMenu             // UIMainMenu
UIManager.Instance.PauseMenu            // UIPauseMenu
UIManager.Instance.ServerBrowser        // UIServerBrowser
UIManager.Instance.Chat                 // UIChat
UIManager.Instance.TeamSelect           // UITeamSelect
UIManager.Instance.PositionSelect       // UIPositionSelect
UIManager.Instance.Scoreboard           // UIScoreboard
UIManager.Instance.Settings             // UISettings
UIManager.Instance.Hud                  // UIHUD
UIManager.Instance.Announcements        // UIAnnouncements
UIManager.Instance.Minimap              // UIMinimap
UIManager.Instance.NewServer            // UINewServer
UIManager.Instance.DirectConnect        // UIDirectConnect
UIManager.Instance.ToastManager         // UIToastManager
UIManager.Instance.OverlayManager       // UIOverlayManager
UIManager.Instance.PlayerMenu           // UIPlayerMenu
UIManager.Instance.Identity             // UIIdentity
UIManager.Instance.Appearance           // UIAppearance
UIManager.Instance.PopupManager         // UIPopupManager
UIManager.Instance.Usernames            // UIUsernames
UIManager.Instance.Debug                // UIDebug
UIManager.Instance.Mods                 // UIMods
UIManager.Instance.Footer               // UIFooter
UIManager.Instance.Friends              // UIFriends
UIManager.Instance.Play                 // UIPlay
UIManager.Instance.Matchmaking          // UIMatchmaking
UIManager.Instance.GameState            // UIGameState

// Methods
UIManager.Instance.PlaySelectSound()
UIManager.Instance.PlayClickSound()
UIManager.Instance.PlayNotificationSound()
```

### UIView
Base class for all UI views (line 30060 in decompiled Puck.dll):

```csharp
public class UIView : MonoBehaviour
{
    public bool FocusRequiresMouse;
    public bool FocusIsInteractive;
    public bool VisibilityRequiresMouse;
    public bool VisibilityIsInteractive;
    public bool AlwaysVisible;
    public VisualElement RootVisualElement;
    public VisualElement View { get; set; }
    public bool IsVisible { get; set; }
    public bool IsFocused { get; set; }
    public int Order { get; }

    public virtual bool Show();
    public virtual bool Hide();
    public virtual bool Toggle();
}
```

Views are initialized by querying named elements from the root visual element:
```csharp
base.View = rootVisualElement.Query<VisualElement>("MyViewName");
```

### UIViewController\<T\>
```csharp
class UIViewController<T> : MonoBehaviour where T : UIView
{
    public virtual void Awake();      // Gets component reference
    public virtual void OnDestroy();  // Unregisters events
}
```

### UIPopupManager
Native popup system. Access via `UIManager.Instance.PopupManager`:

```csharp
// Show a popup with the game's native frame (title bar, close button, footer)
popupManager.ShowPopup(string name, string title, BasePopupContent content,
                       bool showOkButton, bool showCloseButton, object data = null);

// Hide by name
popupManager.HidePopup(string name);

// Check if a popup is open
Popup existing = popupManager.GetPopupByName(string name);
```

The popup template consists of:
- **Header** — `titleLabel` (Label), `CloseIconButtonContainer` → `closeIconButton` (IconButton)
- **Content** — where `Content.VisualElement` is added
- **Footer** — `okButton` (Button, hidden via `display: none` when `showOkButton: false`)

### BasePopupContent
Abstract base for popup content. Pass `null` to constructor, override `Initialize()`:

```csharp
public class BasePopupContent
{
    public VisualElement VisualElement { get; set; }
    public Action Initialized;
    public Action Disposed;

    public BasePopupContent(VisualTreeAsset asset); // pass null
    public virtual void Initialize();
    public virtual void Dispose();
    internal virtual void Update();
}
```

Built-in subclasses: `PopupNotificationContent`, `PopupMissingPasswordContent`, `PopupMissingModsPopupContent`

### Custom UI Elements (found in puck.dll)
These are UXML-compatible custom VisualElements:

| Class | Base | Line | UXML Attributes |
|-------|------|------|-----------------|
| `Icon` | `VisualElement` | 45875 | `texture` |
| `IconButton` | `Button` | 45950 | `texture` |
| `KeyBindField` | `VisualElement` | 46031 | `interaction-type`, `label`, `path` |
| `Link` | `VisualElement` | 46277 | `text` |
| `Mmr` | `VisualElement` | 46381 | (queries internal labels) |
| `PlayButton` | `Button` | 46920 | `text`, `description` |
| `Spinner` | `VisualElement` | 47085 | (animated spinner) |
| `User` | `VisualElement` | 47168 | `username`, `avatar-texture` |
| `Mod` | `VisualElement` | 46557 | `description`, `preview-texture` |
| `ModPreview` | `VisualElement` | 46702 | `is-statistics-visible`, `subscriptions`, `upvotes`, `downvotes` |

### UIMainMenu
```csharp
public class UIMainMenu : UIView
{
    public void Initialize(VisualElement rootVisualElement);
    // Queries: "MainMenuView" → "MainMenu" container
    // Buttons: PlayButton, PlayerButton, SettingsButton, ModsButton, ExitGameButton

    public override bool Show(); // Triggers Event_OnMainMenuShow
    public override bool Hide(); // Triggers Event_OnMainMenuHide
}
```

### UIPauseMenu
```csharp
public class UIPauseMenu : UIView
{
    public void Initialize(VisualElement rootVisualElement);
    // Queries: "PauseMenuView" → "PauseMenu" container
    // Buttons: ReturnToGameButton, SelectTeamButton, SelectPositionButton,
    //          ForfeitButton, ServerBrowserButton, SettingsButton,
    //          DisconnectButton, ExitGameButton

    public void SetSelectTeamEnabled(bool value);
    public void SetSelectPositionEnabled(bool value);
    public void SetForfeitEnabled(bool value);
    public void SetForfeitVoted(bool value);
    public void SetVoiceEnabled(bool value);
    public void SetTickRate(int tickRate);
}
```

### Menu Button Injection Pattern
Inject buttons into menus by reflecting private fields and cloning styles:

```csharp
var settingsField = typeof(UIMainMenu).GetField("settingsButton",
    BindingFlags.Instance | BindingFlags.NonPublic);
var settingsBtn = settingsField.GetValue(uiManager.MainMenu) as Button;
var container = settingsBtn.parent;
var myBtn = new Button(OnClick) { text = "MY MOD" };
foreach (string cls in settingsBtn.GetClasses())
    myBtn.AddToClassList(cls);
container.Insert(container.IndexOf(settingsBtn) + 1, myBtn);
```

### Other Managers (Singletons)

| Class | Type | Access |
|-------|------|--------|
| `AudioManager` | `MonoBehaviourSingleton` | `AudioManager.Instance` |
| `ConnectionManager` | `MonoBehaviourSingleton` | `ConnectionManager.Instance` |
| `CrowdManager` | `MonoBehaviourSingleton` | `CrowdManager.Instance` |
| `PhysicsManager` | `MonoBehaviourSingleton` | `PhysicsManager.Instance` |
| `ReplayManager` | `MonoBehaviourSingleton` | `ReplayManager.Instance` |
| `ThreadManager` | `MonoBehaviourSingleton` | `ThreadManager.Instance` |
| `AdminManager` | `MonoBehaviourSingleton` | `AdminManager.Instance` |
| `BanManager` | `MonoBehaviourSingleton` | `BanManager.Instance` |
| `ConnectionApprovalManager` | `MonoBehaviourSingleton` | `ConnectionApprovalManager.Instance` |
| `TimeoutManager` | `MonoBehaviourSingleton` | `TimeoutManager.Instance` |
| `VoteManager` | `MonoBehaviourSingleton` | `VoteManager.Instance` |
| `WhitelistManager` | `MonoBehaviourSingleton` | `WhitelistManager.Instance` |
| `ServerManager` | `NetworkBehaviourSingleton` | `ServerManager.Instance` |
| `GameModeManager` | `NetworkBehaviourSingleton` | `GameModeManager.Instance` |
| `SynchronizedObjectManager` | `NetworkBehaviourSingleton` | `SynchronizedObjectManager.Instance` |

### Utility Classes

#### Utils
```csharp
Utils.GetCollisionForce(Collision collision)
Utils.GetTeamColor(PlayerTeam team)            // Color
Utils.GetNameFromTeam(PlayerTeam team)
Utils.GetTeamFromName(string name)
Utils.GetNameFromRole(PlayerRole role)
Utils.GetRoleFromName(string name)
Utils.GameUnitsToMetric(float value)
Utils.GameUnitsToImperial(float value)
Utils.Vector2Clamp(Vector2 value, Vector2 min, Vector2 max)
Utils.Vector3Clamp(Vector3 value, Vector3 min, Vector3 max)
Utils.RotatePointAroundPivot(Vector3 point, Vector3 pivot, Vector3 angles)
Utils.WrapEulerAngle(float angle)
Utils.Map(float value, float from1, float to1, float from2, float to2)
```

#### NetworkingUtils
```csharp
NetworkingUtils.GetPlayerFromNetworkObjectReference(NetworkObjectReference)
NetworkingUtils.GetPlayerPositionFromNetworkObjectReference(NetworkObjectReference)
NetworkingUtils.GetPuckFromNetworkObjectReference(NetworkObjectReference)
```

#### BackendUtils
```csharp
// Various backend utility methods
```

#### Constants
```csharp
Constants.APP_ID                    // 2994020u
Constants.TEAM_BLUE_COLOR           // "#3b82f6"
Constants.TEAM_RED_COLOR            // "#d13333"
Constants.DEFAULT_SERVER_PORT       // 30609
Constants.DEFAULT_SERVER_MAX_PLAYERS // 12
Constants.DEFAULT_SERVER_TICK_RATE  // 200
// ... and many more constants
```

---

## Enums

### Player Enums
```csharp
PlayerPhase { None, TeamSelect, PositionSelect, Play, Replay, Spectate }
PlayerTeam { None, Blue, Red, Spectator }
PlayerRole { None, Attacker, Goalie }
PlayerHandedness { None, Left, Right }
```

### Game Enums
```csharp
GamePhase { None, Warmup, PreGame, FaceOff, Play, BlueScore, RedScore, Replay, Intermission, GameOver, PostGame }
ApplicationQuality { Low, Medium, High, Ultra }
ShadowQuality { Low, Medium, High }
Units { Metric, Imperial }
```

### Server/Connection Enums
```csharp
ConnectionPhase
ConnectionRejectionCode { ... }
DisconnectionCode { ... }
ReconnectionPhase
AuthenticationPhase
TransactionPhase { ... }
ServerReadinessPhase { None, AwaitingMods, Ready }
SteamWorkshopItemPhase { ... }
```

### UI Enums
```csharp
UIPhase { ... }
AppearanceCategory { ... }
AppearanceSubcategory { ... }
QuickChatCategory { ... }
HeadgearRole { ... }
JerseyTeam { ... }
StickSkinTeam { ... }
PlayerLegPadState { ... }
AvatarSize { ... }
```

### Item Enums
```csharp
// Categories used as strings in Item.categories[]:
// "flag", "headgear", "mustache", "beard", "jersey", "stickSkin"
// "stickShaftTape", "stickBladeTape", "attacker", "goalie", "unlisted"
```

---

## Patching with Harmony

The game already uses Harmony (the `HarmonyLib` namespace is imported). You can patch any method in your mod:

```csharp
using HarmonyLib;

[HarmonyPatch(typeof(SomeClass), "SomeMethod")]
class SomePatch
{
    static void Postfix()
    {
        // Your code here
    }
}
```

The game uses a `PatchManager` that applies `VisualElementHarmonyPatch` on startup. You can create your own Harmony instances.

---

## Network Behaviour

Custom network-accessible components should extend `NetworkBehaviour` from `Unity.Netcode`.

```csharp
using Unity.Netcode;

public class MyNetworkComponent : NetworkBehaviour
{
    public NetworkVariable<int> MyValue;

    protected override void OnNetworkPreSpawn(ref NetworkManager networkManager)
    {
        MyValue = new NetworkVariable<int>();
        base.OnNetworkPreSpawn(ref networkManager);
    }
}
```

### RPCs
```csharp
[Rpc(SendTo.Server, DeferLocal = true)]
public void Client_DoSomethingRpc(string data, RpcParams rpcParams = default(RpcParams))
{
    // Server-side logic
}

[Rpc(SendTo.ClientsAndHost)]
public void Server_NotifyAllRpc(string data, RpcParams rpcParams = default(RpcParams))
{
    // Client-side logic
}
```

### NetworkVariable\<T\>
```csharp
var variable = new NetworkVariable<T>(defaultValue);
variable.OnValueChanged += (oldValue, newValue) => { ... };
```

---

### USS Styling
Toggle game-defined USS classes on elements:
```csharp
element.EnableInClassList("className", true/false);
```

Known class names: `"owned"`, `"blurred"`, `"small"`, `"hideUsername"`, `"warning"`, `"isLocalPlayer"`, `"claimed"`, `"patreon"`, `"moderator"`, `"admin"`, `"developer"`, `"passwordProtected"`, `"modded"`, `"unreachable"`, `"firstChild"`, `"lastChild"`

---

### PlayerBody (NetworkBehaviour)
```csharp
playerBody.Player       // Player owner
playerBody.transform    // Transform (position/rotation in world)
```

Common usage:
```csharp
Vector3 pos = player.PlayerBody.transform.position;
float dist = Vector3.Distance(myPos, pos);
```

---

## Example Mod: Simple Plugin

```csharp
using Puck;

public class MyMod : IPuckPlugin
{
    public bool OnEnable()
    {
        EventManager.AddEventListener("Event_Everyone_OnPlayerSpawned", OnPlayerSpawned);
        EventManager.AddEventListener("Event_Everyone_OnGoalScored", OnGoalScored);
        return true;
    }

    public bool OnDisable()
    {
        EventManager.RemoveEventListener("Event_Everyone_OnPlayerSpawned", OnPlayerSpawned);
        EventManager.RemoveEventListener("Event_Everyone_OnGoalScored", OnGoalScored);
        return true;
    }

    private void OnPlayerSpawned(Dictionary<string, object> message)
    {
        Player player = (Player)message["player"];
        Debug.Log($"Player {player.Username.Value} spawned on team {player.Team}");
    }

    private void OnGoalScored(Dictionary<string, object> message)
    {
        Debug.Log("Goal scored!");
    }
}
```

## Example Mod: Harmony Patch

```csharp
using HarmonyLib;
using Puck;
using UnityEngine;

public class MyPatcher : IPuckPlugin
{
    private Harmony harmony;

    public bool OnEnable()
    {
        harmony = new Harmony("com.example.mymod");
        harmony.PatchAll();
        return true;
    }

    public bool OnDisable()
    {
        harmony?.UnpatchAll(harmony.Id);
        return true;
    }
}

[HarmonyPatch(typeof(Puck), nameof(Puck.OnCollisionEnter))]
class PuckCollisionPatch
{
    static void Postfix(Puck __instance, Collision collision)
    {
        if (collision.gameObject.TryGetComponent<PlayerBody>(out var body))
        {
            Debug.Log($"Puck hit player {body.Player.Username.Value}!");
        }
    }
}
```

## Example: Custom Game Mode

```csharp
using Puck;

public class MyGameModeConfig : StandardGameModeConfig
{
    public int MyCustomSetting = 5;
}

public class MyGameMode : StandardGameMode<MyGameModeConfig>
{
    public MyGameMode() : base("myconfig.json") { }

    protected override void OnPlayStarted()
    {
        base.OnPlayStarted();
        Debug.Log($"My custom setting: {Config.MyCustomSetting}");
    }

    protected override void OnGoalScored(Dictionary<string, object> message)
    {
        // Custom goal logic
        base.OnGoalScored(message);
    }
}
```

## Example: Popup Settings Panel

```csharp
public class MyPopupContent : BasePopupContent
{
    public MyPopupContent() : base(null) { }
    public override void Initialize()
    {
        var root = new VisualElement();
        root.style.minWidth = 300;
        var toggle = new Toggle("Enable Feature") { value = true };
        root.Add(toggle);
        VisualElement = root;
        Initialized?.Invoke();
    }
}

// Show it:
UIManager.Instance.PopupManager.ShowPopup(
    "myModPopup", "MY MOD", new MyPopupContent(),
    showOkButton: false, showCloseButton: true);
```

## Example: In-Game Proximity Check

```csharp
var local = PlayerManager.Instance.GetLocalPlayer();
if (local?.PlayerBody == null) return;
Vector3 myPos = local.PlayerBody.transform.position;
foreach (var p in PlayerManager.Instance.GetPlayers())
{
    if (p == local || p.PlayerBody == null) continue;
    float dist = Vector3.Distance(myPos, p.PlayerBody.transform.position);
    if (dist < 2f) Debug.Log($"{p.Username.Value} is {dist:F1}m away!");
}
```

## Example: Reading Player Stats

```csharp
public class StatsReader : IPuckPlugin
{
    public bool OnEnable()
    {
        EventManager.AddEventListener("Event_Everyone_OnPlayerSpawned", OnPlayerSpawned);
        return true;
    }

    public bool OnDisable()
    {
        EventManager.RemoveEventListener("Event_Everyone_OnPlayerSpawned", OnPlayerSpawned);
        return true;
    }

    private void OnPlayerSpawned(Dictionary<string, object> msg)
    {
        Player p = (Player)msg["player"];
        Debug.Log($"{p.Username.Value} (#{p.Number.Value}) | " +
                  $"Team: {p.Team}, Role: {p.Role}, " +
                  $"Goals: {p.Goals.Value}, Assists: {p.Assists.Value}, " +
                  $"Ping: {p.Ping.Value}ms, " +
                  $"Steam: {p.SteamId.Value}");
    }
}
```

## Example: Logging Puck Speed

```csharp
public class Speedometer : IPuckPlugin
{
    private Puck puck;

    public bool OnEnable()
    {
        EventManager.AddEventListener("Event_Everyone_OnPuckSpawned", OnPuckSpawned);
        EventManager.AddEventListener("Event_Everyone_OnPuckDespawned", OnPuckDespawned);
        return true;
    }

    public bool OnDisable()
    {
        EventManager.RemoveEventListener("Event_Everyone_OnPuckSpawned", OnPuckSpawned);
        EventManager.RemoveEventListener("Event_Everyone_OnPuckDespawned", OnPuckDespawned);
        return true;
    }

    private void OnPuckSpawned(Dictionary<string, object> msg)
    {
        puck = (Puck)msg["puck"];
    }

    private void OnPuckDespawned(Dictionary<string, object> msg)
    {
        puck = null;
    }

    // Call from Update via Harmony or MonoBehaviour
    public void Update()
    {
        if (puck != null)
            Debug.Log($"Puck speed: {puck.Speed:F1} / {puck.MaxSpeed}");
    }
}
```

---

## MonoBehaviourSingleton\<T\> (base class)
```csharp
public class MonoBehaviourSingleton<T> : MonoBehaviour where T : MonoBehaviour
{
    public static T Instance { get; }
    public virtual void Awake(); // sets Instance, DontDestroyOnLoad
}
```

## NetworkBehaviourSingleton\<T\> (base class)
```csharp
public class NetworkBehaviourSingleton<T> : NetworkBehaviour where T : NetworkBehaviour
{
    public static T Instance { get; }
}
```

## Lifecycle

Initialization order (from LifecycleManager):

1. **BeforeSceneLoad**: PatchManager, EventManager, GlobalStateManager, SaveManager, InputManager, SettingsManager, ApplicationManager, BackendManager, ItemManager, CameraManager
2. **AfterSceneLoad**: SceneManager, WebSocketManager, SteamManager, **ModManager**, ServerReadinessManager
3. ModManager loads plugins from `Plugins/` directory and mods from Steam Workshop
4. Plugins are enabled if their status allows

---

## Tips

- Use `NetworkManager.Singleton.IsServer` to check if you're on the server
- Use `NetworkManager.Singleton.IsHost` to check if you're the host (server + client)
- Events prefixed with `Event_Server_` only fire on the server side
- Events prefixed with `Event_Everyone_` fire on both server and client
- Use `SaveManager` for persisting mod settings (wraps Unity's `PlayerPrefs`)
- The UI system uses Unity UI Toolkit (UIElements/UXML)
- For custom networked data, use `NetworkVariable<T>` or custom `NetworkBehaviour` components
- Debug logging: `Debug.Log()`, `Debug.LogWarning()`, `Debug.LogError()`