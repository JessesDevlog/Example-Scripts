####### THIS SCRIPT IS CURRENTLY BEING DEVELOPED AND REVISED FOR SHYBORG GAMES' UPCOMING TITLE "NO SURVIVORS"
####### IT HANDLES ALL MOVEMENT BEHAVIORS FOR OUR NPCS; RANDOM/SPECIFIC PATHING, ALIVE/UNDEAD STATE, DOCILE/HOSTILE STATE, PLAYER TARGET INPUT, ETC...

extends Node # Extends the Node class

var NPC # Reference to the NPC instance

var last_next_path_position: Vector3 = Vector3.ZERO # Stores last calculated next path position
var cached_direction: Vector3 = Vector3.ZERO # Stores last calculated movement direction

func handle_actions(delta): # Handles NPC actions based on state
	npc_position = NPC.global_transform.origin # Get NPC position
	get_distance_to_player() # Calculate distance to player
	if distance_travelled == 5.0 and NPC.dead == false and currentlyCheckingDistanceTravelled == true: # Check if NPC has travelled a certain distance
		choose_new_docile_target() # Pick a new target if needed
		currentlyCheckingDistanceTravelled = false # Reset distance check flag
	match NPC.curState: # Switch based on NPC state
		NPC.ALIVE_DOCILE: # If NPC is alive and docile
			act_alive_docile(delta) # Run docile behavior
			NPC.safeVelocityFloat = .25 # Set safe velocity float
		NPC.ALIVE_ALARMED: # If NPC is alarmed
			act_alive_alarmed(delta) # Run alarmed behavior
			NPC.safeVelocityFloat = .99 # Set safe velocity float
		NPC.ALIVE_HOSTILE: # If NPC is hostile
			act_alive_hostile(delta) # Run hostile behavior
		NPC.DEAD: # If NPC is dead
			act_dead() # Run dead behavior
		NPC.UNDEAD_DOCILE: # If NPC is undead and docile
			act_undead_docile(delta) # Run undead docile behavior
		NPC.UNDEAD_HOSTILE: # If NPC is undead and hostile
			act_undead_hostile(delta) # Run undead hostile behavior
		NPC.UNDEAD_TARGET: # If NPC is undead and has a target
			act_undead_target(delta) # Run undead target behavior
		NPC.ALIVE_FEAR_STUNNED: # If NPC is fear stunned
			act_alive_fear_stunned(delta) # Run fear stunned behavior
		NPC.ALIVE_SENTRY_DOCILE: # If NPC is sentry and docile
			act_alive_sentry_docile(delta) # Run sentry docile behavior
		NPC.ALIVE_SENTRY_HOSTILE: # If NPC is sentry and hostile
			act_alive_sentry_hostile(delta) # Run sentry hostile behavior

func set_NPC(NPC_ref): # Sets the NPC reference and configures timers
	NPC = NPC_ref # Assign NPC reference
	NPC.nav_3d.avoidance_enabled = true # Enable navigation avoidance
	NPC.add_child(waitingASecondTimer) # Add waiting timer to NPC
	var threat_timer = Timer.new() # Create threat timer
	threat_timer.wait_time = 0.5 # Set timer interval
	threat_timer.one_shot = false # Set timer to repeat
	threat_timer.autostart = true # Start timer automatically
	NPC.add_child(threat_timer) # Add threat timer to NPC
	threat_timer.connect("timeout", _on_threat_timer_timeout) # Connect timer signal
	if waitingASecondTimer.is_connected("timeout", setWaitingASecondToFalse): # Check if timer is connected
		return # Exit if already connected
	elif !waitingASecondTimer.is_connected("timeout", setWaitingASecondToFalse): # If not connected
		waitingASecondTimer.connect("timeout", setWaitingASecondToFalse) # Connect timer signal

func populate_NPCPaths(): # Populates NPC path dictionary
	for path3D_node in NPC.npc_paths_container.get_children(): # Iterate path nodes
		var pathPoints = path3D_node.curve.get_baked_points() # Get path points
		NPC.NPCPathsDict[path3D_node.name] = { # Store path info in dictionary
			"name": path3D_node.name, # Path name
			"points": pathPoints # Path points
			}
	pass # End of function

func find_path(): # Finds a new path for NPC
	return # Early exit (function not implemented)

	@warning_ignore("unreachable_code") # Ignore unreachable code warning
	var closest_point = null # Closest point variable
	var min_distance = INF # Minimum distance variable
	for path in NPC.NPCPathsDict: # Iterate paths
		if NPC.lastPath != null: # Skip last path
			if path == NPC.lastPath["name"]:
				continue
		if NPC.secondToLastPath != null: # Skip second to last path
			if path == NPC.secondToLastPath["name"]:
				continue
		for i in range(len(NPC.NPCPathsDict[path]["points"])): # Iterate points
			var point = NPC.NPCPathsDict[path]["points"][i] # Get point
			var distance = npc_position.distance_to(point) # Calculate distance
			if distance < min_distance: # If closer than previous
				min_distance = distance # Update min distance
				closest_point = point # Update closest point
				NPC.currentPointIndex = i # Set current point index
				NPC.currentPath = NPC.NPCPathsDict[path] # Set current path
	NPC.pathing_timer.wait_time = randi_range(60, 70) # Set pathing timer
	NPC.pathing_timer.start() # Start pathing timer
	if NPC.avoidedOtherNPC == false: # If not avoided other NPC
		if randf() > .5: # Randomly choose direction
			NPC.travelPathBackwards = true
		else:
			NPC.travelPathBackwards = false
	else:
		NPC.avoidedOtherNPC = false # Reset avoidance flag
	if NPC.printPathDebugging == true: # Print debug info
		print("decided path is :", NPC.currentPath["name"])
	return closest_point # Return closest point

func _on_pathing_timer_timeout(): # Called when pathing timer times out
	if NPC.dead == true: # If NPC is dead
		return # Exit
	if NPC.printPathDebugging == true and NPC.currentPath['name'] != null: # Print debug info
		print("current path: ", NPC.currentPath["name"])
		print("finding path")
		pass
	NPC.secondToLastPath = NPC.lastPath # Update path history
	NPC.lastPath = NPC.currentPath
	NPC.currentPath = null # Reset current path
	pass

func _on_waiting_timer_timeout(): # Called when waiting timer times out
	if NPC.waiting == true: # If NPC is waiting
		choose_new_docile_target() # Pick new target
		NPC.waiting = !NPC.waiting # Toggle waiting flag
		NPC.waiting_timer.wait_time = randi_range(3, 5) # Set timer interval
	pass

func mill_around(delta): # NPC milling around behavior
	if NPC.nav_3d.is_navigation_finished() == true: # If navigation finished
		choose_new_docile_target() # Pick new target
		return # Exit
	move_towards_target_location(delta) # Move towards target location
	if NPC.direction.length() > 0: # If direction is valid
		rotate_towards_direction(delta) # Rotate NPC

func reset_path_variables(): # Resets path variables
	NPC.currentPath = null # Reset current path
	NPC.lastPath = null # Reset last path
	NPC.secondToLastPath = null # Reset second to last path
	NPC.travelPathBackwards = !NPC.travelPathBackwards # Toggle path direction

