# Python Dungeon-Crawler

This project is a Python-based, turn-based Dungeon-Crawler game.

This repository serves as a comprehensive case study in modern game development architecture, featuring a **Entity-Component** design, procedural dungeon generation, algorithmic pathfinding, and a **state-driven event system**.

## Core Features Implemented 

* **Entity-Component System:** A highly modular design where entities (`entity.py`) are containers for components (`components/`) that define their behavior (e.g., `Fighter`, `AI`).
* **Procedural Dungeon Generation:** Dungeons are randomly generated for each new game, ensuring high replayability (`procgen.py`).
* **State-Driven Input & Action System:** User input is handled by state-specific handlers (e.g., `MainGameEventHandler`, `InventoryEventHandler`) that generate `Action` objects for the game loop to process.
* **Turn-Based Combat & AI:** Enemies and the player take turns. AI (`components/ai.py`) uses `A* algorithm`'s pathfinding to hunt the player.
* **Field of View (FOV):** The map implements `tcod`'s FOV algorithm to manage exploration and visibility.
* **Items & Inventory:** A complete inventory system allowing for picking up, dropping, and using items (`components/inventory.py`, `components/consumable.py`).
* **Targeting System:** Ranged items (like scrolls) use a targeting handler to allow the player to select a target on the map.
* **Save/Load System:** The complete game state (engine, entities, map) can be saved to a file and reloaded.
* **Multiple Dungeon Floors:** The engine supports transitioning between dungeon levels via stairs, loading new maps as the player descends.

## Technical Stack

* **Python 3**
* **`tcod` (libtcod):** The core library for console rendering.
* **`numpy`:**

## Getting Started

Follow these instructions to set up and run the project locally.

### Prerequisites

* Python 3.8 or newer
* `git` (for cloning the repository)

### Installation & Setup

1.  **Clone the repository:**
    ```sh
    git clone [https://github.com/rem421/dungeon-crawler.git](https://github.com/rem421/dungeon-crawler.git)
    cd dungeon-crawler
    ```

### Running the Game

Execute the `main.py` script to start the game:
```sh
python main.py
```

## Codebase Architecture

The project's architecture is its most significant feature. It is built on an **Entity-Component** model, which avoids deep inheritance hierarchies and favors **composition**.

* An **`Entity`** (`entity.py`) is a simple "bag of components." It has a position, character, and color, but all its logic and data are stored in its attached components.
* A **`Component`** (`components/base_component.py`) is a modular, reusable piece of data or behavior. For example, to make an entity fightable, you simply attach a `Fighter` component to it.

---

### Core Engine & Game Loop

* `main.py`: The main entry point. It initializes the `tcod` context, loads the tileset (e.g., `dejavu10x10_gs_tc.png`), and starts the main game loop.
* `engine.py`: Contains the `Engine` class, the heart of the game. The `Engine` manages the main loop, holds the `game_map` and the `player` entity, and processes events by calling the current `input_handler`.
* `setup_game.py`: Contains the `SetupGame` class, a state-driven game initializer. It manages transitions from the main menu, creates the player entity, and generates new dungeons.
* `entity_factories.py`: A set of factory functions (e.g., `player()`, `orc()`, `health_potion()`) that simplify the creation of new entities by pre-packaging them with the correct components.

### Turn-Based Logic & Input

* `input_handlers.py`: Manages all user input. It uses a **state pattern**, where the `Engine` holds a reference to the *current* handler (e.g., `MainGameEventHandler`, `InventoryEventHandler`). This keeps input logic separate from the main game loop.
* `actions.py`: Defines the `Action` class system. Instead of acting immediately, user input and AI logic *create* `Action` objects (e.g., `MovementAction`, `MeleeAction`). The `Engine` then executes these actions, allowing for a clean, turn-based flow.
* `exceptions.py`: Contains custom exceptions (like `Impossible`) used to handle failed actions gracefully (e.g., trying to walk into a wall).

### Entity Components (`components/`)

This directory is the core of the Entity-Component design.

* `base_component.py`: The abstract base class all other components inherit from.
* `ai.py`: Defines AI behavior. `HostileEnemy` is a component that, when attached to an entity, will cause it to pathfind (A*) toward the player and attack.
* `fighter.py`: Holds all combat-related data: `hp`, `max_hp`, `defense`, and `power`.
* `inventory.py`: Gives an entity the ability to hold a list of other entities (items).
* `consumable.py`: A component for items. When `use`d, it activates its effect (e.g., healing, casting a spell) and is consumed.
* `level.py`: A component (attached to the player) that tracks `current_xp`, `level_up_base`, and handles the logic for advancing levels.

### World & Rendering

* `procgen.py`: Handles all procedural generation. The `generate_dungeon` function uses a random-walk algorithm to carve out rooms and tunnels, placing monsters and items.
* `game_map.py`: The `GameMap` class, which holds a 2D `numpy` array of `tile_types`. It manages collision detection (`in_bounds`, `walkable`) and the `tcod` map for Field of View (FOV) calculations.
* `tile_types.py`: Defines the different types of tiles (e.g., `floor`, `wall`) with their properties (walkable, transparent).
* `render_functions.py`: A collection of functions for drawing all game elements. It handles drawing the map, entities, the UI (like the HP bar), and the message log.
* `render_order.py`: An `Enum` (`RenderOrder`) used to "z-index" entities, ensuring that corpses are drawn under actors, which are drawn under items.
* `message_log.py`: The `MessageLog` class, which stores and manages the list of messages shown to the player.
* `color.py`: A utility module defining common color constants used in rendering.

## 🎮 How to Play (Controls)

Here are the default keyboard controls for the game.

### In-Game (Exploring)

These are the controls for walking around the dungeon.

* **Arrow Keys**: Move your character (`@`). You can also attack enemies by moving into them.
* **`g`**: **G**et / Pick up an item from the floor.
* **`i`**: Open your **I**nventory.
* **`d`**: Open the **D**rop item menu.
* **`>`** (Shift + Period): Use stairs to go **down** to the next dungeon level.
* **`Esc`** (Escape): Open the in-game menu (to Save and Quit).
* **`Alt` + `Enter`**: Toggle fullscreen.

### 🎒 Inventory & Drop Menus

These keys work when you press `i` or `d`.

* **`a`** through **`z`**: Select an item from the list.
* **`Esc`** (Escape): Close the menu and return to the game.

### 🎯 Targeting Mode

This screen appears when you use an item that needs a target (like a Fireball scroll).

* **Arrow Keys**: Move the targeting cursor.
* **`Enter`**: Confirm the target and use the item.
* **`Esc`** (Escape): Cancel targeting and go back.

### 🏁 Main Menu / Game Over Screen

This is the first screen you see, or the screen you get when you die.

* **`n`**: Start a **N**ew game.
* **`c`**: **C**ontinue from your last save file.
* **`q`**: **Q**uit the game.
