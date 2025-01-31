# Feature-Based Game Architecture: Best Practices and Guidelines

## Table of Contents

1. Introduction
2. Core Principles
3. Architecture Overview
4. Component Breakdown
5. Communication Patterns
6. Data Management
7. Best Practices
8. Scalability and Extensibility
9. Testing Strategies
10. Performance Considerations
11. Common Pitfalls and How to Avoid Them
12. Case Studies and Examples
13. Conclusion

## 1. Introduction

This document outlines a feature-based architecture designed for complex game development projects. It provides a flexible, modular approach that emphasizes clean code, separation of concerns, and scalability. While primarily focused on game development, many principles can be applied to other complex software projects.

## 2. Core Principles

- Feature-Centric Organization: Structure code around game features rather than technical layers.
- Modularity: Each feature should be self-contained with minimal dependencies on other features.
- Separation of Concerns: Clearly define responsibilities for each component within a feature.
- Scalability: Design for easy addition of new features and expansion of existing ones.
- Testability: Ensure all components can be easily unit tested.
- Flexibility: Allow for different implementation strategies within the same architectural framework.

## 2. Core Principles (Expanded)

### 2.1 Feature-Centric Organization

- Structure code, scenes, and assets around game features rather than technical layers.
- Each feature should be a self-contained module that represents a distinct aspect of the game, including all relevant scripts, scenes, and assets.

Example structure:

```
/src
    /features
        /combat
            /scripts
                combat_controller.gd
                combat_service.gd
                combat_repository.gd
                combat_data_store.gd
            /scenes
                combat_arena.tscn
                weapon_selector.tscn
            /assets
                /textures
                    sword_icon.png
                    shield_icon.png
                /audio
                    sword_clash.wav
        /inventory
            /scripts
                inventory_controller.gd
                inventory_service.gd
                inventory_repository.gd
                inventory_data_store.gd
            /scenes
                inventory_ui.tscn
                item_detail.tscn
            /assets
                /textures
                    inventory_background.png
                    item_icons/
        /quest
            /scripts
                quest_controller.gd
                quest_service.gd
                quest_repository.gd
                quest_data_store.gd
            /scenes
                quest_log.tscn
                quest_marker.tscn
            /assets
                /textures
                    quest_icons/
                /audio
                    quest_complete.wav
    /core
        /utils
        /config
        /global_services
    /shared_assets
        /fonts
        /common_textures
```

Benefits:

- Easier navigation and maintenance of the entire project, not just code
- Clearer separation of concerns between different game systems
- Facilitates parallel development by different team members
- Keeps related assets and scenes close to the scripts that use them
- Makes it easier to package or remove entire features if needed
- Improves asset management and reduces the risk of unused assets

Additional considerations:

- Use a `shared_assets` folder for truly global assets used across multiple features
- Consider using symbolic links or Godot's `ProjectSettings.globalize_path()` for shared assets to avoid duplication
- Establish clear guidelines for when an asset belongs to a specific feature vs. the shared assets

This organization method extends beyond just scripts to encompass the entire feature, including its UI elements (scenes) and graphical/audio resources (assets). This holistic approach to feature organization can significantly improve project maintainability and scalability.

### 2.2 Modularity

- Each feature should be self-contained with minimal dependencies on other features.
- Use interfaces and dependency injection to manage necessary inter-feature communication.

Example:

```gdscript
# Interface for inventory interaction
class_name InventoryInterface
extends Node

func add_item(item: Item) -> bool:
    pass

func remove_item(item: Item) -> bool:
    pass

# Combat service using inventory interface
class CombatService:
    var inventory: InventoryInterface

    func _init(p_inventory: InventoryInterface):
        inventory = p_inventory

    func use_health_potion():
        var health_potion = Item.new("Health Potion")
        if inventory.remove_item(health_potion):
            player.heal(50)
```

Benefits:

- Easier to test individual features in isolation
- Simplifies adding or removing features
- Reduces risk of changes in one feature affecting others

### 2.3 Separation of Concerns

- Clearly define responsibilities for each component within a feature.
- Use a consistent structure across features (e.g., Controller, Service, Repository, DataStore).

Example:

```gdscript
# Controller: Handles UI and input
class InventoryController:
    var inventory_service: InventoryService

    func _on_use_item_button_pressed(item: Item):
        inventory_service.use_item(item)

# Service: Contains business logic
class InventoryService:
    var inventory_repository: InventoryRepository

    func use_item(item: Item):
        if inventory_repository.remove_item(item):
            apply_item_effect(item)

# Repository: Manages data access
class InventoryRepository:
    var data_store: InventoryDataStore

    func remove_item(item: Item) -> bool:
        return data_store.remove_item(item.id)

# DataStore: Handles data persistence
class InventoryDataStore:
    func remove_item(item_id: String) -> bool:
        # Implementation for removing item from persistent storage
        pass
```

Benefits:

- Improves code organization and readability
- Facilitates unit testing of individual components
- Makes it easier to modify or replace specific parts of a feature

### 2.4 Scalability

- Design for easy addition of new features and expansion of existing ones.
- Use patterns like Strategy and Observer to allow for future extensibility.

Example of extensible enemy behavior:

```gdscript
class_name EnemyBehavior
extends Node

func perform_action(enemy: Enemy):
    pass

class PatrolBehavior extends EnemyBehavior:
    func perform_action(enemy: Enemy):
        # Implement patrol logic

class AggressiveBehavior extends EnemyBehavior:
    func perform_action(enemy: Enemy):
        # Implement aggressive attack logic

class Enemy:
    var behavior: EnemyBehavior

    func set_behavior(new_behavior: EnemyBehavior):
        behavior = new_behavior

    func update():
        behavior.perform_action(self)
```

Benefits:

- Easier to add new types of enemies or behaviors
- Allows for dynamic behavior changes during runtime
- Simplifies balancing and tweaking of game mechanics

### 2.5 Testability

- Ensure all components can be easily unit tested.
- Use dependency injection to facilitate mocking of dependencies.

Example:

```gdscript
class_name PlayerService
extends Node

var health_repository: HealthRepository

func _init(p_health_repository: HealthRepository):
    health_repository = p_health_repository

func take_damage(amount: int):
    var current_health = health_repository.get_health()
    var new_health = max(0, current_health - amount)
    health_repository.set_health(new_health)
    if new_health == 0:
        emit_signal("player_died")

# In a test file
func test_player_dies_when_health_reaches_zero():
    var mock_health_repo = MockHealthRepository.new()
    mock_health_repo.set_health(100)

    var player_service = PlayerService.new(mock_health_repo)
    var death_signal_received = false
    player_service.connect("player_died", self, "on_player_died")

    player_service.take_damage(100)

    assert(death_signal_received)
    assert_eq(mock_health_repo.get_health(), 0)
```

Benefits:

- Increases confidence in code correctness
- Facilitates refactoring and feature additions
- Serves as documentation for expected component behavior

### 2.6 Flexibility

- Allow for different implementation strategies within the same architectural framework.
- Use interfaces and abstract classes to define contracts between components.

Example:

```gdscript
class_name SaveSystem
extends Node

func save_game(data: Dictionary):
    pass

func load_game() -> Dictionary:
    pass

class FileSaveSystem extends SaveSystem:
    func save_game(data: Dictionary):
        # Implementation for file-based saving

    func load_game() -> Dictionary:
        # Implementation for file-based loading

class CloudSaveSystem extends SaveSystem:
    func save_game(data: Dictionary):
        # Implementation for cloud-based saving

    func load_game() -> Dictionary:
        # Implementation for cloud-based loading

# Usage
var save_system: SaveSystem

func _ready():
    if OS.has_feature("mobile"):
        save_system = CloudSaveSystem.new()
    else:
        save_system = FileSaveSystem.new()
```

Benefits:

- Allows for platform-specific implementations
- Facilitates A/B testing of different strategies
- Makes it easier to swap out implementations as requirements change

By adhering to these expanded core principles, developers can create a robust, maintainable, and scalable game architecture that can adapt to changing requirements and grow with the project's needs.

## 3. Architecture Overview

The architecture is built around feature modules, each containing:

- Controller: Handles UI and input
- Service: Contains business logic
- Repository: Manages data access
- DataStore: Handles data persistence

Global components include:

- EventBus: Manages cross-feature communication
- ServiceLocator: Provides centralized service access
- DataStoreManager: Coordinates data persistence across features

## 4. Component Breakdown

### Controller

- Responsibility: UI management and input handling
- Guidelines:
  - Keep UI logic separate from business logic
  - Use dependency injection to access services
  - Emit signals for local communication within the feature

### Service

- Responsibility: Business logic and feature-specific operations
- Guidelines:
  - Implement core feature functionality here
  - Use repositories for data access
  - Avoid direct UI manipulation

### Repository

- Responsibility: Data access and management
- Guidelines:
  - Abstract data operations from services
  - Handle in-memory data storage and retrieval
  - Coordinate with DataStore for persistence

### DataStore

- Responsibility: Data persistence
- Guidelines:
  - Handle saving and loading of feature-specific data
  - Implement caching strategies where appropriate
  - Use appropriate storage methods (files, databases) based on data characteristics

## 5. Communication Patterns

- Dependency Injection: Use for connecting components within a feature
- Direct Signals: For local communication within a feature
- EventBus: For cross-feature communication and global state changes
- Service Calls: For direct communication between services when necessary

## 6. Data Management

- Each feature should have its own data management strategy
- Use repositories for in-memory data handling
- Implement DataStores for persistent storage
- Consider different storage strategies based on data type and access patterns
- Use a DataStoreManager for coordinating game-wide data operations

## 7. Best Practices

- Single Responsibility Principle: Each class should have one primary responsibility
- Dependency Inversion: Depend on abstractions, not concretions
- Open/Closed Principle: Open for extension, closed for modification
- Don't Repeat Yourself (DRY): Avoid code duplication
- KISS (Keep It Simple, Stupid): Favor simple solutions over complex ones
- Composition over Inheritance: Use composition to build complex behaviors
- Fail Fast: Detect and report errors as early as possible

## 7. Best Practices (Expanded)

### 7.1 Single Responsibility Principle (SRP)

- Each class should have one primary responsibility
- Example: Separate player movement logic from player health management

```gdscript
# Good: Separate classes for movement and health
class PlayerMovement:
    func move(direction: Vector2) -> void:
        # Movement logic here

class PlayerHealth:
    func take_damage(amount: int) -> void:
        # Health management logic here

# Bad: Mixing responsibilities
class Player:
    func move(direction: Vector2) -> void:
        # Movement logic here

    func take_damage(amount: int) -> void:
        # Health management logic here
```

### 7.2 Dependency Inversion Principle

- Depend on abstractions, not concretions
- Use interfaces or abstract classes to define contracts

```gdscript
# Good: Depending on an interface
class_name WeaponInterface
extends Node

func attack() -> void:
    pass

class Sword extends WeaponInterface:
    func attack() -> void:
        print("Sword attack!")

class Bow extends WeaponInterface:
    func attack() -> void:
        print("Bow attack!")

class Player:
    var weapon: WeaponInterface

    func set_weapon(new_weapon: WeaponInterface) -> void:
        weapon = new_weapon

    func attack() -> void:
        weapon.attack()

# Usage
var player = Player.new()
player.set_weapon(Sword.new())
player.attack()  # Outputs: Sword attack!
player.set_weapon(Bow.new())
player.attack()  # Outputs: Bow attack!
```

### 7.3 Open/Closed Principle

- Open for extension, closed for modification
- Use inheritance or composition to add new functionality

```gdscript
# Good: Using inheritance for extensibility
class Enemy:
    func take_damage(amount: int) -> void:
        print("Enemy took ", amount, " damage")

class BossEnemy extends Enemy:
    func take_damage(amount: int) -> void:
        super.take_damage(amount / 2)  # Bosses take half damage
        print("Boss unleashes special attack!")

# Usage
var enemy = Enemy.new()
enemy.take_damage(10)  # Outputs: Enemy took 10 damage

var boss = BossEnemy.new()
boss.take_damage(10)  # Outputs: Enemy took 5 damage \n Boss unleashes special attack!
```