func move_to_point(): # Moves NPC to a point along path
	if NPC.npc_avoidance_raycast.is_colliding() == true: # If raycast collides
		if NPC.npc_avoidance_raycast.get_collider() != null and NPC.npc_avoidance_raycast.get_collider().is_in_group("NPCs") and not NPC.avoidingOtherNPC: # If collider is NPC
			avoid_other_NPC() # Avoid other NPC
			return # Exit
	if NPC.avoidingOtherNPC == false: # If not avoiding
		if NPC.currentPath == null: # If no current path
			var closest_point = find_path() # Find new path
			if closest_point == null: # If no path found
				return # Exit
			NPC.nav_3d.target_position = closest_point # Set navigation target
			if NPC.travelPathBackwards: # If traveling backwards
				if NPC.currentPointIndex - 30 > 0:
					NPC.currentPointIndex -= 30
				else:
					NPC.currentPointIndex = len(NPC.currentPath["points"]) - 15
				NPC.nav_3d.target_position = NPC.currentPath["points"][NPC.currentPointIndex]
			else:
				if NPC.currentPointIndex + 30 < len(NPC.currentPath["points"]) - 1:
					NPC.currentPointIndex += 30
				else:
					NPC.currentPointIndex = 15
				NPC.nav_3d.target_position = NPC.currentPath["points"][NPC.currentPointIndex]
		else: # If path exists
			var distanceToTargetPoint = npc_position.distance_to(NPC.nav_3d.target_position) # Calculate distance to target
			if distanceToTargetPoint <= 1.0: # If close to target
				if NPC.travelPathBackwards: # If traveling backwards
					if NPC.currentPointIndex > 0:
						NPC.currentPointIndex -= 1
					else:
						NPC.currentPointIndex = len(NPC.currentPath["points"]) - 1
					NPC.nav_3d.target_position = NPC.currentPath["points"][NPC.currentPointIndex]
				else:
					if NPC.currentPointIndex < len(NPC.currentPath["points"]) - 1:
						NPC.currentPointIndex += 1
					else:
						NPC.currentPointIndex = 0
					NPC.nav_3d.target_position = NPC.currentPath["points"][NPC.currentPointIndex]
		var next_path_position = NPC.nav_3d.get_next_path_position() # Get next path position
		if next_path_position != last_next_path_position: # If position changed
			var raw_direction = next_path_position - npc_position # Calculate direction
			cached_direction = raw_direction.normalized() if raw_direction.length() > 0.01 else Vector3.ZERO # Normalize direction
			last_next_path_position = next_path_position # Update last position
		NPC.direction = cached_direction # Set NPC direction
		if NPC.printPathDebugging == true: # Print debug info
			print("Current point index: ", NPC.currentPointIndex)
			print(len(NPC.currentPath["points"]))
		pass

func avoid_other_NPC(): # Handles NPC avoidance behavior
	NPC.nav_3d.debug_enabled = false # Disable navigation debug
	NPC.nav_3d.debug_use_custom = true # Use custom debug
	NPC.nav_3d.debug_path_custom_color = Color.HOT_PINK # Set debug color

	NPC.avoidingOtherNPC = true # Set avoiding flag
	var current_direction = NPC.direction # Get current direction
	var current_angle = atan2(current_direction.z, current_direction.x) # Calculate angle
	var new_angle = current_angle + deg_to_rad(15) # Offset angle
	var new_direction = Vector3(cos(new_angle), current_direction.y, sin(new_angle)) # Calculate new direction

	var forward_distance = 2 # Set forward distance
	var _new_target_position = npc_position + new_direction * forward_distance # Calculate new target position
	NPC.nav_3d.target_position = _new_target_position # Set navigation target

	var normalized_new_direction = new_direction.normalized() if new_direction.length() > 0.01 else Vector3.ZERO # Normalize direction
	NPC.direction = normalized_new_direction # Set NPC direction

	var timer = Timer.new() # Create timer
	timer.wait_time = 1.2 # Set timer interval
	timer.one_shot = true # Set timer to one shot
	NPC.add_child(timer) # Add timer to NPC
	timer.start() # Start timer
	await timer.timeout # Wait for timer
	timer.queue_free() # Free timer
	NPC.avoidingOtherNPC = false # Reset avoiding flag
	NPC.avoidedOtherNPC = true # Set avoided flag

func alignCollisionShapes(): # Aligns collision shapes with NPC hips
	NPC.get_node("CollisionShape3D").global_position.x = NPC.hips.global_position.x # Align x position
	NPC.get_node("CollisionShape3D").global_position.z = NPC.hips.global_position.z # Align z position
	NPC.get_node("Hitbox/CollisionShape3D").global_position.x = NPC.hips.global_position.x # Align hitbox x
	NPC.get_node("Hitbox/CollisionShape3D").global_position.z = NPC.hips.global_position.z # Align hitbox z
	pass

func die(): # Handles NPC death
	set_nav3d_target_to_self() # Set navigation target to self
	if NPC.healthBar: NPC.healthBar.visible = false # Hide health bar
	NPC.canMove = false # Disable movement
	NPC.punch_blend_val = 0.0 # Reset punch blend
	NPC.set_collision_layer_value(4, false) # Disable collision layer
	NPC.set_collision_mask_value(4, false) # Disable collision mask
	NPC.set_collision_mask_value(2, false) # Disable collision mask
	NPC.curAnim = NPC.DYING # Set animation to dying
	NPC.MOVEMENT_SPEED = 0.0 # Set movement speed to zero
	if NPC.printDamageDebugging == true: # Print debug info
		print("Dead")
	if NPC.healthBar != null: # If health bar exists
		NPC.NPCSTATUS.makeHealthBarFollowHead() # Update health bar position

func stumble(): # Handles NPC stumble animation
	if NPC.hitStumble == true: # If hit stumble is active
		NPC.animation_tree.set("parameters/HitStumbleTimeSeek/seek_request", -1.0) # Seek hit stumble animation
		NPC.currentAction = "HitStumble" # Set current action
		NPC.curAnim = NPC.HITSTUMBLE # Set animation
		if NPC.stumbling == false: # If not already stumbling
			NPC.stumbling = true # Set stumbling flag

func get_direction_towards_player(delta): # Calculates direction towards player
	var player_position = NPC.player.global_transform.origin # Get player position
	if NPC.nav_3d.target_position.distance_squared_to(player_position) > 1.0: # If far from player
		var current_fps = Engine.get_frames_per_second() # Get current FPS
		if current_fps < 30 and randf_range(0.00, 1.00) > .50: # FPS check for performance
			pass
		NPC.nav_3d.target_position = player_position # Set navigation target
		var next_path_position = NPC.nav_3d.get_next_path_position() # Get next path position
		if next_path_position != last_next_path_position: # If position changed
			var raw_direction = next_path_position - NPC.global_transform.origin # Calculate direction
			cached_direction = raw_direction.normalized() if raw_direction.length() > 0.01 else Vector3.ZERO # Normalize direction
			last_next_path_position = next_path_position # Update last position
		var direction = cached_direction # Get cached direction

		if direction.length() > 0: # If direction is valid
			NPC.direction = direction # Set NPC direction
			rotate_towards_direction(delta) # Rotate NPC
			NPC.curAnim = NPC.WALK # Set animation
			NPC.velocity = direction * NPC.MOVEMENT_SPEED # Set velocity
	NPC.move_and_slide() # Move NPC

	if NPC.global_transform.origin.distance_to(NPC.nav_3d.target_position) < 1.5 and print_pathfinding_debug: # If close to target
		print("NPC reached destination:", NPC.nav_3d.target_position) # Print debug info

