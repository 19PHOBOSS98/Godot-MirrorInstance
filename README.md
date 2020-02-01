# Godot-MirrorInstance
A Portable, Self-Contained, Easy To Use Mirror Asset That Hides Stuff Behind it from Showing Up At The Front... For Godot

So I herd ya needed mirrors...[YouTube Video](https://www.youtube.com/embed/sL6519bUJUs)

This is a WIP build of my mirror asset,     
  this is the best that I could do within a week      
  and this is my first time ever using a game engine     
  so no promises :D


What it is:   
   - Stuff behind it doesn't appear on the mirror    
   - It's easy to Install    
   - It's easy to use    
   - It's easy to instantiate      
   - It's portable:(just drag and drop the .tscn file into the project/scene)    
   - The code is well documented:(comments with custom debugging code galore)    
   - and It's free

Instructions, documentations and debugging instructions(is just for fun XD) are all in the main code

I'd like to thank the ff people for their insight and tutorials:    
-Miziziziz: https://www.youtube.com/watch?v=xXUVP6sN-tQ   
-Bastiaan Olij: https://www.youtube.com/watch?v=wPzTmFHrGQk   
-Martin Senges: https://www.youtube.com/watch?v=DDZeW-AqjWI       





setting up:

	1. attach the script provided for the player camera
	2. drop the "MirrorInstance.tscn" file into your main scene with your player model
	3. make sure to set the project display window size to have a width 2xlonger than its height
	4. set each mirror you place with a different "IgnoreLayer" value in the inspector
    5. inspecting the MirrorInstance's Viewport("View") toggle "keep_3d_linear" to find the mirrors easier        
	6. resize the "HideArea" node when you want to resize the mirror      
	7. have fun



When in use:      
- if your using more than one mirror in a single scene you need to set each one with a different 'Ignore Layer' value
	
		unless you're cool with looking at a mirror seeing a bunch of stuff just dissapearing behind another mirror
		The key to this working right is to have the mirror's camera see stuff infront of the mirror and ignore stuff behind it
		
		picture this:
			you set up a huge mirror at the end of a hallway
			then a body mirror in a room right infront of the hallway mirror
			they're both set to ignore layer 2
			you decided to put the body mirror right next the door facing away from the hallway
			(puting the HideaArea right in the hallway to ignore anything that passes behind the body mirror)
			(...right in the hallway mirror's line of sight)
			so each time you passes the body mirror's 'HideArea' all your player model's MeshInstances are set to only appear in layer 2
			effectively makin you invisible to the body mirror... and the hallway mirror too because it's set to ignore layer 2 as well
			
			by having a different 'Ignore Layer' value you can be invisible to one mirror and not the other
			
		
- You can change what layer the mirror ignores in the inspector under ScriptVariable (MirrorInstance > Ignore Layer)

      The mirror uses a layer mask (default:layer 2) to hide anything that moves behind it.
      Remember to save before every time you replay the scene for it to take effect.
		
- You might want to check your objects' layer settings if their reflections don't show up
  
      Be aware that if you set it to ignore a layer,
      it will ignore everything that only exists in that single layer
      even if it's not behind the mirror
		
- Make sure to set MeshInstances that you place behind it by hand to be true at the layer you set the mirror to ignore
		(and false at every other layer)
	
- You need all your collisionShapes to be NOT 'Disabled' to trigger area collision
  
		Moving PhysicsBodies must trigger the 'HideArea' (the back of the mirror) when entering and exiting.
		The mirror SAVES the MeshInstances' layer mask of the body as it enters the HideArea in a dictionary.
		Whenever it leaves it checks out with its original layer mask, saved in the dictionary.
		
- It might be hard to find the mirror if it's perfectly reflective (as it should be)
  
		You can change this by disabling the 'keep_3d_linear' property at the 'View' node.
		
- It maybe over bearing if complex bodies, that have way too many children, were to walk behind it
  
		The code's designed to recursively check for MeshInstances in a body.
		The more children the body has the more branches the code has to go thru
		even if the body hasn't any MeshInstances at all
		so be careful??
		
- if you change your player-camera's render distance you also need to resize the HideArea
  
		say you were right infront of the mirror looking at it
		and there was this hallway right behind it
		the HideArea box would hide everything that has a CollisionShape existing inside it
		that includes the walls and evrything else in the hallway
		but then you had a tree further back OUTSIDE the hallway(i.e. outside the HideArea)
		you take a few step back away from the mirror and all of the sudden a tree comes into view out of nowhere
		
		that's why you have to change the HideArea size to hide everything behind it as far as the player-camera can see



For Debugging (or you know, if you want to know how it works... only if you feel like it tho...its not like I worked hard on this just to give it away for free or anything...*kicks tin can):

	- Observing Mirror Camera Orientation:
  
		There's a MeshInstance called 'This should be Invisible' as the mirror camera's child node
		it basically just lets you see where the camera is actually at in the scene
		just toggle its visibility (The icon that looks like an eye right beside it) if you want to see it
		
		
	- Observing The Recursive Node Search At Work:
  
		Basically how the code Sifts thru what a PhysicsBody has when it enters the HideArea
		I'm using the Depth-First Search method cause it saves a lot of memory space (it says it on the box)

		Go to the following functions and uncomment the indicated Diagnostic lines:
    
			- func _remember_MeshInstances(N):
          print(N.name)
          else:
          print("=================")
		
		remember to recomment the lines when you're done
		
		
	- Observing The HideArea At Work:
  
		Uncomment the global variable declaration: 'BDY' at the top
		Go to the following functions and uncomment the indicated Diagnostic lines:
    
			- func _remember_MeshInstances(N):
      
				print("\n","Before Entering, ",N.name,", from node: ",BDY,", had a layer mask value of: ",N.get_layer_mask())
				print("\n","While Inside, ",N.name,", from node: ",BDY,", has a layer mask value of: ",N.get_layer_mask())

			- func _restore_Mesh_mask(N):
      
				print("\n","Before Exiting, ",N.name,", from node: ",BDY,", had a layer mask value of: ",N.get_layer_mask())
				print("\n","After Exiting, ",N.name,", from node: ",BDY,", has a layer mask value of: ",N.get_layer_mask())
				
			- func _on_HideArea_body_entered(body):
      
				BDY=body.name
				print("\nNodes currently in dictionary 'a':\n",a.keys())
				print("\nTheir Cull Mask Values:\n",a.values())
				
			- func _on_HideArea_body_exited(body):
      
				BDY=body.name
				print("\nNodes currently in dictionary 'a':\n",a.keys())
				print("\nTheir Cull Mask Values:\n",a.values())
		
		Entries are printed on the Output each time a PhysicsBody enters the HideArea
    
		remember to recomment the lines when you're done
		
    
	- Observe the mirror camera's cull layer value and the layer the HideArea sets the PhysicsBodies to appear
  
		  Uncomment the global variable declaration: 'BDY'& 'Clmsk' at the top
    
		  Go to the following functions and uncomment the indicated Diagnostic lines:
      
			- func_ready():
      
                             Clmsk=(log(1048575-get_node("View/Camera").get("cull_mask"))/log(2)+1)
				
			- func _remember_MeshInstances(N):
      
                              print("\n","While Inside, \"",N.name,"\", from node: \"",BDY,"\", is set to appear only in layer: ",(log(N.get_layer_mask())/log(2)+1))
                              print("This Mirror's cull mask is set to ignore layer: ",Clmsk)
                              if((N.get_layer_mask()+get_node("View/Camera").get("cull_mask")==1048575)):
                                  print("The mirror is correctly ignoring this body when it passed behind it")
                              else:
                                 print("I don't know what but something definitely went wrong")
					
			- func _on_HideArea_body_entered(body):
      
                               BDY=body.name
	  
		Entries are printed on the Output each time a PhysicsBody enters the HideArea
		remember to recomment the lines when you're done



Disclaimers and things to improve upon:

	+ There has got to be a better way to do this...I had to do all of this just for an asset...
		I mean I couldn't find any water shader to use as a reflective surface
		the reflection probe is a no go for dynamic bodies
	
	+ I havn't implemented recursive reflections... so, no infinite mirrors yet :(
		(but it be dope if someone can help with that...)
		
	+ The asset changes the layer state of any MeshInstance that 
		belongs to a PhysicsBody that enters the 'HideArea' behind the mirror
		don't worry, the mirror puts it right back when it exits the trigger area
		but as long as that PhysicsBody is in there 
		its layer mask is set to the cull mask layer that the mirror's Camera is set to ignore
	
	+ the code's designed to recursively check for MeshInstances in a body
		the more children the body has the more branches the code has to go thru 
		even if the body hasn't any MeshInstances
		so... it might cause lag when a PhysicsBody with a LOT of children move behind it
	
	+ I have no idea how to test this performance wise, I was just going for 
		the goal of making this portable and a few other things
		
	+ the code could use some improvements, some of the things I used are really primitive
		and maybe optimization thru c++ would be nice
	
	+ a few things may go wrong, cause amanoob and i only did this within a week :)
		(so please have mercy)



Things to remember in recreating this from scratch:
	
I'v set the ff to 'Make Unique' at the main root node when I added the necessary materials in:

		MeshInstance/mesh
		MeshInstance/material/0
		MeshInstance/material/0/shader
		MeshInstance/material/0/shader_param/refl_tx
	
add a 'New ShaderMaterial' to MeshInstance/material/0

add a '.shader' file in MeshInstance/material/0/shader with the shader code ('Mshader.shader') that came with this code
	
	this way the 'shader_param/refl_tx' property appears just right below it so you can add a 'New ViewportTexture'
	(you might be asked to turn local_to_scene 'on' (MeshInstance/material/0/resource_local_to_scene)
	before you can actually add a new one)
	
	(in adding the 'New ViewportTexture' you might have to pick a viewport node initially 
	but this script should be able to find and set the right one just fine

For the Viewport node set the 'shadow_atlas_size' to atleast 1 so the mirror can render shadows

set the Camera node propery: 'current' on

and the Position nodes name has to match what the 'var dummy_cam' is refering to in the script: 'DummyCam'

The mirror's Camera nodes are set to ignore Cull Mask layer 2

	to hide anybody(PhysicsBodies) that moves behind the mirror
	so it be best to not use that layer
	if you really want to use that layer for something else
	there should be a property called 'Layer Bit' if you look at the 'Mirror Instance' in the inspector
		this makes the mirror's Camera node ignore the layer you choose at run time
		and it also sets the HideArea node too
		
The 'HideArea' node has all its Collision Mask enabled by default

	This lets the Mirror detect collisions regardless of the body's set collision layer
	so screw with it at your own risk

Hi! this is the first asset that I created in Godot, actually this is the only thing I'v made in Godot so far...

to be honest this is my first time ever using a game engine ever so bear with me :).        
              
    I liked the idea of mirrors but I only found a few people who talked about the subject within    
    the Godot community.  It all started from the tutorials when I noticed that they all built theirs
    within the main world. I tried finding a mirror asset but nothing ever came up so, I decided to try
    and make one as my intro to the world of game developing.

    I also noticed that the dominant technique in building mirrors in godot couldn't stop things
    moving behind it from showing up in the mirror so I said screw it Imma carry that too no matter how
    awkward I decide to do it.

    soooo, here it is, a self contained, very portable, and cull layer friendly-ish mirror asset 
    that doesn't show objects from behind... :D

  	Thank you for actually taking the time to read all of that
	
    sincerely, PHO-BOSS (KIM NICKHO A. OBORDO)


Shameless Plug:
	check out my YouTube channel...: 
		https://www.youtube.com/channel/UCmdUkIr-zCxO9CvOLxeSzlg?view_as=subscriber

