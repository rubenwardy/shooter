Minetest Mod - Shooter API [shooter]
====================================

Depends: default

Handles raycasting, blasting and audio-visual effects of dependent mods

Crafting
--------

### Gunpowder

1 x Coal Lump + 1 x Clay Lump = 5 x Gunpowder (shapeless)
```
	output = "shooter:gunpowder 5",
	type = "shapeless",
	recipe = {"default:coal_lump", "default:clay_lump"},
```
Configuration
-------------

Override the following default settings by adding them to your minetest.conf file

### Enable automatic weapons

Uses globalstep to detect left mouse button

`shooter_automatic_weapons = true`

### Enable admin super weapons

Allows admins (server priv) shoot all guns automatically

`shooter_admin_weapons = false`

### Enable node destruction with explosives

`shooter_enable_blasting = true`

### Enable Crafting

`shooter_enable_crafting = true`

### Enable hit particle effects

`shooter_enable_particle_fx = true`

### Enable protection

Requires a protection mod that utilizes `minetest.is_protected()`

`shooter_enable_protection = false`

### Particle texture

Particle texture used when a player or entity with the 'fleshy' armor group is hit

`shooter_explosion = "shooter_hit.png"`

### Allow node destruction

`shooter_allow_nodes = true`

### Allow entities

Defaults to `true` in singleplayer mode

`shooter_allow_entities = false`

### Allow players

Defaults to `false` in singleplayer mode

`shooter_allow_players = true`

### Round update time

Maximum round 'step' processing interval, will inversely effect the long-range velocity of the virtual projectiles. This should always be greater than the dedicated server step time

`shooter_rounds_update_time = 0.4`

### Camera Height

Player eye offset used for rayasting and particle effects

`shooter_camera_height = 1.5`

API Documentation
-----------------

### Global tables

* `shooter.registered_weapons`: Registered weapons by itemstring
* `shooter.config`: Present configuration
* `shooter.default_particles`: Default hit particle definition
```Lua
{
	amount = 15,
	time = 0.3,
	minpos = {x=-0.1, y=-0.1, z=-0.1},
	maxpos = {x=0.1, y=0.1, z=0.1},
	minvel = {x=-1, y=1, z=-1},
	maxvel = {x=1, y=2, z=1},
	minacc = {x=-2, y=-2, z=-2},
	maxacc = {x=2, y=-2, z=2},
	minexptime = 0.1,
	maxexptime = 0.75,
	minsize = 1,
	maxsize = 2,
	collisiondetection = false,
	texture = "shooter_hit.png",
}
```
### Methods

* `shooter.register_weapon(name, definition)`: Register a shooting weapon. -- See "Weapon Definition"
* `shooter.get_configuration(config)`: Loads matching config settings into a table ref `config`
* `shooter.spawn_particles(pos, particles)`: Adds particles at the specified position
	* `particles` is an optional table of overrides for `shooter.default_particles`
* `shooter.play_node_sound(node, pos)`: Plays the registered 'dug' sound for the node at `pos`
* `shooter.punch_node(pos, spec)`: Punches the node at `pos` with the `spec` group capabilities
* `shooter.is_valid_object(object)`: Returns `true` if the object can be damaged
* `shooter.fire_weapon(player, itemstack, spec)`: Adds a 'round' with `spec` to the processing que
* `shooter.blast(pos, radius, fleshy, distance, user)`: Create explosion at `pos`
	* `radius`: Blast radius in nodes
	* `fleshy`: Damage to inflict on fleshy objects: `(fleshy * 0.5 ^ distance) * 2`
	* `distance`: Area of effect for objects
	* `user`: A player reference, used for protection and object damage
* `shooter.get_shooting(name)`: Returns `true` if player `name` is holding the left mouse button or `nil`
	* Requires `shooter_automatic_weapons` to be set `true`
* `shooter.set_shooting(name, is_shooting)`: Sets the left mouse button status of player `name`

Weapon Definition
-----------------

Used by `shooter.register_weapon`

```Lua
{
	description = "My Awesome Gun",
	inventory_image = "my_awesome_gun.png",
	reload_item = "itemstring",
		-- Reload Item, "shooter:ammo" is used if omitted
	unloaded_item = {
		-- Optional. Item to be registered as unloaded weapon item
		name = "itemstring",
		description = "My Awesome Gun (Unloaded)",
		inventory_image = "my_awesome_gun_unloaded.png",
	},
	on_hit = function(pointed_thing, spec, dir)
		-- May be used for arbitary shot effects like knock-back, etc.
	end,
	spec = {
		-- Weapon specifications
		rounds = 100,
			-- Number of rounds, refilled by the defined reload item
		range = 200,
			-- Range (in nodes) of each shot
		tool_caps = {
			-- Tool capabilities, used for object/player damage
			full_punch_interval = 1.0,
			damage_groups = {fleshy=3},
		},
		groups = {
			-- Damage groups, used to effect nodes as a normal tool item would
			snappy=3,
			crumbly=3,
			choppy=3,
			fleshy=2,
			oddly_breakable_by_hand=2
		},
		sounds = {
			-- Sound files (defaults)
			shot = "guns_rifle",
			reload = "shooter_reload",
			fail_shot = "shooter_click",
		},
		bullet_texture = "shooter_bullet.png",
			-- Particle texture file name for the projectile
		particles = {},
			-- Optional. Table of overrides for `shooter.default_particles`
	},
}
```