func move_towards_player(delta): # Moves NPC towards player
	if distance_to_player < NPC.zombieGroupIdleRadius: # If within idle radius
		NPC.waiting = true # Set waiting flag
		NPC.change_to_state = true # Request state change
		return # Exit
	elif distance_to_player > NPC.zombieGroupIdleRadius: # If outside idle radius
		NPC.curAnim = NPC.WALK # Set animation
		get_direction_towards_player(delta) # Get direction
		rotate_towards_direction(delta) # Rotate NPC

var position_last_calculated # Stores last calculated position
func get_direction_away_from_threat(): # Calculates direction away from threat
	if NPC.currentThreat == null: # If no threat
		return # Exit
	if NPC.runningFromThreat == false: # If not running
		NPC.threatPositionWhenAlarmed = NPC.currentThreat.global_transform.origin # Store threat position
		NPC.runningFromThreat = true # Set running flag

	var direction_away = npc_position - NPC.currentThreat.global_transform.origin # Calculate direction away
	var normalized_away = direction_away.normalized() if direction_away.length() > 0.01 else Vector3.ZERO # Normalize direction

	if position_last_calculated == null: # If position not set
		position_last_calculated = npc_position # Set position

	if getting_direction_away_from_threat: # If should get direction
		var target_pos = npc_position + normalized_away * NPC.zombieGroupIdleRadius # Calculate target position
		NPC.targetAwayFromThreat = target_pos # Set target away from threat
		NPC.nav_3d.target_position = target_pos # Set navigation target

		var raw_direction = target_pos - npc_position # Calculate direction
		NPC.direction = raw_direction.normalized() if raw_direction.length() > 0.01 else Vector3.ZERO # Normalize direction

		if NPC.npc_avoidance_raycast.get_collider() != null and NPC.npc_avoidance_raycast.get_collider().is_in_group("NPCs") and not NPC.avoidingOtherNPC: # If colliding with NPC
			avoid_other_NPC() # Avoid other NPC
		getting_direction_away_from_threat = false # Reset flag

func move_away_from_threat(delta): # Moves NPC away from threat
	NPC.curAnim = NPC.RUN # Set animation
	if NPC.stumbling != true: # If not stumbling
		NPC.MOVEMENT_SPEED = NPC.running_speed # Set running speed
		NPC.zombie_walk_speed_scale = (NPC.MOVEMENT_SPEED / 2.3) * 1.6 # Set walk speed scale
		get_direction_away_from_threat() # Get direction away
	if NPC.slowed == true: # If slowed
		NPC.MOVEMENT_SPEED = NPC.walking_speed * .5 # Set slowed speed
		NPC.zombie_walk_speed_scale = (NPC.MOVEMENT_SPEED / 2.3) * 1.6 # Set walk speed scale
	if NPC.direction.length() > 0: # If direction is valid
		rotate_towards_direction(delta) # Rotate NPC

func get_direction_towards_target(): # Calculates direction towards current target
	var target_position = NPC.currentTarget.global_transform.origin # Get target position
	NPC.nav_3d.target_position = target_position # Set navigation target
	var next_path_position = NPC.nav_3d.get_next_path_position() # Get next path position
	if next_path_position != last_next_path_position: # If position changed
		var raw_direction = next_path_position - npc_position # Calculate direction
		cached_direction = raw_direction.normalized() if raw_direction.length() > 0.01 else Vector3.ZERO # Normalize direction
		last_next_path_position = next_path_position # Update last position
	NPC.direction = cached_direction # Set NPC direction

func move_towards_target(delta): # Moves NPC towards current target
	if NPC.currentTarget == null: # If no target
		return # Exit
	if NPC.currentTarget is Vector3: # If target is a position
		distance_to_target = npc_position.distance_to(NPC.currentTarget) # Calculate distance
	else:
		distance_to_target = npc_position.distance_to(NPC.currentTarget.global_transform.origin) # Calculate distance
	if NPC.currentTarget is Vector3: # If target is a position
		return # Exit
	if NPC.currentTarget.dead == true: # If target is dead
		NPC.hostile = false # Set hostile flag
		NPC.change_to_state = true # Request state change
		if NPC.hasPlayerTarget == true: # If has player target
			NPC.currentTarget = NPC.playerTarget # Set player target
			NPC.curState = NPC.UNDEAD_TARGET # Set state
			NPC.curAnim = NPC.WALK # Set animation
			return # Exit
	if distance_to_player > NPC.zombieMaxDistanceFromPlayer and NPC.aiming == false: # If too far from player
		NPC.nav_3d.debug_path_custom_color = Color.DEEP_PINK # Set debug color
		NPC.hostile = false # Set hostile flag
		NPC.change_to_state = true # Request state change
		if NPC.hasPlayerTarget == true: # If has player target
			NPC.currentTarget = NPC.playerTarget # Set player target
			NPC.curState = NPC.UNDEAD_TARGET # Set state
			NPC.curAnim = NPC.WALK # Set animation
			return # Exit
		else:
			NPC.currentTarget = null # Clear target
			NPC.curAnim = NPC.WALK # Set animation
	if NPC.attacking == false and distance_to_target > 15: # If not attacking and far
		NPC.curAnim = NPC.WALK # Set animation
		NPC.nav_3d.debug_path_custom_color = Color.FIREBRICK # Set debug color
	elif distance_to_target <= 14 and NPC.attacking == false: # If not attacking and close
		NPC.curAnim = NPC.ZOMBIERUN # Set animation
		NPC.nav_3d.debug_path_custom_color = Color.REBECCA_PURPLE # Set debug color
		NPC.MOVEMENT_SPEED = NPC.zombie_running_speed # Set running speed
		NPC.zombie_walk_speed_scale = (NPC.MOVEMENT_SPEED / 2.3) * 1.6 # Set walk speed scale
	if NPC.currentTarget == null: # If no target
		NPC.hostile = false # Set hostile flag
		NPC.change_to_state = true # Request state change
		if NPC.hasPlayerTarget == true: # If has player target
			NPC.currentTarget = NPC.playerTarget # Set player target
			NPC.curState = NPC.UNDEAD_TARGET # Set state
			NPC.curAnim = NPC.WALK # Set animation
			return # Exit
	if NPC.currentTarget == null: # If no target
		return # Exit
	if distance_to_target < NPC.zombieChaseDistance and distance_to_target < 15: # If close to target
		if NPC.attacking == false: NPC.curAnim = NPC.ZOMBIERUN # Set animation
		get_direction_towards_target() # Get direction
		rotate_towards_direction(delta) # Rotate NPC
		NPC.MOVEMENT_SPEED = NPC.zombie_running_speed # Set running speed
		NPC.zombie_walk_speed_scale = (NPC.MOVEMENT_SPEED / 2.3) * 1.6 # Set walk speed scale
		return # Exit
	if distance_to_target < NPC.zombieChaseDistance and NPC.aiming == false: # If close and not aiming
		get_direction_towards_target() # Get direction
		rotate_towards_direction(delta) # Rotate NPC
		NPC.MOVEMENT_SPEED = NPC.zombie_walking_speed # Set walking speed
		NPC.zombie_walk_speed_scale = (NPC.MOVEMENT_SPEED / 2.3) * 1.6 # Set walk speed scale
		return # Exit

