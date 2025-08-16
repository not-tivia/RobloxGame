# Comprehensive Lua Systems Documentation

## Core Combat & Spell Systems

### SpellSystem.lua
**Location:** ReplicatedStorage  
**Purpose:** Central spell management system that defines all character class abilities and handles spell validation, cooldowns, and casting logic on both client and server.

**Key Responsibilities:**
- Defines class-specific spell databases with damage, cooldowns, ranges, and targeting requirements
- Manages Global Cooldown (GCD) system with 0.5-second default duration
- Handles spell validation including range checking, target validation, and resource requirements
- Integrates with DebuffManager for applying spell effects like DoTs, slows, and stuns
- Supports multiple targeting modes: single target, ground targeted (AOE), and self-targeted spells
- Manages spell-specific mechanics like Mage Frostbite passive stacking and Monk shield degradation
- Coordinates with AOEZoneTracker for area-of-effect spells like Mage Storm
- Provides damage calculation and application through integrated DamageManager
- Handles networking through GameNetwork for spell cast requests/responses
- Supports event hooks for passive effects and spell interactions

**Integration Points:**
- Uses TargetingModule for target validation and range checking
- Communicates with DebuffManager for applying spell effects
- Integrates with GameNetwork for client/server spell communication
- Coordinates with AOEZoneTracker for ground-targeted spells
- Uses SpellValidator for casting prerequisite checks

### DebuffManager.lua
**Location:** ServerScriptService  
**Purpose:** Server-side manager for all character debuffs including damage-over-time effects, movement slows, stuns, and other temporary status effects.

**Key Responsibilities:**
- Tracks debuff duration, damage per tick, and stack counts for all characters
- Processes damage ticks using heartbeat connections for precise timing
- Synchronizes debuff state to all clients through GameNetwork and legacy events
- Manages debuff stacking mechanics with configurable stack limits
- Handles debuff removal including expiration, cleansing, and manual removal
- Supports complex debuff types with additional data (slow amounts, stun effects, etc.)
- Provides immunity and resistance mechanics
- Integrates with ClientEffectManager for visual effect coordination
- Handles multi-debuff removal operations for cleansing abilities
- Tracks recently notified debuffs to prevent spam

**Network Integration:**
- Sends DEBUFF_APPLIED, DEBUFF_REMOVED, and DEBUFF_CLEANSED messages
- Maintains legacy HitEvent compatibility for older systems
- Provides real-time debuff updates to enemy frames and UI systems

### TargetingModule.lua
**Location:** ReplicatedStorage/Modules  
**Purpose:** Singleton system managing which entity a player is targeting, handling target validation, visual highlighting, and synchronizing target state between client and server.

**Key Responsibilities:**
- Maintains current target state per player with validation monitoring
- Provides target validation including distance checks, line-of-sight, and team alignment
- Handles both enemy and friendly targeting with configurable team checking
- Integrates with AutoAttackModule for automatic combat engagement
- Supports mouseover targeting for healing and friendly spells
- Provides range checking utilities for spells and abilities
- Manages target clearing when targets become invalid or are destroyed
- Synchronizes targeting state through GameNetwork messaging
- Handles server-side target validation and client target requests
- Supports target persistence and auto-engagement features

**Special Features:**
- Friendly targeting mode for healing spells that doesn't disrupt combat targeting
- Automatic target validation with heartbeat monitoring
- Integration with auto-attack for seamless combat flow

### AutoAttackModule.lua
**Location:** ReplicatedStorage/Modules  
**Purpose:** Manages automatic attacking systems with configurable timing, line-of-sight checking, target persistence, and event hook integration.

**Key Responsibilities:**
- Handles auto-attack timing with customizable intervals per character/class
- Manages auto-attack target acquisition and persistence
- Provides line-of-sight checking to pause/resume attacks when vision is blocked
- Integrates with TargetingModule for automatic target engagement
- Supports event hooks for pre/post attack processing
- Manages auto-attack state synchronization between client and server
- Handles auto-attack interruption and resumption based on various conditions
- Provides configurable attack patterns and timing variations
- Supports different attack types (melee, ranged) with appropriate logic
- Manages auto-attack failure handling (out of range, no line of sight, etc.)