### 7.4 Don't Repeat Yourself (DRY)

- Avoid code duplication
- Extract common functionality into reusable functions or classes
- Be cautious of over-application in a feature-based architecture

```gdscript
# Bad: Repeating code
func process_player_input():
    if Input.is_action_pressed("move_right"):
        player.position.x += player.speed * delta
    if Input.is_action_pressed("move_left"):
        player.position.x -= player.speed * delta
    if Input.is_action_pressed("move_up"):
        player.position.y -= player.speed * delta
    if Input.is_action_pressed("move_down"):
        player.position.y += player.speed * delta

# Good: Extracting common functionality
func process_player_input():
    var movement = Vector2.ZERO
    movement.x = Input.get_action_strength("move_right") - Input.get_action_strength("move_left")
    movement.y = Input.get_action_strength("move_down") - Input.get_action_strength("move_up")
    player.position += movement.normalized() * player.speed * delta
```

#### Note on DRY in Feature-Based Architecture:

While DRY is a valuable principle, it's important to apply it judiciously in a feature-based system. It's acceptable and often beneficial to have similar code between features. This approach can enhance the isolation and independence of features, making them easier to update or modify without risking unintended effects on other parts of the system.

For example, if two features have similar but not identical functionality:

```gdscript
# Feature A
func process_enemy_movement_feature_a(enemy):
    enemy.position += enemy.direction * enemy.speed * delta
    if enemy.position.x > screen_width:
        enemy.direction.x *= -1

# Feature B
func process_enemy_movement_feature_b(enemy):
    enemy.position += enemy.direction * enemy.speed * delta
    if enemy.position.x < 0 or enemy.position.x > screen_width:
        enemy.direction.x *= -1
```

In this case, while the functions are similar, keeping them separate allows each feature to evolve independently. If Feature A needs to change its boundary condition later, it can do so without affecting Feature B.

The key is to balance the DRY principle with the need for feature isolation. Apply DRY within features, but be cautious about applying it across feature boundaries unless the shared functionality is truly universal and unlikely to diverge.

### 7.5 KISS (Keep It Simple, Stupid)

- Favor simple solutions over complex ones
- Break down complex problems into smaller, manageable parts

```gdscript
# Bad: Overly complex inventory system
class Inventory:
    var items = []
    var weights = []
    var values = []

    func add_item(item, weight, value):
        items.append(item)
        weights.append(weight)
        values.append(value)

    func remove_item(index):
        items.pop_at(index)
        weights.pop_at(index)
        values.pop_at(index)

# Good: Simplified inventory system
class Item:
    var name: String
    var weight: float
    var value: int

    func _init(p_name: String, p_weight: float, p_value: int):
        name = p_name
        weight = p_weight
        value = p_value

class Inventory:
    var items = []

    func add_item(item: Item):
        items.append(item)

    func remove_item(index: int):
        items.pop_at(index)
```

### 7.6 Composition over Inheritance

- Use composition to build complex behaviors
- Favor object composition over class inheritance when possible

```gdscript
# Bad: Using inheritance for different enemy types
class Enemy:
    func attack():
        pass

class FlyingEnemy extends Enemy:
    func fly():
        pass

class SwimmingEnemy extends Enemy:
    func swim():
        pass

# Good: Using composition for flexible enemy behaviors
class Enemy:
    var movement_behavior
    var attack_behavior

    func perform_movement():
        movement_behavior.move()

    func perform_attack():
        attack_behavior.attack()

class FlyingMovement:
    func move():
        print("Flying through the air")

class SwimmingMovement:
    func move():
        print("Swimming in water")

class MeleeAttack:
    func attack():
        print("Performing melee attack")

class RangedAttack:
    func attack():
        print("Performing ranged attack")

# Usage
var flying_melee_enemy = Enemy.new()
flying_melee_enemy.movement_behavior = FlyingMovement.new()
flying_melee_enemy.attack_behavior = MeleeAttack.new()

var swimming_ranged_enemy = Enemy.new()
swimming_ranged_enemy.movement_behavior = SwimmingMovement.new()
swimming_ranged_enemy.attack_behavior = RangedAttack.new()
```

### 7.7 Fail Fast

- Detect and report errors as early as possible
- Use assertions and error checking to catch issues early in development

```gdscript
func divide(a: int, b: int) -> float:
    # Good: Checking for division by zero early
    assert(b != 0, "Cannot divide by zero")
    return a as float / b

func process_user_input(input: String) -> void:
    # Good: Validating input early
    if input.empty():
        printerr("User input cannot be empty")
        return

    # Process valid input...

```

### 7.8 Use Constants for Magic Values

- Avoid hardcoding values directly in the code
- Use constants to improve readability and maintainability

```gdscript
# Bad: Using magic numbers
func calculate_damage(base_damage: int) -> int:
    return base_damage * 1.5 if is_critical_hit else base_damage

# Good: Using constants
const CRITICAL_HIT_MULTIPLIER = 1.5

func calculate_damage(base_damage: int) -> int:
    return base_damage * CRITICAL_HIT_MULTIPLIER if is_critical_hit else base_damage
```

## 8. Scalability and Extensibility

- Design features as self-contained modules
- Use interfaces and abstract classes to define contracts between components
- Implement feature flags for easy enabling/disabling of features
- Design systems to be easily expandable (e.g., new monster types, quest types)
- Use configuration files for easily modifiable game parameters

## 9. Testing Strategies

- Unit Testing: Test individual components in isolation
- Integration Testing: Test interactions between components within a feature
- System Testing: Test entire features and cross-feature interactions
- Mock dependencies to isolate components during testing
- Use dependency injection to facilitate easier testing
- Implement continuous integration to run tests automatically

## 10. Performance Considerations

- Profile code regularly to identify bottlenecks
- Optimize data structures and algorithms in performance-critical sections
- Implement object pooling for frequently created/destroyed objects
- Use appropriate data storage methods (in-memory, disk, database) based on access patterns
- Consider multi-threading for performance-intensive operations
- Implement level-of-detail (LOD) systems for complex game worlds