func move_towards_player_target(delta): # Moves NPC towards player target
	if NPC.currentTarget == null: # If no target
		NPC.hasPlayerTarget = false # Reset player target flag
		NPC.change_to_state = true # Request state change
	if distance_to_player_target > 5: # If far from player target
		NPC.curAnim = NPC.WALK # Set animation
		get_direction_towards_player_target() # Get direction
		rotate_towards_direction(delta) # Rotate NPC
		NPC.MOVEMENT_SPEED = NPC.zombie_walking_speed # Set walking speed
		NPC.zombie_walk_speed_scale = (NPC.MOVEMENT_SPEED / 2.3) * 1.6 # Set walk speed scale
		return # Exit
	elif distance_to_player_target <= 5: # If close to player target
		NPC.hostile = false # Set hostile flag
		NPC.change_to_state = true # Request state change
		NPC.currentTarget = null # Clear target
		NPC.hasPlayerTarget = false # Reset player target flag
		NPC.playerTarget = null # Clear player target
		NPC.curState = NPC.UNDEAD_DOCILE # Set state

func get_direction_towards_player_target(): # Calculates direction towards player target
	var target_position = NPC.currentTarget # Get target position
	NPC.nav_3d.target_position = target_position # Set navigation target
	var next_path_position = NPC.nav_3d.get_next_path_position() # Get next path position
	if next_path_position != last_next_path_position: # If position changed
		var raw_direction = next_path_position - npc_position # Calculate direction
		cached_direction = raw_direction.normalized() if raw_direction.length() > 0.01 else Vector3.ZERO # Normalize direction
		last_next_path_position = next_path_position # Update last position
	NPC.direction = cached_direction # Set NPC direction

func rotate_towards_direction(delta): # Rotates NPC towards direction
	if NPC.dead: # If NPC is dead
		if not NPC.hasTurnedUndead: # If not turned undead
			return # Exit
		else:
			if NPC.direction.length() > 0.01: # If direction is valid
				NPC.visuals.rotation.y = lerp_angle(NPC.visuals.rotation.y, atan2(NPC.direction.x, NPC.direction.z), delta * NPC.angular_acceleration) # Rotate visuals
			return # Exit
	if NPC.direction.length() > 0.01: # If direction is valid
		NPC.visuals.rotation.y = lerp_angle(NPC.visuals.rotation.y, atan2(NPC.direction.x, NPC.direction.z), delta * NPC.angular_acceleration) # Rotate visuals

func wait(): # Sets NPC to idle state
	NPC.curAnim = NPC.IDLE # Set animation
	NPC.MOVEMENT_SPEED = 0.0 # Set movement speed to zero

func toggleNPCMovement(): # Toggles NPC movement
	if NPC.canMove == true: # If can move
		NPC.MOVEMENT_SPEED = 0.0 # Set movement speed to zero
		NPC.velocity = Vector3(0, 0, 0) # Reset velocity
		NPC.canMove = false # Disable movement
		NPC.curAnim = NPC.IDLE # Set animation
	else:
		NPC.canMove = true # Enable movement
		NPC.curAnim = NPC.WALK # Set animation

func _on_detection_area_zombie_entered(body): # Called when zombie enters detection area
	if body != NPC.player and NPC.undead == true and NPC.hasTurnedUndead == true: # If valid zombie
		if body.dead == false: # If zombie is alive
			if NPC.currentTarget and distance_to_target != null: # If current target exists
				if distance_to_target > npc_position.distance_to(body.global_transform.origin): # If closer than current target
					NPC.currentTarget = body # Set new target
			else:
				NPC.currentTarget = body # Set new target
			NPC.hostile = true # Set hostile flag
			NPC.change_to_state = true # Request state change

func _on_threat_detection_area_entered(body): # Called when threat enters detection area
	if body == NPC.player or body.hasTurnedUndead == true: # If threat is player or undead
		if NPC.currentThreat != NPC.player: # If not already current threat
			NPC.currentThreat = body # Set current threat
			NPC.alarmed = true # Set alarmed flag
			NPC.change_to_state = true # Request state change

func choose_attack(): # Handles attack logic
	if NPC.hostile == false: # If not hostile
		return # Exit
	if NPC.currentTarget == null: # If no target
		NPC.hostile = false # Set hostile flag
		NPC.change_to_state = true # Request state change
		return # Exit
	if NPC.currentTarget is Vector3: # If target is position
		if npc_position.distance_to(NPC.currentTarget) < NPC.zombiePunchDistance: # If close enough to punch
			if NPC.attacking == false: # If not already attacking
				NPC.NPCVOCALSFX.manage_sounds(NPC.NPCVOCALSFX.random_SFX_from_list(NPC.NPCVOCALSFX.attacking_sounds)) # Play attack sound
				NPC.animation_tree.set("parameters/PunchTimeSeek/seek_request", -1.0) # Seek punch animation
				NPC.currentAttack = "Punch" # Set current attack
				NPC.attacking = true # Set attacking flag
				NPC.previousAnim = NPC.curAnim # Store previous animation
				NPC.curAnim = NPC.WALKPUNCH # Set animation
	else:
		if npc_position.distance_to(NPC.currentTarget.global_transform.origin) < NPC.zombiePunchDistance: # If close enough to punch
			if NPC.attacking == false: # If not already attacking
				NPC.NPCVOCALSFX.manage_sounds(NPC.NPCVOCALSFX.random_SFX_from_list(NPC.NPCVOCALSFX.attacking_sounds)) # Play attack sound
				NPC.animation_tree.set("parameters/PunchTimeSeek/seek_request", -1.0) # Seek punch animation
				NPC.currentAttack = "Punch" # Set current attack
				NPC.attacking = true # Set attacking flag
				NPC.previousAnim = NPC.curAnim # Store previous animation
				NPC.curAnim = NPC.WALKPUNCH # Set animation