**Event System:**
- Pre-attack, post-attack, target-lost, and attack-failed event hooks
- Integration with NetworkBridge for client/server auto-attack coordination
- Automatic pause/resume based on targeting state changes

## Networking & Communication Systems

### GameNetwork.lua
**Location:** ReplicatedStorage  
**Purpose:** Modern message-based networking system with typed categories and message types that standardizes client/server communication with monitoring capabilities.

**Key Responsibilities:**
- Provides structured message categories (COMBAT, SPELL, TARGETING, PLAYER_DATA)
- Implements type-safe message routing with validation
- Supports both server-to-client and client-to-server messaging
- Provides callback registration system for handling specific message types
- Implements message queuing and rate limiting for performance
- Offers monitoring and debugging capabilities for network traffic
- Handles message serialization and deserialization
- Provides fallback mechanisms for network failures
- Supports broadcast messaging to all clients
- Implements player-specific message routing

**Message Categories:**
- **COMBAT:** Damage, auto-attack, debuff application/removal
- **SPELL:** Cast requests, cooldown sync, spell validation
- **TARGETING:** Target requests, confirmations, friendly targeting
- **PLAYER_DATA:** Character stats, progression, cosmetics

### GameNetworkBridge.lua
**Location:** ReplicatedStorage  
**Purpose:** Bridge system that translates between legacy RemoteEvent-based communication and modern GameNetwork messaging for backward compatibility.

**Key Responsibilities:**
- Maps legacy action strings to GameNetwork message categories and types
- Provides transparent translation layer for existing code
- Handles data format conversion between legacy and modern systems
- Supports both FireEvent and HitEvent legacy patterns
- Implements fallback to legacy events when GameNetwork unavailable
- Provides debug logging for network translation operations
- Handles Instance-to-string conversion for network safety
- Supports callback registration for both network types
- Maintains compatibility with existing client/server code

**Action Mappings:**
- CastSpell → SPELL.CAST_REQUEST
- SetTarget → TARGETING.TARGET_REQUEST
- Hit/AutoAttackHit → COMBAT.DAMAGE
- UpdateDebuffs → COMBAT.DEBUFF_APPLIED

### CombatServer.lua
**Location:** ServerScriptService  
**Purpose:** Main server-side combat coordinator that initializes all combat systems and handles client requests for spells, targeting, and auto-attacks.

**Key Responsibilities:**
- Initializes SpellSystem, DebuffManager, TargetingModule, and other combat systems
- Handles FireEvent requests for spell casting, targeting, and auto-attack toggling
- Manages player combat data and target synchronization
- Coordinates between various combat systems for unified behavior
- Provides spell validation through SpellValidator integration
- Manages cooldown synchronization between client and server
- Handles player class changes and combat state updates
- Integrates with PetHandler for pet combat mechanics
- Processes damage events and coordinates with DamageManager
- Maintains player target state and auto-attack preferences

## Client-Side Visual & UI Systems

### ClientEffectManager.lua
**Location:** ReplicatedStorage/Modules  
**Purpose:** Comprehensive client-side manager for all visual effects including spells, debuffs, range indicators, notifications, and status displays.

**Key Responsibilities:**
- Manages visual effects for spells (ground effects, projectiles, impact effects)
- Handles debuff/buff visualization with icons, timers, and stack displays
- Provides range indicator system for spell targeting
- Manages status effect overlays (stun, slow, burn effects)
- Coordinates with DamageNumbers for damage display timing
- Handles shield effect visualization with health bar integration
- Manages zone status effects for area-of-effect spells
- Provides notification system for combat events
- Maintains status cache for consistent debuff display across UI elements
- Handles effect cleanup and memory management
- Supports ground targeting indicators and spell preview effects