## 11. Common Pitfalls and How to Avoid Them

- Tight Coupling: Use dependency injection and stick to defined interfaces
- God Objects: Break down large, multi-purpose classes into smaller, focused ones
- Feature Creep: Clearly define feature scope and use agile methodologies
- Premature Optimization: Profile first, optimize where necessary
- Inconsistent Style: Use a style guide and automated formatting tools
- Poor Documentation: Document as you go, use clear naming conventions

## 12. Case Studies and Examples

This section provides concrete examples of how our feature-based architecture can be implemented in real-world scenarios. We'll explore three different features: a Quest System, an Inventory System, and a Crafting System. Each example will demonstrate key aspects of our architecture, incorporating practices from the provided code.

### 12.1 Quest System

The Quest System manages available quests, tracks progress, and handles completion.

#### QuestController

```gdscript
extends Control

@onready var quest_list = $QuestList
@onready var accept_button = $AcceptButton

var quest_service: QuestService

func _init(q_quest_service: QuestService):
    quest_service = q_quest_service

func _ready():
    accept_button.connect("pressed", self, "_on_accept_button_pressed")
    quest_service.connect("active_quests_changed", self, "_on_active_quests_changed")
    update_quest_list()

func _on_accept_button_pressed():
    var selected_quest = quest_list.get_selected_quest()
    if selected_quest:
        var success = quest_service.accept_quest(selected_quest)
        if success:
            update_quest_list()

func _on_active_quests_changed(active_quests):
    update_quest_list()

func update_quest_list():
    var quests = quest_service.get_all_quests()
    quest_list.update_quests(quests)
```

#### QuestService

```gdscript
class_name QuestService
extends Node

signal quest_updated(quest: Quest)
signal active_quests_changed(active_quests: Array[Quest])

var quest_repository: QuestRepository
var player_service: PlayerService

func _init(q_quest_repo: QuestRepository, p_player_service: PlayerService) -> void:
    quest_repository = q_quest_repo
    player_service = p_player_service

func get_quest(quest_id: String) -> Quest:
    return quest_repository.fetch_by_id(quest_id)

func get_all_quests() -> Array[Quest]:
    return quest_repository.fetch_all()

func get_active_quests() -> Array[Quest]:
    return quest_repository.fetch_active_quests()

func accept_quest(quest: Quest) -> bool:
    if player_service.can_accept_quest(quest):
        quest_repository.add_active_quest(quest)
        active_quests_changed.emit(get_active_quests())
        save_changes()
        return true
    return false

func complete_quest(quest: Quest) -> void:
    quest_repository.remove_active_quest(quest.id)
    player_service.add_experience(quest.reward_experience)
    player_service.add_money(quest.reward_money)
    active_quests_changed.emit(get_active_quests())
    quest_updated.emit(quest)
    save_changes()

func save_changes() -> void:
    quest_repository.persist()
```

#### QuestRepository

```gdscript
class_name QuestRepository
extends Node

var _data_store: QuestDataStore
var _cache: Dictionary = {}
var _active_quests_cache: Dictionary = {}
var _is_cache_valid: bool = false

func _init(data_store: QuestDataStore) -> void:
    _data_store = data_store

func load_all() -> void:
    _cache = _data_store.quests.duplicate(true)
    _active_quests_cache = _data_store.active_quests.duplicate(true)
    _is_cache_valid = true

func fetch_by_id(quest_id: String) -> Quest:
    if not _is_cache_valid:
        load_all()
    return _cache.get(quest_id)

func fetch_all() -> Array[Quest]:
    if not _is_cache_valid:
        load_all()
    var all: Array[Quest] = []
    all.append_array(_cache.values())
    return all

func fetch_active_quests() -> Array[Quest]:
    if not _is_cache_valid:
        load_all()
    var active: Array[Quest] = []
    active.append_array(_active_quests_cache.values())
    return active

func add_active_quest(quest: Quest) -> void:
    _active_quests_cache[quest.id] = quest
    _is_cache_valid = true

func remove_active_quest(quest_id: String) -> void:
    _active_quests_cache.erase(quest_id)

func persist() -> void:
    _data_store.quests = _cache.duplicate(true)
    _data_store.active_quests = _active_quests_cache.duplicate(true)
    _data_store.save()
```

#### QuestDataStore

```gdscript
class_name QuestDataStore
extends Resource

const SAVE_PATH = "user://data/quests.tres"
const CURRENT_VERSION = 1

@export var quests: Dictionary = {}
@export var active_quests: Dictionary = {}
@export var version: int = CURRENT_VERSION

static func load_or_create() -> QuestDataStore:
    if FileAccess.file_exists(SAVE_PATH):
        var store: QuestDataStore = ResourceLoader.load(SAVE_PATH) as QuestDataStore
        if store:
            store._migrate_store_if_necessary()
            return store

    var new_store: QuestDataStore = QuestDataStore.new()
    new_store.save()
    return new_store

func save() -> void:
    ResourceSaver.save(self, SAVE_PATH)

func clear() -> void:
    quests.clear()
    active_quests.clear()
    save()

func _migrate_store_if_necessary() -> void:
    if version < CURRENT_VERSION:
        match version:
            1:
                _migrate_store_to_v2()

        version = CURRENT_VERSION
        save()

func _migrate_store_to_v2() -> void:
    # Example: Perform store-wide migration
    print("Migrated QuestDataStore to version 2")
```

### 12.2 Inventory System

The Inventory System manages the player's items, including adding, removing, and using items.

#### InventoryDataStore

```gdscript
class_name InventoryDataStore
extends Resource

const SAVE_PATH = "user://data/inventory.tres"
const CURRENT_VERSION = 1

@export var items: Dictionary = {}
@export var version: int = CURRENT_VERSION

static func load_or_create() -> InventoryDataStore:
    if FileAccess.file_exists(SAVE_PATH):
        var store: InventoryDataStore = ResourceLoader.load(SAVE_PATH) as InventoryDataStore
        if store:
            store._migrate_store_if_necessary()
            return store

    var new_store: InventoryDataStore = InventoryDataStore.new()
    new_store.save()
    return new_store

func save() -> void:
    ResourceSaver.save(self, SAVE_PATH)

func clear() -> void:
    items.clear()
    save()

func _migrate_store_if_necessary() -> void:
    if version < CURRENT_VERSION:
        match version:
            1:
                _migrate_store_to_v2()

        version = CURRENT_VERSION
        save()

func _migrate_store_to_v2() -> void:
    # Example: Perform store-wide migration
    print("Migrated InventoryDataStore to version 2")
```