var npc_position # Stores NPC position
@export var overlapping_areas = [] # Stores overlapping areas
var distance_to_player # Stores distance to player
func act_alive_docile(delta): # Handles alive docile behavior
	NPC.nav_3d.debug_enabled = false # Disable navigation debug
	NPC.nav_3d.debug_use_custom = true # Use custom debug
	NPC.nav_3d.debug_path_custom_color = Color.GREEN # Set debug color
	if NPC.zombieBeingTargettedForTesting == true: # If being targeted for testing
		choose_new_docile_target() # Pick new target
	if NPC.nav_3d.is_navigation_finished(): # If navigation finished
		NPC.MOVEMENT_SPEED = 0.0 # Set movement speed to zero
		NPC.curAnim = NPC.IDLE # Set animation
		choose_new_docile_target() # Pick new target
	if NPC.NPCVOCALSFX.idle_sound_is_playing == false: # If idle sound not playing
		NPC.NPCVOCALSFX.play_idle_sound() # Play idle sound
	if NPC.waiting == true: # If waiting
		NPC.MOVEMENT_SPEED = 0.0 # Set movement speed to zero
		NPC.curAnim = NPC.IDLE # Set animation
		choose_new_docile_target() # Pick new target
		return # Exit
	NPC.millingAround = true # Set milling around flag
	if NPC.currentThreat != null: # If there is a threat
		if distance_to_current_threat < NPC.zombieGroupIdleRadius: # If threat is close
			NPC.alarmed = true # Set alarmed flag
			NPC.change_to_state = true # Request state change
			reset_path_variables() # Reset path variables
			return # Exit
	if NPC.millingAround == true and NPC.moveTowardsPlayer == false: # If milling around
		if NPC.hitStumble == true: # If hit stumble
			stumble() # Run stumble behavior
		else:
			if move_towards_target_location_if_this_var_is_modulo10 % 10: # Modulo check for movement
				move_towards_target_location(delta) # Move towards target location
				move_towards_target_location_if_this_var_is_modulo10 += 1 # Increment counter
			else: move_towards_target_location_if_this_var_is_modulo10 += 1 # Increment counter
	elif NPC.moveTowardsPlayer == true: # If moving towards player
		if distance_to_player != null: # If distance is valid
			NPC.NPCMOVEMENT.move_towards_player(delta) # Move towards player
		if NPC.NPCWEAPON.equipped_weapon == null: # If no weapon equipped
			NPC.moveTowardsPlayer = false # Reset flag
	else:
		NPC.MOVEMENT_SPEED = 0.0 # Set movement speed to zero
		NPC.curAnim = NPC.IDLE # Set animation
	if NPC.NPCWEAPON.equipped_weapon != null: # If weapon equipped
		NPC.NPCWEAPON.detect_target_at_a_distance(delta) # Detect target at distance

func act_alive_alarmed(delta): # Handles alive alarmed behavior
	if NPC.NPCVOCALSFX.alive_alarmed_sounds == null:
		NPC.disableNPC()
		return
	get_distance_to_current_threat() # Calculate distance to threat
	NPC.nav_3d.debug_enabled = false # Disable navigation debug
	NPC.nav_3d.debug_use_custom = true # Use custom debug
	NPC.nav_3d.debug_path_custom_color = Color.YELLOW # Set debug color
	if NPC.NPCVOCALSFX.alarmed_sound_is_playing == false: # If alarmed sound not playing
		NPC.NPCVOCALSFX.play_alarmed_sound(NPC.NPCVOCALSFX.random_SFX_from_list(NPC.NPCVOCALSFX.alive_alarmed_sounds)) # Play alarmed sound
	NPC.NPCWEAPON.trying_to_get_a_view_of_player = false # Reset weapon view flag
	if NPC.NPCWEAPON.equipped_weapon != null: # If weapon equipped
		NPC.NPCWEAPON.reset_player_raycast_visual() # Reset weapon raycast
	if print_pathfinding_debug: # Print debug info
		print("CURRENT THREAT:", NPC.currentThreat)
	if distance_to_current_threat < NPC.runFromZombieOuterRadius: # If threat is close
		if NPC.hitStumble == true: # If hit stumble
			stumble() # Run stumble behavior
		else:
			NPC.NPCMOVEMENT.move_away_from_threat(delta) # Move away from threat
	else:
		NPC.alarmed = false # Reset alarmed flag
		NPC.change_to_state = true # Request state change
		NPC.runningFromThreat = false # Reset running flag
		NPC.threatPositionWhenAlarmed = null # Clear threat position
		NPC.currentThreat = null # Clear current threat
	if NPC.check_distance_moved_timer.is_stopped(): # If timer stopped
		NPC.check_distance_moved_timer.start() # Start timer
		checking_distance_of_movement = true # Set checking flag

func act_alive_hostile(delta): # Handles alive hostile behavior
	if NPC.reloading == true: # If reloading
		NPC.MOVEMENT_SPEED = 0 # Set movement speed to zero
		return # Exit
	if NPC.NPCWEAPON.equipped_weapon != null: # If weapon equipped
		if NPC.NPCWEAPON.trying_to_get_a_view_of_player and NPC.NPCWEAPON.distance_to_player <= NPC.NPCWEAPON.equipped_weapon["range"] and !NPC.canMove: # If trying to get view and in range
			toggleNPCMovement() # Toggle movement
	if NPC.currentTarget == NPC.player and NPC.NPCWEAPON.equipped_weapon != null: # If target is player and weapon equipped
		NPC.NPCWEAPON.aim_weapon_at_player(delta) # Aim weapon at player
	if NPC.NPCWEAPON.trying_to_get_a_view_of_player and NPC.reloading == false: # If trying to get view and not reloading
		get_direction_towards_player(delta) # Get direction towards player
		rotate_towards_direction(delta) # Rotate NPC
		NPC.moveTowardsPlayer = true # Set move towards player flag
		NPC.MOVEMENT_SPEED = 2.3 # Set movement speed
		if NPC.slowed == true: # If slowed
			NPC.MOVEMENT_SPEED = NPC.walking_speed * .5 # Set slowed speed
		if NPC.canMove == false: # If cannot move
			NPC.NPCMOVEMENT.toggleNPCMovement() # Toggle movement
	elif NPC.currentTarget != null and NPC.currentTarget != Global.player and "undead" in NPC.currentTarget: # If target is not player and is undead
		if NPC.currentTarget.undead == true and NPC.NPCWEAPON.equipped_weapon != null: # If target is undead and weapon equipped
			NPC.NPCWEAPON.aim_weapon_at_target(delta) # Aim weapon at target
	if overlapping_areas.size() > 0: # If overlapping areas exist
		reset_path_variables() # Reset path variables
		return # Exit
	if NPC.currentThreat != null: # If current threat exists
		if distance_to_current_threat < NPC.zombieGroupIdleRadius: # If threat is close
			NPC.alarmed = true # Set alarmed flag
			NPC.change_to_state = true # Request state change
			reset_path_variables() # Reset path variables
			return # Exit
	elif NPC.direction.length() > 0: # If direction is valid
		NPC.NPCMOVEMENT.rotate_towards_direction(delta) # Rotate NPC
	else:
		NPC.MOVEMENT_SPEED = 0.0 # Set movement speed to zero
		NPC.curAnim = NPC.IDLE # Set animation
	if NPC.NPCWEAPON.equipped_weapon != null: # If weapon equipped
		NPC.NPCWEAPON.detect_target_at_a_distance(delta) # Detect target at distance

func act_alive_sentry_docile(_delta): # Handles sentry docile behavior
	NPC.MOVEMENT_SPEED = 0 # Set movement speed to zero
	NPC.canMove = false # Disable movement
	NPC.curAnim = NPC.SNIPERAIMING # Set animation
	if NPC.sniperCollisionShape.has_overlapping_areas(): # If sniper has overlapping areas
		for area in NPC.sniperCollisionShape.get_overlapping_areas(): # Iterate overlapping areas
			print(area.owner) # Print area owner
			if area.owner.undead == true: # If owner is undead
				NPC.sentryHasTarget = true # Set sentry target flag
				NPC.sentryTarget = area.owner # Set sentry target
				NPC.currentTarget = area.owner # Set current target
				NPC.curState = NPC.ALIVE_SENTRY_HOSTILE # Set state
				NPC.curAnim = NPC.SNIPERSHOOTING # Set animation