**Visual Effect Types:**
- Debuff icons with stack counters and timers
- Spell casting indicators and range displays
- Status overlays (stun lockdown, movement effects)
- Shield health bars and destruction effects
- Ground AOE indicators and zone effects

### EnemyFramesClient.lua
**Location:** StarterPlayerScripts  
**Purpose:** Creates and manages WoW-style enemy frames showing health bars, cast bars, debuff icons, and shield indicators for visible enemies.

**Key Responsibilities:**
- Creates dynamic enemy frames for visible characters within configurable range
- Displays health bars with current/max health values and percentages
- Shows cast bars with spell names and casting progress
- Manages debuff icon displays with timers and stack counts
- Handles target highlighting and selection visual feedback
- Provides distance-based frame management for performance
- Updates frame positioning and sizing based on target importance
- Integrates with ClientEffectManager for status effect synchronization
- Supports shield health overlay for protected characters
- Handles frame cleanup when enemies move out of range or are destroyed

**Frame Features:**
- Health bars with color-coded health levels
- Cast bars with interrupt indicators
- Debuff/buff icon grids with tooltips
- Target selection highlighting
- Distance-based sizing and opacity

### DamageNumbers.lua
**Location:** StarterPlayerScripts  
**Purpose:** Displays floating damage numbers with styling based on damage type (DoT, critical, healing, auto-attack) with advanced animation and queueing systems.

**Key Responsibilities:**
- Creates floating damage numbers with type-specific styling and colors
- Manages damage number positioning to prevent overlap
- Provides specialized DoT tick visualization with symbols and reduced size
- Handles critical hit emphasis with size multipliers and special formatting
- Manages healing number display with distinct colors and styling
- Implements damage number queuing for rapid damage events
- Provides immunity and resistance visual feedback
- Handles shield damage absorption indicators
- Supports auto-attack damage distinction from spell damage
- Manages performance through active number limits and cleanup

**Visual Styling:**
- Color coding: Red (player damage), Orange (team damage), Gray (enemy damage), Green (healing)
- Special symbols for DoT types: 🔥 (burn), ☠ (poison), 🗡 (bleed), ❄ (frost)
- Size variations for damage types and critical hits
- Floating animations with fade effects

### SpellUIScript.lua
**Location:** Client UI Script  
**Purpose:** Manages the spell casting UI including spell buttons, cooldown indicators, ground targeting, and player input handling.

**Key Responsibilities:**
- Creates and manages spell buttons based on player class
- Handles spell button click events and keyboard shortcuts
- Manages cooldown overlays and visual indicators
- Implements ground targeting mode with position indicators and radius visualization
- Provides Global Cooldown (GCD) visual feedback across all spell buttons
- Handles friendly targeting for healing spells without disrupting combat targeting
- Manages auto-attack toggle state and visual feedback
- Integrates with CooldownManager for accurate cooldown tracking
- Provides spell validation feedback and error notifications
- Handles mouseover targeting for instant cast abilities

**UI Features:**
- Dynamic spell button generation based on class
- Cooldown sweep animations and timers
- Ground targeting crosshair and range indicators
- GCD overlay effects across all buttons
- Friendly targeting indicators for healing spells

### DebuffClient.lua
**Location:** StarterPlayerScripts  
**Purpose:** Client-side handler specifically for visualizing debuffs on characters and updating UI elements to show active debuffs with networking integration.

**Key Responsibilities:**
- Receives debuff updates from server through GameNetwork and legacy events
- Updates debuff visual cache for consistent display across UI systems
- Handles debuff application, removal, and cleansing notifications
- Provides debuff stacking visualization with proper stack count display
- Manages debuff timer updates and expiration handling
- Coordinates with ClientEffectManager for visual effect triggers
- Handles multi-debuff operations for cleansing abilities
- Provides debug logging for debuff synchronization issues
- Maintains compatibility with both modern and legacy networking systems

## Data Management & Persistence