#### InventoryController

```gdscript
extends Control

@onready var item_list = $ItemList
@onready var use_button = $UseButton

var inventory_service: InventoryService

func _init(i_inventory_service: InventoryService):
    inventory_service = i_inventory_service

func _ready():
    use_button.connect("pressed", self, "_on_use_button_pressed")
    inventory_service.connect("inventory_updated", self, "_on_inventory_updated")
    update_inventory_display()

func _on_use_button_pressed():
    var selected_item = item_list.get_selected_item()
    if selected_item:
        inventory_service.use_item(selected_item)

func _on_inventory_updated(items):
    update_inventory_display()

func update_inventory_display():
    var items = inventory_service.get_all_items()
    item_list.clear()
    for item in items:
        item_list.add_item(item.name, item.icon)
```

#### InventoryService

```gdscript
class_name InventoryService
extends Node

signal inventory_updated(items: Array[Item])

var inventory_repository: InventoryRepository
var player_service: PlayerService

func _init(i_inventory_repo: InventoryRepository, p_player_service: PlayerService) -> void:
    inventory_repository = i_inventory_repo
    player_service = p_player_service

func add_item(item: Item) -> void:
    inventory_repository.add(item)
    inventory_updated.emit(get_all_items())
    save_changes()

func remove_item(item_id: String) -> void:
    inventory_repository.remove(item_id)
    inventory_updated.emit(get_all_items())
    save_changes()

func use_item(item: Item) -> void:
    if item.can_be_used():
        player_service.apply_item_effect(item)
        remove_item(item.id)

func get_all_items() -> Array[Item]:
    return inventory_repository.fetch_all()

func save_changes() -> void:
    inventory_repository.persist()
```

#### InventoryRepository

```gdscript
class_name InventoryRepository
extends Node

var _data_store: InventoryDataStore
var _cache: Dictionary = {}
var _is_cache_valid: bool = false

func _init(data_store: InventoryDataStore) -> void:
    _data_store = data_store

func load_all() -> void:
    _cache = _data_store.items.duplicate(true)
    _is_cache_valid = true

func fetch_by_id(item_id: String) -> Item:
    if not _is_cache_valid:
        load_all()
    return _cache.get(item_id)

func fetch_all() -> Array[Item]:
    if not _is_cache_valid:
        load_all()
    var all: Array[Item] = []
    all.append_array(_cache.values())
    return all

func add(item: Item) -> void:
    _cache[item.id] = item
    _is_cache_valid = true

func remove(item_id: String) -> void:
    _cache.erase(item_id)

func persist() -> void:
    _data_store.items = _cache.duplicate(true)
    _data_store.save()
```

### 12.3 Crafting System

The Crafting System allows players to combine items to create new ones.

#### CraftingController

```gdscript
extends Control

@onready var recipe_list = $RecipeList
@onready var craft_button = $CraftButton

var crafting_service: CraftingService

func _init(c_crafting_service: CraftingService):
    crafting_service = c_crafting_service

func _ready():
    craft_button.connect("pressed", self, "_on_craft_button_pressed")
    crafting_service.connect("crafting_result", self, "_on_crafting_result")
    update_recipe_list()

func _on_craft_button_pressed():
    var selected_recipe = recipe_list.get_selected_recipe()
    if selected_recipe:
        crafting_service.craft_item(selected_recipe)

func _on_crafting_result(result):
    if result.success:
        update_recipe_list()
        # Show success message to the player
    else:
        # Show failure message to the player

func update_recipe_list():
    var recipes = crafting_service.get_available_recipes()
    recipe_list.clear()
    for recipe in recipes:
        recipe_list.add_item(recipe.name, recipe.icon)
```

#### CraftingService

```gdscript
class_name CraftingService
extends Node

signal crafting_result(result: Dictionary)

var crafting_repository: CraftingRepository
var inventory_service: InventoryService
var player_service: PlayerService

func _init(c_crafting_repo: CraftingRepository, i_inventory_service: InventoryService, p_player_service: PlayerService) -> void:
    crafting_repository = c_crafting_repo
    inventory_service = i_inventory_service
    player_service = p_player_service

func craft_item(recipe: Recipe) -> Dictionary:
    if can_craft(recipe):
        for ingredient in recipe.ingredients:
            inventory_service.remove_item(ingredient.id)
        var crafted_item = create_item(recipe)
        inventory_service.add_item(crafted_item)
        player_service.add_experience(recipe.experience_reward)
        var result = {"success": true, "item": crafted_item}
        crafting_result.emit(result)
        return result
    return {"success": false, "reason": "Missing ingredients or insufficient skill"}

func can_craft(recipe: Recipe) -> bool:
    var player_items = inventory_service.get_all_items()
    for ingredient in recipe.ingredients:
        if not player_items.has(ingredient):
            return false
    return player_service.has_required_skill(recipe.required_skill)

func get_available_recipes() -> Array[Recipe]:
    var all_recipes = crafting_repository.fetch_all()
    return all_recipes.filter(func(recipe): return can_craft(recipe))

func create_item(recipe: Recipe) -> Item:
    # Logic to create the item, potentially with variable stats based on player skill
    var new_item = Item.new()
    # Set properties based on recipe and player skill
    return new_item
```

#### CraftingRepository

