# VRWorlds-Architecture
## General Architecture Documents for VRWorlds

VRWorlds (Current Working Name), is an Architecture for a Virtual World Framework that is modular, fully distributed, and federated (like the web is).

# Browser - Manage the user experience from the POV of the Avatar
* Manage the Aspects and their authentication and the various connections to all servers within the scene conversation.
* Generate a V8 environment for each separate participant.
* Manage subscriptions for teach V8 environment.
* Contribute processing power for the physics and magic engines of the world.  This is pretty much implied by how the system works.  Code is running in a V8 for the world, and it can send data back (so it can get data, run a computation and send back an answer).
* Contribute processing power for the AIs for the entity and avatar servers.
* Marshall all entities in the scene and coordinate with world server for entities in motion (physics) and spells in motion (magic).
# Kudo Server - Manage Certificates and Blockchain Logging
* Debatable if we actually need a blockchain, but we intend to use this for journal integrity.
* CA for servers and avatars.
* Authentication for avatars.
* Avatar state for worlds.
* Holder for proxy ownership delegations
* Assignments of Kudos (reputation points) for Avatars or Worlds
* Blockchain journaling of the all of the above.
# Avatar Server - Manage Avatars and Aspects
This is something of a complex Entity Server (entity servers can man age NPC objects which are nearly an Avatar in how it works)

* Manage the model for the avatar and any AIs that run serverside to support it, such as a movement emulator and the digital proprioception model.
* Allow the signed client code to be downloaded to the browser.
* Handle subscriptions and messages to/from the client
* This may include some bidirectional subscriptions and messages, like the inputs from the user - one of the subscriptions is the data-model for the mesh and such
* Handle procedure calls from the client
* Since the proper future handling of emotional facial representation might be expensive computationally, a lot of this might need to be done on the client — we may want to be able to do GPU acceleration with it.
# World Server - Manage a World
* Manage where everything is in the world
* Manage LOD for a scene
* Manage subscriptions for the browsers and the world’s browser-code on the browser
* Manage any AIs required for world function,
Accept and execute RPC calls from the client browser-codes.
* Management of physics
* Management of the magic systems (this includes what control abstractions that are permitted from the user’s avatar)
# Entity Server - Manage Entities/Assets 
* Manage the assets or a world, possibility including AI driven objects such as NPCs.  These are portable between worlds (with the permission and constraints of the magic/physics/participation rules of a given world).
* Handle the archetype of the entity
* Handle the inventory of all of the instances of that archetype in existence.
* Handle transient instances.
* Handle AIs for entity instances for things like NPCs
* Handle mesh/texture (appearance) subscriptions to the browser-code running on the browser.
* Handle rpc calls from browser-codes.

#Principles
* Client Browsers are not trusted
* So, the client browser-code is not trusted
* The Avatar is trusted as far as necessary
* Servers are trusted, generally any identity and authentication happens on the kudo server alone.  It is the book-of-record for permanent information, though the world and entity servers are permitted to persist their state.
* Built in load balancing, traffic management
* Each server can serve 1 or more shards, but shard affinity is mapped for load balancing (a shard # might be done by calculating a 10 bit piece of a guid), but each of the 1024 slots are assigned to a given server.   Servers don’t really have to be homogeneous.  Clients might receive reshard/shed/down messages with their return packets.
* Cap’n-proto over https
* Server in Go
* Browser in Unity/C#
* CodeBehind/browser-code/VRCode/Something (Non-Web Embedding in V8)  in WebAssembly/Javascript - something like Go/Wasm or Julia/Wasm
Client server use a bidirectional subscription/message/rpc architecture like meteor DDP.
* Browser can run  in game mode or with steamvr/openvr
# Cap’n Proto DDP Protocol
Site: https://capnproto.org/

Go Side:  https://github.com/capnproto/go-capnproto2

C# Side: https://github.com/ThomasBrixLarsen/capnproto-dotnet (latest maintained, we may have to take this over)

Original Protocol: https://github.com/meteor/meteor/blob/master/packages/ddp/DDP.md

We’ll have to figure out server side persistence.  Maybe we can use BSON or something faster than JSON to something to Cap’n Proto conversion.  It would be nice to avoid the whole serialization’/deserialization capnproto avoids.

The one issue with Cap’n Proto is that it doesn’t deal so well with generic schemas like you’d need with ad-hoc mongo tables.  Though if you had to, we can encapsulate the BSON or just do it with a JSON blob.   Maybe you could just pass the entire BSON blob itself down--hackey, but whatever.

This has a slightly different agenda than DDP for meteor.  Have to think about what we really need, thought Mesh DDP is almost the primary purpose.  That and what does a “document” mean in the sense of units that get replicated (though that is kind of up to the server and the browser-code to define).

# Need Some Kind of Object Mesh Format DDP
Which can be serialized down our DDP subscription (one of the default channels).  It needs to be adequately cached for high performance.  It’s the basic bottleneck that we’re totally going around the barn to avoid using Unity’s object marshalling system.  We might even have a separate DDP conversation to exchange this kind of data or have special data-types to efficiently encapsulate it.  Should keep somewhat generic in case someone wants to write a browser in something besides unity (unreal, or just opengl/vulcanvr), though it’s tempting to just save the unity mesh object blob into DDP directly.

We may want to assign GUIDs to fixed mesh constructs, so they can be cached (and so the subscription reactivity code can be smart enough not to resend them), though the server should also not resend.  Or just send the GUIDs and the client can ask for details. 