func act_alive_sentry_hostile(delta): # Handles sentry hostile behavior
	NPC.MOVEMENT_SPEED = 0 # Set movement speed to zero
	NPC.canMove = false # Disable movement
	NPC.curAnim = NPC.SNIPERSHOOTING # Set animation
	if NPC.currentTarget == NPC.player and NPC.NPCWEAPON.equipped_weapon != null: # If target is player and weapon equipped
		NPC.NPCWEAPON.aim_weapon_at_player(delta) # Aim weapon at player
	elif NPC.currentTarget != null and NPC.currentTarget != Global.player and "undead" in NPC.currentTarget: # If target is not player and is undead
		if NPC.currentTarget.undead == true and NPC.NPCWEAPON.equipped_weapon != null: # If target is undead and weapon equipped
			NPC.NPCWEAPON.aim_weapon_at_target(delta) # Aim weapon at target

func act_dead(): # Handles dead behavior
	if NPC.NPCWEAPON.equipped_weapon != null: # If weapon equipped
		NPC.NPCWEAPON.reset_player_raycast_visual() # Reset weapon raycast
	die() # Run die behavior
	NPC.MOVEMENT_SPEED = 0.0 # Set movement speed to zero
	return # Exit

func act_undead_docile(delta): # Handles undead docile behavior
	if NPC.undead == false or NPC.hasTurnedUndead == false: # If not undead or not turned
		NPC.NPCANIMATIONS.turn_undead() # Turn undead
	if NPC.dead == true and NPC.hasTurnedUndead == false: # If dead and not turned
		NPC.NPCANIMATIONS.show_bite_area_indicator() # Show bite area indicator
	if NPC.currentlyTurningUndead == true: # If currently turning undead
		NPC.NPCANIMATIONS.turning_undead(delta) # Run turning undead animation
	if NPC.hasTurnedUndead == false: # If not turned undead
		return # Exit
	distance_to_target = null # Reset distance to target
	if distance_to_player > NPC.zombieGroupIdleRadius and NPC.hasTurnedUndead and NPC.currentlyTurningUndead == false: # If far from player and turned undead
		NPC.waiting = false # Reset waiting flag
		NPC.MOVEMENT_SPEED = NPC.zombie_running_speed # Set running speed
		NPC.zombie_walk_speed_scale = (NPC.MOVEMENT_SPEED / 2.3) * 1.6 # Set walk speed scale
		NPC.curAnim = NPC.WALK # Set animation
		NPC.nav_3d.debug_path_custom_color = Color.CHARTREUSE # Set debug color
	if NPC.undead == true and NPC.moveTowardsPlayer == true and NPC.currentlyTurningUndead == false: # If undead and moving towards player
		if NPC.printPathDebugging == true: # Print debug info
			print("distance to player: ", distance_to_player)
		if NPC.waiting == true: # If waiting
			if NPC.printPathDebugging == true: # Print debug info
				print("waiting: ", NPC.waiting)
			if distance_to_player > NPC.zombieGroupIdleRadius: # If far from player
				NPC.waiting = false # Reset waiting flag
				NPC.MOVEMENT_SPEED = NPC.zombie_running_speed # Set running speed
				NPC.zombie_walk_speed_scale = (NPC.MOVEMENT_SPEED / 2.3) * 1.6 # Set walk speed scale
				NPC.curAnim = NPC.WALK # Set animation
				NPC.nav_3d.debug_path_custom_color = Color.CHARTREUSE # Set debug color
			else:
				NPC.NPCMOVEMENT.wait() # Set NPC to wait
		elif NPC.waiting == false: # If not waiting
			if NPC.currentAttack != null: # If attacking
				NPC.NPCANIMATIONS.resetAttacks() # Reset attacks
			NPC.NPCMOVEMENT.move_towards_player(delta) # Move towards player
			return # Exit
	else:
		pass # No action

func act_undead_hostile(delta): # Handles undead hostile behavior
	if NPC.undead == false or NPC.hasTurnedUndead == false: # If not undead or not turned
		NPC.NPCANIMATIONS.turn_undead() # Turn undead
	if NPC.dead == true and NPC.hasTurnedUndead == false: # If dead and not turned
		NPC.NPCANIMATIONS.show_bite_area_indicator() # Show bite area indicator
	if NPC.currentlyTurningUndead == true: # If currently turning undead
		NPC.NPCANIMATIONS.turning_undead(delta) # Run turning undead animation
	NPC.waiting = false # Reset waiting flag
	NPC.MOVEMENT_SPEED = NPC.zombie_walking_speed # Set walking speed
	NPC.zombie_walk_speed_scale = (NPC.MOVEMENT_SPEED / 2.3) * 1.6 # Set walk speed scale
	NPC.NPCMOVEMENT.move_towards_target(delta) # Move towards target
	NPC.NPCMOVEMENT.choose_attack() # Choose attack

func act_undead_target(delta): # Handles undead target behavior
	if NPC.undead == false or NPC.hasTurnedUndead == false: # If not undead or not turned
		NPC.NPCANIMATIONS.turn_undead() # Turn undead
	if NPC.dead == true and NPC.hasTurnedUndead == false: # If dead and not turned
		NPC.NPCANIMATIONS.show_bite_area_indicator() # Show bite area indicator
	if NPC.currentlyTurningUndead == true: # If currently turning undead
		NPC.NPCANIMATIONS.turning_undead(delta) # Run turning undead animation
	if NPC.hasTurnedUndead == false: # If not turned undead
		return # Exit
	if NPC.playerTarget == null: # If no player target
		return # Exit
	distance_to_player_target = npc_position.distance_to(NPC.playerTarget) # Calculate distance to player target
	if distance_to_player_target > 3: # If far from player target
		NPC.waiting = false # Reset waiting flag
		NPC.MOVEMENT_SPEED = NPC.zombie_walking_speed # Set walking speed
		NPC.zombie_walk_speed_scale = (NPC.MOVEMENT_SPEED / 2.3) * 1.6 # Set walk speed scale
		NPC.curAnim = NPC.WALK # Set animation
		if NPC.currentAttack != null: # If attacking
			NPC.NPCANIMATIONS.resetAttacks() # Reset attacks
		NPC.NPCMOVEMENT.move_towards_player_target(delta) # Move towards player target
	pass
	if NPC.playerTarget == null: return # If no player target, exit
	if npc_position.distance_to(NPC.playerTarget) > 3: # If far from player target
		NPC.waiting = false # Reset waiting flag
		NPC.MOVEMENT_SPEED = NPC.zombie_walking_speed # Set walking speed
		NPC.zombie_walk_speed_scale = (NPC.MOVEMENT_SPEED / 2.3) * 1.6 # Set walk speed scale
		NPC.curAnim = NPC.WALK # Set animation
		if NPC.currentAttack != null: # If attacking
			NPC.NPCANIMATIONS.resetAttacks() # Reset attacks
		NPC.NPCMOVEMENT.move_towards_player_target(delta) # Move towards player target