```gdscript
class_name CraftingRepository
extends Node

var _data_store: CraftingDataStore
var _cache: Dictionary = {}
var _is_cache_valid: bool = false

func _init(data_store: CraftingDataStore) -> void:
    _data_store = data_store

func load_all() -> void:
    _cache = _data_store.recipes.duplicate(true)
    _is_cache_valid = true

func fetch_by_id(recipe_id: String) -> Recipe:
    if not _is_cache_valid:
        load_all()
    return _cache.get(recipe_id)

func fetch_all() -> Array[Recipe]:
    if not _is_cache_valid:
        load_all()
    var all: Array[Recipe] = []
    all.append_array(_cache.values())
    return all

func persist() -> void:
    _data_store.recipes = _cache.duplicate(true)
    _data_store.save()
```

#### CraftingDataStore

```gdscript
class_name CraftingDataStore
extends Resource

const SAVE_PATH = "user://data/crafting.tres"
const CURRENT_VERSION = 1

@export var recipes: Dictionary = {}
@export var version: int = CURRENT_VERSION

static func load_or_create() -> CraftingDataStore:
    if FileAccess.file_exists(SAVE_PATH):
        var store: CraftingDataStore = ResourceLoader.load(SAVE_PATH) as CraftingDataStore
        if store:
            store._migrate_store_if_necessary()
            return store

    var new_store: CraftingDataStore = CraftingDataStore.new()
    new_store.save()
    return new_store

func save() -> void:
    ResourceSaver.save(self, SAVE_PATH)

func clear() -> void:
    recipes.clear()
    save()

func _migrate_store_if_necessary() -> void:
    if version < CURRENT_VERSION:
        match version:
            1:
                _migrate_store_to_v2()

        version = CURRENT_VERSION
        save()

func _migrate_store_to_v2() -> void:
    # Example: Perform store-wide migration
    print("Migrated CraftingDataStore to version 2")
```

These examples demonstrate how our feature-based architecture promotes:

1. Modularity: Each feature (Quest, Inventory, Crafting) is self-contained but can interact with others through well-defined interfaces.

2. Separation of Concerns:

   - Controllers handle UI and user input
   - Services contain business logic
   - Repositories manage data access
   - DataStores handle data persistence and versioning

3. Reusability: Services like PlayerService are used across multiple features.

4. Scalability: New quests, items, or recipes can be easily added without changing the core structure.

5. Testability: Each component can be easily unit tested in isolation.

6. Data Persistence: Consistent use of DataStore classes for saving and loading data, with built-in versioning and migration support.

7. Caching: Repositories implement caching mechanisms to improve performance.

8. Event-driven Architecture: Use of signals for local and cross-component communication.

9. Dependency Injection: Services and repositories are injected into controllers and other services, promoting loose coupling.

10. Version Control for Data: Each DataStore includes version tracking and migration methods, allowing for easy updates to data structures as the game evolves.

By following these patterns, you can create complex game systems that are maintainable, extensible, and robust. The architecture allows for easy addition of new features and modification of existing ones, while ensuring that data can be safely persisted and migrated as the game evolves over time.

This approach also facilitates collaborative development, as different team members can work on separate features with minimal conflicts. The clear separation of concerns makes it easier to understand and maintain each part of the system independently.

Remember that while this architecture provides a solid foundation, it should be adapted as necessary to meet the specific needs of your project. Always consider performance implications, especially for mobile or resource-constrained platforms, and be prepared to optimize where necessary.

### 12.4 Pong Game System

This example demonstrates how to implement a simple Pong game using our feature-based architecture. The game will include player controls, AI opponent, score tracking, and high score saving.

#### PongDataStore

```gdscript
class_name PongDataStore
extends Resource

const SAVE_PATH = "user://data/pong_data.tres"
const CURRENT_VERSION = 1

@export var high_score: int = 0
@export var version: int = CURRENT_VERSION

static func load_or_create() -> PongDataStore:
    if FileAccess.file_exists(SAVE_PATH):
        var store: PongDataStore = ResourceLoader.load(SAVE_PATH) as PongDataStore
        if store:
            store._migrate_store_if_necessary()
            return store

    var new_store: PongDataStore = PongDataStore.new()
    new_store.save()
    return new_store

func save() -> void:
    ResourceSaver.save(self, SAVE_PATH)

func _migrate_store_if_necessary() -> void:
    if version < CURRENT_VERSION:
        match version:
            1:
                _migrate_store_to_v2()

        version = CURRENT_VERSION
        save()

func _migrate_store_to_v2() -> void:
    # Example: Add a new property for total games played
    # self.total_games_played = 0
    print("Migrated PongDataStore to version 2")
```

#### PongRepository

```gdscript
class_name PongRepository
extends Node

var _data_store: PongDataStore
var _high_score: int = 0
var _player_score: int = 0
var _ai_score: int = 0
var _is_cache_valid: bool = false

func _init(data_store: PongDataStore) -> void:
    _data_store = data_store

func load_high_score() -> void:
    _high_score = _data_store.high_score
    _is_cache_valid = true

func get_high_score() -> int:
    if not _is_cache_valid:
        load_high_score()
    return _high_score

func update_high_score(score: int) -> void:
    if score > _high_score:
        _high_score = score
        _data_store.high_score = score
        _data_store.save()

func get_player_score() -> int:
    return _player_score

func get_ai_score() -> int:
    return _ai_score

func update_player_score(score: int) -> void:
    _player_score = score

func update_ai_score(score: int) -> void:
    _ai_score = score

func reset_scores() -> void:
    _player_score = 0
    _ai_score = 0

func persist() -> void:
    _data_store.high_score = _high_score
    _data_store.save()
```

#### PongService

