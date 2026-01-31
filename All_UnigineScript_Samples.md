# Combined C++ Sources

## actor_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
PlayerActor actor;
PlayerPersecutor persecutor;
PlayerActorSkinned actor_skinned;

using Unigine::Samples;

/*
 */
string material_names[] = ( "players_red", "players_green", "players_blue", "players_orange", "players_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
class PlayerActorSkinned {
	
	enum {
		STATE_WALK = 0,
		STATE_CROUCH,
		STATE_RUN,
		STATE_FORWARD,
		STATE_BACKWARD,
		STATE_MOVE_LEFT,
		STATE_MOVE_RIGHT,
		STATE_TURN_LEFT,
		STATE_TURN_RIGHT,
		STATE_WALK_IDLE,
		STATE_WALK_JUMP,
		STATE_WALK_FORWARD,
		STATE_WALK_BACKWARD,
		STATE_WALK_MOVE_LEFT,
		STATE_WALK_MOVE_RIGHT,
		STATE_WALK_TURN_LEFT,
		STATE_WALK_TURN_RIGHT,
		STATE_RUN_FORWARD,
		STATE_RUN_BACKWARD,
		STATE_RUN_MOVE_LEFT,
		STATE_RUN_MOVE_RIGHT,
	};
	
	string animations[] = (
		STATE_WALK_IDLE			:	"idle.anim",
		STATE_WALK_JUMP			:	"jump.anim",
		STATE_WALK_FORWARD		:	"walk_fwd.anim",
		STATE_WALK_BACKWARD		:	"walk_bwd.anim",
		STATE_WALK_MOVE_LEFT	:	"walk_lt.anim",
		STATE_WALK_MOVE_RIGHT	:	"walk_rt.anim",
		STATE_WALK_TURN_LEFT	:	"walk_lt.anim",
		STATE_WALK_TURN_RIGHT	:	"walk_rt.anim",
		STATE_RUN_FORWARD		:	"run_fwd.anim",
		STATE_RUN_BACKWARD		:	"walk_bwd.anim",
		STATE_RUN_MOVE_LEFT		:	"run_lt.anim",
		STATE_RUN_MOVE_RIGHT	:	"run_rt.anim",
	);
	
	float fps[] = (
		STATE_WALK_IDLE			:	25.0f,
		STATE_WALK_JUMP			:	2.0f,
		STATE_WALK_FORWARD		:	25.0f,
		STATE_WALK_BACKWARD		:	45.0f,
		STATE_WALK_MOVE_LEFT	:	45.0f,
		STATE_WALK_MOVE_RIGHT	:	45.0f,
		STATE_WALK_TURN_LEFT	:	45.0f,
		STATE_WALK_TURN_RIGHT	:	45.0f,
		STATE_RUN_FORWARD		:	25.0f,
		STATE_RUN_BACKWARD		:	55.0f,
		STATE_RUN_MOVE_LEFT		:	25.0f,
		STATE_RUN_MOVE_RIGHT	:	25.0f,
	);
	
	PlayerActor actor;
	ObjectMeshSkinned mesh;
	
	float jump = 1.0f;
	float walk = 1.0f;
	float run = 0.0f;
	
	float forward = 0.0f;
	float backward = 0.0f;
	float move_left = 0.0f;
	float move_right = 0.0f;
	float turn_left = 0.0f;
	float turn_right = 0.0f;
	
	PlayerActorSkinned(PlayerActor a) {
		
		actor = a;
		
		mesh = addToEditor(new ObjectMeshSkinned(fullPath("uniginescript_samples/players/meshes/actor_00.mesh")));
		for(int i = 0; i < mesh.getNumSurfaces(); i++) {
			mesh.setMaterial(findMaterialByName(get_material(i)),i);
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setCollision(0,i);
		}
		mesh.setNumLayers(7);
		
		foreach(string animation; animations) {
			animation = "uniginescript_samples/players/meshes/actor_00/" + animation;
			mesh.setLayerAnimationFilePath(0,fullPath(animation));
		}
	}
	
	void set_layer(int layer,float weight) {
		mesh.setLayer(layer,1,weight);
	}
	
	void set_frame(int layer,int state,float time) {
		if(mesh.getLayerWeight(layer) > EPSILON) {
			mesh.setLayerAnimationFilePath(layer,animations[state]);
			mesh.setLayerFrame(layer,time * fps[state]);
		}
	}
	
	void update() {
		
		float time = engine.game.getTime();
		float ifps = engine.game.getIFps() * 4.0f;
		
		// jump
		vec3 p0 = actor.getPosition() + vec3(0.0f,0.0f,0.2f);
		vec3 p1 = actor.getPosition() - vec3(0.0f,0.0f,0.2f);
		if(actor.getGround() == 0 && engine.world.getIntersection(p0,p1,~0,(mesh),NULL) == NULL) {
			jump += ifps * 4.0f;
		} else {
			jump -= ifps * 4.0f;
		}
		
		jump = saturate(jump);
		
		// state
		if(actor.getState(PLAYER_ACTOR_STATE_RUN)) {
			walk -= ifps;
			run += ifps;
		} else {
			walk += ifps;
			run -= ifps;
		}
		
		walk = saturate(walk);
		run = saturate(run);
		
		mesh.setWorldTransform(actor.getWorldTransform() * rotateZ(90.0f));
		
		// front movement
		if(actor.getState(PLAYER_ACTOR_STATE_FORWARD)) {
			forward += ifps;
			backward -= ifps;
		} else if(actor.getState(PLAYER_ACTOR_STATE_BACKWARD)) {
			forward -= ifps;
			backward += ifps;
		} else {
			forward -= ifps;
			backward -= ifps;
		}
		
		forward = saturate(forward);
		backward = saturate(backward);
		
		// side movement
		if(actor.getState(PLAYER_ACTOR_STATE_MOVE_LEFT)) {
			move_left += ifps;
			move_right -= ifps;
		} else if(actor.getState(PLAYER_ACTOR_STATE_MOVE_RIGHT)) {
			move_left -= ifps;
			move_right += ifps;
		} else {
			move_left -= ifps;
			move_right -= ifps;
		}
		
		move_left = saturate(move_left);
		move_right = saturate(move_right);
		
		// turning
		if(actor.getState(PLAYER_ACTOR_STATE_TURN_LEFT)) {
			turn_left += ifps;
			turn_right -= ifps;
		} else if(actor.getState(PLAYER_ACTOR_STATE_TURN_RIGHT)) {
			turn_left -= ifps;
			turn_right += ifps;
		} else {
			turn_left -= ifps;
			turn_right -= ifps;
		}
		
		turn_left = saturate(turn_left);
		turn_right = saturate(turn_right);
		
		// current state
		int front = 0;
		int side = 0;
		int turn = 0;
		
		float idle_weight = 1.0f;
		float front_weight = 0.0f;
		float side_weight = 0.0f;
		float turn_weight = 0.0f;
		
		// front movement
		if(forward > backward) {
			front = STATE_FORWARD;
			front_weight = forward;
		} else if(backward > forward) {
			front = STATE_BACKWARD;
			front_weight = backward;
		}
		
		// side movement
		if(move_left > move_right) {
			side = STATE_MOVE_LEFT;
			side_weight = move_left;
		} else if(move_right > move_left) {
			side = STATE_MOVE_RIGHT;
			side_weight = move_right;
		}
		
		// turning
		if(turn_left > turn_right) {
			turn = STATE_TURN_LEFT;
			turn_weight = turn_left;
		} else if(turn_right > turn_left) {
			turn = STATE_TURN_RIGHT;
			turn_weight = turn_right;
		}
		
		// adjust weights
		idle_weight = (1.0f - jump) * saturate(idle_weight - front_weight - side_weight - turn_weight);
		turn_weight = (1.0f - jump) * saturate(turn_weight - front_weight - side_weight);
		side_weight = (1.0f - jump) * saturate(side_weight - front_weight);
		front_weight = (1.0f - jump) * front_weight;
		
		// jump layers
		set_layer(0,jump);
		set_frame(0,STATE_WALK_JUMP,time);
		
		// walk layers
		set_layer(1,walk * idle_weight + run * idle_weight);
		set_frame(1,STATE_WALK_IDLE,time);
		
		set_layer(2,walk * front_weight);
		if(front == STATE_FORWARD) set_frame(2,STATE_WALK_FORWARD,actor.getStateTime(PLAYER_ACTOR_STATE_FORWARD));
		else if(front == STATE_BACKWARD) set_frame(2,STATE_WALK_BACKWARD,actor.getStateTime(PLAYER_ACTOR_STATE_BACKWARD));
		
		set_layer(3,walk * side_weight);
		if(side == STATE_MOVE_LEFT) set_frame(3,STATE_WALK_MOVE_LEFT,actor.getStateTime(PLAYER_ACTOR_STATE_MOVE_LEFT));
		else if(side == STATE_MOVE_RIGHT) set_frame(3,STATE_WALK_MOVE_RIGHT,actor.getStateTime(PLAYER_ACTOR_STATE_MOVE_RIGHT));
		
		set_layer(4,walk * turn_weight + run * turn_weight);
		if(turn == STATE_TURN_LEFT) set_frame(4,STATE_WALK_TURN_LEFT,actor.getStateTime(PLAYER_ACTOR_STATE_TURN_LEFT));
		else if(turn == STATE_TURN_RIGHT) set_frame(4,STATE_WALK_TURN_RIGHT,actor.getStateTime(PLAYER_ACTOR_STATE_TURN_RIGHT));
		
		// run layers
		set_layer(5,run * front_weight);
		if(front == STATE_FORWARD) set_frame(5,STATE_RUN_FORWARD,actor.getStateTime(PLAYER_ACTOR_STATE_FORWARD));
		else if(front == STATE_BACKWARD) set_frame(5,STATE_RUN_BACKWARD,actor.getStateTime(PLAYER_ACTOR_STATE_BACKWARD));
		
		set_layer(6,run * side_weight);
		if(side == STATE_MOVE_LEFT) set_frame(6,STATE_RUN_MOVE_LEFT,actor.getStateTime(PLAYER_ACTOR_STATE_MOVE_LEFT));
		else if(side == STATE_MOVE_RIGHT) set_frame(6,STATE_RUN_MOVE_RIGHT,actor.getStateTime(PLAYER_ACTOR_STATE_MOVE_RIGHT));
	}
};

/*
 */
int update() {
	
	actor_skinned.update();
	persecutor.setPhiAngle(actor.getPhiAngle());
	persecutor.setThetaAngle(actor.getThetaAngle());
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlane();
	setDescription("Third person Actor");
	
	int size = 2;
	float space = 4.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/players/meshes/sphere_00.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setCollision(1,0);
		}
	}
	
	actor = addToEditor(new PlayerActor());
	actor.setPosition(Vec3(-10.0f,0.0f,2.0f));
	actor.setMinThetaAngle(20.0f);
	actor.setMaxThetaAngle(60.0f);
	actor.setThetaAngle(20.0f);
	actor.setJumping(2.0f);
	
	actor_skinned = new PlayerActorSkinned(actor);
	
	persecutor = addToEditor(new PlayerPersecutor());
	persecutor.setPosition(Vec3(-20.0f,0.0f,2.0f));
	persecutor.setControls(0);
	persecutor.setTarget(actor);
	persecutor.setAnchor(vec3(0.0f,0.0f,1.0f));
	persecutor.setMinDistance(2.0f);
	persecutor.setMaxDistance(4.0f);
	persecutor.setMinThetaAngle(20.0f);
	persecutor.setMaxThetaAngle(60.0f);
	engine.game.setPlayer(persecutor);
	
	return 1;
}

```

## actor_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "players_red", "players_green", "players_blue", "players_orange", "players_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
class PlayerActorSkinned {
	
	enum {
		STATE_WALK = 0,
		STATE_CROUCH,
		STATE_RUN,
		STATE_FORWARD,
		STATE_BACKWARD,
		STATE_MOVE_LEFT,
		STATE_MOVE_RIGHT,
		STATE_TURN_LEFT,
		STATE_TURN_RIGHT,
		STATE_WALK_IDLE,
		STATE_WALK_JUMP,
		STATE_WALK_FORWARD,
		STATE_WALK_BACKWARD,
		STATE_WALK_MOVE_LEFT,
		STATE_WALK_MOVE_RIGHT,
		STATE_WALK_TURN_LEFT,
		STATE_WALK_TURN_RIGHT,
		STATE_RUN_FORWARD,
		STATE_RUN_BACKWARD,
		STATE_RUN_MOVE_LEFT,
		STATE_RUN_MOVE_RIGHT,
	};
	
	string animations[] = (
		STATE_WALK_IDLE			:	"idle.anim",
		STATE_WALK_JUMP			:	"jump.anim",
		STATE_WALK_FORWARD		:	"walk_fwd.anim",
		STATE_WALK_BACKWARD		:	"walk_bwd.anim",
		STATE_WALK_MOVE_LEFT	:	"walk_lt.anim",
		STATE_WALK_MOVE_RIGHT	:	"walk_rt.anim",
		STATE_WALK_TURN_LEFT	:	"walk_lt.anim",
		STATE_WALK_TURN_RIGHT	:	"walk_rt.anim",
		STATE_RUN_FORWARD		:	"run_fwd.anim",
		STATE_RUN_BACKWARD		:	"walk_bwd.anim",
		STATE_RUN_MOVE_LEFT		:	"run_lt.anim",
		STATE_RUN_MOVE_RIGHT	:	"run_rt.anim",
	);
	
	float fps[] = (
		STATE_WALK_IDLE			:	25.0f,
		STATE_WALK_JUMP			:	2.0f,
		STATE_WALK_FORWARD		:	25.0f,
		STATE_WALK_BACKWARD		:	45.0f,
		STATE_WALK_MOVE_LEFT	:	45.0f,
		STATE_WALK_MOVE_RIGHT	:	45.0f,
		STATE_WALK_TURN_LEFT	:	45.0f,
		STATE_WALK_TURN_RIGHT	:	45.0f,
		STATE_RUN_FORWARD		:	25.0f,
		STATE_RUN_BACKWARD		:	55.0f,
		STATE_RUN_MOVE_LEFT		:	25.0f,
		STATE_RUN_MOVE_RIGHT	:	25.0f,
	);
	
	PlayerActor actor;
	ObjectMeshSkinned mesh;
	
	float jump = 1.0f;
	float walk = 1.0f;
	float run = 0.0f;
	
	float forward = 0.0f;
	float backward = 0.0f;
	float move_left = 0.0f;
	float move_right = 0.0f;
	float turn_left = 0.0f;
	float turn_right = 0.0f;
	
	PlayerActorSkinned(PlayerActor a) {
		
		actor = a;
		
		mesh = addToEditor(new ObjectMeshSkinned(fullPath("uniginescript_samples/players/meshes/actor_00.mesh")));
		for(int i = 0; i < mesh.getNumSurfaces(); i++) {
			mesh.setMaterial(findMaterialByName(get_material(i)),i);
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setCollision(0,i);
		}
		mesh.setNumLayers(7);
		
		foreach(string animation; animations) {
			animation = "uniginescript_samples/players/meshes/actor_00/" + animation;
			mesh.setLayerAnimationFilePath(0,fullPath(animation));
		}
	}
	
	void set_layer(int layer,float weight) {
		mesh.setLayer(layer,1,weight);
	}
	
	void set_frame(int layer,int state,float time) {
		if(mesh.getLayerWeight(layer) > EPSILON) {
			mesh.setLayerAnimationFilePath(layer,animations[state]);
			mesh.setLayerFrame(layer,time * fps[state]);
		}
	}
	
	void update() {
		
		float time = engine.game.getTime();
		float ifps = engine.game.getIFps() * 4.0f;
		
		// jump
		vec3 p0 = actor.getPosition() + vec3(0.0f,0.0f,0.2f);
		vec3 p1 = actor.getPosition() - vec3(0.0f,0.0f,0.2f);
		if(actor.getGround() == 0 && engine.world.getIntersection(p0,p1,~0,(mesh),NULL) == NULL) {
			jump += ifps * 4.0f;
		} else {
			jump -= ifps * 4.0f;
		}
		
		jump = saturate(jump);
		
		// state
		if(actor.getState(PLAYER_ACTOR_STATE_RUN)) {
			walk -= ifps;
			run += ifps;
		} else {
			walk += ifps;
			run -= ifps;
		}
		
		walk = saturate(walk);
		run = saturate(run);
		
		mesh.setWorldTransform(actor.getWorldTransform() * rotateZ(90.0f));
		
		// front movement
		if(actor.getState(PLAYER_ACTOR_STATE_FORWARD)) {
			forward += ifps;
			backward -= ifps;
		} else if(actor.getState(PLAYER_ACTOR_STATE_BACKWARD)) {
			forward -= ifps;
			backward += ifps;
		} else {
			forward -= ifps;
			backward -= ifps;
		}
		
		forward = saturate(forward);
		backward = saturate(backward);
		
		// side movement
		if(actor.getState(PLAYER_ACTOR_STATE_MOVE_LEFT)) {
			move_left += ifps;
			move_right -= ifps;
		} else if(actor.getState(PLAYER_ACTOR_STATE_MOVE_RIGHT)) {
			move_left -= ifps;
			move_right += ifps;
		} else {
			move_left -= ifps;
			move_right -= ifps;
		}
		
		move_left = saturate(move_left);
		move_right = saturate(move_right);
		
		// turning
		if(actor.getState(PLAYER_ACTOR_STATE_TURN_LEFT)) {
			turn_left += ifps;
			turn_right -= ifps;
		} else if(actor.getState(PLAYER_ACTOR_STATE_TURN_RIGHT)) {
			turn_left -= ifps;
			turn_right += ifps;
		} else {
			turn_left -= ifps;
			turn_right -= ifps;
		}
		
		turn_left = saturate(turn_left);
		turn_right = saturate(turn_right);
		
		// current state
		int front = 0;
		int side = 0;
		int turn = 0;
		
		float idle_weight = 1.0f;
		float front_weight = 0.0f;
		float side_weight = 0.0f;
		float turn_weight = 0.0f;
		
		// front movement
		if(forward > backward) {
			front = STATE_FORWARD;
			front_weight = forward;
		} else if(backward > forward) {
			front = STATE_BACKWARD;
			front_weight = backward;
		}
		
		// side movement
		if(move_left > move_right) {
			side = STATE_MOVE_LEFT;
			side_weight = move_left;
		} else if(move_right > move_left) {
			side = STATE_MOVE_RIGHT;
			side_weight = move_right;
		}
		
		// turning
		if(turn_left > turn_right) {
			turn = STATE_TURN_LEFT;
			turn_weight = turn_left;
		} else if(turn_right > turn_left) {
			turn = STATE_TURN_RIGHT;
			turn_weight = turn_right;
		}
		
		// adjust weights
		idle_weight = (1.0f - jump) * saturate(idle_weight - front_weight - side_weight - turn_weight);
		turn_weight = (1.0f - jump) * saturate(turn_weight - front_weight - side_weight);
		side_weight = (1.0f - jump) * saturate(side_weight - front_weight);
		front_weight = (1.0f - jump) * front_weight;
		
		// jump layers
		set_layer(0,jump);
		set_frame(0,STATE_WALK_JUMP,time);
		
		// walk layers
		set_layer(1,walk * idle_weight + run * idle_weight);
		set_frame(1,STATE_WALK_IDLE,time);
		
		set_layer(2,walk * front_weight);
		if(front == STATE_FORWARD) set_frame(2,STATE_WALK_FORWARD,actor.getStateTime(PLAYER_ACTOR_STATE_FORWARD));
		else if(front == STATE_BACKWARD) set_frame(2,STATE_WALK_BACKWARD,actor.getStateTime(PLAYER_ACTOR_STATE_BACKWARD));
		
		set_layer(3,walk * side_weight);
		if(side == STATE_MOVE_LEFT) set_frame(3,STATE_WALK_MOVE_LEFT,actor.getStateTime(PLAYER_ACTOR_STATE_MOVE_LEFT));
		else if(side == STATE_MOVE_RIGHT) set_frame(3,STATE_WALK_MOVE_RIGHT,actor.getStateTime(PLAYER_ACTOR_STATE_MOVE_RIGHT));
		
		set_layer(4,walk * turn_weight + run * turn_weight);
		if(turn == STATE_TURN_LEFT) set_frame(4,STATE_WALK_TURN_LEFT,actor.getStateTime(PLAYER_ACTOR_STATE_TURN_LEFT));
		else if(turn == STATE_TURN_RIGHT) set_frame(4,STATE_WALK_TURN_RIGHT,actor.getStateTime(PLAYER_ACTOR_STATE_TURN_RIGHT));
		
		// run layers
		set_layer(5,run * front_weight);
		if(front == STATE_FORWARD) set_frame(5,STATE_RUN_FORWARD,actor.getStateTime(PLAYER_ACTOR_STATE_FORWARD));
		else if(front == STATE_BACKWARD) set_frame(5,STATE_RUN_BACKWARD,actor.getStateTime(PLAYER_ACTOR_STATE_BACKWARD));
		
		set_layer(6,run * side_weight);
		if(side == STATE_MOVE_LEFT) set_frame(6,STATE_RUN_MOVE_LEFT,actor.getStateTime(PLAYER_ACTOR_STATE_MOVE_LEFT));
		else if(side == STATE_MOVE_RIGHT) set_frame(6,STATE_RUN_MOVE_RIGHT,actor.getStateTime(PLAYER_ACTOR_STATE_MOVE_RIGHT));
	}
};

/*
 */
class PlayerActorControls {
	
	float time;
	PlayerActor actor;
	ControlsDummy controls;
	
	PlayerActorControls(PlayerActor a) {
		
		actor = a;
		
		controls = new ControlsDummy();
		actor.setControls(controls);
	}
	
	int sleep(float t) {
		sleep:
		time = t;
		while(time > 0.0f) {
			time -= engine.game.getIFps();
			yield 1;
		}
		yield 0;
		goto sleep;
	}
	
	int get_turn_direction() {
		if(engine.game.getRandom(0,32) % 2) return CONTROLS_STATE_TURN_LEFT;
		return CONTROLS_STATE_TURN_RIGHT;
	}
	
	int get_move_direction() {
		if(engine.game.getRandom(0,32) % 2) return CONTROLS_STATE_MOVE_LEFT;
		return CONTROLS_STATE_MOVE_RIGHT;
	}
	
	void update() {
		
		controls.setState(CONTROLS_STATE_FORWARD,1);
		yield;
		
		if(sleep(1.0f)) return;
		
		// main loop
		update:
			
			// move forward
			controls.setState(CONTROLS_STATE_FORWARD,1);
			yield;
			
			// branching
			if(length(actor.getPosition()) > 12.0f) goto bound;
			if(length(actor.getVelocity()) < 0.5f) goto collision;
			
			goto update;
		
		// bound
		bound:
			
			// stop
			controls.setState(CONTROLS_STATE_FORWARD,0);
			yield;
			
			if(sleep(1.0f)) return;
			
			// jump
			controls.setState(CONTROLS_STATE_JUMP,1);
			yield;
			
			if(sleep(1.0f)) return;
			
			// turn backward
			controls.setState(CONTROLS_STATE_JUMP,0);
			controls.setState(get_turn_direction(),1);
			yield;
			
			// check direction
			vec3 direction = rotation(actor.getWorldTransform()) * vec3(1.0f,0.0f,0.0f);
			if(dot(direction,normalize(actor.getPosition())) > -0.9f) return;
			
			// move forward
			controls.setState(CONTROLS_STATE_TURN_LEFT,0);
			controls.setState(CONTROLS_STATE_TURN_RIGHT,0);
			controls.setState(CONTROLS_STATE_FORWARD,1);
			controls.setState(CONTROLS_STATE_RUN,1);
			yield;
			
			if(sleep(0.25f)) return;
			
			// back to main loop
			controls.setState(CONTROLS_STATE_FORWARD,0);
			controls.setState(CONTROLS_STATE_RUN,0);
			goto update;
		
		// collision
		collision:
			
			// stop
			controls.setState(CONTROLS_STATE_FORWARD,0);
			yield;
			
			if(sleep(0.25f)) return;
			
			// jump
			controls.setState(CONTROLS_STATE_FORWARD,1);
			controls.setState(CONTROLS_STATE_JUMP,1);
			yield;
			
			if(sleep(0.25f)) return;
			
			controls.setState(CONTROLS_STATE_FORWARD,0);
			controls.setState(CONTROLS_STATE_JUMP,0);
			yield;
			
			if(length(actor.getVelocity()) > 0.5f) goto update;
			
			// crouch
			controls.setState(CONTROLS_STATE_CROUCH,1);
			yield;
			
			if(sleep(0.25f)) return;
			
			// move backward
			controls.setState(CONTROLS_STATE_BACKWARD,1);
			yield;
			
			if(sleep(0.25f)) return;
			
			// move side
			controls.setState(CONTROLS_STATE_BACKWARD,0);
			controls.setState(get_move_direction(),1);
			yield;
			
			if(sleep(0.25f)) return;
			
			// stand and move forward
			controls.setState(CONTROLS_STATE_MOVE_LEFT,0);
			controls.setState(CONTROLS_STATE_MOVE_RIGHT,0);
			controls.setState(CONTROLS_STATE_CROUCH,0);
			controls.setState(CONTROLS_STATE_FORWARD,1);
			controls.setState(CONTROLS_STATE_RUN,1);
			yield;
			
			if(sleep(0.25f)) return;
			
			// back to main loop
			controls.setState(CONTROLS_STATE_FORWARD,0);
			controls.setState(CONTROLS_STATE_RUN,0);
			goto update;
	}
};

/*
 */
void player_update(vec3 position,vec3 direction) {
	
	PlayerActor actor = addToEditor(new PlayerActor());
	PlayerActorSkinned actor_skinned = new PlayerActorSkinned(actor);
	PlayerActorControls actor_controls = new PlayerActorControls(actor);
	
	ObjectMeshSkinned mesh = actor_skinned.mesh;
	mesh.setMaterial(findMaterialByName(get_material(engine.game.getRandom(0,32))),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	actor.setJumping(2.0f);
	actor.setPosition(position);
	actor.setDirection(direction);
	
	while(1) {
		
		actor_controls.update();
		actor_skinned.update();
		
		wait;
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlane();
	
	int num = 0;
	int size = 2;
	float space = 4.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			for(int z = 0; z < 4; z++) {
				ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/players/meshes/sphere_00.mesh")));
				mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space + vec3(0.0f,0.0f,0.5f + z * 1.0f)));
				mesh.setMaterial(findMaterialByName(get_material(x ^ y)),"*");
				mesh.setSurfaceProperty("surface_base","*");
				BodyRigid body = class_remove(new BodyRigid(mesh));
				ShapeSphere shape = class_remove(new ShapeSphere(body,0.5f));
				shape.setMass(200.0f);
			}
		}
	}
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			Vec3 position = Vec3(engine.game.getRandom(-8.0f,8.0f),engine.game.getRandom(-8.0f,8.0f),1.0f);
			vec3 direction = vec3(engine.game.getRandom(-1.0f,1.0f),engine.game.getRandom(-1.0f,1.0f),0.0f);
			thread("player_update",position,direction);
			num++;
		}
	}
	
	PlayerSpectator spectator = addToEditor(new PlayerSpectator());
	spectator.setPosition(Vec3(-16.0f,0.0f,8.0f));
	spectator.setDirection(vec3(1.0f,0.0f,-0.5f));
	engine.game.setPlayer(spectator);
	
	setDescription(format("%d Third person Actors",num));
	return 1;
}

```

## ambient_static_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Info info;
AmbientSource source;
float pitch = 1.0f;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	info = new Info("'z' - loop toggle\n'x' - play\n'c' - stop\n'n' - increase pitch\n'b' - decrease pitch");
	
	source = new AmbientSource(fullPath("uniginescript_samples/sounds/sounds/static_mono_00.oga"));
	source.setLoop(1);
	
	setDescription("Static mono AmbientSource");
	
	return 1;
}

int update() {
	
	if(engine.input.isKeyDown(INPUT_KEY_Z)) {
		source.setLoop(!source.getLoop());
		log.message("loop %d\n",source.getLoop());
	}
	if(engine.input.isKeyDown(INPUT_KEY_X)) {
		source.play();
		log.message("play\n");
	}
	if(engine.input.isKeyDown(INPUT_KEY_C)) {
		source.stop();
		log.message("stop\n");
	}
	if(engine.input.isKeyPressed(INPUT_KEY_N)) {
		pitch = clamp(pitch + engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	if(engine.input.isKeyPressed(INPUT_KEY_B)) {
		pitch = clamp(pitch - engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	
	if(source.isPlaying()) {
		engine.console.onscreenMessage("time: %f\n",source.getTime());
	}
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## ambient_static_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Info info;
AmbientSource source;
float pitch = 1.0f;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	info = new Info("'z' - loop toggle\n'x' - play\n'c' - stop\n'n' - increase pitch\n'b' - decrease pitch");
	
	source = new AmbientSource(fullPath("uniginescript_samples/sounds/sounds/static_stereo_00.oga"));
	source.setLoop(1);
	
	setDescription("Static stereo AmbientSource");
	
	return 1;
}

int update() {
	
	if(engine.input.isKeyDown(INPUT_KEY_Z)) {
		source.setLoop(!source.getLoop());
		log.message("loop %d\n",source.getLoop());
	}
	if(engine.input.isKeyDown(INPUT_KEY_X)) {
		source.play();
		log.message("play\n");
	}
	if(engine.input.isKeyDown(INPUT_KEY_C)) {
		source.stop();
		log.message("stop\n");
	}
	if(engine.input.isKeyPressed(INPUT_KEY_N)) {
		pitch = clamp(pitch + engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	if(engine.input.isKeyPressed(INPUT_KEY_B)) {
		pitch = clamp(pitch - engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	
	if(source.isPlaying()) {
		engine.console.onscreenMessage("time: %f\n",source.getTime());
	}
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## ambient_stream_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Info info;
AmbientSource source;
float pitch = 1.0f;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	info = new Info("'z' - loop toggle\n'x' - play\n'c' - stop\n'n' - increase pitch\n'b' - decrease pitch");
	
	source = new AmbientSource(fullPath("uniginescript_samples/sounds/sounds/stream_stereo_00.oga"),1);
	source.setLoop(1);
	
	setDescription("Streamed stereo AmbientSource");
	
	return 1;
}

int update() {
	
	if(engine.input.isKeyDown(INPUT_KEY_Z)) {
		source.setLoop(!source.getLoop());
		log.message("loop %d\n",source.getLoop());
	}
	if(engine.input.isKeyDown(INPUT_KEY_X)) {
		source.play();
		log.message("play\n");
	}
	if(engine.input.isKeyDown(INPUT_KEY_C)) {
		source.stop();
		log.message("stop\n");
	}
	if(engine.input.isKeyPressed(INPUT_KEY_N)) {
		pitch = clamp(pitch + engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	if(engine.input.isKeyPressed(INPUT_KEY_B)) {
		pitch = clamp(pitch - engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	
	if(source.isPlaying()) {
		engine.console.onscreenMessage("time: %f\n",source.getTime());
	}
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## ambient_stream_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Info info;
AmbientSource source;
float pitch = 1.0f;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	info = new Info("'z' - loop toggle\n'x' - play\n'c' - stop\n'n' - increase pitch\n'b' - decrease pitch");
	
	source = new AmbientSource(fullPath("uniginescript_samples/sounds/sounds/stream_stereo_01.oga"),1);
	source.setLoop(1);
	
	setDescription("Streamed stereo AmbientSource");
	
	return 1;
}

int update() {
	
	if(engine.input.isKeyDown(INPUT_KEY_Z)) {
		source.setLoop(!source.getLoop());
		log.message("loop %d\n",source.getLoop());
	}
	if(engine.input.isKeyDown(INPUT_KEY_X)) {
		source.play();
		log.message("play\n");
	}
	if(engine.input.isKeyDown(INPUT_KEY_C)) {
		source.stop();
		log.message("stop\n");
	}
	if(engine.input.isKeyPressed(INPUT_KEY_N)) {
		pitch = clamp(pitch + engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	if(engine.input.isKeyPressed(INPUT_KEY_B)) {
		pitch = clamp(pitch - engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	
	if(source.isPlaying()) {
		engine.console.onscreenMessage("time: %f\n",source.getTime());
	}
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## animation_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
FieldAnimation animations[0];

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("FieldAnimation and ObjectGrass");
	
	ObjectGrass grass = addToEditor(new ObjectGrass());
	grass.setWorldTransform(translate(Vec3(-128.0f,-128.0f,0.0f)));
	grass.setIntersection(1);
	grass.setMaxVisibleDistance(50.0f,0);
	grass.setMaxFadeDistance(200.0f,0);
	grass.setSizeX(256.0f);
	grass.setSizeY(256.0f);
	grass.setStep(64.0f);
	grass.setDensity(0.2f);
	grass.setMinHeight(vec4(3.0f),vec4(1.0f));
	grass.setMaxHeight(vec4(3.0f),vec4(1.0f));
	grass.setAspect(vec4(1.0f),vec4(0.5f));
	grass.setMaterial(findMaterialByName("fields_grass_animation"),"*");
	
	for(int i = 0; i < 8; i++) {
		FieldAnimation animation = addToEditor(new FieldAnimation(vec3(22.0f)));
		animation.setWorldTransform(Mat4(rotateZ(45.0f * i) * translate(32.0f,0.0f,0.0f)));
		animation.setEllipse(i % 2);
		animation.setAttenuation(4.0f);
		animation.setAnimationScale(8.0f);
		animation.setStem(2.0f);
		animation.setWind(Vec3(1.0f,0.0f,-1.0f));
		animations.append(animation);
	}
	
	if(engine.visualizer.isEnabled() != 1) {
		engine.visualizer.setEnabled(1);
	}

	return 1;
}

/*
 */
int update() {
	
	float ifps = engine.game.getIFps();
	foreach(FieldAnimation animation; animations) {
		animation.setWorldTransform(Mat4(rotateZ(ifps * 8.0f)) * animation.getWorldTransform());
		animation.renderVisualizer();
	}
	
	return 1;
}

int shutdown() {

	engine.visualizer.setEnabled(0);
	return 1;
}

```

## animation_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
FieldAnimation animations[0];

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("FieldAnimation and ObjectMeshStatic");
	
	int size = 2;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(node_load(fullPath("uniginescript_samples/fields/meshes/fields_birch.node")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * 16.0f));
		}
	}
	
	for(int i = 0; i < 4; i++) {
		FieldAnimation animation = addToEditor(new FieldAnimation(vec3(32.0f)));
		animation.setWorldTransform(Mat4(rotateZ(90.0f * i) * translate(32.0f,0.0f,0.0f)));
		animation.setEllipse(i % 2);
		animation.setAttenuation(4.0f);
		animation.setStem(3.0f);
		animation.setLeaf(4.0f);
		animation.setAnimationScale(8.0f);
		animation.setWind(vec3(1.0f,0.0f,-1.0f));
		animations.append(animation);
	}

	return 1;
}

/*
 */
int update() {
	engine.visualizer.setEnabled(1);
	float ifps = engine.game.getIFps();
	foreach(FieldAnimation animation; animations) {
		animation.setWorldTransform(Mat4(rotateZ(ifps * 8.0f)) * animation.getWorldTransform());
		animation.renderVisualizer();
	}
	
	return 1;
}

```

## animation_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectMeshSkinned mesh_0;
ObjectMeshSkinned mesh_1;
ObjectMeshSkinned mesh_combined;
float speed = 2.0f;
float weight = 0.0f;
float animation_speed = 25.0f;

using Unigine::Samples;

/*
 */
string material_names[] = ( "animation_red", "animation_green", "animation_blue", "animation_orange", "animation_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
ObjectMeshSkinned create_agent(Mat4 transform) {
	
	ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(fullPath("uniginescript_samples/animation/meshes/agent.mesh")));
	mesh.setWorldTransform(transform);
	
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_eye_lod0");
	mesh.setMaterial(findMaterialByName(get_material(2)),"agent_body_lod0");
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_head_lod0");
	mesh.setMaterial(findMaterialByName(get_material(4)),"agent_holster_lod0");
	
	mesh.setSurfaceProperty("surface_base","*");
	
	return mesh;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(3.0f,0.0f,2.0f));
	createDefaultPlane();
	setDescription("Interpolation between two layers");
	
	mesh_combined = create_agent(Mat4(rotateZ(90.0f)));
	mesh_combined.setNumLayers(3);
	
	// punch animation
	mesh_combined.setLayerAnimationFilePath(0,fullPath("uniginescript_samples/animation/animations/agent_punch.anim"));
	
	// inverse reference frame into the 2 layer
	mesh_combined.setLayerAnimationFilePath(1,fullPath("uniginescript_samples/animation/animations/agent_run.anim"));
	mesh_combined.setLayerFrame(1,5.0f);
	mesh_combined.inverseLayer(2,1);
	
	mesh_0 = create_agent(Mat4(rotateZ(90.0f) * translate(1.5f,0.0f,0.0f)));
	mesh_0.setLayerAnimationFilePath(0,fullPath("uniginescript_samples/animation/animations/agent_punch.anim"));
	
	mesh_1 = create_agent(Mat4(rotateZ(90.0f) * translate(-1.5f,0.0f,0.0f)));
	mesh_1.setLayerAnimationFilePath(0,fullPath("uniginescript_samples/animation/animations/agent_run.anim"));
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	weight += speed * engine.game.getIFps();
	if(weight < -1.0f || weight > 2.0f) speed = -speed;
	
	// set frames
	mesh_combined.setLayerFrame(0,time * animation_speed);
	mesh_combined.setLayerFrame(1,time * animation_speed);
	
	// multiple 1 layer by inverse reference frame
	mesh_combined.mulLayer(1,2,1);
	
	// combine two animations
	mesh_combined.mulLayer(0,0,1,saturate(weight) * 0.5f + 0.5f);
	
	mesh_0.setLayerFrame(0,time * animation_speed);
	mesh_1.setLayerFrame(0,time * animation_speed);
	
	return 1;
}

```

## animation_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectMeshSkinned mesh_0;
ObjectMeshSkinned mesh_1;
ObjectMeshSkinned mesh_combined;
float speed = 2.0f;
float weight = 0.0f;
float animation_speed = 25.0f;

using Unigine::Samples;

/*
 */
string material_names[] = ( "animation_red", "animation_green", "animation_blue", "animation_orange", "animation_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
ObjectMeshSkinned create_agent(Mat4 transform) {
	
	ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(fullPath("uniginescript_samples/animation/meshes/agent.mesh")));
	mesh.setWorldTransform(transform);
	
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_eye_lod0");
	mesh.setMaterial(findMaterialByName(get_material(2)),"agent_body_lod0");
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_head_lod0");
	mesh.setMaterial(findMaterialByName(get_material(4)),"agent_holster_lod0");
	
	mesh.setSurfaceProperty("surface_base","*");
	
	return mesh;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(3.0f,0.0f,2.0f));
	createDefaultPlane();
	setDescription("Combination of two animations");
	
	mesh_combined = create_agent(Mat4(rotateZ(90.0f)));
	mesh_combined.setNumLayers(3);
	
	// punch animation
	mesh_combined.setLayerAnimationFilePath(0,fullPath("uniginescript_samples/animation/animations/agent_punch.anim"));
	
	// inverse reference frame into the 2 layer
	mesh_combined.setLayerAnimationFilePath(1,fullPath("uniginescript_samples/animation/animations/agent_punch.anim"));
	mesh_combined.setLayerFrame(1,0.0f);
	mesh_combined.inverseLayer(2,1);
	
	mesh_0 = create_agent(Mat4(rotateZ(90.0f) * translate(1.5f,0.0f,0.0f)));
	mesh_0.setLayerAnimationFilePath(0,fullPath("uniginescript_samples/animation/animations/agent_punch.anim"));
	
	mesh_1 = create_agent(Mat4(rotateZ(90.0f) * translate(-1.5f,0.0f,0.0f)));
	mesh_1.setLayerAnimationFilePath(0,fullPath("uniginescript_samples/animation/animations/agent_punch.anim"));
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	weight += speed * engine.game.getIFps();
	if(weight < -1.0f || weight > 2.0f) speed = -speed;
	
	// set frames
	mesh_combined.setLayerFrame(0,time * animation_speed);
	mesh_combined.setLayerFrame(1,time * animation_speed);
	
	// multiple 1 layer by inverse reference frame
	mesh_combined.mulLayer(1,2,1);
	
	// combine two animations
	mesh_combined.mulLayer(0,0,1,saturate(weight) * 0.5f);
	
	mesh_0.setLayerFrame(0,time * animation_speed);
	mesh_1.setLayerFrame(0,time * animation_speed);
	
	return 1;
}

```

## animation_04.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectMeshSkinned mesh_0;
ObjectMeshSkinned mesh_1;
ObjectMeshSkinned mesh_combined;
float speed = 2.0f;
float weight = 0.0f;
float animation_speed = 25.0f;

using Unigine::Samples;

/*
 */
string material_names[] = ( "animation_red", "animation_green", "animation_blue", "animation_orange", "animation_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
ObjectMeshSkinned create_agent(Mat4 transform) {
	
	ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(fullPath("uniginescript_samples/animation/meshes/agent.mesh")));
	mesh.setWorldTransform(transform);
	
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_eye_lod0");
	mesh.setMaterial(findMaterialByName(get_material(2)),"agent_body_lod0");
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_head_lod0");
	mesh.setMaterial(findMaterialByName(get_material(4)),"agent_holster_lod0");
	
	mesh.setSurfaceProperty("surface_base","*");
	
	return mesh;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(3.0f,0.0f,2.0f));
	createDefaultPlane();
	setDescription("Combination of two animations");
	
	mesh_combined = create_agent(Mat4(rotateZ(90.0f)));
	mesh_combined.setNumLayers(3);
	
	// punch animation
	mesh_combined.setLayerAnimationFilePath(0,fullPath("uniginescript_samples/animation/animations/agent_punch.anim"));
	
	// inverse reference frame into the 2 layer
	mesh_combined.setLayerAnimationFilePath(1,fullPath("uniginescript_samples/animation/animations/agent_idle.anim"));
	mesh_combined.setLayerFrame(1,0.0f);
	mesh_combined.inverseLayer(2,1);
	
	mesh_0 = create_agent(Mat4(rotateZ(90.0f) * translate(1.5f,0.0f,0.0f)));
	mesh_0.setLayerAnimationFilePath(0,fullPath("uniginescript_samples/animation/animations/agent_punch.anim"));
	
	mesh_1 = create_agent(Mat4(rotateZ(90.0f) * translate(-1.5f,0.0f,0.0f)));
	mesh_1.setLayerAnimationFilePath(0,fullPath("uniginescript_samples/animation/animations/agent_idle.anim"));
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	weight += speed * engine.game.getIFps();
	if(weight < -1.0f || weight > 2.0f) speed = -speed;
	
	// set frames
	mesh_combined.setLayerFrame(0,0.0f);
	mesh_combined.setLayerFrame(1,time * animation_speed);
	
	// multiple 1 layer by inverse reference frame
	mesh_combined.mulLayer(1,2,1);
	
	// combine two animations
	mesh_combined.mulLayer(0,0,1,4.0f);
	
	mesh_0.setLayerFrame(0,0.0f);
	mesh_1.setLayerFrame(0,time * animation_speed);
	
	return 1;
}

```

## arttracker.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

class Info {
		
	string info;
	WidgetLabel label;
	WidgetSprite sprite;
		
	Info() {
		init();
	}
	Info(string s) {
		init();
		set(s);
	}
		
	void __restore__() {
		init();
	}
		
	void init() {
		Gui gui = engine.getGui();
		label = new WidgetLabel(gui);
		label.setFontOutline(1);
		label.setFont("console.ttf");
		sprite = new WidgetSprite(gui,"gui_white.png");
		sprite.setColor(vec4(0.0f,0.0f,0.0f,0.75f));
		set(info);
	}
		
	void set(string s) {
		info = s;
		label.setText(info);
		label.arrange();
		
		if (engine.editor.isLoaded()) return;
		
		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 window_size = main_window.getSize();
		
		label.setPosition((window_size.x - label.getWidth()) / 2,(window_size.y - label.getHeight()) / 3);
		sprite.setPosition(label.getPositionX() - 8,label.getPositionY() - 8);
		sprite.setWidth(label.getWidth() + 16);
		sprite.setHeight(label.getHeight() + 16);
		engine.gui.addChild(sprite,GUI_ALIGN_OVERLAP);
		engine.gui.addChild(label,GUI_ALIGN_OVERLAP);
	}
};

Info info;
 
#ifdef ART
int success = false;
#endif

#ifdef ARTRECORDER
int DATA_COUNT = 11;
string data[0];

void readDTrackRecorder(string filename)
{
	File file = new File();

	if(file.open(filename, "r"))
	{
		while(file.eof() != 1)
			data.append(file.readLine());
	}
	
	file.close();
	delete file;
}

int data_index = 0;
float time = 0.f;
#endif
 
int init() {
	createInterface(engine.world.getPath());

	engine.console.setOnscreen(1);

	Player player = new PlayerSpectator();
	player.setPosition(Vec3(0.0f,-3.401f,1.5f));
	player.setDirection(Vec3(0.0f,1.0f,-0.4f));
	engine.game.setPlayer(player);
	
	info = new Info();
	
	#ifdef ARTRECORDER
	readDTrackRecorder("artdata.drf");
	#endif
	
	#ifdef ART
	if(engine.dtrack.init("192.168.1.100", 5000))
	{
		log.message("success to init ART\n");
		success = true;
		//engine.dtrack.start();
	}
	else log.message("failed to init ART!\n");
	#endif
	
	return 1;
}

/*
 */
int shutdown() {

	engine.console.setOnscreen(0);
	return 1;
}

/*
 */
int update() {
	
	#ifdef ARTRECORDER
	time += engine.game.getIFps();
	if(time < 1.f/60.f) return 1;
	else time = 0.f;
	
	string str = format("ART DataRecorder:\n"); 
	for(int i = 0 ; i < DATA_COUNT; i++)
		str += format("%s", data[i + DATA_COUNT * data_index ]);
	info.set(str);
	
	data_index++;
	if(data_index >= data.size() / DATA_COUNT) data_index = 0;
	#endif
	
	#ifdef ART
	if(success)
	{	
		if(engine.dtrack.receive())
		{
			string str = format("Body num: %d\n", engine.dtrack.getNumBody());
			
			if(engine.dtrack.getNumBody() > 0)
			{
				str += format("Body 1 id: %d\n", engine.dtrack.getBodyId(0));
				
				dvec3 position = engine.dtrack.getBodyLocation(0);
		
				str += format("Body 1 PositionX: %f\n", position.x);			
				str += format("Body 1 PositionY: %f\n", position.y);
				str += format("Body 1 PositionZ: %f\n", position.z);
			}
			info.set(str);
		}
	}
	#endif	
	return 1;
}

```

## async_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

Info info;
Async async;
Image image;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	info = new Info();
	thread("update_thread");
	
	setDescription("Async Image operations");
	
	return 1;
}

/*
 */
void update_thread() {
	
	while(1) {
		if(async == NULL) async = new Async();
		
		if(image == NULL) {
			image = new Image();
			image.create2D(512,512,IMAGE_FORMAT_RGB8);
			forloop(int y = 0; image.getHeight()) {
				forloop(int x = 0; image.getWidth()) {
					image.set2D(x,y,vec4(vec3(((x ^ y) % 255) / 256.0f),1.0f));
				}
			}
		}
		
		float compress_time = clock();
		int compress_frames = engine.game.getFrame();
		
		// compress image
		int id = async.run(image,functionid(Image::compress),IMAGE_FORMAT_DXT1);
		while(async != NULL && async.isRunning(id)) wait;
		if(async == NULL) continue;
		
		compress_time = clock() - compress_time;
		compress_frames = engine.game.getFrame() - compress_frames;
		
		float decompress_time = clock();
		int decompress_frames = engine.game.getFrame();
		
		// decompress image
		id = async.run(image,functionid(Image::decompress));
		while(async != NULL && async.isRunning(id)) wait;
		if(async == NULL) continue;
		
		decompress_time = clock() - decompress_time;
		decompress_frames = engine.game.getFrame() - decompress_frames;
		
		info.set(
			format(
				"Compression: %.1fms %d frames\nDecompression: %.1fms %d frames",
				compress_time * 1000.0f,compress_frames,
				decompress_time * 1000.0f,decompress_frames
			)
		);
	}
}

/*
 */
int shutdown() {
	
	if(async != NULL) async.wait();
	
	return 1;
}

```

## async_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

int id;
Info info;
Async async;
Node nodes[0];

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	info = new Info();
	engine.console.run("render_taa 1 && render_motion_blur 0");
	thread("update_thread");
	
	setDescription("Async Node loading via Async class");
	
	return 1;
}

/*
 */
void update_thread() {
	
	while(1) {
		if(async == NULL) async = new Async();
		
		float loading_time = clock();
		int loading_frames = engine.game.getFrame();
		
		id = async.run(functionid(engine.world.loadNode),fullPath("uniginescript_samples/systems/nodes/async_02.node"),0,0);
		while(async != NULL && async.isRunning(id)) wait;
		if(async == NULL) continue;
		
		loading_time = clock() - loading_time;
		loading_frames = engine.game.getFrame() - loading_frames;
		
		Node node = async.getResult(id);
		id = -1;
		
		if(node == NULL) continue;
		
		node.setEnabled(0);
		node.setEnabled(1);
		nodes.append(node_append(node));
		
		forloop(int i = 0; nodes.size()) {
			float angle = 360.0f * float(i) / (nodes.size() - 1);
			nodes[i].setWorldTransform(Mat4(rotateZ(angle) * translate(32.0f,0.0f,0.0f) * rotateZ(180.0f)));
		}
		
		while(nodes.size() > 16) {
			node_delete(nodes[0]);
			nodes.remove(0);
		}
		
		info.set(format("Loading: %.1fms %d frames",loading_time * 1000.0f,loading_frames));
	}
}

/*
 */
int shutdown() {
	
	if(async != NULL) async.wait();
	
	if(async != NULL && id != -1) {
		Node node = async.getResult(id);
		if(node != NULL) node_delete(node_append(node));
	}
	
	return 1;
}

```

## async_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

int id;
Info info;
Node nodes[0];

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	info = new Info();
	engine.console.run("render_taa 1 && render_motion_blur 0");
	thread("update_thread");
	
	setDescription("Async Node loading via async queue");
	
	return 1;
}

/*
 */
void update_thread() {
	
	while(1) {
		float loading_time = clock();
		int loading_frames = engine.game.getFrame();
		
		id = engine.async.loadNode(fullPath("uniginescript_samples/systems/nodes/async_02.node"));
		
		Node node = NULL;
		while(engine.async.checkNode(id)) {
			node = engine.async.takeNode(id);
			if(node == NULL) wait;
		}
		
		loading_time = clock() - loading_time;
		loading_frames = engine.game.getFrame() - loading_frames;
		
		if(node == NULL) continue;
		
		node.setEnabled(0);
		node.setEnabled(1);
		nodes.append(node_append(node));
		
		forloop(int i = 0; nodes.size()) {
			float angle = 360.0f * float(i) / (nodes.size() - 1);
			nodes[i].setWorldTransform(Mat4(rotateZ(angle) * translate(32.0f,0.0f,0.0f) * rotateZ(180.0f)));
		}
		
		while(nodes.size() > 16) {
			node_delete(nodes[0]);
			nodes.remove(0);
		}
		
		info.set(format("Loading: %.1fms %d frames",loading_time * 1000.0f,loading_frames));
	}
}

/*
 */
int shutdown() {
	
	while(engine.async.checkNode(id)) {
		if(engine.async.removeNode(id)) break;
		usleep(1000);
	}
	
	return 1;
}

```

## auxiliary_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
ObjectMeshStatic meshes[0];

int num = 0;
float interval = 0.0f;

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("Auxiliary buffer with post_blur_mask material");
	
	engine.console.run("render_auxiliary 1");
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			meshes.append(mesh);
		}
	}
	
	engine.render.addScriptableMaterial(findMaterialByName("post_hblur_mask"));
	engine.render.addScriptableMaterial(findMaterialByName("post_vblur_mask"));
	
	return 1;
}

/*
 */
int update() {
	
	interval -= engine.game.getIFps();
	if (interval > 0.0f)
		return 1;
	
	interval += 0.25f;
	
	meshes[num].setMaterialState("auxiliary",0,0);
	meshes[num].setMaterialParameterFloat4("auxiliary_color",vec4_one,0);
	
	num = engine.game.getRandom(0,meshes.size());
	
	meshes[num].setMaterialState("auxiliary",1,0);
	meshes[num].setMaterialParameterFloat4("auxiliary_color",vec4(1.0f,0.0f,0.0f,0.0f),0);
	
	return 1;
}

```

## ball_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
Joint joints[0];

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
void create_joint_default(Body b0,Body b1) {
	JointBall j = class_remove(new JointBall(b0,b1));
	j.setLinearRestitution(0.8f);
	j.setAngularRestitution(0.8f);
	j.setLinearSoftness(0.0f);
	j.setAngularSoftness(0.0f);
	j.setAngularDamping(16.0f);
	j.setNumIterations(16);
	joints.append(j);
}

void create_joint_angular_0(Body b0,Body b1) {
	JointBall j = class_remove(new JointBall(b0,b1));
	j.setLinearRestitution(0.8f);
	j.setAngularRestitution(0.8f);
	j.setLinearSoftness(0.0f);
	j.setAngularSoftness(0.0f);
	j.setAngularDamping(16.0f);
	j.setAngularLimitAngle(140.0f);
	j.setNumIterations(16);
	joints.append(j);
}

void create_joint_angular_1(Body b0,Body b1) {
	JointBall j = class_remove(new JointBall(b0,b1));
	j.setLinearRestitution(0.8f);
	j.setAngularRestitution(0.8f);
	j.setLinearSoftness(0.0f);
	j.setAngularSoftness(0.0f);
	j.setAngularDamping(16.0f);
	j.setAngularLimitAngle(175.0f);
	j.setNumIterations(16);
	joints.append(j);
}

/*
 */
void create_body(string func,Mat4 transform) {
	
	float offset = 1.2f;
	
	Body b0 = createBodyBox(vec3(1.0f),0.0f,0.5f,0.5f,get_material(0),transform * translate(0.0f,0.0f,offset * 0.0f));
	Body b1 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 1.0f));
	Body b2 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 2.0f));
	Body b3 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 3.0f));
	Body b4 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 4.0f));
	Body b5 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 5.0f));
	Body b6 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 6.0f));
	Body b7 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(2),transform * translate(0.0f,0.0f,offset * 7.0f));
	
	call(func,b0,b1);
	call(func,b1,b2);
	call(func,b2,b3);
	call(func,b3,b4);
	call(func,b4,b5);
	call(func,b5,b6);
	call(func,b6,b7);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("JointBall");
	
	create_body("create_joint_default",translate(Vec3(0.0f,-10.0f,10.0f)));
	createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.5f,-9.5f,20.0f)));
	
	create_body("create_joint_angular_0",translate(Vec3(0.0f,0.0f,10.0f)));
	createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.5f,0.5f,20.0f)));
	
	create_body("create_joint_angular_1",translate(Vec3(0.0f,10.0f,10.0f)));
	createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.5f,10.5f,20.0f)));
	
	engine.visualizer.setEnabled(1);
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	
	foreach(Joint j; joints)
		j.renderVisualizer(vec4_one);
	
	return 1;
}

```

## ball_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(int size,Mat4 transform) {
	
	int num = 0;
	
	Body bodies[0];
	
	for(int k = -size; k <= size; k++) {
		for(int j = -size; j <= size; j++) {
			for(int i = -size; i <= size; i++) {
				bodies.append(createBodySphere(0.5f,1.0f,0.5f,0.5f,get_material(i ^ j * 2 ^ k * 4),transform * translate(vec3(i,j,k))));
			}
		}
	}
	
	void create_joint(Body b0,Body b1) {
		JointBall j = class_remove(new JointBall(b0,b1));
		j.setLinearRestitution(0.02f);
		j.setAngularRestitution(0.02f);
		j.setLinearSoftness(0.2f);
		j.setAngularSoftness(0.2f);
		j.setAngularDamping(40.0f);
		j.setNumIterations(2);
		num++;
	}
	
	int size_2 = size * 2;
	
	forloop(int k = 0; size_2 + 1) {
		forloop(int j = 0; size_2 + 1) {
			forloop(int i = 0; size_2 + 1) {
				
				if(i < size_2) {
					Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 1];
					create_joint(b0,b1);
				}
				
				if(j < size_2) {
					Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 1) + i + 0];
					create_joint(b0,b1);
				}
				
				if(k < size_2) {
					Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 1) + (size_2 + 1) * (j + 0) + i + 0];
					create_joint(b0,b1);
				}
			}
		}
	}
	
	return num;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	num += create_body(2,translate(Vec3(0.0f,0.0f,9.0f)));
	
	num += create_body(2,translate(Vec3(0.0f,0.0f,15.0f)));
	
	num += create_body(2,translate(Vec3(0.0f,0.0f,21.0f)) * rotateY(asin(rsqrt(3.0f)) * RAD2DEG) * rotateX(45.0f));
	
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3(-3.5f,-3.5f,2.0f)));
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3( 3.5f,-3.5f,2.0f)));
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3(-3.5f, 3.5f,2.0f)));
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3( 3.5f, 3.5f,2.0f)));
	
	setDescription(format("%d JointBall",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## ball_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(int size,Mat4 transform) {
	
	int num = 0;
	
	Body bodies[0];
	
	for(int k = -size; k <= size; k++) {
		for(int j = -size; j <= size; j++) {
			for(int i = -size; i <= size; i++) {
				bodies.append(createBodySphere(0.5f,1.0f,0.5f,0.5f,get_material(i ^ j * 2 ^ k * 4),transform * translate(vec3(i,j,k))));
			}
		}
	}
	
	void create_joint(Body b0,Body b1) {
		JointBall j = class_remove(new JointBall(b0,b1));
		j.setMaxForce(400.0f);
		j.setMaxTorque(400.0f);
		j.setLinearRestitution(0.02f);
		j.setAngularRestitution(0.02f);
		j.setLinearSoftness(0.2f);
		j.setAngularSoftness(0.2f);
		j.setAngularDamping(40.0f);
		j.setNumIterations(2);
		num++;
	}
	
	int size_2 = size * 2;
	
	forloop(int k = 0; size_2 + 1) {
		forloop(int j = 0; size_2 + 1) {
			forloop(int i = 0; size_2 + 1) {
				
				if(i < size_2) {
					Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 1];
					create_joint(b0,b1);
				}
				
				if(j < size_2) {
					Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 1) + i + 0];
					create_joint(b0,b1);
				}
				
				if(k < size_2) {
					Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 1) + (size_2 + 1) * (j + 0) + i + 0];
					create_joint(b0,b1);
				}
			}
		}
	}
	
	return num;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	num += create_body(2,translate(Vec3(0.0f,0.0f,9.0f)));
	
	num += create_body(2,translate(Vec3(0.0f,0.0f,15.0f)));
	
	num += create_body(2,translate(Vec3(0.0f,0.0f,21.0f)) * rotateY(asin(rsqrt(3.0f)) * RAD2DEG) * rotateX(45.0f));
	
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3(-3.5f,-3.5f,2.0f)));
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3( 3.5f,-3.5f,2.0f)));
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3(-3.5f, 3.5f,2.0f)));
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3( 3.5f, 3.5f,2.0f)));
	
	setDescription(format("%d JointBall with destruction",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## ball_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(int size,Mat4 transform) {
	
	int num = 0;
	float offset = 1.1f;
	
	Body bodies[0];
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			bodies.append(createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),transform * translate(vec3(i,j,0.0f) * offset)));
		}
	}
	
	void create_joint(Body b0,Body b1) {
		JointBall j = class_remove(new JointBall(b0,b1));
		j.setLinearRestitution(0.2f);
		j.setAngularRestitution(0.2f);
		j.setLinearSoftness(0.2f);
		j.setAngularSoftness(0.2f);
		j.setAngularDamping(40.0f);
		j.setAngularLimitAngle(175.0f);
		j.setNumIterations(2);
		num++;
	}
	
	int size_2 = size * 2;
	
	forloop(int j = 0; size_2 + 1) {
		forloop(int i = 0; size_2 + 1) {
			
			if(i < size_2) {
				Body b0 = bodies[(size_2 + 1) * (j + 0) + i + 0];
				Body b1 = bodies[(size_2 + 1) * (j + 0) + i + 1];
				create_joint(b0,b1);
			}
			
			if(j < size_2) {
				Body b0 = bodies[(size_2 + 1) * (j + 0) + i + 0];
				Body b1 = bodies[(size_2 + 1) * (j + 1) + i + 0];
				create_joint(b0,b1);
			}
		}
	}
	
	return num;
}

/*
 */
void create_box(Mat4 transform) {
	return createBodyBox(vec3(2.0f,2.0f,4.0f),1.0f,0.5f,0.5f,get_material(0),transform);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	create_box(translate(Vec3(-4.0f,-4.0f,2.0f)));
	create_box(translate(Vec3( 4.0f,-4.0f,2.0f)));
	create_box(translate(Vec3(-4.0f, 4.0f,2.0f)));
	create_box(translate(Vec3( 4.0f, 4.0f,2.0f)));
	
	num += create_body(5,translate(Vec3(0.0f,0.0f,4.5f)));
	
	create_box(translate(Vec3(-4.0f,-4.0f,7.0f)));
	create_box(translate(Vec3( 4.0f,-4.0f,7.0f)));
	create_box(translate(Vec3(-4.0f, 4.0f,7.0f)));
	create_box(translate(Vec3( 4.0f, 4.0f,7.0f)));
	
	num += create_body(5,translate(Vec3(0.0f,0.0f,9.5f)));
	
	create_box(translate(Vec3(-4.0f,-4.0f,12.0f)));
	create_box(translate(Vec3( 4.0f,-4.0f,12.0f)));
	create_box(translate(Vec3(-4.0f, 4.0f,12.0f)));
	create_box(translate(Vec3( 4.0f, 4.0f,12.0f)));
	
	num += create_body(5,translate(Vec3(0.0f,0.0f,14.5f)));
	
	createBodyBox(vec3(4.0f),4.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.0f,0.0f,20.0f)));
	
	setDescription(format("%d JointBall with destruction",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## ball_04.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(int size,Mat4 transform) {
	
	int num = 0;
	float offset = 1.1f;
	
	Body bodies[0];
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			bodies.append(createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),transform * translate(vec3(i,j,0.0f) * offset)));
		}
	}
	
	void create_joint(Body b0,Body b1) {
		JointBall j = class_remove(new JointBall(b0,b1));
		j.setMaxForce(500.0f);
		j.setMaxTorque(500.0f);
		j.setLinearRestitution(0.2f);
		j.setAngularRestitution(0.2f);
		j.setLinearSoftness(0.2f);
		j.setAngularSoftness(0.2f);
		j.setAngularDamping(40.0f);
		j.setAngularLimitAngle(175.0f);
		j.setNumIterations(2);
		num++;
	}
	
	int size_2 = size * 2;
	
	forloop(int j = 0; size_2 + 1) {
		forloop(int i = 0; size_2 + 1) {
			
			if(i < size_2) {
				Body b0 = bodies[(size_2 + 1) * (j + 0) + i + 0];
				Body b1 = bodies[(size_2 + 1) * (j + 0) + i + 1];
				create_joint(b0,b1);
			}
			
			if(j < size_2) {
				Body b0 = bodies[(size_2 + 1) * (j + 0) + i + 0];
				Body b1 = bodies[(size_2 + 1) * (j + 1) + i + 0];
				create_joint(b0,b1);
			}
		}
	}
	
	return num;
}

/*
 */
void create_box(Mat4 transform) {
	return createBodyBox(vec3(2.0f,2.0f,4.0f),1.0f,0.5f,0.5f,get_material(0),transform);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	create_box(translate(Vec3(-4.0f,-4.0f,2.0f)));
	create_box(translate(Vec3( 4.0f,-4.0f,2.0f)));
	create_box(translate(Vec3(-4.0f, 4.0f,2.0f)));
	create_box(translate(Vec3( 4.0f, 4.0f,2.0f)));
	
	num += create_body(5,translate(Vec3(0.0f,0.0f,4.5f)));
	
	create_box(translate(Vec3(-4.0f,-4.0f,7.0f)));
	create_box(translate(Vec3( 4.0f,-4.0f,7.0f)));
	create_box(translate(Vec3(-4.0f, 4.0f,7.0f)));
	create_box(translate(Vec3( 4.0f, 4.0f,7.0f)));
	
	num += create_body(5,translate(Vec3(0.0f,0.0f,9.5f)));
	
	create_box(translate(Vec3(-4.0f,-4.0f,12.0f)));
	create_box(translate(Vec3( 4.0f,-4.0f,12.0f)));
	create_box(translate(Vec3(-4.0f, 4.0f,12.0f)));
	create_box(translate(Vec3( 4.0f, 4.0f,12.0f)));
	
	num += create_body(5,translate(Vec3(0.0f,0.0f,14.5f)));
	
	createBodyBox(vec3(4.0f),4.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.0f,0.0f,20.0f)));
	
	setDescription(format("%d JointBall with destruction",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## ball_05.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(int size,Vec3 position) {
	
	int num = 0;
	float offset = 1.1f;
	
	Body bodies[0];
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			bodies.append(createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(position + Vec3(size + 1.0f,i,j) * offset)));
		}
	}
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			bodies.append(createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(position + Vec3(-size,i + 1.0f,j) * offset)));
		}
	}
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			bodies.append(createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(position + Vec3(i + 1.0f,size + 1.0f,j) * offset)));
		}
	}
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			bodies.append(createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(position + Vec3(i,-size,j) * offset)));
		}
	}
	
	for(int j = -size; j < size; j++) {
		for(int i = -size; i < size; i++) {
			bodies.append(createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(position + Vec3(i + 1.0f,j + 1.0f,size) * offset)));
		}
	}
	for(int j = -size; j <= size + 1; j++) {
		for(int i = -size; i <= size + 1; i++) {
			bodies.append(createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(position + Vec3(i,j,-1.0f - size) * offset)));
		}
	}
	
	void create_joint(Body b0,Body b1) {
		JointBall j = class_remove(new JointBall(b0,b1));
		j.setMaxForce(800.0f);
		j.setMaxTorque(800.0f);
		j.setLinearRestitution(0.2f);
		j.setAngularRestitution(0.2f);
		j.setLinearSoftness(0.2f);
		j.setAngularSoftness(0.2f);
		j.setAngularDamping(40.0f);
		j.setAngularLimitAngle(170.0f);
		j.setNumIterations(2);
		num++;
	}
	
	int index = 0;
	int size_2 = size * 2;
	
	for(int k = 0; k < 4; k++) {
		
		forloop(int j = 0; size_2 + 1) {
			forloop(int i = 0; size_2 + 1) {
				
				if(i < size_2) {
					Body b0 = bodies[index + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[index + (size_2 + 1) * (j + 0) + i + 1];
					create_joint(b0,b1);
				}
				
				if(j < size_2) {
					Body b0 = bodies[index + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[index + (size_2 + 1) * (j + 1) + i + 0];
					create_joint(b0,b1);
				}
			}
		}
		
		index += (size_2 + 1) * (size_2 + 1);
	}
	
	forloop(int i = 0; size_2 + 1) {
		Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * 0 + (size_2 + 0) + (size_2 + 1) * i];
		Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * 2 + (size_2 + 0) + (size_2 + 1) * i];
		create_joint(b0,b1);
	}
	forloop(int i = 0; size_2 + 1) {
		Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * 0 + (size_2 + 1) * i];
		Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * 3 + (size_2 + 0) + (size_2 + 1) * i];
		create_joint(b0,b1);
	}
	forloop(int i = 0; size_2 + 1) {
		Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * 1 + (size_2 + 0) + (size_2 + 1) * i];
		Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * 2 + (size_2 + 1) * i];
		create_joint(b0,b1);
	}
	forloop(int i = 0; size_2 + 1) {
		Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * 1 + (size_2 + 1) * i];
		Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * 3 + (size_2 + 1) * i];
		create_joint(b0,b1);
	}
	
	forloop(int j = 0; size_2) {
		forloop(int i = 0; size_2) {
			
			if(i < size_2 - 1) {
				Body b0 = bodies[index + (size_2) * (j + 0) + i + 0];
				Body b1 = bodies[index + (size_2) * (j + 0) + i + 1];
				create_joint(b0,b1);
			}
			
			if(j < size_2 - 1) {
				Body b0 = bodies[index + (size_2) * (j + 0) + i + 0];
				Body b1 = bodies[index + (size_2) * (j + 1) + i + 0];
				create_joint(b0,b1);
			}
		}
	}
	
	forloop(int i = 0; size_2) {
		Body b0 = bodies[index + i];
		Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * 3 + (size_2 + 1) * (size_2) + 1 + i];
		create_joint(b0,b1);
	}
	forloop(int i = 0; size_2) {
		Body b0 = bodies[index + (size_2) * (size_2 - 1) + i];
		Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * 2 + (size_2 + 1) * (size_2) + i];
		create_joint(b0,b1);
	}
	forloop(int i = 0; size_2) {
		Body b0 = bodies[index + (size_2) * i];
		Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * 1 + (size_2 + 1) * (size_2) + i];
		create_joint(b0,b1);
	}
	forloop(int i = 0; size_2) {
		Body b0 = bodies[index + (size_2 - 1) + (size_2) * i];
		Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * 0 + (size_2 + 1) * (size_2) + 1 + i];
		create_joint(b0,b1);
	}
	
	index += (size_2) * (size_2);
	
	forloop(int j = 0; size_2 + 2) {
		forloop(int i = 0; size_2 + 2) {
			
			if(i <= size_2) {
				Body b0 = bodies[index + (size_2 + 2) * (j + 0) + i + 0];
				Body b1 = bodies[index + (size_2 + 2) * (j + 0) + i + 1];
				create_joint(b0,b1);
			}
			
			if(j <= size_2) {
				Body b0 = bodies[index + (size_2 + 2) * (j + 0) + i + 0];
				Body b1 = bodies[index + (size_2 + 2) * (j + 1) + i + 0];
				create_joint(b0,b1);
			}
		}
	}
	
	forloop(int i = 0; size_2) {
		Body b0 = bodies[index + i];
		Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * 3 + i];
		create_joint(b0,b1);
	}
	forloop(int i = 0; size_2) {
		Body b0 = bodies[index + (size_2 + 2) * (size_2 + 1) + 1 + i];
		Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * 2 + i];
		create_joint(b0,b1);
	}
	forloop(int i = 0; size_2 + 1) {
		Body b0 = bodies[index + (size_2 + 1) + (size_2 + 2) * i];
		Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * 0 + i];
		create_joint(b0,b1);
	}
	
	forloop(int i = 0; size_2 + 1) {
		Body b0 = bodies[index + (size_2 + 2) + (size_2 + 2) * i];
		Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * 1 + i];
		create_joint(b0,b1);
	}
	
	return num;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	num += create_body(2,Vec3(0.0f,0.0f,10.0f));
	num += create_body(2,Vec3(0.0f,0.0f,20.0f));
	num += create_body(1,Vec3(0.0f,0.0f,30.0f));
	
	setDescription(format("%d JointBall with destruction",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## ball_06.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(int width,int height,Mat4 transform) {
	
	int num = 0;
	float offset = 1.1f;
	
	Body bodies[0];
	
	for(int j = -height; j <= height; j++) {
		for(int i = -width; i <= width; i++) {
			float density = 1.0f;
			if(j == height) density = 0.0f;
			bodies.append(createBodyBox(vec3(1.0f),density,0.5f,0.5f,get_material(i ^ j * 2),transform * translate(vec3(i,j,0.0f) * offset)));
		}
	}
	
	void create_joint(Body b0,Body b1) {
		JointBall j = class_remove(new JointBall(b0,b1));
		j.setMaxForce(1000.0f);
		j.setMaxTorque(1000.0f);
		j.setLinearRestitution(0.8f);
		j.setAngularRestitution(0.8f);
		j.setLinearSoftness(0.8f);
		j.setAngularSoftness(0.8f);
		j.setAngularDamping(10.0f);
		j.setNumIterations(2);
		num++;
	}
	
	int width_2 = width * 2;
	int height_2 = height * 2;
	
	forloop(int j = 0; height_2 + 1) {
		forloop(int i = 0; width_2 + 1) {
			
			if(i < width_2) {
				Body b0 = bodies[(width_2 + 1) * (j + 0) + i + 0];
				Body b1 = bodies[(width_2 + 1) * (j + 0) + i + 1];
				create_joint(b0,b1);
			}
			
			if(j < height_2) {
				Body b0 = bodies[(width_2 + 1) * (j + 0) + i + 0];
				Body b1 = bodies[(width_2 + 1) * (j + 1) + i + 0];
				create_joint(b0,b1);
			}
		}
	}
	
	return num;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	num += create_body(16,8,translate(Vec3(0.0f,0.0f,11.0f)) * rotateY(90.0f) * rotateZ(90.0f));
	
	setDescription(format("%d JointBall with destruction",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## bone_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "worlds_mesh_red", "worlds_mesh_green", "worlds_mesh_blue", "worlds_mesh_orange", "worlds_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
ObjectMeshSkinned mesh;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int size = 2;
	
	mesh = addToEditor(new ObjectMeshSkinned(fullPath("uniginescript_samples/worlds/meshes/bone_00.mesh")));
	mesh.setAnimPath(fullPath("uniginescript_samples/worlds/meshes/bone_00.anim"));
	mesh.setMaterial(findMaterialByName(get_mesh_material(0)),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	for(int i = 0; i < mesh.getNumBones(); i++) {
		
		WorldTransformBone transform = addToEditor(new WorldTransformBone(mesh.getBoneName(i)));
		mesh.addChild(transform);
		
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
				mesh.setWorldTransform(translate(Vec3(0.0f,x,y) * 1.1f));
				mesh.setMaterial(findMaterialByName(get_mesh_material((x ^ y) + i)),"*");
				mesh.setSurfaceProperty("surface_base","*");
				transform.addChild(mesh);
			}
		}
	}
	
	setDescription("WorldTransformBone");
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	mesh.setLayerFrame(0,time * 10.0f);
	mesh.setWorldTransform(Mat4(rotateZ(time * 32.0f) * translate(0.0f,0.0f,1.0f) * scale(vec3(sin(time) * 0.25f + 1.0f))));
	
	return 1;
}

```

## box_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 16;
	vec3 space = vec3(1.0f,1.01f,1.0f);
	
	for(int j = 0; j < size; j++) {
		for(int i = 0; i < size - j; i++) {
			createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(0.0f,i + j * 0.5f - size * 0.5f + 0.5f,j + 0.5f) * space));
			num++;
		}
	}
	
	setDescription(format("%d ShapeBox",num));
	
	return 1;
}

```

## box_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 24;
	vec3 space = vec3(1.0f,1.01f,1.0f);
	
	for(int j = 0; j < size; j++) {
		for(int i = 0; i < size - j; i++) {
			createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(0.0f,i + j * 0.5f - size * 0.5f + 0.5f,j + 0.5f) * space));
			num++;
		}
	}
	
	setDescription(format("%d ShapeBox",num));
	
	return 1;
}

```

## box_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 4;
	int height = 12;
	vec3 space = vec3(1.0f,1.01f,1.0f);
	
	for(int i = 0; i < height; i++) {
		for(int j = -size; j <= size; j += 2) {
			createBodyBox(vec3(1.0f,2.0f,1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(size + 1.0f,j + i % 2 - 0.5f,i + 0.5f) * space));
			num++;
		}
		for(int j = -size; j <= size; j += 2) {
			createBodyBox(vec3(2.0f,1.0f,1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(j - i % 2 + 0.5f,size + 1.0f,i + 0.5f) * space));
			num++;
		}
		for(int j = -size; j <= size; j += 2) {
			createBodyBox(vec3(2.0f,1.0f,1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(j + i % 2 - 0.5f,-size - 1.0f,i + 0.5f) * space));
			num++;
		}
		for(int j = -size; j <= size; j += 2) {
			createBodyBox(vec3(1.0f,2.0f,1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(-size - 1.0f,j - i % 2 + 0.5f,i + 0.5f) * space));
			num++;
		}
	}
	
	createBodyBox(vec3(size * 2.0f + 3.0f,size * 2.0f + 3.0f,1.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3(0.0f,0.0f,height + 0.5f)));
	
	setDescription(format("%d ShapeBox",num));
	
	return 1;
}

```

## box_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 4;
	int height = 8;
	vec3 space = vec3(1.0f,1.01f,1.0f);
	
	for(int i = 0; i < height; i++) {
		if(i % 2) {
			for(int j = -size; j <= size; j += 2) {
				for(int k = -size; k <= size; k++) {
					createBodyBox(vec3(1.0f,2.0f,1.0f),1.0f,0.5f,0.5f,get_material((i ^ j * 2) + k),translate(Vec3(k,j + i % 2 - 0.5f,i + 0.5f) * space));
					num++;
				}
			}
		} else{
			for(int j = -size; j <= size; j++) {
				for(int k = -size; k <= size; k += 2) {
					createBodyBox(vec3(2.0f,1.0f,1.0f),1.0f,0.5f,0.5f,get_material((i ^ j * 2) + k),translate(Vec3(k + i % 2,j + 0.5f,i + 0.5f) * space));
					num++;
				}
			}
		}
	}
	
	setDescription(format("%d ShapeBox",num));
	
	return 1;
}

```

## box_04.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 10;
	vec3 space = vec3(1.0f,1.01f,1.0f);
	
	for(int k = 0; k < size; k++) {
		for(int j = 0; j < size - k; j++) {
			for(int i = 0; i < size - k; i++) {
				createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(i + k * 0.5f - size * 0.5f + 0.5f,j + k * 0.5f - size * 0.5f + 0.5f,k + 0.5f) * space));
				num++;
			}
		}
	}
	
	setDescription(format("%d ShapeBox",num));
	
	return 1;
}

```

## box_05.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 12;
	int counter = 8;
	vec3 space = vec3(2.0f,1.01f,1.0f);
	
	for(int k = 0; k < counter; k++) {
		for(int j = 0; j < size; j++) {
			for(int i = 0; i < size - j; i++) {
				createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material((i ^ j * 2) + k),translate(Vec3(k - counter * 0.5f,i + j * 0.5f - size * 0.5f + 0.5f,j + 0.5f) * space));
				num++;
			}
		}
	}
	
	setDescription(format("%d ShapeBox",num));
	
	return 1;
}

```

## box_06.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 4;
	int counter = 32;
	vec3 space = vec3(1.5f,1.5f,1.5f);
	
	for(int k = 0; k < counter; k++) {
		for(int j = -size; j <= size; j++) {
			for(int i = -size; i <= size; i++) {
				createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material((i ^ j * 2) + k),translate(Vec3(i,j,k + 0.5f) * space));
				num++;
			}
		}
	}
	
	setDescription(format("%d ShapeBox",num));
	
	return 1;
}

```

## box_07.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create shapes
	ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/shapes/meshes/mesh_00.mesh")));
	mesh.setMaterial(findMaterialByName("shapes_ground"),"*");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setMaterial(findMaterialByName(get_material(0)),"boxes");
	mesh.setMaterial(findMaterialByName(get_material(1)),"prisms");
	mesh.setMaterial(findMaterialByName(get_material(2)),"pyramids");
	mesh.setMaterial(findMaterialByName(get_material(3)),"parallelepipeds");
	
	forloop(int i = 0; 5;)
		mesh.setCollision(1,i);
	
	int num = 0;
	
	int size = 8;
	vec3 space = vec3(2.25f,2.25f,1.0f);
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(i,j,8.0f) * space));
			num++;
		}
	}
	
	setDescription(format("%d ShapeBox ObjectMeshStatic",num));
	
	return 1;
}

```

## box_08.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 8;
	int counter = 8;
	vec3 space = vec3(4.0f,1.01f,1.0f);
	
	for(int k = 0; k < counter; k++) {
		for(int j = 0; j < size; j++) {
			for(int i = 0; i < size - j; i++) {
				createBodyBox(vec3(1.0f),1,0.5f,0.5f,get_material((i ^ j * 2) + k),translate(Vec3(k - counter * 0.5f,i + j * 0.5f - size * 0.5f + 0.5f,j + 0.5f) * space));
				num++;
			}
		}
	}
	
	createBodyBox(vec3(1.0f,6.0f,6.0f),0.0f,0.5f,0.5f,get_material(0),translate(Vec3(-40.0f,0.0f,3.0f)));
	
	BodyRigid sphere = createBodySphere(0.25f,1000.0f,0.5f,0.5f,get_material(0),translate(Vec3(20.0f,0.0f,3.6f)));
	sphere.setLinearVelocity(vec3(-100.0f,0.0f,0.0f));
	
	setDescription(format("%d ShapeBox ShapeSphere CCD",num));
	
	return 1;
}

```

## button_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Window {
	
	Gui gui;				// gui
	
	WidgetButton toggle;	// toggle
	
	WidgetWindow window;	// window
	WidgetButton button;	// button
	
	// constructor/destructor
	Window() {
		
		gui = engine.getGui();
		
		// toggle button
		toggle = new WidgetButton(gui,"Press me");
		toggle.setToggleable(1);
		toggle.setFontSize(24);
		toggle.setPosition(128,128);
		gui.addChild(toggle,GUI_ALIGN_OVERLAP);
		toggle.getEventClicked().connect(functionid(callback_redirector), this, functionid(toggle_clicked));
		
		// window
		window = new WidgetWindow(gui,"Window");
		
		// close button
		button = new WidgetButton(gui,"Close");
		window.addChild(button,GUI_ALIGN_CENTER);
		button.getEventClicked().connect(functionid(callback_redirector), this, functionid(button_clicked));
		
		window.arrange();
		window.setSizeable(1);
	}
	~Window() {
		delete toggle;
		delete window;
		delete button;
	}
	
	// save/restore state
	void __restore__() {
		__Window__();
	}
	
	// toggle clicked callback
	void toggle_clicked() {
		if(toggle.isToggled()) gui.addChild(window,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
		else gui.removeChild(window);
	}
	
	// button clicked callback
	void button_clicked() {
		toggle.setToggled(0);
	}
	
	// callback redirector
	void callback_redirector(Window window,string func) {
		window.call(func);
	}
};

Window window;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	window = new Window();
	
	setDescription("Toggle button");
	
	return 1;
}

```

## callbacks_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
void frozen_callback(Body body) {
	Object object = body.getObject();
	object.setMaterial(findMaterialByName(get_material((body.isFrozen()) ? 2 : 0)),"*");
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Body frozen callback");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 8;
	float space = 2.02f;
	
	for(int j = 0; j < size; j++) {
		for(int i = 0; i < size - j; i++) {
			Body body = createBodyBox(vec3(2.0f),1,0.5f,0.5f,get_material(0),translate(Vec3(0.0f,i + j * 0.5f - size * 0.5f + 0.5f,j + 0.5f) * space));
			body.getEventFrozen().connect(functionid(frozen_callback));
		}
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## callbacks_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
void frozen_callback(Body body) {
	Object object = body.getObject();
	if(body.isFrozen()) object.setMaterial(findMaterialByName(get_material(2)),"*");
}

void position_callback(Body body) {
	Object object = body.getObject();
	object.setMaterial(findMaterialByName(get_material(0)),"*");
	log.message("%s is moved\n",typeinfo(body_cast(body)));
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Body position callback");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 8;
	float space = 2.02f;
	
	for(int j = 0; j < size; j++) {
		for(int i = 0; i < size - j; i++) {
			Body body = createBodyBox(vec3(2.0f),1,0.5f,0.5f,get_material(0),translate(Vec3(0.0f,i + j * 0.5f - size * 0.5f + 0.5f,j + 0.5f) * space));
			body.getEventPosition().connect(functionid(position_callback),body);
			body.getEventFrozen().connect(functionid(frozen_callback), body);
		}
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## callbacks_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
void contact_callback(Body body,int num) {
	if(body.getContactBody1(num) != NULL) {
		log.message("%s %s depth %f point %s\n",
			typeinfo(body_cast(body.getContactBody0(num))),
			typeinfo(body_cast(body.getContactBody1(num))),
			body.getContactDepth(num),string(body.getContactPoint(num)));
	} else {
		log.message("%s %s depth %f point %s\n",
			typeinfo(body_cast(body.getContactBody0(num))),
			typeinfo(node_cast(body.getContactObject(num))),
			body.getContactDepth(num),string(body.getContactPoint(num)));
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Body contact callback");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	createBodyBox(vec3(2.0f),1.0f,0.5f,0.5f, get_material(0), translate(Vec3(0.0f,0.0f,3.0f)));

	Body body = createBodyBox(vec3(2.0f),1.0f,0.5f,0.5f, get_material(1), translate(Vec3(0.0f,0.0f,6.0f)));
	body.getEventContactEnter().connect(functionid(contact_callback));
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## callbacks_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
void broken_callback(Joint joint) {
	if(joint.isBroken()) log.message("%s is broken\n",typeinfo(joint_cast(joint)));
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Joint broken callback");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 7;
	
	Body b0 = NULL;
	Body b1 = NULL;
	
	for(int i = -size; i <= size; i++) {
		float density = 1.0f;
		if(i == -size || i == size) density = 0.0f;
		b1 = createBodyBox(vec3(4.0f,2.0f,1.0f),density,0.5f,0.5f,get_material(0),translate(Vec3(0.0f,2.2f * i,8.0f)));
		if(b0 != NULL) {
			JointHinge j = class_remove(new JointHinge(b0,b1,Vec3(0.0f,2.2f * i - 1.1f,8.0f),vec3(1.0f,0.0f,0.0f)));
			j.setAngularDamping(8.0f);
			j.setNumIterations(2);
			j.setLinearRestitution(0.02f);
			j.setAngularRestitution(0.1f);
			j.setMaxForce(24000.0f);
			j.setMaxTorque(16000.0f);
			j.getEventBroken().connect(functionid(broken_callback));
		}
		b0 = b1;
	}
	
	for(int i = 0; i < 3; i++) {
		createBodyBox(vec3(2.0f),8.0f,0.5f,0.5f,get_material(1),translate(Vec3(0.0f,-6.6f,12.0f + i * 3.0f)));
		createBodyBox(vec3(2.0f),8.0f,0.5f,0.5f,get_material(1),translate(Vec3(0.0f,6.6f,12.0f + i * 3.0f)));
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## canvas_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Canvas {
	
	Gui gui;				// gui
	
	WidgetCanvas canvas;	// canvas
	
	// constructor/destructor
	Canvas() {
		
		gui = engine.getGui();
		
		// canvas
		canvas = new WidgetCanvas(gui);
		
		canvas.setLineColor(create_line(0,200.0f,200.0f,100.0f,3,360.0f),vec4(0.0f,0.0f,1.0f,1.0f));
		canvas.setLineColor(create_line(0,200.0f,200.0f,100.0f,4,360.0f),vec4(0.0f,1.0f,0.0f,1.0f));
		canvas.setLineColor(create_line(0,200.0f,200.0f,100.0f,5,360.0f),vec4(1.0f,0.0f,0.0f,1.0f));
		
		canvas.setLineColor(create_line(0,800.0f,400.0f,100.0f,16,360.0f * 9.0f),vec4(1.0f,1.0f,1.0f,1.0f));
		
		canvas.setPolygonColor(create_polygon(0,600.0f,200.0f,100.0f,6,360.0f),vec4(1.0f,0.0f,0.0f,1.0f));
		canvas.setPolygonColor(create_polygon(1,600.0f,200.0f,100.0f,3,360.0f),vec4(0.0f,0.0f,1.0f,1.0f));
		
		canvas.setPolygonColor(create_polygon(0,400.0f,400.0f,100.0f,8,360.0f),vec4(0.0f,1.0f,0.0f,1.0f));
		canvas.setPolygonColor(create_polygon(1,400.0f,400.0f,100.0f,4,360.0f),vec4(1.0f,0.0f,0.0f,1.0f));
		
		create_text(0,200.0f - 64.0f,200.0f - 30.0f,"This is canvas text");
		
		gui.addChild(canvas,GUI_ALIGN_OVERLAP | GUI_ALIGN_BACKGROUND);
	}
	~Canvas() {
		delete canvas;
	}
	
	// lines
	int create_line(int order,float x,float y,float radius,int num,float angle) {
		int line = canvas.addLine(order);
		forloop(int i = 0; num + 1) {
			float s = sin(angle / num * DEG2RAD * i) * radius + x;
			float c = cos(angle / num * DEG2RAD * i) * radius + y;
			canvas.addLinePoint(line,vec3(s,c,0.0f));
		}
		return line;
	}
	
	// polygons
	int create_polygon(int order,float x,float y,float radius,int num,float angle) {
		int polygon = canvas.addPolygon(order);
		forloop(int i = 0; num) {
			float s = sin(angle / num * DEG2RAD * i) * radius + x;
			float c = cos(angle / num * DEG2RAD * i) * radius + y;
			canvas.addPolygonPoint(polygon,vec3(s,c,0.0f));
		}
		return polygon;
	}
	
	// texts
	int create_text(int order,float x,float y,string str) {
		int text = canvas.addText(order);
		canvas.setTextPosition(text,vec3(x,y,0.0f));
		canvas.setTextText(text,str);
		return text;
	}
	
	// update
	void update() {
		float fov = 2.0f;
		float time = engine.game.getTime();
		
		if (engine.editor.isLoaded()) return;

		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 window_size = main_window.getSize();
		float x = float(window_size.x) / 2.0f;
		float y = float(window_size.y) / 2.0f;
		canvas.setTransform(translate(x,y,0.0f) * perspective(fov,1.0f,0.01f,100.0f) * rotateY(sin(time)) * rotateX(cos(time * 0.5f)) * translate(-x,-y,-1.0f / tan(fov * DEG2RAD * 0.5f)));
	}
	
	// save/restore state
	void __restore__() {
		__Canvas__();
	}
};

Canvas canvas;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	canvas = new Canvas();
	
	setDescription("WidgetCanvas");
	
	return 1;
}

/*
 */
int update() {
	
	canvas.update();
	
	return 1;
}

```

## canvas_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Canvas {
	
	Gui gui;				// gui
	
	WidgetCanvas canvas;	// canvas
	
	// constructor/destructor
	Canvas() {
		
		gui = engine.getGui();
		
		// canvas
		canvas = new WidgetCanvas(gui);
		
		canvas.setTexture(fullPath("uniginescript_samples/widgets/textures/background.png"));
		
		canvas.setPolygonColor(create_polygon(0,300.0f,300.0f,200.0f,0.5f,7,360.0f),vec4(0.0f,1.0f,1.0f,1.0f));
		canvas.setPolygonColor(create_polygon(1,300.0f,300.0f,150.0f,0.5f,9,360.0f),vec4(0.0f,1.0f,0.0f,1.0f));
		canvas.setPolygonColor(create_polygon(2,300.0f,300.0f,100.0f,0.5f,13,360.0f),vec4(1.0f,0.0f,0.0f,1.0f));
		
		canvas.setPolygonColor(create_polygon(0,800.0f,300.0f,200.0f,0.5f,7,360.0f),vec4(0.0f,1.0f,1.0f,1.0f));
		canvas.setPolygonColor(create_polygon(1,800.0f,300.0f,150.0f,0.5f,9,360.0f),vec4(1.0f,0.0f,0.0f,1.0f));
		canvas.setPolygonColor(create_polygon(2,800.0f,300.0f,100.0f,0.5f,13,360.0f),vec4(0.0f,1.0f,0.0f,1.0f));
		
		gui.addChild(canvas,GUI_ALIGN_OVERLAP | GUI_ALIGN_BACKGROUND);
	}
	~Canvas() {
		delete canvas;
	}
	
	// polygons
	int create_polygon(int order,float x,float y,float radius,float scale,int num,float angle) {
		int polygon = canvas.addPolygon(order);
		forloop(int i = 0; num) {
			float s = sin(angle / num * DEG2RAD * i);
			float c = cos(angle / num * DEG2RAD * i);
			canvas.addPolygonPoint(polygon,vec3(s * radius + x,c * radius + y,0.0f));
			canvas.setPolygonTexCoord(polygon,vec3(s * scale + 0.5f,c * scale + 0.5f,0.0f));
		}
		return polygon;
	}
	
	// update
	void update() {
		float fov = 2.0f;
		float time = engine.game.getTime();
		canvas.setTransform(translate(600.0f,200,0.0f) * perspective(fov,1.0f,0.01f,100.0f) * rotateY(sin(time)) * rotateX(cos(time * 0.5f)) * translate(-400.0f,-200.0f,-1.0f / tan(fov * DEG2RAD * 0.5f)));
	}
	
	// save/restore state
	void __restore__() {
		__Canvas__();
	}
};

Canvas canvas;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	canvas = new Canvas();
	
	setDescription("WidgetCanvas");
	
	return 1;
}

/*
 */
int update() {
	
	canvas.update();
	
	return 1;
}

```

## canvas_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Canvas {
	
	Gui gui;				// gui
	
	WidgetCanvas canvas;	// canvas
	
	int line;				// line
	int polygon;			// polygon
	
	// constructor/destructor
	Canvas() {
		
		gui = engine.getGui();
		
		// canvas
		canvas = new WidgetCanvas(gui);
		
		// vertices
		vec3 vertex[] = (
			vec3(-1.0f,-1.0f,-1.0f), vec3( 1.0f,-1.0f,-1.0f),
			vec3( 1.0f, 1.0f,-1.0f), vec3(-1.0f, 1.0f,-1.0f),
			vec3(-1.0f,-1.0f, 1.0f), vec3( 1.0f,-1.0f, 1.0f),
			vec3( 1.0f, 1.0f, 1.0f), vec3(-1.0f, 1.0f, 1.0f),
		);
		
		// line indices
		int line_indices[] = (
			0, 1, 1, 2, 2, 3, 3, 0,
			4, 5, 5, 6, 6, 7, 7, 4,
			0, 4, 1, 5, 2, 6, 3, 7,
		);
		
		// polygon indices
		int polygon_indices[] = (
			0, 1, 2, 2, 3, 0,
			4, 6, 5, 6, 4, 7,
			0, 4, 5, 5, 1, 0,
			2, 6, 7, 7, 3, 2,
			0, 7, 4, 7, 0, 3,
			2, 5, 6, 5, 2, 1,
		);
		
		// line
		line = canvas.addLine();
		forloop(int i = 0; vertex.size()) {
			canvas.addLinePoint(line,vertex[i] * 7.0f);
		}
		forloop(int i = 0; line_indices.size()) {
			canvas.addLineIndex(line,line_indices[i]);
		}
		
		// polygon
		polygon = canvas.addPolygon();
		canvas.setPolygonTwoSided(polygon,0);
		forloop(int i = 0; vertex.size()) {
			canvas.addPolygonPoint(polygon,vertex[i] * 7.0f);
		}
		forloop(int i = 0; polygon_indices.size()) {
			canvas.addPolygonIndex(polygon,polygon_indices[i]);
		}
		
		gui.addChild(canvas,GUI_ALIGN_OVERLAP | GUI_ALIGN_BACKGROUND);
	}
	~Canvas() {
		delete canvas;
	}
	
	// update
	void update(Object object) {
		
		if (engine.editor.isLoaded()) return;

		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 window_size = main_window.getClientSize();
		
		Player player = Unigine::getPlayer();
		mat4 projection = player.getProjection();
		Mat4 modelview = player.getIWorldTransform();
		
		projection.m00 *= float(window_size.y) / window_size.x;
		mat4 transform = scale(window_size.x / 2.0f,window_size.y / 2.0f,1.0f) * translate(1.0f,1.0f,0.0f) * scale(1.0f,-1.0f,1.0f) * projection * mat4(modelview * object.getWorldTransform());
		
		canvas.setLineTransform(line,transform);
		canvas.setPolygonTransform(polygon,transform);
		
		ivec2 mouse_coord = engine.input.getMousePosition() - main_window.getClientPosition();
		
		int line_id = canvas.getLineIntersection(mouse_coord.x, mouse_coord.y, 4.0f);
		if(line_id == line) canvas.setLineColor(line,vec4(1.0f,0.0f,0.0f,1.0f));
		else canvas.setLineColor(line,vec4_one);
		
		int polygon_id = canvas.getPolygonIntersection(mouse_coord.x, mouse_coord.y);
		if(polygon_id == polygon && line_id == -1) canvas.setPolygonColor(polygon,vec4(1.0f,0.0f,0.0f,0.5f));
		else canvas.setPolygonColor(polygon,vec4_zero);
	}
	
	// save/restore state
	void __restore__() {
		__Canvas__();
	}
};

Object object;
Canvas canvas;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	object = addToEditor(Unigine::createBox(vec3(14.0f)));
	object.setMaterial(findMaterialByName("mesh_base"),"*");
	
	canvas = new Canvas();
	
	setDescription("WidgetCanvas");
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	object.setWorldTransform(Mat4(translate(0.0f,0.0f,6.0f) * rotateX(time * 8.0f) * rotateY(time * 12.0f) * rotateZ(time * 16.0f)));
	canvas.update(object);
	
	return 1;
}

```

## canvas_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Canvas {
	
	Gui gui;				// gui
	
	WidgetCanvas canvas;	// canvas
	
	int polygons[0];		// polygons
	vec3 offsets[0];
	
	// constructor/destructor
	Canvas() {
		
		srand(4);
		
		gui = engine.getGui();
		
		// create mesh
		Mesh mesh = new Mesh();
		forloop(int i = 0; 32) {
			mesh.addIcosahedronSurface("icosahedron",2.0f);
			mesh.addDodecahedronSurface("deodecahedron",2.0f);
		}
		
		// create canvas
		canvas = new WidgetCanvas(gui);
		forloop(int i = 0; mesh.getNumSurfaces()) {
			vec3 offset = rand(-vec3(8.0f),vec3(8.0f));
			mesh.setSurfaceTransform(translate(offset),i);
			mesh.remapCVertex(i);
			int polygon = canvas.addPolygon();
			canvas.setPolygonTwoSided(polygon,0);
			canvas.setPolygonTexture(polygon,fullPath("uniginescript_samples/widgets/textures/canvas_03.dds"));
			forloop(int j = 0; mesh.getNumTVertex(i)) {
				canvas.addPolygonPoint(polygon,mesh.getVertex(j,i));
				canvas.setPolygonTexCoord(polygon,mesh.getTexCoord0(j,i));
			}
			forloop(int j = 0; mesh.getNumTIndices(i)) {
				canvas.addPolygonIndex(polygon,mesh.getTIndex(j,i));
			}
			polygons.append(polygon);
			offsets.append(offset);
		}
		
		gui.addChild(canvas,GUI_ALIGN_OVERLAP | GUI_ALIGN_BACKGROUND);
		
		delete mesh;
	}
	~Canvas() {
		delete canvas;
	}
	
	// update
	void update(mat4 transform) {
		
		if (engine.editor.isLoaded()) return;
		
		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 window_size = main_window.getSize();
		
		Player player = Unigine::getPlayer();
		mat4 projection = player.getProjection();
		Mat4 modelview = player.getIWorldTransform();
		
		projection.m00 *= float(window_size.y) / window_size.x;
		transform = scale(window_size.x / 2.0f, window_size.y / 2.0f,1.0f) * translate(1.0f,1.0f,0.0f) * scale(1.0f,-1.0f,1.0f) * projection * mat4(modelview * transform);
		
		// polygon transformations
		foreach(int polygon; polygons) {
			canvas.setPolygonTransform(polygon,transform);
		}
		
		// polygon order
		int indices[0];
		float distance[0];
		forloop(int i = 0; polygons.size()) {
			int polygon = polygons[i];
			canvas.setPolygonColor(polygon,vec4_one);
			vec4 vertex = transform * vec4(offsets[i]);
			distance.append(-vertex.z / vertex.w);
			indices.append(polygon);
		}
		
		distance.sort(indices);
		forloop(int i = 0; polygons.size()) {
			canvas.setPolygonOrder(indices[i],i);
		}
		
		ivec2 mouse_coord = engine.input.getMousePosition() - main_window.getPosition();
		
		int polygon = canvas.getPolygonIntersection(mouse_coord.x, mouse_coord.y);
		if(polygon != -1) canvas.setPolygonColor(polygon,vec4(1.0f,0.0f,0.0f,1.0f));
	}
	
	// save/restore state
	void __restore__() {
		__Canvas__();
	}
};

Canvas canvas;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	canvas = new Canvas();
	
	setDescription("WidgetCanvas");
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	mat4 transform = translate(0.0f,0.0f,6.0f) * rotateX(time * 8.0f) * rotateY(time * 12.0f) * rotateZ(time * 16.0f);
	canvas.update(transform);
	
	return 1;
}

```

## capsule_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 3;
	int height = 16;
	vec3 space = vec3(1.01f,1.01f,1.0f);
	
	for(int i = 0; i < height; i += 2) {
		for(int j = -size; j <= size; j++) {
			createBodyCapsule(0.5f,6.0f,1.0f,0.5f,0.5f,get_material(i + j),translate(Vec3(0.0f,j,i + 0.5f) * space) * rotateY(90.0f));
			num++;
		}
	}
	
	for(int i = 1; i < height; i += 2) {
		for(int j = -size; j <= size; j++) {
			createBodyCapsule(0.5f,6.0f,1.0f,0.5f,0.5f,get_material(i + j),translate(Vec3(j,0.0f,i + 0.5f) * space) * rotateX(90.0f));
			num++;
		}
	}
	
	setDescription(format("%d ShapeCapsule",num));
	
	return 1;
}

```

## capsule_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create shapes
	ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/shapes/meshes/mesh_00.mesh")));
	mesh.setMaterial(findMaterialByName("shapes_ground"),"*");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setMaterial(findMaterialByName(get_material(0)),"boxes");
	mesh.setMaterial(findMaterialByName(get_material(1)),"prisms");
	mesh.setMaterial(findMaterialByName(get_material(2)),"pyramids");
	mesh.setMaterial(findMaterialByName(get_material(3)),"parallelepipeds");

	forloop(int i = 0; 5;)
		mesh.setCollision(1,i);
	
	int num = 0;
	
	int size = 8;
	vec3 space = vec3(2.25f,2.25f,1.0f);
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			createBodyCapsule(0.5f,0.5f,1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(i,j,8.0f) * space));
			num++;
		}
	}
	
	setDescription(format("%d ShapeCapsule ObjectMeshStatic",num));
	
	return 1;
}

```

## capsule_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/shapes/meshes/capsule_01.mesh")));
	mesh.setMaterial(findMaterialByName(get_material(0)),"*");
	forloop(int i = 0; mesh.getNumSurfaces()) {
		mesh.setPhysicsRestitution(1.0f,i);
		mesh.setCollision(1,i);
	}
	
	int size = 10;
	for(int i = -size; i <= size; i++) {
		BodyRigid body = createBodyCapsule(0.5f,1.0f,1.0f,0.5f,1.0f,get_material(i),translate(Vec3(i * 1.1f,0.0f,3.0f)) * rotateX(90.0f));
		if(i & 0x01) body.setLinearVelocity(vec3(0.0f,200.0f,0.0f));
		else body.setLinearVelocity(vec3(0.0f,-200.0f,0.0f));
	}
	
	setDescription("ShapeCapsule ObjectMeshStatic CCD");
	
	return 1;
}

```

## car_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Car car = NULL;
ControlsDummy controls;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
class Car {
	
	Object object;
	JointSuspension joints[4];
	
	float angle = 0.0f;
	float velocity = 0.0f;
	
	Car(Mat4 transform) {
		
		BodyRigid body = createBodyBox(vec3(4.0f,2.0f,1.0f),8.0f,1.0f,0.0f,get_material(0),transform);
		object = body.getObject();
		body.setFreezable(0);
		
		Body wheels[4];
		
		wheels[0] = createBodySphere(0.5f,64.0f,2.0f,0.0f,get_material(1),transform * translate(-1.5f, 1.5f,-0.5f) * rotateX(90.0f));
		wheels[1] = createBodySphere(0.5f,64.0f,2.0f,0.0f,get_material(1),transform * translate(-1.5f,-1.5f,-0.5f) * rotateX(90.0f));
		wheels[2] = createBodySphere(0.5f,64.0f,2.0f,0.0f,get_material(1),transform * translate( 1.8f, 1.5f,-0.5f) * rotateX(90.0f));
		wheels[3] = createBodySphere(0.5f,64.0f,2.0f,0.0f,get_material(1),transform * translate( 1.8f,-1.5f,-0.5f) * rotateX(90.0f));
		
		for(int i = 0; i < 4; i++) {
			
			joints[i] = class_remove(new JointSuspension(body,wheels[i],wheels[i].getTransform() * Vec3_zero,vec3(0.0f,0.0f,1.0f),rotation(transform) * vec3(0.0f,1.0f,0.0f)));
			
			joints[i].setLinearRestitution(0.1f);
			joints[i].setAngularRestitution(0.1f);
			joints[i].setLinearSpring(40.0f);
			joints[i].setLinearDamping(2.0f);
			joints[i].setLinearLimitFrom(-1.0f);
			joints[i].setLinearLimitTo(0.0f);
			joints[i].setNumIterations(8);
		}
	}
	
	void update() {
		
		float ifps = engine.game.getIFps();
		
		float torque = 0.0f;
		if(engine.controls.getState(CONTROLS_STATE_FORWARD) || engine.controls.getState(CONTROLS_STATE_TURN_UP)) {
			velocity = max(velocity,0);
			velocity += ifps * 15.0f;
			torque = 50.0f;
		} else if(engine.controls.getState(CONTROLS_STATE_BACKWARD) || engine.controls.getState(CONTROLS_STATE_TURN_DOWN)) {
			velocity = min(velocity,0);
			velocity -= ifps * 15.0f;
			torque = 50.0f;
		} else {
			velocity *= exp(-ifps);
		}
		velocity = clamp(velocity,-90.0f,90.0f);
		
		if(engine.controls.getState(CONTROLS_STATE_MOVE_LEFT) || engine.controls.getState(CONTROLS_STATE_TURN_LEFT)) {
			angle += ifps * 100.0f;
		} else if(engine.controls.getState(CONTROLS_STATE_MOVE_RIGHT) || engine.controls.getState(CONTROLS_STATE_TURN_RIGHT)) {
			angle -= ifps * 100.0f;
		} else {
			if(abs(angle) < 0.25f) angle = 0.0f;
			else angle -= sign(angle) * ifps * 45.0f;
		}
		angle = clamp(angle,-40.0f,40.0f);
		
		float base = 3.3f;
		float width = 3.0f;
		float angle_0 = angle;
		float angle_1 = angle;
		if(abs(angle) > EPSILON) {
			float radius = base / tan(angle * DEG2RAD);
			angle_0 = atan(base / (radius + width / 2.0f)) * RAD2DEG;
			angle_1 = atan(base / (radius - width / 2.0f)) * RAD2DEG;
		}
		
		joints[2].setAngularVelocity(velocity);
		joints[3].setAngularVelocity(velocity);
		
		joints[2].setAngularTorque(torque);
		joints[3].setAngularTorque(torque);
		
		joints[0].setAxis10(rotateZ(angle_0) * vec3(0.0f,1.0f,0.0f));
		joints[1].setAxis10(rotateZ(angle_1) * vec3(0.0f,1.0f,0.0f));
		
		if(engine.controls.getState(CONTROLS_STATE_USE)) {
			joints[0].setAngularDamping(20000.0f);
			joints[1].setAngularDamping(20000.0f);
			joints[2].setAngularDamping(20000.0f);
			joints[3].setAngularDamping(20000.0f);
			velocity = 0.0f;
		} else {
			joints[0].setAngularDamping(0.0f);
			joints[1].setAngularDamping(0.0f);
			joints[2].setAngularDamping(0.0f);
			joints[3].setAngularDamping(0.0f);
		}
	}
};

/*
 */
int update() {
	updateSamplePhysics();
	car.update();
	controls.setMouseDX(engine.controls.getMouseDX());
	controls.setMouseDY(engine.controls.getMouseDY());
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("JointSuspension");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	car = new Car(translate(Vec3(0.0f,0.0f,1.0f)));
	controls = new ControlsDummy();
	
	for(int i = 0; i < 9; i++) {
		createBodyHomuncle(1,vec3_zero,material_names,translate(Vec3(-50.0f - i * 2.5f,-5.0f,0.1f)));
		createBodyHomuncle(1,vec3_zero,material_names,translate(Vec3(-50.0f - i * 2.5f, 0.0f,0.1f)));
		createBodyHomuncle(1,vec3_zero,material_names,translate(Vec3(-50.0f - i * 2.5f, 5.0f,0.1f)));
	}
	
	createBodyBox(vec3(10.0f,20.0f,2.0f),0.0f,0.5f,0.0f,get_material(2),Mat4(translate(-40.0f,0.0f,0.0f) * rotateY( 20.0f)));
	createBodyBox(vec3(10.0f,20.0f,2.0f),0.0f,0.5f,0.0f,get_material(2),Mat4(translate(-80.0f,0.0f,0.0f) * rotateY(-20.0f)));
	
	PlayerPersecutor player = new PlayerPersecutor();
	player.setFixed(1);
	player.setTarget(car.object);
	player.setMinDistance(8.0f);
	player.setMaxDistance(12.0f);
	player.setPosition(Vec3(10.0f,0.0f,6.0f));
	player.setControls(controls);
	engine.game.setPlayer(player);
	
	return 1;
}

```

## car_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Car car = NULL;
ControlsDummy controls;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
class Car {
	
	Object object;
	JointWheel joints[4];
	
	float angle = 0.0f;
	float velocity = 0.0f;
	
	Car(Mat4 transform) {
		
		BodyRigid body = createBodyBox(vec3(4.0f,2.0f,1.0f),8.0f,1.0f,0.0f,get_material(0),transform);
		object = body.getObject();
		body.setFreezable(0);
		
		Body wheels[4];
		
		wheels[0] = createBodySphere(0.5f,64.0f,2.0f,0.0f,get_material(1),transform * translate(-1.5f, 1.5f,-0.5f) * rotateX(90.0f));
		wheels[1] = createBodySphere(0.5f,64.0f,2.0f,0.0f,get_material(1),transform * translate(-1.5f,-1.5f,-0.5f) * rotateX(90.0f));
		wheels[2] = createBodySphere(0.5f,64.0f,2.0f,0.0f,get_material(1),transform * translate( 1.8f, 1.5f,-0.5f) * rotateX(90.0f));
		wheels[3] = createBodySphere(0.5f,64.0f,2.0f,0.0f,get_material(1),transform * translate( 1.8f,-1.5f,-0.5f) * rotateX(90.0f));
		
		for(int i = 0; i < 4; i++) {
			
			joints[i] = class_remove(new JointWheel(body,wheels[i],wheels[i].getTransform() * Vec3_zero,vec3(0.0f,0.0f,1.0f),rotation(transform) * vec3(0.0f,1.0f,0.0f)));
			
			joints[i].setWheelThreshold(0.1f);
			joints[i].setLinearSpring(100.0f);
			joints[i].setLinearDamping(200.0f);
			joints[i].setWheelMass(64.0f);
			joints[i].setWheelRadius(0.5f);
			joints[i].setTangentFriction(4.0f);
			joints[i].setBinormalFriction(5.0f);
			joints[i].setLinearLimitFrom(-1.0f);
			joints[i].setLinearLimitTo(0.0f);
			joints[i].setNumIterations(8);
		}
	}
	
	void update() {
		
		float ifps = engine.game.getIFps();
		
		float torque = 0.0f;
		if(engine.controls.getState(CONTROLS_STATE_FORWARD) || engine.controls.getState(CONTROLS_STATE_TURN_UP)) {
			velocity = max(velocity,0);
			velocity += ifps * 15.0f;
			torque = 50.0f;
		} else if(engine.controls.getState(CONTROLS_STATE_BACKWARD) || engine.controls.getState(CONTROLS_STATE_TURN_DOWN)) {
			velocity = min(velocity,0);
			velocity -= ifps * 15.0f;
			torque = 50.0f;
		} else {
			velocity *= exp(-ifps);
		}
		velocity = clamp(velocity,-90.0f,90.0f);
		
		if(engine.controls.getState(CONTROLS_STATE_MOVE_LEFT) || engine.controls.getState(CONTROLS_STATE_TURN_LEFT)) {
			angle += ifps * 100.0f;
		} else if(engine.controls.getState(CONTROLS_STATE_MOVE_RIGHT) || engine.controls.getState(CONTROLS_STATE_TURN_RIGHT)) {
			angle -= ifps * 100.0f;
		} else {
			if(abs(angle) < 0.25f) angle = 0.0f;
			else angle -= sign(angle) * ifps * 45.0f;
		}
		angle = clamp(angle,-40.0f,40.0f);
		
		float base = 3.3f;
		float width = 3.0f;
		float angle_0 = angle;
		float angle_1 = angle;
		if(abs(angle) > EPSILON) {
			float radius = base / tan(angle * DEG2RAD);
			angle_0 = atan(base / (radius + width / 2.0f)) * RAD2DEG;
			angle_1 = atan(base / (radius - width / 2.0f)) * RAD2DEG;
		}
		
		joints[2].setAngularVelocity(velocity);
		joints[3].setAngularVelocity(velocity);
		
		joints[2].setAngularTorque(torque);
		joints[3].setAngularTorque(torque);
		
		joints[0].setAxis10(rotateZ(angle_0) * vec3(0.0f,1.0f,0.0f));
		joints[1].setAxis10(rotateZ(angle_1) * vec3(0.0f,1.0f,0.0f));
		
		if(engine.controls.getState(CONTROLS_STATE_USE)) {
			joints[0].setAngularDamping(20000.0f);
			joints[1].setAngularDamping(20000.0f);
			joints[2].setAngularDamping(20000.0f);
			joints[3].setAngularDamping(20000.0f);
			velocity = 0.0f;
		} else {
			joints[0].setAngularDamping(0.0f);
			joints[1].setAngularDamping(0.0f);
			joints[2].setAngularDamping(0.0f);
			joints[3].setAngularDamping(0.0f);
		}
	}
};

/*
 */
int update() {
	updateSamplePhysics();
	car.update();
	controls.setMouseDX(engine.controls.getMouseDX());
	controls.setMouseDY(engine.controls.getMouseDY());
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("JointWheel");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	car = new Car(translate(Vec3(0.0f,0.0f,1.0f)));
	controls = new ControlsDummy();
	
	for(int i = 0; i < 9; i++) {
		createBodyHomuncle(1,vec3_zero,material_names,translate(Vec3(-50.0f - i * 2.5f,-5.0f,0.1f)));
		createBodyHomuncle(1,vec3_zero,material_names,translate(Vec3(-50.0f - i * 2.5f, 0.0f,0.1f)));
		createBodyHomuncle(1,vec3_zero,material_names,translate(Vec3(-50.0f - i * 2.5f, 5.0f,0.1f)));
	}
	
	createBodyBox(vec3(10.0f,20.0f,2.0f),0.0f,0.5f,0.0f,get_material(2),Mat4(translate(-40.0f,0.0f,0.0f) * rotateY( 20.0f)));
	createBodyBox(vec3(10.0f,20.0f,2.0f),0.0f,0.5f,0.0f,get_material(2),Mat4(translate(-80.0f,0.0f,0.0f) * rotateY(-20.0f)));
	
	PlayerPersecutor player = new PlayerPersecutor();
	player.setFixed(1);
	player.setTarget(car.object);
	player.setMinDistance(8.0f);
	player.setMaxDistance(12.0f);
	player.setPosition(Vec3(10.0f,0.0f,6.0f));
	player.setControls(controls);
	engine.game.setPlayer(player);
	
	return 1;
}

```

## car_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Car cars[0];
WorldTransformPath transform;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
class Car {
	
	Object object;
	JointSuspension joints[4];
	
	float angle = 0.0f;
	
	Car(float velocity,Mat4 transform) {
		
		BodyRigid body = createBodyBox(vec3(5.0f,3.0f,1.0f),16.0f,1.0f,0.0f,get_material(0),transform);
		object = body.getObject();
		body.setFreezable(0);
		
		Body wheels[4];
		
		wheels[0] = createBodySphere(0.5f,32.0f,1.5f,0.0f,get_material(1),transform * translate(-1.5f, 1.2f,-0.5f) * rotateX(90.0f));
		wheels[1] = createBodySphere(0.5f,32.0f,1.5f,0.0f,get_material(1),transform * translate(-1.5f,-1.2f,-0.5f) * rotateX(90.0f));
		wheels[2] = createBodySphere(0.5f,32.0f,1.5f,0.0f,get_material(1),transform * translate( 1.8f, 1.2f,-0.5f) * rotateX(90.0f));
		wheels[3] = createBodySphere(0.5f,32.0f,1.5f,0.0f,get_material(1),transform * translate( 1.8f,-1.2f,-0.5f) * rotateX(90.0f));
		
		for(int i = 0; i < 4; i++) {
			
			joints[i] = class_remove(new JointSuspension(body,wheels[i],wheels[i].getTransform() * Vec3_zero,vec3(0.0f,0.0f,1.0f),rotation(transform) * vec3(0.0f,1.0f,0.0f)));
			
			joints[i].setLinearRestitution(0.3f);
			joints[i].setAngularRestitution(0.2f);
			joints[i].setLinearSpring(80.0f);
			joints[i].setLinearDamping(2.0f);
			joints[i].setLinearLimitFrom(-0.5f);
			joints[i].setLinearLimitTo(0.0f);
			joints[i].setNumIterations(4);
		}
		
		joints[0].setAngularVelocity(velocity);
		joints[1].setAngularVelocity(velocity);
		joints[2].setAngularVelocity(velocity);
		joints[3].setAngularVelocity(velocity);
		
		joints[0].setAngularTorque(60.0f);
		joints[1].setAngularTorque(60.0f);
		joints[2].setAngularTorque(60.0f);
		joints[3].setAngularTorque(60.0f);
	}
	
	void update() {
		
		Path path = transform.getPath();
		Vec3 p0 = object.getWorldTransform() * Vec3(0.0f,0.0f,0.0f);
		Vec3 p1 = object.getWorldTransform() * Vec3(-20.0f,0.0f,0.0f);
		Vec3 p2 = path.getPosition(path.getClosestTime(p1),1);
		
		vec3 p10 = normalize(vec3(p1 - p0).xy0);
		vec3 p20 = normalize(vec3(p2 - p0).xy0);
		
		p0.z += 1.0f;
		engine.visualizer.renderLine3D(p0,p0 + p20 * 2.0f,vec4_one);
		
		angle = Unigine::getAngle(p10,p20);
		angle = clamp(angle,-40.0f,40.0f);
		
		float base = 3.3f;
		float width = 3.0f;
		float angle_0 = angle;
		float angle_1 = angle;
		if(abs(angle) > EPSILON) {
			float radius = base / tan(angle * DEG2RAD);
			angle_0 = atan(base / (radius + width / 2.0f)) * RAD2DEG;
			angle_1 = atan(base / (radius - width / 2.0f)) * RAD2DEG;
		}
		
		joints[0].setAxis10(rotateZ(angle_0) * vec3(0.0f,1.0f,0.0f));
		joints[1].setAxis10(rotateZ(angle_1) * vec3(0.0f,1.0f,0.0f));
	}
};

/*
 */
int update() {
	updateSamplePhysics();
	
	transform.renderVisualizer();
	foreach(Car car; cars) {
		car.update();
	}
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("JointSuspension");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	transform = addToEditor(new WorldTransformPath(full_path("uniginescript_samples/physics/paths/car_03.path")));
	transform.setLoop(1);
	
	engine.visualizer.setEnabled(1);
	
	forloop(int x = -1; 2) {
		float velocity = engine.game.getRandom(50.0f,60.0f);
		cars.append(new Car(velocity,translate(Vec3(x * 16.0f,-8.0f,1.0f))));
		cars.append(new Car(velocity,translate(Vec3(x * 16.0f, 8.0f,1.0f))));
		cars.append(new Car(velocity,translate(Vec3(x * 16.0f, 0.0f,1.0f))));
	}
	
	cars.append(new Car(60.0f,translate(Vec3(32.0f,0.0f,1.0f))));
	
	PlayerPersecutor player = new PlayerPersecutor();
	player.setFixed(1);
	player.setTarget(cars[cars.size() - 1].object);
	player.setMinDistance(8.0f);
	player.setMaxDistance(12.0f);
	player.setPosition(Vec3(48.0f,0.0f,6.0f));
	engine.game.setPlayer(player);
	
	return 1;
}

```

## cloth_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string cloth_material_names[] = ( "physics_cloth_red", "physics_cloth_green", "physics_cloth_blue", "physics_cloth_orange", "physics_cloth_yellow" );

string get_cloth_material(int material) {
	return cloth_material_names[abs(material) % cloth_material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,30.0f));
	createPlaneWithBody();
	setDescription("BodyCloth rigidity");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	Body body = createBodyBox(vec3(1.0f,36.0f,1.0f),0.0f,0.5f,0.5f,get_material(1),translate(Vec3(0.0f,0.0f,20.0f)));
	
	forloop(int i = 0; 4) {
		
		float y = -13.5f + i * 9.0f;
		
		createBodySphere(2.0f,0.0f,0.5f,0.5f,get_material(1),translate(Vec3(0.0f,y,15.0f)));
		
		BodyCloth cloth = createBodyCloth(8.0f,8.0f,0.5f,50.0f,0.5f,0.5f,get_cloth_material(i),translate(Vec3(4.0f,y,20.0f)));
		cloth.setRigidity(i / 3.0f);
		
		Body b0 = createBodyBox(vec3(1.0f,2.0f,1.0f),1.0f,0.5f,0.5f,get_material(2),translate(Vec3(8.0f,y - 3.0f,20.0f)));
		Body b1 = createBodyBox(vec3(1.0f,2.0f,1.0f),1.0f,0.5f,0.5f,get_material(2),translate(Vec3(8.0f,y + 3.0f,20.0f)));
		
		class_remove(new JointParticles(body,cloth,Vec3(0.0f,y,20.0f),vec3(0.1f,9.0f,0.1f)));
		class_remove(new JointParticles(b0,cloth,Vec3(8.0f,y - 3.0f,20.0f),vec3(2.0f,2.0f,1.0f)));
		class_remove(new JointParticles(b1,cloth,Vec3(8.0f,y + 3.0f,20.0f),vec3(2.0f,2.0f,1.0f)));
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## cloth_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Body bodies[0];

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	float time = engine.game.getTime();
	float offset = 360.0f / bodies.size();
	
	forloop(int i = 0; bodies.size()) {
		bodies[i].setVelocityTransform(Mat4(rotateZ(time * 30.0f + offset * i) * translate(12.0f,0.0f,0.0f) * rotateZ(180.0f)));
		bodies[i].flushTransform();
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyRigid in PhysicalForce");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 4;
	float space = 4.0f;
	forloop(int j = -size; size + 1) {
		forloop(int i = -size; size + 1) {
			if(abs(i) + abs(j) < 3) continue;
			if(abs(i) + abs(j) > 5) continue;
			Node node = addToEditor(node_load(full_path("uniginescript_samples/physics/meshes/cloth_01.node")));
			node.setWorldTransform(translate(Vec3(j,i,0.0f) * space));
		}
	}
	
	int num = 5;
	float offset = 360.0f / num;
	
	forloop(int i = 0; num) {
		
		ObjectMeshSkinned mesh = addToEditor(node_load(full_path("uniginescript_samples/physics/meshes/ragdoll_00.node")));
		mesh.setWorldTransform(Mat4(rotateZ(offset * i) * translate(12.0f,0.0f,0.0f) * rotateZ(180.0f)));
		mesh.setLayerAnimationFilePath(0,full_path("uniginescript_samples/physics/meshes/ragdoll_00.anim"));
		mesh.setTime(2.0f * i);
		mesh.play();
		
		bodies.append(mesh.getBody());
	}
	
	return 1;
}

```

## cloth_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
 int counter = 0;
float step = 1.0f;
float time = step;

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string cloth_material_names[] = ( "physics_cloth_red", "physics_cloth_green", "physics_cloth_blue", "physics_cloth_orange", "physics_cloth_yellow" );

string get_cloth_material(int material) {
	return cloth_material_names[abs(material) % cloth_material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	if(counter > 40) return 1;
	
	time += engine.game.getIFps() * engine.physics.getScale();
	if(time >= step) {
		createBodySphere(1.0f,1.0f,0.5f,0.5f,get_material(counter),translate(Vec3(0.0f,0.0f,20.0f)));
		counter++;
		time -= step;
	}
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyCloth two-way interaction");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	ObjectDummy dummy = addToEditor(new ObjectDummy());
	dummy.setWorldTransform(translate(Vec3(0.0f,0.0f,10.0f)));
	BodyDummy body = class_remove(new BodyDummy(dummy));
	
	BodyCloth cloth = createBodyCloth(16.0f,16.0f,0.5f,300.0f,1.0f,0.5f,get_cloth_material(0),translate(Vec3(0.0f,0.0f,10.0f)));
	cloth.setRadius(0.2f);
	cloth.setRigidity(0.1f);
	cloth.setLinearRestitution(0.5f);
	cloth.setAngularRestitution(0.5f);
	
	class_remove(new JointParticles(body,cloth,Vec3( 8.0f, 8.0f,10.0f),vec3(4.0f,4.0f,1.0f)));
	class_remove(new JointParticles(body,cloth,Vec3(-8.0f, 8.0f,10.0f),vec3(4.0f,4.0f,1.0f)));
	class_remove(new JointParticles(body,cloth,Vec3( 8.0f,-8.0f,10.0f),vec3(4.0f,4.0f,1.0f)));
	class_remove(new JointParticles(body,cloth,Vec3(-8.0f,-8.0f,10.0f),vec3(4.0f,4.0f,1.0f)));
	
	return 1;
}

```

## cloth_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
 int counter = 0;
float step = 1.0f;
float time = step;

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string cloth_material_names[] = ( "physics_cloth_red", "physics_cloth_green", "physics_cloth_blue", "physics_cloth_orange", "physics_cloth_yellow" );

string get_cloth_material(int material) {
	return cloth_material_names[abs(material) % cloth_material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	if(counter > 20) return 1;
	
	time += engine.game.getIFps() * engine.physics.getScale();
	if(time >= step) {
		createBodySphere(0.75f,1.0f,0.5f,0.5f,get_material(counter),translate(Vec3(0.0f,3.75f,30.0f)));
		counter++;
		time -= step;
	}
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyCloth two-way interaction");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	ObjectDummy dummy = addToEditor(new ObjectDummy());
	dummy.setWorldTransform(translate(Vec3(0.0f,0.0f,8.0f)));
	BodyDummy body = class_remove(new BodyDummy(dummy));
	
	ObjectMeshDynamic mesh = addToEditor(new ObjectMeshDynamic(full_path("uniginescript_samples/physics/meshes/cloth_03.mesh")));
	mesh.setWorldTransform(Mat4(translate(0.0f,0.0f,8.0f) * rotateZ(-90.0f)));
	mesh.setMaterial(findMaterialByName(get_cloth_material(0)),"*");
	
	BodyCloth cloth = class_remove(new BodyCloth(mesh));
	cloth.setNumIterations(3);
	cloth.setMass(200.0f);
	cloth.setRadius(0.2f);
	cloth.setRigidity(0.2f);
	cloth.setFriction(2.0f);
	
	class_remove(new JointParticles(body,cloth,Vec3(0.0f, 4.0f,17.0f),vec3(4.0f,4.0f,1.0f)));
	class_remove(new JointParticles(body,cloth,Vec3(0.0f,-9.0f, 4.0f),vec3(4.0f,1.0f,4.0f)));
	
	return 1;
}

```

## cloth_04.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string cloth_material_names[] = ( "physics_cloth_red", "physics_cloth_green", "physics_cloth_blue", "physics_cloth_orange", "physics_cloth_yellow" );

string get_cloth_material(int material) {
	return cloth_material_names[abs(material) % cloth_material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(56.0f,0.0f,56.0f));
	createPlaneWithBody();
	setDescription("BodyCloth tearing");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	Body body = createBodyBox(vec3(1.0f,32.0f,1.0f),0.0f,0.5f,0.5f,get_material(1),translate(Vec3(0.0f,0.0f,48.0f)));
	
	BodyCloth cloth = createBodyCloth(32.0f,32.0f,0.75f,50.0f,0.5f,0.5f,get_cloth_material(0),Mat4(translate(0.0f,0.0f,32.0f) * rotateY(90.0f)));
	cloth.setNumIterations(2);
	cloth.setLinearThreshold(0.75f);
	cloth.setLinearRestitution(1.0f);
	cloth.setAngularRestitution(0.25f);
	
	Body b0 = createBodyBox(vec3(2.0f,4.0f,4.0f),1.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.0f,-14.0f,32.0f)));
	Body b1 = createBodyBox(vec3(2.0f,4.0f,4.0f),1.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.0f, 14.0f,32.0f)));
	
	class_remove(new JointParticles(body,cloth,Vec3(0.0f,0.0f,48.0f),vec3(0.1f,34.0f,0.1f)));
	class_remove(new JointParticles(b0,cloth,Vec3(0.0f,-14.0f,32.0f),vec3(4.0f,4.0f,4.0f)));
	class_remove(new JointParticles(b1,cloth,Vec3(0.0f, 14.0f,32.0f),vec3(4.0f,4.0f,4.0f)));
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## cloth_05.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
int counter = 0;
float step = 1.0f;
float time = step;

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string cloth_material_names[] = ( "physics_cloth_red", "physics_cloth_green", "physics_cloth_blue", "physics_cloth_orange", "physics_cloth_yellow" );

string get_cloth_material(int material) {
	return cloth_material_names[abs(material) % cloth_material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	if(counter > 100) return 1;
	
	time += engine.game.getIFps() * engine.physics.getScale();
	if(time >= step) {
		createBodyBox(vec3(1.5f),2.0f,0.5f,0.5f,get_material(counter),translate(Vec3(0.0f,-18.0f,20.0f)));
		counter++;
		time -= step;
	}
	return 1;
}

/*
 */
void create_body_cloth(float size,float radius,float width,float step,int num,Mat4 transform) {
	
	int size_x = 0;
	int size_y = 0;
	
	ObjectMeshDynamic mesh = addToEditor(new ObjectMeshDynamic());
	for(float x = -step * num; x < step * num; x += size) {
		for(float y = -width / 2.0f; y < width / 2.0f; y += size) {
			mesh.addVertex(transform * vec3(y,x,radius));
			mesh.addTexCoord(vec4(x,y,0.0f,0.0f));
			size_y++;
		}
		size_x++;
	}
	for(float x = 0.0f; x < PI * radius; x += size) {
		float s = sin(x / radius);
		float c = cos(x / radius);
		for(float y = -width / 2.0f; y < width / 2.0f; y += size) {
			mesh.addVertex(transform * vec3(y,step * num + radius * s,radius * c));
			mesh.addTexCoord(vec4(x,y,0.0f,0.0f));
			size_y++;
		}
		size_x++;
	}
	for(float x = step * num; x > -step * num; x -= size) {
		for(float y = -width / 2.0f; y < width / 2.0f; y += size) {
			mesh.addVertex(transform * vec3(y,x,-radius));
			mesh.addTexCoord(vec4(x,y,0.0f,0.0f));
			size_y++;
		}
		size_x++;
	}
	for(float x = 0.0f; x < PI * radius; x += size) {
		float s = sin(x / radius);
		float c = cos(x / radius);
		for(float y = -width / 2.0f; y < width / 2.0f; y += size) {
			mesh.addVertex(transform * vec3(y,-step * num - radius * s,-radius * c));
			mesh.addTexCoord(vec4(x,y,0.0f,0.0f));
			size_y++;
		}
		size_x++;
	}
	size_y /= size_x;
	
	forloop(int y = 1; size_y) {
		int index = y;
		mesh.addIndex(index);
		mesh.addIndex(index + size_x * size_y - size_y);
		mesh.addIndex(index + size_x * size_y - size_y - 1);
		mesh.addIndex(index);
		mesh.addIndex(index + size_x * size_y - size_y - 1);
		mesh.addIndex(index - 1);
	}
	
	forloop(int x = 1; size_x) {
		forloop(int y = 1; size_y) {
			int index = size_y * x + y;
			mesh.addIndex(index);
			mesh.addIndex(index - size_y);
			mesh.addIndex(index - size_y - 1);
			mesh.addIndex(index);
			mesh.addIndex(index - size_y - 1);
			mesh.addIndex(index - 1);
		}
	}
	
	mesh.updateBounds();
	mesh.updateTangents();
	
	mesh.setMaterial(findMaterialByName("physics_cloth_red"),"*");
	
	BodyCloth cloth = class_remove(new BodyCloth(mesh));
	cloth.setNumIterations(3);
	cloth.setMass(1000.0f);
	cloth.setFriction(4.0f);
}

/*
 */
void create_tracks(int num,Mat4 transform) {
	
	void create_joint(Body b0,Body b1) {
		JointHinge j = class_remove(new JointHinge(b0,b1));
		j.setWorldAnchor(b1.getTransform() * Vec3_zero);
		j.setWorldAxis(rotation(transform) * vec3(1.0f,0.0f,0.0f));
		j.setAngularVelocity(4.0f);
		j.setAngularTorque(2000.0f);
		j.setNumIterations(4);
	}
	
	float step = 2.0f;
	float radius = 1.9f;
	
	create_body_cloth(0.6f,radius,6.0f,step,num,transform);
	
	Body b00 = createBodyBox(vec3(0.5f,step * num * 2,6.0f),0.0f,0.0f,0.5f,get_material(1),transform * translate(-3.25f,0.0f, 0.00f));
	Body b01 = createBodyBox(vec3(7.0f,step * num * 2,0.5f),0.0f,0.0f,0.5f,get_material(1),transform * translate( 0.00f,0.0f,-3.25f));
	Body b02 = createBodyBox(vec3(0.5f,step * num * 2,6.0f),0.0f,0.0f,0.5f,get_material(1),transform * translate( 3.25f,0.0f, 0.00f));
	
	Body b10 = createBodyCylinder(radius,5.0f,1.0f,16.0f,0.5f,get_material(3),transform * translate(0.0f,-step * num * 1.0f,0.0f) * rotateY(90.0f));
	Body b11 = createBodyCylinder(radius,5.0f,1.0f,16.0f,0.5f,get_material(3),transform * translate(0.0f,-step * num * 0.5f,0.0f) * rotateY(90.0f));
	Body b12 = createBodyCylinder(radius,5.0f,1.0f,16.0f,0.5f,get_material(3),transform * translate(0.0f, step * num * 0.0f,0.0f) * rotateY(90.0f));
	Body b13 = createBodyCylinder(radius,5.0f,1.0f,16.0f,0.5f,get_material(3),transform * translate(0.0f, step * num * 0.5f,0.0f) * rotateY(90.0f));
	Body b14 = createBodyCylinder(radius,5.0f,1.0f,16.0f,0.5f,get_material(3),transform * translate(0.0f, step * num * 1.0f,0.0f) * rotateY(90.0f));
	
	create_joint(b01,b10);
	create_joint(b01,b11);
	create_joint(b01,b12);
	create_joint(b01,b13);
	create_joint(b01,b14);
	
	b00 = NULL;
	b02 = NULL;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,30.0f));
	createPlaneWithBody();
	setDescription("BodyCloth two-way interaction");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	create_tracks(10,Mat4(translate(0.0f,0.0f,12.0f) * rotateX(12.0f)));
	
	return 1;
}


```

## cloth_06.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Body bodies[0];

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string cloth_material_names[] = ( "physics_cloth_red", "physics_cloth_green", "physics_cloth_blue", "physics_cloth_orange", "physics_cloth_yellow" );

string get_cloth_material(int material) {
	return cloth_material_names[abs(material) % cloth_material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	float time = engine.game.getTime();
	float offset = 360.0f / bodies.size();
	
	forloop(int i = 0; bodies.size()) {
		ObjectMeshSkinned(node_cast(bodies[i].getObject())).setLayerFrame(0,time * 25.0f + i * 128.0f);
		bodies[i].setVelocityTransform(Mat4(rotateZ(time * 25.0f + offset * i) * translate(12.0f,0.0f,0.0f) * rotateZ(180.0f)));
		bodies[i].flushTransform();
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyCloth wearing");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int num = 5;
	float offset = 360.0f / num;
	
	string names[] = (
		"uniginescript_samples/physics/meshes/cloth_06_coat.node",
		"uniginescript_samples/physics/meshes/cloth_06_hair.node",
	);
	
	forloop(int i = 0; num) {
		
		ObjectMeshSkinned mesh = addToEditor(node_load(full_path(names[i % names.size()])));
		mesh.setWorldTransform(Mat4(rotateZ(offset * i) * translate(12.0f,0.0f,0.0f) * rotateZ(180.0f)));
		mesh.setLayerAnimationFilePath(0,full_path("uniginescript_samples/physics/meshes/ragdoll_00.anim"));
		
		for(int j = mesh.getNumChildren() - 1; j >= 0; j--) {
			Node node = mesh.getChild(j);
			if(node.getType() == NODE_OBJECT_MESH_DYNAMIC) {
				node.setWorldParent(NULL);
			}
		}
		
		bodies.append(mesh.getBody());
	}
	
	return 1;
}

```

## cloth_07.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Body bodies[0];

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string cloth_material_names[] = ( "physics_cloth_red", "physics_cloth_green", "physics_cloth_blue", "physics_cloth_orange", "physics_cloth_yellow" );

string get_cloth_material(int material) {
	return cloth_material_names[abs(material) % cloth_material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	float time = engine.game.getTime();
	float offset = 360.0f / bodies.size();
	
	forloop(int i = 0; bodies.size()) {
		ObjectMeshSkinned(node_cast(bodies[i].getObject())).setLayerFrame(0,time * 25.0f + i * 128.0f);
		bodies[i].setVelocityTransform(Mat4(rotateZ(time * 25.0f + offset * i) * translate(12.0f,0.0f,0.0f) * rotateZ(180.0f)));
		bodies[i].flushTransform();
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyCloth wearing");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int num = 5;
	float offset = 360.0f / num;
	
	forloop(int i = 0; num) {
		
		ObjectMeshSkinned mesh = addToEditor(node_load(full_path("uniginescript_samples/physics/meshes/cloth_07_coat.node")));
		mesh.setWorldTransform(Mat4(rotateZ(offset * i) * translate(12.0f,0.0f,0.0f) * rotateZ(180.0f)));
		mesh.setLayerAnimationFilePath(0,full_path("uniginescript_samples/physics/meshes/ragdoll_00.anim"));
		
		for(int j = mesh.getNumChildren() - 1; j >= 0; j--) {
			Node node = node_cast(mesh.getChild(j));
			if(node.getType() == NODE_OBJECT_MESH_DYNAMIC) {
				Object(node).setMaterial(findMaterialByName(get_cloth_material(i)),"*");
				node.setWorldParent(NULL);
			}
		}
		
		bodies.append(mesh.getBody());
	}
	
	return 1;
}

```

## cloth_08.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string cloth_material_names[] = ( "physics_cloth_red", "physics_cloth_green", "physics_cloth_blue", "physics_cloth_orange", "physics_cloth_yellow" );

string get_cloth_material(int material) {
	return cloth_material_names[abs(material) % cloth_material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyCloth rigidity");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 3;
	
	ObjectDummy dummy = addToEditor(new ObjectDummy());
	dummy.setWorldTransform(translate(Vec3(0.0f,0.0f,10.0f)));
	BodyDummy body = class_remove(new BodyDummy(dummy));
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			
			float x = i * 6.0f;
			float y = j * 6.0f;
			
			BodyCloth cloth = createBodyCloth(6.0f,4.0f,0.5f,50.0f,0.5f,0.5f,get_cloth_material(i ^ j),translate(Vec3(x,y,8.0f)));
			cloth.setLinearRestitution(1.0f);
			cloth.setAngularRestitution(0.4f);
			cloth.setRadius(0.2f);
			
			class_remove(new JointParticles(body,cloth,Vec3(x - 3.0f,y,8.0f),vec3(1.1f,5.0f,0.2f)));
		}
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## cluster_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 20;
	
	Mat4 transforms[0];
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				transforms.append(translate(Vec3(x,y,z + 21.0f) * 1.1f));
				num++;
			}
		}
	}
	
	ObjectMeshCluster cluster = addToEditor(new ObjectMeshCluster(fullPath("uniginescript_samples/common/meshes/box.mesh")));
	cluster.setMaterial(findMaterialByName(get_mesh_material(0)),"*");
	cluster.setSurfaceProperty("surface_base","*");
	cluster.createMeshes(transforms);
	
	setDescription(format("ObjectMeshCluster with %d instances",num));
	
	return 1;
}

```

## cluster_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 20;
	
	Mat4 transforms[0];
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				transforms.append(translate(Vec3(x,y,z + 21.0f) * 1.1f));
				num++;
			}
		}
	}
	
	ObjectMeshCluster cluster = addToEditor(new ObjectMeshCluster(fullPath("uniginescript_samples/stress/meshes/surfaces_00.mesh")));
	forloop(int i = 0; cluster.getNumSurfaces()) {
		cluster.setMaterial(findMaterialByName(get_mesh_material(i)),i);
		cluster.setMinVisibleDistance(i * 8.0f - 8.0f,i);
		cluster.setMaxVisibleDistance(i * 8.0f,i);
	}
	cluster.setSurfaceProperty("surface_base","*");
	cluster.createMeshes(transforms);
	
	setDescription(format("ObjectMeshCluster with %d instances",num));
	
	return 1;
}

```

## cluster_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 20;
	
	Mat4 transforms[0];
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				transforms.append(translate(Vec3(x,y,z + 21.0f) * 1.1f));
				num++;
			}
		}
	}
	
	ObjectMeshCluster cluster = addToEditor(new ObjectMeshCluster(fullPath("uniginescript_samples/stress/meshes/surfaces_00.mesh")));
	forloop(int i = 0; cluster.getNumSurfaces()) {
		cluster.setMaterial(findMaterialByName(get_mesh_material(i)),i);
		cluster.setMinVisibleDistance(i * 8.0f - 8.0f,i);
		cluster.setMaxVisibleDistance(i * 8.0f,i);
		cluster.setMinFadeDistance(2.0f,i);
		cluster.setMaxFadeDistance(2.0f,i);
	}
	cluster.setSurfaceProperty("surface_base","*");
	cluster.createMeshes(transforms);
	
	setDescription(format("ObjectMeshCluster with %d instances",num));
	
	return 1;
}

```

## cluster_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

#define NUM_CLUSTERS 4
int size = 60;

class AsyncCluster
{
	public:
	Mat4 transforms[0];
	ObjectMeshCluster cluster;
	ObjectMeshCluster cluster_async;
	Async async;
};
AsyncCluster clusters[NUM_CLUSTERS];

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
template async_transforms<NUM, OFFSET_X, OFFSET_Y> void async_transforms_ ## NUM(ObjectMeshCluster cluster_async, float transforms[], float time, int size) {
	
	Vec3 offset = Vec3(OFFSET_X - 0.5f, OFFSET_Y - 0.5f, 0.0f) * (size + 0.5f) * 2;
	
	int num = 0;
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			float rand = sin(frac(num * 0.333f) + x * y * (NUM + 1));
			
			Vec3 pos = (Vec3(x, y, sin(time * rand * 2.0f) + 1.5f) + offset) * 2.0f;
			transforms[num] = translate(pos) * rotateZ(time * 25 * rand);
			num++;
		}
	}
	
	cluster_async.createMeshes(transforms);
}

async_transforms<0,0,0>;
async_transforms<1,0,1>;
async_transforms<2,1,0>;
async_transforms<3,1,1>;

void update_thread() {
	
	while(1) {
		
		// wait async
		for(int i = 0; i < NUM_CLUSTERS; i++) {
			while(clusters[i].async.isRunning())
				wait;
		}
		
		for(int i = 0; i < NUM_CLUSTERS; i++) {
			AsyncCluster c = clusters[i];
			
			c.async.clearResult();
			c.cluster.swap(c.cluster_async);
			c.cluster.setEnabled(1);
			c.cluster_async.setEnabled(0);
			c.async.run("async_transforms_" + i, c.cluster_async, c.transforms.id(), engine.game.getTime(), size);
		}
		
		wait;
	}
}

int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	for(int i = 0; i < NUM_CLUSTERS; i++) {
		AsyncCluster c = new AsyncCluster();
		c.cluster = new ObjectMeshCluster(fullPath("uniginescript_samples/common/meshes/box.mesh"));
		c.cluster.setMaterial(findMaterialByName(get_mesh_material(0)),"*");
		c.cluster.setSurfaceProperty("surface_base","*");
		c.cluster_async = class_append(node_cast(c.cluster.clone()));
		c.async = new Async();
		int num = pow(size * 2 + 1, 2);
		c.transforms.resize(num);
		clusters[i] = c;
	}
	
	thread("update_thread");
	
	int num = pow(size * 2 + 1, 2) * NUM_CLUSTERS;
	setDescription(format("ObjectMeshCluster with %d dynamic instances",num));
	
	return 1;
}

/*
 */
void shutdown() {
	
	for(int i = 0; i < NUM_CLUSTERS; i++) {
		clusters[i].async.wait();
	}
	
	return 1;
}

```

## clutter_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	ObjectMeshClutter clutter = addToEditor(new ObjectMeshClutter(fullPath("uniginescript_samples/common/meshes/box.mesh")));
	clutter.setWorldTransform(translate(Vec3(-2048.0f,-2048.0f,5.0f)));
	clutter.setMaterial(findMaterialByName(get_mesh_material(0)),"*");
	clutter.setSurfaceProperty("surface_base","*");
	
	clutter.setVisibleDistance(200.0f);
	clutter.setFadeDistance(400.0f);
	
	clutter.setSizeX(4096.0f);
	clutter.setSizeY(4096.0f);
	clutter.setStep(64.0f);
	
	clutter.setDensity(0.75f);
	clutter.setMinScale(1.0f,0.5f);
	clutter.setMaxScale(1.0f,0.5f);
	clutter.setOffset(0.0f,2.0f);
	
	setDescription("ObjectMeshClutter");
	
	return 1;
}

```

## clutter_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	ObjectMeshClutter clutter = addToEditor(new ObjectMeshClutter(fullPath("uniginescript_samples/stress/meshes/surfaces_00.mesh")));
	clutter.setWorldTransform(translate(Vec3(-2048.0f,-2048.0f,5.0f)));
	forloop(int i = 0; clutter.getNumSurfaces()) {
		clutter.setMaterial(findMaterialByName(get_mesh_material(i)),i);
		clutter.setMinVisibleDistance(i * 64.0f - 64.0f,i);
		clutter.setMaxVisibleDistance(i * 64.0f,i);
	}
	clutter.setSurfaceProperty("surface_base","*");
	
	clutter.setVisibleDistance(200.0f);
	clutter.setFadeDistance(400.0f);
	
	clutter.setSizeX(4096.0f);
	clutter.setSizeY(4096.0f);
	clutter.setStep(64.0f);
	
	clutter.setDensity(0.75f);
	clutter.setMinScale(1.0f,0.5f);
	clutter.setMaxScale(1.0f,0.5f);
	clutter.setOffset(0.0f,2.0f);
	
	setDescription("ObjectMeshClutter");
	
	return 1;
}

```

## clutter_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	ObjectMeshClutter clutter = addToEditor(new ObjectMeshClutter(fullPath("uniginescript_samples/stress/meshes/surfaces_00.mesh")));
	clutter.setWorldTransform(translate(Vec3(-2048.0f,-2048.0f,5.0f)));
	forloop(int i = 0; clutter.getNumSurfaces()) {
		clutter.setMaterial(findMaterialByName(get_mesh_material(i)),i);
		clutter.setMinVisibleDistance(i * 64.0f - 64.0f,i);
		clutter.setMaxVisibleDistance(i * 64.0f,i);
		clutter.setMinFadeDistance(16.0f,i);
		clutter.setMaxFadeDistance(16.0f,i);
	}
	clutter.setSurfaceProperty("surface_base","*");
	
	clutter.setVisibleDistance(200.0f);
	clutter.setFadeDistance(400.0f);
	
	clutter.setSizeX(4096.0f);
	clutter.setSizeY(4096.0f);
	clutter.setStep(64.0f);
	
	clutter.setDensity(0.75f);
	clutter.setMinScale(1.0f,0.5f);
	clutter.setMaxScale(1.0f,0.5f);
	clutter.setOffset(0.0f,2.0f);
	
	setDescription("ObjectMeshClutter");
	
	return 1;
}

```

## clutter_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 256;
	
	ObjectMeshStatic ref = new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh"));
	ref.setSurfaceProperty("surface_base","*");
	ref.setMaxVisibleDistance(500.0f,0);
	ref.setImmovable(1);
	
	vec3 position;
	for(int y = -size; y <= size; y++) {
		position.y = y * 4.0f;
		for(int x = -size; x <= size; x++) {
			position.x = x * 4.0f;
			
			position.z = 4.0f;
			ObjectMeshStatic mesh = class_append(node_cast(node_clone(ref)));
			mesh.setWorldTransform(translate(Vec3(position)));
			mesh.setMaterial(findMaterialByName("stress_mesh_blue"),"*");
			
			position.z = 6.0f;
			mesh = class_append(node_cast(node_clone(ref)));
			mesh.setWorldTransform(translate(Vec3(position)));
			mesh.setMaterial(findMaterialByName("stress_mesh_red"),"*");
			
			num += 2;
		}
	}
	
	delete ref;
	
	setDescription(format("%d meshes in clutter spatial tree",num));
	
	return 1;
}

```

## color_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	setDescription("Gui color multiplier");
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	vec4 color;
	color.x = abs(sin(time * 0.25f));
	color.y = abs(cos(time * 0.75f));
	color.z = abs(sin(time * 0.5f));
	color.w = abs(cos(time * 1.0f));
	
	engine.gui.setColor(color);
	
	return 1;
}


/*
 */
int shutdown() {
	
	engine.gui.setColor(vec4_one);
	
	return 1;
}

```

## convex_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	Vec3 min = Vec3(-15.0f,-15.0f, 5.0f);
	Vec3 max = Vec3( 15.0f, 15.0f,25.0f);
	
	forloop(int i = 0; 100) {
		
		float size_0 = engine.game.getRandom(1.0f,3.0f);
		float size_1 = engine.game.getRandom(1.0f,3.0f);
		float height = engine.game.getRandom(1.0f,3.0f);
		int sides = engine.game.getRandom(3,6);
		
		createBodyPrism(size_0,size_1,height,sides,1.0f,0.5f,0.5f,get_material(i),translate(engine.game.getRandom(min,max)));
		num++;
	}
	
	forloop(int i = 0; 50) {
		
		float radius = engine.game.getRandom(1.0f,2.0f);
		
		createBodyIcosahedron(radius,1.0f,0.5f,0.5f,get_material(i),translate(engine.game.getRandom(min,max)));
		num++;
	}
	
	forloop(int i = 0; 50) {
		
		float radius = engine.game.getRandom(1.0f,2.0f);
		
		createBodyDodecahedron(radius,1.0f,0.5f,0.5f,get_material(i),translate(engine.game.getRandom(min,max)));
		num++;
	}
	
	setDescription(format("%d ShapeConvex",num));
	
	return 1;
}

```

## convex_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 10;
	float friction = 0.8f;
	
	float height = 2.0f * sin(PI / 3.0f);
	float radius = 1.0f / sin(PI / 3.0f);
	
	forloop(int j = 0; size) {
		forloop(int i = 0; size - j) {
			createBodyPrism(2.0f,2.0f,2.0f,3,1.0f,friction,0.5f,get_material(i ^ j * 2 + 1),translate(Vec3(0.0f,i * 2.0f + j - size,j * height + height - radius)) * rotateZ(90.0f) * rotateX(-90.0f));
			num++;
		}
		forloop(int i = 0; size - j - 1) {
			createBodyPrism(2.0f,2.0f,2.0f,3,1.0f,friction,0.5f,get_material(i ^ j * 2 + 2),translate(Vec3(0.0f,i * 2.0f + j - size + 1.0f,j * height + radius)) * rotateZ(90.0f) * rotateX(90.0f));
			num++;
		}
	}
	
	setDescription(format("%d ShapeConvex",num));
	
	return 1;
}

```

## convex_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 2;
	int height = 6;
	float space = vec3(3.0f,3.0f,2.0f);
	
	for(int k = 0; k < height; k++) {
		for(int j = -size; j <= size; j++) {
			for(int i = -size; i <= size; i++) {
				createBodyPrism(1.0f,1.0f,2.0f,8,1.0f,0.5f,0.5f,get_material((i ^ j * 2) + k),translate(Vec3(i,j,k + 0.5f) * space));
				num++;
			}
		}
	}
	
	setDescription(format("%d ShapeConvex",num));
	
	return 1;
}

```

## convex_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(float radius_0,float radius_1,int size,int center,Mat4 transform) {
	
	int num = 0;
	
	float size_0 = PI * radius_0 / size;
	float size_1 = PI * radius_1 / size;
	float height = radius_1 - radius_0;
	
	for(int i = 0, j = 0; i <= size; i++, j += 9) {
		float angle = 180.0f / size * i;
		if(center == 0 && i == size / 2) continue;
		Mat4 prism_transform = translate(0.0f,0.0f,(size_0 + size_1) * 0.25f) * rotateX(angle) * translate(0.0f,radius_0 + height * 0.5f,0.0f) * rotateX(90.0f) * rotateZ(45.0f);
		createBodyPrism(size_0,size_1,height,4,1.0f,4.0f,0.5f,get_material(i),transform * prism_transform);
		num++;
	}
	
	return num;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	num += create_body(8.0f,11.0f,18,1,Mat4(rotateZ(0.0f)));
	num += create_body(8.0f,11.0f,18,0,Mat4(rotateZ(90.0f)));
	
	num += create_body(15.0f,20.0f,20,1,Mat4(rotateZ(45.0f)));
	num += create_body(15.0f,20.0f,20,0,Mat4(rotateZ(-45.0f)));
	
	setDescription(format("%d ShapeConvex",num));
	
	return 1;
}

```

## convex_04.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_torus(float density,int material,Mat4 transform) {
	
	ObjectMeshStatic mesh = addToEditor(node_load(fullPath("uniginescript_samples/shapes/nodes/torus_00.node")));
	mesh.setWorldTransform(transform);
	mesh.setMaterial(findMaterialByName(get_material(material)),"*");
	
	BodyRigid body = body_cast(mesh.getBody());
	forloop(int i = 0; body.getNumShapes()) {
		Shape shape = body.getShape(i);
		shape.setMass(shape.getVolume() * density);
	}
	
	return body.getNumShapes();
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 10;
	
	if(0) {
		ObjectMeshStatic mesh = new ObjectMeshStatic(fullPath("uniginescript_samples/shapes/meshes/torus_00.mesh"));
		mesh.setSurfaceProperty("surface_base","*");
		BodyRigid body = class_remove(new BodyRigid(mesh));
		body.createShapes(3,0.03f,0.05f);
		engine.world.saveNode(fullPath("uniginescript_samples/shapes/nodes/torus_00.node"),mesh);
	}
	
	for(int i = -size; i <= size; i++) {
		float density = 1.0f;
		if(i == -size || i == size) density = 0.0f;
		if(i % 2) {
			num += create_torus(density,i + 0,translate(Vec3(0.0f,i * 1.4f,10.0f)));
			num += create_torus(density,i + 1,translate(Vec3(4.0f,i * 1.4f,10.0f)));
			num += create_torus(density,i + 2,translate(Vec3(8.0f,i * 1.4f,10.0f)));
		} else {
			num += create_torus(density,i + 0,translate(Vec3(0.0f,i * 1.4f,10.0f)) * rotateY(90.0f));
			num += create_torus(density,i + 1,translate(Vec3(4.0f,i * 1.4f,10.0f)) * rotateY(90.0f));
			num += create_torus(density,i + 2,translate(Vec3(8.0f,i * 1.4f,10.0f)) * rotateY(90.0f));
		}
	}
	
	setDescription(format("%d ShapeConvex",num));
	
	return 1;
}

```

## convex_05.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create shapes
	ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/shapes/meshes/mesh_00.mesh")));
	mesh.setMaterial(findMaterialByName("shapes_ground"),"*");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setMaterial(findMaterialByName(get_material(0)),"boxes");
	mesh.setMaterial(findMaterialByName(get_material(1)),"prisms");
	mesh.setMaterial(findMaterialByName(get_material(2)),"pyramids");
	mesh.setMaterial(findMaterialByName(get_material(3)),"parallelepipeds");
	
	forloop(int i = 0; 5;)
		mesh.setCollision(1,i);
	
	int num = 0;
	
	int size = 8;
	vec3 space = vec3(2.25f,2.25f,1.0f);
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			createBodyPrism(1.0f,1.0f,1.0f,3,1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(i,j,8.0f) * space));
			num++;
		}
	}
	
	setDescription(format("%d ShapeConvex ObjectMeshStatic",num));
	
	return 1;
}

```

## cylinder_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 16;
	vec3 space = vec3(1.0f,1.01f,1.0f);
	
	for(int j = 0; j < size; j++) {
		for(int i = 0; i < size - j; i++) {
			createBodyCylinder(0.5f,1.0f,1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(0.0f,i + j * 0.5f - size * 0.5f + 0.5f,j + 0.5f) * space));
			num++;
		}
	}
	
	setDescription(format("%d ShapeCylinder",num));
	
	return 1;
}

```

## cylinder_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 16;
	vec3 space = vec3(1.0f,1.2f,1.0f);
	
	for(int j = 0; j < size; j++) {
		for(int i = 0; i < size - j; i++) {
			createBodyCylinder(0.5f,1.0f,1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(0.0f,i + j * 0.5f - size * 0.5f + 0.5f,j + 0.5f) * space) * rotateX(90.0f));
			num++;
		}
	}
	
	setDescription(format("%d ShapeCylinder",num));
	
	return 1;
}

```

## cylinder_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 2;
	int height = 6;
	float space = vec3(3.0f,3.0f,2.0f);
	
	for(int k = 0; k < height; k++) {
		for(int j = -size; j <= size; j++) {
			for(int i = -size; i <= size; i++) {
				createBodyCylinder(1.5f,2.0f,1.0f,0.5f,0.5f,get_material((i ^ j * 2) + k),translate(Vec3(i,j,k + 0.5f) * space));
				num++;
			}
		}
	}
	
	setDescription(format("%d ShapeCylinder",num));
	
	return 1;
}

```

## cylinder_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 3;
	int height = 16;
	float space = 1.01f;
	
	for(int i = 0; i < height; i += 2) {
		for(int j = -size; j <= size; j++) {
			createBodyCylinder(0.5f,7.0f,1.0f,0.5f,0.5f,get_material(i + j),translate(Vec3(0.0f,j,i + 0.5f) * space) * rotateY(90.0f));
			num++;
		}
	}
	
	for(int i = 1; i < height; i += 2) {
		for(int j = -size; j <= size; j++) {
			createBodyCylinder(0.5f,7.0f,1.0f,0.5f,0.5f,get_material(i + j),translate(Vec3(j,0.0f,i + 0.5f) * space) * rotateX(90.0f));
			num++;
		}
	}
	
	setDescription(format("%d ShapeCylinder",num));
	
	return 1;
}

```

## cylinder_04.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create shapes
	ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/shapes/meshes/mesh_00.mesh")));
	mesh.setMaterial(findMaterialByName("shapes_ground"),"*");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setMaterial(findMaterialByName(get_material(0)),"boxes");
	mesh.setMaterial(findMaterialByName(get_material(1)),"prisms");
	mesh.setMaterial(findMaterialByName(get_material(2)),"pyramids");
	mesh.setMaterial(findMaterialByName(get_material(3)),"parallelepipeds");

	forloop(int i = 0; 5;)
		mesh.setCollision(1,i);
	
	int num = 0;
	
	int size = 8;
	vec3 space = vec3(2.25f,2.25f,1.0f);
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			createBodyCylinder(0.5f,1.0f,1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(i,j,8.0f) * space));
			num++;
		}
	}
	
	setDescription(format("%d ShapeCylinder ObjectMeshStatic",num));
	
	return 1;
}

```

## cylindrical_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
Joint joints[0];

/*
 */
int update() {
	updateSamplePhysics();
	
	foreach(Joint j; joints)
		j.renderVisualizer(vec4_one);
	
	return 1;
}

/*
 */
void create_joint_0(Body b0,Body b1) {
	JointCylindrical j = class_remove(new JointCylindrical(b0,b1));
	j.setWorldAxis(vec3(0.0f,0.0f,1.0f));
	j.setLinearRestitution(0.4f);
	j.setAngularRestitution(0.4f);
	j.setLinearSoftness(0.4f);
	j.setAngularSoftness(0.4f);
	j.setLinearDamping(4.0f);
	j.setAngularDamping(2.0f);
	j.setLinearLimitFrom(-1.25f);
	j.setLinearLimitTo(1.25f);
	j.setNumIterations(16);
	joints.append(j);
}

void create_joint_1(Body b0,Body b1) {
	JointCylindrical j = class_remove(new JointCylindrical(b0,b1));
	j.setWorldAxis(vec3(0.0f,0.0f,1.0f));
	j.setLinearRestitution(0.4f);
	j.setAngularRestitution(0.4f);
	j.setLinearSoftness(0.4f);
	j.setAngularSoftness(0.4f);
	j.setLinearDamping(8.0f);
	j.setAngularDamping(2.0f);
	j.setLinearLimitFrom(-1.25f);
	j.setLinearLimitTo(1.25f);
	j.setNumIterations(16);
	joints.append(j);
}

void create_joint_2(Body b0,Body b1) {
	JointCylindrical j = class_remove(new JointCylindrical(b0,b1));
	j.setWorldAxis(vec3(0.0f,0.0f,1.0f));
	j.setLinearRestitution(0.4f);
	j.setAngularRestitution(0.4f);
	j.setLinearSoftness(0.4f);
	j.setAngularSoftness(0.4f);
	j.setLinearDamping(16.0f);
	j.setAngularDamping(2.0f);
	j.setLinearLimitFrom(-1.25f);
	j.setLinearLimitTo(1.25f);
	j.setNumIterations(16);
	joints.append(j);
}

/*
 */
void create_body(string func,Mat4 transform) {
	
	float offset = 1.2f;
	
	Body b0 = createBodyBox(vec3(1.0f),0.0f,0.5f,0.5f,get_material(0),transform * translate(0.0f,0.0f,offset * 0.0f));
	Body b1 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 2.0f));
	Body b2 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 4.0f));
	Body b3 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 6.0f));
	
	call(func,b0,b1);
	call(func,b1,b2);
	call(func,b2,b3);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("JointCylindrical");
	
	create_body("create_joint_0",translate(Vec3(0.0f,-10.0f,10.0f)));
	createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.0f,-10.0f,20.0f)));
	
	create_body("create_joint_1",translate(Vec3(0.0f,0.0f,10.0f)));
	createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.0f,0.0f,20.0f)));
	
	create_body("create_joint_2",translate(Vec3(0.0f,10.0f,10.0f)));
	createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.0f,10.0f,20.0f)));
	
	engine.visualizer.setEnabled(1);
	
	return 1;
}

```

## cylindrical_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
Joint joints[0];

/*
 */
int update() {
	updateSamplePhysics();
	
	foreach(Joint j; joints)
		j.renderVisualizer(vec4_one);
	
	return 1;
}

/*
 */
void create_body(int size,Mat4 transform) {
	
	float offset = 1.0f;
	
	Body b0 = createBodyBox(vec3(1.0f),0.0f,0.5f,0.5f,get_material(0),transform);
	Body b1 = createBodyBox(vec3(1.0f),10.0f,0.5f,0.5f,get_material(1),transform * translate(vec3(0.0f,0.0f,1.0f)));
	
	JointHinge j = class_remove(new JointHinge(b0,b1));
	j.setWorldAnchor(transform * vec3_zero);
	j.setWorldAxis(vec3(1.0f,0.0f,0.0f));
	j.setLinearRestitution(0.2f);
	j.setAngularRestitution(0.2f);
	j.setLinearSoftness(0.0f);
	j.setAngularSoftness(0.0f);
	j.setNumIterations(32);
	j.setAngularVelocity(1.0f);
	j.setAngularTorque(10000.0f);
	
	b0 = b1;
	
	forloop(int i = 1; size + 1) {
		
		b1 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(vec3(0.01f * i,0.5f * i,1.0f + i * offset)));
		
		JointCylindrical j = class_remove(new JointCylindrical(b0,b1));
		j.setWorldAxis(vec3(0.0f,0.0f,1.0f));
		j.setLinearLimitFrom(-1.0f);
		j.setLinearLimitTo(0.0f);
		j.setLinearRestitution(0.2f);
		j.setAngularRestitution(0.2f);
		j.setLinearSoftness(0.2f);
		j.setAngularSoftness(0.2f);
		j.setNumIterations(32);
		joints.append(j);
		
		b0 = b1;
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("JointCylindrical");
	
	create_body(4,translate(Vec3(0.0f,-10.0f,10.0f)));
	create_body(4,translate(Vec3(0.0f,  0.0f,10.0f)));
	create_body(4,translate(Vec3(0.0f, 10.0f,10.0f)));
	
	engine.visualizer.setEnabled(1);
	
	return 1;
}

```

## deform_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

#include <core/systems/widgets/widget.h>
#include <core/systems/widgets/widget_window.h>
#include <core/systems/widgets/widget_icon.h>
#include <core/systems/widgets/widget_deform.h>

/*
 */
Unigine::Widgets::Window window;
Unigine::Widgets::Icon close_icon;
Unigine::Widgets::Deform deform;

using Unigine::Samples;

void on_close()
{
	Unigine::Widgets::removeChild(window);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.setShadowDistance(100.0f);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("Unigine::Widgets::Deform");
	
	using Unigine::Widgets;
	
	// window
	window = new Window("Unigine::Widgets::Deform");
	window.setWidth(min(getWidth() - 128,1280));
	window.setHeight(min(getHeight() - 128,768));
	window.setSizeable(1);
	window.setDragAreaPadding(3, 25, 3, 0);
	
	// deform
	deform = new Deform(512,384);
	window.addChild(deform,ALIGN_EXPAND);
	
	// close icon
	close_icon = new Icon();
	close_icon.setPosition(10, -24);
	close_icon.getEventClicked().connect(functionid(on_close));
	window.addChild(close_icon, GUI_ALIGN_OVERLAP | GUI_ALIGN_TOP | GUI_ALIGN_RIGHT);

	Image img = new Image("core/gui/window_close.png");
	close_icon.setImage(img);

	// window
	window.arrange();
	addChild(window,ALIGN_OVERLAP | ALIGN_CENTER);
	
	return 1;
}

/*
 */
int update() {
	
	deform.update();
	return 1;
}

```

## depth_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("Linear depth");
	
	engine.console.setInt("render_taa",1);
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	engine.render.addDebugMaterial(findMaterialByName("post_deferred_depth_decode"));
	
	return 1;
}

/*
 */
int update() {
	
	float zfar = 64.0f;
	
	Player player = engine.game.getPlayer();
	if(player != NULL) player.setZFar(zfar);
	
	return 1;
}

```

## dialogs_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

#include <core/systems/widgets/widget.h>
#include <core/systems/widgets/widget_vbox.h>
#include <core/systems/widgets/widget_window.h>
#include <core/systems/widgets/widget_icon.h>
#include <core/systems/widgets/widget_dialog_message.h>
#include <core/systems/widgets/widget_dialog_file.h>
#include <core/systems/widgets/widget_dialog_color.h>
#include <core/systems/widgets/widget_dialog_image.h>
#include <core/systems/widgets/widget_dialog_property.h>
#include <core/systems/widgets/widget_dialog_node.h>
#include <uniginescript_samples/interface/widget_dialog_material.h>

Unigine::Widgets::Window window;
Unigine::Widgets::Icon close_icon;

using Unigine::Samples;

void on_close()
{
	Unigine::Widgets::removeChild(window);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.setShadowDistance(100.0f);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("Unigine::Widgets::Dialogs");
	
	using Unigine::Widgets;
	
	// window
	window = new Window("Unigine::Widgets::Dialogs");
	window.setWidth(320);
	window.setHeight(240);
	window.setSizeable(1);
	window.setDragAreaPadding(3, 25, 3, 0);
	
	// close icon
	close_icon = new Icon();
	close_icon.setPosition(10, -24);
	close_icon.getEventClicked().connect(functionid(on_close));
	window.addChild(close_icon, GUI_ALIGN_OVERLAP | GUI_ALIGN_TOP | GUI_ALIGN_RIGHT);

	Image img = new Image("core/gui/window_close.png");
	close_icon.setImage(img);

	VBox vbox = new VBox(0,4);
	window.addChild(vbox,ALIGN_EXPAND);
	
	// message
	Button button = new Button("Message");
	button.getEventClicked().connect("Unigine::Widgets::dialogMessage", "DialogMessage", "Message");
	vbox.addChild(button,ALIGN_EXPAND);
	
	// file
	button = new Button("File");
	button.getEventClicked().connect("Unigine::Widgets::dialogFile", "DialogFile","", "./");
	vbox.addChild(button,ALIGN_EXPAND);
	
	// color
	button = new Button("Color");
	button.getEventClicked().connect("Unigine::Widgets::dialogColor","DialogColor",vec4_one, NULL);
	vbox.addChild(button,ALIGN_EXPAND);
	
	// image
	button = new Button("Image");
	button.getEventClicked().connect("Unigine::Widgets::dialogImage","DialogImage","uniginescript_samples/interface/images/sprite.png");
	vbox.addChild(button,ALIGN_EXPAND);
	
	// material
	button = new Button("Material");
	button.getEventClicked().connect("Unigine::Widgets::dialogMaterial","DialogMaterial","mesh_base");
	vbox.addChild(button,ALIGN_EXPAND);
	
	// property
	button = new Button("Property");
	button.getEventClicked().connect("Unigine::Widgets::dialogProperty","DialogProperty","surface_base");
	vbox.addChild(button,ALIGN_EXPAND);
	
	// node
	button = new Button("Node");
	NodeDummy node = addToEditor(new NodeDummy());
	button.getEventClicked().connect("Unigine::Widgets::dialogNode","DialogNode",node.getID());
	vbox.addChild(button,ALIGN_EXPAND);
	
	// window
	window.arrange();
	addChild(window,ALIGN_OVERLAP | ALIGN_CENTER);
	
	return 1;
}

```

## dummy_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
PlayerDummy dummy;

using Unigine::Samples;

/*
 */
string material_names[] = ( "players_red", "players_green", "players_blue", "players_orange", "players_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	dummy.setDirection(vec3(1.0f,sin(time * 2.0f) * 0.08f,-0.8f + cos(time) * 0.04f),vec3(0.0f,cos(time * 2.0f) * 0.1f,1.0f));
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlane();
	setDescription("Dummy");
	
	int size = 2;
	float space = 4.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/players/meshes/sphere_00.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,1.0f) * space));
			mesh.setMaterial(findMaterialByName(get_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	dummy = addToEditor(new PlayerDummy());
	dummy.setPosition(Vec3(-20.0f,0.0f,15.0f));
	dummy.setDirection(vec3(1.0f,0.0f,-0.8f));
	engine.game.setPlayer(dummy);
	
	return 1;
}

```

## dynamic_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 14;
	
	ObjectMeshDynamic reference = Unigine::createBox(vec3(1.0f));
	
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshDynamic mesh = node_cast(reference.clone());
				mesh.setWorldTransform(translate(Vec3(x,y,z + 9.0f) * 1.1f));
				mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y ^ z)),"*");
				mesh.setSurfaceProperty("surface_base","*");
				num++;
			}
		}
	}
	
	delete reference;
	
	setDescription(format("%d ObjectMeshDynamic",num));
	
	return 1;
}

```

## dynamic_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 10;
	
	ObjectMeshDynamic reference = Unigine::createSphere(0.5f,16,32);
	
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshDynamic mesh = node_cast(reference.clone());
				mesh.setWorldTransform(translate(Vec3(x,y,z + 6.0f) * 1.1f));
				mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y ^ z)),"*");
				mesh.setSurfaceProperty("surface_base","*");
				num++;
			}
		}
	}
	
	delete reference;
	
	setDescription(format("%d ObjectMeshDynamic",num));
	
	return 1;
}

```

## dynamic_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>
#include <uniginescript_samples/objects/dynamic_01.h>

/*
 */
Async async;
int size = 32;
float field_0[size * size * size];
float field_1[size * size * size];
int flags_0[size * size * size];
int flags_1[size * size * size];
ObjectMeshDynamic mesh;

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "objects_mesh_red", "objects_mesh_green", "objects_mesh_blue", "objects_mesh_orange", "objects_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
void update_thread() {
	
	while(1) {
		
		float time = engine.game.getTime();
		
		// wait async
		if(async == NULL) async = new Async();
		while(async != NULL && async.isRunning()) wait;
		if(async == NULL) continue;
		async.clearResult();
		
		// marching cubes
		marching_cubes(mesh,field_1,flags_1,size);
		
		// create field
		float angle = sin(time) + 3.0f;
		mat4 transform = rotateZ(time * 25.0f) * scale(vec3(5.0f / size)) * translate(vec3(-size / 2.0f));
		async.run(functionid(create_field),field_0.id(),flags_0.id(),size,transform,angle);
		
		// swap buffers
		field_1.swap(field_0);
		flags_1.swap(flags_0);
		
		mesh.flushVertex();
		mesh.flushIndices();
		
		wait;
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	mesh = addToEditor(new ObjectMeshDynamic(OBJECT_DYNAMIC_ALL));
	mesh.setWorldTransform(Mat4(scale(vec3(16.0f / size)) * translate(-size / 2.0f,-size / 2.0f,0.0f)));
	mesh.setMaterial(findMaterialByName(get_mesh_material(1)),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	setDescription(format("Async dynamic marching cubes on %dx%dx%d grid",size,size,size));
	
	thread("update_thread");
	
	return 1;
}

/*
 */
void shutdown() {
	
	if(async != NULL) async.wait();
	return 1;
}

```

## dynamic_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>
#include <uniginescript_samples/objects/dynamic_01.h>

/*
 */
Async async_0;
Async async_1;
int size = 32;
float field_0[size * size * size];
float field_1[size * size * size];
int flags_0[size * size * size];
int flags_1[size * size * size];
ObjectMeshDynamic mesh_0;
ObjectMeshDynamic mesh_1;

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "objects_mesh_red", "objects_mesh_green", "objects_mesh_blue", "objects_mesh_orange", "objects_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
void update_thread() {
	
	while(1) {
		
		float time = engine.game.getTime();
		
		// wait async
		if(async_1 == NULL) async_1 = new Async();
		while(async_1 != NULL && async_1.isRunning()) wait;
		if(async_1 == NULL) continue;
		async_1.clearResult();
		
		// copy mesh
		Mesh mesh = new Mesh();
		mesh_1.getMesh(mesh);
		mesh_0.setMesh(mesh);
		mesh_0.setMaterial(findMaterialByName(get_mesh_material(1)),"*");
		mesh_0.setSurfaceProperty("surface_base","*");
		delete mesh;
		
		// wait async
		if(async_0 == NULL) async_0 = new Async();
		while(async_0 != NULL && async_0.isRunning()) wait;
		if(async_0 == NULL) continue;
		async_0.clearResult();
		
		// swap buffers
		field_1.swap(field_0);
		flags_1.swap(flags_0);
		
		// create field
		float angle = sin(time) + 3.0f;
		mat4 transform = rotateZ(time * 25.0f) * scale(vec3(5.0f / size)) * translate(vec3(-size / 2.0f));
		async_0.run(functionid(create_field),field_0.id(),flags_0.id(),size,transform,angle);
		
		// create mesh
		async_1.run(functionid(marching_cubes),mesh_1,field_1.id(),flags_1.id(),size);
		
		wait;
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	mesh_0 = addToEditor(new ObjectMeshDynamic(OBJECT_DYNAMIC_ALL));
	mesh_0.setWorldTransform(Mat4(scale(vec3(16.0f / size)) * translate(-size / 2.0f,-size / 2.0f,0.0f)));
	
	mesh_1 = new ObjectMeshDynamic(1);
	mesh_1.setEnabled(0);
	
	setDescription(format("Async dynamic marching cubes on %dx%dx%d grid",size,size,size));
	
	thread("update_thread");
	
	return 1;
}

/*
 */
void shutdown() {
	
	if(async_0 != NULL) async_0.wait();
	if(async_1 != NULL) async_1.wait();
	return 1;
}

```

## dynamic_04.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectMeshDynamic mesh;

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "objects_mesh_red", "objects_mesh_green", "objects_mesh_blue", "objects_mesh_orange", "objects_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	mesh.setWorldTransform(Mat4(translate(0.0f,0.0f,12.0f) * rotateX(time * 8.0f) * rotateY(time * 12.0f) * rotateZ(time * 16.0f)));
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("ObjectMeshDynamic with multiple surfaces");
	
	mesh = addToEditor(new ObjectMeshDynamic());
	
	Unigine::createBox(mesh,vec3(5.0f),translate(-5.0f,0.0f,0.0f));
	mesh.addSurface("box_0");
	
	Unigine::createBox(mesh,vec3(5.0f),translate(5.0f,0.0f,0.0f));
	mesh.addSurface("box_1");
	
	Unigine::createBox(mesh,vec3(5.0f),translate(0.0f,-5.0f,0.0f));
	mesh.addSurface("box_2");
	
	Unigine::createBox(mesh,vec3(5.0f),translate(0.0f,5.0f,0.0f));
	mesh.addSurface("box_3");
	
	Unigine::createBox(mesh,vec3(5.0f),translate(0.0f,0.0f,-5.0f));
	mesh.addSurface("box_4");
	
	Unigine::createBox(mesh,vec3(5.0f),translate(0.0f,0.0f,5.0f));
	mesh.addSurface("box_5");
	
	mesh.setMaterial(findMaterialByName(get_mesh_material(0)),0);
	mesh.setMaterial(findMaterialByName(get_mesh_material(1)),1);
	mesh.setMaterial(findMaterialByName(get_mesh_material(2)),2);
	mesh.setMaterial(findMaterialByName(get_mesh_material(3)),3);
	mesh.setMaterial(findMaterialByName(get_mesh_material(4)),4);
	mesh.setMaterial(findMaterialByName(get_mesh_material(5)),5);
	mesh.setSurfaceProperty("surface_base","*");
	
	return 1;
}

```

## dynamic_05.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectDynamic dynamic;

using Unigine::Samples;

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	// substitute shader parameters
	dynamic.setParameterFloat("m_color",vec4(abs(sin(time)),abs(cos(time)),0.0f,1.0f),4);
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("Fake ObjectMeshStatic");
	
	{
		ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
		mesh.setWorldTransform(translate(Vec3(0.0f,4.0f,0.0f)));
		mesh.setMaterial(engine.materials.findManualMaterial("Unigine::mesh_base"),"*");
		mesh.setSurfaceProperty("surface_base","*");
	}
	
	dynamic = addToEditor(new ObjectDynamic());
	dynamic.setMaterialNodeType(NODE_OBJECT_MESH_STATIC);
	dynamic.setWorldTransform(translate(Vec3(0.0f,-4.0f,0.0f)));
	dynamic.setMaterial(engine.materials.findMaterialByPath("uniginescript_samples/objects/dynamic_05/dynamic_mesh_base.mat"),"*");
	dynamic.setSurfaceProperty("surface_base","*");
	
	// set vertex format
	dynamic.setVertexFormat((
		ivec3(	0,	OBJECT_DYNAMIC_TYPE_FLOAT,	3),
		ivec3(	12,	OBJECT_DYNAMIC_TYPE_FLOAT,	4),
		ivec3(	28,	OBJECT_DYNAMIC_TYPE_FLOAT,	4),
		ivec3(	44,	OBJECT_DYNAMIC_TYPE_FLOAT,	4),
	));
	
	Mesh mesh = new Mesh(fullPath("uniginescript_samples/common/meshes/statue.mesh"));
	
	// remap vertices
	mesh.remapCVertex(0);
	
	// copy vertices
	forloop(int i = 0; mesh.getNumVertex(0)) {
		vec3 vertex = mesh.getVertex(i,0);
		dynamic.addVertexFloat(0,vertex,3);
		if(mesh.getNumTexCoords0(0)) dynamic.setVertexFloat(1,vec4(mesh.getTexCoord0(i,0)),4);
		if(mesh.getNumTangents(0)) dynamic.setVertexFloat(2,mesh.getTangent(i,0),4);
		if(mesh.getNumColors(0)) dynamic.setVertexFloat(3,mesh.getColor(i,0),4);
	}
	
	// copy surfaces
	forloop(int i = 0; mesh.getNumIndices(0)) {
		dynamic.addIndex(mesh.getIndex(i,0));
	}
	
	// copy bounding box
	dynamic.setBoundBox(mesh.getBoundBox());
	
	delete mesh;
	
	return 1;
}

```

## expression_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "worlds_mesh_red", "worlds_mesh_green", "worlds_mesh_blue", "worlds_mesh_orange", "worlds_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
#ifdef USE_DOUBLE
	string program = "{
	Node node = getNode();
	node.setTransform(rotateX(cos(time * 3.0f) * 2.0f) * rotateY(sin(time * 2.0f) * 4.0f) * rotateZ(sin(time * 1.0f) * 8.0f) * translate(0.0,0.0,1.1));
}";
#else
	string program = "{
	Node node = getNode();
	node.setTransform(rotateX(cos(time * 3.0f) * 2.0f) * rotateY(sin(time * 2.0f) * 4.0f) * rotateZ(sin(time * 1.0f) * 8.0f) * translate(0.0f,0.0f,1.1f));
}";
#endif

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 16;
	
	Node parent = NULL;
	
	for(int i = 0; i < num; i++) {
		
		WorldExpression expression = addToEditor(new WorldExpression());
		if(parent != NULL) parent.addChild(expression);
		expression.setExpression(program);
		expression.setUpdateDistanceLimit(100.0f);
		expression.setIFps(0.02f + 0.02f * i / num);
		parent = expression;
		
		int size = i / 4;
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
				mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * 1.1f));
				mesh.setMaterial(findMaterialByName(get_mesh_material((x ^ y) + i)),"*");
				mesh.setSurfaceProperty("surface_base","*");
				expression.addChild(mesh);
			}
		}
	}
	
	setDescription("WorldExpression");
	
	return 1;
}

```

## expression_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( 
	"uniginescript_samples/worlds/common/worlds/worlds_mesh_red.mat", 
	"uniginescript_samples/worlds/common/worlds/worlds_mesh_green.mat", 
	"uniginescript_samples/worlds/common/worlds/worlds_mesh_blue.mat", 
	"uniginescript_samples/worlds/common/worlds/worlds_mesh_orange.mat", 
	"uniginescript_samples/worlds/common/worlds/worlds_mesh_yellow.mat" 
	);

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create sample
	ObjectWaterGlobal water = addToEditor(new ObjectWaterGlobal());
	water.setWavesMode(2);
	water.setBeaufort(11);

	water.setFetchSteepnessQuality(3);

	int size = 8;
	
	WorldExpression expression = addToEditor(new WorldExpression());
	expression.setExpression("{\n#include <uniginescript_samples/worlds/expression_01.h>\n}");
	expression.setUpdateDistanceLimit(1000.0f);
	water.addChild(expression);
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * 2.0f));
			mesh.setMaterial(engine.materials.findMaterialByPath(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			expression.addChild(mesh);
		}
	}
	
	setDescription("WorldExpression");
	
	return 1;
}

```

## expression_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Node nodes[];

/*
 */
void disable_node(Node node) {
	if(nodes.check(node)) {
		nodes[node].setEnabled(0);
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int size = 8;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			NodeReference node = addToEditor(new NodeReference(fullPath("uniginescript_samples/worlds/nodes/expression_02.node")));
			node.setWorldTransform(translate(Vec3(x,y,1.0f) * 2.0f));
			nodes.append(node.getReference(),node);
		}
	}
	
	setDescription("WorldExpression functions");
	
	return 1;
}

```

## expression_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "worlds_mesh_red", "worlds_mesh_green", "worlds_mesh_blue", "worlds_mesh_orange", "worlds_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
void world_enter_callback(Node node,WorldTrigger trigger) {
	WorldExpression expression = node_cast(trigger.getParent());
	expression.call("enter_callback",expression,node);
}

void world_leave_callback(Node node,WorldTrigger trigger) {
	WorldExpression expression = node_cast(trigger.getParent());
	expression.call("leave_callback",expression,node);
}

/*
 */
ObjectMeshStatic mesh;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
	mesh.setTriggerInteractionEnabled(1);
	mesh.setMaterial(findMaterialByName(get_mesh_material(0)),"*");
	
	int size = 8;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			NodeReference node = addToEditor(new NodeReference(fullPath("uniginescript_samples/worlds/nodes/expression_03.node")));
			node.setWorldTransform(translate(Vec3(x,y,1.0f) * 2.0f));
		}
	}
	
	setDescription("WorldExpression callbacks");
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	mesh.setWorldTransform(rotateZ(time * 64.0f) * translate(Vec3(sin(time * 2.0f) * 16.0f,0.0f,4.0f)));
	
	return 1;
}

```

## expression_04.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "worlds_mesh_red", "worlds_mesh_green", "worlds_mesh_blue", "worlds_mesh_orange", "worlds_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
ObjectMeshStatic mesh;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
	mesh.setTriggerInteractionEnabled(1);
	mesh.setMaterial(findMaterialByName(get_mesh_material(0)),"*");
	
	int size = 8;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			NodeReference node = addToEditor(new NodeReference(fullPath("uniginescript_samples/worlds/nodes/expression_04.node")));
			node.setWorldTransform(translate(Vec3(x,y,1.0f) * 2.0f));
		}
	}
	
	setDescription("WorldExpression direct callbacks");
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	mesh.setWorldTransform(rotateZ(time * 64.0f) * translate(Vec3(sin(time * 2.0f) * 16.0f,0.0f,4.0f)));
	
	return 1;
}

```

## expression_05.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "worlds_mesh_red", "worlds_mesh_green", "worlds_mesh_blue", "worlds_mesh_orange", "worlds_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	WorldExpression expression = addToEditor(new WorldExpression());
	expression.setExpression("{\n#include <uniginescript_samples/worlds/expression_05.h>\n}");
	
	NodeDummy dummy = addToEditor(new NodeDummy());
	expression.addWorldChild(dummy);
	
	forloop(int y = -1; 2) {
		forloop(int x = -2; 3) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
			mesh.setTransform(Mat4(translate(x * 2.0f,y * 2.0f,-8.0f)));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			dummy.addChild(mesh);
		}
	}
	
	setDescription("WorldExpression camera attachments");
	
	return 1;
}

```

## fixed_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;


/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Fixed physics FPS");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	engine.physics.setSyncEngineUpdateWithPhysics(1);
	
	int size = 8;
	float space = 2.02f;
	
	for(int j = 0; j < size; j++) {
		for(int i = 0; i < size - j; i++) {
			createBodyBox(vec3(2.0f),1,0.5f,0.5f,get_material(0),translate(Vec3(0.0f,i + j * 0.5f - size * 0.5f + 0.5f,j + 0.5f) * space));
		}
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## fixed_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(int size,Mat4 transform) {
	
	int num = 0;
	
	Body bodies[0];
	
	for(int k = -size; k <= size; k++) {
		for(int j = -size; j <= size; j++) {
			for(int i = -size; i <= size; i++) {
				bodies.append(createBodySphere(0.5f,1.0f,0.5f,0.5f,get_material(i ^ j * 2 ^ k * 4),transform * translate(vec3(i,j,k))));
			}
		}
	}
	
	void create_joint(Body b0,Body b1) {
		JointFixed j = class_remove(new JointFixed(b0,b1));
		j.setLinearRestitution(0.02f);
		j.setAngularRestitution(0.02f);
		j.setLinearSoftness(0.2f);
		j.setAngularSoftness(0.2f);
		j.setNumIterations(2);
		num++;
	}
	
	int size_2 = size * 2;
	
	forloop(int k = 0; size_2 + 1) {
		forloop(int j = 0; size_2 + 1) {
			forloop(int i = 0; size_2 + 1) {
				
				if(i < size_2) {
					Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 1];
					create_joint(b0,b1);
				}
				
				if(j < size_2) {
					Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 1) + i + 0];
					create_joint(b0,b1);
				}
				
				if(k < size_2) {
					Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 1) + (size_2 + 1) * (j + 0) + i + 0];
					create_joint(b0,b1);
				}
			}
		}
	}
	
	return num;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	num += create_body(2,translate(Vec3(0.0f,0.0f,9.0f)));
	
	num += create_body(2,translate(Vec3(0.0f,0.0f,15.0f)));
	
	num += create_body(2,translate(Vec3(0.0f,0.0f,21.0f)) * rotateY(asin(rsqrt(3.0f)) * RAD2DEG) * rotateX(45.0f));
	
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3(-3.5f,-3.5f,2.0f)));
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3( 3.5f,-3.5f,2.0f)));
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3(-3.5f, 3.5f,2.0f)));
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3( 3.5f, 3.5f,2.0f)));
	
	setDescription(format("%d JointFixed",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## fixed_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(int size,Mat4 transform) {
	
	int num = 0;
	
	Body bodies[0];
	
	for(int k = -size; k <= size; k++) {
		for(int j = -size; j <= size; j++) {
			for(int i = -size; i <= size; i++) {
				bodies.append(createBodySphere(0.5f,1.0f,0.5f,0.5f,get_material(i ^ j * 2 ^ k * 4),transform * translate(vec3(i,j,k))));
			}
		}
	}
	
	void create_joint(Body b0,Body b1) {
		JointFixed j = class_remove(new JointFixed(b0,b1));
		j.setMaxForce(400.0f);
		j.setMaxTorque(400.0f);
		j.setLinearRestitution(0.02f);
		j.setAngularRestitution(0.02f);
		j.setLinearSoftness(0.2f);
		j.setAngularSoftness(0.2f);
		j.setNumIterations(2);
		num++;
	}
	
	int size_2 = size * 2;
	
	forloop(int k = 0; size_2 + 1) {
		forloop(int j = 0; size_2 + 1) {
			forloop(int i = 0; size_2 + 1) {
				
				if(i < size_2) {
					Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 1];
					create_joint(b0,b1);
				}
				
				if(j < size_2) {
					Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 1) + i + 0];
					create_joint(b0,b1);
				}
				
				if(k < size_2) {
					Body b0 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 0) + (size_2 + 1) * (j + 0) + i + 0];
					Body b1 = bodies[(size_2 + 1) * (size_2 + 1) * (k + 1) + (size_2 + 1) * (j + 0) + i + 0];
					create_joint(b0,b1);
				}
			}
		}
	}
	
	return num;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	num += create_body(2,translate(Vec3(0.0f,0.0f,9.0f)));
	
	num += create_body(2,translate(Vec3(0.0f,0.0f,15.0f)));
	
	num += create_body(2,translate(Vec3(0.0f,0.0f,21.0f)) * rotateY(asin(rsqrt(3.0f)) * RAD2DEG) * rotateX(45.0f));
	
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3(-3.5f,-3.5f,2.0f)));
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3( 3.5f,-3.5f,2.0f)));
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3(-3.5f, 3.5f,2.0f)));
	createBodyBox(vec3(4.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3( 3.5f, 3.5f,2.0f)));
	
	setDescription(format("%d JointFixed with destruction",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## fixed_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(Mat4 transform) {
	
	int num = 0;
	
	BodyRigid body = createBodyBox(vec3(2.0f),1.0f,0.5f,0.5f,get_material(0),transform);
	
	void create_joint(Body b0,Body b1) {
		Joint j = class_remove(new JointFixed(b0,b1));
		j.setLinearRestitution(0.8f);
		j.setAngularRestitution(0.1f);
		j.setLinearSoftness(0.4f);
		j.setAngularSoftness(0.4f);
		j.setNumIterations(16);
		num++;
	}
	
	void create_tentacle(Vec3 direction) {
		
		Body b0 = createBodySphere(0.5f,1.0f,0.75f,0.5f,get_material(1),transform * translate(direction * 2.0f));
		Body b1 = createBodySphere(0.5f,1.0f,0.75f,0.5f,get_material(1),transform * translate(direction * 3.0f));
		Body b2 = createBodySphere(0.5f,1.0f,0.75f,0.5f,get_material(1),transform * translate(direction * 4.0f));
		
		create_joint(body,b0);
		create_joint(b0,b1);
		create_joint(b1,b2);
	}
	
	float offset = 0.5f;
	
	create_tentacle(Vec3( offset, offset, offset));
	create_tentacle(Vec3( offset, offset,-offset));
	create_tentacle(Vec3( offset,-offset, offset));
	create_tentacle(Vec3( offset,-offset,-offset));
	create_tentacle(Vec3(-offset, offset, offset));
	create_tentacle(Vec3(-offset, offset,-offset));
	create_tentacle(Vec3(-offset,-offset, offset));
	create_tentacle(Vec3(-offset,-offset,-offset));
	
	return num;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	num += create_body(translate(Vec3(0.0f,0.0f,3.0f)));
	num += create_body(translate(Vec3(0.0f,0.0f,8.0f)));
	num += create_body(translate(Vec3(0.0f,0.0f,13.0f)));
	
	num += create_body(translate(Vec3( 8.0f, 8.0f,3.0f)));
	num += create_body(translate(Vec3(-8.0f, 8.0f,3.0f)));
	num += create_body(translate(Vec3( 8.0f,-8.0f,3.0f)));
	num += create_body(translate(Vec3(-8.0f,-8.0f,3.0f)));
	
	setDescription(format("%d JointFixed",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## fixed_04.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(int width,int height,int depth,Mat4 transform) {
	
	int num = 0;
	
	BodyRigid bodies[0];
	
	forloop(int k = 0; depth) {
		for(int j = -height; j <= height; j++) {
			for(int i = -width; i <= width; i++) {
				if(i != -width) bodies.append(createBodyBox(vec3(1.75f,0.25f,0.25f),1.0f,0.5f,0.5f,get_material(abs(i) + abs(j) + abs(k)),transform * translate(vec3(i,j + 0.5f,k) * 2.0f)));
				else bodies.append(NULL);
				if(j != -height) bodies.append(createBodyBox(vec3(0.25f,1.75f,0.25f),1.0f,0.5f,0.5f,get_material(abs(i) + abs(j) + abs(k)),transform * translate(vec3(i + 0.5f,j,k) * 2.0f)));
				else bodies.append(NULL);
				if(k != depth - 1) bodies.append(createBodyBox(vec3(0.25f,0.25f,1.75f),1.0f,0.5f,0.5f,get_material(abs(i) + abs(j) + abs(k)),transform * translate(vec3(i + 0.5f,j + 0.5f,k + 0.5f) * 2.0f)));
				else bodies.append(NULL);
			}
		}
	}
	
	foreach(Body body; bodies) {
		if(body != NULL) body.setFrozen(1);
	}
	
	void create_joint(Body b0,Body b1,Vec3 offset) {
		if(b0 != NULL && b1 != NULL) {
			JointFixed j = class_remove(new JointFixed(b0,b1));
			j.setMaxForce(30.0f);
			j.setMaxTorque(30.0f);
			j.setWorldAnchor(j.getWorldAnchor() + rotation(transform) * offset);
			j.setLinearRestitution(0.05f);
			j.setAngularRestitution(0.05f);
			j.setLinearSoftness(0.8f);
			j.setAngularSoftness(0.8f);
			j.setNumIterations(2);
			num++;
		}
	}
	
	int width_2 = width * 2;
	int height_2 = height * 2;
	
	forloop(int k = 0; depth) {
		forloop(int j = 0; height_2 + 1) {
			forloop(int i = 0; width_2 + 1) {
				
				Body b0 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 0) + i + 0) * 3 + 0];
				Body b1 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 0) + i + 0) * 3 + 1];
				Body b2 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 0) + i + 0) * 3 + 2];
				create_joint(b0,b1,Vec3(0.5f,0.5f, 0.0f));
				create_joint(b0,b2,Vec3(0.5f,0.0f,-0.5f));
				create_joint(b1,b2,Vec3(0.0f,0.5f,-0.5f));
				
				if(i != width_2) {
					Body b00 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 0) + i + 0) * 3 + 1];
					Body b01 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 0) + i + 1) * 3 + 0];
					create_joint(b00,b01,Vec3(-0.5f,0.5f,0.0f));
					
					Body b10 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 0) + i + 0) * 3 + 0];
					Body b11 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 0) + i + 1) * 3 + 0];
					create_joint(b10,b11,Vec3(0.0f,0.0f,0.0f));
					
					if(k != 0) {
						Body b2 = bodies[((width_2 + 1) * (height_2 + 1) * (k - 1) + (width_2 + 1) * (j + 0) + i + 0) * 3 + 2];
						create_joint(b00,b2,Vec3(0.0f,0.5f,0.5f));
						create_joint(b01,b2,Vec3(-0.5f,0.0f,0.5f));
					}
				}
				
				if(j != height_2) {
					Body b00 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 0) + i + 0) * 3 + 0];
					Body b01 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 1) + i + 0) * 3 + 1];
					create_joint(b00,b01,Vec3(0.5f,-0.5f,0.0f));
					
					Body b10 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 0) + i + 0) * 3 + 1];
					Body b11 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 1) + i + 0) * 3 + 1];
					create_joint(b10,b11,Vec3(0.0f,0.0f,0.0f));
					
					if(k != 0) {
						Body b2 = bodies[((width_2 + 1) * (height_2 + 1) * (k - 1) + (width_2 + 1) * (j + 0) + i + 0) * 3 + 2];
						create_joint(b00,b2,Vec3(0.5f,0.0f,0.5f));
						create_joint(b01,b2,Vec3(0.0f,-0.5f,0.5f));
					}
				}
				
				if(i != width_2 && j != height_2) {
					Body b0 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 0) + i + 1) * 3 + 0];
					Body b1 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 1) + i + 0) * 3 + 1];
					create_joint(b0,b1,Vec3(-0.5f,-0.5f,0.0f));
					
					if(k != 0) {
						Body b2 = bodies[((width_2 + 1) * (height_2 + 1) * (k - 1) + (width_2 + 1) * (j + 0) + i + 0) * 3 + 2];
						create_joint(b0,b2,Vec3(-0.5f,0.0f,0.5f));
						create_joint(b1,b2,Vec3(0.0f,-0.5f,0.5f));
					}
				}
				
				if(k != depth - 1) {
					Body b0 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 0) + (width_2 + 1) * (j + 0) + i + 0) * 3 + 2];
					Body b1 = bodies[((width_2 + 1) * (height_2 + 1) * (k + 1) + (width_2 + 1) * (j + 0) + i + 0) * 3 + 2];
					create_joint(b0,b1,Vec3(0.0f,0.0f,0.0f));
				}
			}
		}
	}
	
	return num;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	num += create_body(2,9,2,translate(Vec3(-1.0f,-1.0f,5.13f)) * rotateY(90.0f));
	
	createBodyBox(vec3(4.0f,2.0f,2.0f),2.0f,0.5f,0.5f,get_material(0),translate(Vec3(0.0f,-10.0f,12.0f)));
	createBodyBox(vec3(4.0f,2.0f,2.0f),2.0f,0.5f,0.5f,get_material(0),translate(Vec3(0.0f,  0.0f,16.0f)));
	createBodyBox(vec3(4.0f,2.0f,2.0f),2.0f,0.5f,0.5f,get_material(0),translate(Vec3(0.0f, 10.0f,12.0f)));
	
	setDescription(format("%d JointFixed with destruction",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## fixed_05.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(int width,int height,Mat4 transform) {
	
	int num = 0;
	float offset = 1.1f;
	
	Body bodies[0];
	
	for(int j = -height; j <= height; j++) {
		for(int i = -width; i <= width; i++) {
			float density = 1.0f;
			if(j == height) density = 0.0f;
			bodies.append(createBodyBox(vec3(1.0f),density,0.5f,0.5f,get_material(i ^ j * 2),transform * translate(vec3(i,j,0.0f) * offset)));
		}
	}
	
	void create_joint(Body b0,Body b1) {
		JointFixed j = class_remove(new JointFixed(b0,b1));
		j.setMaxForce(1000.0f);
		j.setMaxTorque(1000.0f);
		j.setLinearRestitution(0.8f);
		j.setAngularRestitution(0.2f);
		j.setLinearSoftness(1.0f);
		j.setAngularSoftness(1.0f);
		j.setNumIterations(2);
		num++;
	}
	
	int width_2 = width * 2;
	int height_2 = height * 2;
	
	forloop(int j = 0; height_2 + 1) {
		forloop(int i = 0; width_2 + 1) {
			
			if(i < width_2) {
				Body b0 = bodies[(width_2 + 1) * (j + 0) + i + 0];
				Body b1 = bodies[(width_2 + 1) * (j + 0) + i + 1];
				create_joint(b0,b1);
			}
			
			if(j < height_2) {
				Body b0 = bodies[(width_2 + 1) * (j + 0) + i + 0];
				Body b1 = bodies[(width_2 + 1) * (j + 1) + i + 0];
				create_joint(b0,b1);
			}
		}
	}
	
	return num;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	num += create_body(16,8,translate(Vec3(0.0f,0.0f,11.0f)) * rotateY(90.0f) * rotateZ(90.0f));
	
	setDescription(format("%d JointFixed with destruction",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## flags_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Window {
	
	Gui gui;				// gui
	
	WidgetWindow window;	// window
	WidgetLabel label;		// label
	
	// constructor/destructor
	Window() {
		
		gui = engine.getGui();
		
		// window
		window = new WidgetWindow(gui,"Window");
		window.setWidth(256);
		window.setHeight(256);
		
		// label
		label = new WidgetLabel(gui,"Label");
		label.setFontSize(32);
		label.setFontOutline(1);
		window.addChild(label,GUI_ALIGN_CENTER);
		
		window.arrange();
		window.setSizeable(1);
	}
	~Window() {
		delete window;
		delete label;
	}
	
	// show the window
	void show(int x,int y) {
		gui.addChild(window,GUI_ALIGN_OVERLAP);
		window.setPosition(x,y);
	}
	
	// save/restore state
	void __save__(Stream stream) {
		stream.writeInt(window.getPositionX());
		stream.writeInt(window.getPositionY());
	}
	void __restore__(Stream stream) {
		__Window__();
		int x = stream.readInt();
		int y = stream.readInt();
		show(x,y);
	}
};

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	Window window_0 = new Window();
	Window window_1 = new Window();
	
	window_0.show(200,200);
	window_1.show(600,200);
	
	setDescription("Overlapped windows");
	
	return 1;
}

```

## flags_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Window {
	
	Gui gui;				// gui
	
	WidgetWindow window;	// window
	WidgetLabel label;		// label
	
	// constructor/destructor
	Window() {
		
		gui = engine.getGui();
		
		// window
		window = new WidgetWindow(gui,"Window");
		window.setWidth(256);
		window.setHeight(256);
		
		// label
		label = new WidgetLabel(gui,"Label");
		label.setFontSize(32);
		label.setFontOutline(1);
		window.addChild(label,GUI_ALIGN_CENTER);
		
		window.arrange();
		window.setSizeable(1);
	}
	~Window() {
		delete window;
		delete label;
	}
	
	// show the window
	void show(int x,int y) {
		gui.addChild(window,GUI_ALIGN_OVERLAP | GUI_ALIGN_FIXED);
		window.setPosition(x,y);
	}
	
	// save/restore state
	void __save__(Stream stream) {
		stream.writeInt(window.getPositionX());
		stream.writeInt(window.getPositionY());
	}
	void __restore__(Stream stream) {
		__Window__();
		int x = stream.readInt();
		int y = stream.readInt();
		show(x,y);
	}
};

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	Window window_0 = new Window();
	Window window_1 = new Window();
	
	window_0.show(200,200);
	window_1.show(600,200);
	
	setDescription("Fixed overlapped windows");
	
	return 1;
}

```

## font_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Window {
	
	Gui gui;				// gui
	
	WidgetWindow window;	// window
	WidgetLabel label;		// label
	
	// constructor/destructor
	Window() {
		
		gui = engine.getGui();
		
		// window
		window = new WidgetWindow(gui,"Window");
		
		string text = "Jack and Jill\n"
			"Went up the hill "
			"To fetch a pail of water. "
			"Jack fell down "
			"And broke his crown "
			"And Jill came tumbling after. "
			"Up Jack got "
			"And home did trot "
			"As fast as he could caper "
			"Went to bed "
			"And plastered his head "
			"With vinegar and brown paper.";
		
		// label
		label = new WidgetLabel(gui,text);
		window.addChild(label,GUI_ALIGN_EXPAND);
		label.setFontWrap(1);
		label.setWidth(120);
		
		window.arrange();
		window.setSizeable(1);
		gui.addChild(window,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
	}
	~Window() {
		delete window;
		delete label;
	}
	
	// save/restore state
	void __restore__() {
		__Window__();
	}
};

Window window;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	window = new Window();
	
	setDescription("Font wrap");
	
	return 1;
}

```

## font_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Window {
	
	Gui gui;				// gui
	
	WidgetWindow window;	// window
	WidgetLabel label;		// label
	
	// constructor/destructor
	Window() {
		
		gui = engine.getGui();
		
		// window
		window = new WidgetWindow(gui,"Window");
		
		string text = "Jack and Jill<br>"
			"Went up the hill "
			"To fetch a pail of water. "
			"Jack fell down "
			"And broke his crown "
			"And Jill came tumbling after. "
			"Up Jack got "
			"And home did trot "
			"As fast as he could caper "
			"Went to bed "
			"And plastered his head "
			"With vinegar and brown paper.";
		
		text = replace(text,"Jack","<image src=jack.png scale=%80 dy=2/>");
		text = replace(text,"Jill","<font face=console.ttf size=%110 color=#00ff00>Jill</font>");
		
		// label
		label = new WidgetLabel(gui,text);
		window.addChild(label,GUI_ALIGN_EXPAND);
		label.setFontWrap(1);
		label.setFontRich(1);
		label.setWidth(120);
		
		window.arrange();
		window.setSizeable(1);
		gui.addChild(window,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
	}
	~Window() {
		delete window;
		delete label;
	}
	
	// save/restore state
	void __restore__() {
		__Window__();
	}
};

Window window;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	window = new Window();
	
	setDescription("Font rich");
	
	return 1;
}

```

## font_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Window {
	
	Gui gui;				// gui
	
	WidgetWindow window;	// window
	WidgetLabel label;		// label
	
	// constructor/destructor
	Window() {
		
		gui = engine.getGui();
		
		// window
		window = new WidgetWindow(gui,"Window");
		
		string text = "Jack and Jill<br>"
			"Went up the hill "
			"To fetch a pail of water. "
			"Jack fell down "
			"And broke his crown "
			"And Jill came tumbling after. "
			"Up Jack got "
			"And home did trot "
			"As fast as he could caper "
			"Went to bed "
			"And plastered his head "
			"With vinegar and brown paper.";
		
		text = replace(text,"Jack","<image src=jack.png scale=%80 dy=-1/>");
		text = replace(text,"Jill","<font face=console.ttf size=%110 color=#00ff00>Jill</font>");
		
		// label
		label = new WidgetLabel(gui,text);
		window.addChild(label,GUI_ALIGN_EXPAND);
		label.setFontHSpacing(2);
		label.setFontVSpacing(4);
		label.setFontWrap(1);
		label.setFontRich(1);
		label.setWidth(120);
		
		window.arrange();
		window.setSizeable(1);
		gui.addChild(window,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
	}
	~Window() {
		delete window;
		delete label;
	}
	
	// save/restore state
	void __restore__() {
		__Window__();
	}
};

Window window;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	window = new Window();
	
	setDescription("Font rich with spacing");
	
	return 1;
}

```

## font_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Window {
	
	Gui gui;				// gui
	
	WidgetWindow window;	// window
	WidgetLabel label;		// label
	
	// constructor/destructor
	Window() {
		
		gui = engine.getGui();
		
		// window
		window = new WidgetWindow(gui,"Window");
		
		string text = "<p align=justify><table color=#888888 space=4><center>";
		forloop(int x = 0; 16) {
			text += "<tr>";
			forloop(int y = 0; 16) {
				text += format("<td>%x%x</td>",x,y);
			}
			text += "</tr>";
		}
		text += "</center></table></p>";
		
		// label
		label = new WidgetLabel(gui,text);
		window.addChild(label,GUI_ALIGN_EXPAND);
		label.setFontHSpacing(2);
		label.setFontVSpacing(4);
		label.setFontWrap(1);
		label.setFontRich(1);
		label.setWidth(120);
		
		window.arrange();
		window.setSizeable(1);
		gui.addChild(window,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
	}
	~Window() {
		delete window;
		delete label;
	}
	
	// save/restore state
	void __restore__() {
		__Window__();
	}
};

Window window;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	window = new Window();
	
	setDescription("Font rich table");
	
	return 1;
}

```

## font_04.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Label {
	
	WidgetLabel label;		// label
	
	// constructor/destructor
	Label() {
		
		Gui gui = engine.getGui();
		
		string text = "Jack and Jill\n"
			"Went up the hill\n"
			"To fetch a pail of water.\n"
			"Jack fell down\n"
			"And broke his crown\n"
			"And Jill came tumbling after.\n"
			"Up Jack got\n"
			"And home did trot\n"
			"As fast as he could caper\n"
			"Went to bed\n"
			"And plastered his head\n"
			"With vinegar and brown paper.";
		
		// label
		label = new WidgetLabel(gui,text);
		label.setFontOutline(1);
		label.setFontSize(9);
		
		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 window_size = main_window.getSize();
		
		int x = engine.game.getRandom(0,window_size.x - 128);
		int y = engine.game.getRandom(0,window_size.y - 128);
		label.setPosition(x,y);
		
		gui.addChild(label,GUI_ALIGN_OVERLAP);
	}
	~Label() {
		delete label;
	}
	
	// save/restore state
	void __restore__() {
		__Label__();
	}
};

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	Label labels[0];
	forloop(int i = 0; 128) {
		labels.append(new Label());
	}
	
	setDescription("Font ttf with outline");
	
	return 1;
}

```

## font_05.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Label {
	
	WidgetLabel label;		// label
	
	// constructor/destructor
	Label() {
		
		Gui gui = engine.getGui();
		
		string text = "Jack and Jill\n"
			"Went up the hill\n"
			"To fetch a pail of water.\n"
			"Jack fell down\n"
			"And broke his crown\n"
			"And Jill came tumbling after.\n"
			"Up Jack got\n"
			"And home did trot\n"
			"As fast as he could caper\n"
			"Went to bed\n"
			"And plastered his head\n"
			"With vinegar and brown paper.";
		
		// label
		label = new WidgetLabel(gui,text);
		label.setFont(fullPath("uniginescript_samples/widgets/fonts/font.png"));
		label.setFontVSpacing(-4);
		label.setFontOutline(1);
		label.setFontSize(12);
		
		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 window_size = main_window.getSize();
		
		int x = engine.game.getRandom(0, window_size.x - 128);
		int y = engine.game.getRandom(0, window_size.y - 128);
		label.setPosition(x,y);
		
		gui.addChild(label,GUI_ALIGN_OVERLAP);
	}
	~Label() {
		delete label;
	}
	
	// save/restore state
	void __restore__() {
		__Label__();
	}
};

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	Label labels[0];
	forloop(int i = 0; 128) {
		labels.append(new Label());
	}
	
	setDescription("Font texture with outline");
	
	return 1;
}

```

## force_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physicals_red", "physicals_green", "physicals_blue", "physicals_orange", "physicals_yellow" );

/*
 */
 int update() {
	updateSamplePhysics();
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("BodyRigid in PhysicalForce");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 2;
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			createBodyHomuncle(1,vec3(0.0f),material_names,translate(Vec3(i * 2.5f,j * 5.0f,0.0f)));
		}
	}
	
	PhysicalForce force = addToEditor(new PhysicalForce(1000.0f));
	force.setWorldTransform(translate(Vec3(0.0f,0.0f,15.0f)));
	force.setAttractor(-50.0f);
	force.setRotator(-2.0f);
	
	return 1;
}

```

## force_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physicals_red", "physicals_green", "physicals_blue", "physicals_orange", "physicals_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
 int update() {
	updateSamplePhysics();
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(60.0f,0.0f,40.0f));
	createDefaultPlane();
	setDescription("BodyRigid in PhysicalForce");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int num = 200;
	
	forloop(int i = 0; num) {
		Vec3 position = engine.game.getRandom(Vec3(-200.0f,-200.0f,10.0f),Vec3(200.0f,200.0f,20.0f));
		createBodyBox(engine.game.getRandom(vec3(1.0f),vec3(1.0f,1.0f,8.0f)),1.0f,0.5f,0.5f,get_material(i),translate(position));
	}
	
	PhysicalForce force = addToEditor(new PhysicalForce(1000.0f));
	force.setWorldTransform(translate(Vec3(0.0f,0.0f,20.0f)));
	force.setAttractor(-500.0f);
	force.setRotator(-100.0f);
	
	PhysicalWind wind = addToEditor(new PhysicalWind(vec3(1000.0f)));
	wind.setLinearDamping(2.0f);
	wind.setAngularDamping(2.0f);
	
	return 1;
}

```

## fracture_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
void contact_callback(Body body,int num) {
	
	Body b0 = body.getContactBody0(num);
	Body b1 = body.getContactBody1(num);
	
	string n0 = (b0 != NULL) ? b0.getName() : "";
	string n1 = (b1 != NULL) ? b1.getName() : "";
	if(n0 != "sphere" && n1 != "sphere") return;
	
	BodyFracture fracture = NULL;
	while(b0 != NULL && b0.getType() != BODY_FRACTURE) b0 = b0.getParent();
	while(b1 != NULL && b1.getType() != BODY_FRACTURE) b1 = b1.getParent();
	if(b0 != NULL) fracture = body_cast(b0);
	if(b1 != NULL) fracture = body_cast(b1);
	if(fracture == NULL) return;
	
	Vec3 point = body.getContactPoint(num);
	vec3 normal = body.getContactNormal(num);
	float impulse = body.getContactImpulse(num);
	if(impulse > 100.0f) fracture.createCrackPieces(point,normal,7,1,3.0f);
}

/*
 */
void create_body_fracture(Mat4 transform) {
	BodyFracture body = createBodyFracture(vec3(2.0f,32.0f,8.0f),0.2f,0.5f,0.5f,get_material(0),get_material(3),transform);
	body.getEventContactEnter().connect(functionid(contact_callback));
	body.setThreshold(0.2f);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(40.0f,0.0f,30.0f));
	createPlaneWithBody();
	setDescription("BodyFracture crack pieces");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 3;
	
	for(int i = 0; i < size; i++) {
		create_body_fracture(Mat4(translate(-17.0f,  0.0f,4.0f + 8.0f * i)));
		create_body_fracture(Mat4(translate(  0.0f, 17.0f,4.0f + 8.0f * i) * rotateZ(90.0f)));
		create_body_fracture(Mat4(translate(  0.0f,-17.0f,4.0f + 8.0f * i) * rotateZ(90.0f)));
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## fracture_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
class Fracture {
	
	Body body;
	Object object;
	float time;
	
	Fracture(Body b) {
		body = b;
		object = body.getObject();
		time = engine.game.getTime();
	}
};

Fracture fractures[0];

/*
 */
void contact_callback(BodyFracture fracture, Body body,int num) {
	
	Body b0 = body.getContactBody0(num);
	Body b1 = body.getContactBody1(num);
	
	string n0 = (b0 != NULL) ? b0.getName() : "";
	string n1 = (b1 != NULL) ? b1.getName() : "";
	if(n0 != "sphere" && n1 != "sphere") return;
	
	float impulse = body.getContactImpulse(num);
	if(impulse > 10.0f && fracture.createShatterPieces(6)) {
		fractures.append(new Fracture(fracture));
	}
}

/*
 */
int update() {
	updateSamplePhysics();
	
	float time = engine.game.getTime();
	for(int i = 0; i < fractures.size(); i++) {
		float fade = 1.0f - saturate(((time - fractures[i].time) - 8.0f) / 4.0f);
		if(fade < EPSILON) {
			engine.world.removeNode(fractures[i].object);
			fractures.remove(i--);
		}
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyFracture shatter pieces");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int width = 8;
	int height = 4;
	float offset = 2.1f;
	
	Mat4 transform = translate(Vec3(0.0f,0.0f,10.0f));
	Body body = createBodyBox(vec3(1.0f,(width * 2 + 1) * offset,(height * 2 + 1) * offset),0.0f,0.5f,0.5f,get_material(3),transform);
	
	for(int j = -height; j <= height; j++) {
		for(int i = -width; i <= width; i++) {
			
			BodyFracture b = createBodyFracture(vec3(1.0f,2.0f,2.0f),1.0f,0.5f,0.5f,get_material(0),get_material(3),transform * translate(1.0f,i * offset,j * offset));
			b.getEventContactEnter().connect(functionid(contact_callback), b);
			b.setThreshold(0.05f);
			
			JointFixed j = class_remove(new JointFixed(body,b,b.getTransform() * Vec3_zero));
			j.setLinearRestitution(0.75f);
			j.setAngularRestitution(0.75f);
			j.setLinearSoftness(0.0f);
			j.setAngularSoftness(0.0f);
			j.setNumIterations(4);
		}
	}
	
	return 1;
}

```

## fracture_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
void contact_callback(Body body,int num) {
	
	Body b0 = body.getContactBody0(num);
	Body b1 = body.getContactBody1(num);
	
	string n0 = (b0 != NULL) ? b0.getName() : "";
	string n1 = (b1 != NULL) ? b1.getName() : "";
	if(n0 != "sphere" && n1 != "sphere") return;
	
	BodyFracture fracture = NULL;
	while(b0 != NULL && b0.getType() != BODY_FRACTURE) b0 = b0.getParent();
	while(b1 != NULL && b1.getType() != BODY_FRACTURE) b1 = b1.getParent();
	if(b0 != NULL) fracture = body_cast(b0);
	if(b1 != NULL) fracture = body_cast(b1);
	if(fracture == NULL) return;
	
	Vec3 point = body.getContactPoint(num);
	vec3 normal = body.getContactNormal(num);
	float impulse = body.getContactImpulse(num);
	if(impulse > 50.0f) fracture.createCrackPieces(point,normal,7,2,1.0f);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyFracture crack pieces");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 8;
	
	for(int i = 0; i < size; i++) {
		BodyFracture body = createBodyFracture(vec3(0.5f,8.0f,8.0f),1.0f,0.5f,0.5f,get_material(0),get_material(3),translate(Vec3(8.0f - 4.0f * i,0.0f,4.0f)));
		body.getEventContactEnter().connect(functionid(contact_callback));
		body.setThreshold(0.1f);
	}
	
	BodyRigid sphere = createBodySphere(0.25f,2000.0f,0.5f,0.5f,get_material(0),translate(Vec3(20.0f,0.0f,6.0f)));
	sphere.setLinearVelocity(vec3(-100.0f,0.0f,0.0f));
	sphere.setName("sphere");
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## fracture_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
void contact_callback(Body body,int num) {
	
	Body b0 = body.getContactBody0(num);
	Body b1 = body.getContactBody1(num);
	
	BodyFracture fracture = NULL;
	while(b0 != NULL && b0.getType() != BODY_FRACTURE) b0 = b0.getParent();
	while(b1 != NULL && b1.getType() != BODY_FRACTURE) b1 = b1.getParent();
	if(b0 != NULL) fracture = body_cast(b0);
	if(b1 != NULL) fracture = body_cast(b1);
	if(fracture == NULL) return;
	
	Vec3 point = body.getContactPoint(num);
	vec3 normal = body.getContactNormal(num);
	float impulse = body.getContactImpulse(num);
	if(impulse > 20.0f) fracture.createCrackPieces(point,normal,5,2,5.0f);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(60.0f,0.0f,50.0f));
	createPlaneWithBody();
	setDescription("BodyFracture crack pieces");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	
	int size = 3;
	
	for(int i = -size; i <= size; i++) {
		createSpringal(0.4f + abs(i) * 0.4f,material_names,translate(Vec3(20.0f,10.0f * i,0.0f)));
	}
	
	BodyFracture body = createBodyFracture(vec3(4.0f,100.0f,40.0f),0.2f,0.5f,0.5f,get_material(0),get_material(3),translate(Vec3(-60.0f,0.0f,20.0f)));
	body.getEventContactEnter().connect(functionid(contact_callback));
	body.setThreshold(32.0f);
	
	return 1;
}

/*
 */
void updatePhysics() {
	
	unlockSpringals();
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## fracture_04.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
BodyFracture bodies[0];

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update_thread() {
	
	for(int i = 0; i < bodies.size(); i++) {
		BodyFracture body = bodies[i];
		Vec3 point = body.getTransform() * engine.game.getRandom(-Vec3_one,Vec3_one);
		vec3 normal = engine.game.getRandom(-vec3_one,vec3_one);
		switch(i % 3) {
			case 0: body.createSlicePieces(point,normal); break;
			case 1: body.createCrackPieces(point,normal,5,0,2.0f); break;
			case 2: body.createShatterPieces(12); break;
		}
		sleep(0.2f);
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyFracture different pieces");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 2;
	
	ObjectMeshDynamic object = Unigine::createCapsule(2.0f,2.0f,8,8);
	object.setMaterial(findMaterialByName(get_material(0)),"*");
	object.setSurfaceProperty("surface_base", "*");
	
	for(int i = -size; i <= size; i++) {
		for(int j = -size; j <= size; j++) {
			
			ObjectMeshDynamic mesh = addToEditor(node_append(object.clone()));
			mesh.setWorldTransform(translate(Vec3(i * 8.0f,j * 8.0f,3.0f)));
			
			BodyFracture body = class_remove(new BodyFracture(mesh));
			body.setMaterial(findMaterialByName(get_material(3)));
			body.setError(0.05f);
			body.setThreshold(0.1f);
			body.setDensity(1.0f);
			
			bodies.append(body);
		}
	}
	
	delete object;
	
	thread("update_thread");
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## fracture_05.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
void contact_callback(Body body,int num) {
	
	Body b0 = body.getContactBody0(num);
	Body b1 = body.getContactBody1(num);
	
	BodyFracture fracture = NULL;
	while(b0 != NULL && b0.getType() != BODY_FRACTURE) b0 = b0.getParent();
	while(b1 != NULL && b1.getType() != BODY_FRACTURE) b1 = b1.getParent();
	if(b0 != NULL) fracture = body_cast(b0);
	if(b1 != NULL) fracture = body_cast(b1);
	if(fracture == NULL) return;
	
	Vec3 point = body.getContactPoint(num);
	vec3 normal = body.getContactNormal(num);
	float impulse = body.getContactImpulse(num);
	if(impulse > 40.0f) fracture.createCrackPieces(point,normal,5,1,2.0f);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyFracture crack pieces");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 2;
	
	for(int i = -size; i <= size; i++) {
		
		BodyFracture body = createBodyFracture(vec3(8.0f,8.0f,1.0f),0.2f,0.5f,0.5f,get_material(0),get_material(3),translate(Vec3(0.0f,10.0f * i,16.0f + 2.0f * i)));
		body.getEventContactEnter().connect(functionid(contact_callback));
		
		createBodySphere(1.0f,1.0f,0.5f,0.5f,get_material(1),translate(Vec3(0.0f,10.0f * i,1.0f)));
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## fracture_06.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
void contact_callback(Body body,int num) {
	
	Body b0 = body.getContactBody0(num);
	Body b1 = body.getContactBody1(num);
	
	BodyFracture fracture = NULL;
	while(b0 != NULL && b0.getType() != BODY_FRACTURE) b0 = b0.getParent();
	while(b1 != NULL && b1.getType() != BODY_FRACTURE) b1 = b1.getParent();
	if(b0 != NULL) fracture = body_cast(b0);
	if(b1 != NULL) fracture = body_cast(b1);
	if(fracture == NULL) return;
	
	Vec3 point = body.getContactPoint(num);
	vec3 normal = body.getContactNormal(num);
	float impulse = body.getContactImpulse(num);
	if(impulse > 20.0f) fracture.createCrackPieces(point,normal,5,2,2.0f);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyFracture crack pieces");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 5;
	
	for(int i = 0; i < size; i++) {
		
		BodyFracture body = createBodyFracture(vec3(12.0f,12.0f,1.0f),1.0f,0.5f,0.5f,get_material(0),get_material(3),translate(Vec3(0.0f,0.0f,2.5f + 3.0f * i)));
		body.getEventContactEnter().connect(functionid(contact_callback));
		
		createBodySphere(1.0f,2.0f,0.5f,0.5f,get_material(1),translate(Vec3( 5.0f, 5.0f,1.0f + 3.0f * i)));
		createBodySphere(1.0f,2.0f,0.5f,0.5f,get_material(1),translate(Vec3(-5.0f, 5.0f,1.0f + 3.0f * i)));
		createBodySphere(1.0f,2.0f,0.5f,0.5f,get_material(1),translate(Vec3( 5.0f,-5.0f,1.0f + 3.0f * i)));
		createBodySphere(1.0f,2.0f,0.5f,0.5f,get_material(1),translate(Vec3(-5.0f,-5.0f,1.0f + 3.0f * i)));
	}
	
	createBodySphere(1.0f,60.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.0f,0.0f,30.0f)));
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## friction_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Cone friction");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	forloop(int j = 0; 16) {
		
		float radius = 4.0f + j * 1.1f;
		float friction = 1.5f - j / 10.0f;
		
		forloop(int i = 0; 20) {
			
			float angle = i / 20.0f * PI2;
			
			BodyRigid body = createBodyBox(vec3(1.0f),1.0f,friction,0.0f,get_material(j),Mat4(translate(cos(angle) * radius,sin(angle) * radius,0.5f) * rotateZ(angle * RAD2DEG)));
			
			body.setLinearVelocity(vec3(cos(angle),sin(angle),0.0f) * 8.0f);
		}
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## geodetics_transformer_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

Node mercator_geodetic_node;
Node stereographic_geodetic_node;
ObjectText geodetic_coords_label;

int init() {
	
	createInterface(engine.world.getPath());
	
	setDescription("Transforming geodetic coordinates with GeodeticsTransformer"
#ifndef HAS_GEODETICS_TRANSFORMER
		"\n(Geodetics plugin is not loaded)"
#endif
	);
	
	mercator_geodetic_node = engine.world.getNodeByName("mercator_geodetic_node");
	stereographic_geodetic_node = engine.world.getNodeByName("stereographic_geodetic_node");
	geodetic_coords_label = node_cast(engine.world.getNodeByName("geodetic_coords_label"));
	
	return 1;
}

int update() {
#ifdef HAS_GEODETICS_TRANSFORMER
	float time = engine.game.getTime();
	dvec3 geo_coords = dvec3(-75.0  + cos(time) * 10.0, sin(time / 5.0 ) * 180.0, 0.0);
	
	dvec3 mercator_position = dvec3(0,0,0);
	
	{
		// EPSG:3395 WGS 84 / World Mercator 
		engine.geodeticsTransformer.setProjectionEpsg(3395, dvec3(0.0, 0.0, 0.0), "WGS84", 1);
		Mat4 transform = Mat4(engine.geodeticsTransformer.geodeticToWorld(geo_coords));
		
		mercator_position = dvec3(transform * Vec3(mercator_position));
		
		// scale and shift for demonstration puprpose
		double mercator_scale = 1000000.0;
		Mat4 scale_mat = Mat4(scale(mercator_scale, mercator_scale, mercator_scale));
		Mat4 invscale_mat = Mat4(scale(1.0f / mercator_scale, 1.0 / mercator_scale, 1.0 / mercator_scale));
		Mat4 shift = translate(-21.0f, 0.0, 0.0f);
		transform = shift * invscale_mat * transform * scale_mat;
		
		mercator_geodetic_node.setWorldTransform(transform);
		engine.visualizer.renderPoint3D(dvec3(transform * Vec3( dvec3(0,0,0))), 0.2, vec4(0,1,0,1), 0, 5, 0);
	}
	
	// inverse transform
	dvec3 geo_coords_recalc = engine.geodeticsTransformer.worldToGeodetic(mercator_position);
	geodetic_coords_label.setText(format("Lat %.1f Lon %.1f", geo_coords_recalc.x, geo_coords_recalc.y));
	
	{
		// EPSG:3031 WGS 84 / Antarctic Polar Stereographic
		engine.geodeticsTransformer.setProjectionEpsg(3031, dvec3(-90.0, 0.0, 0.0), "WGS84", 1);
		Mat4 transform = Mat4(engine.geodeticsTransformer.geodeticToWorld(geo_coords));
		
		// scale and shift for demonstration puprpose
		double stereographic_scale = 300000.0;
		Mat4 scale_mat = Mat4(scale(stereographic_scale, stereographic_scale, stereographic_scale));
		Mat4 invscale_mat = Mat4(scale(1.0f / stereographic_scale, 1.0 / stereographic_scale, 1.0 / stereographic_scale));
		Mat4 shift = translate(21.0f, 0.0, 0.0f);
		transform = shift * invscale_mat * transform * scale_mat;
		
		stereographic_geodetic_node.setWorldTransform(transform);

		engine.visualizer.renderPoint3D(dvec3(transform * Vec3( dvec3(0,0,0))), 0.2, vec4(1,0,0,1), 0, 5, 0);
	}
#endif
	
	return 1;
}

int shutdown() {

	return 1;
}

```

## gpu_monitor_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

Info info;

/*
 */
int init() {
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	#ifdef HAS_GPU_MONITOR
	setDescription("GPUMonitor plugin");
	#else
	setDescription("GPUMonitor plugin is not loaded");
	#endif
	
	info = new Info();
	return 1;
}

int update() {
	string str = "";
	#ifdef HAS_GPU_MONITOR
	str += format("Max temperature: %d ºC\n\n",engine.gpumonitor.getMaxTemperature());
	
	forloop(int i = 0; engine.gpumonitor.getNumMonitors()) {
		
		GPUMonitor gpumonitor = engine.gpumonitor.getMonitor(i);
		
		str += format("MonitorName:     %s\n",gpumonitor.getName());
		str += format("Max temperature: %d ºC\n\n",gpumonitor.getMaxTemperature());
		
		forloop(int j = 0; gpumonitor.getNumAdapters()) {
			
			float core = gpumonitor.getCoreClock(j);
			float memory = gpumonitor.getMemoryClock(j);
			float shader = gpumonitor.getShaderClock(j);
			float temperature = gpumonitor.getTemperature(j);
			
			str += format("AdapterName:     %s\n",gpumonitor.getAdapterName(j));
			if(core > 0.0f) str += format("Core clock:      %d MHz\n",core);
			if(memory > 0.0f) str += format("Memory clock:    %d MHz\n",memory);
			if(shader > 0.0f) str += format("Shader clock:    %d MHz\n",shader);
			if(temperature > -1000.0f) str += format("Temperature:     %d ºC\n\n",temperature);
		}
	}
	#endif
	info.set(str);
	return 1;
}

int shutdown() {
	delete info;
	return 1;
}

```

## graph_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

#include <core/systems/widgets/widget.h>
#include <core/systems/widgets/widget_vbox.h>
#include <core/systems/widgets/widget_hbox.h>
#include <core/systems/widgets/widget_scrollbox.h>
#include <core/systems/widgets/widget_window.h>
#include <core/systems/widgets/widget_icon.h>
#include <core/systems/widgets/widget_label.h>
#include <core/systems/widgets/widget_listbox.h>
#include <core/systems/widgets/widget_graph.h>

/*
 */
Unigine::Widgets::Graph graph;

/*
 */
Unigine::Widgets::Window window;
Unigine::Widgets::Icon close_icon;

Unigine::Widgets::ListBox types_lb;
Unigine::Widgets::ListBox nodes_lb;
Unigine::Widgets::ListBox joints_lb;

using Unigine::Samples;

void on_close()
{
	Unigine::Widgets::removeChild(window);
}

/*
 */
void update_nodes(Unigine::Widgets::GraphNode node) {
	
	using Unigine::Widgets;
	
	nodes_lb.clear();
	forloop(int i = 0; graph.getNumNodes()) {
		GraphNode n = graph.getNode(i);
		int id = nodes_lb.addItem(n.getName());
		if(node == n) nodes_lb.setCurrentItem(id);
	}
}

void update_joints(Unigine::Widgets::GraphNode joint) {
	
	using Unigine::Widgets;
	
	joints_lb.clear();
	forloop(int i = 0; graph.getNumJoints()) {
		GraphJoint j = graph.getJoint(i);
		GraphNode n0 = j.getNode0();
		GraphNode n1 = j.getNode1();
		int id = joints_lb.addItem(format("%s - %s",n0.getName(),n1.getName()));
		if(joint == j) joints_lb.setCurrentItem(id);
	}
}

/*
 */
Unigine::Widgets::GraphNode create_node(int x,int y) {
	
	using Unigine::Widgets;
	
	GraphNode node = NULL;
	
	int type = types_lb.getCurrentItem();
	
	if(type == 0) {
		node = new GraphNode("First","Data");
		node.addAnchor(new GraphAnchor(0xff | GRAPH_ENTRY,ALIGN_LEFT,GRAPH_SQUARE,"input","In"));
		node.addAnchor(new GraphAnchor(0xff,ALIGN_RIGHT,GRAPH_TRIANGLE,"output","Out"));
		node.setPosition(x,y);
	}
	
	else if(type == 1) {
		node = new GraphNode("Second","Data");
		node.addAnchor(new GraphAnchor(0xff | GRAPH_ENTRY,ALIGN_LEFT,GRAPH_SQUARE,"input","In"));
		node.addAnchor(new GraphAnchor(0xff,ALIGN_RIGHT,GRAPH_TRIANGLE,"output_0","Out 0"));
		node.addAnchor(new GraphAnchor(0xff,ALIGN_RIGHT,GRAPH_TRIANGLE,"output_1","Out 1"));
		node.setPosition(x,y);
	}
	
	else if(type == 2) {
		node = new GraphNode("Third","Data");
		node.addAnchor(new GraphAnchor(0xff | GRAPH_ENTRY,ALIGN_LEFT,GRAPH_SQUARE,"input_0","In 0"));
		node.addAnchor(new GraphAnchor(0xff | GRAPH_ENTRY,ALIGN_LEFT,GRAPH_SQUARE,"input_1","In 1"));
		node.addAnchor(new GraphAnchor(0xff,ALIGN_RIGHT,GRAPH_TRIANGLE,"output","Out"));
		node.setPosition(x,y);
	}
	
	else if(type == 3) {
		node = new GraphNode("Fourth","Data");
		node.addAnchor(new GraphAnchor(0xff | GRAPH_ENTRY,ALIGN_LEFT,GRAPH_SQUARE,"input_0","In 0"));
		node.addAnchor(new GraphAnchor(0xff | GRAPH_ENTRY,ALIGN_LEFT,GRAPH_SQUARE,"input_1","In 1"));
		node.addAnchor(new GraphAnchor(0xff,ALIGN_RIGHT,GRAPH_TRIANGLE,"output_0","Out 0"));
		node.addAnchor(new GraphAnchor(0xff,ALIGN_RIGHT,GRAPH_TRIANGLE,"output_1","Out 1"));
		node.setPosition(x,y);
	}
	
	else if(type == 4) {
		node = new GraphNode("Fifth","Data");
		node.addAnchor(new GraphAnchor(0xff | GRAPH_ENTRY,ALIGN_LEFT,GRAPH_SQUARE,"input_0","In 0"));
		node.addAnchor(new GraphAnchor(0xff | GRAPH_ENTRY,ALIGN_LEFT,GRAPH_SQUARE,"input_1","In 1"));
		node.addAnchor(new GraphAnchor(0xff | GRAPH_ENTRY,ALIGN_LEFT,GRAPH_SQUARE,"input_2","In 2"));
		node.addAnchor(new GraphAnchor(0xff,ALIGN_RIGHT,GRAPH_TRIANGLE,"output_0","Out 0"));
		node.addAnchor(new GraphAnchor(0xff,ALIGN_RIGHT,GRAPH_TRIANGLE,"output_1","Out 1"));
		node.addAnchor(new GraphAnchor(0xff,ALIGN_RIGHT,GRAPH_TRIANGLE,"output_2","Out 2"));
		node.setPosition(x,y);
	}
	
	else if(type == 5) {
		node = new GraphNode("Sixth","Data");
		node.addAnchor(new GraphAnchor(0xff | GRAPH_ENTRY,ALIGN_LEFT,GRAPH_SQUARE,"input_0","In 0"));
		node.addAnchor(new GraphAnchor(0xff | GRAPH_ENTRY,ALIGN_LEFT,GRAPH_SQUARE,"input_1","In 1"));
		node.addAnchor(new GraphAnchor(0xff | GRAPH_ENTRY,ALIGN_LEFT,GRAPH_SQUARE,"input_2","In 2"));
		node.addAnchor(new GraphAnchor(0xff | GRAPH_ENTRY,ALIGN_LEFT,GRAPH_SQUARE,"input_3","In 3"));
		node.addAnchor(new GraphAnchor(0xff,ALIGN_RIGHT,GRAPH_TRIANGLE,"output_0","Out 0"));
		node.addAnchor(new GraphAnchor(0xff,ALIGN_RIGHT,GRAPH_TRIANGLE,"output_1","Out 1"));
		node.addAnchor(new GraphAnchor(0xff,ALIGN_RIGHT,GRAPH_TRIANGLE,"output_2","Out 2"));
		node.addAnchor(new GraphAnchor(0xff,ALIGN_RIGHT,GRAPH_TRIANGLE,"output_3","Out 3"));
		node.setPosition(x,y);
	}
	
	else {
		log.error("create_node(): unknown node type %d\n",type);
	}
	
	return node;
}

/*
 */
void create_drag_drop() {
	
}

/*
 */
void node_changed(Unigine::Widgets::GraphNode node) {
	update_nodes(node);
}

void node_created(Unigine::Widgets::GraphNode node) {
	if(node == NULL) {
		float scale = graph.getScale();
		node = create_node(graph.getMouseX() / scale,graph.getMouseY() / scale);
		if(node == NULL) return;
	}
	graph.addNode(node);
	update_nodes(node);
}

void node_removed(Unigine::Widgets::GraphNode node) {
	update_nodes(NULL);
}

/*
 */
void joint_changed(Unigine::Widgets::GraphJoint joint) {
	update_joints(joint);
}

void joint_created(Unigine::Widgets::GraphJoint joint) {
	update_joints(joint);
}

void joint_removed(Unigine::Widgets::GraphJoint joint) {
	update_joints(NULL);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.setShadowDistance(100.0f);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("Unigine::Widgets::Graph");
	
	using Unigine::Widgets;
	
	// window
	window = new Window("Unigine::Widgets::Graph");
	window.setWidth(min(getWidth() - 128,1280));
	window.setHeight(min(getHeight() - 128,768));
	window.setSizeable(1);
	window.setDragAreaPadding(3, 25, 3, 0);
	
	// close icon
	close_icon = new Icon();
	close_icon.setPosition(10, -24);
	close_icon.getEventClicked().connect(functionid(on_close));
	window.addChild(close_icon, GUI_ALIGN_OVERLAP | GUI_ALIGN_TOP | GUI_ALIGN_RIGHT);

	Image img = new Image("core/gui/window_close.png");
	close_icon.setImage(img);

	// hbox
	HBox hbox = new HBox();
	window.addChild(hbox,ALIGN_EXPAND);
	
	{
		// hbox
		HBox panel = new HBox(4,0);
		panel.setWidth(128);
		hbox.addChild(panel);
		
		// vbox
		VBox vbox = new VBox(4,4);
		panel.addChild(vbox,ALIGN_EXPAND);
		
		// types listbox
		vbox.addChild(new Label("Types:"));
		
		ScrollBox scrollbox = new ScrollBox();
		scrollbox.setHScrollEnabled(0);
		vbox.addChild(scrollbox,ALIGN_EXPAND);
		
		types_lb = new ListBox();
		types_lb.getEventDragDrop().connect(functionid(create_drag_drop));
		types_lb.addItem("inputs 1 : outputs 1");
		types_lb.addItem("inputs 1 : outputs 2");
		types_lb.addItem("inputs 2 : outputs 1");
		types_lb.addItem("inputs 2 : outputs 2");
		types_lb.addItem("inputs 3 : outputs 3");
		types_lb.addItem("inputs 4 : outputs 4");
		scrollbox.addChild(types_lb,ALIGN_EXPAND);
		
		vbox.addChild(new VBox(4,4));
		
		// nodes listbox
		vbox.addChild(new Label("Nodes:"));
		
		scrollbox = new ScrollBox();
		scrollbox.setHScrollEnabled(0);
		vbox.addChild(scrollbox,ALIGN_EXPAND);
		
		nodes_lb = new ListBox();
		scrollbox.addChild(nodes_lb,ALIGN_EXPAND);
		
		vbox.addChild(new VBox(4,4));
		
		// joints listbox
		vbox.addChild(new Label("Joints:"));
		
		scrollbox = new ScrollBox();
		scrollbox.setHScrollEnabled(0);
		vbox.addChild(scrollbox,ALIGN_EXPAND);
		
		joints_lb = new ListBox();
		scrollbox.addChild(joints_lb,ALIGN_EXPAND);
	}
	
	{
		// scrollbox
		ScrollBox scrollbox = new ScrollBox();
		hbox.addChild(scrollbox,ALIGN_EXPAND);
		
		// graph
		graph = new Graph(8192,8192);
		graph.setCallback(GRAPH_NODE_CHANGED,"node_changed");
		graph.setCallback(GRAPH_NODE_CREATED,"node_created");
		graph.setCallback(GRAPH_NODE_REMOVED,"node_removed");
		graph.setCallback(GRAPH_JOINT_CHANGED,"joint_changed");
		graph.setCallback(GRAPH_JOINT_CREATED,"joint_created");
		graph.setCallback(GRAPH_JOINT_REMOVED,"joint_removed");
		scrollbox.addChild(graph,ALIGN_EXPAND);
		
		update_nodes(NULL);
		update_joints(NULL);
		
		window.arrange();
		scrollbox.setHScrollValue((scrollbox.getHScrollObjectSize() - scrollbox.getHScrollFrameSize()) / 2);
		scrollbox.setVScrollValue((scrollbox.getVScrollObjectSize() - scrollbox.getVScrollFrameSize()) / 2);
	}
	
	// window
	window.arrange();
	addChild(window,ALIGN_OVERLAP | ALIGN_CENTER);
	
	return 1;
}

```

## gui_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
WidgetLabel labels[0];
int frame = 0;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
void delete_object_gui(ObjectGui object_gui,WidgetLabel label,WidgetButton button) {
	int id = labels.find(label);
	if(id != -1) labels.remove(id);
	delete label;
	delete button;
	engine.world.removeNode(object_gui);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 10;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectGui object_gui = addToEditor(new ObjectGui(6.0f,4.0f));
			object_gui.setWorldTransform(Mat4(translate(vec3(x,y,1.0f) * 8.0f) * rotateY(45.0f) * rotateZ(90.0f)));
			object_gui.setMouseShow(0);
			object_gui.setControlDistance(100.0f);
			object_gui.setMaterial(findMaterialByName("gui_base"),"*");
			object_gui.setSurfaceProperty("surface_base","*");
			object_gui.setScreenSize(128,96);
			object_gui.setIntersection(1, 0);
			Gui gui = object_gui.getGui();
			WidgetLabel label = new WidgetLabel(gui);
			label.setFontSize(32);
			gui.addChild(label);
			labels.append(label);
			WidgetButton button = new WidgetButton(gui,"Delete");
			gui.addChild(button);
			button.getEventClicked().connect("delete_object_gui", object_gui, label, button);
			num++;
		}
	}
	
	setDescription(format("%d Object guis",num));
	
	return 1;
}

/*
 */
int update() {
	
	forloop(int i = 0; labels.size()) {
		if(labels[i] != NULL) labels[i].setText(format("%d\n%d",i,frame));
	}
	
	frame++;
	return 1;
}

```

## gui_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
WidgetSpriteVideo sprite_video;

using Unigine::Samples;

/*
 */
int update() {
	
	if(sprite_video != NULL && sprite_video.isPlaying() == 0) {
		sprite_video.setVideoTime(0.0f);
		sprite_video.play();
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("ObjectGuiMesh and WidgetSpriteVideo");
	
	ObjectGuiMesh object_gui_mesh = addToEditor(new ObjectGuiMesh(fullPath("uniginescript_samples/objects/meshes/gui_01.mesh")));
	object_gui_mesh.setMaterial(findMaterialByName("gui_base"),"*");
	object_gui_mesh.setSurfaceProperty("surface_base","*");
	
	Gui gui = object_gui_mesh.getGui();
	sprite_video = new WidgetSpriteVideo(gui,fullPath("uniginescript_samples/objects/videos/sanctuary.ogv"));
	gui.addChild(sprite_video,GUI_ALIGN_EXPAND);
	
	return 1;
}

```

## gui_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
WidgetSpriteViewport sprite_viewport;

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "objects_mesh_red", "objects_mesh_green", "objects_mesh_blue", "objects_mesh_orange", "objects_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	if(sprite_viewport != NULL) {
		sprite_viewport.setProjection(perspective(60.0f,2.0f,0.1f,1000.0f));
		sprite_viewport.setModelview(lookAt(Vec3(sin(time),cos(time),0.5f) * 16.0f,Vec3(0.0f,0.0f,8.0f),vec3(0.0f,0.0f,1.0f)));
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("ObjectGui and WidgetSpriteViewport");
	
	ObjectGui object_gui = addToEditor(new ObjectGui(40.0f,20.0f));
	object_gui.setWorldTransform(Mat4(translate(-16.0f,0.0f,12.0f) * rotateY(90.0f) * rotateZ(90.0f)));
	object_gui.setMaterial(findMaterialByName("gui_base"),"*");
	object_gui.setSurfaceProperty("surface_base","*");
	
	Gui gui = object_gui.getGui();
	sprite_viewport = new WidgetSpriteViewport(gui,512,512);
	gui.addChild(sprite_viewport,GUI_ALIGN_EXPAND);
	
	int size = 2;
	
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/objects/meshes/mesh_00.mesh")));
				mesh.setWorldTransform(translate(Vec3(x,y,z + 3.0f) * 1.5f));
				mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y ^ z)),"*");
				mesh.setSurfaceProperty("surface_base","*");
			}
		}
	}
	
	return 1;
}

```

## gui_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectGui object_gui;
WidgetLabel label;

using Unigine::Samples;

/*
 */
int update() {
	
	if(label != NULL) {
		string os_info = engine.system_info.getOSInfo() + "\n";
		string cpu_info = engine.system_info.getCPUInfo();
		string gpu_info = engine.system_info.getGPUDescription() + "\n";
		string fps = "FPS: " + string(engine.getFps());
		string text = os_info + cpu_info + gpu_info + fps;
		label.setText(text);
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("ObjectGui and WidgetLabel");
	
	object_gui = addToEditor(new ObjectGui(20.0f,15.0f));
	object_gui.setWorldTransform(Mat4(translate(0.0f,0.0f,8.0f) * rotateY(90.0f) * rotateZ(90.0f)));
	object_gui.setMaterial(findMaterialByName("gui_base"),"*");
	object_gui.setSurfaceProperty("surface_base","*");
	
	Gui gui = object_gui.getGui();
	label = new WidgetLabel(gui);
	label.setWidth(1024);
	label.setFontWrap(1);
	label.setFontSize(32);
	gui.addChild(label);
	
	return 1;
}

```

## gui_04.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "objects_mesh_red", "objects_mesh_green", "objects_mesh_blue", "objects_mesh_orange", "objects_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	int size = 8;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			
			ObjectGui object_gui = addToEditor(new ObjectGui(2.0f,2.0f));
			object_gui.setWorldTransform(translate(Vec3(x,y,1.5f) * 3.0f));
			object_gui.setBillboard(1);
			object_gui.setBackground(0);
			object_gui.setMaterial(findMaterialByName("objects_gui"),"*");
			object_gui.setSurfaceProperty("surface_base","*");
			object_gui.setScreenSize(96,96);
			Gui gui = object_gui.getGui();
			WidgetLabel label = new WidgetLabel(gui,format("%d",num));
			label.setFontSize(48);
			gui.addChild(label);
			
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/objects/meshes/mesh_00.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,1.0f) * 3.0f));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.addWorldChild(object_gui);
			
			num++;
		}
	}
	
	setDescription(format("%d ObjectGui",num));
	return 1;
}

```

## gui_05.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
WidgetWindow window;

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("ObjectGui with custom skin");
	
	ObjectGui object_gui = addToEditor(new ObjectGui(20.0f,15.0f,dirname(fullPath("uniginescript_samples/objects/interfaces/skin/gui.rc"))));
	object_gui.setWorldTransform(Mat4(translate(0.0f,0.0f,8.0f) * rotateY(90.0f) * rotateZ(90.0f)));
	object_gui.setScreenSize(640,480);
	object_gui.setControlDistance(1000.0f);
	object_gui.setMaterial(findMaterialByName("gui_base"),"*");
	object_gui.setSurfaceProperty("surface_base","*");
	object_gui.setIntersection(true, 0);
	
	Gui gui = object_gui.getGui();
	
	new UserInterface(gui,fullPath("uniginescript_samples/objects/interfaces/gui_07.ui"));
	if(window != NULL) gui.addChild(window,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
	
	return 1;
}

```

## gui_06.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
WidgetSpriteShader sprite;

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("ObjectGui with WidgetSpriteShader");
	
	ObjectGui object_gui = addToEditor(new ObjectGui(20.0f,15.0f));
	object_gui.setWorldTransform(Mat4(translate(0.0f,0.0f,8.0f) * rotateY(90.0f) * rotateZ(90.0f)));
	object_gui.setScreenSize(640,480);
	object_gui.setControlDistance(1000.0f);
	object_gui.setMaterial(findMaterialByName("gui_base"),"*");
	object_gui.setSurfaceProperty("surface_base","*");
	
	Gui gui = object_gui.getGui();
	
	WidgetSpriteShader sprite = new WidgetSpriteShader(gui,fullPath("uniginescript_samples/objects/textures/gui_08.png"));
	sprite.setMaterial(findMaterialByName("post_blur_radial"));
	sprite.setBlendFunc(GUI_BLEND_NONE,GUI_BLEND_NONE);
	sprite.setPosition(object_gui.getScreenWidth() / 2 - 128,object_gui.getScreenHeight() / 2 - 128);
	
	gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_BACKGROUND);
	
	return 1;
}

/*
 */
int shutdown() {
	delete sprite;
	return 1;
}

```

## height_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
FieldHeight heights[0];

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(60.0f,0.0f,20.0f));
	setDescription("FieldHeight and ObjectWaterGlobal");
	
	for(int i = 0; i < 14; i++) {
		FieldHeight height = addToEditor(new FieldHeight());
		height.setTexturePath(fullPath("core/textures/common/white.texture"));
		height.setSize(vec3(12.0f));
		height.setWorldTransform(Mat4(rotateZ(25.8f * i) * translate(40.0f,0.0f,0.0f)));
		height.setAttenuation(0.4f);
		heights.append(height);
	}
	
	return 1;
}

/*
 */
int shutdown() {

	return 1;
}

/*
 */
int update() {
	engine.visualizer.setEnabled(1);
	float ifps = engine.game.getIFps();
	foreach(FieldHeight height; heights) {
		height.setWorldTransform(Mat4(rotateZ(ifps * 16.0f)) * height.getWorldTransform());
		height.renderVisualizer();
	}
	
	return 1;
}

```

## hinge_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
Joint joints[0];

/*
 */
void create_joint_default(Body b0,Body b1) {
	JointHinge j = class_remove(new JointHinge(b0,b1));
	j.setWorldAxis(vec3(1.0f,0.0f,0.0f));
	j.setLinearRestitution(0.4f);
	j.setAngularRestitution(0.4f);
	j.setLinearSoftness(0.4f);
	j.setAngularSoftness(0.4f);
	j.setAngularDamping(8.0f);
	j.setNumIterations(16);
	joints.append(j);
}

void create_joint_angular_0(Body b0,Body b1) {
	JointHinge j = class_remove(new JointHinge(b0,b1));
	j.setWorldAxis(vec3(1.0f,0.0f,0.0f));
	j.setLinearRestitution(0.4f);
	j.setAngularRestitution(0.4f);
	j.setLinearSoftness(0.4f);
	j.setAngularSoftness(0.4f);
	j.setAngularDamping(8.0f);
	j.setAngularLimitFrom(-20.0f);
	j.setAngularLimitTo(20.0f);
	j.setNumIterations(16);
	joints.append(j);
}

void create_joint_angular_1(Body b0,Body b1) {
	JointHinge j = class_remove(new JointHinge(b0,b1));
	j.setWorldAxis(vec3(1.0f,0.0f,0.0f));
	j.setLinearRestitution(0.4f);
	j.setAngularRestitution(0.4f);
	j.setLinearSoftness(0.4f);
	j.setAngularSoftness(0.4f);
	j.setAngularDamping(8.0f);
	j.setAngularLimitFrom(-8.0f);
	j.setAngularLimitTo(8.0f);
	j.setNumIterations(16);
	joints.append(j);
}

/*
 */
void create_body(string func,Mat4 transform) {
	
	float offset = 1.2f;
	
	Body b0 = createBodyBox(vec3(1.0f),0.0f,0.5f,0.5f,get_material(0),transform * translate(0.0f,0.0f,offset * 0.0f));
	Body b1 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 1.0f));
	Body b2 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 2.0f));
	Body b3 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 3.0f));
	Body b4 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 4.0f));
	Body b5 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 5.0f));
	Body b6 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 6.0f));
	Body b7 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(2),transform * translate(0.0f,0.0f,offset * 7.0f));
	
	call(func,b0,b1);
	call(func,b1,b2);
	call(func,b2,b3);
	call(func,b3,b4);
	call(func,b4,b5);
	call(func,b5,b6);
	call(func,b6,b7);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("JointHinge");
	
	create_body("create_joint_default",translate(Vec3(0.0f,-10.0f,10.0f)));
	createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.5f,-9.5f,20.0f)));
	
	create_body("create_joint_angular_0",translate(Vec3(0.0f,0.0f,10.0f)));
	createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.5f,0.5f,20.0f)));
	
	create_body("create_joint_angular_1",translate(Vec3(0.0f,10.0f,10.0f)));
	createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.5f,10.5f,20.0f)));
	
	engine.visualizer.setEnabled(1);
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	
	foreach(Joint j; joints)
		j.renderVisualizer(vec4_one);
	
	return 1;
}

```

## hinge_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(int size,float restitution,Mat4 transform) {
	
	int num = 0;
	
	float offset = 1.0f;
	
	Body b0 = NULL;
	Body b1 = NULL;
	
	for(int i = -size; i <= size; i++) {
		float density = 1.0f;
		if(i == -size || i == size) density = 0;
		b1 = createBodyBox(vec3(1.0f,1.0f,0.25f),density,0.5f,0.5f,get_material(i),transform * translate(vec3(0.0f,i,0.0f) * offset));
		if(b0 != NULL) {
			JointHinge j = class_remove(new JointHinge(b0,b1,));
			j.setWorldAxis(rotation(transform) * vec3(1.0f,0.0f,0.0f));
			j.setLinearRestitution(restitution);
			j.setAngularRestitution(restitution);
			j.setLinearSoftness(0.2f);
			j.setAngularSoftness(0.2f);
			j.setAngularDamping(8.0f);
			j.setNumIterations(4);
			num++;
		}
		b0 = b1;
	}
	
	return num;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	int size = 32;
	
	for(int i = 0; i < size; i++) {
		num += create_body(8,float(i + 1) / size * 0.2f,translate(Vec3(20.0f - i * 1.5f,0.0f,12.0f)));
	}
	
	setDescription(format("%d JointHinge",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## hinge_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	int size = 10;
	
	Body b0 = NULL;
	Body b1 = NULL;
	
	for(int i = -size; i <= size; i++) {
		float density = 1.0f;
		if(i == -size || i == size) density = 0;
		b1 = createBodyBox(vec3(2.0f,2.0f,0.5f),density,0.5f,0.5f,get_material(0),translate(Vec3(0.0f,i * 2.0f,10.0f)));
		if(b0 != NULL) {
			JointHinge j = class_remove(new JointHinge(b0,b1));
			j.setWorldAxis(vec3(1.0f,0.0f,0.0f));
			j.setLinearRestitution(0.1f);
			j.setAngularRestitution(0.1f);
			j.setLinearSoftness(0.01f);
			j.setAngularSoftness(0.01f);
			j.setAngularDamping(32.0f);
			j.setNumIterations(16);
			num++;
		}
		b0 = b1;
	}
	
	size = 8;
	
	for(int j = 0; j < size; j++) {
		for(int i = 0; i < size - j; i++) {
			createBodyBox(vec3(2.0f),0.25f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(0.0f,i * 2.0f + j - size + 1.0f,j * 2.0f + 11.0f) * 1.01f));
		}
	}
	
	setDescription(format("%d JointHinge",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## homuncle_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
 int counter = 0;
float step = 0.75f;
float time = step;
float positions[] = ( -3.2f, -1.6f, 0.0f, 1.6f, 3.2f, 1.6f, 0.0f, -1.6f );

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	time += engine.game.getIFps() * engine.physics.getScale();
	if(time >= step) {
		createBodyHomuncle(0,vec3(0.0f),material_names,translate(Vec3(2.0f,positions[counter++ % positions.size()] - 20.0f,35.0f)));
		createBodyHomuncle(0,vec3(0.0f),material_names,translate(Vec3(2.0f,positions[counter++ % positions.size()] +  0.0f,35.0f)));
		createBodyHomuncle(0,vec3(0.0f),material_names,translate(Vec3(2.0f,positions[counter++ % positions.size()] + 20.0f,35.0f)));
		time -= step;
	}
	return 1;
}

/*
*/
void remove_node(Node node) {
	for(int i = node.getNumChildren() - 1; i >= 0; i--) {
		remove_node(node.getChild(i));
	}
	node.setEnabled(0);
	node.setWorldParent(NULL);
}

/*
 */
void trigger_enter(Node node) {
	if(node.getParent() != NULL && Node(node.getParent()).getName() == "homuncle") {
		remove_node(node.getParent());
	}
}

/*
 */
void create_body(Mat4 transform) {
	createBodyBox(vec3(1.0f,14.0f,32.0f),0.0f,0.5f,0.5f,"Unigine::mesh_base",transform * translate(0.0f,0.0f,16.0f));
	for(int i = 1; i < 8; i++) {
		createBodyCapsule(0.25f,6.0f,0.0f,0.5f,0.5f,get_material(1),transform * translate(3.0f,-5.8f,4.0f * i + 0.0f) * rotateY(90.0f));
		createBodyCapsule(0.25f,6.0f,0.0f,0.5f,0.5f,get_material(1),transform * translate(3.0f,-4.2f,4.0f * i + 2.0f) * rotateY(90.0f));
		createBodyCapsule(0.25f,6.0f,0.0f,0.5f,0.5f,get_material(1),transform * translate(3.0f,-2.6f,4.0f * i + 0.0f) * rotateY(90.0f));
		createBodyCapsule(0.25f,6.0f,0.0f,0.5f,0.5f,get_material(1),transform * translate(3.0f,-0.8f,4.0f * i + 2.0f) * rotateY(90.0f));
		createBodyCapsule(0.25f,6.0f,0.0f,0.5f,0.5f,get_material(1),transform * translate(3.0f, 0.8f,4.0f * i + 0.0f) * rotateY(90.0f));
		createBodyCapsule(0.25f,6.0f,0.0f,0.5f,0.5f,get_material(1),transform * translate(3.0f, 2.6f,4.0f * i + 2.0f) * rotateY(90.0f));
		createBodyCapsule(0.25f,6.0f,0.0f,0.5f,0.5f,get_material(1),transform * translate(3.0f, 4.2f,4.0f * i + 0.0f) * rotateY(90.0f));
		createBodyCapsule(0.25f,6.0f,0.0f,0.5f,0.5f,get_material(1),transform * translate(3.0f, 5.8f,4.0f * i + 2.0f) * rotateY(90.0f));
	}
}

/*
 */
int init() {
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(40.0f,0.0f,30.0f));
	createPlaneWithBody();
	setDescription("Homuncle");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	create_body(translate(Vec3(0.0f,-20.0f,0.0f)));
	create_body(translate(Vec3(0.0f,  0.0f,0.0f)));
	create_body(translate(Vec3(0.0f, 20.0f,0.0f)));
	return 1;
	
	WorldTrigger trigger = addToEditor(new WorldTrigger(vec3(100.0f,100.0f,4.0f)));
	trigger.getEventEnter().connect(functionid(trigger_enter));
	
	return 1;
}

```

## homuncle_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
int counter = 0;
float step = 2.0f;
float time = step;

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	if(counter > 20) return 1;
	
	time += engine.game.getIFps() * engine.physics.getScale();
	if(time >= step) {
		createBodyHomuncle(0,vec3(0.0f),material_names,translate(Vec3(21.0f,-10.0f,20.0f)));
		counter++;
		time -= step;
	}
	return 1;
}

/*
 */
void create_tracks(int num,Mat4 transform) {
	
	void create_joint(Body b0,Body b1) {
		JointHinge j = class_remove(new JointHinge(b0,b1));
		j.setWorldAnchor(b1.getTransform() * Vec3_zero);
		j.setWorldAxis(rotation(transform) * vec3(1.0f,0.0f,0.0f));
		j.setAngularVelocity(4.0f);
		j.setAngularTorque(2000.0f);
		j.setNumIterations(4);
	}
	
	float step = 2.0f;
	float radius = 1.9f;
	
	createTracks(vec3(6.0f,1.9f,0.4f),1.0f,2.0f,0.5f,step,num,get_material(0),transform);
	
	Body b00 = createBodyBox(vec3(0.5f,step * num * 2,6.0f),0.0f,0.0f,0.5f,get_material(1),transform * translate(-3.25f,0.0f, 0.00f));
	Body b01 = createBodyBox(vec3(7.0f,step * num * 2,0.5f),0.0f,0.0f,0.5f,get_material(1),transform * translate( 0.00f,0.0f,-3.25f));
	Body b02 = createBodyBox(vec3(0.5f,step * num * 2,6.0f),0.0f,0.0f,0.5f,get_material(1),transform * translate( 3.25f,0.0f, 0.00f));
	
	Body b10 = createBodyPrism(radius,radius,5.0f,6,1.0f,16.0f,0.5f,get_material(3),transform * translate(0.0f,-step * num * 1.0f,0.0f) * rotateY(90.0f));
	Body b11 = createBodyPrism(radius,radius,5.0f,6,1.0f,16.0f,0.5f,get_material(3),transform * translate(0.0f,-step * num * 0.5f,0.0f) * rotateY(90.0f));
	Body b12 = createBodyPrism(radius,radius,5.0f,6,1.0f,16.0f,0.5f,get_material(3),transform * translate(0.0f, step * num * 0.0f,0.0f) * rotateY(90.0f));
	Body b13 = createBodyPrism(radius,radius,5.0f,6,1.0f,16.0f,0.5f,get_material(3),transform * translate(0.0f, step * num * 0.5f,0.0f) * rotateY(90.0f));
	Body b14 = createBodyPrism(radius,radius,5.0f,6,1.0f,16.0f,0.5f,get_material(3),transform * translate(0.0f, step * num * 1.0f,0.0f) * rotateY(90.0f));
	
	create_joint(b01,b10);
	create_joint(b01,b11);
	create_joint(b01,b12);
	create_joint(b01,b13);
	create_joint(b01,b14);
	
	b00 = NULL;
	b02 = NULL;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(50.0f,0.0f,40.0f));
	createPlaneWithBody();
	setDescription("Homuncle");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	create_tracks(10,Mat4(rotateZ(-10.0f) * translate(  3.0f, 22.0f,10.0f) * rotateZ(100.0f) * rotateX(16.0f)));
	create_tracks(10,Mat4(rotateZ(-10.0f) * translate( -3.0f,-22.0f,10.0f) * rotateZ(280.0f) * rotateX(16.0f)));
	create_tracks(10,Mat4(rotateZ(-10.0f) * translate( 22.0f, -3.0f,10.0f) * rotateZ( 10.0f) * rotateX(16.0f)));
	create_tracks(10,Mat4(rotateZ(-10.0f) * translate(-22.0f,  3.0f,10.0f) * rotateZ(190.0f) * rotateX(16.0f)));
	
	return 1;
}

```

## homuncle_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(60.0f,0.0f,50.0f));
	createPlaneWithBody();
	setDescription("Homuncle");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 5;
	
	for(int i = -size; i <= size; i++) {
		createSpringal(0.42f + abs(i) * 0.2f,material_names,Mat4(translate(20.0f,10.0f * i,0.0f)));
		createSpringal(0.42f + abs(i) * 0.2f,material_names,Mat4(translate(-100.0f,10.0f * i,0.0f) * rotateZ(180.0f)));
	}
	
	return 1;
}

/*
 */
void updatePhysics() {
	
	unlockSpringals();
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## homuncle_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
 int counter = 0;
float step = 2.0f;
float time = step;
float positions[] = ( 23.0f, 19.0f, 15.0f, 11.0f, 7.0f );

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	if(counter > 30) return 1;
	
	time += engine.game.getIFps() * engine.physics.getScale();
	if(time >= step) {
		createBodyHomuncle(0,vec3(-100.0f,0.0f,10.0f),material_names,translate(Vec3(20.0f,0.0f,positions[counter % positions.size()])));
		counter++;
		time -= step;
	}
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(50.0f,0.0f,40.0f));
	createPlaneWithBody();
	setDescription("Homuncle");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 8;
	float space = 4.01f;
	
	for(int k = 0; k < 4; k++) {
		for(int j = 0; j < size; j++) {
			for(int i = 0; i < size - j; i++) {
				createBodyBox(vec3(4.0f),1.5f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(-k * 2.0f,i + j * 0.5f - size * 0.5f + 0.5f,j + 0.5f) * space));
			}
		}
	}
	
	return 1;
}

```

## homuncle_04.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
 int counter = 0;
float step = 2.0f;
float time = step;

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	if(counter > 5) return 1;
	
	time += engine.game.getIFps() * engine.physics.getScale();
	if(time >= step) {
		create_car(Mat4(translate(50.0f,0.0f,15.0f) * rotateZ(engine.game.getRandom(-10.0f,10.0f))));
		create_car(Mat4(translate(-50.0f,0.0f,15.0f) * rotateZ(engine.game.getRandom(-10.0f,10.0f) + 180.0f)));
		counter++;
		time -= step;
	}
	return 1;
}

/*
 */
void create_car(Mat4 transform) {
	
	JointSuspension create_wheel(Body b0,Body b1) {
		JointSuspension j = class_remove(new JointSuspension(b0,b1));
		j.setWorldAxis0(rotation(transform) * vec3(0.0f,0.0f,1.0f));
		j.setWorldAxis1(rotation(transform) * vec3(0.0f,1.0f,0.0f));
		j.setLinearSpring(200.0f);
		j.setLinearDamping(2.0f);
		j.setLinearLimitFrom(-1.0f);
		j.setLinearLimitTo(0.0f);
		j.setAngularVelocity(20.0f);
		j.setAngularTorque(40.0f);
		j.setNumIterations(4);
		return j;
	}
	
	Body body = createBodyBox(vec3(6.0f,3.0f,1.0f),1.0f,0.5f,0.5f,get_material(0),transform);
	
	Body wheel_0 = createBodySphere(0.75f,1.0f,1.0f,0.5f,get_material(1),transform * translate( 2.0f, 1.75f,-0.5f));
	Body wheel_1 = createBodySphere(0.75f,1.0f,1.0f,0.5f,get_material(1),transform * translate( 2.0f,-1.75f,-0.5f));
	Body wheel_2 = createBodySphere(0.75f,1.0f,1.0f,0.5f,get_material(1),transform * translate(-2.0f, 1.75f,-0.5f));
	Body wheel_3 = createBodySphere(0.75f,1.0f,1.0f,0.5f,get_material(1),transform * translate(-2.0f,-1.75f,-0.5f));
	
	create_wheel(body,wheel_0);
	create_wheel(body,wheel_1);
	create_wheel(body,wheel_2);
	create_wheel(body,wheel_3);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Homuncle");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 3;
	
	createBodyBox(vec3(50.0f,100.0f,2.0f),0.0f,0.5f,0.5f,get_material(2),Mat4(translate(-40.0f,0.0f,7.0f) * rotateY(20.0f)));
	createBodyBox(vec3(50.0f,100.0f,2.0f),0.0f,0.5f,0.5f,get_material(2),Mat4(translate( 40.0f,0.0f,7.0f) * rotateY(-20.0f)));
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			createBodyHomuncle(1,vec3(0.0f),material_names,translate(Vec3(i * 2.5f,j * 5.0f,0.1f)));
		}
	}
	
	return 1;
}

```

## homuncle_05.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int save;
int play;
float time;
float ifps;

int scenes[0];

BodyRigid car;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	engine.console.onscreenMessage("%f km/h\n",length(car.getLinearVelocity()) * 3.6f / 2.5f);
	
	if(play && scenes.size() > 0) {
		
		time += engine.game.getIFps() * 0.25f;
		int scene = floor(time / ifps);
		if(scene >= scenes.size()) {
			time = 0.0f;
			scene = 0;
		}
		
		engine.physics.setCurrentSubframeTime(time - scene * ifps);
		engine.physics.restoreScene(scenes[scene]);
	}
	
	return 1;
}

/*
 */
void updatePhysics() {
	
	if(save) {
		scenes.append(engine.physics.saveScene());
	}
	
	return 1;
}

/*
 */
void trigger_0_enter() {
	save = 1;
}

void trigger_1_enter() {
	if (save == 0)
		return;

	engine.physics.setEnabled(0);
	save = 0;
	play = 1;
}

/*
 */
BodyRigid create_car(Mat4 transform) {
	
	JointSuspension create_wheel(Body b0,Body b1) {
		JointSuspension j = class_remove(new JointSuspension(b0,b1));
		j.setWorldAxis0(rotation(transform) * vec3(0.0f,0.0f,1.0f));
		j.setWorldAxis1(rotation(transform) * vec3(0.0f,1.0f,0.0f));
		j.setLinearSpring(200.0f);
		j.setLinearDamping(2.0f);
		j.setLinearLimitFrom(-1.0f);
		j.setLinearLimitTo(0.0f);
		j.setNumIterations(4);
		return j;
	}
	
	Body body = createBodyBox(vec3(6.0f,3.0f,2.0f),2.0f,0.5f,0.5f,get_material(0),transform);
	
	Body wheel_0 = createBodySphere(0.75f,2.0f,1.0f,0.5f,get_material(1),transform * translate( 2.0f, 1.75f,-1.25f));
	Body wheel_1 = createBodySphere(0.75f,2.0f,1.0f,0.5f,get_material(1),transform * translate( 2.0f,-1.75f,-1.25f));
	Body wheel_2 = createBodySphere(0.75f,2.0f,1.0f,0.5f,get_material(1),transform * translate(-2.0f, 1.75f,-1.25f));
	Body wheel_3 = createBodySphere(0.75f,2.0f,1.0f,0.5f,get_material(1),transform * translate(-2.0f,-1.75f,-1.25f));
	
	create_wheel(body,wheel_0);
	create_wheel(body,wheel_1);
	create_wheel(body,wheel_2);
	create_wheel(body,wheel_3);
	
	return body;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Homuncle");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	createBodyBox(vec3(400.0f,20.0f,2.0f),0.0f,0.5f,0.5f,get_material(2),Mat4(translate(-200.0f,0.0f,65.0f) * rotateY(20.0f)));
	
	car = create_car(Mat4(translate(-370.0f,0.0f,132.0f) * rotateZ(180.0f)));
	car.getObject().setTriggerInteractionEnabled(true);
	
	createBodyHomuncle(1,vec3(0.0f),material_names,translate(Vec3( 0.0f,-5.0f,0.1f)));
	createBodyHomuncle(1,vec3(0.0f),material_names,translate(Vec3( 0.0f, 0.0f,0.1f)));
	createBodyHomuncle(1,vec3(0.0f),material_names,translate(Vec3( 0.0f, 5.0f,0.1f)));
	createBodyHomuncle(1,vec3(0.0f),material_names,translate(Vec3(-2.0f,-3.0f,0.1f)));
	createBodyHomuncle(1,vec3(0.0f),material_names,translate(Vec3(-2.0f, 3.0f,0.1f)));
	createBodyHomuncle(1,vec3(0.0f),material_names,translate(Vec3(-4.0f, 0.0f,0.1f)));
	
	WorldTrigger trigger = addToEditor(new WorldTrigger(vec3(4.0f,20.0f,4.0f)));
	trigger.setWorldTransform(translate(Vec3(-10.0f,0.0f,2.5f)));
	trigger.getEventEnter().connect(functionid(trigger_0_enter));
	trigger.setTargetNodes((car.getObject()));
	trigger.setTouch(1);
	
	trigger = addToEditor(new WorldTrigger(vec3(4.0f,100.0f,4.0f)));
	trigger.setWorldTransform(translate(Vec3(50.0f,0.0f,2.5f)));
	trigger.getEventEnter().connect(functionid(trigger_1_enter));
	trigger.setTargetNodes((car.getObject()));
	trigger.setTouch(1);
	
	PlayerPersecutor player = new PlayerPersecutor();
	player.setFixed(1);
	player.setTarget(car.getObject());
	player.setMinDistance(12.0f);
	player.setMaxDistance(16.0f);
	player.setPosition(Vec3(-400.0f,0.0f,150.0f));
	engine.game.setPlayer(player);
	
	ifps = engine.physics.getIFps();
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## icon_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

Image icons[5];

float time = 0.0f;
int direction = 1;
int icon = -1;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	setDescription("Application icon");
	
	engine.input.setMouseHandle(MOUSE_HANDLE_SOFT);
	
	return 1;
}

/*
 */
int update()
{
	
	if(icons[0] == NULL)
	{
		forloop(int i = 0; icons.size())
		{
			icons[i] = new Image();
			icons[i].load(fullPath(format("uniginescript_samples/systems/textures/icon_%d.png",i)));
			icons[i].convertToFormat(IMAGE_FORMAT_RGBA8);
		}
	}
	
	time += engine.getIFps();
	
	if(time > 0.2f)
	{
		time -= 0.2f;
		icon += direction;
		if(icon < 0 || icon == icons.size())
		{
			direction = -direction;
			icon += direction;
			icon += direction;
		}
		
		EngineWindow main_window = engine.window_manager.getMainWindow();
		main_window.setIcon(icons[icon]);
	}
	
	return 1;
}

int shutdown() {
	
	engine.input.setMouseHandle(MOUSE_HANDLE_GRAB);
	
	return 1;
}

```

## intersection_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Image image;
WidgetSprite sprite;
ObjectMeshStatic mesh;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create sample
	mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
	mesh.setMaterial(findMaterialByName(get_mesh_material(0)),"*");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setIntersection(1, 0);
	
	engine.console.setOnscreen(1);
	
	setDescription("Single mesh intersections");
	
	return 1;
}

/*
 */
int update() {
	
	int size = 64;
	
	if(image == NULL) {
		image = new Image();
		image.create2D(size,size,IMAGE_FORMAT_RGBA8);
		sprite = new WidgetSprite(engine.getGui());
		engine.gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_RIGHT);
		sprite.setTransform(scale(256.0f / size,512.0f / size,1.0f));
	}
	
	image.setChannelInt(0,0);
	image.setChannelInt(1,0);
	image.setChannelInt(2,255);
	image.setChannelInt(3,255);
	
	mesh.setWorldTransform(Mat4(rotateZ(engine.game.getTime() * 32.0f)));
	
	float isize8 = 8.0f / size;
	float isize16 = 16.0f / size;
	
	Vec3 p0 = Vec3(32.0f,0.0f,0.0f);
	Vec3 p1 = Vec3(-32.0f,0.0f,0.0f);
	
	float begin = clock();
	
	WorldIntersectionNormal intersection = new WorldIntersectionNormal();
	
	forloop(int z = 0; size) {
		
		p0.z = 15.0f - z * isize16;
		p1.z = p0.z;
		
		forloop(int y = 0; size) {
			
			p0.y = y * isize8 - 4.0f;
			p1.y = p0.y;
			
			engine.world.getIntersection(p0,p1,~0,intersection);
			engine.world.getIntersection(p0,p1,~0,intersection);
			engine.world.getIntersection(p0,p1,~0,intersection);
			engine.world.getIntersection(p0,p1,~0,intersection);
			engine.world.getIntersection(p0,p1,~0,intersection);
			engine.world.getIntersection(p0,p1,~0,intersection);
			engine.world.getIntersection(p0,p1,~0,intersection);
			Object o = engine.world.getIntersection(p0,p1,~0,intersection);
			
			if(o != NULL) {
				float c = dot(intersection.getNormal(),vec3(1.0f,0.0f,0.0f));
				image.set2D(y,z,vec4(c,c,c,1.0f));
			}
		}
	}
	
	float time = clock() - begin;
	engine.console.onscreenMessage("KRays: %f\n",size * size * 8 / time / 1000.0f);
	
	sprite.setImage(image);
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## intersection_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Image image;
WidgetSprite sprite;
ObjectMeshStatic meshes[8];

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create sample
	foreach(ObjectMeshStatic mesh; meshes) {
		mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
		mesh.setMaterial(findMaterialByName(get_mesh_material(0)),"*");
		mesh.setSurfaceProperty("surface_base","*");
		mesh.setIntersection(1, 0);
	}
	
	engine.console.setOnscreen(1);
	
	setDescription("Multiple mesh intersections");
	
	return 1;
}

/*
 */
int update() {
	
	int size = 64;
	
	if(image == NULL) {
		image = new Image();
		image.create2D(size,size,IMAGE_FORMAT_RGBA8);
		sprite = new WidgetSprite(engine.getGui());
		engine.gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_RIGHT);
		sprite.setTransform(scale(256.0f / size,512.0f / size,1.0f));
	}
	
	image.setChannelInt(0,0);
	image.setChannelInt(1,0);
	image.setChannelInt(2,255);
	image.setChannelInt(3,255);
	
	forloop(int i = 0; meshes.size()) {
		float offset = float(i) / meshes.size() * 360.0f;
		meshes[i].setWorldTransform(Mat4(rotateZ(offset + engine.game.getTime() * 32.0f) * translate(8.0f,0.0f,0.0f)));
	}
	
	float isize8 = 8.0f / size;
	float isize16 = 16.0f / size;
	
	Vec3 p0 = Vec3(32.0f,0.0f,0.0f);
	Vec3 p1 = Vec3(-32.0f,0.0f,0.0f);
	
	float begin = clock();
	
	WorldIntersectionNormal intersection = new WorldIntersectionNormal();
	
	forloop(int z = 0; size) {
		
		p0.z = 15.0f - z * isize16;
		p1.z = p0.z;
		
		forloop(int y = 0; size) {
			
			p0.y = y * isize8 - 4.0f;
			p1.y = p0.y;
			
			engine.world.getIntersection(p0,p1,~0,intersection);
			engine.world.getIntersection(p0,p1,~0,intersection);
			engine.world.getIntersection(p0,p1,~0,intersection);
			engine.world.getIntersection(p0,p1,~0,intersection);
			engine.world.getIntersection(p0,p1,~0,intersection);
			engine.world.getIntersection(p0,p1,~0,intersection);
			engine.world.getIntersection(p0,p1,~0,intersection);
			Object o = engine.world.getIntersection(p0,p1,~0,intersection);
			
			if(o != NULL) {
				float c = dot(intersection.getNormal(),vec3(1.0f,0.0f,0.0f));
				image.set2D(y,z,vec4(c,c,c,1.0f));
			}
		}
	}
	
	float time = clock() - begin;
	engine.console.onscreenMessage("KRays: %f\n",size * size * 8 / time / 1000.0f);
	
	sprite.setImage(image);
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## intersection_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Image image;
WidgetSprite sprite;
ObjectMeshStatic mesh;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create sample
	mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
	Body body = class_remove(new BodyDummy(mesh));
	class_remove(new ShapeBox(body,vec3(1.0f)));
	mesh.setMaterial(findMaterialByName(get_mesh_material(0)),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	engine.console.setOnscreen(1);
	
	setDescription("Single physics intersections");
	
	return 1;
}

/*
 */
int update() {
	
	int size = 64;
	
	if(image == NULL) {
		image = new Image();
		image.create2D(size,size,IMAGE_FORMAT_RGBA8);
		sprite = new WidgetSprite(engine.getGui());
		engine.gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_RIGHT);
		sprite.setTransform(scale(256.0f / size,256.0f / size,1.0f));
	}
	
	image.setChannelInt(0,0);
	image.setChannelInt(1,0);
	image.setChannelInt(2,255);
	image.setChannelInt(3,255);
	
	mesh.setWorldTransform(Mat4(rotateY(engine.game.getTime() * 48.0f) * rotateZ(engine.game.getTime() * 32.0f)));
	
	float isize2 = 2.0f / size;
	
	Vec3 p0 = Vec3(32.0f,0.0f,0.0f);
	Vec3 p1 = Vec3(-32.0f,0.0f,0.0f);
	
	float begin = clock();
	
	PhysicsIntersectionNormal intersection = new PhysicsIntersectionNormal();
	
	forloop(int z = 0; size) {
		
		p0.z = 1.0f - z * isize2;
		p1.z = p0.z;
		
		forloop(int y = 0; size) {
			
			p0.y = y * isize2 - 1.0f;
			p1.y = p0.y;
			
			engine.physics.getIntersection(p0,p1,~0,intersection);
			engine.physics.getIntersection(p0,p1,~0,intersection);
			engine.physics.getIntersection(p0,p1,~0,intersection);
			engine.physics.getIntersection(p0,p1,~0,intersection);
			engine.physics.getIntersection(p0,p1,~0,intersection);
			engine.physics.getIntersection(p0,p1,~0,intersection);
			engine.physics.getIntersection(p0,p1,~0,intersection);
			Object o = engine.physics.getIntersection(p0,p1,~0,intersection);
			
			if(o != NULL) {
				float c = dot(intersection.getNormal(),vec3(1.0f,0.0f,0.0f));
				image.set2D(y,z,vec4(c,c,c,1.0f));
			}
		}
	}
	
	float time = clock() - begin;
	engine.console.onscreenMessage("KRays: %f\n",size * size * 8 / time / 1000.0f);
	
	sprite.setImage(image);
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## intersection_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Image image;
WidgetSprite sprite;
ObjectMeshStatic mesh;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create sample
	mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
	mesh.addChild(addToEditor(new ObstacleBox(vec3(1.0f))));
	mesh.setMaterial(findMaterialByName(get_mesh_material(0)),"*");
	mesh.setSurfaceProperty("surface_base","*");

	engine.console.setOnscreen(1);
	
	setDescription("Single game intersections");
	
	return 1;
}

/*
 */
int update() {
	
	int size = 64;
	
	if(image == NULL) {
		image = new Image();
		image.create2D(size,size,IMAGE_FORMAT_RGBA8);
		sprite = new WidgetSprite(engine.getGui());
		engine.gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_RIGHT);
		sprite.setTransform(scale(256.0f / size,256.0f / size,1.0f));
	}
	
	image.setChannelInt(0,0);
	image.setChannelInt(1,0);
	image.setChannelInt(2,255);
	image.setChannelInt(3,255);
	
	mesh.setWorldTransform(Mat4(rotateY(engine.game.getTime() * 48.0f) * rotateZ(engine.game.getTime() * 32.0f)));
	
	float isize2 = 2.0f / size;
	
	Vec3 p0 = Vec3(32.0f,0.0f,0.0f);
	Vec3 p1 = Vec3(-32.0f,0.0f,0.0f);
	
	float begin = clock();
	
	GameIntersection intersection = new GameIntersection();
	
	forloop(int z = 0; size) {
		
		p0.z = 1.0f - z * isize2;
		p1.z = p0.z;
		
		forloop(int y = 0; size) {
			
			p0.y = y * isize2 - 1.0f;
			p1.y = p0.y;
			
			engine.game.getIntersection(p0,p1,0.1f,~0,intersection);
			engine.game.getIntersection(p0,p1,0.1f,~0,intersection);
			engine.game.getIntersection(p0,p1,0.1f,~0,intersection);
			engine.game.getIntersection(p0,p1,0.1f,~0,intersection);
			engine.game.getIntersection(p0,p1,0.1f,~0,intersection);
			engine.game.getIntersection(p0,p1,0.1f,~0,intersection);
			engine.game.getIntersection(p0,p1,0.1f,~0,intersection);
			Obstacle o = engine.game.getIntersection(p0,p1,0.1f,~0,intersection);
			
			if(o != NULL) {
				image.set2D(y,z,vec4(1.0f));
			}
		}
	}
	
	float time = clock() - begin;
	engine.console.onscreenMessage("KRays: %f\n",size * size * 8 / time / 1000.0f);
	
	sprite.setImage(image);
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## kinect_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>
using Unigine::Samples;

#ifdef HAS_KINECT

/*
 */
int IMAGE_HEIGHT = 424;
int NUM_BUFFERS = 3;

WidgetSprite sprites[NUM_BUFFERS] = (
	NULL,
	NULL,
	NULL,
);

WidgetLabel labels[NUM_BUFFERS] = (
	NULL,
	NULL,
	NULL,
);

string names[NUM_BUFFERS] = (
	"Color",
	"Depth",
	"Infrared",
);

string functions[NUM_BUFFERS] = (
	functionid(engine.kinect.getColorBuffer),
	functionid(engine.kinect.getDepthBuffer),
	functionid(engine.kinect.getInfraredBuffer),
);

/*
 */
void update_buffer(int index) {
	
	Image image = call(functions[index]);
	if(image == NULL) return;
	
	WidgetLabel label = labels[index];
	WidgetSprite sprite = sprites[index];
	string name = names[index];
	
	int width = image.getWidth();
	int height = image.getHeight();
	
	label.setText(format("%s (%dx%d)",name,width,height));
	float aspect = float(width) / float(height);
	
	int new_width = int(IMAGE_HEIGHT * aspect);
	int new_height = IMAGE_HEIGHT;
	
	image.resize(new_width,new_height);
	sprite.setImage(image);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	if(engine.kinect.init(KINECT_STREAM_INFRARED | KINECT_STREAM_DEPTH | KINECT_STREAM_COLOR) == 0) {
		setDescription("Kinect plugin is not initialized");
		return 1;
	}
	
	Gui gui = engine.getGui();
	
	WidgetGridBox gbox = new WidgetGridBox(gui,3);
	gbox.setPadding(0,0,150,0);
	
	forloop(int i = 0; NUM_BUFFERS) {
		labels[i] = new WidgetLabel(gui);
		labels[i].setFontSize(20);
		gbox.addChild(labels[i]);
	}
	
	forloop(int i = 0; NUM_BUFFERS) {
		sprites[i] = new WidgetSprite(gui);
		gbox.addChild(sprites[i]);
	}
	
	gui.addChild(gbox,GUI_ALIGN_OVERLAP);
	
	setDescription("Kinect buffers");
	
	return 1;
}

int shutdown() {
	engine.kinect.shutdown();
	return 1;
}

/*
 */
int update() {
	forloop(int i = 0; NUM_BUFFERS) {
		update_buffer(i);
	}
	
	return 1;
}

#else

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	setDescription("Kinect plugin is not loaded");
	
	return 1;
}

#endif

```

## kinect_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>
using Unigine::Samples;

#ifdef HAS_KINECT

/*
 */
vec4 colors[KINECT_NUM_BODIES] = (
	vec4(1.0f,0.0f,0.0f,1.0f),
	vec4(0.0f,1.0f,0.0f,1.0f),
	vec4(0.0f,0.0f,1.0f,1.0f),
	vec4(1.0f,1.0f,0.0f,1.0f),
	vec4(0.0f,1.0f,1.0f,1.0f),
	vec4(1.0f,0.0f,1.0f,1.0f),
);

int hierarchy[] = (
	
	// torso
	KINECT_BONE_HEAD : KINECT_BONE_NECK,
	KINECT_BONE_NECK : KINECT_BONE_SPINE_SHOULDER,
	KINECT_BONE_SPINE_SHOULDER : KINECT_BONE_SPINE_MID,
	KINECT_BONE_SHOULDER_LEFT : KINECT_BONE_SPINE_SHOULDER,
	KINECT_BONE_SHOULDER_RIGHT : KINECT_BONE_SPINE_SHOULDER,
	KINECT_BONE_SPINE_MID : KINECT_BONE_SPINE_BASE,
	KINECT_BONE_HIP_LEFT : KINECT_BONE_SPINE_BASE,
	KINECT_BONE_HIP_RIGHT : KINECT_BONE_SPINE_BASE,
	
	// left hand
	KINECT_BONE_ELBOW_LEFT : KINECT_BONE_SHOULDER_LEFT,
	KINECT_BONE_WRIST_LEFT : KINECT_BONE_ELBOW_LEFT,
	KINECT_BONE_HAND_LEFT : KINECT_BONE_WRIST_LEFT,
	KINECT_BONE_HAND_TIP_LEFT : KINECT_BONE_HAND_LEFT,
	KINECT_BONE_THUMB_LEFT : KINECT_BONE_HAND_LEFT,
	
	// right hand
	KINECT_BONE_ELBOW_RIGHT : KINECT_BONE_SHOULDER_RIGHT,
	KINECT_BONE_WRIST_RIGHT : KINECT_BONE_ELBOW_RIGHT,
	KINECT_BONE_HAND_RIGHT : KINECT_BONE_WRIST_RIGHT,
	KINECT_BONE_HAND_TIP_RIGHT : KINECT_BONE_HAND_RIGHT,
	KINECT_BONE_THUMB_RIGHT : KINECT_BONE_HAND_RIGHT,
	
	// left leg
	KINECT_BONE_KNEE_LEFT : KINECT_BONE_HIP_LEFT,
	KINECT_BONE_ANKLE_LEFT : KINECT_BONE_KNEE_LEFT,
	KINECT_BONE_FOOT_LEFT : KINECT_BONE_ANKLE_LEFT,
	
	// right leg
	KINECT_BONE_KNEE_RIGHT : KINECT_BONE_HIP_RIGHT,
	KINECT_BONE_ANKLE_RIGHT : KINECT_BONE_KNEE_RIGHT,
	KINECT_BONE_FOOT_RIGHT : KINECT_BONE_ANKLE_RIGHT,
);

int UNTRACKED_TRESHOLD = 2.0f;

/*
 */
class Skeleton {
		
	private:
		
		int index;
		vec4 color;
		vec3 positions[KINECT_NUM_BONES];
		quat orientations[KINECT_NUM_BONES];
		float untracked_time;
		
	public:
		
		Skeleton(int i,vec4 c) {
			index = i;
			color = c;
			
			forloop(int bone = 0; KINECT_NUM_BONES) {
				positions[bone] = vec3_zero;
				orientations[bone] = vec3_zero;
			}
		}
		
		void update(float ifps) {
			if(engine.kinect.isBodyTracked(index) == 0) {
				untracked_time = max(untracked_time - ifps,0.0f);
				return;
			}
			
			untracked_time = UNTRACKED_TRESHOLD;
			
			forloop(int bone = 0; KINECT_NUM_BONES) {
				positions[bone] = engine.kinect.getBonePosition(index,bone);
				orientations[bone] = engine.kinect.getBoneOrientation(index,bone);
			}
		}
		
		void visualizer() {
			if(untracked_time < EPSILON) return;
			
			forloop(int bone = 0; KINECT_NUM_BONES) {
				int parent = hierarchy.check(bone,-1);
				if(parent == -1) continue;
				
				Vec3 v1 = Vec3(positions[bone]);
				Vec3 v2 = Vec3(positions[parent]);
				
				engine.visualizer.renderLine3D(v1,v2,color);
			}
		}
};

Skeleton skeletons[0];

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	if(engine.kinect.init(KINECT_STREAM_BODY) == 0) {
		setDescription("Kinect plugin is not initialized");
		return 1;
	}
	
	engine.console.run("show_visualizer 1");
	
	PlayerDummy player = new PlayerDummy();
	Mat4 modelview = lookAt(Vec3(0.0f,-2.0f,0.0f),Vec3_zero,Vec3(0.0f,0.0f,1.0f));
	player.setWorldTransform(inverse(modelview));
	
	engine.game.setPlayer(player);
	
	forloop(int body = 0; KINECT_NUM_BODIES) {
		skeletons.append(new Skeleton(body,colors[body]));
	}
	
	setDescription("Kinect skeleton");
	
	return 1;
}

int shutdown() {
	engine.kinect.shutdown();
	return 1;
}

/*
 */
int update() {
	skeletons.call("update",engine.game.getIFps());
	skeletons.call("visualizer");
	
	return 1;
}

#else

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	setDescription("Kinect plugin is not loaded");
	
	return 1;
}

#endif

```

## kinect_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>
using Unigine::Samples;

#ifdef HAS_KINECT

/*
 */
int UNTRACKED_TRESHOLD = 2.0f;

/*
 */
vec4 colors[KINECT_NUM_BODIES] = (
	vec4(1.0f,0.0f,0.0f,1.0f),
	vec4(0.0f,1.0f,0.0f,1.0f),
	vec4(0.0f,0.0f,1.0f,1.0f),
	vec4(1.0f,1.0f,0.0f,1.0f),
	vec4(0.0f,1.0f,1.0f,1.0f),
	vec4(1.0f,0.0f,1.0f,1.0f),
);

/*
 */
class WidgetKinectFaces {
		
	private:
		
		int buffer_function;
		int bounds_function;
		int points_function;
		WidgetCanvas canvas;
		
		ivec4 face_bounds[KINECT_NUM_BODIES];
		vec3 face_points[KINECT_NUM_BODIES * KINECT_NUM_FACE_POINTS];
		float untracked_time[KINECT_NUM_BODIES];
		
	public:
		
		/*
		 */
		WidgetKinectFaces(int buffer_func,int bounds_func,int points_func) {
			buffer_function = buffer_func;
			bounds_function = bounds_func;
			points_function = points_func;
			
			canvas = new WidgetCanvas(engine.getGui());
		}
		
		~WidgetKinectFaces() {
			delete canvas;
		}
		
		/*
		 */
		Widget getWidget() { return canvas; }
		
		/*
		 */
		void update(float ifps) {
			forloop(int face = 0; KINECT_NUM_BODIES) {
				if(engine.kinect.isFaceTracked(face) == 0) {
					untracked_time[face] = max(untracked_time[face] - ifps,0.0f);
					continue;
				}
				
				untracked_time[face] = UNTRACKED_TRESHOLD;
				face_bounds[face] = call(bounds_function,face);
				
				forloop(int point = 0; KINECT_NUM_FACE_POINTS) {
					face_points[KINECT_NUM_FACE_POINTS * face + point] = call(points_function,face,point);
				}
			}
		}
		
		void visualizer() {
			
			// buffer
			Image buffer = call(buffer_function);
			if(buffer == NULL) return;
			
			canvas.clear();
			
			canvas.setWidth(buffer.getWidth());
			canvas.setHeight(buffer.getHeight());
			canvas.setImage(buffer);
			
			canvas.arrange();
			
			int poly = canvas.addPolygon();
			
			vec3 p0 = vec3(0.0f,0.0f,0.0f);
			vec3 p1 = vec3(buffer.getWidth(),0.0f,0.0f);
			vec3 p2 = vec3(buffer.getWidth(),buffer.getHeight(),0.0f);
			vec3 p3 = vec3(0.0f,buffer.getHeight(),0.0f);
			
			vec3 t0 = vec3(0.0f,0.0f,0.0f);
			vec3 t1 = vec3(1.0f,0.0f,0.0f);
			vec3 t2 = vec3(1.0f,1.0f,0.0f);
			vec3 t3 = vec3(0.0f,1.0f,0.0f);
			
			canvas.addPolygonPoint(poly,p0);
			canvas.setPolygonTexCoord(poly,t0);
			
			canvas.addPolygonPoint(poly,p1);
			canvas.setPolygonTexCoord(poly,t1);
			
			canvas.addPolygonPoint(poly,p2);
			canvas.setPolygonTexCoord(poly,t2);
			
			canvas.addPolygonPoint(poly,p3);
			canvas.setPolygonTexCoord(poly,t3);
			
			// faces
			forloop(int face = 0; KINECT_NUM_BODIES) {
				
				if(untracked_time[face] < EPSILON) continue;
				
				// bounds
				ivec4 bounds = face_bounds[face];
				
				int line = canvas.addLine();
				canvas.setLineColor(line,colors[face]);
				canvas.addLinePoint(line,vec3(bounds.x,bounds.y,0.0f));
				canvas.addLinePoint(line,vec3(bounds.z,bounds.y,0.0f));
				canvas.addLinePoint(line,vec3(bounds.z,bounds.w,0.0f));
				canvas.addLinePoint(line,vec3(bounds.x,bounds.w,0.0f));
				
				forloop(int index = 0; 4) {
					canvas.addLineIndex(line,index);
					canvas.addLineIndex(line,(index + 1) % 4);
				}
				
				// points
				forloop(int point = 0; KINECT_NUM_FACE_POINTS) {
					int point_line = canvas.addLine();
					canvas.setLineColor(point_line,colors[face]);
					
					vec3 point = face_points[KINECT_NUM_FACE_POINTS * face + point];
					
					int num_segments = 3;
					forloop(int segment = 0; num_segments) {
						float angle = PI2 * segment / num_segments;
						
						canvas.addLinePoint(point_line,point + vec3(sin(angle),cos(angle),0.0f));
						canvas.addLineIndex(point_line,segment);
						canvas.addLineIndex(point_line,(segment + 1) % num_segments);
					}
				}
			}
		}
};

WidgetKinectFaces faces[0];

/*
 */
int update() {
	faces.call("update",engine.game.getIFps());
	faces.call("visualizer");
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	if(engine.kinect.init(KINECT_STREAM_INFRARED | KINECT_STREAM_COLOR | KINECT_STREAM_BODY) == 0) {
		setDescription("Kinect plugin is not initialized");
		return 1;
	}
	
	faces.append(
		new WidgetKinectFaces(
			functionid(engine.kinect.getColorBuffer),
			functionid(engine.kinect.getFaceBoundsInColorSpace),
			functionid(engine.kinect.getFacePointInColorSpace)
		)
	);
	
	Gui gui = engine.getGui();
	
	gui.addChild(faces[0].getWidget(),GUI_ALIGN_OVERLAP | GUI_ALIGN_BACKGROUND | GUI_ALIGN_EXPAND);
	
	setDescription("Kinect faces");
	
	return 1;
}

int shutdown() {
	engine.kinect.shutdown();
	return 1;
}

#else

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	setDescription("Kinect plugin is not loaded");
	
	return 1;
}

#endif

```

## lights_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>
#include <core/systems/tracker/tracker.h>

using Unigine::Samples;
using Unigine::Tracker;

/*
 */
string mesh_material_names[] = ( "tracker_mesh_red", "tracker_mesh_green", "tracker_mesh_blue", "tracker_mesh_orange", "tracker_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
void update_track(Unigine::Tracker::TrackerTrack track) {
	
	if(track == NULL) return;
	
	float time = track.getMinTime();
	float min_time = track.getMinTime();
	float max_time = track.getMaxTime();
	float unit_time = track.getUnitTime();
	
	while(1) {
		
		// update time
		time += engine.game.getIFps() / unit_time;
		time = min_time + ((time - min_time)) % (max_time - min_time);
		
		// set track
		if(engine.game.isEnabled()) track.set(time);
		
		wait;
	}
}

/*
 */
void create_mesh_scene() {
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setName(format("statue_%d_%d",size + x,size + y));
		}
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create sample
	create_mesh_scene();
	
	LightOmni light_0 = addToEditor(new LightOmni(vec4(1.0f,0.0f,0.0f,1.0f),50.0f));
	light_0.setIntensity(35.0f);
	light_0.setName("light_spot_0");
	
	LightOmni light_1 = addToEditor(new LightOmni(vec4(0.0f,1.0f,0.0f,1.0f),50.0f));
	light_1.setIntensity(35.0f);
	light_1.setName("light_spot_1");
	
	Tracker tracker = new Tracker(TRACKER_CHECK_OBJECTS);
	TrackerTrack track = tracker.loadTrack(fullPath("uniginescript_samples/tracker/tracks/lights_00.track"));
	
	thread("update_track",track);
	
	setDescription("Tracker Light parameters");
	
	return 1;
}

```

## line_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

#include <core/systems/widgets/widget.h>
#include <core/systems/widgets/widget_window.h>
#include <core/systems/widgets/widget_icon.h>
#include <core/systems/widgets/widget_line_color.h>
#include <core/systems/widgets/widget_line_environment.h>
#include <core/systems/widgets/widget_line_toggle.h>
#include <core/systems/widgets/widget_line_switch.h>
#include <core/systems/widgets/widget_line_file.h>
#include <uniginescript_samples/interface/widget_line_material.h>
#include <core/systems/widgets/widget_line_property.h>
#include <core/systems/widgets/widget_line_node.h>

/*
 */
Unigine::Widgets::Window window;
Unigine::Widgets::Icon close_icon;

Unigine::Widgets::Line line;

using Unigine::Samples;

void on_close()
{
	Unigine::Widgets::removeChild(window);
}

/*
 */
void update() {
	
	line.update();
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.setShadowDistance(100.0f);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("Unigine::Widgets::Line");
	
	using Unigine::Widgets;
	
	// window
	window = new Window("Unigine::Widgets::Line");
	window.setWidth(768);
	window.setSizeable(1);
	window.setDragAreaPadding(3, 25, 3, 0);
	
	// close icon
	close_icon = new Icon();
	close_icon.setPosition(10, -24);
	close_icon.getEventClicked().connect(functionid(on_close));
	window.addChild(close_icon, GUI_ALIGN_OVERLAP | GUI_ALIGN_TOP | GUI_ALIGN_RIGHT);

	Image img = new Image("core/gui/window_close.png");
	close_icon.setImage(img);

	// line
	float min_key = 0.0f;
	float max_key = 1.0f;
	line = new Line(min_key,max_key);
	window.addChild(line,ALIGN_EXPAND);
	ScrollBox scrollbox = line.getScrollBox();
	scrollbox.setVScrollEnabled(0);
	
	// toggle
	{
		int values[] = ( 0.05f : 0, 0.1f : 1, 0.3f : 0, 0.5f : 1, 0.7f : 0 );
		
		LineCurveToggle curve = new LineCurveToggle(vec4(1.0f,0.0f,0.0f,1.0f));
		foreachkey(float key; values) {
			curve.addKey(new LineKeyToggle(key,values[key]));
		}
		line.addCurve(curve);
	}
	
	// switch
	{
		int values[] = ( 0.1f : 0, 0.15f : 1, 0.35f : 0, 0.55f : 2, 0.75f : 0 );
		
		LineCurveSwitch curve = new LineCurveSwitch(vec4(0.0f,1.0f,0.0f,1.0f),0,16);
		foreachkey(float key; values) {
			curve.addKey(new LineKeySwitch(key,values[key]));
		}
		line.addCurve(curve);
	}
	
	// file
	{
		int values[] = ( 0.15f : 0, 0.2f : 1, 0.4f : 0, 0.6f : 1, 0.8f : 0 );
		
		LineCurveFile curve = new LineCurveFile(vec4(0.0f,0.0f,1.0f,1.0f));
		foreachkey(float key; values) {
			curve.addKey(new LineKeyFile(key,values[key],"file.txt"));
		}
		line.addCurve(curve);
	}
	
	// material
	{
		int values[] = ( 0.2f : 0, 0.25f : 1, 0.45f : 0, 0.65f : 1, 0.85f : 0 );
		
		LineCurveMaterial curve = new LineCurveMaterial(vec4(1.0f,1.0f,0.0f,1.0f));
		foreachkey(float key; values) {
			curve.addKey(new LineKeyMaterial(key,values[key],"mesh_base"));
		}
		line.addCurve(curve);
	}
	
	// property
	{
		int values[] = ( 0.25f : 0, 0.3f : 1, 0.5f : 0, 0.7f : 1, 0.9f : 0 );
		
		LineCurveProperty curve = new LineCurveProperty(vec4(0.0f,1.0f,1.0f,1.0f));
		foreachkey(float key; values) {
			curve.addKey(new LineKeyProperty(key,values[key],"surface_base"));
		}
		line.addCurve(curve);
	}
	
	// node
	{
		int values[] = ( 0.3f : 0, 0.35f : 1, 0.55f : 0, 0.75f : 1, 0.95f : 0 );
		
		LineCurveNode curve = new LineCurveNode(vec4(1.0f,1.0f,1.0f,1.0f));
		NodeDummy node = addToEditor(new NodeDummy());
		foreachkey(float key; values) {
			curve.addKey(new LineKeyNode(key,values[key],node.getID()));
		}
		line.addCurve(curve);
	}
	
	// colors
	{
		vec4 colors[] = ( 0.0f : vec4(0.0f,0.0f,1.0f,1.0f), 0.2f : vec4(0.0f,1.0f,0.0f,1.0f), 0.4f : vec4(1.0f,0.0f,0.0f,1.0f),
			0.6f : vec4(1.0f,0.0f,1.0f,1.0f), 0.8f : vec4(0.0f,1.0f,1.0f,1.0f), 1.0f : vec4(1.0f,1.0f,0.0f,1.0f) );
		
		forloop(int i = 0; 3) {
			LineCurveColor curve = new LineCurveColor();
			foreachkey(float key; colors) {
				curve.addKey(new LineKeyColor(key,colors[key]));
			}
			line.addCurve(curve);
		}
	}
	
	// environments
	{
		vec3 coefficients_0[] = (
			vec3(0.149614f,0.202668f,0.266977f), vec3(0.00799391f,0.00919635f,0.00990062f), vec3(-0.0704251f,-0.105397f,-0.133287f),
			vec3(0.00614425f,0.0102188f,0.0139381f), vec3(0.00122227f,0.0024002f,0.00313488f), vec3(-0.0144464f,-0.0173937f,-0.0208934f),
			vec3(-0.00486974f,-0.00860146f,-0.0122311f), vec3(0.00423214f,0.00653711f,0.00960071f), vec3(-0.00313404f,-0.00357011f,-0.00410586f) );
		
		vec3 coefficients_1[] = (
			vec3(0.913649f,0.790863f,0.769772f), vec3(-0.00970268f,-0.013706f,-0.0166136f), vec3(-0.151665f,-0.0147749f,0.146271f),
			vec3(-0.193757f,-0.110774f,-0.0161521f), vec3(0.0199107f,0.0159948f,0.0102464f), vec3(0.0092347f,0.0095584f,0.00833615f),
			vec3(0.00163678f,0.00808679f,0.010913f), vec3(-0.032251f,-0.0137814f,0.0129462f), vec3(0.0250893f,0.0222975f,0.0175732f) );
		
		vec3 coefficients_2[] = (
			vec3(0.798464f,0.916919f,1.09147f), vec3(-0.00774049f,-0.0101453f,-0.0172661f), vec3(-0.372166f,-0.263384f,-0.125905f),
			vec3(0.00726575f,0.00561858f,0.00233812f), vec3(0.0074139f,0.00908535f,0.0134694f), vec3(-0.0338371f,-0.0413103f,-0.0594847f),
			vec3(-0.00793087f,-0.00943771f,-0.0129308f), vec3(-0.00597846f,-0.00638183f,-0.00547286f), vec3(-0.00775822f,-0.0086536f,-0.0114637f) );
		
		forloop(int i = 0; 3) {
			LineCurveEnvironment curve = new LineCurveEnvironment();
			curve.addKey(new LineKeyEnvironment(0.1f,coefficients_0));
			curve.addKey(new LineKeyEnvironment(0.5f,coefficients_1));
			curve.addKey(new LineKeyEnvironment(0.9f,coefficients_2));
			line.addCurve(curve);
		}
	}
	
	// update line
	line.update();
	
	// window
	window.arrange();
	addChild(window,ALIGN_OVERLAP | ALIGN_CENTER);
	
	return 1;
}

```

## lines_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	Player player = createDefaultPlayer(Vec3(12.0f,0.0f,4.0f));
	createDefaultPlane();
	
	setDescription("ObjectDynamic lines shader");
	
	player.setDirection(vec3(-1.0f,0.0f,-0.25f));
	
	ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/shaders/meshes/palm.mesh")));
	mesh.setWorldTransform(translate(Vec3(0.0f,2.0f,0.0f)));
	mesh.setMaterial(findMaterialByName("mesh_base"),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	ObjectDynamic dynamic = addToEditor(new ObjectDynamic());
	dynamic.setMaterialNodeType(NODE_OBJECT_MESH_STATIC);
	dynamic.setSurfaceMode(OBJECT_DYNAMIC_MODE_LINES,0);
	dynamic.setWorldTransform(translate(Vec3(0.0f,-2.0f,0.0f)));
	dynamic.setMaterial(findMaterialByName("lines"),"*");
	dynamic.setSurfaceProperty("surface_base","*");
	
	// set vertex format to standard MeshStatic format
	dynamic.setVertexFormat((
		ivec3(	0,	OBJECT_DYNAMIC_TYPE_FLOAT,	3),
		ivec3(	12,	OBJECT_DYNAMIC_TYPE_FLOAT,	4),
		ivec3(	28,	OBJECT_DYNAMIC_TYPE_HALF,	4),
		ivec3(	36,	OBJECT_DYNAMIC_TYPE_UCHAR,	4),
	));
	
	// lines rendering mode
	dynamic.setInstancing(0);
	Mesh mesh_static = mesh.getMeshForceRAM();
	// copy surfaces
	forloop(int i = 0; mesh_static.getNumSurfaces()) {
		
		int offset = dynamic.getNumVertex();
		
		// copy vertices
		forloop(int j = 0; mesh_static.getNumVertex(i)) {
			vec3 vertex = mesh_static.getVertex(j,i);
			dynamic.addVertexFloat(0,vertex,3);
		}
		
		// copy indices
		forloop(int j = 0; mesh_static.getNumCIndices(i); 3) {
			dynamic.addIndex(offset + mesh_static.getCIndex(j + 0,i));
			dynamic.addIndex(offset + mesh_static.getCIndex(j + 1,i));
			dynamic.addIndex(offset + mesh_static.getCIndex(j + 1,i));
			dynamic.addIndex(offset + mesh_static.getCIndex(j + 2,i));
			dynamic.addIndex(offset + mesh_static.getCIndex(j + 2,i));
			dynamic.addIndex(offset + mesh_static.getCIndex(j + 0,i));
		}
	}
	
	// copy bounding box
	dynamic.setBoundBox(mesh.getBoundBox());
	
	return 1;
}

```

## loading_screen_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class LoadingScreen {
	
	Gui gui;				// gui
	
	WidgetButton button;	// button
	
	Image previous_loading_screen;

	// constructor/destructor
	LoadingScreen() {
		
		gui = engine.getGui();
		
		// button
		button = new WidgetButton(gui,"Press me");
		button.setFontSize(24);
		button.arrange();
		gui.addChild(button,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
		button.getEventClicked().connect(functionid(callback_redirector), this, functionid(button_clicked));
		previous_loading_screen = new Image();
		engine.loading_screen.getImage(previous_loading_screen);
	}
	~LoadingScreen() {
		engine.loading_screen.setImage(previous_loading_screen);
		delete button;
	}
	
	// save/restore state
	void __restore__() {
		__LoadingScreen__();
	}
	
	// button clicked callback
	void button_clicked() {
		
		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 window_size = main_window.getSize();

		float aspect = float(window_size.x) / window_size.y;
		if(aspect < 1.5f)
			engine.loading_screen.setTexturePath(fullPath("uniginescript_samples/widgets/loading_screens/loading_screen_4x3.png"),);
		else
			engine.loading_screen.setTexturePath(fullPath("uniginescript_samples/widgets/loading_screens/loading_screen_16x9.png"));

		engine.loading_screen.setThreshold(16);
		engine.loading_screen.setEnabled(1);
			
			float begin = clock();
			float progress = 0.0f;
			
			do {
				progress = (clock() - begin) * 0.5f;
				engine.loading_screen.render(progress  * 100.0f);
			} while(progress < 1.0f && engine.input.isKeyDown(INPUT_KEY_ESC) == 0);
			
		engine.loading_screen.setEnabled(0);
	}
	
	// callback redirector
	void callback_redirector(LoadingScreen loading_screen,string func) {
		loading_screen.call(func);
	}
};

LoadingScreen loading_screen;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	loading_screen = new LoadingScreen();
	
	setDescription("Image loading screen");
	
	return 1;
}

```

## loading_screen_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class LoadingScreen {
	
	Gui gui;				// gui
	
	WidgetButton button;	// button
	
	Image previous_loading_screen;

	// constructor/destructor
	LoadingScreen() {
		
		gui = engine.getGui();
		
		// button
		button = new WidgetButton(gui,"Press me");
		button.setFontSize(24);
		button.arrange();
		gui.addChild(button,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
		button.getEventClicked().connect(functionid(callback_redirector), this, functionid(button_clicked));
		previous_loading_screen = new Image();
		engine.loading_screen.getImage(previous_loading_screen);
	}
	~LoadingScreen() {
		engine.loading_screen.setImage(previous_loading_screen);
		delete button;
	}
	
	// save/restore state
	void __restore__() {
		__LoadingScreen__();
	}
	
	// button clicked callback
	void button_clicked() {
		
		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 window_size = main_window.getSize();

		Image image = new Image();

		float aspect = float(window_size.x) / window_size.y;
		if(aspect < 1.5f)
			image.load(fullPath("uniginescript_samples/widgets/loading_screens/loading_screen_4x3.png"));
		else
			image.load(fullPath("uniginescript_samples/widgets/loading_screens/loading_screen_16x9.png"));
		
		engine.loading_screen.setEnabled(1);
			
			float begin = clock();
			float progress = 0.0f;
			
			do {
				forloop(int i = 0; 256) {
					int x = rand(0,image.getWidth());
					int y = rand(0,image.getHeight() / 2);
					vec4 color = rand(vec4_zero,vec4_one);
					image.set2D(x,y,color);
					image.set2D(x,y + image.getHeight() / 2,color);
				}
				progress = (clock() - begin) * 0.5f;
				engine.loading_screen.setImage(image);
				engine.loading_screen.render(progress  * 100.0f);
			} while(progress < 1.0f && engine.input.isKeyDown(INPUT_KEY_ESC) == 0);
			
		engine.loading_screen.setEnabled(0);
	}
	
	// callback redirector
	void callback_redirector(LoadingScreen loading_screen,string func) {
		loading_screen.call(func);
	}
};

LoadingScreen loading_screen;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	loading_screen = new LoadingScreen();
	
	setDescription("Procedural image loading screen");
	
	return 1;
}


```

## loading_screen_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class LoadingScreen {
	
	Gui gui;				// gui
	
	WidgetButton button;	// button
	
	Image previous_loading_screen;

	// constructor/destructor
	LoadingScreen() {
		
		gui = engine.getGui();
		
		// button
		button = new WidgetButton(gui,"Press me");
		button.setFontSize(24);
		button.arrange();
		gui.addChild(button,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
		button.getEventClicked().connect(functionid(callback_redirector), this, functionid(button_clicked));
		previous_loading_screen = new Image();
		engine.loading_screen.getImage(previous_loading_screen);
	}
	~LoadingScreen() {
		engine.loading_screen.setImage(previous_loading_screen);
		delete button;
	}
	
	// save/restore state
	void __restore__() {
		__LoadingScreen__();
	}
	
	// button clicked callback
	void button_clicked() {
		
		engine.loading_screen.setTexturePath(fullPath("uniginescript_samples/widgets/loading_screens/loading_screen.png"));
		engine.loading_screen.setThreshold(4);
		engine.loading_screen.setTransform(vec4(0.5f,0.33f,0.5f,0.5f));
		engine.loading_screen.setBackgroundColor(engine.gui.parseColor("000b11"));
		engine.loading_screen.setText("<p align=center><font color=#545456 size=16><xy y=%100 dy=-20/>UNIGINE. (c) 2005-2022</font></p>");
		
		engine.loading_screen.setEnabled(1);
			
			float begin = clock();
			float progress = 0.0f;
			
			do {
				progress = (clock() - begin) * 0.5f;
				engine.loading_screen.render(progress  * 100.0f);
			} while(progress < 1.0f && engine.input.isKeyDown(INPUT_KEY_ESC) == 0);
			
		engine.loading_screen.setEnabled(0);
	}
	
	// callback redirector
	void callback_redirector(LoadingScreen loading_screen,string func) {
		loading_screen.call(func);
	}
};

LoadingScreen loading_screen;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	loading_screen = new LoadingScreen();
	
	setDescription("Image loading screen");
	
	return 1;
}

```

## lookup_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Image lut;
Image lut_0;
Image lut_1;

Image initial_lut;

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("Dynamic color lookup table");

	initial_lut = new Image();
	engine.render.getColorCorrectionLUTImage(initial_lut);
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	engine.render.setExposure(0.6f);
	
	return 1;
}

/*
 */
int update() {
	
	if(lut == NULL) {
		lut = new Image();
		lut_0 = new Image(fullPath("uniginescript_samples/render/textures/lookup_first.dds"));
		lut_1 = new Image(fullPath("uniginescript_samples/render/textures/lookup_second.dds"));
	}
	
	float k = sin(engine.game.getTime() * 2.0f) * 0.5f + 0.5f;
	
	lut.assignFrom(lut_0);
	lut.blend(lut_1,0,0,0,0,lut.getWidth(),lut.getHeight(),k);
	
	engine.render.setColorCorrectionLUTImage(lut);
	
	return 1;
}

int shutdown() {
	engine.render.setColorCorrectionLUTImage(initial_lut);
	return 1;
}

```

## materials_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>
#include <core/systems/tracker/tracker.h>

using Unigine::Samples;
using Unigine::Tracker;

/*
 */
string mesh_material_names[] = ( "tracker_mesh_red", "tracker_mesh_green", "tracker_mesh_blue", "tracker_mesh_orange", "tracker_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
void update_track(Unigine::Tracker::TrackerTrack track) {
	
	if(track == NULL) return;
	
	float time = track.getMinTime();
	float min_time = track.getMinTime();
	float max_time = track.getMaxTime();
	float unit_time = track.getUnitTime();
	
	while(1) {
		
		// update time
		time += engine.game.getIFps() / unit_time;
		time = min_time + ((time - min_time)) % (max_time - min_time);
		
		// set track
		if(engine.game.isEnabled()) track.set(time);
		
		wait;
	}
}

/*
 */
void create_mesh_scene() {
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setName(format("statue_%d_%d",size + x,size + y));
		}
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create sample
	create_mesh_scene();
	
	Tracker tracker = new Tracker(TRACKER_CHECK_OBJECTS);
	TrackerTrack track = tracker.loadTrack(fullPath("uniginescript_samples/tracker/tracks/materials_00.track"));
	
	thread("update_track",track);
	
	setDescription("Tracker Material parameters");
	
	return 1;
}

```

## menubox_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class MenuBox {
	
	Gui gui;				// gui
	
	WidgetMenuBox menubox;	// menubox
	
	WidgetMenuBox children[0];
	
	// constructor/destructor
	MenuBox() {
		
		gui = engine.getGui();
		
		menubox = new WidgetMenuBox(gui,8,8);
		menubox.addItem("One <right>&#9658;");
		menubox.addItem("Two <right>&#9658;");
		menubox.addItem("Three <right>&#9658;");
		menubox.addItem("Four <right>&#9658;");
		menubox.getEventClicked().connect(functionid(menubox_clicked_redirector), this, menubox);
		menubox.setFontRich(1);
		
		forloop(int i = 0; 4) {
			WidgetMenuBox child = new WidgetMenuBox(gui,8,8);
			child.addItem(format("Child %d One",i));
			child.addItem(format("Child %d Two",i));
			child.addItem(format("Child %d Three",i));
			child.addItem(format("Child %d Four",i));
			forloop(int j = 0; child.getNumItems()) {
				child.setItemToolTip(j,child.getItemText(j));
			}
			child.setItemWidget(child.addItem(NULL),new WidgetCheckBox(gui,"CheckBox"));
			child.getEventClicked().connect(functionid(menubox_clicked_redirector), this, child);
			menubox.setItemWidget(i,child);
			children.append(child);
		}
	}
	~MenuBox() {
		delete menubox;
		children.delete();
	}
	
	// save/restore state
	void __restore__() {
		__MenuBox__();
	}
	
	// show menu
	void show() {
		
		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 mouse_coord = engine.input.getMousePosition() - main_window.getPosition();
		
		menubox.setPosition(mouse_coord.x, mouse_coord.y);
		gui.addChild(menubox,GUI_ALIGN_OVERLAP);
		menubox.setFocus();
	}
	
	// callbacks
	void menubox_clicked(WidgetMenuBox widget) {
		int item = widget.getCurrentItem();
		if(item != -1) log.message("%s\n",widget.getCurrentItemText());
		else log.message("Closed\n");
		gui.removeChild(menubox);
	}
	void menubox_clicked_redirector(MenuBox menubox,WidgetMenuBox widget) {
		menubox.menubox_clicked(widget);
	}
};

MenuBox menubox;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	menubox = new MenuBox();
	
	setDescription("WidgetMenuBox press right mouse button");
	
	return 1;
}

/*
 */
int update() {
	
	if(engine.input.isMouseButtonDown(INPUT_MOUSE_BUTTON_RIGHT)) {
		menubox.show();
	}
	
	if(engine.input.isMouseButtonPressed(INPUT_MOUSE_BUTTON_RIGHT)) {
		menubox.gui.setMouseButtons(menubox.gui.getMouseButtons() & ~GUI_MOUSE_MASK_RIGHT);
	}

	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## menubox_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
Label labels[0];
MenuBox menubox;

class Label {
	
	Gui gui;				// gui
	
	WidgetLabel label;		// label
	
	int position_x;			// position x
	int position_y;			// position y
	string text;			// text
	
	// constructor/destructor
	Label() {
		
		gui = engine.getGui();
		
		label = new WidgetLabel(gui,text);
		label.setFontSize(24);
		
		label.setPosition(position_x,position_y);
		gui.addChild(label,GUI_ALIGN_OVERLAP);
		
		label.getEventPressed().connect(functionid(label_pressed_redirector), this);
	}
	Label(int x,int y,string t) {
		
		position_x = x;
		position_y = y;
		text = t;
		
		__Label__();
	}
	~Label() {
		delete label;
	}
	
	// save/restore state
	void __restore__() {
		__Label__();
	}
	
	// callbacks
	void label_pressed() {
		if(gui.getMouseButtons() == GUI_MOUSE_MASK_RIGHT) {
			menubox.show(( text, "One", "Two", "Three" ));
		}
	}
	void label_pressed_redirector(Label label) {
		label.label_pressed();
	}
};

class MenuBox {
	
	Gui gui;				// gui
	
	WidgetMenuBox menubox;	// menubox
	
	int skip;				// skip callback flag
	
	// constructor/destructor
	MenuBox() {
		
		gui = engine.getGui();
		
		menubox = new WidgetMenuBox(gui,8,8);
		
		menubox.getEventClicked().connect(functionid(menubox_clicked_redirector), this);
	}
	~MenuBox() {
		delete menubox;
	}
	
	// save/restore state
	void __restore__() {
		__MenuBox__();
	}
	
	// show menu
	void show(string items[]) {
		menubox.clear();
		foreach(string item; items) {
			menubox.addItem(item);
		}
		
		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 mouse_coord = engine.input.getMousePosition() - main_window.getPosition();
		
		menubox.setPosition(mouse_coord.x, mouse_coord.y);
		gui.addChild(menubox,GUI_ALIGN_OVERLAP);
		menubox.setFocus();
		skip = 1;
	}
	
	// callbacks
	void menubox_clicked() {
		if(skip) {
			skip = 0;
			return;
		}
		int item = menubox.getCurrentItem();
		if(item != -1) log.message("%s\n",menubox.getCurrentItemText());
		else log.message("Closed\n");
		gui.removeChild(menubox);
	}
	void menubox_clicked_redirector(MenuBox menubox) {
		menubox.menubox_clicked();
	}
};

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	forloop(int y = 0; 6) {
		forloop(int x = 0; 6) {
			Label label = new Label(200 + x * 120,200 + y * 50,format("Label %dx%d",x,y));
			labels.append(label);
		}
	}
	
	menubox = new MenuBox();
	
	setDescription("WidgetMenuBox press right mouse button on labels");
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## menubox_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class MenuBox {
	
	Gui gui;				// gui
	
	WidgetMenuBox menubox;	// menubox
	
	Widget children[0];
	
	// constructor/destructor
	MenuBox() {
		
		gui = engine.getGui();
		
		menubox = new WidgetMenuBox(gui,8,8);
		
		menubox.addItem("First item");
		
		WidgetCheckBox checkbox = new WidgetCheckBox(gui,"CheckBox");
		checkbox.getEventClicked().connect(functionid(menubox_checkbox_clicked_redirector), this, checkbox);
		menubox.setItemWidget(menubox.addItem(NULL),checkbox);
		children.append(checkbox);
		
		menubox.addItem("Second item");
		
		WidgetSlider slider = new WidgetSlider(gui);
		slider.getEventChanged().connect(functionid(menubox_slider_changed_redirector),this,slider);
		menubox.setItemWidget(menubox.addItem(NULL),slider);
		slider.setFlags(GUI_ALIGN_EXPAND);
		children.append(slider);
		
		menubox.addItem("Third item");
		
		menubox.getEventClicked().connect(functionid(menubox_clicked_redirector),this,menubox);
		menubox.setFontRich(1);
	}
	~MenuBox() {
		delete menubox;
		children.delete();
	}
	
	// save/restore state
	void __restore__() {
		__MenuBox__();
	}
	
	// show menu
	void show() {
		
		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 mouse_coord = engine.input.getMousePosition() - main_window.getPosition();
		
		menubox.setPosition(mouse_coord.x,mouse_coord.y);
		gui.addChild(menubox,GUI_ALIGN_OVERLAP);
		menubox.setFocus();
	}
	
	// callbacks
	void menubox_clicked(WidgetMenuBox widget) {
		int item = widget.getCurrentItem();
		if(item != -1 && widget.getItemWidget(item) != NULL) return;
		if(item != -1) log.message("%s\n",widget.getCurrentItemText());
		else log.message("Closed\n");
		gui.removeChild(menubox);
	}
	void menubox_clicked_redirector(MenuBox menubox,WidgetMenuBox widget) {
		menubox.menubox_clicked(widget);
	}
	
	void checkbox_clicked(WidgetCheckBox widget) {
		if(widget.isChecked()) log.message("Checked\n");
		else log.message("Unchecked\n");
	}
	void menubox_checkbox_clicked_redirector(MenuBox menubox,WidgetCheckBox widget) {
		menubox.checkbox_clicked(widget);
	}
	
	void slider_changed(WidgetSlider widget) {
		log.message("Changed %d\n",widget.getValue());
	}
	void menubox_slider_changed_redirector(MenuBox menubox,WidgetSlider widget) {
		menubox.slider_changed(widget);
	}
};

MenuBox menubox;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	menubox = new MenuBox();
	
	setDescription("WidgetMenuBox press right mouse button");
	
	return 1;
}

/*
 */
int update() {
	
	if(engine.input.isMouseButtonDown(INPUT_MOUSE_BUTTON_RIGHT)) {
		menubox.show();
	}
	
	if(engine.input.isMouseButtonPressed(INPUT_MOUSE_BUTTON_RIGHT)) {
		menubox.gui.setMouseButtons(menubox.gui.getMouseButtons() & ~GUI_MOUSE_MASK_RIGHT);
	}

	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## mesh_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 14;
	
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
				mesh.setWorldTransform(translate(Vec3(x,y,z + 15.0f) * 1.1f));
				mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y ^ z)),"*");
				mesh.setSurfaceProperty("surface_base","*");
				num++;
			}
		}
	}
	
	setDescription(format("%d ObjectMeshStatic",num));
	
	return 1;
}

```

## mesh_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 10;
	
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/stress/meshes/sphere_00.mesh")));
				mesh.setWorldTransform(translate(Vec3(x,y,z + 11.0f) * 1.1f));
				mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y ^ z)),"*");
				mesh.setSurfaceProperty("surface_base","*");
				num++;
			}
		}
	}
	
	setDescription(format("%d ObjectMeshStatic",num));
	
	return 1;
}

```

## mesh_lines_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
ObjectMeshDynamic mesh;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	Player player = createDefaultPlayer(Vec3(12.0f,0.0f,4.0f));
	
	setDescription("ObjectMeshDynamic lines shader");
	
	player.setDirection(vec3(-1.0f,0.0f,-0.25f));
	
	mesh = addToEditor(new ObjectMeshDynamic());
	mesh.setMaterial(findMaterialByName("mesh_lines"),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	int num_vertex = 8192;
	
	// indices
	forloop(int i = 0; num_vertex - 2) {
		mesh.addIndex(i + 0);
		mesh.addIndex(i + 1);
		mesh.addIndex(i + 2);
	}
	
	// vertices
	vec3 previous = vec3_zero;
	forloop(int i = 0; num_vertex) {
		float t = float(i) / (num_vertex - 1) * PI;
		float r = sin(t) * 3.0f;
		vec3 vertex = rotateZ(t * 180.0f) * vec3(0.0f,2.0f * r + sin(t * 256.0f) * r,cos(t * 256.0f) * r);
		vec4 color = abs(vec4(sin(t * 133.0f),cos(t * 237.0f),sin(t * 373.0f),1.0f));
		mesh.addVertex(vertex);
		mesh.addTangent(quat(vec4(previous,8.0f)));
		mesh.addColor(color);
		previous = vertex;
	}
	
	mesh.updateBounds();
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	mesh.setWorldTransform(Mat4(translate(0.0f,0.0f,3.0f) * rotateX(time * 8.0f) * rotateY(time * 12.0f) * rotateZ(time * 16.0f)));
	
	return 1;
}

```

## mesh_lines_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
ObjectMeshDynamic mesh;

/*
 */
void create_circle(ObjectMeshDynamic mesh,mat4 transform,int num_vertex,float radius,float size,float period,vec4 color) {
	
	int num = 0;
	int enable = 1;
	int offset = 0;
	float distance = 0.0f;
	
	vec3 previous = transform * vec3(radius,0.0f,0.0f);
	forloop(int i = 0; num_vertex) {
		
		float angle = 360.0f * i / (num_vertex - 1);
		vec3 vertex = transform * rotateZ(angle) * vec3(radius,0.0f,0.0f);
		
		distance += length(previous - vertex);
		if(distance > period) {
			enable = !enable;
			distance = 0.0f;
		}
		
		if(enable) {
			if(num++ == 0) offset = mesh.getNumVertex();
			mesh.addVertex(vertex);
			mesh.addTangent(quat(vec4(previous,size)));
			mesh.addColor(color);
		} else if(num) {
			forloop(int j = offset; offset + num - 2) {
				mesh.addIndex(j + 0);
				mesh.addIndex(j + 1);
				mesh.addIndex(j + 2);
			}
			offset += num;
			num = 0;
		}
		
		previous = vertex;
	}
	
	if(num) {
		forloop(int i = offset; offset + num - 2) {
			mesh.addIndex(i + 0);
			mesh.addIndex(i + 1);
			mesh.addIndex(i + 2);
		}
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	Player player = createDefaultPlayer(Vec3(12.0f,0.0f,4.0f));
	
	setDescription("ObjectMeshDynamic lines shader");
	
	player.setDirection(vec3(-1.0f,0.0f,-0.25f));
	
	mesh = addToEditor(new ObjectMeshDynamic());
	mesh.setMaterial(findMaterialByName("mesh_lines"),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	forloop(int i = 0; 16) {
		float angle = 180.0f * i / 15;
		create_circle(mesh,rotateY(90.0f) * rotateX(angle),1024,4.0f,4.0f,0.3f,vec4(0.0f,1.0f,0.0f,1.0f));
	}
	
	forloop(int i = 0; 16) {
		float angle = 180.0f * i / 15;
		create_circle(mesh,rotateZ(90.0f) * rotateX(angle),1024,5.0f,8.0f,0.4f,vec4(1.0f,0.0f,0.0f,1.0f));
	}
	
	forloop(int i = 0; 16) {
		float angle = 180.0f * i / 15;
		create_circle(mesh,rotateX(angle),1024,6.0f,16.0f,0.5f,vec4(0.0f,0.0f,1.0f,1.0f));
	}
	
	mesh.updateBounds();
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	mesh.setWorldTransform(Mat4(translate(0.0f,0.0f,3.0f) * rotateX(time * 8.0f) * rotateY(time * 12.0f) * rotateZ(time * 16.0f)));
	
	return 1;
}

```

## modifiers_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

LightWorld light_world_0;
LightWorld light_world_1;


int init() {

	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);

	// render parameters
	engine.render.setShadowDistance(100.0f);

	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));

	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);

	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);

	void get_file(string modifier,string name) {
		engine.filesystem.clearModifiers();
		if(strlen(modifier)) {
			engine.filesystem.addModifier(modifier);
			File file = new File();
			file.open(name, "r");
			log.message("%s -> %s (%s)\n",name,file.getName(),modifier);
		} else {
			File file = new File();
			file.open(name, "r");
			log.message("%s -> %s\n",name,file.getName());
		}
		engine.filesystem.clearModifiers();
	}

	get_file("","uniginescript_samples/systems/modifiers/dir0/file.txt");
	get_file("en","uniginescript_samples/systems/modifiers/dir0/file.txt");
	get_file("us","uniginescript_samples/systems/modifiers/dir0/file.txt");

	get_file("","uniginescript_samples/systems/modifiers/dir0/dir1/file.txt");
	get_file("en","uniginescript_samples/systems/modifiers/dir0/dir1/file.txt");
	get_file("us","uniginescript_samples/systems/modifiers/dir0/dir1/file.txt");

	setDescription("FileSystem modifiers (check console)");

	return 1;
}

int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}



```

## motion_blur_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
ObjectMeshStatic meshes[0];

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	Player player = createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("Motion blur camera velocity only");
	
	engine.console.run("render_motion_blur 1");
	
	player.setControlled(0);
	
	int num = 60;
	
	forloop(int i = 0; num) {
		ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
		mesh.setMaterial(findMaterialByName(get_phong_mesh_material(i)),"*");
		mesh.setSurfaceProperty("surface_base","*");
		meshes.append(mesh);
	}
	
	forloop(int i = 0; num; 2) {
		ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
		mesh.setWorldTransform(Mat4(rotateZ(360.0f * i / num) * translate(120.0f,0.0f,0.0f) * rotateZ(180.0f)));
		mesh.setMaterial(findMaterialByName(get_phong_mesh_material(i)),"*");
		mesh.setSurfaceProperty("surface_base","*");
	}
	
	engine.render.setMotionBlurVelocityScale(4.0f);
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	float s0 = sin(time);
	float c0 = cos(time);
	
	Player player = Unigine::getPlayer();
	player.setPosition(Vec3(s0 * 48.0f,c0 * 48.0f,16.0f));
	player.setDirection(vec3(c0,-s0,0.0f));
	
	forloop(int i = 0; meshes.size()) {
		float angle = 8.0f * PI * i / meshes.size() + 2.0f * time * i / meshes.size();
		float r = 20.0f + 80.0f * i / meshes.size();
		float s = sin(angle);
		float c = cos(angle);
		meshes[i].setWorldTransform(Mat4(translate(s * r,c * r,0.0f) * rotateZ(-angle * RAD2DEG + 180.0f)));
	}
	
	return 1;
}

/*
 */
int shutdown() {
	
	engine.console.run("render_motion_blur 0");
	engine.console.flush();
	
	return 1;
}

```

## motion_blur_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Body bodies[0];

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("Motion blur body rigid velocities");
	
	engine.console.run("render_motion_blur 1");
	
	int size = 5;
	
	for(int y = -size; y <= size; y++) {
		ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
		mesh.setMaterial(findMaterialByName(get_phong_mesh_material(y)),"*");
		mesh.setSurfaceProperty("surface_base","*");
		bodies.append(class_remove(new BodyRigid(mesh)));
	}
	
	updatePhysics();
	
	forloop(int i = 0; bodies.size()) {
		bodies[i].flushTransform();
	}
	
	engine.render.setMotionBlurVelocityScale(8.0f);
	
	return 1;
}

/*
 */
int updatePhysics() {
	
	float time = engine.game.getTime() - engine.physics.getCurrentSubframeTime();
	
	float offsets[] = ( 4.0f, 0.0f, -4.0f, -8.0f );
	float linears[] = ( 3.0f, -1.0f, 2.0f, 1.0f );
	float angulars[] = ( 0.0f, -2.3f, 1.7f, 1.0f );
	
	forloop(int i = 0; bodies.size()) {
		float offset = offsets[i % offsets.size()];
		float linear = time * linears[i % linears.size()];
		float angular = time * angulars[i % angulars.size()];
		bodies[i].setVelocityTransform(translate(Vec3(offset,i * 4.0f - 20.0f,12.0f + sin(linear) * 4.0f)) * rotateX(angular * RAD2DEG) * translate(0.0f,0.0f,-7.0f));
	}
	
	return 1;
}

/*
 */
int shutdown() {
	
	engine.console.run("render_motion_blur 0");
	engine.console.flush();
	
	return 1;
}

```

## mouse_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

Image cursor_0;
Image cursor_1;

float time = 0.0f;
int cursor = 0;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	setDescription("Hardware mouse");
	
	if(cursor_0 == NULL) {
		
		cursor_0 = new Image();
		cursor_1 = new Image();
		
		cursor_0.load(fullPath("uniginescript_samples/systems/textures/cursor_0.png"));
		cursor_1.load(fullPath("uniginescript_samples/systems/textures/cursor_1.png"));
		
		cursor_0.resize(64,64);
		cursor_1.resize(64,64);
		
		engine.input.setMouseCursorCustom(cursor_0);
	}
	
	return 1;
}

/*
 */
int update() {
	
	time += engine.getIFps();
	
	if(time > 0.5f) {
		time -= 0.5f;
		cursor = !cursor;
		if(cursor)
			engine.input.setMouseCursorCustom(cursor_1);
		else
			engine.input.setMouseCursorCustom(cursor_0);
	}
	
	return 1;
}

/*
 */
int shutdown() {
	
	engine.input.clearMouseCursorCustom();
	
	return 1;
}

```

## mouse_skin_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

void system_skin_clicked() {
	engine.input.setMouseCursorSkinSystem();
}

void default_skin_clicked() {
	engine.input.setMouseCursorSkinDefault();
}

void custom_skin_clicked() {
	Image image = new Image("uniginescript_samples/systems/textures/gui_mouse.png");
	engine.input.setMouseCursorSkinCustom(image);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	setDescription("Hardware mouse skins");
	
	custom_skin_clicked();
	
	Gui gui = engine.getGui();
	
	WidgetButton message_button = new WidgetButton(gui,"System skin");
	message_button.setPosition(100,100);
	message_button.getEventClicked().connect("system_skin_clicked");
	gui.addChild(message_button,GUI_ALIGN_OVERLAP);
	
	message_button = new WidgetButton(gui,"Default skin");
	message_button.setPosition(100,130);
	message_button.getEventClicked().connect("default_skin_clicked");
	gui.addChild(message_button,GUI_ALIGN_OVERLAP);
	
	message_button = new WidgetButton(gui,"Custom skin");
	message_button.setPosition(100,160);
	message_button.getEventClicked().connect("custom_skin_clicked");
	gui.addChild(message_button,GUI_ALIGN_OVERLAP);
	
	engine.input.setMouseHandle(MOUSE_HANDLE_SOFT);
	
	return 1;
}

/*
 */
int shutdown() {
	
	engine.input.setMouseHandle(MOUSE_HANDLE_GRAB);
	engine.input.setMouseCursorSkinDefault();
	
	return 1;
}

```

## node_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Node node;
WidgetSpriteNode sprite;
float angle = 0.0f;

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("WidgetSpriteNode");
	
	int size = 3;
	float space = 16.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			if(x == 0 && y == 0) node = mesh;
		}
	}
	
	return 1;
}

/*
 */
int update() {
	
	int size = 256;
	
	angle += engine.game.getIFps();
	mat4 projection = perspective(60.0f,1.0f,0.1f,100.0f);
	Mat4 modelview = lookAt(Vec3(14.0f),Vec3(0.0f,0.0f,8.0f),vec3(0.0f,0.0f,1.0f)) * rotateZ(angle * 32.0f);
	
	if(sprite == NULL) {
		sprite = new WidgetSpriteNode(engine.getGui(),size,size);
		engine.gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_RIGHT);
		sprite.setBlendFunc(GUI_BLEND_SRC_ALPHA,GUI_BLEND_ONE_MINUS_SRC_ALPHA);
		sprite.setTransform(scale(256.0f / size,256.0f / size,1.0f));
		sprite.setNode(node);
	}
	
	sprite.setProjection(projection);
	sprite.setModelview(modelview);
	
	return 1;
}

```

## nodes_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>
#include <core/systems/tracker/tracker.h>

using Unigine::Samples;
using Unigine::Tracker;

/*
 */
string mesh_material_names[] = ( "tracker_mesh_red", "tracker_mesh_green", "tracker_mesh_blue", "tracker_mesh_orange", "tracker_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
void update_track(Unigine::Tracker::TrackerTrack track) {
	
	if(track == NULL) return;
	
	float time = track.getMinTime();
	float min_time = track.getMinTime();
	float max_time = track.getMaxTime();
	float unit_time = track.getUnitTime();
	
	while(1) {
		
		// update time
		time += engine.game.getIFps() / unit_time;
		time = min_time + ((time - min_time)) % (max_time - min_time);
		
		// set track
		if(engine.game.isEnabled()) track.set(time);
		
		wait;
	}
}

/*
 */
void create_mesh_scene() {
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setName(format("statue_%d_%d",size + x,size + y));
		}
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create sample
	create_mesh_scene();
	
	Tracker tracker = new Tracker(TRACKER_CHECK_OBJECTS);
	TrackerTrack track = tracker.loadTrack(fullPath("uniginescript_samples/tracker/tracks/nodes_00.track"));
	
	thread("update_track",track);
	
	setDescription("Tracker Node parameters");
	
	return 1;
}

```

## nodes_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>
#include <core/systems/tracker/tracker.h>

using Unigine::Samples;
using Unigine::Tracker;

/*
 */
void update_track(Unigine::Tracker::TrackerTrack track) {
	
	if(track == NULL) return;
	
	float time = track.getMinTime();
	float min_time = track.getMinTime();
	float max_time = track.getMaxTime();
	float unit_time = track.getUnitTime();
	
	while(1) {
		
		// update time
		time += engine.game.getIFps() / unit_time;
		time = min_time + ((time - min_time)) % (max_time - min_time);
		
		// set track
		if(engine.game.isEnabled()) track.set(time);
		
		wait;
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create samples
	Tracker tracker = new Tracker(TRACKER_CHECK_OBJECTS);
	TrackerTrack track = tracker.loadTrack(fullPath("uniginescript_samples/tracker/tracks/nodes_01.track"));
	
	thread("update_track",track);
	
	setDescription("Tracker NodeReference parameters");
	
	return 1;
}

```

## nodes_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>
#include <core/systems/tracker/tracker.h>

using Unigine::Samples;
using Unigine::Tracker;

/*
 */
void update_track(Unigine::Tracker::TrackerTrack track) {
	
	if(track == NULL) return;
	
	float time = track.getMinTime();
	float min_time = track.getMinTime();
	float max_time = track.getMaxTime();
	float unit_time = track.getUnitTime();
	
	while(1) {
		
		// update time
		time += engine.game.getIFps() / unit_time;
		time = min_time + ((time - min_time)) % (max_time - min_time);
		
		// set track
		if(engine.game.isEnabled()) track.set(time);
		
		wait;
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create samples
	Tracker tracker = new Tracker(TRACKER_CHECK_OBJECTS);
	TrackerTrack track = tracker.loadTrack(fullPath("uniginescript_samples/tracker/tracks/nodes_02.track"));
	
	thread("update_track",track);
	
	setDescription("Tracker NodeReference parameters");
	
	return 1;
}

```

## noise_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

Image image;
WidgetSprite sprite;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	setDescription("Perlin noise generator");
	
	return 1;
}

/*
 */
int update() {
	
	int size = 64;
	
	if(image == NULL) {
		image = new Image();
		image.create2D(size,size,IMAGE_FORMAT_RGBA8);
		sprite = new WidgetSprite(engine.getGui());
		sprite.setImage(image);
		sprite.setTransform(scale(8.0f,8.0f,1.0f));
		sprite.arrange();
		engine.gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
		image.setChannelInt(3,255);
	}
	
	float time = engine.game.getTime();
	
	vec3 position;
	position.z = time;
	forloop(int y = 0; size) {
		position.y = y * 0.05f;
		forloop(int x = 0; size) {
			position.x = x * 0.05f;
			
			float n = engine.game.getNoise(position,vec3(32.0f,32.0f,32.0f),6);
			
			image.set2D(x,y,n,-n,n * n);
		}
	}
	
	sprite.setImage(image);
	
	return 1;
}

```

## noise_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

Info info;
Async async;
Image image;
WidgetSprite sprite;

/*
 */
void update_image(Image image) {
	
	int size = image.getWidth();
	float time = engine.game.getTime();
	
	vec3 position;
	position.z = time;
	forloop(int y = 0; size) {
		position.y = y * 0.05f;
		forloop(int x = 0; size) {
			position.x = x * 0.05f;
			float n = engine.game.getNoise(position,vec3(32.0f,32.0f,32.0f),6);
			image.set2D(x,y,n,-n,n * n);
		}
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	info = new Info();
	thread("update_thread");
	
	setDescription("Async Perlin noise generator");
	
	return 1;
}

/*
 */
void update_thread() {
	
	while(1) {
		int size = 128;
		
		if(async == NULL) async = new Async();
		
		if(image == NULL) {
			image = new Image();
			image.create2D(size,size,IMAGE_FORMAT_RGBA8);
			sprite = new WidgetSprite(engine.getGui());
			sprite.setImage(image);
			sprite.setTransform(scale(4.0f,4.0f,1.0f));
			sprite.arrange();
			engine.gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
			image.setChannelInt(3,255);
		}
		
		float update_time = clock();
		int update_frames = engine.game.getFrame();
		
		int id = async.run(functionid(update_image),image);
		while(async != NULL && async.isRunning(id)) wait;
		if(async == NULL) continue;
		
		update_time = clock() - update_time;
		update_frames = engine.game.getFrame() - update_frames;
		
		sprite.setImage(image);
		async.clearResult();
		
		info.set(format("Update : %.1fms %d frames",
			update_time * 1000.0f,update_frames));
	}
}

/*
 */
int shutdown() {
	
	if(async != NULL) async.wait();
	
	return 1;
}

```

## noise_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

Info info;
Async async_0;
Async async_1;
Async async_2;
Async async_3;
Image image;
WidgetSprite sprite;

/*
 */
template update_image<NUM> void update_image_ ## NUM(Image image,int y0,int y1) {
	
	int size = image.getWidth();
	float time = engine.game.getTime();
	
	vec3 position;
	position.z = time;
	forloop(int y = y0; y1) {
		position.y = y * 0.05f;
		forloop(int x = 0; size) {
			position.x = x * 0.05f;
			float n = engine.game.getNoise(position,vec3(32.0f,32.0f,32.0f),6);
			image.set2D(x,y,n,-n,n * n);
		}
	}
}

update_image<0>;
update_image<1>;
update_image<2>;
update_image<3>;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	info = new Info();
	thread("update_thread");
	
	setDescription("Async Perlin noise generator");
	
	return 1;
}

/*
 */
void update_thread() {
	
	while(1) {
		int size = 128;
		
		if(async_0 == NULL) async_0 = new Async();
		if(async_1 == NULL) async_1 = new Async();
		if(async_2 == NULL) async_2 = new Async();
		if(async_3 == NULL) async_3 = new Async();
		
		if(image == NULL) {
			image = new Image();
			image.create2D(size,size,IMAGE_FORMAT_RGBA8);
			sprite = new WidgetSprite(engine.getGui());
			sprite.setImage(image);
			sprite.setTransform(scale(4.0f,4.0f,1.0f));
			sprite.arrange();
			engine.gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
			image.setChannelInt(3,255);
		}
		
		float update_time = clock();
		int update_frames = engine.game.getFrame();
		
		int step = size / 4;
		async_0.run(functionid(update_image_0),image,step * 0,step * 1);
		async_1.run(functionid(update_image_1),image,step * 1,step * 2);
		async_2.run(functionid(update_image_2),image,step * 2,step * 3);
		async_3.run(functionid(update_image_3),image,step * 3,step * 4);
		
		while(async_0 != NULL && async_0.isRunning()) wait;
		while(async_1 != NULL && async_1.isRunning()) wait;
		while(async_2 != NULL && async_2.isRunning()) wait;
		while(async_3 != NULL && async_3.isRunning()) wait;
		if(async_0 == NULL || async_1 == NULL || async_2 == NULL || async_3 == NULL) continue;
		
		update_time = clock() - update_time;
		update_frames = engine.game.getFrame() - update_frames;
		
		sprite.setImage(image);
		async_0.clearResult();
		async_1.clearResult();
		async_2.clearResult();
		async_3.clearResult();
		
		info.set(format("Update : %.1fms %d frames",
				update_time * 1000.0f,update_frames));
	}
}

/*
 */
int shutdown() {
	
	if(async_0 != NULL) async_0.wait();
	if(async_1 != NULL) async_1.wait();
	if(async_2 != NULL) async_2.wait();
	if(async_3 != NULL) async_3.wait();
	
	return 1;
}

```

## oblique_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("Oblique Frustum");
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	Vec4 plane = Vec4(0.0f,0.0f,-1.0f,8.0f + sin(time) * 4.0f);
	
	Player player = engine.game.getPlayer();
	if(player != NULL) {
		player.setObliqueFrustum(1);
		player.setObliqueFrustumPlane(plane);
	}
	
	player = engine.editor.getPlayer();
	if(player != NULL) {
		player.setObliqueFrustum(1);
		player.setObliqueFrustumPlane(plane);
	}
	
	return 1;
}

/*
 */
int shutdown() {
	
	Player player = engine.game.getPlayer();
	if(player != NULL)
		player.setObliqueFrustum(0);
	
	player = engine.editor.getPlayer();
	if(player != NULL)
		player.setObliqueFrustum(0);
	
	return 1;
}

```

## obstacle_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
PathRoute route;
Obstacle obstacles[0];
Vec3 offset = Vec3(0.0f,0.0f,0.0f);
Vec3 p0 = Vec3(-60.0f,-60.0f,5.0f) + offset;
Vec3 p1 = Vec3( 60.0f, 60.0f,5.0f) + offset;
Vec3 p2 = Vec3(-60.0f, 60.0f,5.0f) + offset;
Vec3 p3 = Vec3( 60.0f,-60.0f,5.0f) + offset;

using Unigine::Samples;

/*
 */
int update() {
	
	float time = engine.game.getTime();
	forloop(int i = 0; obstacles.size()) {
		float angle = 360.0f * i / obstacles.size();
		obstacles[i].setWorldTransform(translate(offset) * rotateZ(angle + time * 4.0f) * translate(32.0f,0.0f,1.0f) * rotateX(time * 24.0f) * rotateY(time * 12.0f));
		obstacles[i].renderVisualizer();
	}
	
	route.create2D(p0,p1);
	if(route.isReached()) route.renderVisualizer(vec4_one);
	else engine.console.onscreenMessage("PathRoute failed\n");
	
	route.create2D(p2,p3);
	if(route.isReached()) route.renderVisualizer(vec4_one);
	else engine.console.onscreenMessage("PathRoute failed\n");
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("PathRoute2D in NavigationSector with obstacles");
	
	int num = 13;
	
	NavigationSector navigation = addToEditor(new NavigationSector(vec3(128.0f,128.0f,8.0f)));
	navigation.setWorldTransform(translate(Vec3(0.0f,0.0f,5.0f) + offset));
	
	forloop(int i = 0; num) {
		switch(i % 3) {
		case 0:
			obstacles.append(addToEditor(new ObstacleBox(vec3(4.0f,8.0f,12.0f))));
			break;
		case 1:
			obstacles.append(addToEditor(new ObstacleSphere(4.0f)));
			break;
		case 2:
			obstacles.append(addToEditor(new ObstacleCapsule(4.0f,12.0f)));
			break;
		}
	}
	
	engine.world.updateSpatial();
	
	route = new PathRoute();
	route.setRadius(2.0f);
	
	if(engine.visualizer.isEnabled() != 1) {
		engine.visualizer.setEnabled(1);
	}

	return 1;
}

/*
*/
int shutdown() {

	engine.console.setOnscreen(0);
	engine.visualizer.setEnabled(0);
	return 1;
}

```

## occluder_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	engine.console.run("render_show_occluder 1");
	
	int num = 1024;
	int size = 10;
	
	forloop(int i = 0; num) {
		WorldOccluder occluder = addToEditor(new WorldOccluder(vec3(0.0f,1.0f,2.0f)));
		occluder.setWorldTransform(Mat4(rotateZ(360.0f * 8.0f * i / num) * translate(16.0f,0.0f,16.0f * i / num)));
	}
	
	ObjectMeshDynamic reference = Unigine::createBox(vec3(1.0f));
	
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshDynamic mesh = node_cast(reference.clone());
				mesh.setWorldTransform(translate(Vec3(x,y,z + 11.0f) * 1.1f));
				mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y ^ z)),"*");
				mesh.setSurfaceProperty("surface_base","*");
			}
		}
	}
	
	delete reference;
	
	setDescription(format("%d World occluders",num));
	
	return 1;
}

/*
 */
int shutdown() {
	
	engine.console.run("render_show_occluder 0");
	engine.console.flush();
	
	return 1;
}

```

## occluder_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	engine.console.run("render_show_occluder 1");
	
	int num = 0;
	int size = 10;
	
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
				WorldOccluder occluder = addToEditor(new WorldOccluder(vec3(1.0f)));
				mesh.setWorldTransform(translate(Vec3(x,y,z + 11.0f)));
				mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y ^ z)),"*");
				mesh.setSurfaceProperty("surface_base","*");
				mesh.addChild(occluder);
				num++;
			}
		}
	}
	
	setDescription(format("%d World occluders",num));
	
	return 1;
}

/*
 */
int shutdown() {
	
	engine.console.run("render_show_occluder 0");
	engine.console.flush();
	
	return 1;
}

```

## occluder_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	engine.console.run("render_show_occluder 1");
	
	int num = 0;
	int size = 4;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			WorldOccluderMesh occluder = addToEditor(new WorldOccluderMesh(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.addChild(occluder);
			num++;
		}
	}
	
	setDescription(format("%d WorldOccluderMeshes",num));
	
	return 1;
}

/*
 */
int shutdown() {
	
	engine.console.run("render_show_occluder 0");
	engine.console.flush();
	
	return 1;
}

```

## occluder_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	engine.console.run("render_show_occluder 1");
	
	int num = 0;
	int size = 64;
	float step = 16.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			
			float X = x * step + engine.game.getRandom(0.0f,step);
			float Y = y * step + engine.game.getRandom(0.0f,step);
			vec3 size = engine.game.getRandom(vec3(4.0f,4.0f,8.0f),vec3(8.0f,8.0f,32.0f));
			
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setTransform(Mat4(scale(size)));
			
			WorldOccluder occluder = addToEditor(new WorldOccluder(size));
			occluder.setWorldTransform(translate(Vec3(X,Y,size.z * 0.5f)));
			occluder.setDistance(100.0f);
			occluder.addChild(mesh);
			
			num++;
		}
	}
	
	setDescription(format("%d World occluders",num));
	
	return 1;
}

/*
 */
int shutdown() {
	
	engine.console.run("render_show_occluder 0");
	engine.console.flush();
	
	return 1;
}

```

## occlusion_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("Screen Space Ambient Occlusion");
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	return 1;
}

/*
 */
int update() {
	
	engine.render.setSSAO((int(engine.game.getTime()) % 2) == 0);
	
	return 1;
}

```

## omni_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
void update_light(LightOmni omni) {
	
	float size = 15.0f;
	Vec3 position = Vec3(engine.game.getRandom(-size,size),engine.game.getRandom(-size,size),0.5f);
	size += 1.0f;
	
	float angle = engine.game.getRandom(-PI,PI);
	Vec3 velocity = Vec3(sin(angle),cos(angle),0.0f) * engine.game.getRandom(1.0f,2.0f);
	
	while(1) {
		if(abs(position.x) > size) velocity.x = -velocity.x;
		if(abs(position.y) > size) velocity.y = -velocity.y;
		position += velocity * engine.game.getIFps();
		omni.setWorldTransform(translate(position));
		wait;
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	Node environment = engine.world.getNodeByName("world_environment");
	environment.setEnabled(0);
	
	int num = 0;
	int size = 24;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			vec3 color = engine.game.getRandom(vec3_zero,vec3_one);
			LightOmni omni = addToEditor(new LightOmni(vec4(color,1.0f),2.0f));
			omni.setShadow(0);
			thread("update_light",omni);
			num++;
		}
	}
	
	setDescription(format("%d Omni lights",num));
	
	return 1;
}

```

## packages_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	string name = fullPath("uniginescript_samples/systems/packages/package._ung_");
	if(engine.filesystem.loadPackage(name,"ung")) {
		
		string names[0];
		engine.filesystem.getPackageVirtualFiles(name,"ung",names);
		
		foreach(string name; names) {
			log.message("%s\n",name);
		}
		
		engine.filesystem.removePackage(name);
	}
	
	setDescription("FileSystem UNG packages");
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## packages_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	string name = fullPath("uniginescript_samples/systems/packages/package._zip_");
	if(engine.filesystem.loadPackage(name,"zip")) {
		
		string names[0];
		engine.filesystem.getPackageVirtualFiles(name,"zip",names);
		
		foreach(string name; names) {
			log.message("%s\n",name);
		}
		
		engine.filesystem.removePackage(name);
	}
	
	setDescription("FileSystem ZIP packages");
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## particles_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create sample
	engine.render.setEnvironment(0);
	
	int num = 0;
	int size = 10;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectParticles particles = addToEditor(new ObjectParticles());
			particles.setWorldTransform(translate(Vec3(x,y,0.0f) * 6.0f));
			particles.setMaterial(findMaterialByName("stress_particles"),"*");
			particles.setParticlesType(OBJECT_PARTICLES_TYPE_POINT);
			particles.setSpawnRate(2000.0f);
			particles.setEmitterEnabled(1);
			particles.setLife(1.0f,0.0f);
			
			ParticleModifierScalar radius_modifier = particles.getRadiusOverTimeModifier();
			radius_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
			radius_modifier.setConstant(0.1f);
			
			ParticleModifierScalar velocity_modifier = particles.getVelocityOverTimeModifier();
			velocity_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
			velocity_modifier.setConstant(3.0f);
			
			num++;
		}
	}
	
	setDescription(format("%d point ObjectParticles",num));
	
	return 1;
}

```

## particles_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create sample
	engine.render.setEnvironment(0);
	
	int num = 0;
	int size = 10;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectParticles particles = addToEditor(new ObjectParticles());
			particles.setWorldTransform(translate(Vec3(x,y,0.0f) * 6.0f));
			particles.setMaterial(findMaterialByName("stress_particles"),"*");
			particles.setParticlesType(OBJECT_PARTICLES_TYPE_BILLBOARD);
			particles.setSpawnRate(2000.0f);
			particles.setEmitterEnabled(1);
			particles.setLife(1.0f,0.0f);

			ParticleModifierVector gravity_modifier = particles.getGravityOverTimeModifier();
			gravity_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
			gravity_modifier.setConstant(vec3(0.0f));
			
			ParticleModifierScalar radius_modifier = particles.getRadiusOverTimeModifier();
			radius_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
			radius_modifier.setConstant(0.1f);
			
			ParticleModifierScalar velocity_modifier = particles.getVelocityOverTimeModifier();
			velocity_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
			velocity_modifier.setConstant(3.0f);
			
			num++;
		}
	}
	
	setDescription(format("%d billboard ObjectParticles",num));
	
	return 1;
}

```

## particles_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create sample
	engine.render.setEnvironment(0);
	
	int num = 0;
	int size = 10;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectParticles particles = addToEditor(new ObjectParticles());
			particles.setWorldTransform(translate(Vec3(x,y,0.0f) * 6.0f));
			particles.setMaterial(findMaterialByName("stress_particles"),"*");
			particles.setParticlesType(OBJECT_PARTICLES_TYPE_FLAT);
			particles.setSpawnRate(2000.0f);
			particles.setEmitterEnabled(1);
			particles.setLife(1.0f,0.0f);
	
			ParticleModifierVector gravity_modifier = particles.getGravityOverTimeModifier();
			gravity_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
			gravity_modifier.setConstant(vec3(0.0f));
			
			ParticleModifierScalar radius_modifier = particles.getRadiusOverTimeModifier();
			radius_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
			radius_modifier.setConstant(0.1f);
			
			ParticleModifierScalar velocity_modifier = particles.getVelocityOverTimeModifier();
			velocity_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
			velocity_modifier.setConstant(3.0f);
			
			num++;
		}
	}
	
	setDescription(format("%d flat ObjectParticles",num));
	
	return 1;
}

```

## particles_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create sample
	engine.render.setEnvironment(0);
	
	int num = 0;
	int size = 10;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectParticles particles = addToEditor(new ObjectParticles());
			particles.setWorldTransform(translate(Vec3(x,y,0.0f) * 6.0f));
			particles.setMaterial(findMaterialByName("stress_particles"),"*");
			particles.setParticlesType(OBJECT_PARTICLES_TYPE_LENGTH);
			particles.setSpawnRate(2000.0f);
			particles.setEmitterEnabled(1);
			particles.setLife(1.0f,0.0f);
			
			ParticleModifierScalar radius_modifier = particles.getRadiusOverTimeModifier();
			radius_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
			radius_modifier.setConstant(0.1f);
			
			ParticleModifierScalar velocity_modifier = particles.getVelocityOverTimeModifier();
			velocity_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
			velocity_modifier.setConstant(3.0f);
			
			num++;
		}
	}
	
	setDescription(format("%d length ObjectParticles",num));
	
	return 1;
}

```

## path_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
BodyPath path;
PhysicalForce force;

/*
 */
void update() {
	updateSamplePhysics();
	
	path.renderVisualizer();
	force.setRotator(cos(engine.game.getTime() * 0.1f) * 60.0f);
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	if(0) {
		int num = 256;
		Path path = new Path();
		forloop(int i = 0; num + 1) {
			float k = float(i) / num;
			float angle = k * 360.0f * 5.0f;
			float radius = 12.0f + cos(k * PI * 3.0f) * sin(k * PI * 3.0f) * 10.0f;
			float height = 8.0f + pow(cos(k * PI),2.0f) * 6.0f;
			int frame = path.addFrame();
			path.setFrameTime(frame,k);
			path.setFrameTransform(frame,rotateZ(angle) * translate(radius,0.0f,height));
		}
		path.save(fullPath("uniginescript_samples/joints/paths/path_00.path"));
		delete path;
	}
	
	ObjectDummy dummy = addToEditor(new ObjectDummy());
	path = class_remove(new BodyPath(dummy));
	path.setPathName(fullPath("uniginescript_samples/joints/paths/path_00.path"));
	
	int num = 32;
	forloop(int i = 0; num) {
		Body body = createBodyBox(vec3(1.0f),1,0.5f,0.5f,get_material(0),translate(engine.game.getRandom(Vec3(-20.0f,-20.0f,10.0f),Vec3(20.0f,20.0f,20.0f))));
		JointPath joint = class_remove(new JointPath(body,path));
		joint.setLinearRestitution(0.75f);
		joint.setLinearDamping(0.5f);
		joint.setNumIterations(2);
	}
	
	force = addToEditor(new PhysicalForce(100.0f));
	
	setDescription(format("%d JointPath",num));
	
	engine.visualizer.setEnabled(1);
	
	return 1;
}

```

## persecutor_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectMeshStatic object;
PlayerPersecutor persecutor;

using Unigine::Samples;

/*
 */
string material_names[] = ( "players_red", "players_green", "players_blue", "players_orange", "players_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	object.setWorldTransform(translate(Vec3(sin(time),cos(time),0.7f) * 8.0f) * rotateZ(-time * RAD2DEG) *
		rotateX(sin(time * 4.0f) * 8.0f) * rotateY(sin(time * 3.0f) * 4.0f));
	
	persecutor.setPhiAngle(persecutor.getPhiAngle() + sin(time * 10.0f) * 0.1f);
	persecutor.setThetaAngle(persecutor.getThetaAngle() + cos(time * 10.0f) * 0.1f);
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlane();
	setDescription("Persecutor");
	
	int size = 2;
	float space = 4.0f;
	
	string mesh_path = fullPath("uniginescript_samples/players/meshes/sphere_00.mesh");
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(mesh_path));
			mesh.setWorldTransform(translate(Vec3(x,y,1.0f) * space));
			mesh.setMaterial(findMaterialByName(get_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	object = addToEditor(new ObjectMeshStatic(mesh_path));
	object.setMaterial(findMaterialByName(get_material(0)),"*");
	object.setSurfaceProperty("surface_base","*");
	
	ObjectMeshStatic mesh_0 = addToEditor(new ObjectMeshStatic(mesh_path));
	mesh_0.setWorldTransform(translate(Vec3(-2.0f,-1.0f,0.0f)));
	mesh_0.setMaterial(findMaterialByName(get_material(1)),"*");
	mesh_0.setSurfaceProperty("surface_base","*");
	object.addChild(mesh_0);
	
	ObjectMeshStatic mesh_1 = addToEditor(new ObjectMeshStatic(mesh_path));
	mesh_1.setWorldTransform(translate(Vec3(-2.0f,1.0f,0.0f)));
	mesh_1.setMaterial(findMaterialByName(get_material(1)),"*");
	mesh_1.setSurfaceProperty("surface_base","*");
	object.addChild(mesh_1);
	
	persecutor = addToEditor(new PlayerPersecutor());
	persecutor.setTarget(object);
	persecutor.setMinDistance(2.0f);
	persecutor.setMaxDistance(6.0f);
	persecutor.setMinThetaAngle(20.0f);
	persecutor.setMaxThetaAngle(60.0f);
	engine.game.setPlayer(persecutor);
	
	return 1;
}

```

## persecutor_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectMeshStatic object;
PlayerPersecutor persecutor;

using Unigine::Samples;

/*
 */
string material_names[] = ( "players_red", "players_green", "players_blue", "players_orange", "players_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	object.setWorldTransform(translate(Vec3(sin(time),cos(time),0.7f) * 8.0f) * rotateZ(-time * RAD2DEG) *
		rotateX(sin(time * 4.0f) * 8.0f) * rotateY(sin(time * 3.0f) * 4.0f));
	
	persecutor.setPhiAngle(persecutor.getPhiAngle() + sin(time * 10.0f) * 0.1f);
	persecutor.setThetaAngle(persecutor.getThetaAngle() + cos(time * 10.0f) * 0.1f);
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlane();
	setDescription("Fixed Persecutor");
	
	int size = 2;
	float space = 4.0f;
	
	string mesh_path = fullPath("uniginescript_samples/players/meshes/sphere_00.mesh");
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(mesh_path));
			mesh.setWorldTransform(translate(Vec3(x,y,1.0f) * space));
			mesh.setMaterial(findMaterialByName(get_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	object = addToEditor(new ObjectMeshStatic(mesh_path));
	object.setMaterial(findMaterialByName(get_material(0)),"*");
	object.setSurfaceProperty("surface_base","*");
	
	ObjectMeshStatic mesh_0 = addToEditor(new ObjectMeshStatic(mesh_path));
	mesh_0.setWorldTransform(translate(Vec3(-2.0f,-1.0f,0.0f)));
	mesh_0.setMaterial(findMaterialByName(get_material(1)),"*");
	mesh_0.setSurfaceProperty("surface_base","*");
	object.addChild(mesh_0);
	
	ObjectMeshStatic mesh_1 = addToEditor(new ObjectMeshStatic(mesh_path));
	mesh_1.setWorldTransform(translate(Vec3(-2.0f,1.0f,0.0f)));
	mesh_1.setMaterial(findMaterialByName(get_material(1)),"*");
	mesh_1.setSurfaceProperty("surface_base","*");
	object.addChild(mesh_1);
	
	persecutor = addToEditor(new PlayerPersecutor());
	persecutor.setFixed(1);
	persecutor.setTarget(object);
	persecutor.setMinDistance(2.0f);
	persecutor.setMaxDistance(6.0f);
	persecutor.setMinThetaAngle(20.0f);
	persecutor.setMaxThetaAngle(60.0f);
	engine.game.setPlayer(persecutor);
	
	return 1;
}

```

## pivot_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
float geodetic_tile_size = 40000.0f;
int geodetic_grid_radius = 1;

dvec3 tomsk_origin = dvec3(58.49771,84.97437,117.0);

/*
 */
class GeodeticTile {
	private:
		
		NodeDummy root;
		ObjectMeshDynamic ground;
		ObjectGrass grass;
		WorldClutter near_lod;
		ObjectMeshClutter far_lod;
		
	public:
		
		//
		GeodeticTile(int x,int y,GeodeticPivot pivot) {
			int seed = engine.game.getRandom(0,0x0fffffff);
			
			Mat4 offset = Mat4(translate(-geodetic_tile_size * 0.5f,-geodetic_tile_size * 0.5f,0.0f));
			
			ground = addToEditor(new ObjectMeshDynamic(fullPath("uniginescript_samples/geodetics/meshes/plane_40km.mesh")));
			ground.setName("ground");
			ground.setMaterial(findMaterialByName("mesh_base"),"*");
			ground.setSurfaceProperty("surface_base","*");
			
			grass = addToEditor(new ObjectGrass());
			grass.setName("grass");
			grass.setOrientation(1);
			grass.setIntersection(1);
			grass.setSizeX(geodetic_tile_size);
			grass.setSizeY(geodetic_tile_size);
			grass.setStep(100.0f);
			grass.setDensity(0.03f);
			grass.setSeed(seed);
			grass.setTransform(offset);
			
			forloop(int i = 0; grass.getNumSurfaces()) {
				grass.setMaxVisibleDistance(100.0f,i);
				grass.setMaxFadeDistance(100.0f,i);
				grass.setMaterial(findMaterialByName("grass_base"),i);
				grass.setSurfaceProperty("surface_base",i);
			}
			
			near_lod = addToEditor(new WorldClutter());
			near_lod.setName("near_lod");
			near_lod.addReference(fullPath("uniginescript_samples/geodetics/nodes/platform.node"));
			near_lod.setVisibleDistance(500.0f);
			near_lod.setSizeX(geodetic_tile_size);
			near_lod.setSizeY(geodetic_tile_size);
			near_lod.setStep(1000.0f);
			near_lod.setDensity(0.0001f);
			near_lod.setSeed(seed);
			near_lod.setTransform(offset);
			
			far_lod = addToEditor(new ObjectMeshClutter(fullPath("uniginescript_samples/geodetics/meshes/platform.mesh")));
			far_lod.setName("far_lod");
			far_lod.setVisibleDistance(5000.0f);
			far_lod.setFadeDistance(80.0f);
			far_lod.setSizeX(geodetic_tile_size);
			far_lod.setSizeY(geodetic_tile_size);
			far_lod.setStep(1000.0f);
			far_lod.setDensity(0.0001f);
			far_lod.setSeed(seed);
			far_lod.setTransform(offset);
			
			forloop(int i = 0; far_lod.getNumSurfaces()) {
				far_lod.setMinVisibleDistance(500.0f,i);
				far_lod.setMaterial(findMaterialByName("mesh_base"),i);
				far_lod.setSurfaceProperty("surface_base",i);
			}
			
			root = addToEditor(new NodeDummy());
			root.setName(format("tile_%2d_%2d",x,y));
			
			root.addChild(ground);
			root.addChild(grass);
			root.addChild(near_lod);
			root.addChild(far_lod);
			
			root.setParent(pivot);
			root.setTransform(Mat4(translate(geodetic_tile_size * x,geodetic_tile_size * y,0.0f)));
		}
		
		void updateTopology() {
			GeodeticPivot pivot = node_cast(root.getParent());
			
			Mesh mesh = new Mesh();
			ground.getMesh(mesh);
			
			Mat4 flat_transform = pivot.getIWorldTransform() * ground.getWorldTransform();
			Mat4 ellipsoid_transform = pivot.mapMeshFlatToEllipsoid(mesh,flat_transform);
			
			// Note: by default setMesh call will create additional internal mesh resource.
			//       it'll allow us to map each mesh on the ellipsoid individually
			ground.setMesh(mesh);
			ground.setWorldTransform(pivot.getWorldTransform() * ellipsoid_transform);
			
			delete mesh;
		}
		
		void clearTopology() {
			GeodeticPivot pivot = node_cast(root.getParent());
			
			Mesh mesh = new Mesh();
			ground.getMesh(mesh);
			
			Mat4 ellipsoid_transform = pivot.getIWorldTransform() * ground.getWorldTransform();
			Mat4 flat_transform = pivot.mapMeshEllipsoidToFlat(mesh,ellipsoid_transform);
			
			// Note: by default setMesh call will create additional internal mesh resource.
			//       it'll allow us to map each mesh on the ellipsoid individually
			ground.setMesh(mesh);
			ground.setWorldTransform(pivot.getWorldTransform() * flat_transform);
			
			delete mesh;
		}
};

/*
 */
using Unigine::Samples;

/*
 */
GeodeticPivot pivot;
GeodeticTile tiles[0];

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f),20.0f,200.0f);
	
	setDescription("GeodeticPivot ellipsoid placement");
	
	pivot = addToEditor(new GeodeticPivot());
	
	Ellipsoid ellipsoid = pivot.getEllipsoid();
	ellipsoid.setSemimajorAxis(80000.0f);
	ellipsoid.setMode(ELLIPSOID_MODE_FAST);
	
	pivot.setOrigin(tomsk_origin);
	pivot.setEllipsoid(ellipsoid);
	
	for(int x = -geodetic_grid_radius; x <= geodetic_grid_radius; x++) {
		for(int y = -geodetic_grid_radius; y <= geodetic_grid_radius; y++) {
			tiles.append(new GeodeticTile(x,y,pivot));
		}
	}
	
	tiles.call(functionid(GeodeticTile::updateTopology));
	
	return 1;
}

/*
 */
int update() {
	return 1;
}

```

## player_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>
#include <core/systems/tracker/tracker.h>

using Unigine::Samples;
using Unigine::Tracker;

/*
 */
string mesh_material_names[] = ( "tracker_mesh_red", "tracker_mesh_green", "tracker_mesh_blue", "tracker_mesh_orange", "tracker_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
void update_track(Unigine::Tracker::TrackerTrack track) {
	
	if(track == NULL) return;
	
	float time = track.getMinTime();
	float min_time = track.getMinTime();
	float max_time = track.getMaxTime();
	float unit_time = track.getUnitTime();
	
	while(1) {
		
		// update time
		time += engine.game.getIFps() / unit_time;
		time = min_time + ((time - min_time)) % (max_time - min_time);
		
		// set track
		if(engine.game.isEnabled()) track.set(time);
		
		wait;
	}
}

/*
 */
void create_mesh_scene() {
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setName(format("statue_%d_%d",size + x,size + y));
		}
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create sample
	create_mesh_scene();
	
	Tracker tracker = new Tracker(TRACKER_CHECK_OBJECTS);
	TrackerTrack track = tracker.loadTrack(fullPath("uniginescript_samples/tracker/tracks/player_00.track"));
	
	thread("update_track",track);
	
	setDescription("Tracker Player parameters");
	
	return 1;
}

```

## player_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>
#include <core/systems/tracker/tracker.h>

using Unigine::Samples;
using Unigine::Tracker;

/*
 */
string mesh_material_names[] = ( "tracker_mesh_red", "tracker_mesh_green", "tracker_mesh_blue", "tracker_mesh_orange", "tracker_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
void update_track(Unigine::Tracker::TrackerTrack track) {
	
	if(track == NULL) return;
	
	float time = track.getMinTime();
	float min_time = track.getMinTime();
	float max_time = track.getMaxTime();
	float unit_time = track.getUnitTime();
	
	while(1) {
		
		// update time
		time += engine.game.getIFps() / unit_time;
		time = min_time + ((time - min_time)) % (max_time - min_time);
		
		// set track
		if(engine.game.isEnabled()) track.set(time);
		
		wait;
	}
}

/*
 */
void create_mesh_scene() {
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setName(format("statue_%d_%d",size + x,size + y));
		}
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create sample
	create_mesh_scene();
	
	Tracker tracker = new Tracker(TRACKER_CHECK_OBJECTS);
	TrackerTrack track = tracker.loadTrack(fullPath("uniginescript_samples/tracker/tracks/player_01.track"));
	
	thread("update_track",track);
	
	setDescription("Tracker Player parameters");
	
	return 1;
}

```

## player_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>
#include <core/systems/tracker/tracker.h>

using Unigine::Samples;
using Unigine::Tracker;

/*
 */
string mesh_material_names[] = ( "tracker_mesh_red", "tracker_mesh_green", "tracker_mesh_blue", "tracker_mesh_orange", "tracker_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
void update_track(Unigine::Tracker::TrackerTrack track) {
	
	if(track == NULL) return;
	
	float time = track.getMinTime();
	float min_time = track.getMinTime();
	float max_time = track.getMaxTime();
	float unit_time = track.getUnitTime();
	
	while(1) {
		
		// update time
		time += engine.game.getIFps() / unit_time;
		time = min_time + ((time - min_time)) % (max_time - min_time);
		
		// set track
		if(engine.game.isEnabled()) track.set(time);
		
		wait;
	}
}

/*
 */
void create_mesh_scene() {
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setName(format("statue_%d_%d",size + x,size + y));
		}
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create sample
	create_mesh_scene();
	
	Tracker tracker = new Tracker(TRACKER_CHECK_OBJECTS);
	TrackerTrack track = tracker.loadTrack(fullPath("uniginescript_samples/tracker/tracks/player_02.track"));
	
	thread("update_track",track);
	
	setDescription("Tracker Player parameters");
	
	return 1;
}

```

## post_selection_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
ObjectMeshStatic meshes[0];
int num = 0;
float interval = 0.0f;

/*
 */
string phong_mesh_material_names[] = ( "shaders_phong_mesh_red", "shaders_phong_mesh_green", "shaders_phong_mesh_blue", "shaders_phong_mesh_orange", "shaders_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	Player player = createDefaultPlayer(Vec3(12.0f,0.0f,4.0f));
	createDefaultPlane();
	
	setDescription("Object selection");
	
	player.setDirection(vec3(-1.0f,0.0f,-0.25f));
	
	engine.console.run("render_auxiliary 1");
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(scale(vec3(0.25f)) * translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			meshes.append(mesh);
		}
	}
	
	engine.render.addScriptableMaterial(findMaterialByName("post_filter_selection"));
	
	return 1;
}

/*
 */
int update() {
	
	interval -= engine.game.getIFps();
	if (interval > 0.0f)
		return 1;
	
	interval += 0.25f;
	
	meshes[num].setMaterialState("auxiliary",0,0);
	meshes[num].setMaterialParameterFloat4("auxiliary_color",vec4_one,0);
	
	num = engine.game.getRandom(0,meshes.size());
	
	meshes[num].setMaterialState("auxiliary",1,0);
	meshes[num].setMaterialParameterFloat4("auxiliary_color",engine.game.getRandom(vec4(0.0f),vec4(1.0f,1.0f,1.0f,0.0f)),0);
	
	return 1;
}

```

## prismatic_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
Joint joints[0];

/*
 */
void create_joint_0(Body b0,Body b1) {
	JointPrismatic j = class_remove(new JointPrismatic(b0,b1));
	j.setWorldAxis(vec3(0.0f,0.0f,1.0f));
	j.setLinearRestitution(0.4f);
	j.setAngularRestitution(0.4f);
	j.setLinearSoftness(0.4f);
	j.setAngularSoftness(0.4f);
	j.setLinearDamping(4.0f);
	j.setLinearLimitFrom(-1.25f);
	j.setLinearLimitTo(1.25f);
	j.setNumIterations(16);
	joints.append(j);
}

void create_joint_1(Body b0,Body b1) {
	JointPrismatic j = class_remove(new JointPrismatic(b0,b1));
	j.setWorldAxis(vec3(0.0f,0.0f,1.0f));
	j.setLinearRestitution(0.4f);
	j.setAngularRestitution(0.4f);
	j.setLinearSoftness(0.4f);
	j.setAngularSoftness(0.4f);
	j.setLinearDamping(8.0f);
	j.setLinearLimitFrom(-1.25f);
	j.setLinearLimitTo(1.25f);
	j.setNumIterations(16);
	joints.append(j);
}

void create_joint_2(Body b0,Body b1) {
	JointPrismatic j = class_remove(new JointPrismatic(b0,b1));
	j.setWorldAxis(vec3(0.0f,0.0f,1.0f));
	j.setLinearRestitution(0.4f);
	j.setAngularRestitution(0.4f);
	j.setLinearSoftness(0.4f);
	j.setAngularSoftness(0.4f);
	j.setLinearDamping(16.0f);
	j.setLinearLimitFrom(-1.25f);
	j.setLinearLimitTo(1.25f);
	j.setNumIterations(16);
	joints.append(j);
}

/*
 */
void create_body(string func,Mat4 transform) {
	
	float offset = 1.2f;
	
	Body b0 = createBodyBox(vec3(1.0f),0.0f,0.5f,0.5f,get_material(0),transform * translate(0.0f,0.0f,offset * 0.0f));
	Body b1 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 2.0f));
	Body b2 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 4.0f));
	Body b3 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,offset * 6.0f));
	
	call(func,b0,b1);
	call(func,b1,b2);
	call(func,b2,b3);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("JointPrismatic");
	
	create_body("create_joint_0",translate(Vec3(0.0f,-10.0f,10.0f)));
	createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.0f,-10.0f,20.0f)));
	
	create_body("create_joint_1",translate(Vec3(0.0f,0.0f,10.0f)));
	createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.0f,0.0f,20.0f)));
	
	create_body("create_joint_2",translate(Vec3(0.0f,10.0f,10.0f)));
	createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.0f,10.0f,20.0f)));
	
	engine.visualizer.setEnabled(1);
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	
	foreach(Joint j; joints)
		j.renderVisualizer(vec4_one);
	
	return 1;
}

```

## prismatic_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
Joint joints[0];

/*
 */
void create_body(int size,Mat4 transform) {
	
	float offset = 1.0f;
	
	Body b0 = createBodyBox(vec3(1.0f),0.0f,0.5f,0.5f,get_material(0),transform);
	Body b1 = createBodyBox(vec3(1.0f),10.0f,0.5f,0.5f,get_material(1),transform * translate(vec3(0.0f,0.0f,1.0f)));
	
	JointHinge j = class_remove(new JointHinge(b0,b1));
	j.setWorldAnchor(transform * vec3_zero);
	j.setWorldAxis(vec3(1.0f,0.0f,0.0f));
	j.setLinearRestitution(0.2f);
	j.setAngularRestitution(0.2f);
	j.setLinearSoftness(0.0f);
	j.setAngularSoftness(0.0f);
	j.setNumIterations(32);
	j.setAngularVelocity(1.0f);
	j.setAngularTorque(10000.0f);
	
	b0 = b1;
	
	forloop(int i = 1; size + 1) {
		
		b1 = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(1),transform * translate(vec3(0.0f,0.0f,1.0f + i * offset)));
		
		JointPrismatic j = class_remove(new JointPrismatic(b0,b1));
		j.setWorldAxis(vec3(0.0f,0.0f,1.0f));
		j.setLinearLimitFrom(-1.0f);
		j.setLinearLimitTo(0.0f);
		j.setLinearRestitution(0.2f);
		j.setAngularRestitution(0.2f);
		j.setLinearSoftness(0.2f);
		j.setAngularSoftness(0.2f);
		j.setNumIterations(32);
		joints.append(j);
		
		b0 = b1;
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	create_body(4,translate(Vec3(0.0f,-10.0f,10.0f)));
	create_body(4,translate(Vec3(0.0f,  0.0f,10.0f)));
	create_body(4,translate(Vec3(0.0f, 10.0f,10.0f)));
	
	return "JointPrismatic";
}

/*
 */
int update() {
	updateSamplePhysics();
	
	foreach(Joint j; joints)
		j.renderVisualizer(vec4_one);
	
	return 1;
}

```

## procedural_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
Object objects[0];
Image image;

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("Procedural textures");
	
	forloop(int i = 0; 3) {
		Object object = addToEditor(Unigine::createBox(vec3(12.0f)));
		object.setWorldTransform(translate(Vec3(0.0f,-16.0f + i * 16.0f,10.0f)));
		object.setMaterial(findMaterialByName("materials_procedural"),"*");
		object.setSurfaceProperty("surface_base","*");
		objects.append(object);
	}
	
	thread([]() {
		while(1) {
			if(image == NULL) image = new Image(fullPath("uniginescript_samples/materials/textures/procedural_d.png"));
			
			int width = image.getWidth();
			int height = image.getHeight();
			int width_1 = width - 1;
			
			forloop(int i = 0; 64) {
				image.set2D(rand(0,width),height - 1,vec4(rand(0.0f,1.0f)) * vec4(1.0f,0.4f,0.0f,0.0f));
			}
			
			forloop(int i = 0; 256) {
				int x = rand(1,width - 1);
				int y = rand(height - 16,height);
				vec4 color = image.get2D(x,y);
				image.set2D(x,y,color + vec4(1.0f,0.4f,0.0f,0.0f) * rand(0.0f,0.5f));
			}
			
			for(int y = 0, y1 = 1; y < height - 1; y++, y1++) {
				forloop(int x = 1; width_1) {
					image.set2D(x,y,(image.get2D(x - 1,y1) + image.get2D(x,y1) + image.get2D(x + 1,y1)) / 3.1f);
				}
			}
			
			Material material = objects[1].getMaterialInherit(0);
			material.setTextureImage(material.findTexture("diffuse"),image);
			
			sleep(1.0f / 25.0f);
		}
	});
	
	return 1;
}

```

## procedural_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
//#define ASYNC

/*
 */
Async async;
ObjectVolumeBox object;
Material material;

using Unigine::Samples;

/*
 */
int update_thread() {
	
	Image image;
	
	float size = 2.0f;
	float velocity = 1.0f;
	float radius = 0.5f;
	
	int num_fields = 16;
	vec3 positions[num_fields];
	vec3 velocities[num_fields];
	float radiuses[num_fields];
	
	for(int i = 0; i < num_fields; i++) {
		positions[i].x = rand(0.0f,size);
		positions[i].y = rand(0.0f,size);
		positions[i].z = rand(0.0f,size);
		velocities[i].x = rand(-velocity,velocity);
		velocities[i].y = rand(-velocity,velocity);
		velocities[i].z = rand(-velocity,velocity);
		radiuses[i] = rand(radius / 2.0f,radius);
	}
	
	while(1) {
		
		if(image == NULL) {
			image = new Image();
			image.create3D(32,32,32,IMAGE_FORMAT_RGBA8);
		}
		
		float ifps = engine.game.getIFps();
		
		for(int i = 0; i < num_fields; i++) {
			vec3 p = positions[i] + velocities[i] * ifps;
			if(p.x < 0.0f || p.x > size) velocities[i].x = -velocities[i].x;
			if(p.y < 0.0f || p.y > size) velocities[i].y = -velocities[i].y;
			if(p.z < 0.0f || p.z > size) velocities[i].z = -velocities[i].z;
			positions[i] += velocities[i] * ifps;
		}
		
		#ifdef ASYNC
			if(async == NULL) async = new Async();
			if(async.getQueue() < 1) {
				async.clearResult();
				async.run([](Image image,int num_fields) {
		#endif
		
		vec3 position;
		int width = image.getWidth();
		int height = image.getHeight();
		int depth = image.getDepth();
		float iwidth = size / width;
		float iheight = size / height;
		float idepth = size / depth;
		forloop(int z = 0; depth) {
			position.z = z * idepth;
			forloop(int y = 0; height) {
				position.y = y * iheight;
				forloop(int x = 0; width) {
					position.x = x * iwidth;
					float field = 0.0f;
					forloop(int i = 0; num_fields) {
						float distance = length2(positions[i] - position);
						if(distance < radiuses[i]) field += radiuses[i] - distance;
					}
					if(field > 1.0f) field = 1.0f;
					image.set3D(x,y,z,field,field,field,field);
				}
			}
		}
		
		#ifdef ASYNC
				},image,num_fields);
			}
		#endif
		
		int index = material.findTexture("density_3d");
		material.setTextureImage(index,image);
		material.setTextureSamplerFlags(index, TEXTURE_SAMPLER_WRAP_CLAMP | TEXTURE_SAMPLER_FILTER_TRILINEAR);

		wait;
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("Procedural textures");
	
	object = new ObjectVolumeBox(vec3(20.0f));
	object.setMaterial(findMaterialByName("volume_cloud_base"),"*");
	object.setTransform(translate(Vec3(0.0f,0.0f,1.0f)) * rotateZ(45.0f));
	material = object.getMaterialInherit(0);
	
	thread("update_thread");
	
	return 1;
}

/*
 */
void clear_scene() {
	
	if(async != NULL) async.wait();
}

```

## property_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("Property");
	
	Property base = engine.properties.findProperty("property_base_0");
	
	forloop(int i = 0; base.getNumChildren()) {
		Property property = base.getChild(i);
		PropertyParameter pp = property.getParameterPtr();
		
		// check enabled flag
		if(PropertyParameter(pp.getChild("enabled")).getValueInt() == 0) continue;
		
		// property parameters
		vec4 color = PropertyParameter(pp.getChild("color")).getValueVec4();
		vec3 scale = PropertyParameter(pp.getChild("scale")).getValueVec3();
		vec3 position = PropertyParameter(pp.getChild("position")).getValueVec3();
		
		// create object
		ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
		mesh.setSurfaceProperty("surface_base","*");
		mesh.setMaterial(findMaterialByName("properties_mesh_base"),"*");
		mesh.setMaterialParameterFloat4("diffuse_color",color,0);
		mesh.setWorldTransform(translate(Vec3(position)) * ::scale(scale));
	}
	
	return 1;
}

```

## property_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("Property");
	
	Property base = engine.properties.findProperty("property_base_1");
	
	forloop(int i = 0; base.getNumChildren()) {
		Property property = base.getChild(i);
		PropertyParameter pp = property.getParameterPtr();
		
		// check enabled flag
		if(PropertyParameter(pp.getChild("enabled")).getValueInt() == 0) continue;
		
		// create object
		string name = PropertyParameter(pp.getChild("mesh")).getValueFile();
		ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(name));
		mesh.setSurfaceProperty("surface_base","*");
		mesh.setCollision(1, 0);
		
		// material name
		int num = PropertyParameter(pp.getChild("material")).getValueInt();
		mesh.setMaterial(findMaterialByName(PropertyParameter(pp.getChild("material")).getSwitchItemName(num)),"*");
		
		// dynamic object
		if(pp.findChild("dynamic")) {
			vec3 size = PropertyParameter(pp.getChild("size")).getValueVec3();
			float mass = PropertyParameter(pp.getChild("mass")).getValueFloat();
			BodyRigid body = class_remove(new BodyRigid(mesh));
			ShapeBox shape = class_remove(new ShapeBox(body,size));
			shape.setMass(mass);
		}
		
		// set position
		vec3 position = PropertyParameter(pp.getChild("position")).getValueVec3();
		mesh.setWorldTransform(translate(Vec3(position)));
	}
	
	return 1;
}

```

## query_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 4;
	
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/stress/meshes/sphere_01.mesh")));
				mesh.setWorldTransform(translate(Vec3(x,y,z + 9.0f) * 1.1f));
				mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y ^ z)),"*");
				mesh.setSurfaceProperty("surface_base","*");
				mesh.setQuery(1);
				num++;
			}
		}
	}
	
	setDescription(format("%d ObjectMeshStatic with occlusion query",num));
	
	return 1;
}

```

## radial_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("Radial blur");
	
	engine.render.addScriptableMaterial(findMaterialByName("render_blur_radial"));
	
	int size = 3;
	float space = 16.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	return 1;
}

```

## ragdoll_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
BodyRagdoll bodies[0];

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
void contact_callback(BodyRagdoll ragdoll, Body body,int num) {
	
	Body b0 = body.getContactBody0(num);
	Body b1 = body.getContactBody1(num);
	
	string n0 = (b0 != NULL) ? b0.getName() : "";
	string n1 = (b1 != NULL) ? b1.getName() : "";
	
	if(n0 == "car" || n1 == "car" || n0 == "sphere" || n1 == "sphere") {
		ObjectMeshSkinned(node_cast(ragdoll.getObject())).stop();
		ragdoll.setFrameBased(0);
	}
}

/*
 */
int update() {
	updateSamplePhysics();
	
	float time = engine.game.getTime();
	float offset = 360.0f / bodies.size();
	
	forloop(int i = 0; bodies.size()) {
		if(bodies[i].isFrameBased()) {
			bodies[i].setVelocityTransform(Mat4(rotateZ(time * 30.0f + offset * i) * translate(12.0f,0.0f,0.0f) * rotateZ(180.0f)));
		}
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyRagdoll bone velocities");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int num = 5;
	int size = 10;
	float space = 1.01f;
	float offset = 360.0f / num;
	
	forloop(int i = 0; num) {
		
		ObjectMeshSkinned mesh = addToEditor(node_load(full_path("uniginescript_samples/physics/meshes/ragdoll_00.node")));
		mesh.setWorldTransform(Mat4(rotateZ(offset * i) * translate(12.0f,0.0f,0.0f) * rotateZ(180.0f)));
		mesh.setLayerAnimationFilePath(0,full_path("uniginescript_samples/physics/meshes/ragdoll_00.anim"));
		mesh.setTime(2.0f * i);
		mesh.play();
		
		BodyRagdoll body = body_cast(mesh.getBody());
		body.getEventContactEnter().connect(functionid(contact_callback),body);
		body.setRigidity(0.25f);
		bodies.append(body);
		
		Mat4 transform = Mat4(rotateZ(offset * i + offset * 0.5f) * translate(12.0f,0.0f,0.0f) * rotateZ(180.0f));
		
		for(int j = 0; j < size; j++) {
			for(int i = 0; i < size - j; i++) {
				createBodyBox(vec3(1.0f),20.0f,0.5f,0.5f,get_material(i ^ j * 2),transform * translate(vec3(i + j * 0.5f - size * 0.5f + 0.5f,0.0f,j + 0.5f) * space));
			}
		}
	}
	
	return 1;
}

```

## ragdoll_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Manually constructed BodyRagdoll");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 6;
	float space = 1.5f;
	vec3 offset = vec3(-1.5f * 3.0f + 0.75f,-1.5f * 3.0f + 0.75f,0.0f);
	
	Body bodies[0];
	
	ObjectDummy object = addToEditor(new ObjectDummy());
	
	forloop(int j = 0; size) {
		forloop(int i = 0; size) {
			Body body = createBodyBox(vec3(1.5f),0.25f,1.0f,0.5f,get_material(0),translate(Vec3(i,j,0.0f) * space + offset) * rotateZ(180.0f));
			body.setName(format("bone_%d%d",i,size - 1 - j));
			object.addChild(body.getObject());
			bodies.append(body);
		}
	}
	
	void create_joint(Body b0,Body b1) {
		JointBall j = class_remove(new JointBall(b0,b1));
		j.setLinearSoftness(0.8f);
		j.setAngularSoftness(0.8f);
		j.setLinearRestitution(0.1f);
		j.setAngularRestitution(0.1f);
		j.setAngularDamping(20.0f);
		j.setAngularLimitAngle(150);
	}
	
	forloop(int j = 0; size) {
		forloop(int i = 0; size) {
			if(i < size - 1) {
				Body b0 = bodies[size * (j + 0) + i + 0];
				Body b1 = bodies[size * (j + 0) + i + 1];
				create_joint(b0,b1);
			}
			if(j < size - 1) {
				Body b0 = bodies[size * (j + 0) + i + 0];
				Body b1 = bodies[size * (j + 1) + i + 0];
				create_joint(b0,b1);
			}
		}
	}
	
	void create_ragdoll(int material,Mat4 transform) {
		
		ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(full_path("uniginescript_samples/physics/meshes/ragdoll_01.mesh")));
		mesh.setWorldTransform(transform);
		mesh.setMaterial(findMaterialByName(get_material(material)),"*");
		mesh.setSurfaceProperty("surface_base","*");
		
		BodyRagdoll ragdoll = class_remove(new BodyRagdoll(mesh));
		ragdoll.setBones(object);
		ragdoll.setFrameBased(0);
		ragdoll.setRigidity(0.25f);
	}
	
	create_ragdoll(0,Mat4(translate(0.0f,0.0f,11.0f)));
	create_ragdoll(1,Mat4(translate(0.0f,0.0f,13.0f)));
	create_ragdoll(2,Mat4(translate(0.0f,0.0f,15.0f)));
	
	create_ragdoll(1,Mat4(translate(0.0f, 0.0f,5.0f) * rotateX(90.0f)));
	create_ragdoll(2,Mat4(translate(0.0f, 4.0f,5.0f) * rotateX(90.0f)));
	create_ragdoll(3,Mat4(translate(0.0f, 2.0f,5.0f) * rotateX(90.0f)));
	create_ragdoll(3,Mat4(translate(0.0f,-2.0f,5.0f) * rotateX(90.0f)));
	create_ragdoll(2,Mat4(translate(0.0f,-4.0f,5.0f) * rotateX(90.0f)));
	
	object.setEnabled(0);
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## ragdoll_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Automatically constructed convex BodyRagdoll");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 1;
	float space = 7.0f;
	
	ObjectMeshSkinned object = new ObjectMeshSkinned(full_path("uniginescript_samples/physics/meshes/ragdoll_02.mesh"));
	BodyRagdoll ragdoll = class_remove(new BodyRagdoll(object));
	ragdoll.createBones(0.3f,0.2f,0);
	ragdoll.setMass(100.0f);
	
	for(int k = 0; k <= size; k++) {
		for(int j = -size; j <= size; j++) {
			for(int i = -size; i <= size; i++) {
				
				ObjectMeshSkinned mesh = addToEditor(class_append(node_cast(node_clone(object))));
				mesh.setWorldTransform(translate(Vec3(i * space,j * space,k * space + 6.0f)));
				mesh.setMaterial(findMaterialByName(get_material(i + j + k)),"*");
				mesh.setSurfaceProperty("surface_base","*");
				
				BodyRagdoll ragdoll = body_cast(mesh.getBody());
				ragdoll.setFrameBased(0);
				ragdoll.setRigidity(0.25f);
			}
		}
	}
	
	delete object;
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## ragdoll_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Automatically constructed capsule BodyRagdoll");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 4;
	float space = 1.5f;
	
	ObjectMeshSkinned object = new ObjectMeshSkinned(full_path("uniginescript_samples/physics/meshes/ragdoll_03.mesh"));
	BodyRagdoll ragdoll = class_remove(new BodyRagdoll(object));
	ragdoll.createBones(0.3f,0.2f,1);
	ragdoll.setMass(100.0f);
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			
			ObjectMeshSkinned mesh = addToEditor(class_append(node_cast(node_clone(object))));
			mesh.setWorldTransform(translate(Vec3(i,j,6.0f) * space));
			mesh.setMaterial(findMaterialByName(get_material(i + j)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			
			BodyRagdoll ragdoll = body_cast(mesh.getBody());
			ragdoll.setFrameBased(0);
			ragdoll.setRigidity(0.25f);
		}
	}
	
	delete object;
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## ragdoll_04.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;


/*
 */
BodyRagdoll bodies[0];

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
BodyRagdoll create_ragdoll(int frozen,vec3 velocity,Mat4 transform) {
	
	mat4 rotate = rotateZ(-90.0f);
	
	// ragdoll mesh
	ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(full_path("uniginescript_samples/physics/meshes/ragdoll_00.mesh")));
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_eye_lod0");
	mesh.setMaterial(findMaterialByName(get_material(2)),"agent_body_lod0");
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_head_lod0");
	mesh.setMaterial(findMaterialByName(get_material(4)),"agent_holster_lod0");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setWorldTransform(transform);
	
	// create body
	BodyRigid create_body(string name,float radius,float height,float mass,mat4 offset) {
		int bone = mesh.findBone(name);
		ObjectDummy object = new ObjectDummy();
		object.setName(name);
		BodyRigid body = class_remove(new BodyRigid(object));
		Shape shape = class_remove(new ShapeCapsule(radius,height));
		shape.setMass(mass);
		shape.setFriction(0.5f);
		shape.setRestitution(0.0f);
		body.setName(name);
		body.addShape(shape,orthonormalize(mesh.getBoneTransform(bone)) * rotateY(90.0f) * offset);
		body.setLinearVelocity(velocity);
		body.setLinearDamping(0.1f);
		body.setAngularDamping(0.1f);
		body.setFrozenLinearVelocity(0.1f);
		body.setFrozenAngularVelocity(0.1f);
		return body;
	}
	
	// create ball joint
	void create_joint(Body b0,Body b1,vec3 axis,float angle,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointBall joint = class_remove(new JointBall(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitAngle(angle);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// create hinge joint
	void create_joint(Body b0,Body b1,vec3 axis,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointHinge joint = class_remove(new JointHinge(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// torso
	Body spine = create_body("Bip01 Spine",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body spine1 = create_body("Bip01 Spine1",0.5f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body pelvis = create_body("Bip01 Pelvis",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.0f) * rotateY(90.0f));
	
	create_joint(spine,spine1,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	create_joint(pelvis,spine,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	
	// head
	Body head = create_body("Bip01 Head",0.3f,0.15f,5.0f,translate(0.0f,0.12f,0.3f));
	
	create_joint(spine1,head,vec3(0.0f,0.0f,1.0f),140.0f,-25.0f,25.0f);
	
	// thighs
	Body left_thigh = create_body("Bip01 L Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	Body right_thigh = create_body("Bip01 R Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	
	create_joint(pelvis,left_thigh,vec3(0.0f,0.2f,-1.0f),140.0f,-25.0f,25.0f);
	create_joint(pelvis,right_thigh,vec3(0.0f,-0.2f,-1.0f),140.0f,-25.0f,25.0f);
	
	// calfs
	Body left_calf = create_body("Bip01 L Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	Body right_calf = create_body("Bip01 R Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	
	create_joint(left_thigh,left_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	create_joint(right_thigh,right_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	
	// foots
	Body left_foot = create_body("Bip01 L Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	Body right_foot = create_body("Bip01 R Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	
	create_joint(left_calf,left_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	create_joint(right_calf,right_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	
	// upperarms
	Body left_upperarm = create_body("Bip01 L UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	Body right_upperarm = create_body("Bip01 R UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(spine1,left_upperarm,vec3(0.0f,1.0f,-0.5f),120.0f,-60.0f,60.0f);
	create_joint(spine1,right_upperarm,vec3(0.0f,-1.0f,-0.5f),120.0f,-60.0f,60.0f);
	
	// forearms
	Body left_forearm = create_body("Bip01 L Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	Body right_forearm = create_body("Bip01 R Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(left_upperarm,left_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	create_joint(right_upperarm,right_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	
	// hands
	Body left_hand = create_body("Bip01 L Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	Body right_hand = create_body("Bip01 R Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	
	create_joint(left_forearm,left_hand,vec3(0.0f,-0.2f,1.1f),120.0f,-25.0f,25.0f);
	create_joint(right_forearm,right_hand,vec3(0.0f,0.2f,1.1f),120.0f,-25.0f,25.0f);
	
	// ragdoll bones
	Node bones = new NodeDummy();
	
	bones.addWorldChild(spine.getObject());
	bones.addWorldChild(spine1.getObject());
	bones.addWorldChild(pelvis.getObject());
	
	bones.addWorldChild(head.getObject());
	
	bones.addWorldChild(left_thigh.getObject());
	bones.addWorldChild(right_thigh.getObject());
	
	bones.addWorldChild(left_calf.getObject());
	bones.addWorldChild(right_calf.getObject());
	
	bones.addWorldChild(left_foot.getObject());
	bones.addWorldChild(right_foot.getObject());
	
	bones.addWorldChild(left_upperarm.getObject());
	bones.addWorldChild(right_upperarm.getObject());
	
	bones.addWorldChild(left_forearm.getObject());
	bones.addWorldChild(right_forearm.getObject());
	
	bones.addWorldChild(left_hand.getObject());
	bones.addWorldChild(right_hand.getObject());
	
	// ragdoll
	BodyRagdoll ragdoll = class_remove(new BodyRagdoll(mesh));
	ragdoll.setBones(bones);
	
	node_delete(bones);
	
	return ragdoll;
}

/*
 */
void contact_callback(BodyRagdoll ragdoll, Body body,int num) {
	
	Body b0 = body.getContactBody0(num);
	Body b1 = body.getContactBody1(num);
	
	string n0 = (b0 != NULL) ? b0.getName() : "";
	string n1 = (b1 != NULL) ? b1.getName() : "";
	
	if(n0 == "car" || n1 == "car" || n0 == "sphere" || n1 == "sphere") {
		ObjectMeshSkinned(node_cast(ragdoll.getObject())).stop();
		ragdoll.setFrameBased(0);
	}
}

/*
 */
int update() {
	updateSamplePhysics();
	
	float time = engine.game.getTime();
	float offset = 360.0f / bodies.size();
	
	forloop(int i = 0; bodies.size()) {
		if(bodies[i].isFrameBased()) {
			bodies[i].setVelocityTransform(Mat4(rotateZ(time * 20.0f + offset * i) * translate(12.0f,0.0f,0.0f) * rotateZ(180.0f)));
			bodies[i].flushTransform();
		}
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Partially animated BodyRagdoll");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int num = 12;
	float offset = 360.0f / num;
	
	forloop(int i = 0; num) {
		
		BodyRagdoll body = create_ragdoll(0,vec3_zero,translate(Vec3(0.0f,0.0f,0.0f)));
		body.getEventContactEnter().connect(functionid(contact_callback),body);
		body.setRigidity(0.25f);
		
		ObjectMeshSkinned mesh = node_cast(body.getObject());
		mesh.setWorldTransform(Mat4(rotateZ(offset * i) * translate(12.0f,0.0f,0.0f) * rotateZ(180.0f)));
		mesh.setLayerAnimationFilePath(0,full_path("uniginescript_samples/physics/meshes/ragdoll_00.anim"));
		mesh.setSpeed(20.0f);
		mesh.setTime(20.0f * i);
		mesh.setLoop(1);
		mesh.play();
		
		body.setBoneFrameBased(body.findBone("Bip01 Head"),0);
		body.setBoneFrameBased(body.findBone("Bip01 Spine1"),0);
		body.setBoneFrameBased(body.findBone("Bip01 L UpperArm"),0);
		body.setBoneFrameBased(body.findBone("Bip01 R UpperArm"),0);
		body.setBoneFrameBased(body.findBone("Bip01 L Forearm"),0);
		body.setBoneFrameBased(body.findBone("Bip01 R Forearm"),0);
		body.setBoneFrameBased(body.findBone("Bip01 L Hand"),0);
		body.setBoneFrameBased(body.findBone("Bip01 R Hand"),0);
		
		bodies.append(body);
	}
	
	return 1;
}

```

## ragdoll_05.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Body target = 0;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
BodyRagdoll create_ragdoll(int frozen,vec3 velocity,Mat4 transform) {
	
	mat4 rotate = rotateZ(-90.0f);
	
	// ragdoll mesh
	ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(full_path("uniginescript_samples/physics/meshes/ragdoll_00.mesh")));
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_eye_lod0");
	mesh.setMaterial(findMaterialByName(get_material(2)),"agent_body_lod0");
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_head_lod0");
	mesh.setMaterial(findMaterialByName(get_material(4)),"agent_holster_lod0");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setWorldTransform(transform);
	
	// create body
	BodyRigid create_body(string name,float radius,float height,float mass,mat4 offset) {
		int bone = mesh.findBone(name);
		ObjectDummy object = new ObjectDummy();
		object.setName(name);
		BodyRigid body = class_remove(new BodyRigid(object));
		Shape shape = class_remove(new ShapeCapsule(radius,height));
		shape.setMass(mass);
		shape.setFriction(0.5f);
		shape.setRestitution(0.0f);
		body.setName(name);
		body.addShape(shape,orthonormalize(mesh.getBoneTransform(bone)) * rotateY(90.0f) * offset);
		body.setLinearVelocity(velocity);
		body.setLinearDamping(0.1f);
		body.setAngularDamping(0.1f);
		body.setFrozenLinearVelocity(0.1f);
		body.setFrozenAngularVelocity(0.1f);
		return body;
	}
	
	// create ball joint
	void create_joint(Body b0,Body b1,vec3 axis,float angle,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointBall joint = class_remove(new JointBall(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitAngle(angle);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// create hinge joint
	void create_joint(Body b0,Body b1,vec3 axis,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointHinge joint = class_remove(new JointHinge(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// torso
	Body spine = create_body("Bip01 Spine",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body spine1 = create_body("Bip01 Spine1",0.5f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body pelvis = create_body("Bip01 Pelvis",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.0f) * rotateY(90.0f));
	
	create_joint(spine,spine1,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	create_joint(pelvis,spine,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	
	// head
	Body head = create_body("Bip01 Head",0.3f,0.15f,5.0f,translate(0.0f,0.12f,0.3f));
	
	create_joint(spine1,head,vec3(0.0f,0.0f,1.0f),140.0f,-25.0f,25.0f);
	
	// thighs
	Body left_thigh = create_body("Bip01 L Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	Body right_thigh = create_body("Bip01 R Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	
	create_joint(pelvis,left_thigh,vec3(0.0f,0.2f,-1.0f),140.0f,-25.0f,25.0f);
	create_joint(pelvis,right_thigh,vec3(0.0f,-0.2f,-1.0f),140.0f,-25.0f,25.0f);
	
	// calfs
	Body left_calf = create_body("Bip01 L Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	Body right_calf = create_body("Bip01 R Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	
	create_joint(left_thigh,left_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	create_joint(right_thigh,right_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	
	// foots
	Body left_foot = create_body("Bip01 L Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	Body right_foot = create_body("Bip01 R Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	
	create_joint(left_calf,left_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	create_joint(right_calf,right_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	
	// upperarms
	Body left_upperarm = create_body("Bip01 L UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	Body right_upperarm = create_body("Bip01 R UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(spine1,left_upperarm,vec3(0.0f,1.0f,-0.5f),120.0f,-60.0f,60.0f);
	create_joint(spine1,right_upperarm,vec3(0.0f,-1.0f,-0.5f),120.0f,-60.0f,60.0f);
	
	// forearms
	Body left_forearm = create_body("Bip01 L Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	Body right_forearm = create_body("Bip01 R Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(left_upperarm,left_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	create_joint(right_upperarm,right_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	
	// hands
	Body left_hand = create_body("Bip01 L Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	Body right_hand = create_body("Bip01 R Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	
	create_joint(left_forearm,left_hand,vec3(0.0f,-0.2f,1.1f),120.0f,-25.0f,25.0f);
	create_joint(right_forearm,right_hand,vec3(0.0f,0.2f,1.1f),120.0f,-25.0f,25.0f);
	
	// ragdoll bones
	Node bones = new NodeDummy();
	
	bones.addWorldChild(spine.getObject());
	bones.addWorldChild(spine1.getObject());
	bones.addWorldChild(pelvis.getObject());
	
	bones.addWorldChild(head.getObject());
	
	bones.addWorldChild(left_thigh.getObject());
	bones.addWorldChild(right_thigh.getObject());
	
	bones.addWorldChild(left_calf.getObject());
	bones.addWorldChild(right_calf.getObject());
	
	bones.addWorldChild(left_foot.getObject());
	bones.addWorldChild(right_foot.getObject());
	
	bones.addWorldChild(left_upperarm.getObject());
	bones.addWorldChild(right_upperarm.getObject());
	
	bones.addWorldChild(left_forearm.getObject());
	bones.addWorldChild(right_forearm.getObject());
	
	bones.addWorldChild(left_hand.getObject());
	bones.addWorldChild(right_hand.getObject());
	
	// ragdoll
	BodyRagdoll ragdoll = class_remove(new BodyRagdoll(mesh));
	ragdoll.setBones(bones);
	
	node_delete(bones);
	
	return ragdoll;
}


/*
 */
void contact_callback(BodyRagdoll ragdoll, Body body,int num) {
	
	Body b0 = body.getContactBody0(num);
	Body b1 = body.getContactBody1(num);
	
	string n0 = (b0 != NULL) ? b0.getName() : "";
	string n1 = (b1 != NULL) ? b1.getName() : "";
	
	if(n0 == "box" || n1 == "box" || n0 == "sphere" || n1 == "sphere") {
		ObjectMeshSkinned(node_cast(ragdoll.getObject())).stop();
		ragdoll.setFrameBased(0);
		target = ragdoll;
	}
}

/*
 */
BodyRagdoll create_ragdoll(Mat4 transform) {
	
	BodyRagdoll body = create_ragdoll(0,vec3_zero,transform);
	body.getEventContactEnter().connect(functionid(contact_callback),body);
	
	ObjectMeshSkinned mesh = node_cast(body.getObject());
	mesh.setLayerAnimationFilePath(0,full_path("uniginescript_samples/physics/meshes/ragdoll_00.anim"));
	mesh.setSpeed(50.0f);
	mesh.setLoop(1);
	mesh.play();
	return body;
}

/*
 */
int update_thread() {
	
	int counter = 0;
	
	while(1) {
		
		float time = 0.0f;
		int y = engine.game.getRandom(-16,16);
		BodyRagdoll body = create_ragdoll(Mat4(translate(15.0f,y,0.0f) * rotateZ(-90.0f)));
		body.setRigidity(0.25f);
		
		while(1) {
			
			body.setVelocityTransform(Mat4(translate(15.0f - time * 20.0f,y,0.0f) * rotateZ(-90.0f)));
			
			time += engine.game.getIFps();
			if(time > 2.0f || target == body) break;
			
			wait;
		}
		
		target = 0;
		ObjectMeshSkinned(node_cast(body.getObject())).stop();
		body.setFrameBased(0);
		
		if(counter++ > 26) break;
		
		wait;
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyRagdoll callback");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int width = 40;
	int height = 6;
	float space = 1.01f;
	
	for(int j = 0; j < height; j++) {
		for(int i = 0; i < width - j; i++) {
			Body b0 = createBodyBox(vec3(1.0f,1.0f,1.0f),80.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(-10.0f,i + j * 0.5f - width * 0.5f + 0.5f,j + 0.5f) * space));
			b0.setName("box");
		}
	}
	
	thread("update_thread");
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## ragdoll_06.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
BodyRagdoll bodies[0];

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
BodyRagdoll create_ragdoll(int frozen,vec3 velocity,Mat4 transform) {
	
	mat4 rotate = rotateZ(-90.0f);
	
	// ragdoll mesh
	ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(full_path("uniginescript_samples/physics/meshes/ragdoll_00.mesh")));
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_eye_lod0");
	mesh.setMaterial(findMaterialByName(get_material(2)),"agent_body_lod0");
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_head_lod0");
	mesh.setMaterial(findMaterialByName(get_material(4)),"agent_holster_lod0");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setWorldTransform(transform);
	
	// create body
	BodyRigid create_body(string name,float radius,float height,float mass,mat4 offset) {
		int bone = mesh.findBone(name);
		ObjectDummy object = new ObjectDummy();
		object.setName(name);
		BodyRigid body = class_remove(new BodyRigid(object));
		Shape shape = class_remove(new ShapeCapsule(radius,height));
		shape.setMass(mass);
		shape.setFriction(0.5f);
		shape.setRestitution(0.0f);
		body.setName(name);
		body.addShape(shape,orthonormalize(mesh.getBoneTransform(bone)) * rotateY(90.0f) * offset);
		body.setLinearVelocity(velocity);
		body.setLinearDamping(0.1f);
		body.setAngularDamping(0.1f);
		body.setFrozenLinearVelocity(0.1f);
		body.setFrozenAngularVelocity(0.1f);
		return body;
	}
	
	// create ball joint
	void create_joint(Body b0,Body b1,vec3 axis,float angle,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointBall joint = class_remove(new JointBall(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitAngle(angle);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// create hinge joint
	void create_joint(Body b0,Body b1,vec3 axis,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointHinge joint = class_remove(new JointHinge(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// torso
	Body spine = create_body("Bip01 Spine",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body spine1 = create_body("Bip01 Spine1",0.5f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body pelvis = create_body("Bip01 Pelvis",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.0f) * rotateY(90.0f));
	
	create_joint(spine,spine1,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	create_joint(pelvis,spine,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	
	// head
	Body head = create_body("Bip01 Head",0.3f,0.15f,5.0f,translate(0.0f,0.12f,0.3f));
	
	create_joint(spine1,head,vec3(0.0f,0.0f,1.0f),140.0f,-25.0f,25.0f);
	
	// thighs
	Body left_thigh = create_body("Bip01 L Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	Body right_thigh = create_body("Bip01 R Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	
	create_joint(pelvis,left_thigh,vec3(0.0f,0.2f,-1.0f),140.0f,-25.0f,25.0f);
	create_joint(pelvis,right_thigh,vec3(0.0f,-0.2f,-1.0f),140.0f,-25.0f,25.0f);
	
	// calfs
	Body left_calf = create_body("Bip01 L Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	Body right_calf = create_body("Bip01 R Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	
	create_joint(left_thigh,left_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	create_joint(right_thigh,right_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	
	// foots
	Body left_foot = create_body("Bip01 L Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	Body right_foot = create_body("Bip01 R Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	
	create_joint(left_calf,left_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	create_joint(right_calf,right_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	
	// upperarms
	Body left_upperarm = create_body("Bip01 L UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	Body right_upperarm = create_body("Bip01 R UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(spine1,left_upperarm,vec3(0.0f,1.0f,-0.5f),120.0f,-60.0f,60.0f);
	create_joint(spine1,right_upperarm,vec3(0.0f,-1.0f,-0.5f),120.0f,-60.0f,60.0f);
	
	// forearms
	Body left_forearm = create_body("Bip01 L Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	Body right_forearm = create_body("Bip01 R Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(left_upperarm,left_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	create_joint(right_upperarm,right_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	
	// hands
	Body left_hand = create_body("Bip01 L Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	Body right_hand = create_body("Bip01 R Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	
	create_joint(left_forearm,left_hand,vec3(0.0f,-0.2f,1.1f),120.0f,-25.0f,25.0f);
	create_joint(right_forearm,right_hand,vec3(0.0f,0.2f,1.1f),120.0f,-25.0f,25.0f);
	
	// ragdoll bones
	Node bones = new NodeDummy();
	
	bones.addWorldChild(spine.getObject());
	bones.addWorldChild(spine1.getObject());
	bones.addWorldChild(pelvis.getObject());
	
	bones.addWorldChild(head.getObject());
	
	bones.addWorldChild(left_thigh.getObject());
	bones.addWorldChild(right_thigh.getObject());
	
	bones.addWorldChild(left_calf.getObject());
	bones.addWorldChild(right_calf.getObject());
	
	bones.addWorldChild(left_foot.getObject());
	bones.addWorldChild(right_foot.getObject());
	
	bones.addWorldChild(left_upperarm.getObject());
	bones.addWorldChild(right_upperarm.getObject());
	
	bones.addWorldChild(left_forearm.getObject());
	bones.addWorldChild(right_forearm.getObject());
	
	bones.addWorldChild(left_hand.getObject());
	bones.addWorldChild(right_hand.getObject());
	
	// ragdoll
	BodyRagdoll ragdoll = class_remove(new BodyRagdoll(mesh));
	ragdoll.setBones(bones);
	
	node_delete(bones);
	
	return ragdoll;
}


/*
 */
void contact_callback(BodyRagdoll ragdoll, Body body,int num) {
	
	Body b0 = body.getContactBody0(num);
	Body b1 = body.getContactBody1(num);
	
	string n0 = (b0 != NULL) ? b0.getName() : "";
	string n1 = (b1 != NULL) ? b1.getName() : "";
	
	if(n0 == "car" || n1 == "car" || n0 == "sphere" || n1 == "sphere") {
		ObjectMeshSkinned(node_cast(ragdoll.getObject())).stop();
		ragdoll.setFrameBased(0);
	}
}

/*
 */
int update() {
	updateSamplePhysics();
	
	float time = engine.game.getTime();
	
	forloop(int i = 0; bodies.size()) {
		if(bodies[i].isFrameBased()) {
			float y = i * 3.0f - bodies.size() * 1.5f;
			bodies[i].setVelocityTransform(Mat4(translate(-40.0f + time * 10.0f,y,0.0f) * rotateZ(90.0f)));
			bodies[i].flushTransform();
		}
	}
	
	return 1;
}

/*
 */
BodyRagdoll create_ragdoll(mat4 transform) {
	
	BodyRagdoll body = create_ragdoll(0,vec3_zero,transform);
	body.getEventContactEnter().connect(functionid(contact_callback),body);
	body.setRigidity(0.25f);
	
	ObjectMeshSkinned mesh = node_cast(body.getObject());
	mesh.setLayerAnimationFilePath(0,full_path("uniginescript_samples/physics/meshes/ragdoll_00.anim"));
	mesh.setSpeed(30.0f);
	mesh.setLoop(1);
	mesh.play();
	
	return body;
}

/*
 */
int create_body(Mat4 transform) {
	
	void create_joint(Body b0,Body b1) {
		JointSuspension j = new JointSuspension(b0,b1);
		j.setWorldAxis0(rotation(transform) * vec3(0.0f,0.0f,1.0f));
		j.setWorldAxis1(rotation(transform) * vec3(0.0f,1.0f,0.0f));
		j.setLinearSpring(100.0f);
		j.setLinearDamping(8.0f);
		j.setLinearLimitFrom(-0.5f);
		j.setLinearLimitTo(0.0f);
		j.setAngularVelocity(-20.0f);
		j.setAngularTorque(30.0f);
		j.setLinearRestitution(0.8f);
		j.setAngularRestitution(0.2f);
		j.setLinearSoftness(0.01f);
		j.setAngularSoftness(0.01f);
		j.setNumIterations(2);
	}
	
	BodyRigid body = createBodyBox(vec3(4.0f,2.0f,1.25f),4.0f,0.5f,0.5f,get_material(0),transform);
	body.setMaxAngularVelocity(10.0f);
	body.setName("car");
	
	Body b0 = createBodySphere(0.6f,4.0f,1.0f,0.5f,get_material(1),transform * translate( 1.75f, 1.25f,-0.75f));
	Body b1 = createBodySphere(0.6f,4.0f,1.0f,0.5f,get_material(1),transform * translate( 1.75f,-1.25f,-0.75f));
	Body b2 = createBodySphere(0.6f,4.0f,1.0f,0.5f,get_material(1),transform * translate(-1.75f, 1.25f,-0.75f));
	Body b3 = createBodySphere(0.6f,4.0f,1.0f,0.5f,get_material(1),transform * translate(-1.75f,-1.25f,-0.75f));
	
	create_joint(body,b0);
	create_joint(body,b1);
	create_joint(body,b2);
	create_joint(body,b3);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyRagdoll callback");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int num = 16;
	
	for(int i = 0; i <= num; i++) {
		float y = i * 3.0f - num * 1.5f;
		bodies.append(create_ragdoll(translate(Vec3(-40.0f,y,0.0f))));
		if(i & 0x01) create_body(Mat4(translate(10.0f,y * 2.0f,1.25f) * rotateZ(180.0f + y * 2.0f)));
		else create_body(Mat4(translate(30.0f,y * 3.0f,1.25f) * rotateZ(180.0f + y * 3.0f)));
	}
	
	return 1;
}

```

## ragdoll_07.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
RagDoll ragdolls[0];

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
BodyRagdoll create_ragdoll(int frozen,vec3 velocity,Mat4 transform) {
	
	mat4 rotate = rotateZ(-90.0f);
	
	// ragdoll mesh
	ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(full_path("uniginescript_samples/physics/meshes/ragdoll_00.mesh")));
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_eye_lod0");
	mesh.setMaterial(findMaterialByName(get_material(2)),"agent_body_lod0");
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_head_lod0");
	mesh.setMaterial(findMaterialByName(get_material(4)),"agent_holster_lod0");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setWorldTransform(transform);
	
	// create body
	BodyRigid create_body(string name,float radius,float height,float mass,mat4 offset) {
		int bone = mesh.findBone(name);
		ObjectDummy object = new ObjectDummy();
		object.setName(name);
		BodyRigid body = class_remove(new BodyRigid(object));
		Shape shape = class_remove(new ShapeCapsule(radius,height));
		shape.setMass(mass);
		shape.setFriction(0.5f);
		shape.setRestitution(0.0f);
		body.setName(name);
		body.addShape(shape,orthonormalize(mesh.getBoneTransform(bone)) * rotateY(90.0f) * offset);
		body.setLinearVelocity(velocity);
		body.setLinearDamping(0.1f);
		body.setAngularDamping(0.1f);
		body.setFrozenLinearVelocity(0.1f);
		body.setFrozenAngularVelocity(0.1f);
		return body;
	}
	
	// create ball joint
	void create_joint(Body b0,Body b1,vec3 axis,float angle,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointBall joint = class_remove(new JointBall(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitAngle(angle);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// create hinge joint
	void create_joint(Body b0,Body b1,vec3 axis,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointHinge joint = class_remove(new JointHinge(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// torso
	Body spine = create_body("Bip01 Spine",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body spine1 = create_body("Bip01 Spine1",0.5f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body pelvis = create_body("Bip01 Pelvis",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.0f) * rotateY(90.0f));
	
	create_joint(spine,spine1,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	create_joint(pelvis,spine,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	
	// head
	Body head = create_body("Bip01 Head",0.3f,0.15f,5.0f,translate(0.0f,0.12f,0.3f));
	
	create_joint(spine1,head,vec3(0.0f,0.0f,1.0f),140.0f,-25.0f,25.0f);
	
	// thighs
	Body left_thigh = create_body("Bip01 L Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	Body right_thigh = create_body("Bip01 R Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	
	create_joint(pelvis,left_thigh,vec3(0.0f,0.2f,-1.0f),140.0f,-25.0f,25.0f);
	create_joint(pelvis,right_thigh,vec3(0.0f,-0.2f,-1.0f),140.0f,-25.0f,25.0f);
	
	// calfs
	Body left_calf = create_body("Bip01 L Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	Body right_calf = create_body("Bip01 R Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	
	create_joint(left_thigh,left_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	create_joint(right_thigh,right_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	
	// foots
	Body left_foot = create_body("Bip01 L Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	Body right_foot = create_body("Bip01 R Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	
	create_joint(left_calf,left_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	create_joint(right_calf,right_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	
	// upperarms
	Body left_upperarm = create_body("Bip01 L UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	Body right_upperarm = create_body("Bip01 R UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(spine1,left_upperarm,vec3(0.0f,1.0f,-0.5f),120.0f,-60.0f,60.0f);
	create_joint(spine1,right_upperarm,vec3(0.0f,-1.0f,-0.5f),120.0f,-60.0f,60.0f);
	
	// forearms
	Body left_forearm = create_body("Bip01 L Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	Body right_forearm = create_body("Bip01 R Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(left_upperarm,left_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	create_joint(right_upperarm,right_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	
	// hands
	Body left_hand = create_body("Bip01 L Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	Body right_hand = create_body("Bip01 R Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	
	create_joint(left_forearm,left_hand,vec3(0.0f,-0.2f,1.1f),120.0f,-25.0f,25.0f);
	create_joint(right_forearm,right_hand,vec3(0.0f,0.2f,1.1f),120.0f,-25.0f,25.0f);
	
	// ragdoll bones
	Node bones = new NodeDummy();
	
	bones.addWorldChild(spine.getObject());
	bones.addWorldChild(spine1.getObject());
	bones.addWorldChild(pelvis.getObject());
	
	bones.addWorldChild(head.getObject());
	
	bones.addWorldChild(left_thigh.getObject());
	bones.addWorldChild(right_thigh.getObject());
	
	bones.addWorldChild(left_calf.getObject());
	bones.addWorldChild(right_calf.getObject());
	
	bones.addWorldChild(left_foot.getObject());
	bones.addWorldChild(right_foot.getObject());
	
	bones.addWorldChild(left_upperarm.getObject());
	bones.addWorldChild(right_upperarm.getObject());
	
	bones.addWorldChild(left_forearm.getObject());
	bones.addWorldChild(right_forearm.getObject());
	
	bones.addWorldChild(left_hand.getObject());
	bones.addWorldChild(right_hand.getObject());
	
	// ragdoll
	BodyRagdoll ragdoll = class_remove(new BodyRagdoll(mesh));
	ragdoll.setBones(bones);
	
	node_delete(bones);
	
	return ragdoll;
}


/*
 */
void contact_callback(BodyRagdoll ragdoll, Body body, int num) {
	
	Body b0 = body.getContactBody0(num);
	Body b1 = body.getContactBody1(num);
	
	string n0 = (b0 != NULL) ? b0.getName() : "";
	string n1 = (b1 != NULL) ? b1.getName() : "";
	
	if(n0 == "sphere" || n1 == "sphere") {
		ObjectMeshSkinned(node_cast(ragdoll.getObject())).stop();
		ragdoll.setFrameBased(0);
	}
}

/*
 */
class RagDoll {
	
	BodyRagdoll body;
	
	int foot_0;
	int foot_1;
	
	Body body_0;
	Body body_1;
	
	Mat4 transform_0;
	Mat4 transform_1;
	
	RagDoll(Mat4 transform) {
		
		body = create_ragdoll(0,vec3_zero,transform);
		body.getEventContactEnter().connect(functionid(contact_callback),body);
		body.setRigidity(0.25f);
		
		ObjectMeshSkinned mesh = node_cast(body.getObject());
		mesh.setLayerAnimationFilePath(0,full_path("uniginescript_samples/physics/meshes/ragdoll_00.anim"));
		mesh.setSpeed(20.0f);
		mesh.setTime(11.0f);
		mesh.setLoop(1);
		mesh.play();
		
		forloop(int i = 0; body.getNumChildren()) {
			Body b = body.getChild(i);
			Shape s = b.getShape(0);
			s.setPhysicsIntersectionMask(0x02);
		}
		
		body.setBoneFrameBased(body.findBone("Bip01 L Foot"),0);
		body.setBoneFrameBased(body.findBone("Bip01 R Foot"),0);
		body.setBoneFrameBased(body.findBone("Bip01 L Calf"),0);
		body.setBoneFrameBased(body.findBone("Bip01 R Calf"),0);
		body.setBoneFrameBased(body.findBone("Bip01 L Thigh"),0);
		body.setBoneFrameBased(body.findBone("Bip01 R Thigh"),0);
		
		foot_0 = body.findBone("Bip01 L Foot");
		foot_1 = body.findBone("Bip01 R Foot");
		
		body_0 = body.getChild(foot_0);
		body_1 = body.getChild(foot_1);
	}
	
	void update() {
		
		transform_0 = body.getBoneTransform(foot_0);
		transform_1 = body.getBoneTransform(foot_1);
		
		Vec3 point_0 = (transform_0 * translate(0.0f,-0.3f,-0.1f) * body_0.getShapeTransform(0)) * Vec3_zero;
		Vec3 point_1 = (transform_1 * translate(0.0f,-0.3f,-0.1f) * body_1.getShapeTransform(0)) * Vec3_zero;
		
		PhysicsIntersection intersection = new PhysicsIntersection();
		if(engine.physics.getIntersection(point_0 + Vec3(0.0f,0.0f,2.0f),point_0,0x01,intersection) != NULL) {
			transform_0 = (translate(0.0f,0.0f,length(point_0 - intersection.getPoint()))) * transform_0;
		}
		if(engine.physics.getIntersection(point_1 + Vec3(0.0f,0.0f,2.0f),point_1,0x01,intersection) != NULL) {
			transform_1 = (translate(0.0f,0.0f,length(point_1 - intersection.getPoint()))) * transform_1;
		}
	}
	
	void updatePhysics() {
		if(body.isFrameBased()) {
			body_0.setVelocityTransform(transform_0);
			body_1.setVelocityTransform(transform_1);
		}
	}
};

/*
 */
int update() {
	updateSamplePhysics();
	
	foreach(RagDoll ragdoll; ragdolls) {
		ragdoll.update();
	}
	
	return 1;
}

/*
 */
int updatePhysics() {
	
	foreach(RagDoll ragdoll; ragdolls) {
		ragdoll.updatePhysics();
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(15.0f,0.0f,8.0f));
	createPlaneWithBody();
	setDescription("BodyRagdoll inverse kinematics");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int width = 2;
	int height = 1;
	
	for(int j = -width; j <= width; j++) {
		for(int i = -height; i <= height; i++) {
			ragdolls.append(new RagDoll(Mat4(translate(i * 4.0f,j * 4.0f,0.0f) * rotateZ(90.0f))));
		}
	}
	
	createBodyBox(vec3(16.0f,4.0f,1.5f),0.0f,0.5f,0.5f,get_material(0),Mat4(translate(0.0f, 8.0f,0.0f) * rotateX( 20.0f)));
	createBodyBox(vec3(16.0f,4.0f,1.5f),0.0f,0.5f,0.5f,get_material(0),Mat4(translate(0.0f,-8.0f,0.0f) * rotateX(-20.0f)));
	createBodyBox(vec3(16.0f,4.0f,1.5f),0.0f,0.5f,0.5f,get_material(0),Mat4(translate(0.0f, 0.0f,0.0f) * rotateX(  0.0f)));
	
	return 1;
}

```

## ragdoll_08.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
RagDoll ragdolls[0];

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string wire_material_names[] = ( "physics_wire_red", "physics_wire_green", "physics_wire_blue", "physics_wire_orange", "physics_wire_yellow" );

string get_wire_material(int material) {
	return wire_material_names[abs(material) % wire_material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
BodyRagdoll create_ragdoll(int frozen,vec3 velocity,Mat4 transform) {
	
	mat4 rotate = rotateZ(-90.0f);
	
	// ragdoll mesh
	ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(full_path("uniginescript_samples/physics/meshes/ragdoll_00.mesh")));
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_eye_lod0");
	mesh.setMaterial(findMaterialByName(get_material(2)),"agent_body_lod0");
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_head_lod0");
	mesh.setMaterial(findMaterialByName(get_material(4)),"agent_holster_lod0");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setWorldTransform(transform);
	
	// create body
	BodyRigid create_body(string name,float radius,float height,float mass,mat4 offset) {
		int bone = mesh.findBone(name);
		ObjectDummy object = new ObjectDummy();
		object.setName(name);
		BodyRigid body = class_remove(new BodyRigid(object));
		Shape shape = class_remove(new ShapeCapsule(radius,height));
		shape.setMass(mass);
		shape.setFriction(0.5f);
		shape.setRestitution(0.0f);
		body.setName(name);
		body.addShape(shape,orthonormalize(mesh.getBoneTransform(bone)) * rotateY(90.0f) * offset);
		body.setLinearVelocity(velocity);
		body.setLinearDamping(0.1f);
		body.setAngularDamping(0.1f);
		body.setFrozenLinearVelocity(0.1f);
		body.setFrozenAngularVelocity(0.1f);
		return body;
	}
	
	// create ball joint
	void create_joint(Body b0,Body b1,vec3 axis,float angle,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointBall joint = class_remove(new JointBall(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitAngle(angle);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// create hinge joint
	void create_joint(Body b0,Body b1,vec3 axis,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointHinge joint = class_remove(new JointHinge(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// torso
	Body spine = create_body("Bip01 Spine",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body spine1 = create_body("Bip01 Spine1",0.5f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body pelvis = create_body("Bip01 Pelvis",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.0f) * rotateY(90.0f));
	
	create_joint(spine,spine1,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	create_joint(pelvis,spine,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	
	// head
	Body head = create_body("Bip01 Head",0.3f,0.15f,5.0f,translate(0.0f,0.12f,0.3f));
	
	create_joint(spine1,head,vec3(0.0f,0.0f,1.0f),140.0f,-25.0f,25.0f);
	
	// thighs
	Body left_thigh = create_body("Bip01 L Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	Body right_thigh = create_body("Bip01 R Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	
	create_joint(pelvis,left_thigh,vec3(0.0f,0.2f,-1.0f),140.0f,-25.0f,25.0f);
	create_joint(pelvis,right_thigh,vec3(0.0f,-0.2f,-1.0f),140.0f,-25.0f,25.0f);
	
	// calfs
	Body left_calf = create_body("Bip01 L Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	Body right_calf = create_body("Bip01 R Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	
	create_joint(left_thigh,left_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	create_joint(right_thigh,right_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	
	// foots
	Body left_foot = create_body("Bip01 L Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	Body right_foot = create_body("Bip01 R Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	
	create_joint(left_calf,left_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	create_joint(right_calf,right_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	
	// upperarms
	Body left_upperarm = create_body("Bip01 L UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	Body right_upperarm = create_body("Bip01 R UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(spine1,left_upperarm,vec3(0.0f,1.0f,-0.5f),120.0f,-60.0f,60.0f);
	create_joint(spine1,right_upperarm,vec3(0.0f,-1.0f,-0.5f),120.0f,-60.0f,60.0f);
	
	// forearms
	Body left_forearm = create_body("Bip01 L Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	Body right_forearm = create_body("Bip01 R Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(left_upperarm,left_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	create_joint(right_upperarm,right_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	
	// hands
	Body left_hand = create_body("Bip01 L Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	Body right_hand = create_body("Bip01 R Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	
	create_joint(left_forearm,left_hand,vec3(0.0f,-0.2f,1.1f),120.0f,-25.0f,25.0f);
	create_joint(right_forearm,right_hand,vec3(0.0f,0.2f,1.1f),120.0f,-25.0f,25.0f);
	
	// ragdoll bones
	Node bones = new NodeDummy();
	
	bones.addWorldChild(spine.getObject());
	bones.addWorldChild(spine1.getObject());
	bones.addWorldChild(pelvis.getObject());
	
	bones.addWorldChild(head.getObject());
	
	bones.addWorldChild(left_thigh.getObject());
	bones.addWorldChild(right_thigh.getObject());
	
	bones.addWorldChild(left_calf.getObject());
	bones.addWorldChild(right_calf.getObject());
	
	bones.addWorldChild(left_foot.getObject());
	bones.addWorldChild(right_foot.getObject());
	
	bones.addWorldChild(left_upperarm.getObject());
	bones.addWorldChild(right_upperarm.getObject());
	
	bones.addWorldChild(left_forearm.getObject());
	bones.addWorldChild(right_forearm.getObject());
	
	bones.addWorldChild(left_hand.getObject());
	bones.addWorldChild(right_hand.getObject());
	
	// ragdoll
	BodyRagdoll ragdoll = class_remove(new BodyRagdoll(mesh));
	ragdoll.setBones(bones);
	
	node_delete(bones);
	
	return ragdoll;
}


/*
 */
void contact_callback(BodyRagdoll ragdoll, Body body,int num) {
	
	Body b0 = body.getContactBody0(num);
	Body b1 = body.getContactBody1(num);
	
	string n0 = (b0 != NULL) ? b0.getName() : "";
	string n1 = (b1 != NULL) ? b1.getName() : "";
	
	if(n0 == "sphere" || n1 == "sphere") {
		ObjectMeshSkinned(node_cast(ragdoll.getObject())).stop();
		ragdoll.setFrameBased(0);
	}
}

/*
 */
class RagDoll {
	
	BodyRagdoll body;
	
	float phase;
	
	Body body_0;
	Body body_1;
	
	BodyRope rope_0;
	BodyRope rope_1;
	
	Mat4 transform_0;
	Mat4 transform_1;
	
	Mat4 itransform_0;
	Mat4 itransform_1;
	
	RagDoll(Mat4 transform) {
		
		body = create_ragdoll(0,vec3_zero,transform);
		body.getEventContactEnter().connect(functionid(contact_callback),body);
		body.setRigidity(0.1f);
		
		ObjectMeshSkinned mesh = node_cast(body.getObject());
		mesh.setLayerAnimationFilePath(0,full_path("uniginescript_samples/physics/meshes/ragdoll_00.anim"));
		mesh.setSpeed(25.0f);
		mesh.setLoop(1);
		mesh.play();
		
		forloop(int i = 0; body.getNumBones()) {
			body.setBoneFrameBased(i,0);
		}
		
		body.setBoneFrameBased(body.findBone("Bip01 Head"),1);
		body.setBoneFrameBased(body.findBone("Bip01 Spine"),1);
		
		phase = engine.game.getRandom(0.0f,100.0f);
		
		body_0 = body.getChild(body.findBone("Bip01 L Hand"));
		body_1 = body.getChild(body.findBone("Bip01 R Hand"));
		
		itransform_0 = inverse(body_0.getShapeTransform(0));
		itransform_1 = inverse(body_1.getShapeTransform(0));
		
		transform_0 = transform * body_0.getShapeTransform(0);
		transform_1 = transform * body_1.getShapeTransform(0);
		
		float length = 2.0f;
		
		rope_0 = createBodyRope(0.05f,length,0.1f,5.0f,0.5f,0.5f,get_wire_material(4),transform_0 * translate(0.0f,0.0f,-length * 0.5f));
		rope_1 = createBodyRope(0.05f,length,0.1f,5.0f,0.5f,0.5f,get_wire_material(4),transform_1 * translate(0.0f,0.0f,-length * 0.5f));
		
		rope_0.setRigidity(0.1f);
		rope_1.setRigidity(0.1f);
		
		rope_0.setNumIterations(4);
		rope_1.setNumIterations(4);
		
		class_remove(new JointParticles(body_0,rope_0,transform_0 * Vec3_zero,vec3(0.3f)));
		class_remove(new JointParticles(body_1,rope_1,transform_1 * Vec3_zero,vec3(0.3f)));
	}
	
	void update() {
		
		float time = phase + engine.game.getTime();
		
		float s0 = sin(time * 4.0f);
		float c0 = cos(time * 4.0f);
		float s1 = sin(time * 7.0f);
		float c1 = cos(time * 7.0f);
		
		vec3 position_0 = vec3( 1.0f + c0 * 0.5f - s1 * 0.2f,-1.3f + s0 * 0.25f,4.0f + c1 * 0.4f);
		vec3 position_1 = vec3(-1.0f + c0 * 0.5f - s1 * 0.2f,-1.3f + s0 * 0.25f,4.0f + c1 * 0.4f);
		
		Mat4 transform = body.getTransform();
		
		transform_0 = transform * translate(position_0) * rotateY(180.0f) * rotateX(120.0f) * itransform_0;
		transform_1 = transform * translate(position_1) * rotateY(180.0f) * rotateX(120.0f) * itransform_1;
	}
	
	void updatePhysics() {
		if(body.isFrameBased()) {
			body_0.setVelocityTransform(transform_0);
			body_1.setVelocityTransform(transform_1);
		}
	}
};

/*
 */
int update() {
	updateSamplePhysics();
	
	foreach(RagDoll ragdoll; ragdolls) {
		ragdoll.update();
	}
	
	return 1;
}

/*
 */
int updatePhysics() {
	
	foreach(RagDoll ragdoll; ragdolls) {
		ragdoll.updatePhysics();
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(15.0f,0.0f,8.0f));
	createPlaneWithBody();
	setDescription("BodyRagdoll inverse kinematics with procedural animation");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int width = 2;
	int height = 1;
	
	for(int j = -width; j <= width; j++) {
		for(int i = -height; i <= height; i++) {
			ragdolls.append(new RagDoll(Mat4(translate(i * 4.0f,j * 4.0f,-0.1f) * rotateZ(90.0f - j * 45.0f))));
		}
	}
	
	return 1;
}

```

## ragdoll_09.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
RagDoll ragdolls[0];

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
BodyRagdoll create_ragdoll(int frozen,vec3 velocity,Mat4 transform) {
	
	mat4 rotate = rotateZ(-90.0f);
	
	// ragdoll mesh
	ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(full_path("uniginescript_samples/physics/meshes/ragdoll_00.mesh")));
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_eye_lod0");
	mesh.setMaterial(findMaterialByName(get_material(2)),"agent_body_lod0");
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_head_lod0");
	mesh.setMaterial(findMaterialByName(get_material(4)),"agent_holster_lod0");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setWorldTransform(transform);
	
	// create body
	BodyRigid create_body(string name,float radius,float height,float mass,mat4 offset) {
		int bone = mesh.findBone(name);
		ObjectDummy object = new ObjectDummy();
		object.setName(name);
		BodyRigid body = class_remove(new BodyRigid(object));
		Shape shape = class_remove(new ShapeCapsule(radius,height));
		shape.setMass(mass);
		shape.setFriction(0.5f);
		shape.setRestitution(0.0f);
		body.setName(name);
		body.addShape(shape,orthonormalize(mesh.getBoneTransform(bone)) * rotateY(90.0f) * offset);
		body.setLinearVelocity(velocity);
		body.setLinearDamping(0.1f);
		body.setAngularDamping(0.1f);
		body.setFrozenLinearVelocity(0.1f);
		body.setFrozenAngularVelocity(0.1f);
		return body;
	}
	
	// create ball joint
	void create_joint(Body b0,Body b1,vec3 axis,float angle,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointBall joint = class_remove(new JointBall(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitAngle(angle);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// create hinge joint
	void create_joint(Body b0,Body b1,vec3 axis,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointHinge joint = class_remove(new JointHinge(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// torso
	Body spine = create_body("Bip01 Spine",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body spine1 = create_body("Bip01 Spine1",0.5f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body pelvis = create_body("Bip01 Pelvis",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.0f) * rotateY(90.0f));
	
	create_joint(spine,spine1,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	create_joint(pelvis,spine,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	
	// head
	Body head = create_body("Bip01 Head",0.3f,0.15f,5.0f,translate(0.0f,0.12f,0.3f));
	
	create_joint(spine1,head,vec3(0.0f,0.0f,1.0f),140.0f,-25.0f,25.0f);
	
	// thighs
	Body left_thigh = create_body("Bip01 L Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	Body right_thigh = create_body("Bip01 R Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	
	create_joint(pelvis,left_thigh,vec3(0.0f,0.2f,-1.0f),140.0f,-25.0f,25.0f);
	create_joint(pelvis,right_thigh,vec3(0.0f,-0.2f,-1.0f),140.0f,-25.0f,25.0f);
	
	// calfs
	Body left_calf = create_body("Bip01 L Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	Body right_calf = create_body("Bip01 R Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	
	create_joint(left_thigh,left_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	create_joint(right_thigh,right_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	
	// foots
	Body left_foot = create_body("Bip01 L Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	Body right_foot = create_body("Bip01 R Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	
	create_joint(left_calf,left_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	create_joint(right_calf,right_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	
	// upperarms
	Body left_upperarm = create_body("Bip01 L UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	Body right_upperarm = create_body("Bip01 R UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(spine1,left_upperarm,vec3(0.0f,1.0f,-0.5f),120.0f,-60.0f,60.0f);
	create_joint(spine1,right_upperarm,vec3(0.0f,-1.0f,-0.5f),120.0f,-60.0f,60.0f);
	
	// forearms
	Body left_forearm = create_body("Bip01 L Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	Body right_forearm = create_body("Bip01 R Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(left_upperarm,left_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	create_joint(right_upperarm,right_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	
	// hands
	Body left_hand = create_body("Bip01 L Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	Body right_hand = create_body("Bip01 R Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	
	create_joint(left_forearm,left_hand,vec3(0.0f,-0.2f,1.1f),120.0f,-25.0f,25.0f);
	create_joint(right_forearm,right_hand,vec3(0.0f,0.2f,1.1f),120.0f,-25.0f,25.0f);
	
	// ragdoll bones
	Node bones = new NodeDummy();
	
	bones.addWorldChild(spine.getObject());
	bones.addWorldChild(spine1.getObject());
	bones.addWorldChild(pelvis.getObject());
	
	bones.addWorldChild(head.getObject());
	
	bones.addWorldChild(left_thigh.getObject());
	bones.addWorldChild(right_thigh.getObject());
	
	bones.addWorldChild(left_calf.getObject());
	bones.addWorldChild(right_calf.getObject());
	
	bones.addWorldChild(left_foot.getObject());
	bones.addWorldChild(right_foot.getObject());
	
	bones.addWorldChild(left_upperarm.getObject());
	bones.addWorldChild(right_upperarm.getObject());
	
	bones.addWorldChild(left_forearm.getObject());
	bones.addWorldChild(right_forearm.getObject());
	
	bones.addWorldChild(left_hand.getObject());
	bones.addWorldChild(right_hand.getObject());
	
	// ragdoll
	BodyRagdoll ragdoll = class_remove(new BodyRagdoll(mesh));
	ragdoll.setBones(bones);
	
	node_delete(bones);
	
	return ragdoll;
}


/*
 */
void contact_callback(RagDoll ragdoll, Body body, int num) {
	
	Body b0 = body.getContactBody0(num);
	Body b1 = body.getContactBody1(num);
	
	string n0 = (b0 != NULL) ? b0.getName() : "";
	string n1 = (b1 != NULL) ? b1.getName() : "";
	
	if(n0 == "sphere" || n1 == "sphere") {
		ragdoll.activate();
	}
}

/*
 */
class RagDoll {
	
	Mat4 transform;
	
	BodyRagdoll body;
	
	ObjectMeshSkinned mesh;
	
	Body body_0;
	Body body_1;
	Body body_2;
	
	mat4 offset_0;
	mat4 offset_1;
	mat4 offset_2;
	
	BodyRigid attachment_0;
	BodyRigid attachment_1;
	BodyRigid attachment_2;
	
	RagDoll(mat4 t) {
		
		transform = t;
		
		body = create_ragdoll(0,vec3_zero,transform);
		body.getEventContactEnter().connect(functionid(contact_callback),this);
		body.setRigidity(0.25f);
		
		mesh = node_cast(body.getObject());
		mesh.setLayerAnimationFilePath(0,full_path("uniginescript_samples/physics/meshes/ragdoll_00.anim"));
		mesh.setLayerFrame(0,0.0f);
		mesh.setSpeed(20.0f);
		mesh.setLoop(1);
		mesh.play();
		
		body.setTransform(body.getTransform());
		
		body_0 = body.getChild(body.findBone("Bip01 Head"));
		body_1 = body.getChild(body.findBone("Bip01 R Hand"));
		body_2 = body.getChild(body.findBone("Bip01 L Hand"));
		
		offset_0 = rotateY(90.0f) * translate(0.0f,0.0f,0.8f);
		offset_1 = rotateX(45.0f) * translate(0.2f,0.0f,1.0f);
		offset_2 = rotateX(135.0f) * translate(0.2f,0.0f,1.0f);
		
		Mat4 transform_0 = orthonormalize(mesh.getBoneWorldTransform(mesh.findBone("Bip01 Head")));
		Mat4 transform_1 = orthonormalize(mesh.getBoneWorldTransform(mesh.findBone("Bip01 R Hand")));
		Mat4 transform_2 = orthonormalize(mesh.getBoneWorldTransform(mesh.findBone("Bip01 L Hand")));
		
		attachment_0 = createBodyBox(vec3(1.0f,0.4f,0.4f),40.0f,0.5f,0.5f,get_material(1),transform_0 * offset_0);
		attachment_1 = createBodyCapsule(0.1f,2.0f,40.0f,0.5f,0.5f,get_material(4),transform_1 * offset_1);
		attachment_2 = createBodyCapsule(0.1f,2.0f,40.0f,0.5f,0.5f,get_material(4),transform_2 * offset_2);
		
		attachment_0.getEventContactEnter().connect(functionid(contact_callback),this);
		attachment_1.getEventContactEnter().connect(functionid(contact_callback),this);
		attachment_2.getEventContactEnter().connect(functionid(contact_callback),this);
		
		JointFixed joint_0 = class_remove(new JointFixed(body_0,attachment_0,transform_0 * Vec3_zero));
		JointFixed joint_1 = class_remove(new JointFixed(body_1,attachment_1,transform_1 * Vec3_zero));
		JointFixed joint_2 = class_remove(new JointFixed(body_2,attachment_2,transform_2 * Vec3_zero));
		joint_0.setNumIterations(4);
		joint_1.setNumIterations(4);
		joint_2.setNumIterations(4);
	}
	
	void activate() {
		
		mesh.stop();
		body.setFrameBased(0);
	}
	
	void postUpdate() {
		
		if(body.isFrameBased()) {
			
			Mat4 transform_0 = orthonormalize(mesh.getBoneWorldTransform(mesh.findBone("Bip01 Head")));
			Mat4 transform_1 = orthonormalize(mesh.getBoneWorldTransform(mesh.findBone("Bip01 R Hand")));
			Mat4 transform_2 = orthonormalize(mesh.getBoneWorldTransform(mesh.findBone("Bip01 L Hand")));
			
			Object(attachment_0.getObject()).setWorldTransform(transform_0 * offset_0);
			Object(attachment_1.getObject()).setWorldTransform(transform_1 * offset_1);
			Object(attachment_2.getObject()).setWorldTransform(transform_2 * offset_2);
		}
	}
};

/*
 */
int postUpdate() {
	
	foreach(RagDoll ragdoll; ragdolls) {
		ragdoll.postUpdate();
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(15.0f,0.0f,8.0f));
	createPlaneWithBody();
	setDescription("BodyRagdoll attachments");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int width = 2;
	int height = 1;
	
	for(int j = -width; j <= width; j++) {
		for(int i = -height; i <= height; i++) {
			ragdolls.append(new RagDoll(Mat4(translate(i * 4.0f,j * 4.0f,0.0f) * rotateZ(90.0f))));
		}
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## ragdoll_10.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
RagDoll ragdolls[0];

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
BodyRagdoll create_ragdoll(int frozen,vec3 velocity,Mat4 transform) {
	
	mat4 rotate = rotateZ(-90.0f);
	
	// ragdoll mesh
	ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(full_path("uniginescript_samples/physics/meshes/ragdoll_00.mesh")));
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_eye_lod0");
	mesh.setMaterial(findMaterialByName(get_material(2)),"agent_body_lod0");
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_head_lod0");
	mesh.setMaterial(findMaterialByName(get_material(4)),"agent_holster_lod0");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setWorldTransform(transform);
	
	// create body
	BodyRigid create_body(string name,float radius,float height,float mass,mat4 offset) {
		int bone = mesh.findBone(name);
		ObjectDummy object = new ObjectDummy();
		object.setName(name);
		BodyRigid body = class_remove(new BodyRigid(object));
		Shape shape = class_remove(new ShapeCapsule(radius,height));
		shape.setMass(mass);
		shape.setFriction(0.5f);
		shape.setRestitution(0.0f);
		body.setName(name);
		body.addShape(shape,orthonormalize(mesh.getBoneTransform(bone)) * rotateY(90.0f) * offset);
		body.setLinearVelocity(velocity);
		body.setLinearDamping(0.1f);
		body.setAngularDamping(0.1f);
		body.setFrozenLinearVelocity(0.1f);
		body.setFrozenAngularVelocity(0.1f);
		return body;
	}
	
	// create ball joint
	void create_joint(Body b0,Body b1,vec3 axis,float angle,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointBall joint = class_remove(new JointBall(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitAngle(angle);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// create hinge joint
	void create_joint(Body b0,Body b1,vec3 axis,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointHinge joint = class_remove(new JointHinge(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// torso
	Body spine = create_body("Bip01 Spine",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body spine1 = create_body("Bip01 Spine1",0.5f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body pelvis = create_body("Bip01 Pelvis",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.0f) * rotateY(90.0f));
	
	create_joint(spine,spine1,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	create_joint(pelvis,spine,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	
	// head
	Body head = create_body("Bip01 Head",0.3f,0.15f,5.0f,translate(0.0f,0.12f,0.3f));
	
	create_joint(spine1,head,vec3(0.0f,0.0f,1.0f),140.0f,-25.0f,25.0f);
	
	// thighs
	Body left_thigh = create_body("Bip01 L Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	Body right_thigh = create_body("Bip01 R Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	
	create_joint(pelvis,left_thigh,vec3(0.0f,0.2f,-1.0f),140.0f,-25.0f,25.0f);
	create_joint(pelvis,right_thigh,vec3(0.0f,-0.2f,-1.0f),140.0f,-25.0f,25.0f);
	
	// calfs
	Body left_calf = create_body("Bip01 L Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	Body right_calf = create_body("Bip01 R Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	
	create_joint(left_thigh,left_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	create_joint(right_thigh,right_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	
	// foots
	Body left_foot = create_body("Bip01 L Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	Body right_foot = create_body("Bip01 R Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	
	create_joint(left_calf,left_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	create_joint(right_calf,right_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	
	// upperarms
	Body left_upperarm = create_body("Bip01 L UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	Body right_upperarm = create_body("Bip01 R UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(spine1,left_upperarm,vec3(0.0f,1.0f,-0.5f),120.0f,-60.0f,60.0f);
	create_joint(spine1,right_upperarm,vec3(0.0f,-1.0f,-0.5f),120.0f,-60.0f,60.0f);
	
	// forearms
	Body left_forearm = create_body("Bip01 L Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	Body right_forearm = create_body("Bip01 R Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(left_upperarm,left_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	create_joint(right_upperarm,right_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	
	// hands
	Body left_hand = create_body("Bip01 L Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	Body right_hand = create_body("Bip01 R Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	
	create_joint(left_forearm,left_hand,vec3(0.0f,-0.2f,1.1f),120.0f,-25.0f,25.0f);
	create_joint(right_forearm,right_hand,vec3(0.0f,0.2f,1.1f),120.0f,-25.0f,25.0f);
	
	// ragdoll bones
	Node bones = new NodeDummy();
	
	bones.addWorldChild(spine.getObject());
	bones.addWorldChild(spine1.getObject());
	bones.addWorldChild(pelvis.getObject());
	
	bones.addWorldChild(head.getObject());
	
	bones.addWorldChild(left_thigh.getObject());
	bones.addWorldChild(right_thigh.getObject());
	
	bones.addWorldChild(left_calf.getObject());
	bones.addWorldChild(right_calf.getObject());
	
	bones.addWorldChild(left_foot.getObject());
	bones.addWorldChild(right_foot.getObject());
	
	bones.addWorldChild(left_upperarm.getObject());
	bones.addWorldChild(right_upperarm.getObject());
	
	bones.addWorldChild(left_forearm.getObject());
	bones.addWorldChild(right_forearm.getObject());
	
	bones.addWorldChild(left_hand.getObject());
	bones.addWorldChild(right_hand.getObject());
	
	// ragdoll
	BodyRagdoll ragdoll = class_remove(new BodyRagdoll(mesh));
	ragdoll.setBones(bones);
	
	node_delete(bones);
	
	return ragdoll;
}


/*
 */
class RagDoll {
	
	Mat4 transform;
	
	BodyRagdoll body;
	
	ObjectMeshSkinned mesh;
	
	Body body_0;
	Body body_1;
	
	mat4 offset_0;
	mat4 offset_1;
	
	BodyRigid attachment_0;
	BodyRigid attachment_1;
	
	JointFixed joint_0;
	JointFixed joint_1;
	
	float sleep;
	
	RagDoll(Mat4 t) {
		
		transform = t;
		
		body = create_ragdoll(0,vec3_zero,transform);
		body.setRigidity(0.25f);
		
		mesh = node_cast(body.getObject());
		mesh.setLayerAnimationFilePath(0,full_path("uniginescript_samples/physics/meshes/ragdoll_12.anim"));
		mesh.setLayerFrame(0,0.0f);
		mesh.setSpeed(25.0f);
		mesh.play();
		
		body.setTransform(body.getTransform());
		
		body_0 = body.getChild(body.findBone("Bip01 L Hand"));
		body_1 = body.getChild(body.findBone("Bip01 R Hand"));
		
		offset_0 = rotateY(90.0f) * translate(0.0f,0.0f,0.4f);
		offset_1 = rotateY(135.0f) * translate(-1.2f,0.0f,0.2f);
		
		Mat4 transform_0 = orthonormalize(mesh.getBoneWorldTransform(mesh.findBone("Bip01 L Hand")));
		Mat4 transform_1 = orthonormalize(mesh.getBoneWorldTransform(mesh.findBone("Bip01 R Hand")));
		
		attachment_0 = createBodySphere(0.2f,30.0f,0.5f,0.5f,get_material(1),transform_0 * offset_0);
		attachment_1 = createBodyBox(vec3(1.6f,0.2f,0.6f),20.0f,0.5f,0.5f,get_material(4),transform_1 * offset_1);
		
		joint_0 = class_remove(new JointFixed(body_0,attachment_0,transform_0 * Vec3_zero));
		joint_1 = class_remove(new JointFixed(body_1,attachment_1,transform_1 * Vec3_zero));
		joint_0.setNumIterations(4);
		joint_1.setNumIterations(4);
	}
	
	void updatePhysics() {
		
		float time = mesh.getTime();
		
		if(joint_0.isEnabled() && time > engine.game.getRandom(11.75f,12.25f)) joint_0.setEnabled(0);
		
		if(body.isFrameBased()) {
			Mat4 transform_0 = orthonormalize(mesh.getBoneWorldTransform(mesh.findBone("Bip01 L Hand")));
			Mat4 transform_1 = orthonormalize(mesh.getBoneWorldTransform(mesh.findBone("Bip01 R Hand")));
			if(time < 4.0f) attachment_0.setTransform(transform_0 * offset_0);
			attachment_1.setVelocityTransform(transform_1 * offset_1);
		}
		
		if(time > 37.0f) {
			mesh.stop();
			body.setFrameBased(0);
			sleep += engine.physics.getIFps();
			if(sleep > 2.0f) {
				joint_0.setEnabled(1);
				body.setFrameBased(1);
				mesh.setTime(0.0f);
				mesh.play();
				sleep = 0.0f;
			}
		}
	}
};

/*
 */
int updatePhysics() {
	
	foreach(RagDoll ragdoll; ragdolls) {
		ragdoll.updatePhysics();
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(15.0f,0.0f,8.0f));
	createPlaneWithBody();
	setDescription("BodyRagdoll attachments");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int width = 2;
	int height = 1;
	
	for(int j = -width; j <= width; j++) {
		for(int i = -height; i <= height; i++) {
			ragdolls.append(new RagDoll(Mat4(translate(i * 6.0f,j * 6.0f,0.0f) * rotateZ(90.0f))));
		}
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## ragdoll_11.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
RagDoll ragdolls[0];

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
BodyRagdoll create_ragdoll(int frozen,vec3 velocity,Mat4 transform) {
	
	mat4 rotate = rotateZ(-90.0f);
	
	// ragdoll mesh
	ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(full_path("uniginescript_samples/physics/meshes/ragdoll_00.mesh")));
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_eye_lod0");
	mesh.setMaterial(findMaterialByName(get_material(2)),"agent_body_lod0");
	mesh.setMaterial(findMaterialByName(get_material(0)),"agent_head_lod0");
	mesh.setMaterial(findMaterialByName(get_material(4)),"agent_holster_lod0");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setWorldTransform(transform);
	
	// create body
	BodyRigid create_body(string name,float radius,float height,float mass,mat4 offset) {
		int bone = mesh.findBone(name);
		ObjectDummy object = new ObjectDummy();
		object.setName(name);
		BodyRigid body = class_remove(new BodyRigid(object));
		Shape shape = class_remove(new ShapeCapsule(radius,height));
		shape.setMass(mass);
		shape.setFriction(0.5f);
		shape.setRestitution(0.0f);
		body.setName(name);
		body.addShape(shape,orthonormalize(mesh.getBoneTransform(bone)) * rotateY(90.0f) * offset);
		body.setLinearVelocity(velocity);
		body.setLinearDamping(0.1f);
		body.setAngularDamping(0.1f);
		body.setFrozenLinearVelocity(0.1f);
		body.setFrozenAngularVelocity(0.1f);
		return body;
	}
	
	// create ball joint
	void create_joint(Body b0,Body b1,vec3 axis,float angle,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointBall joint = class_remove(new JointBall(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitAngle(angle);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// create hinge joint
	void create_joint(Body b0,Body b1,vec3 axis,float from,float to) {
		int bone = mesh.findBone(b1.getName());
		JointHinge joint = class_remove(new JointHinge(b0,b1));
		joint.setWorldAnchor(mesh.getBoneTransform(bone) * Vec3_zero);
		joint.setWorldAxis(rotate * axis);
		joint.setAngularLimitFrom(from);
		joint.setAngularLimitTo(to);
		joint.setAngularDamping(16.0f);
		joint.setLinearRestitution(0.9f);
		joint.setAngularRestitution(0.1f);
		joint.setLinearSoftness(0.0f);
		joint.setAngularSoftness(0.0f);
		joint.setNumIterations(2);
	}
	
	// torso
	Body spine = create_body("Bip01 Spine",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body spine1 = create_body("Bip01 Spine1",0.5f,0.2f,8.0f,translate(0.0f,0.0f,0.36f) * rotateY(90.0f));
	Body pelvis = create_body("Bip01 Pelvis",0.45f,0.2f,8.0f,translate(0.0f,0.0f,0.0f) * rotateY(90.0f));
	
	create_joint(spine,spine1,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	create_joint(pelvis,spine,vec3(0.0f,0.0f,1.0f),170.0f,-20.0f,20.0f);
	
	// head
	Body head = create_body("Bip01 Head",0.3f,0.15f,5.0f,translate(0.0f,0.12f,0.3f));
	
	create_joint(spine1,head,vec3(0.0f,0.0f,1.0f),140.0f,-25.0f,25.0f);
	
	// thighs
	Body left_thigh = create_body("Bip01 L Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	Body right_thigh = create_body("Bip01 R Thigh",0.25f,0.6f,5.0f,translate(0.0f,0.0f,0.96f));
	
	create_joint(pelvis,left_thigh,vec3(0.0f,0.2f,-1.0f),140.0f,-25.0f,25.0f);
	create_joint(pelvis,right_thigh,vec3(0.0f,-0.2f,-1.0f),140.0f,-25.0f,25.0f);
	
	// calfs
	Body left_calf = create_body("Bip01 L Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	Body right_calf = create_body("Bip01 R Calf",0.22f,0.7f,4.0f,translate(0.0f,0.0f,0.66f));
	
	create_joint(left_thigh,left_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	create_joint(right_thigh,right_calf,vec3(0.0f,1.0f,0.0f),-90.0f,-2.0f);
	
	// foots
	Body left_foot = create_body("Bip01 L Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	Body right_foot = create_body("Bip01 R Foot",0.2f,0.6f,3.0f,translate(0.0f,0.23f,0.2f) * rotateX(90.0f));
	
	create_joint(left_calf,left_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	create_joint(right_calf,right_foot,vec3(0.0f,1.0f,0.0f),-20.0f,20.0f);
	
	// upperarms
	Body left_upperarm = create_body("Bip01 L UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	Body right_upperarm = create_body("Bip01 R UpperArm",0.21f,0.5f,5.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(spine1,left_upperarm,vec3(0.0f,1.0f,-0.5f),120.0f,-60.0f,60.0f);
	create_joint(spine1,right_upperarm,vec3(0.0f,-1.0f,-0.5f),120.0f,-60.0f,60.0f);
	
	// forearms
	Body left_forearm = create_body("Bip01 L Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	Body right_forearm = create_body("Bip01 R Forearm",0.21f,0.5f,4.0f,translate(0.0f,0.0f,0.33f));
	
	create_joint(left_upperarm,left_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	create_joint(right_upperarm,right_forearm,vec3(0.0f,1.0f,0.2f),0.0f,140.0f);
	
	// hands
	Body left_hand = create_body("Bip01 L Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	Body right_hand = create_body("Bip01 R Hand",0.2f,0.1f,3.0f,translate(0.0f,0.0f,0.21f));
	
	create_joint(left_forearm,left_hand,vec3(0.0f,-0.2f,1.1f),120.0f,-25.0f,25.0f);
	create_joint(right_forearm,right_hand,vec3(0.0f,0.2f,1.1f),120.0f,-25.0f,25.0f);
	
	// ragdoll bones
	Node bones = new NodeDummy();
	
	bones.addWorldChild(spine.getObject());
	bones.addWorldChild(spine1.getObject());
	bones.addWorldChild(pelvis.getObject());
	
	bones.addWorldChild(head.getObject());
	
	bones.addWorldChild(left_thigh.getObject());
	bones.addWorldChild(right_thigh.getObject());
	
	bones.addWorldChild(left_calf.getObject());
	bones.addWorldChild(right_calf.getObject());
	
	bones.addWorldChild(left_foot.getObject());
	bones.addWorldChild(right_foot.getObject());
	
	bones.addWorldChild(left_upperarm.getObject());
	bones.addWorldChild(right_upperarm.getObject());
	
	bones.addWorldChild(left_forearm.getObject());
	bones.addWorldChild(right_forearm.getObject());
	
	bones.addWorldChild(left_hand.getObject());
	bones.addWorldChild(right_hand.getObject());
	
	// ragdoll
	BodyRagdoll ragdoll = class_remove(new BodyRagdoll(mesh));
	ragdoll.setBones(bones);
	
	node_delete(bones);
	
	return ragdoll;
}


/*
 */
class RagDoll {
	
	BodyRagdoll body;
	
	ObjectMeshSkinned mesh;
	
	float time = 0.0f;
	
	RagDoll(Mat4 transform) {
		
		body = create_ragdoll(0,vec3_zero,transform);
		body.setRigidity(0.25f);
		
		mesh = node_cast(body.getObject());
		mesh.setNumLayers(3);
		mesh.setLayerAnimationFilePath(0,full_path("uniginescript_samples/physics/meshes/ragdoll_00.anim"));
		mesh.setLayerFrame(0,0.0f);
		mesh.setSpeed(25.0f);
		mesh.setLoop(1);
		mesh.play();
	}
	
	void postUpdate() {
		
		time += engine.game.getIFps();
		
		// enable animation
		if(time > 6.0f) {
			mesh.setLayerFrame(0,mesh.getTime());
			body.setFrameBased(1);
			time = 0.0f;
		}
		// interpolate from ragdoll to animation
		else if(time > 5.0f) {
			
			mesh.play();
			
			// copy ragdoll into the layer
			mesh.importLayer(1);
			
			// set animation frame
			mesh.setLayerFrame(0,mesh.getTime());
			
			// interpolate layers
			mesh.lerpLayer(0,1,0,time - 5.0f);
		}
		// enable ragdoll
		else if(time > 1.0f) {
			mesh.stop();
			body.setFrameBased(0);
		}
	}
};

/*
 */
int postUpdate() {
	
	foreach(RagDoll ragdoll; ragdolls) {
		ragdoll.postUpdate();
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(15.0f,0.0f,8.0f));
	createPlaneWithBody();
	setDescription("BodyRagdoll interpolation");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int width = 2;
	int height = 2;
	
	for(int j = -width; j <= width; j++) {
		for(int i = -height; i <= height; i++) {
			ragdolls.append(new RagDoll(Mat4(translate(i * 4.0f,j * 4.0f,0.0f) * rotateZ(90.0f))));
		}
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## random_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

Image image;
WidgetSprite sprite;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	setDescription("Random number generator");
	
	return 1;
}

/*
 */
int update() {
	
	int size = 512;
	
	if(image == NULL) {
		image = new Image();
		image.create2D(size,size,IMAGE_FORMAT_RGBA8);
		sprite = new WidgetSprite(engine.getGui());
		sprite.setImage(image);
		sprite.arrange();
		engine.gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
		image.setChannelInt(3,255);
	}
	
	forloop(int i = 0; 256) {
		
		int x = engine.game.getRandom(0,size);
		int y = engine.game.getRandom(0,size);
		
		float r = engine.game.getRandom(0.0f,1.0f);
		float g = engine.game.getRandom(0.0f,1.0f);
		float b = engine.game.getRandom(0.0f,1.0f);
		
		image.set2D(x,y,r,g,b);
	}
	
	sprite.setImage(image);
	
	return 1;
}

```

## render_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
ObjectGui object_gui;
WidgetSpriteNode sprite_node;
WidgetSprite sprite;
WidgetCanvas canvas;
Texture texture;

/*
 */
int create_line(int order,float x,float y,float radius,int num,float angle,float time) {
	int line = canvas.addLine(order);
	forloop(int i = 0; num + 1) {
		float s = sin(angle / num * DEG2RAD * i + time) * radius + x;
		float c = cos(angle / num * DEG2RAD * i + time) * radius + y;
		canvas.addLinePoint(line,vec3(s,c,0.0f));
	}
	return line;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	int size = 256;
	
	Gui gui = engine.getGui();
	
	// gui object
	object_gui = new ObjectGui(size * 2,size * 2);
	object_gui.setScreenSize(size * 2,size * 2);
	object_gui.setMaterial(findMaterialByName("gui_base"),"*");
	object_gui.setBackground(0);
	
	// render sprite
	float fov = 2.0f;
	sprite_node = new WidgetSpriteNode(gui,size * 2,size * 2);
	sprite_node.setProjection(ortho(-size,size,-size,size,-100.0f,100.0f) * perspective(fov,1.0f,0.01f,100.0f) * translate(0.0f,0.0f,-1.0f / tan(fov * DEG2RAD * 0.5f) - 5.0f));
	sprite_node.setNode(object_gui);
	sprite_node.appendSkipFlags(VIEWPORT_SKIP_FORMAT_RG11B10);
	sprite_node.appendSkipFlags(VIEWPORT_SKIP_VELOCITY_BUFFER);
	
	// render canvas
	canvas = new WidgetCanvas(object_gui.getGui());
	Gui(object_gui.getGui()).addChild(canvas,GUI_ALIGN_OVERLAP);
	
	// display sprite
	sprite = new WidgetSprite(gui);
	sprite.setWidth(size * 2);
	sprite.setHeight(size * 2);
	gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
	
	// scene object
	ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
	mesh.setMaterial(findMaterialByName("mesh_base"),"*");
	
	texture = new Texture();
	
	setDescription("Render to Texture");
	
	return 1;
}

/*
 */
int update() {
	
	canvas.clear();
	
	float time = engine.game.getTime();
	
	sprite_node.setModelview(Mat4(rotateY(sin(time * 4.0f))));
	
	// lines
	canvas.setLineColor(create_line(0,256.0f,256.0f,200.0f,3,360.0f,time),vec4(0.0f,1.0f,1.0f,1.0f));
	canvas.setLineColor(create_line(0,256.0f,256.0f,200.0f,4,360.0f,time),vec4(0.0f,0.0f,1.0f,1.0f));
	canvas.setLineColor(create_line(0,256.0f,256.0f,200.0f,5,360.0f,time),vec4(0.0f,1.0f,0.0f,1.0f));
	canvas.setLineColor(create_line(0,256.0f,256.0f,200.0f,7,360.0f,time),vec4(1.0f,0.0f,0.0f,1.0f));
	
	// border
	float min = 0.5f;
	float max = 511.5f;
	int line = canvas.addLine(0);
	canvas.addLinePoint(line,vec3(min,min,0.0f));
	canvas.addLinePoint(line,vec3(max,min,0.0f));
	canvas.addLinePoint(line,vec3(max,max,0.0f));
	canvas.addLinePoint(line,vec3(min,max,0.0f));
	canvas.addLinePoint(line,vec3(min,min,0.0f));
	
	// text
	int text = canvas.addText(1);
	canvas.setTextSize(text,32);
	canvas.setTextOutline(text,1);
	canvas.setTextText(text,"WidgetCanvas");
	canvas.setTextPosition(text,vec3(256.0f - canvas.getTextWidth(text) / 2.0f,64.0f,0.0f));
	
	// render texture
	object_gui.setEnabled(1);
	sprite_node.renderTexture(texture);
	object_gui.setEnabled(0);
	
	// update texture
	sprite.setRender(texture);
	
	return 1;
}

```

## restitution_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("Restitution");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 32;
	
	ObjectMeshDynamic plane = createPlaneWithBody();
	BodyDummy body = class_remove(new BodyDummy(plane));
	ShapeBox shape = class_remove(new ShapeBox(vec3(1000.0f,1000.0f,10.0f)));
	shape.setRestitution(1.0f);
	body.addShape(shape,translate(0.0f,0.0f,-5.0f));
	
	engine.physics.setLinearDamping(0.0f);
	
	forloop(int i = 0; size + 1) {
		
		float restitution = float(i) / size;
		
		createBodySphere(1.0f,1.0f,1.0f,restitution,get_material(i),Mat4(translate(-16.0f,(restitution - 0.5f) * 2.1f * size,16.0f)));
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## reverse_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
int play = 0;
float time = 0.0f;
float begin = INFINITY;
int frames = 0;
int scenes[0];

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
void flush_reverse() {
	
	if(play == 0 && engine.game.getTime() > begin && engine.physics.getFrame()) {
		scenes.append(engine.physics.saveScene());
		if(scenes.size() >= frames) {
			engine.physics.setEnabled(0);
			play = 1;
			time = 0.0f;
		}
	}
}

/*
 */
void update_reverse() {
	
	while(1) {
		
		float ifps = engine.physics.getIFps();
		
		if(play) {
			time += engine.game.getIFps() * engine.physics.getScale();
			while(scenes.size() > 1 && time > ifps) {
				engine.physics.removeScene(scenes[scenes.size() - 1]);
				scenes.remove();
				time -= ifps;
			}
			if(scenes.size() > 1) {
				engine.physics.setCurrentSubframeTime(max(ifps - time,0.0f));
				engine.physics.restoreScene(scenes[scenes.size() - 1]);
			} else {
				engine.physics.setCurrentSubframeTime(0.0f);
				engine.physics.restoreScene(scenes[0]);
				engine.physics.setEnabled(1);
				play = 0;
			}
		}
		
		wait;
	}
}

/*
 */
void reverse(float b,int f) {
	begin = b;
	frames = f;
	thread("update_reverse");
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Time Reverse");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 8;
	float space = 1.01f;
	
	for(int k = 0; k < size; k++) {
		for(int j = 0; j < size - k; j++) {
			for(int i = 0; i < size - k; i++) {
				createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(i + k * 0.5f - size * 0.5f + 0.5f,j + k * 0.5f - size * 0.5f + 0.5f,k + 0.5f) * space));
			}
		}
	}
	
	void create_body_sphere(Mat4 transform) {
		createBodyBox(vec3(20.0f,4.0f,1.0f),0.0f,0.5f,0.5f,get_material(1),transform * translate(0.0f,0.0f,9.0f) * rotateY(60.0f));
		createBodySphere(2.0f,4.0f,0.5f,1.0f,get_material(0),transform * translate(-2.0f,0.0f,30.0f));
	}
	
	create_body_sphere(Mat4(rotateZ( 45.0f) * translate(-24.0f,0.0f,0.0f)));
	create_body_sphere(Mat4(rotateZ(-45.0f) * translate(-24.0f,0.0f,0.0f)));
	
	reverse(0.5f,350);
	
	return 1;
}

/*
 */
void updatePhysics() {
	
	flush_reverse();
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## reverse_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
int play = 0;
float time = 0.0f;
float begin = INFINITY;
int frames = 0;
int scenes[0];

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
void flush_reverse() {
	
	if(play == 0 && engine.game.getTime() > begin && engine.physics.getFrame()) {
		scenes.append(engine.physics.saveScene());
		if(scenes.size() >= frames) {
			engine.physics.setEnabled(0);
			play = 1;
			time = 0.0f;
		}
	}
}

/*
 */
void update_reverse() {
	
	while(1) {
		
		float ifps = engine.physics.getIFps();
		
		if(play) {
			time += engine.game.getIFps() * engine.physics.getScale();
			while(scenes.size() > 1 && time > ifps) {
				engine.physics.removeScene(scenes[scenes.size() - 1]);
				scenes.remove();
				time -= ifps;
			}
			if(scenes.size() > 1) {
				engine.physics.setCurrentSubframeTime(max(ifps - time,0.0f));
				engine.physics.restoreScene(scenes[scenes.size() - 1]);
			} else {
				engine.physics.setCurrentSubframeTime(0.0f);
				engine.physics.restoreScene(scenes[0]);
				engine.physics.setEnabled(1);
				play = 0;
			}
		}
		
		wait;
	}
}

/*
 */
void reverse(float b,int f) {
	begin = b;
	frames = f;
	thread("update_reverse");
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Time Reverse");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 16;
	float space = 1.01f;
	
	for(int j = 0; j < size; j++) {
		for(int i = 0; i < size - j; i++) {
			BodyRigid body = createBodyBox(vec3(1.0f),1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(0.0f,i + j * 0.5f - size * 0.5f + 0.5f,j + 8.0f) * space));
			body.setLinearScale(vec3(0.0f,1.0f,1.0f));
			body.setAngularScale(vec3(1.0f,0.0f,0.0f));
		}
	}
	
	createBodyBox(vec3(3.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3(0.0f, 3.0f,1.5f)));
	createBodyBox(vec3(3.0f),1.0f,0.5f,0.5f,get_material(0),translate(Vec3(0.0f,-3.0f,1.5f)));
	
	reverse(0.0f,300);
	
	return 1;
}

/*
 */
void updatePhysics() {
	
	flush_reverse();
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## reverse_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
int play = 0;
float time = 0.0f;
float begin = INFINITY;
int frames = 0;
int scenes[0];

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
void flush_reverse() {
	
	if(play == 0 && engine.game.getTime() > begin && engine.physics.getFrame()) {
		scenes.append(engine.physics.saveScene());
		if(scenes.size() >= frames) {
			engine.physics.setEnabled(0);
			play = 1;
			time = 0.0f;
		}
	}
}

/*
 */
void update_reverse() {
	
	while(1) {
		
		float ifps = engine.physics.getIFps();
		
		if(play) {
			time += engine.game.getIFps() * engine.physics.getScale();
			while(scenes.size() > 1 && time > ifps) {
				engine.physics.removeScene(scenes[scenes.size() - 1]);
				scenes.remove();
				time -= ifps;
			}
			if(scenes.size() > 1) {
				engine.physics.setCurrentSubframeTime(max(ifps - time,0.0f));
				engine.physics.restoreScene(scenes[scenes.size() - 1]);
			} else {
				engine.physics.setCurrentSubframeTime(0.0f);
				engine.physics.restoreScene(scenes[0]);
				engine.physics.setEnabled(1);
				play = 0;
			}
		}
		
		wait;
	}
}

/*
 */
void reverse(float b,int f) {
	begin = b;
	frames = f;
	thread("update_reverse");
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(50.0f,0.0f,50.0f));
	createPlaneWithBody();
	setDescription("Time Reverse");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 16;
	float space = 3.01f;
	
	for(int j = 0; j < size; j++) {
		for(int i = 0; i < size - j; i++) {
			createBodyBox(vec3(3.0f),0.1f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(-28.0f,i + j * 0.5f - size * 0.5f + 0.5f,j + 0.5f) * space));
		}
	}
	
	createSpringal(0.3f,material_names,Mat4(translate( 0.0f,-40.0f,0.0f) * rotateZ(-20.0f)));
	createSpringal(0.2f,material_names,Mat4(translate( 8.0f,-20.0f,0.0f) * rotateZ( -8.0f)));
	createSpringal(0.1f,material_names,Mat4(translate(10.0f,  0.0f,0.0f) * rotateZ(  0.0f)));
	createSpringal(0.2f,material_names,Mat4(translate( 8.0f, 20.0f,0.0f) * rotateZ(  8.0f)));
	createSpringal(0.3f,material_names,Mat4(translate( 0.0f, 40.0f,0.0f) * rotateZ( 20.0f)));
	
	reverse(0.0f,400);
	
	return 1;
}

/*
 */
void updatePhysics() {
	
	unlockSpringals();
	
	flush_reverse();
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## rocket_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
class Rocket {
	
	BodyRigid body;
	ObjectParticles particles;
	
	Rocket(mat4 transform) {
		
		body = createBodyCapsule(0.2f,1.8f,10.0f,0.5f,0.5f,get_material(1),transform);
		body.setAngularDamping(0.5f);
		
		particles = addToEditor(new ObjectParticles());
		particles.setWorldTransform(translate(Vec3(0.0f,0.0f,-1.2f)));
		particles.setMaterial(findMaterialByName("particles_base"),"*");
		particles.setSurfaceProperty("surface_base","*");
		particles.setEmitterEnabled(1);
		particles.setLife(0.08f,0.01f);
		
		ParticleModifierScalar radius_modifier = particles.getRadiusOverTimeModifier();
		radius_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
		radius_modifier.setConstant(0.1f);
		
		ParticleModifierScalar growth_modifier = particles.getGrowthOverTimeModifier();
		growth_modifier.setMode(PARTICLE_MODIFIER_MODE_RANDOM_BETWEEN_TWO_CONSTANTS);
		growth_modifier.setConstantMin(1.1f);
		growth_modifier.setConstantMax(2.9f);
		
		Object object = body.getObject();
		object.addChild(particles);
	}
	
	void update() {
		
		Mat4 transform = body.getTransform();
		Mat4 itransform = inverse(transform);
		
		Vec3 position = transform.m03m13m23;
		vec3 direction = transform.m02m12m22;
		vec3 nozzle = position - direction * 1.2f;
		
		float scale = 2.0f;
		vec3 force = normalize(vec3(direction.x * scale,direction.y * scale,direction.z));
		
		force *= clamp(40.0f - position.z,5.0f,20.0f) * 9.0f;
		if(direction.z < 0.2f) force *= 0.0f;
		
		body.addWorldForce(nozzle,force);
		
		particles.setSpawnRate(length(force) * 10.0f);
		
		ParticleModifierScalar velocity_modifier = particles.getVelocityOverTimeModifier();
		velocity_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
		velocity_modifier.setConstant(length(force) * 0.25f);
		
		ParticleModifierVector direction_modifier = particles.getDirectionOverTimeModifier();
		direction_modifier.setMode(PARTICLE_MODIFIER_MODE_RANDOM_BETWEEN_TWO_CONSTANTS);
		direction_modifier.setConstantMin(rotation(itransform) * vec3(-1.0f,-1.0f,-11.0f));
		direction_modifier.setConstantMax(rotation(itransform) * vec3(1.0f,1.0f,-9.0f));
	}
};

Rocket rockets[0];

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,40.0f));
	createPlaneWithBody();
	setDescription("Real rockets");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	engine.physics.setGravity(vec3(0.0f,0.0f,-20.0f));
	
	BodyRigid body = createBodyBox(vec3(10.0f,10.0f,1.0f),5.0f,1.0f,0.0f,get_material(0),translate(Vec3(0.0f,0.0f,25.0f)));
	body.setFreezable(0);
	
	for(int j = -4; j <= 4; j++) {
		for(int i = -4; i <= 4; i++) {
			rockets.append(new Rocket(translate(Vec3(-j,i,10.0f))));
		}
	}
	
	return 1;
}

/*
 */
int updatePhysics() {
	
	foreach(Rocket rocket; rockets) rocket.update();
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## rope_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string wire_material_names[] = ( "physics_wire_red", "physics_wire_green", "physics_wire_blue", "physics_wire_orange", "physics_wire_yellow" );

string get_wire_material(int material) {
	return wire_material_names[abs(material) % wire_material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
Body create_body_rope(float radius,float height,float step,float mass,Mat4 transform) {
	
	BodyRope rope = createBodyRope(radius,height,step,mass,0.5f,0.5f,get_wire_material(0),transform);
	rope.setNumIterations(16);
	rope.setLinearDamping(0.5f);
	rope.setAngularRestitution(0.2f);
	rope.setLinearThreshold(1.1f);
	
	return rope;
}

/*
 */
Body create_bar(Body b0,Body b1,vec3 size,float density,string material,Mat4 transform,float pin) {
	
	Body body = createBodyBox(size,density,0.5f,0.5f,material,transform);
	
	class_remove(new JointParticles(body,b0,transform * Vec3( size.x / 2.0f,0.0f,0.0f),vec3(pin)));
	class_remove(new JointParticles(body,b1,transform * Vec3(-size.x / 2.0f,0.0f,0.0f),vec3(pin)));
	
	return body;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyRope");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	float height = 18.0f;
	
	Body b0 = create_body_rope(0.01f,25.5f,0.25f,200.0f,Mat4(translate( 2.5f,0.0f,height) * rotateX(-90.0f)));
	Body b1 = create_body_rope(0.01f,25.5f,0.25f,200.0f,Mat4(translate(-2.5f,0.0f,height) * rotateX(-90.0f)));
	
	Body b2 = create_bar(b0,b1,vec3(5.0f,1.0f,1.0f),0.0f,get_material(2),translate(Vec3(0.0f,-12.75f,height)),1.0f);
	Body b3 = create_bar(b0,b1,vec3(5.0f,1.0f,1.0f),0.0f,get_material(2),translate(Vec3(0.0f, 12.75f,height)),1.0f);
	
	forloop(int i = 1; 17) {
		create_bar(b0,b1,vec3(5.0f,1.0f,0.5f),5.0f,get_material(1),translate(Vec3(0.0f,-12.75f + i * 1.5f,height)),0.5f);
	}
	
	Body b4 = create_body_rope(0.01f,10.0f,0.5f,200.0f,translate(Vec3( 2.5f,-17.75f,height)) * rotateX(-90.0f));
	Body b5 = create_body_rope(0.01f,10.0f,0.5f,200.0f,translate(Vec3(-2.5f,-17.75f,height)) * rotateX(-90.0f));
	
	Body b6 = create_body_rope(0.01f,10.0f,0.5f,200.0f,translate(Vec3( 2.5f, 17.75f,height)) * rotateX(-90.0f));
	Body b7 = create_body_rope(0.01f,10.0f,0.5f,200.0f,translate(Vec3(-2.5f, 17.75f,height)) * rotateX(-90.0f));
	
	class_remove(new JointParticles(b2,b4,Vec3( 2.5f,-12.75f,height),vec3(1.0f)));
	class_remove(new JointParticles(b2,b5,Vec3(-2.5f,-12.75f,height),vec3(1.0f)));
	
	class_remove(new JointParticles(b3,b6,Vec3( 2.5f, 12.75f,height),vec3(1.0f)));
	class_remove(new JointParticles(b3,b7,Vec3(-2.5f, 12.75f,height),vec3(1.0f)));
	
	create_bar(b4,b5,vec3(5.0f,3.0f,3.0f),50.0f,get_material(3),translate(Vec3(0.0f,-22.75f,height)),2.0f);
	create_bar(b6,b7,vec3(5.0f,3.0f,3.0f),50.0f,get_material(3),translate(Vec3(0.0f, 22.75f,height)),2.0f);
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## rope_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string wire_material_names[] = ( "physics_wire_red", "physics_wire_green", "physics_wire_blue", "physics_wire_orange", "physics_wire_yellow" );

string get_wire_material(int material) {
	return wire_material_names[abs(material) % wire_material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyRope");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	float height = 18.0f;
	float radius = 10.0f;
	
	Body body = createBodyCylinder(2.0f,1.0f,0.0f,0.5f,0.5f,get_material(1),translate(Vec3(0.0f,0.0f,height + 1.0f)));
	Body wheel = createBodyCylinder(2.0f,1.0f,1000.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.0f,0.0f,height)));
	
	JointHinge joint = class_remove(new JointHinge(body,wheel,Vec3(0.0f,0.0f,height),vec3(0.0f,0.0f,1.0f)));
	joint.setAngularTorque(400000.0f);
	joint.setAngularVelocity(4.0f);
	joint.setNumIterations(4);
	
	int num = 12;
	
	forloop(int i = 0; num) {
		
		Mat4 rotate = Mat4(rotateZ(360.0f * i / num));
		
		BodyRope rope = createBodyRope(0.01f,radius,0.5f,3.0f,0.5f,0.5f,get_wire_material(4),rotate * translate(0.0f,radius / 2.0f,height) * rotateX(90.0f));
		rope.setNumIterations(16);
		rope.setLinearDamping(0.5f);
		
		class_remove(new JointParticles(wheel,rope,Vec3(0.0f,0.0f,height),vec3(4.0f)));
		
		Node node = createBodyHomuncle(0,vec3(0.0f),material_names,rotate * translate(0.0f,radius - 0.1f,height - 3.8f));
		Body homuncle = Object(node_cast(node.getChild(0))).getBody();
		
		class_remove(new JointParticles(homuncle,rope,rotate * Vec3(0.0f,radius,height),vec3(1.5f)));
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## rope_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string wire_material_names[] = ( "physics_wire_red", "physics_wire_green", "physics_wire_blue", "physics_wire_orange", "physics_wire_yellow" );

string get_wire_material(int material) {
	return wire_material_names[abs(material) % wire_material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyRope");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int num = 16;
	
	float radius = 12.0f;
	float height = 20.0f;
	
	forloop(int i = 0; num) {
		
		float x = (i - num / 2) * 1.2f;
		
		Body body_0 = createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(0),translate(Vec3(x,radius,height)));
		Body body_1 = createBodySphere(0.5f,10.0f,0.5f,0.5f,get_material(0),translate(Vec3(x,-radius,height)));
		
		BodyRope rope = createBodyRope(0.01f,radius * 2.0f,0.25f,80.0f,0.5f,0.5f,get_wire_material(4),translate(Vec3(x,0.0f,height)) * rotateX(90.0f));
		rope.setNumIterations(8);
		rope.setRadius(0.15f);
		
		class_remove(new JointParticles(body_0,rope,Vec3(x,radius,height),vec3(1.2f)));
		class_remove(new JointParticles(body_1,rope,Vec3(x,-radius,height),vec3(1.2f)));
	}
	
	createBodyCylinder(1.0f,32.0f,0.0f,0.5f,0.5f,get_material(1),translate(Vec3(0.0f,5.0f,16.0f)) * rotateY(90.0f));
	createBodyCylinder(1.0f,32.0f,0.0f,0.5f,0.5f,get_material(1),translate(Vec3(0.0f,-5.0f,16.0f)) * rotateY(90.0f));
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## rope_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string wire_material_names[] = ( "physics_wire_red", "physics_wire_green", "physics_wire_blue", "physics_wire_orange", "physics_wire_yellow" );

string get_wire_material(int material) {
	return wire_material_names[abs(material) % wire_material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyRope");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int width = 7;
	int height = 7;
	vec3 size = vec3(8.0f,8.0f,1.0f);
	
	Body body_0 = createBodyBox(size,2.0f,0.5f,0.5f,get_material(0),translate(Vec3(0.0f,0.0f,10.0f)));
	Body body_1 = createBodyBox(size,2.0f,0.5f,0.5f,get_material(0),translate(Vec3(0.0f,0.0f,15.0f)));
	Body body_2 = createBodyBox(size,2.0f,0.5f,0.5f,get_material(0),translate(Vec3(0.0f,0.0f,20.0f)));
	
	forloop(int y = 1; height) {
		forloop(int x = 1; width) {
			
			Vec3 position = Vec3(float(x) / width - 0.5f,float(y) / height - 0.5f,15.0f) * size;
			
			BodyRope rope = createBodyRope(0.2f,20.0f,0.5f,20.0f,0.5f,0.5f,get_wire_material(1),translate(position));
			rope.setNumIterations(8);
			
			class_remove(new JointParticles(body_0,rope,position + Vec3(0.0f,0.0f,-5.0f),vec3(3.0f)));
			class_remove(new JointParticles(body_1,rope,position + Vec3(0.0f,0.0f, 0.0f),vec3(3.0f)));
			class_remove(new JointParticles(body_2,rope,position + Vec3(0.0f,0.0f, 5.0f),vec3(3.0f)));
		}
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## route_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
string material_names[] = ( "paths_red", "paths_green", "paths_blue", "paths_orange", "paths_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
class Coin {
	
	float angle;
	Vec3 position;
	Object object;
	Obstacle obstacle;
	
	Coin() {
		
		angle = engine.game.getRandom(0.0f,360.0f);
		
		object = createObjectCylinder(1.0f,0.25f,get_material(4),Mat4_identity);
		object.setEnabled(0);
		
		obstacle = new ObstacleSphere(2.0f);
		object.addChild(obstacle);
	}
	~Coin() {
		removeFromEditor(object);
		delete obstacle;
	}
	
	int isEnabled() {
		return object.isEnabled();
	}
	void setEnabled(int enable) {
		object.setEnabled(enable);
	}
	
	void setPosition(Vec3 position_) {
		position = position_;
	}
	
	void update() {
		angle += engine.game.getIFps() * 256.0f;
		object.setWorldTransform(translate(position) * rotateZ(angle) * rotateX(90.0f));
	}
};

/*
 */
class Robot2D {
	
	Coin coin;
	int counter;
	Vec3 position;
	quat orientation;
	Object object;
	Obstacle obstacle;
	PathRoute route;
	vec3 offset = vec3(0.0f,0.0f,2.0f);
	
	Robot2D(Vec3 position_) {
		
		coin = new Coin();
		
		position = position_;
		orientation = quat(0.0f,0.0f,1.0f,90.0f);
		
		object = createObjectBox(vec3(2.0f),get_material(0),translate(position));
		object.addChild(createObjectCylinder(1.0f,0.5f,get_material(1),Mat4(rotateY(90.0f) * translate(0.0f,0.0f, 1.0f))));
		object.addChild(createObjectCylinder(1.0f,0.5f,get_material(1),Mat4(rotateY(90.0f) * translate(0.0f,0.0f,-1.0f))));
		
		obstacle = new ObstacleSphere(2.0f);
		object.addChild(obstacle);
		
		route = new PathRoute(2.0f);
		route.setExcludeObstacles((obstacle,coin.obstacle));
		route.setMaxAngle(0.5f);
	}
	~Robot2D() {
		delete coin;
		removeFromEditor(object);
		delete obstacle;
		delete route;
	}
	
	int needCoin() {
		return (coin.isEnabled() == 0);
	}
	void createCoin(Vec3 position_) {
		coin.setPosition(position_);
		coin.setEnabled(1);
		coin.update();
		engine.world.updateSpatial();
		route.create2D(position + offset,coin.position + offset);
		if(route.isReached() == 0) removeCoin();
	}
	void removeCoin() {
		coin.setEnabled(0);
	}
	
	void update(int visualizer) {
		
		vec3 p = object.getWorldPosition();
		position.x = p.x;
		position.y = p.y;

		if(coin.isEnabled()) {
			
			coin.update();
			
			if(length(position - coin.position) < 2.0f || counter++ > 32) {
				if(counter > 32) engine.console.onscreenMessage("PathRoute failed\n");
				removeCoin();
			}
			else if(route.isReady()) {
				counter = 0;
				if(route.isReached()) {
					float ifps = engine.game.getIFps();
					Vec3 direction = route.getPoint(1) - route.getPoint(0);
					if(length(direction) > EPSILON) orientation = lerp(orientation,quat(setTo(Vec3_zero,direction,vec3(0.0f,0.0f,1.0f), AXIS_NY)),ifps * 8.0f);
					position += object.getWorldDirection(AXIS_NY) * ifps * 16.0f;
					object.setWorldTransform(translate(position) * orientation);
					if(visualizer) route.renderVisualizer(vec4_one);
					route.create2D(position + offset,coin.position + offset,1);
				} else {
					engine.console.onscreenMessage("PathRoute failed\n");
					removeCoin();
				}
			}
			else if(route.isQueued() == 0) {
				route.create2D(position + offset,coin.position + offset,1);
			}
		}
	}
};

/*
 */
Robot2D robots[0];
Vec3 offset = Vec3(0.0f,0.0f,0.0f);
Vec3 min = Vec3(-256.0f,-256.0f,1.0f) + offset;
Vec3 max = Vec3( 256.0f, 256.0f,1.0f) + offset;

using Unigine::Samples;

/*
 */
int update() {
	
	if(engine.game.isEnabled()) {
		forloop(int i = 0; robots.size()) {
			Robot2D robot = robots[i];
			robot.update((i < 4));
			if(robot.needCoin()) robot.createCoin(engine.game.getRandom(min,max));
		}
	}
	
	return 1;
}

/*
 */
int init() {
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	NavigationSector navigation = addToEditor(new NavigationSector(vec3(1024.0f,1024.0f,9.0f)));
	navigation.setWorldTransform(translate(Vec3(0.0f,0.0f,5.0f) + offset));
	
	int num_robots = 64;
	forloop(int i = 0; num_robots) {
		robots.append(new Robot2D(engine.game.getRandom(min,max)));
	}
	
	setDescription(format("%d PathRoute2D in NavigationSector with %d obstacles",num_robots,num_robots * 2));

	if(engine.visualizer.isEnabled() != 1) {
		engine.visualizer.setEnabled(1);
	}

	return 1;
}

/*
*/
int shutdown() {

	engine.console.setOnscreen(0);
	engine.visualizer.setEnabled(0);
	return 1;
}

```

## route_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
string material_names[] = ( "paths_red", "paths_green", "paths_blue", "paths_orange", "paths_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
class Coin {
	
	float angle;
	Vec3 position;
	Object object;
	Obstacle obstacle;
	
	Coin() {
		
		angle = engine.game.getRandom(0.0f,360.0f);
		
		object = createObjectCylinder(1.0f,0.25f,get_material(4),Mat4_identity);
		object.setEnabled(0);
		
		obstacle = new ObstacleSphere(2.0f);
		object.addChild(obstacle);
	}
	~Coin() {
		removeFromEditor(object);
		delete obstacle;
	}
	
	int isEnabled() {
		return object.isEnabled();
	}
	void setEnabled(int enable) {
		object.setEnabled(enable);
	}
	
	void setPosition(Vec3 position_) {
		position = position_;
	}
	
	void update() {
		angle += engine.game.getIFps() * 256.0f;
		object.setWorldTransform(translate(position) * rotateZ(angle) * rotateX(90.0f));
	}
};

/*
 */
class Robot2D {
	
	Coin coin;
	int counter;
	Vec3 position;
	quat orientation;
	Object object;
	Obstacle obstacle;
	PathRoute route;
	vec3 offset = vec3(0.0f,0.0f,2.0f);
	
	Robot2D(Vec3 position_) {
		
		coin = new Coin();
		
		position = position_;
		orientation = quat(0.0f,0.0f,1.0f,90.0f);
		
		object = createObjectBox(vec3(2.0f),get_material(0),translate(position));
		object.addChild(createObjectCylinder(1.0f,0.5f,get_material(1),Mat4(rotateY(90.0f) * translate(0.0f,0.0f, 1.0f))));
		object.addChild(createObjectCylinder(1.0f,0.5f,get_material(1),Mat4(rotateY(90.0f) * translate(0.0f,0.0f,-1.0f))));
		
		obstacle = new ObstacleSphere(2.0f);
		object.addChild(obstacle);
		
		route = new PathRoute(2.0f);
		route.setExcludeObstacles((obstacle,coin.obstacle));
		route.setMaxAngle(0.5f);
	}
	~Robot2D() {
		delete coin;
		removeFromEditor(object);
		delete obstacle;
		delete route;
	}
	
	int needCoin() {
		return (coin.isEnabled() == 0);
	}
	void createCoin(Vec3 position_) {
		coin.setPosition(position_);
		coin.setEnabled(1);
		coin.update();
		engine.world.updateSpatial();
		route.create2D(position + offset,coin.position + offset);
		if(route.isReached() == 0) removeCoin();
	}
	void removeCoin() {
		coin.setEnabled(0);
	}
	
	void update(int visualizer) {
		
		vec3 p = object.getWorldPosition();
		position.x = p.x;
		position.y = p.y;
		
		if(coin.isEnabled()) {
			
			coin.update();
			
			if(length(position - coin.position) < 2.0f || counter++ > 32) {
				if(counter > 32) engine.console.onscreenMessage("PathRoute failed\n");
				removeCoin();
			}
			else if(route.isReady()) {
				counter = 0;
				if(route.isReached()) {
					float ifps = engine.game.getIFps();
					Vec3 direction = route.getPoint(1) - route.getPoint(0);
					if(length(direction) > EPSILON) orientation = lerp(orientation,quat(setTo(Vec3_zero,direction,vec3(0.0f,0.0f,1.0f), AXIS_NY)), ifps * 8.0f);
					position += object.getWorldDirection(AXIS_NY) * ifps * 16.0f;
					object.setWorldTransform(translate(position) * orientation);
					if(visualizer) route.renderVisualizer(vec4_one);
					route.create2D(position + offset,coin.position + offset,1);
				} else {
					engine.console.onscreenMessage("PathRoute failed\n");
					removeCoin();
				}
			}
			else if(route.isQueued() == 0) {
				route.create2D(position + offset,coin.position + offset,1);
			}
		}
	}
};

/*
 */
Robot2D robots[0];
Vec3 offset = Vec3(0.0f,0.0f,0.0f);
Vec3 min = Vec3(-256.0f,-256.0f,1.0f) + offset;
Vec3 max = Vec3( 256.0f, 256.0f,1.0f) + offset;

using Unigine::Samples;

/*
 */
int update() {
	
	if(engine.game.isEnabled()) {
		forloop(int i = 0; robots.size()) {
			Robot2D robot = robots[i];
			robot.update((i < 4));
			if(robot.needCoin()) robot.createCoin(engine.game.getRandom(min,max));
		}
	}
	
	return 1;
	}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	NavigationSector navigation = addToEditor(new NavigationSector(vec3(1024.0f,1024.0f,9.0f)));
	navigation.setWorldTransform(translate(Vec3(0.0f,0.0f,5.0f) + offset));
	
	Object mesh;
	int num_obstacles = 64;
	forloop(int i = 0; num_obstacles) {
		vec3 size = engine.game.getRandom(vec3(10.0f,10.0f,4.0f),vec3(40.0f,40.0f,20.0f));
		switch(i % 3) {
			case 0:
				mesh = createObjectBox(size,get_material(2),translate(engine.game.getRandom(min,max)));
				mesh.addChild(addToEditor(new ObstacleBox(size)));
				break;
			case 1:
				mesh = createObjectSphere(size.x / 2.0f,get_material(2),translate(engine.game.getRandom(min,max)));
				mesh.addChild(addToEditor(new ObstacleSphere(size.x / 2.0f)));
				break;
			case 2:
				mesh = createObjectCapsule(size.x / 2.0f,size.y / 2.0f,get_material(2),translate(engine.game.getRandom(min,max)));
				mesh.addChild(addToEditor(new ObstacleCapsule(size.x / 2.0f,size.y / 2.0f)));
				break;
		}
	}
	
	int num_robots = 16;
	forloop(int i = 0; num_robots) {
		robots.append(new Robot2D(engine.game.getRandom(min,max)));
	}
	
	setDescription(format("%d PathRoute2D in NavigationSector with %d obstacles",num_robots,num_obstacles + num_robots * 2));

	if(engine.visualizer.isEnabled() != 1) {
		engine.visualizer.setEnabled(1);
	}

	return 1;
}

/*
*/
int shutdown() {

	engine.console.setOnscreen(0);
	engine.visualizer.setEnabled(0);
	return 1;
}

```

## route_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
string material_names[] = ( "paths_red", "paths_green", "paths_blue", "paths_orange", "paths_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
class Coin {
	
	float angle;
	Vec3 position;
	Object object;
	Obstacle obstacle;
	
	Coin() {
		
		angle = engine.game.getRandom(0.0f,360.0f);
		
		object = createObjectCylinder(1.0f,0.25f,get_material(4),Mat4_identity);
		object.setEnabled(0);
		
		obstacle = new ObstacleSphere(2.0f);
		object.addChild(obstacle);
	}
	~Coin() {
		removeFromEditor(object);
		delete obstacle;
	}
	
	int isEnabled() {
		return object.isEnabled();
	}
	void setEnabled(int enable) {
		object.setEnabled(enable);
	}
	
	void setPosition(Vec3 position_) {
		position = position_;
	}
	
	void update() {
		angle += engine.game.getIFps() * 256.0f;
		object.setWorldTransform(translate(position) * rotateZ(angle) * rotateX(90.0f));
	}
};

/*
 */
class Robot3D {
	
	Coin coin;
	int counter;
	Vec3 position;
	quat orientation;
	Object object;
	Obstacle obstacle;
	PathRoute route;
	
	Robot3D(Vec3 position_) {
		
		coin = new Coin();
		
		position = position_;
		orientation = quat(1.0f,0.0f,0.0f,90.0f);
		
		object = createObjectBox(vec3(2.0f),get_material(0),translate(position));
		object.addChild(createObjectCylinder(0.5f,2.5f,get_material(1),translate(Vec3( 0.0f, 1.5f,0.0f))));
		object.addChild(createObjectCylinder(0.5f,2.5f,get_material(1),translate(Vec3( 0.0f,-1.5f,0.0f))));
		object.addChild(createObjectCylinder(0.5f,2.5f,get_material(1),translate(Vec3( 1.5f, 0.0f,0.0f))));
		object.addChild(createObjectCylinder(0.5f,2.5f,get_material(1),translate(Vec3(-1.5f, 0.0f,0.0f))));
		
		obstacle = new ObstacleSphere(2.0f);
		object.addChild(obstacle);
		
		route = new PathRoute(2.0f);
		route.setExcludeObstacles((obstacle,coin.obstacle));
	}
	~Robot3D() {
		delete coin;
		removeFromEditor(object);
		delete obstacle;
		delete route;
	}
	
	int needCoin() {
		return (coin.isEnabled() == 0);
	}
	void createCoin(Vec3 pos) {
		coin.setPosition(pos);
		coin.setEnabled(1);
		coin.update();
		engine.world.updateSpatial();
		route.create3D(position,coin.position);
		if(route.isReached() == 0) removeCoin();
	}
	void removeCoin() {
		coin.setEnabled(0);
	}
	
	void update(int visualizer) {
		
		position = object.getWorldPosition();
		
		if(coin.isEnabled()) {
			
			coin.update();
			
			if(length(position - coin.position) < 2.0f || counter++ > 32) {
				if(counter > 32) engine.console.onscreenMessage("PathRoute failed\n");
				removeCoin();
			}
			else if(route.isReady()) {
				counter = 0;
				if(route.isReached()) {
					float ifps = engine.game.getIFps();
					Vec3 direction = route.getPoint(1) - route.getPoint(0);
					if(length(direction) > EPSILON) orientation = lerp(orientation,quat(setTo(Vec3_zero,direction,vec3(0.0f,0.0f,1.0f))),ifps * 8.0f);
					position += object.getWorldDirection() * ifps * 16.0f;
					object.setWorldTransform(translate(position) * orientation);
					if(visualizer) route.renderVisualizer(vec4_one);
					route.create3D(position,coin.position,1);
				} else {
					engine.console.onscreenMessage("PathRoute failed\n");
					removeCoin();
				}
			}
			else if(route.isQueued() == 0) {
				route.create3D(position,coin.position,1);
			}
		}
	}
};


/*
 */
Robot3D robots[0];
Vec3 offset = Vec3(0.0f,0.0f,0.0f);
Vec3 min = Vec3(-128.0f,-128.0f,1.0f) + offset;
Vec3 max = Vec3( 128.0f, 128.0f,64.0f) + offset;

using Unigine::Samples;

/*
 */
int update() {
	
	if(engine.game.isEnabled()) {
		forloop(int i = 0; robots.size()) {
			Robot3D robot = robots[i];
			robot.update((i < 4));
			if(robot.needCoin()) robot.createCoin(engine.game.getRandom(min,max));
		}
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	NavigationSector navigation = addToEditor(new NavigationSector(vec3(1024.0f,1024.0f,256.0f)));
	navigation.setWorldTransform(translate(Vec3(0.0f,0.0f,129.0f) + offset));
	
	Object mesh;
	int num_obstacles = 30;
	forloop(int i = 0; num_obstacles) {
		vec3 size = engine.game.getRandom(vec3(10.0f,10.0f,10.0f),vec3(40.0f,40.0f,40.0f));
		switch(i % 3) {
			case 0:
				mesh = createObjectBox(size,get_material(2),translate(engine.game.getRandom(min,max)));
				mesh.addChild(addToEditor(new ObstacleBox(size)));
				break;
			case 1:
				mesh = createObjectSphere(size.x / 2.0f,get_material(2),translate(engine.game.getRandom(min,max)));
				mesh.addChild(addToEditor(new ObstacleSphere(size.x / 2.0f)));
				break;
			case 2:
				mesh = createObjectCapsule(size.x / 2.0f,size.y / 2.0f,get_material(2),translate(engine.game.getRandom(min,max)));
				mesh.addChild(addToEditor(new ObstacleCapsule(size.x / 2.0f,size.y / 2.0f)));
				break;
		}
	}
	
	int num_robots = 16;
	forloop(int i = 0; num_robots) {
		robots.append(new Robot3D(engine.game.getRandom(min,max)));
	}
	
	setDescription(format("%d PathRoute3D in NavigationSector with %d obstacles",num_robots,num_obstacles + num_robots * 2));

	if(engine.visualizer.isEnabled() != 1) {
		engine.visualizer.setEnabled(1);
	}

	return 1;
}

/*
*/
int shutdown() {

	engine.console.setOnscreen(0);
	engine.visualizer.setEnabled(0);
	return 1;
}

```

## route_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
string material_names[] = ( "paths_red", "paths_green", "paths_blue", "paths_orange", "paths_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
class Coin {
	
	float angle;
	Vec3 position;
	Object object;
	Obstacle obstacle;
	
	Coin() {
		
		angle = engine.game.getRandom(0.0f,360.0f);
		
		object = createObjectCylinder(1.0f,0.25f,get_material(4),Mat4_identity);
		object.setEnabled(0);
		
		obstacle = new ObstacleSphere(2.0f);
		object.addChild(obstacle);
	}
	~Coin() {
		removeFromEditor(object);
		delete obstacle;
	}
	
	int isEnabled() {
		return object.isEnabled();
	}
	void setEnabled(int enable) {
		object.setEnabled(enable);
	}
	
	void setPosition(Vec3 position_) {
		position = position_;
	}
	
	void update() {
		angle += engine.game.getIFps() * 256.0f;
		object.setWorldTransform(translate(position) * rotateZ(angle) * rotateX(90.0f));
	}
};

/*
 */
class Robot2D {
	
	Coin coin;
	int counter;
	Vec3 position;
	quat orientation;
	Object object;
	Obstacle obstacle;
	PathRoute route;
	vec3 offset = vec3(0.0f,0.0f,2.0f);
	
	Robot2D(Vec3 position_) {
		
		coin = new Coin();
		
		position = position_;
		orientation = quat(1.0f,0.0f,0.0f,90.0f);
		
		object = createObjectBox(vec3(2.0f),get_material(0),translate(position));
		object.addChild(createObjectCylinder(1.0f,0.5f,get_material(1),Mat4(rotateY(90.0f) * translate(0.0f,0.0f, 1.0f))));
		object.addChild(createObjectCylinder(1.0f,0.5f,get_material(1),Mat4(rotateY(90.0f) * translate(0.0f,0.0f,-1.0f))));
		
		obstacle = new ObstacleSphere(2.0f);
		object.addChild(obstacle);
		
		route = new PathRoute(2.0f);
		route.setExcludeObstacles((obstacle,coin.obstacle));
		route.setMaxAngle(0.5f);
	}
	~Robot2D() {
		delete coin;
		removeFromEditor(object);
		delete obstacle;
		delete route;
	}
	
	int needCoin() {
		return (coin.isEnabled() == 0);
	}
	void createCoin(Vec3 position_) {
		coin.setPosition(position_);
		coin.setEnabled(1);
		coin.update();
		engine.world.updateSpatial();
		route.create2D(position + offset,coin.position + offset);
		if(route.isReached() == 0) removeCoin();
	}
	void removeCoin() {
		coin.setEnabled(0);
	}
	
	void update(int visualizer) {
		
		vec3 p = object.getWorldPosition();
		position.x = p.x;
		position.y = p.y;
		
		if(coin.isEnabled()) {
			
			coin.update();
			
			if(length(position - coin.position) < 2.0f || counter++ > 32) {
				if(counter > 32) engine.console.onscreenMessage("PathRoute failed\n");
				removeCoin();
			}
			else if(route.isReady()) {
				counter = 0;
				if(route.isReached()) {
					float ifps = engine.game.getIFps();
					Vec3 direction = route.getPoint(1) - route.getPoint(0);
					if(length(direction) > EPSILON) orientation = lerp(orientation,quat(setTo(Vec3_zero,direction,vec3(0.0f,0.0f,1.0f))),ifps * 8.0f);
					position += object.getWorldDirection() * ifps * 16.0f;
					object.setWorldTransform(translate(position) * orientation);
					if(visualizer) route.renderVisualizer(vec4_one);
					route.create2D(position + offset,coin.position + offset,1);
				} else {
					engine.console.onscreenMessage("PathRoute failed\n");
					removeCoin();
				}
			}
			else if(route.isQueued() == 0) {
				route.create2D(position + offset,coin.position + offset,1);
			}
		}
	}
};

/*
 */
ObjectMeshStatic mesh;
NavigationMesh navigation;
Robot2D robots[0];
Vec3 offset = Vec3(0.0f,0.0f,0.0f);
Vec3 min = Vec3(-256.0f,-256.0f,1.0f) + offset;
Vec3 max = Vec3( 256.0f, 256.0f,1.0f) + offset;

using Unigine::Samples;

/*
 */
float get_height(Vec3 position) {
	Vec3 point_0 = position - offset - vec3(0.0f,0.0f,100.0f);
	Vec3 point_1 = position - offset + vec3(0.0f,0.0f,100.0f);
	ObjectIntersection intersection = new ObjectIntersection();
	if(mesh.getIntersection(point_0,point_1,intersection,0)) {
		vec3 point = intersection.getPoint();
		return point.z + 1.0f;
	}
	return 1.0f;
}

/*
 */
int update() {
	
	if(engine.game.isEnabled()) {
		forloop(int i = 0; robots.size()) {
			Robot2D robot = robots[i];
			robot.update((i < 4));
			robot.position.z = get_height(robot.position);
			if(navigation.inside2D(robot.position,2.0f) == 0) {
				engine.console.onscreenMessage("Robot is outside the mesh\n");
				while(1) {
					Vec3 position = engine.game.getRandom(min,max);
					position.z = get_height(robot.position);
					if(navigation.inside2D(position,2.0f)) {
						robot.object.setWorldTransform(translate(position) * robot.orientation);
						break;
					}
				}
			}
			if(robot.needCoin()) {
				while(1) {
					Vec3 position = engine.game.getRandom(min,max);
					position.z = get_height(position);
					if(navigation.inside2D(position,2.0f)) {
						robot.createCoin(position);
						break;
					}
				}
			}
		}
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/paths/meshes/route_04.mesh")));
	mesh.setWorldTransform(translate(offset));
	mesh.setMaterial(findMaterialByName("paths_ground"),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	navigation = addToEditor(new NavigationMesh(fullPath("uniginescript_samples/paths/meshes/route_04.mesh")));
	navigation.setWorldTransform(translate(offset + Vec3(0.0f,0.0f,0.5f)));
	navigation.setHeight(10.0f);
	
	int num_robots = 8;
	forloop(int i = 0; num_robots) {
		Vec3 position = engine.game.getRandom(min,max);
		position.z = get_height(position);
		robots.append(new Robot2D(engine.game.getRandom(min,max)));
	}
	
	setDescription(format("%d PathRoute2D in NavigationMesh with %d obstacles",num_robots,num_robots * 2));

	if(engine.visualizer.isEnabled() != 1) {
		engine.visualizer.setEnabled(1);
	}

	return 1;
}

/*
*/
int shutdown() {

	engine.console.setOnscreen(0);
	engine.visualizer.setEnabled(0);
	return 1;
}

```

## scriptable_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
ObjectMeshStatic meshes[0];
Gui gui;


/*
 */
string phong_mesh_material_names[] = ( "shaders_phong_mesh_red", "shaders_phong_mesh_green", "shaders_phong_mesh_blue", "shaders_phong_mesh_orange", "shaders_phong_mesh_yellow" );

string text[] = ( "post_blur_2d", "post_water", "post_fire", "post_chromatic_aberration", "post_scriptable_blur_radial", "post_sobel", "post_vignette" );
string material_names[] = ( "post_blur_2d", "post_water", "post_fire", "post_chromatic_aberration", "post_scriptable_blur_radial", "post_sobel", "post_vignette_0" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

void toggle_clicked(int index)
{
	Material material = findMaterialByName(material_names[index]);
	
	if (material != NULL)
	{
		int id = engine.render.findScriptableMaterial(material);
		if (id != -1)
		{
			int value = !engine.render.getScriptableMaterialEnabled(id);
			engine.render.setScriptableMaterialEnabled(id, value);
		}
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	Player player = createDefaultPlayer(Vec3(12.0f,0.0f,4.0f));
	createDefaultPlane();
	
	setDescription("Scriptable materials");
	
	player.setDirection(vec3(-1.0f,0.0f,-0.25f));
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++)
	{
		for(int x = -size; x <= size; x++)
		{
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(scale(vec3(0.25f)) * translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			meshes.append(mesh);
		}
	}
	
	gui = engine.getGui();
	for (int i = 0; i < text.size(); i++)
	{
		WidgetCheckBox toggle = new WidgetCheckBox(gui, text[i]);
		toggle.setFontSize(16);
		toggle.setPosition(4, 4 + i * 24);
		gui.addChild(toggle, GUI_ALIGN_OVERLAP);
		toggle.getEventClicked().connect(functionid(toggle_clicked), i);
	}
	
	// init scriptable materials
	engine.render.clearScriptableMaterials();
	for (int i = 0; i < material_names.size(); i++)
	{
		engine.render.addScriptableMaterial(findMaterialByName(material_names[i]));
		engine.render.setScriptableMaterialEnabled(i, false);
	}
	
	engine.render.setSSAO(true);
	
	return 1;
}

/*
 */
int update() {
	
	
	
	return 1;
}

int shutdown() {
	engine.render.setSSAO(false);
	
	return 1;
}

```

## scroll_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Window {
	
	Gui gui;					// gui
	
	WidgetWindow window;		// window
	WidgetScrollBox scroll;		// scroll
	WidgetLabel labels[0];		// labels
	
	// constructor/destructor
	Window() {
		
		int restore = (labels.size() != 0);
		
		labels.clear();
		
		gui = engine.getGui();
		
		// window
		window = new WidgetWindow(gui,"Window");
		
		// scroll
		scroll = new WidgetScrollBox(gui);
		scroll.setWidth(512);
		scroll.setHeight(256);
		scroll.setHScrollEnabled(0);
		window.addChild(scroll,GUI_ALIGN_EXPAND);
		
		window.arrange();
		window.setSizeable(1);
		gui.addChild(window,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
		
		if(restore == 0) thread("Window::update_redirector",this);
	}
	~Window() {
		delete window;
		delete scroll;
		labels.delete();
	}
	
	// save/restore state
	void __restore__() {
		__Window__();
	}
	
	// update
	void update() {
		
		WidgetLabel label = new WidgetLabel(gui,format("Label %d",labels.size()));
		scroll.addChild(label);
		labels.append(label);
		
		scroll.arrange();
		scroll.setVScrollValue(scroll.getVScrollObjectSize() - scroll.getVScrollFrameSize());
	}
	
	void update_redirector(Window window) {
		while(1) {
			window.update();
			sleep(0.125f);
		}
	}
};

Window window;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	window = new Window();
	
	setDescription("ScrollBox");
	
	return 1;
}

```

## sector_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
PathRoute route;
Navigation navigations[0];
Vec3 offset = Vec3(0.0f,0.0f,0.0f);
Vec3 p0 = Vec3(-60.0f,-60.0f,5.0f) + offset;
Vec3 p1 = Vec3( 60.0f, 60.0f,5.0f) + offset;
Vec3 p2 = Vec3(-60.0f, 60.0f,5.0f) + offset;
Vec3 p3 = Vec3( 60.0f,-60.0f,5.0f) + offset;

using Unigine::Samples;

/*
 */
int update() {
	
	foreach(Navigation n; navigations) {
		n.renderVisualizer();
	}
	
	route.create2D(p0,p1);
	if(route.isReached()) route.renderVisualizer(vec4_one);
	else engine.console.onscreenMessage("PathRoute failed\n");
	
	route.create2D(p2,p3);
	if(route.isReached()) route.renderVisualizer(vec4_one);
	else engine.console.onscreenMessage("PathRoute failed\n");
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("PathRoute2D in single NavigationSector");
	
	NavigationSector navigation = addToEditor(new NavigationSector(vec3(128.0f,128.0f,8.0f)));
	navigation.setWorldTransform(translate(Vec3(0.0f,0.0f,5.0f) + offset));
	navigations.append(navigation);
	
	engine.world.updateSpatial();
	
	route = new PathRoute();
	route.setRadius(2.0f);
	
	if(engine.visualizer.isEnabled() != 1) {
		engine.visualizer.setEnabled(1);
	}

	return 1;
}

/*
*/
int shutdown() {

	engine.console.setOnscreen(0);
	engine.visualizer.setEnabled(0);
	return 1;
}

```

## sector_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
PathRoute route;
Navigation navigations[0];
Vec3 offset = Vec3(0.0f,0.0f,0.0f);
Vec3 p0 = Vec3(-60.0f,-60.0f,5.0f) + offset;
Vec3 p1 = Vec3( 60.0f, 60.0f,5.0f) + offset;
Vec3 p2 = Vec3(-60.0f, 60.0f,5.0f) + offset;
Vec3 p3 = Vec3( 60.0f,-60.0f,5.0f) + offset;

using Unigine::Samples;

/*
 */
int update() {
	
	foreach(Navigation n; navigations) {
		n.renderVisualizer();
	}
	
	route.create2D(p0,p1);
	if(route.isReached()) route.renderVisualizer(vec4_one);
	else engine.console.onscreenMessage("PathRoute failed\n");
	
	route.create2D(p2,p3);
	if(route.isReached()) route.renderVisualizer(vec4_one);
	else engine.console.onscreenMessage("PathRoute failed\n");
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("PathRoute2D in multiple NavigationSector");
	
	int size = 2;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			NavigationSector navigation = addToEditor(new NavigationSector(vec3(16.0f,16.0f,8.0f)));
			navigation.setWorldTransform(translate(Vec3(x,y,5.0f) * Vec3(30.0f,30.0f,1.0f)));
			navigations.append(navigation);
		}
	}
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x < size; x++) {
			NavigationSector navigation = addToEditor(new NavigationSector(vec3(19.0f,8.0f,6.0f)));
			navigation.setWorldTransform(translate(Vec3(x + 0.5f,y,4.0f) * Vec3(30.0f,30.0f,1.0f)));
			navigations.append(navigation);
		}
	}
	
	for(int y = -size; y < size; y++) {
		for(int x = -size; x <= size; x++) {
			NavigationSector navigation = addToEditor(new NavigationSector(vec3(8.0f,19.0f,6.0f)));
			navigation.setWorldTransform(translate(Vec3(x,y + 0.5f,4.0f) * Vec3(30.0f,30.0f,1.0f)));
			navigations.append(navigation);
		}
	}
	
	engine.world.updateSpatial();
	
	route = new PathRoute();
	route.setRadius(1.0f);
	
	if(engine.visualizer.isEnabled() != 1) {
		engine.visualizer.setEnabled(1);
	}
	return 1;
}

/*
*/
int shutdown() {

	engine.console.setOnscreen(0);
	engine.visualizer.setEnabled(0);
	return 1;
}

```

## sector_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
PathRoute route;
Navigation navigations[0];
Vec3 offset = Vec3(0.0f,0.0f,0.0f);
Vec3 p0 = Vec3(-60.0f,-60.0f,5.0f) + offset;
Vec3 p1 = Vec3( 60.0f, 60.0f,5.0f) + offset;
Vec3 p2 = Vec3(-60.0f, 60.0f,5.0f) + offset;
Vec3 p3 = Vec3( 60.0f,-60.0f,5.0f) + offset;

using Unigine::Samples;

/*
 */
int update() {
	
	foreach(Navigation n; navigations) {
		n.renderVisualizer();
	}
	
	route.create3D(p0,p1);
	if(route.isReached()) route.renderVisualizer(vec4_one);
	else engine.console.onscreenMessage("PathRoute failed\n");
	
	route.create3D(p2,p3);
	if(route.isReached()) route.renderVisualizer(vec4_one);
	else engine.console.onscreenMessage("PathRoute failed\n");
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("PathRoute3D in multiple NavigationSector");
	
	int size = 2;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			NavigationSector navigation = addToEditor(new NavigationSector(vec3(16.0f,16.0f,8.0f)));
			navigation.setWorldTransform(translate(Vec3(x,y,5.0f) * Vec3(30.0f,30.0f,1.0f)));
			navigations.append(navigation);
		}
	}
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x < size; x++) {
			NavigationSector navigation = addToEditor(new NavigationSector(vec3(19.0f,8.0f,6.0f)));
			navigation.setWorldTransform(translate(Vec3(x + 0.5f,y,4.0f) * Vec3(30.0f,30.0f,1.0f)));
			navigations.append(navigation);
		}
	}
	
	for(int y = -size; y < size; y++) {
		for(int x = -size; x <= size; x++) {
			NavigationSector navigation = addToEditor(new NavigationSector(vec3(8.0f,19.0f,6.0f)));
			navigation.setWorldTransform(translate(Vec3(x,y + 0.5f,4.0f) * Vec3(30.0f,30.0f,1.0f)));
			navigations.append(navigation);
		}
	}
	
	engine.world.updateSpatial();
	
	route = new PathRoute();
	route.setRadius(1.0f);
	
	if(engine.visualizer.isEnabled() != 1) {
		engine.visualizer.setEnabled(1);
	}

	return 1;
}

/*
*/
int shutdown() {

	engine.console.setOnscreen(0);
	engine.visualizer.setEnabled(0);
	return 1;
}

```

## selection_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

Object object = NULL;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	int num = 0;
	int size = 5;
	engine.visualizer.setEnabled(1);
	light_world_0.setScattering(LIGHT_WORLD_SCATTERING_SUN);
	light_world_1.setScattering(LIGHT_WORLD_SCATTERING_SUN);
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/objects/meshes/mesh_00.mesh")));
			mesh.setMaterial(findMaterialByName("mesh_base"),"*");
			mesh.getMaterialInherit(0);
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setWorldTransform(translate(Vec3(x,y,2.0f) * 2.0f));
			mesh.setIntersection(1, 0);
			mesh.setData("coords", format("x: %d y: %d",x,y));
			num++;
		}
	}
	
	setDescription("Select object by right button");
	
	return 1;
}

/*
 */
int update() {
	
	Vec3 p0,p1;
	Unigine::getPlayerMouseDirection(p0,p1);
	
	WorldIntersectionNormal intersection = new WorldIntersectionNormal();
	Object o = engine.world.getIntersection(p0,p1,~0,intersection);
	if(o != NULL) engine.visualizer.renderDirection(intersection.getPoint(),intersection.getNormal(),vec4_one);
	
	if(engine.controls.isMouseEnabled() == 0 && o != NULL) {
		if(object != o) log.message("%s\n",o.getData("coords"));
		if(engine.gui.isActive() == 0 &&  engine.input.isMouseButtonDown(INPUT_MOUSE_BUTTON_RIGHT) == 1) {
			if(object != NULL) object.setMaterialParameterFloat4("albedo_color",vec4_one,0);
			o.setMaterialParameterFloat4("albedo_color",vec4(0.0f,1.0f,0.0f,1.0f),0);
			object = o;
		} else if(object != o) {
			if(object != NULL) object.setMaterialParameterFloat4("albedo_color",vec4_one,0);
			o.setMaterialParameterFloat4("albedo_color",vec4(1.0f,0.0f,0.0f,1.0f),0);
			object = o;
		}
	}
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## shapes_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
void create_body(float density,int cork,Mat4 transform) {
	
	ObjectMeshDynamic mesh = addToEditor(new ObjectMeshDynamic());
	mesh.setMaterial(findMaterialByName(get_material(0)),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	Body body = class_remove(new BodyRigid(mesh));
	
	void create_box(vec3 size,Mat4 transform) {
		Unigine::createBox(mesh,size,transform);
		ShapeBox shape = class_remove(new ShapeBox(size));
		body.addShape(shape,transform);
		shape.setMass(shape.getVolume() * density);
		shape.setFriction(0.5f);
		shape.setRestitution(0.5f);
	}
	
	if(cork) create_box(vec3(5.0f,5.0f,0.5f),translate(0.0f,0.0f,0.25f));
	create_box(vec3(15.0f,8.0f,0.5f),translate( 0.0f,-5.0f,3.5f) * rotateX(-52.0f));
	create_box(vec3(15.0f,8.0f,0.5f),translate( 0.0f, 5.0f,3.5f) * rotateX( 52.0f));
	create_box(vec3(8.0f,15.0f,0.5f),translate(-5.0f, 0.0f,3.5f) * rotateY( 52.0f));
	create_box(vec3(8.0f,15.0f,0.5f),translate( 5.0f, 0.0f,3.5f) * rotateY(-52.0f));
	
	mesh.setWorldTransform(transform);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(40.0f,0.0f,30.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 50;
	
	Vec3 min = Vec3(-6.0f,-6.0f,25.0f);
	Vec3 max = Vec3( 6.0f, 6.0f,65.0f);
	
	forloop(int i = 0; num) {
		createBodySphere(0.75f,1.0f,0.5f,0.5f,get_material(i),translate(engine.game.getRandom(min,max)));
	}
	
	forloop(int i = 0; num) {
		createBodyCapsule(0.75f,0.75f,1.0f,0.5f,0.5f,get_material(i),translate(engine.game.getRandom(min,max)));
	}
	
	forloop(int i = 0; num) {
		createBodyBox(vec3(0.75f,0.75f,1.0f),1.0f,0.5f,0.5f,get_material(i),translate(engine.game.getRandom(min,max)));
	}
	
	forloop(int i = 0; num) {
		createBodyPrism(0.75f,1.0f,0.75f,3,1.0f,0.5f,0.5f,get_material(i),translate(engine.game.getRandom(min,max)));
	}
	
	forloop(int i = 0; num) {
		createBodyIcosahedron(0.75f,1.0f,0.5f,0.5f,get_material(i),translate(engine.game.getRandom(min,max)));
	}
	
	forloop(int i = 0; num) {
		createBodyDodecahedron(0.75f,1.0f,0.5f,0.5f,get_material(i),translate(engine.game.getRandom(min,max)));
	}
	
	forloop(int i = 0; num) {
		createBodyCylinder(0.75f,1.0f,1.0f,0.5f,0.5f,get_material(i),translate(engine.game.getRandom(min,max)));
	}
	
	create_body(0.0f,0,translate(Vec3(0.0f,0.0f,14.0f)));
	create_body(1.0f,1,translate(Vec3(0.0f,0.0f,0.0f)));
	
	setDescription(format("%d Different Shapes",num * 6));
	
	return 1;
}

```

## skinned_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
ObjectMeshSkinned meshes[0];

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int size = 10;
	int num = 0;
	
	for(int z = 0; z < size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(fullPath("uniginescript_samples/common/meshes/capsule.mesh")));
				mesh.setAnimPath(fullPath("uniginescript_samples/common/meshes/capsule.anim"));
				mesh.setWorldTransform(translate(Vec3(x,y,z + 1.0f) * 2.0f));
				mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y ^ z)),"*");
				mesh.setSurfaceProperty("surface_base","*");
				meshes.append(mesh);
				num++;
			}
		}
	}
	
	setDescription(format("%d Object skinned meshes",num));
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime() * 2.0f;
	float scale = (meshes[0].getLayerNumFrames(0) - 1.0f) / meshes.size();
	
	if(engine.game.isEnabled()) {
		foreach(ObjectMeshSkinned mesh, i = 0; meshes; i++) {
			mesh.setLayerFrame(0,time + i * scale,1);
		}
	}
	
	return 1;
}

```

## skinned_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int size = 24;
	int num = 0;
	
	for(int y = -size, i = 0; y <= size; y++) {
		for(int x = -size; x <= size; x++, i++) {
			ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(fullPath("uniginescript_samples/stress/meshes/skinned_01.mesh")));
			mesh.setAnimPath(fullPath("uniginescript_samples/stress/meshes/skinned_01.anim"));
			mesh.setWorldTransform(Mat4(translate(x,y,0.0f) * rotateZ(90.0f)));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setLoop(1);
			mesh.setTime(i * mesh.getLayerNumFrames(0) * 0.25f);
			mesh.setSpeed(25.0f);
			mesh.play();
			num++;
		}
	}
	
	setDescription(format("%d Object skinned meshes",num));
	
	return 1;
}

```

## skinned_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectMeshSkinned meshes[0];
Blob blob;

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "objects_mesh_red", "objects_mesh_green", "objects_mesh_blue", "objects_mesh_orange", "objects_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int update() {
	
	float time = engine.game.getTime() * 2.0f;
	
	if(blob == NULL) blob = new Blob();
	
	// source mesh
	blob.seekSet(0);
	meshes[0].setLayerFrame(0,time,1);
	meshes[0].saveState(blob);
	
	forloop(int i = 1; meshes.size()) {
		ObjectMeshSkinned mesh = meshes[i];
		
		// save mesh parameters
		Mat4 transform = mesh.getWorldTransform();
		string material = mesh.getMaterial(0).getFilePath();
		
		// destination mesh
		blob.seekSet(0);
		mesh.restoreState(blob);
		
		// restore mesh parameters
		mesh.setWorldTransform(transform);
		mesh.setOldWorldTransform(transform);
		mesh.setMaterial(engine.materials.findMaterialByPath(material),0);
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	engine.console.setInt("render_motion_blur",0);
	
	int size = 8;
	
	int num = 0;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(fullPath("uniginescript_samples/common/meshes/capsule.mesh")));
			mesh.setAnimPath(fullPath("uniginescript_samples/common/meshes/capsule.anim"));
			mesh.setWorldTransform(translate(Vec3(x,y,1.0f) * 2.0f));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			meshes.append(mesh);
			num++;
		}
	}
	
	setDescription(format("%d ObjectMeshSkinned save/restore state",num));
	return 1;
}

```

## skinned_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectMeshStatic meshes[4];
ObjectMeshSkinned mesh;
vec3 offset[4] = ( vec3(2.0f,0.0f,2.0f), vec3(-2.0f,0.0f,2.0f), vec3(0.0f,2.0f,2.0f), vec3(0.0f,-2.0f,2.0f) );

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "objects_mesh_red", "objects_mesh_green", "objects_mesh_blue", "objects_mesh_orange", "objects_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	mesh.setLayerFrame(0,time * 10.0f);
	mesh.setWorldTransform(Mat4(rotateZ(time * 32.0f) * translate(0.0f,0.0f,1.0f) * scale(vec3(sin(time) * 0.25f + 1.0f))));
	
	int bone = mesh.findBone("bone_04");
	mat4 transform = mesh.getBoneWorldTransform(bone) * rotateY(90.0f);
	
	foreach(ObjectMeshStatic mesh, i = 0; meshes; i++) {
		mesh.setWorldTransform(Mat4(transform * translate(offset[i]) * scale(vec3(2.0f))));
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("Objects binded to bones");
	
	foreach(ObjectMeshStatic mesh, i = 1; meshes; i++) {
		mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/cbox.mesh")));
		mesh.setSurfaceProperty("surface_base","*");
		mesh.setMaterial(findMaterialByName(get_mesh_material(i)),"*");
	}
	
	mesh = addToEditor(new ObjectMeshSkinned(fullPath("uniginescript_samples/objects/meshes/skinned_01.mesh")));
	mesh.setAnimPath(fullPath("uniginescript_samples/objects/meshes/skinned_01.anim"));
	mesh.setMaterial(findMaterialByName(get_mesh_material(0)),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	return 1;
}

```

## skinned_04.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectMeshSkinned meshes[0];

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "objects_mesh_red", "objects_mesh_green", "objects_mesh_blue", "objects_mesh_orange", "objects_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int update() {
	
	float time = engine.game.getTime() * 2.0f;
	
	foreach(ObjectMeshSkinned mesh, i = 0; meshes; i++) {
		mesh.setLayerFrame(0,time + i * 0.05f,1,7);
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("Bone scaling");
	
	int size = 4;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(fullPath("uniginescript_samples/objects/meshes/skinned_02.mesh")));
			mesh.setAnimPath(fullPath("uniginescript_samples/objects/meshes/skinned_02.anim"));
			mesh.setWorldTransform(Mat4(translate(vec3(x,y,0.0f) * 4.0f) * scale(0.5f,0.5f,0.5f)));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			meshes.append(mesh);
		}
	}
	
	return 1;
}

```

## skinned_05.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectMeshSkinned meshes[0];

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "objects_mesh_red", "objects_mesh_green", "objects_mesh_blue", "objects_mesh_orange", "objects_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	foreach(ObjectMeshSkinned mesh, i = 0; meshes; i++) {
		mesh.setLayerFrame(0,time + i * 0.05f,1);
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("Dual quaternions");
	
	int size = 4;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(fullPath("uniginescript_samples/objects/meshes/skinned_03.mesh")));
			mesh.setAnimPath(fullPath("uniginescript_samples/objects/meshes/skinned_03.anim"));
			mesh.setWorldTransform(Mat4(translate(vec3(x,y,1.0f) * 9.0f) * scale(0.4f,0.4f,0.4f) * rotateZ(45.0f)));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setQuaternion(1);
			meshes.append(mesh);
		}
	}
	
	return 1;
}

```

## skinned_06.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectMeshSkinned meshes[0];

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "objects_mesh_red", "objects_mesh_green", "objects_mesh_blue", "objects_mesh_orange", "objects_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int update() {
	
	float time = engine.game.getTime() * 2.0f;
	
	foreach(ObjectMeshSkinned mesh, i = 0; meshes; i++) {
		mesh.setLayerFrame(0,time * 4.0f + i * 0.2f,1);
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	vec3 noise;
	
	int num = 0;
	int size = 5;
	
	Mesh reference = new Mesh(fullPath("uniginescript_samples/objects/meshes/skinned_06.mesh"));
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			
			mat4 transform = translate(vec3(x * 6.0f,y * 6.0f,2.0f)) * rotateZ(y * size * 2.0f);
			
			// dynamic mesh
			Mesh dynamic = new Mesh(reference);
			dynamic.createNormals();
			
			// deform vertices
			forloop(int i = 0; dynamic.getNumSurfaces()) {
				forloop(int j = 0; dynamic.getNumVertex(i)) {
					vec3 xyz = dynamic.getVertex(j,i);
					vec3 normal = dynamic.getNormal(j,i);
					vec3 offset = transform.col33 + xyz;
					noise.x = engine.game.getNoise(offset + vec3( 3.0f),vec3(8.0f),1.0f);
					noise.y = engine.game.getNoise(offset + vec3(13.0f),vec3(8.0f),1.0f);
					noise.z = engine.game.getNoise(offset + vec3(73.0f),vec3(8.0f),1.0f);
					dynamic.setNormal(j,normalize(normal + noise * 2.0f),i);
					dynamic.setVertex(j,xyz + noise * 1.2f,i);
				}
			}
			
			// update tangent space
			dynamic.createTangents();
			
			// create mesh
			ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned());
			mesh.setMeshProceduralMode(true);
			mesh.applyMeshProcedural(dynamic);
			mesh.setAnimPath(fullPath("uniginescript_samples/objects/meshes/skinned_06.anim"));
			
			delete dynamic;
			
			// mesh parameters
			mesh.setWorldTransform(Mat4(transform));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			meshes.append(mesh);
			
			num++;
		}
	}
	
	delete reference;
	
	setDescription(format("%d Dynamic ObjectMeshSkinned",num));
	return 1;
}

```

## skinned_07.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
ObjectMeshSkinned meshes[0];

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "objects_mesh_red", "objects_mesh_green", "objects_mesh_blue", "objects_mesh_orange", "objects_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int update() {
	
	float time = engine.game.getTime() * 2.0f;
	
	foreach(ObjectMeshSkinned mesh, i = 0; meshes; i++) {
		float k0 = sin(time + i * 3.0f) + 0.75f;
		float k1 = cos(time + i * 3.0f) + 0.75f;
		mesh.setSurfaceTargetWeight(0,0,1.0f - k0 -k1);
		mesh.setSurfaceTargetWeight(0,1,k0);
		mesh.setSurfaceTargetWeight(0,2,k1);
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	int size = 5;
	
	if(0) {
		
		// create mesh
		Mesh mesh = new Mesh();
		
		// sphere surface
		int surface = mesh.addSphereSurface("surface",2.0f,32,64);
		
		// create targets
		int target_0 = mesh.addSurfaceTarget(0,"box");
		int target_1 = mesh.addSurfaceTarget(0,"star");
		forloop(int i = 0; mesh.getNumVertex(surface)) {
			vec3 vertex = normalize(mesh.getVertex(i,surface));
			vec3 scale = vec3_one - vertex.yzx * vertex.yzx * 0.5f - vertex.zxy * vertex.zxy * 0.5f;
			mesh.setVertex(i,vertex * scale * 2.5f,surface,target_0);
			mesh.setVertex(i,vertex / scale * 2.0f,surface,target_1);
		}
		
		// update tangent space
		mesh.createTangents(surface,target_0);
		mesh.createTangents(surface,target_1);
		
		mesh.save(fullPath("uniginescript_samples/objects/meshes/skinned_07.mesh"));
		
		delete mesh;
	}
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			
			mat4 transform = translate(vec3(x * 6.0f,y * 6.0f,3.0f)) * rotateZ(num * 32.0f);
			
			// create mesh
			ObjectMeshSkinned mesh = addToEditor(new ObjectMeshSkinned(fullPath("uniginescript_samples/objects/meshes/skinned_07.mesh")));
			
			// mesh parameters
			mesh.setWorldTransform(Mat4(transform));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			meshes.append(mesh);
			
			// mesh targets
			mesh.setSurfaceTargetEnabled(0,0,1);
			mesh.setSurfaceTargetEnabled(0,1,1);
			mesh.setSurfaceTargetEnabled(0,2,1);
			
			num++;
		}
	}
	
	setDescription(format("%d ObjectMeshSkinned Morphing",num));
	return 1;
}

```

## sky_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	PlayerSpectator player = createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("ObjectSky");
	
	player.setZNear(0.1f);
	player.setZFar(20000.0f);
	player.setDirection(vec3(1.0f,0.0f,0.2f));
	
	ObjectSky sky = addToEditor(new ObjectSky());
	sky.setWorldTransform(translate(Vec3(0.0f,0.0f,2000.0f)));
	
	return 1;
}

```

## sky_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	PlayerSpectator player = createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("ObjectSky cubemap");
	
	player.setZNear(0.1f);
	player.setZFar(20000.0f);
	player.setDirection(vec3(1.0f,1.0f,0.5f));
	
	ObjectSky sky = addToEditor(new ObjectSky());
	sky.setSpherical(1);
	sky.setMaterial(findMaterialByName("objects_sky_01"),"sphere");
	
	return 1;
}

```

## sobel_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("Sobel filter");
	
	engine.console.setInt("render_taa",1);
	engine.render.addScriptableMaterial(findMaterialByName("render_filter_sobel"));
	
	int size = 3;
	float space = 16.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	return 1;
}

```

## socket_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string host = "unigine.com";
int port = 80;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

Info info;
Socket socket;
float time = 0.0f;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	info = new Info();
	
	setDescription(format("Socket to http://%s:%d",host,port));
	
	return 1;
}

/*
 */
int update() {
	
	time -= engine.game.getIFps();
	
	if(time < 0.0f) {
		
		if(socket == NULL) socket = new Socket(SOCKET_SOCKET_TYPE_STREAM);
		
		if(socket.open(host,port)) {
			if(socket.connect()) {
				socket.printf("GET / HTTP/1.0\r\nUser-Agent: Unigine\r\n\r\n");
				info.set(socket.gets());
			} else {
				info.set(format("Can't connect to %s:%d",host,port));
			}
			socket.close();
		}
		
		time = 1.0f;
	}
	
	return 1;
}

```

## socket_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string host = "unigine.com";
int port = 80;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

Info info;
Async async;
Socket socket;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	info = new Info();
	thread("update_thread");
	
	setDescription(format("Async Socket to http://%s:%d",host,port));
	
	return 1;
}

/*
 */
void update_thread() {
	
	while(1) {
		if(async == NULL) {
			async = new Async();
			socket = new Socket(SOCKET_SOCKET_TYPE_STREAM);
		}
		
		if(socket.open(host,port)) {
			
			int id = async.run(socket,functionid(Socket::connect));
			while(async != NULL && async.isRunning(id)) wait;
			if(async == NULL) continue;
			
			if(async.getResult(id)) {
				
				socket.printf("GET / HTTP/1.0\r\nUser-Agent: Unigine\r\n\r\n");
				
				int id = async.run(socket,functionid(Socket::gets));
				while(async != NULL && async.isRunning(id)) wait;
				if(async == NULL) continue;
				
				info.set(async.getResult(id));
			}
			else {
				info.set(format("Can't connect to %s:%d",host,port));
			}
			
			socket.close();
		}
	}
}

/*
 */
int shutdown() {
	
	if(async != NULL) async.wait();
	if(socket != NULL && socket.isOpened()) socket.close();
	
	return 1;
}

```

## socket_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string host = "unigine.com";
int port = 80;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

Info info;
Async async;

/*
 */
string async_socket(string host,int port) {
	
	string ret = "";
	
	Socket socket = new Socket(SOCKET_SOCKET_TYPE_STREAM);
	
	if(socket.open(host,port)) {
		
		if(socket.connect()) {
			socket.printf("GET / HTTP/1.0\r\nUser-Agent: Unigine\r\n\r\n");
			ret = socket.gets();
		}
		else {
			ret = format("Can't connect to %s:%d",host,port);
		}
		
		socket.close();
	}
	
	delete socket;
	
	return ret;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	info = new Info();
	thread("update_thread");
	
	setDescription(format("Async Socket to http://%s:%d",host,port));
	
	return 1;
}

/*
 */
void update_thread() {
	
	while(1) {
		if(async == NULL) async = new Async();
		
		int id = async.run(functionid(async_socket),host,port);
		while(async != NULL && async.isRunning(id)) wait;
		if(async == NULL) continue;
		
		info.set(async.getResult(id));
	}
}

/*
 */
int shutdown() {
	
	if(async != NULL) async.wait();
	
	return 1;
}

```

## socket_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string host = "unigine.com";
int port = 80;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

Info info;
Async async;
Socket socket;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	info = new Info();
	thread("update_thread");
	
	setDescription(format("Async Socket to http://%s:%d",host,port));
	
	return 1;
}

/*
 */
void update_thread() {
	
	while(1) {
		if(async == NULL) {
			async = new Async();
			socket = new Socket(SOCKET_SOCKET_TYPE_STREAM);
		}
		
		int id = async.run([]() {
			if(socket.open(host,port)) {
				if(socket.connect()) {
					socket.printf("GET / HTTP/1.0\r\nUser-Agent: Unigine\r\n\r\n");
					info.set(socket.gets());
				} else {
					info.set(format("Can't connect to %s:%d",host,port));
				}
				socket.close();
			}
		});
		
		while(async != NULL && async.isRunning(id)) wait;
		if(async != NULL) async.clearResult();
	}
}

/*
 */
int shutdown() {
	
	if(async != NULL) async.wait();
	
	return 1;
}

```

## socket_04.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

int app_update;
ObjectMeshStatic mesh;

/*
 */
class ServerSocket {
	
	Socket socket;
	
	void create() {
		socket = new Socket(SOCKET_SOCKET_TYPE_STREAM,8080);
		socket.bind();
		socket.listen(10);
		socket.nonblock();
	}
	
	ServerSocket() {
		create();
	}
	~ServerSocket() {
		socket.close();
		delete socket;
	}
	
	void __restore__() {
		create();
	}
	
	int accept(Socket s) {
		return socket.accept(s);
	}

};

Async async;
ServerSocket socket;
Socket client_socket;

/*
 */
string generate_page_content() {
	
	// http page
	string page_content = "<html>\r\n"
		"\t<head>\r\n"
		"\t\t<title>UNIGINE Web Server</title>\r\n"
		"\t\t<script>\r\n"
		"\t\t\tfunction reload() { setTimeout(\"location.reload(true);\",100); }\r\n"
		"\t\t\twindow.onload=reload;\r\n"
		"\t\t</script>\r\n"
		"\t</head>\r\n"
		"\t<body>\r\n"
		"\t\t<h2>Hello from UNIGINE</h2>\r\n"
		"\t\tBinary: " + engine.console.getString("binary_info") + "<br>\r\n"
		"\t\tCPU: " + engine.console.getString("cpu_info") + "<br>\r\n"
		"\t\tTime: " + string(engine.game.getTime()) + "<br>\r\n"
		"\t\tPosition: " + string(mesh.getTransform() * vec3_zero) + "<br><br>\r\n"
		"\t</body>\r\n"
		"</html>\r\n";

	return page_content;
}

/*
 */
void update_client(Socket socket, string page_content) {
	string get = socket.readLine();
	get = get;

	int buf[1];
	int old = '\n';
	while(1) {
		if(socket.read(buf,1) != 1) break;
		if(buf[0] == '\r') continue;
		if(buf[0] == '\n' && old == '\n') break;
		old = buf[0];
	}

	// answer
	socket.puts("HTTP/1.0 200 Ok\r\n"
		"Server: Unigine\r\n"
		"Content-Type: text/html; charset=ISO-8859-1\r\n"
		"Content-Length: " + strlen(page_content) + "\r\n\r\n");
	
	socket.puts(page_content);
	
	socket.close();
	delete socket;
}

/*
 */
void update_socket() {
	
	client_socket = NULL;
	
	while(1) {
		
		if(socket == NULL) {
			socket = new ServerSocket();
		}

		if(async == NULL) {
			async = new Async();
		}
		
		if(client_socket == NULL) {
			client_socket = new Socket(SOCKET_SOCKET_TYPE_STREAM);
		}
		
		if(socket.accept(client_socket))
		{
			int id = async.run("update_client", client_socket, generate_page_content());

			while(async != NULL && async.isRunning(id)) wait;
			if(async != NULL) async.clearResult();
			client_socket = NULL;
		}
		
		wait;
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	light_world_0.setScattering(LIGHT_WORLD_SCATTERING_SUN);
	light_world_1.setScattering(LIGHT_WORLD_SCATTERING_SUN);
	engine.setBackgroundUpdate(1);
	
	mesh = new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh"));
	mesh.setMaterial(findMaterialByName("mesh_base"),"*");
	
	thread("update_socket");
	
	setDescription("Web server on http://localhost:8080");
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	mesh.setWorldTransform(Mat4(rotateX(time * 32.0f) * translate(cos(time * 1.0f) * 8.0f,sin(time * 1.5f) * 16.0f,8.0f) * rotateY(time * 64.0f) * rotateZ(time * 48.0f) * scale(8.0f,6.0f,4.0f)));
	
	return 1;
}

/*
 */
int shutdown() {

	// Forced connection reset
	if(client_socket != NULL && client_socket.isOpened())
		client_socket.close();

	if(async != NULL) 
		async.wait();

	engine.setBackgroundUpdate(app_update);
	
	return 1;
}

```

## sound_reverb_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Info info;
SoundReverb reverb;
SoundSource source;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	info = new Info("'x' - play\n'c' - stop\n");
	
	reverb = addToEditor(new SoundReverb(vec3(100.0f,100.0f,100.0f)));
	reverb.setThreshold(vec3(10.0f,10.0f,10.0f));
	reverb.setDensity(0.2f);
	reverb.setDiffusion(0.5f);
	reverb.setDecayTime(8.0f);
	reverb.setReflectionGain(2.0f);
	reverb.setLateReverbGain(8.0f);
	
	source = addToEditor(new SoundSource(fullPath("uniginescript_samples/sounds/sounds/static_mono_00.oga")));
	source.setWorldTransform(translate(Vec3(0.0f,0.0f,8.0f)));
	source.setLoop(1);
	source.setMinDistance(50.0f);
	source.setMaxDistance(100.0f);
	source.play();
	
	setDescription("Single environment SoundReverb");
	
	return 1;
}

int update() {
	
	if(engine.input.isKeyDown(INPUT_KEY_X)) {
		source.play();
		log.message("play\n");
	}
	if(engine.input.isKeyDown(INPUT_KEY_C)) {
		source.stop();
		log.message("stop\n");
	}
	
	reverb.renderVisualizer();
	source.renderVisualizer();
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## sound_reverb_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Info info;

SoundReverb reverb_0;
SoundReverb reverb_1;
SoundSource source;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	info = new Info("'x' - play\n'c' - stop\n");
	
	reverb_0 = addToEditor(new SoundReverb(vec3(100.0f,100.0f,100.0f)));
	reverb_0.setThreshold(vec3(10.0f,10.0f,10.0f));
	reverb_0.setDensity(0.2f);
	reverb_0.setDiffusion(0.5f);
	reverb_0.setDecayTime(8.0f);
	reverb_0.setReflectionGain(2.0f);
	reverb_0.setLateReverbGain(8.0f);
	
	reverb_1 = addToEditor(new SoundReverb(vec3(20.0f,20.0f,20.0f)));
	reverb_1.setThreshold(vec3(1.0f,1.0f,1.0f));
	reverb_1.setDensity(0.5f);
	reverb_1.setDiffusion(0.2f);
	reverb_1.setDecayTime(4.0f);
	reverb_1.setReflectionGain(2.0f);
	reverb_1.setReflectionDelay(0.2f);
	reverb_1.setLateReverbGain(8.0f);
	reverb_1.setLateReverbDelay(0.05f);
	
	source = addToEditor(new SoundSource(fullPath("uniginescript_samples/sounds/sounds/static_mono_00.oga")));
	source.setWorldTransform(translate(Vec3(0.0f,0.0f,8.0f)));
	source.setLoop(1);
	source.setMinDistance(50.0f);
	source.setMaxDistance(100.0f);
	source.play();
	
	setDescription("Multiple environment SoundReverb");
	
	return 1;
}

int update() {
	
	if(engine.input.isKeyDown(INPUT_KEY_X)) {
		source.play();
		log.message("play\n");
	}
	if(engine.input.isKeyDown(INPUT_KEY_C)) {
		source.stop();
		log.message("stop\n");
	}
	
	reverb_0.renderVisualizer();
	reverb_1.renderVisualizer();
	source.renderVisualizer();
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## sound_static_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Info info;
SoundSource source;
float pitch = 1.0f;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	info = new Info("'z' - loop toggle\n'x' - play\n'c' - stop\n'n' - increase pitch\n'b' - decrease pitch");
	
	source = addToEditor(new SoundSource(fullPath("uniginescript_samples/sounds/sounds/static_mono_00.oga")));
	source.setWorldTransform(translate(Vec3(0.0f,0.0f,8.0f)));
	source.setLoop(1);
	source.setMinDistance(50.0f);
	source.setMaxDistance(100.0f);
	source.play();
	
	setDescription("Static SoundSource");
	
	return 1;
}

int update() {
	
	if(engine.input.isKeyDown(INPUT_KEY_Z)) {
		source.setLoop(!source.getLoop());
		log.message("loop %d\n",source.getLoop());
	}
	if(engine.input.isKeyDown(INPUT_KEY_X)) {
		source.play();
		log.message("play\n");
	}
	if(engine.input.isKeyDown(INPUT_KEY_C)) {
		source.stop();
		log.message("stop\n");
	}
	if(engine.input.isKeyPressed(INPUT_KEY_N)) {
		pitch = clamp(pitch + engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	if(engine.input.isKeyPressed(INPUT_KEY_B)) {
		pitch = clamp(pitch - engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	
	if(source.isPlaying()) {
		engine.console.onscreenMessage("time: %f\n",source.getTime());
	}
	
	source.renderVisualizer();
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## sound_static_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Info info;
SoundSource source;
float pitch = 1.0f;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	info = new Info("'z' - loop toggle\n'x' - play\n'c' - stop\n'n' - increase pitch\n'b' - decrease pitch");
	
	source = addToEditor(new SoundSource(fullPath("uniginescript_samples/sounds/sounds/static_mono_00.oga")));
	source.setWorldTransform(translate(Vec3(0.0f,0.0f,16.0f)) * rotateY(90.0f));
	source.setLoop(1);
	source.setMinDistance(50.0f);
	source.setMaxDistance(100.0f);
	source.setConeInnerAngle(75.0f);
	source.setConeOuterAngle(90.0f);
	source.play();
	
	setDescription("Static Cone SoundSource");
	
	return 1;
}

int update() {
	
	if(engine.input.isKeyDown(INPUT_KEY_Z)) {
		source.setLoop(!source.getLoop());
		log.message("loop %d\n",source.getLoop());
	}
	if(engine.input.isKeyDown(INPUT_KEY_X)) {
		source.play();
		log.message("play\n");
	}
	if(engine.input.isKeyDown(INPUT_KEY_C)) {
		source.stop();
		log.message("stop\n");
	}
	if(engine.input.isKeyPressed(INPUT_KEY_N)) {
		pitch = clamp(pitch + engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	if(engine.input.isKeyPressed(INPUT_KEY_B)) {
		pitch = clamp(pitch - engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	
	if(source.isPlaying()) {
		engine.console.onscreenMessage("time: %f\n",source.getTime());
	}
	
	float time = engine.game.getTime();
	
	source.setWorldTransform(translate(Vec3(0.0f,0.0f,16.0f)) * rotateZ(time * 64.0f) * rotateY(90.0f));
	source.renderVisualizer();
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## sound_static_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 16;
	
	string names[] = (
		"uniginescript_samples/sounds/sounds/static_mono_00.oga",
		"uniginescript_samples/sounds/sounds/static_mono_01.oga",
		"uniginescript_samples/sounds/sounds/static_mono_02.oga",
		"uniginescript_samples/sounds/sounds/static_wave_01.wav",
		"uniginescript_samples/sounds/sounds/static_wave_02.wav",
		"uniginescript_samples/sounds/sounds/static_wave_03.wav",
	);
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			string path = fullPath(names[num % names.size()]);
			SoundSource source = addToEditor(new SoundSource(path));
			source.setWorldTransform(translate(Vec3(x,y,1.0f) * 20.0f));
			source.setLoop(1);
			source.setMinDistance(20.0f);
			source.setMaxDistance(40.0f);
			source.play();
			
			num++;
		}
	}
	
	setDescription(format("%d Static SoundSources",num));
	
	return 1;
}

```

## sound_static_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
SoundSource source;
SoundSource sources[0];

/*
 */
void contact_callback(Body body,int num) {
	Vec3 position = body.getContactPoint(num);
	float impulse = body.getContactImpulse(num);
	float volume = impulse * 0.5f - 1.0f;
	if(volume < 0.1f) return;
	SoundSource s = NULL;
	forloop(int i = 0; sources.size()) {
		if(sources[i].isPlaying() == 0) {
			s = sources[i];
			break;
		}
	}
	if(s == NULL) {
		s = node_append(source.clone());
		sources.append(s);
	}
	s.setEnabled(1);
	s.setGain(saturate(volume));
	s.setWorldTransform(translate(position));
	s.play();
}

/*
 */
void create_box(Vec3 position) {
	
	ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
	mesh.setWorldTransform(translate(position));
	mesh.setMaterial(findMaterialByName("mesh_base"),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	BodyRigid body = class_remove(new BodyRigid(mesh));
	class_remove(new ShapeBox(body,vec3_one));
	
	body.getEventContactEnter().connect(functionid(contact_callback));
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	source = addToEditor(new SoundSource(fullPath("uniginescript_samples/sounds/sounds/static_impact_00.wav")));
	source.setEnabled(0);
	source.setOcclusion(0);
	source.setMinDistance(10.0f);
	source.setMaxDistance(100.0f);
	
	for(int i = -5; i <= 5; i++) {
		create_box(Vec3(0.0f,i * 2.0f,8.0f + i * 1.0f));
	}
	
	create_box(Vec3(0.0f,0.0f,50.0f));
	
	setDescription("Contact SoundSource");
	
	return 1;
}

int update() {
	
	foreach(SoundSource s; sources) {
		if(s.isPlaying()) s.renderVisualizer();
		else s.setEnabled(0);
	}
	
	return 1;
}

```

## sound_static_04.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
ObjectMeshStatic meshes[0];
SoundSource source;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	for(int i = 0; i < 5; i++) {
		ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
		mesh.setMaterial(findMaterialByName("mesh_base"),"*");
		mesh.setSurfaceProperty("surface_base","*");
		mesh.setSoundOcclusion(0.5f, 0);
		meshes.append(mesh);
	}
	
	source = addToEditor(new SoundSource(fullPath("uniginescript_samples/sounds/sounds/static_mono_02.oga")));
	source.setWorldTransform(translate(Vec3(0.0f,0.0f,10.2f)));
	source.setLoop(1);
	source.setMinDistance(50.0f);
	source.setMaxDistance(100.0f);
	source.play();
	
	setDescription("SoundSource occlusion");
	
	return 1;
}

int update() {
	
	float time = engine.game.getTime();
	
	foreach(ObjectMeshStatic mesh, i = 0; meshes; i++) {
		mesh.setWorldTransform(Mat4(rotateZ(-time * 30.0f) * rotateZ(72.0f * i + 180.0f) * translate(6.0f,0.0f,0.0f) * rotateZ(180.0f)));
	}
	
	source.renderVisualizer();
	
	return 1;
}

```

## sound_stream_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Info info;
SoundSource source;
float pitch = 1.0f;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	info = new Info("'z' - loop toggle\n'x' - play\n'c' - stop\n'n' - increase pitch\n'b' - decrease pitch");
	
	source = addToEditor(new SoundSource(fullPath("uniginescript_samples/sounds/sounds/stream_mono_00.oga"),1));
	source.setWorldTransform(translate(Vec3(0.0f,0.0f,8.0f)));
	source.setLoop(1);
	source.setMinDistance(50.0f);
	source.setMaxDistance(100.0f);
	source.play();
	
	setDescription("Stream SoundSource");
	
	return 1;
}

int update() {
	
	if(engine.input.isKeyDown(INPUT_KEY_Z)) {
		source.setLoop(!source.getLoop());
		log.message("loop %d\n",source.getLoop());
	}
	if(engine.input.isKeyDown(INPUT_KEY_X)) {
		source.play();
		log.message("play\n");
	}
	if(engine.input.isKeyDown(INPUT_KEY_C)) {
		source.stop();
		log.message("stop\n");
	}
	if(engine.input.isKeyPressed(INPUT_KEY_N)) {
		pitch = clamp(pitch + engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	if(engine.input.isKeyPressed(INPUT_KEY_B)) {
		pitch = clamp(pitch - engine.game.getIFps(),0.1f,4.0f);
		source.setPitch(pitch);
		log.message("pitch %f\n",pitch);
	}
	
	if(source.isPlaying()) {
		engine.console.onscreenMessage("time: %f\n",source.getTime());
	}
	
	source.renderVisualizer();
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## spacer_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
FieldSpacer spacers[0];

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("FieldSpacer and ObjectGrass");
	
	ObjectGrass grass = addToEditor(new ObjectGrass());
	grass.setWorldTransform(translate(Vec3(-128.0f,-128.0f,0.0f)));
	grass.setIntersection(1);
	grass.setMaxVisibleDistance(50.0f,0);
	grass.setMaxFadeDistance(200.0f,0);
	grass.setSizeX(256.0f);
	grass.setSizeY(256.0f);
	grass.setStep(64.0f);
	grass.setDensity(0.2f);
	grass.setMinHeight(vec4(3.0f),vec4(1.0f));
	grass.setMaxHeight(vec4(3.0f),vec4(1.0f));
	grass.setAspect(vec4(1.0f),vec4(0.5f));
	grass.setMaterial(findMaterialByName("fields_grass_spacer"),"*");
	
	for(int i = 0; i < 8; i++) {
		FieldSpacer spacer = addToEditor(new FieldSpacer(vec3(16.0f)));
		spacer.setWorldTransform(Mat4(rotateZ(45.0f * i) * translate(32.0f,0.0f,0.0f)));
		spacer.setAttenuation(8.0f);
		spacer.setEllipse(i % 2);
		spacers.append(spacer);
	}
	
	return 1;
}

/*
 */
int update() {
	engine.visualizer.setEnabled(1);
	float ifps = engine.game.getIFps();
	foreach(FieldSpacer spacer; spacers) {
		spacer.setWorldTransform(Mat4(rotateZ(ifps * 16.0f)) * spacer.getWorldTransform());
		spacer.renderVisualizer();
	}
	
	return 1;
}

```

## spacer_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
FieldSpacer spacers[0];

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("FieldSpacer and ObjectWater");
	
	for(int i = 0; i < 8; i++) {
		FieldSpacer spacer = addToEditor(new FieldSpacer(vec3(16.0f)));
		spacer.setWorldTransform(Mat4(rotateZ(45.0f * i) * translate(32.0f,0.0f,0.0f)));
		spacer.setAttenuation(8.0f);
		spacer.setEllipse(i % 2);
		spacers.append(spacer);
	}
	
	return 1;
}

/*
 */
int update() {
	engine.visualizer.setEnabled(1);
	float ifps = engine.game.getIFps();
	foreach(FieldSpacer spacer; spacers) {
		spacer.setWorldTransform(Mat4(rotateZ(ifps * 16.0f)) * spacer.getWorldTransform());
		spacer.renderVisualizer();
	}
	
	return 1;
}

```

## spacer_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
FieldSpacer spacers[0];

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("FieldSpacer and ObjectWaterMesh");
	
	//ObjectMeshDynamic mesh = Unigine::createPlane(128.0f,128.0f,2.0f);
	//mesh.save("uniginescript_samples/fields/meshes/spacer_02.mesh");
	
	ObjectWaterMesh water = addToEditor(new ObjectWaterMesh(fullPath("uniginescript_samples/lights/meshes/water.mesh")));
	water.setMeshPath(fullPath("uniginescript_samples/lights/meshes/water.mesh"));
	water.setWorldTransform(translate(Vec3(0.0f,0.0f,0.0f)));
	water.setMaterial(findMaterialByName("fields_water_spacer"),"*");
	water.setSurfaceProperty("surface_base","*");
	
	for(int i = 0; i < 8; i++) {
		FieldSpacer spacer = addToEditor(new FieldSpacer(vec3(16.0f)));
		spacer.setWorldTransform(Mat4(rotateZ(45.0f * i) * translate(32.0f,0.0f,0.0f)));
		spacer.setAttenuation(8.0f);
		spacer.setEllipse(i % 2);
		spacers.append(spacer);
	}
	
	return 1;
}

/*
 */
int update() {
	engine.visualizer.setEnabled(1);
	float ifps = engine.game.getIFps();
	foreach(FieldSpacer spacer; spacers) {
		spacer.setWorldTransform(Mat4(rotateZ(ifps * 16.0f)) * spacer.getWorldTransform());
		spacer.renderVisualizer();
	}
	
	return 1;
}

```

## spectator_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
PlayerSpectator spectator;

using Unigine::Samples;

/*
 */
string material_names[] = ( "players_red", "players_green", "players_blue", "players_orange", "players_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	spectator.setPhiAngle(spectator.getPhiAngle() + sin(time * 10.0f) * 0.02f);
	spectator.setThetaAngle(spectator.getThetaAngle() + cos(time * 10.0f) * 0.02f);
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlane();
	setDescription("Spectator");
	
	int size = 2;
	float space = 4.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/players/meshes/sphere_00.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,1.0f) * space));
			mesh.setMaterial(findMaterialByName(get_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	spectator = addToEditor(new PlayerSpectator());
	spectator.setPosition(Vec3(-20.0f,0.0f,15.0f));
	spectator.setDirection(vec3(1.0f,0.0f,-0.8f));
	engine.game.setPlayer(spectator);
	
	return 1;
}

```

## sphere_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 2;
	int height = 20;
	
	for(int k = 0; k <= height; k++) {
		for(int j = -size; j <= size; j++) {
			for(int i = -size; i <= size; i++) {
				createBodySphere(0.5f,1.0f,0.5f,0.5f,get_material((i ^ j * 2) + k),translate(Vec3(i,j,k + 0.5f)));
				num++;
			}
		}
	}
	
	setDescription(format("%d ShapeSphere",num));
	
	return 1;
}

```

## sphere_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 2;
	int height = 20;
	
	for(int k = 0; k <= height; k++) {
		for(int j = -size; j <= size; j++) {
			for(int i = -size; i <= size; i++) {
				createBodySphere(0.5f,1.0f,0.5f,0.5f,get_material((i ^ j * 2) + k),translate(Vec3(i,j,k + 0.5f)));
				num++;
			}
		}
	}
	
	createBodyBox(vec3(17.0f,1.0f,2.0f),0.0f,0.5f,0.5f,get_material(0),translate(Vec3( 0.5f, 8.5f,1.0f)));
	createBodyBox(vec3(17.0f,1.0f,2.0f),0.0f,0.5f,0.5f,get_material(0),translate(Vec3(-0.5f,-8.5f,1.0f)));
	createBodyBox(vec3(1.0f,17.0f,2.0f),0.0f,0.5f,0.5f,get_material(0),translate(Vec3( 8.5f,-0.5f,1.0f)));
	createBodyBox(vec3(1.0f,17.0f,2.0f),0.0f,0.5f,0.5f,get_material(0),translate(Vec3(-8.5f, 0.5f,1.0f)));
	
	createBodyBox(vec3(5.0f,5.0f,1.0f),1.0f,0.5f,0.5f,get_material(2),translate(Vec3(0.0f,0.0f,30.0f)));
	
	setDescription(format("%d ShapeSphere",num));
	
	return 1;
}

```

## sphere_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
void create_body(float density,int cork,Mat4 transform) {
	
	ObjectMeshDynamic mesh = addToEditor(new ObjectMeshDynamic());
	mesh.setMaterial(findMaterialByName(get_material(0)),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	Body body = class_remove(new BodyRigid(mesh));
	
	void create_box(vec3 size,Mat4 transform) {
		Unigine::createBox(mesh,size,transform);
		ShapeBox shape = class_remove(new ShapeBox(size));
		body.addShape(shape,transform);
		shape.setMass(shape.getVolume() * density);
		shape.setFriction(0.5f);
		shape.setRestitution(0.5f);
	}
	
	if(cork) create_box(vec3(4.0f,4.0f,0.5f),translate(0.0f,0.0f,0.25f));
	create_box(vec3(13.0f,8.0f,0.5f),translate( 0.0f,-4.0f,3.5f) * rotateX(-52.0f));
	create_box(vec3(13.0f,8.0f,0.5f),translate( 0.0f, 4.0f,3.5f) * rotateX( 52.0f));
	create_box(vec3(8.0f,13.0f,0.5f),translate(-4.0f, 0.0f,3.5f) * rotateY( 52.0f));
	create_box(vec3(8.0f,13.0f,0.5f),translate( 4.0f, 0.0f,3.5f) * rotateY(-52.0f));
	
	mesh.setWorldTransform(transform);
}
/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(40.0f,0.0f,30.0f));
	createPlaneWithBody();
	
	// create shapes
	int num = 0;
	
	int size = 2;
	int height = 20;
	
	for(int k = 0; k <= height; k++) {
		for(int j = -size; j <= size; j++) {
			for(int i = -size; i <= size; i++) {
				createBodySphere(0.5f,1.0f,0.5f,0.5f,get_material((i ^ j * 2) + k),translate(Vec3(i,j,k + 20.0f)));
				num++;
			}
		}
	}
	
	create_body(0.0f,0,translate(Vec3(0.0f,0.0f,12.0f)));
	create_body(1.0f,1,translate(Vec3(0.0f,0.0f,0.0f)));
	
	setDescription(format("%d ShapeSphere",num));
	
	return 1;
}

```

## sphere_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// create shapes
	ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/shapes/meshes/mesh_00.mesh")));
	mesh.setMaterial(findMaterialByName("shapes_ground"),"*");
	mesh.setSurfaceProperty("surface_base","*");
	mesh.setMaterial(findMaterialByName(get_material(0)),"boxes");
	mesh.setMaterial(findMaterialByName(get_material(1)),"prisms");
	mesh.setMaterial(findMaterialByName(get_material(2)),"pyramids");
	mesh.setMaterial(findMaterialByName(get_material(3)),"parallelepipeds");

	forloop(int i = 0; 5;)
		mesh.setCollision(1,i);
	
	int num = 0;
	
	int size = 8;
	vec3 space = vec3(2.25f,2.25f,1.0f);
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			createBodySphere(0.5f,1.0f,0.5f,0.5f,get_material(i ^ j * 2),translate(Vec3(i,j,8.0f) * space));
			num++;
		}
	}
	
	setDescription(format("%d ShapeSphere ObjectMeshStatic",num));
	
	return 1;
}

```

## sphere_04.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "shapes_red", "shapes_green", "shapes_blue", "shapes_orange", "shapes_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.05f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create shapes
	ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/shapes/meshes/sphere_03.mesh")));
	mesh.setMaterial(findMaterialByName(get_material(0)),"*");
	forloop(int i = 0; mesh.getNumSurfaces()) {
		mesh.setPhysicsRestitution(1.0f,i);
		mesh.setCollision(1,i);
	}
	
	int size = 10;
	for(int i = -size; i <= size; i++) {
		BodyRigid body = createBodySphere(0.5f,1.0f,0.5f,1.0f,get_material(i),translate(Vec3(i * 1.1f,0.0f,6.0f)));
		if(i & 0x01) body.setLinearVelocity(vec3(0.0f,200.0f,0.0f));
		else body.setLinearVelocity(vec3(0.0f,-200.0f,0.0f));
	}
	
	setDescription("ShapeSphere ObjectMeshStatic CCD");
	
	return 1;
}
	

```

## spline_graph_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "worlds_mesh_red", "worlds_mesh_green", "worlds_mesh_blue", "worlds_mesh_orange", "worlds_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
SplineGraph spline;
ObjectMeshStatic mesh;

/*
 */
vec3 rand_cube_vector(float min, float max) {
	vec3 ret = Vec3_zero;
	ret.x = rand(min, max);
	ret.y = rand(min, max);
	ret.z = rand(1.0f, 2.0f);
	return ret;
}

void generate_graph(SplineGraph g) {
	g.addPoint(rand_cube_vector(-10.0f, 10.0f));
	g.addPoint(rand_cube_vector(-10.0f, 10.0f));
	g.addPoint(rand_cube_vector(-10.0f, 10.0f));
	g.addPoint(rand_cube_vector(-10.0f, 10.0f));

	g.addSegment(0, rand_cube_vector(-10.0f, 10.0f), vec3(0.0f,0.0f,1.0f), 1, rand_cube_vector(-10.0f, 10.0f), vec3(0.0f,0.0f,1.0f));
	g.addSegment(1, rand_cube_vector(-10.0f, 10.0f), vec3(0.0f,0.0f,1.0f), 2, rand_cube_vector(-10.0f, 10.0f), vec3(0.0f,0.0f,1.0f));
	g.addSegment(2, rand_cube_vector(-10.0f, 10.0f), vec3(0.0f,0.0f,1.0f), 3, rand_cube_vector(-10.0f, 10.0f), vec3(0.0f,0.0f,1.0f));
	g.addSegment(3, rand_cube_vector(-10.0f, 10.0f), vec3(0.0f,0.0f,1.0f), 0, rand_cube_vector(-10.0f, 10.0f), vec3(0.0f,0.0f,1.0f));
}

void smooth_tangents(SplineGraph g) {

	for (int i = 0; i < g.getNumSegments() - 1; i++)
	{
		vec3 t0 = g.getSegmentEndTangent(i);
		vec3 t1 = g.getSegmentStartTangent(i + 1);

		vec3 sum = t0 + t1;

		t0 = normalize(t0 * 2.0f - sum) * length(t0) * 0.5f;
		t1 = normalize(t1 * 2.0f - sum) * length(t1) * 0.5f;

		g.setSegmentEndTangent(i, t0);
		g.setSegmentStartTangent(i + 1, t1);
	}
}

void render_graph(SplineGraph g) {

	int num_segments = 50;
	forloop (int i = 0; g.getNumPoints()) {
		Vec3 p = Vec3(g.getPoint(i));
		engine.visualizer.renderPoint3D(p, 0.05f, vec4(1.0f, 0.0f, 0.0f, 1.0f));
	}

	forloop (int i = 0; g.getNumSegments()) {
		Vec3 start_point = Vec3(g.getSegmentStartPoint(i));
		Vec3 start_tangent = Vec3(g.getSegmentStartTangent(i));
		Vec3 end_point = Vec3(g.getSegmentEndPoint(i));
		Vec3 end_tangent = Vec3(g.getSegmentEndTangent(i));

		engine.visualizer.renderVector(start_point, start_point + start_tangent, vec4(1.0f, 0.0f, 0.0f, 0.3f), 0.05f);
		engine.visualizer.renderVector(end_point, end_point + end_tangent, vec4(0.0f, 1.0f, 0.0f, 0.3f), 0.05f);

		forloop (int j = 0; num_segments) {
			Vec3 p0 = Vec3(g.calcSegmentPoint(i, float(j) / num_segments));
			Vec3 p1 = Vec3(g.calcSegmentPoint(i, float(j + 1) / num_segments));
			engine.visualizer.renderLine3D(p0, p1, vec4(1.0f,1.0f,1.0f,1.0f));
		}
	}
}

void move_mesh()
{
	float time = engine.game.getTime();

	float t = time - floor(time / spline.getNumSegments()) * spline.getNumSegments();
	int segment_id = int(t);
	t -= float(segment_id);

	Vec3 p = Vec3(spline.calcSegmentPoint(segment_id, t));
	Vec3 direction = Vec3(spline.calcSegmentTangent(segment_id, t));
	Vec3 up = Vec3(spline.calcSegmentUpVector(segment_id, t));

	mesh.setWorldPosition(p);
	mesh.setWorldDirection(direction, up);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	engine.console.run("show_visualizer 1");
	
	mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
	mesh.setMaterial(findMaterialByName(get_mesh_material(0)),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	// create spline graph
	spline = new SplineGraph();
	
	// generate spline
	generate_graph(spline);
	
	// smooth tangengs
	smooth_tangents(spline);
	
	setDescription("Spline Graph");
	
	return 1;
}

/*
 */
int update() {
	
	render_graph(spline);
	move_mesh();
	
	return 1;
}

/*
 */
int shutdown() {
	
	engine.console.run("show_visualizer 0");
	engine.console.flush();
	
	return 1;
}

```

## sprite_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Sprite {
	
	Gui gui;				// gui
	
	WidgetSprite sprite;	// sprite
	
	float scales[0];		// scales
	vec3 positions[0];		// positions
	vec3 velocities[0];		// velocities
	
	// constructor/destructor
	Sprite() {
		
		int restore = (scales.size() != 0);
		
		scales.clear();
		positions.clear();
		velocities.clear();
		
		gui = engine.getGui();
		
		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 window_size = main_window.getSize();
		
		// sprite
		sprite = new WidgetSprite(gui,fullPath("uniginescript_samples/widgets/textures/background.png"));
		sprite.setWrapRepeat(1);
		sprite.arrange();
		
		string names[] = (
			fullPath("uniginescript_samples/widgets/textures/sprite_00.png"),
			fullPath("uniginescript_samples/widgets/textures/sprite_01.png"),
		);
		
		for(int i = 0; i < 64; i++) {
			sprite.setLayerTexture(sprite.addLayer(),names[i % names.size()]);
			scales.append(engine.game.getRandom(0.25f,1.0f));
			float size = 256.0f * scales[scales.size() - 1];
			positions.append(engine.game.getRandom(vec3(0.0f,0.0f,0.0f),vec3(window_size.x - size, window_size.y - size,360.0f)));
			velocities.append(engine.game.getRandom(vec3(-64.0f,-64.0f,-128.0f),vec3(64.0f,64.0f,128.0f)));
		}
		
		gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_BACKGROUND);
		
		if(restore == 0) thread("Sprite::update_redirector",this);
	}
	~Sprite() {
		delete sprite;
	}
	
	// save/restore state
	void __restore__() {
		__Sprite__();
	}
	
	// update
	void update() {
		
		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 window_size = main_window.getSize();
		
		float tile_x = float(window_size.x) / sprite.getLayerWidth(0);
		float tile_y = float(window_size.y) / sprite.getLayerHeight(0);
		sprite.setTransform(scale(tile_x,tile_y,1.0f));
		sprite.setTexCoord(vec4(0.0f,0.0f,tile_x,tile_y));
		
		float hsize = 128.0f;
		float ifps = engine.game.getIFps();
		forloop(int i = 0; positions.size()) {
			float size = 256.0f * scales[i];
			vec3 p = positions[i] + velocities[i] * ifps;
			if(p.x < 0.0f || p.x + size > window_size.x) velocities[i].x = -velocities[i].x;
			if(p.y < 0.0f || p.y + size > window_size.y) velocities[i].y = -velocities[i].y;
			positions[i] += velocities[i] * ifps;
			mat4 rotate = translate(hsize,hsize,0.0f) * rotateZ(positions[i].z) * translate(-hsize,-hsize,0.0f);
			sprite.setLayerTransform(i + 1,translate(positions[i]) * scale(vec3(scales[i])) * rotate);
		}
	}
	
	void update_redirector(Sprite sprite) {
		while(1) {
			sprite.update();
			wait;
		}
	}
};

Sprite sprite;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	sprite = new Sprite();
	
	setDescription("WidgetSprite");
	
	return 1;
}

```

## sprite_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Sprite {
	
	int x,y;
	string material;
	WidgetSpriteShader sprite;	// sprite
	
	// constructor/destructor
	Sprite(int x,int y,string material = 0) {
		
		this.x = x;
		this.y = y;
		this.material = material;
		
		Gui gui = engine.getGui();
		
		// sprite
		sprite = new WidgetSpriteShader(gui,fullPath("uniginescript_samples/widgets/textures/sprite_00.png"));
		sprite.setMaterial(findMaterialByName(material));
		sprite.setBlendFunc(GUI_BLEND_NONE,GUI_BLEND_NONE);
		sprite.setPosition(x,y);
		
		gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_BACKGROUND);
	}
	~Sprite() {
		delete sprite;
	}
	
	// save/restore state
	void __restore__() {
		__Sprite__(x,y,material);
	}
};

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	Sprite sprites[0];
	string materials[] = ( "post_blur_radial", "post_filter_sobel", "post_filter_rgb2yuv" );
	
	EngineWindow main_window = engine.window_manager.getMainWindow();
	ivec2 window_size = main_window.getSize();
	
	for(int i = 0; i < materials.size(); i++)
	{
		int x = window_size.x / 2 - materials.size() * 128;
		sprites.append(new Sprite(x + 256 * i,256,materials[i]));
	}
	
	setDescription("WidgetSpriteShader");
	
	return 1;
}

```

## sprite_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Grid {
	
	// constructor/destructor
	Grid() {
		
		Gui gui = engine.getGui();
		
		// grid
		WidgetGridBox grid = new WidgetGridBox(gui,6);
		
		// sprites
		forloop(int i = 0; 24) {
			vec4 color = engine.game.getRandom(vec4(0.0f,0.0f,0.0f,1.0f),vec4(1.0f,1.0f,1.0f,1.0f));
			WidgetSprite sprite = new WidgetSprite(gui,fullPath("core/gui/gui_white.png"));
			sprite.getEventEnter().connect("Grid::sprite_enter",sprite,color,i);
			sprite.getEventLeave().connect("Grid::sprite_leave",sprite,color,i);
			sprite.getEventClicked().connect("Grid::sprite_clicked",sprite,color,i);
			sprite.setWidth(64);
			sprite.setHeight(64);
			sprite.setColor(color);
			grid.addChild(sprite);
		}
		
		grid.arrange();
		gui.addChild(grid,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
	}
	~Grid() {
		
	}
	
	// save/restore state
	void __restore__() {
		__Grid__();
	}
	
	void sprite_enter(WidgetSprite sprite,vec4 color,int id) {
		engine.console.onscreenMessage("enter: %d\n",id);
		sprite.setColor(vec4_one);
	}
	void sprite_leave(WidgetSprite sprite,vec4 color,int id) {
		engine.console.onscreenMessage("leave: %d\n",id);
		sprite.setColor(color);
	}
	void sprite_clicked(WidgetSprite sprite,vec4 color,int id) {
		engine.console.onscreenMessage("clicked: %d\n",id);
	}
};

Grid grid;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	grid = new Grid();
	
	
	setDescription("WidgetSprite callbacks");
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## sprite_03.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Grid {
	
	// constructor/destructor
	Grid() {
		
		Gui gui = engine.getGui();
		
		// grid
		WidgetGridBox grid = new WidgetGridBox(gui,6);
		
		// sprites
		forloop(int i = 0; 24) {
			vec4 color = engine.game.getRandom(vec4(0.0f,0.0f,0.0f,1.0f),vec4(1.0f,1.0f,1.0f,1.0f));
			WidgetSprite sprite = new WidgetSprite(gui,fullPath("core/gui/gui_white.png"));
			sprite.getEventDragMove().connect("Grid::sprite_drag_move",sprite);
			sprite.getEventDragDrop().connect("Grid::sprite_drag_drop",sprite);
			sprite.setWidth(64);
			sprite.setHeight(64);
			sprite.setColor(color);
			grid.addChild(sprite);
		}
		
		grid.arrange();
		gui.addChild(grid,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
	}
	~Grid() {
		
	}
	
	// save/restore state
	void __restore__() {
		__Grid__();
	}
	
	void sprite_drag_move(WidgetSprite sprite,Widget widget) {
		WidgetSprite dest = widget_cast(widget);
		engine.console.onscreenMessage("move: %s->%s\n",string(sprite.getColor()),string(dest.getColor()));
	}
	
	void sprite_drag_drop(WidgetSprite sprite,Widget widget, Widget widget2) {
		WidgetSprite source = widget_cast(widget2);
		vec4 color = source.getColor();
		source.setColor(sprite.getColor());
		sprite.setColor(color);
	}
};

Grid grid;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	grid = new Grid();
	
	setDescription("WidgetSprite drag and drop");
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## steam_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
WidgetWindow window;
WidgetEditLine score_el;
WidgetGridBox table_gb;

#ifdef HAS_STEAM
SteamLeaderboard leaderboard;

/*
 */
void upload_clicked() {
	leaderboard.uploadScore(int(score_el.getText()),1);
}

void download_clicked() {
	leaderboard.downloadScores(STEAM_DATA_REQUEST_GLOBAL_AROUND_USER,-19,20);
}

void overlay_clicked() {
	steam.showOverlay("Friends");
}

/*
 */
void overlay_shown(int shown) {
	log.message("overlay_shown(%d): called\n",shown);
}

void leaderboard_found(SteamLeaderboard leaderboard) {
	log.message("leaderboard_found(\"%s\"): called\n",leaderboard.getName());
	download_clicked();
}

void leaderboard_scores_downloaded(SteamLeaderboard leaderboard,int failed,int data_request) {
	if(failed) {
		log.error("leaderboard_scores_downloaded(): can't download \"%s\" leaderboard scores\n",leaderboard.getName());
		return;
	}
	log.message("leaderboard_scores_downloaded(\"%s\",%d,%d): called\n",leaderboard.getName(),failed,data_request);
	reload();
}

void leaderboard_score_uploaded(SteamLeaderboard leaderboard,int failed,int score_changed) {
	if(failed) {
		log.error("leaderboard_scores_downloaded(): can't upload \"%s\" leaderboard scores\n",leaderboard.getName());
		return;
	}
	log.message("leaderboard_score_uploaded(\"%s\",%d,%d): called\n",leaderboard.getName(),failed,score_changed);
	if(score_changed) download_clicked();
}

/*
 */
void reload() {
	
	Gui gui = engine.getGui();
	
	// delete widgets
	for(int i = table_gb.getNumChildren() - 1; i >= 0; i--) {
		Widget child = widget_cast(table_gb.getChild(i));
		table_gb.removeChild(child);
		delete child;
	}
	
	// create label
	WidgetLabel create_label(string text,int flags) {
		WidgetLabel label = new WidgetLabel(gui,text);
		table_gb.addChild(label,flags);
		return label;
	}
	
	// create leaderboard table
	forloop(int i = 0; leaderboard.getNumEntries()) {
		long user = leaderboard.getEntryUserID(i);
		int score = leaderboard.getEntryScore(i);
		int rank = leaderboard.getEntryRank(i);
		
		WidgetLabel rank_l = create_label(format("#%d",rank),GUI_ALIGN_RIGHT);
		
		WidgetSprite avatar_s = new WidgetSprite(gui);
		avatar_s.setImage(steam.getUserAvatarMedium(user));
		table_gb.addChild(avatar_s);
		
		string name = steam.getUserName(user);
		WidgetLabel name_l = create_label(name,GUI_ALIGN_EXPAND);
		WidgetLabel score_l = create_label(format("%d",score),GUI_ALIGN_RIGHT);
		
		if(name == steam.getMyName()) {
			rank_l.setFontColor(vec4_one);
			name_l.setFontColor(vec4_one);
			score_l.setFontColor(vec4_one);
		}
	}
}

#endif

/*
 */
int init() {
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	Gui gui = engine.getGui();
	
	// window
	window = new WidgetWindow(gui);
	window.setSizeable(1);
	window.setSpace(8,8);
	
	// table scrollbox
	WidgetScrollBox scrollbox = new WidgetScrollBox(gui,4,4);
	window.addChild(scrollbox,GUI_ALIGN_EXPAND);
	scrollbox.setHScrollEnabled(0);
	scrollbox.setWidth(400);
	scrollbox.setHeight(300);
	
	// table gridbox
	table_gb = new WidgetGridBox(gui,4,8,4);
	scrollbox.addChild(table_gb,GUI_ALIGN_EXPAND);
	
	// bottom vbox
	WidgetVBox vbox = new WidgetVBox(gui);
	window.addChild(vbox);
	
	// bottom hbox
	WidgetHBox hbox = new WidgetHBox(gui,4,4);
	vbox.addChild(hbox,GUI_ALIGN_EXPAND);
	
	// score
	hbox.addChild(new WidgetLabel(gui,"Score:"),GUI_ALIGN_RIGHT);
	score_el = new WidgetEditLine(gui,"10000");
	hbox.addChild(score_el,GUI_ALIGN_EXPAND);
	
	// upload button
	WidgetButton button = new WidgetButton(gui,"Upload");
	button.getEventClicked().connect("upload_clicked");
	hbox.addChild(button,GUI_ALIGN_EXPAND);
	
	// download button
	button = new WidgetButton(gui,"Download");
	button.getEventClicked().connect("download_clicked");
	hbox.addChild(button,GUI_ALIGN_EXPAND);
	
	// overlay button
	button = new WidgetButton(gui,"Overlay");
	button.getEventClicked().connect("overlay_clicked");
	hbox.addChild(button,GUI_ALIGN_EXPAND);
	
	// show window
	window.arrange();
	gui.addChild(window,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
	
	#ifdef HAS_STEAM
	// steam callbacks
	steam.setCallback(STEAM_CALLBACK_OVERLAY_SHOWN,"overlay_shown");
	steam.setCallback(STEAM_CALLBACK_LEADERBOARD_FOUND,"leaderboard_found");
	steam.setCallback(STEAM_CALLBACK_LEADERBOARD_SCORES_UPLOADED,"leaderboard_score_uploaded");
	steam.setCallback(STEAM_CALLBACK_LEADERBOARD_SCORES_DOWNLOADED,"leaderboard_scores_downloaded");
	
	// create leaderboard
	leaderboard = new SteamLeaderboard("Feet Traveled");
	leaderboard.find();
	
	setDescription("Steam plugin");
	#else
	setDescription("Steam plugin is not loaded");
	#endif
	
	return 1;
}

int update() {
	#ifdef HAS_STEAM
	string name = leaderboard.getName();
	
	if(leaderboard.isFound() == 0) window.setText(format("Finding leaderboard %s",name));
	else if(leaderboard.isUploading() != 0) window.setText(format("Uploading leaderboard %s",name));
	else if(leaderboard.isDownloading() != 0) window.setText(format("Downloading leaderboard %s",name));
	else if(leaderboard.isLastUploadFailed() != 0) window.setText(format("Upload failed for leaderboard %s",name));
	else if(leaderboard.isLastDownloadFailed() != 0) window.setText(format("Download failed for leaderboard %s",name));
	else if(leaderboard.getNumEntries() == 0) window.setText(format("No scores for %s",name));
	else window.setText(leaderboard.getName());
	#endif
	
	return 1;
}

int shutdown() {
	#ifdef HAS_STEAM
	// delete leaderboard
	delete leaderboard;
	
	// steam callbacks
	steam.setCallback(STEAM_CALLBACK_OVERLAY_SHOWN,NULL);
	steam.setCallback(STEAM_CALLBACK_LEADERBOARD_FOUND,NULL);
	steam.setCallback(STEAM_CALLBACK_LEADERBOARD_SCORES_UPLOADED,NULL);
	steam.setCallback(STEAM_CALLBACK_LEADERBOARD_SCORES_DOWNLOADED,NULL);
	#endif
	
	engine.console.setOnscreen(0);

	return 1;
}

```

## stream_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

ObjectMeshStatic mesh;
Material material;
int texture = -1;

string names[] = (
	fullPath("uniginescript_samples/systems/textures/streams_00.dds"),
	fullPath("uniginescript_samples/systems/textures/streams_01.dds"),
	fullPath("uniginescript_samples/systems/textures/streams_02.dds"),
	fullPath("uniginescript_samples/systems/textures/streams_03.dds"),
);

int ids[] = (
	-1,
	-1,
	-1,
	-1,
);

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	light_world_0.setScattering(LIGHT_WORLD_SCATTERING_SUN);
	light_world_1.setScattering(LIGHT_WORLD_SCATTERING_SUN);
	
	mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
	mesh.setWorldTransform(Mat4(scale(20.0f,20.0f,20.0f)));
	mesh.setMaterial(findMaterialByName("mesh_base"),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	material = mesh.getMaterialInherit(0);
	texture = material.findTexture("albedo");
	material.setTexturePath(texture,names[0]);
	
	thread("update_thread");
	
	setDescription("Streams");
	
	return 1;
}

/*
 */
void update_thread() {
	
	while(1) {
		forloop(int i = 0; names.size()) {
			
			string name = names[i];
			
			// threaded file loading
			if(engine.async.checkImage(ids[i]) == 0) {
				ids[i] = engine.async.loadImage(name);
			}
			
			// waiting until file has loaded and remove it from queue
			Image image = NULL;
			while(image == NULL)
			{
				image = engine.async.takeImage(ids[i]);
				wait;
			}
			
			// file is ready
			if (texture != -1) material.setTextureImage(texture,image);
			delete image;

			wait;
		}
	}
}

/*
 */
int shutdown() {
	forloop(int i = 0; names.size()) {
		engine.async.removeFile(ids[i]);
		ids[i] = -1;
	}
	return 1;
}

```

## surfaces_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 12;
	
	ObjectMeshStatic reference = new ObjectMeshStatic(fullPath("uniginescript_samples/stress/meshes/surfaces_00.mesh"));
	forloop(int i = 0; reference.getNumSurfaces()) {
		reference.setMaterial(findMaterialByName(get_mesh_material(i)),i);
		reference.setMinVisibleDistance(i * 8.0f - 8.0f,i);
		reference.setMaxVisibleDistance(i * 8.0f,i);
	}
	reference.setSurfaceProperty("surface_base","*");
	
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshStatic mesh = node_cast(reference.clone());
				mesh.setWorldTransform(translate(Vec3(x,y,z + 13.0f) * 1.1f));
				num++;
			}
		}
	}
	
	delete reference;
	
	setDescription(format("%d ObjectMeshStatic",num));
	
	return 1;
}

```

## surfaces_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 12;
	
	ObjectMeshStatic reference = new ObjectMeshStatic(fullPath("uniginescript_samples/stress/meshes/surfaces_00.mesh"));
	forloop(int i = 0; reference.getNumSurfaces()) {
		reference.setMaterial(findMaterialByName(get_mesh_material(i)),i);
		reference.setMinVisibleDistance(i * 8.0f - 8.0f,i);
		reference.setMaxVisibleDistance(i * 8.0f,i);
		reference.setMinFadeDistance(2.0f,i);
		reference.setMaxFadeDistance(2.0f,i);
	}
	reference.setSurfaceProperty("surface_base","*");
	
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshStatic mesh = node_cast(reference.clone());
				mesh.setWorldTransform(translate(Vec3(x,y,z + 13.0f) * 1.1f));
				num++;
			}
		}
	}
	
	delete reference;
	
	setDescription(format("%d ObjectMeshStatic with lod fading",num));
	
	return 1;
}

```

## surfaces_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string mesh_material_names[] = ( "stress_mesh_red", "stress_mesh_green", "stress_mesh_blue", "stress_mesh_orange", "stress_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 0;
	int size = 16;
	
	ObjectMeshStatic reference = new ObjectMeshStatic(fullPath("uniginescript_samples/stress/meshes/surfaces_00.mesh"));
	forloop(int i = 0; reference.getNumSurfaces()) {
		reference.setMaterial(findMaterialByName(get_mesh_material(i)),i);
		reference.setMinVisibleDistance(i * 64.0f - 64.0f,i);
		reference.setMaxVisibleDistance(i * 64.0f,i);
	}
	reference.setSurfaceProperty("surface_base","*");
	
	for(int z = -size; z <= size; z++) {
		for(int y = -size; y <= size; y++) {
			for(int x = -size; x <= size; x++) {
				ObjectMeshStatic mesh = node_cast(reference.clone());
				mesh.setWorldTransform(translate(Vec3(x,y,z + 17.0f) * 1.1f));
				num++;
			}
		}
	}
	
	delete reference;
	
	setDescription(format("%d ObjectMeshStatic",num));
	
	return 1;
}

```

## surround_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */

#ifdef HAS_SURROUND

WidgetLabel labels[0];
ObjectMeshStatic meshes[0];

/*
 */
int get_screen_position(Vec3 point,int &x,int &y)
{
	EngineWindow main_window = engine.window_manager.getMainWindow();
	ivec2 window_size = main_window.getSize();
	
	int width = int(float(window_size.x) / 3);
	int height = window_size.y;
	forloop(int i = 0; 3)
	{
		Camera cam = engine.surround.getCamera(i);
		
		mat4 projection = cam.getProjection();
		Mat4 modelview = cam.getModelview();
		projection.m00 *= float(height) / width;
		vec4 p = projection * vec4(modelview * Vec4(point,1.0f));
		if(p.w > 0.0f)
		{
			x = int(width * (0.5f + p.x * 0.5f / p.w));
			y = int(height * (0.5f - p.y * 0.5f / p.w));
			if(x < 0 || x >= width) continue;
			if(y < 0 || y >= height) continue;
			
			x += width*i;
			return i;
		}
	}
	return -1;
}

#endif

/*
 */
int init() {
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	#ifdef HAS_SURROUND
	setDescription("Surround gui");
	forloop(int i = 0; 64) {
		ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh")));
		mesh.setMaterial(findMaterialByName("mesh_base"),"*");
		mesh.setSurfaceProperty("surface_base","*");
		meshes.append(mesh);
	}
	#else
	setDescription("Surround plugin is not loaded");
	#endif
	
	return 1;
}

int update() {
	#ifdef HAS_SURROUND
	float time = engine.game.getTime();

	Mat4 transform = Mat4_identity;
	Player player = engine.editor.getPlayer();
	if(player == NULL) player = engine.game.getPlayer();
	if(player != NULL) transform = player.getWorldTransform();
	Gui gui = engine.getGui();
	if(labels.size() == 0 || labels[0] == NULL) {
		forloop(int i = 0; meshes.size()) {
			WidgetLabel label = new WidgetLabel(gui);
			label.setFontOutline(1);
			label.setFontSize(16);
			labels.append(label);
		}
	}

	int x,y;
	foreach(ObjectMeshStatic mesh, i = 0; meshes; i++) {
		float angle = time * 16.0f + 360.0f * i / meshes.size();
		mesh.setWorldTransform(transform * rotateY(angle) * translate(16.0f,sin(angle * 0.1f) * 2.0f,0.0f) * rotateY(angle));
		Vec3 position = mesh.getWorldPosition();
		int index = get_screen_position(position,x,y);
		WidgetLabel label = labels[i];
		if(index != -1) {
			label.setText(format("%.1f %.1f %.1f\n",position.x,position.y,position.z));
			gui.addChild(label,GUI_ALIGN_OVERLAP);
			label.arrange();
			label.setPosition(x - label.getWidth() / 2,y - label.getHeight() / 4);
		} else {
			label.setParent(NULL);
		}
	}
	#endif
	return 1;
}

int shutdown() {
	return 1;
}

```

## suspension_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(Mat4 transform) {
	
	int num = 0;
	
	void create_joint(Body b0,Body b1) {
		JointSuspension j = new JointSuspension(b0,b1);
		j.setWorldAxis0(rotation(transform) * vec3(0.0f,0.0f,1.0f));
		j.setWorldAxis1(rotation(transform) * vec3(0.0f,1.0f,0.0f));
		j.setLinearSpring(200.0f);
		j.setLinearDamping(2.0f);
		j.setLinearLimitFrom(-0.5f);
		j.setLinearLimitTo(0.0f);
		j.setAngularVelocity(-20.0f);
		j.setAngularTorque(10.0f);
		j.setLinearRestitution(0.2f);
		j.setAngularRestitution(0.2f);
		j.setLinearSoftness(0.2f);
		j.setAngularSoftness(0.2f);
		j.setNumIterations(2);
		num++;
	}
	
	Body body = createBodyBox(vec3(3.0f,2.0f,0.75f),1.0f,0.5f,0.5f,get_material(0),transform);
	
	Body b0 = createBodySphere(0.5f,1.0f,1.0f,0.5f,get_material(1),transform * translate( 1.25f, 1.25f,-0.5f));
	Body b1 = createBodySphere(0.5f,1.0f,1.0f,0.5f,get_material(1),transform * translate( 1.25f,-1.25f,-0.5f));
	Body b2 = createBodySphere(0.5f,1.0f,1.0f,0.5f,get_material(1),transform * translate( 0.00f, 1.25f,-0.5f));
	Body b3 = createBodySphere(0.5f,1.0f,1.0f,0.5f,get_material(1),transform * translate( 0.00f,-1.25f,-0.5f));
	Body b4 = createBodySphere(0.5f,1.0f,1.0f,0.5f,get_material(1),transform * translate(-1.25f, 1.25f,-0.5f));
	Body b5 = createBodySphere(0.5f,1.0f,1.0f,0.5f,get_material(1),transform * translate(-1.25f,-1.25f,-0.5f));
	
	create_joint(body,b0);
	create_joint(body,b1);
	create_joint(body,b2);
	create_joint(body,b3);
	create_joint(body,b4);
	create_joint(body,b5);
	
	return num;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	int size = 8;
	
	createBodyBox(vec3(50.0f,100.0f,2.0f),0.0f,0.5f,0.5f,get_material(2),translate(Vec3(-40.0f,0.0f,7.0f)) * rotateY(20.0f));
	createBodyBox(vec3(50.0f,100.0f,2.0f),0.0f,0.5f,0.5f,get_material(2),translate(Vec3( 40.0f,0.0f,7.0f)) * rotateY(-20.0f));
	
	for(int i = -size; i <= size; i++) {
		num += create_body(translate(Vec3(-60.0f,i * 4.0f,18.0f)));
		num += create_body(translate(Vec3(-40.0f,i * 4.0f,10.0f)));
		num += create_body(translate(Vec3(-20.0f,i * 4.0f, 3.0f)));
		num += create_body(translate(Vec3( 20.0f,i * 4.0f, 3.0f)) * rotateZ(180.0f));
		num += create_body(translate(Vec3( 40.0f,i * 4.0f,10.0f)) * rotateZ(180.0f));
		num += create_body(translate(Vec3( 60.0f,i * 4.0f,18.0f)) * rotateZ(180.0f));
	}
	
	setDescription(format("%d JointSuspension",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## switcher_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Node nodes[0];

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	// create sample
	int num = 32;
	
	for(int i = 0; i < num; i++) {
		nodes.append(addToEditor(node_load(fullPath("uniginescript_samples/worlds/meshes/switcher_00.node"))));
	}
	
	setDescription("WorldSwitcher");
	
	return 1;
}

/*
 */
int update() {
	
	float time = engine.game.getTime();
	
	for(int i = 0; i < nodes.size(); i++) {
		float angle = float(i) / nodes.size() * 2.0f * 360.0f;
		nodes[i].setWorldTransform(Mat4(rotateZ(time * 32.0f + angle) * translate(8.0f,0.0f,4.0f + i / 2.0f)));
	}
	
	return 1;
}

```

## system_dialog_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

WidgetButton message_button;
WidgetButton file_button;

string text = "Hello World!\nこんにちは世界！\nԲարեւ, աշխարհի!\n여보세요 세계!\n你好，世界！\nनमस्ते, दुनिया!\n!שלום, עולם\n!مرحبا ، العالم";

void system_dialog_clicked(SystemDialog dialog) {
	int ret = engine.window_manager.showSystemDialog(dialog);
	if (ret == -1)
	{
		log.message("esc\n");
		return;
	}
	log.message("%s\n\n", dialog.getButtonName(ret));
}

void message_clicked() {
	engine.window_manager.dialogMessage("Dialog message title", text);
}

void warning_clicked() {
	engine.window_manager.dialogWarning("Dialog warning title", text);
}

void error_clicked() {
	engine.window_manager.dialogError("Dialog error title", text);
}

void folder_clicked() {
	string path = engine.window_manager.dialogOpenFolder(engine.getDataPath());
	log.message("Folder:%s\n\n", path);
}

void files_open_clicked() {
	string files[0];
	engine.window_manager.dialogOpenFiles(files, engine.getDataPath(), "h,cpp,usc");
	if (files.size())
	{
		log.message("Files:\n");
		for(int i = 0; i < files.size(); i++)
			log.message("%s\n", files[i]);
		log.message("\n");
	}
}

void file_open_clicked() {
	string file = engine.window_manager.dialogOpenFile(engine.getDataPath(), "h,cpp,usc");
	if (strlen(file))
		log.message("File: %s\n\n", file);
}

void file_save_clicked() {
	string file = engine.window_manager.dialogSaveFile(engine.getDataPath(), "h,cpp,usc");
	if (strlen(file))
		log.message("File: %s\n\n", file);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	setDescription("System dialogs");
	
	return 1;
}

/*
 */
int update() {
	if(message_button != NULL)
		return 1;
	
	Gui gui = engine.getGui();
	
	SystemDialog system_dialog = new SystemDialog();
	system_dialog.setTitle("System dialog title");
	system_dialog.setMessage(text);
	system_dialog.addButton("Yes");
	system_dialog.addButton("No");
	system_dialog.addButton("Cancel");
	system_dialog.setDefaultButtonReturn(0);
	system_dialog.setDefaultButtonEscape(2);
	message_button = new WidgetButton(gui,"System dialog");
	message_button.setPosition(100,100);
	message_button.getEventClicked().connect("system_dialog_clicked", system_dialog);
	gui.addChild(message_button,GUI_ALIGN_OVERLAP);
	
	message_button = new WidgetButton(gui,"Dialog message");
	message_button.setPosition(100,140);
	message_button.getEventClicked().connect("message_clicked");
	gui.addChild(message_button,GUI_ALIGN_OVERLAP);
	
	message_button = new WidgetButton(gui,"Dialog warning");
	message_button.setPosition(100,170);
	message_button.getEventClicked().connect("warning_clicked");
	gui.addChild(message_button,GUI_ALIGN_OVERLAP);
	
	message_button = new WidgetButton(gui,"Dialog error");
	message_button.setPosition(100,200);
	message_button.getEventClicked().connect("error_clicked");
	gui.addChild(message_button,GUI_ALIGN_OVERLAP);
	
	file_button = new WidgetButton(gui,"Dialog open folder");
	file_button.setPosition(100,240);
	file_button.getEventClicked().connect("folder_clicked");
	gui.addChild(file_button,GUI_ALIGN_OVERLAP);
	
	file_button = new WidgetButton(gui,"Dialog open files (h,cpp,usc)");
	file_button.setPosition(100,280);
	file_button.getEventClicked().connect("files_open_clicked");
	gui.addChild(file_button,GUI_ALIGN_OVERLAP);
	
	file_button = new WidgetButton(gui,"Dialog open file (h,cpp,usc)");
	file_button.setPosition(100,310);
	file_button.getEventClicked().connect("file_open_clicked");
	gui.addChild(file_button,GUI_ALIGN_OVERLAP);
	
	file_button = new WidgetButton(gui,"Dialog save file (h,cpp,usc)");
	file_button.setPosition(100,340);
	file_button.getEventClicked().connect("file_save_clicked");
	gui.addChild(file_button,GUI_ALIGN_OVERLAP);
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## table_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

#include <core/systems/widgets/widget.h>
#include <core/systems/widgets/widget_window.h>
#include <core/systems/widgets/widget_icon.h>
#include <core/systems/widgets/widget_table.h>

/*
 */
Unigine::Widgets::Window window;
Unigine::Widgets::Icon close_icon;

Unigine::Widgets::Table table;

using Unigine::Samples;

void on_close()
{
	Unigine::Widgets::removeChild(window);
}

/*
 */
int update() {
	
	table.update();
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.setShadowDistance(100.0f);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("Unigine::Widgets::Table");
	
	using Unigine::Widgets;
	
	// window
	window = new Window("Unigine::Widgets::Table");
	window.setWidth(512);
	window.setSizeable(1);
	window.setDragAreaPadding(3, 25, 3, 0);
	
	// close icon
	close_icon = new Icon();
	close_icon.setPosition(10, -24);
	close_icon.getEventClicked().connect(functionid(on_close));
	window.addChild(close_icon, GUI_ALIGN_OVERLAP | GUI_ALIGN_TOP | GUI_ALIGN_RIGHT);

	Image img = new Image("core/gui/window_close.png");
	close_icon.setImage(img);

	// table
	table = new Table(("A","B","C","D"));
	window.addChild(table,ALIGN_EXPAND);
	ScrollBox scrollbox = table.getScrollBox();
	scrollbox.setVScrollEnabled(0);
	
	// create rows
	forloop(int i = 0; 8) {
		TableRow row = new TableRow(format("Unigine::Widgets::TableRow number %d",i),table.getNumColumns());
		table.addRow(row);
	}
	
	// update table
	table.update();
	
	// window
	window.arrange();
	addChild(window,ALIGN_OVERLAP | ALIGN_CENTER);
	
	return 1;
}

```

## terrain_global_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int terrain_num_lods = 3;

float terrain_base_height_scale = 3000.0f;
float terrain_base_density = 10.0f;
float terrain_base_density_power = 10.0f;
float terrain_base_visible_distance = 10000.0f;
float terrain_base_visible_distance_power = 10.0f;

int terrain_tileset_radius = 5;

double terrain_noise_size = 20000.0;
double iterrain_noise_size = rcp(terrain_noise_size);
int terrain_noise_frequency = 512;

vec4 terrain_lod_colors[] = (
	vec4(1.0f, 0.0f, 0.0f, 1.0f),
	vec4(0.0f, 1.0f, 0.0f, 1.0f),
	vec4(0.0f, 0.0f, 1.0f, 1.0f),
);

ObjectTerrainGlobal terrain;

float max_visible_distance = 400000.0f;

/*
 */
int init() {

	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(00.0f,0.0f,3500.0f), 200.0f, 20000.0f);
	setDescription("Generated ObjectTerrainGlobal with insets");

	Player player = engine.game.getPlayer();
	player.setZFar(max_visible_distance);

	RenderEnvironmentPreset preset = engine.render.getEnvironmentPreset(0);
	preset.setHazeMaxDistance(max_visible_distance);

	terrain = addToEditor(new ObjectTerrainGlobal());

	terrain.setMaterial(findMaterialByName("terrain_global_base"), "*");
	terrain.setSurfaceProperty("surface_base", "*");

	forloop(int i = 0; terrain_num_lods)
	{
		TerrainGlobalLods height_lods = terrain.getHeightLods();
		TerrainGlobalLods albedo_lods = terrain.getAlbedoLods();
		TerrainGlobalLods normal_lods = terrain.getNormalLods();

		height_lods.addLod();
		albedo_lods.addLod();
		normal_lods.addLod();

		mkdir(engine.filesystem.getAbsolutePath(fullPath(format("uniginescript_samples/objects/terrains/global_00/heights/lod%d/", i))), 1);
		mkdir(engine.filesystem.getAbsolutePath(fullPath(format("uniginescript_samples/objects/terrains/global_00/imagery/lod%d/", i))), 1);
		mkdir(engine.filesystem.getAbsolutePath(fullPath(format("uniginescript_samples/objects/terrains/global_00/normals/lod%d/", i))), 1);

		string height_lod_path = fullPath(format("uniginescript_samples/objects/terrains/global_00/heights/lod%d/lod%d.meta", i, i));
		string albedo_lod_path = fullPath(format("uniginescript_samples/objects/terrains/global_00/imagery/lod%d/lod%d.meta", i, i));
		string normal_lod_path = fullPath(format("uniginescript_samples/objects/terrains/global_00/normals/lod%d/lod%d.meta", i, i));

		TerrainGlobalLod height_lod = height_lods.getLod(i);
		TerrainGlobalLod albedo_lod = albedo_lods.getLod(i);
		TerrainGlobalLod normal_lod = normal_lods.getLod(i);

		height_lod.setPath(height_lod_path);
		albedo_lod.setPath(albedo_lod_path);
		normal_lod.setPath(normal_lod_path);

		float density_power = pow(terrain_base_density_power, i);
		height_lod.setTileDensity(terrain_base_density * density_power);
		albedo_lod.setTileDensity(terrain_base_density * density_power);
		normal_lod.setTileDensity(terrain_base_density * density_power);

		float visible_distance_power = pow(terrain_base_visible_distance_power, i);
		height_lod.setVisibleDistance(terrain_base_visible_distance * visible_distance_power);
		albedo_lod.setVisibleDistance(terrain_base_visible_distance * visible_distance_power);
		normal_lod.setVisibleDistance(terrain_base_visible_distance * visible_distance_power);

		fill_tilesets(terrain, i, terrain_tileset_radius);
	}

	engine.visualizer.setEnabled(1);

	return 1;
}

int shutdown() {
	removeFromEditor(terrain);
	rmdir(engine.filesystem.getAbsolutePath(fullPath("uniginescript_samples/objects/terrains")), 1);
	return 1;
}

int update() {

	forloop(int i = 0; terrain_num_lods) {
		TerrainGlobalLod height_lod = terrain.getHeightLods().getLod(i);
		Tileset tileset = height_lod.getTileset();

		dvec3 size = dvec3(tileset.getTileSize() * terrain_tileset_radius * 2.0);
		size.z = INFINITY;

		engine.visualizer.renderBox(vec3(size), Mat4_identity, terrain_lod_colors[i]);
	}

	return 1;
}

void fill_tilesets(ObjectTerrainGlobal terrain, int lod, int radius) {

	TerrainGlobalLod height_lod = terrain.getHeightLods().getLod(lod);
	TerrainGlobalLod albedo_lod = terrain.getAlbedoLods().getLod(lod);
	TerrainGlobalLod normal_lod = terrain.getNormalLods().getLod(lod);

	Tileset heights = height_lod.getTileset();
	Tileset albedoes = albedo_lod.getTileset();
	Tileset normals = normal_lod.getTileset();

	int tile_resolution = heights.getTileResolution();
	float tile_density = heights.getTileDensity();

	Image tile_heights = new Image();
	tile_heights.create2D(tile_resolution, tile_resolution, IMAGE_FORMAT_R32F);

	Image tile_normals = new Image();

	Image tile_albedoes = new Image();
	tile_albedoes.create2D(tile_resolution, tile_resolution, IMAGE_FORMAT_RGBA8);

	double noise_multiplier = tile_density * iterrain_noise_size;

	forloop(int x = -radius; radius) {
		int tile_x = x * tile_resolution;
		forloop(int y = -radius; radius) {
			int tile_y = y * tile_resolution;

			ivec2 position = ivec2(x, y);

			forloop(int pixel_x = 0; tile_resolution) {
				forloop(int pixel_y = 0; tile_resolution) {
					dvec2 noise_position = dvec2(tile_x + pixel_x, tile_y + pixel_y) * noise_multiplier;
					noise_position.x %= terrain_noise_size;
					noise_position.y %= terrain_noise_size;

					float height = engine.game.getNoise2(vec2(noise_position), vec2(terrain_noise_size), terrain_noise_frequency) * terrain_base_height_scale;

					tile_heights.set2D(pixel_x, pixel_y, vec4(height));
					tile_albedoes.set2D(pixel_x, pixel_y, get_color(height));
				}
			}

			engine.utils.convertHeightsToNormals(tile_normals, tile_heights, tile_density);

			heights.setTileData(position, tile_heights);
			normals.setTileData(position, tile_normals);
			albedoes.setTileData(position, tile_albedoes);
		}
	}

	heights.saveAll();
	normals.saveAll();
	albedoes.saveAll();
}

vec4 get_color(float height) {
	float thresholds[] = (
		-3000.0f,
		-300.0f,
		60.0f,
		600.0f,
		1200.0f,
	);

	vec4 colors[] = (
		vec4(0.0f, 0.0f, 0.1f, 1.0f),
		vec4(0.0f, 0.0f, 1.0f, 1.0f),
		vec4(1.0f, 1.0f, 0.0f, 1.0f),
		vec4(0.0f, 1.0f, 0.0f, 1.0f),
		vec4(1.0f, 1.0f, 1.0f, 1.0f),
	);

	forloop(int i = 1; thresholds.size()) {
		if(height < thresholds[i]) {
			float k = saturate((height - thresholds[i - 1]) / abs(thresholds[i] - thresholds[i - 1]));
			return lerp(colors[i - 1], colors[i], k);
		}
	}

	return vec4(1.0f, 1.0f, 1.0f, 1.0f);
}

```

## text_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Text {
	
	// constructor/destructor
	Text() {
		
		Gui gui = engine.getGui();
		
		WidgetVBox vbox = new WidgetVBox(gui,0,4);
		
		// keyboard
		#ifdef HAS_ACTIVITY
			WidgetButton button = new WidgetButton(gui,"Keyboard");
			button.setToggleable(1);
			button.getEventClicked().connect("Text::button_clicked",button);
			vbox.addChild(button,GUI_ALIGN_EXPAND);
		#endif
		
		// text
		WidgetEditText text = new WidgetEditText(gui,"Editable text...");
		text.getEventClicked().connect("Text::text_clicked",text);
		text.getEventPressed().connect("Text::text_key_pressed");
		text.setWidth(512);
		text.setHeight(256);
		
		vbox.addChild(text,GUI_ALIGN_EXPAND);
		
		vbox.arrange();
		gui.addChild(vbox,GUI_ALIGN_OVERLAP | GUI_ALIGN_CENTER);
	}
	~Text() {
		
	}
	
	// save/restore state
	void __restore__() {
		__Text__();
	}
	
	void text_clicked(WidgetEditText text) {
		engine.console.onscreenMessage("clicked: %s\n",text.getText());
	}
	void text_key_pressed(int key) {
		engine.console.onscreenMessage("pressed: %d\n",key);
	}
	
	#ifdef HAS_ACTIVITY
		void button_clicked(WidgetButton button) {
			engine.activity.setKeyboardShow(button.isToggled());
		}
	#endif
};

Text text;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	text = new Text();
	
	setDescription("WidgetEditText callbacks");
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## texture_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Texture texture;
WidgetSprite sprite;
Camera camera;
float angle = 0.0f;

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("Render to GPU Texture");
	
	int size = 3;
	float space = 16.0f;
	camera = new Camera();
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	return 1;
}

/*
 */
int update() {
	
	int size = 256;
	
	angle += engine.game.getIFps();
	camera.setProjection(perspective(60.0f,1.0f,0.1f,100.0f));
	camera.setModelview(lookAt(Vec3(14.0f),Vec3(0.0f,0.0f,8.0f),vec3(0.0f,0.0f,1.0f)) * rotateZ(angle * 32.0f));
	camera.clearScriptableMaterials();
	camera.addScriptableMaterial(findMaterialByName("Unigine::debug_normals"));

	if(texture == NULL) {
		texture = new Texture();
		sprite = new WidgetSprite(engine.getGui());
		engine.gui.addChild(sprite, GUI_ALIGN_OVERLAP | GUI_ALIGN_RIGHT);
		sprite.setTransform(scale(256.0f / size, 256.0f / size, 1.0f));
	}

	int flip_render = 0;
	if (!engine.render.isFlipped())
		flip_render = 1;

	engine.render.renderTexture2D(camera, texture, size, size, 0, 0);
	sprite.setRender(texture, flip_render);
	
	return 1;
}

```

## thread_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

float spawn = 0.0f;

/*
 */
void mesh_thread(float life,float time) {
	
	ObjectMeshStatic mesh = new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/box.mesh"));
	mesh.setMaterial(findMaterialByName("mesh_base"),"*");
	mesh.setSurfaceProperty("surface_base","*");
	
	while(life > 0.0f) {
		
		life -= engine.game.getIFps();
		time += engine.game.getIFps();
		
		mesh.setWorldTransform(Mat4(rotateX(time * 32.0f) * translate(cos(time * 1.0f) * 10.0f,sin(time * 1.5f) * 10.0f,0.0f) * rotateY(time * 64.0f) * rotateZ(time * 48.0f)));
		
		wait;
	}
	
	delete mesh;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	light_world_0.setShadow(0);
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	light_world_1.setShadow(0);
	
	// create sample
	light_world_0.setScattering(LIGHT_WORLD_SCATTERING_SUN);
	light_world_1.setScattering(LIGHT_WORLD_SCATTERING_SUN);
	
	setDescription("Threads");
	
	return 1;
}

/*
 */
int update() {
	
	spawn += engine.game.getIFps() * 20.0f;
	
	int num_meshes = int(spawn);
	
	for(int i = 0; i < num_meshes; i++) {
		thread("mesh_thread",10.0f,-engine.game.getTime() * 10.0f);
	}
	
	spawn -= num_meshes;

	return 1;
}

```

## time_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

#include <core/systems/widgets/widget.h>
#include <core/systems/widgets/widget_window.h>
#include <core/systems/widgets/widget_icon.h>
#include <core/systems/widgets/widget_time.h>

/*
 */
Unigine::Widgets::Window window;
Unigine::Widgets::Icon close_icon;

Unigine::Widgets::Time times[0];

using Unigine::Samples;

void on_close()
{
	Unigine::Widgets::removeChild(window);
}

/*
 */
int update() {
	
	using Unigine::Widgets;
	
	foreach(Time time; times)
		time.update();
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.setShadowDistance(100.0f);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("Unigine::Widgets::Time");
	
	using Unigine::Widgets;
	
	// window
	window = new Window("Unigine::Widgets::Time");
	window.setWidth(768);
	window.setSizeable(1);
	window.setDragAreaPadding(3, 25, 3, 0);
	
	// close icon
	close_icon = new Icon();
	close_icon.setPosition(10, -24);
	close_icon.getEventClicked().connect(functionid(on_close));
	window.addChild(close_icon, GUI_ALIGN_OVERLAP | GUI_ALIGN_TOP | GUI_ALIGN_RIGHT);

	Image img = new Image("core/gui/window_close.png");
	close_icon.setImage(img);

	float min_keys[] = ( 0.0f, 0.0f, 0.0f, 0.0f );
	float max_keys[] = ( 1.0f, 10.0f, 100.0f, 1000.0f );
	
	forloop(int i = 0; 4) {
		Time time = new Time(min_keys[i],max_keys[i]);
		window.addChild(time,ALIGN_EXPAND);
		times.append(time);
	}
	
	// window
	window.arrange();
	addChild(window,ALIGN_OVERLAP | ALIGN_CENTER);
	
	return 1;
}

```

## tooltip_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

string text;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	text = "Jack and Jill<br>"
		"Went up the hill "
		"To fetch a pail of water. "
		"Jack fell down "
		"And broke his crown "
		"And Jill came tumbling after. "
		"Up Jack got "
		"And home did trot "
		"As fast as he could caper "
		"Went to bed "
		"And plastered his head "
		"With vinegar and brown paper.";
	
	text = replace(text,"Jack","<font size=%110 color=#ff0000><b>Jack</b></font>");
	text = replace(text,"Jill","<font size=%110 color=#00ff00><bi>Jill</bi></font>");
	
	setDescription("ToolTips");
	
	return 1;
}

/*
 */
int update() {
	
	Gui gui = engine.getGui();
	
	float time = engine.game.getTime();
	gui.setToolTipWidth(64 + 256 + sin(time * 2.0f) * 256);
	
	EngineWindow main_window = engine.window_manager.getMainWindow();
	ivec2 mouse_coord = engine.input.getMousePosition() - main_window.getPosition();
	
	gui.setToolTip(mouse_coord.x,mouse_coord.y,text);
	
	return 1;
}

```

## track_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

#include <core/systems/widgets/widget.h>
#include <core/systems/widgets/widget_window.h>
#include <core/systems/widgets/widget_icon.h>
#include <core/systems/widgets/widget_track.h>

/*
 */
Unigine::Widgets::Window window;
Unigine::Widgets::Icon close_icon;

Unigine::Widgets::Track track;

using Unigine::Samples;

void on_close()
{
	Unigine::Widgets::removeChild(window);
}

/*
 */
int update() {
	
	track.update();
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.setShadowDistance(100.0f);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("Unigine::Widgets::Track");
	
	using Unigine::Widgets;
	
	// window
	window = new Window("Unigine::Widgets::Track");
	window.setWidth(min(getWidth() - 128,1280));
	window.setHeight(min(getHeight() - 128,768));
	window.setSizeable(1);
	window.setDragAreaPadding(3, 25, 3, 0);
	
	// close icon
	close_icon = new Icon();
	close_icon.setPosition(10, -24);
	close_icon.getEventClicked().connect(functionid(on_close));
	window.addChild(close_icon, GUI_ALIGN_OVERLAP | GUI_ALIGN_TOP | GUI_ALIGN_RIGHT);

	Image img = new Image("core/gui/window_close.png");
	close_icon.setImage(img);

	// values
	int values[];
	values.clear();
	int num_keys = 32;
	float min_key = 0.0f;
	float max_key = 100.0f;
	forloop(int i = 1; num_keys) {
		values.append(min_key + (max_key - min_key) * i / num_keys,rand(0.0f,1.0f));
	}
	
	// curves
	float scales[] = ( 1.0f, 0.75f, 0.5f, 0.25f );
	vec4 colors[] = ( vec4(1.0f,1.0f,1.0f,1.0f), vec4(1.0f,0.0f,0.0f,1.0f), vec4(0.0f,1.0f,0.0f,1.0f), vec4(0.0f,0.0f,1.0f,1.0f) );
	
	// track
	track = new Track(min_key,max_key);
	forloop(int i = 0; colors.size()) {
		TrackCurve curve = new TrackCurve(colors[i]);
		foreachkey(float key; values) {
			curve.addKey(new TrackKey(key,values[key] * scales[i]));
		}
		forloop(int j = 0; curve.getNumKeys()) {
			curve.setKeyType(curve.getKey(j),TRACK_KEY_SMOOTH);
		}
		track.addCurve(curve);
	}
	window.addChild(track,ALIGN_EXPAND);
	
	// window
	window.arrange();
	track.arrangeXY();
	addChild(window,ALIGN_OVERLAP | ALIGN_CENTER);
	
	return 1;
}

```

## track_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

#include <core/systems/widgets/widget.h>
#include <core/systems/widgets/widget_window.h>
#include <core/systems/widgets/widget_icon.h>
#include <core/systems/widgets/widget_track.h>
#include <core/systems/widgets/widget_line_track.h>

/*
 */
Unigine::Widgets::Window window;
Unigine::Widgets::Icon close_icon;

Unigine::Widgets::Track track;
Unigine::Widgets::Line line;

using Unigine::Samples;

void on_close()
{
	Unigine::Widgets::removeChild(window);
}

/*
 */
int update() {
	
	using Unigine::Widgets;
	
	// synchronize scrolls
	ScrollBox line_sb = line.getScrollBox();
	ScrollBox track_sb = track.getScrollBox();
	if(line_sb.widget != NULL && track_sb.widget != NULL) {
		if(track.isCanvasFocused()) {
			line.setCanvasWidth(track.getCanvasWidth());
			line_sb.arrange();
			line_sb.setHScrollValue(track_sb.getHScrollValue());
		} else if(line.isCanvasFocused()) {
			track.setCanvasWidth(line.getCanvasWidth());
			track_sb.arrange();
			track_sb.setHScrollValue(line_sb.getHScrollValue());
		} else {
			track_sb.setHScrollValue(line_sb.getHScrollValue());
		}
	}
	
	line.setKeySnap(track.getKeySnap());
	
	track.update();
	line.update();
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.setShadowDistance(100.0f);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("Unigine::Widgets::Track");
	
	using Unigine::Widgets;
	
	// window
	window = new Window("Unigine::Widgets::Track");
	window.setWidth(min(getWidth() - 128,1280));
	window.setHeight(min(getHeight() - 128,768));
	window.setSizeable(1);
	window.setDragAreaPadding(3, 25, 3, 0);
	
	// close icon
	close_icon = new Icon();
	close_icon.setPosition(10, -24);
	close_icon.getEventClicked().connect(functionid(on_close));
	window.addChild(close_icon, GUI_ALIGN_OVERLAP | GUI_ALIGN_TOP | GUI_ALIGN_RIGHT);

	Image img = new Image("core/gui/window_close.png");
	close_icon.setImage(img);

	// values
	int values[];
	values.clear();
	int num_keys = 32;
	float min_key = 0.0f;
	float max_key = 100.0f;
	forloop(int i = 1; num_keys) {
		values.append(min_key + (max_key - min_key) * i / num_keys,rand(0.0f,1.0f));
	}
	
	// curves
	float scales[] = ( 1.0f, 0.75f, 0.5f, 0.25f );
	vec4 colors[] = ( vec4(1.0f,1.0f,1.0f,1.0f), vec4(1.0f,0.0f,0.0f,1.0f), vec4(0.0f,1.0f,0.0f,1.0f), vec4(0.0f,0.0f,1.0f,1.0f) );
	
	// track
	track = new Track(min_key,max_key);
	ScrollBox scrollbox = track.getScrollBox();
	scrollbox.setHScrollHidden(2);
	window.addChild(track,ALIGN_EXPAND);
	
	// line
	VBox vbox = new VBox();
	line = new Line(min_key,max_key);
	scrollbox = line.getScrollBox();
	scrollbox.setVScrollEnabled(0);
	vbox.addChild(line,ALIGN_EXPAND);
	window.addChild(vbox);
	
	// track callbacks
	track.setCallback(TRACK_KEY_CREATED,"Unigine::Widgets::LineTrack::trackKeyCreated",line,track);
	track.setCallback(TRACK_KEY_CHANGED,"Unigine::Widgets::LineTrack::trackKeyChanged",line,track);
	track.setCallback(TRACK_KEY_REMOVED,"Unigine::Widgets::LineTrack::trackKeyRemoved",line,track);
	
	// line callbacks
	line.setCallback(LINE_KEY_CREATED,"Unigine::Widgets::LineTrack::lineKeyCreated",line,track);
	line.setCallback(LINE_KEY_CHANGED,"Unigine::Widgets::LineTrack::lineKeyChanged",line,track);
	line.setCallback(LINE_KEY_REMOVED,"Unigine::Widgets::LineTrack::lineKeyRemoved",line,track);
	
	// range from
	float range_from = 20.0f;
	track.setRangeFrom(range_from);
	line.setRangeFrom(range_from);
	
	// range to
	float range_to = 80.0f;
	track.setRangeTo(range_to);
	line.setRangeTo(range_to);
	
	// keys
	forloop(int i = 0; colors.size()) {
		
		TrackCurve track_curve = new TrackCurve(colors[i]);
		LineCurve line_curve = new LineCurveTrack(track_curve);
		track_curve.setData(line_curve);
		
		track.addCurve(track_curve);
		line.addCurve(line_curve);
		
		foreachkey(float key; values) {
			
			TrackKey track_key = new TrackKey(key,values[key] * scales[i]);
			LineKeyTrack line_key = new LineKeyTrack(key,track_key);
			track_key.setData(line_key);
			
			track_curve.addKey(track_key);
			line_curve.addKey(line_key);
		}
		
		forloop(int j = 0; track_curve.getNumKeys()) {
			track_curve.setKeyType(track_curve.getKey(j),TRACK_KEY_SMOOTH);
		}
	}
	
	// update
	track.update();
	line.update();
	
	// window
	window.arrange();
	track.arrangeXY();
	addChild(window,ALIGN_OVERLAP | ALIGN_CENTER);
	
	return 1;
}

```

## tracker_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

#include <core/systems/tracker/tracker.h>
#include <core/systems/tracker/editor/tracker_editor.h>

#include <core/systems/widgets/widget.h>
#include <core/systems/widgets/widget_hbox.h>
#include <core/systems/widgets/widget_groupbox.h>
#include <core/systems/widgets/widget_window.h>
#include <core/systems/widgets/widget_button.h>
#include <core/systems/widgets/widget_slider.h>

using Unigine::Samples;
using Unigine::Tracker;

/*
 */
Tracker tracker;
TrackerEditor tracker_editor;

Unigine::Widgets::Window tracker_window;
Unigine::Widgets::Slider slider;

/*
 */
string mesh_material_names[] = ( "tracker_mesh_red", "tracker_mesh_green", "tracker_mesh_blue", "tracker_mesh_orange", "tracker_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
void editor_clicked(Unigine::Widgets::Button button) {
	
	using Unigine::Widgets;
	
	tracker_window.arrange();
	tracker_editor.arrange();
	
	// handle window
	if(button.isToggled()) addChild(tracker_window,ALIGN_OVERLAP | ALIGN_CENTER);
	else removeChild(tracker_window);
}

/*
 */
void clear_clicked() {
	
	// clear editor
	tracker_editor.clear();
}

/*
 */
void load_clicked() {
	
	using Unigine::Widgets;
	
	// dialog file
	string name = pathname(fullPath("uniginescript_samples/tracker/tracks/nodes_00.track"));
	name += "/";
	
	if(dialogFile("Select file",".track",name) == 0) return;
	
	// load track
	if(tracker_editor.loadTrack(name) == 0) dialogMessage("Error","Can't load track\n" + engine.console.getLastMessage());
}

void save_clicked() {
	
	using Unigine::Widgets;
	
	// dialog file
	string name = pathname(fullPath("uniginescript_samples/tracker/tracks/nodes_00.track"));
	name += "/";
	
	if(dialogFile("Select file",".track",name) == 0) return;
	
	// save track
	if(tracker_editor.saveTrack(name) == 0) dialogMessage("Error","Can't save track\n" + engine.console.getLastMessage());
}

/*
 */
void create_mesh_scene() {
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setName(format("statue_%d_%d",size + x,size + y));
		}
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create sample
	create_mesh_scene();
	
	// create tracker
	tracker = new Tracker(TRACKER_SAVE_RESTORE | TRACKER_CHECK_OBJECTS);
	
	using Unigine::Widgets;
	
	// window
	tracker_window = new Window("Unigine::Tracker::TrackerEditor");
	tracker_window.setWidth(getWidth() - 256);
	tracker_window.setHeight(getHeight() - 256);
	tracker_window.setSizeable(1);
	
	// create tracker editor
	tracker_editor = new TrackerEditor(tracker);
	tracker_window.addChild(tracker_editor.getWidget(),ALIGN_EXPAND);
	
	// panel
	GroupBox panel = new GroupBox("",4,4);
	panel.setBackground(1);
	
	// slider
	slider = new Slider(0,1000,0);
	panel.addChild(slider,ALIGN_EXPAND);
	
	// hbox
	HBox hbox = new HBox(4,4);
	panel.addChild(hbox,ALIGN_RIGHT);
	
	// editor button
	Button button = new Button("Editor");
	button.setToggleable(1);
	button.getEventClicked().connect("editor_clicked",button);
	hbox.addChild(button);
	
	// clear button
	button = new Button("Clear");
	button.getEventClicked().connect("clear_clicked");
	hbox.addChild(button);
	
	// load button
	button = new Button("Load");
	button.getEventClicked().connect("load_clicked");
	hbox.addChild(button);
	
	// save button
	button = new Button("Save");
	button.getEventClicked().connect("save_clicked");
	hbox.addChild(button);
	
	// add panel
	addChild(panel,ALIGN_OVERLAP | ALIGN_RIGHT | ALIGN_TOP);
	
	// load track
	tracker_editor.loadTrack(fullPath("uniginescript_samples/tracker/tracks/nodes_00.track"));
	tracker_editor.setState(EDITOR_STATE_LOOP);
	
	setDescription("Unigine::Tracker::TrackerEditor");
	
	return 1;
}

/*
 */
int update() {
	
	tracker_editor.update();
	
	float min_time = tracker_editor.getMinTime();
	float max_time = tracker_editor.getMaxTime();
	float current_time = tracker_editor.getTime();
	
	if(slider.isFocused() && engine.gui.getMouseGrab()) tracker_editor.setTime(min_time + slider.getValue() * (max_time - min_time) / 1000.0f);
	else slider.setValue(floor(1000.0f * (current_time - min_time) / (max_time - min_time)));
	
	return 1;
}

```

## tracks_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>
#include <core/systems/tracker/tracker.h>

using Unigine::Samples;
using Unigine::Tracker;

/*
 */
string mesh_material_names[] = ( "tracker_mesh_red", "tracker_mesh_green", "tracker_mesh_blue", "tracker_mesh_orange", "tracker_mesh_yellow" );

string get_mesh_material(int material) {
	return mesh_material_names[abs(material) % mesh_material_names.size()];
}

/*
 */
void update_track(Unigine::Tracker::TrackerTrack track) {
	
	if(track == NULL) return;
	
	float time = track.getMinTime();
	float min_time = track.getMinTime();
	float max_time = track.getMaxTime();
	float unit_time = track.getUnitTime();
	
	while(1) {
		
		// update time
		time += engine.game.getIFps() / unit_time;
		time = min_time + ((time - min_time)) % (max_time - min_time);
		
		// set track
		if(engine.game.isEnabled()) track.set(time);
		
		wait;
	}
}

/*
 */
void create_mesh_scene() {
	
	int size = 2;
	float space = 8.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
			mesh.setName(format("statue_%d_%d",size + x,size + y));
		}
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// create sample
	create_mesh_scene();
	
	Tracker tracker = new Tracker(TRACKER_CHECK_OBJECTS);
	TrackerTrack track = tracker.loadTrack(fullPath("uniginescript_samples/tracker/tracks/tracks_00.track"));
	
	thread("update_track",track);
	
	setDescription("Tracker Track parameters");
	
	return 1;
}

```

## tracks_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
class Track {
	
	Body b0,b1;
	
	JointHinge rollers[0];
	
	Track(Mat4 transform) {
		
		void create_joint(Body b,Vec3 position) {
			JointHinge j = class_remove(new JointHinge(b0,b,transform * position,rotation(transform) * vec3(1.0f,0.0f,0.0f)));
			j.setAngularSpring(100.0f);
			j.setLinearRestitution(0.4f);
			j.setAngularRestitution(0.1f);
			j.setNumIterations(8);
			rollers.append(j);
		}
		
		createTracks(vec3(2.0f,0.9f,0.3f),2.0f,1.0f,0.5f,1.0f,4,get_material(0),transform);
		
		b0 = createBodyBox(vec3(0.25f,10.0f,1.5f),2.0f,0.5f,0.5f,get_material(1),transform * translate( 1.2f,0.0f,0.35f));
		b1 = createBodyBox(vec3(0.25f,10.0f,1.5f),2.0f,0.5f,0.5f,get_material(1),transform * translate(-1.2f,0.0f,0.35f));
		
		JointFixed j0 = class_remove(new JointFixed(b0,b1));
		j0.setNumIterations(8);
		j0.setLinearRestitution(0.8f);
		j0.setAngularRestitution(0.2f);
		
		Body b2 = createBodyPrism(0.9f,0.9f,2.0f,6,2.0f,1.0f,0.5f,get_material(3),transform * translate(0.0f, 4.0f,0.0f) * rotateY(90.0f));
		Body b3 = createBodyPrism(0.9f,0.9f,2.0f,6,2.0f,1.0f,0.5f,get_material(3),transform * translate(0.0f,-4.0f,0.0f) * rotateY(90.0f));
		Body b4 = createBodyPrism(0.9f,0.9f,2.0f,6,2.0f,1.0f,0.5f,get_material(3),transform * translate(0.0f, 0.0f,0.0f) * rotateY(90.0f));
		
		create_joint(b2,Vec3(0.0f, 4.0f,0.0f));
		create_joint(b3,Vec3(0.0f,-4.0f,0.0f));
		create_joint(b4,Vec3(0.0f, 0.0f,0.0f));
	}
	
	float angle = 0.0f;
	float velocity = 0.0f;
	
	void setVelocity(float v) {
		velocity = v;
	}
	
	void updatePhysics() {
		angle += velocity;
		if(angle > 180.0f) angle -= 360.0f;
		if(angle < -180.0f) angle += 360.0f;
		foreach(JointHinge j; rollers) j.setAngularAngle(angle);
	}
};

/*
 */
class Robot {
	
	Object object;
	Track tracks[2];
	
	float angle = 0.0f;
	float velocity = 0.0f;
	
	Robot(Mat4 transform) {
		
		Body body = createBodyBox(vec3(4.0f,8.0f,2.0f),0.5f,0.5f,0.5f,get_material(0),transform);
		object = body.getObject();
		
		tracks[0] = new Track(transform * translate( 3.5f,0.0f,-1.5f));
		tracks[1] = new Track(transform * translate(-3.5f,0.0f,-1.5f));
		
		JointFixed j0 = class_remove(new JointFixed(body,tracks[0].b1));
		JointFixed j1 = class_remove(new JointFixed(body,tracks[1].b0));
		
		j0.setLinearRestitution(0.8f);
		j0.setAngularRestitution(0.2f);
		j0.setNumIterations(8);
		
		j1.setLinearRestitution(0.8f);
		j1.setAngularRestitution(0.2f);
		j1.setNumIterations(8);
	}
	
	void update() {
		
		float ifps = engine.game.getIFps();
		
		if(engine.controls.getState(CONTROLS_STATE_FORWARD) || engine.controls.getState(CONTROLS_STATE_TURN_UP)) {
			velocity = max(velocity,0.0f);
			velocity += ifps * 8.0f;
		} else if(engine.controls.getState(CONTROLS_STATE_BACKWARD) || engine.controls.getState(CONTROLS_STATE_TURN_DOWN)) {
			velocity = min(velocity,0.0f);
			velocity -= ifps * 8.0f;
		} else {
			velocity *= exp(-ifps);
		}
		velocity = clamp(velocity,-8.0f,8.0f);
		
		if(engine.controls.getState(CONTROLS_STATE_MOVE_LEFT) || engine.controls.getState(CONTROLS_STATE_TURN_LEFT)) {
			angle += ifps * 8.0f;
		} else if(engine.controls.getState(CONTROLS_STATE_MOVE_RIGHT) || engine.controls.getState(CONTROLS_STATE_TURN_RIGHT)) {
			angle -= ifps * 8.0f;
		} else {
			angle *= exp(-ifps);
		}
		angle = clamp(angle,-4.0f,4.0f);
		
		tracks[0].setVelocity(velocity + angle);
		tracks[1].setVelocity(velocity - angle);
	}
	
	void updatePhysics() {
		foreach(Track track; tracks) track.updatePhysics();
	}
};

/*
 */
Robot robot = NULL;
ControlsDummy controls;

/*
 */
int update() {
	updateSamplePhysics();
	robot.update();
	controls.setMouseDX(engine.controls.getMouseDX());
	controls.setMouseDY(engine.controls.getMouseDY());
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Tracks");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	robot = new Robot(translate(Vec3(0.0f,0.0f,2.5f)) * rotateZ(90.0f));
	controls = new ControlsDummy();
	
	for(int i = 0; i < 16; i++) {
		createBodyHomuncle(1,vec3_zero,material_names,translate(Vec3(-10.0f - i * 3.0f,-3.5f,0.1f)));
		createBodyHomuncle(1,vec3_zero,material_names,translate(Vec3(-10.0f - i * 3.0f, 3.5f,0.1f)));
	}
	
	PlayerPersecutor player = new PlayerPersecutor();
	player.setFixed(1);
	player.setTarget(robot.object);
	player.setMinDistance(16.0f);
	player.setMaxDistance(20.0f);
	player.setPosition(Vec3(10.0f,0.0f,6.0f));
	player.setControls(controls);
	engine.game.setPlayer(player);
	
	return 1;
}

/*
 */
int updatePhysics() {
	
	robot.updatePhysics();
	
	return 1;
}

```

## train_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
BodyPath path;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
class Wagon {
	
	Object object;
	BodyRigid body;
	
	Wagon(Mat4 transform) {
		
		body = createBodyBox(vec3(3.0f,12.0f,3.0f),1.0f,1.0f,0.0f,get_material(0),transform * translate(0.0f,0.0f,2.5f));
		object = body.getObject();
		
		BodyRigid wheels_0 = createBodyBox(vec3(3.0f,1.0f,1.0f),64.0f,1.0f,0.0f,get_material(1),transform * translate(0.0f, 4.0f,0.5f));
		BodyRigid wheels_1 = createBodyBox(vec3(3.0f,1.0f,1.0f),64.0f,1.0f,0.0f,get_material(1),transform * translate(0.0f,-4.0f,0.5f));
		
		void create_hinge(BodyRigid wheels) {
			JointHinge joint = class_remove(new JointHinge(body,wheels,wheels.getTransform() * Vec3_zero,vec3(0.0f,0.0f,1.0f)));
			joint.setLinearRestitution(0.5f);
			joint.setAngularRestitution(0.5f);
			joint.setLinearSoftness(0.25f);
			joint.setAngularSoftness(0.25f);
			joint.setNumIterations(4);
		}
		
		create_hinge(wheels_0);
		create_hinge(wheels_1);
		
		void create_path(BodyRigid wheels) {
			JointPath joint = class_remove(new JointPath(wheels,path));
			joint.setLinearVelocity(-100.0f);
			joint.setLinearForce(1000.0f);
			joint.setRotation0(rotateZ(90.0f));
			joint.setAngularRestitution(0.75f);
			joint.setNumIterations(4);
		}
		
		create_path(wheels_0);
		create_path(wheels_1);
	}
};

/*
 */
int update() {
	updateSamplePhysics();
	
	path.renderVisualizer();
	
	return 1;
}

/*
 */
int init() {
	
	engine.visualizer.setEnabled(1);
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	ObjectDummy dummy = addToEditor(new ObjectDummy());
	dummy.setWorldTransform(Mat4(translate(0.0f,0.0f,2.0f)));
	path = class_remove(new BodyPath(dummy));
	path.setPathName(full_path("uniginescript_samples/physics/paths/train_00.path"));
	
	Wagon wagons[0];
	forloop(int i = 0; 30) {
		wagons.append(new Wagon(Mat4(translate(0.0f,i * 13.0f - 270.0f,1.5f))));
	}
	
	forloop(int i = 1; wagons.size()) {
		void create_ball(BodyRigid body_0,BodyRigid body_1) {
			JointBall joint = class_remove(new JointBall(body_0,body_1));
			joint.setLinearSoftness(0.1f);
			joint.setNumIterations(2);
		}
		create_ball(wagons[i - 1].body,wagons[i].body);
	}
	
	Wagon wagon = wagons[wagons.size() - 1];
	PlayerPersecutor player = new PlayerPersecutor();
	player.setFixed(1);
	player.setTarget(wagon.object);
	player.setMinDistance(8.0f);
	player.setMaxDistance(12.0f);
	player.setPosition(wagon.object.getWorldPosition() + Vec3(0.0f,16.0f,8.0f));
	engine.game.setPlayer(player);
	
	setDescription(format("Train with %d wagons",wagons.size()));
	return 1;
}

```

## transform_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	setDescription("Gui transformation");
	
	return 1;
}

/*
 */
int update() {
	
	EngineWindow main_window = engine.window_manager.getMainWindow();
	ivec2 window_size = main_window.getSize();
	ivec2 mouse_coord = engine.input.getMousePosition() - main_window.getPosition();
	
	float fov = 2.0f;
	float hwidth = window_size.x / 2.0f;
	float hheight = window_size.y / 2.0f;
	
	float y_angle = (float(mouse_coord.x) - hwidth) / window_size.x;
	float x_angle = (float(mouse_coord.y) - hheight) / window_size.y;
	
	engine.gui.setTransform(translate(hwidth,hheight,0.0f) * perspective(fov,1.0f,0.01f,100.0f) * rotateY(y_angle) * rotateX(-x_angle) * translate(-hwidth,-hheight,-1.0f / tan(fov * DEG2RAD * 0.5f)));
	
	return 1;
}

/*
 */
int shutdown() {
	
	engine.gui.setTransform(mat4_identity);
	
	return 1;
}

```

## transform_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

float x = 0.0f;
float y = 0.0f;

int dx = 64;
int dy = 64;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	setDescription("Gui transformation");
	
	return 1;
}

/*
 */
int update() {
	
	float scale = 15.0f / 16.0f;
	
	EngineWindow main_window = engine.window_manager.getMainWindow();
	ivec2 window_size = main_window.getSize();
	
	float swidth = window_size.x * scale;
	float sheight = window_size.y * scale;
	
	if(x < 0 || x + swidth >= window_size.x) dx = -dx;
	if(y < 0 || y + sheight >= window_size.y) dy = -dy;
	
	x += dx * engine.game.getIFps();
	y += dy * engine.game.getIFps();
	
	engine.gui.setTransform(translate(x,y,0.0f) * ::scale(scale,scale,1.0f));
	
	return 1;
}

/*
 */
int shutdown() {
	
	engine.gui.setTransform(mat4_identity);
	
	return 1;
}

```

## trigger_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
PhysicalTrigger trigger;
float step = 1.0f;
float time = step;
int counter = 0;

using Unigine::Samples;

/*
 */
string material_names[] = ( "physicals_red", "physicals_green", "physicals_blue", "physicals_orange", "physicals_yellow" );

/*
 */
void enter_callback(Body body) {
	Object object = body.getObject();
	if(object != NULL) object.setMaterialState("emission",1,0);
}

void leave_callback(Body body) {
	Object object = body.getObject();
	if(object != NULL) object.setMaterialState("emission",0,0);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	trigger.renderVisualizer();
	
	time += engine.game.getIFps() * engine.physics.getScale();
	
	if(time >= step) {
		if(counter++ < 10) createBodyHomuncle(0,vec3(0.0f),material_names,translate(Vec3(0.0f,0.0f,25.0f)) * rotateX(180.0f));
		time -= step;
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("PhysicalTrigger CCD");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	trigger = addToEditor(new PhysicalTrigger(SHAPE_BOX,vec3(20.0f,20.0f,4.0f)));
	trigger.setWorldTransform(translate(Vec3(0.0f,0.0f,12.0f)));
	trigger.getEventEnter().connect(functionid(enter_callback));
	trigger.getEventLeave().connect(functionid(leave_callback));
	
	engine.visualizer.setEnabled(1);
	
	return 1;
}

```

## trigger_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physicals_red", "physicals_green", "physicals_blue", "physicals_orange", "physicals_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
class Robot {
	
	Object object;
	BodyRigid body;
	PhysicalTrigger trigger;
	
	Robot(Mat4 transform) {
		
		body = createBodyBox(vec3_one,10.0f,0.5f,0.5f,get_material(1),transform);
		object = body.getObject();
		
		trigger = addToEditor(new PhysicalTrigger(SHAPE_BOX,vec3(0.8f,1.5f,0.5f)));
		trigger.setWorldTransform(translate(Vec3(1.0f,0.0f,0.0f)));
		object.addChild(trigger);
	}
	
	void update() {
		updateSamplePhysics();
		
		trigger.renderVisualizer();
	}
	
	void updatePhysics() {
		
		Mat4 transform = object.getWorldTransform();
		Mat4 itransform = object.getIWorldTransform();
		
		float rotate = 0.0f;
		forloop(int i = 0; trigger.getNumContacts()) {
			Vec3 p = itransform * trigger.getContactPoint(i);
			rotate -= sign(p.y);
		}
		
		float force = 1.0f;
		if(rotate == 0.0f && trigger.getNumContacts()) {
			force = -force;
		}
		
		body.addForce(rotation(transform) * vec3(force * 100.0f,0.0f,0.0f));
		body.addTorque(vec3(0.0f,0.0f,rotate * 20.0f));
	}
};

Robot robots[0];

/*
 */
int update() {
	updateSamplePhysics();
	
	foreach(Robot robot; robots) robot.update();
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("PhysicalTrigger");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	createBodyBox(vec3(1.0f,32.0f,1.0f),10.0f,0.5f,0.5f,get_material(0),translate(Vec3( 16.0f,  0.5f,0.5f)));
	createBodyBox(vec3(1.0f,32.0f,1.0f),10.0f,0.5f,0.5f,get_material(0),translate(Vec3(-16.0f, -0.5f,0.5f)));
	createBodyBox(vec3(32.0f,1.0f,1.0f),10.0f,0.5f,0.5f,get_material(0),translate(Vec3( -0.5f, 16.0f,0.5f)));
	createBodyBox(vec3(32.0f,1.0f,1.0f),10.0f,0.5f,0.5f,get_material(0),translate(Vec3(  0.5f,-16.0f,0.5f)));
	
	int num = 28;
	float offset = 360.0f / num;
	
	forloop(int i = 0; num) {
		
		Mat4 transform = Mat4(rotateZ(offset * i + offset * 0.5f) * translate(14.0f,0.0f,0.5f) * rotateZ(180.0f));
		
		robots.append(new Robot(transform));
	}
	
	engine.visualizer.setEnabled(1);
	
	return 1;
}

/*
 */
int updatePhysics() {
	
	foreach(Robot robot; robots) robot.updatePhysics();
	
	return 1;
}

```

## trigger_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
PhysicalTrigger trigger;
float step = 1.0f;
float time = step;

using Unigine::Samples;

/*
 */
string material_names[] = ( "physicals_red", "physicals_green", "physicals_blue", "physicals_orange", "physicals_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
void enter_callback(Body body) {
	Object object = body.getObject();
	if(object != NULL) engine.world.removeNode(object);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	trigger.renderVisualizer();
	
	time += engine.game.getIFps() * engine.physics.getScale();
	
	if(time >= step) {
		BodyRigid body = createBodySphere(0.5f,1.0f,1.0f,0.5f,get_material(0),translate(Vec3(15.0f,0.0f,10.0f)));
		body.setLinearVelocity(vec3(-80.0f,0.0f,0.0f));
		time -= step;
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("PhysicalTrigger CCD");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	engine.physics.setMaxLinearVelocity(200.0f);
	
	trigger = addToEditor(new PhysicalTrigger(SHAPE_BOX,vec3(0.01f,40.0f,20.0f)));
	trigger.setWorldTransform(translate(Vec3(-10.0f,0.0f,10.0f)));
	trigger.getEventEnter().connect(functionid(enter_callback));
	
	engine.visualizer.setEnabled(1);
	
	return 1;
}

/*
 */
int updatePhysics() {
	
	trigger.updateContacts();
	
	return 1;
}

```

## ui_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Window {
	
	string name;			// window name
	
	Window instance;		// current instance
	WidgetWindow window;	// window widget
	
	// constructor/destructor
	Window(string str) {
		name = str;
		instance = this;
		new UserInterface(engine.getGui(),fullPath("uniginescript_samples/widgets/ui_00.ui"));
		window.setText(name);
	}
	
	// show the window
	void show(int x,int y) {
		Gui gui = engine.getGui();
		gui.addChild(window,GUI_ALIGN_OVERLAP);
		window.setPosition(x,y);
	}
	
	// save/restore state
	void __save__(Stream stream) {
		stream.writeInt(window.getPositionX());
		stream.writeInt(window.getPositionY());
	}
	void __restore__(Stream stream) {
		__Window__(name);
		int x = stream.readInt();
		int y = stream.readInt();
		show(x,y);
	}
	
	// clicked callback
	void clicked() {
		engine.console.onscreenMessage("%s clicked\n",name);
		engine.dialogMessage(name, name + " clicked");
	}
	
	// callback redirector
	void callback_redirector(string func,Window window) {
		window.call(func);
	}
};

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	Window window_0 = new Window("First window");
	Window window_1 = new Window("Second window");
	Window window_2 = new Window("Third window");
	Window window_3 = new Window("Fourth window");
	
	// show windows
	window_0.show(200,200);
	window_1.show(400,200);
	window_2.show(200,400);
	window_3.show(400,400);
	
	setDescription("User interfaces with User classes");
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## ui_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
template declareCallback<WIDGET,VALUE> {
	
	// declare callbacks
	void WIDGET ## _pressed() {
		log.message("%s pressed %s\n",#WIDGET,WIDGET.getText());
	}
	void WIDGET ## _focus_out() {
		log.message("%s focus out %s\n",#WIDGET,WIDGET.getText());
	}
	
	// set callbacks
	init_id.append([]() {
		WIDGET.getEventPressed().connect(functionid(callback_redirector),functionid(WIDGET ## _pressed),instance);
		WIDGET.getEventFocusOut().connect(functionid(callback_redirector),functionid(WIDGET ## _focus_out),instance);
	});
	
	// update text
	update_id.append([]() {
		WIDGET.setText(string(VALUE));
	});
}

/*
 */
class Window {
	
	Window instance;			// instance
	
	int init_id[0];				// init functions
	int update_id[0];			// update functions
	
	string name;				// window name
	
	WidgetWindow window;		// window
	
	WidgetEditLine editline_0;	// editlines
	WidgetEditLine editline_1;
	WidgetEditLine editline_2;
	WidgetEditLine editline_3;
	
	// constructor/destructor
	Window(string str) {
		
		name = str;
		instance = this;
		new UserInterface(engine.getGui(),fullPath("uniginescript_samples/widgets/ui_01.ui"),"Window::");
		window.setText(name);
		
		init_id.call();
		update_id.call();
	}
	
	// show the window
	void show(int x,int y) {
		Gui gui = engine.getGui();
		gui.addChild(window,GUI_ALIGN_OVERLAP);
		window.setPosition(x,y);
	}
	
	// save/restore state
	void __save__(Stream stream) {
		stream.writeInt(window.getPositionX());
		stream.writeInt(window.getPositionY());
	}
	void __restore__(Stream stream) {
		__Window__(name);
		int x = stream.readInt();
		int y = stream.readInt();
		show(x,y);
	}
	
	// editline callbacks
	declareCallback<editline_0,0>;
	declareCallback<editline_1,1>;
	declareCallback<editline_2,2>;
	declareCallback<editline_3,3>;
	
	// callback redirector
	void callback_redirector(int func,Window window) {
		window.call(func);
	}
};

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	Window window_0 = new Window("First window");
	Window window_1 = new Window("Second window");
	Window window_2 = new Window("Third window");
	Window window_3 = new Window("Fourth window");
	
	// show windows
	window_0.show(200,200);
	window_1.show(400,200);
	window_2.show(200,400);
	window_3.show(400,400);
	
	setDescription("User interfaces with User classes");
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## ui_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Window {
	
	Window instance;		// current instance
	WidgetVBox vbox;		// window vbox
	WidgetSprite sprite;	// window sprite
	WidgetSprite button;	// window button
	
	// constructor
	Window(vec4 color) {
		instance = this;
		new UserInterface(engine.getGui(),fullPath("uniginescript_samples/widgets/ui_02.ui"));
		sprite.setColor(color);
	}
	
	// show the window
	void show(int x,int y) {
		Gui gui = engine.getGui();
		gui.addChild(vbox,GUI_ALIGN_OVERLAP);
		vbox.setPosition(x,y);
	}
	
	// save/restore state
	void __save__(Stream stream) {
		stream.writeVec4(sprite.getColor());
		stream.writeInt(vbox.getPositionX());
		stream.writeInt(vbox.getPositionY());
	}
	void __restore__(Stream stream) {
		__Window__(stream.readVec4());
		int x = stream.readInt();
		int y = stream.readInt();
		show(x,y);
	}
	
	// pressed callback
	void pressed() {
		Gui gui = engine.getGui();
		int x = vbox.getPositionX();
		int y = vbox.getPositionY();
		vbox.setPosition(x + gui.getMouseDX(),y + gui.getMouseDY());
	}
	
	// clicked callback
	void clicked() {
		log.message("clicked\n");
	}
	
	// enter/leave callbacks
	void enter() {
		button.setTexCoord(vec4(0.0f,0.25f,1.0f,0.5f));
	}
	void leave() {
		button.setTexCoord(vec4(0.0f,0.0f,1.0f,0.25f));
	}
	
	// callback redirector
	void callback_redirector(string func,Window window) {
		window.call(func);
	}
};

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.console.setOnscreen(1);
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	Window window_0 = new Window(vec4(0.8f,1.0f,1.0f,1.0f));
	Window window_1 = new Window(vec4(1.0f,0.8f,0.8f,1.0f));
	
	// show windows
	window_0.show(200,200);
	window_1.show(600,200);
	
	setDescription("User interfaces with User classes");
	
	return 1;
}

/*
*/
int shutdown()
{
	engine.console.setOnscreen(0);
	return 1;
}

```

## unigine.cpp

```cpp
/* Copyright (C) 2005-2025, UNIGINE. All rights reserved.
 *
 * This file is a part of the UNIGINE 2 SDK.
 *
 * Your use and / or redistribution of this software in source and / or
 * binary form, with or without modification, is subject to: (i) your
 * ongoing acceptance of and compliance with the terms and conditions of
 * the UNIGINE License Agreement; and (ii) your inclusion of this notice
 * in any version of this software that you use or redistribute.
 * A copy of the UNIGINE License Agreement is available by contacting
 * UNIGINE. at http://unigine.com/
 */



#include <core/unigine.h>
#include <core/scripts/system/system.h>
#include <core/scripts/system/stereo.h>
#include <core/scripts/system/wall.h>

/******************************************************************************\
*
* init/shutdown
*
\******************************************************************************/

/*
 */
int init() {
	
	systemInit();
	stereoInit();
	wallInit();
	
	engine.console.run("world_load uniginescript_samples");
	
	return 1;
}

/*
 */
int shutdown() {
	
	systemShutdown();
	stereoShutdown();
	wallShutdown();
	
	return 1;
}

/******************************************************************************\
*
* update/postUpdate
*
\******************************************************************************/

/*
 */
int update() {
	
	systemUpdate();
	stereoUpdate();
	
	return 1;
}

/*
 */
int postUpdate() {
	
	wallPostUpdate();
	
	return 1;
}

```

## uniginescript_samples.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
int init() {
	Unigine::Samples::createDirInterface("uniginescript_samples/uniginescript_samples.world");
	if (engine.editor.isLoaded()) return 1;
	EngineWindow main_window = engine.window_manager.getMainWindow();
	main_window.setTitle("UNIGINE Engine" + Unigine::Samples::getVideoApp());
	return 1;
}

```

## velocity_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
Body body;
float angle;
float height;

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("Body velocity transformation");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	body = createBodyBox(vec3(2.0f,2.0f,1.0f),1.0f,0.25f,0.5f,get_material(1),translate(Vec3(0.0f,0.0f,2.0f)));
	
	return 1;
}

/*
 */
void updatePhysics() {
	
	angle += 16.0f;
	height += 0.2f;
	
	if(angle < 64.0f) {
		body.setVelocityTransform(translate(Vec3(0.0f,0.0f,2.0f + height)) * rotateZ(angle));
	}
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## video_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Video {
	
	Gui gui;
	
	WidgetSpriteVideo sprite_video;
	
	WidgetLabel label;
	
	// constructor/destructor
	Video() {
		
		gui = engine.getGui();
		
		// sprite video
		sprite_video = new WidgetSpriteVideo(gui,fullPath("uniginescript_samples/widgets/videos/unigine.ogv"));
		sprite_video.arrange();
		sprite_video.setPosition((gui.getWidth() - sprite_video.getWidth()) / 2,(gui.getHeight() - sprite_video.getHeight()) / 2);
		gui.addChild(sprite_video,GUI_ALIGN_OVERLAP | GUI_ALIGN_BACKGROUND);
		
		// label
		label = new WidgetLabel(gui);
		label.setFontSize(32);
		label.setFontOutline(1);
		label.setPosition(sprite_video.getPositionX(),sprite_video.getPositionY());
		gui.addChild(label,GUI_ALIGN_OVERLAP);
		
		// update thread
		thread("Video::update",sprite_video,label);
		
		// run video
		sprite_video.setLoop(1);
		sprite_video.play();
	}
	~Video() {
		delete sprite_video;
		delete label;
	}
	
	// save/restore state
	void __save__(Stream stream) {
		stream.writeFloat(sprite_video.getVideoTime());
	}
	void __restore__(Stream stream) {
		__Video__();
		sprite_video.setVideoTime(stream.readFloat());
	}
	
	// thread
	void update(WidgetSpriteVideo sprite_video,WidgetLabel label) {
		while(sprite_video != NULL) {
			label.setText(format("vtime: %.2fs",sprite_video.getVideoTime()));
			wait;
		}
	}
};

Video video;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	video = new Video();
	
	setDescription("WidgetSpriteVideo");
	
	return 1;
}

```

## video_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Video {
	
	Gui gui;
	
	WidgetSpriteVideo sprite_video;
	AmbientSource sprite_audio;
	
	WidgetLabel label;
	
	// constructor/destructor
	Video() {
		
		gui = engine.getGui();
		
		// sprite video
		sprite_video = new WidgetSpriteVideo(gui,fullPath("uniginescript_samples/widgets/videos/winter.ogv"),0);
		sprite_video.arrange();
		sprite_video.setPosition((gui.getWidth() - sprite_video.getWidth()) / 2,(gui.getHeight() - sprite_video.getHeight()) / 2);
		gui.addChild(sprite_video,GUI_ALIGN_OVERLAP | GUI_ALIGN_BACKGROUND);
		
		// sprite audio
		sprite_audio = new AmbientSource(fullPath("uniginescript_samples/widgets/videos/winter.ogv"));
		
		// label
		label = new WidgetLabel(gui);
		label.setFontSize(32);
		label.setFontOutline(1);
		label.setPosition(sprite_video.getPositionX(),sprite_video.getPositionY());
		gui.addChild(label,GUI_ALIGN_OVERLAP);
		
		// update thread
		thread("Video::update",sprite_video,sprite_audio,label);
		
		// sync video with audio
		sprite_video.setAmbientSource(sprite_audio);
		
		// run audio
		sprite_audio.setLoop(1);
		sprite_audio.play();
		sprite_video.play();
	}
	~Video() {
		delete sprite_video;
		delete sprite_audio;
		delete label;
	}
	
	// save/restore state
	void __save__(Stream stream) {
		stream.writeFloat(sprite_video.getVideoTime());
		stream.writeFloat(sprite_audio.getTime());
	}
	void __restore__(Stream stream) {
		__Video__();
		sprite_video.setVideoTime(stream.readFloat());
		sprite_audio.setTime(stream.readFloat());
	}
	
	// thread
	void update(WidgetSpriteVideo sprite_video,AmbientSource sprite_audio,WidgetLabel label) {
		while(sprite_video != NULL) {
			label.setText(format("vtime: %.2fs\natime: %.2fs",sprite_video.getVideoTime(),sprite_audio.getTime()));
			wait;
		}
	}
};

Video video;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	video = new Video();
	
	setDescription("WidgetSpriteVideo with AmbientSource");
	
	return 1;
}

```

## viewport_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
WidgetSpriteViewport sprite;
float angle = 0.0f;

/*
 */
string phong_mesh_material_names[] = ( 
	"uniginescript_samples/render/common/render/render_phong_mesh_red.mat", 
	"uniginescript_samples/render/common/render/render_phong_mesh_green.mat", 
	"uniginescript_samples/render/common/render/render_phong_mesh_blue.mat", 
	"uniginescript_samples/render/common/render/render_phong_mesh_orange.mat", 
	"uniginescript_samples/render/common/render/render_phong_mesh_yellow.mat"
	);

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("WidgetSpriteViewport");
	
	int size = 3;
	float space = 16.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(engine.materials.findMaterialByPath(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	return 1;
}

/*
 */
int update() {
	
	int size = 256;
	
	angle += engine.game.getIFps();
	mat4 projection = perspective(60.0f,1.0f,0.1f,100.0f);
	Mat4 modelview = lookAt(Vec3(14.0f),Vec3(0.0f,0.0f,8.0f),vec3(0.0f,0.0f,1.0f)) * rotateZ(angle * 32.0f);
	
	if(sprite == NULL) {
		sprite = new WidgetSpriteViewport(engine.getGui(),size,size);
		engine.gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_RIGHT);
		sprite.setTransform(scale(256.0f / size,256.0f / size,1.0f));
	}
	
	sprite.setProjection(projection);
	sprite.setModelview(modelview);
	
	return 1;
}

```

## viewport_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
WidgetSpriteViewport sprites[0];

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {

	createInterface(engine.world.getPath());

	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();

	setDescription("WidgetSpriteViewport");

	int size = 1;
	float space = 16.0f;

	engine.render.setEnabled(0);

	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}

	return 1;
}

/*
 */
int update() {

	int grid_x = 3;
	int grid_y = 2;

	EngineWindow main_window = engine.window_manager.getMainWindow();
	ivec2 window_size = main_window.getSize();

	int space_x = window_size.x / 64;
	int space_y = window_size.y / 64;

	int width  = (window_size.x - space_x * 2) / grid_x;
	int height = (window_size.y - space_y * 2) / grid_y;

	float aspect = float(width) / height;

	float bezel_x = 0.04f;
	float bezel_y = 0.02f;
	float angle = 0.0f;

	if(sprites.size() == 0 || sprites[0] == NULL) {

		WidgetVBox vbox = new WidgetVBox(engine.getGui());
		vbox.setBackground(1);
		vbox.setWidth(window_size.x);
		vbox.setHeight(window_size.y);
		engine.gui.addChild(vbox,GUI_ALIGN_OVERLAP | GUI_ALIGN_BACKGROUND);

		sprites.clear();
		forloop(int y = 0; grid_y) {
			forloop(int x = 0; grid_x) {
				WidgetSpriteViewport sprite = new WidgetSpriteViewport(engine.getGui(),width - space_x,height - space_y);
				sprite.setPosition(space_x * 3 / 2 + width * x,space_y * 3 / 2 + height * y);
				engine.gui.addChild(sprite,GUI_ALIGN_OVERLAP | GUI_ALIGN_BACKGROUND);
				sprites.append(sprite);
			}
		}
	}

	Player player = Unigine::getPlayer();
	mat4 projection = player.getProjection();
	Mat4 modelview = player.getIWorldTransform();

	forloop(int y = 0; grid_y) {
		forloop(int x = 0; grid_x) {
			mat4 ret_projection;
			Mat4 ret_modelview;
			WidgetSpriteViewport sprite = sprites[grid_x * y + x];
			Utils::getWallProjection(projection,modelview,aspect,grid_x,grid_y,x,y,bezel_x,bezel_y,angle,ret_projection,ret_modelview);
			sprite.setProjection(ret_projection);
			sprite.setModelview(ret_modelview);
		}
	}

	return 1;
}

/*
 */
int shutdown() {

	engine.render.setEnabled(1);
	return 1;
}
namespace Utils
{

void getWallProjection(mat4 projection, Mat4 modelview, float aspect, int grid_x, int grid_y, int view_x, int view_y, float bezel_x, float bezel_y, float angle, mat4 &ret_projection, Mat4 &ret_modelview)
{
	assert(grid_x > 0 && grid_y > 0 && "getWallProjection(): bad grid configuration");
	assert(view_x >= 0 && view_x < grid_x && "getWallProjection(): bad viewport X index");
	assert(view_y >= 0 && view_y < grid_y && "getWallProjection(): bad viewport Y index");

	// bounding frustum points
	vec3 points_0[8] = (vec3(-1.0f, -1.0f, -1.0f), vec3(1.0f, -1.0f, -1.0f),
		vec3(-1.0f, 1.0f, -1.0f), vec3(1.0f, 1.0f, -1.0f),
		vec3(-1.0f, -1.0f, 1.0f), vec3(1.0f, -1.0f, 1.0f),
		vec3(-1.0f, 1.0f, 1.0f), vec3(1.0f, 1.0f, 1.0f), );

	// transform points
	projection.m00 /= aspect;
	mat4 iprojection = inverse(projection);
	forloop(int i = 0; 8)
	{
		vec4 p = iprojection * vec4(points_0[i]);
		points_0[i] = vec3(p) / p.w;
	}

	// central viewport
	int center_x = grid_x / 2;
	int center_y = grid_y / 2;

	// even vertical split
	if ((grid_x & 0x01) == 0)
	{
		float w1 = points_0[1] - points_0[0];
		float w3 = points_0[3] - points_0[2];
		float offset = 0.5f;
		if (view_x >= center_x)
		{
			offset = -0.5f;
			center_x--;
		}
		points_0[0] += w1 * offset;
		points_0[1] += w1 * offset;
		points_0[2] += w3 * offset;
		points_0[3] += w3 * offset;
	}

	// even horizontal split
	if ((grid_y & 0x01) == 0)
	{
		float h2 = points_0[2] - points_0[0];
		float h3 = points_0[3] - points_0[1];
		float offset = -0.5f;
		if (view_y >= center_y)
		{
			offset = 0.5f;
			center_y--;
		}
		points_0[0] += h2 * offset;
		points_0[1] += h3 * offset;
		points_0[2] += h2 * offset;
		points_0[3] += h3 * offset;
	}

	// projection points
	vec3 points_1[8];

	// modelview matrix
	ret_modelview = modelview;

	// horizontal angle
	if (grid_x >= grid_y)
	{
		if (view_y < center_y)
		{
			for (int i = center_y; i > view_y; i--)
				move_points_up(points_0, points_0);
		} else if (view_y > center_y)
		{
			for (int i = center_y; i < view_y; i++)
				move_points_down(points_0, points_0);
		}
		if (view_x > center_x)
		{
			for (int i = center_x; i < view_x; i++)
			{
				transform_points(points_1, rotateY(-angle), points_0);
				move_points_right(points_1, points_0);
				transform_points(points_0, rotateY(angle), points_1);
				ret_modelview = rotateY(angle) * ret_modelview;
			}
		} else if (view_x < center_x)
		{
			for (int i = center_x; i > view_x; i--)
			{
				transform_points(points_1, rotateY(angle), points_0);
				move_points_left(points_1, points_0);
				transform_points(points_0, rotateY(-angle), points_1);
				ret_modelview = rotateY(-angle) * ret_modelview;
			}
		}
	}

	// vertical angle
	else
	{
		if (view_x > center_x)
		{
			for (int i = center_x; i < view_x; i++)
				move_points_right(points_0, points_0);
		} else if (view_x < center_x)
		{
			for (int i = center_x; i > view_x; i--)
				move_points_left(points_0, points_0);
		}
		if (view_y < center_y)
		{
			for (int i = center_y; i > view_y; i--)
			{
				transform_points(points_1, rotateX(-angle), points_0);
				move_points_up(points_1, points_0);
				transform_points(points_0, rotateX(angle), points_1);
				ret_modelview = rotateX(angle) * ret_modelview;
			}
		} else if (view_y > center_y)
		{
			for (int i = center_y; i < view_y; i++)
			{
				transform_points(points_1, rotateX(angle), points_0);
				move_points_down(points_1, points_0);
				transform_points(points_0, rotateX(-angle), points_1);
				ret_modelview = rotateX(-angle) * ret_modelview;
			}
		}
	}

	// projection matrix
	float left = points_0[0].x;
	float right = points_0[1].x;
	float bottom = points_0[0].y;
	float top = points_0[2].y;

	// horizontal bezel compensation
	float width = (right - left) * 0.5f;
	left += width * bezel_x;
	right -= width * bezel_x;

	// vertical bezel compensation
	float height = (top - bottom) * 0.5f;
	bottom += height * bezel_y;
	top -= height * bezel_y;

	ret_projection = frustum(left, right, bottom, top, -points_0[0].z, -points_0[4].z);
	ret_projection.m00 *= aspect;
}

void transform_points(vec3 dest[], mat4 transform, vec3 src[])
{
	dest[0] = transform * src[0];
	dest[1] = transform * src[1];
	dest[2] = transform * src[2];
	dest[3] = transform * src[3];
}

void move_points_left(vec3 dest[], vec3 src[])
{
	float s0 = src[0];
	float s2 = src[2];
	dest[0] = s0 + dest[0] - dest[1];
	dest[2] = s2 + dest[2] - dest[3];
	dest[1] = s0;
	dest[3] = s2;
}

void move_points_right(vec3 dest[], vec3 src[])
{
	float s1 = src[1];
	float s3 = src[3];
	dest[1] = s1 + dest[1] - dest[0];
	dest[3] = s3 + dest[3] - dest[2];
	dest[0] = s1;
	dest[2] = s3;
}

void move_points_up(vec3 dest[], vec3 src[])
{
	float s2 = src[2];
	float s3 = src[3];
	dest[2] = s2 + dest[2] - dest[0];
	dest[3] = s3 + dest[3] - dest[1];
	dest[0] = s2;
	dest[1] = s3;
}

void move_points_down(vec3 dest[], vec3 src[])
{
	float s0 = src[0];
	float s1 = src[1];
	dest[0] = s0 + dest[0] - dest[2];
	dest[1] = s1 + dest[1] - dest[3];
	dest[2] = s0;
	dest[3] = s1;
}

}

```

## viewport_02.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
ObjectGui guis[0];
WidgetSpriteViewport sprites[0];

float width = 30.0f;
float height = 20.0f;
float angle = 40.0f;

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {

	createInterface(engine.world.getPath());

	Player player = createDefaultPlayer(Vec3(20.0f,0.0f,9.0f));
	ObjectMeshDynamic plane = createDefaultPlane();

	setDescription("WidgetSpriteViewport");

	player.setDirection(Vec3(-1.0f,0.0f,0.0f));
	plane.setWorldTransform(Mat4(translate(2048.0f,0.0f,0.0f)));

	int size = 1;
	float space = 16.0f;

	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x + 128,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}

	float dx = sin(angle * DEG2RAD) * width / 2.0f;
	float dy = width / 2.0f + cos(angle * DEG2RAD) * width / 2.0f;

	ObjectGui gui = addToEditor(new ObjectGui(width,height));
	gui.setWorldTransform(Mat4(translate(0.0f,-dy,9.0f) * rotateZ(angle) * rotateY(90.0f) * rotateZ(90.0f)));
	gui.setMaterial(findMaterialByName("gui_base"),"*");
	guis.append(gui);

	gui = addToEditor(new ObjectGui(width,height));
	gui.setWorldTransform(Mat4(translate(-dx,0.0f,9.0f) * rotateY(90.0f) * rotateZ(90.0f)));
	gui.setMaterial(findMaterialByName("gui_base"),"*");
	guis.append(gui);

	gui = addToEditor(new ObjectGui(width,height));
	gui.setWorldTransform(Mat4(translate(0.0f,dy,9.0f) * rotateZ(-angle) * rotateY(90.0f) * rotateZ(90.0f)));
	gui.setMaterial(findMaterialByName("gui_base"),"*");
	guis.append(gui);

	return 1;
}

/*
 */
int update() {

	float aspect = float(width) / height;

	if(sprites.size() == 0 || sprites[0] == NULL) {
		sprites.clear();
		forloop(int i = 0; guis.size()) {
			WidgetSpriteViewport sprite = new WidgetSpriteViewport(guis[i].getGui(),int(width) * 32,int(height) * 32);
			Gui(guis[i].getGui()).addChild(sprite,GUI_ALIGN_EXPAND);
			sprites.append(sprite);
		}
	}

	mat4 projection = perspective(60.0f,1.0f,0.1f,1000.0f);
	Mat4 modelview = lookAt(Vec3(14.0f),Vec3(0.0f,0.0f,8.0f),vec3(0.0f,0.0f,1.0f)) * rotateZ(engine.game.getTime() * 32.0f) * translate(-2048.0f,0.0f,0.0f);

	forloop(int i = 0; sprites.size()) {

		mat4 ret_projection;
		Mat4 ret_modelview;
		WidgetSpriteViewport sprite = sprites[i];
		Utils::getWallProjection(projection,modelview,aspect,sprites.size(),1,i,0,0.0f,0.0f,angle,ret_projection,ret_modelview);
		sprite.setProjection(ret_projection);
		sprite.setModelview(ret_modelview);
	}

	return 1;
}

/*
 */
int shutdown() {

	engine.render.setEnabled(1);
	return 1;
}

namespace Utils
{

void getWallProjection(mat4 projection, Mat4 modelview, float aspect, int grid_x, int grid_y, int view_x, int view_y, float bezel_x, float bezel_y, float angle, mat4 &ret_projection, Mat4 &ret_modelview)
{
	assert(grid_x > 0 && grid_y > 0 && "getWallProjection(): bad grid configuration");
	assert(view_x >= 0 && view_x < grid_x && "getWallProjection(): bad viewport X index");
	assert(view_y >= 0 && view_y < grid_y && "getWallProjection(): bad viewport Y index");

	// bounding frustum points
	vec3 points_0[8] = (vec3(-1.0f, -1.0f, -1.0f), vec3(1.0f, -1.0f, -1.0f),
		vec3(-1.0f, 1.0f, -1.0f), vec3(1.0f, 1.0f, -1.0f),
		vec3(-1.0f, -1.0f, 1.0f), vec3(1.0f, -1.0f, 1.0f),
		vec3(-1.0f, 1.0f, 1.0f), vec3(1.0f, 1.0f, 1.0f), );

	// transform points
	projection.m00 /= aspect;
	mat4 iprojection = inverse(projection);
	forloop(int i = 0; 8)
	{
		vec4 p = iprojection * vec4(points_0[i]);
		points_0[i] = vec3(p) / p.w;
	}

	// central viewport
	int center_x = grid_x / 2;
	int center_y = grid_y / 2;

	// even vertical split
	if ((grid_x & 0x01) == 0)
	{
		float w1 = points_0[1] - points_0[0];
		float w3 = points_0[3] - points_0[2];
		float offset = 0.5f;
		if (view_x >= center_x)
		{
			offset = -0.5f;
			center_x--;
		}
		points_0[0] += w1 * offset;
		points_0[1] += w1 * offset;
		points_0[2] += w3 * offset;
		points_0[3] += w3 * offset;
	}

	// even horizontal split
	if ((grid_y & 0x01) == 0)
	{
		float h2 = points_0[2] - points_0[0];
		float h3 = points_0[3] - points_0[1];
		float offset = -0.5f;
		if (view_y >= center_y)
		{
			offset = 0.5f;
			center_y--;
		}
		points_0[0] += h2 * offset;
		points_0[1] += h3 * offset;
		points_0[2] += h2 * offset;
		points_0[3] += h3 * offset;
	}

	// projection points
	vec3 points_1[8];

	// modelview matrix
	ret_modelview = modelview;

	// horizontal angle
	if (grid_x >= grid_y)
	{
		if (view_y < center_y)
		{
			for (int i = center_y; i > view_y; i--)
				move_points_up(points_0, points_0);
		} else if (view_y > center_y)
		{
			for (int i = center_y; i < view_y; i++)
				move_points_down(points_0, points_0);
		}
		if (view_x > center_x)
		{
			for (int i = center_x; i < view_x; i++)
			{
				transform_points(points_1, rotateY(-angle), points_0);
				move_points_right(points_1, points_0);
				transform_points(points_0, rotateY(angle), points_1);
				ret_modelview = rotateY(angle) * ret_modelview;
			}
		} else if (view_x < center_x)
		{
			for (int i = center_x; i > view_x; i--)
			{
				transform_points(points_1, rotateY(angle), points_0);
				move_points_left(points_1, points_0);
				transform_points(points_0, rotateY(-angle), points_1);
				ret_modelview = rotateY(-angle) * ret_modelview;
			}
		}
	}

	// vertical angle
	else
	{
		if (view_x > center_x)
		{
			for (int i = center_x; i < view_x; i++)
				move_points_right(points_0, points_0);
		} else if (view_x < center_x)
		{
			for (int i = center_x; i > view_x; i--)
				move_points_left(points_0, points_0);
		}
		if (view_y < center_y)
		{
			for (int i = center_y; i > view_y; i--)
			{
				transform_points(points_1, rotateX(-angle), points_0);
				move_points_up(points_1, points_0);
				transform_points(points_0, rotateX(angle), points_1);
				ret_modelview = rotateX(angle) * ret_modelview;
			}
		} else if (view_y > center_y)
		{
			for (int i = center_y; i < view_y; i++)
			{
				transform_points(points_1, rotateX(angle), points_0);
				move_points_down(points_1, points_0);
				transform_points(points_0, rotateX(-angle), points_1);
				ret_modelview = rotateX(-angle) * ret_modelview;
			}
		}
	}

	// projection matrix
	float left = points_0[0].x;
	float right = points_0[1].x;
	float bottom = points_0[0].y;
	float top = points_0[2].y;

	// horizontal bezel compensation
	float width = (right - left) * 0.5f;
	left += width * bezel_x;
	right -= width * bezel_x;

	// vertical bezel compensation
	float height = (top - bottom) * 0.5f;
	bottom += height * bezel_y;
	top -= height * bezel_y;

	ret_projection = frustum(left, right, bottom, top, -points_0[0].z, -points_0[4].z);
	ret_projection.m00 *= aspect;
}

void transform_points(vec3 dest[], mat4 transform, vec3 src[])
{
	dest[0] = transform * src[0];
	dest[1] = transform * src[1];
	dest[2] = transform * src[2];
	dest[3] = transform * src[3];
}

void move_points_left(vec3 dest[], vec3 src[])
{
	float s0 = src[0];
	float s2 = src[2];
	dest[0] = s0 + dest[0] - dest[1];
	dest[2] = s2 + dest[2] - dest[3];
	dest[1] = s0;
	dest[3] = s2;
}

void move_points_right(vec3 dest[], vec3 src[])
{
	float s1 = src[1];
	float s3 = src[3];
	dest[1] = s1 + dest[1] - dest[0];
	dest[3] = s3 + dest[3] - dest[2];
	dest[0] = s1;
	dest[2] = s3;
}

void move_points_up(vec3 dest[], vec3 src[])
{
	float s2 = src[2];
	float s3 = src[3];
	dest[2] = s2 + dest[2] - dest[0];
	dest[3] = s3 + dest[3] - dest[1];
	dest[0] = s2;
	dest[1] = s3;
}

void move_points_down(vec3 dest[], vec3 src[])
{
	float s0 = src[0];
	float s1 = src[1];
	dest[0] = s0 + dest[0] - dest[2];
	dest[1] = s1 + dest[1] - dest[3];
	dest[2] = s0;
	dest[3] = s1;
}

}

```

## viewport_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
ObjectGui guis[];
WidgetSpriteViewport sprites[];
vec3 tangents[];
vec3 binormals[];
vec3 positions[];

PlayerSpectator player;

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {

	createInterface(engine.world.getPath());

	ObjectMeshDynamic plane = createDefaultPlane();
	player = createDefaultPlayer(Vec3(25.0f,0.0f,2.0f));

	setDescription("WidgetSpriteViewport");

	player.setDirection(vec3(-1.0f,0.0f,-0.0f));
	plane.setWorldTransform(Mat4(translate(2048.0f,0.0f,0.0f)));

	engine.console.setInt("render_taa",1);
	int size = 1;
	float space = 16.0f;

	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x + 128,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}

	Utils::getMeshProjection(fullPath("uniginescript_samples/render/meshes/viewport_03.mesh"),tangents,binormals,positions);

	foreachkey(string name = 0; positions) {

		vec3 tangent = tangents[name];
		vec3 binormal = binormals[name];
		vec3 position = positions[name];

		Mat4 transform = Mat4_identity;
		transform.col33 = position;
		transform.col03 = normalize(tangent);
		transform.col13 = normalize(binormal);
		transform.col23 = normalize(cross(tangent,binormal));

		float width = length(tangent);
		float height = length(binormal);

		float aspect = height / width;

		ObjectGui gui = addToEditor(new ObjectGui(width,height));
		gui.setWorldTransform(transform);
		gui.setMaterial(findMaterialByName("gui_base"),"*");
		gui.setScreenSize(1024,int(1024 * aspect));
		guis.append(name,gui);
	}

	return 1;
}

/*
 */
int update() {

	mat4 projection = mat4_identity;
	Mat4 modelview = Mat4_identity;

	vec3 camera = player.getWorldPosition();

	foreachkey(string name; positions) {

		ObjectGui gui = guis[name];
		vec3 tangent = tangents[name];
		vec3 binormal = binormals[name];

		WidgetSpriteViewport sprite = sprites.check(name,NULL);
		if(sprite == NULL) {
			int width = 1024;
			float aspect = length(binormal) / length(tangent);
			int height = int(width * aspect);
			sprite = new WidgetSpriteViewport(gui.getGui(),width,height);
			sprite.setAspectCorrection(0);
			gui.getGui().addChild(sprite,GUI_ALIGN_EXPAND);
			sprites.append(name,sprite);
		}
		vec3 offset = vec3(2048.0f,0.0f,10.0f);
		Utils::getViewProjection(vec3_zero,positions[name],tangents[name],binormals[name],0.01f,1000.0f,projection,modelview);
		sprite.setProjection(projection);
		modelview *= translate(-offset - camera);

		sprite.setModelview(modelview);

	}

	return 1;
}

namespace Utils
{
	void getViewProjection(Vec3 camera, Vec3 position, vec3 tangent, vec3 binormal, float near, float far, mat4 &projection, Mat4 &modelview)
	{
		// view basis
		vec3 normal = normalize(cross(tangent, binormal));
		vec3 right = normalize(tangent);
		vec3 up = normalize(binormal);

		// relative position
		position = vec3(position - camera);

		// projection matrix
		float scale = -near / dot(normal, position);
		float l = dot(right, position - tangent * 0.5f) * scale;
		float r = dot(right, position + tangent * 0.5f) * scale;
		float b = dot(up, position - binormal * 0.5f) * scale;
		float t = dot(up, position + binormal * 0.5f) * scale;
		projection = frustum(l, r, b, t, near, far);

		// modelview matrix
		modelview = Mat4_identity;
		modelview.row03 = right;
		modelview.row13 = up;
		modelview.row23 = normal;
		modelview.row33 = -camera;
	}

	void getMeshProjection(string name, vec4 tangents[], vec3 binormals[], vec3 positions[])
	{
		// load mesh
		Mesh mesh = new Mesh(name);

		// process surfaces
		forloop(int i = 0; mesh.getNumSurfaces())
		{
			string name = mesh.getSurfaceName(i);

			// surface triangles
			forloop(int j = 0; mesh.getNumCIndices(i); 3)
			{
				// first triangle
				int i0 = mesh.getCIndex(j + 0, i);
				int i1 = mesh.getCIndex(j + 1, i);
				int i2 = mesh.getCIndex(j + 2, i);
				vec3 p0 = mesh.getVertex(i0, i);
				vec3 p1 = mesh.getVertex(i1, i);
				vec3 p2 = mesh.getVertex(i2, i);

				// unique indices
				int indices[0];
				forloop(int k = 0; mesh.getNumCIndices(i))
				{
					int index = indices.find(mesh.getCIndex(k, i));
					if (index != -1)
						indices.remove(index);
					else
						indices.append(mesh.getCIndex(k, i));
				}

				vec3 tangent;
				vec3 binormal;
				vec3 position;

				// surface orientation
				if (indices[0] == i0)
				{
					vec3 p10 = p1 - p0;
					vec3 p20 = p2 - p0;
					tangent = p10;
					binormal = p20;
					position = p0;
				} else if (indices[0] == i1)
				{
					vec3 p01 = p0 - p1;
					vec3 p21 = p2 - p1;
					tangent = p01;
					binormal = p21;
					position = p1;
				} else if (indices[0] == i2)
				{
					vec3 p02 = p0 - p2;
					vec3 p12 = p1 - p2;
					tangent = p02;
					binormal = p12;
					position = p2;
				}

				// add surface
				tangents.append(name, tangent);
				binormals.append(name, binormal);
				positions.append(name, position + (tangent + binormal) * 0.5f);

				break;
			}
		}

		delete mesh;
	}
}

```

## visualizer_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
ObjectMeshStatic mesh_wire;
ObjectMeshStatic mesh_solid;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	setDescription("Visualizer object");
	
	engine.visualizer.setEnabled(1);
	
	mesh_wire = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
	mesh_solid = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
	
	mesh_wire.setWorldTransform(translate(Vec3(0.0f,-5.0f,0.0f)));
	mesh_solid.setWorldTransform(translate(Vec3(0.0f,5.0f,0.0f)));
	
	mesh_wire.setMaterial(findMaterialByName("mesh_visualizer_base"),"*");
	mesh_solid.setMaterial(findMaterialByName("mesh_visualizer_base"),"*");
	
	return 1;
}

/*
 */
int update() {
	
	engine.visualizer.renderObject(mesh_wire,vec4(1.0f, 1.0f, 1.0f, 0.5f));
	engine.visualizer.renderSolidObject(mesh_solid,vec4(1.0f, 1.0f, 1.0f, 0.5f));
	
	return 1;
}

```

## volumetric_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	setDescription("Light scattering");
	
	engine.console.run("render_screen_space_shadow_shafts_mode 1");
	
	Node sun = engine.world.getNodeByName("sun");
	sun.setWorldTransform(Mat4(rotateY(-50.0f)));
	
	int size = 3;
	float space = 16.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	return 1;
}

```

## volumetric_01.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	setDescription("Light scattering");
	
	engine.console.run("render_screen_space_shadow_shafts_mode 1");
	
	Node sun = engine.world.getNodeByName("sun");
	sun.setWorldTransform(Mat4(rotateY(-50.0f)));
	
	ObjectParticles particles = addToEditor(new ObjectParticles());
	particles.setWorldTransform(translate(Vec3(0.0f,0.0f,2.0f)));
	particles.setMaterial(findMaterialByName("render_particles"),"*");
	
	particles.setSpawnRate(2000.0f);
	
	particles.setEmitterEnabled(1);
	particles.setEmitterType(OBJECT_PARTICLES_EMITTER_BOX);
	particles.setEmitterSize(vec3(40.0f,40.0f,20.0f));
	
	particles.setLife(1.0f,0.5f);
	
	ParticleModifierScalar radius_modifier = particles.getRadiusOverTimeModifier();
	radius_modifier.setMode(PARTICLE_MODIFIER_MODE_RANDOM_BETWEEN_TWO_CONSTANTS);
	radius_modifier.setConstantMin(0.1f);
	radius_modifier.setConstantMax(0.3f);
	
	ParticleModifierScalar growth_modifier = particles.getGrowthOverTimeModifier();
	growth_modifier.setMode(PARTICLE_MODIFIER_MODE_RANDOM_BETWEEN_TWO_CONSTANTS);
	growth_modifier.setConstantMin(0.0f);
	growth_modifier.setConstantMax(0.2f);
	
	ParticleModifierScalar velocity_modifier = particles.getVelocityOverTimeModifier();
	velocity_modifier.setMode(PARTICLE_MODIFIER_MODE_RANDOM_BETWEEN_TWO_CONSTANTS);
	velocity_modifier.setConstantMin(0.0f);
	velocity_modifier.setConstantMax(2.0f);
	
	ParticleModifierVector gravity_modifier = particles.getGravityOverTimeModifier();
	gravity_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
	gravity_modifier.setConstant(vec3(0.0f,0.0f,4.0f));
	
	return 1;
}

```

## vr_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>
#include <core/scripts/euler.h>

using Unigine::Samples;

namespace VR 
{
	#define CONTROLLER_COUNT 2
	#define BASESTATION_COUNT 2

	PlayerDummy player;
	Node controller[0];
	ObjectMeshStatic controller_objects[0];
	Mesh controller_meshes[0];
	Texture controller_textures[0];

	Node basestation[0];
	ObjectMeshStatic basestation_objects[0];
	Mesh basestation_meshes[0];
	Texture basestation_textures[0];

	int controller_loaded[0];
	float controller_loading_time[0];
	float max_controller_loading_time[0];

	// teleportation	
	Node teleport_marker;
	ObjectMeshDynamic teleport_ray;
	int teleport_button_pressed[2];
	WorldIntersection intersection = new WorldIntersection();

	// VR
	InputVRHead head_device = NULL;
	InputVRController left_controller_device = NULL;
	InputVRController right_controller_device = NULL;

	void findDevices()
	{
		head_device = engine.input.getVRHead();
		if(left_controller_device == NULL)
			left_controller_device = engine.input.getVRControllerLeft();
		if(right_controller_device == NULL)
			right_controller_device = engine.input.getVRControllerRight();
	}

	int init(Node camera_node)
	{
        engine.vr.resetZeroPose();

		head_device = engine.input.getVRHead();
		if(head_device == NULL)
			log.warning("init(): head device not found\n");

		left_controller_device = engine.input.getVRControllerLeft();
		if(left_controller_device == NULL)
			log.warning("init(): left controller device not found\n");

		right_controller_device = engine.input.getVRControllerRight();
		if(right_controller_device == NULL)
			log.warning("init(): right controller device not found\n");

		Player player_ref = node_cast(camera_node);

		// create new player
		player = new PlayerDummy();
		player.setZNear(player_ref.getZNear());
		player.setZFar(player_ref.getZFar());
		teleport_and_rotate_to(player_ref.getWorldPosition(), player_ref.getWorldDirection());
		engine.game.setPlayer(player);

		for (int i = 0; i < CONTROLLER_COUNT; i++)
		{
			controller.append(new NodeDummy());
			controller_loaded.append(0);
			controller_loading_time.append(0.0f);
			max_controller_loading_time.append(2.0f);
		}

		for (int i = 0; i < BASESTATION_COUNT; i++)
			basestation.append(new NodeDummy());

		teleport_init("Teleport", "uniginescript_samples/systems/common/teleport_ray_mat.mat");
		
		return 1;
	}

	int shutdown()
	{
		delete teleport_ray;
		return 1;
	}

	int update()
	{
		// check devices (on/off controllers, etc.)
		findDevices();

		if (head_device == NULL)
			return;

		// get head position in local/world coords
		Mat4 player_transform = player.getWorldTransform();
		Mat4 hmd_transform = head_device.getTransform();
		Mat4 hmd_transform_world = player_transform * hmd_transform;
		Vec3 head_offset = (head_device != NULL && head_device.isAvailable()) ? (player_transform.col33 - hmd_transform_world.col33) : Vec3(0, 0, 0);
		head_offset.z = 0;
		
		controller_update(left_controller_device, 0);
		controller_update(right_controller_device, 1);

		basestation_update(0);
		basestation_update(1);

		// update teleportations
		teleport_update(0, left_controller_device, head_offset);
		teleport_update(1, right_controller_device, head_offset);
		
		// set vibration when player want to teleport
		if (left_controller_device != NULL && is_teleport_button_pressed(left_controller_device))
			left_controller_device.applyHaptic(1000);

		if (right_controller_device != NULL && is_teleport_button_pressed(right_controller_device))
			right_controller_device.applyHaptic(1000);
		
		return 1;
	}
	
	void teleport_init(string marker_node, string marker_mat)
	{
		// marker init
		teleport_marker = engine.world.getNodeByName(marker_node);
		teleport_marker.setEnabled(0);

		// ray init
		teleport_ray = new ObjectMeshDynamic();
		for (int i = 0; i < teleport_ray.getNumSurfaces(); i++)
		{
			teleport_ray.setMaterial(engine.materials.findMaterialByPath(marker_mat), i);
			teleport_ray.setSurfaceProperty("surface_base", i);
			teleport_ray.setCastShadow(0, i);
			teleport_ray.setCastWorldShadow(0, i);
			teleport_ray.setCollision(0, i);
			teleport_ray.setIntersection(0, i);
		}
		teleport_ray.setPosition(Vec3(0, 0, 0));
		teleport_ray.setRotation(quat_identity);

		// clear array
		for (int i = 0; i < 2; i++)
			teleport_button_pressed[i] = 0;
	}

	void load_glove(int index)
	{
		controller_objects[index] = new ObjectMeshStatic();
		controller_objects[index].setName("controller");
		controller_objects[index].setParent(controller[index]);
		controller_objects[index].setMeshPath(index == 0 ? "uniginescript_samples/systems/common/gloves/glove_left.mesh" : "uniginescript_samples/systems/common/gloves/glove_right.mesh");
		Material material = engine.materials.findMaterialByPath("uniginescript_samples/systems/common/gloves/gloves.mat");
		if (material != NULL)
			controller_objects[index].setMaterial(material, "*");
	
		for (int j = 0; j < controller_objects[index].getNumSurfaces(); j++)
		{
			controller_objects[index].setCastWorldShadow(0, j);
			controller_objects[index].setCastShadow(0, j);
			controller_objects[index].setCastEnvProbeShadow(0, j);
		}
		controller_objects[index].setTransform(Mat4_identity);
	};

	int load_controller_combined(int index, InputVRController controller_device)
	{ 
		controller_meshes[index] = controller_device.getCombinedModelMesh();
		controller_textures[index] = controller_device.getCombinedModelTexture();
		if (controller_meshes[index] != NULL && controller_textures[index] != NULL)
		{
			controller_objects[index] = new ObjectMeshStatic();
			controller_objects[index].setName("controller");
			controller_objects[index].setParent(controller[index]);
			controller_objects[index].setMeshProceduralMode(1);
			controller_objects[index].applyCopyMeshProceduralForce(controller_meshes[index]);
			Material material = engine.materials.findMaterialByPath("uniginescript_samples/systems/common/vr_controller.mgraph");
			if (material != NULL)
			{
				controller_objects[index].setMaterial(material, "*");
				Material mat = controller_objects[index].getMaterialInherit(0);
	
				int id = mat.findTexture("albedo");
				if (id != -1)
					mat.setTexture(id, controller_textures[index]);
			}
	
			for (int j = 0; j < controller_objects[index].getNumSurfaces(); j++)
			{
				controller_objects[index].setCastWorldShadow(0, j);
				controller_objects[index].setCastShadow(0, j);
				controller_objects[index].setCastEnvProbeShadow(0, j);
			}
	
			return 1;
		}
	
		return 0;
	};

	int load_controller_components(int index, InputVRController controller_device)
	{
		int idx = 0;
		int loaded = 1;
	
		int num_components = controller_device.getNumModels();
		for (int i = 0; i < num_components; i++)
		{
			string str = controller_device.getModelName(i);
			if (str == "base" || str == "status")
				continue;
	
			int model_index = index * num_components + idx;
			if (controller_objects[model_index] == NULL)
			{
				controller_meshes[model_index] = controller_device.getModelMesh(i);
				controller_textures[model_index] = controller_device.getModelTexture(i);
				if (controller_meshes[model_index] != NULL && controller_textures[model_index] != NULL)
				{
					controller_objects[model_index] = new ObjectMeshStatic();
					controller_objects[model_index].setName(str);
					controller_objects[model_index].setParent(controller[index]);
					controller_objects[model_index].setMeshProceduralMode(1);
					controller_objects[model_index].applyCopyMeshProceduralForce(controller_meshes[model_index]);
					Material material = engine.materials.findMaterialByPath("uniginescript_samples/systems/common/vr_controller.mgraph");
					if (material != NULL)
					{
						controller_objects[model_index].setMaterial(material, "*");
						Material mat = controller_objects[model_index].getMaterialInherit(0);
	
						int id = mat.findTexture("albedo");
						if (id != -1)
							mat.setTexture(id, controller_textures[model_index]);
					}
	
					for (int j = 0; j < controller_objects[model_index].getNumSurfaces(); j++)
					{
						controller_objects[model_index].setCastWorldShadow(0, j);
						controller_objects[model_index].setCastShadow(0, j);
						controller_objects[model_index].setCastEnvProbeShadow(0, j);
					}
				}
				else
				{
					loaded = 0;
				}
			}
			idx++;
		}
	
		return loaded;
	};

	void controller_update(InputVRController controller_device, int index)
	{
		if (index == -1 || controller_device == NULL)
			return;

		if (controller_device.isAvailable() == 0 || controller_device.isTransformValid() == 0)
			return;

		int num_components = controller_device.getNumModels();
		int visible = engine.vr.isSteamVRDashboardActive() == 0;

		if (controller_objects.size() == 0)
		{
			if (num_components == 0)
			{
				controller_objects.resize(1 * CONTROLLER_COUNT);
				controller_meshes.resize(1 * CONTROLLER_COUNT);
				controller_textures.resize(1 * CONTROLLER_COUNT);
			} else
			{
				controller_objects.resize(num_components * CONTROLLER_COUNT);
				controller_meshes.resize(num_components * CONTROLLER_COUNT);
				controller_textures.resize(num_components * CONTROLLER_COUNT);
			}
		}

		controller[index].setWorldTransform(controller_device.getWorldTransform());

		if (controller_loaded[index] == 1)
		{
			if (num_components == 0)
			{
				controller_objects[index].setWorldTransform(controller_device.getWorldTransform());
				controller_objects[index].setEnabled(visible);
			}
			else
			{
				int idx = 0;
				for (int i = 0; i < num_components; i++)
				{
					string str = controller_device.getModelName(i);
					if (str == "base" || str == "status")
						continue;
					
					int model_index = index * num_components + idx;
					if (controller_objects[model_index] != NULL)
					{
						controller_objects[model_index].setWorldTransform(controller_device.getModelWorldTransform(i));
						controller_objects[model_index].setEnabled(visible);
					}
					
					idx++;
				}
			}
		}
		else
		{
			controller_loading_time[index] += engine.game.getIFps();
		
			if (controller_loading_time[index] > max_controller_loading_time[index])
			{
				if(controller_objects[index] == NULL)
					load_glove(index);

				controller_loaded[index] = 1;
			}
			else
			{
				if (num_components == 0)
					controller_loaded[index] = load_controller_combined(index, controller_device);
				else
					controller_loaded[index] = load_controller_components(index, controller_device);
			}
		}
	}

	InputVRDevice get_basestation(int index)
	{
		int cnt = -1;
		for (int i = 0; i < engine.input.getNumVRDevices(); i++)
		{
			InputVRDevice device = engine.input.getVRDevice(i);

			if(device == NULL)
				continue;
			
			if (device.getDeviceType() == INPUT_VR_DEVICE_INPUT_VR_BASE_STATION)
				cnt++;

			if (cnt == index)
				return device;
		}

		return NULL;
	}

	void basestation_update(int index)
	{
		if (index < 0 || index >= BASESTATION_COUNT)
			return;

		InputVRDevice basestation_device = get_basestation(index);

		if (basestation_device == NULL)
		{
			basestation[index].setEnabled(false);
			return;
		} else
		{
			basestation[index].setEnabled(true);
		}

		if (basestation_device.isTransformValid() == false)
			return;


		int num_components = basestation_device.getNumModels();
		if (basestation_objects.size() == 0)
		{
			if (num_components == 0)
			{
				basestation_objects.resize(1 * BASESTATION_COUNT);
				basestation_meshes.resize(1 * BASESTATION_COUNT);
				basestation_textures.resize(1 * BASESTATION_COUNT);
			} else
			{
				basestation_objects.resize(num_components * BASESTATION_COUNT);
				basestation_meshes.resize(num_components * BASESTATION_COUNT);
				basestation_textures.resize(num_components * BASESTATION_COUNT);
			}
		}

		basestation[index].setWorldTransform(basestation_device.getWorldTransform());

		int visible = (engine.vr.isSteamVRDashboardActive() == false);
		if (num_components == 0)
		{
			if (basestation_objects[index] == NULL)
			{
				if (basestation_meshes[index] == NULL)
					basestation_meshes[index] = new Mesh();

				basestation_meshes[index] = basestation_device.getCombinedModelMesh();
				if (basestation_meshes[index] != NULL)
				{
					basestation_textures[index] = new Texture();
					basestation_textures[index] = basestation_device.getCombinedModelTexture();
					if (basestation_textures[index] != NULL)
					{
						basestation_objects[index] = new ObjectMeshStatic();
						basestation_objects[index].setParent(basestation[index]);
						basestation_objects[index].setMeshProceduralMode(true);
						basestation_objects[index].applyCopyMeshProceduralAsync(basestation_meshes[index]);
						Material material = engine.materials.findMaterialByPath("uniginescript_samples/systems/common/vr_controller.mgraph");
						if (material == NULL)
						{
							basestation_objects[index].setMaterial(material, "*");
							Material mat = basestation_objects[index].getMaterialInherit(0);

							int id = mat.findTexture("albedo");
							if(id != -1)
								mat.setTexture(id, basestation_textures[index]);
						}

						for (int j = 0; j < basestation_objects[index].getNumSurfaces(); j++)
						{
							basestation_objects[index].setCastWorldShadow(false, j);
							basestation_objects[index].setCastShadow(false, j);
							basestation_objects[index].setCastEnvProbeShadow(false, j);
						}
					}
				}
			} else
			{
				basestation_objects[index].setWorldTransform(basestation_device.getWorldTransform());
				basestation_objects[index].setEnabled(visible);
			}
		} else
		{
			int idx = 0;

			for (int i = 0; i < num_components; i++)
			{
				string str = basestation_device.getModelName(i);
				if (str == "base" || str == "status")
					continue;

				int model_index = index * num_components + idx;

				if (basestation_objects[model_index] == NULL)
				{
					if (basestation_meshes[model_index] == NULL)
						basestation_meshes[model_index] = new Mesh();

					basestation_meshes[model_index] = basestation_device.getModelMesh(i);
					if (basestation_meshes[model_index] != NULL)
					{
						basestation_textures[model_index] = new Texture();
						basestation_textures[model_index] = basestation_device.getModelTexture(i);
						if (basestation_textures[model_index] != NULL)
						{
							basestation_objects[model_index] = new ObjectMeshStatic();
							basestation_objects[model_index].setParent(basestation[index]);
							basestation_objects[model_index].setMeshProceduralMode(true);
							basestation_objects[model_index].applyCopyMeshProceduralAsync(basestation_meshes[model_index]);
							Material material = engine.materials.findMaterialByPath("uniginescript_samples/systems/common/vr_controller.mgraph");
							if (material == NULL)
							{
								basestation_objects[model_index].setMaterial(material, "*");
								Material mat = basestation_objects[model_index].getMaterialInherit(0);

								int id = mat.findTexture("albedo");
								if(id != -1)
									mat.setTexture(id, basestation_textures[model_index]);
							}

							for (int j = 0; j < basestation_objects[model_index].getNumSurfaces(); j++)
							{
								basestation_objects[model_index].setCastWorldShadow(false, j);
								basestation_objects[model_index].setCastShadow(false, j);
								basestation_objects[model_index].setCastEnvProbeShadow(false, j);
							}
						}
					}
				}
				else
				{
					basestation_objects[index].setWorldTransform(basestation_device.getModelWorldTransform(i));
					basestation_objects[index].setEnabled(visible);
				}

				idx++;
			}
		}
	}

	int is_teleport_button_pressed(InputVRController controller_device)
	{
		if (controller_device == NULL || controller_device.isAvailable() == 0 || controller_device.isTransformValid() == 0)
			return 0;

		int trackpad_index = controller_device.findAxisByType(INPUT_VR_CONTROLLER_AXIS_TYPE_TRACKPAD_X);
		int joystick_index = controller_device.findAxisByType(INPUT_VR_CONTROLLER_AXIS_TYPE_JOYSTICK_X);

		int trackpad_button = INPUT_VR_BUTTON_AXIS_0 + trackpad_index;
		int joystick_button = INPUT_VR_BUTTON_AXIS_0 + joystick_index;

		return controller_device.isButtonPressed(trackpad_button) || controller_device.isButtonPressed(joystick_button);
	}

	void teleport_update(int num, InputVRController controller_device, Vec3 offset)
	{
		if(controller_device == NULL)
			return;

		if (controller_device.isTransformValid() == false)
			return;

		int button_pressed = is_teleport_button_pressed(controller_device);

		// check if nobody using our teleport
		if (button_pressed &&
			!teleport_button_pressed[num] &&
			teleport_button_pressed[1 - num]) // if controllers > 2 it doesn't work!
			return;

		if (!button_pressed && !teleport_button_pressed[num])
			return;

		int last_button_state = teleport_button_pressed[num];
		teleport_button_pressed[num] = button_pressed;

		Mat4 aim_transform = controller_device.getWorldTransform(INPUT_VR_DEVICE_TRANSFORM_TYPE_AIM);
		Vec3 pos1 = Vec3(aim_transform.m03, aim_transform.m13, aim_transform.m23);
		Vec3 pos2 = pos1 - Vec3(aim_transform.m02, aim_transform.m12, aim_transform.m22) * player.getZFar();

		if (button_pressed == 1 || (button_pressed == 0 && last_button_state == 1))
		{
			Object hitObj = engine.world.getIntersection(pos1, pos2, 1, intersection);
			if (hitObj != NULL)
			{
				// show marker
				if (button_pressed == 1)
				{
					pos2 = intersection.getPoint() + Vec3(0.0f, 0.0f, 0.1f);
					teleport_marker.setPosition(pos2);
					teleport_marker.setEnabled(1);
				}

				// teleport!
				if (button_pressed == 0 && last_button_state == 1)
				{
					player.setPosition(intersection.getPoint() + offset);
					teleport_marker.setEnabled(0);
				}
			}
			else
			{
				teleport_marker.setEnabled(0);
			}
		}

		// show ray
		if (button_pressed)
		{
			teleport_ray.clearVertex();
			teleport_ray.clearIndices();

			int num = 30;	// num of quads
			float inum = 1.0f / num;

			Vec3 last_p = pos1;
			for (int i = 1; i <= num; i++)
			{
				Vec3 p = getHermiteSpline(pos1, pos1, pos2, pos2 + Vec3(0, 0, -3.0f), inum * i);
				addLineSegment(teleport_ray, last_p, p, 0.025f);
				last_p = p;
			}

			teleport_ray.updateBounds();
			teleport_ray.updateTangents();
			teleport_ray.flushVertex();
			teleport_ray.flushIndices();

			teleport_ray.setEnabled(1);
		}
		else
		{
			teleport_ray.setEnabled(0);
		}
	}
	
	void teleport_and_rotate_to(Vec3 position, Vec3 dir)
	{
		// rotate player to "dir"
		dir.z = 0;
		dir = normalize(dir);
		quat rot = conjugate(quat(lookAt(Vec3(0, 0, 0), Vec3(dir), Vec3(0, 0, 1))));
		Mat4 hmd_transform = head_device.getTransform();
		vec3 angles = Unigine::decomposeRotationYXZ(hmd_transform);
		player.setRotation(rot * quat(0.0f, -angles.y, 0.0f));

		// move player to "position"
		Mat4 player_transform = player.getWorldTransform();
		Mat4 hmd_transform_world = player_transform * hmd_transform;
		Vec3 head_offset = (head_device != NULL && head_device.isAvailable()) ? (player_transform.col33 - hmd_transform_world.col33) : Vec3(0, 0, 0);
		head_offset.z = 0;
		
		// put player to the ground
		Vec3 pos2 = position + Vec3(0, 0, -1) * player.getZFar();
		Object hitObj = engine.world.getIntersection(position, pos2, 1, intersection);
		if (hitObj != NULL)
			player.setPosition(intersection.getPoint() + head_offset);
	}
	
	Vec3 getHermiteSpline(Vec3 p0, Vec3 p1, Vec3 p2, Vec3 p3, float t)
	{
		float t2 = t * t;
		float t3 = t2 * t;

		float tension = 0.5f;	// 0.5 equivale a catmull-rom

		Vec3 pp1 = (p2 - p0) * tension;
		Vec3 pp2 = (p3 - p1) * tension;

		float blend1 = 2.0f * t3 - 3.0f * t2 + 1.0f;
		float blend2 = -2.0f * t3 + 3.0f * t2;
		float blend3 = t3 - 2.0f * t2 + t;
		float blend4 = t3 - t2;

		return p1 * blend1 + p2 * blend2 + pp1 * blend3 + pp2 * blend4;
	}
	
	void addLineSegment(ObjectMeshDynamic mesh, Vec3 from, Vec3 to, Vec3 from_forward, float width)
	{
		Vec3 up = Vec3(0, 0, 1);
		Vec3 to_forward = normalize(to - from);
		Vec3 to_right = normalize(cross(to_forward, up));
		Vec3 from_right = normalize(cross(from_forward, up));

		mesh.addTriangleQuads(1);
		Vec3 p0 = from - from_right * width * 0.5f;	// 0, 0
		Vec3 p1 = from + from_right * width * 0.5f;  // 1, 0
		Vec3 p2 = to + to_right * width * 0.5f;	// 1, 1
		Vec3 p3 = to - to_right * width * 0.5f;	// 0, 1
		mesh.addVertex(p0); mesh.addTexCoord(Vec4(0, 0, 0, 0));
		mesh.addVertex(p1); mesh.addTexCoord(Vec4(1, 0, 0, 0));
		mesh.addVertex(p2); mesh.addTexCoord(Vec4(1, 1, 0, 0));
		mesh.addVertex(p3); mesh.addTexCoord(Vec4(0, 1, 0, 0));
	}

	void addLineSegment(ObjectMeshDynamic mesh, Vec3 from, Vec3 to, float width)
	{
		Vec3 from_forward = normalize(to - from);
		addLineSegment(mesh, from, to, from_forward, width);
	}	

} // namespace VR

/*
 */
int init()
{
	createInterface(engine.world.getPath());
	
	if(engine.vr.getApiType() == VR_API_NULL)
	{
		createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
		setDescription("VR is not loaded");
	}
	else
	{
		VR::init(engine.world.getNodeByName("camera"));
	}

	return 1;
}

int update()
{
	if (engine.vr.getApiType() != VR_API_NULL)
		VR::update();

	return 1;
}

int shutdown()
{
	if (engine.vr.getApiType() != VR_API_NULL)
		VR::shutdown();

	return 1;
}

```

## vrpn_client_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>
using Unigine::Samples;

#ifdef HAS_VRPN_CLIENT

#define VRPN_NAME "DTrack@localhost"
#define FLYSTICK_BUTTONS 6
#define FLYSTICK_CHANNELS 2
#define NUM_SENSORS 2

/*
 */
Info info;
VrpnTrackerDevice tracker;
VrpnButtonDevice button;
VrpnAnalogDevice analog;

int buttons[FLYSTICK_BUTTONS];
float channels[FLYSTICK_CHANNELS];

vec3 sensor_positions[NUM_SENSORS];
quat sensor_orientations[NUM_SENSORS];

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	info = new Info();
	
	tracker = new VrpnTrackerDevice(VRPN_NAME);
	tracker.setTransformCallback("tracker_callback");
	
	button = new VrpnButtonDevice(VRPN_NAME);
	button.setButtonCallback("button_callback");
	
	analog = new VrpnAnalogDevice(VRPN_NAME);
	analog.setAnalogCallback("analog_callback");
	
	setDescription("VRPN Client sample");
	
	return 1;
}

/*
 */
int update() {
	
	tracker.update();
	button.update();
	analog.update();
	
	string str = "VRPN ART Controls\n";
	
	forloop(int i = 0; NUM_SENSORS) {
		str += format("\nSensor %d:\n",i);
		str += format(
			"    vec3: %.3f %.3f %.3f\n",
			sensor_positions[i].x,
			sensor_positions[i].y,
			sensor_positions[i].z
		);
		
		str += format(
			"    quat: %.3f %.3f %.3f %.3f\n",
			sensor_orientations[i].x,
			sensor_orientations[i].y,
			sensor_orientations[i].z,
			sensor_orientations[i].w
		);
	}
	
	str += format("\nChannels: %d\n",FLYSTICK_CHANNELS);
	forloop(int i = 0; FLYSTICK_CHANNELS) {
		str += format("    %.2f\n",channels[i]);
	}
	
	str += format("\nButtons: %d\n    ",FLYSTICK_BUTTONS);
	forloop(int i = 0; FLYSTICK_BUTTONS) {
		str += format("%d ",buttons[i]);
	}
	
	info.set(str);
	
	return 1;
}

/*
 */
void tracker_callback(int sensor,vec3 position,quat orientation) {
	if(sensor >= NUM_SENSORS) return;
	
	sensor_positions[sensor] = position;
	sensor_orientations[sensor] = orientation;
}

void button_callback(int button,int state) {
	if(button >= FLYSTICK_BUTTONS) return;
	
	buttons[button] = state;
}

void analog_callback(VrpnAnalogDevice device) {
	if(device.getNumChannels() > FLYSTICK_CHANNELS) return;
	
	forloop(int i = 0; device.getNumChannels()) {
		channels[i] = device.getChannel(i);
	}
}

#else

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	setDescription("VprnClient plugin is not loaded");
	
	return 1;
}

#endif

```

## vrpn_client_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>
using Unigine::Samples;

#ifdef HAS_VRPN_CLIENT

#define VRPN_NAME "DTrack@localhost"
#define FLYSTICK_BUTTONS 6
#define FLYSTICK_CHANNELS 2
#define NUM_SENSORS 2
#define RAYCAST_DISTANCE 1000.0f

/*
 */
VrpnTrackerDevice tracker;
VrpnButtonDevice button;
VrpnAnalogDevice analog;

int buttons[FLYSTICK_BUTTONS];
float channels[FLYSTICK_CHANNELS];

Vec3 sensor_positions[NUM_SENSORS];
quat sensor_orientations[NUM_SENSORS];

/*
 */
float rotation_speed = 20.0f;
float movement_speed = 10.0f;

vec4 highlight_color = vec4(1.0f,0.0f,0.0f,1.0f);
vec4 grab_color = vec4(1.0f,1.0f,0.0f,1.0f);

/*
 */
Node body;
Node head;
Node flystick;
Node ray_pivot;
Node ray;
Node highlightable_objects;

Object current_grab = NULL;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// vrpn devices
	tracker = new VrpnTrackerDevice(VRPN_NAME);
	tracker.setTransformCallback("tracker_callback");
	
	button = new VrpnButtonDevice(VRPN_NAME);
	button.setButtonCallback("button_callback");
	
	analog = new VrpnAnalogDevice(VRPN_NAME);
	analog.setAnalogCallback("analog_callback");
	
	// scene
	body = engine.world.getNodeByName("local_center");
	head = engine.world.getNodeByName("head");
	flystick = engine.world.getNodeByName("flystick");
	ray_pivot = engine.world.getNodeByName("ray_pivot");
	ray = engine.world.getNodeByName("ray");
	highlightable_objects = engine.world.getNodeByName("highlightable_objects");
	
	engine.game.setPlayer(node_cast(head));
	
	setDescription("VRPN Client sample");
	
	return 1;
}

/*
 */
int update() {
	
	tracker.update();
	button.update();
	analog.update();
	
	float ifps = engine.game.getIFps();
	
	// joystick inputs
	float channel_rotation = -channels[0] * ifps * rotation_speed;
	float channel_movement = channels[1] * ifps * movement_speed;
	
	// flystick params
	Mat4 flystick_transform = flystick.getWorldTransform();
	Vec3 flystick_forward = flystick_transform.m01m11m21;
	Vec3 flystick_position = flystick.getWorldPosition();
	
	// head params
	Vec3 head_position = head.getWorldPosition() - body.getWorldPosition();
	
	// body params
	quat body_rotation = quat(rotateZ(channel_rotation));
	Mat4 body_transform = translate(-head_position) * body_rotation * translate(head_position);
	
	// flystick
	flystick.setPosition(sensor_positions[1]);
	flystick.setRotation(sensor_orientations[1]);
	
	// head
	head.setPosition(-sensor_positions[0]);
	
	// body
	body.setWorldTransform(body.getWorldTransform() * body_transform);
	body.setPosition(body.getPosition() + flystick_forward * channel_movement);
	
	// selection
	Vec3 p0 = flystick_position;
	Vec3 p1 = flystick_position + flystick_forward * RAYCAST_DISTANCE;
	
	WorldIntersection data = class_manage(new WorldIntersection());
	Object obj = engine.world.getIntersection(p0,p1,~0,(flystick,ray),data); 
	
	// ray
	if(obj != NULL) {
		float ray_scale = length(data.getPoint() - p0);
		ray_pivot.setScale(vec3(1.0f,ray_scale,1.0f));
	}
	
	// highlight
	forloop(int i = 0; highlightable_objects.getNumChildren()) {
		Node child = highlightable_objects.getChild(i);
		if(child.isObject() == 0) continue;
		
		Object child_obj = node_cast(child);
		if(child_obj == current_grab) continue;
		
		if(child_obj == obj) {
			obj.setMaterialParameterFloat4("diffuse_color",highlight_color,data.getSurface());
			continue;
		}
		
		forloop(int j = 0; child_obj.getNumSurfaces()) {
			child_obj.setMaterialParameterFloat4("diffuse_color",vec4_one,j);
		}
	}
	
	// grab
	if(buttons[0] && current_grab == NULL) {
		current_grab = obj;
		
		if(current_grab != NULL) {
			mat4 transform = current_grab.getWorldTransform();
			current_grab.setParent(flystick);
			current_grab.setWorldTransform(transform);
			
			forloop(int j = 0; current_grab.getNumSurfaces()) {
				current_grab.setMaterialParameterFloat4("diffuse_color",grab_color,j);
			}
		}
	}
	
	if(buttons[0] == 0 && current_grab != NULL) {
		mat4 transform = current_grab.getWorldTransform();
		current_grab.setParent(highlightable_objects);
		current_grab.setWorldTransform(transform);
		current_grab = NULL;
	}
	
	return 1;
}

/*
 */
void tracker_callback(int sensor,vec3 position,quat orientation) {
	if(sensor >= NUM_SENSORS) return;
	
	sensor_positions[sensor] = position;
	sensor_orientations[sensor] = orientation;
}

void button_callback(int button,int state) {
	if(button >= FLYSTICK_BUTTONS) return;
	
	buttons[button] = state;
}

void analog_callback(VrpnAnalogDevice device) {
	if(device.getNumChannels() > FLYSTICK_CHANNELS) return;
	
	forloop(int i = 0; device.getNumChannels()) {
		channels[i] = device.getChannel(i);
	}
}

#else

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	setDescription("VprnClient plugin is not loaded");
	
	return 1;
}

#endif

```

## water_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
Body bodies[0];

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	float time = engine.game.getTime();
	float offset = 360.0f / bodies.size();
	
	forloop(int i = 0; bodies.size()) {
		bodies[i].setVelocityTransform(Mat4(rotateZ(time * 30.0f + offset * i) * translate(12.0f,0.0f,0.0f) * rotateZ(180.0f)));
		bodies[i].flushTransform();
	}
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyWater two-way interaction");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int num = 5;
	float offset = 360.0f / num;
	
	forloop(int i = 0; num) {
		
		ObjectMeshSkinned mesh = addToEditor(node_load(full_path("uniginescript_samples/physics/meshes/ragdoll_00.node")));
		mesh.setWorldTransform(Mat4(rotateZ(offset * i) * translate(12.0f,0.0f,0.0f) * rotateZ(180.0f)));
		mesh.setLayerAnimationFilePath(0,full_path("uniginescript_samples/physics/meshes/ragdoll_00.anim"));
		mesh.setTime(2.0f * i);
		mesh.play();
		
		bodies.append(mesh.getBody());
	}
	
	int size = 3;
	float step = 2.0f;
	forloop(int j = -size; size + 1) {
		forloop(int i = -size; size + 1) {
			createBodyBox(vec3(1.0f),20.0f,0.5f,0.5f,get_material(i ^ j),translate(Vec3(i * step,j * step,2.0f)));
		}
	}
	
	ObjectWaterMesh surface = addToEditor(new ObjectWaterMesh(full_path("uniginescript_samples/physics/meshes/water_00.mesh")));
	BodyWater water = class_remove(new BodyWater(surface));
	surface.setWorldTransform(translate(Vec3(0.0f,0.0f,2.0f)));
	surface.setMaterial(findMaterialByName("physics_water"),"*");
	water.setDepth(2.0f);
	water.setDensity(35.0f);
	water.setLiquidity(0.25f);
	water.setLinearDamping(8.0f);
	water.setAngularDamping(8.0f);
	water.setInteractionForce(4.0f);
	
	return 1;
}

```

## water_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyWater two-way interaction");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 7;
	float step = 2.0f;
	forloop(int j = -size; size + 1) {
		forloop(int i = -size; size + 1) {
			createBodyBox(vec3(1.0f),20.0f,0.5f,0.5f,get_material(i ^ j),translate(Vec3(i * step,j * step,4.0f)));
		}
	}
	
	ObjectWaterMesh surface = addToEditor(new ObjectWaterMesh(full_path("uniginescript_samples/physics/meshes/water_00.mesh")));
	BodyWater water = class_remove(new BodyWater(surface));
	surface.setWorldTransform(translate(Vec3(0.0f,0.0f,4.0f)));
	surface.setMaterial(findMaterialByName("physics_water"),"*");
	surface.setWave(0,vec4(0.0f,0.1f,1.0f,2.0f));
	surface.setWave(1,vec4(0.3f,0.2f,0.4f,0.5f));
	water.setAbsorption(1);
	water.setDepth(4.0f);
	water.setDensity(25.0f);
	water.setLiquidity(0.2f);
	water.setLinearDamping(8.0f);
	water.setAngularDamping(8.0f);
	water.setInteractionForce(2.0f);
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## water_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
 int counter = 0;
float step = 2.0f;
float time = step;

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int update() {
	updateSamplePhysics();
	
	if(counter > 32) return 1;
	
	time += engine.game.getIFps() * engine.physics.getScale();
	if(time >= step) {
		float px = engine.game.getRandom(-8.0f,8.0f);
		float py = engine.game.getRandom(-8.0f,8.0f);
		float rx = engine.game.getRandom(-16.0f,16.0f);
		float ry = engine.game.getRandom(-16.0f,16.0f);
		createBodyHomuncle(0,vec3(0.0f,0.0f,-20.0f),material_names,translate(Vec3(px,py,20.0f)) * rotateX(rx) * rotateY(ry) * rotateX(180.0f));
		counter++;
		time -= step;
	}
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyWater two-way interaction");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	ObjectMeshStatic pool = addToEditor(new ObjectMeshStatic(full_path("uniginescript_samples/physics/meshes/water_01_pool.mesh")));
	pool.setWorldTransform(translate(Vec3(0.0f,0.0f,0.0f)));
	pool.setMaterial(findMaterialByName(get_material(0)),"*");
	pool.setSurfaceProperty("surface_base","*");
	pool.setCollision(1,0);
	
	ObjectWaterMesh surface = addToEditor(new ObjectWaterMesh(full_path("uniginescript_samples/physics/meshes/water_01_surface.mesh")));
	BodyWater water = class_remove(new BodyWater(surface));
	surface.setWorldTransform(translate(Vec3(0.0f,0.0f,6.5f)));
	surface.setMaterial(findMaterialByName("physics_water"),"*");
	water.setDepth(6.5f);
	water.setDensity(16.0f);
	water.setLiquidity(0.25f);
	water.setLinearDamping(16.0f);
	water.setAngularDamping(16.0f);
	water.setInteractionForce(2.0f);
	
	ObjectParticles particles = addToEditor(new ObjectParticles());
	particles.setMaterial(findMaterialByName("particles_base"),"*");
	particles.setEmitterType(OBJECT_PARTICLES_EMITTER_SPARK);
	particles.setEmitterEnabled(1);
	particles.setSpawnRate(2.0f);
	particles.setSpawnThreshold(18.0f);
	particles.setLife(1.0f,0.5f);
	
	ParticleModifierScalar radius_modifier = particles.getRadiusOverTimeModifier();
	radius_modifier.setMode(PARTICLE_MODIFIER_MODE_RANDOM_BETWEEN_TWO_CONSTANTS);
	radius_modifier.setConstantMin(0.15f);
	radius_modifier.setConstantMax(0.35f);
	
	ParticleModifierScalar velocity_modifier = particles.getVelocityOverTimeModifier();
	velocity_modifier.setMode(PARTICLE_MODIFIER_MODE_RANDOM_BETWEEN_TWO_CONSTANTS);
	velocity_modifier.setConstantMin(1.5f);
	velocity_modifier.setConstantMax(2.5f);
	
	ParticleModifierVector direction_modifier = particles.getDirectionOverTimeModifier();
	direction_modifier.setMode(PARTICLE_MODIFIER_MODE_RANDOM_BETWEEN_TWO_CONSTANTS);
	direction_modifier.setConstantMin(vec3(-1.0f,-1.0f,0.0f));
	direction_modifier.setConstantMax(vec3(1.0f,1.0f,2.0f));
	
	particles.setParent(surface);
	
	return 1;
}

```

## water_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physics_red", "physics_green", "physics_blue", "physics_orange", "physics_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string full_path(string path) {
	return engine.filesystem.resolvePartialVirtualPath(path);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.loadSettings(full_path("uniginescript_samples/common/world/render.render"));
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createPlaneWithBody();
	setDescription("BodyWater two-way interaction");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	createBodyBox(vec3(1.0f,32.0f,8.0f),0.0f,0.5f,0.5f,get_material(0),translate(Vec3( 16.0f,  0.5f,1.5f)));
	createBodyBox(vec3(1.0f,32.0f,8.0f),0.0f,0.5f,0.5f,get_material(0),translate(Vec3(-16.0f, -0.5f,1.5f)));
	createBodyBox(vec3(32.0f,1.0f,8.0f),0.0f,0.5f,0.5f,get_material(0),translate(Vec3( -0.5f, 16.0f,1.5f)));
	createBodyBox(vec3(32.0f,1.0f,8.0f),0.0f,0.5f,0.5f,get_material(0),translate(Vec3(  0.5f,-16.0f,1.5f)));
	
	int num = 6;
	int size = 8;
	float space = 1.01f;
	float offset = 360.0f / num;
	
	forloop(int i = 0; num) {
		
		Mat4 transform = Mat4(rotateZ(offset * i + offset * 0.5f) * translate(12.0f,0.0f,0.0f) * rotateZ(90.0f));
		
		for(int j = 0; j < size; j++) {
			for(int i = 0; i < size - j; i++) {
				createBodyBox(vec3(1.0f),10.0f,0.5f,0.5f,get_material(i ^ j * 2),transform * translate(vec3(i + j * 0.5f - size * 0.5f + 0.5f,0.0f,j + 0.5f) * space));
			}
		}
	}
	
	ObjectWaterMesh surface = addToEditor(new ObjectWaterMesh(full_path("uniginescript_samples/physics/meshes/water_02.mesh")));
	BodyWater water = class_remove(new BodyWater(surface));
	surface.setWorldTransform(translate(Vec3(0.0f,0.0f,0.5f)));
	surface.setMaterial(findMaterialByName("physics_water"),"*");
	water.setDepth(0.4f);
	water.setDensity(20.0f);
	water.setLiquidity(1.0f);
	water.setLinearDamping(64.0f);
	water.setAngularDamping(64.0f);
	water.setInteractionForce(2.0f);
	
	ObjectParticles particles = addToEditor(new ObjectParticles());
	particles.setMaterial(findMaterialByName("particles_base"),"*");
	particles.setEmitterType(OBJECT_PARTICLES_EMITTER_SPARK);
	particles.setEmitterEnabled(1);
	particles.setSpawnRate(2.0f);
	particles.setSpawnThreshold(8.0f);
	particles.setLife(1.0f,0.5f);
	
	ParticleModifierScalar radius_modifier = particles.getRadiusOverTimeModifier();
	radius_modifier.setMode(PARTICLE_MODIFIER_MODE_RANDOM_BETWEEN_TWO_CONSTANTS);
	radius_modifier.setConstantMin(0.25f);
	radius_modifier.setConstantMax(0.75f);
	
	ParticleModifierScalar velocity_modifier = particles.getVelocityOverTimeModifier();
	velocity_modifier.setMode(PARTICLE_MODIFIER_MODE_RANDOM_BETWEEN_TWO_CONSTANTS);
	velocity_modifier.setConstantMin(1.5f);
	velocity_modifier.setConstantMax(2.5f);
	
	ParticleModifierVector direction_modifier = particles.getDirectionOverTimeModifier();
	direction_modifier.setMode(PARTICLE_MODIFIER_MODE_RANDOM_BETWEEN_TWO_CONSTANTS);
	direction_modifier.setConstantMin(vec3(-1.0f,-1.0f,0.0f));
	direction_modifier.setConstantMax(vec3(1.0f,1.0f,2.0f));
	
	particles.setParent(surface);
	
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## wet_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string phong_mesh_material_names[] = ( "render_phong_mesh_red", "render_phong_mesh_green", "render_phong_mesh_blue", "render_phong_mesh_orange", "render_phong_mesh_yellow" );

string get_phong_mesh_material(int material) {
	return phong_mesh_material_names[abs(material) % phong_mesh_material_names.size()];
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	Player player = createDefaultPlayer(Vec3(39.3f,0.28f,11.6f));
	createDefaultPlane();
	
	setDescription("post_filter_wet material");
	
	player.setDirection(vec3(-0.9f,0.0f,-0.3f));
	
	int size = 2;
	float space = 16.0f;
	
	for(int y = -size; y <= size; y++) {
		for(int x = -size; x <= size; x++) {
			ObjectMeshStatic mesh = addToEditor(new ObjectMeshStatic(fullPath("uniginescript_samples/common/meshes/statue.mesh")));
			mesh.setWorldTransform(translate(Vec3(x,y,0.0f) * space));
			mesh.setMaterial(findMaterialByName(get_phong_mesh_material(x ^ y)),"*");
			mesh.setMaterialState("auxiliary",1,0);
			mesh.setMaterialParameterFloat4("auxiliary_color",vec4(1.0f,0.0f,0.0f,0.0f),0);
			mesh.setSurfaceProperty("surface_base","*");
		}
	}
	
	engine.render.addScriptableMaterial(findMaterialByName("render_filter_wet"));
	
	return 1;
}

```

## wheel_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "joints_red", "joints_green", "joints_blue", "joints_orange", "joints_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
int create_body(Mat4 transform) {
	
	int num = 0;
	
	void create_joint(Body b0,Body b1) {
		JointWheel j = new JointWheel(b0,b1);
		j.setWorldAxis0(rotation(transform) * vec3(0.0f,0.0f,1.0f));
		j.setWorldAxis1(rotation(transform) * vec3(0.0f,1.0f,0.0f));
		j.setLinearSpring(50.0f);
		j.setLinearDamping(8.0f);
		j.setLinearLimitFrom(-0.5f);
		j.setLinearLimitTo(0.0f);
		j.setAngularVelocity(-20.0f);
		j.setAngularTorque(10.0f);
		j.setNumIterations(2);
		j.setWheelRadius(0.5f);
		num++;
	}
	
	Body body = createBodyBox(vec3(3.0f,2.0f,0.75f),1.0f,0.5f,0.5f,get_material(0),transform);
	
	Body b0 = createBodySphere(0.5f,1.0f,1.0f,0.5f,get_material(1),transform * translate( 1.25f, 1.25f,-0.5f));
	Body b1 = createBodySphere(0.5f,1.0f,1.0f,0.5f,get_material(1),transform * translate( 1.25f,-1.25f,-0.5f));
	Body b2 = createBodySphere(0.5f,1.0f,1.0f,0.5f,get_material(1),transform * translate( 0.00f, 1.25f,-0.5f));
	Body b3 = createBodySphere(0.5f,1.0f,1.0f,0.5f,get_material(1),transform * translate( 0.00f,-1.25f,-0.5f));
	Body b4 = createBodySphere(0.5f,1.0f,1.0f,0.5f,get_material(1),transform * translate(-1.25f, 1.25f,-0.5f));
	Body b5 = createBodySphere(0.5f,1.0f,1.0f,0.5f,get_material(1),transform * translate(-1.25f,-1.25f,-0.5f));
	
	create_joint(body,b0);
	create_joint(body,b1);
	create_joint(body,b2);
	create_joint(body,b3);
	create_joint(body,b4);
	create_joint(body,b5);
	
	return num;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	
	int num = 0;
	
	int size = 8;
	
	createBodyBox(vec3(50.0f,100.0f,2.0f),0.0f,0.5f,0.5f,get_material(2),translate(Vec3(-40.0f,0.0f,7.0f)) * rotateY(20.0f));
	createBodyBox(vec3(50.0f,100.0f,2.0f),0.0f,0.5f,0.5f,get_material(2),translate(Vec3( 40.0f,0.0f,7.0f)) * rotateY(-20.0f));
	
	for(int i = -size; i <= size; i++) {
		num += create_body(translate(Vec3(-60.0f,i * 4.0f,18.0f)));
		num += create_body(translate(Vec3(-40.0f,i * 4.0f,10.0f)));
		num += create_body(translate(Vec3(-20.0f,i * 4.0f, 3.0f)));
		num += create_body(translate(Vec3( 20.0f,i * 4.0f, 3.0f)) * rotateZ(180.0f));
		num += create_body(translate(Vec3( 40.0f,i * 4.0f,10.0f)) * rotateZ(180.0f));
		num += create_body(translate(Vec3( 60.0f,i * 4.0f,18.0f)) * rotateZ(180.0f));
	}
	
	setDescription(format("%d JointWheel",num));
	return 1;
}

/*
 */
int update() {
	updateSamplePhysics();
	return 1;
}

```

## widgets_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

#include <core/systems/widgets/widget.h>
#include <core/systems/widgets/widget_vbox.h>
#include <core/systems/widgets/widget_hbox.h>
#include <core/systems/widgets/widget_vpaned.h>
#include <core/systems/widgets/widget_hpaned.h>
#include <core/systems/widgets/widget_tabbox.h>
#include <core/systems/widgets/widget_gridbox.h>
#include <core/systems/widgets/widget_scrollbox.h>
#include <core/systems/widgets/widget_window.h>
#include <core/systems/widgets/widget_icon.h>
#include <core/systems/widgets/widget_label.h>
#include <core/systems/widgets/widget_button.h>
#include <core/systems/widgets/widget_menubox.h>
#include <core/systems/widgets/widget_menubar.h>
#include <core/systems/widgets/widget_checkbox.h>
#include <core/systems/widgets/widget_combobox.h>
#include <core/systems/widgets/widget_sprite.h>
#include <core/systems/widgets/widget_canvas.h>
#include <core/systems/widgets/widget_slider.h>
#include <core/systems/widgets/widget_listbox.h>
#include <core/systems/widgets/widget_treebox.h>
#include <core/systems/widgets/widget_editline.h>
#include <core/systems/widgets/widget_edittext.h>

Unigine::Widgets::Window window;
Unigine::Widgets::Icon close_icon;

using Unigine::Samples;

void on_close()
{
	Unigine::Widgets::removeChild(window);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	engine.render.setShadowDistance(100.0f);
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	setDescription("Unigine::Widgets::Widgets");
	
	using Unigine::Widgets;
	
	// window
	window = new Window("Unigine::Widgets::Window",4,4);
	window.setWidth(640);
	window.setHeight(480);
	window.setSizeable(1);
	window.setDragAreaPadding(3, 25, 3, 0);
	
	// close icon
	close_icon = new Icon();
	close_icon.setPosition(10, -24);
	close_icon.getEventClicked().connect(functionid(on_close));
	window.addChild(close_icon, GUI_ALIGN_OVERLAP | GUI_ALIGN_TOP | GUI_ALIGN_RIGHT);

	Image img = new Image("core/gui/window_close.png");
	close_icon.setImage(img);

	// menubar
	MenuBar menubar = new MenuBar(8,2);
	window.addChild(menubar,ALIGN_LEFT);
	
	// menubox
	MenuBox menubox = new MenuBox(16,8);
	menubox.addItem("Open");
	menubox.setItemSeparator(menubox.addItem("Save"),1);
	int item = menubox.addItem(NULL);
	menubox.setItemWidget(item,new CheckBox("Check"));
	menubox.setItemSeparator(item,1);
	menubox.addItem("Exit");
	menubar.addItem("File",menubox);
	
	// tabbox
	TabBox tabbox = new TabBox(4,4);
	tabbox.setTexture(fullPath("uniginescript_samples/interface/images/images.png"));
	window.addChild(tabbox,ALIGN_EXPAND);
	
	/////////////////////////////////
	// Widgets
	/////////////////////////////////
	
	{
		tabbox.addTab("Widgets",tabbox.getNumTabs());
		VBox vbox = new VBox();
		tabbox.addChild(vbox);
		
		GridBox gridbox = new GridBox(2,4,4);
		vbox.addChild(gridbox,ALIGN_EXPAND);
		
		// label
		Label label = new Label("This is <b>Rich</b>&nbsp;<i>Label</i>");
		gridbox.addChild(label,ALIGN_LEFT);
		label.setFontRich(1);
		label.setWidth(120);
		
		// icon
		gridbox.addChild(new Icon(fullPath("uniginescript_samples/interface/images/icon.png")),ALIGN_EXPAND);
		
		// button
		gridbox.addChild(new Button("Button"),ALIGN_EXPAND);
		
		// checkbox
		gridbox.addChild(new CheckBox("CheckBox"),ALIGN_EXPAND);
		
		// slider
		Slider slider = new Slider();
		slider.addAttach(label,"Label %d");
		gridbox.addChild(slider,ALIGN_EXPAND);
		
		// combobox
		ComboBox combobox = new ComboBox();
		combobox.setTexture(fullPath("uniginescript_samples/interface/images/images.png"));
		combobox.addItem("First",0);
		combobox.addItem("Second",1);
		combobox.addItem("Third",2);
		combobox.addItem("Fourth",3);
		gridbox.addChild(combobox,ALIGN_EXPAND);
	}
	
	/////////////////////////////////
	// Paned
	/////////////////////////////////
	
	{
		tabbox.addTab("Paned",tabbox.getNumTabs());
		HPaned hpaned = new HPaned();
		tabbox.addChild(hpaned,ALIGN_EXPAND);
		hpaned.setFixed(1);
		
		VPaned vpaned = new VPaned();
		hpaned.addChild(vpaned,ALIGN_EXPAND);
		
		vpaned.addChild(new EditText("EditText 0"),ALIGN_EXPAND);
		vpaned.addChild(new EditText("EditText 1"),ALIGN_EXPAND);
		vpaned.setFixed(1);
		
		vpaned = new VPaned();
		hpaned.addChild(vpaned,ALIGN_EXPAND);
		
		vpaned.addChild(new EditText("EditText 2"),ALIGN_EXPAND);
		vpaned.addChild(new EditText("EditText 3"),ALIGN_EXPAND);
		vpaned.setFixed(2);
	}
	
	/////////////////////////////////
	// Dynamic
	/////////////////////////////////
	
	{
		tabbox.addTab("Dynamic",tabbox.getNumTabs());
		HBox hbox = new HBox(4,4);
		tabbox.addChild(hbox);
		
		// sprite
		Sprite sprite = new Sprite(fullPath("uniginescript_samples/interface/images/sprite.png"));
		hbox.addChild(sprite);
		
		// canvas
		Canvas canvas = new Canvas();
		canvas.setWidth(256);
		canvas.setHeight(256);
		canvas.setColor(vec4(0.0f,0.0f,0.0f,1.0f));
		
		int id = canvas.addText();
		canvas.setTextPosition(id,vec3(8.0f,160.0f,0.0f));
		canvas.setTextColor(id,vec4(0.0f,0.0f,1.0f,1.0f));
		canvas.setTextText(id,"Canvas");
		canvas.setTextSize(id,64);
		
		id = canvas.addLine();
		canvas.setLineColor(id,vec4(1.0f,0.0f,0.0f,1.0f));
		canvas.setLineTransform(id,translate(64.0f,64.0f,0.0f));
		forloop(int i = 0; 8) {
			float angle = PI2 * i * 3 / 7;
			canvas.addLinePoint(id,vec3(sin(angle),cos(angle),0.0f) * 64.0f);
		}
		
		id = canvas.addPolygon();
		canvas.setPolygonColor(id,vec4(0.0f,1.0f,0.0f,1.0f));
		canvas.setPolygonTransform(id,translate(192.0f,64.0f,0.0f));
		forloop(int i = 0; 7) {
			float angle = PI2 * i / 6;
			canvas.addPolygonPoint(id,vec3(sin(angle),cos(angle),0.0f) * 64.0f);
		}
		
		hbox.addChild(canvas);
	}
	
	/////////////////////////////////
	// Lists
	/////////////////////////////////
	
	{
		tabbox.addTab("Lists",tabbox.getNumTabs());
		GridBox gridbox = new GridBox(2,4,4);
		tabbox.addChild(gridbox,ALIGN_EXPAND);
		
		gridbox.addChild(new Label("Single selection:"));
		gridbox.addChild(new Label("Multiple selection:"));
		
		// listbox
		ListBox listbox = new ListBox();
		listbox.setTexture(fullPath("uniginescript_samples/interface/images/images.png"));
		forloop(int i = 0; 8) {
			listbox.addItem(format("Item %d",i),i);
		}
		ScrollBox scrollbox = new ScrollBox();
		gridbox.addChild(scrollbox,ALIGN_EXPAND);
		scrollbox.addChild(listbox,ALIGN_EXPAND);
		
		// multiselection listbox
		listbox = new ListBox();
		listbox.setMultiSelection(1);
		listbox.setTexture(fullPath("uniginescript_samples/interface/images/images.png"));
		forloop(int i = 0; 8) {
			listbox.addItem(format("Item %d",i),i);
		}
		scrollbox = new ScrollBox();
		gridbox.addChild(scrollbox,ALIGN_EXPAND);
		scrollbox.addChild(listbox,ALIGN_EXPAND);
		
		gridbox.addChild(new Label("Single selection:"));
		gridbox.addChild(new Label("Multiple selection:"));
		
		// treebox
		TreeBox treebox = new TreeBox();
		treebox.setTexture(fullPath("uniginescript_samples/interface/images/images.png"));
		treebox.setEditable(1);
		forloop(int i = 0; 8) {
			treebox.addItem(format("Item %d",i),i);
		}
		scrollbox = new ScrollBox();
		gridbox.addChild(scrollbox,ALIGN_EXPAND);
		scrollbox.addChild(treebox,ALIGN_EXPAND);
		
		// multiselection treebox
		treebox = new TreeBox();
		treebox.setTexture(fullPath("uniginescript_samples/interface/images/images.png"));
		treebox.setEditable(1);
		treebox.setMultiSelection(1);
		forloop(int i = 0; 8) {
			treebox.addItem(format("Item %d",i),i);
		}
		scrollbox = new ScrollBox();
		gridbox.addChild(scrollbox,ALIGN_EXPAND);
		scrollbox.addChild(treebox,ALIGN_EXPAND);
	}
	
	/////////////////////////////////
	// Edits
	/////////////////////////////////
	
	{
		tabbox.addTab("Edits",tabbox.getNumTabs());
		VBox vbox = new VBox(0,4);
		tabbox.addChild(vbox);
		
		// editline
		vbox.addChild(new EditLine("EditLine 0"),ALIGN_EXPAND);
		vbox.addChild(new EditLine("EditLine 1"),ALIGN_EXPAND);
		vbox.addChild(new EditLine("EditLine 2"),ALIGN_EXPAND);
		vbox.addChild(new EditLine("EditLine 3"),ALIGN_EXPAND);
		
		// edittext
		tabbox.addChild(new EditText("EditText"),ALIGN_EXPAND);
	}
	
	// set first tab
	tabbox.setCurrentTab(0);
	
	// window
	window.arrange();
	addChild(window,ALIGN_OVERLAP | ALIGN_CENTER);
	
	return 1;
}

```

## wind_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physicals_red", "physicals_green", "physicals_blue", "physicals_orange", "physicals_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string cloth_material_names[] = ( "physicals_cloth_red", "physicals_cloth_green", "physicals_cloth_blue", "physicals_cloth_orange", "physicals_cloth_yellow" );

string get_cloth_material(int material) {
	return cloth_material_names[abs(material) % cloth_material_names.size()];
}

/*
 */
int update() {
	updateSamplePhysics();
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(48.0f,0.0f,48.0f));
	createDefaultPlane();
	setDescription("BodyRigid and BodyCloth in PhysicalWind");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 4;
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			
			float x = engine.game.getRandom(-8.0f,8.0f);
			float y = engine.game.getRandom(-8.0f,8.0f);
			Mat4 transform = translate(Vec3(i,j,3.0f) * 8.0f) * rotateX(x) * rotateY(y);
			
			if((i + j) % 2) {
				createBodyBox(vec3(4.0f,4.0f,0.2f),1.0f,0.5f,0.5f,get_material(i ^ j),transform);
			} else {
				BodyCloth cloth = createBodyCloth(4.0f,4.0f,0.5f,4.0f,0.5f,0.5f,get_cloth_material(i ^ j),transform);
				cloth.setLinearRestitution(0.8f);
				cloth.setAngularRestitution(0.2f);
				cloth.setRadius(0.5f);
			}
		}
	}
	
	PhysicalWind wind = addToEditor(new PhysicalWind(vec3(1000.0f)));
	wind.setVelocity(vec3(0.0f,0.0f,1.0f));
	wind.setLinearDamping(8.0f);
	wind.setAngularDamping(8.0f);
	
	return 1;
}

```

## wind_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physicals_red", "physicals_green", "physicals_blue", "physicals_orange", "physicals_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
Node nodes[0];

/*
 */
Body create_fan(float angle,Mat4 transform) {
	
	Body create_blade(Mat4 transform) {
		return createBodyBox(vec3(4.0f,1.5f,0.02f),0.1f,0.5f,0.5f,get_material(1),transform);
	}
	
	void create_joint(Body b0,Body b1) {
		JointFixed j = class_remove(new JointFixed(b0,b1,b1.getTransform() * Vec3_zero));
		j.setLinearRestitution(0.8f);
		j.setAngularRestitution(0.8f);
		j.setNumIterations(16);
		j.setMaxForce(600.0f);
	}
	
	Body hub = createBodyBox(vec3(1.0f,1.0f,0.5f),4.0f,0.5f,0.5f,get_material(0),transform);
	
	ObjectParticles particles = addToEditor(new ObjectParticles());
	particles.setMaterial(findMaterialByName("particles_base"),"*");
	particles.setDepthSort(1);
	particles.setEmitterEnabled(1);
	particles.setSpawnRate(40.0f);
	particles.setParent(hub.getObject());
	particles.setLife(3.0f,1.0f);
	
	ParticleModifierScalar linear_damping_modifier = particles.getLinearDampingOverTimeModifier();
	linear_damping_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
	linear_damping_modifier.setConstant(0.5f);

	ParticleModifierVector gravity_modifier = particles.getGravityOverTimeModifier();
	gravity_modifier.setMode(PARTICLE_MODIFIER_MODE_CONSTANT);
	gravity_modifier.setConstant(vec3(0.0f));
	
	particles.setMaterialParameterFloat4("albedo_color",vec4(0.3f,0.3f,0.3f,1.0f),0);
	
	ParticleModifierScalar growth_modifier = particles.getGrowthOverTimeModifier();
	growth_modifier.setMode(PARTICLE_MODIFIER_MODE_RANDOM_BETWEEN_TWO_CONSTANTS);
	growth_modifier.setConstantMin(0.5f);
	growth_modifier.setConstantMax(1.5f);
	
	nodes.append(particles);
	
	Body b0 = create_blade(transform * rotateZ(  0.0f) * translate(2.0f,0.0f,0.5f) * rotateX(angle));
	Body b1 = create_blade(transform * rotateZ( 90.0f) * translate(2.0f,0.0f,0.5f) * rotateX(angle));
	Body b2 = create_blade(transform * rotateZ(180.0f) * translate(2.0f,0.0f,0.5f) * rotateX(angle));
	Body b3 = create_blade(transform * rotateZ(270.0f) * translate(2.0f,0.0f,0.5f) * rotateX(angle));
	
	create_joint(hub,b0);
	create_joint(hub,b1);
	create_joint(hub,b2);
	create_joint(hub,b3);
	
	return hub;
}

/*
 */
void create_wing(Body body,float width,float height,string material,int mask,Mat4 transform) {
	
	Body wing = createBodyBox(vec3(height,width,0.05f),1.0f,0.5f,0.5f,material,transform);
	Shape shape = wing.getShape(0);
	shape.setCollisionMask(mask);
	
	Joint joint = class_remove(new JointFixed(body,wing,transform * vec3_zero));
	joint.setLinearRestitution(0.4f);
	joint.setAngularRestitution(0.05f);
	joint.setAngularSoftness(0.01f);
	joint.setMaxForce(20000.0f);
	joint.setNumIterations(4);
}

/*
 */
Body create_plane(string material,Mat4 transform) {
	
	void create_motor(Body b0,Body b1,float velocity) {
		JointHinge j = class_remove(new JointHinge(b0,b1));
		j.setWorldAnchor(b1.getTransform() * Vec3_zero);
		j.setWorldAxis(rotation(transform) * vec3(1.0f,0.0f,0.0f));
		j.setAngularVelocity(velocity);
		j.setAngularTorque(40.0f);
		j.setNumIterations(4);
	}
	
	void create_suspension(Body b0,Body b1) {
		JointSuspension j = new JointSuspension(b0,b1);
		j.setWorldAxis0(rotation(transform) * vec3(0.0f,0.0f,1.0f));
		j.setWorldAxis1(rotation(transform) * vec3(0.0f,1.0f,0.0f));
		j.setLinearLimitFrom(-0.5f);
		j.setLinearLimitTo(0.0f);
		j.setLinearSpring(100.0f);
		j.setLinearDamping(1.0f);
		j.setNumIterations(4);
	}
	
	Body fuselage = createBodyBox(vec3(25.0f,1.0f,1.0f),0.5f,0.5f,0.5f,get_material(3),transform * translate(7.5f,0.0f,0.0f));
	
	create_wing(fuselage,45.0f,6.0f,material,1,transform * translate( 0.0f,0.0f,0.5f) * rotateY(12.0f));
	create_wing(fuselage,45.0f,6.0f,material,1,transform * translate( 0.0f,0.0f,6.0f) * rotateY(12.0f));
	create_wing(fuselage,15.0f,6.0f,material,1,transform * translate(17.0f,0.0f,0.5f) * rotateY(10.0f));
	create_wing(fuselage,6.0f, 6.0f,material,0,transform * translate( 0.0f,0.0f,3.0f) * rotateX(90.0f));
	create_wing(fuselage,8.0f, 6.0f,material,1,transform * translate(17.0f,0.0f,4.0f) * rotateX(90.0f));
	
	Body f0 = create_fan(-16.0f,transform * translate(-3.0f,-5.5f,2.0f) * rotateY(-90.0f));
	Body f1 = create_fan( 16.0f,transform * translate(-3.0f, 5.5f,2.0f) * rotateY(-90.0f));
	
	create_motor(fuselage,f0,-32.0f);
	create_motor(fuselage,f1, 32.0f);
	
	Body s0 = createBodySphere(1.0f,0.5f,1.0f,0.5f,get_material(4),transform * translate(-2.0f, 4.0f,-2.0f));
	Body s1 = createBodySphere(1.0f,0.5f,1.0f,0.5f,get_material(4),transform * translate(-2.0f,-4.0f,-2.0f));
	Body s2 = createBodySphere(0.5f,2.0f,1.0f,0.5f,get_material(4),transform * translate(19.0f, 0.0f,-1.0f));
	
	create_suspension(fuselage,s0);
	create_suspension(fuselage,s1);
	create_suspension(fuselage,s2);
	
	return fuselage;
}

/*
 */
int update() {
	updateSamplePhysics();
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("BodyRigid in PhysicalWind");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int num = 6;
	float distance = 350.0f;
	
	Body plane;
	for(int i = -num; i <= num; i++) {
		create_plane(get_material(i),translate(Vec3(-distance,50.0f * i,3.0f)) * rotateZ(180.0f));
		Body p = create_plane(get_material(i),translate(Vec3(distance,50.0f * i,3.0f)) * rotateZ(0.0f));
		if(i == 0) plane = p;
	}
	
	PlayerPersecutor persecutor = new PlayerPersecutor();
	persecutor.setFixed(1);
	persecutor.setTarget(plane.getObject());
	persecutor.setMinDistance(80.0f);
	persecutor.setMaxDistance(80.0f);
	persecutor.setPosition(plane.getTransform() * Vec3(20.0f,0.0f,10.0f));
	engine.game.setPlayer(persecutor);
	
	PhysicalWind wind = addToEditor(new PhysicalWind(vec3(10000.0f)));
	wind.setLinearDamping(0.16f);
	wind.setAngularDamping(0.16f);
	
	return 1;
}

```

## wind_02.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physicals_red", "physicals_green", "physicals_blue", "physicals_orange", "physicals_yellow" );

/*
 */
string cloth_material_names[] = ( "physicals_cloth_red", "physicals_cloth_green", "physicals_cloth_blue", "physicals_cloth_orange", "physicals_cloth_yellow" );

string get_cloth_material(int material) {
	return cloth_material_names[abs(material) % cloth_material_names.size()];
}

/*
 */
int update() {
	updateSamplePhysics();
	
	return 1;
}

/*
 */
void create_body_homuncle(int material,Mat4 transform) {
	
	Node homuncle = createBodyHomuncle(0,vec3_zero,material_names,transform);
	Body body = Object(node_cast(homuncle.getChild(0))).getBody();
	
	if(material != -1) {
		
		ObjectMeshDynamic mesh = addToEditor(new ObjectMeshDynamic("uniginescript_samples/physicals/meshes/wind_02.mesh"));
		mesh.setMaterial(findMaterialByName(get_cloth_material(material)),"*");
		mesh.setWorldTransform(transform * translate(-0.2f,0.0f,3.1f));
		
		BodyCloth cloth = class_remove(new BodyCloth(mesh));
		cloth.setMass(10.0f);
		cloth.setNumIterations(2);
		cloth.setLinearRestitution(1.0f);
		cloth.setAngularRestitution(0.05f);
		
		class_remove(new JointParticles(body,cloth,transform * Vec3(-0.2f,0.0f,3.1f),vec3(1.0f)));
	}
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(64.0f,0.0f,48.0f));
	createDefaultPlane();
	setDescription("BodyCloth in PhysicalWind");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int num = 4;
	
	for(int i = 0; i <= num; i++) {
		create_body_homuncle(i,translate(Vec3(  0.0f,20.0f * i - 40.0f,50.0f)));
		create_body_homuncle(i,translate(Vec3(-20.0f,20.0f * i - 40.0f,50.0f)));
	}
	
	for(int i = 0; i < num; i++) {
		create_body_homuncle(-1,translate(Vec3(  0.0f,20.0f * i - 30.0f,50.0f)));
		create_body_homuncle(-1,translate(Vec3(-20.0f,20.0f * i - 30.0f,50.0f)));
	}
	
	PhysicalWind wind = addToEditor(new PhysicalWind(vec3(10000.0f)));
	wind.setLinearDamping(1.0f);
	wind.setAngularDamping(1.0f);
	
	return 1;
}

```

## wind_03.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physicals_red", "physicals_green", "physicals_blue", "physicals_orange", "physicals_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string cloth_material_names[] = ( "physicals_cloth_red", "physicals_cloth_green", "physicals_cloth_blue", "physicals_cloth_orange", "physicals_cloth_yellow" );

string get_cloth_material(int material) {
	return cloth_material_names[abs(material) % cloth_material_names.size()];
}

/*
 */
int update() {
	updateSamplePhysics();
	
	return 1;
}

/*
 */
void create_flag(int material,Mat4 transform) {
	
	Body body = createBodyBox(vec3(0.25f,0.25f,8.0f),0.0f,0.5f,0.5f,get_material(2),transform * translate(0.0f,0.0f,4.0f));
	
	BodyCloth flag = createBodyCloth(3.0f,4.0f,0.4f,1.0f,0.5f,0.5f,get_cloth_material(material),transform * translate(0.0f,2.0f,6.0f) * rotateY(90.0f));
	flag.setCollision(0);
	flag.setLinearRestitution(1.0f);
	flag.setAngularRestitution(0.1f);
	
	class_remove(new JointParticles(body,flag,transform * Vec3(0.0f,0.0f,6.0f),vec3(0.1f,0.1f,6.0f)));
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	createDefaultPlane();
	setDescription("BodyCloth in PhysicalWind");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int size = 2;
	
	for(int j = -size; j <= size; j++) {
		for(int i = -size; i <= size; i++) {
			create_flag(i ^ j,translate(Vec3(i * 6.0f,j * 6.0f,0.0f)));
		}
	}
	
	PhysicalWind wind = addToEditor(new PhysicalWind(vec3(10000.0f)));
	wind.setVelocity(vec3(2.0f,4.0f,2.0f));
	wind.setLinearDamping(1.0f);
	wind.setAngularDamping(1.0f);
	
	return 1;
}

```

## wind_04.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
string material_names[] = ( "physicals_red", "physicals_green", "physicals_blue", "physicals_orange", "physicals_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string volume_material_names[] = ( "physicals_volume_red", "physicals_volume_green", "physicals_volume_blue", "physicals_volume_orange", "physicals_volume_yellow" );

string get_volume_material(int material) {
	return volume_material_names[abs(material) % volume_material_names.size()];
}

/*
 */
int update() {
	updateSamplePhysics();
	
	return 1;
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(80.0f,0.0f,40.0f));
	createDefaultPlane();
	setDescription("PhysicalWind super damping");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	int num = 200;
	
	forloop(int i = 0; num) {
		float tx = engine.game.getRandom(-40.0f,40.0f);
		float ty = engine.game.getRandom(-40.0f,40.0f);
		float tz = engine.game.getRandom( 40.0f,50.0f);
		float rx = engine.game.getRandom(-16.0f,16.0f);
		float ry = engine.game.getRandom(-16.0f,16.0f);
		createBodyBox(engine.game.getRandom(vec3(0.25f),vec3(4.0f)),10.0f,0.5f,0.5f,get_material(i),translate(Vec3(tx,ty,tz)) * rotateX(rx) * rotateY(ry));
	}
	
	ObjectVolumeBox volume = addToEditor(new ObjectVolumeBox(vec3(80.0f,80.0f,20.0f)));
	volume.setWorldTransform(translate(Vec3(0.0f,0.0f,10.0f)));
	volume.setMaterial(findMaterialByName(get_volume_material(1)),"*");
	
	PhysicalWind wind = addToEditor(new PhysicalWind(vec3(80.0f,80.0f,20.0f)));
	wind.setWorldTransform(translate(Vec3(0.0f,0.0f,10.0f)));
	wind.setLinearDamping(500.0f);
	wind.setAngularDamping(500.0f);
	
	return 1;
}

```

## wind_05.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

/*
 */
int counter = 0;
float step = 1.0f;
float time = step;

using Unigine::Samples;

/*
 */
string material_names[] = ( "physicals_red", "physicals_green", "physicals_blue", "physicals_orange", "physicals_yellow" );

string get_material(int material) {
	return material_names[abs(material) % material_names.size()];
}

/*
 */
string volume_material_names[] = ( "physicals_volume_red", "physicals_volume_green", "physicals_volume_blue", "physicals_volume_orange", "physicals_volume_yellow" );

string get_volume_material(int material) {
	return volume_material_names[abs(material) % volume_material_names.size()];
}

/*
 */
int update() {
	updateSamplePhysics();
	
	return 1;
}

/*
 */
void create_wind(vec3 size,int material,Mat4 transform) {
	
	ObjectVolumeBox volume = addToEditor(new ObjectVolumeBox(size));
	volume.setMaterial(findMaterialByName(get_volume_material(material)),"*");
	volume.setWorldTransform(transform);
	
	PhysicalWind wind = addToEditor(new PhysicalWind(size));
	wind.setWorldTransform(transform);
	wind.setThreshold(vec3(2.0f,2.0f,0.0f));
	wind.setVelocity(vec3(20.0f,0.0f,5.0f));
	wind.setLinearDamping(20.0f);
	wind.setAngularDamping(20.0f);
}

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	createDefaultPlayer(Vec3(60.0f,20.0f,25.0f));
	createDefaultPlane();
	setDescription("PhysicalWind velocity");
	
	// physics parameters
	engine.physics.setGravity(vec3(0.0f,0.0f,-9.8f * 2.0f));
	engine.physics.setFrozenLinearVelocity(0.1f);
	engine.physics.setFrozenAngularVelocity(0.1f);
	
	create_wind(vec3(40.0f,10.0f,15.0f),0,translate(Vec3(  0.0f,  0.0f,7.5f)) * rotateZ(  0.0f));
	create_wind(vec3(40.0f,10.0f,15.0f),1,translate(Vec3( 25.0f, 15.0f,7.5f)) * rotateZ( 90.0f));
	create_wind(vec3(40.0f,10.0f,15.0f),2,translate(Vec3( 10.0f, 40.0f,7.5f)) * rotateZ(180.0f));
	create_wind(vec3(40.0f,10.0f,15.0f),3,translate(Vec3(-15.0f, 25.0f,7.5f)) * rotateZ(270.0f));
	
	thread([]() {
		while(1) {
			time += engine.game.getIFps() * engine.physics.getScale();
			if(time >= step) {
				createBodyHomuncle(0,vec3(0.0f),material_names,translate(Vec3(25.0f,15.0f,25.0f)) * rotateX(180.0f));
				if(counter++ > 30) break;
				time -= step;
			}
			wait;
		}
	});
	
	return 1;
}

```

## window_00.cpp

```cpp
#include <core/unigine.h>
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
LightWorld light_world_0;
LightWorld light_world_1;

/*
 */
class Window {
	
	Gui gui;				// gui
	
	WidgetWindow window;	// window
	WidgetLabel label;		// label
	
	WidgetButton button;	// button
	
	// constructor/destructor
	Window(int x,int y) {
		
		gui = engine.getGui();
		
		// window
		window = new WidgetWindow(gui,"Window");
		
		string text = "Jack and Jill\n"
			"Went up the hill "
			"To fetch a pail of water. "
			"Jack fell down "
			"And broke his crown "
			"And Jill came tumbling after. "
			"Up Jack got "
			"And home did trot "
			"As fast as he could caper "
			"Went to bed "
			"And plastered his head "
			"With vinegar and brown paper.\n ";
		
		// label
		label = new WidgetLabel(gui,text);
		window.addChild(label,GUI_ALIGN_EXPAND);
		label.setFontOutline(1);
		label.setFontSize(18);
		label.setFontWrap(1);
		label.setWidth(300);
		
		// button
		button = new WidgetButton(gui,"Press Me");
		window.addChild(button);
		
		window.arrange();
		window.setSizeable(1);
		window.setPosition(x,y);
		gui.addChild(window,GUI_ALIGN_OVERLAP);
	}
	~Window() {
		delete window;
		delete label;
	}
	
	// update
	void update() {
		
		EngineWindow main_window = engine.window_manager.getMainWindow();
		ivec2 window_size = main_window.getSize();
		ivec2 mouse_coord = engine.input.getMousePosition() - main_window.getPosition();
		
		float fov = 8.0f;
		float hwidth = window.getWidth() / 2.0f;
		float hheight = window.getHeight() / 2.0f;
		float y_angle = (float(mouse_coord.x) - window.getPositionX() + hwidth) / window_size.x;
		float x_angle = (float(mouse_coord.y) - window.getPositionY() + hheight) / window_size.y;
		window.setTransform(translate(hwidth,hheight,0.0f) * perspective(fov,1.0f,0.01f,100.0f) * rotateY(y_angle) * rotateX(-x_angle) * translate(-hwidth,-hheight,-1.0f / tan(fov * DEG2RAD * 0.5f)));
	}
	
	// save/restore state
	void __save__(Stream stream) {
		stream.writeInt(window.getPositionX());
		stream.writeInt(window.getPositionY());
	}
	void __restore__(Stream stream) {
		int x = stream.readInt();
		int y = stream.readInt();
		__Window__(x,y);
	}
};

Window windows[0];

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// render parameters
	engine.render.setShadowDistance(100.0f);
	
	// create scene
	createDefaultPlayer(Vec3(30.0f,0.0f,20.0f));
	
	// world lights
	light_world_0 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_0.setWorldTransform(Mat4(rotateZ(45.0f) * rotateY(45.0f)));
	
	light_world_1 = addToEditor(new LightWorld(vec4(0.5f,0.5f,0.5f,1.0f)));
	light_world_1.setWorldTransform(Mat4(rotateZ(-45.0f) * rotateY(45.0f)));
	
	// create sample
	windows.append(new Window(128,128));
	windows.append(new Window(512,128));
	windows.append(new Window(128,384));
	windows.append(new Window(512,384));
	
	setDescription("Window transformation");
	
	return 1;
}

/*
 */
int update() {
	
	foreach(Window window; windows) {
		window.update();
	}
	
	return 1;
}

```

## world_spline_graph_00.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
WorldSplineGraph node;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(-92.0f,94.0f,190.0f));
	
	ObjectMeshDynamic plane = addToEditor(Unigine::createPlane(10000.0f, 10000.0f, 10000.0f));
	plane.setWorldTransform(Mat4_identity);
	plane.setMaterial(findMaterialByName("grid_ground"), "*");
	plane.setSurfaceProperty("surface_base", "*");
	
	// create sample
	node = addToEditor(new WorldSplineGraph());

	// load spline data
	node.load(fullPath("uniginescript_samples/worlds/splines/spline.spl"));

	// assign all segments to geometry from file
	// SEGMENT_STRETCH - for stretching single node by segment
	// SEGMENT_TILING - for tiling nodes above segment
	string geometry_name = fullPath("uniginescript_samples/worlds/nodes/road_hi_poly.node");
	SplineSegment segments[0];
	node.getSplineSegments(segments);
	forloop (int i = 0; segments.size())
	{
		segments[i].assignSource(geometry_name, FORWARD_X);
		segments[i].setSegmentMode(geometry_name, SEGMENT_STRETCH);
	}
	
	setDescription("WorldSplineGraph: stretching geometry");
	
	return 1;
}

```

## world_spline_graph_01.cpp

```cpp
#include <uniginescript_samples/uniginescript_samples.h>

using Unigine::Samples;

/*
 */
WorldSplineGraph node;

/*
 */
int init() {
	
	createInterface(engine.world.getPath());
	
	// create scene
	createDefaultPlayer(Vec3(-92.0f,94.0f,190.0f));
	
	ObjectMeshDynamic plane = addToEditor(Unigine::createPlane(10000.0f, 10000.0f, 10000.0f));
	plane.setWorldTransform(Mat4_identity);
	plane.setMaterial(findMaterialByName("grid_ground"), "*");
	plane.setSurfaceProperty("surface_base", "*");
	
	// create sample
	node = addToEditor(new WorldSplineGraph());

	// load spline data
	node.load(fullPath("uniginescript_samples/worlds/splines/spline.spl"));

	// assign all segments to geometry from file
	// SEGMENT_STRETCH - for stretching single node by segment
	// SEGMENT_TILING - for tiling nodes above segment
	string geometry_name = fullPath("uniginescript_samples/worlds/nodes/road_low_poly.node");
	SplineSegment segments[0];
	node.getSplineSegments(segments);
	forloop (int i = 0; segments.size())
	{
		segments[i].assignSource(geometry_name, FORWARD_X);
		segments[i].setSegmentMode(geometry_name, SEGMENT_TILING);
	}	
	
	setDescription("WorldSplineGraph: tiling geometry");
	
	return 1;
}

```