func act_alive_fear_stunned(delta): # Handles alive fear stunned behavior
	NPC.slowed = true # Set slowed flag
	NPC.MOVEMENT_SPEED = NPC.walking_speed * .5 # Set slowed speed
	NPC.NPCWEAPON.trying_to_get_a_view_of_player = false # Reset weapon view flag
	if NPC.NPCWEAPON.equipped_weapon != null: # If weapon equipped
		NPC.NPCWEAPON.reset_player_raycast_visual() # Reset weapon raycast
	if NPC.currentThreat == null: return # If no threat, exit
	if distance_to_current_threat < NPC.runFromZombieOuterRadius: # If threat is close
		if NPC.hitStumble == true: # If hit stumble
			stumble() # Run stumble behavior
		else:
			NPC.NPCMOVEMENT.move_away_from_threat(delta) # Move away from threat

@export var waiting_a_second_before_recalculating = false # Exported variable for waiting

func tell_LEO_to_mill_around(): # Sets NPC to mill around
	NPC.millingAround = true # Set milling around flag
	NPC.aiming = false # Reset aiming flag

func choose_new_docile_target(): # Picks a new docile target
	if Global.NPCSpawnerPositions.size() == 0 or NPC.aiming == true: # If no spawner positions or aiming
		return # Exit
	var max_retries = 5 # Max retries for finding target
	var retry_count = 0 # Retry counter
	var new_target = null # New target variable
	while retry_count < max_retries and new_target == null: # Retry loop
		var target_position = Global.NPCSpawnerPositions.pick_random() # Pick random position
		if is_position_walkable(target_position): # If position is walkable
			new_target = target_position # Set new target
		retry_count += 1 # Increment retry counter
	if new_target: # If new target found
		NPC.nav_3d.target_position = new_target # Set navigation target
		NPC.MOVEMENT_SPEED = NPC.walking_speed # Set walking speed
		NPC.waiting = false # Reset waiting flag
		if print_pathfinding_debug: # Print debug info
			print("NPC moving to:", new_target)
	else:
		if print_pathfinding_debug: # Print debug info
			print("Failed to find a valid position after retries.")
	NPC.check_distance_moved_timer.start(2) # Start distance moved timer

func move_towards_target_location(delta): # Moves NPC towards target location
	if NPC.npc_avoidance_raycast.get_collider() != null and NPC.npc_avoidance_raycast.get_collider().is_in_group("NPCs") and not NPC.avoidingOtherNPC: # If colliding with NPC
		avoid_other_NPC() # Avoid other NPC
		return # Exit
	NPC.MOVEMENT_SPEED = NPC.walking_speed # Set movement speed
	var navigation_agent = NPC.nav_3d # Get navigation agent
	if NPC.check_distance_moved_timer.is_stopped(): # If timer stopped
		NPC.check_distance_moved_timer.start() # Start timer
		checking_distance_of_movement = true # Set checking flag
	var next_path_position = navigation_agent.get_next_path_position() # Get next path position
	if next_path_position != last_next_path_position: # If position changed
		var raw_direction = next_path_position - NPC.global_transform.origin # Calculate direction
		cached_direction = raw_direction.normalized() if raw_direction.length() > 0.01 else Vector3.ZERO # Normalize direction
		last_next_path_position = next_path_position # Update last position
	var direction = cached_direction # Get cached direction
	if direction.length() > 0: # If direction is valid
		NPC.direction = direction # Set NPC direction
		rotate_towards_direction(delta) # Rotate NPC
		NPC.curAnim = NPC.WALK # Set animation
		NPC.velocity = direction * NPC.MOVEMENT_SPEED # Set velocity
	NPC.velocity = NPC.velocity.move_toward(NPC.safeVelocity, NPC.safeVelocityFloat) # Move velocity towards safe velocity
	NPC.move_and_slide() # Move NPC
	if NPC.global_transform.origin.distance_to(NPC.nav_3d.target_position) < 1.5 and print_pathfinding_debug: # If close to target
		print("NPC reached destination:", NPC.nav_3d.target_position) # Print debug info

@export var print_pathfinding_debug = false # Exported debug variable

func is_position_walkable(position: Vector3) -> bool: # Checks if position is walkable
	var space_state = NPC.get_world_3d().direct_space_state # Get space state
	var query = PhysicsShapeQueryParameters3D.new() # Create query
	query.transform = Transform3D(Basis(), position) # Set query transform
	query.shape = SphereShape3D.new() # Set query shape
	query.shape.radius = 0.5 # Set shape radius
	query.collision_mask = 1 # Set collision mask
	var results = space_state.intersect_shape(query) # Get intersection results
	return results.size() == 0 # Return true if no obstacles

var distance_to_current_threat # Stores distance to current threat
var distance_to_player_target # Stores distance to player target

var last_position: Vector3 = Vector3(1, 1, 1) # Stores last position for movement check
var distance_travelled = 5 # Stores distance travelled
func check_distance_of_movement(): # Checks distance NPC has moved
	var current_position = NPC.global_transform.origin # Get current position
	distance_travelled = current_position.distance_to(last_position) # Calculate distance travelled
	last_position = NPC.global_transform.origin # Update last position
	var movement_threshold = 15.00 # Set movement threshold
	if distance_travelled < movement_threshold or distance_travelled > 300 and !NPC.dead and !NPC.aiming or distance_travelled == 5.0: # If not moved enough or moved too far
		NPC.waiting = true # Set waiting flag
		NPC.change_to_state = true # Request state change
	else:
		pass # No action
	last_position = NPC.global_transform.origin # Update last position
	currentlyCheckingDistanceTravelled = true # Set checking flag

var checking_distance_of_movement = true # Flag for checking movement
func choose_new_hostile_target(): # Picks a new hostile target
	if NPC.currentTarget == null: # If no target
		print("No hostile target available.") # Print debug info
		pass # No action
		return # Exit
	var target_center: Vector3 = NPC.currentTarget.global_transform.origin # Get target center
	var max_retries: int = 5 # Max retries
	var retry_count: int = 0 # Retry counter
	var new_target = null # New target variable
	while retry_count < max_retries and new_target == null: # Retry loop
		var random_offset: Vector3 = Vector3(randf_range(-3, 3), 0, randf_range(-3, 3)) # Get random offset
		var candidate: Vector3 = target_center + random_offset # Calculate candidate position
		new_target = candidate # Set new target
	if new_target: # If new target found
		NPC.nav_3d.target_position = new_target # Set navigation target
		if print_pathfinding_debug: # Print debug info
			print("New hostile target chosen: ", new_target)
	else:
		if print_pathfinding_debug: # Print debug info
			print("Failed to find a valid hostile target near the current target.")

func move_towards_hostile_target(delta): # Moves NPC towards hostile target
	if NPC.currentTarget == null: # If no target
		NPC.hostile = false # Set hostile flag
		NPC.change_to_state = true # Request state change
		return # Exit
	if NPC.nav_3d.is_navigation_finished(): # If navigation finished
		NPC.hostile = false # Set hostile flag
		NPC.change_to_state = true # Request state change
		return # Exit
	var next_path_position: Vector3 = NPC.nav_3d.get_next_path_position() # Get next path position
	if next_path_position != last_next_path_position: # If position changed
		var raw_direction: Vector3 = next_path_position - NPC.global_transform.origin # Calculate direction
		cached_direction = raw_direction.normalized() if raw_direction.length() > 0.01 else Vector3.ZERO # Normalize direction
		last_next_path_position = next_path_position # Update last position
	var direction: Vector3 = cached_direction # Get cached direction
	if NPC.currentTarget == null or NPC.currentTarget.dead == true: # If no target or target is dead
		NPC.hostile = false # Set hostile flag
		NPC.change_to_state = true # Request state change
		if NPC.hasPlayerTarget == true: # If has player target
			NPC.currentTarget = NPC.playerTarget # Set player target
			NPC.curState = NPC.UNDEAD_TARGET # Set state
			NPC.curAnim = NPC.WALK # Set animation
			return # Exit
	if NPC.currentTarget == null: # If no target
		return # Exit
	if direction.length() > 0: # If direction is valid
		NPC.direction = direction # Set NPC direction
		rotate_towards_direction(delta) # Rotate NPC
		NPC.velocity = direction * NPC.MOVEMENT_SPEED # Set velocity
	NPC.velocity = NPC.velocity.move_toward(NPC.safeVelocity, NPC.safeVelocityFloat) # Move velocity towards safe velocity
	NPC.move_and_slide() # Move NPC