```gdscript
class_name PongService
extends Node

signal score_updated(player_score: int, ai_score: int)
signal game_over(final_score: int, is_high_score: bool)

var pong_repository: PongRepository
var audio_manager: AudioManager

func _init(p_pong_repo: PongRepository, p_audio_manager: AudioManager) -> void:
    pong_repository = p_pong_repo
    audio_manager = p_audio_manager

func start_new_game() -> void:
    pong_repository.reset_scores()
    score_updated.emit(pong_repository.get_player_score(), pong_repository.get_ai_score())

func update_score(is_player_point: bool) -> void:
    if is_player_point:
        pong_repository.update_player_score(pong_repository.get_player_score() + 1)
    else:
        pong_repository.update_ai_score(pong_repository.get_ai_score() + 1)

    audio_manager.play_score()
    score_updated.emit(pong_repository.get_player_score(), pong_repository.get_ai_score())

    if pong_repository.get_player_score() >= 11 or pong_repository.get_ai_score() >= 11:
        end_game()

func end_game() -> void:
    var player_score = pong_repository.get_player_score()
    var high_score = pong_repository.get_high_score()
    var is_new_high_score = player_score > high_score
    if is_new_high_score:
        pong_repository.update_high_score(player_score)
    game_over.emit(player_score, is_new_high_score)

func get_high_score() -> int:
    return pong_repository.get_high_score()

func get_player_score() -> int:
    return pong_repository.get_player_score()

func get_ai_score() -> int:
    return pong_repository.get_ai_score()


func move_player_paddle(delta: float) -> void:
    # Implement player paddle movement logic

func update_ai_paddle(ball_position: Vector2, delta: float) -> void:
    # Implement AI paddle movement logic

func update_ball_position(delta: float) -> void:
    # Implement ball movement and collision logic
```

#### PongController

```gdscript
extends Node2D

@onready var player_paddle = $PlayerPaddle
@onready var ai_paddle = $AIPaddle
@onready var ball = $Ball
@onready var score_label = $ScoreLabel
@onready var high_score_label = $HighScoreLabel

var pong_service: PongService

func _init(p_pong_service: PongService):
    pong_service = p_pong_service

func _ready():
    pong_service.connect("score_updated", self, "_on_score_updated")
    pong_service.connect("game_over", self, "_on_game_over")
    start_new_game()

func _process(delta):
    pong_service.move_player_paddle(delta)
    pong_service.update_ai_paddle(ball.position, delta)
    pong_service.update_ball_position(delta)

func _on_score_updated(player_score: int, ai_score: int):
    score_label.text = str(player_score) + " - " + str(ai_score)

func _on_game_over(final_score: int, is_high_score: bool):
    if is_high_score:
        high_score_label.text = "New High Score: " + str(final_score)
    # Show game over screen, option to restart, etc.

func start_new_game():
    pong_service.start_new_game()
    high_score_label.text = "High Score: " + str(pong_service.get_high_score())
    # Reset paddle and ball positions
```

This Pong game example demonstrates:

1. Separation of Concerns:

   - PongController handles the game's visual elements and user input.
   - PongService contains the game logic, including score tracking and game state.
   - PongRepository manages data access and persistence.
   - PongDataStore handles saving and loading of high scores, with version control.

2. Data Persistence: The high score is saved to disk and can be loaded when the game starts.

3. Scalability: It's easy to add new features, such as different AI difficulties or power-ups, by extending the PongService.

4. Testability: Each component (especially PongService) can be unit tested in isolation.

5. Event-driven Architecture: The use of signals (score_updated, game_over) allows for loose coupling between the service and controller.

6. Version Control for Data: The PongDataStore includes version tracking and migration methods, allowing for easy updates to saved data structure as the game evolves.

This architecture allows for easy expansion of the game. For example, you could add:

- Multiple AI difficulties by creating different AI strategy classes.
- A replay system by recording and playing back paddle movements.
- Online multiplayer by extending the PongService to handle network communication.

By following these patterns, even a simple game like Pong can be structured in a way that's maintainable, extensible, and robust.

### 12.4 Pong Game System (continued)

#### Completing the Pong Game

While our example provides the architectural foundation, several components are needed to create a fully functional Pong game. Here's what's missing and how to complete it:

1. Game Objects

   - Ball
   - Player Paddle
   - AI Paddle

2. Game Logic

   - Ball movement and collision detection
   - AI paddle movement
   - Score tracking and win conditions

3. User Interface

   - Start menu
   - In-game UI (score display)
   - Game over screen

4. Sound Effects

Step-by-Step Instructions to Complete the Game:

1. Create Game Objects:

   a. Ball:

   ```gdscript
   class_name Ball
   extends Area2D

   var speed: float = 400.0
   var direction: Vector2 = Vector2.ZERO

   func _ready():
       randomize()
       reset_ball()

   func _process(delta):
       position += direction * speed * delta

   func reset_ball():
       position = Vector2(512, 300)
       direction = Vector2([-1, 1].pick_random(), randf_range(-0.8, 0.8)).normalized()
   ```

   b. Paddle (base class for both player and AI):

   ```gdscript
   class_name Paddle
   extends Area2D

   var speed: float = 400.0

   func move_up(delta: float):
       position.y = max(position.y - speed * delta, 0)

   func move_down(delta: float):
       position.y = min(position.y + speed * delta, 600)
   ```

2. Implement Game Logic in PongService:

   a. Update ball position and handle collisions:

   ```gdscript
   func update_ball_position(delta: float, ball: Ball, player_paddle: Paddle, ai_paddle: Paddle) -> void:
       ball.position += ball.direction * ball.speed * delta

       if ball.position.y <= 0 or ball.position.y >= 600:
           ball.direction.y *= -1

       if ball.overlaps_body(player_paddle) or ball.overlaps_body(ai_paddle):
           ball.direction.x *= -1
           ball.speed += 20  # Increase difficulty

       if ball.position.x <= 0:
           update_score(false)
           ball.reset_ball()
       elif ball.position.x >= 1024:
           update_score(true)
           ball.reset_ball()
   ```

   b. Implement AI paddle movement:

   ```gdscript
   func update_ai_paddle(ball_position: Vector2, ai_paddle: Paddle, delta: float) -> void:
       var paddle_center = ai_paddle.position.y + ai_paddle.height / 2
       if ball_position.y < paddle_center - 10:
           ai_paddle.move_up(delta)
       elif ball_position.y > paddle_center + 10:
           ai_paddle.move_down(delta)
   ```