### PlayerDataManager.lua
**Location:** ServerScriptService  
**Purpose:** Comprehensive player data management system handling persistent storage, character progression, currency, PvP statistics, and session management.

**Key Responsibilities:**
- Manages ProfileStore integration for persistent data storage with fallback systems
- Handles player data loading/saving with auto-save capabilities
- Manages character progression including levels, experience, and unlocks
- Provides currency management (coins, tokens, premium currency) with transaction logging
- Tracks PvP statistics including wins, losses, ratings, and seasonal data
- Handles cosmetic unlocks and inventory management
- Manages tutorial progress and completion tracking
- Provides location and activity tracking for player state
- Implements friend systems and social features
- Handles data validation and sanitization for security

**Data Categories:**
- Character progression (level, experience, stats)
- Currency and transactions
- PvP stats and rankings
- Cosmetics and unlocks
- Tutorial and achievement progress
- Social data (friends, blocks)

### DamageManager.lua
**Location:** ServerScriptService  
**Purpose:** Centralized damage calculation and application system with passive effect integration, team checking, and comprehensive event hooks.

**Key Responsibilities:**
- Calculates final damage values with resistance, immunity, and absorption checks
- Applies damage to characters with health validation and death handling
- Manages passive effect system for damage modification and triggers
- Provides team-based damage validation to prevent friendly fire
- Handles damage absorption through shields and temporary immunity
- Implements critical hit calculation and damage type processing
- Provides damage event hooks for passive abilities and reactive effects
- Manages damage logging and analytics for balancing
- Coordinates with DebuffManager for damage-over-time effects
- Handles healing application with overheal prevention

**Event Hooks:**
- Pre-damage calculation for damage modification
- Post-damage application for reactive abilities
- Damage resistance and immunity checks
- Shield absorption and destruction events

## Utility & Support Modules

### SpellValidationModule.lua
**Location:** ReplicatedStorage/Modules  
**Purpose:** Validates spell casting prerequisites including range checking, target validation, and provides visual feedback for failed casts on both client and server.

**Key Responsibilities:**
- Validates spell casting range between caster and target
- Checks target validity (alive, targetable, not immune)
- Provides line-of-sight validation for targeted spells
- Validates resource requirements (mana, energy, etc.)
- Checks team alignment for friendly/enemy targeting restrictions
- Provides visual feedback for validation failures
- Handles special targeting cases (self-cast, ground-targeted)
- Validates spell prerequisites and conditions
- Supports both client prediction and server authority validation

### CooldownManager.lua
**Location:** ReplicatedStorage/Modules  
**Purpose:** Comprehensive cooldown tracking system that manages spell cooldowns, Global Cooldown (GCD), and provides callback registration for cooldown events.

**Key Responsibilities:**
- Tracks individual spell cooldowns with precise timing
- Manages Global Cooldown system affecting all spells simultaneously
- Provides cooldown validation for casting attempts
- Handles server-client cooldown synchronization and correction
- Supports callback registration for cooldown start/end events
- Provides cooldown percentage calculation for UI animations
- Handles cooldown clearing and manual manipulation
- Integrates with spell UI for visual cooldown displays
- Supports different cooldown types (spell, GCD, item)

**Cooldown Features:**
- Precise timing with tick-based calculations
- Server synchronization for anti-cheat protection
- Visual callback system for UI integration
- GCD management affecting all abilities

---

## System Integration Overview

These systems work together to create a cohesive combat experience:

1. **Combat Flow:** SpellSystem → TargetingModule → SpellValidator → DamageManager → DebuffManager
2. **Visual Flow:** Server Effects → ClientEffectManager → EnemyFramesClient → DamageNumbers
3. **Network Flow:** GameNetwork → GameNetworkBridge → Legacy Events → Client Systems
4. **Data Flow:** PlayerDataManager → Character Stats → Combat Systems → Persistent Storage

Each system maintains modular design with clear interfaces, allowing for easy modification and extension while preserving system stability and performance.