func hostile_undead_move_towards_target_location(delta): # Moves hostile undead NPC towards target
	NPC.MOVEMENT_SPEED = NPC.zombie_walking_speed # Set walking speed
	NPC.zombie_walk_speed_scale = (NPC.MOVEMENT_SPEED / 2.3) * 1.6 # Set walk speed scale
	var navigation_agent = NPC.nav_3d # Get navigation agent
	if NPC.check_distance_moved_timer.is_stopped(): # If timer stopped
		NPC.check_distance_moved_timer.start() # Start timer
		checking_distance_of_movement = true # Set checking flag
	if navigation_agent.is_navigation_finished(): # If navigation finished
		choose_new_hostile_target() # Pick new hostile target
		return # Exit
	var next_path_position = navigation_agent.get_next_path_position() # Get next path position
	if next_path_position != last_next_path_position: # If position changed
		var raw_direction = next_path_position - NPC.global_transform.origin # Calculate direction
		cached_direction = raw_direction.normalized() if raw_direction.length() > 0.01 else Vector3.ZERO # Normalize direction
		last_next_path_position = next_path_position # Update last position
	var direction = cached_direction # Get cached direction
	if direction.length() > 0: # If direction is valid
		NPC.direction = direction # Set NPC direction
		rotate_towards_direction(delta) # Rotate NPC
		NPC.curAnim = NPC.WALK # Set animation
		NPC.velocity = direction * NPC.MOVEMENT_SPEED # Set velocity
	NPC.velocity = NPC.velocity.move_toward(NPC.safeVelocity, NPC.safeVelocityFloat) # Move velocity towards safe velocity
	NPC.move_and_slide() # Move NPC

func update_current_target(): # Updates NPC's current target
	var candidates = overlapping_areas # Get candidates from overlapping areas
	var closest_target = null # Closest target variable
	var min_distance = INF # Minimum distance variable
	var giving_up = false # Giving up flag
	var increment = 0 # Increment counter
	while giving_up == false: # While not giving up
		print("ZOMBIE LOOKING FOR TARGETS") # Print debug info
		increment += 1 # Increment counter
		for candidate in candidates: # Iterate candidates
			print("ZOMBIE TARGET ", increment, " : ", candidate.name) # Print candidate name
			if "dead" in candidate: # If candidate has dead property
				print("ZOMBIE TARGET ", increment, " : ", " DEAD IS IN CANDIDATE ") # Print debug info
				if not candidate.dead: # If candidate is not dead
					NPC.currentTarget = candidate # Set current target
					print("ZOMBIE TARGET  ", increment, " : ", "  CANDIDATE IS NOT DEAD ") # Print debug info
					var nav_distance = NPC.nav_3d.distance_to_target(candidate.global_transform.origin) # Calculate navigation distance
					if nav_distance < min_distance: # If closer than previous
						print("ZOMBIE TARGET  ", increment, " : ", " IS THE NEW CLOSEST TARGET! ") # Print debug info
						min_distance = nav_distance # Update min distance
						closest_target = candidate # Update closest target
			if closest_target: # If closest target found
				NPC.currentTarget = closest_target # Set current target
				NPC.hostile = true # Set hostile flag
				NPC.change_to_state = true # Request state change
				giving_up = true # Set giving up flag
			else:
				NPC.hostile = false # Reset hostile flag
				NPC.change_to_state = true # Request state change
				print("ZOMBIE FOUND NO HUMANS") # Print debug info
				giving_up = true # Set giving up flag
		giving_up = true # Set giving up flag

var distance_to_target # Stores distance to target
var timerTrackingTurningUndead: Timer = Timer.new() # Timer for turning undead
var waitingASecondToUpdateComputationallyExpensiveVariables = false # Flag for waiting before expensive tasks
var waitingASecondTimer: Timer = Timer.new() # Timer for waiting a second
func set_nav3d_target_to_self(): # Sets navigation target to NPC's own position
	NPC.velocity = Vector3(0, 0, 0) # Reset velocity
	NPC.direction = Vector3(0, 0, 0) # Reset direction
	if timerTrackingTurningUndead.is_stopped() == false: # If timer not stopped
		return # Exit
	if NPC.hasTurnedUndead == false: # If not turned undead
		if !timerTrackingTurningUndead.is_connected("timeout", set_nav3d_target_to_self): # If timer not connected
			timerTrackingTurningUndead.connect("timeout", set_nav3d_target_to_self) # Connect timer signal
			NPC.add_child(timerTrackingTurningUndead) # Add timer to NPC
		timerTrackingTurningUndead.one_shot = true # Set timer to one shot
		timerTrackingTurningUndead.wait_time = 2.0 # Set timer interval
		timerTrackingTurningUndead.start() # Start timer
	else: return # Exit
	var target_position = NPC.global_transform.origin # Get NPC position
	NPC.nav_3d.target_position = target_position # Set navigation target

func setWaitingASecondToFalse(): # Sets waiting flag to false
	waitingASecondToUpdateComputationallyExpensiveVariables = false # Reset flag

func get_distance_to_current_threat(): # Calculates distance to current threat
	if NPC.currentThreat != null and !NPC.dead: # If threat exists and NPC is alive
		distance_to_current_threat = npc_position.distance_to(NPC.currentThreat.global_transform.origin) # Calculate distance

func get_overlapping_areas(): # Gets overlapping areas from threat detection
	if NPC.change_to_state and waitingASecondToUpdateComputationallyExpensiveVariables == false: # If should update
		overlapping_areas = NPC.threat_detection_area.get_overlapping_areas() # Get overlapping areas

func get_distance_to_player(): # Calculates distance to player
	distance_to_player = NPC.global_transform.origin.distance_to(NPC.player.global_transform.origin) # Calculate distance
	waitingASecondTimer.start(1.0) # Start waiting timer

var currentlyCheckingDistanceTravelled = true # Flag for checking distance travelled

func perform_computationally_expensive_tasks(): # Performs expensive tasks
	get_distance_to_current_threat() # Get distance to threat
	get_overlapping_areas() # Get overlapping areas
	get_distance_to_player() # Get distance to player

func _on_threat_timer_timeout(): # Called when threat timer times out
	getting_direction_away_from_threat = true # Set flag to get direction away from threat

var getting_direction_away_from_threat = true # Flag for getting direction away from threat
var move_towards_target_location_if_this_var_is_modulo10 = 1 # Counter for movement modulo