3. Enhance PongController:

   a. Add input handling for player paddle:

   ```gdscript
   func _process(delta):
       if Input.is_action_pressed("move_up"):
           player_paddle.move_up(delta)
       if Input.is_action_pressed("move_down"):
           player_paddle.move_down(delta)

       pong_service.update_ai_paddle(ball.position, ai_paddle, delta)
       pong_service.update_ball_position(delta, ball, player_paddle, ai_paddle)
   ```

   b. Implement game states (menu, playing, game over):

   ```gdscript
   enum GameState { MENU, PLAYING, GAME_OVER }
   var current_state: GameState = GameState.MENU

   func _ready():
       set_state(GameState.MENU)

   func set_state(new_state: GameState):
       current_state = new_state
       match current_state:
           GameState.MENU:
               show_menu()
           GameState.PLAYING:
               start_new_game()
           GameState.GAME_OVER:
               show_game_over()
   ```

4. Create User Interface:

   a. Start Menu:

   ```gdscript
   func show_menu():
       $MenuContainer.show()
       $GameContainer.hide()
       $GameOverContainer.hide()

   func _on_start_button_pressed():
       set_state(GameState.PLAYING)
   ```

   b. In-game UI:

   ```gdscript
   func update_score_display():
       score_label.text = str(pong_service.player_score) + " - " + str(pong_service.ai_score)
   ```

   c. Game Over Screen:

   ```gdscript
   func show_game_over():
       $GameOverContainer.show()
       $GameContainer.hide()
       $FinalScoreLabel.text = "Final Score: " + str(pong_service.player_score)
       $HighScoreLabel.text = "High Score: " + str(pong_service.get_high_score())

   func _on_play_again_button_pressed():
       set_state(GameState.PLAYING)
   ```

5. Add Sound Effects:

   a. Create an AudioManager:

   ```gdscript
   class_name AudioManager
   extends Node

   var paddle_hit_sound: AudioStream = preload("res://assets/sounds/paddle_hit.wav")
   var wall_hit_sound: AudioStream = preload("res://assets/sounds/wall_hit.wav")
   var score_sound: AudioStream = preload("res://assets/sounds/score.wav")

   func play_paddle_hit():
       $PaddleHitPlayer.stream = paddle_hit_sound
       $PaddleHitPlayer.play()

   func play_wall_hit():
       $WallHitPlayer.stream = wall_hit_sound
       $WallHitPlayer.play()

   func play_score():
       $ScorePlayer.stream = score_sound
       $ScorePlayer.play()
   ```

   b. Integrate AudioManager with PongService:

   ```gdscript
   var audio_manager: AudioManager

   func _init(p_pong_repo: PongRepository, p_audio_manager: AudioManager):
       pong_repository = p_pong_repo
       audio_manager = p_audio_manager

   func update_ball_position(delta: float, ball: Ball, player_paddle: Paddle, ai_paddle: Paddle) -> void:
       # ... existing code ...
       if ball.overlaps_body(player_paddle) or ball.overlaps_body(ai_paddle):
           audio_manager.play_paddle_hit()
       # ... rest of the code ...
   ```

6. Final Integration:

   a. Update PongController to use all new components:

   ```gdscript
   func _ready():
       pong_service.connect("score_updated", self, "_on_score_updated")
       pong_service.connect("game_over", self, "_on_game_over")
       set_state(GameState.MENU)

   func _process(delta):
       if current_state == GameState.PLAYING:
           handle_input(delta)
           pong_service.update_ai_paddle(ball.position, ai_paddle, delta)
           pong_service.update_ball_position(delta, ball, player_paddle, ai_paddle)

   func handle_input(delta):
       if Input.is_action_pressed("move_up"):
           player_paddle.move_up(delta)
       if Input.is_action_pressed("move_down"):
           player_paddle.move_down(delta)
   ```

   b. Ensure all signals are connected and UI elements are updated appropriately.

By following these steps, you'll have a complete, functioning Pong game that adheres to the feature-based architecture. This structure allows for easy expansion, such as adding power-ups, different AI difficulties, or even networked multiplayer in the future.

Remember to create appropriate scenes in Godot, connecting the script instances and setting up the node hierarchy to match the structure outlined in these steps.

### Pong Architecture Diagram

```mermaid
classDiagram
    class PongController {
        -pong_service: PongService
        -player_paddle: Paddle
        -ai_paddle: Paddle
        -ball: Ball
        -score_label: Label
        -high_score_label: Label
        -current_state: GameState
        +_ready()
        +_process(delta)
        +start_new_game()
        +show_menu()
        +show_game_over()
        +handle_input(delta)
    }

    class PongService {
        -pong_repository: PongRepository
        -audio_manager: AudioManager
        +start_new_game()
        +update_score(is_player_point: bool)
        +end_game()
        +get_high_score(): int
        +get_player_score(): int
        +get_ai_score(): int
        +update_ball_position(delta, ball, player_paddle, ai_paddle)
        +update_ai_paddle(ball_position, ai_paddle, delta)
    }

    class PongRepository {
        -_data_store: PongDataStore
        -_high_score: int
        -_player_score: int
        -_ai_score: int
        -_is_cache_valid: bool
        +load_high_score()
        +get_high_score(): int
        +update_high_score(score: int)
        +get_player_score(): int
        +get_ai_score(): int
        +update_player_score(score: int)
        +update_ai_score(score: int)
        +reset_scores()
        +persist()
    }

    class PongDataStore {
        +high_score: int
        +version: int
        +load_or_create(): PongDataStore
        +save()
        -_migrate_store_if_necessary()
    }

    class Ball {
        +speed: float
        +direction: Vector2
        +reset_ball()
        +_process(delta)
    }

    class Paddle {
        +speed: float
        +move_up(delta: float)
        +move_down(delta: float)
    }

    class AudioManager {
        +play_paddle_hit()
        +play_wall_hit()
        +play_score()
    }

    PongController --> PongService : uses
    PongController --> Ball : manages
    PongController --> Paddle : manages
    PongService --> PongRepository : uses
    PongService --> AudioManager : uses
    PongRepository --> PongDataStore : uses
    PongController ..> Paddle : creates
    PongController ..> Ball : creates
```

## 13. Conclusion

This feature-based architecture provides a robust foundation for complex game development projects. By adhering to these principles and guidelines, developers can create maintainable, scalable, and efficient game systems. Remember that this architecture is a guideline and should be adapted as necessary to meet the specific needs of each project.
