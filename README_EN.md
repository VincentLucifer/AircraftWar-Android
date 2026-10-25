# Aircraft War · Android Arcade Shooter

[简体中文](README.md) | [English](README_EN.md)

A vertical shooter built with **Java and native Android components**, combining touch controls, enemies and bosses, difficulty progression, firepower pickups, local leaderboards and a networked score client.

## Gameplay

- **Touch control and automatic shooting**: drag the hero aircraft to avoid enemies and bullets.
- **Enemy types**: basic enemies, elite enemies and bosses, with separate movement, shooting and supply logic.
- **Three difficulty levels**: Easy, Medium and Hard use distinct game subclasses.
- **Difficulty progression**: Hard mode adjusts enemy probability, spawn frequency, attributes and boss-health multipliers over time.
- **Pickups**: health and timed firepower upgrades; firepower is restored after a background timer completes.
- **Audio feedback**: background/boss music and effects for hits, pickups and game over, with a menu music option.
- **Score records**: player name, score, time and rank persisted in app-private files and sorted by score.
- **Network client**: TCP Socket messages for matching, score exchange and results.

## Object-oriented design

| Approach | Code | Purpose |
| --- | --- | --- |
| Singleton | `HeroAircraft` with volatile and double-checked locking | One current hero instance |
| Factory | `EnemyFactory` / `SupplyFactory` implementations | Separate object construction from the game loop |
| Strategy | `ShootStrategy`, `DirectShoot`, `DisperseShoot` | Separate shooting behaviour from aircraft |
| Template-style game flow | `BaseGame` and `EasyGame/MediumGame/HardGame` | Reuse updates, collisions and drawing while varying difficulty rules |
| DAO | `PlayerScoreDAO` / `PlayerScoreDAOImpl` | Separate score storage from UI code |

## Game loop and threading

`BaseGame` extends `SurfaceView` and uses `SurfaceHolder.Callback` for drawing lifecycle entry points. A background thread runs the loop; Canvas renders backgrounds, aircraft, bullets, health and scores.

Updates cover spawning, shooting, movement, collision checks and invalid-object removal. A `Handler` carries game events to the UI thread; the network receiver also uses messages to update opponent scores.

## Project map

| Path under `app/src/main/` | Contents |
| --- | --- |
| `java/.../activity/` | Menu, difficulty selection, game and score screens |
| `java/.../game/` | SurfaceView loop and difficulty implementations |
| `java/.../aircraft/` / `bullet/` | Aircraft and bullet objects |
| `java/.../factory/` | Enemy and pickup factories |
| `java/.../shoot/` | Shooting strategies |
| `java/.../supply/` | Pickup effects |
| `java/.../DAO/` | Score models and file persistence |
| `java/.../music/` | MediaPlayer / SoundPool wrappers |
| `res/` | Layouts, images and audio |

## Build and launch

| Setting | Repository value |
| --- | --- |
| Android Gradle Plugin | 8.0.2 |
| Gradle Wrapper | 8.0 |
| compileSdk / targetSdk | 33 / 33 |
| minSdk | 30 (Android 11) |
| Java source compatibility | Java 8 |

```bash
git clone https://github.com/VincentLucifer/AircraftWar-Android.git
cd AircraftWar-Android

# Linux / macOS
./gradlew assembleDebug

# Windows
gradlew.bat assembleDebug
```

Alternatively, open the repository root in Android Studio, prepare Android SDK 33, sync Gradle and run `app` on an API 30+ device or emulator. Use a build JDK compatible with AGP 8.

Choose music and game mode from the home screen. Offline play opens difficulty selection before starting the game.

## Network message contract

The client lives in `MainActivity.NetConn` and currently targets `10.0.2.2:9999`. Network play requires a server implementing the same text protocol. `10.0.2.2` addresses the host from an Android emulator; physical devices need a reachable server address.

| Direction | Example messages |
| --- | --- |
| Client → server | `Match`, `Score:<score>`, `end:<score>` |
| Server → client | `connecting`, `StartGame`, `opponentPlayer:<score>`, `EndGame` |

This component connects Socket communication and background reception with Android UI updates.

## Background

An Android implementation based on the HITSZ object-oriented software construction Aircraft War coursework, retaining course objects and resources while exploring game flow, design patterns, persistence and client interaction.
