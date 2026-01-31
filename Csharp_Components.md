# Сборник C# компонентов
## AbstractComponent.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "c26624845abbfd3d0bbe7d4c15d8572a9682b68e")]
public abstract class AbstractComponent : Component
{
	public abstract void DoSomething();
}
```
---
## AbstractComponentImplementation.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "75fc620fe6076a02782bc4674dba55bcc1e0a699")]
public class AbstractComponentImplementation : AbstractComponent
{
	public override void DoSomething()
	{
		Log.MessageLine("AbstractComponentImplementation::DoSomething()");
	}
}
```
---
## AnimationAdditive.cs
```csharp
﻿using System;
using Unigine;
using static Unigine.VREyeTracking;
[Component(PropertyGuid = "d61485a9563c12f466dd21c3dd285c468f1586ed")]
public class AnimationAdditive : Component
{
	public float firstAnimationSpeed = 30.0f;
	public float secondAnimationSpeed = 30.0f;
	[ParameterFile(Filter = ".anim")]
	public string firstAnimation = "";
	[ParameterFile(Filter = ".anim")]
	public string secondAnimation = "";
	public int ReferncesFramesCount => meshSkinned.GetLayerNumFrames((int)LAYERS.SECOND_ANIMATION);
	private ObjectMeshSkinned meshSkinned = null;
	private float currentTime = 0.0f;
	private float animationRefereceFrame = 0.0f;
	private float weight = 0.5f;
	private enum LAYERS
	{
		FIRST_ANIMATION = 0,
		SECOND_ANIMATION,
		AUXILIARY,
		COUNT
	}
	private SampleDescriptionWindow sampleDescriptionWindow;
	private void Init()
	{
		sampleDescriptionWindow = new SampleDescriptionWindow();
		sampleDescriptionWindow.createWindow();
		meshSkinned = node as ObjectMeshSkinned;
		if (meshSkinned != null)
		{
			meshSkinned.NumLayers = (int)LAYERS.COUNT;
			meshSkinned.SetLayerAnimationFilePath((int)LAYERS.FIRST_ANIMATION, firstAnimation);
			meshSkinned.SetLayerAnimationFilePath((int)LAYERS.SECOND_ANIMATION, secondAnimation);
			// inverse reference frame into the auxiliary layer
			meshSkinned.SetLayerFrame((int)LAYERS.SECOND_ANIMATION, animationRefereceFrame);
			meshSkinned.InverseLayer((int)LAYERS.AUXILIARY, (int)LAYERS.SECOND_ANIMATION);
		}
		sampleDescriptionWindow.addFloatParameter("Weight:", "Weight", weight, 0.0f, 4.0f, (float value) =>
		{
			weight = value;
		});
		sampleDescriptionWindow.addFloatParameter("Reference Frame:", "ReferenceFrame", animationRefereceFrame, 0.0f, ReferncesFramesCount,
			(float value) =>
			{
				animationRefereceFrame = value;
				meshSkinned.SetLayerFrame((int)LAYERS.SECOND_ANIMATION, animationRefereceFrame);
				meshSkinned.InverseLayer((int)LAYERS.AUXILIARY, (int)LAYERS.SECOND_ANIMATION);
			}
		);
	}
	private void Update()
	{
		// set frames
		meshSkinned.SetLayerFrame((int)LAYERS.FIRST_ANIMATION, currentTime * firstAnimationSpeed);
		meshSkinned.SetLayerFrame((int)LAYERS.SECOND_ANIMATION, currentTime * secondAnimationSpeed);
		currentTime += Game.IFps;
		// multiple second layer by inverse reference frame
		meshSkinned.MulLayer((int)LAYERS.SECOND_ANIMATION, (int)LAYERS.AUXILIARY, (int)LAYERS.SECOND_ANIMATION);
		// combine two animations
		meshSkinned.MulLayer((int)LAYERS.FIRST_ANIMATION, (int)LAYERS.FIRST_ANIMATION, (int)LAYERS.SECOND_ANIMATION, weight);
	}
	private void Shutdown()
	{
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## AnimationInterpolation.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "4a532b80657d28df3fd9e0b23b684869dfd576b4")]
public class AnimationInterpolation : Component
{
	public float firstAnimationSpeed = 30.0f;
	public float secondAnimationSpeed = 30.0f;
	[ParameterFile(Filter = ".anim")]
	public string firstAnimation = "";
	[ParameterFile(Filter = ".anim")]
	public string secondAnimation = "";
	private ObjectMeshSkinned meshSkinned = null;
	private float currentTime = 0.0f;
	private float weight = 0.5f;
	private enum LAYERS
	{
		FIRST_ANIMATION = 0,
		SECOND_ANIMATION,
		COUNT
	}
	private SampleDescriptionWindow sampleDescriptionWindow;
	private void Init()
	{
		sampleDescriptionWindow = new SampleDescriptionWindow();
		sampleDescriptionWindow.createWindow();
		meshSkinned = node as ObjectMeshSkinned;
		if (meshSkinned != null)
		{
			meshSkinned.NumLayers = (int)LAYERS.COUNT;
			meshSkinned.SetLayerAnimationFilePath((int)LAYERS.FIRST_ANIMATION, firstAnimation);
			meshSkinned.SetLayerAnimationFilePath((int)LAYERS.SECOND_ANIMATION, secondAnimation);
		}
		sampleDescriptionWindow.addFloatParameter("Weight:", "Weight", weight, 0.0f, 1.0f, (float value) =>
		{
			weight = value;
		});
	}
	private void Update()
	{
		if (meshSkinned == null)
			return;
		// set current frame for first and second animation
		meshSkinned.SetLayerFrame((int)LAYERS.FIRST_ANIMATION, currentTime * firstAnimationSpeed);
		meshSkinned.SetLayerFrame((int)LAYERS.SECOND_ANIMATION, currentTime * secondAnimationSpeed);
		currentTime += Game.IFps;
		// interpolate between layers
		meshSkinned.LerpLayer((int)LAYERS.FIRST_ANIMATION, (int)LAYERS.FIRST_ANIMATION, (int)LAYERS.SECOND_ANIMATION, weight);
	}
	private void Shutdown()
	{
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## AnimationPartialInterpolation.cs
```csharp
﻿using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "a94fe3f2ecf1f2a95de89bf4ea3616c3e74acd46")]
public class AnimationPartialInterpolation : Component
{
	public float firstAnimationSpeed = 30.0f;
	public float secondAnimationSpeed = 30.0f;
	[ParameterFile(Filter = ".anim")]
	public string firstAnimation = "";
	[ParameterFile(Filter = ".anim")]
	public string secondAnimation = "";
	public List<string> interpolatedBones = null;
	private ObjectMeshSkinned meshSkinned = null;
	private float currentTime = 0.0f;
	private List<int> bonesNumbers = null;
	private float weight = 0.5f;
	private enum LAYERS
	{
		FIRST_ANIMATION = 0,
		SECOND_ANIMATION,
		COUNT
	}
	private SampleDescriptionWindow sampleDescriptionWindow;
	private void Init()
	{
		sampleDescriptionWindow = new SampleDescriptionWindow();
		sampleDescriptionWindow.createWindow();
		meshSkinned = node as ObjectMeshSkinned;
		if (meshSkinned != null)
		{
			// set layers and animations
			meshSkinned.NumLayers = (int)LAYERS.COUNT;
			meshSkinned.SetLayerAnimationFilePath((int)LAYERS.FIRST_ANIMATION, firstAnimation);
			meshSkinned.SetLayerAnimationFilePath((int)LAYERS.SECOND_ANIMATION, secondAnimation);
			// find bones in mesh and save their numbers
			bonesNumbers = new List<int>();
			for (int i = 0; i < meshSkinned.NumBones; i++)
			{
				string name = meshSkinned.GetBoneName(i);
				if (interpolatedBones.Contains(name))
					bonesNumbers.Add(i);
			}
		}
		sampleDescriptionWindow.addFloatParameter("Weight:", "Weight", weight, 0.0f, 1.0f, (float value) =>
		{
			weight = value;
		});
	}
	private void Update()
	{
		if (meshSkinned == null)
			return;
		// set current frame for first and second animation
		meshSkinned.SetLayerFrame((int)LAYERS.FIRST_ANIMATION, currentTime * firstAnimationSpeed);
		meshSkinned.SetLayerFrame((int)LAYERS.SECOND_ANIMATION, currentTime * secondAnimationSpeed);
		currentTime += Game.IFps;
		// interpolate between layers
		meshSkinned.LerpLayer((int)LAYERS.SECOND_ANIMATION, (int)LAYERS.FIRST_ANIMATION, (int)LAYERS.SECOND_ANIMATION, weight);
		// set transform for interpolated bones
		if (bonesNumbers != null)
			foreach (int bone in bonesNumbers)
				meshSkinned.SetLayerBoneTransform((int)LAYERS.FIRST_ANIMATION, bone, meshSkinned.GetLayerBoneTransform((int)LAYERS.SECOND_ANIMATION, bone));
	}
	private void Shutdown()
	{
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## AnimationRotation.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "5de064783893dd77a6cd9c6ae78c20c10672832c")]
public class AnimationRotation : Component
{
	[ParameterFile(Filter = ".anim")]
	public string idleAnim = "";
	[ParameterFile(Filter = ".anim")]
	public string leftShootAnim = "";
	[ParameterFile(Filter = ".anim")]
	public string rightShootAnim = "";
	private ObjectMeshSkinned meshSkinned = null;
	private const int horizontalJointBone = 1;
	private enum SIDE
	{
		LEFT = 0,
		RIGHT
	}
	private enum LAYERS
	{
		IDLE = 0,
		LEFT,
		RIGHT,
		REFERENCE_FRAME,
		COUNT
	}
	private float animationSpeed = 30.0f;
	private float currentTime = 0.0f;
	private float shootAnimationTime = 0.6f;
	private float currentShootTime = 0.0f;
	private SIDE currentSide = SIDE.LEFT;
	private int leftFrameCount = 0;
	private int rightFrameCount = 0;
	private Mat4 boneTransform = Mat4.IDENTITY;
	private void Init()
	{
		meshSkinned = node as ObjectMeshSkinned;
		if (!meshSkinned)
			return;
		// set animation on different layers
		meshSkinned.NumLayers = (int)LAYERS.COUNT;
		meshSkinned.SetLayerAnimationFilePath((int)LAYERS.IDLE, idleAnim);
		meshSkinned.SetLayerAnimationFilePath((int)LAYERS.LEFT, leftShootAnim);
		meshSkinned.SetLayerAnimationFilePath((int)LAYERS.RIGHT, rightShootAnim);
		// set inverse transform for blend calculation on auxiliary layer
		meshSkinned.SetLayerFrame((int)LAYERS.IDLE, 0);
		meshSkinned.InverseLayer((int)LAYERS.REFERENCE_FRAME, (int)LAYERS.IDLE);
		// get number of frames in animations
		leftFrameCount = meshSkinned.GetLayerNumFrames((int)LAYERS.LEFT);
		rightFrameCount = meshSkinned.GetLayerNumFrames((int)LAYERS.RIGHT);
		boneTransform = meshSkinned.GetBoneWorldTransform(horizontalJointBone);
	}
	private void Update()
	{
		if (!meshSkinned)
			return;
		meshSkinned.SetLayerFrame((int)LAYERS.IDLE, currentTime * animationSpeed);
		currentTime += Game.IFps;
		// update side of shoot
		currentShootTime += Game.IFps;
		if (currentShootTime > shootAnimationTime)
		{
			currentShootTime = 0.0f;
			currentSide = (currentSide == SIDE.LEFT ? SIDE.RIGHT : SIDE.LEFT);
		}
		float k = MathLib.Saturate(currentShootTime / shootAnimationTime);
		switch (currentSide)
		{
			// 1. set the layer to the corresponding frame of animation
			// 2. find local transformations of all bones relative to reference frame
			// 3. add local transform﻿ations of shoot animation bones to the idle animation
			case SIDE.LEFT:
				meshSkinned.SetLayerFrame((int)LAYERS.LEFT, leftFrameCount * k);
				meshSkinned.MulLayer((int)LAYERS.LEFT, (int)LAYERS.REFERENCE_FRAME, (int)LAYERS.LEFT);
				meshSkinned.MulLayer((int)LAYERS.IDLE, (int)LAYERS.IDLE, (int)LAYERS.LEFT);
				break;
			case SIDE.RIGHT:
				meshSkinned.SetLayerFrame((int)LAYERS.RIGHT, rightFrameCount * k);
				meshSkinned.MulLayer((int)LAYERS.RIGHT, (int)LAYERS.REFERENCE_FRAME, (int)LAYERS.RIGHT);
				meshSkinned.MulLayer((int)LAYERS.IDLE, (int)LAYERS.IDLE, (int)LAYERS.RIGHT);
				break;
		}
		// set the bone transform for the horizontal joint
		boneTransform =  boneTransform * new Mat4(MathLib.RotateZ(45.0f * Game.IFps));
		meshSkinned.SetBoneWorldTransformWithChildren(horizontalJointBone, boneTransform);
	}
}
```
---
## AsyncQueueSample.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "17b88eee316007eac924ae078e5a90550c3746be")]
public class AsyncQueueSample : Component
{
	[ShowInEditor][ParameterFile]
	private string[] meshes = null;
	[ShowInEditor][ParameterFile]
	private string[] textures = null;
	struct AsyncLoadRequest
	{
		public string name;
		public int id;
	}
	private List<AsyncLoadRequest> meshLoadRequest = new List<AsyncLoadRequest>();
	private int objectsPlaced = 0;
	private List<WidgetSprite> sprites = new List<WidgetSprite>();
	private EventConnection imageLoadedConnection;
	void Init()
	{
		for(int i =0; i < meshes.Length; i++)
		{
			string name = meshes[i];
			AsyncLoadRequest request = new AsyncLoadRequest();
			request.name = name;
			request.id = AsyncQueue.LoadMesh(name);
			meshLoadRequest.Add(request);
		}
		for(int i =0; i < textures.Length; i++)
		{
			AsyncQueue.LoadImage(textures[i]);
		}
		imageLoadedConnection = AsyncQueue.EventImageLoaded.Connect(ImageLoadedCallback);
		Console.Onscreen = true;
	}
	void Update()
	{
		for(int i =0; i < meshLoadRequest.Count; i++)
		{
			AsyncLoadRequest request = meshLoadRequest[i];
			if (AsyncQueue.CheckMesh(request.id) == 0)
				continue;
			Mesh mesh = AsyncQueue.TakeMesh(request.id);
			if(mesh != null)
			{
				ObjectMeshDynamic objectMeshDynamic = new ObjectMeshDynamic(mesh);
				Scalar initialPos = -5;
				Scalar step = 5;
				objectMeshDynamic.Position = new Vec3(initialPos + (float)objectsPlaced * step, 0.0f, 0.0f);
				objectsPlaced++;
				AsyncQueue.RemoveMesh(request.id);
				Log.MessageLine($"Loaded mesh {request.name}");
				meshLoadRequest.RemoveAt(i--);
			}
		}
	}
	void Shutdown()
	{
		foreach(WidgetSprite sprite in sprites)
		{
			sprite.DeleteLater();
		}
		Console.Onscreen = false;
		imageLoadedConnection.Disconnect();
	}
	private void ImageLoadedCallback(string name, int id)
	{
		Image loadedImage = AsyncQueue.TakeImage(id);
		if (loadedImage == null)
			return;
		AsyncQueue.RemoveImage(id);
		Log.MessageLine($"Image {name} loaded");
		var sprite = new WidgetSprite();
		sprites.Add(sprite);
		sprite.SetImage(loadedImage);
		sprite.Width = 100;
		sprite.Height = 100;
		WindowManager.MainWindow.AddChild(sprite, Gui.ALIGN_OVERLAP | Gui.ALIGN_BACKGROUND);
		ivec2 initialSpritePosition = new ivec2(0, WindowManager.MainWindow.Size.y - 200);
		ivec2 newPos = new ivec2(initialSpritePosition.x + sprites.Count * 100, initialSpritePosition.y);
		sprite.SetPosition(newPos.x, newPos.y);
	}
}
```
---
## AsyncQueueStressSample.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System.Threading;
using Unigine;
[Component(PropertyGuid = "a8bfb39278ee1070f8d9f70d5e424754152a26ed")]
public class AsyncQueueStressSample : Component
{
	[ShowInEditor][ParameterFile]
	private string nodeToSpawn = null;
	private int numNodesLoaded;
	private WidgetLabel numNodesLoadedLabel;
	private SampleDescriptionWindow sampleDescriptionWindow = new SampleDescriptionWindow();
	void Init()
	{
		Profiler.Enabled = true;
		numNodesLoaded = 0;
		sampleDescriptionWindow.createWindow(Gui.ALIGN_RIGHT);
		WidgetGroupBox parameters = sampleDescriptionWindow.getParameterGroupBox();
		var numNodesHBox = new WidgetHBox(5);
		parameters.AddChild(numNodesHBox, Gui.ALIGN_EXPAND);
		var multithreadLabel = new WidgetLabel("Num nodes");
		numNodesHBox.AddChild(multithreadLabel);
		var spinboxHBox = new WidgetHBox();
		var editline = new WidgetEditLine();
		editline.Validator = Gui.VALIDATOR_INT;
		var spinbox = new WidgetSpinBox();
		editline.AddAttach(spinbox);
		spinbox.MinValue = 1;
		spinbox.MaxValue = 10000;
		spinbox.Value = 100;
		spinboxHBox.AddChild(editline);
		spinboxHBox.AddChild(spinbox);
		numNodesHBox.AddChild(spinboxHBox, Gui.ALIGN_RIGHT);
		var requestLoadNodesButton = new WidgetButton("Request Load Nodes Async");
		parameters.AddChild(requestLoadNodesButton, Gui.ALIGN_EXPAND);
		numNodesLoadedLabel = new WidgetLabel();
		parameters.AddChild(numNodesLoadedLabel, Gui.ALIGN_EXPAND);
		requestLoadNodesButton.EventClicked.Connect(() =>
		{
			AsyncQueue.RunAsync(AsyncQueue.ASYNC_THREAD.BACKGROUND, () => { LoadNodes(spinbox.Value); });
		});
	}
	void Update()
	{
		numNodesLoadedLabel.Text = "Num nodes loaded " + numNodesLoaded.ToString();
		if (numNodesLoaded > 2000)
			numNodesLoadedLabel.FontColor = vec4.RED;
	}
	void Shutdown()
	{
		Profiler.Enabled = false;
		sampleDescriptionWindow.shutdown();
	}
	private void LoadNodes(int num)
	{
		for (int i = 0; i < num; ++i)
		{
			// here we are loading the node not in the main thread, so it will not be added to the spatial tree
			Node loadedNode = World.LoadNode(nodeToSpawn, false);
			Vec3 position = new Vec3();
			position.x = Game.GetRandomFloat(-100.0f, 100.0f);
			position.y = Game.GetRandomFloat(-100.0f, 100.0f);
			position.z = Game.GetRandomFloat(0.0f, 50.0f);
			loadedNode.WorldPosition = position;
			Interlocked.Add(ref numNodesLoaded, 1);
			AsyncQueue.RunAsync(AsyncQueue.ASYNC_THREAD.MAIN, () =>
			{
				// call updateEnabled which will recursively go through all the children and add them to the spatial tree
				loadedNode.UpdateEnabled();
			});
	}
}
}
```
---
## AsyncQueueTasksSample.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using System.Threading;
using Unigine;
[Component(PropertyGuid = "de9cb9b47ac32b0e4f34b127fa6dedebf19a3fc7")]
public class AsyncQueueTasksSample : Component
{
	private SampleDescriptionWindow sampleDescriptionWindow = new SampleDescriptionWindow();
	void Init()
	{
		Console.Onscreen = true;
		// create sample UI
		sampleDescriptionWindow.createWindow(Gui.ALIGN_RIGHT);
		WidgetGroupBox parameters = sampleDescriptionWindow.getParameterGroupBox();
		var asyncThreadTypeHBox = new WidgetHBox(5);
		parameters.AddChild(asyncThreadTypeHBox, Gui.ALIGN_EXPAND);
		var asyncThreadTypeLabel = new WidgetLabel("Task Thread Type");
		asyncThreadTypeHBox.AddChild(asyncThreadTypeLabel);
		var asyncThreadTypeCombobox = new WidgetComboBox();
		asyncThreadTypeCombobox.AddItem("BACKGORUND");
		asyncThreadTypeCombobox.AddItem("ASYNC");
		asyncThreadTypeCombobox.AddItem("GPU STREAM");
		asyncThreadTypeCombobox.AddItem("FILE STREAM");
		asyncThreadTypeCombobox.AddItem("MAIN");
		asyncThreadTypeCombobox.AddItem("NEW");
		asyncThreadTypeHBox.AddChild(asyncThreadTypeCombobox, Gui.ALIGN_EXPAND);
		var runAsyncButton = new WidgetButton("Run Async");
		runAsyncButton.EventClicked.Connect(() =>
		{
			// run a task asynchronously in a specified thread
			// also you can specify priority of your task
			// ASYNC_PRIORITY_CRITICAL - hight
			// ASYNC_PRIORITY_DEFAULT - medium
			// ASYNC_PRIORITY_BACKGROUND - low
			AsyncQueue.RunAsync((AsyncQueue.ASYNC_THREAD)(asyncThreadTypeCombobox.CurrentItem), AsyncTask);
		});
		parameters.AddChild(runAsyncButton, Gui.ALIGN_EXPAND);
		var spacer = new WidgetSpacer();
		parameters.AddChild(spacer, Gui.ALIGN_EXPAND);
		var multithreadHBox = new WidgetHBox(5);
		parameters.AddChild(multithreadHBox, Gui.ALIGN_EXPAND);
		var multithreadLabel = new WidgetLabel("Num threads");
		multithreadHBox.AddChild(multithreadLabel);
		var spinboxHBox = new WidgetHBox();
		var multithreadEditline = new WidgetEditLine();
		multithreadEditline.Editable = false;
		var multithreadSpinbox = new WidgetSpinBox();
		multithreadEditline.AddAttach(multithreadSpinbox);
		multithreadSpinbox.MinValue = 1;
		multithreadSpinbox.MaxValue = 20;
		multithreadSpinbox.Value = 1;
		spinboxHBox.AddChild(multithreadEditline);
		spinboxHBox.AddChild(multithreadSpinbox);
		multithreadHBox.AddChild(spinboxHBox, Gui.ALIGN_RIGHT);
		var frame_checkbox = new WidgetCheckBox("Wait for multithreaded task to complete in frame");
		parameters.AddChild(frame_checkbox, Gui.ALIGN_LEFT);
		var run_async_multithread_button = new WidgetButton("Run Async Multithread");
		run_async_multithread_button.EventClicked.Connect(() =>
		{
			// run a task in a multithread mode, current thread number and total amount of thread are passed to the callback
			// does not block the thread from which it is called
			if (frame_checkbox.Checked)
				AsyncQueue.RunFrameAsyncMultiThread(MultithreadTask, multithreadSpinbox.Value);
			else
				AsyncQueue.RunAsyncMultiThread(MultithreadTask, multithreadSpinbox.Value);
		});
		parameters.AddChild(run_async_multithread_button, Gui.ALIGN_EXPAND);
		var run_sync_multithread_button = new WidgetButton("Run Sync Multithread");
		run_sync_multithread_button.EventClicked.Connect(() =>
		{
			// run a task in a multithread mode, current thread number and total amount of thread are passed to the callback
			// blocks the thread from which it was called (the calling thread will be unblocked after the task is completed in all threads)
			if (frame_checkbox.Checked)
				AsyncQueue.RunFrameSyncMultiThread(MultithreadTask, multithreadSpinbox.Value);
			else
				AsyncQueue.RunSyncMultiThread(MultithreadTask, multithreadSpinbox.Value);
		});
		parameters.AddChild(run_sync_multithread_button, Gui.ALIGN_EXPAND);
	}
	void Shutdown()
	{
		Console.Onscreen = false;
		sampleDescriptionWindow.shutdown();
	}
	private void AsyncTask()
	{
		// simulate task work
		Thread.Sleep(200);
		Log.MessageLine("This is async task, thread id: " + Thread.CurrentThread.ManagedThreadId.ToString());
	}
	private void MultithreadTask(int currentThread, int totalThreads)
	{
		// simulate task work
		Thread.Sleep(200);
		Log.MessageLine($"This is multithread task(current thread: {currentThread}, total number of threads: {totalThreads}), thread id: " + Thread.CurrentThread.ManagedThreadId.ToString());
	}
}
```
---
## BodyCallbacks.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
#endif
[Component(PropertyGuid = "723d60191b1c46188aaf19df9e26724a53a6d73d")]
public class BodyCallbacks : Component
{
	// tower params
	public float space = 1.2f;
	public int tower_level = 10;
	// different materials for different callbacks
	public Material frozen_material;
	public Material position_material;
	// mesh used to build a tower
	[ParameterFile(Filter = ".mesh")]
	public string mesh_file = "";
	private List<Node> objects = new List<Node>();
	private EventConnections body_connections = new EventConnections();
	private bool visualizer_state;
	void Init()
	{
		visualizer_state = Visualizer.Enabled;
		Visualizer.Enabled = true;
		// parameters validation
		if (mesh_file.Length <= 0)
			Log.Error("BodyCallbacks.Init(): Mesh File parameter is empty!\n");
		if (!frozen_material)
			Log.Error("BodyCallbacks.Init(): Frozen Matreial parameter is empty!\n");
		if (!position_material)
			Log.Error("BodyCallbacks.Init(): Position Matreial parameter is empty!\n");
		// general Physics settings
		Physics.FrozenLinearVelocity = 0.1f;
		Physics.FrozenAngularVelocity = 0.1f;
		Physics.NumIterations = 4;
		// create object and physical body with a box shape
		ObjectMeshStatic obj = new ObjectMeshStatic(mesh_file);
		BodyRigid body = new BodyRigid(obj);
		ShapeBox shape = new ShapeBox(body, new vec3(1));
		obj.SetMaterial(position_material, "*");
		// create tower
		for (int i = 0; i < tower_level; i++)
		{
			for (int j = 0; j < tower_level - i; j++)
			{
				// clone created earlier object and calculate it's position in a tower
				ObjectMeshStatic mesh = obj.Clone() as ObjectMeshStatic;
				mesh.WorldTransform = MathLib.Translate(new Vec3(0.0f, j - 0.5f * (tower_level - i) + 0.5f, i + 0.5f) * space);
				// add Frozen, Position and Contact callbacks to new object's body
				body = mesh.BodyRigid;
				body.EventFrozen.Connect(body_connections, b => b.Object.SetMaterial(frozen_material, "*"));
				body.EventPosition.Connect(body_connections, b => b.Object.SetMaterial(position_material, "*"));
				body.EventContactEnter.Connect(body_connections, (b, num) => b.RenderContacts());
				objects.Add(mesh);
			}
		}
		obj.DeleteLater();
	}
	void Shutdown()
	{
		// remove all connections
		body_connections.DisconnectAll();
		objects.Clear();
		// restore visualizer state
		Visualizer.Enabled = visualizer_state;
	}
}
```
---
## BodyFractureExplosion.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "ac14cbc709b25cc2da74edb278f89a4ac116cbac")]
public class BodyFractureExplosion : Component
{
	public float MaxRadius = 10.0f;
	public float Speed = 100.0f;
	public float Force = 100.0f;
	private float radius = 0.0f;
	public void Explode()
	{
		radius = 0.0f;
	}
	void Init()
	{
		radius = MaxRadius;	
	}
	void Update()
	{
		if (MaxRadius == 0 || radius > MaxRadius)
			return;
		BoundSphere sphere = new(new vec3(node.WorldPosition), radius);
		var actualForce = Force * (1 - radius / MaxRadius);
		Visualizer.RenderBoundSphere(sphere, mat4.IDENTITY, vec4.RED, 0.01f);
		List<Object> objects = [];
		if (World.GetIntersection(new WorldBoundSphere(sphere), objects))
		{
			foreach (var obj in objects)
			{
				var body = obj.Body;
				if (body == null)
					continue;
				var dir = new vec3(obj.WorldPosition - node.WorldPosition);
				if (dir.Length2 != 0)
					dir.Normalize();
				var fracture = GetComponent<BodyFractureUnit>(obj);
				fracture?.Crack(obj.WorldPosition, -dir, actualForce);
				var rigid = body as BodyRigid;
				rigid?.AddLinearImpulse(dir * actualForce);
			}
		}
		radius += Speed * Game.IFps;
	}
}
```
---
## BodyFractureExplosionSample.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "f1ef0201346f7487ef96082467dee7850d6d5d2e")]
public class BodyFractureExplosionSample : Component
{
	public BodyFractureExplosion explosion = null;
	public SampleDescriptionWindow sampleDescriptionWindow = new();
	void Init()
	{
		sampleDescriptionWindow.createWindow();
		var btn = new WidgetButton("Explode!");
		btn.EventClicked.Connect(() => explosion?.Explode());
		sampleDescriptionWindow.getParameterGroupBox().AddChild(btn);
		Visualizer.Enabled = true;
	}
	void Shutdown()
	{
		Visualizer.Enabled = false;
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## BodyFractureFallingSpheresSample.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "d16617a2e6277e064d73c9810c14f58d672051de")]
public class BodyFractureFallingSpheresSample : Component
{
	void Init()
	{
		Visualizer.Enabled = true;
	}
	void Shutdown()
	{
		Visualizer.Enabled = false;
	}
}
```
---
## BodyFractureShootingGalleryGun.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using Unigine;
[Component(PropertyGuid = "46b8b84b2dc018a78753ab7dd80b07824cfdbe4f")]
public class BodyFractureShootingGalleryGun : Component
{
	public float Force = 50.0f;
	[ParameterFile(Filter = ".node")]
	public string InstanceNode = "";
	[ParameterFile]
	public string CrosshairImage = "";
	public ivec2 CrosshairSize = new(25, 25);
	private WidgetSprite crosshair = null;
	void Init()
	{
		if (CrosshairImage != "")
		{
			crosshair = new(CrosshairImage) { Width = CrosshairSize.x, Height = CrosshairSize.y };
			Gui.GetCurrent().AddChild(crosshair, Gui.ALIGN_OVERLAP | Gui.ALIGN_CENTER);
		}
	}
	void Update()
	{
		if (Console.Active && InstanceNode != "")
			return;
		if (Input.IsMouseButtonDown(Input.MOUSE_BUTTON.LEFT))
		{
			var inst = World.LoadNode(InstanceNode);
			inst.WorldPosition = node.WorldPosition;
			var body = inst.ObjectBodyRigid;
			if (!body)
				return;
			body.AddLinearImpulse(node.GetWorldDirection() * Force);
		}
	}
	void Shutdown()
	{
		crosshair?.DeleteLater();
	}
}
```
---
## BodyFractureShootingGallerySample.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "fd2486a7ca974d39ef6d932909b6a2636f183941")]
public class BodyFractureShootingGallerySample : Component
{
	private Input.MOUSE_HANDLE mouse_handle;
	void Init()
	{
		Visualizer.Enabled = true;
		mouse_handle = Input.MouseHandle;
		Input.MouseHandle = Input.MOUSE_HANDLE.GRAB;
	}
	void Shutdown()
	{
		Visualizer.Enabled = false;
		Input.MouseHandle = mouse_handle;
	}
}
```
---
## BoundBoxIntersection.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System.Collections.Generic;
using Unigine;
using static Unigine.VREyeTracking;
[Component(PropertyGuid = "b1b8facb749391ae2df6ad35c525dd10ebff8d2f")]
public class BoundBoxIntersection : Component
{
	public dvec3 minPoint = dvec3.ZERO;
	public dvec3 maxPoint = dvec3.ZERO;
	private WorldBoundBox boundBox;
	private List<Node> nodes = null;
	private SampleDescriptionWindow sampleDescriptionWindow = new SampleDescriptionWindow();
	private void Init()
	{
		// create bound box based on minimum and maximum world coordinates
		boundBox = new WorldBoundBox(new Vec3(minPoint), new Vec3(maxPoint));
		// create collection for intersecting nodes
		nodes = new List<Node>();
		Visualizer.Enabled = true;
		sampleDescriptionWindow.createWindow();
	}
	private void Update()
	{
		// show bound box
		Visualizer.RenderBoundBox(new BoundBox(new vec3(boundBox.minimum), new vec3(boundBox.maximum)), Mat4.IDENTITY, new vec4(0.0f, 1.0f, 0.0f, 1.0f));
		// try get nodes inside bound box
		var status = "Inside bound box:";
		bool res = World.GetIntersection(boundBox, Node.TYPE.OBJECT_MESH_STATIC, nodes);
		if (res)
		{
			// show nodes names
			foreach (Node n in nodes)
				status += $" {n.Name}";
		}
		else
			status += " empty";
		sampleDescriptionWindow.setStatus(status);
	}
	private void Shutdown()
	{
		Visualizer.Enabled = false;
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## BoundFrustumIntersection.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "3be055ec0d1b585b6ce9b1fe43063145eb1df1e0")]
public class BoundFrustumIntersection : Component
{
	public float fov = 100.0f;
	public float aspect = 1.0f;
	public float zNear = 0.001f;
	public float zFar = 10.0f;
	public vec3 position = vec3.ZERO;
	public vec3 rotation = vec3.ZERO;
	private mat4 frustumTransform = mat4.IDENTITY;
	private WorldBoundFrustum boundFrustum;
	private List<Node> nodes = null;
	private SampleDescriptionWindow sampleDescriptionWindow;
	private void Init()
	{
		// create frustum transform based on fov, aspect, znear and zfar in world coordinates
		frustumTransform = MathLib.Perspective(fov, aspect, zNear, zFar);
		frustumTransform = frustumTransform * MathLib.RotateX(-90);
		frustumTransform = frustumTransform * MathLib.Translate(-position);
		frustumTransform = frustumTransform * MathLib.Rotate(new quat(rotation.x, rotation.y, rotation.z));
		// create bound frustum
		boundFrustum = new WorldBoundFrustum(frustumTransform, Mat4.IDENTITY);
		// create collection for intersecting nodes
		nodes = new List<Node>();
		Visualizer.Enabled = true;
		sampleDescriptionWindow = new SampleDescriptionWindow();
		sampleDescriptionWindow.createWindow();
	}
	private void Update()
	{
		// show bound frustum
		Visualizer.RenderFrustum(frustumTransform, Mat4.IDENTITY, new vec4(0.0f, 1.0f, 0.0f, 1.0f));
		var status = "Inside bound frustum:";
		// try get nodes inside bound frustum
		bool res = World.GetIntersection(boundFrustum, Node.TYPE.OBJECT_MESH_STATIC, nodes);
		if (res)
		{
			// show nodes names
			foreach (Node n in nodes)
				status += $" {n.Name}";
		}
		else
			status += " empty";
		sampleDescriptionWindow.setStatus(status);
	}
	private void Shutdown()
	{
		Visualizer.Enabled = false;
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## BoundSphereIntersection.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "126bc4774bce576ad6179fc11f21f62c6389fb9b")]
public class BoundSphereIntersection : Component
{
	public dvec3 center = dvec3.ZERO;
	public float radius = 1.0f;
	private WorldBoundSphere boundSphere;
	private List<Node> nodes = null;
	private SampleDescriptionWindow sampleDescriptionWindow;
	private void Init()
	{
		// create bound sphere based on center and radius in world coordinates
		boundSphere = new WorldBoundSphere(new Vec3(center), radius);
		// create collection for intersecting nodes
		nodes = new List<Node>();
		Visualizer.Enabled = true;
		sampleDescriptionWindow = new SampleDescriptionWindow();
		sampleDescriptionWindow.createWindow();
	}
	private void Update()
	{
		// show bound sphere
		Visualizer.RenderBoundSphere(new BoundSphere(new vec3(boundSphere.Center), (float)boundSphere.Radius), Mat4.IDENTITY, new vec4(0.0f, 1.0f, 0.0f, 1.0f));
		// try get nodes inside bound sphere
		var status = "Inside bound sphere:";
		bool res = World.GetIntersection(boundSphere, Node.TYPE.OBJECT_MESH_STATIC, nodes);
		if (res)
		{
			// show nodes names
			foreach (Node n in nodes)
				status += $" {n.Name}";
		}
		else
			status += " empty";
		sampleDescriptionWindow.setStatus(status);
	}
	private void Shutdown()
	{
		Visualizer.Enabled = false;
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## BuoyComponent.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "57c9f45104f1a17c88278eabd7fc540ef31acc15")]
public class BuoyComponent : Component
{
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f)]
	private float mass = 1.0f;
	[ShowInEditor]
	private Node pointFrontCenter = null;
	[ShowInEditor]
	private Node pointBackLeft = null;
	[ShowInEditor]
	private Node pointBackRight = null;
	private ObjectWaterGlobal water = null;
	private Scalar singleValue = 1.0f;
	private Scalar zeroValue = 0.0f;
	private void Init()
	{
		water = World.GetNodeByType((int)Node.TYPE.OBJECT_WATER_GLOBAL) as ObjectWaterGlobal;
		if (water == null)
			Log.ErrorLine("BuoyComponent Init(): can't find ObjectWaterGlobal on scene!");
	}
	private void Update()
	{
		float massLerpC = mass * 0.01f;
		if (mass == 0.0f)
			massLerpC = 0.00001f;
		if (water)
		{
			Mat4 nodeTransform = node.WorldTransform;
			// get water height in entity position
			Vec3 point0 = pointFrontCenter.WorldPosition;
			Vec3 point1 = pointBackLeft.WorldPosition;
			Vec3 point2 = pointBackRight.WorldPosition;
			//create basis by 3 point
			Vec3 tmpZ = MathLib.Normalize(MathLib.Cross(MathLib.Normalize(point1 - point0), MathLib.Normalize(point2 - point0)));
			if (MathLib.Angle(tmpZ, Vec3.UP) > 90)
				tmpZ = -tmpZ;
			Vec3 tmpY = MathLib.Normalize(point1 - point0);
			Vec3 tmpX = tmpY;
			tmpX = MathLib.Cross(tmpY, tmpZ).Normalized;
			tmpY = MathLib.Cross(tmpZ, tmpX).Normalized;
			Mat4 oldBasis = Mat4.IDENTITY;
			oldBasis.Translate = point0;
			oldBasis.SetColumn3(0, tmpX);
			oldBasis.SetColumn3(1, tmpY);
			oldBasis.SetColumn3(2, tmpZ);
			// find height of each point
			Scalar h0 = water.FetchHeight(new Vec3(point0.x, point0.y, 0.0f));
			Scalar h1 = water.FetchHeight(new Vec3(point1.x, point1.y, 0.0f));
			Scalar h2 = water.FetchHeight(new Vec3(point2.x, point2.y, 0.0f));
			Scalar lerpK = Game.IFps * BuoySample.GlobalBuoancy / massLerpC;
			Scalar diff = MathLib.Max(MathLib.Abs((h0 - point0.z) + (h1 - point1.z) + (h2 - point2.z)), singleValue);
			lerpK = MathLib.Clamp(lerpK * diff, zeroValue, singleValue);
			point0.z = MathLib.Lerp(point0.z, h0, lerpK);
			point1.z = MathLib.Lerp(point1.z, h1, lerpK);
			point2.z = MathLib.Lerp(point2.z, h2, lerpK);
			// calculate new basis for changed point
			tmpZ = MathLib.Normalize(MathLib.Cross(MathLib.Normalize(point1 - point0), MathLib.Normalize(point2 - point0)));
			tmpY = MathLib.Normalize(point1 - point0);
			tmpX = MathLib.Cross(tmpY, tmpZ).Normalized;
			tmpY = MathLib.Cross(tmpZ, tmpX).Normalized;
			Mat4 newBasis = Mat4.IDENTITY;
			newBasis.Translate = point0;
			newBasis.SetColumn3(0, tmpX);
			newBasis.SetColumn3(1, tmpY);
			newBasis.SetColumn3(2, tmpZ);
			// calculate translation from old basis to new basis
			Mat4 translationBasis = MathLib.Mul(newBasis, MathLib.Inverse(oldBasis));
			// apply transformation to node transform
			Mat4 newTransform = MathLib.Mul(translationBasis, nodeTransform);
			node.WorldTransform = newTransform;
		}
	}
}
```
---
## BuoySample.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using System.Globalization;
using Unigine;
[Component(PropertyGuid = "0f0af9cec4a3b6d54e651a300eb067cf67f7b2e2")]
public class BuoySample : Component
{
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f)]
	private float buoyancy = 1.0f;
	[ShowInEditor]
	private ObjectWaterGlobal water = null;
	static public float GlobalBuoancy { get; private set; }
	private SampleDescriptionWindow window = new();
	private void Init()
	{
		if (water != null)
		{	
			water.FetchSteepnessQuality = ObjectWaterGlobal.STEEPNESS_QUALITY.HIGH;
			water.FetchAmplitudeThreshold = 0.001f;
		}
		window.createWindow();
		window.addFloatParameter("Buoyancy", null, buoyancy, 0.01f, 1.0f, on_buoyancy_cahnged);
		window.addFloatParameter("Beaufort", null, 4.0f, 0.0f, 12.0f, on_beaufort_cahnged);
		GlobalBuoancy = buoyancy;
	}
	private void Shutdown()
	{
		window.shutdown();
	}
	private void on_buoyancy_cahnged(float value)
	{
		GlobalBuoancy = value;
	}
	private void on_beaufort_cahnged(float value)
	{
		if (water != null)
			water.Beaufort = value;
	}
}
```
---
## Buoyancy.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
using System.Runtime.Intrinsics.X86;
using System.Drawing;
using System.Runtime.InteropServices;
using Microsoft.Win32.SafeHandles;
[Component(PropertyGuid = "391587e0fb8a19bf8c5b5c12c183cc4986a66914")]
public class Buoyancy : Component
{
	[ShowInEditor]
	private Node centerOfMassNode = null;
	[ShowInEditor]
	private ivec2 volumeGridSize = new ivec2(5, 10);
	[ShowInEditor]
	private float waterDensity = 45.0f;
	[ShowInEditor]
	private float waterLinearDamping = 0.02f;
	[ShowInEditor]
	private float waterAngularDamping = 0.02f;
	[ShowInEditor]
	private bool UseVisualizer = false;
	private ObjectWaterGlobal water = null;
	private BodyRigid bodyRigid = null;
	private ShapeBox volumeBoxShape = null;
	private struct VolumePart
	{
		static public vec3 size = vec3.ZERO;
		static public mat3 rotation = mat3.IDENTITY;
		static public float Width => size.x;
		static public float Depth => size.y;
		static public float Height => size.z;
		static public float HalfWidth => size.x * 0.5f;
		static public float HalfDepth => size.y * 0.5f;
		static public float HalfHeight => size.z * 0.5f;
		static public vec3 AxisX => rotation.AxisX;
		static public vec3 AxisY => rotation.AxisY;
		static public vec3 AxisZ => rotation.AxisZ;
		public Vec3 center;
		public float waterHeight;
		public Vec3 AnchorPoint => center - AxisZ * HalfHeight;
		public bool IsUnderWater => waterHeight != 0.0f;
		public float WaterVolume => size.x * size.y * waterHeight;
	}
	private VolumePart[] volumeParts = null;
	private void Init()
	{
		water = World.GetNodeByType((int)Node.TYPE.OBJECT_WATER_GLOBAL) as ObjectWaterGlobal;
		if (water == null)
			Log.ErrorLine("BuoyComponent Init(): can't find ObjectWaterGlobal on scene!");
		bodyRigid = node.ObjectBodyRigid;
		if (bodyRigid == null)
			Log.ErrorLine("BuoyComponent Init(): body rigid is null");
		for (int i = 0; i < bodyRigid.NumShapes; i++)
		{
			Shape s = bodyRigid.GetShape(i);
			if (s.Type == Shape.TYPE.SHAPE_BOX && string.Equals(s.Name, "volume"))
			{
				volumeBoxShape = (ShapeBox)s;
				break;
			}
		}
		if (volumeBoxShape == null)
			Log.ErrorLine("BuoyComponent Init(): volume shape box is null");
		volumeParts = new VolumePart[volumeGridSize.x * volumeGridSize.y];
		if (centerOfMassNode != null)
		{
			bodyRigid.ShapeBased = false;
			bodyRigid.CenterOfMass = new vec3(centerOfMassNode.Position);
		}
		else
		{
			Log.WarningLine("BuoyComponent Init(): center of mass node is null");
		}
	}
	private void Update()
	{
		if (water == null || bodyRigid == null || volumeBoxShape == null)
			return;
		VolumePart.size = new vec3(volumeBoxShape.Size.x / volumeGridSize.x, volumeBoxShape.Size.y / volumeGridSize.y, volumeBoxShape.Size.z);
		VolumePart.rotation = new mat3(volumeBoxShape.Transform);
		Mat4 t = volumeBoxShape.Transform;
		Vec3 size = new Vec3(volumeBoxShape.Size);
		Vec3 start = new Vec3(t.Translate - t.AxisX * size.x * 0.5 - t.AxisY * size.y * 0.5);
		for (int i = 0; i < volumeParts.Length; i++)
		{
			int y = i / volumeGridSize.x;
			int x = i % volumeGridSize.x;
			volumeParts[i].center = start + VolumePart.AxisX * (x * VolumePart.Width + VolumePart.HalfWidth)
				+ VolumePart.AxisY * (y * VolumePart.Depth + VolumePart.HalfDepth);
		}
		for (int i = 0; i < volumeParts.Length; i++)
		{
			float h = water.FetchHeight(new Vec3(volumeParts[i].AnchorPoint.x, volumeParts[i].AnchorPoint.y, 0.0f));
			if (volumeParts[i].AnchorPoint.z < h)
				volumeParts[i].waterHeight = MathLib.Clamp((float)(h - volumeParts[i].AnchorPoint.z), 0.0f, VolumePart.Height);
			else
				volumeParts[i].waterHeight = 0.0f;
		}
		if (UseVisualizer)
		{
			Visualizer.Enabled = true;
			Visualizer.Mode = Visualizer.MODE.ENABLED_DEPTH_TEST_DISABLED;
			for (int i = 0; i < volumeParts.Length; i++)
			{
				Visualizer.RenderPoint3D(volumeParts[i].AnchorPoint, 0.3f, vec4.WHITE);
				vec3 volume_size = new vec3(VolumePart.size.x, VolumePart.size.y, volumeParts[i].waterHeight);
				Mat4 volume_transform = new Mat4(VolumePart.rotation, volumeParts[i].AnchorPoint + VolumePart.AxisZ * volume_size.z * 0.5f);
				if (volumeParts[i].IsUnderWater)
				{
					Visualizer.RenderVector(volumeParts[i].AnchorPoint, volumeParts[i].AnchorPoint + VolumePart.AxisZ * volume_size.z, vec4.RED);
					Visualizer.RenderBox(new vec3(volume_size), volume_transform, vec4.BLUE);
				}
			}
		}
		else
		{
			Visualizer.Enabled = false;
		}
	}
	private void UpdatePhysics()
	{
		if (water == null || bodyRigid == null || volumeBoxShape == null)
			return;
		float water_volume = 0.0f;
		float all_volume = volumeBoxShape.Size.x * volumeBoxShape.Size.y * volumeBoxShape.Size.z;
		for (int i = 0; i < volumeParts.Length; i++)
		{
			if (volumeParts[i].IsUnderWater == false)
				continue;
			vec3 force = -Physics.Gravity * volumeParts[i].WaterVolume * waterDensity;
			force /= volumeParts.Length;
			bodyRigid.AddForce(force);
			vec3 radius = new vec3(volumeParts[i].AnchorPoint + VolumePart.AxisZ * volumeParts[i].waterHeight - bodyRigid.WorldCenterOfMass);
			vec3 torque = MathLib.Cross(radius, force);
			bodyRigid.AddTorque(torque);
			water_volume += volumeParts[i].WaterVolume;
		}
		float coeff = 0.0f;
		if (all_volume > 0.0f)
			coeff= water_volume / all_volume;
		bodyRigid.AddLinearImpulse(bodyRigid.Mass * (-bodyRigid.LinearVelocity * waterLinearDamping * coeff));
		bodyRigid.AddAngularImpulse((-bodyRigid.AngularVelocity * waterAngularDamping * coeff) * MathLib.Inverse(bodyRigid.IWorldInertia));
	}
	public void SetVisualizer(bool enabled)
	{
		UseVisualizer = enabled;
	}
}
```
---
## CameraOrbit.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "0e56a38e74ddd5956a5929f0421bb48f43bf18a2")]
public class CameraOrbit : Component
{
	public CameraControls controls = null;
	public float angularSpeed = 90.0f;
	public float zoomSpeed = 3.0f;
	public float minDistance = 5.0f;
	public float maxDistance = 10.0f;
	public float minVerticalAngle = -89.9f;
	public float maxVerticalAngle = 89.9f;
	public Node target = null;
	private PlayerDummy camera = null;
	private float horizontalAngle = 0.0f;
	private float verticalAngle = 0.0f;
	private float distance = 0.0f;
	private Input.MOUSE_HANDLE init_mouse_handle;
	private void Init()
	{
		camera = node as PlayerDummy;
		if (!camera)
			return;
		if (!target)
			return;
		init_mouse_handle = Input.MouseHandle;
		Input.MouseHandle = Input.MOUSE_HANDLE.GRAB;
		// get camera direction
		vec3 direction = new vec3(target.WorldPosition - camera.WorldPosition);
		direction.Normalize();
		// get projection of direction on XY plane
		vec3 horizontalDirection = direction;
		horizontalDirection.z = 0;
		horizontalDirection.Normalize();
		// get current vertical angle of camera
		verticalAngle = MathLib.Angle(direction, horizontalDirection);
		verticalAngle *= -MathLib.Sign(direction.z);
		verticalAngle = MathLib.Clamp(verticalAngle, minVerticalAngle, maxVerticalAngle);
		// get current horizontal angle of camera
		horizontalAngle = MathLib.Angle(horizontalDirection, vec3.FORWARD);
		horizontalAngle *= MathLib.Sign(direction.x);
		// set target camera direction and position
		distance = minDistance + (maxDistance - minDistance) * 0.5f;
		camera.SetWorldDirection(direction, vec3.UP);
		camera.WorldPosition = target.WorldPosition - direction * distance;
	}
	private void Update()
	{
		if (!camera || !target || controls == null)
			return;
		// update vertical and horizontal angles
		verticalAngle -= controls.TurnUp * angularSpeed * Game.IFps;
		verticalAngle += controls.TurnDown * angularSpeed * Game.IFps;
		verticalAngle = MathLib.Clamp(verticalAngle, minVerticalAngle, maxVerticalAngle);
		horizontalAngle += controls.TurnRight * angularSpeed * Game.IFps;
		horizontalAngle -= controls.TurnLeft * angularSpeed * Game.IFps;
		if (horizontalAngle < -180 || 180 < horizontalAngle)
			horizontalAngle = MathLib.Clamp(-horizontalAngle, -180.0f, 180.0f);
		// calculate new camera direction
		vec3 cameraDirection = vec3.FORWARD * MathLib.RotateZ(horizontalAngle);
		cameraDirection = cameraDirection * MathLib.Rotate(MathLib.Cross(cameraDirection, vec3.UP), verticalAngle);
		// update distance
		distance -= controls.ZoomIn * zoomSpeed * Game.IFps;
		distance += controls.ZoomOut * zoomSpeed * Game.IFps;
		distance = MathLib.Clamp(distance, minDistance, maxDistance);
		// set new direction amd position of camera
		camera.SetWorldDirection(cameraDirection, vec3.UP);
		camera.WorldPosition = target.WorldPosition - cameraDirection * distance;
	}
	private void Shutdown()
	{
		Input.MouseHandle = init_mouse_handle;
	}
}
```
---
## CameraPanning.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "f54dff80b8b1ecdfa39fd4b2ffadac6941338ab4")]
public class CameraPanning : Component
{
	public float defaultLinearSpeed = 0.02f;
	public float mouseSensitivity = 0.08f;
	public float mouseWheelSensitivity = 0.5f;
	private PlayerDummy camera = null;
	private float horizontalAngle = 0.0f;
	private float verticalAngle = 0.0f;
	private Vec3? grabbingPoint = Vec3.ZERO;
	private Vec3 grabbingCameraPos = Vec3.ZERO;
	private vec3 planeNormal = vec3.UP;
	private WorldIntersection intersection = null;
	private ivec2 savedMousePos = ivec2.ZERO;
	private Input.MOUSE_HANDLE init_mouse_handle;
	private void Init()
	{
		camera = node as PlayerDummy;
		if (!camera)
			return;
		init_mouse_handle = Input.MouseHandle;
		Input.MouseHandle = Input.MOUSE_HANDLE.USER;
		intersection = new WorldIntersection();
		vec3 direction = camera.GetWorldDirection();
		// get projection of direction on XY plane
		vec3 horizontalDirection = direction;
		horizontalDirection.z = 0;
		horizontalDirection.Normalize();
		// get current vertical angle of camera
		verticalAngle = MathLib.Angle(direction, horizontalDirection);
		verticalAngle *= -MathLib.Sign(direction.z);
		// get current horizontal angle of camera
		horizontalAngle = MathLib.Angle(horizontalDirection, vec3.FORWARD);
		horizontalAngle *= MathLib.Sign(direction.x);
	}
	private void Update()
	{
		if (!camera)
			return;
		if (Input.IsMouseButtonDown(Input.MOUSE_BUTTON.LEFT))
		{
			// try to get target object
			ivec2 mouse_coord = Input.MousePosition;
			camera.GetDirectionFromMainWindow(out var p0, out var p1, mouse_coord.x, mouse_coord.y);
			Node obj = World.GetIntersection(p0, p1, 0xFFFFFF, intersection);
			grabbingPoint = null;
			if (obj != null)
			{
				// save for camera movement
				grabbingPoint = intersection.Point;
				grabbingCameraPos = camera.WorldPosition;
				planeNormal = camera.ViewDirection;
			}
		}
		if (Input.IsMouseButtonDown(Input.MOUSE_BUTTON.RIGHT))
		{
			// save mouse position and start rotate mode
			savedMousePos = Input.MousePosition;
			Input.MouseHandle = Input.MOUSE_HANDLE.GRAB;
			Input.MouseGrab = true;
		}
		if (Input.IsMouseButtonPressed(Input.MOUSE_BUTTON.LEFT))
		{
			if (grabbingPoint != null)
			{
				// move camera based on target object plane
				ivec2 mouse_coord = Input.MousePosition;
				camera.GetDirectionFromMainWindow(out var p0, out var p1, mouse_coord.x, mouse_coord.y);
				MathLib.LinePlaneIntersection(new vec3(p0), new vec3(p1), new vec3(grabbingPoint.Value), new vec3(planeNormal), out vec3 currentPoint);
				camera.WorldTranslate(grabbingPoint.Value - currentPoint);
			}
			else
				camera.Translate(new Vec3(-Input.MouseDeltaPosition.x * defaultLinearSpeed, Input.MouseDeltaPosition.y * defaultLinearSpeed, 0));
		}
		else if (Input.IsMouseButtonPressed(Input.MOUSE_BUTTON.RIGHT))
		{
			// update vertical and horizontal angles
			verticalAngle += Input.MouseDeltaPosition.y * mouseSensitivity;
			verticalAngle = MathLib.Clamp(verticalAngle, -89.9f, 89.9f);
			horizontalAngle += Input.MouseDeltaPosition.x * mouseSensitivity;
			if (horizontalAngle < -180 || 180 < horizontalAngle)
				horizontalAngle = MathLib.Clamp(-horizontalAngle, -180.0f, 180.0f);
			// calculate new camera direction
			vec3 cameraDirection = vec3.FORWARD * MathLib.RotateZ(horizontalAngle);
			cameraDirection = cameraDirection * MathLib.Rotate(MathLib.Cross(cameraDirection, vec3.UP), verticalAngle);
			// set new direction of camera
			camera.SetWorldDirection(cameraDirection, vec3.UP);
		}
		if (Input.IsMouseButtonUp(Input.MOUSE_BUTTON.RIGHT))
		{
			// stop rotate mode
			Input.MouseHandle = Input.MOUSE_HANDLE.USER;
			Input.MouseGrab = false;
			Input.MousePosition = savedMousePos;
		}
		// zooming
		if (Input.MouseWheel != 0)
			camera.Translate(new Vec3(0, 0, -Input.MouseWheel * mouseWheelSensitivity));
	}
	private void Shutdown()
	{
		Input.MouseHandle = init_mouse_handle;
	}
}
```
---
## CameraPersecutor.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "b524375744d16ccacc23f5a1ead9928c298b2c7b")]
public class CameraPersecutor : Component
{
	public CameraControls controls = null;
	public float angularSpeed = 90.0f;
	public float zoomSpeed = 3.0f;
	public float minDistance = 5.0f;
	public float maxDistance = 10.0f;
	public float minVerticalAngle = -89.9f;
	public float maxVerticalAngle = 89.9f;
	public bool useFixedAngles = false;
	public Node target = null;
	private PlayerDummy camera = null;
	private float horizontalAngle = 0.0f;
	private float verticalAngle = 0.0f;
	private float distance = 0.0f;
	private Input.MOUSE_HANDLE init_mouse_handle;
	private void Init()
	{
		camera = node as PlayerDummy;
		if (!camera)
			return;
		if (!target)
			return;
		init_mouse_handle = Input.MouseHandle;
		Input.MouseHandle = Input.MOUSE_HANDLE.GRAB;
		// get camera direction
		vec3 direction = new vec3(target.WorldPosition - camera.WorldPosition);
		direction.Normalize();
		SetAngles(direction);
		// set target camera direction and position
		distance = minDistance + (maxDistance - minDistance) * 0.5f;
		camera.SetWorldDirection(direction, vec3.UP);
		camera.WorldPosition = target.WorldPosition - direction * distance;
	}
	private void Update()
	{
		if (!camera || !target || controls == null)
			return;
		// get direction on target
		vec3 direction = new vec3(target.WorldPosition - camera.WorldPosition);
		// update current distance
		distance = direction.Length;
		distance -= controls.ZoomIn * zoomSpeed * Game.IFps;
		distance += controls.ZoomOut * zoomSpeed * Game.IFps;
		distance = MathLib.Clamp(distance, minDistance, maxDistance);
		// calculate current angles
		if (!useFixedAngles)
			SetAngles(direction.Normalized);
		// update current angles
		verticalAngle -= controls.TurnUp * angularSpeed * Game.IFps;
		verticalAngle += controls.TurnDown * angularSpeed * Game.IFps;
		verticalAngle = MathLib.Clamp(verticalAngle, minVerticalAngle, maxVerticalAngle);
		horizontalAngle += controls.TurnRight * angularSpeed * Game.IFps;
		horizontalAngle -= controls.TurnLeft * angularSpeed * Game.IFps;
		// calculate new camera direction
		direction = vec3.FORWARD * MathLib.RotateZ(horizontalAngle);
		direction = direction * MathLib.Rotate(MathLib.Cross(direction, vec3.UP), verticalAngle);
		// set new direction amd position of camera
		camera.SetWorldDirection(direction, vec3.UP);
		camera.WorldPosition = target.WorldPosition - direction * distance;
	}
	private void SetAngles(vec3 currentDirection)
	{
		// get projection of direction on XY plane
		vec3 horizontalDirection = currentDirection;
		horizontalDirection.z = 0;
		horizontalDirection.Normalize();
		// get current vertical angle of camera
		verticalAngle = MathLib.Angle(currentDirection, horizontalDirection);
		verticalAngle *= -MathLib.Sign(currentDirection.z);
		verticalAngle = MathLib.Clamp(verticalAngle, minVerticalAngle, maxVerticalAngle);
		// get current horizontal angle of camera
		horizontalAngle = MathLib.Angle(horizontalDirection, vec3.FORWARD);
		horizontalAngle *= MathLib.Sign(currentDirection.x);
	}
	private void Shutdown()
	{
		Input.MouseHandle = init_mouse_handle;
	}
}
```
---
## CameraPersecutorTarget.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "5db3a2ae745d8a35e96d1610b553722fd7c179c2")]
public class CameraPersecutorTarget : Component
{
	public float radius = 5.0f;
	public float speed = 0.2f;
	private void Update()
	{
		float x = radius * MathLib.Cos(Game.Time * speed);
		float y = radius * MathLib.Sin(Game.Time * speed);
		node.WorldPosition = new Vec3(x, y, 0);
	}
}
```
---
## CameraSelection.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using System.Numerics;
using Unigine;
#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
[Component(PropertyGuid = "ca0b0bfe1193c345127a5be0e620ddbfac1b5012")]
public class CameraSelection : Component
{
	private Vec3 selectedObjectsBoundSpherePosition;
	public Vec3 Center
	{
		get 
		{
			UpdateBoundSphere();
			return selectedObjectsBoundSpherePosition; 
		}
	}
	private Scalar selectedObjectsBoundSphereRadius;
	public Scalar BoundRadius
	{
		get 
		{
			UpdateBoundSphere();
			return selectedObjectsBoundSphereRadius; 
		}
	}
	public bool Selection
	{
		get { return selectedObjects.Count != 0; }
	}
	private bool isSelection;
	private ivec2 selectionStartMousePosition;
	private vec2 upperLeftSelectionCorner;
	private vec2 bottomRightSelectionCorner;
	private WorldBoundFrustum frustum;
	private List<Unigine.Object> selectedObjects = new List<Unigine.Object>(0);
	private void UpdateBoundSphere()
	{
		WorldBoundSphere bs = new WorldBoundSphere();
		for(int i =0; i < selectedObjects.Count; i++)
		{
			bs.Expand(selectedObjects[i].WorldBoundSphere);
		}
		selectedObjectsBoundSpherePosition = bs.Center;
		selectedObjectsBoundSphereRadius = bs.Radius * 4.0f;
	}
	private void Update()
	{
		if(!Unigine.Console.Active)
		{
			Visualizer.Enabled = true;
			if(Input.IsMouseButtonDown(Input.MOUSE_BUTTON.LEFT))
			{
				isSelection = true;
				selectionStartMousePosition = Input.MousePosition - WindowManager.MainWindow.ClientPosition;
			}
			if(isSelection)
			{
				ivec2 windowSize = WindowManager.MainWindow.ClientRenderSize;
				upperLeftSelectionCorner.x = selectionStartMousePosition.x * 1.0f / windowSize.x;
				upperLeftSelectionCorner.y = 1.0f - selectionStartMousePosition.y * 1.0f / windowSize.y;
				ivec2 currentMousePosition = Input.MousePosition - WindowManager.MainWindow.ClientPosition;
				bottomRightSelectionCorner.x = currentMousePosition.x * 1.0f / windowSize.x;
				bottomRightSelectionCorner.y = 1.0f - currentMousePosition.y * 1.0f / windowSize.y;
				Visualizer.RenderRectangle(new vec4(upperLeftSelectionCorner.x , upperLeftSelectionCorner.y, bottomRightSelectionCorner.x, bottomRightSelectionCorner.y), vec4.GREEN);
			}
			if(Input.IsMouseButtonUp(Input.MOUSE_BUTTON.LEFT))
			{
				Player camera = Game.Player;
				foreach(var it in selectedObjects)
				{
					var cameraUnitSelectionComponent = GetComponent<CameraUnitSelection>(it);
					cameraUnitSelectionComponent.Selected = false;
				}
				isSelection = false;
				ivec2 selectionFinishPosition = Input.MousePosition - WindowManager.MainWindow.ClientPosition;
				float width = MathLib.Abs(selectionStartMousePosition.x - selectionFinishPosition.x);
				float height = MathLib.Abs(selectionStartMousePosition.y - selectionFinishPosition.y);
				mat4 perspective = camera.GetProjectionFromMainWindow(selectionStartMousePosition.x, selectionStartMousePosition.y, selectionFinishPosition.x, selectionFinishPosition.y);
				frustum.Set(perspective, MathLib.Inverse(camera.WorldTransform));
				World.GetIntersection(frustum, selectedObjects);
				if(selectedObjects.Count == 0)
				{
					ivec2 mouse = Input.MousePosition;
					vec3 dir = camera.GetDirectionFromMainWindow(mouse.x, mouse.y);
					Unigine.Object obj = World.GetIntersection(camera.WorldPosition, camera.WorldPosition + dir * 10000, ~0);
					if(obj != null)
					{
						selectedObjects.Add(obj);
					}
				}
				for (int i =0; i < selectedObjects.Count; i++)
				{
					var cameraUnitSelectionComponent = GetComponent<CameraUnitSelection>(selectedObjects[i]);
					if (cameraUnitSelectionComponent)
					{
						cameraUnitSelectionComponent.Selected = true;
					}
					else
					{
						selectedObjects.Remove(selectedObjects[i]);
						i--;
					}
				}
				UpdateBoundSphere();
			}
		}
	}
}
```
---
## CameraSpectator.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "14e937489e55720774e387ba153af3d19cfb9afc")]
public class CameraSpectator : Component
{
	public float speed = 3.0f;
	public float angularSpeed = 90.0f;
	public CameraControls controls = null;
	private PlayerDummy camera = null;
	private float horizontalAngle = 0.0f;
	private float verticalAngle = 0.0f;
	private void Init()
	{
		camera = node as PlayerDummy;
		if (!camera)
			return;
		Input.MouseHandle = Input.MOUSE_HANDLE.GRAB;
		vec3 direction = camera.GetWorldDirection();
		// get projection of direction on XY plane
		vec3 horizontalDirection = direction;
		horizontalDirection.z = 0;
		horizontalDirection.Normalize();
		// get current vertical angle of camera
		verticalAngle = MathLib.Angle(direction, horizontalDirection);
		verticalAngle *= -MathLib.Sign(direction.z);
		// get current horizontal angle of camera
		horizontalAngle = MathLib.Angle(horizontalDirection, vec3.FORWARD);
		horizontalAngle *= MathLib.Sign(direction.x);
	}
	private void Update()
	{
		if (!camera || controls == null)
			return;
		vec3 forward = camera.GetWorldDirection();
		vec3 right = camera.GetWorldDirection(MathLib.AXIS.X);
		vec3 up = camera.GetWorldDirection(MathLib.AXIS.Y);
		// get moving direction of camera
		vec3 targetVelocityDirection = vec3.ZERO;
		targetVelocityDirection += forward * controls.Forward;
		targetVelocityDirection -= forward * controls.Backward;
		targetVelocityDirection += right * controls.Right;
		targetVelocityDirection -= right * controls.Left;
		targetVelocityDirection += up * controls.Up;
		targetVelocityDirection -= up * controls.Down;
		// move camera in target direction
		float currentSpeed = speed * controls.Acceleration;
		if (targetVelocityDirection.Length2 > 0)
			targetVelocityDirection.Normalize();
		camera.WorldTranslate(new Vec3(targetVelocityDirection) * currentSpeed * Game.IFps);
		// update vertical and horizontal angles
		verticalAngle -= controls.TurnUp * angularSpeed * Game.IFps;
		verticalAngle += controls.TurnDown * angularSpeed * Game.IFps;
		verticalAngle = MathLib.Clamp(verticalAngle, -89.9f, 89.9f);
		horizontalAngle += controls.TurnRight * angularSpeed * Game.IFps;
		horizontalAngle -= controls.TurnLeft * angularSpeed * Game.IFps;
		if (horizontalAngle < -180 || 180 < horizontalAngle)
			horizontalAngle = MathLib.Clamp(-horizontalAngle, -180.0f, 180.0f);
		// calculate new camera direction
		vec3 cameraDirection = vec3.FORWARD * MathLib.RotateZ(horizontalAngle);
		cameraDirection = cameraDirection * MathLib.Rotate(MathLib.Cross(cameraDirection, vec3.UP), verticalAngle);
		// set new direction of camera
		camera.SetWorldDirection(cameraDirection, vec3.UP);
	}
}
```
---
## CameraTopDown.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using System.Runtime.CompilerServices;
using Unigine;
#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
[Component(PropertyGuid = "a965eb3d6b4b3378ac7a60772cb4436ae9cc9bc1")]
public class CameraTopDown : Component
{
	public float phi = 180.0f;
	public float theta = 40.0f;
	public vec2 thetaMinMax = new vec2(10.0f, 70.0f);
	public float distance = 25.0f;
	public vec2 distanceMinMax = new vec2(5.0f, 45.0f);
	public float zoomSpeed = 5.0f;
	private Player camera;
	private WorldIntersection intersection = new WorldIntersection();
	private vec3 previousMouseToIntersectionPointVector;
	private vec3 currentMouseToIntersectionPointVector;
	private bool isPreviousHooked;
	private float currentPhi = 180.0f;
	private float targetPhi = 180.0f;
	private float currentTheta = 40.0f;
	private float targetTheta = 40.0f;
	private float maxTheta = 70.0f;
	private float minTheta = 10.0f;
	private Scalar currentDistance = 25.0f;
	private Scalar targetDistance = 25.0f;
	private Scalar maxDistance = 45.0f;
	private Scalar minDistance = 5.0f;
	private float interpolationFactor = 1.0f;
	private Vec3 currentCameraPivotPosition;
	private Vec3 targetCameraPivotPosition;
	private float degreesPerUnit = 1.0f;
	private Input.MOUSE_HANDLE init_mouse_handle;
	CameraSelection selection;
	private void Init()
	{
		targetPhi = phi;
		currentPhi = targetPhi;
		targetTheta = theta;
		currentTheta = targetTheta;
		minTheta = thetaMinMax.x;
		maxTheta = thetaMinMax.y;
		targetDistance = distance;
		currentDistance = targetDistance;
		minDistance = distanceMinMax.x;
		maxDistance = distanceMinMax.y;
		interpolationFactor = zoomSpeed;
		if (maxDistance != minDistance)
			degreesPerUnit =  (maxTheta - minTheta) / (float)(maxDistance - minDistance);
		else
			degreesPerUnit = 0.0f;
		camera = node as Player;
		if(camera == null)
		{
			Log.Error("CameraTopDown::init(): camera is not valid\n");
			return;
		}
		targetCameraPivotPosition = node.WorldPosition;
		targetCameraPivotPosition.z = 2.0f;
		currentCameraPivotPosition = targetCameraPivotPosition;
		init_mouse_handle = Input.MouseHandle;
		Input.MouseHandle = Input.MOUSE_HANDLE.USER;
		vec3 cameraViewDirection = new quat(vec3.UP, currentPhi) * vec3.FORWARD;
		cameraViewDirection = new quat(MathLib.Cross(vec3.UP, cameraViewDirection), -currentTheta) * cameraViewDirection * -1;
		cameraViewDirection.Normalize();
		camera.ViewDirection = cameraViewDirection;
		camera.WorldPosition = currentCameraPivotPosition - cameraViewDirection * currentDistance;
		NodeDummy logic = new NodeDummy();
		selection = ComponentSystem.AddComponent<CameraSelection>(logic);
	}
	private void Update()
	{
		if (!camera)
			return;
		if(!Unigine.Console.Active)
		{
			if(Input.IsMouseButtonDown(Input.MOUSE_BUTTON.MIDDLE))
			{
				ivec2 mouse = Input.MousePosition;
				vec3 rayDir = camera.GetDirectionFromMainWindow(mouse.x, mouse.y);
				Unigine.Object obj = World.GetIntersection(camera.WorldPosition, camera.WorldPosition + rayDir * 10000, ~0, intersection);
				if (obj != null)
				{
					isPreviousHooked = true;
					previousMouseToIntersectionPointVector = new vec3(intersection.Point - camera.WorldPosition);
				}
				else
				{
					isPreviousHooked = false;
				}
			}
			int mouseAxis = Input.MouseWheel;
			if(mouseAxis != 0)
			{
				targetDistance = MathLib.Clamp(targetDistance - mouseAxis, minDistance, maxDistance);
				targetTheta = MathLib.Clamp(targetTheta - mouseAxis, minTheta, maxTheta);
			}
			if(Input.IsKeyPressed(Input.KEY.Q))
			{
				targetPhi -= 50.0f * Game.IFps;
			}
			if (Input.IsKeyPressed(Input.KEY.E))
			{
				targetPhi += 50.0f * Game.IFps;
			}
			if(Input.IsKeyPressed(Input.KEY.F) && selection.Selection)
			{
				targetDistance = selection.BoundRadius;
				targetCameraPivotPosition = selection.Center;
				targetCameraPivotPosition.z = 2.0f;
				targetDistance = MathLib.Clamp(targetDistance, minDistance, maxDistance);
				targetTheta = MathLib.Clamp(minTheta + (float)(targetDistance - minDistance) * degreesPerUnit, minTheta, maxTheta);
			}
			if(Input.IsMouseButtonPressed(Input.MOUSE_BUTTON.MIDDLE) && isPreviousHooked)
			{
				ivec2 mouse = Input.MousePosition;
				currentMouseToIntersectionPointVector = camera.GetDirectionFromMainWindow(mouse.x, mouse.y);
				currentMouseToIntersectionPointVector *= previousMouseToIntersectionPointVector.z / currentMouseToIntersectionPointVector.z;
				vec3 displacement = currentMouseToIntersectionPointVector - previousMouseToIntersectionPointVector;
				targetCameraPivotPosition -= displacement;
				currentCameraPivotPosition -= displacement;
				previousMouseToIntersectionPointVector = currentMouseToIntersectionPointVector;
			}
			else
			{
				vec3 forward = new quat(vec3.UP, targetPhi) * vec3.FORWARD * -1;
				forward.Normalize();
				vec3 right = new quat(vec3.UP, targetPhi) * vec3.RIGHT * -1;
				right.Normalize();
				ivec2 mouse = Input.MousePosition;
				ivec2 windowPos = WindowManager.MainWindow.Position;
				ivec2 windowSize = WindowManager.MainWindow.RenderSize;
				if (mouse.x < windowPos.x + 10)
					targetCameraPivotPosition -= right * 10.0f * Game.IFps;
				if (mouse.y < windowPos.y + 10)
					targetCameraPivotPosition += forward * 10.0f * Game.IFps;
				if (mouse.x > windowPos.x + windowSize.x - 10)
					targetCameraPivotPosition += right * 10.0f * Game.IFps;
				if (mouse.y > windowPos.y + windowSize.y - 10)
					targetCameraPivotPosition -= forward * 10.0f * Game.IFps;
			}
		}
		currentPhi = MathLib.Lerp(currentPhi, targetPhi, interpolationFactor * Game.IFps);
		currentTheta = MathLib.Lerp(currentTheta, targetTheta, interpolationFactor * Game.IFps);
		currentDistance = MathLib.Lerp(currentDistance, targetDistance, interpolationFactor * Game.IFps);
		currentCameraPivotPosition = MathLib.Lerp(currentCameraPivotPosition, targetCameraPivotPosition, interpolationFactor * Game.IFps);
	}
	private void PostUpdate()
	{
		vec3 cameraViewDirection = new quat(vec3.UP, currentPhi) * vec3.FORWARD;
		cameraViewDirection = new quat(MathLib.Cross(vec3.UP, cameraViewDirection), -currentTheta) * cameraViewDirection * -1;
		cameraViewDirection.Normalize();
		camera.ViewDirection = cameraViewDirection;
		camera.WorldPosition = currentCameraPivotPosition - cameraViewDirection * currentDistance;
	}
	private void Shutdown()
	{
		Input.MouseHandle = init_mouse_handle;
	}
	public void SetPosition(vec3 pos)
	{
		targetCameraPivotPosition = pos;
		targetCameraPivotPosition.z = 2.0f;
		currentCameraPivotPosition = targetCameraPivotPosition;
	}
	public void SetTargetPosition(vec3 pos)
	{
		targetCameraPivotPosition = pos;
		targetCameraPivotPosition.z = 2.0f;
	}
	public void SetDistance(float dist)
	{
		targetDistance = dist;
		targetDistance = MathLib.Clamp(targetDistance, minDistance, maxDistance);
		targetTheta = MathLib.Clamp(minTheta + (float)(targetDistance - minDistance) * degreesPerUnit, minTheta, maxTheta);
		currentDistance = targetDistance;
		currentTheta = targetTheta;
	}
	public void SetTargetDistance(float dist)
	{
		targetDistance = dist;
		targetDistance = MathLib.Clamp(targetDistance, minDistance, maxDistance);
		targetTheta = MathLib.Clamp(minTheta + (float)(targetDistance - minDistance) * degreesPerUnit, minTheta, maxTheta);
	}
}
```
---
## CameraUnitPathControl.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
[Component(PropertyGuid = "ca65a6e04ff5acbf0d63d3124d5bd749c800f814")]
public class CameraUnitPathControl : Component
{
	public float speed = 13;
	public float torque = 1;
	public Node pathNode;
	public float epsD = 0.1f;
	private List<Mat4> path = new List<Mat4>(0);
	private int currentPathIndex = 0;
	private int dir = 1;
	private float rotAcceleration;
	private void Init()
	{
		float minDistance = 1e+9f;
		if (pathNode != null)
		{
			for (int i = 0; i < pathNode.NumChildren; i++)
			{
				var p = pathNode.GetChild(i).Transform;
				float distance = (float)MathLib.Length2(node.WorldPosition - p.Translate);
				if (distance < minDistance)
				{
					currentPathIndex = i;
					minDistance = distance;
				}
				path.Add(p);
			}
		}
	}
	private void Update()
	{
		if (path.Count == 0)
			return;
		var currentPosition = node.WorldPosition;
		var currentRot = node.GetWorldRotation();
		var route = path[currentPathIndex];
		float ifps = Game.IFps;
		if (ifps == 0)
			return;
		vec3 ddir = new vec3(route.Translate - currentPosition);
		ddir.Normalize();
		float d = MathLib.Dot(currentRot.Mat3.AxisX, ddir);
		if (MathLib.IsNaN(d))
			d = 0;
		rotAcceleration += (d > rotAcceleration ? ifps : -ifps);
		currentRot = currentRot * new quat(vec3.UP, torque * ifps * -rotAcceleration);
		currentPosition = currentPosition + new vec3(currentRot.Mat3.AxisY * ifps * speed);
		node.WorldPosition = currentPosition;
		node.SetWorldRotation(currentRot);
		float distanceToTarget = (float)MathLib.Length(route.Translate - currentPosition);
		if(distanceToTarget < epsD)
		{
			currentPathIndex += dir;
			if (currentPathIndex == path.Count)
				currentPathIndex = 0;
			if(currentPathIndex == -1)
				currentPathIndex = path.Count - 1;
		}
	}
}
```
---
## CameraUnitSelection.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "4b55a8aba9afa60e0de1f6150101d0705d83290f")]
public class CameraUnitSelection : Component
{
	public AssetLink selectionCircle;
	public vec3 offset = new vec3(0f, 0f, 0.01f);
	private bool selected = false;
	public bool Selected
	{
		get { return selected; }
		set 
		{
			selected = value;
			if (selectionCircleNode != null) 
				selectionCircleNode.Enabled = selected; 
		}
	}
	private Node selectionCircleNode = null;
	private void Init()
	{
		selectionCircleNode = World.LoadNode(selectionCircle.Path);
		if(selectionCircleNode == null)
		{
			Log.Error($"UnitSelectionCircle::init(): cannot load node {selectionCircle.Path}\n");
			return;
		}
		selectionCircleNode.Parent = node;
		selectionCircleNode.Position = offset;
		selectionCircleNode.Enabled = false;
	}
}
```
---
## Canvas.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "2eae4a513d0b30b551ded61db2489c740c493f43")]
public class Canvas : Component
{
	private WidgetCanvas canvas;
	void Init()
	{
		// create canvas
		canvas = new WidgetCanvas();
		canvas.SetLineColor(create_line(canvas, 0, 200.0f, 200.0f, 100.0f, 3, 360.0f), new vec4(0.0f, 0.0f, 1.0f, 1.0f));
		canvas.SetLineColor(create_line(canvas, 0, 200.0f, 200.0f, 100.0f, 4, 360.0f), new vec4(0.0f, 1.0f, 0.0f, 1.0f));
		canvas.SetLineColor(create_line(canvas, 0, 200.0f, 200.0f, 100.0f, 5, 360.0f), new vec4(1.0f, 0.0f, 0.0f, 1.0f));
		canvas.SetLineColor(create_line(canvas, 0, 800.0f, 400.0f, 100.0f, 16, 360.0f * 9.0f), new vec4(1.0f, 1.0f, 1.0f, 1.0f));
		canvas.SetPolygonColor(create_polygon(canvas, 0, 600.0f, 200.0f, 100.0f, 6, 360.0f), new vec4(1.0f, 0.0f, 0.0f, 1.0f));
		canvas.SetPolygonColor(create_polygon(canvas, 1, 600.0f, 200.0f, 100.0f, 3, 360.0f), new vec4(0.0f, 0.0f, 1.0f, 1.0f));
		canvas.SetPolygonColor(create_polygon(canvas, 0, 400.0f, 400.0f, 100.0f, 8, 360.0f), new vec4(0.0f, 1.0f, 0.0f, 1.0f));
		canvas.SetPolygonColor(create_polygon(canvas, 1, 400.0f, 400.0f, 100.0f, 4, 360.0f), new vec4(1.0f, 0.0f, 0.0f, 1.0f));
		create_text(canvas, 0, 200.0f - 64.0f, 200.0f - 30.0f, "This is C# canvas text");
		Gui.GetCurrent().AddChild(canvas, Gui.ALIGN_OVERLAP | Gui.ALIGN_BACKGROUND);
		Engine.BackgroundUpdate = Engine.BACKGROUND_UPDATE.BACKGROUND_UPDATE_RENDER_NON_MINIMIZED;
	}
	void Update()
	{
		// get gui
		Gui gui = Gui.GetCurrent();
		float fov = 2.0f;
		float time = Game.Time;
		float x = gui.Width / 2.0f;
		float y = gui.Height / 2.0f;
		canvas.Transform = MathLib.Translate(new vec3(x, y, 0.0f)) * MathLib.Perspective(fov, 1.0f, 0.01f, 100.0f) * MathLib.RotateY((float)Math.Sin(time)) * MathLib.RotateX((float)Math.Cos(time * 0.5f)) * MathLib.Translate(new vec3(-x, -y, -1.0f / (float)Math.Tan(fov * MathLib.DEG2RAD * 0.5f)));
	}
	void Shutdown()
	{
		canvas.DeleteLater();
	}
	private int create_line(WidgetCanvas canvas, int order, float x, float y, float radius, int num, float angle)
	{
		int line = canvas.AddLine(order);
		for (int i = 0; i <= num; i++)
		{
			float s = (float)Math.Sin(angle / num * MathLib.DEG2RAD * i) * radius + x;
			float c = (float)Math.Cos(angle / num * MathLib.DEG2RAD * i) * radius + y;
			canvas.AddLinePoint(line, new vec3(s, c, 0.0f));
		}
		return line;
	}
	private int create_polygon(WidgetCanvas canvas, int order, float x, float y, float radius, int num, float angle)
	{
		int polygon = canvas.AddPolygon(order);
		for (int i = 0; i < num; i++)
		{
			float s = (float)Math.Sin(angle / num * MathLib.DEG2RAD * i) * radius + x;
			float c = (float)Math.Cos(angle / num * MathLib.DEG2RAD * i) * radius + y;
			canvas.AddPolygonPoint(polygon, new vec3(s, c, 0.0f));
		}
		return polygon;
	}
	private int create_text(WidgetCanvas canvas, int order, float x, float y, string str)
	{
		int text = canvas.AddText(order);
		canvas.SetTextPosition(text, new vec2(x, y));
		canvas.SetTextText(text, str);
		return text;
	}
}
```
---
## ClusterSample.cs
```csharp
﻿using Unigine;
#region Math Variables
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Mat4 = Unigine.mat4;
#endif
#endregion
[Component(PropertyGuid = "c02b0b3e3e273985a354dd20eff8edffd8cda352")]
public class ClusterSample : Component
{
	[Parameter(Title = "Cluster Node")]
	public ObjectMeshCluster cluster = null;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.INTERSECTION)]
	public int intersectionMask = 1;
	private WorldIntersection intersection = new WorldIntersection();
	private SampleDescriptionWindow sampleDescriptionWindow = new SampleDescriptionWindow();
	// z-coordinate for meshes
	private const float OFFSET_Z = 0.5f;
	private void Init()
	{
		if (!cluster)
			Log.Error("ClusterSample.Init(): cannot get Cluster property\n");
		InitGui();
	}
	private void Update()
	{
		if (Unigine.Console.Active)
			return;
		// remove a mesh from the cluster or add a new one to it
		if (Unigine.Input.IsMouseButtonDown(Unigine.Input.MOUSE_BUTTON.LEFT))
		{
			// select a mesh or empty space using the mouse
			ivec2 mouse = Unigine.Input.MousePosition;
			Vec3 p0 = Game.Player.WorldPosition;
			Vec3 p1 = p0 + Game.Player.GetDirectionFromMainWindow(mouse.x, mouse.y) * 100;
			// check for an intersection with the cluster or ground
			Unigine.Object obj = World.GetIntersection(p0, p1, intersectionMask, intersection);
			if (obj)
			{
				// if obj is ObjectMeshCluster then remove a mesh
				if (obj == cluster)
				{
					int num = intersection.Instance;
					cluster.RemoveMeshTransform(num);
				}
				else 
				{
					// create transformation matrix for the new mesh
					Vec3 point = intersection.Point;
					point.z = OFFSET_Z;
					// add a single mesh in local space
					int new_index = cluster.AddMeshTransform();
					cluster.SetMeshTransform(new_index, new mat4(cluster.IWorldTransform * MathLib.Translate(point)));
					// add multiple meshes in global space
					// Mat4[] tr = {MathLib.Translate(point)};
					// cluster.AppendMeshes(tr);
				}
			}
			UpdateGui();
		}
	}
	private void Shutdown()
	{
		ShutdownGui();
	}
	private void InitGui()
	{
		sampleDescriptionWindow.createWindow();
		sampleDescriptionWindow.setStatus($"Number of meshes in the cluster: {cluster.NumMeshes}");
	}
	private void UpdateGui()
	{
		string status = $"Number of meshes in the cluster: {cluster.NumMeshes}";
		sampleDescriptionWindow.setStatus(status);
	}
	private void ShutdownGui()
	{
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## ClutterConverter.cs
```csharp
﻿using System.Collections.Generic;
using Unigine;
#region Math Variables
#if UNIGINE_DOUBLE
using Mat4 = Unigine.dmat4;
#else
using Mat4 = Unigine.mat4;
#endif
#endregion
[Component(PropertyGuid = "5f874774f719cc4302ac04c66068156db1e677e8")]
public class ClutterConverter : Component
{
	[Parameter(Title = "Clutter Node")]
	public ObjectMeshClutter clutter;
	public ObjectMeshStatic clusterParent;
	private bool is_convertred = false;
	private ObjectMeshCluster cluster;
	public void ConvertToCluster()
	{
		RemoveCluster();
		cluster = ConvertMesh(clutter);
		if (cluster)
		{
			is_convertred = true;
			// move cluster to another parent
			cluster.Parent = clusterParent;
		}
	}
	public void generateClutter()
	{
		clutter.Seed = Unigine.Random.Get().Int();
	}
	private void RemoveCluster()
	{
		if (!is_convertred)
			return;
		cluster.DeleteLater();
		is_convertred = false;
	}
	private ObjectMeshCluster ConvertMesh(ObjectMeshClutter clutter)
	{
		ObjectMeshCluster cluster = new ObjectMeshCluster(clutter.MeshPath);
		cluster.Name = clutter.Name + "_Cluster";
		// copy the hierarchy
		cluster.Parent = clutter.Parent;
		// copy node transformation
		cluster.WorldTransform = clutter.WorldTransform;
		// copy necessary parameters for surfaces
		cluster.VisibleDistance = clutter.VisibleDistance;
		cluster.FadeDistance = clutter.FadeDistance;
		int suf_num = clutter.NumSurfaces;
		for (int suf_index = 0; suf_index < suf_num; suf_index++)
		{
			cluster.SetEnabled(cluster.IsEnabled(suf_index), suf_index);
			cluster.SetViewportMask(cluster.GetViewportMask(suf_index), suf_index);
			cluster.SetShadowMask(clutter.GetShadowMask(suf_index), suf_index);
			cluster.SetCastShadow(clutter.GetCastShadow(suf_index), suf_index);
			cluster.SetCastWorldShadow(clutter.GetCastWorldShadow(suf_index), suf_index);
			cluster.SetBakeToEnvProbe(clutter.GetBakeToEnvProbe(suf_index), suf_index);
			cluster.SetBakeToGI(clutter.GetBakeToGI(suf_index), suf_index);
			cluster.SetCastEnvProbeShadow(clutter.GetCastEnvProbeShadow(suf_index), suf_index);
			cluster.SetShadowMode(clutter.GetShadowMode(suf_index), suf_index);
			cluster.SetMinVisibleDistance(clutter.GetMinVisibleDistance(suf_index), suf_index);
			cluster.SetMaxVisibleDistance(clutter.GetMaxVisibleDistance(suf_index), suf_index);
			cluster.SetMinFadeDistance(clutter.GetMinFadeDistance(suf_index), suf_index);
			cluster.SetMaxFadeDistance(clutter.GetMaxFadeDistance(suf_index), suf_index);
			cluster.SetMinParent(clutter.GetMinParent(suf_index), suf_index);
			cluster.SetMaxParent(clutter.GetMaxParent(suf_index), suf_index);
			cluster.SetIntersection(clutter.GetIntersection(suf_index), suf_index);
			cluster.SetIntersectionMask(clutter.GetIntersectionMask(suf_index), suf_index);
			cluster.SetCollision(clutter.GetCollision(suf_index), suf_index);
			cluster.SetCollisionMask(clutter.GetCollisionMask(suf_index), suf_index);
			cluster.SetPhysicsIntersection(clutter.GetPhysicsIntersection(suf_index), suf_index);
			cluster.SetPhysicsIntersectionMask(clutter.GetPhysicsIntersectionMask(suf_index),
			suf_index);
			cluster.SetSoundOcclusion(clutter.GetSoundOcclusion(suf_index), suf_index);
			cluster.SetSoundOcclusionMask(clutter.GetSoundOcclusionMask(suf_index), suf_index);
			cluster.SetPhysicsFriction(clutter.GetPhysicsFriction(suf_index), suf_index);
			cluster.SetPhysicsRestitution(clutter.GetPhysicsRestitution(suf_index), suf_index);
			cluster.SetMaterial(clutter.GetMaterial(suf_index), suf_index);
			cluster.SetSurfaceProperty(clutter.GetSurfaceProperty(suf_index), suf_index);
		}
		// copy transforms from the clutter to the cluster
		List<Mat4> transforms = new List<Mat4>();
		clutter.CreateClutterTransforms();
		if(!clutter.GetClutterWorldTransforms(transforms))
		{
			Log.Warning("ClutterConverter.ConvertMesh(): empty set of transforms\n");
			return cluster;
		}
		cluster.CreateMeshes(transforms.ToArray());
		return cluster;
	}
}
```
---
## ClutterSample.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "6734288337043406c8f5fbad0718a928f0fd7a67")]
public class ClutterSample : Component
{
	public Node converterNode = null;
	private ClutterConverter clutter_converter = null;
	private SampleDescriptionWindow sampleDescriptionWindow = new SampleDescriptionWindow();
	private void Init()
	{
		clutter_converter = GetComponent<ClutterConverter>(converterNode);
		if (!clutter_converter)
			Log.Error("ClutterSample.Init(): cannot find ClutterConverter component!\n");
		InitGui();
	}
	private void Shutdown()
	{
		ShutdownGui();
	}
	private void InitGui()
	{
		sampleDescriptionWindow.createWindow();
		var parameters = sampleDescriptionWindow.getParameterGroupBox();
		WidgetHBox hbox = new WidgetHBox(5, 0);
		parameters.AddChild(hbox, Gui.ALIGN_CENTER);
		WidgetButton button = new WidgetButton("Generate Clutter");
		button.EventClicked.Connect(GenerateButtonCallback);
		hbox.AddChild(button, Gui.ALIGN_LEFT);
		button = new WidgetButton("Convert to Cluster");
		button.EventClicked.Connect(ConvertButtonCallback);
		hbox.AddChild(button, Gui.ALIGN_LEFT);
	}
	private void ShutdownGui()
	{
		sampleDescriptionWindow.shutdown();
	}
	private void GenerateButtonCallback()
	{
		clutter_converter.generateClutter();
	}
	private void ConvertButtonCallback()
	{
		clutter_converter.ConvertToCluster();
	}
}
```
---
## ComponentParameters.cs
```csharp
﻿using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "b73d8487fe8b783a79152a51f3475f7c5a3634ab")]
public class ComponentParameters : Component
{
	// all standard types with public modifier are shown in the editor
	public int intVariable;
	public float floatVariable;
	public double doubleVariable;
	public bool boolVariable;
	public string stringVariable;
	public ivec2 integerVec2Variable;
	public vec2 floatVec2Variable;
	public dvec2 doubleVec2Variable;
	public ivec3 integerVec3Variable;
	public vec3 floatVec3Variable;
	public dvec3 doubleVec3Variable;
	public ivec4 integerVec4Variable;
	public vec4 floatVec4Variable;
	public dvec4 doubleVec4Variable;
	public enum MyEnum
	{
		Item0,
		Item1,
		Item2
	}
	public MyEnum enumVariable;
	public Material materialVariable;
	public Property propertyVariable;
	public Node nodeVariable;
	// all descendants of Node also are shown in the editor
	public NodeDummy nodeDummyVariable;
	public ObjectMeshStatic meshStaticVariable;
	// all components and their descendants with a public modifier are shown in the editor
	public Component componentVariable;
	public ComponentParameters inheritedComponentVariable;
	// you can also specify an abstract component as a parameter -
	// it will accept any component derived from this abstract component
	public AbstractComponent abstractComponentVariable;
	// and you can even specify a C# interface as a parameter -
	// it will accept any component that implements this interface
	public Interface interfaceComponentVariable;
	// empty arrays and arrays with values are shown in the editor
	public int[] intEmptyArray;
	public int[] intValuesArray = { 1, 2, 3 };
	// empty Lists and Lists with values are shown in the editor
	public List<int> intEmptyList;
	public List<int> intValuesList = new List<int> { 1, 2, 3 };
	// internal, protected and private variables are not shown in the editor in the general case
	#pragma warning disable CS0414, CS0169, CS0649
	internal int hiddenInternalIntVariable;
	protected int hiddenProtectedIntVariable;
	private int hiddenPrivateIntVariable;
	// but you can use ShowInEditor attribute to show internal, protected and private variables
	[ShowInEditor]
	internal int internalIntVariable;
	[ShowInEditor]
	private int privateIntVariable;
	[ShowInEditor]
	protected int protectedIntVariable;
	// also you can hide public variable in the editor using the HideInEditor attribute
	[HideInEditor]
	public int hiddenIntVariable = 1;
	#pragma warning restore CS0414, CS0169, CS0649
	// nested classes are shown in the editor
	public class MyClass
	{
		public int myClassParameter = 1;
	}
	public MyClass myClassVaribale;
	// and nested structures are shown it the editor too
	public struct MyStruct
	{
		public int myStructParameter;
	}
	public MyStruct myStructVariable;
	// you can use the Parameter attribute to change the display name of the variable,
	// add a tooltip for the variable, and organize part of the variables into groups
	[Parameter(Group = "Group Name", Title = "New Name", Tooltip = "Tooltip Text")]
	public int parameterInt;
	// you can use the ParameterMask attribute to display the int variable as various types of masks
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.COLLISION)]
	public int collisionMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.EXCLUSION)]
	public int exclusionMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.FIELD)]
	public int fieldMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.GENERAL)]
	public int generalMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.INTERSECTION)]
	public int intersectionMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.MATERIAL)]
	public int materialMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.NAVIGATION)]
	public int navigationMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.OBSTACLE)]
	public int obstacleMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.PHYSICAL)]
	public int physicalMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.PHYSICS_INTERSECTION)]
	public int physicsIntersectionMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.SHADOWS)]
	public int shadowsMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.SOUND_OCCLUSION)]
	public int soundOcclusionMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.SOUND_REVERB)]
	public int soundReverbMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.SOUND_SOURCE)]
	public int soundSourceMask = 1;
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.VIEWPORT)]
	public int viewportMask = 1;
	// you can use the ParameterSlider attribute to display variables of type int,
	// float and double with various restrictions: minimum and maximum values, the ability
	// to change the minimum and maximum values, the ability to only reduce the minimum value,
	// the ability to only increase the maximum value, changes  the slider to a logarithmic
	[ParameterSlider(Min = -100.0f, Max = 100.0f)]
	public int intSlider = 0;
	[ParameterSlider(Min = -100.0f, Max = 100.0f)]
	public float floatSlider = 0.0f;
	[ParameterSlider(Min = -100.0f, Max = 100.0f)]
	public double doubleSlider = 0.0f;
	// you can use the ParameterColor attribute to set the vec4 variable as color using the color widget
	[ParameterColor]
	public vec4 color = vec4.ONE;
	// link to any asset
	public AssetLink assetLink;
	// you can use the ParamterAsset attribute to set an asset link filter
	[ParameterAsset(Filter = ".world")]
	public AssetLink assetLinkFilter;
	// link to node asset
	public AssetLinkNode assetLinkNode;
	// you can use the ParameterFile attribute to set the string variable as a path to a file with fixed extensions
	[ParameterFile(Filter = ".node")]
	public string file = "";
	// you can use the ParameterMaterial for Material
	[ParameterMaterial(ParentGUID = "83eb006a9be234c16ec9872c5bd2589b5c8e353e")]
	public Material material;
	// you can use the ParameterProperty for Property
	[ParameterProperty(InternalOnly = false, ParentGUID = "b73d8487fe8b783a79152a51f3475f7c5a3634ab")]
	public Property property;
	// you can use ParameterSwitch attribute to set the int variable as the index of one of the items
	[ParameterSwitch(Items = "switch_item_0,switch_item_1,switch_item_2")]
	public int switchVariable = 1;
	// you can use ParameterCondition to selectively display component parameters if certain conditions are met
	public bool showVariable = true;
	[ParameterCondition(nameof(showVariable), 1)]
	public int conditionVariable;
	// you can use curve for non-linear value dependencies
	public Curve2d curve2D = new Curve2d();
	// This method is called by the Engine, when the component is ready for initialization
	protected override void OnReady()
	{
		Log.Message("OnReady\n");
	}
	// This method is called by the Engine, when the component and node become enabled and active
	protected override void OnEnable()
	{
		Log.Message("OnEnable\n");
	}
	// This method is called by the Engine, when the component and node become disabled
	protected override void OnDisable()
	{
		Log.Message("OnDisable\n");
	}
	// Engine calls this function on world initialization
	// Using the Order parameter of the Method attribute, you can change the order of method calls
	// Also, using the InvokeDisabled parameter, you can configure the method call for the disabled component
	[Method(InvokeDisabled = true, Order = 0)]
	private void Init()
	{
		// create and initialize all necessary resources
		Log.Message("Init\n");
	}
	// Engine calls this function before the update() and the postUpdate()
	private void UpdateSyncThread()
	{
		// specify all logic functions you want to be executed before the Update()
		Log.Message("UpdateSyncThread\n");
	}
	// Engine calls this function after execution of all updateSyncThread() functions
	private void UpdateAsyncThread()
	{
		// specify all logic functions you want to be called every frame independently of the rendering framerate
		Log.Message("UpdateAsyncThread\n");
	}
	// Engine calls this function before updating each render frame
	private void Update()
	{
		// specify all logic functions you want to be called every frame
		Log.Message("Update\n");
	}
	// Engine calls this function before rendering each render frame
	private void PostUpdate()
	{
		// correct behavior according to the updated node states in the same frame
		Log.Message("PostUpdate\n");
	}
	// Engine calls this function before updating each physics frame
	private void UpdatePhysics()
	{
		// simulate physics: perform continuous operations
		Log.Message("UpdatePhysics\n");
	}
	// Engine calls this function before updating each render frame
	private void Swap()
	{
		Log.Message("Swap\n");
	}
	// Engine calls this function on the world shutdown
	private void Shutdown()
	{
		// perform cleanup on component shutdown
		Log.Message("Shutdown\n");
	}
	// Using the MethodInit attribute, you can create any number of Init methods
	[MethodInit(Order = 1)]
	private void MyInit()
	{
		Log.Message("MyInit\n");
	}
	// Using the MethodUpdateSyncThread attribute, you can create any number of UpdateSyncThread methods
	[MethodUpdateSyncThread]
	private void MyUpdateSyncThread()
	{
		Log.Message("MyUpdateSyncThread\n");
	}
	// Using the MethodUpdateAsyncThread attribute, you can create any number of UpdateAsyncThread methods
	[MethodUpdateAsyncThread]
	private void MyUpdateAsyncThread()
	{
		Log.Message("MyUpdateAsyncThread\n");
	}
	// Using the MethodUpdate attribute, you can create any number of Update methods
	[MethodUpdate(Order = 1)]
	private void MyUpdate()
	{
		Log.Message("MyUpdate\n");
	}
	// Using the MethodPostUpdate attribute, you can create any number of PostUpdate methods
	[MethodPostUpdate(Order = 1)]
	private void MyPostUpdate()
	{
		Log.Message("MyPostUpdate\n");
	}
	// Using the MethodUpdatePhysics attribute, you can create any number of UpdatePhysics methods
	[MethodUpdatePhysics(Order = 1)]
	private void MyUpdatePhysics()
	{
		Log.Message("MyUpdatePhysics\n");
	}
	// Using the MethodSwap attribute, you can create any number of Swap methods
	[MethodSwap(Order = 1)]
	private void MySwap()
	{
		Log.Message("MySwap\n");
	}
	// Using the MethodShutdown attribute, you can create any number of Shutdown methods
	[MethodShutdown(Order = 1)]
	private void MyShutdown()
	{
		Log.Message("MyShutdown\n");
	}
}
```
---
## ConsoleSample.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using System.Globalization;
using Unigine;
using static Unigine.Console;
[Component(PropertyGuid = "4a09e0b28e49cc455a0eb7d09be530dfd3a97092")]
public class ConsoleSample : Component
{
	public Node controllable_node;
	void Init()
	{
		// show console
		Unigine.Console.Active = true;
		// here we add a new console command and add callback to process this command from our c# code
		Unigine.Console.AddCommand("my_console_command", "my_console_command decription", command_callback);
		// run console command
		Unigine.Console.Run("my_console_command \"Hello from C++\"");
		Unigine.Console.AddCommand("control_node", "With this command you can control node, pass desired position through the arguments",
		 move_node_callback);
		if (!controllable_node)
		{
			Log.Message("ConsoleSample.Init(): No node was provided!\n");
		}
		Log.Message("To move the node, you can use control_node command and 3 arguments to specify node position\n");
	}
	void Shutdown()
	{
		Unigine.Console.RemoveCommand("my_console_command");
		Unigine.Console.RemoveCommand("control_node");
		Unigine.Console.Active = false;
	}
	private void command_callback(int argc, string[] argv)
	{
		Log.Message("my_console_command(): called\n");
		for (int i = 0; i < argc; i++)
			Log.Message("{0}: {1}\n", i, argv[i]);
	}
	private void move_node_callback(int argc, string[] argv)
	{
		if (!controllable_node)
			return;
		vec3 node_position = new vec3();
		if (argc != 4)
		{
			Log.Warning("control_node was called incorrectly, you need to pass 3 coordinates to move the node\n");
			return;
		}
		var parse_arg = (int index) =>
		{
			string a_value = argv[index];
			bool is_number = float.TryParse(a_value, NumberStyles.Any, CultureInfo.InvariantCulture, out float res);
			float value = is_number ? res : 0.0f;
			return value;
		};
		node_position.x = parse_arg(1);
		node_position.y = parse_arg(2);
		node_position.z = parse_arg(3);
		controllable_node.WorldPosition = node_position;
	}
};
```
---
## DayNightSwitchSample.cs
```csharp
﻿using System;
using System.Globalization;
using Unigine;
using System.Reflection.Metadata;
#region Math Variables
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Mat4 = Unigine.mat4;
#endif
#endregion
[Component(PropertyGuid = "05d43b0308d1815f029839d37ebd85f0aac46582")]
public class DayNightSwitchSample : Component
{
	[ShowInEditor]
	[Parameter(Title = "Sun Controller")]
	private SunController sun = null;
	[ShowInEditor]
	[Parameter(Title = "Day night swithcer")]
	private DayNightSwitcher switcher = null;
	private SampleDescriptionWindow window = null;
	private WidgetLabel timeLabel = null;
	private WidgetSlider timeSlider = null;
	void Init()
	{
		InitComponents();
		const int maxMinutes = 24 * 60;
		const int noonInMinutes = 12 * 60;
		window = new SampleDescriptionWindow();
		window.createWindow();
		WidgetGroupBox parameters = window.getParameterGroupBox();
		Widget grid = parameters.GetChild(0);
		//========== Global time=========//
		var label = new WidgetLabel("Global time:");
		label.Width = 100;
		grid.AddChild(label, Gui.ALIGN_LEFT);
		timeSlider = new WidgetSlider();
		timeSlider.MinValue = 0;
		timeSlider.MaxValue = maxMinutes;
		timeSlider.Width = 200;
		timeSlider.ButtonWidth = 20;
		timeSlider.ButtonHeight = 20;
		grid.AddChild(timeSlider, Gui.ALIGN_LEFT);
		timeLabel = new WidgetLabel("00:00");
		timeLabel.Width = 20;
		grid.AddChild(timeLabel, Gui.ALIGN_LEFT);
		timeSlider.EventChanged.Connect(() =>
		{
			int time = timeSlider.Value;
			String timeString = GetTimeString(time);
			timeLabel.Text = timeString;
			sun.SetTime(time * 60);
		});
		timeSlider.Value = noonInMinutes;
		//========== Threshold Slider=========//
		window.addFloatParameter("Zenith angle threshold:", "Zenith angle threshold", (float)switcher.SunZenitThreshold, 70.0f, 120.0f, (float val) =>
		{
			float floatValue = val / 100.0f;
		});
		//========== Morning Slider=========//
		label = new WidgetLabel("Morning time bound:");
		label.Width = 100;
		grid.AddChild(label, Gui.ALIGN_LEFT);
		var morningSlider = new WidgetSlider();
		morningSlider.MinValue = 0;
		morningSlider.MaxValue = maxMinutes;
		morningSlider.Width = 200;
		morningSlider.ButtonWidth = 20;
		morningSlider.ButtonHeight = 20;
		grid.AddChild(morningSlider, Gui.ALIGN_LEFT);
		var morningTimeLabel = new WidgetLabel("00:00");
		morningTimeLabel.Width = 20;
		grid.AddChild(morningTimeLabel, Gui.ALIGN_LEFT);
		morningSlider.EventChanged.Connect(() =>
		{
			int time = morningSlider.Value;
			String timeString = GetTimeString(time);
			morningTimeLabel.Text = timeString;
			int hours = time / 60;
			int minutes = time % 60;
			switcher.SetMorningControlTime(new ivec2(hours, minutes));
		});
		morningSlider.Value = switcher.GetControlMorningTime();
		//========== Evening Slider=========//
		label = new WidgetLabel("Evening time bound:");
		label.Width = 100;
		grid.AddChild(label, Gui.ALIGN_LEFT);
		var eveningSlider = new WidgetSlider();
		eveningSlider.MinValue = 0;
		eveningSlider.MaxValue = maxMinutes;
		eveningSlider.Width = 200;
		eveningSlider.ButtonWidth = 20;
		eveningSlider.ButtonHeight = 20;
		grid.AddChild(eveningSlider, Gui.ALIGN_LEFT);
		var eveningTimeLabel = new WidgetLabel("00:00");
		eveningTimeLabel.Width = 20;
		grid.AddChild(eveningTimeLabel, Gui.ALIGN_LEFT);
		eveningSlider.EventChanged.Connect(() =>
		{
			int time = eveningSlider.Value;
			String timeString = GetTimeString(time);
			eveningTimeLabel.Text = timeString;
			int hours = time / 60;
			int minutes = time % 60;
			switcher.SetEveningControlTime(new ivec2(hours, minutes));
		});
		eveningSlider.Value = switcher.GetControlEveningTime();
		//========== Control type Switch =========//
		WidgetComboBox controlType = new WidgetComboBox();
		parameters.AddChild(controlType, Gui.ALIGN_LEFT);
		controlType.AddItem("Zenith angle threshold");
		controlType.AddItem("Time");
		controlType.EventChanged.Connect(() =>
		{
			int item = controlType.CurrentItem;
			var type = (DayNightSwitcher.CONTROL_TYPE)(item);
			switcher.SetControlType(type);
		});
		//========== Timescale slider =========//
		window.addIntParameter("Timescale:", "Timescale", (int)sun.Timescale, 0, 60 * 60 * 12, (int val) =>
		{
			sun.Timescale = val;
		});
		//========== Continuous rotation checkbox =========//
		WidgetCheckBox continuousCheckBox = new WidgetCheckBox("Continuous sun rotation");
		parameters.AddChild(continuousCheckBox, Gui.ALIGN_LEFT);
		continuousCheckBox.EventChanged.Connect(() =>
		{
			sun.IsContinuous = continuousCheckBox.Checked;
		});
		continuousCheckBox.Checked = sun.IsContinuous;
		sun.EventOnTimeChanged.Connect((double time) =>
		{
			// updting slider to current time coming from sun controller
			sun.EventOnTimeChanged.Enabled = false;
			timeSlider.Value = ((int)time / 60);
			sun.EventOnTimeChanged.Enabled = true;
		});
	}
	void Update()
	{
		Visualizer.RenderVector(new Vec3(0, 0, 2), new Vec3(0, 0, 7), vec4.RED,0.5f);
		Visualizer.RenderVector(new Vec3(0, 0, 2), new Vec3(0, 0, 2) + /*Vec3 */sun.node.GetWorldDirection(MathLib.AXIS.Z) * 5, vec4.BLUE, 0.5f);
	}
	private void Shutdown()
	{
		window.shutdown();
	}
	public string GetTimeString(int minutes)
	{
		int hours = minutes / 60;
		int leftMinutes = minutes % 60;
		String str = "";
		str += hours.ToString("00", CultureInfo.InvariantCulture);
		str += ":";
		str += leftMinutes.ToString("00", CultureInfo.InvariantCulture);
		return str;
	}
	private void InitComponents()
	{
		if (!sun)
		{
			Log.Error("DayNightSwitchSample.InitComponents SunController is not assigned!\n");
		}
		if (!switcher)
		{
			Log.Error("DayNightSwitchSample.InitComponents DayNightSwithcer is not assigned!\n");
		}
		Visualizer.Enabled = true;
	}
}
```
---
## DayNightSwitcher.cs
```csharp
﻿using System;
using System.Collections.Generic;
using Unigine;
#region Math Variables
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Mat4 = Unigine.mat4;
#endif
#endregion
[Component(PropertyGuid = "2552d56023b38ff9364cb250207756c3868aee8b")]
public class DayNightSwitcher : Component
{
	public enum CONTROL_TYPE
	{
		Zenith = 0,
		Time = 1,
	};
	[ShowInEditor]
	[Parameter(Title = "Switch control type")]
	private CONTROL_TYPE switchControlType = CONTROL_TYPE.Zenith;
	[ShowInEditor]
	[Parameter(Title = "Sun controller")]
	private SunController sun = null;
	[ShowInEditor]
	[Parameter(Title = "Sun zenit threshold")]
	private double sunZenitThreshold = 85.0d;
	public double SunZenitThreshold { get { return sunZenitThreshold; } }
	[ShowInEditor]
	[Parameter(Title = "Morning time bound")]
	private ivec2 timeMorning = new ivec2(7, 30);
	[ShowInEditor]
	[Parameter(Title = "Evening time bound")]
	private ivec2 timeEvening = new ivec2(19, 30);
	public ivec2 TimeEvening { get { return timeEvening; } }
	[ShowInEditor]
	[Parameter(Title = "Emission material parameter name")]
	private String emissionMaterialParameterName = "emission_scale";
	[ShowInEditor]
	[Parameter(Title = "Materials that enabled during day")]
	private List<Material> materialsDayEnabled = new List<Material>();
	[ShowInEditor]
	[Parameter(Title = "Materials that enabled during night")]
	private List<Material> materialsNightEnabled = new List<Material>();
	[ShowInEditor]
	[Parameter(Title = "Nodes that enabled during day")]
	private List<Node> nodesDayEnabled = new List<Node>();
	[ShowInEditor]
	[Parameter(Title = "Nodes that enabled during night")]
	private List<Node> nodesNightEnabled = new List<Node>();
	private Dictionary<UGUID, float> defaultEmissionScale = new Dictionary<UGUID, float>();
	private int isDay = -1;
	private void Init()
	{
		if (!sun)
		{
			Log.Error("DayNightSwitchSample::init can't find SunController component on the sun node!\n");
		}
		sun.EventOnTimeChanged.Connect(OnTimeChange);
		foreach (var mat in materialsDayEnabled)
		{
			int param = mat.FindParameter(emissionMaterialParameterName);
			if (param == -1)
			{
				Log.Error("DayNightSwitchSample::init materials_day_enabled got wrong material without emission!\n");
			}
			defaultEmissionScale.Add(mat.GUID, mat.GetParameterFloat(param));
		}
		foreach (var mat in materialsNightEnabled)
		{
			int param = mat.FindParameter(emissionMaterialParameterName);
			if (param == -1)
			{
				Log.Error("DayNightSwitchSample::init materials_night_enabled got wrong material without emission!\n");
			}
			defaultEmissionScale.Add(mat.GUID, mat.GetParameterFloat(param));
		}
		OnTimeChange();
	}
	private void Shutdown()
	{
		defaultEmissionScale.Clear();
		sun.EventOnTimeChanged.Disconnect(OnTimeChange);
	}
	public void SetControlType(CONTROL_TYPE type)
	{
		switchControlType = type;
		OnTimeChange();
	}
	public void SetZenithThreshold(float value)
	{
		value = MathLib.Clamp(value, 0.0f, 180.0f);
		sunZenitThreshold = value;
		OnTimeChange();//so that new threshold would apply instantly
	}
	public void SetMorningControlTime(ivec2 timeMorning)
	{
		this.timeMorning = timeMorning;
		OnTimeChange();//so that change would apply instantly
	}
	public void SetEveningControlTime(ivec2 timeEvening)
	{
		this.timeEvening = timeEvening;
		OnTimeChange();//so that change would apply instantly
	}
	public int GetControlMorningTime() { return timeMorning.x * 60 + timeMorning.y; }
	public int GetControlEveningTime() { return timeEvening.x * 60 + timeEvening.y; }
	private void OnTimeChange()
	{
		switch (switchControlType)
		{
			case CONTROL_TYPE.Zenith:
				{//Zenth angle
					bool day = true;// is day after sun is rotated
					if (sun.node)
					{
						double currentAngle = MathLib.Angle(Vec3.UP, sun.node.GetWorldDirection(MathLib.AXIS.Z));
						day = currentAngle < sunZenitThreshold;
					}
					if ((day ? 1 : 0) != isDay)//isDay initialized value is -1 so that we are always swithcing nodes for the firts check depending on world initial setup
					{
						SwitchNodes(day);
						isDay = day ? 1 : 0;
					}
					break;
				}
			case CONTROL_TYPE.Time:
				{// Time
					int time = ((int)sun.Time / 60);
					bool day = time > (timeMorning.x * 60 + timeMorning.y)
							&& time < (timeEvening.x * 60 + timeEvening.y); // is day after sun is rotated
					if ((day ? 1 : 0) != isDay)//isDay initialized value is -1 so that we are always swithcing nodes for the firts time depending on world initial setup
					{
						SwitchNodes(day);
						isDay = day ? 1 : 0;
					}
					break;
				}
			default:
				break;
		}
	}
	private void SwitchNodes(bool day)
	{
		//Materials
		if (materialsDayEnabled != null)
		{
			foreach (var mat in materialsDayEnabled)
			{
				if (mat != null)
				{
					mat.SetParameterFloat(emissionMaterialParameterName, day ? defaultEmissionScale[mat.GUID] : 0);
				}
				else
				{
					Log.Warning("DayNightSwitcher::on_time_changed: day materail is null\n");
				}
			}
		}
		foreach (var mat in materialsNightEnabled)
		{
			if (mat != null)
			{
				mat.SetParameterFloat(emissionMaterialParameterName, !day ? defaultEmissionScale[mat.GUID] : 0);
			}
			else
			{
				Log.Warning("DayNightSwitcher::on_time_changed: day materail is null\n");
			}
		}
		//Nodes
		foreach (var node in nodesDayEnabled)
		{
			if (node)
			{
				node.Enabled = day;
			}
			else
			{
				Log.Warning("DayNightSwitcher::on_time_changed: got null node \n");
			}
		}
		foreach (var node in nodesNightEnabled)
		{
			if (node)
			{
				node.Enabled = !day;
			}
			else
			{
				Log.Warning("DayNightSwitcher::on_time_changed: got null node \n");
			}
		}
	}
}
```
---
## Dialog.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "b8f11faa3a1d55832158966ef902f118084e7538")]
public class Dialog : Component
{
	public AssetLink image;
	private WidgetWindow window;
	void Init()
	{
		Gui gui = Gui.GetCurrent();
		window = new WidgetWindow(gui, "Dialogs", 4, 4);
		window.Flags = Gui.ALIGN_OVERLAP | Gui.ALIGN_CENTER;
		var button_0 = new WidgetButton(gui, "Message");
		button_0.Flags = Gui.ALIGN_EXPAND;
		button_0.EventClicked.Connect(() => button_message_clicked("DialogMessage", "Message"));
		window.AddChild(button_0);
		var button_1 = new WidgetButton(gui, "File");
		button_1.Flags = Gui.ALIGN_EXPAND;
		button_1.EventClicked.Connect(() => button_file_clicked("DialogFile", "./"));
		window.AddChild(button_1);
		var button_2 = new WidgetButton(gui, "Color");
		button_2.Flags = Gui.ALIGN_EXPAND;
		button_2.EventClicked.Connect(() => button_color_clicked("DialogColor", new vec4(1.0f)));
		window.AddChild(button_2);
		if (image.Path.Length > 0)
		{
			var button_3 = new WidgetButton(gui, "Image");
			button_3.Flags = Gui.ALIGN_EXPAND;
			button_3.EventClicked.Connect(() => button_image_clicked("DialogImage", image.Path));
			window.AddChild(button_3);
		}
		else
			Log.Warning("Dialogs.Init(): image parameter not specified.");
		window.Arrange();
		gui.AddChild(window);
		Console.Onscreen = true;
	}
	void Shutdown()
	{
		window.DeleteLater();
		Console.Onscreen = false;
	}
	private void dialog_ok_clicked(WidgetDialog dialog, int type)
	{
		Log.Message("{0} ok clicked\n", dialog.Text);
		if (type == 1)
			Log.Message("{0}\n", (dialog as WidgetDialogFile).File);
		if (type == 2)
			Log.Message("{0}\n", (dialog as WidgetDialogColor).WebColor);
		Gui.GetCurrent().RemoveChild(dialog);
	}
	private void dialog_cancel_clicked(Widget widget, WidgetDialog dialog)
	{
		Log.Message("{0} cancel clicked\n", dialog.Text);
		Gui.GetCurrent().RemoveChild(dialog);
	}
	private void dialog_show(WidgetDialog dialog, int type)
	{
		dialog.GetOkButton().EventClicked.Connect(() => dialog_ok_clicked(dialog, type));
		dialog.GetCancelButton().EventClicked.Connect(widget => dialog_cancel_clicked(widget, dialog));
		Gui.GetCurrent().AddChild(dialog, Gui.ALIGN_OVERLAP | Gui.ALIGN_CENTER);
		dialog.SetPermanentFocus();
	}
	private void button_message_clicked(string str, string message)
	{
		var dialog_message = new WidgetDialogMessage(Gui.GetCurrent(), str);
		dialog_message.MessageText = message;
		dialog_show(dialog_message, 0);
	}
	private void button_file_clicked(string str, string path)
	{
		var dialog_file = new WidgetDialogFile(Gui.GetCurrent(), str);
		dialog_file.Path = path;
		dialog_show(dialog_file, 1);
	}
	private void button_color_clicked(string str, vec4 color)
	{
		var dialog_color = new WidgetDialogColor(Gui.GetCurrent(), str);
		dialog_color.Color = color;
		dialog_show(dialog_color, 2);
	}
	private void button_image_clicked(string str, string name)
	{
		var dialog_image = new WidgetDialogImage(Gui.GetCurrent(), str);
		dialog_image.Texture = name;
		dialog_show(dialog_image, 3);
	}
}
```
---
## EventsAdvancedSample.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "eb20e44b63e9ec504c235d116762b533005661fd")]
public class EventsAdvancedSample : Component
{
	public Event<float> EventRotateX { get { return rotate_x_event; } }
	public Event<float> EventRotateY { get { return rotate_y_event; } }
	public Event<float> EventRotateZ { get { return rotate_z_event; } }
	public Event<float, float, float, EventsAdvancedSample> EventRotate { get { return rotate_event; } }
	private readonly EventInvoker<float> rotate_x_event = new();
	private readonly EventInvoker<float> rotate_y_event = new();
	private readonly EventInvoker<float> rotate_z_event = new();
	private readonly EventInvoker<float, float, float, EventsAdvancedSample> rotate_event = new();
	public vec3 rotation_speed = new(3.0f, 3.0f, 3.0f);
	private void Update()
	{
		if (Console.Active)
			return;
		if (Input.IsKeyPressed(Input.KEY.T))
			rotate_x_event.Run(rotation_speed.x);
		if (Input.IsKeyPressed(Input.KEY.Y))
			rotate_y_event.Run(rotation_speed.y);
		if (Input.IsKeyPressed(Input.KEY.U))
			rotate_z_event.Run(rotation_speed.z);
		if (Input.IsKeyPressed(Input.KEY.I))
			rotate_event.Run(rotation_speed.x, rotation_speed.y, rotation_speed.z, this);
	}
}
```
---
## EventsAdvancedUnit.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "fbbbeabda92b13cabac199b32f43a802e6e6cbbf")]
public class EventsAdvancedUnit : Component
{
	public EventsAdvancedSample input_manager;
	private EventConnection rotate_x_connection;
	private EventConnection rotate_y_connection;
	private EventConnection rotate_z_connection;
	private EventConnection rotate_connection;
	private readonly EventConnections connections = new();
	private EventDelegate<float, float, float> rotate_delegate;
	private void Init()
	{
		if (input_manager is null)
			return;
		rotate_delegate += rotate;
		// Connect to method with additional arguments
		rotate_x_connection = input_manager.EventRotateX.Connect(rotateNode, 0.0f, 0.0f, node);
		rotate_y_connection = input_manager.EventRotateY.Connect(connections, rotateNodeY, node);
		// Connect to lambda
		rotate_z_connection = input_manager.EventRotateZ.Connect(connections, 
		angle => {
			node.Rotate(0, 0, angle);
		});
		// Connect to delegate with discarding last argument
		rotate_connection = input_manager.EventRotate.Connect(rotate_delegate);
	}
	private void Shutdown()
	{
		rotate_x_connection.Disconnect();
		rotate_y_connection.Disconnect();
		rotate_z_connection.Disconnect();
		rotate_connection.Disconnect();
	}
	private void rotate(float angleX, float angleY, float angleZ)
	{
		rotateNode(angleX, angleY, angleZ, node);
	}
	private static void rotateNode(float angleX, float angleY, float angleZ, Node node)
	{
		node?.Rotate(angleX, angleY, angleZ);
	}
	private static void rotateNodeY(float angle, Node node)
	{
		rotateNode(0, angle, 0, node);
	}
}
```
---
## ExternalPackageSample.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using System.Drawing;
using System.IO;
using System.Runtime.InteropServices;
using Unigine;
using static Unigine.Image;
[Component(PropertyGuid = "9f731f078717404b7f37b701d6863f28d2364131")]
public class ExternalPackageSample : Component
{
	private int num_files = 64;
	private ExternalPackage package;
	void Init()
	{
		package = new ExternalPackage(num_files);
		FileSystem.AddExternPackage("package", package);
		for (int i = 0; i < num_files; i += 1)
		{
			ObjectMeshStatic mesh_static = new ObjectMeshStatic(String.Format("{0}.mesh", i));
			vec3 position = random_vec3(new vec3(4.0f, 4.0f, 2.0f)) + vec3.UP * 2.0f;
			quat rotation = new quat(Game.GetRandomFloat(0.0f, 360.0f), Game.GetRandomFloat(0.0f, 360.0f), Game.GetRandomFloat(0.0f, 360.0f));
			mesh_static.WorldTransform = new mat4(rotation, position);
		}
	}
	void Shutdown()
	{
		FileSystem.RemovePackage("package");
		package.Dispose();
	}
	private vec3 random_vec3(vec3 size)
	{
		return random_vec3(-size * 0.5f, size * 0.5f);
	}
	private vec3 random_vec3(vec3 from, vec3 to)
	{
		return new vec3 {
			Game.GetRandomFloat(from.x, to.x),
			Game.GetRandomFloat(from.y, to.y),
			Game.GetRandomFloat(from.z, to.z)
		};
	}
}
class ExternalPackage : Package, IDisposable
{
	private Unigine.File file;
	private int num_files = 0;
	private bool disposed = false;
	public ExternalPackage(int num_files)
	{
		this.num_files = num_files;
		file = new Unigine.File();
		Mesh mesh = new Mesh();
		mesh.AddBoxSurface("box", vec3.ONE);
		string path = World.Path + @"/" + ".." + @"/" + ".temporary" + @"/" + "box.mesh";
		if (mesh.Save(path) > 0)
		{
			file.Open(path, "rb");
		}
	}
	// list of files
	public override int GetNumFiles()
	{
		return num_files;
	}
	public override string GetFilePath(int num)
	{
		return String.Format("{0}.mesh", num);
	}
	// select file
	public override bool SelectFile(string name, out ulong size)
	{
		bool exists = FindFile(name) == 1 ? true : false;
		size = 0;
		if (exists)
			size = file.GetSize();
		return exists;
	}
	// read file
	public override bool ReadFile(IntPtr data, ulong size)
	{
		if (!file.IsOpened)
			return false;
		file.SeekSet(0);
		ulong written = file.Read(data, size);
		return written == size;
	}
	public override int FindFile(string name)
	{
		for (int i = 0; i < num_files; i += 1)
		{
			if (String.Format("{0}.mesh", i) == name)
				return 1;
		}
		return 0;
	}
	public override ulong GetFileSize(int num)
	{
		return file.GetSize();
	}
	~ExternalPackage()
	{
		Dispose(disposing: false);
	}
	protected virtual void Dispose(bool disposing)
	{
		if (disposed)
			return;
		if (disposing)
		{
			file.Dispose();
		}
		disposed = true;
	}
	public void Dispose()
	{
		Dispose(true);
		GC.SuppressFinalize(this);
	}
}
```
---
## FFPSample.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "3272b1603a8da87d8f3af3cef30e0951d5b1fe40")]
public class FFPSample : Component
{
	EventConnections connection = new EventConnections();
	void Init()
	{
		Engine.EventEndPluginsGui.Connect(connection, Render);
	}
	void Render()
	{
		// screen size
		int width = 0;
		int height = 0;
		float time = Game.Time;
		EngineWindow main_window = WindowManager.MainWindow;
		if (main_window != null)
		{
			width = main_window.ClientRenderSize.x;
			height = main_window.ClientRenderSize.y;
		}
		float radius = height / 2.0f;
		Ffp.Enable(Ffp.MODE_SOLID);
		Ffp.SetOrtho(width, height);
		// begin triangles
		Ffp.BeginTriangles();
		// vertex colors
		uint[] colors = { 0xffff0000, 0xff00ff00, 0xff0000ff };
		// create vertices
		int num_vertices = 16;
		for (int i = 0; i < num_vertices; i++)
		{
			float angle = MathLib.PI2 * i / (num_vertices - 1) - time;
			float x = width / 2 + (float)Math.Sin(angle) * radius;
			float y = height / 2 + (float)Math.Cos(angle) * radius;
			Ffp.AddVertex(x, y);
			Ffp.SetColor(colors[i % 3]);
		}
		// create indices
		for (int i = 1; i < num_vertices; i++)
		{
			Ffp.AddIndex(0);
			Ffp.AddIndex(i);
			Ffp.AddIndex(i - 1);
		}
		// end triangles
		Ffp.EndTriangles();
		Ffp.Disable();
	}
	void Shutdown()
	{
		connection.DisconnectAll();
	}
}
```
---
## FMODCoreSample.cs
```csharp
﻿using System;
using Unigine;
using Unigine.Plugins.FMOD;
[Component(PropertyGuid = "95d526786760889d8859911ec8ef8bf20e579a00")]
public class FMODCoreSample : Component
{
	private bool pluginInitialized = false;
	private ObjectMeshDynamic carSphere = null;
	private Unigine.Plugins.FMOD.Sound musicSound = null;
	private Unigine.Plugins.FMOD.Sound musicSound3D = null;
	private Channel musicChannel = null;
	private Channel musicChannel3D = null;
	private SampleDescriptionWindow sampleDescriptionWindow = null;
	private WidgetSlider musicPositionSlider;
	private WidgetSlider distortionSlider;
	private WidgetSlider volumeSlider;
	void Init()
	{
		sampleDescriptionWindow = new SampleDescriptionWindow();
		sampleDescriptionWindow.createWindow(Gui.ALIGN_LEFT, 500);
		// load the FMOD plugin
		if (Engine.FindPlugin("UnigineFMOD") == -1)
			Engine.AddPlugin("UnigineFMOD");
		if (!FMOD.CheckPlugin())
		{
			WidgetGroupBox parametersGroupbox = sampleDescriptionWindow.getParameterGroupBox();
			var infoLabel = new WidgetLabel();
			infoLabel.FontWrap = 1;
			infoLabel.Text = "Cannot find FMOD plugin. Please check UnigineFMOD and fmod.dll, fmodL.dll, fmodstudio.dll, fmodstudioL.dll (You can download these files from official site) in bin directory.";
			parametersGroupbox.AddChild(infoLabel);
			return;
		}
		pluginInitialized = true;
		// initialize FMOD Core with 1024 channels in NORMAL mode
		FMODCore core = FMOD.Core;
		core.InitCore(1024, FMODEnums.INIT_FLAGS.NORMAL);
		// load a 2D sound file
		musicSound = core.CreateSound(
			FileSystem.GetAbsolutePath(FileSystem.ResolvePartialVirtualPath("fmod_core/sounds/soundtrack.oga")),
			FMODEnums.FMOD_MODE._2D);
		// load the same file as a 3D sound
		musicSound3D = core.CreateSound(
			FileSystem.GetAbsolutePath(FileSystem.ResolvePartialVirtualPath("fmod_core/sounds/soundtrack.oga")),
			FMODEnums.FMOD_MODE._3D);
		// create a sphere to visualize the 3D sound source
		carSphere = Primitives.CreateSphere(1.0f);
		carSphere.SetMaterialParameterFloat4("albedo_color", new vec4(0.4f, 0.0f, 0.0f, 1.0f), 0);
		Visualizer.Enabled = true;
		InitDescriptionWindow();
	}
	void Update()
	{
		if (!pluginInitialized)
			return;
		// display the "Music 3D" label above the car sphere in the 3D world
		var len = musicSound.GetLength(FMODEnums.TIME_UNIT.MS);
		Visualizer.RenderMessage3D(carSphere.WorldPosition, vec3.ZERO, "Music 3D", vec4.WHITE, 0, 25);
		// update music progress slider
		uint pos;
		if (musicChannel)
		{
			musicChannel.GetPositionTimeLine(out pos, FMODEnums.TIME_UNIT.MS);
			int progress = Convert.ToInt32(pos / Convert.ToSingle(len) * 100);
			if (progress >= 100)
			{
				musicChannel.SetPositionTimeLine(0, FMODEnums.TIME_UNIT.MS);
				StopMusic();
				progress = 0;
			}
			musicPositionSlider.Value = progress;
		}
	}
	void Shutdown()
	{
		// release all FMOD sounds and channels
		if (musicSound)
		{
			musicSound.Release();
			musicSound = null;
		}
		if (musicSound3D)
		{
			musicSound3D.Release();
			musicSound3D = null;
		}
		if (musicChannel)
		{
			musicChannel.Release();
			musicChannel = null;
		}
		if (musicChannel3D)
		{
			musicChannel3D.Release();
			musicChannel3D = null;
		}
		if (pluginInitialized)
		{
			carSphere.DeleteLater();
		}
		Unigine.Console.Run("plugin_unload UnigineFMOD");
		pluginInitialized = false;
		Visualizer.Enabled = false;
		sampleDescriptionWindow.shutdown();
	}
	void InitDescriptionWindow()
	{
		// create GUI tabs and controls
		WidgetGroupBox parametersGroupbox = sampleDescriptionWindow.getParameterGroupBox();
		WidgetTabBox tab = new WidgetTabBox(4, 4);
		parametersGroupbox.AddChild(tab, Gui.ALIGN_EXPAND);
		// tab music
		{
			tab.AddTab("Music");
			// playback controls
			var playButton = new WidgetButton("Play");
			var stopButton = new WidgetButton("Stop");
			var pauseButton = new WidgetButton("Pause/Resume");
			var plusButton = new WidgetButton("+ 10 sec");
			var minusButton = new WidgetButton("- 10 sec");
			var hbox = new WidgetHBox();
			playButton.EventClicked.Connect(PlayMusic);
			stopButton.EventClicked.Connect(StopMusic);
			pauseButton.EventClicked.Connect(TogglePauseMusic);
			minusButton.EventClicked.Connect(MinusMS);
			plusButton.EventClicked.Connect(PlusMS);
			musicPositionSlider = new WidgetSlider();
			distortionSlider = new WidgetSlider();
			distortionSlider.EventChanged.Connect(DistortionChanged);
			volumeSlider = new WidgetSlider();
			volumeSlider.EventChanged.Connect(VolumeChanged);
			volumeSlider.Value = 100;
			hbox.AddChild(minusButton);
			hbox.AddChild(plusButton);
			tab.AddChild(new WidgetLabel("Time Line"), Gui.ALIGN_EXPAND);
			tab.AddChild(musicPositionSlider, Gui.ALIGN_EXPAND);
			tab.AddChild(hbox, Gui.ALIGN_EXPAND);
			tab.AddChild(playButton, Gui.ALIGN_EXPAND);
			tab.AddChild(stopButton, Gui.ALIGN_EXPAND);
			tab.AddChild(pauseButton, Gui.ALIGN_EXPAND);
			tab.AddChild(new WidgetLabel("Distortion Mix"), Gui.ALIGN_EXPAND);
			tab.AddChild(distortionSlider, Gui.ALIGN_EXPAND);
			tab.AddChild(new WidgetLabel("Volume"), Gui.ALIGN_EXPAND);
			tab.AddChild(volumeSlider, Gui.ALIGN_EXPAND);
		}
		// tab music 3D
		{
			tab.AddTab("Music 3D");
			var playButton = new WidgetButton("Play");
			var stopButton = new WidgetButton("Stop");
			var pauseButton = new WidgetButton("Pause/Resume");
			var hbox = new WidgetHBox();
			playButton.EventClicked.Connect(PlayMusic3D);
			stopButton.EventClicked.Connect(StopMusic3D);
			pauseButton.EventClicked.Connect(TogglePauseMusic3D);
			tab.AddChild(playButton, Gui.ALIGN_EXPAND);
			tab.AddChild(stopButton, Gui.ALIGN_EXPAND);
			tab.AddChild(pauseButton, Gui.ALIGN_EXPAND);
		}
		parametersGroupbox.Arrange();
	}
	void DistortionChanged()
	{
		if (!musicChannel)
		{
			return;
		}
		// adjust distortion effect on the music channel
		musicChannel.GetDSP(0).SetParameterFloat(0, distortionSlider.Value * 0.01f);
	}
	void VolumeChanged()
	{
		if (!musicChannel)
		{
			return;
		}
		// adjust volume on the music channel
		musicChannel.Volume = volumeSlider.Value * 0.01f;
	}
	void PlusMS()
	{
		if (!musicChannel)
		{
			return;
		}
		// jump forward by 10 seconds in the timeline
		uint currTimeLine;
		uint len = musicSound.GetLength(FMODEnums.TIME_UNIT.MS);
		musicChannel.GetPositionTimeLine(out currTimeLine, FMODEnums.TIME_UNIT.MS);
		if (currTimeLine + 10000 >= len)
		{
			musicChannel.SetPositionTimeLine(0, FMODEnums.TIME_UNIT.MS);
		}
		else
		{
			musicChannel.SetPositionTimeLine(currTimeLine + 10000, FMODEnums.TIME_UNIT.MS);
		}
	}
	void MinusMS()
	{
		if (!musicChannel)
		{
			return;
		}
		// jump backward by 10 seconds in the timeline
		uint currTimeLine;
		musicChannel.GetPositionTimeLine(out currTimeLine, FMODEnums.TIME_UNIT.MS);
		if (currTimeLine < 10000)
		{
			musicChannel.SetPositionTimeLine(0, FMODEnums.TIME_UNIT.MS);
		}
		else
		{
			musicChannel.SetPositionTimeLine(currTimeLine - 10000, FMODEnums.TIME_UNIT.MS);
		}
	}
	void PlayMusic()
	{
		// start 2D music playback if 3D music is not playing
		// adds a distortion DSP effect and sets the volume
		if (!musicChannel3D || !musicChannel3D.IsPlaying)
		{
			StopMusic();
			musicChannel = musicSound.Play();
			musicChannel.AddDSP(0, DSPType.TYPE.DISTORTION).SetParameterFloat(0, distortionSlider.Value * 0.01f);
			musicChannel.Volume = volumeSlider.Value * 0.01f;
		}
	}
	void StopMusic()
	{
		if (!musicChannel)
		{
			return;
		}
		musicChannel.Stop();
		musicChannel = null;
	}
	void TogglePauseMusic()
	{
		if (!musicChannel)
		{
			return;
		}
		musicChannel.Paused = !musicChannel.Paused;
	}
	void PlayMusic3D()
	{
		// start 3D music playback at the car_sphere position
		if (!musicChannel || !musicChannel.IsPlaying)
		{
			StopMusic3D();
			musicChannel3D = musicSound3D.Play();
			musicChannel3D.SetPosition(carSphere.WorldPosition);
		}
	}
	void StopMusic3D()
	{
		if (!musicChannel3D)
		{
			return;
		}
		musicChannel3D.Stop();
		musicChannel3D = null;
	}
	void TogglePauseMusic3D()
	{
		if (!musicChannel3D)
		{
			return;
		}
		musicChannel3D.Paused = !musicChannel3D.Paused;
	}
}
```
---
## FMODStudioSample.cs
```csharp
﻿using Unigine;
using Unigine.Plugins.FMOD;
using System;
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
#endif
[Component(PropertyGuid = "5cb85699ce485ecd4f4c9e8bead8eb7993f5d703")]
public class FMODStudioSample : Component
{
	private bool pluginInitialized = false;
	private float timer = 0.0f;
	private ObjectMeshDynamic carSphere;
	private ObjectMeshDynamic dopplerSphere;
	private EventInstance engineEvent = null;
	private EventInstance dopplerEngineEvent = null;
	private EventInstance forestEvent = null;
	private VCA envVCA = null;
	private Vec3 velocity;
	private Vec3 startPoint = new Vec3(-5, 80, 0);
	private SampleDescriptionWindow sampleDescriptionWindow = null;
	private WidgetSlider engineSlider;
	private WidgetSlider windForestSlider;
	private WidgetSlider rainForestSlider;
	private WidgetSlider coverForestSlider;
	private WidgetSlider envVCASlider;
	private WidgetSlider dopplerRPMSlider;
	private WidgetSlider dopplerVelocitySlider;
	private WidgetCheckBox showDopplerBoxCheckBox;
	void Init()
	{
		sampleDescriptionWindow = new SampleDescriptionWindow();
		sampleDescriptionWindow.createWindow(Gui.ALIGN_LEFT, 500);
		// load the FMOD plugin
		if (Engine.FindPlugin("UnigineFMOD") == -1)
			Engine.AddPlugin("UnigineFMOD");
		if (!FMOD.CheckPlugin())
		{
			WidgetGroupBox parametersGroupbox = sampleDescriptionWindow.getParameterGroupBox();
			var infoLabel = new WidgetLabel();
			infoLabel.FontWrap = 1;
			infoLabel.Text = "Cannot find FMOD plugin. Please check UnigineFMOD and fmod.dll, fmodL.dll, fmodstudio.dll, fmodstudioL.dll (You can download these files from official site) in bin directory.";
			parametersGroupbox.AddChild(infoLabel);
			return;
		}
		pluginInitialized = true;
		// create two spheres: one static (car) and one moving (doppler)
		carSphere = Primitives.CreateSphere(2.0f);
		carSphere.SetMaterialParameterFloat4("albedo_color", new vec4(0.4f, 0.0f, 0.0f, 1.0f), 0);
		dopplerSphere = Primitives.CreateSphere(1.0f);
		dopplerSphere.SetMaterialParameterFloat4("albedo_color", new vec4(0.0f, 4.0f, 0.0f, 1.0f), 0);
		dopplerSphere.WorldPosition = startPoint;
		// initialize FMOD Studio and load sound banks
		FMODStudio studio = FMOD.Studio;
		studio.UseStudioLiveUpdateFlag();
		studio.InitStudio();
		studio.LoadBank(FileSystem.GetAbsolutePath(FileSystem.ResolvePartialVirtualPath("fmod_studio/fmod_banks/Master.bank")));
		studio.LoadBank(FileSystem.GetAbsolutePath(FileSystem.ResolvePartialVirtualPath("fmod_studio/fmod_banks/Master.strings.bank")));
		studio.LoadBank(FileSystem.GetAbsolutePath(FileSystem.ResolvePartialVirtualPath("fmod_studio/fmod_banks/Vehicles.bank")));
		studio.LoadBank(FileSystem.GetAbsolutePath(FileSystem.ResolvePartialVirtualPath("fmod_studio/fmod_banks/SFX.bank")));
		// play ambient forest and engine events
		forestEvent = studio.GetEvent("event:/Ambience/Forest");
		forestEvent.Play();
		engineEvent = studio.GetEvent("event:/Vehicles/Car Engine");
		engineEvent.Play();
		// doppler effect setup
		dopplerEngineEvent = studio.GetEvent("event:/Vehicles/Car Engine");
		dopplerEngineEvent.SetParent(dopplerSphere);
		dopplerEngineEvent.SetParameter("RPM", 4000);
		// initial movement direction
		velocity = -Vec3.FORWARD;
		// load VCA for master environmental volume
		envVCA = studio.GetVCA("vca:/Environment");
		Visualizer.Enabled = true;
		InitDescriptionWindow();
	}
	void Update()
	{
		if (!pluginInitialized)
			return;
		float t = Game.Time;
		float dt = Game.IFps;
		// Doppler simulation logic
		if (showDopplerBoxCheckBox.Checked)
		{
			dopplerSphere.Enabled = true;
			// reset position after 2.5 seconds
			if (timer >= 2.5f)
			{
				dopplerSphere.WorldPosition = startPoint;
				timer = 0.0f;
			}
			timer += dt;
			// restart Doppler sound if not playing
			if (!dopplerEngineEvent.IsPlaying && !dopplerEngineEvent.IsStarting)
			{
				dopplerEngineEvent.Play();
			}
			// move the Doppler object and update sound velocity
			dopplerSphere.WorldPosition = dopplerSphere.WorldPosition + velocity;
			Visualizer.RenderMessage3D(dopplerSphere.WorldPosition, vec3.ZERO, "Doppler", vec4.WHITE, 0, 20);
		}
		else
		{
			dopplerEngineEvent.Stop();
			dopplerSphere.Enabled = false;
		}
		// show label above the car
		Visualizer.RenderMessage3D(carSphere.WorldPosition, vec3.ZERO, "Car", vec4.WHITE, 0, 20);
	}
	void Shutdown()
	{
		// release all FMOD events
		if (engineEvent)
		{
			engineEvent.Release();
			engineEvent = null;
		}
		if (dopplerEngineEvent)
		{
			dopplerEngineEvent.Release();
			dopplerEngineEvent = null;
		}
		if (forestEvent)
		{
			forestEvent.Release();
			forestEvent = null;
		}
		if (envVCA)
		{
			envVCA.Release();
			envVCA = null;
		}
		if (pluginInitialized)
		{
			carSphere.DeleteLater();
			dopplerSphere.DeleteLater();
		}
		Unigine.Console.Run("plugin_unload UnigineFMOD");
		pluginInitialized = false;
		Visualizer.Enabled = false;
		sampleDescriptionWindow.shutdown();
	}
	void InitDescriptionWindow()
	{
		// create GUI tabs and controls
		WidgetGroupBox parametersGroupbox = sampleDescriptionWindow.getParameterGroupBox();
		WidgetTabBox tab = new WidgetTabBox(4, 4);
		parametersGroupbox.AddChild(tab, Gui.ALIGN_EXPAND);
		// ambience tab
		{
			tab.AddTab("Ambience");
			windForestSlider = new WidgetSlider();
			var wind_label = new WidgetLabel("Wind");
			tab.AddChild(wind_label, Gui.ALIGN_EXPAND);
			tab.AddChild(windForestSlider, Gui.ALIGN_EXPAND);
			windForestSlider.EventChanged.Connect(WindForestSliderChanged);
			rainForestSlider = new WidgetSlider();
			var forest_label = new WidgetLabel("Rain");
			tab.AddChild(forest_label, Gui.ALIGN_EXPAND);
			tab.AddChild(rainForestSlider, Gui.ALIGN_EXPAND);
			rainForestSlider.EventChanged.Connect(RainForestSliderChanged);
			coverForestSlider = new WidgetSlider();
			var cover_label = new WidgetLabel("Cover");
			tab.AddChild(cover_label, Gui.ALIGN_EXPAND);
			tab.AddChild(coverForestSlider, Gui.ALIGN_EXPAND);
			coverForestSlider.EventChanged.Connect(CoverForestSliderChanged);
		}
		// engine tab
		{
			tab.AddTab("Engine");
			engineSlider = new WidgetSlider();
			engineSlider.MinValue = 0;
			engineSlider.MaxValue = 8000;
			var label = new WidgetLabel("RPM");
			tab.AddChild(label, Gui.ALIGN_EXPAND);
			tab.AddChild(engineSlider, Gui.ALIGN_EXPAND);
			engineSlider.EventChanged.Connect(EngineSliderChanged);
		}
		// doppler tab
		{
			tab.AddTab("Doppler");
			showDopplerBoxCheckBox = new WidgetCheckBox();
			showDopplerBoxCheckBox.Checked = false;
			var label = new WidgetLabel("Show Doppler Effect");
			tab.AddChild(label, Gui.ALIGN_EXPAND);
			tab.AddChild(showDopplerBoxCheckBox, Gui.ALIGN_EXPAND);
			dopplerRPMSlider = new WidgetSlider();
			dopplerRPMSlider.MaxValue = 8000;
			dopplerRPMSlider.Value = 4000;
			tab.AddChild(new WidgetLabel("RPM"), Gui.ALIGN_EXPAND);
			tab.AddChild(dopplerRPMSlider, Gui.ALIGN_EXPAND);
			dopplerVelocitySlider = new WidgetSlider();
			tab.AddChild(new WidgetLabel("Velocity"), Gui.ALIGN_EXPAND);
			tab.AddChild(dopplerVelocitySlider, Gui.ALIGN_EXPAND);
			dopplerRPMSlider.EventChanged.Connect(DopplerRPMSliderChanged);
			dopplerVelocitySlider.EventChanged.Connect(DopplerVelocitySliderChanged);
			dopplerVelocitySlider.Value = 5;
		}
		// vca tab
		{
			tab.AddTab("VCA");
			envVCASlider = new WidgetSlider();
			envVCASlider.Value = 100;
			var label = new WidgetLabel("Sounds Volume");
			tab.AddChild(label, Gui.ALIGN_EXPAND);
			tab.AddChild(envVCASlider, Gui.ALIGN_EXPAND);
			envVCASlider.EventChanged.Connect(EnvVCASliderChanged);
		}
		parametersGroupbox.Arrange();
	}
	void EnvVCASliderChanged()
	{
		// adjust environment volume
		envVCA.Volume = envVCASlider.Value * 0.01f;
	}
	void EngineSliderChanged()
	{
		// set RPM for engine event
		engineEvent.SetParameter("RPM", Convert.ToSingle(engineSlider.Value));
	}
	void WindForestSliderChanged()
	{
		// set wind intensity in forest ambience
		forestEvent.SetParameter("Wind", windForestSlider.Value * 0.01f);
	}
	void RainForestSliderChanged()
	{
		// set rain intensity in forest ambience
		forestEvent.SetParameter("Rain", rainForestSlider.Value * 0.01f);
	}
	void CoverForestSliderChanged()
	{
		// set cover parameter in forest ambience
		forestEvent.SetParameter("Cover", coverForestSlider.Value * 0.01f);
	}
	void DopplerRPMSliderChanged()
	{
		// set RPM for Doppler engine event
		dopplerEngineEvent.SetParameter("RPM", Convert.ToSingle(dopplerRPMSlider.Value));
	}
	void DopplerVelocitySliderChanged()
	{
		// adjust velocity of Doppler object
		velocity.y = -dopplerVelocitySlider.Value * 0.1f;
	}
}
```
---
## Fan.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "881e74a476453dee3919dabfdaf3f4bf1a0b16ed")]
public class Fan : Toggleable
{
	public float rotation_speed = 120;
	private float target_speed = 0;
	private float actual_speed = 0;
	protected override bool On()
	{
		Log.MessageLine("Fan::On()");
		target_speed = rotation_speed;
		return true;
	}
	protected override bool Off()
	{
		Log.MessageLine("Fan::Off()");
		target_speed = 0;
		return true;
	}
	private void Init()
	{
		target_speed = Toggled ? rotation_speed : 0;
	}
	private void Update()
	{
		actual_speed = MathLib.Lerp(actual_speed, target_speed, Game.IFps);
		node.Rotate(0, 0, actual_speed * Game.IFps);
	}
}
```
---
## FirstPersonController.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "7abd3d1dd6399bd8498bed57e699aacada031ba1")]
public class FirstPersonController : Component
{
	#region Editor parameters
	[ShowInEditor]
	[Parameter(Group = "Input", Tooltip = "Move forward key")]
	private Input.KEY forwardKey = Input.KEY.W;
	[ShowInEditor]
	[Parameter(Group = "Input", Tooltip = "Move backward key")]
	private Input.KEY backwardKey = Input.KEY.S;
	[ShowInEditor]
	[Parameter(Group = "Input", Tooltip = "Move right key")]
	private Input.KEY rightKey = Input.KEY.D;
	[ShowInEditor]
	[Parameter(Group = "Input", Tooltip = "Move left key")]
	private Input.KEY leftKey = Input.KEY.A;
	[ShowInEditor]
	[Parameter(Group = "Input", Tooltip = "Run mode activation key")]
	private Input.KEY runKey = Input.KEY.ANY_SHIFT;
	[ShowInEditor]
	[Parameter(Group = "Input", Tooltip = "Jump key")]
	private Input.KEY jumpKey = Input.KEY.SPACE;
	[ShowInEditor]
	[Parameter(Group = "Input", Tooltip = "Crouch mode activation key")]
	private Input.KEY crouchKey = Input.KEY.ANY_CTRL;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Input", Tooltip = "Mouse sensitivity multiplier")]
	private float mouseSensitivity = 1.0f;
	public enum GamepadStickSide
	{
		LEFT = 0,
		RIGHT
	}
	[ShowInEditor]
	[Parameter(Group = "Gamepad Input", Tooltip = "Horizontal movement stick")]
	private GamepadStickSide moveStick = GamepadStickSide.LEFT;
	[ShowInEditor]
	[Parameter(Group = "Gamepad Input", Tooltip = "Camera rotation stick")]
	private GamepadStickSide cameraStick = GamepadStickSide.RIGHT;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Gamepad Input", Tooltip = "Sensitivity of the camera stick")]
	private float cameraStickSensitivity = 0.8f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Gamepad Input", Tooltip = "Deadzone deviation, set for both sticks")]
	private float sticksDeadZone = 0.3f;
	[ShowInEditor]
	[Parameter(Group = "Gamepad Input", Tooltip = "Run mode activation button")]
	private Input.GAMEPAD_BUTTON runButton = Input.GAMEPAD_BUTTON.SHOULDER_RIGHT;
	[ShowInEditor]
	[Parameter(Group = "Gamepad Input", Tooltip = "Jump button")]
	private Input.GAMEPAD_BUTTON jumpButton = Input.GAMEPAD_BUTTON.A;
	[ShowInEditor]
	[Parameter(Group = "Gamepad Input", Tooltip = "Crouch mode activation button")]
	private Input.GAMEPAD_BUTTON crouchButton = Input.GAMEPAD_BUTTON.SHOULDER_LEFT;
	[ShowInEditor]
	[Parameter(Group = "Body", Tooltip = "Use the body and shape of the current node.\nIf this option is enabled, the node should have\na dummy body and a capsule shape assigned.\n" +
		"If the custom body or shape has invalid settings,\na default body and capsule shape are created\nand used instead.")]
	private bool useObjectBody = false;
	[ShowInEditor]
	[ParameterSlider(Group = "Body", Tooltip = "Radius of the capsule shape")]
	[ParameterCondition(nameof(useObjectBody), 0)]
	private float capsuleRadius = 0.3f;
	[ShowInEditor]
	[ParameterSlider(Group = "Body", Tooltip = "Height of the capsule shape (cylindrical part only)")]
	[ParameterCondition(nameof(useObjectBody), 0)]
	private float capsuleHeight = 1.15f;
	[ShowInEditor]
	[ParameterMask(Group = "Body", MaskType = ParameterMaskAttribute.TYPE.PHYSICS_INTERSECTION, Tooltip = "Mask used for selective detection of physics intersections\n(with other physical objects having bodies and collider shapes,\nor ray intersections with collider geometry)")]
	[ParameterCondition(nameof(useObjectBody), 0)]
	private int physicsIntersectionMask = 1;
	[ShowInEditor]
	[ParameterMask(Group = "Body", MaskType = ParameterMaskAttribute.TYPE.COLLISION, Tooltip = "Mask used for selective collision detection")]
	[ParameterCondition(nameof(useObjectBody), 0)]
	private int collisionMask = 1;
	[ShowInEditor]
	[ParameterMask(Group = "Body", MaskType = ParameterMaskAttribute.TYPE.COLLISION, Tooltip = "Mask used for exclusion of collision detection")]
	[ParameterCondition(nameof(useObjectBody), 0)]
	private int exclusionMask = 0;
	public enum CameraMode
	{
		NONE = 0,
		CREATE_AUTOMATICALLY,
		USE_EXTERNAL
	}
	[ShowInEditor]
	[Parameter(Group = "Camera", Tooltip = "The mode of assigning the camera to the player:\n\nNONE — camera is not created. The user implements its own camera,\nthis component is used for the movement logic only.\n\n" + 
		"CREATE_AUTOMATICALLY — camera is created automatically\nat initialization of the component and set as the main camera.\nThis camera uses the values set for the parameters in this section.\n\n" + 
		"USE_EXTERNAL — a Player Dummy assigned via the Editor.")]
	private CameraMode cameraMode = CameraMode.CREATE_AUTOMATICALLY;
	[ShowInEditor]
	[Parameter(Group = "Camera", Tooltip = "Drag the camera node from the World Nodes hierarchy here")]
	[ParameterCondition(nameof(cameraMode), (int)CameraMode.USE_EXTERNAL)]
	private PlayerDummy camera = null;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Max = 180.0f, Group = "Camera", Tooltip = "Field of View")]
	[ParameterCondition(nameof(cameraMode), (int)CameraMode.CREATE_AUTOMATICALLY)]
	private float fov = 60.0f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Camera", Tooltip = "Distance to the near clipping plane of the player's viewing frustum, in units.")]
	[ParameterCondition(nameof(cameraMode), (int)CameraMode.CREATE_AUTOMATICALLY)]
	private float nearClipping = 0.01f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Camera", Tooltip = "Distance to the far clipping plane of the player's viewing frustum, in units.")]
	[ParameterCondition(nameof(cameraMode), (int)CameraMode.CREATE_AUTOMATICALLY)]
	private float farClipping = 1000.0f;
	[ShowInEditor]
	[Parameter(Group = "Camera", Tooltip = "Distance from the camera to the player's position.")]
	[ParameterCondition(nameof(cameraMode), (int)CameraMode.CREATE_AUTOMATICALLY)]
	private vec3 cameraPositionOffset = new vec3(0.0f, 0.0f, 1.65f);
	[ShowInEditor]
	[ParameterSlider(Min = -89.9f, Max = 89.9f, Group = "Camera", Tooltip = "The minimum vertical angle of the camera, i.e. maximum possible angle to look down.")]
	[ParameterCondition(nameof(cameraMode), (int)CameraMode.CREATE_AUTOMATICALLY, (int)CameraMode.USE_EXTERNAL)]
	private float minVerticalAngle = -89.9f;
	[ShowInEditor]
	[ParameterSlider(Min = -89.9f, Max = 89.9f, Group = "Camera", Tooltip = "The maximum vertical angle of the camera, i.e. maximum possible angle to look up.")]
	[ParameterCondition(nameof(cameraMode), (int)CameraMode.CREATE_AUTOMATICALLY, (int)CameraMode.USE_EXTERNAL)]
	private float maxVerticalAngle = 89.9f;
	[ShowInEditor]
	[Parameter(Group = "Movement", Tooltip = "Enable jumping and all relevant parameters in the player logic")]
	private bool useJump = true;
	[ShowInEditor]
	[Parameter(Group = "Movement", Tooltip = "Enable crouching and all relevant parameters in the player logic")]
	private bool useCrouch = true;
	[ShowInEditor]
	[Parameter(Group = "Movement", Tooltip = "Enable running and all relevant parameters in the player logic")]
	private bool useRun = true;
	[ShowInEditor]
	[Parameter(Group = "Movement", Tooltip = "Set running as a default state of the player's movement.\nIf this state is enabled, the walk state is enabled by using the Run Key.")]
	[ParameterCondition(nameof(useRun), 1)]
	private bool useRunDefault = false;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Movement", Tooltip = "Crouching speed of the player")]
	[ParameterCondition(nameof(useCrouch), 1)]
	private float crouchSpeed = 1.0f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Movement", Tooltip = "Walking speed of the player")]
	private float walkSpeed = 3.0f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Movement", Tooltip = "Running speed of the player")]
	[ParameterCondition(nameof(useRun), 1)]
	private float runSpeed = 6.0f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Movement", Tooltip = "Horizontal acceleration applied when the player is on the ground")]
	private float groundAcceleration = 25.0f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Movement", Tooltip = "Horizontal acceleration applied when the player is in the air")]
	private float airAcceleration = 10.0f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Movement", Tooltip = "Damping of the horizontal speed when the player is on the ground")]
	private float groundDamping = 25.0f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Movement", Tooltip = "Damping of the horizontal speed when the player is in the air")]
	private float airDamping = 0.05f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Movement", Tooltip = "Jump power of the player in the Run, Walk and Idle modes")]
	[ParameterCondition(nameof(useJump), 1)]
	private float jumpPower = 6.0f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Movement", Tooltip = "Jump power of the player in the Crouch mode")]
	[ParameterCondition(nameof(useJump), 1)]
	[ParameterCondition(nameof(useCrouch), 1)]
	private float crouchJumpPower = 3.0f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Movement", Tooltip = "The height of the camera when the player is crouching.\nThe minimum value equals to capsule radius x 2,\nthe maximum height is equal to the capsule height.")]
	[ParameterCondition(nameof(useCrouch), 1)]
	private float crouchHeight = 1.0f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Movement", Tooltip = "Time required for transition between the crouching and standing states")]
	[ParameterCondition(nameof(useCrouch), 1)]
	private float crouchTransitionTime = 0.3f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Max = 90.0f, Group = "Movement", Tooltip = "Maximum surface angle up to which the player is considered to be standing on the ground")]
	private float maxGroundAngle = 60.0f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Max = 90.0f, Group = "Movement", Tooltip = "Maximum surface angle up to which the player is considered to be touching the ceiling")]
	private float maxCeilingAngle = 60.0f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Max = 0.1f, Group = "Movement", Tooltip = "Offset of the ray that checks the surface it would potentially move to")]
	private float checkMoveRayOffset = 0.01f;
	[ShowInEditor]
	[ParameterMask(Group = "Movement", MaskType = ParameterMaskAttribute.TYPE.INTERSECTION, Tooltip = "Mask used to sort out intersections with the surface")]
	private int checkMoveMask = 1;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Max = 90.0f, Group = "Movement", Tooltip = "Horizontal angle between the direction ray and contact point of the capsule with the wall,\nwithin which the player doesn't slide along the wall.")]
	private float wallStopSlidingAngle = 15.0f;
	[ShowInEditor]
	[Parameter(Group = "Auto Stepping", Tooltip = "Automatic climbing upon obstacles up to a certain height (stairs, for example)")]
	private bool useAutoStepping = true;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Auto Stepping", Tooltip = "Minimum height of an obstacle that can be climbed upon.\nAllows avoiding false positives on flat surfaces.")]
	[ParameterCondition(nameof(useAutoStepping), 1)]
	private float minStepHeight = 0.05f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Auto Stepping", Tooltip = "Maximum height of an obstacle that can be climbed upon")]
	[ParameterCondition(nameof(useAutoStepping), 1)]
	private float maxStepHeight = 0.3f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Auto Stepping", Tooltip = "Maximum stair angle. If the character steps on a surface with a slope,\nthe angle of which is greater than the Max Stair Angle value,\nthis last step is canceled.")]
	[ParameterCondition(nameof(useAutoStepping), 1)]
	private float maxStairAngle = 30.0f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Group = "Auto Stepping", Tooltip = "Offset of the ray that checks the level of the potential stair for Auto Stepping.\nThis offset is vertical and in the direction of the potential movement.")]
	[ParameterCondition(nameof(useAutoStepping), 1)]
	private float checkStairRayOffset = 0.1f;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Max = 90.0f, Group = "Auto Stepping", Tooltip = "Horizontal angle that defines the area for checking the contacts\nof the capsule with other objects in the movement direction.\n" +
		"Allows avoiding false Auto Stepping near walls.")]
	[ParameterCondition(nameof(useAutoStepping), 1)]
	private float stairsDetectionAngle = 45.0f;
	[ShowInEditor]
	[ParameterMask(Group = "Auto Stepping", MaskType = ParameterMaskAttribute.TYPE.INTERSECTION, Tooltip = "The mask for checking intersections with the stairs for Auto Stepping.\n" +
		"If you have stairs with a complex geometry and want to simplify it for the Auto Stepping process,\ndisable this mask for the complex geometry and enable for the simplified model.")]
	private int stairDetectionMask = 1;
	[ShowInEditor]
	[Parameter(Group = "Objects Interaction", Tooltip = "Toggles physical interaction with rigid bodies on anf off")]
	private bool useObjectsInteraction = true;
	[ShowInEditor]
	[ParameterSlider(Min = 0.0f, Max = 1.0f, Group = "Objects Interaction", Tooltip = "Multiplier for the impulse applied to the Rigid Body the player collides with")]
	[ParameterCondition(nameof(useObjectsInteraction), 1)]
	private float impulseMultiplier = 0.05f;
	[ShowInEditor]
	[ParameterSlider(Min = 15, Max = 240, Group = "Advanced Settings", Tooltip = "Minimum update framerate for the player.\n" +
	"If the current FPS is less than this value, the player is updated several times per frame.")]
	private int playerFps = 60;
	[ShowInEditor]
	[ParameterSlider(Min = 1, Max = 128, Group = "Advanced Settings", Tooltip = "Number of iterations for resolving collisions")]
	private int collisionIterations = 4;
	[ShowInEditor]
	[ParameterSlider(Min = 4, Max = 1000, Group = "Advanced Settings", Tooltip = "Maximum number of contacts processed for a collision")]
	private int contactsBufferSize = 16;
	[ShowInEditor]
	[ParameterSlider(Min = 4, Max = 1000, Group = "Advanced Settings", Tooltip = "Maximum number of contacts up to which the Collision Iterations value is applied at processing contacts.\n" +
		"If the number of contacts exceeds this value, then only one iteration is used to avoid performance drop.")]
	private int heavyContactsCount = 100;
#if DEBUG
	public struct DebugCamera
	{
		[Parameter(Tooltip = "Enable debug camera — a third-person camera that observes the player.\nControls: cursor arrows and +/- for zooming.")]
		public bool enabled;
		[Parameter(Tooltip = "Debug camera with fixed angles — is positioned behind the player and in the player's direction")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool useFixedAngles;
		[HideInEditor] public PlayerDummy camera;
		[HideInEditor] public float angularSpeed;
		[HideInEditor] public float zoomSpeed;
		[HideInEditor] public float maxDistance;
		[HideInEditor] public float horizontalAngle;
		[HideInEditor] public float verticalAngle;
		[HideInEditor] public float distance;
	}
	[ShowInEditor]
	[Parameter(Group = "Debug")]
	private DebugCamera debugCamera;
	public struct DebugVisualizer
	{
		[Parameter(Tooltip = "Enable visualizer")]
		public bool enabled;
		[Parameter(Tooltip = "Visualisation of polygons")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool triangles;
		[Parameter(Tooltip = "Visualisation of shapes")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool shapes;
		[Parameter(Tooltip = "Visualisation of the player's capsule")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool playerShape;
		[Parameter(Tooltip = "Arrow that shows the player's direction, i.e. its Y axis")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool playerDirection;
		[Parameter(Tooltip = "Visualisation of the player's camera frustum and direction")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool camera;
		[Parameter(Tooltip = "Visualisation of the basis of the surface the player is standing on")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool slopeBasis;
		[Parameter(Tooltip = "Visualisation of the current horizontal speed vector applied to the player")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool appliedHorizontalVelocity;
		[Parameter(Tooltip = "Visualisation of the current vertical speed vector applied to the player")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool appliedVerticalVelocity;
		[Parameter(Tooltip = "Visualisation of contacts during the upward phase of Auto Stepping")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool upPassContacts;
		[Parameter(Tooltip = "Visualisation of contacts during the horizontal movement to any side")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool sidePassContacts;
		[Parameter(Tooltip = "Visualisation of contacts during the downward phase of Auto Stepping")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool downPassContacts;
		[Parameter(Tooltip = "Visualisation of the ray that checks the surface to perform potential movement")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool checkMoveRay;
		[Parameter(Tooltip = "Visualisation of the ray that checks obstacles during Auto Stepping")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool stairDetectionRay;
	}
	[ShowInEditor]
	[Parameter(Group = "Debug")]
#pragma warning disable CS0649
	private DebugVisualizer debugVisualizer;
#pragma warning restore CS0649
	public struct DebugProfiler
	{
		[Parameter(Tooltip = "Enable Profiler")]
		public bool enabled;
		[Parameter(Tooltip = "Scalar value of the horizontal velocity applied to the player")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool appliedHorizontalSpeed;
		[Parameter(Tooltip = "Scalar value of the vertical velocity applied to the player")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool appliedVerticalSpeed;
		[Parameter(Tooltip = "Contacts during the upward phase of Auto Stepping")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool upPassContact;
		[Parameter(Tooltip = "Contacts during the horizontal movement to any side")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool sidePassContact;
		[Parameter(Tooltip = "Contacts during the downward phase of Auto Stepping")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool downPassContact;
		[Parameter(Tooltip = "Changes in the state of touching the ground")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool isGround;
		[Parameter(Tooltip = "Changes in the state of touching the ceiling")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool isCeiling;
		[Parameter(Tooltip = "Changes in the state of crouching")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool isCrouch;
		[Parameter(Tooltip = "Average player's speed, which is calculated based on the coordinates changes")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool averageSpeed;
		[Parameter(Tooltip = "Application of Auto Stepping")]
		[ParameterCondition(nameof(enabled), 1)]
		public bool autoStepping;
	}
	[ShowInEditor]
	[Parameter(Group = "Debug")]
#pragma warning disable CS0649
	private DebugProfiler debugProfiler;
#pragma warning restore CS0649
	public struct DebugColors
	{
		[ParameterColor] public vec4 playerShape;
		[ParameterColor] public vec4 playerDirection;
		[ParameterColor] public vec4 cameraColor;
		[ParameterColor] public vec4 appliedHorizontalVelocity;
		[ParameterColor] public vec4 appliedVerticalVelocity;
		[ParameterColor] public vec4 upPassContacts;
		[ParameterColor] public vec4 sidePassContacts;
		[ParameterColor] public vec4 downPassContacts;
		[ParameterColor] public vec4 isGround;
		[ParameterColor] public vec4 isCeiling;
		[ParameterColor] public vec4 isCrouch;
		[ParameterColor] public vec4 averageSpeed;
		[ParameterColor] public vec4 autoStepping;
		[HideInEditor] public float[] arrayAppliedHorizontalVelocity;
		[HideInEditor] public float[] arrayAppliedVerticalVelocity;
		[HideInEditor] public float[] arrayUpPassContacts;
		[HideInEditor] public float[] arraySidePassContacts;
		[HideInEditor] public float[] arrayDownPassContacts;
		[HideInEditor] public float[] arrayIsGround;
		[HideInEditor] public float[] arrayIsCeiling;
		[HideInEditor] public float[] arrayIsCrouch;
		[HideInEditor] public float[] arrayAverageSpeed;
		[HideInEditor] public float[] arrayAutoStepping;
		public DebugColors(bool useDefault)
		{
			playerShape = new vec4(0.0f, 0.0f, 1.0f, 0.098f);
			playerDirection = vec4.YELLOW;
			cameraColor = vec4.CYAN;
			appliedHorizontalVelocity = vec4.BLACK;
			appliedVerticalVelocity = vec4.GREY;
			upPassContacts = vec4.CYAN;
			sidePassContacts = vec4.MAGENTA;
			downPassContacts = vec4.RED;
			isGround = vec4.RED;
			isCeiling = vec4.GREEN;
			isCrouch = vec4.BLUE;
			averageSpeed = vec4.YELLOW;
			autoStepping = new vec4(0.66f, 0.33f, 0.0f, 1.0f);
			arrayAppliedHorizontalVelocity = null;
			arrayAppliedVerticalVelocity = null;
			arrayUpPassContacts = null;
			arraySidePassContacts = null;
			arrayDownPassContacts = null;
			arrayIsGround = null;
			arrayIsCeiling = null;
			arrayIsCrouch = null;
			arrayAverageSpeed = null;
			arrayAutoStepping = null;
		}
	}
	[ShowInEditor]
	[Parameter(Group = "Debug")]
	private DebugColors debugColors = new DebugColors(true);
#endif
	#endregion editor parameters
	#region Player properties
	public bool IsInitialized { get; private set; } = false;
	public vec3 AdditionalCameraOffset { get; set; } = vec3.ZERO;
	public quat AdditionalCameraRotation { get; set; } = quat.IDENTITY;
	public bool IsGround { get; private set; } = false;
	public bool IsCeiling { get; private set; } = false;
	public bool IsCrouch { get; private set; } = false;
	public bool IsHorizontalFrozen { get; private set; } = false;
	public Vec3 SlopeNormal { get; private set; } = Vec3.UP;
	public Vec3 SlopeAxisX { get; private set; } = Vec3.RIGHT;
	public Vec3 SlopeAxisY { get; private set; } = Vec3.FORWARD;
	public Vec3 HorizontalVelocity { get; private set; } = Vec3.ZERO; // velocity in current slope basis
	public float VerticalVelocity { get; private set; } = 0.0f;
	#endregion Player properties
	// additional constants
	static private readonly vec2 forward = new vec2(0, 1);
	static private readonly vec2 right = new vec2(1, 0);
	private const float skinWidthOffset = 0.05f;
	private const float autoSteppingSpeedThreshold = 0.1f;
	private const float largeEpsilon = 0.001f;
	private float playerIFps = 1.0f;
	private InputGamePad gamePad = null;
	private BodyDummy body = null;
	private ShapeCapsule shape = null;
	private ShapeCylinder interactionShape = null;
	private List<ShapeContact> contacts = new List<ShapeContact>();
	private bool isAvailableSideMove = false;
	private bool isAvailableStair = false;
	private bool hasBottomContacts = false;
	private bool isHeavyContacts = false;
	private float cameraVerticalAngle = 0.0f;
	private float cameraHorizontalAngle = 0.0f;
	private vec3 cameraCrouchOffset = vec3.ZERO;
	private vec2 horizontalMoveDirection = vec2.ZERO;
	private float verticalMoveDirection = 0.0f;
	private float maxAirSpeed = 0.0f;
	private bool usedAutoStepping = false;
	private Scalar lastStepHeight = 0.0f;
	private float cosGroundAngle = 0.5f;
	private float cosCeilingAngle = 0.5f;
	private float cosStairAngle = 0.5f;
	private float cosStairsDetectionAngle = 0.5f;
	private float cosWallStopSlidingAngle = 0.96f;
	private Mat4 worldTransform = Mat4.IDENTITY;
	WorldIntersectionNormal normalIntersection = new WorldIntersectionNormal();
	public enum CrouchPhase
	{
		STAND,
		MOVE_DOWN,
		CROUCH,
		MOVE_UP
	}
	private struct CrouchState
	{
		public CrouchPhase phase;
		public float currentTime;
		public Scalar currentHeight;
		public Scalar startHeight;
		public Scalar endHeight;
	}
	private CrouchState crouchState;
	private Input.MOUSE_HANDLE mouse_handle;
	#region Debug
#if DEBUG
	private float maxAppliedHorizontalSpeed = 0.0f;
	private float maxAppliedVerticalSpeed = 20.0f;
	private int maxPassContacts = 200;
	private float maxFlagValue = 1.05f;
	private float[] speedsBuffer = new float[10];
	private Vec3 lastPlayerPosition = Vec3.ZERO;
	private bool autoSteppingApplied = false;
#endif
	#endregion Debug
	private void Init()
	{
		mouse_handle = Input.MouseHandle;
		Input.MouseHandle = Input.MOUSE_HANDLE.GRAB;
		// check object type
		Object obj = node as Object;
		if (!obj)
		{
			Log.Error("FirstPersonController: can't cast node to Object\n");
			return;
		}
		// fix player transformation
		// player can only have vertical position, Y axis is used for forward direction
		Vec3 axisY = new Vec3(obj.WorldTransform.AxisY);
		axisY.z = 0;
		axisY = (axisY.Length2 > MathLib.EPSILON ? axisY.Normalized : Vec3.FORWARD);
		obj.WorldTransform = MathLib.SetTo(node.WorldPosition, node.WorldPosition + axisY, vec3.UP, MathLib.AXIS.Y);
		// set dummy body
		if (useObjectBody)
		{
			body = obj.Body as BodyDummy;
			if (!body)
				Log.Warning("FirstPersonController: object doesn't contain BodyDummy, it was created automatically\n");
		}
		if (!body)
		{
			body = new BodyDummy(obj);
			body.Transform = obj.WorldTransform;
		}
		// set capsule shape
		if (useObjectBody)
		{
			if (body.NumShapes > 0)
			{
				for (int i = 0; i < body.NumShapes; i++)
					if (!shape)
						shape = body.GetShape(i) as ShapeCapsule;
				if (!shape)
					Log.Warning("FirstPersonController: body doesn't contain ShapeCapsule, it was created automatically\n");
			}
			else
				Log.Warning("FirstPersonController: body doesn't contain shapes, it was created automatically\n");
		}
		if (!shape)
		{
			shape = new ShapeCapsule(body, capsuleRadius, capsuleHeight);
			body.SetShapeTransform(body.NumShapes - 1, MathLib.Translate(vec3.UP * (capsuleRadius + 0.5f * capsuleHeight)));
			shape.PhysicsIntersectionMask = physicsIntersectionMask;
			shape.CollisionMask = collisionMask;
			shape.ExclusionMask = exclusionMask;
		}
		capsuleHeight = shape.Height;
		crouchHeight = MathLib.Clamp(crouchHeight, 2.0f * shape.Radius, shape.Height + 2.0f * shape.Radius);
		crouchState.currentHeight = shape.Height + 2.0f * shape.Radius;
		// set camera
		if (cameraMode == CameraMode.USE_EXTERNAL && !camera)
			Log.Warning("FirstPersonController: camera is null, it was created automatically\n");
		if (!camera || cameraMode == CameraMode.CREATE_AUTOMATICALLY)
		{
			camera = new PlayerDummy();
			camera.Parent = obj;
			camera.Fov = fov;
			camera.ZNear = nearClipping;
			camera.ZFar = farClipping;
			camera.WorldPosition = obj.WorldTransform * new Vec3(cameraPositionOffset);
			camera.SetWorldDirection(new vec3(axisY), vec3.UP);
			camera.MainPlayer = true;
		}
		if (camera && cameraMode != CameraMode.NONE)
		{
			cameraVerticalAngle = MathLib.Angle(vec3.DOWN, camera.GetWorldDirection());
			cameraVerticalAngle = MathLib.Clamp(cameraVerticalAngle, minVerticalAngle + 90.0f, maxVerticalAngle + 90.0f);
			cameraHorizontalAngle = node.GetWorldRotation().GetAngle(vec3.UP);
			cameraPositionOffset = new vec3(node.IWorldTransform * camera.WorldPosition);
			vec3 cameraDirection = vec3.FORWARD * MathLib.RotateZ(-cameraHorizontalAngle);
			cameraDirection = cameraDirection * MathLib.Rotate(MathLib.Cross(cameraDirection, vec3.UP), 90.0f - cameraVerticalAngle);
			cameraDirection.Normalize();
			camera.SetWorldDirection(cameraDirection, vec3.UP);
		}
		// create cylinder shape for interacting with objects
		if (useObjectsInteraction)
		{
			interactionShape = new ShapeCylinder();
			interactionShape.Enabled = false;
		}
		// set auxiliary parameters
		playerIFps = 1.0f / playerFps;
		crouchTransitionTime = MathLib.Max(crouchTransitionTime, MathLib.EPSILON);
		cosGroundAngle = MathLib.Cos(maxGroundAngle * MathLib.DEG2RAD);
		cosCeilingAngle = MathLib.Cos(maxCeilingAngle * MathLib.DEG2RAD);
		cosStairAngle = MathLib.Cos(maxStairAngle * MathLib.DEG2RAD);
		cosStairsDetectionAngle = MathLib.Cos(stairsDetectionAngle * MathLib.DEG2RAD + MathLib.PI05);
		cosWallStopSlidingAngle = MathLib.Cos(wallStopSlidingAngle * MathLib.DEG2RAD);
#if DEBUG
		maxPassContacts = contactsBufferSize;
#endif
		worldTransform = obj.WorldTransform;
		for (int i = 0; i < Input.NumGamePads; i++)
		{
			Unigine.InputGamePad game_pad = Input.GetGamePad(i);
			if (game_pad.IsAvailable)
			{
				gamePad = game_pad;
				break;
			}
		}
		IsInitialized = true;
	}
	private void Shutdown()
	{
		Input.MouseHandle = mouse_handle;
	}
	private void Update()
	{
		if (!IsInitialized)
			return;
		worldTransform = node.WorldTransform;
		float ifps = Game.IFps * Physics.Scale;
		UpdateMoveDirections(ifps);
		CheckMoveAndStair();
		if (!isAvailableSideMove)
		{
			if (hasBottomContacts)
			{
				HorizontalVelocity = Vec3.ZERO;
				horizontalMoveDirection = vec2.ZERO;
			}
		}
		float updateTime = ifps;
		hasBottomContacts = false;
		while (updateTime > 0.0f)
		{
			float adaptiveTimeStep = MathLib.Min(updateTime, playerIFps);
			updateTime -= adaptiveTimeStep;
			UpdateVelocity(adaptiveTimeStep, adaptiveTimeStep / ifps);
			worldTransform = MathLib.Translate((HorizontalVelocity + Vec3.UP * VerticalVelocity) * adaptiveTimeStep) * worldTransform;
			UpdateCollisions(adaptiveTimeStep);
		}
		UpdateCrouch(ifps);
		// update player transformation
		node.WorldTransform = worldTransform;
		body.Transform = node.WorldTransform;
		if (IsCrouch)
			shape.Transform = worldTransform * MathLib.Translate(Vec3.UP * (shape.Radius + shape.Height * 0.5f));
		UpdateCamera();
	}
	private void UpdatePhysics()
	{
		if (!IsInitialized)
			return;
		if (useObjectsInteraction)
		{
			// enable interaction cylinder shape and set parameters
			// this shape is slightly larger than the capsule shape
			// this allows to get more correct contacts with objects, especially with bottom of player
			interactionShape.Enabled = true;
			interactionShape.Radius = shape.Radius + checkMoveRayOffset + skinWidthOffset;
			interactionShape.Height = shape.Height + 2.0f * (shape.Radius + checkMoveRayOffset - skinWidthOffset);
			interactionShape.Transform = MathLib.Translate(worldTransform.Translate + Vec3.UP * (0.5f * interactionShape.Height + skinWidthOffset));
			interactionShape.GetCollision(contacts);
			int contactsCount = MathLib.Min(contactsBufferSize, contacts.Count);
			Scalar speed = HorizontalVelocity.Length + MathLib.Abs(VerticalVelocity);
			speed = MathLib.Max(speed, 1.0f);
			for (int i = 0; i < contactsCount; i++)
			{
				ShapeContact c = contacts[i];
				if (c.Object && c.Object.BodyRigid)
				{
					c.Object.BodyRigid.Frozen = false;
					// multiply by 0.5f only for normalization impulse multiplier in editor settings
					c.Object.BodyRigid.AddWorldImpulse(c.Point, -c.Normal * c.Object.BodyRigid.Mass * (float)speed * impulseMultiplier * 0.5f);
				}
			}
			interactionShape.Enabled = false;
		}
	}
	private void UpdateMoveDirections(float ifps)
	{
		// reset all directions
		horizontalMoveDirection = vec2.ZERO;
		verticalMoveDirection = 0.0f;
		if (!Input.MouseGrab)
			return;
		// update horizontal direction
		if (Input.IsKeyPressed(forwardKey))
			horizontalMoveDirection += forward;
		if (Input.IsKeyPressed(backwardKey))
			horizontalMoveDirection -= forward;
		if (Input.IsKeyPressed(rightKey))
			horizontalMoveDirection += right;
		if (Input.IsKeyPressed(leftKey))
			horizontalMoveDirection -= right;
		if (horizontalMoveDirection.Length2 > 0)
			horizontalMoveDirection.Normalize();
		if (gamePad != null)
		{
			vec2 moveValue = (moveStick == GamepadStickSide.LEFT ? gamePad.AxesLeft : gamePad.AxesRight);
			if (moveValue.Length > sticksDeadZone && moveValue.Length2 > horizontalMoveDirection.Length2)
				horizontalMoveDirection = moveValue;
		}
		// update vertical direction
		if (useJump && IsGround && (Input.IsKeyDown(jumpKey) || gamePad && gamePad.IsButtonDown(jumpButton)))
			verticalMoveDirection = (IsCrouch ? crouchJumpPower : jumpPower) / ifps;
	}
	private void CheckMoveAndStair()
	{
		isAvailableSideMove = false;
		isAvailableStair = false;
		vec3 horizontalDirection = worldTransform.GetRotate() * new vec3(horizontalMoveDirection);
		if (horizontalMoveDirection.Length2 > 0)
			horizontalMoveDirection.Normalize();
		// check angle of surface for possible movement
		if (horizontalMoveDirection.Length2 > 0.0f)
		{
			Vec3 p2 = worldTransform.Translate + horizontalDirection * (shape.Radius + checkMoveRayOffset) + Vec3.DOWN * checkMoveRayOffset;
			Vec3 p1 = p2 + Vec3.UP * (MathLib.Max(shape.Radius, maxStepHeight) + checkMoveRayOffset);
			var hitObj = World.GetIntersection(p1, p2, checkMoveMask, normalIntersection);
			if (hitObj)
			{
				if (MathLib.Dot(vec3.UP, normalIntersection.Normal) > cosGroundAngle)
					isAvailableSideMove = true;
				// this check allows movement through elevations
				Scalar cos = MathLib.Dot(SlopeNormal, new Vec3(normalIntersection.Normal));
				if (cos < largeEpsilon)
					isAvailableSideMove = true;
			}
			else
			{
				// allow to move in air
				isAvailableSideMove = true;
			}
#if DEBUG
			if (debugVisualizer.enabled && debugVisualizer.checkMoveRay)
			{
				if (isAvailableSideMove)
					Visualizer.RenderVector(p1, p2, vec4.GREEN);
				else
					Visualizer.RenderVector(p1, p2, vec4.RED);
			}
#endif
		}
		// check stair surface angle for auto stepping
		if (useAutoStepping && horizontalMoveDirection.Length2 > 0.0f)
		{
			Vec3 p2 = worldTransform.Translate + horizontalDirection * (shape.Radius + checkStairRayOffset) + Vec3.UP * minStepHeight;
			Vec3 p1 = p2 + vec3.UP * (maxStepHeight - minStepHeight + checkStairRayOffset);
			var hitObj = World.GetIntersection(p1, p2, stairDetectionMask, normalIntersection);
			if (hitObj)
			{
				if (MathLib.Dot(vec3.UP, normalIntersection.Normal) > cosStairAngle)
					isAvailableStair = true;
			}
#if DEBUG
			if (debugVisualizer.enabled && debugVisualizer.stairDetectionRay)
			{
				if (isAvailableStair)
					Visualizer.RenderVector(p1, p2, vec4.GREEN);
				else
					Visualizer.RenderVector(p1, p2, vec4.RED);
			}
#endif
		}
	}
	private void UpdateVelocity(float ifps, float updatePart)
	{
		// update current slope basis
		// check vectors for collinearity and, depending on this, calculate the slope basis
		Scalar cosAngle = MathLib.Dot(new Vec3(worldTransform.AxisY), SlopeNormal);
		if (MathLib.Equals(MathLib.Abs(cosAngle), 1.0f))
		{
			SlopeAxisY = MathLib.Cross(new Vec3(worldTransform.AxisX) * MathLib.Sign(cosAngle), SlopeNormal).Normalized;
			SlopeAxisX = MathLib.Cross(SlopeAxisY, SlopeNormal).Normalized;
		}
		else
		{
			SlopeAxisX = MathLib.Cross(new Vec3(worldTransform.AxisY), SlopeNormal).Normalized;
			SlopeAxisY = MathLib.Cross(SlopeNormal, SlopeAxisX).Normalized;
		}
		// get decomposition of velocity for instant change on ground
		Vec3 horizontalVelocityDecomposition = Vec3.ZERO;
		if (IsGround)
		{
			horizontalVelocityDecomposition.x = MathLib.Dot(SlopeAxisX, HorizontalVelocity);
			horizontalVelocityDecomposition.y = MathLib.Dot(SlopeAxisY, HorizontalVelocity);
			horizontalVelocityDecomposition.z = MathLib.Dot(SlopeNormal, HorizontalVelocity);
		}
		// player rotation
		if (Input.MouseGrab)
		{
			worldTransform *= new Mat4(MathLib.Rotate(new quat(vec3.UP, -Input.MouseDeltaPosition.x * mouseSensitivity * 0.1f * updatePart)));
			float delta = -Input.MouseDeltaPosition.x * mouseSensitivity * 0.1f;
			if (gamePad)
			{
				vec2 rotateValue = (cameraStick == GamepadStickSide.LEFT ? gamePad.AxesLeft : gamePad.AxesRight);
				if (rotateValue.Length > sticksDeadZone && MathLib.Abs(rotateValue.x * cameraStickSensitivity) > MathLib.Abs(delta))
					delta = -rotateValue.x * cameraStickSensitivity;
			}
			cameraHorizontalAngle += delta * updatePart;
			if (cameraHorizontalAngle < -180.0f || 180.0f < cameraHorizontalAngle)
				cameraHorizontalAngle -= MathLib.Sign(cameraHorizontalAngle) * 360.0f;
			Vec3 position = worldTransform.Translate;
			worldTransform.SetRotate(Vec3.UP, cameraHorizontalAngle);
			worldTransform.SetColumn3(3, position);
		}
		// on the ground change velocity without inertia
		if (IsGround)
		{
			// again check vectors for collinearity and, depending on this, update the slope basis
			cosAngle = MathLib.Dot(new Vec3(worldTransform.AxisY), SlopeNormal);
			if (MathLib.Equals(MathLib.Abs(cosAngle), 1.0f))
			{
				SlopeAxisY = MathLib.Cross(new Vec3(worldTransform.AxisX) * MathLib.Sign(cosAngle), SlopeNormal).Normalized;
				SlopeAxisX = MathLib.Cross(SlopeAxisY, SlopeNormal).Normalized;
			}
			else
			{
				SlopeAxisX = MathLib.Cross(new Vec3(worldTransform.AxisY), SlopeNormal).Normalized;
				SlopeAxisY = MathLib.Cross(SlopeNormal, SlopeAxisX).Normalized;
			}
			// restore velocity in new basis
			HorizontalVelocity = SlopeAxisX * horizontalVelocityDecomposition.x +
								 SlopeAxisY * horizontalVelocityDecomposition.y +
								 SlopeNormal * horizontalVelocityDecomposition.z;
		}
		// add horizontal velocity in slope basis
		float acceleration = (IsGround ? groundAcceleration : airAcceleration);
		HorizontalVelocity += SlopeAxisX * horizontalMoveDirection.x * acceleration * ifps;
		HorizontalVelocity += SlopeAxisY * horizontalMoveDirection.y * acceleration * ifps;
		// update vertical velocity
		VerticalVelocity += verticalMoveDirection * ifps;
		if (!IsGround)
			VerticalVelocity += Physics.Gravity.z * ifps;
		// get current max speed
		float maxSpeed = maxAirSpeed;
		if (IsGround)
		{
			maxSpeed = (useRun && useRunDefault) ? runSpeed : walkSpeed;
			if (useRun && (Input.IsKeyPressed(runKey) || gamePad && gamePad.IsButtonPressed(runButton)))
				maxSpeed = useRunDefault ? walkSpeed : runSpeed;
			if (IsGround && IsCrouch)
				maxSpeed = crouchSpeed;
			maxAirSpeed = maxSpeed;
		}
		// apply damping to horizontal velocity when it exceeds target speed
		// or target speed too low (not pressed horizontal movement keys)
		vec2 targetSpeed = horizontalMoveDirection * maxSpeed;
		if (targetSpeed.Length < MathLib.EPSILON || targetSpeed.Length < HorizontalVelocity.Length)
			HorizontalVelocity *= MathLib.Exp((IsGround ? -groundDamping : -airDamping) * ifps);
		// clamp horizontal velocity if it greater than current max speed
		if (HorizontalVelocity.Length > maxSpeed)
			HorizontalVelocity = HorizontalVelocity.Normalized * maxSpeed;
		// check frozen state for horizontal velocity
		// IsGround needed in case of slipping from the edges
		// contacts will be pushed player in all directions, and not just up
		IsHorizontalFrozen = IsGround && (HorizontalVelocity.Length < Physics.FrozenLinearVelocity);
	}
	private void UpdateCollisions(float ifps)
	{
		// set default collision parameters
		IsGround = false;
		IsCeiling = false;
		SlopeNormal = Vec3.UP;
		isHeavyContacts = false;
		// resolve current collisions
		for (int j = 0; j < collisionIterations; j++)
		{
			if (useAutoStepping)
			{
#if DEBUG
				autoSteppingApplied = false;
#endif
				if (isAvailableStair)
					TryMoveUp(ifps);
			}
			MoveSide(ifps);
			if (useAutoStepping && usedAutoStepping && isAvailableStair)
			{
#if DEBUG
				autoSteppingApplied = true;
#endif
				TryMoveDown(ifps);
			}
			if (isHeavyContacts)
				break;
		}
	}
	private void TryMoveUp(float ifps)
	{
		usedAutoStepping = false;
		lastStepHeight = 0.0f;
		if (horizontalMoveDirection.Length2 > 0.0f && !IsHorizontalFrozen && VerticalVelocity < 0.0f)
		{
			body.Transform = worldTransform;
			if (IsCrouch)
				shape.Transform = worldTransform * MathLib.Translate(Vec3.UP * (shape.Radius + shape.Height * 0.5f));
			// find collisions with the capsule
			shape.GetCollision(contacts);
			if (contacts.Count == 0)
				return;
			if (contacts.Count > heavyContactsCount)
				isHeavyContacts = true;
			int contactsCount = MathLib.Min(contactsBufferSize, contacts.Count);
			// find max step height
			Vec2 velocityXY = new Vec2(HorizontalVelocity);
			if (velocityXY.Length2 < autoSteppingSpeedThreshold)
			{
				// set minimal velocity for climb
				velocityXY = new Vec2(worldTransform.GetRotate() * new Vec3(horizontalMoveDirection));
				velocityXY.Normalize();
				HorizontalVelocity = new Vec3(velocityXY * walkSpeed);
			}
			for (int i = 0; i < contactsCount; i++)
			{
				ShapeContact c = contacts[i];
				Vec2 normalXY = new Vec2(c.Normal);
				// skip contacts opposite to movement
				if (MathLib.Dot(normalXY, velocityXY) > cosStairsDetectionAngle)
					continue;
				Scalar step = MathLib.Dot(c.Point - worldTransform.Translate, Vec3.UP);
				if (lastStepHeight < step)
					lastStepHeight = step;
			}
			// apply auto stepping
			if (minStepHeight < lastStepHeight && lastStepHeight < maxStepHeight)
			{
				worldTransform.SetColumn3(3, worldTransform.Translate + vec3.UP * lastStepHeight);
				// check contacts with other objects after elevating
				// and cancel automatic step if contacts exist
				body.Transform = worldTransform;
				if (IsCrouch)
					shape.Transform = worldTransform * MathLib.Translate(Vec3.UP * (shape.Radius + shape.Height * 0.5f));
				shape.GetCollision(contacts);
				if (contacts.Count == 0)
					usedAutoStepping = true;
				else
					worldTransform.SetColumn3(3, worldTransform.Translate + vec3.DOWN * lastStepHeight);
			}
#if DEBUG
			if (debugVisualizer.enabled && debugVisualizer.upPassContacts)
			{
				foreach (var c in contacts)
					Visualizer.RenderVector(c.Point, c.Point + c.Normal, debugColors.upPassContacts);
			}
			if (debugProfiler.enabled && debugProfiler.upPassContact)
				Profiler.SetValue("Up Pass Contacts ", "", contacts.Count, maxPassContacts, debugColors.arrayUpPassContacts);
#endif
		}
	}
	private void MoveSide(float ifps)
	{
		// apply new player transformation for physic body
		body.Transform = worldTransform;
		if (IsCrouch)
			shape.Transform = worldTransform * MathLib.Translate(Vec3.UP * (shape.Radius + shape.Height * 0.5f));
		// get contacts in new position and process them
		shape.GetCollision(contacts);
		if (contacts.Count == 0)
			return;
		if (contacts.Count > heavyContactsCount)
			isHeavyContacts = true;
		int contactsCount = MathLib.Min(contactsBufferSize, contacts.Count);
		// total position offset for all contacts depth
		var positionOffset = vec3.ZERO;
		// maximum angle of inclination of the surface under the player
		float maxCosAngle = 1.0f;
		float inum = 1.0f / contactsCount;
		for (int i = 0; i < contactsCount; i++)
		{
			var c = contacts[i];
			// when horizontal velocity is frozen, we can move player only in vertical direction
			// this help to avoid sliding on slopes
			// in other cases, move player in all directions
			// use epsilon offset with depth for accuracy ground detection
			if (IsHorizontalFrozen)
			{
				float depth = MathLib.Dot(vec3.UP, c.Normal) * (c.Depth - MathLib.EPSILON);
				positionOffset += vec3.UP * depth * inum;
			}
			else
			{
				positionOffset += c.Normal * (c.Depth - MathLib.EPSILON) * inum;
				// remove part of horizontal velocity that is projected onto normal of current contact
				Scalar normalSpeed = MathLib.Dot(new Vec3(c.Normal), HorizontalVelocity);
				HorizontalVelocity -= c.Normal * normalSpeed;
			}
			// stop sliding near the wall at a certain angle
			if ((c.Object && c.Object.BodyRigid == null) && shape.BottomCap.z < c.Point.z && c.Point.z < shape.TopCap.z)
			{
				float cos = MathLib.Dot(worldTransform.GetRotate() * new vec3(horizontalMoveDirection), -c.Normal);
				if (cos > cosWallStopSlidingAngle)
					HorizontalVelocity = Vec3.ZERO;
			}
			// check ground state
			// first part of expression checks that current contact belongs to bottom sphere of capsule
			// second part of expression checks that current angle of inclination of surface
			// not exceed maximum allowed angle
			if (MathLib.Dot(c.Point - shape.BottomCap, Vec3.UP) < 0.0f)
			{
				hasBottomContacts = true;
				if (MathLib.Dot(c.Normal, vec3.UP) > cosGroundAngle)
				{
					VerticalVelocity = Physics.Gravity.z * ifps;
					IsGround = true;
				}
				// find to maximum angle of inclination of surface under player
				// and save normal of this surface
				float cosAngle = MathLib.Dot(vec3.UP, c.Normal);
				if (MathLib.Equals(cosAngle, 0.0f, 0.01f) == false && cosAngle < maxCosAngle)
				{
					SlopeNormal = new Vec3(contacts[i].Normal);
					maxCosAngle = cosAngle;
				}
			}
			// check ceiling state
			// first part of expression checks that current angle of inclination of ceiling
			// not exceed maximum allowed angle
			// second part of expression checks that current contact belongs to top sphere of capsule
			if (MathLib.Dot(contacts[i].Normal, vec3.DOWN) > cosCeilingAngle && MathLib.Dot(contacts[i].Point - shape.TopCap, Vec3.DOWN) < 0.0f)
			{
				IsCeiling = true;
				// stop moving up
				VerticalVelocity = 0.0f;
			}
		}
		// add total position offset to player transformation
		worldTransform.SetColumn3(3, worldTransform.Translate + positionOffset);
#if DEBUG
		if (debugVisualizer.enabled && debugVisualizer.sidePassContacts)
		{
			foreach (var c in contacts)
				Visualizer.RenderVector(c.Point, c.Point + c.Normal, debugColors.sidePassContacts);
		}
		if (debugProfiler.enabled && debugProfiler.sidePassContact)
			Profiler.SetValue("Side Pass Contacts ", "", contacts.Count, maxPassContacts, debugColors.arraySidePassContacts);
#endif
	}
	private void TryMoveDown(float ifps)
	{
		// this correction allows to avoid jittering on large stairs
		if (lastStepHeight > shape.Radius)
			lastStepHeight = shape.Radius - Physics.PenetrationTolerance;
		// try to drop down the player
		worldTransform.SetColumn3(3, worldTransform.Translate - vec3.UP * lastStepHeight);
		body.Transform = worldTransform;
		if (IsCrouch)
			shape.Transform = worldTransform * MathLib.Translate(Vec3.UP * (shape.Radius + shape.Height * 0.5f));
		// find collisions with the capsule
		shape.GetCollision(contacts);
		if (contacts.Count == 0)
			return;
		if (contacts.Count > heavyContactsCount)
			isHeavyContacts = true;
		int contactsCount = MathLib.Min(contactsBufferSize, contacts.Count);
		float inumContacts = 1.0f / MathLib.ToFloat(contactsCount);
		for (int i = 0; i < contactsCount; i++)
		{
			ShapeContact c = contacts[i];
			float depth = MathLib.Dot(vec3.UP, c.Normal) * c.Depth;
			worldTransform.SetColumn3(3, worldTransform.Translate + vec3.UP * depth * inumContacts);
			if (MathLib.Dot(c.Normal, vec3.UP) > cosGroundAngle && MathLib.Dot(c.Point - shape.BottomCap, Vec3.UP) < 0.0f)
			{
				IsGround = true;
				VerticalVelocity = Physics.Gravity.z * ifps;
			}
		}
#if DEBUG
		if (debugVisualizer.enabled && debugVisualizer.downPassContacts)
		{
			foreach (var c in contacts)
				Visualizer.RenderVector(c.Point, c.Point + c.Normal, debugColors.downPassContacts);
		}
		if (debugProfiler.enabled && debugProfiler.downPassContact)
			Profiler.SetValue("Down Pass Contacts ", "", contacts.Count, maxPassContacts, debugColors.arrayDownPassContacts);
#endif
	}
	private void UpdateCrouch(float ifps)
	{
		if (!useCrouch)
			return;
		// get state of crouch key
		bool isKey = (Input.IsKeyPressed(crouchKey) || gamePad && gamePad.IsButtonPressed(crouchButton));
		// determine the subsequent behavior depending on the current phase
		switch (crouchState.phase)
		{
			case CrouchPhase.STAND:
				if (isKey)
				{
					// go into a state of smooth movement down
					// set begin height to full player height
					// set end height to crouch player height
					// and activate crouch state
					crouchState.phase = CrouchPhase.MOVE_DOWN;
					SwapInterpolationDirection(capsuleHeight + 2.0f * shape.Radius, crouchHeight);
					IsCrouch = true;
				}
				break;
			case CrouchPhase.MOVE_DOWN:
			case CrouchPhase.CROUCH:
				if (!isKey)
				{
					// set player’s full height and check if we can get up
					bool canStandUp = true;
					// set shape parameters for standing position
					// use width offset to avoid false wall contacts
					float radius = shape.Radius;
					shape.Radius = radius - skinWidthOffset;
					UpdatePlayerHeight(capsuleHeight + 2.0f * skinWidthOffset);
					// check current collisions
					shape.GetCollision(contacts);
					Scalar topPoint = worldTransform.Translate.z + crouchHeight;
					for (int i = 0; i < contacts.Count; i++)
						if (contacts[i].Point.z > topPoint)
						{
							// some collisions are higher than crouch height and we can't stand up
							canStandUp = false;
							break;
						}
					// set current shape parameters
					shape.Radius = radius;
					UpdatePlayerHeight(crouchState.currentHeight - 2.0f * shape.Radius);
					if (canStandUp)
					{
						// go into a state of smooth movement up
						// set begin height to crouch player height
						// set end height to full player height
						crouchState.phase = CrouchPhase.MOVE_UP;
						SwapInterpolationDirection(crouchHeight, capsuleHeight + 2.0f * shape.Radius);
					}
				}
				break;
			case CrouchPhase.MOVE_UP:
				if (IsCeiling || isKey)
				{
					// if we touched an obstacle from above or the key is pressed again,
					// we go into a state of smooth movement down
					// set begin height to full player height
					// set end height to crouch player height
					crouchState.phase = CrouchPhase.MOVE_DOWN;
					SwapInterpolationDirection(capsuleHeight + 2.0f * shape.Radius, crouchHeight);
				}
				break;
			default: break;
		}
		// handle smooth motion
		if (crouchState.currentTime > 0.0f)
		{
			// get current linear interpolation coefficient based on current phase time
			float t = 1.0f;
			if (MathLib.Equals(crouchTransitionTime, MathLib.EPSILON) == false)
				t = MathLib.Saturate(1.0f - crouchState.currentTime / crouchTransitionTime);
			// update current player height
			crouchState.currentHeight = MathLib.Lerp(crouchState.startHeight, crouchState.endHeight, t);
			UpdatePlayerHeight(crouchState.currentHeight - 2.0f * shape.Radius);
			crouchState.currentTime -= ifps;
			// handle final step of smooth motion
			if (crouchState.currentTime <= 0.0f)
			{
				// set final time and height
				crouchState.currentTime = 0.0f;
				crouchState.currentHeight = crouchState.endHeight;
				switch (crouchState.phase)
				{
					case CrouchPhase.MOVE_DOWN:
						// set crouch player height and go into crouch phase
						UpdatePlayerHeight(crouchState.currentHeight - 2.0f * shape.Radius);
						crouchState.phase = CrouchPhase.CROUCH;
						break;
					case CrouchPhase.MOVE_UP:
						// set full player height and go into stand phase
						// also disable crouch state
						UpdatePlayerHeight(crouchState.currentHeight - 2.0f * shape.Radius);
						crouchState.phase = CrouchPhase.STAND;
						IsCrouch = false;
						break;
					default: break;
				}
			}
		}
	}
	private void UpdateCamera()
	{
		if (!camera || cameraMode == CameraMode.NONE)
			return;
		if (Input.MouseGrab)
		{
			float delta = -Input.MouseDeltaPosition.y * mouseSensitivity * 0.1f;
			if (gamePad)
			{
				vec2 rotateValue = (cameraStick == GamepadStickSide.LEFT ? gamePad.AxesLeft : gamePad.AxesRight);
				if (rotateValue.Length > sticksDeadZone && MathLib.Abs(rotateValue.y * cameraStickSensitivity) > MathLib.Abs(delta))
					delta = rotateValue.y * cameraStickSensitivity;
			}
			cameraVerticalAngle += delta;
			cameraVerticalAngle = MathLib.Clamp(cameraVerticalAngle, minVerticalAngle + 90.0f, maxVerticalAngle + 90.0f);
		}
		// update camera transformation taking into account all additional offsets of position and rotation
		camera.WorldPosition = worldTransform * (new Vec3(cameraPositionOffset) + cameraCrouchOffset + AdditionalCameraOffset);
		vec3 cameraDirection = vec3.FORWARD * MathLib.RotateZ(-cameraHorizontalAngle);
		cameraDirection = cameraDirection * MathLib.Rotate(MathLib.Cross(cameraDirection, vec3.UP), 90.0f - cameraVerticalAngle);
		cameraDirection = AdditionalCameraRotation * cameraDirection;
		cameraDirection.Normalize();
		camera.SetWorldDirection(cameraDirection, vec3.UP);
	}
	private void SwapInterpolationDirection(Scalar startHeight, Scalar endHeight)
	{
		crouchState.currentTime = MathLib.Max(MathLib.EPSILON, crouchTransitionTime - crouchState.currentTime);
		crouchState.startHeight = startHeight;
		crouchState.endHeight = endHeight;
	}
	private void UpdatePlayerHeight(Scalar height)
	{
		shape.Height = (float)height;
		cameraCrouchOffset = vec3.UP * (float)(height - capsuleHeight);
		shape.Transform = worldTransform * MathLib.Translate(Vec3.UP * (shape.Radius + height * 0.5f));
	}
	#region Debug
#if DEBUG
	[MethodInit]
	private void InitDebug()
	{
		if (!IsInitialized)
			return;
		// debug camera
		if (debugCamera.enabled)
		{
			debugCamera.camera = new PlayerDummy(); ;
			debugCamera.angularSpeed = 90.0f;
			debugCamera.zoomSpeed = 3.0f;
			debugCamera.maxDistance = 10.0f;
			debugCamera.horizontalAngle = 0.0f;
			debugCamera.verticalAngle = 0.0f;
			debugCamera.distance = debugCamera.maxDistance * 0.5f;
			Game.Player = debugCamera.camera;
			debugCamera.camera.SetWorldDirection(vec3.FORWARD, vec3.UP);
			debugCamera.camera.WorldPosition = (worldTransform.Translate + vec3.UP * (shape.Radius + shape.Height * 0.5f)) - vec3.FORWARD * debugCamera.distance;
		}
		// use visualizer
		Visualizer.Enabled = debugVisualizer.enabled;
		if (debugVisualizer.enabled)
		{
			Render.ShowTriangles = (debugVisualizer.triangles ? 1 : 0);
			int showShapes = (debugVisualizer.shapes ? 1 : 0);
			Unigine.Console.Run($"physics_show_shapes {showShapes}");
		}
		// use profiler
		int showProfiler = (debugProfiler.enabled ? 1 : 0);
		Unigine.Console.Run($"show_profiler {showProfiler}");
		if (debugProfiler.enabled)
		{
			// applied horizontal speed
			maxAppliedHorizontalSpeed = MathLib.Max(crouchSpeed, walkSpeed);
			maxAppliedHorizontalSpeed = MathLib.Max(maxAppliedHorizontalSpeed, runSpeed);
			maxAppliedHorizontalSpeed *= 1.1f;
			debugColors.arrayAppliedHorizontalVelocity = new float[4]
			{
				debugColors.appliedHorizontalVelocity.x,
				debugColors.appliedHorizontalVelocity.y,
				debugColors.appliedHorizontalVelocity.z,
				1.0f
			};
			// applied vertical speed
			debugColors.arrayAppliedVerticalVelocity = new float[4]
			{
				debugColors.appliedVerticalVelocity.x,
				debugColors.appliedVerticalVelocity.y,
				debugColors.appliedVerticalVelocity.z,
				1.0f
			};
			// up pass contacts
			debugColors.arrayUpPassContacts = new float[4]
			{
				debugColors.upPassContacts.x,
				debugColors.upPassContacts.y,
				debugColors.upPassContacts.z,
				1.0f
			};
			// side pass contacts
			debugColors.arraySidePassContacts = new float[4]
			{
				debugColors.sidePassContacts.x,
				debugColors.sidePassContacts.y,
				debugColors.sidePassContacts.z,
				1.0f
			};
			// down pass contacts
			debugColors.arrayDownPassContacts = new float[4]
			{
				debugColors.downPassContacts.x,
				debugColors.downPassContacts.y,
				debugColors.downPassContacts.z,
				1.0f
			};
			// is ground
			debugColors.arrayIsGround = new float[4]
			{
				debugColors.isGround.x,
				debugColors.isGround.y,
				debugColors.isGround.z,
				1.0f
			};
			// is ceiling
			debugColors.arrayIsCeiling = new float[4]
			{
				debugColors.isCeiling.x,
				debugColors.isCeiling.y,
				debugColors.isCeiling.z,
				1.0f
			};
			// is crouch
			debugColors.arrayIsCrouch = new float[4]
			{
				debugColors.isCrouch.x,
				debugColors.isCrouch.y,
				debugColors.isCrouch.z,
				1.0f
			};
			// average speed
			debugColors.arrayAverageSpeed = new float[4]
			{
				debugColors.averageSpeed.x,
				debugColors.averageSpeed.y,
				debugColors.averageSpeed.z,
				1.0f
			};
			// profile auto stepping
			debugColors.arrayAutoStepping = new float[4]
			{
				debugColors.autoStepping.x,
				debugColors.autoStepping.y,
				debugColors.autoStepping.z,
				1.0f
			};
		}
		lastPlayerPosition = worldTransform.Translate;
	}
	[MethodUpdate]
	private void UpdateDebug()
	{
		if (!IsInitialized)
			return;
		// debug camera
		if (debugCamera.enabled)
		{
			if (!debugCamera.useFixedAngles)
			{
				if (Input.IsKeyPressed(Input.KEY.UP))
					debugCamera.verticalAngle += debugCamera.angularSpeed * Game.IFps;
				if (Input.IsKeyPressed(Input.KEY.DOWN))
					debugCamera.verticalAngle -= debugCamera.angularSpeed * Game.IFps;
				debugCamera.verticalAngle = MathLib.Clamp(debugCamera.verticalAngle, -89.9f, 89.9f);
				if (Input.IsKeyPressed(Input.KEY.RIGHT))
					debugCamera.horizontalAngle -= debugCamera.angularSpeed * Game.IFps;
				if (Input.IsKeyPressed(Input.KEY.LEFT))
					debugCamera.horizontalAngle += debugCamera.angularSpeed * Game.IFps;
				if (debugCamera.horizontalAngle < -180.0f || 180.0f < debugCamera.horizontalAngle)
					debugCamera.horizontalAngle -= MathLib.Sign(debugCamera.horizontalAngle) * 360.0f;
			}
			if (Input.IsKeyPressed(Input.KEY.EQUALS))
				debugCamera.distance -= debugCamera.zoomSpeed * Game.IFps;
			if (Input.IsKeyPressed(Input.KEY.MINUS))
				debugCamera.distance += debugCamera.zoomSpeed * Game.IFps;
			debugCamera.distance = MathLib.Clamp(debugCamera.distance, 0.0f, debugCamera.maxDistance);
			vec3 cameraDirection = debugCamera.camera.GetDirection();
			if (debugCamera.useFixedAngles && camera)
			{
				if (MathLib.Dot(camera.GetDirection(), vec3.DOWN) < 1.0f)
					cameraDirection = camera.GetWorldDirection();
			}
			else
			{
				cameraDirection = vec3.FORWARD * MathLib.RotateZ(debugCamera.horizontalAngle);
				cameraDirection = cameraDirection * MathLib.Rotate(MathLib.Cross(cameraDirection, vec3.UP), debugCamera.verticalAngle);
			}
			debugCamera.camera.SetWorldDirection(cameraDirection, vec3.UP);
			debugCamera.camera.WorldPosition = (worldTransform.Translate + vec3.UP * (shape.Radius + shape.Height * 0.5f)) - cameraDirection * debugCamera.distance;
		}
		// use visualizer
		if (debugVisualizer.enabled)
		{
			if (debugVisualizer.playerShape)
				shape.RenderVisualizer(debugColors.playerShape);
			if (debugVisualizer.playerDirection)
			{
				Vec3 p0 = worldTransform.Translate + Vec3.UP * (shape.Radius + shape.Height * 0.5f);
				Vec3 p1 = p0 + new vec3(worldTransform.AxisY);
				Visualizer.RenderVector(p0, p1, debugColors.playerDirection);
			}
			if (debugVisualizer.camera && camera)
			{
				Vec3 p0 = camera.WorldPosition;
				Vec3 p1 = p0 + camera.GetWorldDirection();
				Visualizer.RenderVector(p0, p1, debugColors.cameraColor);
				camera.RenderVisualizer();
			}
			if (debugVisualizer.slopeBasis)
			{
				Vec3 p0 = worldTransform.Translate;
				Visualizer.RenderVector(p0, p0 + SlopeAxisX, vec4.RED);
				Visualizer.RenderVector(p0, p0 + SlopeAxisY, vec4.GREEN);
				Visualizer.RenderVector(p0, p0 + SlopeNormal, vec4.BLUE);
			}
			if (debugVisualizer.appliedHorizontalVelocity)
			{
				Vec3 p0 = worldTransform.Translate + Vec3.UP * (shape.Radius + shape.Height * 0.5f);
				Vec3 p1 = p0 + HorizontalVelocity;
				Visualizer.RenderVector(p0, p1, debugColors.appliedHorizontalVelocity);
			}
			if (debugVisualizer.appliedVerticalVelocity)
			{
				Vec3 p0 = worldTransform.Translate + Vec3.UP * (shape.Radius + shape.Height * 0.5f);
				Vec3 p1 = p0 + vec3.UP * VerticalVelocity;
				Visualizer.RenderVector(p0, p1, debugColors.appliedVerticalVelocity);
			}
		}
		// use profiler
		if (debugProfiler.enabled)
		{
			if (debugProfiler.appliedHorizontalSpeed)
				Profiler.SetValue("Applied Horizontal Speed", "m/s", (float)HorizontalVelocity.Length, maxAppliedHorizontalSpeed, debugColors.arrayAppliedHorizontalVelocity);
			if (debugProfiler.appliedVerticalSpeed)
				Profiler.SetValue("|Applied Vertical Speed|", "m/s", MathLib.Abs(VerticalVelocity), maxAppliedVerticalSpeed, debugColors.arrayAppliedVerticalVelocity);
			if (debugProfiler.isGround)
				Profiler.SetValue("Is Ground", "", (IsGround ? 1.0f : 0.0f), maxFlagValue, debugColors.arrayIsGround);
			if (debugProfiler.isCeiling)
				Profiler.SetValue("Is Ceiling", "", (IsCeiling ? 1.0f : 0.0f), maxFlagValue, debugColors.arrayIsCeiling);
			if (debugProfiler.isCrouch)
				Profiler.SetValue("Is Crouch", "", (IsCrouch ? 1.0f : 0.0f), maxFlagValue, debugColors.arrayIsCrouch);
			if (debugProfiler.averageSpeed)
			{
				for (int i = 0; i < speedsBuffer.Length - 1; i++)
					speedsBuffer[i] = speedsBuffer[i + 1];
				speedsBuffer[speedsBuffer.Length - 1] = (float)(worldTransform.Translate - lastPlayerPosition).Length / Game.IFps;
				lastPlayerPosition = worldTransform.Translate;
				float avgSpeed = 0.0f;
				for (int i = 0; i < speedsBuffer.Length; i++)
					avgSpeed += speedsBuffer[i];
				avgSpeed /= (float)speedsBuffer.Length;
				Profiler.SetValue("Avg Speed", "m/s", avgSpeed, maxAppliedHorizontalSpeed * 1.75f, debugColors.arrayAverageSpeed);
			}
			if (debugProfiler.autoStepping)
				Profiler.SetValue("Auto Stepping", "", (autoSteppingApplied ? 1.0f : 0.0f), maxFlagValue, debugColors.arrayAutoStepping);
		}
	}
#endif
	#endregion Debug
}
```
---
## GuiToTexture.cs
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Diagnostics;
using Unigine;
using Object = Unigine.Object;
/// <summary>
/// This sample demonstrates how to render gui onto a custom texture
/// In <c> GuiToTexture </c> component we pass the texture into material's texture slot
/// </summary>
[Component(PropertyGuid = "58738de84f233d8ef90217a386f54c5fa43dd9c7")]
public class GuiToTexture : Component
{
	/// <summary>
	/// Name of surface to which the material is applied
	/// </summary>
	[ShowInEditor] private string surfaceName = "";
	/// <summary>
	/// Names of texture slots in material
	/// </summary>
	[ShowInEditor] private string[] textureSlotNames = [];
	/// <summary>
	/// Resolution of texture that will be created and assigned to material
	/// </summary>
	[ShowInEditor] public ivec2 TextureResolution = new(2048, 2048);
	/// <summary>
	/// Texture in which gui will be rendered
	/// </summary>
	private Texture guiTexture;
	/// <summary>
	/// Render target that will be used to render gui on texture
	/// </summary>
	private RenderTarget renderTarget;
	/// <summary>
	/// If this flag is enabled, gui texture be updated each frame
	/// </summary>
	public bool AutoUpdateEnabled = true;
	/// <summary>
	/// Gui that is rendered
	/// </summary>
	public Gui Gui { get; private set; }
	public void RenderToTexture()
	{
		// Render gui onto texture
		// Save render state and put it at the top of the stack
		// To pop current settings, we will need to call RenderState.RestoreState() at the and of this method
		RenderState.SaveState();
		// Now we clear state, so that our rendered texture won't be affected by other render activities
		RenderState.ClearStates();
		// Set viewport size matching texture resolution
		RenderState.SetViewport(0, 0, TextureResolution.x, TextureResolution.y);
		// Now we bind gui texture to slot 0, because gui renders in slot 0
		renderTarget.BindColorTexture(0, guiTexture);
		// Enable render target
		renderTarget.Enable();
		// Clear texture and fill it with black color
		RenderState.ClearBuffer(RenderState.BUFFER_COLOR, vec4.BLACK);
		// Now we need to perform the whole gui render loop
		// Enable gui so that it will be updated and rendered
		Gui.Enable();
		// Update all widgets
		Gui.Update();
		// Render gui
		Gui.PreRender();
		Gui.Render();
		// Disable gui
		Gui.Disable();
		// Now we need to free render target and unbind texture
		renderTarget.Disable();
		renderTarget.UnbindColorTexture(0);
		// Create texture mipmaps (set of textures of different resolutions to ensure correct rendering at longer distances)
		guiTexture.CreateMipmaps();
		// Pop render state from top of the stack to let render pipeline continue as usual
		RenderState.RestoreState();
	}
	[MethodInit(Order = -1)] // We need to initialize this component before other components that will use this gui
	void Init()
	{
		// Obtain object from the node this component is attached to
		var obj = node as Object;
		if (obj == null)
		{
			Log.Error("GuiToTexture.Init(): component must be assigned to an object");
			return;
		}
		// Find the required surface
		int surface = obj.FindSurface(surfaceName);
		if (surface == -1)
		{
			Log.Error("GuiToTexture.Init(): surface with name %s not found", surfaceName);
			return;
		}
		renderTarget = new RenderTarget();
		// We need to inherit material, because there might be other objects that are using this material
		// and we don't want all objects in the scene to get gui from this component
		Material material = obj.GetMaterialInherit(surface);
		Gui = Gui.Create();
		Gui.Size = TextureResolution;
		guiTexture = new Texture();
		// here we need to specify format of texture: rgba8
		// and set the flag FORMAT_USAGE_RENDER to be able to render into the texture
		// we also need to specify the sampler by setting another flag (bilinear sampler in our case)
		guiTexture.Create2D(TextureResolution.x, TextureResolution.y, Texture.FORMAT_RGBA8,
			Texture.FORMAT_USAGE_RENDER | Texture.SAMPLER_FILTER_BILINEAR);
		for (int i = 0, num_textures = textureSlotNames.Length; i < num_textures; i++)
		{
			material.SetTexture(textureSlotNames[i], guiTexture);
		}
	}
	void Update()
	{
		// Update gui if flag is set
		if (AutoUpdateEnabled)
			RenderToTexture();
	}
}
```
---
## IFpsMovementController.cs
```csharp
﻿using Unigine;
#region Math Variables
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Mat4 = Unigine.mat4;
#endif
#endregion
[Component(PropertyGuid = "151c20a121dd9f07fca828d340bb5962f855e124")]
public class IFpsMovementController : Component
{
	[ShowInEditor]
	[Parameter(Title = "Use IFps")]
	private bool useIFps = false;
	[ShowInEditor]
	[Parameter(Title = "Movement speed")]
	private float movementSpeed = 1.0f;
	Vec3 current_dir = Vec3.RIGHT;
	void Update()
	{
		if (useIFps)
		{
			node.Translate(current_dir * movementSpeed * Game.IFps);
		}
		else
		{
			node.Translate(current_dir * movementSpeed);
		}
		if (node.WorldPosition.x > 5)
			current_dir = Vec3.LEFT;
		if (node.WorldPosition.x < -5)
			current_dir = Vec3.RIGHT;
	}
}
```
---
## IFpsMovementSample.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "736f477ad32150586cf44810ad19da883434cf17")]
public class IFpsMovementSample : Component
{
	private SampleDescriptionWindow window = null;
	private WidgetSlider maxFpsSlider = null;
	private void Init()
	{
		window = new SampleDescriptionWindow();
		window.createWindow();
		WidgetGroupBox parameters = window.getParameterGroupBox();
		maxFpsSlider = window.addIntParameter("Max render fps:", "Max render fps:", Render.MaxFPS, 15, 150, (int value) =>
		{
			Render.MaxFPS = value;
		});
	}
	private void Shutdown()
	{
		window.shutdown();
	}
}
```
---
## ImagesSample.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "2041c058f0c844eadd8476060da062b46f82f438")]
public class ImagesSample : Component
{
	private float size;
	private float velocity;
	private float radius;
	private int num_fields;
	private vec3[] positions;
	private vec3[] velocities;
	private float[] radiuses;
	private Image image;
	Material material;
	void Init()
	{
		image = new Image();
		image.Create3D(32, 32, 32, Image.FORMAT_RGBA8);
		image_init();
		ObjectVolumeBox obj = new ObjectVolumeBox(new vec3(20.0f));
		obj.SetMaterial(Materials.FindManualMaterial("Unigine::volume_cloud_base"), "*");
		obj.SetMaterialState("samples", 2, 0);
		obj.Transform = MathLib.Translate(new vec3(0.0f, 0.0f, 1.0f));
		material = obj.GetMaterialInherit(0);
	}
	void Update()
	{
		image_update();
		material.SetTextureImage(material.FindTexture("density_3d"), image);
	}
	private void image_init()
	{
		size = 2.0f;
		velocity = 1.0f;
		radius = 0.5f;
		num_fields = 16;
		positions = new vec3[num_fields];
		velocities = new vec3[num_fields];
		radiuses = new float[num_fields];
		for (int i = 0; i < num_fields; i++)
		{
			positions[i] = MathLib.RandVec3(0.0f, size);
			velocities[i] = MathLib.RandVec3(-velocity, velocity);
			radiuses[i] = MathLib.RandFloat(radius / 2.0f, radius);
		}
	}
	private void image_update()
	{
		float ifps = Game.IFps;
		for (int i = 0; i < num_fields; i++)
		{
			vec3 p = positions[i] + velocities[i] * ifps;
			if (p.x < 0.0f || p.x > size) velocities[i].x = -velocities[i].x;
			if (p.y < 0.0f || p.y > size) velocities[i].y = -velocities[i].y;
			if (p.z < 0.0f || p.z > size) velocities[i].z = -velocities[i].z;
			positions[i] += velocities[i] * ifps;
		}
		int width = image.Width;
		int height = image.Height;
		int depth = image.Depth;
		float iwidth = size / width;
		float iheight = size / height;
		float idepth = size / depth;
		vec3 position = new vec3(0.0f);
		Image.Pixel pixel = new Image.Pixel();
		for (int z = 0; z < depth; z++)
		{
			position.z = z * idepth;
			for (int y = 0; y < height; y++)
			{
				position.y = y * iheight;
				for (int x = 0; x < width; x++)
				{
					position.x = x * iwidth;
					float field = 0.0f;
					for (int i = 0; i < num_fields; i++)
					{
						float distance = MathLib.Distance2(positions[i], position);
						if (distance < radiuses[i])
							field += radiuses[i] - distance;
						pixel.i.x = pixel.i.y = pixel.i.z = pixel.i.w = (int)(MathLib.Saturate(field) * 255.0f);
						image.Set3D(x, y, z, pixel);
					}
				}
			}
		}
	}
}
```
---
## InputGamePadComponent.cs
```csharp
﻿using System;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "878de438f109da37b63ff22a9e44fb8151be1eec")]
public class InputGamePadComponent : Component
{
	public int CountGamePads { get; private set; } = 0;
	public int CountActiveGamePads { get; private set; } = 0;
	public class GamepadInfo
	{
		public string name;
		public int number;
		public int playerIndex;
		public Input.DEVICE deviceType;
		public InputGamePad.MODEL_TYPE modelType;
		public bool isAvailable;
		public float filter;
		public Input.GAMEPAD_BUTTON? lastButtonDown;
		public Input.GAMEPAD_BUTTON? lastButtonPressed;
		public Input.GAMEPAD_BUTTON? lastButtonUp;
		public vec2 axesLeft;
		public vec2 axesLeftLastDelta;
		public vec2 axesRight;
		public vec2 axesRightLastDelta;
		public float triggerLeft;
		public float triggerLeftLastDelta;
		public float triggerRight;
		public float triggerRightLastDelta;
	}
	private List<InputGamePad> activeGamepads = null;
	public List<GamepadInfo> GamepadsInfo { get; private set; } = null;
	private Array buttons = null;
	public Action<InputGamePad, int> onTouch;
	private void Init()
	{
		// get all available gamepad buttons
		buttons = Enum.GetValues(typeof(Input.GAMEPAD_BUTTON));
		// set collections of active gamepads and their states
		activeGamepads = new List<InputGamePad>();
		GamepadsInfo = new List<GamepadInfo>();
		InputGamepadUI.filterChanged += OnFilterChanged;
		InputGamepadUI.setVibration += OnSetVibration;
	}
	[MethodUpdate(Order=1)]
	private void Update()
	{
		// update current active gamepads
		if (CountActiveGamePads != Input.NumGamePads)
		{
			activeGamepads.Clear();
			GamepadsInfo.Clear();
			CountActiveGamePads = Input.NumGamePads;
			for (int i = 0; i < CountActiveGamePads; i++)
			{
				InputGamePad gamepad = Input.GetGamePad(i);
				activeGamepads.Add(gamepad);
				GamepadInfo info = new GamepadInfo()
				{
					number = gamepad.Number,
					isAvailable = gamepad.IsAvailable
				};
				GamepadsInfo.Add(info);
			}
		}
		// update information about gamepads
		for (int i = 0; i < CountActiveGamePads; i++)
		{
			GamepadsInfo[i].name = activeGamepads[i].Name;
			GamepadsInfo[i].playerIndex = activeGamepads[i].PlayerIndex;
			GamepadsInfo[i].deviceType = activeGamepads[i].DeviceType;
			GamepadsInfo[i].modelType = activeGamepads[i].ModelType;
			GamepadsInfo[i].isAvailable = activeGamepads[i].IsAvailable;
			if (!GamepadsInfo[i].isAvailable)
				continue;
			GamepadsInfo[i].filter = activeGamepads[i].Filter;
			foreach (var button in buttons)
			{
				Input.GAMEPAD_BUTTON currentButton = (Input.GAMEPAD_BUTTON)button;
				if (currentButton == Input.GAMEPAD_BUTTON.NUM_GAMEPAD_BUTTONS)
					continue;
				// update buttons
				if (activeGamepads[i].IsButtonDown(currentButton))
					GamepadsInfo[i].lastButtonDown = currentButton;
				if (activeGamepads[i].IsButtonPressed(currentButton))
					GamepadsInfo[i].lastButtonPressed = currentButton;
				if (activeGamepads[i].IsButtonUp(currentButton))
					GamepadsInfo[i].lastButtonUp = currentButton;
			}
			// update axes and deltas
			GamepadsInfo[i].axesLeft = activeGamepads[i].AxesLeft;
			if (activeGamepads[i].AxesLeftDelta.Length2 > 0.0f)
				GamepadsInfo[i].axesLeftLastDelta = activeGamepads[i].AxesLeftDelta;
			GamepadsInfo[i].axesRight = activeGamepads[i].AxesRight;
			if (activeGamepads[i].AxesRightDelta.Length2 > 0.0f)
				GamepadsInfo[i].axesRightLastDelta = activeGamepads[i].AxesRightDelta;
			// update tirggers and deltas
			GamepadsInfo[i].triggerLeft = activeGamepads[i].TriggerLeft;
			if (activeGamepads[i].TriggerLeftDelta > 0.0f)
				GamepadsInfo[i].triggerLeftLastDelta = activeGamepads[i].TriggerLeftDelta;
			GamepadsInfo[i].triggerRight = activeGamepads[i].TriggerRight;
			if (activeGamepads[i].TriggerRightDelta > 0.0f)
				GamepadsInfo[i].triggerRightLastDelta = activeGamepads[i].TriggerRightDelta;
			if (activeGamepads[i].NumTouches > 0)
			{
				onTouch.Invoke(activeGamepads[i], i);
			}
		}
	}
	private void Shutdown()
	{
		InputGamepadUI.filterChanged -= OnFilterChanged;
		InputGamepadUI.setVibration -= OnSetVibration;
	}
	private void OnFilterChanged(int number, float value)
	{
		if (0 <= number && number < activeGamepads.Count)
			activeGamepads[number].Filter = value;
		Log.MessageLine($"{value}");
	}
	private void OnSetVibration(int number, float lowFrequency, float highFrequency, float duration)
	{
		if (0 <= number && number < activeGamepads.Count)
			activeGamepads[number].SetVibration(lowFrequency, highFrequency, duration);
	}
}
```
---
## InputGamepadUI.cs
```csharp
﻿using System;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "740808362162e21d6960fcd8ec31da576a74b3ca")]
public class InputGamepadUI : Component
{
	public InputGamePadComponent gamepadComponent = null;
	[ParameterFile(Filter = ".ui")]
	public string uiFile = null;
	[ParameterFile(Filter = ".ui")]
	public string uiInfoFile = null;
	static public event Action<int, float> filterChanged;
	static public event Action<int, float, float, float> setVibration;
	private class InfoWidgets
	{
		public WidgetVBox infoVBox = null;
		public WidgetLabel nameLabel = null;
		public WidgetLabel numberLabel = null;
		public WidgetLabel playerIndexLabel = null;
		public WidgetLabel deviceTypeLabel = null;
		public WidgetLabel modelTypeLabel = null;
		public WidgetLabel availableLabel = null;
		public WidgetSlider filterSlider = null;
		public WidgetLabel lastButtonDownLabel = null;
		public WidgetLabel lastButtonPressedLabel = null;
		public WidgetLabel lastButtonUpLabel = null;
		public WidgetLabel leftAxesXLabel = null;
		public WidgetLabel leftAxesYLabel = null;
		public WidgetLabel rightAxesXLabel = null;
		public WidgetLabel rightAxesYLabel = null;
		public WidgetLabel leftLastDeltaXLabel = null;
		public WidgetLabel leftLastDeltaYLabel = null;
		public WidgetLabel rightLastDeltaXLabel = null;
		public WidgetLabel rightLastDeltaYLabel = null;
		public WidgetLabel leftTriggerLabel = null;
		public WidgetLabel rightTriggerLabel = null;
		public WidgetLabel lastLeftTriggerDeltaLabel = null;
		public WidgetLabel lastRightTriggerDeltaLabel = null;
		public WidgetSlider lowFrequencySlider = null;
		public WidgetSlider highFrequencySlider = null;
		public WidgetSlider durationSlider = null;
		public WidgetButton vibrationButton = null;
		public WidgetCanvas touchpadCanvas = null;
	}
	private UserInterface ui = null;
	private List<UserInterface> infosUi = null;
	private List<InfoWidgets> infoWidgets = null;
	private WidgetHBox mainHBox = null;
	private WidgetVBox backgroundVBox = null;
	private WidgetLabel errorMessageLabel = null;
	private WidgetVBox generalMessageInfo = null;
	private WidgetLabel countGamepadsLabel = null;
	private WidgetLabel countActiveGamepadsLabel = null;
	private WidgetHBox gamepadsInfoBox = null;
	private int lastGamepadsCount = 0;
	[MethodInit(Order = -1)]
	private void Init()
	{
		Input.MouseHandle = Input.MOUSE_HANDLE.USER;
		infosUi = new List<UserInterface>();
		infoWidgets = new List<InfoWidgets>();
		Gui gui = Gui.GetCurrent();
		ui = new UserInterface(gui, uiFile);
		InitWidget(nameof(mainHBox), out mainHBox, ui);
		InitWidget(nameof(backgroundVBox), out backgroundVBox, ui);
		InitWidget(nameof(errorMessageLabel), out errorMessageLabel, ui);
		InitWidget(nameof(generalMessageInfo), out generalMessageInfo, ui);
		InitWidget(nameof(countGamepadsLabel), out countGamepadsLabel, ui);
		InitWidget(nameof(countActiveGamepadsLabel), out countActiveGamepadsLabel, ui);
		InitWidget(nameof(gamepadsInfoBox), out gamepadsInfoBox, ui);
		gui.AddChild(mainHBox);
		backgroundVBox.BackgroundColor = new vec4(0.0f, 0.0f, 0.0f, 0.5f);
		gamepadComponent.onTouch += DrawTouchInfo;
	}
	private void Update()
	{
		if (gamepadComponent == null)
			return;
		// update ui for current gamepads
		if (lastGamepadsCount != gamepadComponent.CountActiveGamePads)
		{
			UpdateUI();
			lastGamepadsCount = gamepadComponent.CountActiveGamePads;
		}
		// show information about gamepads
		for (int i = 0; i < gamepadComponent.CountActiveGamePads; i++)
		{
			infoWidgets[i].nameLabel.Text = gamepadComponent.GamepadsInfo[i].name;
			infoWidgets[i].deviceTypeLabel.Text = gamepadComponent.GamepadsInfo[i].deviceType.ToString();
			infoWidgets[i].modelTypeLabel.Text = gamepadComponent.GamepadsInfo[i].modelType.ToString();
			infoWidgets[i].playerIndexLabel.Text = gamepadComponent.GamepadsInfo[i].playerIndex.ToString();
			infoWidgets[i].availableLabel.Text = gamepadComponent.GamepadsInfo[i].isAvailable.ToString();
			infoWidgets[i].lastButtonDownLabel.Text = gamepadComponent.GamepadsInfo[i].lastButtonDown?.ToString();
			infoWidgets[i].lastButtonPressedLabel.Text = gamepadComponent.GamepadsInfo[i].lastButtonPressed?.ToString();
			infoWidgets[i].lastButtonUpLabel.Text = gamepadComponent.GamepadsInfo[i].lastButtonUp?.ToString();
			infoWidgets[i].leftAxesXLabel.Text = gamepadComponent.GamepadsInfo[i].axesLeft.x.ToString("0.000");
			infoWidgets[i].leftAxesYLabel.Text = gamepadComponent.GamepadsInfo[i].axesLeft.y.ToString("0.000");
			infoWidgets[i].rightAxesXLabel.Text = gamepadComponent.GamepadsInfo[i].axesRight.x.ToString("0.000");
			infoWidgets[i].rightAxesYLabel.Text = gamepadComponent.GamepadsInfo[i].axesRight.y.ToString("0.000");
			infoWidgets[i].leftLastDeltaXLabel.Text = gamepadComponent.GamepadsInfo[i].axesLeftLastDelta.x.ToString("0.000");
			infoWidgets[i].leftLastDeltaYLabel.Text = gamepadComponent.GamepadsInfo[i].axesLeftLastDelta.y.ToString("0.000");
			infoWidgets[i].rightLastDeltaXLabel.Text = gamepadComponent.GamepadsInfo[i].axesRightLastDelta.x.ToString("0.000");
			infoWidgets[i].rightLastDeltaYLabel.Text = gamepadComponent.GamepadsInfo[i].axesRightLastDelta.y.ToString("0.000");
			infoWidgets[i].leftTriggerLabel.Text = gamepadComponent.GamepadsInfo[i].triggerLeft.ToString("0.000");
			infoWidgets[i].rightTriggerLabel.Text = gamepadComponent.GamepadsInfo[i].triggerRight.ToString("0.000");
			infoWidgets[i].lastLeftTriggerDeltaLabel.Text = gamepadComponent.GamepadsInfo[i].triggerLeftLastDelta.ToString("0.000");
			infoWidgets[i].lastRightTriggerDeltaLabel.Text = gamepadComponent.GamepadsInfo[i].triggerRightLastDelta.ToString("0.000");
			infoWidgets[i].touchpadCanvas.Clear();
		}
	}
	private void Shutdown()
	{
		Input.MouseHandle = Input.MOUSE_HANDLE.GRAB;
		if (mainHBox)
		{
			if (infosUi != null)
				for (int i = 0; i < infosUi.Count; i++)
					mainHBox.RemoveChild(infosUi[i].GetWidget(0));
			Gui.GetCurrent().RemoveChild(mainHBox);
		}
	}
	private void UpdateUI()
	{
		countActiveGamepadsLabel.Text = gamepadComponent.CountActiveGamePads.ToString();
		if (gamepadComponent.CountActiveGamePads != 0)
		{
			errorMessageLabel.Hidden = true;
			generalMessageInfo.Hidden = false;
		}
		else
		{
			errorMessageLabel.Hidden = false;
			generalMessageInfo.Hidden = true;
		}
		countGamepadsLabel.Text = gamepadComponent.CountGamePads.ToString();
		// remove current ui with gamepads information
		for (int i = 0; i < infosUi.Count; i++)
		{
			mainHBox.RemoveChild(infosUi[i].GetWidget(0));
			infosUi[i].DeleteLater();
		}
		infosUi.Clear();
		infoWidgets.Clear();
		for (int i = 0; i < gamepadComponent.CountActiveGamePads; i++)
		{
			UserInterface uiInfo = new UserInterface(Gui.GetCurrent(), uiInfoFile);
			infosUi.Add(uiInfo);
			InfoWidgets info = new InfoWidgets();
			infoWidgets.Add(info);
			InitWidget(nameof(info.infoVBox), out info.infoVBox, uiInfo);
			InitWidget(nameof(info.nameLabel), out info.nameLabel, uiInfo);
			InitWidget(nameof(info.numberLabel), out info.numberLabel, uiInfo);
			InitWidget(nameof(info.playerIndexLabel), out info.playerIndexLabel, uiInfo);
			InitWidget(nameof(info.deviceTypeLabel), out info.deviceTypeLabel, uiInfo);
			InitWidget(nameof(info.modelTypeLabel), out info.modelTypeLabel, uiInfo);
			InitWidget(nameof(info.availableLabel), out info.availableLabel, uiInfo);
			InitWidget(nameof(info.filterSlider), out info.filterSlider, uiInfo);
			InitWidget(nameof(info.lastButtonDownLabel), out info.lastButtonDownLabel, uiInfo);
			InitWidget(nameof(info.lastButtonPressedLabel), out info.lastButtonPressedLabel, uiInfo);
			InitWidget(nameof(info.lastButtonUpLabel), out info.lastButtonUpLabel, uiInfo);
			InitWidget(nameof(info.leftAxesXLabel), out info.leftAxesXLabel, uiInfo);
			InitWidget(nameof(info.leftAxesYLabel), out info.leftAxesYLabel, uiInfo);
			InitWidget(nameof(info.rightAxesXLabel), out info.rightAxesXLabel, uiInfo);
			InitWidget(nameof(info.rightAxesYLabel), out info.rightAxesYLabel, uiInfo);
			InitWidget(nameof(info.leftLastDeltaXLabel), out info.leftLastDeltaXLabel, uiInfo);
			InitWidget(nameof(info.leftLastDeltaYLabel), out info.leftLastDeltaYLabel, uiInfo);
			InitWidget(nameof(info.rightLastDeltaXLabel), out info.rightLastDeltaXLabel, uiInfo);
			InitWidget(nameof(info.rightLastDeltaYLabel), out info.rightLastDeltaYLabel, uiInfo);
			InitWidget(nameof(info.leftTriggerLabel), out info.leftTriggerLabel, uiInfo);
			InitWidget(nameof(info.rightTriggerLabel), out info.rightTriggerLabel, uiInfo);
			InitWidget(nameof(info.lastLeftTriggerDeltaLabel), out info.lastLeftTriggerDeltaLabel, uiInfo);
			InitWidget(nameof(info.lastRightTriggerDeltaLabel), out info.lastRightTriggerDeltaLabel, uiInfo);
			InitWidget(nameof(info.lowFrequencySlider), out info.lowFrequencySlider, uiInfo);
			InitWidget(nameof(info.highFrequencySlider), out info.highFrequencySlider, uiInfo);
			InitWidget(nameof(info.durationSlider), out info.durationSlider, uiInfo);
			InitWidget(nameof(info.vibrationButton), out info.vibrationButton, uiInfo);
			InitWidget(nameof(info.touchpadCanvas), out info.touchpadCanvas, uiInfo);
			gamepadsInfoBox.AddChild(info.infoVBox);
			infoWidgets[i].numberLabel.Text = gamepadComponent.GamepadsInfo[i].number.ToString();
			info.filterSlider.Data = i.ToString();
			info.filterSlider.EventChanged.Connect(ChangeFilter);
			info.vibrationButton.Data = i.ToString();
			info.vibrationButton.EventClicked.Connect(ClickVibrate);
		}
	}
	private void InitWidget<T>(string name, out T widget, UserInterface ui) where T : Widget
	{
		widget = null;
		if (!ui)
			return;
		int id = ui.FindWidget(name);
		if (id != -1)
			widget = ui.GetWidget(id) as T;
	}
	private void ChangeFilter(Widget sender)
	{
		int number = 0;
		bool res = int.TryParse(sender.Data, out number);
		WidgetSlider slider = sender as WidgetSlider;
		if (res && slider)
			filterChanged?.Invoke(number, slider.Value / 100.0f);
	}
	private void ClickVibrate(Widget widget)
	{
		bool res = int.TryParse(widget.Data, out var number);
		if (res)
			setVibration?.Invoke(
				number,
				infoWidgets[number].lowFrequencySlider.Value / 100.0f,
				infoWidgets[number].highFrequencySlider.Value / 100.0f,
				infoWidgets[number].durationSlider.Value
				);
	}
	private void DrawTouchInfo(InputGamePad gamepad, int gamepadNum)
	{
		if (infoWidgets.Count == 0) return;
		const int numberOfColors = 10;
		vec4[] colors = new vec4[numberOfColors];
		colors[0] = new vec4(1.0f, 0.0f, 0.0f, 1.0f);
		colors[1] = new vec4(0.0f, 1.0f, 0.0f, 1.0f);
		colors[2] = new vec4(0.0f, 0.0f, 1.0f, 1.0f);
		colors[3] = new vec4(1.0f, 1.0f, 0.0f, 1.0f);
		colors[4] = new vec4(0.0f, 1.0f, 1.0f, 1.0f);
		colors[5] = new vec4(1.0f, 0.0f, 1.0f, 1.0f);
		colors[6] = new vec4(1.0f, 0.0f, 1.0f, 1.0f);
		colors[7] = new vec4(0.5f, 0.0f, 0.0f, 1.0f);
		colors[8] = new vec4(0.0f, 0.5f, 0.0f, 1.0f);
		colors[9] = new vec4(0.0f, 0.0f, 0.5f, 1.0f);
		var drawCircle = (vec2 pos, WidgetCanvas canvas, vec4 color, float pressure) =>
		{
			int polygon = canvas.AddPolygon();
			canvas.SetPolygonColor(polygon, color);
			const int num = 10;
			const float radius = 10;
			for (int i = 0; i < num; ++i)
			{
				float s = Unigine.MathLib.Sin(Unigine.MathLib.PI2 * i / num) * radius * pressure + pos.x * canvas.Width;
				float c = Unigine.MathLib.Cos(Unigine.MathLib.PI2 * i / num) * radius + pressure * pos.y * canvas.Height;
				canvas.AddPolygonPoint(polygon, new vec3(s, c, 0.0f));
			}
		};
		for (int i = 0; i < gamepad.NumTouches; ++i)
		{
			for (int j = 0; j < gamepad.GetNumTouchFingers(i); ++j)
			{
				if (!(gamepad.GetTouchPressure(i, j) > MathLib.EPSILON)) continue;
				vec4 color = colors[i * gamepad.NumTouches + j];
				drawCircle.Invoke(gamepad.GetTouchPosition(i, j), infoWidgets[gamepadNum].touchpadCanvas, color,
					gamepad.GetTouchPressure(i, j));
			}
		}
	}
}
```
---
## InputJoystickComponent.cs
```csharp
﻿using System;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "e5fd1a76585a4f5d74b90918c6cbf997f9fbca0a")]
public class InputJoystickComponent : Component
{
	public int JoysticksCount { get { return Joysticks.Count; } }
	/// <summary>
	/// Holds force feedback effect data for joysticks.
	/// </summary>
	public class FfbEffectData
	{
		public Input.JOYSTICK_FORCE_FEEDBACK_EFFECT EffectType =
			Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.JOYSTICK_FORCE_FEEDBACK_CONSTANT;
		public float Force = 0;
		public bool IsPlaying = false;
		public float? Magnitude = null;
		public float? Frequency = null;
		public ulong? Duration = null;
	}
	/// <summary>
	/// Stores available force feedback effects for each joystick instance.
	/// </summary>
	public class FfbData
	{
		public FfbData() { }
		public FfbData(InputJoystick joystick)
		{
			// Check which force feedback effects are supported by the joystick.
			for (Input.JOYSTICK_FORCE_FEEDBACK_EFFECT i = 0;
			     i < Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.NUM_JOYSTICK_FORCE_FEEDBACKS; ++i)
			{
				if (joystick.IsForceFeedbackEffectSupported(i))
				{
					AvailableFfbEffects.Add(i);
				}
			}
		}
		public readonly List<Input.JOYSTICK_FORCE_FEEDBACK_EFFECT> AvailableFfbEffects = [];
	}
	/// <summary>
	/// Stores joystick details including button, axis, and force feedback data.
	/// </summary>
	public class JoystickInfo
	{
		public string Name;
		public int Number;
		public int NumButtons;
		public int NumAxes;
		public int NumPovs;
		public bool IsAvailable;
		public float Filter;
		public List<string> ButtonsName;
		public List<string> AxesName;
		public List<string> PovsName;
		public int? LastPressedButton;
		public List<float> Axes;
		public List<Input.JOYSTICK_POV> Povs;
		public InputJoystick Joystick;
		public FfbData FfbData;
	}
	private List<InputJoystick> Joysticks { get; set; } = null;
	public List<JoystickInfo> JoysticksInfo { get; private set; } = null;
	/// <summary>
	/// Plays a specified force feedback effect on a given joystick.
	/// Ensure the joystick is available and supports the effect type.
	/// <param name="data">Values of force desired force feedback effect</param>
	/// <param name="joystickId">Id of joystick, on which effect will be played</param>
	/// </summary>
	public void PlayFfbEffect(FfbEffectData data, int joystickId)
	{
		// To start ffb effect, we need a joystick to start it on
		var joystick = Joysticks[joystickId];
		if (!joystick.IsAvailable || !joystick.IsForceFeedbackEffectSupported(data.EffectType))
			return;
		// Apply the correct force feedback effect based on type.
		// Every force feedback has different arguments, but in this example we use single class to pass all data
		// So here we check if required values is present, but if you call this function directly, you don't need to check each value before call
		// For example: joystick.PlayForceFeedbackEffectConstant(0.1, -0.3);
		switch (data.EffectType)
		{
			case Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.JOYSTICK_FORCE_FEEDBACK_CONSTANT:
				if (!data.Magnitude.HasValue)
					return;
				joystick.PlayForceFeedbackEffectConstant(data.Force);
				break;
			case Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.JOYSTICK_FORCE_FEEDBACK_DAMPER:
				joystick.PlayForceFeedbackEffectDamper(data.Force);
				break;
			case Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.JOYSTICK_FORCE_FEEDBACK_FRICTION:
				joystick.PlayForceFeedbackEffectFriction(data.Force);
				break;
			case Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.JOYSTICK_FORCE_FEEDBACK_INERTIA:
				joystick.PlayForceFeedbackEffectInertia(data.Force);
				break;
			case Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.JOYSTICK_FORCE_FEEDBACK_RAMP:
				if (!data.Magnitude.HasValue || !data.Duration.HasValue)
					return;
				joystick.PlayForceFeedbackEffectRamp(data.Force, data.Duration.Value);
				break;
			case Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.JOYSTICK_FORCE_FEEDBACK_SPRING:
				joystick.PlayForceFeedbackEffectSpring(data.Force);
				break;
			//case Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.JOYSTICK_FORCE_FEEDBACK_SAWTOOTHDOWNWAVE:
			//	if (!data.Frequency.HasValue)
			//		return;
			//	joystick.PlayForceFeedbackEffectSawtoothDownWave(data.Force, data.Frequency.Value);
			//	break;
			//case Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.JOYSTICK_FORCE_FEEDBACK_SAWTOOTHUPWAVE:
			//	if (!data.Frequency.HasValue)
			//		return;
			//	joystick.PlayForceFeedbackEffectSawtoothUpWave(data.Force, data.Frequency.Value);
			//	break;
			//case Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.JOYSTICK_FORCE_FEEDBACK_SINEWAVE:
			//	if (!data.Frequency.HasValue)
			//		return;
			//	joystick.PlayForceFeedbackEffectSineWave(data.Force, data.Frequency.Value);
			//	break;
			//case Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.JOYSTICK_FORCE_FEEDBACK_SQUAREWAVE:
			//	if (!data.Frequency.HasValue)
			//		return;
			//	joystick.PlayForceFeedbackEffectSquareWave(data.Force, data.Frequency.Value);
			//	break;
			//case Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.JOYSTICK_FORCE_FEEDBACK_TRIANGLEWAVE:
			//	if (!data.Frequency.HasValue)
			//		return;
			//	joystick.PlayForceFeedbackEffectTriangleWave(data.Force, data.Frequency.Value);
			//	break;
			case Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.NUM_JOYSTICK_FORCE_FEEDBACKS:
			default:
				throw new ArgumentException(null, nameof(data));
		}
	}
	/// <summary>
	/// Stops the specified force feedback effect if supported by the joystick.
	/// <param name="effectType">Type of effect to be stopped</param>
	/// <param name="joystick">Joystick on which effect will be stopped</param>
	/// </summary>
	public void StopFfbEffect(Input.JOYSTICK_FORCE_FEEDBACK_EFFECT effectType, InputJoystick joystick)
	{
		if (joystick.IsAvailable && joystick.IsForceFeedbackEffectSupported(effectType))
			joystick.StopForceFeedbackEffect(effectType);
	}
	private void OnJoystickConnected(int number)
	{
		InputJoystick joystick = Input.GetJoystick(number);
		if (!joystick)
			return;
		AddJoystick(joystick);
	}
	private void OnJoystickDisconnected(int number)
	{
		InputJoystick joystick = Input.GetJoystick(number);
		if (!joystick)
			return;
		ClearJoystick(joystick, number);
	}
	private void Init()
	{
		// Register event listeners for joystick connections.
		Input.EventJoyConnected.Connect(OnJoystickConnected);
		Input.EventJoyDisconnected.Connect(OnJoystickDisconnected);
		Joysticks = new List<InputJoystick>();
		JoysticksInfo = new List<JoystickInfo>();
		// Initialize available joysticks.
		for (int i = 0; i < Input.NumJoysticks; i++)
		{
			// get joystick
			InputJoystick joystick = Input.GetJoystick(i);
			if (!joystick)
				break;
			AddJoystick(joystick);
		}
		InputJoystickUI.filterChanged += OnFilterChanged;
	}
	/// <summary>
	/// Updates joystick input states each frame.
	/// </summary>
	private void Update()
	{
		// Refresh joystick status and button/axis states.
		for (int i = 0; i < Joysticks.Count; i++)
		{
			JoysticksInfo[i].IsAvailable = Joysticks[i].IsAvailable;
			if (!JoysticksInfo[i].IsAvailable)
				continue;
			JoysticksInfo[i].Filter = Joysticks[i].Filter;
			// Detect last pressed button.
			for (int j = 0; j < Joysticks[i].NumButtons; j++)
			{
				if (!Joysticks[i].IsButtonPressed((uint)j))
					continue;
				JoysticksInfo[i].LastPressedButton = j;
				break;
			}
			// update axes
			for (int j = 0; j < Joysticks[i].NumAxes; j++)
				JoysticksInfo[i].Axes[j] = Joysticks[i].GetAxis((uint)j);
			// update povs
			for (int j = 0; j < Joysticks[i].NumPovs; j++)
				JoysticksInfo[i].Povs[j] = Joysticks[i].GetPov((uint)j);
		}
	}
	/// <summary>
	/// Cleans up resources when shutting down.
	/// </summary>
	private void Shutdown()
	{
		InputJoystickUI.filterChanged -= OnFilterChanged;
		// Disable all force feedback effects
		for (Input.JOYSTICK_FORCE_FEEDBACK_EFFECT i = 0;
		     i < Input.JOYSTICK_FORCE_FEEDBACK_EFFECT.NUM_JOYSTICK_FORCE_FEEDBACKS;
		     ++i)
		{
			foreach (var joystick in Joysticks)
			{
				if (joystick && joystick.IsAvailable && joystick.IsForceFeedbackEffectSupported(i))
					joystick.StopForceFeedbackEffect(i);
			}
		}
	}
	private void OnFilterChanged(int number, float value)
	{
		if (0 <= number && number < Joysticks.Count)
			Joysticks[number].Filter = value;
	}
	/// <summary>
	///  In this method we get information about joystick and store it in our wrapper
	/// </summary>
	/// <param name="joystick"></param>
	private void AddJoystick(InputJoystick joystick)
	{
		JoystickInfo info = null;
		if (joystick.Number >= 0 && joystick.Number < Joysticks.Count)
		{
			info = JoysticksInfo[joystick.Number];
		}
		else
		{
			// add joystick info and set static fields
			info = new JoystickInfo();
			JoysticksInfo.Add(info);
			Joysticks.Add(joystick);
		}
		info.Joystick = joystick;
		info.Name = joystick.Name;
		info.Number = joystick.Number;
		info.IsAvailable = joystick.IsAvailable;
		info.NumButtons = joystick.NumButtons;
		info.NumAxes = joystick.NumAxes;
		info.NumPovs = joystick.NumPovs;
		// set buttons names
		info.ButtonsName = new List<string>();
		for (uint j = 0; j < joystick.NumButtons; j++)
			info.ButtonsName.Add(joystick.GetButtonName(j));
		// set axes names and axes values
		info.AxesName = new List<string>();
		info.Axes = new List<float>();
		for (uint j = 0; j < joystick.NumAxes; j++)
		{
			info.AxesName.Add(joystick.GetAxisName(j));
			info.Axes.Add(0.0f);
		}
		// set povs names and povs values
		info.PovsName = new List<string>();
		info.Povs = new List<Input.JOYSTICK_POV>();
		for (uint j = 0; j < joystick.NumPovs; j++)
		{
			info.PovsName.Add(joystick.GetPovName(j));
			info.Povs.Add(Input.JOYSTICK_POV.NOT_PRESSED);
		}
		info.FfbData = new FfbData(joystick);
	}
	private void ClearJoystick(InputJoystick joystick, int number)
	{
		if (number < 0 && number >= Joysticks.Count)
			return;
		JoystickInfo info = JoysticksInfo[number];
		info.Name = joystick.Name;
		info.Number = number;
		info.IsAvailable = joystick.IsAvailable;
		info.NumButtons = joystick.NumButtons;
		info.NumAxes = joystick.NumAxes;
		info.NumPovs = joystick.NumPovs;
		info.LastPressedButton = null;
		// set buttons names
		info.ButtonsName = new List<string>();
		for (uint j = 0; j < joystick.NumButtons; j++)
			info.ButtonsName.Add(joystick.GetButtonName(j));
		// set axes names and axes values
		info.AxesName = new List<string>();
		info.Axes = new List<float>();
		for (uint j = 0; j < joystick.NumAxes; j++)
		{
			info.AxesName.Add(joystick.GetAxisName(j));
			info.Axes.Add(0.0f);
		}
		// set povs names and povs values
		info.PovsName = new List<string>();
		info.Povs = new List<Input.JOYSTICK_POV>();
		for (uint j = 0; j < joystick.NumPovs; j++)
		{
			info.PovsName.Add(joystick.GetPovName(j));
			info.Povs.Add(Input.JOYSTICK_POV.NOT_PRESSED);
		}
	}
}
```
---
## InputJoystickUI.cs
```csharp
﻿using System;
using System.Collections.Generic;
using System.Linq;
using Unigine;
using static InputJoystickComponent;
using FfbEffectType = Unigine.Input.JOYSTICK_FORCE_FEEDBACK_EFFECT;
[Component(PropertyGuid = "badceed413838c23535e03263c2c404d33d1e548")]
public class InputJoystickUI : Component
{
	public InputJoystickComponent joystickComponent = null;
	static public event Action<int, float> filterChanged;
	/// <summary>
	/// Wraps a widget with a label inside a horizontal layout for consistent UI.
	/// </summary>
	static Widget WrapWidget(string name, Widget toWrap, Widget parent)
	{
		var hbox = new WidgetHBox();
		var nameLabel = new WidgetLabel(name) { FontSize = 16, FontOutline = 1 };
		nameLabel.FontSize = 16;
		nameLabel.FontOutline = 1;
		hbox.SetSpace(5, 0);
		parent.AddChild(hbox, Gui.ALIGN_LEFT);
		hbox.AddChild(nameLabel);
		hbox.AddChild(toWrap);
		return hbox;
	}
	/// <summary>
	/// Handles UI elements displaying joystick information.
	/// Updates UI components dynamically based on joystick state.
	/// </summary>
	private class InfoWidgets
	{
		/// <summary>
		/// Updates the force feedback effect button state for ramp effect.
		/// </summary>
		public void Update()
		{
			rampEffectButtonController?.Update();
		}
		/// <summary>
		/// Initializes UI elements for displaying joystick information.
		/// </summary>
		public InfoWidgets()
		{
			infoVBox = new WidgetVBox();
			var hbox = new WidgetHBox { Width = 325, FontSize = 16, FontOutline = 1 };
			hbox.SetSpace(5, 0);
			infoVBox.AddChild(hbox, Gui.ALIGN_LEFT);
			infoContainer = new WidgetVBox() { FontSize = 16, FontOutline = 1 };
			hbox.AddChild(infoContainer);
			nameLabel = new WidgetLabel() { FontSize = 16, FontOutline = 1 };
			WrapWidget("Name:", nameLabel, infoContainer);
			numberLabel = new WidgetLabel() { FontSize = 16, FontOutline = 1 };
			WrapWidget("Number:", numberLabel, infoContainer);
			buttonsCountLabel = new WidgetLabel() { FontSize = 16, FontOutline = 1 };
			WrapWidget("Buttons Count:", buttonsCountLabel, infoContainer);
			axesCountLabel = new WidgetLabel() { FontSize = 16, FontOutline = 1 };
			WrapWidget("Axes Count", axesCountLabel, infoContainer);
			povsCountLabel = new WidgetLabel() { FontSize = 16, FontOutline = 1 };
			WrapWidget("Povs Count:", povsCountLabel, infoContainer);
			availableLabel = new WidgetLabel() { FontSize = 16, FontOutline = 1 };
			WrapWidget("Available:", availableLabel, infoContainer);
			{
				// wrap slider
				var filterSliderContainer = new WidgetHBox() { Width = 100 };
				filterSliderContainer.SetSpace(5, 0);
				filterSliderContainer.SetPadding(0, 0, 8, 0);
				infoContainer.AddChild(filterSliderContainer, Gui.ALIGN_LEFT);
				filterSliderContainer.AddChild(new WidgetLabel("Filter:") { FontSize = 16, FontOutline = 1 });
				filterSlider = new WidgetSlider() { Width = 100 };
				filterSliderContainer.AddChild(filterSlider);
				var filterSliderValueLabel = new WidgetLabel("0.00") { FontSize = 16, FontOutline = 1 };
				filterSliderContainer.AddChild(filterSliderValueLabel);
				filterSlider.AddAttach(filterSliderValueLabel, "%.2f", 100);
			}
			lastPressedButtonLabel = new WidgetLabel() { FontSize = 16, FontOutline = 1 };
			WrapWidget("Pressed Button", lastPressedButtonLabel, infoContainer);
			clearWidgets = [];
			axesLabels = [];
			povsLabels = [];
		}
		public WidgetVBox infoContainer = null;
		public WidgetVBox infoVBox = null;
		public WidgetLabel nameLabel = null;
		public WidgetLabel numberLabel = null;
		public WidgetLabel buttonsCountLabel = null;
		public WidgetLabel axesCountLabel = null;
		public WidgetLabel povsCountLabel = null;
		public WidgetLabel availableLabel = null;
		public WidgetSlider filterSlider = null;
		public WidgetLabel lastPressedButtonLabel = null;
		public List<WidgetLabel> axesLabels = null;
		public List<WidgetLabel> povsLabels = null;
		public List<Widget> clearWidgets = null;
		private readonly Dictionary<FfbEffectType, FfbEffectData> effectData = [];
		private FfbDurationEffectButtonController rampEffectButtonController;
		private class FfbDurationEffectButtonController(FfbEffectData effect, WidgetButton button)
		{
			private float currentTime;
			public void Update()
			{
				if (currentTime >= effect.Duration || !effect.IsPlaying)
					return;
				var iFps = Game.IFps;
				currentTime += iFps;
				if (!(currentTime >= (float)effect.Duration / 1_000_000))
					return;
				button.Toggled = false;
				effect.IsPlaying = false;
			}
		}
		public void Set(JoystickInfo info, InputJoystickComponent joystickComponent)
		{
			availableLabel.Text = info.IsAvailable.ToString();
			filterSlider.Value = (int)(info.Filter * 100.0f);
			nameLabel.Text = info.Name;
			numberLabel.Text = info.Number.ToString();
			buttonsCountLabel.Text = info.NumButtons.ToString();
			axesCountLabel.Text = info.NumAxes.ToString();
			povsCountLabel.Text = info.NumPovs.ToString();
			filterSlider.Data = info.Number.ToString();
			int? lastPressedButton = info.LastPressedButton;
			if (lastPressedButton != null)
				lastPressedButtonLabel.Text = info.ButtonsName[lastPressedButton.Value];
			if (axesLabels.Count == info.NumAxes && povsLabels.Count == info.NumPovs)
			{
				for (int i = 0; i < info.NumAxes; i++)
				{
					float value = info.Axes[i];
					axesLabels[i].Text = value.ToString("0.000");
				}
				for (int i = 0; i < info.NumPovs; i++)
				{
					Input.JOYSTICK_POV value = info.Povs[i];
					povsLabels[i].Text = value.ToString();
				}
				return;
			}
			axesLabels.Clear();
			povsLabels.Clear();
			Gui gui = Gui.GetCurrent();
			foreach (Widget w in clearWidgets)
			{
				infoContainer.RemoveChild(w);
				w.DeleteLater();
			}
			clearWidgets.Clear();
			for (int i = 0; i < info.NumAxes; i++)
			{
				WidgetHBox hbox = new WidgetHBox(gui);
				hbox.SetSpace(5, 0);
				hbox.SetPadding(0, 0, 5, 0);
				clearWidgets.Add(hbox);
				infoContainer.AddChild(hbox, Gui.ALIGN_LEFT);
				WidgetLabel nameLabel = new WidgetLabel(gui);
				nameLabel.Text = info.AxesName[i] + ":";
				nameLabel.FontSize = 16;
				nameLabel.FontOutline = 1;
				hbox.AddChild(nameLabel);
				WidgetLabel axisValue = new WidgetLabel(gui, "0.0");
				axisValue.FontSize = 16;
				axisValue.FontOutline = 1;
				float value = info.Axes[i];
				axisValue.Text = value.ToString("0.000");
				hbox.AddChild(axisValue);
				axesLabels.Add(axisValue);
			}
			for (int i = 0; i < info.NumPovs; i++)
			{
				WidgetHBox hbox = new WidgetHBox(gui);
				hbox.SetSpace(5, 0);
				hbox.SetPadding(0, 0, i == 0 ? 16 : 5, 0);
				clearWidgets.Add(hbox);
				infoContainer.AddChild(hbox, Gui.ALIGN_LEFT);
				WidgetLabel nameLabel = new WidgetLabel(gui);
				nameLabel.Text = info.PovsName[i] + ":";
				nameLabel.FontSize = 16;
				nameLabel.FontOutline = 1;
				hbox.AddChild(nameLabel);
				WidgetLabel povValue = new WidgetLabel(gui, "NOT_PRESSED");
				povValue.FontSize = 16;
				povValue.FontOutline = 1;
				Input.JOYSTICK_POV value = info.Povs[i];
				povValue.Text = value.ToString();
				hbox.AddChild(povValue);
				povsLabels.Add(povValue);
			}
			effectData.Clear();
			foreach (var effectType in info.FfbData.AvailableFfbEffects)
			{
				var data = new FfbEffectData();
				data.EffectType = effectType;
				effectData.Add(effectType, data);
			}
			SetupFfbUi(info, infoContainer, joystickComponent);
		}
		private static void CreateFfbEffectParameterSlider(string name, float initialValue, float min, float max,
			Widget parent,
			Action<float> onChange)
		{
			var label = new WidgetLabel(name) { FontSize = 16, FontOutline = 1, Width = 75 };
			parent.AddChild(label, Gui.ALIGN_LEFT);
			var slider = new WidgetSlider { Width = 200, MinValue = 0, MaxValue = 1000 };
			parent.AddChild(slider, Gui.ALIGN_CENTER);
			var valueLabel = new WidgetLabel(initialValue.ToString("F2")) { FontSize = 16, FontOutline = 1, Width = 75 };
			parent.AddChild(valueLabel, Gui.ALIGN_LEFT);
			slider.Value = (int)MapRange(initialValue, min, max, 0, 1000);
			valueLabel.Text = initialValue.ToString("F2");
			slider.EventChanged.Connect(() =>
			{
				var newValue = MapRange(slider.Value, 0, 1000, min, max);
				valueLabel.Text = newValue.ToString("F2");
				onChange(newValue);
			});
			onChange(initialValue);
			return;
			static float MapRange(float input, float inMin, float inMax, float outMin, float outMax)
			{
				if (Math.Abs(inMin - inMax) < MathLib.EPSILON)
					return outMin;
				float lowerBound = Math.Min(inMin, inMax);
				float upperBound = Math.Max(inMin, inMax);
				input = Math.Clamp(input, lowerBound, upperBound);
				float normalized = (input - inMin) / (inMax - inMin);
				return normalized * (outMax - outMin) + outMin;
			}
		}
		private void CreateFfbEffectUi(string name, InputJoystickComponent joystickComponent, Widget parent,
			JoystickInfo joystickInfo, FfbEffectType effectType, bool needMagnitude, bool needFrequency,
			bool needDuration)
		{
			var group = new WidgetGroupBox(name) { FontSize = 16, FontOutline = 1 };
			group.SetSpace(30, 0);
			parent.AddChild(group);
			var grid = new WidgetGridBox(3) { FontSize = 16, FontOutline = 1 };
			grid.SetSpace(30, 10);
			group.AddChild(grid);
			var playButton = new WidgetButton("Play Effect") { Toggleable = true };
			CreateFfbEffectParameterSlider("Force", 0, 0, 1, grid, value =>
			{
				effectData[effectType].Force = value;
				if (playButton.Toggled)
				{
					joystickComponent.PlayFfbEffect(effectData[effectType], joystickInfo.Number);
				}
			});
			if (needMagnitude)
			{
				CreateFfbEffectParameterSlider("Magnitude", 0, -1, 1, grid, value =>
				{
					effectData[effectType].Magnitude = value;
					if (playButton.Toggled)
					{
						joystickComponent.PlayFfbEffect(effectData[effectType], joystickInfo.Number);
					}
					if (needDuration)
						rampEffectButtonController =
							new FfbDurationEffectButtonController(effectData[effectType], playButton);
				});
			}
			if (needFrequency)
			{
				CreateFfbEffectParameterSlider("Frequency", 0, 0, 10, grid, value =>
				{
					effectData[effectType].Frequency = value;
					if (playButton.Toggled)
					{
						joystickComponent.PlayFfbEffect(effectData[effectType], joystickInfo.Number);
					}
					if (needDuration)
						rampEffectButtonController =
							new FfbDurationEffectButtonController(effectData[effectType], playButton);
				});
			}
			if (needDuration)
			{
				CreateFfbEffectParameterSlider("Duration", 0, 0, 10, grid, value =>
				{
					effectData[effectType].Duration = (ulong)(value * 1_000_000);
					rampEffectButtonController =
						new FfbDurationEffectButtonController(effectData[effectType], playButton);
					if (playButton.Toggled)
					{
						joystickComponent.PlayFfbEffect(effectData[effectType], joystickInfo.Number);
					}
				});
			}
			group.AddChild(playButton, Gui.ALIGN_EXPAND);
			playButton.EventClicked.Connect(() =>
			{
				if (playButton.Toggled)
				{
					joystickComponent.PlayFfbEffect(effectData[effectType], joystickInfo.Number);
					effectData[effectType].IsPlaying = true;
				}
				else if (!playButton.Toggled)
				{
					joystickComponent.StopFfbEffect(effectType, joystickInfo.Joystick);
					effectData[effectType].IsPlaying = false;
				}
				if (needDuration)
					rampEffectButtonController =
						new FfbDurationEffectButtonController(effectData[effectType], playButton);
			});
		}
		private void SetupFfbUi(JoystickInfo joystickInfo, Widget parent, InputJoystickComponent joystickComponent)
		{
			var scroll = new WidgetScrollBox() { Width = 325, Height = 200 };
			scroll.HScrollEnabled = false;
			parent.AddChild(scroll, Gui.ALIGN_EXPAND);
			foreach (var effectType in joystickInfo.FfbData.AvailableFfbEffects)
			{
				switch (effectType)
				{
					case FfbEffectType.JOYSTICK_FORCE_FEEDBACK_CONSTANT:
						CreateFfbEffectUi("Constant", joystickComponent, scroll, joystickInfo, effectType, true, false,
							false);
						break;
					case FfbEffectType.JOYSTICK_FORCE_FEEDBACK_RAMP:
						CreateFfbEffectUi("Ramp", joystickComponent, scroll, joystickInfo, effectType, true, false,
							true);
						break;
					case FfbEffectType.JOYSTICK_FORCE_FEEDBACK_SINEWAVE:
						CreateFfbEffectUi("Sine Wave", joystickComponent, scroll, joystickInfo, effectType, true, true,
							false);
						break;
					case FfbEffectType.JOYSTICK_FORCE_FEEDBACK_SQUAREWAVE:
						CreateFfbEffectUi("Square Wave", joystickComponent, scroll, joystickInfo, effectType, true,
							true, false);
						break;
					case FfbEffectType.JOYSTICK_FORCE_FEEDBACK_TRIANGLEWAVE:
						CreateFfbEffectUi("Triangle Wave", joystickComponent, scroll, joystickInfo, effectType, true,
							true, false);
						break;
					case FfbEffectType.JOYSTICK_FORCE_FEEDBACK_SAWTOOTHUPWAVE:
						CreateFfbEffectUi("Saw Tooth Up", joystickComponent, scroll, joystickInfo, effectType, true,
							true, false);
						break;
					case FfbEffectType.JOYSTICK_FORCE_FEEDBACK_SAWTOOTHDOWNWAVE:
						CreateFfbEffectUi("Saw Tooth Down", joystickComponent, scroll, joystickInfo, effectType, true,
							true, false);
						break;
					case FfbEffectType.JOYSTICK_FORCE_FEEDBACK_SPRING:
						CreateFfbEffectUi("Spring", joystickComponent, scroll, joystickInfo, effectType, false, false,
							false);
						break;
					case FfbEffectType.JOYSTICK_FORCE_FEEDBACK_FRICTION:
						CreateFfbEffectUi("Friction", joystickComponent, scroll, joystickInfo, effectType, false, false,
							false);
						break;
					case FfbEffectType.JOYSTICK_FORCE_FEEDBACK_DAMPER:
						CreateFfbEffectUi("Damper", joystickComponent, scroll, joystickInfo, effectType, false, false,
							false);
						break;
					case FfbEffectType.JOYSTICK_FORCE_FEEDBACK_INERTIA:
						CreateFfbEffectUi("Inertia", joystickComponent, scroll, joystickInfo, effectType, false, false,
							false);
						break;
					case FfbEffectType.NUM_JOYSTICK_FORCE_FEEDBACKS:
					default:
						throw new ArgumentOutOfRangeException(nameof(joystickInfo));
				}
			}
		}
	}
	private List<InfoWidgets> infoWidgets = null;
	private WidgetHBox mainHBox = null;
	private WidgetVBox backgroundVBox = null;
	private WidgetLabel errorMessageLabel = null;
	private WidgetVBox generalMessageInfo = null;
	private WidgetLabel countActiveJoysticksLabel = null;
	private Widget countActiveJoysticksContainer = null;
	private WidgetHBox joysticksInfoBox = null;
	/// <summary>
	/// Initializes the joystick UI elements and sets up event handlers.
	/// </summary>
	[MethodInit(Order = 1)]
	private void Init()
	{
		Input.MouseHandle = Input.MOUSE_HANDLE.USER;
		Gui gui = Gui.GetCurrent();
		mainHBox = new WidgetHBox();
		backgroundVBox = new WidgetVBox { FontSize = 16, FontOutline = 1, Background = 1, Width = 400 };
		backgroundVBox.SetPadding(8, 8, 8, 8);
		mainHBox.AddChild(backgroundVBox, Gui.ALIGN_CENTER);
		var commonInfoContainer = new WidgetHBox();
		backgroundVBox.AddChild(commonInfoContainer);
		errorMessageLabel = new WidgetLabel("ACTIVE JOYSTICKS NOT FOUND") { FontSize = 16, FontOutline = 1, Hidden = true };
		commonInfoContainer.AddChild(errorMessageLabel);
		generalMessageInfo = new WidgetVBox();
		commonInfoContainer.AddChild(generalMessageInfo);
		countActiveJoysticksLabel = new WidgetLabel() { FontSize = 16, FontOutline = 1 };
		countActiveJoysticksContainer = WrapWidget("Active Joysticks:", countActiveJoysticksLabel, generalMessageInfo);
		joysticksInfoBox = new WidgetHBox();
		joysticksInfoBox.SetSpace(30, 0);
		backgroundVBox.AddChild(joysticksInfoBox);
		gui.AddChild(mainHBox, Gui.ALIGN_CENTER);
		backgroundVBox.BackgroundColor = new vec4(0.0f, 0.0f, 0.0f, 0.5f);
		if (joystickComponent == null || joystickComponent.JoysticksCount == 0)
			return;
		errorMessageLabel.Hidden = true;
		generalMessageInfo.Hidden = false;
		countActiveJoysticksLabel.Text = joystickComponent.JoysticksCount.ToString();
		infoWidgets = new List<InfoWidgets>();
		for (int i = 0; i < joystickComponent.JoysticksCount; i++)
		{
			CreateInterface(joystickComponent.JoysticksInfo[i]);
		}
	}
	/// <summary>
	/// Updates joystick status and UI elements each frame.
	/// </summary>
	private void Update()
	{
		if (joystickComponent == null || joystickComponent.JoysticksCount == 0 || !HasActiveJoysticks())
		{
			if (infoWidgets != null)
			{
				foreach (var infoWidget in infoWidgets)
				{
					infoWidget.infoContainer.Hidden = true;
				}
			}
			countActiveJoysticksContainer.Hidden = true;
			errorMessageLabel.Hidden = false;
			return;
		}
		foreach (var infoWidget in infoWidgets)
		{
			infoWidget.infoContainer.Hidden = false;
		}
		countActiveJoysticksContainer.Hidden = false;
		errorMessageLabel.Hidden = true;
		infoWidgets ??= [];
		for (int i = 0; i < joystickComponent.JoysticksCount; i++)
		{
			JoystickInfo info = joystickComponent.JoysticksInfo[i];
			if (i >= infoWidgets.Count)
			{
				CreateInterface(info);
				continue;
			}
			infoWidgets[i].Set(info, joystickComponent);
		}
		foreach (var info in infoWidgets)
		{
			info.Update();
		}
		return;
		bool HasActiveJoysticks()
		{
			return joystickComponent.JoysticksInfo.Any(info => info != null && info.Joystick.IsAvailable);
		}
	}
	/// <summary>
	/// Cleans up UI elements and removes event listeners on shutdown.
	/// </summary>
	private void Shutdown()
	{
		Input.MouseHandle = Input.MOUSE_HANDLE.GRAB;
		if (mainHBox)
		{
			if (infoWidgets != null)
			{
				foreach (InfoWidgets iw in infoWidgets)
				{
					mainHBox.RemoveChild(iw.infoContainer);
				}
			}
			Gui.GetCurrent().RemoveChild(mainHBox);
		}
	}
	/// <summary>
	/// Handles filter value changes and invokes corresponding events.
	/// </summary>
	private void ChangeFilter(Widget sender)
	{
		int number = 0;
		bool res = int.TryParse(sender.Data, out number);
		WidgetSlider slider = sender as WidgetSlider;
		if (res && slider)
			filterChanged?.Invoke(number, slider.Value / 100.0f);
	}
	private void CreateInterface(JoystickInfo joystickInfo)
	{
		Gui gui = Gui.GetCurrent();
		InfoWidgets info = new InfoWidgets();
		infoWidgets.Add(info);
		joysticksInfoBox.AddChild(info.infoVBox);
		info.filterSlider.EventChanged.Connect(ChangeFilter);
		info.Set(joystickInfo, joystickComponent);
	}
}
```
---
## InputKeyboardAndMouse.cs
```csharp
﻿using System;
using System.Collections.Generic;
using System.Numerics;
using System.Text;
using Unigine;
[Component(PropertyGuid = "a10fa015460f72701fc0630f2d22f5226532f219")]
public class InputKeyboardAndMouse : Component
{
	public string LastInputSymbol { get; private set; } = null;
	public Input.KEY? LastKeyDown { get; private set; } = null;
	public Input.KEY? LastKeyPressed { get; private set; } = null;
	public Input.KEY? LastKeyUp { get; private set; } = null;
	public Input.MOUSE_BUTTON? LastMouseButtonDown { get; private set; } = null;
	public Input.MOUSE_BUTTON? LastMouseButtonPressed { get; private set; } = null;
	public Input.MOUSE_BUTTON? LastMouseButtonUp { get; private set; } = null;
	public ivec2? MouseCoord { get; private set; } = null;
	public ivec2? LastMouseCoordDelta { get; private set; } = null;
	public vec2? LastMouseDelta { get; private set; } = null;
	public int? LastMouseWheel { get; private set; } = null;
	public int? LastMouseWheelHorizontal { get; private set; } = null;
	public Input.MOUSE_HANDLE? MouseHandle { get; private set; } = null;
	private Array keys = null;
	private Array mouseButtons = null;
	private HashSet<Input.KEY> pressedKeys = null;
	private HashSet<Input.MOUSE_BUTTON> pressedMouseButtons = null;
	private void Init()
	{
		keys = Enum.GetValues(typeof(Input.KEY));
		mouseButtons = Enum.GetValues(typeof(Input.MOUSE_BUTTON));
		pressedKeys = new HashSet<Input.KEY>();
		pressedMouseButtons = new HashSet<Input.MOUSE_BUTTON>();
		InputKeyboardAndMouseUI.mouseHandleChanged += OnMouseHandleChanged;
		Input.EventTextPress.Connect(OnTextPressed);
	}
	private void Update()
	{
		// update keyboards
		foreach (var key in keys)
		{
			Input.KEY currentKey = (Input.KEY)key;
			// skip any ANY_* keys
			if (currentKey >= Input.KEY.ANY_SHIFT)
				continue;
			if (Input.IsKeyDown(currentKey))
				LastKeyDown = currentKey;
			if (Input.IsKeyPressed(currentKey) && !pressedKeys.Contains(currentKey))
			{
				LastKeyPressed = currentKey;
				pressedKeys.Add(currentKey);
			}
			if (Input.IsKeyUp(currentKey))
			{
				LastKeyUp = currentKey;
				pressedKeys.Remove(currentKey);
			}
		}
		// update mouse buttons
		foreach (var button in mouseButtons)
		{
			Input.MOUSE_BUTTON currentButton = (Input.MOUSE_BUTTON)button;
			if (currentButton == Input.MOUSE_BUTTON.MOUSE_NUM_BUTTONS)
				continue;
			if (Input.IsMouseButtonDown(currentButton))
				LastMouseButtonDown = currentButton;
			if (Input.IsMouseButtonPressed(currentButton) && !pressedMouseButtons.Contains(currentButton))
			{
				LastMouseButtonPressed = currentButton;
				pressedMouseButtons.Add(currentButton);
			}
			if (Input.IsMouseButtonUp(currentButton))
			{
				LastMouseButtonUp = currentButton;
				pressedMouseButtons.Remove(currentButton);
			}
		}
		// update mouse coords and deltas
		MouseCoord = Input.MousePosition;
		if (Input.MouseDeltaPosition.Length2 > 0)
			LastMouseCoordDelta = Input.MouseDeltaPosition;
		if (Input.MouseDeltaPosition.Length2 > 0)
			LastMouseDelta = Input.MouseDeltaPosition;
		if (Input.MouseWheel != 0)
			LastMouseWheel = Input.MouseWheel;
		if (Input.MouseWheelHorizontal != 0)
			LastMouseWheelHorizontal = Input.MouseWheelHorizontal;
		MouseHandle = Input.MouseHandle;
	}
	private void Shutdown()
	{
		InputKeyboardAndMouseUI.mouseHandleChanged -= OnMouseHandleChanged;
	}
	private void OnMouseHandleChanged(Input.MOUSE_HANDLE handle)
	{
		Input.MouseHandle = handle;
	}
	private void OnTextPressed(uint unicode)
	{
		byte[] bytes = BitConverter.GetBytes(unicode);
		LastInputSymbol = Encoding.Unicode.GetString(bytes);
	}
}
```
---
## InputKeyboardAndMouseUI.cs
```csharp
﻿using System;
using Unigine;
[Component(PropertyGuid = "fce2c9811dd594ec64a98b5b827448fe5239dc45")]
public class InputKeyboardAndMouseUI : Component
{
	public InputKeyboardAndMouse inputComponent = null;
	[ParameterFile(Filter = ".ui")]
	public string uiFile = "";
	static public event Action<Input.MOUSE_HANDLE> mouseHandleChanged;
	private UserInterface ui = null;
	private WidgetHBox mainHBox = null;
	private WidgetVBox backgroundVBox = null;
	private WidgetLabel lastInputSymbolLabel = null;
	private WidgetLabel lastKeyDownLabel = null;
	private WidgetLabel lastKeyPressedLabel = null;
	private WidgetLabel lastKeyUpLabel = null;
	private WidgetLabel lastMouseDownLabel = null;
	private WidgetLabel lastMousePressedLabel = null;
	private WidgetLabel lastMouseUpLabel = null;
	private WidgetLabel mouseCoordXLabel = null;
	private WidgetLabel mouseCoordYLabel = null;
	private WidgetLabel lastMouseCoordDeltaXLabel = null;
	private WidgetLabel lastMouseCoordDeltaYLabel = null;
	private WidgetLabel lastMouseDeltaXLabel = null;
	private WidgetLabel lastMouseDeltaYLabel = null;
	private WidgetLabel lastWheelVerticalLabel = null;
	private WidgetLabel lastWheelHorizontalLabel = null;
	private WidgetLabel mouseHandleLabel = null;
	private WidgetComboBox mouseHandleCombobox = null;
	[MethodInit(Order = -1)]
	private void Init()
	{
		Input.MouseHandle = Input.MOUSE_HANDLE.USER;
		Gui gui = Gui.GetCurrent();
		ui = new UserInterface(gui, uiFile);
		InitWidget(nameof(mainHBox), out mainHBox);
		InitWidget(nameof(backgroundVBox), out backgroundVBox);
		InitWidget(nameof(lastInputSymbolLabel), out lastInputSymbolLabel);
		InitWidget(nameof(lastKeyDownLabel), out lastKeyDownLabel);
		InitWidget(nameof(lastKeyPressedLabel), out lastKeyPressedLabel);
		InitWidget(nameof(lastKeyUpLabel), out lastKeyUpLabel);
		InitWidget(nameof(lastMouseDownLabel), out lastMouseDownLabel);
		InitWidget(nameof(lastMousePressedLabel), out lastMousePressedLabel);
		InitWidget(nameof(lastMouseUpLabel), out lastMouseUpLabel);
		InitWidget(nameof(mouseCoordXLabel), out mouseCoordXLabel);
		InitWidget(nameof(mouseCoordYLabel), out mouseCoordYLabel);
		InitWidget(nameof(lastMouseCoordDeltaXLabel), out lastMouseCoordDeltaXLabel);
		InitWidget(nameof(lastMouseCoordDeltaYLabel), out lastMouseCoordDeltaYLabel);
		InitWidget(nameof(lastMouseDeltaXLabel), out lastMouseDeltaXLabel);
		InitWidget(nameof(lastMouseDeltaYLabel), out lastMouseDeltaYLabel);
		InitWidget(nameof(lastWheelVerticalLabel), out lastWheelVerticalLabel);
		InitWidget(nameof(lastWheelHorizontalLabel), out lastWheelHorizontalLabel);
		InitWidget(nameof(mouseHandleLabel), out mouseHandleLabel);
		InitWidget(nameof(mouseHandleCombobox), out mouseHandleCombobox);
		gui.AddChild(mainHBox);
		backgroundVBox.BackgroundColor = new vec4(0.0f, 0.0f, 0.0f, 0.5f);
		mouseHandleCombobox.CurrentItem = (int)Input.MOUSE_HANDLE.USER;
		mouseHandleCombobox.EventChanged.Connect(() =>
		{
			mouseHandleChanged?.Invoke((Input.MOUSE_HANDLE)mouseHandleCombobox.CurrentItem);
		});
	}
	private void Update()
	{
		if (inputComponent == null)
			return;
		lastInputSymbolLabel.Text = inputComponent.LastInputSymbol;
		lastKeyDownLabel.Text = inputComponent.LastKeyDown.ToString();
		lastKeyPressedLabel.Text = inputComponent.LastKeyPressed.ToString();
		lastKeyUpLabel.Text = inputComponent.LastKeyUp.ToString();
		lastMouseDownLabel.Text = inputComponent.LastMouseButtonDown.ToString();
		lastMousePressedLabel.Text = inputComponent.LastMouseButtonPressed.ToString();
		lastMouseUpLabel.Text = inputComponent.LastMouseButtonUp.ToString();
		mouseCoordXLabel.Text = inputComponent.MouseCoord?.x.ToString();
		mouseCoordYLabel.Text = inputComponent.MouseCoord?.y.ToString();
		lastMouseCoordDeltaXLabel.Text = inputComponent.LastMouseCoordDelta?.x.ToString();
		lastMouseCoordDeltaYLabel.Text = inputComponent.LastMouseCoordDelta?.y.ToString();
		lastMouseDeltaXLabel.Text = inputComponent.LastMouseDelta?.x.ToString("0.000");
		lastMouseDeltaYLabel.Text = inputComponent.LastMouseDelta?.y.ToString("0.000");
		lastWheelVerticalLabel.Text = inputComponent.LastMouseWheel?.ToString();
		lastWheelHorizontalLabel.Text = inputComponent.LastMouseWheelHorizontal?.ToString();
		mouseHandleLabel.Text = inputComponent.MouseHandle?.ToString();
		if (inputComponent.MouseHandle == Input.MOUSE_HANDLE.GRAB && Input.MouseGrab == true)
			mouseHandleLabel.Text += " (press ESC to show cursor)";
	}
	private void Shutdown()
	{
		Input.MouseHandle = Input.MOUSE_HANDLE.GRAB;
		if (mainHBox)
			Gui.GetCurrent().RemoveChild(mainHBox);
	}
	private void InitWidget<T>(string name, out T widget) where T : Widget
	{
		widget = null;
		int id = ui.FindWidget(name);
		if (id != -1)
			widget = ui.GetWidget(id) as T;
	}
}
```
---
## InputTouches.cs
```csharp
﻿using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "587166faf7a2ca1167defc3cfd1daa6b8d4a3cce")]
public class InputTouches : Component
{
	public ivec2[] TouchesPositions { get; private set; } = new ivec2[Input.NUM_TOUCHES];
	private void Init()
	{
		for (int i = 0; i < Input.NUM_TOUCHES; i++)
			TouchesPositions[i] = -ivec2.ONE;
	}
	private void Update()
	{
		for (int i = 0; i < Input.NUM_TOUCHES; i++)
		{
			if (Input.IsTouchPressed(i))
				TouchesPositions[i] = Input.GetTouchPosition(i);
			if (Input.IsTouchUp(i))
				TouchesPositions[i] = -ivec2.ONE;
		}
	}
}
```
---
## InputTouchesUI.cs
```csharp
﻿using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "8166e8007718bb5f6f9f51324fc1fee9799abb34")]
public class InputTouchesUI : Component
{
	public InputTouches touchesComponent = null;
	[ParameterFile(Filter = ".ui")]
	public string uiFile = "";
	private UserInterface ui = null;
	private WidgetHBox mainHBox = null;
	private WidgetVBox backgroundVBox = null;
	private WidgetLabel maxTouchesCountLabel = null;
	private WidgetLabel touchesCountLabel = null;
	private List<WidgetLabel> xValues = null;
	private List<WidgetLabel> yValues = null;
	private WidgetCanvas canvas = null;
	private List<int> polygonsId = null;
	private List<int> textsId = null;
	private int polygonsHalsSize = 25;
	private vec4[] colors =
	{
		new vec4(0.0f, 0.0f, 0.0f, 1.0f),
		new vec4(0.0f, 0.0f, 1.0f, 1.0f),
		new vec4(0.0f, 1.0f, 0.0f, 1.0f),
		new vec4(0.0f, 1.0f, 1.0f, 1.0f),
		new vec4(1.0f, 0.0f, 0.0f, 1.0f),
		new vec4(1.0f, 0.0f, 1.0f, 1.0f),
		new vec4(1.0f, 1.0f, 0.0f, 1.0f),
		new vec4(1.0f, 1.0f, 1.0f, 1.0f),
		new vec4(0.5f, 0.0f, 0.0f, 1.0f),
		new vec4(0.0f, 0.5f, 1.0f, 1.0f),
		new vec4(0.0f, 1.0f, 0.5f, 1.0f),
		new vec4(0.5f, 1.0f, 1.0f, 1.0f),
		new vec4(1.0f, 0.5f, 0.0f, 1.0f),
		new vec4(1.0f, 0.5f, 1.0f, 1.0f),
		new vec4(1.0f, 1.0f, 0.5f, 1.0f),
		new vec4(1.0f, 0.5f, 1.0f, 1.0f),
		new vec4(0.0f, 0.0f, 0.0f, 0.0f)
	};
	private int lastAppWidth = 0;
	private int lastAppHeight = 0;
	[MethodInit(Order = -1)]
	private void Init()
	{
		EngineWindow main_window = WindowManager.MainWindow;
		if (!main_window)
		{
			Engine.Quit();
			return;
		}
		ivec2 main_size = main_window.Size;
		Input.MouseHandle = Input.MOUSE_HANDLE.USER;
		Gui gui = Gui.GetCurrent();
		ui = new UserInterface(gui, uiFile);
		InitWidget(nameof(mainHBox), out mainHBox);
		InitWidget(nameof(backgroundVBox), out backgroundVBox);
		InitWidget(nameof(canvas), out canvas);
		InitWidget(nameof(maxTouchesCountLabel), out maxTouchesCountLabel);
		InitWidget(nameof(touchesCountLabel), out touchesCountLabel);
		maxTouchesCountLabel.Text = Input.NUM_TOUCHES.ToString();
		xValues = new List<WidgetLabel>();
		for (int i = 0; i < Input.NUM_TOUCHES; i++)
		{
			WidgetLabel xLabel = null;
			InitWidget($"xTouch{i}Label", out xLabel);
			xValues.Add(xLabel);
		}
		yValues = new List<WidgetLabel>();
		for (int i = 0; i < Input.NUM_TOUCHES; i++)
		{
			WidgetLabel yLabel = null;
			InitWidget($"yTouch{i}Label", out yLabel);
			yValues.Add(yLabel);
		}
		gui.AddChild(mainHBox, Gui.ALIGN_CENTER);
		backgroundVBox.BackgroundColor = new vec4(0.0f, 0.0f, 0.0f, 0.5f);
		gui.AddChild(canvas);
		canvas.Width = main_size.x;
		canvas.Height = main_size.y;
		// create polygons and texts for visualization of touches
		polygonsId = new List<int>(Input.NUM_TOUCHES);
		textsId = new List<int>(Input.NUM_TOUCHES);
		for (int i = 0; i < Input.NUM_TOUCHES; i++)
		{
			int id = canvas.AddPolygon();
			polygonsId.Add(id);
			canvas.AddPolygonPoint(id, new vec3(-polygonsHalsSize, -polygonsHalsSize, 0));
			canvas.AddPolygonPoint(id, new vec3(-polygonsHalsSize, polygonsHalsSize, 0));
			canvas.AddPolygonPoint(id, new vec3(polygonsHalsSize, polygonsHalsSize, 0));
			canvas.AddPolygonPoint(id, new vec3(polygonsHalsSize, -polygonsHalsSize, 0));
			canvas.AddPolygonIndex(id, 0);
			canvas.AddPolygonIndex(id, 1);
			canvas.AddPolygonIndex(id, 2);
			canvas.AddPolygonIndex(id, 2);
			canvas.AddPolygonIndex(id, 3);
			canvas.AddPolygonIndex(id, 0);
			canvas.SetPolygonColor(id, vec4.ZERO);
			id = canvas.AddText();
			textsId.Add(id);
			canvas.SetTextText(id, $"Touch {i}");
			canvas.SetTextSize(id, 16);
			canvas.SetTextPosition(id, new vec2(-30, -polygonsHalsSize * 2));
		}
	}
	private void Update()
	{
		EngineWindow mainWindow = WindowManager.MainWindow;
		if (!mainWindow)
		{
			Engine.Quit();
			return;
		}
		ivec2 mainSize = mainWindow.Size;
		ivec2 mainPosition = mainWindow.Position;
		// resize canvas
		int width = mainSize.x;
		int height = mainSize.y;
		if (width != lastAppWidth || height != lastAppHeight)
		{
			lastAppWidth = width;
			lastAppHeight = height;
			canvas.Width = lastAppWidth;
			canvas.Height = lastAppHeight;
		}
		// update polygons, texts and values of touches
		int touchesCount = 0;
		for (int i = 0; i < Input.NUM_TOUCHES; i++)
		{
			int x = touchesComponent.TouchesPositions[i].x;
			int y = touchesComponent.TouchesPositions[i].y;
			vec4 touchColor = vec4.ZERO;
			if (x != -1 || y != -1)
			{
				touchColor = colors[i % (colors.Length - 1)];
				touchesCount++;
			}
			xValues[i].Text = x.ToString();
			yValues[i].Text = y.ToString();
			canvas.SetPolygonTransform(polygonsId[i], MathLib.Translate(new vec3(x - mainPosition.x, y - mainPosition.y, 0)));
			canvas.SetPolygonColor(polygonsId[i], touchColor);
			canvas.SetTextTransform(textsId[i], MathLib.Translate(new vec3(x - mainPosition.x, y - mainPosition.y, 0)));
			canvas.SetTextColor(textsId[i], touchColor);
		}
		touchesCountLabel.Text = touchesCount.ToString();
	}
	private void Shutdown()
	{
		Input.MouseHandle = Input.MOUSE_HANDLE.GRAB;
		if (mainHBox)
			Gui.GetCurrent().RemoveChild(mainHBox);
		if (canvas)
			Gui.GetCurrent().RemoveChild(canvas);
	}
	private void InitWidget<T>(string name, out T widget) where T : Widget
	{
		widget = null;
		int id = ui.FindWidget(name);
		if (id != -1)
			widget = ui.GetWidget(id) as T;
	}
}
```
---
## Interface.cs
```csharp
﻿
public interface Interface
{
	public void DoSomething();
}
```
---
## InterfaceComponentImplementation.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "3ae25d97a71237e98e1e42efe9611bc8a3afd961")]
public class InterfaceComponentImplementation : Component, Interface
{
	public void DoSomething()
	{
		Log.MessageLine("InterfaceComponentImplementation::DoSomething()");
	}
}
```
---
## IntersectionTriggerComponent.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
using System.Collections.Generic;
[Component(PropertyGuid = "6bd4a7db685ab8df907188cb5ecf4bcf1054599f")]
public class IntersectionTriggerComponent : Component
{
	[ShowInEditor]
	private float boundSphereSize = 5.0f;
	[ShowInEditor]
	private float boundBoxSize = 5.0f;
	[ShowInEditor]
	private bool isSphere = false;
	[ShowInEditor]
	private bool debug = false;
	[ShowInEditor][ParameterMask]
	private int materialBallIntersectionMask = 0x7;
	public int MaterialBallIntersectionMask { get { return materialBallIntersectionMask; } }
	private List<Node> entered = new List<Node>();
	private List<Node> inside = new List<Node>();
	private WorldBoundBox boundBox;
	private WorldBoundSphere boundSphere;
	private EventInvoker<Node> eventEnter = new EventInvoker<Node>();
	public Event<Node> EventEnter { get { return eventEnter; } }
	private EventInvoker<Node> eventLeave = new EventInvoker<Node>();
	public Event<Node> EventLeave { get { return eventLeave; } }
	void Init()
	{
		Vec3 translation = node.WorldTransform.Translate;
		Vec3 coordinatesForBoundBoxMin = new Vec3(-boundBoxSize * 0.5f) + translation;
		Vec3 coordinatesForBoundBoxMax = new Vec3(boundBoxSize * 0.5f) + translation;
		boundBox.Set(new vec3(coordinatesForBoundBoxMin), new vec3(coordinatesForBoundBoxMax));
		boundSphere.Set(new vec3(translation), boundSphereSize * 0.5f);
	}
	void Update()
	{
		ReplaceBounds();
		if(debug)
		{
			VisualizeBounds();
		}
		GetInsideNodes();
		CheckEnter();
		CheckLeave();
	}
	private void CheckEnter()
	{
		foreach(Node obj in inside)
		{
			if(!entered.Contains(obj))
			{
				entered.Add(obj);
				eventEnter.Run(obj);
			}
		}
	}
	private void CheckLeave()
	{
		List<Node> toRemove = new List<Node>();
		foreach (Node obj in entered)
		{
			if (obj.IsDeleted)
				toRemove.Add(obj);
			if (!inside.Contains(obj))
			{
				toRemove.Add(obj);
				eventLeave.Run(obj);
			}
		}
		foreach(Node obj in toRemove)
		{
			entered.Remove(obj);
		}
	}
	private void ReplaceBounds()
	{
		Vec3 translation = node.WorldTransform.Translate;
		Vec3 coordinatesForBoundBoxMin = new Vec3(-boundBoxSize * 0.5f) + translation;
		Vec3 coordinatesForBoundBoxMax = new Vec3(boundBoxSize * 0.5f) + translation;
		boundBox.Set(new vec3(coordinatesForBoundBoxMin), new vec3(coordinatesForBoundBoxMax));
		boundSphere.Set(new vec3(translation), boundSphereSize * 0.5f);
	}
	private void VisualizeBounds()
	{
		if (isSphere)
			Visualizer.RenderSphere(boundSphereSize * 0.5f, node.WorldTransform, vec4.RED);
		else
			Visualizer.RenderBoundBox(new BoundBox(new vec3(boundBox.minimum), new vec3(boundBox.maximum)), Mat4.IDENTITY, vec4.RED);
	}
	private void GetInsideNodes()
	{
		if (isSphere)
			World.GetIntersection(boundSphere, inside);
		else
			World.GetIntersection(boundBox, inside);
	}
}
```
---
## JointCallbacks.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
#endif
[Component(PropertyGuid = "ec557ee1fa8be1a22fe1df67565afa155e001e85")]
public class JointCallbacks : Component
{
	public int bridge_sections = 14;
	// different materials for still joint and already broken sections
	public Material broken_materal;
	public Material joint_materal;
	// mesh used as a section
	[ParameterFile(Filter = ".mesh")]
	public string mesh_file = "";
	private List<Node> objects = new List<Node>();
	private EventConnections joint_connections = new EventConnections();
	private float space = 1.1f;
	void Init()
	{
		// parameters validation
		if (mesh_file.Length <= 0)
			Log.Error("JointCallbacks.Init(): Mesh File parameter is empty!\n");
		if (!broken_materal)
			Log.Error("JointCallbacks.Init(): Broken Matreial parameter is empty!\n");
		if (!joint_materal)
			Log.Error("JointCallbacks.Init(): Joint Matreial parameter is empty!\n");
		// general Physics settings
		Physics.FrozenLinearVelocity = 0.1f;
		Physics.FrozenAngularVelocity = 0.1f;
		Physics.NumIterations = 4;
		// create object from mesh file
		ObjectMeshStatic orig_object = new ObjectMeshStatic(mesh_file);
		// create weights to break bridge
		BodyRigid body = new BodyRigid(orig_object);
		ShapeBox shape = new ShapeBox(body, new vec3(1));
		shape.Density = 80.0f;
		for (int i = 0; i < 3; i++)
		{
			ObjectMeshStatic mesh = orig_object.Clone() as ObjectMeshStatic;
			mesh.WorldTransform = MathLib.Translate(new Vec3(3.0f * (i - 1), 0.0f, 12.0f));
			objects.Add(mesh);
		}
		// remove body from object
		orig_object.Body = null;
		body.DeleteLater();
		// create bridge via boxes and joints
		orig_object.SetMaterial(joint_materal, "*");
		Body b0 = null, b1;
		for (int i = 0; i < bridge_sections; i++)
		{
			ObjectMeshStatic mesh = orig_object.Clone() as ObjectMeshStatic;
			float pos = space * (i - (bridge_sections - 1) / 2.0f);
			mesh.WorldTransform = MathLib.Translate(new Vec3(pos, 0.0f, 8.0f));
			// set first and last bridge section as BodyDummy so they won't fall
			if (i == 0 || i == bridge_sections - 1)
				b1 = new BodyDummy(mesh);
			else
				b1 = new BodyRigid(mesh);
			shape = new ShapeBox(b1, new vec3(1));
			objects.Add(mesh);
			// create joint between two neighbour boxes
			if (b0 != null)
			{
				JointHinge joint = new JointHinge(b0, b1, new Vec3(pos - space, 0.0f, 8.0f), new vec3(1.0f, 0.0f, 0.0f));
				joint.AngularDamping = 8.0f;
				joint.NumIterations = 2;
				joint.LinearRestitution = 0.02f;
				joint.AngularRestitution = 0.02f;
				joint.MaxForce = 1000.0f;
				joint.MaxTorque = 16000.0f;
				// subscribind to joint breaking event
				joint.EventBroken.Connect(joint_connections, j => {
					// change material of broken parts
					joint.Body0.Object.SetMaterial(broken_materal, "*");
					joint.Body1.Object.SetMaterial(broken_materal, "*");
				});
			}
			b0 = b1;
		}
		orig_object.DeleteLater();
	}
	void Shutdown()
	{
		// remove all connections
		joint_connections.DisconnectAll();
		objects.Clear();
	}
}
```
---
## Lamp.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "6e8e87bcdefeca656e6329c53069243985a5d2f2")]
public class Lamp : Toggleable
{
	[ParameterColor]
	public vec4 emission_color = vec4.WHITE;
	protected override bool On()
	{
		Log.MessageLine("Lamp::On()");
		return SetEmissionColor(emission_color);
	}
	protected override bool Off()
	{
		Log.MessageLine("Lamp::Off()");
		return SetEmissionColor(vec4.ZERO);
	}
	private bool SetEmissionColor(vec4 emission_color)
	{
		Object obj = (Object)node;
		if (obj == null)
			return false;
		for (var surface = 0; surface < obj.NumSurfaces; surface += 1)
			obj.SetMaterialParameterFloat4("emission_color", emission_color, surface);
		return true;
	}
	private void Init()
	{
		SetEmissionColor(Toggled ? emission_color : vec4.ZERO);
	}
}
```
---
## LaserRayIntersection.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "d86ba149dc4a3a14ab4bde63e18fb0662af98c8d")]
public class LaserRayIntersection : Component
{
	public Node laserRay = null;
	public Node laserHit = null;
	public float laserDistance = 25.0f;
	// use mask to separate objects for intersection
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.INTERSECTION)]
	public int mask = 1;
	private WorldIntersectionNormal intersection = null;
	private vec3 laserRayScale = vec3.ONE;
	private SampleDescriptionWindow sampleDescriptionWindow;
	private void Init()
	{
		sampleDescriptionWindow = new SampleDescriptionWindow();
		sampleDescriptionWindow.createWindow();
		// check parts of laser
		if (!laserRay || !laserHit)
			return;
		// create an intersection object to obtain the necessary information
		// about the intersection result
		intersection = new WorldIntersectionNormal();
		// save the source laser ray scale for changing length after intersection
		laserRayScale = laserRay.WorldScale;
	}
	private void Update()
	{
		// check parts of laser
		if (!laserRay || !laserHit)
			return;
		// get points to detect intersection based on the direction of the laser ray
		Vec3 firstPoint = laserRay.WorldPosition;
		Vec3 secondPoint = firstPoint + laserRay.GetWorldDirection(MathLib.AXIS.Y) * laserDistance;
		var status = "Hit Object:";
		// try to get intersection object
		Unigine.Object hitObject = World.GetIntersection(firstPoint, secondPoint, mask, intersection);
		if (hitObject)
		{
			// show object name
			status += $" {hitObject.Name}";
			// set current laser ray length
			float length = (float)(intersection.Point - laserRay.WorldPosition).Length;
			laserRayScale.y = length;
			laserRay.WorldScale = laserRayScale;
			// activate laserHit if it was hidden earlier
			if (!laserHit.Enabled)
				laserHit.Enabled = true;
			// update laser hit transform based on intersection information
			laserHit.WorldPosition = intersection.Point;
			laserHit.SetWorldDirection(intersection.Normal, vec3.UP, MathLib.AXIS.Y);
		}
		else
		{
			status += " none";
			// set default ray length
			laserRayScale.y = laserDistance;
			laserRay.WorldScale = laserRayScale;
			// hide hit point
			laserHit.Enabled = false;
		}
		sampleDescriptionWindow.setStatus(status);
	}
	private void Shutdown()
	{
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## LifetimeController.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "70b43839418d8444ddf5c2d37424ea16b3b99a7d")]
public class LifetimeController : Component
{
	public float lifetime = 1.0f;
	private float starttime = 0.0f;
	private void Init()
	{
		starttime = Game.Time;
	}
	private void Update()
	{
		if (Game.Time - starttime > lifetime)
			node.DeleteLater();
	}
	private void Shutdown()
	{
		node.DeleteLater();
	}
}
```
---
## Manipulators.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "7115b15ea4639d29e8e01b9227d75746985e637b")]
public class Manipulators : Component
{
	[ShowInEditor][ParameterMask]
	private int intersectionMask = 1;
	[ShowInEditor]
	private bool transformParent = true;
	private bool xAxisRotation = true;
	public bool XAxisRotation
	{
		get { return xAxisRotation; }
		set
		{
			xAxisRotation = value;
			if (xAxisRotation)
				objectRotator.Mask = objectRotator.Mask | WidgetManipulator.MASK_X;
			else
				objectRotator.Mask = objectRotator.Mask & ~(WidgetManipulator.MASK_X);
		}
	}
	private bool yAxisRotation = true;
	public bool YAxisRotation
	{
		get { return yAxisRotation; }
		set
		{
			yAxisRotation = value;
			if (yAxisRotation)
				objectRotator.Mask = objectRotator.Mask | WidgetManipulator.MASK_Y;
			else
				objectRotator.Mask = objectRotator.Mask & ~(WidgetManipulator.MASK_Y);
		}
	}
	private bool zAxisRotation = true;
	public bool ZAxisRotation
	{
		get { return zAxisRotation; }
		set
		{
			zAxisRotation = value;
			if (zAxisRotation)
				objectRotator.Mask = objectRotator.Mask | WidgetManipulator.MASK_Z;
			else
				objectRotator.Mask = objectRotator.Mask & ~(WidgetManipulator.MASK_Z);
		}
	}
	private bool xAxisTranslation = true;
	public bool XAxisTranslation
	{
		get { return xAxisTranslation; }
		set
		{
			xAxisTranslation = value;
			if (xAxisTranslation)
				objectTranslator.Mask = objectTranslator.Mask | WidgetManipulator.MASK_X;
			else
				objectTranslator.Mask = objectTranslator.Mask & ~(WidgetManipulator.MASK_X);
		}
	}
	private bool yAxisTranslation = true;
	public bool YAxisTranslation
	{
		get { return yAxisTranslation; }
		set
		{
			yAxisTranslation = value;
			if (yAxisTranslation)
				objectTranslator.Mask = objectTranslator.Mask | WidgetManipulator.MASK_Y;
			else
				objectTranslator.Mask = objectTranslator.Mask & ~(WidgetManipulator.MASK_Y);
		}
	}
	private bool zAxisTranslation = true;
	public bool ZAxisTranslation
	{
		get { return zAxisTranslation; }
		set
		{
			zAxisTranslation = value;
			if (zAxisTranslation)
				objectTranslator.Mask = objectTranslator.Mask | WidgetManipulator.MASK_Z;
			else
				objectTranslator.Mask = objectTranslator.Mask & ~(WidgetManipulator.MASK_Z);
		}
	}
	private bool xAxisScale = true;
	public bool XAxisScale
	{
		get { return xAxisScale; }
		set
		{
			xAxisScale = value;
			if (xAxisScale)
				objectScaler.Mask = objectScaler.Mask | WidgetManipulator.MASK_X;
			else
				objectScaler.Mask = objectScaler.Mask & ~(WidgetManipulator.MASK_X);
		}
	}
	private bool yAxisScale = true;
	public bool YAxisScale
	{
		get { return yAxisScale; }
		set
		{
			yAxisScale = value;
			if (yAxisScale)
				objectScaler.Mask = objectScaler.Mask | WidgetManipulator.MASK_Y;
			else
				objectScaler.Mask = objectScaler.Mask & ~(WidgetManipulator.MASK_Y);
		}
	}
	private bool zAxisScale = true;
	public bool ZAxisScale
	{
		get { return zAxisScale; }
		set
		{
			zAxisScale = value;
			if (zAxisScale)
				objectScaler.Mask = objectScaler.Mask | WidgetManipulator.MASK_Z;
			else
				objectScaler.Mask = objectScaler.Mask & ~(WidgetManipulator.MASK_Z);
		}
	}
	private bool localBasis = false;
	public bool LocalBasis
	{
		get { return localBasis; }
		set
		{
			localBasis = value;
			SetManipulatorsBasis();
		}
	}
	public bool Active { get { return obj != null; } }
	private WidgetManipulator currentObjectManipulator;
	private WidgetManipulatorTranslator objectTranslator;
	private WidgetManipulatorRotator objectRotator;
	private WidgetManipulatorScaler objectScaler;
	private Unigine.Object obj;
	private Gui gui;
	void Init()
	{
		gui = WindowManager.MainWindow.Gui;
		objectTranslator = new WidgetManipulatorTranslator(gui);
		objectRotator = new WidgetManipulatorRotator(gui);
		objectScaler = new WidgetManipulatorScaler(gui);
		gui.AddChild(objectTranslator);
		gui.AddChild(objectRotator);
		gui.AddChild(objectScaler);
		objectTranslator.Hidden = true;
		objectRotator.Hidden = true;
		objectScaler.Hidden = true;
		currentObjectManipulator = objectTranslator;
		objectTranslator.EventChanged.Connect(ApplyTransform);
		objectRotator.EventChanged.Connect(ApplyTransform);
		objectScaler.EventChanged.Connect(ApplyTransform);
		Player player = Game.Player;
		player.Controlled = false;
	}
	void Update()
	{
		Player player = Game.Player;
		// if mouse is grabbed, we enable player control making hotkeys for switching manipulators unavailable
		if (Input.MouseGrab)
			player.Controlled = true;
		// reset projection and view matrices for correct rendering of the widget
		if (player)
		{
			objectTranslator.Projection = player.Projection;
			objectRotator.Projection = player.Projection;
			objectScaler.Projection = player.Projection;
			objectTranslator.Modelview = player.Camera.Modelview;
			objectRotator.Modelview = player.Camera.Modelview;
			objectScaler.Modelview = player.Camera.Modelview;
		}
		// trying to get an object to manipulate
		if (Input.IsMouseButtonDown(Input.MOUSE_BUTTON.RIGHT) && !Input.MouseGrab)
		{
			obj = GetNodeUnderCursor();
			if (obj)
			{
				SwitchManipulator(currentObjectManipulator);
			}
			else
			{
				Unselect();
			}
		}
		// if some object is selected and it is not a ComponentLogic node
		if (obj)
		{
			// reset manipulator's transform matrix, after moving the object
			if (Input.IsMouseButtonUp(Input.MOUSE_BUTTON.LEFT))
			{
				SwitchManipulator(currentObjectManipulator);
			}
			// if mouse is not grabbed, disable player control and enable hotkeys to switch manipulators
			if (!Input.MouseGrab)
			{
				player.Controlled = false;
				if (Input.IsKeyDown(Input.KEY.W))
					SwitchManipulator(objectTranslator);
				if (Input.IsKeyDown(Input.KEY.E))
					SwitchManipulator(objectRotator);
				if (Input.IsKeyDown(Input.KEY.R))
					SwitchManipulator(objectScaler);
			}
			// hotkey to focus on an object at focus_distance distance
			if (Input.IsKeyDown(Input.KEY.F))
			{
				vec3 inversePlayerViewDirection = -player.ViewDirection;
				WorldBoundSphere bs = new WorldBoundSphere();
				GetMeshBS(obj, ref bs);
				player.WorldPosition = bs.Center + new Vec3(inversePlayerViewDirection * ((float)bs.Radius * 2.0f));
			}
			// hotkeys to unselect an object
			if (Input.IsKeyDown(Input.KEY.U) || Input.IsKeyDown(Input.KEY.ESC)) // || Input::isMouseCursorHide())
			{
				Unselect();
			}
		}
	}
	void Shutdown()
	{
		objectTranslator.DeleteLater();
		objectRotator.DeleteLater();
		objectScaler.DeleteLater();
	}
	private void GetMeshBS(in Node n, ref WorldBoundSphere bs)
	{
		if (!n) return;
		if (n.IsObject)
			bs.Expand(node.WorldBoundSphere);
		if (node.Type == Node.TYPE.NODE_REFERENCE)
			GetMeshBS((n as NodeReference).Reference, ref bs);
		for (int i = 0; i < n.NumChildren; i++)
			GetMeshBS(node.GetChild(i), ref bs);
	}
	private void ApplyTransform()
	{
		// applying transformation of the widget to the object
		if (obj)
		{
			Node manipulateNode = obj;
			if (transformParent && manipulateNode.Parent)
				manipulateNode = manipulateNode.Parent;
			manipulateNode.WorldTransform = currentObjectManipulator.Transform;
		}
	}
	private Unigine.Object GetNodeUnderCursor()
	{
		Player player = Game.Player;
		ivec2 mouse = Input.MousePosition;
		// returns the first object intersected by the ray casted from the player in the forward direction at a distance of 10000 units
		return World.GetIntersection(player.WorldPosition, player.WorldPosition + new Vec3(player.GetDirectionFromMainWindow(mouse.x, mouse.y) * 10000), intersectionMask);
	}
	private void SwitchManipulator(WidgetManipulator currentManipulator)
	{
		if (obj)
		{
			SetManipulatorsBasis();
			currentObjectManipulator = currentManipulator;
			currentObjectManipulator.Hidden = false;
			Node manipulateNode = obj;
			if (transformParent && manipulateNode.Parent)
				manipulateNode = manipulateNode.Parent;
			currentObjectManipulator.Transform = manipulateNode.WorldTransform;
			// show only selected manipulator
			if (objectTranslator != currentObjectManipulator)
				objectTranslator.Hidden = true;
			if (objectRotator != currentObjectManipulator)
				objectRotator.Hidden = true;
			if (objectScaler != currentObjectManipulator)
				objectScaler.Hidden =true;
		}
	}
	private void Unselect()
	{
		// reset selection to the DummyNode(ComponentLogic node in the world)
		obj = null;
		// hide all manipulators
		objectTranslator.Hidden = true;
		objectRotator.Hidden = true;
		objectScaler.Hidden = true;
	}
	private void SetManipulatorsBasis()
	{
		if (obj)
		{
			if (localBasis)
			{
				// reset the basis of manipulators to object's local basis
				objectRotator.Basis = obj.WorldTransform;
				objectTranslator.Basis = obj.WorldTransform;
				objectScaler.Basis = obj.WorldTransform;
			}
			else
			{
				// resets the basis of manipulators to world basis
				objectRotator.Basis = Mat4.IDENTITY;
				objectTranslator.Basis = Mat4.IDENTITY;
				objectScaler.Basis = Mat4.IDENTITY;
			}
		}
	}
}
```
---
## ManipulatorsSample.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "7c1daf3defe81fe33439f5d9f68d3b0091e560b2")]
public class ManipulatorsSample : Component
{
	private Manipulators component;
	private WidgetCheckBox xAxisRotationCheckBox;
	private WidgetCheckBox yAxisRotationCheckBox;
	private WidgetCheckBox zAxisRotationCheckBox;
	private WidgetCheckBox xAxisTranslationCheckBox;
	private WidgetCheckBox yAxisTranslationCheckBox;
	private WidgetCheckBox zAxisTranslationCheckBox;
	private WidgetCheckBox xAxisScaleCheckBox;
	private WidgetCheckBox yAxisScaleCheckBox;
	private WidgetCheckBox zAxisScaleCheckBox;
	private WidgetButton localBasisButton;
	private WidgetButton worldBasisButton;
	private bool localBasis = false;
	private Input.MOUSE_HANDLE previousHandle;
	private SampleDescriptionWindow sampleDescriptionWindow = new SampleDescriptionWindow();
	void Init()
	{
		previousHandle = Input.MouseHandle;
		Input.MouseHandle = Input.MOUSE_HANDLE.GRAB;
		component = GetComponent<Manipulators>(node);
		if(component == null )
		{
			Log.ErrorLine("ManipulatorsSample::init: cannot find WidgetManipulators component!");
		}
		sampleDescriptionWindow.createWindow();
		WidgetGroupBox parameters = sampleDescriptionWindow.getParameterGroupBox();
		var hBox = new WidgetHBox(10);
		parameters.AddChild(hBox, Gui.ALIGN_LEFT);
		var label = new WidgetLabel("Translation:");
		label.FontWrap = 1;
		label.FontRich = 1;
		label.Width = 100;
		hBox.AddChild(label, Gui.ALIGN_LEFT);
		xAxisTranslationCheckBox = new WidgetCheckBox("X");
		xAxisTranslationCheckBox.Checked = true;
		xAxisTranslationCheckBox.EventChanged.Connect(() =>
		{
			component.XAxisTranslation = xAxisTranslationCheckBox.Checked;
		});
		hBox.AddChild(xAxisTranslationCheckBox, Gui.ALIGN_LEFT);
		yAxisTranslationCheckBox = new WidgetCheckBox("Y");
		yAxisTranslationCheckBox.Checked = true;
		yAxisTranslationCheckBox.EventChanged.Connect(() =>
		{
			component.YAxisTranslation = yAxisTranslationCheckBox.Checked;
		});
		hBox.AddChild(yAxisTranslationCheckBox, Gui.ALIGN_LEFT);
		zAxisTranslationCheckBox = new WidgetCheckBox("Z");
		zAxisTranslationCheckBox.Checked = true;
		zAxisTranslationCheckBox.EventChanged.Connect(() =>
		{
			component.ZAxisTranslation = zAxisTranslationCheckBox.Checked;
		});
		hBox.AddChild(zAxisTranslationCheckBox, Gui.ALIGN_LEFT);
		hBox = new WidgetHBox(10);
		parameters.AddChild(hBox, Gui.ALIGN_LEFT);
		label = new WidgetLabel("Rotation:");
		label.FontWrap = 1;
		label.FontRich = 1;
		label.Width = 100;
		hBox.AddChild(label, Gui.ALIGN_LEFT);
		xAxisRotationCheckBox = new WidgetCheckBox("X");
		xAxisRotationCheckBox.Checked = true;
		xAxisRotationCheckBox.EventChanged.Connect(() =>
		{
			component.XAxisRotation = xAxisRotationCheckBox.Checked;
		});
		hBox.AddChild(xAxisRotationCheckBox, Gui.ALIGN_LEFT);
		yAxisRotationCheckBox = new WidgetCheckBox("Y");
		yAxisRotationCheckBox.Checked = true;
		yAxisRotationCheckBox.EventChanged.Connect(() =>
		{
			component.YAxisRotation = yAxisRotationCheckBox.Checked;
		});
		hBox.AddChild(yAxisRotationCheckBox, Gui.ALIGN_LEFT);
		zAxisRotationCheckBox = new WidgetCheckBox("Z");
		zAxisRotationCheckBox.Checked = true;
		zAxisRotationCheckBox.EventChanged.Connect(() =>
		{
			component.ZAxisRotation = zAxisRotationCheckBox.Checked;
		});
		hBox.AddChild(zAxisRotationCheckBox, Gui.ALIGN_LEFT);
		hBox = new WidgetHBox(10);
		parameters.AddChild(hBox, Gui.ALIGN_LEFT);
		label = new WidgetLabel("Scale:");
		label.FontWrap = 1;
		label.FontRich = 1;
		label.Width = 100;
		hBox.AddChild(label, Gui.ALIGN_LEFT);
		xAxisScaleCheckBox = new WidgetCheckBox("X");
		xAxisScaleCheckBox.Checked = true;
		xAxisScaleCheckBox.EventChanged.Connect(() =>
		{
			component.XAxisScale = xAxisScaleCheckBox.Checked;
		});
		hBox.AddChild(xAxisScaleCheckBox, Gui.ALIGN_LEFT);
		yAxisScaleCheckBox = new WidgetCheckBox("Y");
		yAxisScaleCheckBox.Checked = true;
		yAxisScaleCheckBox.EventChanged.Connect(() =>
		{
			component.YAxisScale = yAxisScaleCheckBox.Checked;
		});
		hBox.AddChild(yAxisScaleCheckBox, Gui.ALIGN_LEFT);
		zAxisScaleCheckBox = new WidgetCheckBox("Z");
		zAxisScaleCheckBox.Checked = true;
		zAxisScaleCheckBox.EventChanged.Connect(() =>
		{
			component.ZAxisScale = zAxisScaleCheckBox.Checked;
		});
		hBox.AddChild(zAxisScaleCheckBox, Gui.ALIGN_LEFT);
		hBox = new WidgetHBox(10);
		parameters.AddChild(hBox, Gui.ALIGN_LEFT);
		worldBasisButton = new WidgetButton("World");
		hBox.AddChild(worldBasisButton, Gui.ALIGN_LEFT);
		worldBasisButton.Toggleable = true;
		worldBasisButton.Toggled = true;
		worldBasisButton.EventChanged.Connect(() =>
		{
			localBasis = !worldBasisButton.Toggled;
			localBasisButton.Toggled = localBasis;
			component.LocalBasis = localBasis;
		});
		localBasisButton = new WidgetButton("Local");
		hBox.AddChild(localBasisButton, Gui.ALIGN_LEFT);
		localBasisButton.Toggleable = true;
		localBasisButton.Toggled = false;
		localBasisButton.EventChanged.Connect(() =>
		{
			localBasis = localBasisButton.Toggled;
			worldBasisButton.Toggled = !localBasis;
			component.LocalBasis = localBasis;
		});
	}
	void Shutdown()
	{
		Input.MouseHandle = previousHandle;
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## MaterialsAlbedoColor.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "204cfc442f7edad38fab22160bda727d0e5991b8")]
public class MaterialsAlbedoColor : Component
{
	[ParameterColor]
	public vec4 beginColor = vec4.ZERO;
	[ParameterColor]
	public vec4 endColor = vec4.ONE;
	public float time = 5.0f;
	private Material material = null;
	private float currentTime = 0.0f;
	private float timeSign = 1.0f;
	private void Init()
	{
		// try cast node to object
		Unigine.Object obj = node as Unigine.Object;
		if (!obj)
			return;
		// try get material from 0 surface
		if (obj.NumSurfaces != 0)
			material = obj.GetMaterial(0);
	}
	private void Update()
	{
		if (!material)
			return;
		// update current time
		currentTime += Game.IFps * timeSign;
		if (currentTime < 0 || time < currentTime)
			timeSign = -timeSign;
		// change albedo color of material
		material.SetParameterFloat4("albedo_color", MathLib.Lerp(beginColor, endColor, MathLib.Saturate(currentTime / time)));
	}
}
```
---
## MaterialsAlbedoTexture.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "c92fe3f5cb9e24d02e71b1ef45713682319883f9")]
public class MaterialsAlbedoTexture : Component
{
	[ParameterFile]
	public string firstTextureImage;
	[ParameterFile]
	public string secondTextureImage;
	public float time = 5.0f;
	private Material material = null;
	private float currentTime = 0.0f;
	private float timeSign = 1.0f;
	private Texture firstTexture = null;
	private Texture secondTexture = null;
	private void Init()
	{
		// try cast node to object
		Unigine.Object obj = node as Unigine.Object;
		if (!obj)
			return;
		// try get material from 0 surface
		if (obj.NumSurfaces != 0)
			material = obj.GetMaterial(0);
		if (!material)
			return;
		// create first texture
		firstTexture = new Texture();
		firstTexture.Load(firstTextureImage);
		// create second texture
		secondTexture = new Texture();
		secondTexture.Load(secondTextureImage);
		// apply first texture to material
		material.SetTexture("albedo", firstTexture);
	}
	private void Update()
	{
		if (!material)
			return;
		// update current time
		currentTime += Game.IFps * timeSign;
		if (currentTime < 0 || time < currentTime)
		{
			// change albedo texture
			if (timeSign > 0)
				material.SetTexture("albedo", secondTexture);
			else
				material.SetTexture("albedo", firstTexture);
			timeSign = -timeSign;
		}
	}
}
```
---
## MaterialsCastWorldShadow.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "9b2c8cbb9c079ade503444a5d68d2dc0d994c61f")]
public class MaterialsCastWorldShadow : Component
{
	public float time = 5.0f;
	private Material material = null;
	private float currentTime = 0.0f;
	private float timeSign = 1.0f;
	private void Init()
	{
		// try cast node to object
		Unigine.Object obj = node as Unigine.Object;
		if (!obj)
			return;
		// try get material from 0 surface
		if (obj.NumSurfaces != 0)
			material = obj.GetMaterial(0);
	}
	private void Update()
	{
		if (!material)
			return;
		// update current time
		currentTime += Game.IFps * timeSign;
		if (currentTime < 0 || time < currentTime)
		{
			// change world cast shadow
			material.CastWorldShadow = !material.CastWorldShadow;
			timeSign = -timeSign;
		}
	}
}
```
---
## MaterialsEmission.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "cf8e6ff8767f72f742d533eaa5cbd24165e786ac")]
public class MaterialsEmission : Component
{
	public float time = 5.0f;
	private Material material = null;
	private float currentTime = 0.0f;
	private float timeSign = 1.0f;
	private void Init()
	{
		// try cast node to object
		Unigine.Object obj = node as Unigine.Object;
		if (!obj)
			return;
		// try get material from 0 surface
		if (obj.NumSurfaces != 0)
			material = obj.GetMaterial(0);
	}
	private void Update()
	{
		if (!material)
			return;
		// update current time
		currentTime += Game.IFps * timeSign;
		if (currentTime < 0 || time < currentTime)
		{
			// change emission state
			material.SetState("emission", 1 - material.GetState("emission"));
			timeSign = -timeSign;
		}
	}
}
```
---
## MaterialsMetalness.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "1d252ff437419ac34b788146b4e567edf9a2a868")]
public class MaterialsMetalness : Component
{
	public float beginMetalness = 0.0f;
	public float endMetalness = 1.0f;
	public float time = 5.0f;
	private Material material = null;
	private float currentTime = 0.0f;
	private float timeSign = 1.0f;
	private void Init()
	{
		// try cast node to object
		Unigine.Object obj = node as Unigine.Object;
		if (!obj)
			return;
		// try get material from 0 surface
		if (obj.NumSurfaces != 0)
			material = obj.GetMaterial(0);
	}
	private void Update()
	{
		if (!material)
			return;
		// update current time
		currentTime += Game.IFps * timeSign;
		if (currentTime < 0 || time < currentTime)
			timeSign = -timeSign;
		// change metalness of material
		material.SetParameterFloat("metalness", MathLib.Lerp(beginMetalness, endMetalness, MathLib.Saturate(currentTime / time)));
	}
}
```
---
## MathTriggerComponent.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "33a82db84a2489447af346085b2b1c2e128d4d4c")]
public class MathTriggerComponent : Component
{
	[ShowInEditor]
	private float boundSphereSize = 5.0f;
	[ShowInEditor]
	private float boundBoxSize = 5.0f;
	[ShowInEditor]
	private bool isSphere = false;
	[ShowInEditor]
	private bool debug = false;
	private List<Node> objects = new List<Node>();
	public int NumObjects { get { return objects.Count; } }
	private List<Node> entered = new List<Node>();
	private WorldBoundBox boundBox;
	private WorldBoundSphere boundSphere;
	private EventInvoker<Node> eventEnter = new EventInvoker<Node>();
	public Event<Node> EventEnter {  get { return eventEnter; } }
	private EventInvoker<Node> eventLeave = new EventInvoker<Node>();
	public Event<Node> EventLeave {  get { return eventLeave; } }
	void Init()
	{
		Vec3 translation = node.WorldTransform.Translate;
		Vec3 coordinatesForBoundBoxMin = new Vec3(-boundBoxSize * 0.5f) + translation;
		Vec3 coordinatesForBoundBoxMax = new Vec3(boundBoxSize * 0.5f) + translation;
		boundBox.Set(new vec3(coordinatesForBoundBoxMin), new vec3(coordinatesForBoundBoxMax));
		boundSphere.Set(new vec3(translation), boundSphereSize * 0.5f);
	}
	void Update()
	{
		ReplaceBounds();
		if(debug)
		{
			VisualizeBounds();
		}
		CheckEnter();
		CheckLeave();
	}
	public void AddObject(Node obj)
	{
		objects.Add(obj);
	}
	public void AddObjects(ICollection<Node> inputObjects)
	{
		objects.AddRange(inputObjects);
	}
	public Node GetObjectByIndex(int index)
	{
		return objects[index];
	}
	public void RemoveObject(Node obj)
	{
		objects.Remove(obj);
	}
	public void RemoveObjectByIndex(int index)
	{
		objects.RemoveAt(index);
	}
	public void ClearObjects()
	{
		objects.Clear();
	}
	private void CheckEnter()
	{
		foreach(Node obj in objects)
		{
			if (entered.Contains(obj))
				continue;
			bool isInside = isSphere ? CheckSphere(obj) : CheckBox(obj);
			if(isInside)
			{
				entered.Add(obj);
				eventEnter.Run(obj);
			}
		}
	}
	private void CheckLeave()
	{
		foreach (Node obj in objects)
		{
			if (!entered.Contains(obj))
				continue;
			if(obj.IsDeleted)
			{
				entered.Remove(obj);
				continue;
			}
			bool isInside = isSphere ? CheckSphere(obj) : CheckBox(obj);
			if (!isInside)
			{
				entered.Remove(obj);
				eventLeave.Run(obj);
			}
		}
	}
	private void ReplaceBounds()
	{
		Vec3 translation = node.WorldTransform.Translate;
		Vec3 coordinatesForBoundBoxMin = new Vec3(-boundBoxSize * 0.5f) + translation;
		Vec3 coordinatesForBoundBoxMax = new Vec3(boundBoxSize * 0.5f) + translation;
		boundBox.Set(new vec3(coordinatesForBoundBoxMin), new vec3(coordinatesForBoundBoxMax));
		boundSphere.Set(new vec3(translation), boundSphereSize * 0.5f);
	}
	private void VisualizeBounds()
	{
		if (isSphere)
			Visualizer.RenderSphere(boundSphereSize * 0.5f, node.WorldTransform, vec4.RED);
		else
			Visualizer.RenderBoundBox(new BoundBox(new vec3(boundBox.minimum), new vec3(boundBox.maximum)), Mat4.IDENTITY, vec4.RED);
	}
	private bool CheckSphere(Node obj)
	{
		return boundSphere.InsideValid(new vec3(obj.Transform.Translate));
	}
	private bool CheckBox(Node obj)
	{
		return boundBox.InsideValid(new vec3(obj.Transform.Translate));
	}
}
```
---
## MeshDigger.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
#endif
#endregion
using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
using System.Reflection;
[Component(PropertyGuid = "7e58ea61a9b1c73a78316091d4bcb7343928ed47")]
public class MeshDigger : Component
{
	[Parameter(Tooltip = "Number of marching cubes along one side of the field")]
	public int fieldSize = 64;
	[Parameter(Tooltip = "Marching Cube edge length")]
	public float marchingCubeSize = 0.2f;
	public float diggingRadius = 2.0f;
	[Parameter(Title = "Material", Tooltip = "Ground material")]
	public Material mat;
	private AsyncMarchingCubes marchingCubes;
	private Unigine.Object groundObject;
	private Mat4 groundITransform;
	private WorldIntersection intersection = new();
	private SampleDescriptionWindow samplesDescriptionWindow = new();
	void Init()
	{
		// check the minimum values of field_size and marching_cube_size
		if (fieldSize < 4)
			fieldSize = 4;
		marchingCubeSize = MathLib.Clamp(marchingCubeSize, 0.05f, 1.0f);
		// check if the ground material exists and set mesh_height parameter to ensure correct texture clamping
		if (mat == null)
			Log.Error("MeshDigger.Init(): set Ground Material!\n");
		else
			mat.SetParameterFloat("mesh_height", fieldSize * marchingCubeSize);
		marchingCubes = new AsyncMarchingCubes(fieldSize, marchingCubeSize);
		marchingCubes.setMaterial(mat);
		// get the object created in the AsyncMarchingCubes() method to check for intersections
		groundObject = marchingCubes.getObject();
		groundITransform = groundObject.IWorldTransform;
		Visualizer.Enabled = true;
		samplesDescriptionWindow.createWindow();
		samplesDescriptionWindow.addFloatParameter("Digging Radius", "Radius in marching cubes units",
			diggingRadius, 1.0f, 30.0f, (float value) => { diggingRadius = value; });
	}
	void Update()
	{
		marchingCubes.Update();
		var w = Gui.GetCurrent().GetUnderCursorWidget();
		if (w && w.Type != Widget.TYPE.WIDGET_ENGINE)
			return;
		// perform an intersection check at the current mouse cursor position
		ivec2 mouse = Input.MousePosition;
		Vec3 p0 = Game.Player.WorldPosition;
		Vec3 p1 = p0 + new Vec3(Game.Player.GetDirectionFromMainWindow(mouse.x, mouse.y)) * 100.0f;
		// check for an intersection with the ground object
		Unigine.Object obj = World.GetIntersection(p0, p1, ~0, intersection);
		if (obj && obj == groundObject)
		{
			// try digging the mesh when the left mouse button is clicked
			if (Input.IsMouseButtonDown(Input.MOUSE_BUTTON.LEFT))
			{
				// convert the intersection point coordinates from world space to Marching Cubes coordinates
				var pos = groundITransform * intersection.Point;
				pos = pos / marchingCubeSize;
				AsyncMarchingCubes.BrushSphere sphere = new AsyncMarchingCubes.BrushSphere();
				sphere.pos = new vec3(pos);
				sphere.radius = diggingRadius;
				sphere.k = Input.IsKeyPressed(Input.KEY.ANY_CTRL) ? 1 : -1;
				marchingCubes.addBrush(sphere);
			}
			Visualizer.RenderSphere(diggingRadius * marchingCubeSize,
				MathLib.Translate(intersection.Point), vec4.BLUE);
		}
	}
	void Shutdown()
	{
		marchingCubes = null;
		groundObject = null;
		Visualizer.Enabled = false;
		samplesDescriptionWindow.shutdown();
	}
}
class AsyncMarchingCubes
{
	public AsyncMarchingCubes(int num_cubes, float cube_edge = 0.2f) { Create(num_cubes, cube_edge); }
	~AsyncMarchingCubes() { Destroy(); }
	public void addBrush(BrushSphere a) { actions.Add(a); }
	public Unigine.Object getObject() { return objectMesh; }
	public void setMaterial(Material mat) { material = mat; }
	public class BrushSphere
	{
		public vec3 pos = new();
		public float radius = 6;
		public float k = 2;
	};
	private ObjectMeshStatic objectMesh = new();
	private Unigine.Image field = new();
	private float[] fieldData;
	private int size = 0;
	private int size2 = 0;
	private float cubeEdgeLength = 0.2f;
	private List<BrushSphere> actions = [];
	private List<BrushSphere> asyncActions = [];
	private Material material;
	// flag indicating whether the object is already deleted
	private bool isDeleted = false;
	private static vec3 cell_0 = new(0.0f, 0.0f, 0.0f);
	private static vec3 cell_1 = new(1.0f, 0.0f, 0.0f);
	private static vec3 cell_2 = new(1.0f, 1.0f, 0.0f);
	private static vec3 cell_3 = new(0.0f, 1.0f, 0.0f);
	private static vec3 cell_4 = new(0.0f, 0.0f, 1.0f);
	private static vec3 cell_5 = new(1.0f, 0.0f, 1.0f);
	private static vec3 cell_6 = new(1.0f, 1.0f, 1.0f);
	private static vec3 cell_7 = new(0.0f, 1.0f, 1.0f);
	public void Create(int num_cubes, float cube_edge = 0.2f)
	{
		//destroy();
		isDeleted = false;
		size = num_cubes;
		size2 = size * size;
		cubeEdgeLength = cube_edge;
		objectMesh = new ObjectMeshStatic();
		objectMesh.SetMeshProceduralMode(ObjectMeshStatic.PROCEDURAL_MODE.DYNAMIC);
		// place the ground object at the center of the world
		float object_offset = num_cubes * cubeEdgeLength * 0.5f;
		objectMesh.WorldPosition = new Vec3(-object_offset);
		//field = new Unigine.Image();
		field.Create3D(size, size, size + 1, Image.FORMAT_R32F);
		fieldData = new float[size * size * (size + 1)];
		// generate voxel field data
		CreateField();
		Run(true);
	}
	public void Destroy()
	{
		isDeleted = true;
		size = 0;
		lock (objectMesh)
		{
			objectMesh = null;
			if (field.IsValidPtr)
				field.Clear();
			actions.Clear();
			asyncActions.Clear();
		}
	}
	public void Update() { Run(); }
	private void Run(bool force = false)
	{
		if (objectMesh.IsMeshProceduralActive || (actions.Count == 0 && !force))
			return;
		// pass all previously saved actions to apply in an async thread
		Swap(ref asyncActions, ref actions);
		actions.Clear();
		// update the mesh asynchronously in a separated thread
		objectMesh.RunGenerateMeshProceduralAsync(UpdateRam, UpdateRamDone, MeshRender.USAGE_DYNAMIC_ALL);
	}
	private void CreateField()
	{
		Image.Pixel pixel = new Image.Pixel();
		int id;
		for (int y = 0; y < size; y++)
		{
			for (int x = 0; x < size; x++)
			{
				// scale coordinates for noise generation
				float scale = 10.0f;
				vec2 p2D = new vec2(x, y) / size * scale;
				// noise generated height value in range [0.f, 1.f]
				float h = MathLib.Clamp(Game.GetNoise2(p2D, new vec2(size), 1), -0.5f, 0.5f);
				h = MathLib.Saturate(h * 0.5f + 0.5f);
				for (int z = 0; z < size; z++)
				{
					// normalized voxel's z-position
					float v = 1.0f - (float)z / size;
					id = z * size * size + y * size + x;
					// calculate voxel height value. where 1.0f corresponds to the top and -1.f - to the bottom
					float val = MathLib.Clamp(v * 6.0f - h, -1.0f, 1.0f);
					fieldData[id] = val;
					pixel.f.x = val;
					field.Set3D(x, y, z, pixel);
				}
				id = size * size * size + y * size + x;
				fieldData[id] = -1.0f;
				pixel.f.x = -1.0f;
				field.Set3D(x, y, size, pixel);
			}
		}
	}
	private void MarchingCubes(Mesh mesh)
	{
		if (mesh.NumSurfaces <= 0)
		{
			mesh.Clear();
			mesh.AddSurface("");
		}
		else
		{
			mesh.ClearSurface();
		}
		// reserve for new mesh data
		var vertices = new List<vec3>();
		var tangents = new List<quat>();
		var indices = new List<int>();
		vec4 yzwx(vec4 v) => new vec4(v.y, v.z, v.w, v.x);
		int toInt(bool val) => val ? 1 : 0;
		Dictionary<string, int> vertex_hash = [];
		float d = 1.0f / (float)size;
		vec3 o100 = new vec3(d, 0, 0);
		vec3 o010 = new vec3(0, d, 0);
		vec3 o001 = new vec3(0, 0, d);
		vec4 field_03 = new vec4();
		vec4 field_47 = new vec4();
		vec3[] vertex = new vec3[12];
		// create a mesh using the Marching Cubes algorithm
		for (int z = 0; z <= size - 1; z++)
		{
			for (int y = 0; y < size - 1; y++)
			{
				for (int x = 0; x < size - 1; x++)
				{
					int id = z * size2 + y * size + x;
					field_03.x = fieldData[id + 0];
					field_03.y = fieldData[id + 1];
					field_03.w = fieldData[id + size + 0];
					field_03.z = fieldData[id + size + 1];
					id += size2;
					field_47.x = fieldData[id + 0];
					field_47.y = fieldData[id + 1];
					field_47.w = fieldData[id + size + 0];
					field_47.z = fieldData[id + size + 1];
					int index = toInt(field_03.x > 0) + toInt(field_03.y > 0) * 2 + toInt(field_03.z > 0) * 4
						+ toInt(field_03.w > 0) * 8;
					index |= toInt(field_47.x > 0) * 16 + toInt(field_47.y > 0) * 32
						+ toInt(field_47.z > 0) * 64 + toInt(field_47.w > 0) * 128;
					if (index == 0 || index == 255)
						continue;
					short edges = marchingCubesEdges[index];
					if ((edges & 0x00f) != 0)
					{
						vec4 k = field_03 / (field_03 - yzwx(field_03));
						if ((edges & 0x001) != 0)
							vertex[0] = MathLib.Lerp(cell_0, cell_1, k.x);
						if ((edges & 0x002) != 0)
							vertex[1] = MathLib.Lerp(cell_1, cell_2, k.y);
						if ((edges & 0x004) != 0)
							vertex[2] = MathLib.Lerp(cell_2, cell_3, k.z);
						if ((edges & 0x008) != 0)
							vertex[3] = MathLib.Lerp(cell_3, cell_0, k.w);
					}
					if ((edges & 0x0f0) != 0)
					{
						vec4 k = field_47 / (field_47 - yzwx(field_47));
						if ((edges & 0x010) != 0)
							vertex[4] = MathLib.Lerp(cell_4, cell_5, k.x);
						if ((edges & 0x020) != 0)
							vertex[5] = MathLib.Lerp(cell_5, cell_6, k.y);
						if ((edges & 0x040) != 0)
							vertex[6] = MathLib.Lerp(cell_6, cell_7, k.z);
						if ((edges & 0x080) != 0)
							vertex[7] = MathLib.Lerp(cell_7, cell_4, k.w);
					}
					if ((edges & 0xf00) != 0)
					{
						vec4 k = field_03 / (field_03 - field_47);
						if ((edges & 0x100) != 0)
							vertex[8] = MathLib.Lerp(cell_0, cell_4, k.x);
						if ((edges & 0x200) != 0)
							vertex[9] = MathLib.Lerp(cell_1, cell_5, k.y);
						if ((edges & 0x400) != 0)
							vertex[10] = MathLib.Lerp(cell_2, cell_6, k.z);
						if ((edges & 0x800) != 0)
							vertex[11] = MathLib.Lerp(cell_3, cell_7, k.w);
					}
					vec3 xyz = new vec3(x, y, z);
					for (index *= 16; marchingCubesTriangles[index] != -1; index += 3)
					{
						var addVertex = (vec3 p) =>
						{
							string hash = p.ToString();
							if (vertex_hash.TryGetValue(hash, out var value))
							{
								indices.Add(value);
							}
							else
							{
								vec3 uvw = p * d;
								vec3 N = new vec3
								{
									x = field.Get3D(MathLib.Saturate(uvw - o100 * 0.5f)).f.r
										- field.Get3D(MathLib.Saturate(uvw + o100 * 0.5f)).f.r,
									y = field.Get3D(MathLib.Saturate(uvw - o010 * 0.5f)).f.r
										- field.Get3D(MathLib.Saturate(uvw + o010 * 0.5f)).f.r,
									z = field.Get3D(MathLib.Saturate(uvw - o001 * 0.5f)).f.r
										- field.Get3D(MathLib.Saturate(uvw + o001 * 0.5f)).f.r
								};
								N.Normalize();
								vec3 T = new vec3(N.z, 0.0f, -N.x);
								T = T - N * MathLib.Dot(T, N);
								T.Normalize();
								quat q = new quat();
								var B = MathLib.Cross(N, T);
								B.Normalize();
								q.Set(T, B, N);
								vertex_hash.Add(hash, vertices.Count);
								indices.Add(vertices.Count);
								vertices.Add(p * cubeEdgeLength);
								tangents.Add(q);
							}
						};
						addVertex(vertex[marchingCubesTriangles[index + 2]] + xyz);
						addVertex(vertex[marchingCubesTriangles[index + 1]] + xyz);
						addVertex(vertex[marchingCubesTriangles[index + 0]] + xyz);
					}
				}
			}
		}
		mesh.AddVertex(vertices.ToArray());
		mesh.AddTangents(tangents.ToArray());
		mesh.AddIndices(indices.ToArray());
		// create collision data for effective intersection and collision detection
		mesh.ClearCollisionData();
		mesh.CreateCollisionData();
		mesh.CreateBounds();
	}
	// apply actions to the mesh
	private void BrushField()
	{
		foreach (var action in asyncActions)
			AddSphere(action.pos, action.radius, action.k);
		asyncActions.Clear();
	}
	// add a sphere to the current mesh
	private void AddSphere(vec3 pos, float radius, float k)
	{
		Image.Pixel pixel = new Image.Pixel();
		// define the bounds of affected voxels
		ivec3 minPos = new ivec3(MathLib.Max(MathLib.Floor(pos - radius), vec3.ZERO));
		ivec3 maxPos = new ivec3(MathLib.Min(MathLib.Ceil(pos + radius), new vec3(size - 1)));
		// find affected voxels and adjust their height
		for (int z = minPos.z; z <= maxPos.z; z++)
		{
			for (int y = minPos.y; y <= maxPos.y; y++)
			{
				for (int x = minPos.x; x <= maxPos.x; x++)
				{
					vec3 voxCoord = new vec3(x, y, z);
					float dist = MathLib.Distance2(voxCoord, pos);
					if (dist > radius * radius)
						continue;
					int id = z * size * size + y * size + x;
					float value = (MathLib.Saturate(1.0f - MathLib.Sqrt(dist) / radius)) * k;
					float height = MathLib.Clamp(fieldData[id] + value, -1.0f, 1.0f);
					// update field value
					fieldData[id] = height;
					pixel.f.x = height;
					field.Set3D(x, y, z, pixel);
				}
			}
		}
	}
	// update the mesh
	private void UpdateRam(Mesh mesh)
	{
		if (isDeleted) return;
		lock (objectMesh)
		{
			BrushField();
			MarchingCubes(mesh);
		}
	}
	private void UpdateRamDone()
	{
		if (isDeleted)
			return;
		lock (objectMesh)
		{
			if (objectMesh && objectMesh.NumSurfaces > 0)
			{
				objectMesh.SetIntersection(true, 0);
				objectMesh.SetIntersectionMask(~0, 0);
				objectMesh.SetCollision(true, 0);
				objectMesh.SetCollisionMask(~0, 0);
				objectMesh.SetMaterial(material, 0);
			}
		}
	}
	private void Swap<T>(ref List<T> a, ref List<T> b)
	{
		var temp = a;
		a = b;
		b = temp;
	}
	// Marching Cubes algorithm constants
	static short[] marchingCubesEdges = {0x000, 0x109, 0x203, 0x30a, 0x406, 0x50f, 0x605,
												 0x70c, 0x80c, 0x905, 0xa0f, 0xb06, 0xc0a, 0xd03, 0xe09, 0xf00, 0x190, 0x099, 0x393, 0x29a,
												 0x596, 0x49f, 0x795, 0x69c, 0x99c, 0x895, 0xb9f, 0xa96, 0xd9a, 0xc93, 0xf99, 0xe90, 0x230,
												 0x339, 0x033, 0x13a, 0x636, 0x73f, 0x435, 0x53c, 0xa3c, 0xb35, 0x83f, 0x936, 0xe3a, 0xf33,
												 0xc39, 0xd30, 0x3a0, 0x2a9, 0x1a3, 0x0aa, 0x7a6, 0x6af, 0x5a5, 0x4ac, 0xbac, 0xaa5, 0x9af,
												 0x8a6, 0xfaa, 0xea3, 0xda9, 0xca0, 0x460, 0x569, 0x663, 0x76a, 0x066, 0x16f, 0x265, 0x36c,
												 0xc6c, 0xd65, 0xe6f, 0xf66, 0x86a, 0x963, 0xa69, 0xb60, 0x5f0, 0x4f9, 0x7f3, 0x6fa, 0x1f6,
												 0x0ff, 0x3f5, 0x2fc, 0xdfc, 0xcf5, 0xfff, 0xef6, 0x9fa, 0x8f3, 0xbf9, 0xaf0, 0x650, 0x759,
												 0x453, 0x55a, 0x256, 0x35f, 0x055, 0x15c, 0xe5c, 0xf55, 0xc5f, 0xd56, 0xa5a, 0xb53, 0x859,
												 0x950, 0x7c0, 0x6c9, 0x5c3, 0x4ca, 0x3c6, 0x2cf, 0x1c5, 0x0cc, 0xfcc, 0xec5, 0xdcf, 0xcc6,
												 0xbca, 0xac3, 0x9c9, 0x8c0, 0x8c0, 0x9c9, 0xac3, 0xbca, 0xcc6, 0xdcf, 0xec5, 0xfcc, 0x0cc,
												 0x1c5, 0x2cf, 0x3c6, 0x4ca, 0x5c3, 0x6c9, 0x7c0, 0x950, 0x859, 0xb53, 0xa5a, 0xd56, 0xc5f,
												 0xf55, 0xe5c, 0x15c, 0x055, 0x35f, 0x256, 0x55a, 0x453, 0x759, 0x650, 0xaf0, 0xbf9, 0x8f3,
												 0x9fa, 0xef6, 0xfff, 0xcf5, 0xdfc, 0x2fc, 0x3f5, 0x0ff, 0x1f6, 0x6fa, 0x7f3, 0x4f9, 0x5f0,
												 0xb60, 0xa69, 0x963, 0x86a, 0xf66, 0xe6f, 0xd65, 0xc6c, 0x36c, 0x265, 0x16f, 0x066, 0x76a,
												 0x663, 0x569, 0x460, 0xca0, 0xda9, 0xea3, 0xfaa, 0x8a6, 0x9af, 0xaa5, 0xbac, 0x4ac, 0x5a5,
												 0x6af, 0x7a6, 0x0aa, 0x1a3, 0x2a9, 0x3a0, 0xd30, 0xc39, 0xf33, 0xe3a, 0x936, 0x83f, 0xb35,
												 0xa3c, 0x53c, 0x435, 0x73f, 0x636, 0x13a, 0x033, 0x339, 0x230, 0xe90, 0xf99, 0xc93, 0xd9a,
												 0xa96, 0xb9f, 0x895, 0x99c, 0x69c, 0x795, 0x49f, 0x596, 0x29a, 0x393, 0x099, 0x190, 0xf00,
												 0xe09, 0xd03, 0xc0a, 0xb06, 0xa0f, 0x905, 0x80c, 0x70c, 0x605, 0x50f, 0x406, 0x30a, 0x203,
												 0x109, 0x000};
	static int[] marchingCubesTriangles = {
	-1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 8, 3, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 1, 9, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 1, 8, 3, 9, 8, 1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	1, 2, 10, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 8, 3, 1, 2, 10, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 9, 2, 10, 0, 2, 9, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 2, 8, 3, 2, 10, 8, 10, 9, 8, -1, -1, -1, -1, -1, -1, -1,
	3, 11, 2, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 11, 2, 8, 11, 0, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 1, 9, 0, 2, 3, 11, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 1, 11, 2, 1, 9, 11, 9, 8, 11, -1, -1, -1, -1, -1, -1, -1,
	3, 10, 1, 11, 10, 3, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 10, 1, 0, 8, 10, 8, 11, 10, -1, -1, -1, -1, -1, -1, -1, 3, 9, 0, 3, 11, 9, 11, 10, 9, -1, -1, -1, -1, -1, -1, -1, 9, 8, 10, 10, 8, 11, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	4, 7, 8, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 4, 3, 0, 7, 3, 4, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 1, 9, 8, 4, 7, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 4, 1, 9, 4, 7, 1, 7, 3, 1, -1, -1, -1, -1, -1, -1, -1,
	1, 2, 10, 8, 4, 7, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 3, 4, 7, 3, 0, 4, 1, 2, 10, -1, -1, -1, -1, -1, -1, -1, 9, 2, 10, 9, 0, 2, 8, 4, 7, -1, -1, -1, -1, -1, -1, -1, 2, 10, 9, 2, 9, 7, 2, 7, 3, 7, 9, 4, -1, -1, -1, -1,
	8, 4, 7, 3, 11, 2, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 11, 4, 7, 11, 2, 4, 2, 0, 4, -1, -1, -1, -1, -1, -1, -1, 9, 0, 1, 8, 4, 7, 2, 3, 11, -1, -1, -1, -1, -1, -1, -1, 4, 7, 11, 9, 4, 11, 9, 11, 2, 9, 2, 1, -1, -1, -1, -1,
	3, 10, 1, 3, 11, 10, 7, 8, 4, -1, -1, -1, -1, -1, -1, -1, 1, 11, 10, 1, 4, 11, 1, 0, 4, 7, 11, 4, -1, -1, -1, -1, 4, 7, 8, 9, 0, 11, 9, 11, 10, 11, 0, 3, -1, -1, -1, -1, 4, 7, 11, 4, 11, 9, 9, 11, 10, -1, -1, -1, -1, -1, -1, -1,
	9, 5, 4, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 9, 5, 4, 0, 8, 3, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 5, 4, 1, 5, 0, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 8, 5, 4, 8, 3, 5, 3, 1, 5, -1, -1, -1, -1, -1, -1, -1,
	1, 2, 10, 9, 5, 4, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 3, 0, 8, 1, 2, 10, 4, 9, 5, -1, -1, -1, -1, -1, -1, -1, 5, 2, 10, 5, 4, 2, 4, 0, 2, -1, -1, -1, -1, -1, -1, -1, 2, 10, 5, 3, 2, 5, 3, 5, 4, 3, 4, 8, -1, -1, -1, -1,
	9, 5, 4, 2, 3, 11, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 11, 2, 0, 8, 11, 4, 9, 5, -1, -1, -1, -1, -1, -1, -1, 0, 5, 4, 0, 1, 5, 2, 3, 11, -1, -1, -1, -1, -1, -1, -1, 2, 1, 5, 2, 5, 8, 2, 8, 11, 4, 8, 5, -1, -1, -1, -1,
	10, 3, 11, 10, 1, 3, 9, 5, 4, -1, -1, -1, -1, -1, -1, -1, 4, 9, 5, 0, 8, 1, 8, 10, 1, 8, 11, 10, -1, -1, -1, -1, 5, 4, 0, 5, 0, 11, 5, 11, 10, 11, 0, 3, -1, -1, -1, -1, 5, 4, 8, 5, 8, 10, 10, 8, 11, -1, -1, -1, -1, -1, -1, -1,
	9, 7, 8, 5, 7, 9, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 9, 3, 0, 9, 5, 3, 5, 7, 3, -1, -1, -1, -1, -1, -1, -1, 0, 7, 8, 0, 1, 7, 1, 5, 7, -1, -1, -1, -1, -1, -1, -1, 1, 5, 3, 3, 5, 7, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	9, 7, 8, 9, 5, 7, 10, 1, 2, -1, -1, -1, -1, -1, -1, -1, 10, 1, 2, 9, 5, 0, 5, 3, 0, 5, 7, 3, -1, -1, -1, -1, 8, 0, 2, 8, 2, 5, 8, 5, 7, 10, 5, 2, -1, -1, -1, -1, 2, 10, 5, 2, 5, 3, 3, 5, 7, -1, -1, -1, -1, -1, -1, -1,
	7, 9, 5, 7, 8, 9, 3, 11, 2, -1, -1, -1, -1, -1, -1, -1, 9, 5, 7, 9, 7, 2, 9, 2, 0, 2, 7, 11, -1, -1, -1, -1, 2, 3, 11, 0, 1, 8, 1, 7, 8, 1, 5, 7, -1, -1, -1, -1, 11, 2, 1, 11, 1, 7, 7, 1, 5, -1, -1, -1, -1, -1, -1, -1,
	9, 5, 8, 8, 5, 7, 10, 1, 3, 10, 3, 11, -1, -1, -1, -1, 5, 7, 0, 5, 0, 9, 7, 11, 0, 1, 0, 10, 11, 10, 0, -1, 11, 10, 0, 11, 0, 3, 10, 5, 0, 8, 0, 7, 5, 7, 0, -1, 11, 10, 5, 7, 11, 5, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	10, 6, 5, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 8, 3, 5, 10, 6, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 9, 0, 1, 5, 10, 6, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 1, 8, 3, 1, 9, 8, 5, 10, 6, -1, -1, -1, -1, -1, -1, -1,
	1, 6, 5, 2, 6, 1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 1, 6, 5, 1, 2, 6, 3, 0, 8, -1, -1, -1, -1, -1, -1, -1, 9, 6, 5, 9, 0, 6, 0, 2, 6, -1, -1, -1, -1, -1, -1, -1, 5, 9, 8, 5, 8, 2, 5, 2, 6, 3, 2, 8, -1, -1, -1, -1,
	2, 3, 11, 10, 6, 5, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 11, 0, 8, 11, 2, 0, 10, 6, 5, -1, -1, -1, -1, -1, -1, -1, 0, 1, 9, 2, 3, 11, 5, 10, 6, -1, -1, -1, -1, -1, -1, -1, 5, 10, 6, 1, 9, 2, 9, 11, 2, 9, 8, 11, -1, -1, -1, -1,
	6, 3, 11, 6, 5, 3, 5, 1, 3, -1, -1, -1, -1, -1, -1, -1, 0, 8, 11, 0, 11, 5, 0, 5, 1, 5, 11, 6, -1, -1, -1, -1, 3, 11, 6, 0, 3, 6, 0, 6, 5, 0, 5, 9, -1, -1, -1, -1, 6, 5, 9, 6, 9, 11, 11, 9, 8, -1, -1, -1, -1, -1, -1, -1,
	5, 10, 6, 4, 7, 8, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 4, 3, 0, 4, 7, 3, 6, 5, 10, -1, -1, -1, -1, -1, -1, -1, 1, 9, 0, 5, 10, 6, 8, 4, 7, -1, -1, -1, -1, -1, -1, -1, 10, 6, 5, 1, 9, 7, 1, 7, 3, 7, 9, 4, -1, -1, -1, -1,
	6, 1, 2, 6, 5, 1, 4, 7, 8, -1, -1, -1, -1, -1, -1, -1, 1, 2, 5, 5, 2, 6, 3, 0, 4, 3, 4, 7, -1, -1, -1, -1, 8, 4, 7, 9, 0, 5, 0, 6, 5, 0, 2, 6, -1, -1, -1, -1, 7, 3, 9, 7, 9, 4, 3, 2, 9, 5, 9, 6, 2, 6, 9, -1,
	3, 11, 2, 7, 8, 4, 10, 6, 5, -1, -1, -1, -1, -1, -1, -1, 5, 10, 6, 4, 7, 2, 4, 2, 0, 2, 7, 11, -1, -1, -1, -1, 0, 1, 9, 4, 7, 8, 2, 3, 11, 5, 10, 6, -1, -1, -1, -1, 9, 2, 1, 9, 11, 2, 9, 4, 11, 7, 11, 4, 5, 10, 6, -1,
	8, 4, 7, 3, 11, 5, 3, 5, 1, 5, 11, 6, -1, -1, -1, -1, 5, 1, 11, 5, 11, 6, 1, 0, 11, 7, 11, 4, 0, 4, 11, -1, 0, 5, 9, 0, 6, 5, 0, 3, 6, 11, 6, 3, 8, 4, 7, -1, 6, 5, 9, 6, 9, 11, 4, 7, 9, 7, 11, 9, -1, -1, -1, -1,
	10, 4, 9, 6, 4, 10, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 4, 10, 6, 4, 9, 10, 0, 8, 3, -1, -1, -1, -1, -1, -1, -1, 10, 0, 1, 10, 6, 0, 6, 4, 0, -1, -1, -1, -1, -1, -1, -1, 8, 3, 1, 8, 1, 6, 8, 6, 4, 6, 1, 10, -1, -1, -1, -1,
	1, 4, 9, 1, 2, 4, 2, 6, 4, -1, -1, -1, -1, -1, -1, -1, 3, 0, 8, 1, 2, 9, 2, 4, 9, 2, 6, 4, -1, -1, -1, -1, 0, 2, 4, 4, 2, 6, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 8, 3, 2, 8, 2, 4, 4, 2, 6, -1, -1, -1, -1, -1, -1, -1,
	10, 4, 9, 10, 6, 4, 11, 2, 3, -1, -1, -1, -1, -1, -1, -1, 0, 8, 2, 2, 8, 11, 4, 9, 10, 4, 10, 6, -1, -1, -1, -1, 3, 11, 2, 0, 1, 6, 0, 6, 4, 6, 1, 10, -1, -1, -1, -1, 6, 4, 1, 6, 1, 10, 4, 8, 1, 2, 1, 11, 8, 11, 1, -1,
	9, 6, 4, 9, 3, 6, 9, 1, 3, 11, 6, 3, -1, -1, -1, -1, 8, 11, 1, 8, 1, 0, 11, 6, 1, 9, 1, 4, 6, 4, 1, -1, 3, 11, 6, 3, 6, 0, 0, 6, 4, -1, -1, -1, -1, -1, -1, -1, 6, 4, 8, 11, 6, 8, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	7, 10, 6, 7, 8, 10, 8, 9, 10, -1, -1, -1, -1, -1, -1, -1, 0, 7, 3, 0, 10, 7, 0, 9, 10, 6, 7, 10, -1, -1, -1, -1, 10, 6, 7, 1, 10, 7, 1, 7, 8, 1, 8, 0, -1, -1, -1, -1, 10, 6, 7, 10, 7, 1, 1, 7, 3, -1, -1, -1, -1, -1, -1, -1,
	1, 2, 6, 1, 6, 8, 1, 8, 9, 8, 6, 7, -1, -1, -1, -1, 2, 6, 9, 2, 9, 1, 6, 7, 9, 0, 9, 3, 7, 3, 9, -1, 7, 8, 0, 7, 0, 6, 6, 0, 2, -1, -1, -1, -1, -1, -1, -1, 7, 3, 2, 6, 7, 2, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	2, 3, 11, 10, 6, 8, 10, 8, 9, 8, 6, 7, -1, -1, -1, -1, 2, 0, 7, 2, 7, 11, 0, 9, 7, 6, 7, 10, 9, 10, 7, -1, 1, 8, 0, 1, 7, 8, 1, 10, 7, 6, 7, 10, 2, 3, 11, -1, 11, 2, 1, 11, 1, 7, 10, 6, 1, 6, 7, 1, -1, -1, -1, -1,
	8, 9, 6, 8, 6, 7, 9, 1, 6, 11, 6, 3, 1, 3, 6, -1, 0, 9, 1, 11, 6, 7, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 7, 8, 0, 7, 0, 6, 3, 11, 0, 11, 6, 0, -1, -1, -1, -1, 7, 11, 6, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	7, 6, 11, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 3, 0, 8, 11, 7, 6, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 1, 9, 11, 7, 6, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 8, 1, 9, 8, 3, 1, 11, 7, 6, -1, -1, -1, -1, -1, -1, -1,
	10, 1, 2, 6, 11, 7, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 1, 2, 10, 3, 0, 8, 6, 11, 7, -1, -1, -1, -1, -1, -1, -1, 2, 9, 0, 2, 10, 9, 6, 11, 7, -1, -1, -1, -1, -1, -1, -1, 6, 11, 7, 2, 10, 3, 10, 8, 3, 10, 9, 8, -1, -1, -1, -1,
	7, 2, 3, 6, 2, 7, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 7, 0, 8, 7, 6, 0, 6, 2, 0, -1, -1, -1, -1, -1, -1, -1, 2, 7, 6, 2, 3, 7, 0, 1, 9, -1, -1, -1, -1, -1, -1, -1, 1, 6, 2, 1, 8, 6, 1, 9, 8, 8, 7, 6, -1, -1, -1, -1,
	10, 7, 6, 10, 1, 7, 1, 3, 7, -1, -1, -1, -1, -1, -1, -1, 10, 7, 6, 1, 7, 10, 1, 8, 7, 1, 0, 8, -1, -1, -1, -1, 0, 3, 7, 0, 7, 10, 0, 10, 9, 6, 10, 7, -1, -1, -1, -1, 7, 6, 10, 7, 10, 8, 8, 10, 9, -1, -1, -1, -1, -1, -1, -1,
	6, 8, 4, 11, 8, 6, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 3, 6, 11, 3, 0, 6, 0, 4, 6, -1, -1, -1, -1, -1, -1, -1, 8, 6, 11, 8, 4, 6, 9, 0, 1, -1, -1, -1, -1, -1, -1, -1, 9, 4, 6, 9, 6, 3, 9, 3, 1, 11, 3, 6, -1, -1, -1, -1,
	6, 8, 4, 6, 11, 8, 2, 10, 1, -1, -1, -1, -1, -1, -1, -1, 1, 2, 10, 3, 0, 11, 0, 6, 11, 0, 4, 6, -1, -1, -1, -1, 4, 11, 8, 4, 6, 11, 0, 2, 9, 2, 10, 9, -1, -1, -1, -1, 10, 9, 3, 10, 3, 2, 9, 4, 3, 11, 3, 6, 4, 6, 3, -1,
	8, 2, 3, 8, 4, 2, 4, 6, 2, -1, -1, -1, -1, -1, -1, -1, 0, 4, 2, 4, 6, 2, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 1, 9, 0, 2, 3, 4, 2, 4, 6, 4, 3, 8, -1, -1, -1, -1, 1, 9, 4, 1, 4, 2, 2, 4, 6, -1, -1, -1, -1, -1, -1, -1,
	8, 1, 3, 8, 6, 1, 8, 4, 6, 6, 10, 1, -1, -1, -1, -1, 10, 1, 0, 10, 0, 6, 6, 0, 4, -1, -1, -1, -1, -1, -1, -1, 4, 6, 3, 4, 3, 8, 6, 10, 3, 0, 3, 9, 10, 9, 3, -1, 10, 9, 4, 6, 10, 4, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	4, 9, 5, 7, 6, 11, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 8, 3, 4, 9, 5, 11, 7, 6, -1, -1, -1, -1, -1, -1, -1, 5, 0, 1, 5, 4, 0, 7, 6, 11, -1, -1, -1, -1, -1, -1, -1, 11, 7, 6, 8, 3, 4, 3, 5, 4, 3, 1, 5, -1, -1, -1, -1,
	9, 5, 4, 10, 1, 2, 7, 6, 11, -1, -1, -1, -1, -1, -1, -1, 6, 11, 7, 1, 2, 10, 0, 8, 3, 4, 9, 5, -1, -1, -1, -1, 7, 6, 11, 5, 4, 10, 4, 2, 10, 4, 0, 2, -1, -1, -1, -1, 3, 4, 8, 3, 5, 4, 3, 2, 5, 10, 5, 2, 11, 7, 6, -1,
	7, 2, 3, 7, 6, 2, 5, 4, 9, -1, -1, -1, -1, -1, -1, -1, 9, 5, 4, 0, 8, 6, 0, 6, 2, 6, 8, 7, -1, -1, -1, -1, 3, 6, 2, 3, 7, 6, 1, 5, 0, 5, 4, 0, -1, -1, -1, -1, 6, 2, 8, 6, 8, 7, 2, 1, 8, 4, 8, 5, 1, 5, 8, -1,
	9, 5, 4, 10, 1, 6, 1, 7, 6, 1, 3, 7, -1, -1, -1, -1, 1, 6, 10, 1, 7, 6, 1, 0, 7, 8, 7, 0, 9, 5, 4, -1, 4, 0, 10, 4, 10, 5, 0, 3, 10, 6, 10, 7, 3, 7, 10, -1, 7, 6, 10, 7, 10, 8, 5, 4, 10, 4, 8, 10, -1, -1, -1, -1,
	6, 9, 5, 6, 11, 9, 11, 8, 9, -1, -1, -1, -1, -1, -1, -1, 3, 6, 11, 0, 6, 3, 0, 5, 6, 0, 9, 5, -1, -1, -1, -1, 0, 11, 8, 0, 5, 11, 0, 1, 5, 5, 6, 11, -1, -1, -1, -1, 6, 11, 3, 6, 3, 5, 5, 3, 1, -1, -1, -1, -1, -1, -1, -1,
	1, 2, 10, 9, 5, 11, 9, 11, 8, 11, 5, 6, -1, -1, -1, -1, 0, 11, 3, 0, 6, 11, 0, 9, 6, 5, 6, 9, 1, 2, 10, -1, 11, 8, 5, 11, 5, 6, 8, 0, 5, 10, 5, 2, 0, 2, 5, -1, 6, 11, 3, 6, 3, 5, 2, 10, 3, 10, 5, 3, -1, -1, -1, -1,
	5, 8, 9, 5, 2, 8, 5, 6, 2, 3, 8, 2, -1, -1, -1, -1, 9, 5, 6, 9, 6, 0, 0, 6, 2, -1, -1, -1, -1, -1, -1, -1, 1, 5, 8, 1, 8, 0, 5, 6, 8, 3, 8, 2, 6, 2, 8, -1, 1, 5, 6, 2, 1, 6, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	1, 3, 6, 1, 6, 10, 3, 8, 6, 5, 6, 9, 8, 9, 6, -1, 10, 1, 0, 10, 0, 6, 9, 5, 0, 5, 6, 0, -1, -1, -1, -1, 0, 3, 8, 5, 6, 10, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 10, 5, 6, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	11, 5, 10, 7, 5, 11, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 11, 5, 10, 11, 7, 5, 8, 3, 0, -1, -1, -1, -1, -1, -1, -1, 5, 11, 7, 5, 10, 11, 1, 9, 0, -1, -1, -1, -1, -1, -1, -1, 10, 7, 5, 10, 11, 7, 9, 8, 1, 8, 3, 1, -1, -1, -1, -1,
	11, 1, 2, 11, 7, 1, 7, 5, 1, -1, -1, -1, -1, -1, -1, -1, 0, 8, 3, 1, 2, 7, 1, 7, 5, 7, 2, 11, -1, -1, -1, -1, 9, 7, 5, 9, 2, 7, 9, 0, 2, 2, 11, 7, -1, -1, -1, -1, 7, 5, 2, 7, 2, 11, 5, 9, 2, 3, 2, 8, 9, 8, 2, -1,
	2, 5, 10, 2, 3, 5, 3, 7, 5, -1, -1, -1, -1, -1, -1, -1, 8, 2, 0, 8, 5, 2, 8, 7, 5, 10, 2, 5, -1, -1, -1, -1, 9, 0, 1, 5, 10, 3, 5, 3, 7, 3, 10, 2, -1, -1, -1, -1, 9, 8, 2, 9, 2, 1, 8, 7, 2, 10, 2, 5, 7, 5, 2, -1,
	1, 3, 5, 3, 7, 5, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 8, 7, 0, 7, 1, 1, 7, 5, -1, -1, -1, -1, -1, -1, -1, 9, 0, 3, 9, 3, 5, 5, 3, 7, -1, -1, -1, -1, -1, -1, -1, 9, 8, 7, 5, 9, 7, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	5, 8, 4, 5, 10, 8, 10, 11, 8, -1, -1, -1, -1, -1, -1, -1, 5, 0, 4, 5, 11, 0, 5, 10, 11, 11, 3, 0, -1, -1, -1, -1, 0, 1, 9, 8, 4, 10, 8, 10, 11, 10, 4, 5, -1, -1, -1, -1, 10, 11, 4, 10, 4, 5, 11, 3, 4, 9, 4, 1, 3, 1, 4, -1,
	2, 5, 1, 2, 8, 5, 2, 11, 8, 4, 5, 8, -1, -1, -1, -1, 0, 4, 11, 0, 11, 3, 4, 5, 11, 2, 11, 1, 5, 1, 11, -1, 0, 2, 5, 0, 5, 9, 2, 11, 5, 4, 5, 8, 11, 8, 5, -1, 9, 4, 5, 2, 11, 3, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	2, 5, 10, 3, 5, 2, 3, 4, 5, 3, 8, 4, -1, -1, -1, -1, 5, 10, 2, 5, 2, 4, 4, 2, 0, -1, -1, -1, -1, -1, -1, -1, 3, 10, 2, 3, 5, 10, 3, 8, 5, 4, 5, 8, 0, 1, 9, -1, 5, 10, 2, 5, 2, 4, 1, 9, 2, 9, 4, 2, -1, -1, -1, -1,
	8, 4, 5, 8, 5, 3, 3, 5, 1, -1, -1, -1, -1, -1, -1, -1, 0, 4, 5, 1, 0, 5, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 8, 4, 5, 8, 5, 3, 9, 0, 5, 0, 3, 5, -1, -1, -1, -1, 9, 4, 5, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	4, 11, 7, 4, 9, 11, 9, 10, 11, -1, -1, -1, -1, -1, -1, -1, 0, 8, 3, 4, 9, 7, 9, 11, 7, 9, 10, 11, -1, -1, -1, -1, 1, 10, 11, 1, 11, 4, 1, 4, 0, 7, 4, 11, -1, -1, -1, -1, 3, 1, 4, 3, 4, 8, 1, 10, 4, 7, 4, 11, 10, 11, 4, -1,
	4, 11, 7, 9, 11, 4, 9, 2, 11, 9, 1, 2, -1, -1, -1, -1, 9, 7, 4, 9, 11, 7, 9, 1, 11, 2, 11, 1, 0, 8, 3, -1, 11, 7, 4, 11, 4, 2, 2, 4, 0, -1, -1, -1, -1, -1, -1, -1, 11, 7, 4, 11, 4, 2, 8, 3, 4, 3, 2, 4, -1, -1, -1, -1,
	2, 9, 10, 2, 7, 9, 2, 3, 7, 7, 4, 9, -1, -1, -1, -1, 9, 10, 7, 9, 7, 4, 10, 2, 7, 8, 7, 0, 2, 0, 7, -1, 3, 7, 10, 3, 10, 2, 7, 4, 10, 1, 10, 0, 4, 0, 10, -1, 1, 10, 2, 8, 7, 4, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	4, 9, 1, 4, 1, 7, 7, 1, 3, -1, -1, -1, -1, -1, -1, -1, 4, 9, 1, 4, 1, 7, 0, 8, 1, 8, 7, 1, -1, -1, -1, -1, 4, 0, 3, 7, 4, 3, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 4, 8, 7, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	9, 10, 8, 10, 11, 8, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 3, 0, 9, 3, 9, 11, 11, 9, 10, -1, -1, -1, -1, -1, -1, -1, 0, 1, 10, 0, 10, 8, 8, 10, 11, -1, -1, -1, -1, -1, -1, -1, 3, 1, 10, 11, 3, 10, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	1, 2, 11, 1, 11, 9, 9, 11, 8, -1, -1, -1, -1, -1, -1, -1, 3, 0, 9, 3, 9, 11, 1, 2, 9, 2, 11, 9, -1, -1, -1, -1, 0, 2, 11, 8, 0, 11, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 3, 2, 11, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	2, 3, 8, 2, 8, 10, 10, 8, 9, -1, -1, -1, -1, -1, -1, -1, 9, 10, 2, 0, 9, 2, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 2, 3, 8, 2, 8, 10, 0, 1, 8, 1, 10, 8, -1, -1, -1, -1, 1, 10, 2, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	1, 3, 8, 9, 1, 8, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 9, 1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 0, 3, 8, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
};
}
```
---
## MicroprofilerSample.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "0bf2c8c22c3dc582e4207a776cee495b07b0b055")]
public class MicroprofilerSample : Component
{
	private Engine.BACKGROUND_UPDATE previous_bg_update = Engine.BACKGROUND_UPDATE.BACKGROUND_UPDATE_DISABLED;
	private SampleDescriptionWindow sampleDescriptionWindow = new();
	void Init()
	{	
		string description;
		if (Profiler.MicroprofileUrl == "")
		{
			WindowManager.DialogError("Warning", "Microprofiler is not available!");
			description = "<font color=\"#de4a14\"><p>Microprofiler is not compiled.</p><p>Use development-release binaries.</p></font>";
		}
		else
		{
			description = "<p>Microprofiler url <font color=\"#de4a14\">" + Profiler.MicroprofileUrl + "</font></p>";
		}
		sampleDescriptionWindow.createWindow();
		WidgetLabel label = new(description)
		{
			FontRich = 1,
			FontWrap = 1,
			Width = 300
		};
		sampleDescriptionWindow.getParameterGroupBox().AddChild(label, Gui.ALIGN_LEFT);
		previous_bg_update = Engine.BackgroundUpdate;
		Engine.BackgroundUpdate = Engine.BACKGROUND_UPDATE.BACKGROUND_UPDATE_RENDER_NON_MINIMIZED;		
	}
	void Shutdown()
	{
		Engine.BackgroundUpdate = previous_bg_update;
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## MicroprofilerSleepyNode.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using System.Diagnostics;
using System.Threading;
using Unigine;
[Component(PropertyGuid = "24efca8e3d316e27c0870fada6cf2b5ea1511be4")]
public class MicroprofilerSleepyNode : Component
{
	private void Init()
	{
		int id = Profiler.BeginMicro("MicroprofilerSleepyUnit::Init()");
		Sleep(1000);
		Profiler.EndMicro(id);
	}
	private void UpdateAsyncThread()
	{
		int id = Profiler.BeginMicro("MicroprofilerSleepyUnit::UpdateAsyncThread()");
		Sleep(2000);
		Profiler.EndMicro(id);
	}
	private void UpdateSyncThread()
	{
		int id = Profiler.BeginMicro("MicroprofilerSleepyUnit::UpdateSyncThread()");
		Sleep(500);
		Profiler.EndMicro(id);
	}
	private void Update()
	{
		int id = Profiler.BeginMicro("MicroprofilerSleepyUnit::Update()");
		node.Rotate(0.0f, 0.0f, 3.0f);
		Sleep(750);
		Profiler.EndMicro(id);
	}
	private void PostUpdate()
	{
		int id = Profiler.BeginMicro("MicroprofilerSleepyUnit::PostUpdate()");
		Sleep(500);
		Profiler.EndMicro(id);
	}
	private void UpdatePhysics()
	{
		int id = Profiler.BeginMicro("MicroprofilerSleepyUnit::UpdatePhysics()");
		Sleep(20);
		Profiler.EndMicro(id);
	}
	private void Swap()
	{
		int id = Profiler.BeginMicro("MicroprofilerSleepyUnit::Swap()");
		Sleep(10);
		Profiler.EndMicro(id);
	}
	private void Shutdown()
	{
		int id = Profiler.BeginMicro("MicroprofilerSleepyUnit::Shutdown()");
		Sleep(1000);
		Profiler.EndMicro(id);
	}
	private static void Sleep(int microseconds)
	{
		Thread.Sleep(new TimeSpan(0, 0, 0, 0, 0, microseconds));
	}
}
```
---
## MouseRayIntersection.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "b9a76f513f32e07ca391c1ab87b2eb0e5e8a7715")]
public class MouseRayIntersection : Component
{
	public float distance = 100.0f;
	// use mask to separate objects for intersection
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.INTERSECTION)]
	public int mask = 1;
	private WidgetLabel label = null;
	Input.MOUSE_HANDLE initHandle;
	private void Init()
	{
		// show cursor when player is not rotate
		initHandle = Input.MouseHandle;
		Input.MouseHandle = Input.MOUSE_HANDLE.SOFT;
		// create label for target object name
		label = new WidgetLabel(Gui.GetCurrent());
		label.FontSize = 30;
		label.FontOutline = 1;
		Gui.GetCurrent().AddChild(label, Gui.ALIGN_OVERLAP);
	}
	private void Update()
	{
		// get points to detect intersection based on mouse position and player direction
		Vec3 firstPoint = Game.Player.WorldPosition;
		ivec2 mouse_coord = Input.MousePosition;
		Vec3 secondPoint = firstPoint + Game.Player.GetDirectionFromMainWindow(mouse_coord.x, mouse_coord.y) * distance;
		// try to get intersection object
		Unigine.Object hitObject = World.GetIntersection(firstPoint, secondPoint, mask);
		if (hitObject)
		{
			// change object name
			label.Text = hitObject.Name;
		}
		else
			label.Text = "empty hit object";
		// update cursor position
		label.SetPosition(WindowManager.MainWindow.Gui.MouseX + 25, WindowManager.MainWindow.Gui.MouseY + 25);
	}
	private void Shutdown()
	{
		Gui.GetCurrent().RemoveChild(label);
		Input.MouseHandle = initHandle;
	}
}
```
---
## NavigationMeshVisualizer.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "a556356231a6b559e3d89ee4a201dcf3c84afdb6")]
public class NavigationMeshVisualizer : Component
{
	private NavigationMesh navigationMesh = null;
	private void Init()
	{
		navigationMesh = node as NavigationMesh;
		if (navigationMesh)
			Visualizer.Enabled = true;
	}
	private void Update()
	{
		if (navigationMesh)
			navigationMesh.RenderVisualizer();
	}
	private void Shutdown()
	{
		Visualizer.Enabled = false;
	}
}
```
---
## NavigationSectorVisualizer.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "095d1febeed4344e05aaaf55a742b8b05998744f")]
public class NavigationSectorVisualizer : Component
{
	private NavigationSector navigationSector = null;
	private void Init()
	{
		navigationSector = node as NavigationSector;
		if (navigationSector)
			Visualizer.Enabled = true;
	}
	private void Update()
	{
		if (navigationSector)
			navigationSector.RenderVisualizer();
	}
	private void Shutdown()
	{
		Visualizer.Enabled = false;
	}
}
```
---
## NodeSpawner.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "3e8dfa296951fd7400e90a4322e62ac6e480036d")]
public class NodeSpawner : Component
{
	[ParameterFile(Filter = ".node")]
	public string instancePath = "";
	// collection of created nodes
	private List<Node> instances = null;
	// time between creation and deletion
	private const float spawnTimer = 0.5f;
	private float currentTime = 0.0f;
	private const int count = 20;
	private int currentIndex = 0;
	private void Init()
	{
		instances = new List<Node>();
		currentTime = spawnTimer;
	}
	private void Update()
	{
		currentTime -= Game.IFps;
		if (currentTime < 0)
		{
			// create a new instance of the node and add it to the collection
			Node newNode = World.LoadNode(instancePath);
			instances.Add(newNode);
			// set world position of node based on current index
			float x = 4.0f - 2.0f * (currentIndex / 4);
			float y = -4.0f + 2.0f * (currentIndex % 4);
			newNode.WorldPosition = new Vec3(x, y, 0.0f);
			if (instances.Count > count / 2)
			{
				// delete first node in collection
				instances[0].DeleteLater();
				instances.RemoveAt(0);
			}
			currentIndex++;
			if (currentIndex == count)
				currentIndex = 0;
			currentTime = spawnTimer;
		}
	}
}
```
---
## ObserverController.cs
```csharp
﻿using System;
using System.Collections.Generic;
using Unigine;
using System.Linq;
#region Math
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Mat4 = Unigine.mat4;
#endif
#endregion
[Component(PropertyGuid = "3bf2472427c699325eb25ef019b9a56619cd1934")]
public class ObserverController : Component
{
	[ShowInEditor, Parameter(Group = "Input", Title = "Toggle Camera Menu", Tooltip = "Key to toggle the visibility of the camera menu.")]
	Input.KEY toggleCameraMenu = Input.KEY.F3;
	[ShowInEditor, Parameter(Group = "Input", Title = "Focusing Key", Tooltip = "Key to focus the camera on the object, zooming in for a closer view.")]
	Input.KEY focusKey = Input.KEY.F;
	[ShowInEditor, Parameter(Group = "Input", Title = "Camera Control Modifier", Tooltip = "Modifier key for switching between different camera control modes.")]
	Input.MODIFIER altCameraModKey = Input.MODIFIER.ANY_ALT;
	[ShowInEditor, Parameter(Group = "Input", Title = "Acceleration Key", Tooltip = "Key to activate acceleration mode, typically used for faster movement.")]
	Input.KEY accelerationKey = Input.KEY.ANY_SHIFT;
	[ShowInEditor, Parameter(Group = "Input", Title = "Spectator Mode Mouse Button", Tooltip = "Mouse button to toggle spectator mode.")]
	Input.MOUSE_BUTTON spectatorModeMouseButton = Input.MOUSE_BUTTON.RIGHT;
	[ShowInEditor, Parameter(Group = "Input", Title = "Rail Mode Mouse Button", Tooltip = "Mouse button to activate rail mode.")]
	Input.MOUSE_BUTTON railModeMouseButton = Input.MOUSE_BUTTON.RIGHT;
	[ShowInEditor, Parameter(Group = "Input", Title = "Panning Mode Mouse Button", Tooltip = "Mouse button to initiate panning mode.")]
	Input.MOUSE_BUTTON panningModeMouseButton = Input.MOUSE_BUTTON.MIDDLE;
	[ShowInEditor, Parameter(Group = "Input", Title = "Panning/Rail Scale", Tooltip = "Scale factor for panning and rail modes, controlling the movement speed.")]
	float panningRailScale = 0.01f;
	[ShowInEditor, Parameter(Group = "Velocity", Title = "First Gear Key", Tooltip = "Key to activate the first gear (low speed).")]
	Input.KEY firstGearKey = Input.KEY.DIGIT_1;
	[ShowInEditor, Parameter(Group = "Velocity", Title = "First Gear Speed", Tooltip = "Velocity for the first gear.")]
	float firstGearVelocity = 5.0f;
	[ShowInEditor, Parameter(Group = "Velocity", Title = "Second Gear Key", Tooltip = "Key to activate the second gear (medium speed).")]
	Input.KEY secondGearKey = Input.KEY.DIGIT_2;
	[ShowInEditor, Parameter(Group = "Velocity", Title = "Second Gear Speed", Tooltip = "Velocity for the second gear.")]
	float secondGearVelocity = 50.0f;
	[ShowInEditor, Parameter(Group = "Velocity", Title = "Third Gear Key", Tooltip = "Key to activate the third gear (high speed).")]
	Input.KEY thirdGearKey = Input.KEY.DIGIT_3;
	[ShowInEditor, Parameter(Group = "Velocity", Title = "Third Gear Speed", Tooltip = "Velocity for the third gear.")]
	float thirdGearVelocity = 500.0f;
	[ShowInEditor, Parameter(Group = "Velocity", Title = "Acceleration Multiplier", Tooltip = "Multiplier for acceleration when the assigned key is held.")]
	float accelerationMultiplier = 2.0f;
	float _defaultVelocity = 5.0f;
	float _defaultPositionValue = 0f;
	bool _tryEndFocusing = false;
	bool _editText = false;
	WidgetHBox _menuLayout = null;
	WidgetCheckBox _firstGearCheckbox = null;
	WidgetCheckBox _secondGearCheckbox = null;
	WidgetCheckBox _thirdGearCheckBox = null;
	List<WidgetEditLine> _widgetEditLines = [];
	WidgetEditLine _currentGearVelocityTextField = null;
	WidgetEditLine _widgetCameraPositionX = null;
	WidgetEditLine _widgetCameraPositionY = null;
	WidgetEditLine _widgetCameraPositionZ = null;
	WorldIntersection _intersection = new WorldIntersection();
	Vec3 _targetPoint;
	PlayerSpectator _playerSpectator = null;
	bool _enterMouseGrabMode = false;
	public enum PlayerMovementState
	{
		IDLE,
		SPECTATOR,
		RAIL,
		FOCUSING,
		PANNING
	}
	PlayerMovementState _playerState = PlayerMovementState.IDLE;
	private readonly struct StateTransition(Func<bool> condition, PlayerMovementState tragetState)
	{
		public Func<bool> Condition { get; } = condition;
		public PlayerMovementState TargetState { get; } = tragetState;
	}
	private readonly struct MovementState(IEnumerable<StateTransition> transitions, Action onEnter = null, Action onExit = null, Action onUpdate = null)
	{
		public Action OnEnter { get; } = onEnter;
		public Action OnExit { get; } = onExit;
		public Action OnUpdate { get; } = onUpdate;
		public List<StateTransition> Transitions { get; } = transitions.ToList();
	}
	private Dictionary<PlayerMovementState, MovementState> _stateMap = null;
	public enum VelocityGear
	{
		GEAR_FIRST = 1,
		GEAR_SECOND,
		GEAR_THIRD
	}
	VelocityGear _velocityGear = VelocityGear.GEAR_FIRST;
	bool TryFocusing => Input.IsKeyDown(focusKey);
	bool TryEnterSpectatorMode => !Input.IsModifierEnabled(altCameraModKey) && Input.IsMouseButtonPressed(spectatorModeMouseButton);
	bool TryEnterRailMode => Input.IsModifierEnabled(altCameraModKey) && Input.IsMouseButtonPressed(railModeMouseButton);
	bool TryEnterPannigMode => Input.IsModifierEnabled(altCameraModKey) && Input.IsMouseButtonPressed(panningModeMouseButton);
	bool TryExitRailMode => !Input.IsMouseButtonPressed(railModeMouseButton);
	bool TryExitPanningMode => !Input.IsMouseButtonPressed(panningModeMouseButton);
	void Init()
	{
		_playerSpectator = Game.Player as PlayerSpectator;
		_enterMouseGrabMode = Input.MouseGrab;
		CreateStateTable();
		UpdateGear(_velocityGear);
		GenerateMenu();
	}
	void Update()
	{
		if (Unigine.Console.Active)
			return;
		if (_editText)
		{
			UpdateEditFieldSubmission();
			return;
		}
		UpdateVelocityGear();
		UpdateStates();
		UpdateMenuParams();
	}
	void Shutdown()
	{
		_menuLayout.DeleteLater();
		Input.MouseGrab = _enterMouseGrabMode;
	}
	// This method sets up the state transitions and associated actions for each movement state of the player.
	// Each state (e.g., IDLE, SPECTATOR, FOCUSING, etc.) is mapped to a set of conditions for state transitions, 
	// as well as initialization, update, and end functions for when the state is entered or exited.
	private void CreateStateTable()
	{
		_stateMap = new Dictionary<PlayerMovementState, MovementState>
		{
			[PlayerMovementState.IDLE] = new MovementState
			(
				transitions:
				[
					new (() => TryFocusing, PlayerMovementState.FOCUSING),
					new (() => TryEnterSpectatorMode, PlayerMovementState.SPECTATOR),
					new (() => TryEnterRailMode, PlayerMovementState.RAIL),
					new (() => TryEnterPannigMode, PlayerMovementState.PANNING)
				]
			),
			[PlayerMovementState.FOCUSING] = new MovementState
			(
				transitions:
				[
					new (() => _tryEndFocusing, PlayerMovementState.IDLE),
					new (() => TryEnterSpectatorMode, PlayerMovementState.SPECTATOR),
					new (() => TryEnterPannigMode, PlayerMovementState.PANNING),
					new (() => TryEnterRailMode, PlayerMovementState.RAIL)
				],
				onEnter: InitFocusing,
				onExit: EndFocusing,
				onUpdate: UpdateFocusing
			),
			[PlayerMovementState.SPECTATOR] = new MovementState
			(
				transitions:
				[
					new (() => !TryEnterSpectatorMode, PlayerMovementState.IDLE)
				],
				onEnter: InitSpectator,
				onExit: EndSpectator,
				onUpdate: UpdateSpectator
			),
			[PlayerMovementState.RAIL] = new MovementState
			(
				transitions:
				[
					new (() => TryExitRailMode, PlayerMovementState.IDLE)
				],
				onUpdate: UpdateRail
			),
			[PlayerMovementState.PANNING] = new MovementState
			(
				transitions:
				[
					new (() => TryExitPanningMode, PlayerMovementState.IDLE)
				],
				onEnter: InitPanning,
				onExit: EndPanning,
				onUpdate: UpdatePanning
			)
		};
	}
	private void UpdateStates()
	{
		var state = _stateMap[_playerState];
		foreach (var transition in state.Transitions)
		{
			if (transition.Condition())
			{
				SwitchState(transition.TargetState);
				return;
			}
		}
		state.OnUpdate?.Invoke();
	}
	private void SwitchState(PlayerMovementState newState)
	{
		if (_playerState == newState)
			return;
		_stateMap[_playerState].OnExit?.Invoke();
		_playerState = newState;
		_stateMap[_playerState].OnEnter?.Invoke();
	}
	void UpdateVelocityGear()
	{
		if (Input.IsKeyDown(firstGearKey))
		{
			_firstGearCheckbox.Checked = true;
		}
		if (Input.IsKeyDown(secondGearKey))
		{
			_secondGearCheckbox.Checked = true;
		}
		if (Input.IsKeyDown(thirdGearKey))
		{
			_thirdGearCheckBox.Checked = true;
		}
	}
	void UpdateGear(VelocityGear newGear)
	{
		_velocityGear = newGear;
		_playerSpectator.MinVelocity = GetVelocity();
		_playerSpectator.MaxVelocity = GetVelocityAcceleration();
	}
	float GetVelocity()
	{
		return _velocityGear switch
		{
			VelocityGear.GEAR_FIRST => firstGearVelocity,
			VelocityGear.GEAR_SECOND => secondGearVelocity,
			VelocityGear.GEAR_THIRD => thirdGearVelocity,
			_ => 0.0f
		};
	}
	float GetVelocityAcceleration()
	{
		return GetVelocity() * accelerationMultiplier;
	}
	void SetVelocity(VelocityGear targetGear, float velocity)
	{
		switch (targetGear)
		{
			case VelocityGear.GEAR_FIRST:
				firstGearVelocity = velocity;
				break;
			case VelocityGear.GEAR_SECOND:
				secondGearVelocity = velocity;
				break;
			case VelocityGear.GEAR_THIRD:
				thirdGearVelocity = velocity;
				break;
		}
		UpdateGear(targetGear);
	}
	void InitSpectator()
	{
		_playerSpectator.Controlled = true;
		ControlsApp.MouseEnabled = true;
	}
	void UpdateSpectator()
	{
		Input.MouseCursorHide = true;
	}
	void EndSpectator()
	{
		_playerSpectator.Controlled = false;
	}
	void UpdateRail()
	{
		Input.MouseCursorHide = true;
		ivec2 mouseDelta = Input.MouseDeltaPosition;
		if (mouseDelta != ivec2.ZERO)
			MathLib.Normalize(mouseDelta);
		float currentAcceleration = Input.IsKeyPressed(accelerationKey) ? GetVelocity() : GetVelocityAcceleration();
		float delta = -(mouseDelta.x + mouseDelta.y) * currentAcceleration * ControlsApp.MouseSensitivity * panningRailScale;
		_playerSpectator.Translate(new Vec3(0, 0, delta));
	}
	void InitPanning()
	{
		Input.MouseGrab = true;
		ControlsApp.MouseEnabled = true;
	}
	void UpdatePanning()
	{
		ivec2 mouse_delta = Input.MouseDeltaPosition;
		float current_acceleration = Input.IsKeyPressed(accelerationKey) ? GetVelocity() : GetVelocityAcceleration();
		_playerSpectator.Translate(new Vec3(-mouse_delta.x, mouse_delta.y, 0) * current_acceleration * ControlsApp.MouseSensitivity * panningRailScale);
		Input.MouseCursorHide = true;
	}
	void EndPanning()
	{
		Input.MouseGrab = false;
	}
	void InitFocusing()
	{
		_playerSpectator.GetDirectionFromMainWindow(out Vec3 startPoint, out Vec3 endPoint, Input.MousePosition.x, Input.MousePosition.y);
		Vec3 objPosition;
		double focusRadius;
		Node obj = World.GetIntersection(startPoint, endPoint, ~0, _intersection);
		if (!obj)
		{
			_tryEndFocusing = true;
			return;
		}
		objPosition = obj.WorldPosition;
		focusRadius = obj.WorldBoundSphere.Radius;
		// Block, with an example for focusing specifically on instances of ObjectMeshCluster
		if (obj as ObjectMeshCluster != null)
		{
			ObjectMeshCluster cluster = obj as ObjectMeshCluster;
			int instanceIndex = _intersection.Instance;
			Mesh mesh = cluster.GetMeshCurrentRAM();
			Mat4 transform = cluster.GetMeshTransform(instanceIndex);
			objPosition = cluster.WorldTransform * transform.Translate;
			//When searching the radius, we take into account both the scale of the instance and the ObjectMeshCluster itself.
			focusRadius = mesh.BoundSphere.Radius * cluster.Scale.Maximum * transform.Scale.Maximum;
		}
		//We take the doubled radius so that the camera is at a distance from the focus point.
		_targetPoint = objPosition - (Vec3)(_playerSpectator.GetWorldDirection() * focusRadius * 2);
	}
	void UpdateFocusing()
	{
		_playerSpectator.WorldPosition = MathLib.Lerp(_playerSpectator.WorldPosition, _targetPoint, Game.IFps * 5);
		if (MathLib.Distance2(_playerSpectator.WorldPosition, _targetPoint) < 0.01f)
			_tryEndFocusing = true;
	}
	void EndFocusing()
	{
		_tryEndFocusing = false;
	}
	void GenerateMenu()
	{
		_menuLayout = new WidgetHBox(0, 4)
		{
			Background = 1,
		};
		Gui.GetCurrent().AddChild(_menuLayout, Gui.ALIGN_TOP);	
		WidgetHBox gearsLayout = new (5, 0);
		_menuLayout.AddChild(gearsLayout,Gui.ALIGN_LEFT);
		WidgetLabel labelName = new WidgetLabel("Player speed:");
		gearsLayout.AddChild(labelName, Gui.ALIGN_LEFT);
		_currentGearVelocityTextField = new WidgetEditLine(GetVelocity().ToString("F5"))
		{
			Validator = 3,
			Width = 100
		};
		gearsLayout.AddChild(_currentGearVelocityTextField, Gui.ALIGN_LEFT);
		_widgetEditLines.Add(_currentGearVelocityTextField);
		_currentGearVelocityTextField.EventFocusIn.Connect(() => _editText = true);
		_currentGearVelocityTextField.EventFocusOut.Connect(() =>
		{
			_editText = false;
			SetVelocity
			(
				_velocityGear,
				_currentGearVelocityTextField.Text.Length == 0 ? _defaultVelocity : float.Parse(_currentGearVelocityTextField.Text)
			);
		});
		_firstGearCheckbox = new WidgetCheckBox("1")
		{
			Checked = true
		};
		gearsLayout.AddChild(_firstGearCheckbox, Gui.ALIGN_LEFT);
		_firstGearCheckbox.EventChanged.Connect(() =>
		{
			if (_firstGearCheckbox.Checked)
				ChangeGearTextField(VelocityGear.GEAR_FIRST);
		});
		_secondGearCheckbox = new WidgetCheckBox("2");
		_firstGearCheckbox.AddAttach(_secondGearCheckbox);
		gearsLayout.AddChild(_secondGearCheckbox, Gui.ALIGN_LEFT);
		_secondGearCheckbox.EventChanged.Connect(() =>
		{
			if (_secondGearCheckbox.Checked)
				ChangeGearTextField(VelocityGear.GEAR_SECOND);
		});
		_thirdGearCheckBox = new WidgetCheckBox("3");
		_firstGearCheckbox.AddAttach(_thirdGearCheckBox);
		gearsLayout.AddChild(_thirdGearCheckBox, Gui.ALIGN_LEFT);
		_thirdGearCheckBox.EventChanged.Connect(() =>
		{
			if (_thirdGearCheckBox.Checked)
				ChangeGearTextField(VelocityGear.GEAR_THIRD);
		});
		WidgetSpacer gearSpacer = new WidgetSpacer()
		{
			Orientation = 0
		};
		_menuLayout.AddChild(gearSpacer,Gui.ALIGN_LEFT);
		WidgetHBox positionLayout = new WidgetHBox(5,0);
		_menuLayout.AddChild(positionLayout, Gui.ALIGN_LEFT);
		labelName = new WidgetLabel("X:");
		positionLayout.AddChild(labelName, Gui.ALIGN_LEFT);
		_widgetCameraPositionX = new WidgetEditLine(_playerSpectator.WorldPosition.x.ToString("F5"))
		{
			Validator = 3,
			Width = 100
		};
		positionLayout.AddChild(_widgetCameraPositionX, Gui.ALIGN_LEFT);
		_widgetEditLines.Add(_widgetCameraPositionX);
		_widgetCameraPositionX.EventFocusIn.Connect(() =>  _editText = true);
		_widgetCameraPositionX.EventKeyPressed.Connect(() => 
		{
			float value = _widgetCameraPositionX.Text.Length == 0 ? _defaultPositionValue : float.Parse(_widgetCameraPositionX.Text);
			Vec3 position = _playerSpectator.WorldPosition;
			_playerSpectator.WorldPosition = new Vec3(value, position.y, position.z);
		});
		_widgetCameraPositionX.EventFocusOut.Connect(() =>
		{
			_editText = false;
			if (_widgetCameraPositionX.Text.Length == 0)
				_widgetCameraPositionX.Text = _defaultPositionValue.ToString("F5");
		});
		labelName = new WidgetLabel("Y:");
		positionLayout.AddChild(labelName, Gui.ALIGN_LEFT);
		_widgetCameraPositionY = new WidgetEditLine(_playerSpectator.WorldPosition.y.ToString("F5"))
		{
			Validator = 3,
			Width = 100
		};
		positionLayout.AddChild(_widgetCameraPositionY, Gui.ALIGN_LEFT);
		_widgetEditLines.Add(_widgetCameraPositionY);
		_widgetCameraPositionY.EventFocusIn.Connect(() => _editText = true);
		_widgetCameraPositionY.EventKeyPressed.Connect(() =>
		{
			float value = _widgetCameraPositionY.Text.Length == 0 ? _defaultPositionValue : float.Parse(_widgetCameraPositionY.Text);
			Vec3 position = _playerSpectator.WorldPosition;
			_playerSpectator.WorldPosition = new Vec3(position.x, value, position.z);
		});
		_widgetCameraPositionY.EventFocusOut.Connect(() =>
		{
			_editText = false;
			if (_widgetCameraPositionY.Text.Length == 0)
			_widgetCameraPositionY.Text = _defaultPositionValue.ToString("F5");
		});
		labelName = new WidgetLabel("Z:");
		positionLayout.AddChild(labelName, Gui.ALIGN_LEFT);
		_widgetCameraPositionZ = new WidgetEditLine(_playerSpectator.WorldPosition.z.ToString("F5"))
		{
			Validator = 3,
			Width = 100
		};
		positionLayout.AddChild(_widgetCameraPositionZ, Gui.ALIGN_LEFT);
		_widgetEditLines.Add(_widgetCameraPositionZ);
		_widgetCameraPositionZ.EventFocusIn.Connect(() => _editText = true);
		_widgetCameraPositionZ.EventKeyPressed.Connect(() =>
		{
			float value = _widgetCameraPositionZ.Text.Length == 0 ? _defaultPositionValue : float.Parse(_widgetCameraPositionZ.Text);
			Vec3 position = _playerSpectator.WorldPosition;
			_playerSpectator.WorldPosition = new Vec3(position.x, position.y, value);
		});
		_widgetCameraPositionZ.EventFocusOut.Connect(() => 
		{
			_editText = false;
			if (_widgetCameraPositionY.Text.Length == 0)
				_widgetCameraPositionY.Text = _defaultPositionValue.ToString("F5");
		});
	}
	void UpdateMenuParams()
	{
		if (Input.IsKeyDown(toggleCameraMenu))
			_menuLayout.Hidden = !_menuLayout.Hidden;
		_widgetCameraPositionX.Text = _playerSpectator.WorldPosition.x.ToString("F5");
		_widgetCameraPositionY.Text = _playerSpectator.WorldPosition.y.ToString("F5");
		_widgetCameraPositionZ.Text = _playerSpectator.WorldPosition.z.ToString("F5");
	}
	void UpdateEditFieldSubmission()
	{
		if (_widgetEditLines.Count != 0 && Input.IsKeyDown(Input.KEY.ENTER))
		{
			foreach (var widget in _widgetEditLines)
				widget.RemoveFocus();
		}
	}
	void ChangeGearTextField(VelocityGear newGear)
	{
		UpdateGear(newGear);
		_currentGearVelocityTextField.Text = GetVelocity().ToString();
	}
}
```
---
## ObstacleVisualizer.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "51c164503d08eb6fe188220240177979b4d75484")]
public class ObstacleVisualizer : Component
{
	private Obstacle obstacle = null;
	private void Init()
	{
		obstacle = node as Obstacle;
		if (obstacle)
			Visualizer.Enabled = true;
	}
	private void Update()
	{
		if (obstacle)
			obstacle.RenderVisualizer();
	}
	private void Shutdown()
	{
		Visualizer.Enabled = false;
	}
}
```
---
## PathRoute2D.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "ec10a5720354c62988b25974f975b6439a92e6c6")]
public class PathRoute2D : Component
{
	public Node startPoint = null;
	public Node finishPoint = null;
	public bool visualizeRoute = false;
	[ParameterColor]
	public vec4 routeColor = vec4.ZERO;
	private PathRoute route = null;
	private void Init()
	{
		if (startPoint && finishPoint)
		{
			// create route to path calculation
			route = new PathRoute();
			// set point radius inside navigation mesh
			route.Radius = 0.5f;
			// enabled for visualization
			Visualizer.Enabled = true;
		}
	}
	private void Update()
	{
		// check points for correctness
		if (startPoint && finishPoint)
		{
			// try to calculate path from start to finish
			route.Create2D(startPoint.WorldPosition, finishPoint.WorldPosition);
			if (route.IsReached)
			{
				// if successful, show the current route
				if (visualizeRoute)
					route.RenderVisualizer(routeColor);
			}
			else
				Log.Message($"{node.Name} PathRoute not reached yet\n");
		}
	}
	private void Shutdown()
	{
		Visualizer.Enabled = false;
	}
}
```
---
## PathRoute2DWithTarget.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "3fee417fcf07e0d2d7eb0a09d43f0bd44a0d0523")]
public class PathRoute2DWithTarget : Component
{
	public float speed = 4.0f;
	public NavigationMesh navigationMesh = null;
	[ParameterFile(Filter = ".node")]
	public string targetReferencePath = "";
	public bool visualizeRoute = false;
	[ParameterColor]
	public vec4 routeColor = vec4.ZERO;
	private PathRoute route = null;
	private Node target = null;
	private void Init()
	{
		if (!navigationMesh)
			return;
		// create target
		target = World.LoadNode(targetReferencePath);
		if (target)
		{
			// set random position in navigation mesh
			target.WorldPosition = new Vec3(Game.GetRandomFloat(-55.0f, 55.0f), Game.GetRandomFloat(-55.0f, 55.0f), 0.5f);
			// save target position in navigation mesh
			while (!navigationMesh.Inside2D(target.WorldPosition, 0.5f))
				target.WorldPosition = new Vec3(Game.GetRandomFloat(-55.0f, 55.0f), Game.GetRandomFloat(-55.0f, 55.0f), 0.5f);
			// create route to path calculation
			route = new PathRoute();
			// set point radius inside navigation mesh
			route.Radius = 0.5f;
			// does not use negative speed
			speed = MathLib.Max(0.0f, speed);
			// enabled for visualization
			if (visualizeRoute)
				Visualizer.Enabled = true;
		}
	}
	private void Update()
	{
		if (!navigationMesh || !target)
			return;
		// change position of target if it is near to current node
		if ((target.WorldPosition - node.WorldPosition).Length < 1.0f)
		{
			target.WorldPosition = new Vec3(Game.GetRandomFloat(-55.0f, 55.0f), Game.GetRandomFloat(-55.0f, 55.0f), 0.5f);
			// save target position in navigation mesh
			while (!navigationMesh.Inside2D(target.WorldPosition, 0.5f))
				target.WorldPosition = new Vec3(Game.GetRandomFloat(-55.0f, 55.0f), Game.GetRandomFloat(-55.0f, 55.0f), 0.5f);
		}
		// if current path is ready, try to move node
		if (route.IsReady)
		{
			// in successful case change direction and position
			if (route.IsReached)
			{
				// enable target from inside obstacle
				if (!target.Enabled)
					target.Enabled = true;
				// get new direction for node
				vec3 direction = new vec3(route.GetPoint(1) - route.GetPoint(0));
				if (direction.Length2 > MathLib.EPSILON)
				{
					// get rotation target based on new direction
					quat targetRotation = new quat(MathLib.SetTo(vec3.ZERO, direction.Normalized, vec3.UP, MathLib.AXIS.Y));
					// smoothly change rotation
					quat currentRotation = MathLib.Slerp(node.GetWorldRotation(), targetRotation, Game.IFps * 8.0f);
					node.SetWorldRotation(currentRotation);
					// translate in forward direction and try to save node position in navigation mesh
					Vec3 lastValidPosition = node.WorldPosition;
					node.Translate(Vec3.FORWARD * Game.IFps * speed);
					// restore last position if node is outside navigation mesh
					if (!navigationMesh.Inside2D(node.WorldPosition, route.Radius))
						node.WorldPosition = lastValidPosition;
				}
				// render current path
				if (visualizeRoute)
					route.RenderVisualizer(routeColor);
				// try to create new path
				route.Create2D(node.WorldPosition + vec3.UP * 0.5f, target.WorldPosition, 1);
			}
			else
			{
				// hide target and change position, because it can be in obstacle
				target.Enabled = false;
				target.WorldPosition = new Vec3(Game.GetRandomFloat(-55.0f, 55.0f), Game.GetRandomFloat(-55.0f, 55.0f), 0.5f);
				// save target position in navigation mesh
				while (!navigationMesh.Inside2D(target.WorldPosition, 0.5f))
					target.WorldPosition = new Vec3(Game.GetRandomFloat(-55.0f, 55.0f), Game.GetRandomFloat(-55.0f, 55.0f), 0.5f);
				// try to create new path
				route.Create2D(node.WorldPosition + vec3.UP * 0.5f, target.WorldPosition, 1);
			}
		}
		// try to create new path
		else if (!route.IsQueued)
			route.Create2D(node.WorldPosition + vec3.UP * 0.5f, target.WorldPosition, 1);
	}
	private void Shutdown()
	{
		Visualizer.Enabled = false;
	}
}
```
---
## PathRoute3D.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "a245a802a31b47795e1cca844379d413e9f94a56")]
public class PathRoute3D : Component
{
	public Node startPoint = null;
	public Node finishPoint = null;
	public bool visualizeRoute = false;
	[ParameterColor]
	public vec4 routeColor = vec4.ZERO;
	private PathRoute route = null;
	private void Init()
	{
		if (startPoint && finishPoint)
		{
			// create route to path calculation
			route = new PathRoute();
			// set point radius inside navigation mesh
			route.Radius = 0.5f;
			// enabled for visualization
			Visualizer.Enabled = true;
		}
	}
	private void Update()
	{
		// check points for correctness
		if (startPoint && finishPoint)
		{
			// try to calculate path from start to finish
			route.Create3D(startPoint.WorldPosition, finishPoint.WorldPosition);
			if (route.IsReached)
			{
				// if successful, show the current route
				if (visualizeRoute)
					route.RenderVisualizer(routeColor);
			}
			else
				Log.Message($"{node.Name} PathRoute not reached yet\n");
		}
	}
	private void Shutdown()
	{
		Visualizer.Enabled = false;
	}
}
```
---
## PathRoute3DWithTarget.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "b5fc1ffe0ec9c745d7d08fe0583981bf4cb3a412")]
public class PathRoute3DWithTarget : Component
{
	public float speed = 10.0f;
	public NavigationSector navigationSector = null;
	[ParameterFile(Filter = ".node")]
	public string targetReferencePath = "";
	public bool visualizeRoute = false;
	[ParameterColor]
	public vec4 routeColor = vec4.ZERO;
	private PathRoute route = null;
	private Node target = null;
	private void Init()
	{
		if (!navigationSector)
			return;
		// create target
		target = World.LoadNode(targetReferencePath);
		if (target)
		{
			// set random position in navigation sector
			target.WorldPosition = new Vec3()
			{
				x = Game.GetRandomFloat(-60.0f, 60.0f),
				y = Game.GetRandomFloat(-60.0f, 60.0f),
				z = Game.GetRandomFloat(0.0f, 60.0f)
			};
			// save target position in navigation sector
			while (!navigationSector.Inside3D(target.WorldPosition, 0.5f))
				target.WorldPosition = new Vec3()
				{
					x = Game.GetRandomFloat(-60.0f, 60.0f),
					y = Game.GetRandomFloat(-60.0f, 60.0f),
					z = Game.GetRandomFloat(0.0f, 60.0f)
				};
			// create route to path calculation
			route = new PathRoute();
			// set point radius inside navigation sector
			route.Radius = 0.5f;
			// does not use negative speed
			speed = MathLib.Max(0.0f, speed);
			// enabled for visualization
			if (visualizeRoute)
				Visualizer.Enabled = true;
		}
	}
	private void Update()
	{
		if (!navigationSector || !target)
			return;
		// change position of target if it is near to current node
		if ((target.WorldPosition - node.WorldPosition).Length < 1.0f)
		{
			target.WorldPosition = new Vec3()
			{
				x = Game.GetRandomFloat(-60.0f, 60.0f),
				y = Game.GetRandomFloat(-60.0f, 60.0f),
				z = Game.GetRandomFloat(0.0f, 60.0f)
			};
			// save target position in navigation sector
			while (!navigationSector.Inside3D(target.WorldPosition, 0.5f))
				target.WorldPosition = new Vec3()
				{
					x = Game.GetRandomFloat(-60.0f, 60.0f),
					y = Game.GetRandomFloat(-60.0f, 60.0f),
					z = Game.GetRandomFloat(0.0f, 60.0f)
				};
		}
		// if current path is ready, try to move node
		if (route.IsReady)
		{
			// in successful case change direction and position
			if (route.IsReached)
			{
				// enable target from inside obstacle
				if (!target.Enabled)
					target.Enabled = true;
				// get new direction for node
				vec3 direction = new vec3(route.GetPoint(1) - route.GetPoint(0));
				if (direction.Length2 > MathLib.EPSILON)
				{
					// get rotation target based on new direction
					quat targetRotation = new quat(MathLib.SetTo(vec3.ZERO, direction.Normalized, vec3.UP, MathLib.AXIS.Y));
					// smoothly change rotation
					quat currentRotation = MathLib.Slerp(node.GetWorldRotation(), targetRotation, Game.IFps * 8.0f);
					node.SetWorldRotation(currentRotation);
					// translate in forward direction and try to save node position in navigation sector
					Vec3 lastValidPosition = node.WorldPosition;
					node.Translate(Vec3.FORWARD * Game.IFps * speed);
					// restore last position if node is outside navigation sector
					if (!navigationSector.Inside3D(node.WorldPosition, route.Radius))
						node.WorldPosition = lastValidPosition;
				}
				// render current path
				if (visualizeRoute)
					route.RenderVisualizer(routeColor);
				// try to create new path
				route.Create3D(node.WorldPosition + vec3.UP * 0.5f, target.WorldPosition, 1);
			}
			else
			{
				// hide target and change position, because it can be in obstacle
				target.Enabled = false;
				target.WorldPosition = new Vec3()
				{
					x = Game.GetRandomFloat(-60.0f, 60.0f),
					y = Game.GetRandomFloat(-60.0f, 60.0f),
					z = Game.GetRandomFloat(0.0f, 60.0f)
				};
				// save target position in navigation sector
				while (!navigationSector.Inside3D(target.WorldPosition, 0.5f))
					target.WorldPosition = new Vec3()
					{
						x = Game.GetRandomFloat(-60.0f, 60.0f),
						y = Game.GetRandomFloat(-60.0f, 60.0f),
						z = Game.GetRandomFloat(0.0f, 60.0f)
					};
				// try to create new path
				route.Create3D(node.WorldPosition + vec3.UP * 0.5f, target.WorldPosition, 1);
			}
		}
		// try to create new path
		else if (!route.IsQueued)
			route.Create3D(node.WorldPosition + vec3.UP * 0.5f, target.WorldPosition, 1);
	}
	private void Shutdown()
	{
		Visualizer.Enabled = false;
	}
}
```
---
## PathTrajectorySaver.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "450a58bb1478b012e23ce0771dc58a21add5b15e")]
public class PathTrajectorySaver : Component
{
	[ShowInEditor][ParameterFile]
	private string pathFile = null;
	[ShowInEditor]
	private Node pathNode = null;
	[ShowInEditor]
	private int quality = 25;
	[ShowInEditor]
	private bool autosave = true;
	void Init()
	{
		if (autosave)
			Save();
	}
	private void Save()
	{
		Path path = new Path();
		path.Clear();
		int points_count = pathNode.NumChildren;
		double frame_time = 0;
		for (int j = 0; j < points_count; j++)
		{
			int j_prev = (j - 1 < 0) ? (points_count - 1) : j - 1;
			int j_cur = j;
			int j_next = (j + 1) % points_count;
			int j_next_next = (j + 2) % points_count;
			// get current control position and rotation
			Vec3 p0 = pathNode.GetChild(j_prev).WorldPosition;
			Vec3 p1 = pathNode.GetChild(j_cur).WorldPosition;
			Vec3 p2 = pathNode.GetChild(j_next).WorldPosition;
			Vec3 p3 = pathNode.GetChild(j_next_next).WorldPosition;
			quat q0 = pathNode.GetChild(j_prev).GetWorldRotation();
			quat q1 = pathNode.GetChild(j_cur).GetWorldRotation();
			quat q2 = pathNode.GetChild(j_next).GetWorldRotation();
			quat q3 = pathNode.GetChild(j_next_next).GetWorldRotation();
			// calculate curve
			Vec3 start = Utils.CatmullRomCentripetal(p0, p1, p2, p3, 0);
			for (int i = 1; i < quality; i++)
			{
				path.AddFrame();
				float time = (float)i / (quality - 1);
				// calculate segment position and rotation
				Vec3 end = Utils.CatmullRomCentripetal(p0, p1, p2, p3, time);
				quat rot = Utils.Squad(q0, q1, q2, q3, time);
				// calculate current frame time
				double len = MathLib.Length(start - end);
				frame_time += len;
				path.SetFramePosition(path.NumFrames - 1, end);
				path.SetFrameRotation(path.NumFrames - 1, rot);
				path.SetFrameTime(path.NumFrames - 1, (float)frame_time);
				start = end;
			}
		}
		// save to file
		path.Save(pathFile);
	}
}
```
---
## PhysicalBuyoancy.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "95dd6a8567f4cce4bdb16d41c8c0dc03fa138a93")]
public class PhysicalBuyoancy : Component
{
	private SampleDescriptionWindow window = new();
	public Node waves_node = null;
	public Node buoyancy_node = null;
	private Waves waves = null;
	private Buoyancy buoyancy = null;
	void Init()
	{
		if (waves_node == null)
			return;
		if (buoyancy_node == null)
			return;
		waves = GetComponent<Waves>(waves_node);
		buoyancy = GetComponent<Buoyancy>(buoyancy_node);
		buoyancy.SetVisualizer(false);
		window.createWindow();
		window.addFloatParameter("Beaufort", null, 0.0f, 0.0f, 8.00f, on_beaufort_slider_changed);
		window.addBoolParameter("Debug", null, false, on_debug_checkbox_clicked);
	}
	void Shutdown()
	{
		window.shutdown();
	}
	private void on_beaufort_slider_changed(float value)
	{
		waves.SetBeaufort(value);
	}
	private void on_debug_checkbox_clicked(bool value)
	{
		buoyancy.SetVisualizer(value);
	}
}
```
---
## PrimitivesSpawner.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "fc0ff54b594d4a2ecf680bb55e0ea99069ccefaf")]
public class PrimitivesSpawner : Component
{
	private void Init()
	{
		CreateBox();
		CreateSphere();
		CreateCylinder();
		CreateCapsule();
		CreatePrism();
		CreatePlane();
	}
	private void CreateBox()
	{
		// first, we're creating a mesh instance as it is required to call an ObjectMeshDynamic constructor
		Mesh boxMesh = new Mesh();
		// creating a box surface with a given size to the mesh
		boxMesh.AddBoxSurface("box_surface", new vec3(1.0f, 1.0f, 1.0f));
		//create an ObjectMeshDynamic node and set position
		ObjectMeshDynamic box = new ObjectMeshDynamic(boxMesh);
		box.WorldPosition = new Vec3(-5.0f, 0.0f, 1.5f);
		// clearing the mesh
		boxMesh.Clear();
	}
	private void CreateSphere()
	{
		Mesh sphereMesh = new Mesh();
		sphereMesh.AddSphereSurface("sphere_surface", 0.5f, 16, 16);
		ObjectMeshDynamic sphere = new ObjectMeshDynamic(sphereMesh);
		sphere.WorldPosition = new Vec3(-3.0f, 0.0f, 1.5f);
		sphereMesh.Clear();
	}
	private void CreateCylinder()
	{
		Mesh cylinderMesh = new Mesh();
		cylinderMesh.AddCylinderSurface("cylinder_surface", 0.5f, 1.0f, 16, 16);
		ObjectMeshDynamic cylinder = new ObjectMeshDynamic(cylinderMesh);
		cylinder.WorldPosition = new Vec3(-1.0f, 0.0f, 1.5f);
		cylinderMesh.Clear();
	}
	private void CreateCapsule()
	{
		Mesh capsuleMesh = new Mesh();
		capsuleMesh.AddCapsuleSurface("capsule_surface", 0.5f, 1.0f, 16, 16);
		ObjectMeshDynamic capsule = new ObjectMeshDynamic(capsuleMesh);
		capsule.WorldPosition = new Vec3(1.0f, 0.0f, 1.5f);
		capsuleMesh.Clear();
	}
	private void CreatePrism()
	{
		Mesh prismMesh = new Mesh();
		prismMesh.AddPrismSurface("prism_surface", 0.5f, 1.0f, 0.5f, 5);
		ObjectMeshDynamic prism = new ObjectMeshDynamic(prismMesh);
		prism.WorldPosition = new Vec3(3.0f, 0.0f, 1.5f);
		prismMesh.Clear();
	}
	private void CreatePlane()
	{
		Mesh planeMesh = new Mesh();
		planeMesh.AddPlaneSurface("plane_surface", 1.0f, 1.0f, 1);
		ObjectMeshDynamic plane = new ObjectMeshDynamic(planeMesh);
		plane.WorldPosition = new Vec3(5.0f, 0.0f, 1.5f);
		planeMesh.Clear();
	}
}
```
---
## ProceduralMeshApply.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using System.Numerics;
using Unigine;
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
#endif
[Component(PropertyGuid = "f57beaf90b900209989d36d8cad84699660d88e5")]
public class ProceduralMeshApply : Component
{
	private Mesh mesh;
	private ObjectMeshCluster cluster;
	private float radius = 0.5f;
	private const int maxNumStacks = 30;
	private const int minNumStacks = 2;
	private int numStacks = 2;
	private int numSlices = 3;
	// signals if we increase or decrease number of slices and stacks
	private bool isIncreasing = true;
	// timer to change sphere parameters
	private float changeRate = 0.1f;
	private float currentTime = 0.0f;
	// x/y-size of cluster field
	private const int size = 20;
	// offset between meshes
	private float offset = 1.0f;
	void Init()
	{
		mesh = new Mesh();
		cluster = new ObjectMeshCluster();
		// before changing mesh choose Procedural Mode:
		// - Disable - procedural mode is disabled
		// - Dynamic - fastest performance, stored in RAM and VRAM, not automatically unloaded from
		// memory.
		// - Blob - moderate performance, stored in RAM and VRAM, automatically unloaded from memory.
		// - File - slowest performance, all data stored on disk, automatically unloaded from memory.
		cluster.SetMeshProceduralMode(ObjectMeshStatic.PROCEDURAL_MODE.DYNAMIC);
		cluster.WorldPosition = new Vec3(0.0f, 0.0f, 3.0f);
		// create cluster transforms
		List<Mat4> transforms = new List<Mat4>();
		float fieldOffset = (1.0f + offset) * size / 2.0f;
		for (int y = 0; y < size; y++)
		{
			for (int x = 0; x < size; x++)
			{
				transforms.Add(MathLib.Translate(new Vec3(x + x * offset - fieldOffset, y + y * offset - fieldOffset, 1.5f)));
			}
		}
		cluster.AppendMeshes(transforms.ToArray());
		Visualizer.Enabled = true;
	}
	void Update()
	{
		// change mesh before applying
		UpdateMesh(mesh);
		// Apply new mesh. You can do it Force or Async.
		// Changing mesh_render_flag you can choose where to store MeshRender data: in RAM or VRAM
		// 0 - store everything in VRAM (default behavior)
		// USAGE_DYNAMIC_VERTEX - store vertices on RAM
		// USAGE_DYNAMIC_INDICES - store indices on RAM
		// USAGE_DYNAMIC_ALL - store both vertices and indices on RAM
		cluster.ApplyMoveMeshProceduralAsync(mesh, 0);
		Visualizer.RenderObject(cluster, vec4.GREEN);
	}
	void Shitdown()
	{
		mesh.Clear();
		cluster.DeleteLater();
		Visualizer.Enabled = false;
	}
	private void UpdateMesh(Mesh mesh)
	{
		currentTime += Game.IFps;
		if (currentTime > changeRate)
		{
			currentTime = 0.0f;
			numSlices = isIncreasing ? numSlices + 1 : numSlices - 1;
			numStacks = isIncreasing ? numStacks + 1 : numStacks - 1;
			if (numStacks == maxNumStacks)
				isIncreasing = false;
			if (numStacks <= minNumStacks)
			{
				isIncreasing = true;
				numStacks = minNumStacks;
				numSlices = numStacks + 1;
			}
		}
		mesh.Clear();
		mesh.AddSphereSurface("sphere", radius, numStacks, numSlices);
	}
}
```
---
## ProceduralMeshGenerator.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
using System.Threading;
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
#else
using Vec3 = Unigine.vec3;
#endif
[Component(PropertyGuid = "38c7ebbf881a8e43654c3dec38ba576198467a58")]
public class ProceduralMeshGenerator : Component
{
	// field size
	private int size = 128;
	private int newSize;
	private int numObjects;
	// offset between boxes
	private float offset = 0.5f;
	private int numCreatedObjects = 0;
	private bool isCreatingObjects = false;
	private bool isDeletingDone = false;
	private List<ObjectMeshStatic> boxObjects = new List<ObjectMeshStatic>();
	private ObjectMeshStatic.PROCEDURAL_MODE currentMode;
	private ObjectMeshStatic.PROCEDURAL_MODE lastMode;
	int currentMeshRenderFlag;
	// UI
	private SampleDescriptionWindow sampleDescriptionWindow;
	private Dictionary<string, ObjectMeshStatic.PROCEDURAL_MODE> modesMap;
	private Dictionary<string, int> flagsMap;
	private WidgetComboBox modeCombo;
	private WidgetComboBox flagsCombo;
	private WidgetEditLine editline;
	private WidgetSpinBox spinbox;
	private WidgetButton generateButton;
	private WidgetButton clearButton;
	void Init()
	{
		numObjects = size * size;
		newSize = size;
		InitGui();
		Profiler.Enabled = true;
	}
	void Update()
	{
		// check that all objects are created
		if (isCreatingObjects && numCreatedObjects == numObjects)
		{
			SetGuiEnabled(true);
			isCreatingObjects = false;
		}
		// check that deleting of all objects is finished
		if (isDeletingDone)
		{
			isDeletingDone = false;
			if (isCreatingObjects)
				CreateObjects();
			else
				SetGuiEnabled(true);
		}
		// change to new field size if there are no active object creation processes
		if (!isCreatingObjects && newSize != size)
		{
			size = newSize;
			numObjects = size * size;
		}
		// update memory usage statistic
		UpdateStats();
	}
	void Shutdown()
	{
		ClearObjects();
		ShutdownGui();
		Profiler.Enabled = false;
	}
	private void CreateObjects()
	{
		isCreatingObjects = true;
		if (boxObjects.Count > 0)
		{
			ClearObjects();
			return;
		}
		float fieldOffset = (1.0f + offset) * size / 2.0f;
		boxObjects.Capacity = numObjects;
		for (int x = 0; x < size; x++)
		{
			for (int y = 0; y < size; y++)
			{
				var obj = new ObjectMeshStatic();
				obj.WorldPosition = new Vec3(x + x * offset - fieldOffset, y + y * offset - fieldOffset, 1.5f);
				obj.SetMeshProceduralMode(currentMode);
				// this method will itself generate new mesh by create_mesh callback, load MeshRender,
				// apply new mesh to object and then call create_done callback when everything is done
				obj.RunGenerateMeshProceduralAsync(CreateMesh, () => { CreateDone(obj); }, currentMeshRenderFlag);
				boxObjects.Add(obj);
			}
		}
	}
	private void CreateMesh(Mesh mesh)
	{
		mesh.AddBoxSurface("box", vec3.ONE);
	}
	private void CreateDone(ObjectMeshStatic obj)
	{
		Interlocked.Increment(ref numCreatedObjects);
	}
	private void ClearObjects()
	{
		Interlocked.Exchange(ref numCreatedObjects, 0);
		foreach (var obj in boxObjects)
		{
			obj.DeleteLater();
		}
		boxObjects.Clear();
		isDeletingDone = true;
	}
	private void UpdateStats()
	{
		float ram = 0, activeVram = 0;
		string status;
		if (Profiler.Enabled)
		{
			switch (lastMode)
			{
				// Memory usage by Dynamic procedural meshes is constant throught all the time we use these meshes
				case ObjectMeshStatic.PROCEDURAL_MODE.DYNAMIC:
					ram = Profiler.GetValue("RAM Meshes Procedural Dynamic: ");
					activeVram = Profiler.GetValue("VRAM Meshes Procedural Dynamic: ");
					break;
				// File and Blob procedural meshes occupy memory only when thea are activce, otherwise they
				// are unloaded to cache
				case ObjectMeshStatic.PROCEDURAL_MODE.BLOB:
				case ObjectMeshStatic.PROCEDURAL_MODE.FILE:
					ram = Profiler.GetValue("RAM Meshes Static Used: ");
					activeVram = Profiler.GetValue("VRAM Meshes Static Used: ");
					break;
				default: break;
			}
			status = $"Num ready objects: {numCreatedObjects}\nRAM: {ram} MB\nVRAM Active: {activeVram} MB";
		}
		else
		{
			status = $"To see memory usage enable profiler.";
		}
		sampleDescriptionWindow.setStatus(status);
	}
	private void InitGui()
	{
		sampleDescriptionWindow = new SampleDescriptionWindow();
		sampleDescriptionWindow.createWindow(Gui.ALIGN_RIGHT);
		var params_box = sampleDescriptionWindow.getParameterGroupBox();
		var gridbox = new WidgetGridBox(2, 10);
		params_box.AddChild(gridbox);
		//	--------Procedural Mode Selector--------
		modesMap = new Dictionary<string, ObjectMeshStatic.PROCEDURAL_MODE>();
		modesMap["Dynamic"] = ObjectMeshStatic.PROCEDURAL_MODE.DYNAMIC;
		modesMap["File"] = ObjectMeshStatic.PROCEDURAL_MODE.FILE;
		modesMap["Blob"] = ObjectMeshStatic.PROCEDURAL_MODE.BLOB;
		var label = new WidgetLabel("Procedural Mode");
		gridbox.AddChild(label, Gui.ALIGN_LEFT);
		modeCombo = new WidgetComboBox();
		modeCombo.AddItem("Dynamic");
		modeCombo.AddItem("File");
		modeCombo.AddItem("Blob");
		gridbox.AddChild(modeCombo, Gui.ALIGN_EXPAND);
		currentMode = ObjectMeshStatic.PROCEDURAL_MODE.DYNAMIC;
		modeCombo.CurrentItem = 0;
		modeCombo.EventChanged.Connect(() =>
		{
			var item = modeCombo.GetCurrentItemText();
			currentMode = modesMap[item];
		});
		//	--------MeshRender Dynamic Usage Selector--------
		flagsMap = new Dictionary<string, int>();
		flagsMap["None"] = 0;
		flagsMap["DYNAMIC_VERTEX"] = MeshRender.USAGE_DYNAMIC_VERTEX;
		flagsMap["DYNAMIC_INDICES"] = MeshRender.USAGE_DYNAMIC_INDICES;
		flagsMap["DYNAMIC_ALL"] = MeshRender.USAGE_DYNAMIC_ALL;
		label = new WidgetLabel("MeshRender Flag");
		gridbox.AddChild(label, Gui.ALIGN_LEFT);
		flagsCombo = new WidgetComboBox();
		flagsCombo.AddItem("None");
		flagsCombo.AddItem("DYNAMIC_VERTEX");
		flagsCombo.AddItem("DYNAMIC_INDICES");
		flagsCombo.AddItem("DYNAMIC_ALL");
		gridbox.AddChild(flagsCombo, Gui.ALIGN_EXPAND);
		currentMeshRenderFlag = 0;
		flagsCombo.CurrentItem = 0;
		flagsCombo.EventChanged.Connect(() =>
		{
			var item = flagsCombo.GetCurrentItemText();
			currentMeshRenderFlag = flagsMap[item];
		});
		//	--------Fiels Size Input--------
		label = new WidgetLabel("Field Size");
		gridbox.AddChild(label, Gui.ALIGN_LEFT);
		var spinbox_hbox = new WidgetHBox();
		gridbox.AddChild(spinbox_hbox, Gui.ALIGN_EXPAND);
		spinbox = new WidgetSpinBox(1, 1000);
		spinbox.Value = size;
		spinbox.EventChanged.Connect(() => { newSize = spinbox.Value; });
		editline = new WidgetEditLine();
		editline.Validator = Gui.VALIDATOR_UINT;
		editline.EventFocusOut.Connect(() =>
		{
			int text = int.Parse(editline.Text);
			newSize = MathLib.Clamp(text, spinbox.MinValue, spinbox.MaxValue);
			if (text != newSize)
				editline.Text = newSize.ToString();
		});
		editline.Text = size.ToString();
		editline.AddAttach(spinbox);
		spinbox_hbox.AddChild(editline);
		spinbox_hbox.AddChild(spinbox, Gui.ALIGN_EXPAND);
		//	--------Spacer--------
		var spacer = new WidgetSpacer();
		spacer.Orientation = 1;
		params_box.AddChild(spacer, Gui.ALIGN_EXPAND);
		//	--------Create buttons--------
		var hbox = new WidgetHBox(10);
		params_box.AddChild(hbox, Gui.ALIGN_EXPAND);
		generateButton = new WidgetButton("Generate");
		generateButton.EventClicked.Connect(OnGenerateButton);
		hbox.AddChild(generateButton, Gui.ALIGN_EXPAND);
		clearButton = new WidgetButton("Clear");
		clearButton.EventClicked.Connect(OnClearButton);
		hbox.AddChild(clearButton, Gui.ALIGN_EXPAND);
	}
	private void ShutdownGui()
	{
		sampleDescriptionWindow.shutdown();
		modesMap.Clear();
		flagsMap.Clear();
	}
	private void SetGuiEnabled(bool enabled)
	{
		generateButton.Enabled = enabled;
		clearButton.Enabled = enabled;
	}
	private void OnGenerateButton()
	{
		SetGuiEnabled(false);
		lastMode = currentMode;
		CreateObjects();
	}
	private void OnClearButton()
	{
		SetGuiEnabled(false);
		ClearObjects();
	}
}
```
---
## ProceduralMeshModifier.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using System.Threading;
using Unigine;
using System;
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
#else
using Vec3 = Unigine.vec3;
#endif
[Component(PropertyGuid = "8fc3ca2973f720aa9d76649e232482266fdbdeea")]
public class ProceduralMeshModifier : Component
{
	private int size = 128;
	private float isize;
	private ObjectMeshStatic objectMesh;
	private Mesh meshRAM;
	private MeshRender meshVRAM;
	// parameter to coordinate different threads
	private bool isDeleted;
	private ObjectMeshStatic.PROCEDURAL_MODE currentMode;
	private int currentMeshRenderFlag;
	// enables creating CollisionData for mesh
	private bool isCollisionEnabled = false;
	// enables mesh creation in async thread
	private bool isThreadAsync = true;
	// enables apply*MeshProceduralAsync instead of apply*MeshProceduralForce
	private bool isAsyncMode = true;
	// enables applyCopyMeshProcedural* instead of applyMoveMeshProcedural*
	private bool isCopyMode = true;
	// enables creating MeshRender yourself and not by engine inside applyMeshProcedural methods
	private bool isMeshvramManual = false;
	// reserve meshvram manual state, so it won't be changed mid mesh update in different thread
	private bool updatedMeshvramManual;
	// check if we already in the process of modifying a mesh
	private bool isRunning = false;
	// UI
	private SampleDescriptionWindow sampleDescriptionWindow;
	private WidgetComboBox threadCombo;
	private WidgetComboBox asyncCombo;
	private WidgetComboBox moveCombo;
	private Dictionary<string, ObjectMeshStatic.PROCEDURAL_MODE> modesMap;
	private Dictionary<string, int> usageMap;
	private WidgetComboBox modeCombo;
	private WidgetComboBox usageCombo;
	private WidgetCheckBox meshvramCheckbox;
	private WidgetLabel warningLabel;
	void Init()
	{
		InitGui();
		isDeleted = false;
		updatedMeshvramManual = isMeshvramManual;
		isize = 30.0f / size;
		meshRAM = new Mesh();
		meshVRAM = new MeshRender();
		objectMesh = new ObjectMeshStatic();
		objectMesh.SetMeshProceduralMode(currentMode);
		objectMesh.WorldPosition = Vec3.ONE;
	}
	void Update()
	{
		int id = Profiler.BeginMicro("Component Update");
		// check if mesh modification or applying is already in active state
		if (isRunning || objectMesh.IsMeshProceduralActive)
		{
			Profiler.EndMicro(id);
			return;
		}
		isRunning = true;
		isMeshvramManual = updatedMeshvramManual;
		// set new procedural mode if needed
		if (objectMesh.MeshProceduralMode != currentMode)
		{
			objectMesh.DeleteLater();
			objectMesh = new ObjectMeshStatic();
			objectMesh.SetMeshProceduralMode(currentMode);
			objectMesh.WorldPosition = Vec3.ONE;
		}
		// check if mesh must be generated on Background or Main thread
		if (isThreadAsync)
		{
			// updates mesh on Background thread without blocking Main thread
			// mesh modification will be executed on different thread for as long as it's needed without
			// affecting perfomance
			AsyncQueue.RunAsync(AsyncQueue.ASYNC_THREAD.BACKGROUND, AsyncUpdateRAM);
		}
		else
		{
			// updates mesh in current frame before update ends
			UpdateRAM();
			if (isMeshvramManual)
				UpdateVRAM();
			ApplyData();
		}
		Profiler.EndMicro(id);
	}
	void Shutdown()
	{
		// signal for other threads that Shutdown() was called on Main thread
		isDeleted = true;
		ShutdownGui();
		// if there is some active mesh modification in progress, wait till it's finished before
		// clearing meshes
		lock (meshRAM)
		{
			meshRAM.Clear();
			meshVRAM.Clear();
			objectMesh.DeleteLater();
		}
	}
	// updates geometry
	private void UpdateMesh(Mesh mesh)
	{
		int id = Profiler.BeginMicro("ProceduralMeshModifier.UpdateMesh");
		float time = Game.Time;
		if (mesh.NumSurfaces != 1)
		{
			mesh.Clear();
			mesh.AddSurface("");
		}
		else
		{
			mesh.ClearSurface();
		}
		var vertices = new vec3[size * size];
		for (int y = 0; y < size; y++)
		{
			float Y = y * isize - 15.0f;
			float Z = MathLib.Cos(Y + time);
			for (int x = 0; x < size; x++)
			{
				float X = x * isize - 15.0f;
				vertices[y * size + x] = (new vec3(X, Y, Z * MathLib.Sin(X + time)));
			}
		}
		mesh.AddVertex(vertices);
		// reserve enough memory for indices so vector won't be reallocated every time it's capacity
		// ends
		var cindices = new List<int>();
		cindices.Capacity = (size - 1) * (size - 1) * 6;
		var tindices = new List<int>();
		tindices.Capacity = (size - 1) * (size - 1) * 6;
		var addIndex = (int index) =>
		{
			cindices.Add(index);
			tindices.Add(index);
		};
		for (int y = 0; y < size - 1; y++)
		{
			int offset = size * y;
			for (int x = 0; x < size - 1; x++)
			{
				addIndex(offset);
				addIndex(offset + 1);
				addIndex(offset + size);
				addIndex(offset + size);
				addIndex(offset + 1);
				addIndex(offset + size + 1);
				offset++;
			}
		}
		mesh.AddCIndices(cindices.ToArray());
		mesh.AddTIndices(tindices.ToArray());
		cindices.Clear();
		tindices.Clear();
		mesh.CreateTangents();
		{
			int idScope = Profiler.BeginMicro("CreateCollisionData");
			// if you plan to use intersection and collision with this mesh then it's better to create
			// CollisionData. Otherwise intersections and collisions would be highly ineffective in
			// terms of perfomance
			if (isCollisionEnabled)
			{
				// creates both Spatial Tree and Edges for effective intersection and collision
				// respectively
				mesh.CreateCollisionData();
				// you can create only Spatial Tree if you need only intersection
				//		mesh.CreateSpatialTree();
				// or you can create only Edges if you need only collision
				//		mesh.CreateEdges();
			}
			else
				// or you can create mesh without collision data or even delete existing one if there is
				// no need for it.
				// to check if mesh has collisionData you can use:
				//		mesh.HasCollisionData();
				//		mesh.HasSpatialTree();
				//		mesh.HasEdges();
				mesh.ClearCollisionData();
			Profiler.EndMicro(idScope);
		}
		mesh.CreateBounds();
		Profiler.EndMicro(id);
	}
	// updates Mesh
	private void UpdateRAM()
	{
		int id = Profiler.BeginMicro("ProceduralMeshModifier.UpdateRAM");
		// check that Main thread is stil active
		if (isDeleted)
		{
			Profiler.EndMicro(id);
			return;
		}
		// lock mesh_ram so other threads won't interfere mesh update
		lock (meshRAM)
		{
			UpdateMesh(meshRAM);
		}
		Profiler.EndMicro(id);
	}
	private void AsyncUpdateRAM()
	{
		// modify mesh
		UpdateRAM();
		if (isMeshvramManual)
		{
			// if you need to load MeshRender manualy, do it on GPU_STREAM thread
			AsyncQueue.RunAsync(AsyncQueue.ASYNC_THREAD.GPU_STREAM, AsyncUpdateVRAM);
		}
		else
		{
			// if you don't need to load MeshRender manualy and automatic loading inside
			// apllyMeshProcedural methods is enough then return to Main thread
			AsyncQueue.RunAsync(AsyncQueue.ASYNC_THREAD.MAIN, () =>
			{
				// check that sample's logic on Main thread is still alive. If it's not then stop
				// modification and return
				if (!node || node.IsDeleted)
					return;
				ApplyData();
			});
		}
	}
	// updates MeshRender
	private void UpdateVRAM()
	{
		int id = Profiler.BeginMicro("ProceduralMeshModifier.UpdateVRAM");
		// check that Main thread is stil active
		if (isDeleted)
		{
			Profiler.EndMicro(id);
			return;
		}
		// lock so other threads won't interfere mesh update
		lock (meshRAM)
		{
			meshVRAM.Load(meshRAM);
		}
		Profiler.EndMicro(id);
	}
	private void AsyncUpdateVRAM()
	{
		// update MeshRender
		UpdateVRAM();
		// return to Main thread to apply new mesh
		AsyncQueue.RunAsync(AsyncQueue.ASYNC_THREAD.MAIN, () => {
			// check that sample's logic on Main thread is still alive. If it's not then stop
			// modification and return
			if (!node || node.IsDeleted)
				return;
			ApplyData();
		});
	}
	// applies updated procedural mesh only on Main thread!
	private void ApplyData()
	{
		int id = Profiler.BeginMicro("ProceduralMeshModifier.ApplyData");
		// if apply happens in async mode it will be processed on another thread without blocking Main
		// thread. Otherwise in Force mode Main thread won't leave the scope of this function until
		// apply is finished.
		if (isAsyncMode)
		{
			if (isMeshvramManual)
			{
				// you can use manualy created MeshRender only in Move mode
				objectMesh.ApplyMoveMeshProceduralAsync(meshRAM, meshVRAM);
			}
			else
			{
				if (isCopyMode)
					// with Copy mode data from mesh_ram will be copied for internal use and mesh_ram
					// itself won't be changed
					objectMesh.ApplyCopyMeshProceduralAsync(meshRAM, currentMeshRenderFlag);
			else
					// with Move mode data will be taken from mesh_ram for internal use so mesh_ram
					// will change
					objectMesh.ApplyMoveMeshProceduralAsync(meshRAM, currentMeshRenderFlag);
			}
		}
		else
		{
			if (isMeshvramManual)
			{
				objectMesh.ApplyMoveMeshProceduralForce(meshRAM, meshVRAM);
			}
			else
			{
				if (isCopyMode)
					objectMesh.ApplyCopyMeshProceduralForce(meshRAM, currentMeshRenderFlag);
			else
					objectMesh.ApplyMoveMeshProceduralForce(meshRAM, currentMeshRenderFlag);
			}
		}
		// full cycle of mesh modification is finished
		isRunning = false;
		Profiler.EndMicro(id);
	}
	// UI methods
	private void InitGui()
	{
		sampleDescriptionWindow = new SampleDescriptionWindow();
		sampleDescriptionWindow.createWindow(Gui.ALIGN_RIGHT);
		var params_box = sampleDescriptionWindow.getParameterGroupBox();
		var gridbox = new WidgetGridBox(2, 10);
		params_box.AddChild(gridbox, Gui.ALIGN_EXPAND);
		//	--------Async/Force Mode Selector--------
		var label = new WidgetLabel("Thread");
		gridbox.AddChild(label, Gui.ALIGN_LEFT);
		threadCombo = new WidgetComboBox();
		threadCombo.AddItem("Main");
		threadCombo.AddItem("Background");
		gridbox.AddChild(threadCombo, Gui.ALIGN_EXPAND);
		threadCombo.CurrentItem = 1;
		threadCombo.EventChanged.Connect(() =>
		{
			isThreadAsync = threadCombo.CurrentItem != 0;
		});
		//	--------Procedural Mode Selector--------
		modesMap = new Dictionary<string, ObjectMeshStatic.PROCEDURAL_MODE>();
		modesMap["Dynamic"] = ObjectMeshStatic.PROCEDURAL_MODE.DYNAMIC;
		modesMap["File"] = ObjectMeshStatic.PROCEDURAL_MODE.FILE;
		modesMap["Blob"] = ObjectMeshStatic.PROCEDURAL_MODE.BLOB;
		label = new WidgetLabel("Procedural Mode");
		gridbox.AddChild(label, Gui.ALIGN_LEFT);
		modeCombo = new WidgetComboBox();
		modeCombo.AddItem("Dynamic");
		modeCombo.AddItem("File");
		modeCombo.AddItem("Blob");
		gridbox.AddChild(modeCombo, Gui.ALIGN_EXPAND);
		currentMode = ObjectMeshStatic.PROCEDURAL_MODE.DYNAMIC;
		modeCombo.CurrentItem = 0;
		modeCombo.EventChanged.Connect(() =>
		{
			var item = modeCombo.GetCurrentItemText();
			currentMode = modesMap[item];
		});
		//	--------MeshRender Flag Selector--------
		usageMap = new Dictionary<string, int>();
		usageMap["None"] = 0;
		usageMap["DYNAMIC_VERTEX"] = MeshRender.USAGE_DYNAMIC_VERTEX;
		usageMap["DYNAMIC_INDICES"] = MeshRender.USAGE_DYNAMIC_INDICES;
		usageMap["DYNAMIC_ALL"] = MeshRender.USAGE_DYNAMIC_ALL;
		label = new WidgetLabel("MeshRender Flag");
		gridbox.AddChild(label, Gui.ALIGN_LEFT);
		usageCombo = new WidgetComboBox();
		usageCombo.AddItem("None");
		usageCombo.AddItem("DYNAMIC_VERTEX");
		usageCombo.AddItem("DYNAMIC_INDICES");
		usageCombo.AddItem("DYNAMIC_ALL");
		gridbox.AddChild(usageCombo, Gui.ALIGN_EXPAND);
		currentMeshRenderFlag = 0;
		usageCombo.CurrentItem = 0;
		usageCombo.EventChanged.Connect(() =>
		{
			var item = usageCombo.GetCurrentItemText();
			currentMeshRenderFlag = usageMap[item];
		});
		//	--------Async/Force Mode Selector--------
		label = new WidgetLabel("Async Mode");
		gridbox.AddChild(label, Gui.ALIGN_LEFT);
		asyncCombo = new WidgetComboBox();
		asyncCombo.AddItem("Async");
		asyncCombo.AddItem("Force");
		gridbox.AddChild(asyncCombo, Gui.ALIGN_EXPAND);
		asyncCombo.CurrentItem = 0;
		asyncCombo.EventChanged.Connect(() =>
		{
			isAsyncMode = asyncCombo.CurrentItem == 0;
		});
		//	--------MeshRender Dynamic Usage Selector--------
		label = new WidgetLabel("Apply mode");
		gridbox.AddChild(label, Gui.ALIGN_LEFT);
		moveCombo = new WidgetComboBox();
		moveCombo.AddItem("Copy");
		moveCombo.AddItem("Move");
		gridbox.AddChild(moveCombo, Gui.ALIGN_EXPAND);
		isCopyMode = true;
		moveCombo.CurrentItem = 0;
		moveCombo.EventChanged.Connect(() =>
		{
			isCopyMode = moveCombo.CurrentItem == 0;
		});
		//	--------Create Collision Data--------
		label = new WidgetLabel("Create CollisionData");
		gridbox.AddChild(label, Gui.ALIGN_LEFT);
		var collison_checkbox = new WidgetCheckBox();
		gridbox.AddChild(collison_checkbox, Gui.ALIGN_EXPAND);
		collison_checkbox.EventChanged.Connect(() => { isCollisionEnabled = collison_checkbox.Checked; });
		//	--------Create MeshRender Manualy--------
		label = new WidgetLabel("Manual MeshRender");
		gridbox.AddChild(label, Gui.ALIGN_LEFT);
		meshvramCheckbox = new WidgetCheckBox();
		gridbox.AddChild(meshvramCheckbox, Gui.ALIGN_EXPAND);
		meshvramCheckbox.EventChanged.Connect(() =>
		{
			warningLabel.Hidden = !meshvramCheckbox.Checked;
		});
		//	--------Create MeshRender Warning--------
		warningLabel = new WidgetLabel("MeshRender can be used only with mode \"Move\". Apply mode \"Copy\" will be ignored ")
		{
			FontWrap = 1,
			Hidden = true,
			FontColor = vec4.RED
		};
		params_box.FontWrap = 1;
		params_box.AddChild(warningLabel, Gui.ALIGN_EXPAND);
	}
	private void ShutdownGui()
	{
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## Projectile.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "bfa1e43dc28cd8785f8dc1777656306a5073a4c1")]
public class Projectile : Component
{
	public float moveSpeed = 5.0f;
	[ParameterFile(Filter = ".node")]
	public string bulletSpawnFx;
	[ParameterFile(Filter = ".node")]
	public string bulletHitFx;
	private WorldIntersectionNormal intersection = null;
	private void Init()
	{
		if (bulletSpawnFx != "")
		{
			Node bulletSpawn = World.LoadNode(bulletSpawnFx);
			bulletSpawn.WorldPosition = node.WorldPosition;
		}
		intersection = new WorldIntersectionNormal();
	}
	private void Update()
	{
		Vec3 oldPosition = node.WorldPosition;
		vec3 direction = node.GetWorldDirection(MathLib.AXIS.Y);
		node.WorldPosition += direction * moveSpeed * Game.IFps;
		WorldIntersectionNormal intersection = new WorldIntersectionNormal();
		Unigine.Object hitObj = World.GetIntersection(oldPosition, node.WorldPosition, 0xFFFFFF, intersection);
		if (hitObj)
		{
			if (hitObj.Name == "turret" || hitObj.Name == "bullet")
				return;
			Robo robo = hitObj.GetComponent<Robo>();
			if (robo != null)
				robo.Hit(20);
			if (bulletHitFx != "")
			{
				Node bulletHit = World.LoadNode(bulletHitFx);
				bulletHit.WorldPosition = intersection.Point;
				bulletHit.SetWorldDirection(intersection.Normal, vec3.UP, MathLib.AXIS.Y);
			}
			node.DeleteLater();
		}
	}
}
```
---
## Robo.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "7424e4b32ae8e2765203775d7040111821d76197")]
public class Robo : Component
{
	public float moveSpeed = 5.0f;
	public float turnSpeed = 90.0f;
	public int health = 50;
	[ParameterFile(Filter = ".node")]
	public string deathFx = "";
	private SampleDescriptionWindow sampleDescriptionWindow;
	private void Init()
	{
		var description = ComponentSystem.FindComponentInWorld<DescriptionWindowCreator>();
		if (description)
			sampleDescriptionWindow = description.getWindow();
	}
	private void Update()
	{
		if (Input.IsKeyPressed(Input.KEY.UP))
			Move(moveSpeed);
		if (Input.IsKeyPressed(Input.KEY.DOWN))
			Move(-moveSpeed);
		if (Input.IsKeyPressed(Input.KEY.LEFT))
			Turn(turnSpeed);
		if (Input.IsKeyPressed(Input.KEY.RIGHT))
			Turn(-turnSpeed);
	}
	private void Move(float speed)
	{
		node.WorldPosition += node.GetWorldDirection(MathLib.AXIS.Y) * speed * Game.IFps;
	}
	private void Turn(float speed)
	{
		node.Rotate(0, 0, speed * Game.IFps);
	}
	public void Hit(int damage)
	{
		health -= damage;
		var status = $"Health: {health}";
		if (sampleDescriptionWindow != null)
			sampleDescriptionWindow.setStatus(status);
		if (health <= 0)
			node.DeleteLater();
	}
	private void Shutdown()
	{
		if (deathFx != "")
		{
			Node deathFxNode = World.LoadNode(deathFx);
			deathFxNode.WorldPosition = node.WorldPosition;
		}
	}
}
```
---
## RobotArmConnection.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "14dbb4a732390adb693f20c20b2d7b277512f948")]
public class RobotArmConnection : Component
{
	[ShowInEditor]
	private Node connectionPoint = null;
	[ShowInEditor]
	private PhysicalTrigger connectionTrigger = null;
	private JointFixed jointFixed;
	private BodyRigid connectionCandidate = null;
	private void Init()
	{
		connectionTrigger.EventEnter.Connect(OnTriggerEnter);
		connectionTrigger.EventLeave.Connect(OnTriggerLeave);
		int num = node.ObjectBody.FindJoint("connection_joint");
		if (num != -1)
		{
			jointFixed = node.ObjectBody.GetJoint(num) as JointFixed;
		}
	}
	private void Update()
	{
		if(Input.IsKeyDown(Input.KEY.C) && connectionCandidate != null)
		{
			jointFixed.Body1 = connectionCandidate;
			var itransform = MathLib.Inverse(connectionCandidate.Transform);
			var anchor_1_transform = itransform * connectionPoint.WorldTransform;
			jointFixed.Anchor1 = anchor_1_transform.Translate;
			jointFixed.Rotation1 = anchor_1_transform.GetRotate().Mat3;
			jointFixed.Enabled = true;
		}
		if (Input.IsKeyDown(Input.KEY.V))
		{
			jointFixed.Enabled = false;
		}
	}
	private void OnTriggerEnter(Body body)
	{
		if (!connectionCandidate)
			connectionCandidate = body as BodyRigid;
	}
	private void OnTriggerLeave(Body body)
	{
		if(connectionCandidate == (body as BodyRigid))
		{
			connectionCandidate = null;
			jointFixed.Enabled = false;
		}
	}
}
```
---
## RobotArmController.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "5e20f9d3c8d23de58480e7bd3ce7084517bdcc7a")]
public class RobotArmController : Component
{
	public struct ArmJoint
	{
		public Node armNode;
		public float speed;
		private JointHinge armJointHinge;
		public ArmJoint()
		{
			armNode = null;
			speed = 100.0f;
			armJointHinge = null;
		}
		public void init()
		{
			armJointHinge = armNode.ObjectBody.GetJoint(0) as JointHinge;
		}
		public void update(float ifps, Input.KEY positiveAxis, Input.KEY negativeAxis)
		{
			if (Input.IsKeyPressed(positiveAxis))
				rotate(speed * ifps);
			if (Input.IsKeyPressed(negativeAxis))
				rotate(-speed * ifps);
		}
		private void rotate(float angle)
		{
			if (armJointHinge)
			{
				float currentAngle = armJointHinge.AngularAngle;
				currentAngle += angle;
				if (currentAngle < -180)
					currentAngle += 360.0f;
				if (currentAngle > 180)
					currentAngle -= 360.0f;
				if (currentAngle < armJointHinge.AngularLimitFrom)
					currentAngle = armJointHinge.AngularLimitFrom;
				if (currentAngle > armJointHinge.AngularLimitTo)
					currentAngle = armJointHinge.AngularLimitTo;
				armJointHinge.AngularAngle = currentAngle;
			}
		}
	}
	public ArmJoint armJoint0 = new ArmJoint();
	public ArmJoint armJoint1 = new ArmJoint();
	public ArmJoint armJoint2 = new ArmJoint();
	public ArmJoint armJoint3 = new ArmJoint();
	public ArmJoint armJoint4 = new ArmJoint();
	public ArmJoint armJoint5 = new ArmJoint();
	void Init()
	{
		armJoint0.init();
		armJoint1.init();
		armJoint2.init();
		armJoint3.init();
		armJoint4.init();
		armJoint5.init();
	}
	void Update()
	{
		armJoint0.update(Game.IFps, Input.KEY.H, Input.KEY.F);
		armJoint1.update(Game.IFps, Input.KEY.T, Input.KEY.G);
		armJoint2.update(Game.IFps, Input.KEY.I, Input.KEY.K);
		armJoint3.update(Game.IFps, Input.KEY.J, Input.KEY.L);
		armJoint4.update(Game.IFps, Input.KEY.U, Input.KEY.O);
		armJoint5.update(Game.IFps, Input.KEY.R, Input.KEY.Y);
	}
}
```
---
## SavedPathTrajectory.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "b9943918b39397a29c8751ec6b1d21fb4398cc4b")]
public class SavedPathTrajectory : Component
{
	[ShowInEditor][ParameterFile]
	private string trajectoryFilePath = null;
	[ShowInEditor]
	private float velocity = 10.0f;
	public float Velocity { get { return velocity; } set { velocity = value; } }
	[ShowInEditor]
	private bool debug;
	public bool Debug { get { return debug; } set { debug = value; } }
	private WorldTransformPath transformPath;
	void Init()
	{
		node.WorldPosition = new Vec3(0, 0, 0);
		transformPath = new WorldTransformPath(trajectoryFilePath);
		transformPath.Loop = 1;
		transformPath.Time = 0.0f;
		transformPath.Speed = velocity;
		transformPath.Play();
		transformPath.AddChild(node);
	}
	void Update()
	{
		transformPath.Speed = velocity;
		if (debug)
			VisualizePath();
	}
	private void VisualizePath()
	{
		Path path = transformPath.Path;
		int num_frames = path.NumFrames;
		for (int i = 0; i < num_frames; i++)
		{
			Vec3 curr_point = path.GetFramePosition(i);
			Vec3 next_point = path.GetFramePosition((i + 1) % num_frames);
			Visualizer.RenderLine3D(curr_point, next_point, vec4.WHITE);
		}
	}
}
```
---
## ScriptArrays.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
public partial class InterpreterRegistrator
{
	private InterpreterRegistrator.InterpreterRegistratorAction my_array_action
		= new InterpreterRegistrator.InterpreterRegistratorAction(() =>
		{
			// export functions
			Interpreter.AddExternFunction("my_array_vector_set", new Interpreter.Function3(MyArray.my_array_vector_set), "[]");
			Interpreter.AddExternFunction("my_array_vector_get", new Interpreter.Function2v(MyArray.my_array_vector_get), "[]");
			Interpreter.AddExternFunction("my_array_map_set", new Interpreter.Function3(MyArray.my_array_map_set), "[]");
			Interpreter.AddExternFunction("my_array_map_get", new Interpreter.Function2v(MyArray.my_array_map_get), "[]");
			Interpreter.AddExternFunction("my_array_vector_generate", new Interpreter.Function1(MyArray.my_array_vector_generate), "[]");
			Interpreter.AddExternFunction("my_array_map_generate", new Interpreter.Function1(MyArray.my_array_map_generate), "[]");
			Interpreter.AddExternFunction("my_array_vector_enumerate", new Interpreter.Function1(MyArray.my_array_vector_enumerate), "[]");
		}
	);
}
public class MyArray
{
	private const string sourse_str = "From [C++]:";
	public static void my_array_vector_set(Variable id, Variable index, Variable val)
	{
		ArrayVector vector = ArrayVector.Get(Interpreter.Get(), id);
		vector.Set(index.Int, val);
	}
	public static Variable my_array_vector_get(Variable id, Variable index)
	{
		ArrayVector vector = ArrayVector.Get(Interpreter.Get(), id);
		return vector.Get(index.Int);
	}
	public static void my_array_map_set(Variable id, Variable key, Variable val)
	{
		ArrayMap map = ArrayMap.Get(Interpreter.Get(), id);
		map.Set(key, val);
	}
	public static Variable my_array_map_get(Variable id, Variable key)
	{
		ArrayMap map = ArrayMap.Get(Interpreter.Get(), id);
		return map.Get(key);
	}
	public static void my_array_vector_generate(Variable id)
	{
		ArrayVector vector = ArrayVector.Get(Interpreter.Get(), id);
		vector.Clear();
		for (int i = 0; i < 4; i++)
		{
			vector.Append(new Variable(i * i));
		}
		vector.Remove(0);
		vector.Append(new Variable("128"));
	}
	public static void my_array_map_generate(Variable id)
	{
		ArrayMap map = ArrayMap.Get(Interpreter.Get(), id);
		map.Clear();
		for (int i = 0; i < 4; i++)
		{
			map.Append(new Variable(i * i), new Variable(i * i));
		}
		map.Remove(new Variable(0));
		map.Append(new Variable(128), new Variable("128"));
	}
	public static void my_array_vector_enumerate(Variable id)
	{
		ArrayVector vector = ArrayVector.Get(Interpreter.Get(), id);
		for (int i = 0; i < vector.Size; i++)
		{
			Log.Message("{0} {1}: {2}\n", sourse_str, i, vector.Get(i).TypeInfo);
		}
	}
}
[Component(PropertyGuid = "e48b0c1950f63af408657bb35c6d0aed736bedc7")]
public class ScriptArrays : Component
{
	private float onscreenTime;
	void Init()
	{
		Unigine.Console.Onscreen = true;
		Unigine.Console.OnscreenFontSize = 15;
		Unigine.Console.OnscreenHeight = 100;
		onscreenTime = Unigine.Console.OnscreenTime;
		Unigine.Console.OnscreenTime = 1000;
	}
	void Shutdown()
	{
		Unigine.Console.Onscreen = false;
		Unigine.Console.OnscreenHeight = 30;
		Unigine.Console.OnscreenTime = onscreenTime;
	}
}
```
---
## ScriptCallback.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
public class MyCallback
{
	public const string sourse_str = "From [C++]:";
	public static Variable runWorldFunction(Variable name, Variable v)
	{
		Log.Message("{0} runWorldFunction({1},{2}): called\n", sourse_str, name.TypeName, v.TypeName);
		return Engine.RunWorldFunction(name, v);
	}
}
public partial class InterpreterRegistrator
{
	private InterpreterRegistrator.InterpreterRegistratorAction my_callback_action
		= new InterpreterRegistrator.InterpreterRegistratorAction(() =>
		{
			// export functions
			Interpreter.AddExternFunction("runWorldFunction", new Interpreter.Function2v(MyCallback.runWorldFunction));
		}
	);
}
[Component(PropertyGuid = "0619b8285b053d9ed75eeb42cc54c484aeb84189")]
public class ScriptCallback : Component
{
	private float onscreenTime;
	void Init()
	{
		Unigine.Console.Onscreen = true;
		Unigine.Console.OnscreenFontSize = 15;
		Unigine.Console.OnscreenHeight = 100;
		onscreenTime = Unigine.Console.OnscreenTime;
		Unigine.Console.OnscreenTime = 1000;
	}
	void Update()
	{
		Variable ret = Engine.RunWorldFunction(new Variable("counter"));
		if (ret.Int != -1)
			Log.Message("{0} counter is: {1}\n", MyCallback.sourse_str, ret.Int);
		if (ret.Int == 3)
			Log.Message("\n{0} world path is: \"{1}\"\n", MyCallback.sourse_str, Engine.RunWorldFunction(new Variable("engine.world.getPath")).String);
	}
	void Shutdown()
	{
		Unigine.Console.Onscreen = false;
		Unigine.Console.OnscreenHeight = 30;
		Unigine.Console.OnscreenTime = onscreenTime;
	}
}
```
---
## ScriptVariables.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
public partial class InterpreterRegistrator
{
	private InterpreterRegistrator.InterpreterRegistratorAction my_variable_action
		= new InterpreterRegistrator.InterpreterRegistratorAction(() =>
		{
			// export functions
			Interpreter.AddExternVariable("my_variable_int", 13);
			Interpreter.AddExternVariable("my_variable_float", 13.17f);
			Interpreter.AddExternVariable("my_variable_double", 13.17);
			Interpreter.AddExternVariable("my_variable_string", "13.17s");
			Interpreter.AddExternVariable("my_variable_vec3", new Variable(new vec3(13.0f, 17.0f, 137.0f)));
			Interpreter.AddExternVariable("my_variable_vec4", new Variable(new vec4(13.0f, 17.0f, 137.0f, 173.0f)));
		}
	);
}
[Component(PropertyGuid = "acf95e6aeb518c3dfe47d9943ed37d78894211b3")]
public class ScriptVariables : Component
{
	private float onscreenTime;
	void Init()
	{
		Unigine.Console.Onscreen = true;
		Unigine.Console.OnscreenFontSize = 15;
		Unigine.Console.OnscreenHeight = 100;
		onscreenTime = Unigine.Console.OnscreenTime;
		Unigine.Console.OnscreenTime = 1000;
	}
	void Shutdown()
	{
		Unigine.Console.Onscreen = false;
		Unigine.Console.OnscreenHeight = 30;
		Unigine.Console.OnscreenTime = onscreenTime;
	}
}
```
---
## SimpleTrajectoryMovement.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System.Collections.Generic;
using Unigine;
using System;
[Component(PropertyGuid = "e54b808ee042217251a38fc2f09ce89441d44429")]
public class SimpleTrajectoryMovement : Component
{
	[ShowInEditor]
	private Node pathNode = null;
	[ShowInEditor]
	private float velocity = 10.0f;
	public float Velocity { get { return velocity; } set { velocity = value; } }
	[ShowInEditor]
	private bool debug;
	public bool Debug { get { return debug; } set { debug = value; } }
	private List<Vec3> pointsPos = new List<Vec3>();
	private List<quat> pointsRot = new List<quat>();
	private Vec3 prevPoint = Vec3.ZERO;
	private quat prevRot = quat.IDENTITY;
	private int pointsIndex = 0;
	private float time = 0.0f;
	void Init()
	{
		int num_childs = pathNode.NumChildren;
		for (int i = 0; i < num_childs; i++)
		{
			Node nc = pathNode.GetChild(i);
			pointsPos.Add(nc.WorldPosition);
			pointsRot.Add(nc.GetWorldRotation());
		}
		prevPoint = node.WorldPosition;
	}
	void Update()
	{
		UpdateTime();
		node.WorldPosition = MathLib.Lerp(prevPoint, pointsPos[pointsIndex], time);
		node.SetWorldRotation(MathLib.Slerp(prevRot, pointsRot[pointsIndex], time));
		if (debug)
			VisualizePath();
	}
	private void VisualizePath()
	{
		for (int i = 0; i < pointsPos.Count; i++)
		{
			int next = (i + 1) % pointsPos.Count;
			Visualizer.RenderLine3D(pointsPos[i], pointsPos[next], vec4.WHITE);
		}
	}
	private void UpdateTime()
	{
		float len = (float)MathLib.Length(pointsPos[pointsIndex] - prevPoint);
		// update time with stable velocity
		time += velocity / len * Game.IFps;
		if (time >= 1.0f)
		{
			prevPoint = pointsPos[pointsIndex];
			prevRot = pointsRot[pointsIndex];
			// next point
			pointsIndex = (pointsIndex + Convert.ToInt32(time)) % pointsPos.Count;
			time = MathLib.Frac(time);
		}
	}
}
```
---
## SoundAmbient.cs
```csharp
﻿using System;
using Unigine;
[Component(PropertyGuid = "45d24605667f7fa0b4622e7f0e58e9d2399e7f09")]
public class SoundAmbient : Component
{
	[ParameterFile]
	public string soundFile = "";
	private AmbientSource ambientSource = null;
	private bool isStream = false;
	private SampleDescriptionWindow window = null;
	private void Init()
	{
		ambientSource = new AmbientSource(soundFile);
		if (!ambientSource)
			return;
		ambientSource.Loop = 1;
		ambientSource.Gain = 1.0f;
		ambientSource.Play();
		window = new SampleDescriptionWindow();
		window.createWindow();
		Widget parameters = window.getParameterGroupBox();
		WidgetButton button_play = new WidgetButton("Play");
		WidgetButton button_stop = new WidgetButton("Stop");
		button_play.EventClicked.Connect(() =>
		{
			ambientSource.Play();
		});
		button_stop.EventClicked.Connect(() =>
		{
			ambientSource.Stop();
		});
		WidgetHBox buttons = new WidgetHBox();
		buttons.SetSpace(10, 0);
		buttons.AddChild(button_play);
		buttons.AddChild(button_stop);
		parameters.AddChild(buttons, Gui.ALIGN_LEFT);
		window.addBoolParameter("Loop:", "Loop", Convert.ToBoolean(ambientSource.Loop), (bool active) =>
		{
			ambientSource.Loop = Convert.ToInt32(active);
		});
		window.addBoolParameter("Stream:", "Stream", isStream, (bool active) =>
		{
			ChangeSourceType();
		});
		window.addFloatParameter("Gain:", "Gain", ambientSource.Gain, 0.0f, 1.0f, (float val) =>
		{
			ambientSource.Gain = val;
		});
		window.addFloatParameter("Pitch:", "Pitch", ambientSource.Pitch, 0.1f, 5.0f, (float val) =>
		{
			ambientSource.Pitch = val;
		});
	}
	private void Update()
	{
	}
	private void Shutdown()
	{
		ambientSource.DeleteLater();
		window.shutdown();
	}
	private void ChangeSourceType()
	{
		int isLoop = ambientSource.Loop;
		bool isPlaying = ambientSource.IsPlaying;
		float gain = ambientSource.Gain;
		float pitch = ambientSource.Pitch;
		ambientSource.DeleteLater();
		ambientSource = null;
		isStream = !isStream;
		if (isStream)
			ambientSource = new AmbientSource(soundFile, 1);
		else
			ambientSource = new AmbientSource(soundFile, 0);
		ambientSource.Loop = isLoop;
		ambientSource.Gain = gain;
		ambientSource.Pitch = pitch;
		if (isPlaying)
			ambientSource.Play();
		else
			ambientSource.Stop();
	}
}
```
---
## SoundReverbController.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "871eb0829003b5f9cc716704dca0c693e4129ad6")]
public class SoundReverbController : Component
{
	public SoundSource soundSource = null;
	private SoundReverb reverb = null;
	private float reverbPower = 0.5f;
	SampleDescriptionWindow window = null;
	private void Init()
	{
		reverb = new SoundReverb(new vec3(20.0f, 20.0f, 20.0f));
		reverb.WorldTransform = Mat4.IDENTITY;
		reverb.Threshold = new vec3(10.0f, 10.0f, 10.0f);
		UpdateReverbSettings();
		Visualizer.Enabled = true;
		window = new SampleDescriptionWindow();
		window.createWindow();
		window.addFloatParameter("Gain:", "Gain", reverbPower, 0.0f, 1.0f, (float val) =>
		{
			reverbPower = val;
			UpdateReverbSettings();
		});
	}
	private void Update()
	{
		if (!reverb || !soundSource)
			return;
		reverb.RenderVisualizer();
		soundSource.RenderVisualizer();
	}
	private void Shutdown()
	{
		Visualizer.Enabled = false;
		window.shutdown();
	}
	private void UpdateReverbSettings()
	{
		reverb.Density = MathLib.Clamp(1.0f - reverbPower, 0.0f, 1.0f);
		reverb.Diffusion = MathLib.Clamp(1.0f - reverbPower, 0.0f, 1.0f);
		reverb.DecayTime = MathLib.Clamp(0.1f + 19.9f * reverbPower, 0.1f, 20.0f);
		reverb.ReflectionGain = MathLib.Clamp(3.16f * reverbPower, 0.0f, 2.16f);
		reverb.LateReverbGain = MathLib.Clamp(10.0f * reverbPower, 0.0f, 10.0f);
	}
}
```
---
## SoundSourceController.cs
```csharp
﻿using System;
using Unigine;
[Component(PropertyGuid = "5f06580f304e6622e66daf19bd35cda45d1dbcb3")]
public class SoundSourceController : Component
{
	[ParameterFile]
	public string soundFile = "";
	private SoundSource sound = null;
	SampleDescriptionWindow window = null;
	private void Init()
	{
		sound = new SoundSource(soundFile);
		if (!sound)
			return;
		sound.MinDistance = 5.0f;
		sound.MaxDistance = 50.0f;
		sound.Loop = 1;
		sound.Play();
		Visualizer.Enabled = true;
		window = new SampleDescriptionWindow();
		window.createWindow();
		Widget parameters = window.getParameterGroupBox();
		WidgetButton button_play = new WidgetButton("Play");
		WidgetButton button_stop = new WidgetButton("Stop");
		button_play.EventClicked.Connect(() =>
		{
			sound.Play();
		});
		button_stop.EventClicked.Connect(() =>
		{
			sound.Stop();
		});
		WidgetHBox buttons = new WidgetHBox();
		buttons.SetSpace(10, 0);
		buttons.AddChild(button_play);
		buttons.AddChild(button_stop);
		parameters.AddChild(buttons, Gui.ALIGN_LEFT);
		window.addBoolParameter("Loop:", "Loop", Convert.ToBoolean(sound.Loop), (bool active) =>
		{
			sound.Loop = Convert.ToInt32(active);
		});
		window.addBoolParameter("Stream:", "Stream", sound.Stream, (bool active) =>
		{
			sound.Stream = active;
		});
		window.addFloatParameter("Gain:", "Gain", sound.Gain, 0.0f, 1.0f, (float val) =>
		{
			sound.Gain = val;
		});
		window.addFloatParameter("Pitch:", "Pitch", sound.Pitch, 0.1f, 5.0f, (float val) =>
		{
			sound.Pitch = val;
		});
	}
	private void Update()
	{
		sound.RenderVisualizer();
	}
	private void Shutdown()
	{
		Visualizer.Enabled = false;
		window.shutdown();
	}
}
```
---
## SpectatorController.cs
```csharp
﻿using System.Collections.Generic;
using System.Diagnostics;
using Unigine;
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Mat4 = Unigine.mat4;
#endif
[Component(PropertyGuid = "4feac9fb387f583654c7454fba0ef83faa3f39dd")]
public class SpectatorController : Component
{
	[ShowInEditor, Parameter(Group="Input", Title="Controlled", Tooltip="Toggle spectator inputs")] 
	public bool isControlled = true; 
	[ShowInEditor, Parameter(Group="Input", Title="Forward Key", Tooltip="Move forward key")] 
	public string forwardKey = "W";
	[ShowInEditor, Parameter(Group="Input", Title="Left Key", Tooltip="Move left key")] 
	public string leftKey = "A";
	[ShowInEditor, Parameter(Group="Input", Title="Backward Key", Tooltip="Move backward key")] 
	public string backwardKey = "S";
	[ShowInEditor, Parameter(Group="Input", Title="Right Key", Tooltip="Move right key")] 
	public string rightKey = "D";
	[ShowInEditor, Parameter(Group="Input", Title="Down key", Tooltip="Move down key")]  
	public string downKey = "Q";
	[ShowInEditor, Parameter(Group="Input", Title="Up key", Tooltip="Move up key")] 
	public string upKey = "E";
	[ShowInEditor, Parameter(Group="Input", Title="Turn up key", Tooltip="Turn up")] 
	public string turnUpKey = "UP";
	[ShowInEditor, Parameter(Group="Input", Title="Turn down key", Tooltip="Turn down")] 
	public string turnDownKey = "DOWN";
	[ShowInEditor, Parameter(Group="Input", Title="Turn left key", Tooltip="Turn left")] 
	public string turnLeftKey = "LEFT";	
	[ShowInEditor, Parameter(Group="Input", Title="Turn right key", Tooltip="Turn right")] 
	public string turnRightKey = "RIGHT";
	[ShowInEditor, Parameter(Group="Input", Title="Sprint Key", Tooltip="Sprint mode activation key")] 
	public string accelerateKey = "ANY_SHIFT";
	[ShowInEditor, Parameter(Group="Input", Title="Mouse Sensitivity", Tooltip="Mouse sensitivity multiplier")] 
	public float mouseSensitivity = 0.4f;
	[ShowInEditor, Parameter(Group="Movement", Title="Velocity", Tooltip="Movement velocity")]
	public float velocity = 2.0f;
	[ShowInEditor, Parameter(Group="Movement", Title="Sprint velocity", Tooltip="Sprint movement velocity")]
	public float sprintVelocity = 4.0f;
	[ShowInEditor, Parameter(Group="Movement", Title="Acceleration", Tooltip="Movement acceleration")]
	public float acceleration = 4.0f;
	[ShowInEditor, Parameter(Group="Movement", Title="Damping", Tooltip="Movement damping")]
	public float damping = 8.0f;
	[ShowInEditor, Parameter(Group="Collision", Title="Collision", Tooltip="Toggle spectator collision")]
	public bool isCollided = true;
	[ShowInEditor, ParameterMask(Group="Collision", MaskType = ParameterMaskAttribute.TYPE.COLLISION, Title="Collision Mask", Tooltip="A bit mask of collisions")]
	public int mask = 1;
	[ShowInEditor, Parameter(Group="Collision", Title="Collision radius", Tooltip="The radius of the collision sphere")]
	public float collisionRadius = 0.5f;
	[ShowInEditor, Parameter(Group="Rotation", Title="Turning speed", Tooltip="Velocity of the spectator's turning action")]
	public float turning = 90f;
	[ShowInEditor, Parameter(Group="Rotation", Title="Min theta angle", Tooltip="Minimun pitch angle")]
	public float minThetaAngle = -89.9f;
	[ShowInEditor, Parameter(Group="Rotation", Title="Max theta angle", Tooltip="Maximum pitch angle")]
	public float maxThetaAngle = 89.9f;
	private const float PLAYER_SPECTATOR_IFPS = 1.0f / 60.0f;
	private const int PLAYER_SPECTATOR_COLLISIONS = 4;
	private Vec3 _position = Vec3.ZERO;
	private Unigine.vec3 _direction = Unigine.vec3.ZERO;
	private float _phiAngle = 0;
	private float _thetaAngle = 0;
	private Vec3 _lastPosition = 0;
	private Unigine.vec3 _lastDirection = 0;
	private Unigine.vec3 _lastUp = 0;
	private Mat4 transform;
	private List<ShapeContact> _contacts = [];
	// These properties provide convenient access to the specific key bindings 
	// for movement and camera control in the application. 
	// Each property retrieves the corresponding Input.KEY value using a key name string.
	// This design allows key mappings to be configured via string identifiers, 
	// making the input system more flexible and customizable.
	private Input.KEY ForwardKey => Input.GetKeyByName(forwardKey);
	private Input.KEY BackwardKey => Input.GetKeyByName(backwardKey);
	private Input.KEY RightKey => Input.GetKeyByName(rightKey);
	private Input.KEY LeftKey => Input.GetKeyByName(leftKey);
	private Input.KEY UpKey => Input.GetKeyByName(upKey);
	private Input.KEY DownKey => Input.GetKeyByName(downKey);
	private Input.KEY TurnUpKey => Input.GetKeyByName(turnUpKey);
	private Input.KEY TurnDownKey => Input.GetKeyByName(turnDownKey);
	private Input.KEY TurnLeftKey => Input.GetKeyByName(turnLeftKey);
	private Input.KEY TurnRightKey => Input.GetKeyByName(turnRightKey);
	private Input.KEY AccelerateKey => Input.GetKeyByName(accelerateKey);
	private ShapeSphere _shapeSphere = null;
	private Player _player = null;
	// These methods are used to control the player's view direction 
	// externally from this class. They provide access to and allow manipulation of 
	// the camera's orientation
	public void SetPhiAngle(float phiAngle)
	{
		phiAngle -= _phiAngle;
		_direction = new quat(_player.Up, phiAngle) * _direction;
		_phiAngle += phiAngle;
		FlushTransform();
	}
	public void SetThetaAngle(float thetaAngle)
	{
		thetaAngle = MathLib.Clamp(thetaAngle, minThetaAngle, maxThetaAngle) - _thetaAngle;
		_direction = new quat(MathLib.Cross(_player.Up, _direction), thetaAngle) * _direction;
		_thetaAngle += thetaAngle;
		FlushTransform(); 
	}
	public void SetViewDirection(vec3 direction)
	{
		_direction = MathLib.Normalize(direction);
		vec3 tangent = vec3.ZERO;
		vec3 binormal = vec3.ZERO;
		MathLib.OrthoBasis(_player.Up,out tangent,out binormal);
		_phiAngle = MathLib.Atan2(MathLib.Dot(_direction, tangent), MathLib.Dot(_direction, binormal)) * MathLib.RAD2DEG;
		_thetaAngle = MathLib.Acos(MathLib.Clamp(MathLib.Dot(_direction, _player.Up), -1.0f, 1.0f)) * MathLib.RAD2DEG - 90.0f;
		_thetaAngle = MathLib.Clamp(_thetaAngle,minThetaAngle,maxThetaAngle);
		FlushTransform();
	}
	public float GetPhiAngle() =>_phiAngle;
	public float GetThetaAngle() => _thetaAngle;
	public Unigine.vec3 GetViewdirection() => _direction;
	// This block contains methods for querying detailed information about collision contacts.
	// These accessors provide data such as contact points, normals, depth of penetration, and 
	// the involved objects and surfaces. Useful for physics responses.
	public int GetNumContacts() => _contacts.Count;
	public ShapeContact GetContact(int num)
	{
		Debug.Assert(num >= 0 && num < GetNumContacts(), "SpectatorController.GetContact(): bad contact number");
		return _contacts[num];
	}
	public float GetContactDepth(int num)
	{
		Debug.Assert(num >= 0 && num < GetNumContacts(), "SpectatorController.GetContact(): bad contact number");
		return _contacts[num].Depth;
	}
	public Unigine.vec3 GetContactNormal(int num)
	{
		Debug.Assert(num >= 0 && num < GetNumContacts(), "SpectatorController.GetContact(): bad contact number");
		return _contacts[num].Normal;
	}
	public Unigine.Object GetContactObject(int num)
	{
		Debug.Assert(num >= 0 && num < GetNumContacts(), "SpectatorController.GetContact(): bad contact number");
		return _contacts[num].Object;
	}
	public Vec3 GetContactPoint(int num)
	{
		Debug.Assert(num >= 0 && num < GetNumContacts(), "SpectatorController.GetContact(): bad contact number");
		return _contacts[num].Point;
	}
	public Shape GetContactShape(int num)
	{
		Debug.Assert(num >= 0 && num < GetNumContacts(), "SpectatorController.GetContact(): bad contact number");
		return _contacts[num].Shape0;
	}
	public int GetContactSurface(int num)
	{
		Debug.Assert(num >= 0 && num < GetNumContacts(), "SpectatorController.GetContact(): bad contact number");
		return _contacts[num].Surface;
	}
	// UpdateControls handles user input processing and triggers movement updates.
	// This method reads control states (mouse and buttons) and passes 
	// the relevant data to UpdateMovement for velocity computation.
	public void UpdateControls()
	{
		Unigine.vec3 up = _player.Up;
		Unigine.vec3 impulse = Unigine.vec3.ZERO;
		Unigine.vec3 tangent = Unigine.vec3.ZERO;
		Unigine.vec3 binormal = Unigine.vec3.ZERO;
		MathLib.OrthoBasis(_player.Up, out tangent, out binormal);
		if (isControlled && !Unigine.Console.Active)
		{
			if (Input.MouseCursorHide)
			{
				_phiAngle += Input.MouseDeltaPosition.x * mouseSensitivity;
				_thetaAngle += Input.MouseDeltaPosition.y * mouseSensitivity;
			}
			_thetaAngle += turning * Game.IFps * (MathLib.ToInt(Input.IsKeyPressed(TurnDownKey)) - MathLib.ToInt(Input.IsKeyPressed(TurnUpKey)));
			_thetaAngle = MathLib.Clamp(_thetaAngle, minThetaAngle, maxThetaAngle);
			_phiAngle += turning * Game.IFps * (MathLib.ToInt(Input.IsKeyPressed(TurnRightKey)) - MathLib.ToInt(Input.IsKeyPressed(TurnLeftKey)));
			Unigine.vec3 x = (new quat (up, -_phiAngle) * new quat(tangent, -_thetaAngle)) * binormal;
			Unigine.vec3 y = MathLib.Normalize(MathLib.Cross(up, x));
			Unigine.vec3 z = MathLib.Normalize(MathLib.Cross(x,y));
			_direction = x;
			impulse += x * (MathLib.ToInt(Input.IsKeyPressed(ForwardKey)) - MathLib.ToInt(Input.IsKeyPressed(BackwardKey)));
			impulse += y * (MathLib.ToInt(Input.IsKeyPressed(LeftKey)) - MathLib.ToInt(Input.IsKeyPressed(RightKey)));
			impulse += z * (MathLib.ToInt(Input.IsKeyPressed(UpKey)) - MathLib.ToInt(Input.IsKeyPressed(DownKey)));
			if (impulse != vec3.ZERO)
				impulse.Normalize();
			impulse *= Input.IsKeyPressed(AccelerateKey) ? sprintVelocity : velocity;
		}
		float time = Game.IFps;
		float targetVelocity = MathLib.Length(impulse);
		Vec3 playerVelocity = _player.Velocity;
		// Use do-while to ensure at least one update is processed,
		// even when the remaining time is very small (e.g., at high FPS).
		do
		{
			// Clamp the simulation step to a maximum fixed time interval (PLAYER_SPECTATOR_IFPS).
    		// This prevents instability or large jumps in movement and collisions when frame time is high (e.g., during frame drops). 
			float ifps = MathLib.Min(time, PLAYER_SPECTATOR_IFPS); 
			time -= ifps;
			UpdateMovement(impulse, ifps, targetVelocity);
		} while (time > MathLib.EPSILON);
	}
	// Applies the final transformation matrix to the scene Node based on 
	// the updated movement and orientation values computed in the previous steps.
	public void FlushTransform()
	{
		Unigine.vec3 up = _player.Up;
		if (_lastPosition != _position || _lastDirection != _direction || _lastUp != up)
		{
			node.WorldTransform = MathLib.SetTo(_position,_position + (Vec3)_direction, up);
			OnTransformChanged(); // update all internal params
			_lastPosition = _position;
			_lastDirection = _direction;
			_lastUp = up;
		}
	}
	// Syncs this component's internal state with the transform of the node.
	private void OnTransformChanged()
	{	
		Unigine.vec3 up = _player.Up;
		Unigine.vec3 tangent = Unigine.vec3.ZERO; 
		Unigine.vec3 binormal = Unigine.vec3.ZERO;
		MathLib.OrthoBasis(up,out tangent, out binormal);
		_position = node.WorldTransform.GetColumn3(3);
		_direction = MathLib.Normalize(new Unigine.vec3(-node.WorldTransform.GetColumn3(2)));
		_phiAngle = MathLib.Atan2(MathLib.Dot(_direction, tangent), MathLib.Dot(_direction, binormal)) * MathLib.RAD2DEG;
		_thetaAngle = MathLib.Acos(MathLib.Clamp(MathLib.Dot(_direction, _player.Up), -1.0f, 1.0f)) * MathLib.RAD2DEG - 90.0f;
		_thetaAngle = MathLib.Clamp(_thetaAngle,minThetaAngle,maxThetaAngle);
		_direction = new quat(up, -_phiAngle) * new quat(tangent, -_thetaAngle) * binormal;
		_lastPosition = _position;
		_lastDirection = _direction;
		_lastUp = up;
	}
	// Calculates the player's current velocity and adjusts camera position 
	// if collisions are detected. Called after input is processed in UpdateControls.
	private void UpdateMovement(Unigine.vec3 impulse, float ifps, float targetVelocity)
	{
		float oldVelocity = MathLib.Length(_player.Velocity);
		_player.Velocity += impulse * acceleration * ifps;
		float currentVelocity = MathLib.Length(_player.Velocity);
		if (targetVelocity < MathLib.EPSILON || currentVelocity > targetVelocity)
			_player.Velocity *= MathLib.Exp(-damping * ifps);
		currentVelocity = MathLib.Length(_player.Velocity);
		if(currentVelocity > oldVelocity && currentVelocity > targetVelocity)
			_player.Velocity *= targetVelocity / currentVelocity;
		if(currentVelocity < MathLib.EPSILON)
			_player.Velocity = Unigine.vec3.ZERO;
		_position += _player.Velocity * ifps;
		_contacts.Clear();
		if (_player.Enabled && isCollided)
		{
			for (int i = 0; i < PLAYER_SPECTATOR_COLLISIONS; i++)
			{
				_shapeSphere.Center = _position;
				_shapeSphere.GetCollision(_contacts, ifps); 
				if (_contacts.Count == 0)
					break;
				// Calculate the inverse of the number of contacts to evenly distribute the total push-out
				// This prevents applying the full depth for every contact, which would overcompensate the position
				float inversContacts = 1.0f / MathLib.ToFloat(_contacts.Count); 
				for (int j = 0; j < _contacts.Count; j++)
				{
					ShapeContact contact = _contacts[j];
					// Push the player out along the contact normal, scaled by penetration depth and evenly divided by contact count
					_position += new Vec3(contact.Normal * (contact.Depth * inversContacts));
					// Remove the velocity component that's directed into the contact surface
    				// This prevents the player from continuing to move into the object
					_player.Velocity -= contact.Normal * MathLib.Dot(_player.Velocity, contact.Normal);  
				}
			}
		}
		_shapeSphere.Center = _position;
	}
	private void Init()
	{
		_shapeSphere = new ShapeSphere(collisionRadius);
		_shapeSphere.Continuous = false;
		_player = node as Player;
		OnTransformChanged();
	}
	private void Update()
	{
		if (!transform.Equals(node.Transform)) // if somebody change our position outside -> update all internal params.
			OnTransformChanged();
		UpdateControls();
		FlushTransform();
		transform = node.Transform;
	}
	private void Shutdown()
	{
		_shapeSphere.DeleteLater();
	}
}
```
---
## SpectatorControllerSample.cs
```csharp
﻿using Unigine;
using System.Globalization;
[Component(PropertyGuid = "539a342ac6b92616947fd05d13b99e043485856a")]
public class SpectatorControllerSample : Component
{
	private SampleDescriptionWindow _sampleDescriptionWindow = new();
	private Input.MOUSE_HANDLE _mouseHandler = Input.MOUSE_HANDLE.USER;
	private bool _isControlled = false;
	private bool _isCollided = false;
	private float _minMouseSensetivity = 0.1f;
	private float _maxMouseSensetivity = 1.0f;
	private float _currentMouseSensetivity = 0.0f;
	private float _minTurning = 15.0f;
	private float _maxTurning = 120.0f;
	private float _currentTurning = 0.0f;
	private float _minVelocity = 1.0f;
	private float _maxVelocity = 4.0f;
	private float _currentVelocity = 0.0f;
	private float _minSprintVelocity = 5.0f;
	private float _maxSprintVelocity = 10.0f;
	private float _currentSprintVelocity = 0.0f;
	private SpectatorController _controller;
	private void Init()
	{
		_mouseHandler = Input.MouseHandle;
		Input.MouseHandle = Input.MOUSE_HANDLE.GRAB;
		_controller = ComponentSystem.GetComponent<SpectatorController>(Game.Player);
		_isControlled = _controller.isControlled;
		_isCollided = _controller.isCollided;
		_currentMouseSensetivity = _controller.mouseSensitivity;
		_currentTurning = _controller.turning;
		_currentVelocity = _controller.velocity;
		_currentSprintVelocity = _controller.sprintVelocity;
		_sampleDescriptionWindow.createWindow();
		_sampleDescriptionWindow.addFloatParameter(
			"Mouse sensetivity",
			"Сontrols the mouse sensetivity",
			_currentMouseSensetivity,
			_minMouseSensetivity,
			_maxMouseSensetivity,
			(float v) =>
			{
				_currentMouseSensetivity = v;
				_controller.mouseSensitivity = _currentMouseSensetivity;
			});
		_sampleDescriptionWindow.addFloatParameter(
		"Turning speed",
		"Controls the turning speed using the arrow keys.",
		_currentTurning,
		_minTurning,
		_maxTurning,
		(float v) =>
		{
			_currentTurning = v;
			_controller.turning = _currentTurning;
		});
		_sampleDescriptionWindow.addFloatParameter(
		"Velocity",
		"Controls the base velocity",
		_currentVelocity,
		_minVelocity,
		_maxVelocity,
		(float v) =>
		{
			_currentVelocity = v;
			_controller.velocity = _currentVelocity;
		});
		_sampleDescriptionWindow.addFloatParameter(
			"Sprint velocity",
			"Controls the sprint velocity",
			_currentSprintVelocity,
			_minSprintVelocity,
			_maxSprintVelocity,
			(float v) =>
			{
				_currentSprintVelocity = v;
				_controller.sprintVelocity = _currentSprintVelocity;
			});
		_sampleDescriptionWindow.addBoolParameter(
			"Set camera receive inputs",
			"Controls whether the camera receives input.",
			_isControlled,
			(bool k) =>
			{
				_isControlled = k;
				_controller.isControlled = _isControlled;
			});
		_sampleDescriptionWindow.addBoolParameter(
			"Set camera collision",
			"Controls whether the camera collision enabled",
			_isControlled,
			(bool k) =>
			{
				_isCollided = k;
				_controller.isCollided = _isCollided;
			});
	}
	private void Shutdown()
	{
		_sampleDescriptionWindow.shutdown();
		Input.MouseHandle = _mouseHandler;
	}
}
```
---
## Spinner.cs
```csharp
﻿using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "4a3bf23b8bc0c25e1dd84625c5fa41405f63c39f")]
public class Spinner : Component
{
	[Parameter(Title = "Angular Speed", Tooltip = "Rotation speed, in degrees per second", Group = "General")]
	public float rotationSpeed = 50.0f;
	[ParameterFile(Filter = ".node")]
	public string bulletAsset = "";
	public List<Node> spawnPoints = null;
	public float spawnDelay = 2.0f;
	private float spawnTimer = 0.0f;
	private void Init()
	{
		spawnTimer = spawnDelay;
	}
	private void Update()
	{
		node.Rotate(0, 0, rotationSpeed * Game.IFps);
		if (spawnPoints == null)
			return;
		spawnTimer -= Game.IFps;
		if (spawnTimer < 0)
		{
			spawnTimer = spawnDelay;
			foreach (Node point in spawnPoints)
			{
				Node spawnedBullet = World.LoadNode(bulletAsset);
				spawnedBullet.WorldTransform = point.WorldTransform;
			}
		}
	}
}
```
---
## SplineTrajectoryMovement.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System.Collections.Generic;
using Unigine;
using System.Linq;
using static Unigine.Animations;
[Component(PropertyGuid = "24bbd2ad0b15daff35490849acddb59617ca6fe8")]
public class SplineTrajectoryMovement : Component
{
	[ShowInEditor]
	private Node pathNode = null;
	[ShowInEditor]
	private float velocity = 10.0f;
	public float Velocity { get { return velocity; } set { velocity = value; } }
	[ShowInEditor]
	private int quality = 25;
	[ShowInEditor]
	private bool debug;
	public bool Debug { get { return debug; } set { debug = value; } }
	private List<List<float>> lengths = new List<List<float>>();
	private List<Vec3> pointsPos = new List<Vec3>();
	private List<quat> pointsRot = new List<quat>();
	private int pointsIndex = 0;
	private float time = 0.0f;
	void Init()
	{
		// save positions and rotations
		int numChilds = pathNode.NumChildren;
		for (int i = 0; i < numChilds; i++)
		{
			Node nc = pathNode.GetChild(i);
			pointsPos.Add(nc.WorldPosition);
			pointsRot.Add(nc.GetWorldRotation());
		}
		int pointsCount = pointsPos.Count;
		for (int j = 0; j < pointsCount; j++)
		{
			int j_prev = (j - 1 < 0) ? (pointsCount - 1) : j - 1;
			int j_cur = j;
			int j_next = (j + 1) % pointsCount;
			int j_next_next = (j + 2) % pointsCount;
			Vec3 p0 = pointsPos[j_prev];
			Vec3 p1 = pointsPos[j_cur];
			Vec3 p2 = pointsPos[j_next];
			Vec3 p3 = pointsPos[j_next_next];
			lengths.Add(Utils.GetLengthCatmullRomCentripetal(p0, p1, p2, p3, quality));
		}
	}
	void Update()
	{
		float speed = velocity / (lengths[pointsIndex][(int)(time * (quality - 1))] * quality);
		UpdateTime(speed);
		Vec3[] p = GetCurrentPoints();
		quat[] q = GetCurrentQuats();
		Vec3 pos = Utils.CatmullRomCentripetal(p[0], p[1], p[2], p[3], time);
		quat rot = Utils.Squad(q[0], q[1], q[2], q[3], time);
		node.WorldPosition = pos;
		node.SetWorldRotation(rot, true);
		if (debug)
			VisualizePath();
	}
	private void VisualizePath()
	{
		int points_count = pointsPos.Count;
		for (int j = 0; j < points_count; j++)
		{
			int j_prev = (j - 1 < 0) ? (points_count - 1) : j - 1;
			int j_cur = j;
			int j_next = (j + 1) % points_count;
			int j_next_next = (j + 2) % points_count;
			Vec3 p0 = pointsPos[j_prev];
			Vec3 p1 = pointsPos[j_cur];
			Vec3 p2 = pointsPos[j_next];
			Vec3 p3 = pointsPos[j_next_next];
			// draw curve
			Vec3 start = Utils.CatmullRomCentripetal(p0, p1, p2, p3, 0);
			int quality = 10;
			for (int i = 1; i < quality; i++)
			{
				Vec3 end = Utils.CatmullRomCentripetal(p0, p1, p2, p3, (float)i / (quality - 1));
				Visualizer.RenderLine3D(start, end, vec4.WHITE);
				start = end;
			}
		}
	}
	private void UpdateTime(float speed)
	{
		time += speed * Game.IFps;
		if (time >= 1.0f)
		{
			pointsIndex = (pointsIndex + (int)time) % pointsPos.Count; // loop
			time = MathLib.Frac(time);
		}
	}
	private Vec3[] GetCurrentPoints()
	{
		int points_count = pointsPos.Count;
		int i_prev = (pointsIndex - 1 < 0) ? (points_count - 1) : pointsIndex - 1;
		int i_cur = pointsIndex;
		int i_next = (pointsIndex + 1) % points_count;
		int i_next_next = (pointsIndex + 2) % points_count;
		Vec3[] result = new Vec3[4];
		result[0] = pointsPos[i_prev];
		result[1] = pointsPos[i_cur];
		result[2] = pointsPos[i_next];
		result[3] = pointsPos[i_next_next];
		return result;
	}
	private quat[] GetCurrentQuats()
	{
		int points_count = pointsPos.Count;
		int i_prev = (pointsIndex - 1 < 0) ? (points_count - 1) : pointsIndex - 1;
		int i_cur = pointsIndex;
		int i_next = (pointsIndex + 1) % points_count;
		int i_next_next = (pointsIndex + 2) % points_count;
		quat[] result = new quat[4];
		result[0] = pointsRot[i_prev];
		result[1] = pointsRot[i_cur];
		result[2] = pointsRot[i_next];
		result[3] = pointsRot[i_next_next];
		return result;
	}
}
```
---
## SunController.cs
```csharp
﻿using Unigine;
using static Unigine.MathLib;
#region Math Variables
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Mat4 = Unigine.mat4;
#endif
#endregion
[Component(PropertyGuid = "e9f7147ab66bfdcf2382a8eac3bb09a65652d53d")]
public class SunController : Component
{
	public const string COMPONENT_DESCRIPTION = "This component controls sun based on its own game time and various adjustable parameters";
	[ShowInEditor]
	[Parameter(Title = "Is sun moving continuously")]
	public bool isContinuous = true;
	public bool IsContinuous { get { return isContinuous; } set { isContinuous = value; } }
	[ShowInEditor]
	[Parameter(Title = "Scale of continuous time rotation")]
	private float timescale = 2000.0f;
	public float Timescale { get { return timescale; } set { timescale = value; } }
	private quat sunInitTilt = quat.IDENTITY;
	private double currentTime = 720 * 60; // in secons
	/// <summary>
	/// Returns current time in seconds
	/// </summary>
	/// <returns></returns>
	public double Time { get { return currentTime; } }
	private const int maxTimeSec = 60 * 60 * 24; // 60 minutes in 24 hours
	public int MaxTimeSec => maxTimeSec;
	private EventInvoker<double> timeChangedEvent = new EventInvoker<double>();
	public Event<double> EventOnTimeChanged
	{
		get
		{
			return timeChangedEvent;
		}
	}
	public void SetTime(double time)
	{
		currentTime = time;
		RefreshSunPostion();
		timeChangedEvent.Run((int)currentTime);
	}
	private void Init()
	{
		sunInitTilt = node.GetWorldRotation();
	}
	private void Update()
	{
		if (IsContinuous)
		{
			currentTime += Game.IFps * Timescale;
			if (currentTime > maxTimeSec)
				currentTime -= maxTimeSec;//so we wont loose delta time 
			RefreshSunPostion();
			timeChangedEvent.Run(currentTime); // displaying only integer part
		}
	}
	private void RefreshSunPostion()
	{
		//converting time to rotation
		double time = currentTime % maxTimeSec;
		double k = MathLib.InverseLerp(0.0, maxTimeSec, time);
		float angle = (float)MathLib.Lerp(-180, 180.0, k);
		node.SetWorldRotation(sunInitTilt * new quat(angle, 0.0f, 0.0f));
	}
}
```
---
## TargetGui.cs
```csharp
﻿using System.Globalization;
using Unigine;
[Component(PropertyGuid = "89571ccf605ba9481e3600165449e4341baa6d50")]
public class TargetGui : Component
{
	private ObjectGui objectGui;
	private Gui gui;
	private WidgetLabel distanceLabel;
	void Init()
	{
		objectGui = node as ObjectGui;
		if (!objectGui)
		{
			Log.Error("TargetGui.Init is not ObjectGui!\n");
		}
		gui = objectGui.GetGui();
		WidgetWindow window = new WidgetWindow();
		gui.AddChild(window, Gui.ALIGN_CENTER);
		WidgetVBox vbox = new WidgetVBox();
		vbox.SetSpace(1, 3);
		window.AddChild(vbox, Gui.ALIGN_CENTER);
		window.Width = gui.Width;
		window.Height = gui.Height;
		WidgetLabel targetLabel = new WidgetLabel(node.Name);
		targetLabel.FontSize = 50;
		vbox.AddChild(targetLabel);
		double distance = (node.WorldPosition - Game.Player.WorldPosition).Length;
		distanceLabel = new WidgetLabel("Distance to label : " + distance.ToString("0.00", CultureInfo.InvariantCulture) + " units");
		distanceLabel.FontSize = 50;
		vbox.AddChild(distanceLabel);
		WidgetLabel sizeLabel = new WidgetLabel("Size of label : " + objectGui.PhysicalWidth + "x" + objectGui.PhysicalHeight);
		sizeLabel.FontSize = 50;
		vbox.AddChild(sizeLabel);
	}
	void Update()
	{
		double distance = (node.WorldPosition - Game.Player.WorldPosition).Length;
		distanceLabel.Text = "Distance to label : " + distance.ToString("0.00", CultureInfo.InvariantCulture) + " units";
	}
}
```
---
## Toggleable.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "a7d5d2a62572dd16887cc1793473380ff98c0197")]
public abstract class Toggleable : Component
{
	[ShowInEditor]
	private bool isToggled = false;
	public bool Toggled
	{
		get => isToggled;
		set
		{
			if (value != isToggled)
			{
				bool ok = value ? On() : Off();
				isToggled = isToggled ^ ok;
			}
		}
	}
	public bool Toggle() => isToggled = isToggled ^ (isToggled ? Off() : On());
	protected abstract bool On();
	protected abstract bool Off();
}
```
---
## Toggler.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "725d757e1636d8f8c2c1da6f63cf62b3ae34b5a4")]
public class Toggler : Component
{
	[ParameterMask(MaskType = ParameterMaskAttribute.TYPE.INTERSECTION)]
	public int interaction_intersection_mask = 1 << 16;
	private void Update()
	{
		if (!Input.IsMouseButtonDown(Input.MOUSE_BUTTON.LEFT))
			return;
		ivec2 mouse = Input.MousePosition;
		Player player = (Player) node;
		if (player == null)
			return;
		vec3 direction = player.GetDirectionFromMainWindow(mouse.x, mouse.y);
		vec3 p0 = new vec3(player.WorldPosition);
		vec3 p1 = p0 + direction * player.ZFar;
		Object obj = World.GetIntersection(p0, p1, interaction_intersection_mask);
		if (obj)
		{
			var toggleable = obj.GetComponent<Toggleable>();
			if (toggleable)
			{
				toggleable.Toggle();
			}
		}
	}
	private void Init()
	{
		consoleOnScreen = Console.Onscreen;
		Console.Onscreen = true;
	}
	private void Shutdown()
	{
		Console.Onscreen = consoleOnScreen;
	}
	private bool consoleOnScreen = false;
}
```
---
## TrackPlayback.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "5d3fa800bd9a7b5f76b611327c69bee9549f7b34")]
public class TrackPlayback : Component
{
	[ShowInEditor]
	[ParameterFile(Filter = ".track")]
	private string scaleTrackPath = "";
	private int positionTrackID = -1;
	private int scaleTrackID = -1;
	private float positionTrackTime = 0.0f;
	private float rotationTrackTime = 0.0f;
	private float scaleTrackTime = 0.0f;
	private void Init()
	{
		if (!Tracker.IsInitialized)
			return;
		// get position track id that was added to tracker in editor
		if (Tracker.ContainsTrack("position_track"))
		{
			positionTrackID = Tracker.GetTrackID("position_track");
			positionTrackTime = Tracker.GetMinTime(positionTrackID);
		}
		// get rotation track time using track name
		if (Tracker.ContainsTrack("rotation_track"))
			rotationTrackTime = Tracker.GetMinTime("rotation_track");
		// add new track to tracker
		scaleTrackID = Tracker.AddTrack(scaleTrackPath);
		if (scaleTrackID != -1)
			scaleTrackTime = Tracker.GetMinTime(scaleTrackID);
	}
	private void Update()
	{
		if (!Tracker.IsInitialized)
			return;
		// set position track time using id
		if (positionTrackID != -1)
		{
			float minTime = Tracker.GetMinTime(positionTrackID);
			float maxTime = Tracker.GetMaxTime(positionTrackID);
			float unitTime = Tracker.GetUnitTime(positionTrackID);
			positionTrackTime += Game.IFps / unitTime;
			if (positionTrackTime >= maxTime)
				positionTrackTime = minTime;
			Tracker.SetTime(positionTrackID, positionTrackTime);
		}
		// set rotation track time using track name
		if (Tracker.ContainsTrack("rotation_track"))
		{
			float minTime = Tracker.GetMinTime("rotation_track");
			float maxTime = Tracker.GetMaxTime("rotation_track");
			float unitTime = Tracker.GetUnitTime("rotation_track");
			rotationTrackTime += Game.IFps / unitTime;
			if (rotationTrackTime >= maxTime)
				rotationTrackTime = minTime;
			Tracker.SetTime("rotation_track", rotationTrackTime);
		}
		// update scale track time
		if (scaleTrackID != -1)
		{
			float minTime = Tracker.GetMinTime(scaleTrackID);
			float maxTime = Tracker.GetMaxTime(scaleTrackID);
			float unitTime = Tracker.GetUnitTime(scaleTrackID);
			scaleTrackTime += Game.IFps / unitTime;
			if (scaleTrackTime >= maxTime)
				scaleTrackTime = minTime;
			Tracker.SetTime(scaleTrackID, scaleTrackTime);
		}
	}
}
```
---
## Tracker.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "65c47ed0144043ef811aa082136ee5f9de02d704")]
public class Tracker : Component
{
	[ShowInEditor]
	[ParameterFile]
	private string trackerWrapperUsc = "";
	[ShowInEditor]
	[ParameterFile(Filter = ".track")]
	private List<string> trackFiles = new List<string>();
	private static bool isWrapperLoaded = false;
	private static readonly Variable initFunc = new Variable("TrackerWrapper::init");
	private static readonly Variable shutdownFunc = new Variable("TrackerWrapper::shutdown");
	private static readonly Variable addTrackFunc = new Variable("TrackerWrapper::addTrack");
	private static readonly Variable removeTrackFunc = new Variable("TrackerWrapper::removeTrack");
	private static readonly Variable getMinTimeFunc = new Variable("TrackerWrapper::getMinTime");
	private static readonly Variable getMaxTimeFunc = new Variable("TrackerWrapper::getMaxTime");
	private static readonly Variable getUnitTimeFunc = new Variable("TrackerWrapper::getUnitTime");
	private static readonly Variable setFunc = new Variable("TrackerWrapper::set");
	private static Dictionary<string, int> trackIDs = new Dictionary<string, int>();
	private static Variable trackFileVar = new Variable("");
	private static Variable trackIdVar = new Variable(0);
	private static Variable timeVar = new Variable(0.0f);
	public static bool IsInitialized { get; private set; } = false;
	public static int AddTrack(string trackFile)
	{
		if (IsInitialized && FileSystem.IsFileExist(trackFile) && FileSystem.GetExtension(trackFile) == "track")
		{
			string trackName = trackFile.Split('.')[0];
			string[] parts = trackName.Split('/');
			trackName = parts[parts.Length - 1];
			if (!trackIDs.ContainsKey(trackName))
			{
				trackFileVar.String = trackFile;
				int trackID = Engine.RunWorldFunction(addTrackFunc, trackFileVar).Int;
				trackIDs[trackName] = trackID;
				return trackID;
			}
			else
			{
				Log.Warning($"Tracker::AddTrack: {trackFile} already added\n");
				return trackIDs[trackName];
			}
		}
		return -1;
	}
	public static void RemoveTrack(int trackID)
	{
		if (IsInitialized && trackIDs.ContainsValue(trackID))
		{
			trackIdVar.Int = trackID;
			Engine.RunWorldFunction(removeTrackFunc, trackIdVar);
			string itemToRemove = "";
			foreach (var pair in trackIDs)
			{
				if (pair.Value.Equals(trackID))
					itemToRemove = pair.Key;
			}
			trackIDs.Remove(itemToRemove);
		}
	}
	public static void RemoveTrack(string trackName)
	{
		if (IsInitialized && trackIDs.ContainsKey(trackName))
			RemoveTrack(trackIDs[trackName]);
	}
	public static bool ContainsTrack(string trackName)
	{
		if (IsInitialized)
			return trackIDs.ContainsKey(trackName);
		return false;
	}
	public static int GetTrackID(string trackName)
	{
		if (IsInitialized && trackIDs.ContainsKey(trackName))
			return trackIDs[trackName];
		return -1;
	}
	public static float GetMinTime(int trackID)
	{
		if (IsInitialized)
		{
			trackIdVar.Int = trackID;
			return Engine.RunWorldFunction(getMinTimeFunc, trackIdVar).Float;
		}
		return 0.0f;
	}
	public static float GetMinTime(string trackName)
	{
		if (IsInitialized && trackIDs.ContainsKey(trackName))
			return GetMinTime(trackIDs[trackName]);
		return 0.0f;
	}
	public static float GetMaxTime(int trackID)
	{
		if (IsInitialized)
		{
			trackIdVar.Int = trackID;
			return Engine.RunWorldFunction(getMaxTimeFunc, trackIdVar).Float;
		}
		return 0.0f;
	}
	public static float GetMaxTime(string trackName)
	{
		if (IsInitialized && trackIDs.ContainsKey(trackName))
			return GetMaxTime(trackIDs[trackName]);
		return 0.0f;
	}
	public static float GetUnitTime(int trackID)
	{
		if (IsInitialized)
		{
			trackIdVar.Int = trackID;
			return Engine.RunWorldFunction(getUnitTimeFunc, trackIdVar).Float;
		}
		return 0.0f;
	}
	public static float GetUnitTime(string trackName)
	{
		if (IsInitialized && trackIDs.ContainsKey(trackName))
			return GetUnitTime(trackIDs[trackName]);
		return 0.0f;
	}
	public static void SetTime(int trackID, float time)
	{
		if (IsInitialized)
		{
			timeVar.Float = time;
			trackIdVar.Int = trackID;
			Engine.RunWorldFunction(setFunc, trackIdVar, timeVar);
		}
	}
	public static void SetTime(string trackName, float time)
	{
		if (IsInitialized && trackIDs.ContainsKey(trackName))
			SetTime(trackIDs[trackName], time);
	}
	// initialize tracker before all components
	[MethodInit(Order = int.MinValue)]
	private void Init()
	{
		// check tracker wrapper
		string trackerWrapperGUID = FileSystem.GuidToPath(FileSystem.GetGUID(trackerWrapperUsc));
		isWrapperLoaded = (World.ScriptName == trackerWrapperGUID);
		// add tracker_wrapper to world logic if not set in editor
		if (!isWrapperLoaded)
		{
			Log.WarningLine($"{trackerWrapperUsc} not setup as world script. Setup it in Editor.");
			return;
		}
		// init tracker in wrapper
		Engine.RunWorldFunction(initFunc);
		IsInitialized = true;
		// add tracks
		if (trackFiles != null)
		{
			foreach (var track in trackFiles)
				AddTrack(track);
		}
	}
	// shutdown tracker after all components
	[MethodShutdown(Order = int.MaxValue)]
	private void Shutdown()
	{
		if (IsInitialized)
		{
			// shutdown tracker in wrapper
			Engine.RunWorldFunction(shutdownFunc);
			trackIDs.Clear();
			isWrapperLoaded = false;
			IsInitialized = false;
		}
	}
}
```
---
## TrajectoryLogic.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "3dbf3dd482e5e570fb4168689d535522a7a0fc17")]
public class TrajectoryLogic : Component
{
	[ShowInEditor]
	private Node airplane1 = null;
	[ShowInEditor]
	private Node airplane2 = null;
	[ShowInEditor]
	private Node airplane3 = null;
	private WidgetCheckBox enableVisualizePath;
	private enum Players
	{
		MAIN = 0,
		ONE,
		TWO,
		THREE,
		TOTAL_PLAYERS
	}
	private List<Player> mainPlayers = new List<Player>();
	Players currentPlayer = Players.MAIN;
	SampleDescriptionWindow sampleDescriptionWindow = new SampleDescriptionWindow();
	void Init()
	{
		Game.GetMainPlayers(mainPlayers);
		Game.Player = mainPlayers[(int)currentPlayer];
		InitGui();
		Visualizer.Enabled = true;
	}
	void Shutdown()
	{
		sampleDescriptionWindow.shutdown();
		Visualizer.Enabled = false;
	}
	private void InitGui()
	{
		sampleDescriptionWindow.createWindow();
		sampleDescriptionWindow.addFloatParameter("Velocity", "Velicity", 5.0f, 1.0f, 50.0f, (float v) => {
			GetComponent<SimpleTrajectoryMovement>(airplane1).Velocity = v;
			GetComponent<SplineTrajectoryMovement>(airplane2).Velocity = v;
			GetComponent<SavedPathTrajectory>(airplane3).Velocity = v;
		});
		WidgetGroupBox parameters = sampleDescriptionWindow.getParameterGroupBox();
		var changeCameraButton = new WidgetButton();
		parameters.AddChild(changeCameraButton, Gui.ALIGN_LEFT);
		changeCameraButton.Text = "Switch Camera";
		changeCameraButton.EventClicked.Connect(SwitchTrajectoryCallback);
		var gridbox = new WidgetGridBox();
		parameters.AddChild(gridbox, Gui.ALIGN_LEFT);
		enableVisualizePath = new WidgetCheckBox();
		enableVisualizePath.Checked = false;
		enableVisualizePath.EventClicked.Connect(EnableVisualizeCallback);
		var visualizeLabel = new WidgetLabel("Visualize Path");
		gridbox.AddChild(visualizeLabel);
		gridbox.AddChild(enableVisualizePath);
	}
	private void SwitchTrajectoryCallback()
	{
		currentPlayer = (Players)(((int)currentPlayer + 1) % (int)Players.TOTAL_PLAYERS);
		Game.Player = mainPlayers[(int)currentPlayer];
	}
	private void EnableVisualizeCallback()
	{
		bool enabled = enableVisualizePath.Checked;
		GetComponent<SimpleTrajectoryMovement>(airplane1).Debug = enabled;
		GetComponent<SplineTrajectoryMovement>(airplane2).Debug = enabled;
		GetComponent<SavedPathTrajectory>(airplane3).Debug = enabled;
	}
}
```
---
## TransformEulerAngles.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using System;
using Unigine;
[Component(PropertyGuid = "20cf275447d36aa388934fa6a4dc22cdb77f65c8")]
public class TransformEulerAngles : Component
{
	public enum COMPOSITION_TYPE
	{
		XYZ = 0,
		XZY,
		YXZ,
		YZX,
		ZXY,
		ZYX,
		COUNT
	}
	private vec3 EulerAngles = vec3.ZERO;
	private vec3 DecompositionAngles = vec3.ZERO;
	private COMPOSITION_TYPE compositionType = COMPOSITION_TYPE.XYZ;
	private COMPOSITION_TYPE decompositionType = COMPOSITION_TYPE.XYZ;
	private bool visualizerEnabled = false;
	private SampleDescriptionWindow sampleDescriptionWindow;
	string status = null;
	private void Init()
	{
		initGui();
		UpdateDecompositionAngles();
		visualizerEnabled = Visualizer.Enabled;
		Visualizer.Enabled = true;
	}
	private void Update()
	{
		Vec3 planePos = node.WorldPosition;
		vec3 planeX = node.GetWorldDirection(MathLib.AXIS.X);
		vec3 planeY = node.GetWorldDirection(MathLib.AXIS.Y);
		vec3 planeZ = node.GetWorldDirection(MathLib.AXIS.Z);
		// render local axes of plane
		Visualizer.RenderVector(planePos, planePos + planeX * 1.5f, vec4.RED);
		Visualizer.RenderVector(planePos, planePos + planeY * 1.5f, vec4.GREEN);
		Visualizer.RenderVector(planePos, planePos + planeZ * 1.5f, vec4.BLUE);
		Visualizer.RenderMessage3D(planePos + planeX * 1.5f, vec3.ZERO, "X", vec4.BLACK, 1);
		Visualizer.RenderMessage3D(planePos + planeY * 1.5f, vec3.ZERO, "Y", vec4.BLACK, 1);
		Visualizer.RenderMessage3D(planePos + planeZ * 1.5f, vec3.ZERO, "Z", vec4.BLACK, 1);
		// render global axes
		Vec3 start = new Vec3(-2.0f, -2.0f, 0.2f);
		Visualizer.RenderVector(start, start + vec3.RIGHT * 2.0f, vec4.RED);
		Visualizer.RenderVector(start, start + vec3.FORWARD * 2.0f, vec4.GREEN);
		Visualizer.RenderVector(start, start + vec3.UP * 2.0f, vec4.BLUE);
		// render circles
		mat4 x = mat4.IDENTITY;
		mat4 y = mat4.IDENTITY;
		mat4 z = mat4.IDENTITY;
		vec3 radii = vec3.ZERO;
		GetCircleMatrix(out x, out y, out z, out radii);
		Visualizer.RenderCircle(radii.x, new Mat4(x), vec4.RED);
		Visualizer.RenderCircle(radii.y, new Mat4(y), vec4.GREEN);
		Visualizer.RenderCircle(radii.z, new Mat4(z), vec4.BLUE);
		// render circle arrows
		Vec3 arrowStart = planePos + new vec3(x.AxisY) * radii.x;
		Vec3 arrowFinish = arrowStart + new vec3(x.AxisY) * 0.1f;
		Visualizer.RenderVector(arrowStart, arrowFinish, vec4.RED, 1.0f);
		arrowStart = planePos + new vec3(y.AxisY) * radii.y;
		arrowFinish = arrowStart + new vec3(y.AxisY) * 0.1f;
		Visualizer.RenderVector(arrowStart, arrowFinish, vec4.GREEN, 1.0f);
		arrowStart = planePos + new vec3(z.AxisY) * radii.z;
		arrowFinish = arrowStart + new vec3(z.AxisY) * 0.1f;
		Visualizer.RenderVector(arrowStart, arrowFinish, vec4.BLUE, 1.0f);
	}
	private void Shutdown()
	{
		Visualizer.Enabled = visualizerEnabled;
		sampleDescriptionWindow.shutdown();
	}
	private void initGui()
	{
		sampleDescriptionWindow = new SampleDescriptionWindow();
		sampleDescriptionWindow.createWindow();
		var pitch_slider = sampleDescriptionWindow.addIntParameter("Pitch (X)", "Pitch (X)", 0, -180, 180, (int value) =>
		{
			EulerAngles.x = value;
			UpdateRotation();
			UpdateDecompositionAngles();
		});
		var roll_slider = sampleDescriptionWindow.addIntParameter("Roll (Y)", "Roll (Y)", 0, -180, 180, (int value) =>
		{
			EulerAngles.y = value;
			UpdateRotation();
			UpdateDecompositionAngles();
		});
		var yaw_slider = sampleDescriptionWindow.addIntParameter("Yaw (Z)", "Yaw (Z)", 0, -180, 180, (int value) =>
		{
			EulerAngles.z = value;
			UpdateRotation();
			UpdateDecompositionAngles();
		});
		var parameters = sampleDescriptionWindow.getParameterGroupBox();
		WidgetComboBox composition_combo_box;
		WidgetComboBox decomposition_combo_box;
		{
			var hbox = new WidgetHBox();
			hbox.AddChild(new WidgetLabel("Composition sequence: "), Gui.ALIGN_LEFT);
			var combo_box = new WidgetComboBox();
			combo_box.AddItem("XYZ");
			combo_box.AddItem("XZY");
			combo_box.AddItem("YXZ");
			combo_box.AddItem("YZX");
			combo_box.AddItem("ZXY");
			combo_box.AddItem("ZYX");
			combo_box.EventChanged.Connect(() =>
			{
				compositionType = (COMPOSITION_TYPE)combo_box.CurrentItem;
				UpdateRotation();
			});
			composition_combo_box = combo_box;
			hbox.AddChild(combo_box);
			parameters.AddChild(hbox, Gui.ALIGN_LEFT);
		}
		{
			var hbox = new WidgetHBox();
			hbox.AddChild(new WidgetLabel("Decomposition sequence: "), Gui.ALIGN_LEFT);
			var combo_box = new WidgetComboBox();
			combo_box.AddItem("XYZ");
			combo_box.AddItem("XZY");
			combo_box.AddItem("YXZ");
			combo_box.AddItem("YZX");
			combo_box.AddItem("ZXY");
			combo_box.AddItem("ZYX");
			combo_box.EventChanged.Connect(() =>
			{
				decompositionType = (COMPOSITION_TYPE)combo_box.CurrentItem;
				UpdateDecompositionAngles();
			});
			decomposition_combo_box = combo_box;
			hbox.AddChild(combo_box);
			parameters.AddChild(hbox, Gui.ALIGN_LEFT);
		}
		var reset_button = new WidgetButton("Reset");
		reset_button.EventClicked.Connect(() =>
		{
			EulerAngles = vec3.ZERO;
			UpdateRotation();
			UpdateDecompositionAngles();
			yaw_slider.Value = 0;
			pitch_slider.Value = 0;
			roll_slider.Value = 0;
			composition_combo_box.CurrentItem = 0;
			decomposition_combo_box.CurrentItem = 0;
		});
		parameters.AddChild(reset_button, Gui.ALIGN_LEFT);
	}
	private void UpdateRotation()
	{
		// get new rotation based on current composition type
		mat4 rot = mat4.IDENTITY;
		switch (compositionType)
		{
			case COMPOSITION_TYPE.XYZ: rot = MathLib.ComposeRotationXYZ(EulerAngles); break;
			case COMPOSITION_TYPE.XZY: rot = MathLib.ComposeRotationXZY(EulerAngles); break;
			case COMPOSITION_TYPE.YXZ: rot = MathLib.ComposeRotationYXZ(EulerAngles); break;
			case COMPOSITION_TYPE.YZX: rot = MathLib.ComposeRotationYZX(EulerAngles); break;
			case COMPOSITION_TYPE.ZXY: rot = MathLib.ComposeRotationZXY(EulerAngles); break;
			case COMPOSITION_TYPE.ZYX: rot = MathLib.ComposeRotationZYX(EulerAngles); break;
		}
		node.SetWorldRotation(new quat(rot));
	}
	private void UpdateDecompositionAngles()
	{
		// update decomposition angles based on current decomposition type
		mat3 rot = node.GetWorldRotation().Mat3;
		switch (decompositionType)
		{
			case COMPOSITION_TYPE.XYZ: DecompositionAngles = MathLib.DecomposeRotationXYZ(rot); break;
			case COMPOSITION_TYPE.XZY: DecompositionAngles = MathLib.DecomposeRotationXZY(rot); break;
			case COMPOSITION_TYPE.YXZ: DecompositionAngles = MathLib.DecomposeRotationYXZ(rot); break;
			case COMPOSITION_TYPE.YZX: DecompositionAngles = MathLib.DecomposeRotationYZX(rot); break;
			case COMPOSITION_TYPE.ZXY: DecompositionAngles = MathLib.DecomposeRotationZXY(rot); break;
			case COMPOSITION_TYPE.ZYX: DecompositionAngles = MathLib.DecomposeRotationZYX(rot); break;
		}
		status = String.Format($"Decomposition angles:\nPitch (X):\t{DecompositionAngles.x:0.00}\nRoll (Y):\t{DecompositionAngles.y:0.00}\nYaw (Z):\t{DecompositionAngles.z:0.00}\n");
		sampleDescriptionWindow.setStatus(status);
	}
	private void GetCircleMatrix(out mat4 xMat, out mat4 yMat, out mat4 zMat, out vec3 radii)
	{
		float x = EulerAngles.x;
		float y = EulerAngles.y;
		float z = EulerAngles.z;
		float bigRadius = 1.4f;
		float middleRadius = 1.3f;
		float smallRadius = 1.2f;
		xMat = MathLib.Rotate(new quat(x, 0.0f, 0.0f));
		yMat = MathLib.Rotate(new quat(x, y, 0.0f));
		zMat = MathLib.Rotate(new quat(x, y, z));
		radii = new vec3(1.4f, 1.3f, 1.2f);
		switch (compositionType)
		{
			case COMPOSITION_TYPE.XYZ:
				xMat = MathLib.ComposeRotationXYZ(new vec3(x, 0.0f, 0.0f));
				yMat = MathLib.ComposeRotationXYZ(new vec3(x, y, 0.0f));
				zMat = MathLib.ComposeRotationXYZ(new vec3(x, y, z));
				radii = new vec3(bigRadius, middleRadius, smallRadius);
				break;
			case COMPOSITION_TYPE.XZY:
				xMat = MathLib.ComposeRotationXZY(new vec3(x, 0.0f, 0.0f));
				zMat = MathLib.ComposeRotationXZY(new vec3(x, 0.0f, z));
				yMat = MathLib.ComposeRotationXZY(new vec3(x, y, z));
				radii = new vec3(bigRadius, smallRadius, middleRadius);
				break;
			case COMPOSITION_TYPE.YXZ:
				yMat = MathLib.ComposeRotationYXZ(new vec3(0.0f, y, 0.0f));
				xMat = MathLib.ComposeRotationYXZ(new vec3(x, y, 0.0f));
				zMat = MathLib.ComposeRotationYXZ(new vec3(x, y, z));
				radii = new vec3(middleRadius, bigRadius, smallRadius);
				break;
			case COMPOSITION_TYPE.YZX:
				yMat = MathLib.ComposeRotationYZX(new vec3(0.0f, y, 0.0f));
				zMat = MathLib.ComposeRotationYZX(new vec3(0.0f, y, z));
				xMat = MathLib.ComposeRotationYZX(new vec3(x, y, z));
				radii = new vec3(smallRadius, bigRadius, middleRadius);
				break;
			case COMPOSITION_TYPE.ZXY:
				zMat = MathLib.ComposeRotationZXY(new vec3(0.0f, 0.0f, z));
				xMat = MathLib.ComposeRotationZXY(new vec3(x, 0.0f, z));
				yMat = MathLib.ComposeRotationZXY(new vec3(x, y, z));
				radii = new vec3(middleRadius, smallRadius, bigRadius);
				break;
			case COMPOSITION_TYPE.ZYX:
				zMat = MathLib.ComposeRotationZYX(new vec3(0.0f, 0.0f, z));
				yMat = MathLib.ComposeRotationZYX(new vec3(0.0f, y, z));
				xMat = MathLib.ComposeRotationZYX(new vec3(x, y, z));
				radii = new vec3(smallRadius, middleRadius, bigRadius);
				break;
		}
		mat4 tr = new mat4(MathLib.Translate(node.WorldPosition));
		xMat = tr * xMat * MathLib.Rotate(new quat(0, 90, 0));
		yMat = tr * yMat * MathLib.Rotate(new quat(90, 0, 0));
		zMat = tr * zMat;
	}
}
```
---
## TransformRotate.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "2c37e3b1fc47095c43c1150ba9b266eefcbf0175")]
public class TransformRotate : Component
{
	public vec3 angularVelocity = new vec3(0.0f, 0.0f, 45.0f);
	private void Update()
	{
		node.Rotate(angularVelocity * Game.IFps);
	}
}
```
---
## TransformTranslate.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "ee05af9cbced064be24d272e4b7444b51a638e90")]
public class TransformTranslate : Component
{
	public dvec3 linearVelocity = dvec3.FORWARD;
	private float time = 0.0f;
	private float timeSign = 1.0f;
	private void Update()
	{
		if (time < -3.0f || time > 3.0f)
			timeSign *= -1.0f;
		time += Game.IFps * timeSign;
		node.Translate(new Vec3(linearVelocity * Game.IFps * timeSign));
	}
}
```
---
## TransformWorldRotate.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "8b0e57758d6f0250dfcdf16f3bed7002d2506f79")]
public class TransformWorldRotate : Component
{
	public vec3 angularVelocity = new vec3(0.0f, 0.0f, 45.0f);
	private void Update()
	{
		vec3 offset = angularVelocity * Game.IFps;
		node.WorldRotate(offset.x, offset.y, offset.z);
	}
}
```
---
## TransformWorldTranslate.cs
```csharp
﻿#region Math Variables
#if UNIGINE_DOUBLE
using Scalar = System.Double;
using Vec2 = Unigine.dvec2;
using Vec3 = Unigine.dvec3;
using Vec4 = Unigine.dvec4;
using Mat4 = Unigine.dmat4;
#else
using Scalar = System.Single;
using Vec2 = Unigine.vec2;
using Vec3 = Unigine.vec3;
using Vec4 = Unigine.vec4;
using Mat4 = Unigine.mat4;
using WorldBoundBox = Unigine.BoundBox;
using WorldBoundSphere = Unigine.BoundSphere;
using WorldBoundFrustum = Unigine.BoundFrustum;
#endif
#endregion
using Unigine;
[Component(PropertyGuid = "57238fcedfa754d58c882030c0dbb5cf533ea8e2")]
public class TransformWorldTranslate : Component
{
	public dvec3 linearVelocity = dvec3.FORWARD;
	private float time = 0.0f;
	private float timeSign = 1.0f;
	private void Update()
	{
		if (time < -3.0f || time > 3.0f)
			timeSign *= -1.0f;
		time += Game.IFps * timeSign;
		node.WorldTranslate(new Vec3(linearVelocity * Game.IFps * timeSign));
	}
}
```
---
## TriggerSample.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "c59b7f924043770709e88a625e90acd2ad16ebc6")]
public class TriggerSample : Component
{
	[ShowInEditor]
	private Node targetToCheck = null;
	[ShowInEditor]
	private Node postamentPhysicsSphere = null;
	[ShowInEditor]
	private Node postamentPhysicsCapsule = null;
	[ShowInEditor]
	private Node postamentPhysicsCylinder = null;
	[ShowInEditor]
	private Node postamentPhysicsBox = null;
	[ShowInEditor]
	private Node postamentWorld = null;
	[ShowInEditor]
	private Node postamentMathSphere = null;
	[ShowInEditor]
	private Node postamentMathBox = null;
	[ShowInEditor]
	private Node postamentIntersectionSphere = null;
	[ShowInEditor]
	private Node postamentIntersectionBox = null;
	[ShowInEditor]
	PhysicalTrigger physicalTriggerSphere = null;
	[ShowInEditor]
	PhysicalTrigger physicalTriggerCapsule = null;
	[ShowInEditor]
	PhysicalTrigger physicalTriggerCylinder = null;
	[ShowInEditor]
	PhysicalTrigger physicalTriggerBox = null;
	[ShowInEditor]
	WorldTrigger worldTrigger = null;
	[ShowInEditor]
	private MathTriggerComponent mathTriggerSphere = null;
	[ShowInEditor]
	private MathTriggerComponent mathTriggerBox = null;
	[ShowInEditor]
	private IntersectionTriggerComponent intersectionTriggerSphere = null;
	[ShowInEditor]
	private IntersectionTriggerComponent intersectionTriggerBox = null;
	[ShowInEditor]
	private NodeTrigger nodeTrigger = null;
	[ShowInEditor]
	private Node triggerNodeParentNode = null;
	[ShowInEditor]
	private Node triggerNodeText = null;
	[ShowInEditor]
	private Material postamentMat = null;
	[ShowInEditor]
	private Material postamentMatTriggered = null;
	SampleDescriptionWindow sampleDescriptionWindow = new SampleDescriptionWindow();
	Visualizer.MODE visualizerMode;
	void Init()
	{
		Visualizer.Enabled = true;
		visualizerMode = Visualizer.Mode;
		Visualizer.Mode = Visualizer.MODE.ENABLED_DEPTH_TEST_ENABLED;
		// setting callbacks
		mathTriggerSphere.AddObject(targetToCheck);
		mathTriggerBox.AddObject(targetToCheck);
		physicalTriggerSphere.EventEnter.Connect(() =>
		{
			ObjectMeshStatic obj = postamentPhysicsSphere as ObjectMeshStatic;
			obj.SetMaterial(postamentMatTriggered, 0);
		});
		physicalTriggerSphere.EventLeave.Connect(() =>
		{
			ObjectMeshStatic obj = postamentPhysicsSphere as ObjectMeshStatic;
			obj.SetMaterial(postamentMat, 0);
		});
		physicalTriggerCapsule.EventEnter.Connect(() =>
		{
			ObjectMeshStatic obj = postamentPhysicsCapsule as ObjectMeshStatic;
			obj.SetMaterial(postamentMatTriggered, 0);
		});
		physicalTriggerCapsule.EventLeave.Connect(() =>
		{
			ObjectMeshStatic obj = postamentPhysicsCapsule as ObjectMeshStatic;
			obj.SetMaterial(postamentMat, 0);
		});
		physicalTriggerCylinder.EventEnter.Connect(() =>
		{
			ObjectMeshStatic obj = postamentPhysicsCylinder as ObjectMeshStatic;
			obj.SetMaterial(postamentMatTriggered, 0);
		});
		physicalTriggerCylinder.EventLeave.Connect(() =>
		{
			ObjectMeshStatic obj = postamentPhysicsCylinder as ObjectMeshStatic;
			obj.SetMaterial(postamentMat, 0);
		});
		physicalTriggerBox.EventEnter.Connect(() =>
		{
			ObjectMeshStatic obj = postamentPhysicsBox as ObjectMeshStatic;
			obj.SetMaterial(postamentMatTriggered, 0);
		});
		physicalTriggerBox.EventLeave.Connect(() =>
		{
			ObjectMeshStatic obj = postamentPhysicsBox as ObjectMeshStatic;
			obj.SetMaterial(postamentMat, 0);
		});
		worldTrigger.EventEnter.Connect(() =>
		{
			ObjectMeshStatic obj = postamentWorld as ObjectMeshStatic;
			obj.SetMaterial(postamentMatTriggered, 0);
		});
		worldTrigger.EventLeave.Connect(() =>
		{
			ObjectMeshStatic obj = postamentWorld as ObjectMeshStatic;
			obj.SetMaterial(postamentMat, 0);
		});
		mathTriggerSphere.EventEnter.Connect(() =>
		{
			ObjectMeshStatic obj = postamentMathSphere as ObjectMeshStatic;
			obj.SetMaterial(postamentMatTriggered, 0);
		});
		mathTriggerSphere.EventLeave.Connect(() =>
		{
			ObjectMeshStatic obj = postamentMathSphere as ObjectMeshStatic;
			obj.SetMaterial(postamentMat, 0);
		});
		mathTriggerBox.EventEnter.Connect(() =>
		{
			ObjectMeshStatic obj = postamentMathBox as ObjectMeshStatic;
			obj.SetMaterial(postamentMatTriggered, 0);
		});
		mathTriggerBox.EventLeave.Connect(() =>
		{
			ObjectMeshStatic obj = postamentMathBox as ObjectMeshStatic;
			obj.SetMaterial(postamentMat, 0);
		});
		intersectionTriggerSphere.EventEnter.Connect((Node node_trigger) =>
		{
			Unigine.Object obj = node_trigger as Unigine.Object;
			if (obj && (obj.GetIntersectionMask(0) == intersectionTriggerSphere.MaterialBallIntersectionMask))
			{
				ObjectMeshStatic postament = postamentIntersectionSphere as ObjectMeshStatic;
				postament.SetMaterial(postamentMatTriggered, 0);
			}
		});
		intersectionTriggerSphere.EventLeave.Connect((Node node_trigger) =>
		{
			Unigine.Object obj = node_trigger as Unigine.Object;
			if (obj && (obj.GetIntersectionMask(0) == intersectionTriggerSphere.MaterialBallIntersectionMask))
			{
				ObjectMeshStatic postament = postamentIntersectionSphere as ObjectMeshStatic;
				postament.SetMaterial(postamentMat, 0);
			}
		});
		intersectionTriggerBox.EventEnter.Connect((Node node_trigger) =>
		{
			Unigine.Object obj = node_trigger as Unigine.Object;
			if (obj && (obj.GetIntersectionMask(0) == intersectionTriggerSphere.MaterialBallIntersectionMask))
			{
				ObjectMeshStatic postament = postamentIntersectionBox as ObjectMeshStatic;
				postament.SetMaterial(postamentMatTriggered, 0);
			}
		});
		intersectionTriggerBox.EventLeave.Connect((Node node_trigger) =>
		{
			Unigine.Object obj = node_trigger as Unigine.Object;
			if (obj && (obj.GetIntersectionMask(0) == intersectionTriggerSphere.MaterialBallIntersectionMask))
			{
				ObjectMeshStatic postament = postamentIntersectionBox as ObjectMeshStatic;
				postament.SetMaterial(postamentMat, 0);
			}
		});
		nodeTrigger.EventEnabled.Connect((NodeTrigger trigger) =>
		{
			var  objectText = triggerNodeText as ObjectText;
			if (trigger.Enabled)
				objectText.TextColor = vec4.WHITE;
			else
				objectText.TextColor = vec4.RED;
		});
		nodeTrigger.EventPosition.Connect((NodeTrigger trigger) =>
		{
			Unigine.Object parent = trigger.Parent as Unigine.Object;
			Material material = parent.GetMaterialInherit(0);
			vec4 color = material.GetParameterFloat4("albedo_color");
			color.z += Game.IFps;
			if (color.z > 1.0f)
				color.z = 0.0f;
			material.SetParameterFloat4("albedo_color", color);
		});
		sampleDescriptionWindow.createWindow();
		WidgetGroupBox parameters = sampleDescriptionWindow.getParameterGroupBox();
		var nodeTriggerCheckbox = new WidgetCheckBox("Cube Active");
		parameters.AddChild(nodeTriggerCheckbox, Gui.ALIGN_LEFT);
		nodeTriggerCheckbox.EventChanged.Connect(() =>
		{
			triggerNodeParentNode.Enabled = nodeTriggerCheckbox.Checked;
		});
		nodeTriggerCheckbox.Checked = true;
	}
	void Update()
	{
		Visualizer.RenderBoundBox(worldTrigger.BoundBox, worldTrigger.WorldTransform, vec4.RED);
		physicalTriggerSphere.RenderVisualizer();
		physicalTriggerCapsule.RenderVisualizer();
		physicalTriggerCylinder.RenderVisualizer();
		physicalTriggerBox.RenderVisualizer();
	}
	void Shutdown()
	{
		Visualizer.Mode = visualizerMode;
		Visualizer.Enabled = false;
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## UpdatePhysicsUsageController.cs
```csharp
﻿using Unigine;
#region Math Variables
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Mat4 = Unigine.mat4;
#endif
#endregion
[Component(PropertyGuid = "de217aaf7595a5baab09a9b2f5bcb45b4870c8ee")]
public class UpdatePhysicsUsageController : Component
{
	[ShowInEditor]
	[Parameter(Title = "Use update function")]
	private bool useUdpate=false;
	[ShowInEditor]
	[Parameter(Title = "Linear force applied to body")]
	private float linearForce=5.0f;
	private BodyRigid rigidBody;
	private float currentForce = 0.0f;
	void Init()
	{
		rigidBody = node.ObjectBodyRigid;
		if (!rigidBody)
		{
			Log.Error("PhysicsIFpsController.Init() can't find rigid body on the node!\n");
		}
		currentForce = linearForce;
	}
	void Update()
	{
		//visualizing current linear velocity
		Visualizer.Enabled = true;
		Visualizer.RenderVector(rigidBody.Position, rigidBody.Position + new Vec3(rigidBody.LinearVelocity), vec4.RED,0.5f);
		//NOTICE that methods: Update and UdpatePhysics registered in different component Macros and code is the same for both usage examples
		// using Update() to move node with physics
		if (useUdpate)
		{
			Movement();
		}
	}
	void UpdatePhysics()
	{
		//NOTICE that methods: Update and UdpatePhysics registered in different component Macros and code is the same for both usage examples
		// using Update() to move node with physics
		if (!useUdpate)
		{
			Movement();
		}
	}
	private void Movement()
	{
		rigidBody.AddForce(vec3.RIGHT * currentForce);
		if (node.WorldPosition.x > 5)
			currentForce= -linearForce;
		if (node.WorldPosition.x < -5)
			currentForce = linearForce;
	}
}
```
---
## UpdatePhysicsUsageSample.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using System.Globalization;
using Unigine;
[Component(PropertyGuid = "e56cecddd17f9708f2cc09e6d846de17e4005379")]
public class UpdatePhysicsUsageSample : Component
{
	private SampleDescriptionWindow window = null;
	private WidgetSlider maxFpsSlider = null;
	private void Init()
	{
		window = new SampleDescriptionWindow();
		window.createWindow();
		maxFpsSlider = window.addIntParameter("Max render fps:", "Max render fps:", Render.MaxFPS, 15, 150, (int value) =>
		{
			Render.MaxFPS = value;
		});
	}
	private void Shutdown()
	{
		window.shutdown();
	}
}
```
---
## UserInterfaceSample.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "b4a1c14d6eb72caa79943c59f9360e9a2421d91f")]
public class UserInterfaceSample : Component
{
	public AssetLink ui_file;
	private UserInterface ui;
	private Widget window;
	void Init()
	{
		if (ui_file.Path.Length < 1)
		{
			Log.Warning("UserInterfaceSample::init(): ui_file is not assigned.\n");
			return;
		}
		Gui gui = Gui.GetCurrent();
		ui = new UserInterface(gui, ui_file.Path);
		if (!ui)
			Log.Error("UserInterfaceSample::init(): can't created UserInterface.\n");
		ui.GetWidgetByName("edittext").EventChanged.Connect(EdittextChanged);
		ui.GetWidgetByName("menubox_0").EventClicked.Connect(Menubox0Clicked);
		window = ui.GetWidget(ui.FindWidget("window"));
		window.Arrange();
		gui.AddChild(window, Gui.ALIGN_OVERLAP | Gui.ALIGN_CENTER);
		Console.Onscreen = true;
	}
	void Shutdown()
	{
		if (window)
			window.DeleteLater();
		if (ui)
			ui.DeleteLater();
		Console.Onscreen = false;
	}
	private void EdittextChanged (Widget widget)
	{
		WidgetEditText edittext = widget as WidgetEditText;
		Log.Message("EditText changed: {0}\n", edittext.Text);
	}
	private void Menubox0Clicked(Widget widget)
	{
		WidgetMenuBox menubox = widget as WidgetMenuBox;
		Log.Message("MenuBox clicked: {0}\n", menubox.CurrentItemText);
		if (menubox.CurrentItem == 2)
			Unigine.Console.Run("quit");
	}
}
```
---
## VisualizerSample.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "3a4bfa242c0f8179abfaded08f54d57662e23496")]
public class VisualizerSample : Component
{
	[ShowInEditor]
	[Parameter(Title = "Visualizer usage")]
	private VisualizerUsage visualizer_usage = null;
	private SampleDescriptionWindow window = null;
	void Init()
	{
		window = new SampleDescriptionWindow();
		window.createWindow();
		var parameters = window.getParameterGroupBox();
		//========== Enable visualizer checkbox =========//
		WidgetCheckBox visualizer_check_box = new WidgetCheckBox("Enable visualizer");
		parameters.AddChild(visualizer_check_box, Gui.ALIGN_LEFT);
		visualizer_check_box.EventChanged.Connect(() =>
		{
			Visualizer.Enabled = visualizer_check_box.Checked;
		});
		visualizer_check_box.Checked = true;
		//========== Enable visualizer checkbox =========//
		WidgetCheckBox depth_test_check_box = new WidgetCheckBox("Enable depth test");
		parameters.AddChild(depth_test_check_box, Gui.ALIGN_LEFT);
		depth_test_check_box.EventChanged.Connect(() =>
		{
			if (depth_test_check_box.Checked)
			{
				Visualizer.Mode = Visualizer.MODE.ENABLED_DEPTH_TEST_ENABLED;
			}
			else
			{
				Visualizer.Mode = Visualizer.MODE.ENABLED_DEPTH_TEST_DISABLED;
			}
		});
		depth_test_check_box.Checked = true;
		//========== Enable point2D checkbox =========//
		WidgetCheckBox point2D_check_box = new WidgetCheckBox("Point2D");
		parameters.AddChild(point2D_check_box, Gui.ALIGN_LEFT);
		point2D_check_box.EventChanged.Connect(() =>
		{
			visualizer_usage.renderPoint2D = point2D_check_box.Checked;
		});
		point2D_check_box.Checked = visualizer_usage.renderPoint2D;
		//========== Enable line2D checkbox =========//
		WidgetCheckBox line2D_check_box = new WidgetCheckBox("Line2D");
		parameters.AddChild(line2D_check_box, Gui.ALIGN_LEFT);
		line2D_check_box.EventChanged.Connect(() =>
		{
			visualizer_usage.renderLine2D = line2D_check_box.Checked;
		});
		line2D_check_box.Checked = visualizer_usage.renderPoint2D;
		//========== Enable triangle2D checkbox =========//
		WidgetCheckBox triangle2D_check_box = new WidgetCheckBox("Triangle2D");
		parameters.AddChild(triangle2D_check_box, Gui.ALIGN_LEFT);
		triangle2D_check_box.EventChanged.Connect(() =>
		{
			visualizer_usage.renderTriangle2D = triangle2D_check_box.Checked;
		});
		triangle2D_check_box.Checked = visualizer_usage.renderTriangle2D;
		//========== Enable quad2D checkbox =========//
		WidgetCheckBox quad2D_check_box = new WidgetCheckBox("Quad2D");
		parameters.AddChild(quad2D_check_box, Gui.ALIGN_LEFT);
		quad2D_check_box.EventChanged.Connect(() =>
		{
			visualizer_usage.renderQuad2D = quad2D_check_box.Checked;
		});
		quad2D_check_box.Checked = visualizer_usage.renderQuad2D;
		//========== Enable rectangle checkbox =========//
		WidgetCheckBox rectangle_check_box = new WidgetCheckBox("Rectangle");
		parameters.AddChild(rectangle_check_box, Gui.ALIGN_LEFT);
		rectangle_check_box.EventChanged.Connect(() =>
		{
			visualizer_usage.renderRectangle = rectangle_check_box.Checked;
		});
		rectangle_check_box.Checked = visualizer_usage.renderRectangle;
		//========== Enable message2D checkbox =========//
		WidgetCheckBox message2D_check_box = new WidgetCheckBox("Message2D");
		parameters.AddChild(message2D_check_box, Gui.ALIGN_LEFT);
		message2D_check_box.EventChanged.Connect(() =>
		{
			visualizer_usage.renderMessage2D = message2D_check_box.Checked;
		});
		message2D_check_box.Checked = visualizer_usage.renderMessage2D;
	}
	void Shutdown()
	{
		window.shutdown();
	}
}
```
---
## VisualizerUsage.cs
```csharp
﻿using System.Collections.Generic;
using Unigine;
using System.Linq;
#region Math Variables
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Mat4 = Unigine.mat4;
#endif
#endregion
[Component(PropertyGuid = "e1c2e6615a64f81d79850545bb83d4115d89f8af")]
public class VisualizerUsage : Component
{
	public bool renderPoint2D = false;
	public bool renderLine2D = false;
	public bool renderTriangle2D = false;
	public bool renderQuad2D = false;
	public bool renderRectangle = false;
	public bool renderMessage2D = false;
	[ShowInEditor]
	[Parameter(Title = "Node bound box example")]
	private Node node_boundBox_example = null;
	[ShowInEditor]
	[Parameter(Title = "Node bound sphere example")]
	private Node node_boundSphere_example = null;
	[ShowInEditor]
	[Parameter(Title = "Object example")]
	private Object object_example = null;
	[ShowInEditor]
	[Parameter(Title = "Object solid example")]
	private Object object_solid_example = null;
	[ShowInEditor]
	[Parameter(Title = "Surface example")]
	private Object surface_example = null;
	[ShowInEditor]
	[Parameter(Title = "Surface solid example")]
	private Object surface_solid_example = null;
	[ShowInEditor]
	[Parameter(Title = "Object surface bound box example")]
	private Object object_surface_boundBox_example = null;
	[ShowInEditor]
	[Parameter(Title = "Object surface bound sphere example")]
	private Object object_surface_boundSphere_example = null;
	[ShowInEditor]
	[Parameter(Title = "Object surface bound sphere example")]
	private List<Node> postament_nodes = null;
	void Init()
	{
		if (!node_boundBox_example || !node_boundSphere_example)
		{
			Log.Error("VisualizerUsage.Init() example nodes are not assigned: \n");
		}
		if (!object_example || !object_solid_example || !surface_example || !surface_solid_example
			|| !object_surface_boundBox_example || !object_surface_boundSphere_example)
		{
			Log.Error("VisualizerUsage.Init() example objects are not assigned: \n");
		}
		foreach (var item in postament_nodes)
		{
			if (!node)
			{
				Log.Error($"VisualizerUsage.Init() postament node with index: {postament_nodes.IndexOf(item)} is not assigned\n");
			}
		}
		Visualizer.Mode = Visualizer.MODE.ENABLED_DEPTH_TEST_DISABLED;
	}
	void Update()
	{
		Update3D();
		Update2D();
	}
	private void Update2D()
	{
		if (renderPoint2D) Visualizer.RenderPoint2D(new vec2(0.1f, 0.5), 0.01f, vec4.RED);
		if (renderLine2D) Visualizer.RenderLine2D(new vec2(0.1, 0.55), new vec2(0.15, 0.55), new vec2(0.15, 0.60), new vec2(0.20, 0.60), vec4.RED);
		if (renderTriangle2D) Visualizer.RenderTriangle2D(new vec2(0.2, 0.65), new vec2(0.1, 0.62), new vec2(0.1, 0.68), vec4.RED);
		if (renderQuad2D) Visualizer.RenderQuad2D(new vec2(0.1, 0.8), new vec2(0.08, 0.75), new vec2(0.1, 0.70), new vec2(0.12, 0.75), vec4.RED);
		if (renderRectangle) Visualizer.RenderRectangle(new vec4(0.1f, 0.1f, 0.15f, 0.15f), vec4.RED);
		if (renderMessage2D) Visualizer.RenderMessage2D(new vec3(0.1, 0.95, 0), new vec3(0, 0, 0), "renderMessage2D example", vec4.RED);
	}
	private void Update3D()
	{
		int i = 0;
		//1-10
		Vec3 current_point = GetPostamentPoint(i);
		Visualizer.RenderLine3D(current_point, current_point + Vec3.UP, current_point + Vec3.UP + Vec3.RIGHT / 2, current_point + Vec3.UP * 2, vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderPoint3D(current_point, 0.5f, vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderTriangle3D(current_point, current_point + Vec3.DOWN / 2 + Vec3.LEFT / 2, current_point + Vec3.DOWN / 2 + Vec3.RIGHT / 2, vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderQuad3D(current_point + Vec3.LEFT / 2, current_point + Vec3.LEFT / 2 + Vec3.UP, current_point + Vec3.UP + Vec3.RIGHT / 2, current_point + Vec3.RIGHT / 2, vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderBillboard3D(current_point, 0.5f, vec4.ZERO, true);// try use setTextureName
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderVector(current_point + Vec3.DOWN / 2 + Vec3.LEFT / 2, current_point + Vec3.UP / 2 + Vec3.RIGHT / 2, vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderDirection(current_point + Vec3.DOWN / 2 + Vec3.LEFT / 2, new vec3(1, 0, 1), vec4.RED, 0.25f, false);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderBox(vec3.ONE, new Mat4(quat.IDENTITY, current_point), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderSolidBox(vec3.ONE, new Mat4(quat.IDENTITY, current_point), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		mat4 proj = MathLib.Perspective(60, 16 / 9, 0.1f, 1);
		Mat4 modelview = MathLib.LookAt(current_point + vec3.LEFT / 2, current_point, vec3.UP);
		//transformation matrix equals inversed modelview
		Visualizer.RenderFrustum(proj, MathLib.Inverse(modelview), vec4.RED);
		i++;
		//10-20
		current_point = GetPostamentPoint(i);
		Visualizer.RenderCircle(0.5f, new Mat4(quat.IDENTITY * new quat(90, 0, 0), current_point), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderSector(0.5f, 60, new Mat4(quat.IDENTITY * new quat(90, 0, 0), current_point), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderCone(0.5f, 30, new Mat4(quat.IDENTITY, current_point), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderSphere(0.5f, new Mat4(quat.IDENTITY, current_point), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderSolidSphere(0.5f, new Mat4(quat.IDENTITY, current_point), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderCapsule(0.5f, 0.5f, new Mat4(quat.IDENTITY, current_point), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderSolidCapsule(0.5f, 0.5f, new Mat4(quat.IDENTITY, current_point), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderCylinder(1, 1, new Mat4(quat.IDENTITY, current_point), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderSolidCylinder(1, 1, new Mat4(quat.IDENTITY, current_point), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		vec3 center = new vec3(0, 0, 0);// check out documentation for center format information
		Visualizer.RenderMessage3D(current_point, center, "renderMessage3D exapmle", vec4.RED);
		i++;
		//20-30
		current_point = GetPostamentPoint(i);
		Visualizer.RenderEllipse(new vec3(0.5, 1, 1.5), new Mat4(quat.IDENTITY, current_point + Vec3.UP), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderSolidEllipse(new vec3(0.5, 1, 1.5), new Mat4(quat.IDENTITY, current_point + Vec3.UP), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		BoundBox bb = new BoundBox(new vec3(-0.5f, -0.5f, -0.5f), new vec3(0.5f, 0.5f, 0.5f));
		Visualizer.RenderBoundBox(bb, new Mat4(quat.IDENTITY * new quat(90, 0, 0), current_point), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		BoundSphere bs = new BoundSphere(vec3.ZERO, 0.5f);
		Visualizer.RenderBoundSphere(bs, new Mat4(quat.IDENTITY, current_point), vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderNodeBoundBox(node_boundBox_example, vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderNodeBoundSphere(node_boundSphere_example, vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderObject(object_example, vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderSolidObject(object_solid_example, vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderObjectSurface(surface_example, 0, vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderSolidObjectSurface(surface_solid_example, 0, vec4.RED);
		i++;
		//30-32
		current_point = GetPostamentPoint(i);
		Visualizer.RenderObjectSurfaceBoundBox(object_surface_boundBox_example, 0, vec4.RED);
		i++;
		current_point = GetPostamentPoint(i);
		Visualizer.RenderObjectSurfaceBoundSphere(object_surface_boundSphere_example, 0, vec4.RED);
		i++;
	}
	Vec3 GetPostamentPoint(int index)
	{
		Vec3 result = Vec3.ONE;
		if (index < postament_nodes.Count)
		{
			Node node = postament_nodes[index];
			if (node)
			{
				result = node.WorldPosition;
			}
			else
			{
				Log.Message($"VisualizerUsage.GetNextPostamentPoint postamentNodes: Node with index: {index} is null \n");
			}
		}
		else
		{
			Log.Message("Visualizer usage doesn't have enough pedestal's display nodes to draw all visualizer examples\n");
		}
		return result;
	}
}
```
---
## Waves.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
using Console = System.Console;
[Component(PropertyGuid = "f821909f5549a5721139c6297b53597471a2b643")]
public class Waves : Component
{
	private ObjectWaterGlobal water = null;
	private void Init()
	{
		water = World.GetNodeByType((int)Node.TYPE.OBJECT_WATER_GLOBAL) as ObjectWaterGlobal;
		if (water == null)
		{
			Log.Error("Now ObjectWaterGlobal in world!");
			return;
		}
		water.Beaufort = 0;
	}
	public void SetBeaufort(float beaufort)
	{
		water.Beaufort = beaufort;
	}
}
```
---
## WeaponClipping.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using System.Runtime.InteropServices;
using Unigine;
[Component(PropertyGuid = "c4af7e3a794d78510337557ccb69b2ab20b0280e")]
public class WeaponClipping : Component
{
	[ShowInEditor]
	private Player mainPlayer = null;
	[ShowInEditor]
	private Player weaponPlayer = null;
	private Viewport viewport = null;
	public Viewport RenderViewport { get { return viewport; } }
	private Texture texture = null;
	private int currentWidth = 0;
	private int currentHeight = 0;
	private bool isRenderingWeapon = false;
	private EventConnections componentConnections = new EventConnections();
	private void Init()
	{
		EngineWindow main_window = WindowManager.MainWindow;
		if (!main_window)
		{
			Engine.Quit();
			return;
		}
		ivec2 main_size = main_window.ClientRenderSize;
		currentWidth = main_size.x;
		currentHeight = main_size.y;
		viewport = new Viewport();
		viewport.NodeLightUsage = Viewport.USAGE_WORLD_LIGHT;
		viewport.SkipFlags = Viewport.SKIP_VELOCITY_BUFFER | Viewport.SKIP_SHADOWS;
		texture = new Texture();
		CreateTexture2d(ref texture);
		Render.EventBeginPostMaterials.Connect(componentConnections, RenderCallback);
		WindowManager.MainWindow.EventResized.Connect(componentConnections, UpdateScreenSize);
	}
	private void Update()
	{
		weaponPlayer.Transform = mainPlayer.Camera.IModelview;
	}
	private void PostUpdate()
	{
		if (Game.Player != mainPlayer)
			return;
		RenderState.SaveState();
		RenderState.ClearStates();
		RenderState.SetViewport(0, 0, currentWidth, currentHeight);
		var target = Render.GetTemporaryRenderTarget();
		target.BindColorTexture(0, texture);
		target.Enable();
		{
			bool flare = Render.LightsLensFlares;
			Render.LightsLensFlares = false;
			RenderState.ClearBuffer(RenderState.BUFFER_ALL, vec4.ZERO);
			RenderState.FlushStates();
			// render near plane with weapon to texture
			if (texture != null)
			{
				isRenderingWeapon = true;
				viewport.RenderTexture2D(weaponPlayer.Camera, texture);
				isRenderingWeapon = false;
			}
			Render.LightsLensFlares = flare;
		}
		target.Disable();
		target.UnbindColorTexture(0);
		RenderState.RestoreState();
		Render.ReleaseTemporaryRenderTarget(target);
	}
	private void RenderCallback()
	{
		if (Game.Player != mainPlayer)
			return;
		if (isRenderingWeapon)
		{
			// skip render to screen when we rendering weapon into custom texture
			return;
		}
		RenderState.SaveState();
		RenderState.ClearStates();
		RenderState.SetViewport(0, 0, currentWidth, currentHeight);
		var target = Render.GetTemporaryRenderTarget();
		target.BindColorTexture(0, Renderer.TextureColor);
		target.Enable();
		{
			RenderState.SetBlendFunc(RenderState.BLEND_SRC_ALPHA, RenderState.BLEND_ONE_MINUS_SRC_ALPHA);
			RenderState.FlushStates();
			// render texture with weapon to screen
			if (texture != null)
			{
				Render.RenderScreenMaterial("Unigine::render_copy_2d", texture);
			}
		}
		target.Disable();
		target.UnbindColorTexture(0);
		RenderState.RestoreState();
		Render.ReleaseTemporaryRenderTarget(target);
	}
	private void UpdateScreenSize()
	{
		EngineWindow main_window = WindowManager.MainWindow;
		if (!main_window)
		{
			Engine.Quit();
			return;
		}
		ivec2 main_size = main_window.ClientRenderSize;
		int appWidth = main_size.x;
		int appHeight = main_size.y;
		if (appWidth != currentWidth || appHeight != currentHeight)
		{
			currentWidth = appWidth;
			currentHeight = appHeight;
			CreateTexture2d(ref texture);
		}
	}
	private void Shutdown()
	{
		texture = null;
		viewport = null;
		componentConnections.DisconnectAll();
	}
	private void CreateTexture2d(ref Texture texture)
	{
		texture.Create2D(currentWidth, currentHeight, Texture.FORMAT_RGBA8,
			Texture.SAMPLER_FILTER_LINEAR | Texture.SAMPLER_ANISOTROPY_16 | Texture.FORMAT_USAGE_RENDER);
	}
}
```
---
## WeaponClippingSample.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "1c692232a97a420aa8ff22fad0d3f4984bc0ce17")]
public class WeaponClippingSample : Component
{
	[ShowInEditor]
	private WeaponClipping weaponClipping = null;
	private SampleDescriptionWindow sampleDescriptionWindow = new SampleDescriptionWindow();
	private Input.MOUSE_HANDLE mouse_handle = Input.MOUSE_HANDLE.USER;
	void Init()
	{
		sampleDescriptionWindow.createWindow();
		mouse_handle = Input.MouseHandle;
		Input.MouseHandle = Input.MOUSE_HANDLE.GRAB;
		sampleDescriptionWindow.addBoolParameter("Skip Shadow", null, true, OnShadowsCheckboxChanged);		
	}
	private void OnShadowsCheckboxChanged(bool is_checked)
	{
		if (weaponClipping == null)
		{
			Log.Message("WeaponClippingSample::OnShadowsCheckboxChanged(): weaponClippingNode is not set");
			return;
		}
		int flags = Viewport.SKIP_VELOCITY_BUFFER;
		if (is_checked)
		{
			flags |= Viewport.SKIP_SHADOWS;
		}
		weaponClipping.RenderViewport.SkipFlags = flags;
	}
	private void Shutdown()
	{
		Input.MouseHandle = mouse_handle;
		sampleDescriptionWindow.shutdown();
	}
}
```
---
## WidgetClock.cs
```csharp
using System;
using Unigine;
/// <summary>
/// This sample demonstrates how the <c> GuiToTexture </c> component can be used to render custom widgets
/// In this component we're going to use <c> GuiToTexture </c> with auto update disabled and will update gui manually in SetTime method
/// </summary>
[Component(PropertyGuid = "168dc2de5e12d126e849878199728ecf744bc3c5")]
public class WidgetClock : Component
{
	void Init()
	{
		guiToTexture = ComponentSystem.GetComponent<GuiToTexture>(node);
		if (guiToTexture == null)
		{
			Log.Error("WidgetClock.Init(): No GuiToTexture component found\n");
			return;
		}
		// Get our custom gui
		var gui = guiToTexture.Gui;
		widgetTimer = new WidgetLabel(gui) { FontSize = 150, FontColor = vec4.RED };
		// Disable auto update, because we will update gui manually in <c> SetTime </c> method
		guiToTexture.AutoUpdateEnabled = false;
		// Add widget as a child in gui
		gui.AddChild(widgetTimer, Gui.ALIGN_OVERLAP);
		CenterPosition = guiToTexture.TextureResolution / 2;
		previousTime = DateTime.Now.TimeOfDay;
		// Set time and update gui
		SetTime(previousTime);
		// Now we don't need to interact with GuiToTexture, it will be updated on its own
		// starting from here, we will just update the state of our custom widget
	}
	void Update()
	{
		var now = DateTime.Now.TimeOfDay;
		if (now - previousTime < TimeSpan.FromSeconds(1))
		{
			return;
		}
		SetTime(now);
		previousTime = now;
	}
	public ivec2 CenterPosition
	{
		get
		{
			return centerPosition;
		}
		set
		{
			centerPosition = value;
			AdjustScreenPosition();
		}
	}
	private void SetTime(TimeSpan time)
	{
		widgetTimer.Text = time.ToString("hh\\:mm\\:ss");
		AdjustScreenPosition();
		guiToTexture.RenderToTexture();
	}
	private void AdjustScreenPosition()
	{
		ivec2 widgetSize;
		widgetSize.y = widgetTimer.GetTextRenderSize(widgetTimer.Text).y;
		widgetSize.x = widgetTimer.GetTextRenderSize(widgetTimer.Text).x;
		widgetTimer.SetPosition(centerPosition.x - widgetSize.x / 2, centerPosition.y - widgetSize.y / 2);
	}
	private ivec2 centerPosition;
	private WidgetLabel widgetTimer;
	private TimeSpan previousTime;
	private GuiToTexture guiToTexture;
}
```
---
## WidgetNoSignal.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using Unigine;
/// <summary>
/// This sample demonstrates how the <c> GuiToTexture </c> component can be used to render custom widgets
/// In this component we're going to use <c> GuiToTexture </c> with auto update enabled and will update the widget state only
/// </summary>
[Component(PropertyGuid = "acf913707bbb76a1b995c159da51b3bec3aaab94")]
public class WidgetNoSignal : Component
{
	void Init()
	{
		// get GuiToTexture component
		GuiToTexture guiToTexture = ComponentSystem.GetComponent<GuiToTexture>(node);
		// get gui from GuiToTexture component
		Gui gui = guiToTexture.Gui;
		// create a widget that you want to render in gui
		label = new WidgetLabel(gui) { FontSize = 150, Text = "No Signal", FontColor = vec4.RED };
		container = new WidgetVBox() { Background = 1, BackgroundColor = vec4.BLUE };
		container.AddChild(label, Gui.ALIGN_EXPAND | Gui.ALIGN_BACKGROUND);
		// add the widget to gui children
		gui.AddChild(container, Gui.ALIGN_OVERLAP | Gui.ALIGN_CENTER);
		// now we don't need to interact with GuiToTexture, it will be updated on its own
		// starting from here, we will just update the state of our custom widget
	}
	void Update()
	{
		float frameSpeed = labelSpeed * Game.IFps;
		vec2 delta = direction * frameSpeed;
		int posX = container.PositionX;
		int posY = container.PositionY;
		if (new ivec2(posX, posY) + new ivec2(accumulatedDelta) == new ivec2(posX, posY))
		{
			accumulatedDelta += delta;
			return;
		}
		container.PositionX = posX + (int)accumulatedDelta.x;
		container.PositionY = posY + (int)accumulatedDelta.y;
		accumulatedDelta = new vec2(0, 0);
		ReflectDirection();
	}
	private void ReflectDirection()
	{
		ivec2 size = container.ParentGui.Size;
		int labelPosX = container.PositionX;
		int labelPosY = container.PositionY;
		int xRightCornerDelta = label.GetTextRenderSize(label.Text).x;
		int yBottomCornerDelta = label.GetTextRenderSize(label.Text).y;
		var leftTopCornerPos = new ivec2(labelPosX, labelPosY);
		var rightTopCornerPos = new ivec2(labelPosX + xRightCornerDelta, labelPosY);
		var leftBottomCornerPos = new ivec2(labelPosX, labelPosY + yBottomCornerDelta);
		var rightBottomCornerPos = new ivec2(labelPosX + xRightCornerDelta,
			labelPosY + yBottomCornerDelta);
		// check the top left corner
		{
			// intersecting with top
			if (leftTopCornerPos.x > 0 && leftTopCornerPos.y < 0)
			{
				container.PositionY = 0;
				direction = ReflectVector(direction, new vec2(0, 1));
				return;
			}
			if (leftTopCornerPos.x < 0 && leftTopCornerPos.y > 0)
			{
				container.PositionX = 0;
				direction = ReflectVector(direction, new vec2(1, 0));
				return;
			}
			// intersecting with corner
			if (leftTopCornerPos.x < 0 && leftTopCornerPos.y < 0)
			{
				direction = MathLib.Normalize(new vec2(1, 1));
				container.PositionX = 0;
				container.PositionY = 0;
				return;
			}
		}
		// check the top right corner
		{
			// right corner
			if (rightTopCornerPos.x > size.x && rightTopCornerPos.y > 0)
			{
				container.PositionX = size.x - xRightCornerDelta;
				direction = ReflectVector(direction, new vec2(-1, 0));
				return;
			}
			if (rightTopCornerPos.x > size.x && rightTopCornerPos.y < 0)
			{
				container.PositionX = size.x - xRightCornerDelta;
				container.PositionY = 0;
				direction = new vec2(-1, 1);
				return;
			}
		}
		// check the bottom left corner
		{
			if (leftBottomCornerPos.x < 0 && leftBottomCornerPos.y < size.y)
			{
				container.PositionX = 0;
				direction = ReflectVector(direction, new vec2(1, 0));
				return;
			}
			if (leftBottomCornerPos.x > 0 && leftBottomCornerPos.y > size.y)
			{
				container.PositionY = size.y - yBottomCornerDelta;
				direction = ReflectVector(direction, new vec2(0, -1));
				return;
			}
			if (leftBottomCornerPos.x < 0 && leftBottomCornerPos.y > size.y)
			{
				container.PositionX = 0;
				container.PositionY = yBottomCornerDelta;
				direction = new vec2(1, -1);
				return;
			}
		}
		// bottom right corner
		{
			if (rightBottomCornerPos.x > size.x && rightBottomCornerPos.y < size.y)
			{
				container.PositionX = size.x - xRightCornerDelta;
				direction = ReflectVector(direction, new vec2(-1, 0));
				return;
			}
			if (rightBottomCornerPos.x > size.x && rightBottomCornerPos.y > size.y)
			{
				container.PositionX = size.x - xRightCornerDelta;
				container.PositionY = size.y - yBottomCornerDelta;
				direction = new vec2(-1, -1);
				return;
			}
		}
	}
	static vec2 ReflectVector(vec2 vector, vec2 normal)
	{
		return MathLib.Normalize(vector - normal * MathLib.Dot(vector, normal) * 2);
	}
	[ShowInEditor] private float labelSpeed = 1000;
	private WidgetVBox container;
	private WidgetLabel label;
	private vec2 accumulatedDelta;
	private vec2 direction = new vec2(1, 1);
}
```
---
## WidgetsButton.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "abe43a0d916c1408a3af27c7389c427adcc4ef91")]
public class WidgetsButton : Component
{
	public int x = 250;
	public int y = 50;
	public int width = 100;
	public int height = 50;
	public string text = "Press Me";
	public int fontSize = 16;
	private WidgetButton button = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create button
		button = new WidgetButton(gui, text);
		button.SetPosition(x, y);
		button.Width = width;
		button.Height = height;
		button.FontSize = fontSize;
		button.EventClicked.Connect(() => Unigine.Console.OnscreenMessageLine("Button Clicked!"));
		// add button to current gui
		gui.AddChild(button, Gui.ALIGN_OVERLAP);
		Unigine.Console.Onscreen = true;
	}
	private void Shutdown()
	{
		// remove button from current gui
		Gui.GetCurrent().RemoveChild(button);
		Unigine.Console.Onscreen = false;
	}
}
```
---
## WidgetsCheckbox.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "19255e459ba13ae020f430bff6e887129cd0fb49")]
public class WidgetsCheckbox : Component
{
	public int x = 450;
	public int y = 50;
	public string text = "Check Me";
	public int fontSize = 16;
	private WidgetCheckBox checkBox = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create checkbox
		checkBox = new WidgetCheckBox(gui, text);
		checkBox.SetPosition(x, y);
		checkBox.FontSize = fontSize;
		checkBox.FontOutline = 1;
		checkBox.EventChanged.Connect(() => Unigine.Console.OnscreenMessageLine($"Checkbox: {checkBox.Checked}"));
		// add checkbox to current gui
		gui.AddChild(checkBox, Gui.ALIGN_OVERLAP);
		Unigine.Console.Onscreen = true;
	}
	private void Shutdown()
	{
		// remove checkbox from current gui
		Gui.GetCurrent().RemoveChild(checkBox);
		Unigine.Console.Onscreen = false;
	}
}
```
---
## WidgetsCombobox.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "2b37ec3ac80f0e51cacff37c2b1db8ab4d38760e")]
public class WidgetsCombobox : Component
{
	public int x = 600;
	public int y = 50;
	public int fontSize = 16;
	private WidgetComboBox comboBox = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create combobox
		comboBox = new WidgetComboBox(gui);
		comboBox.SetPosition(x, y);
		comboBox.FontSize = fontSize;
		// add items
		comboBox.AddItem("item 0");
		comboBox.AddItem("item 1");
		comboBox.AddItem("item 2");
		comboBox.EventChanged.Connect(() => Unigine.Console.OnscreenMessageLine($"Combobox: {comboBox.GetCurrentItemText()}"));
		// add combobox to current gui
		gui.AddChild(comboBox, Gui.ALIGN_OVERLAP);
		Unigine.Console.Onscreen = true;
	}
	private void Shutdown()
	{
		// remove combobox from current gui
		Gui.GetCurrent().RemoveChild(comboBox);
		Unigine.Console.Onscreen = false;
	}
}
```
---
## WidgetsContainers.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "04560204ebfea593d9b0edd603144b3dd3661378")]
public class WidgetsContainers : Component
{
	private WidgetVBox vBox = null;
	private WidgetVPaned vPaned = null;
	private WidgetHBox hBoxTop = null;
	private WidgetHBox hBoxBottom = null;
	private WidgetHPaned hPanedTop = null;
	private WidgetGridBox gridBox = null;
	private WidgetGroupBox groupBox = null;
	private WidgetTabBox tabBox = null;
	private WidgetScrollBox scrollBox = null;
	[MethodInit(Order = -1)]
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// add vbox
		vBox = new WidgetVBox(gui);
		vBox.Background = 1;
		gui.AddChild(vBox, Gui.ALIGN_EXPAND);
		// add vpaned to vbox
		vPaned = new WidgetVPaned(gui);
		vPaned.Width = 450;
		vBox.AddChild(vPaned, Gui.ALIGN_EXPAND);
		// add top hbox to vpaned
		hBoxTop = new WidgetHBox(gui);
		hBoxTop.Background = 1;
		hBoxTop.BackgroundColor = new vec4(0.0f, 0.0f, 1.0f, 0.5f);
		vPaned.AddChild(hBoxTop, Gui.ALIGN_EXPAND);
		// add bottom hbox to vpaned
		hBoxBottom = new WidgetHBox(gui);
		hBoxBottom.Background = 1;
		hBoxBottom.BackgroundColor = new vec4(0.0f, 1.0f, 1.0f, 0.5f);
		vPaned.AddChild(hBoxBottom, Gui.ALIGN_EXPAND);
		// add hpaned to top hbox
		hPanedTop = new WidgetHPaned(gui);
		hBoxTop.AddChild(hPanedTop, Gui.ALIGN_EXPAND);
		// add gridbox to top hpaned
		gridBox = new WidgetGridBox(gui, 3, 100, 100);
		gridBox.Background = 1;
		gridBox.BackgroundColor = new vec4(1.0f, 0.0f, 0.0f, 0.5f);
		gridBox.AddChild(new WidgetLabel(gui, "Item 0") { FontSize = 30 }, Gui.ALIGN_CENTER);
		gridBox.AddChild(new WidgetLabel(gui, "Item 1") { FontSize = 30 }, Gui.ALIGN_CENTER);
		gridBox.AddChild(new WidgetLabel(gui, "Item 2") { FontSize = 30 }, Gui.ALIGN_CENTER);
		gridBox.AddChild(new WidgetLabel(gui, "Item 3") { FontSize = 30 }, Gui.ALIGN_CENTER);
		gridBox.AddChild(new WidgetLabel(gui, "Item 4") { FontSize = 30 }, Gui.ALIGN_CENTER);
		gridBox.AddChild(new WidgetLabel(gui, "Item 5") { FontSize = 30 }, Gui.ALIGN_CENTER);
		gridBox.AddChild(new WidgetLabel(gui, "Item 6") { FontSize = 30 }, Gui.ALIGN_CENTER);
		gridBox.AddChild(new WidgetLabel(gui, "Item 7") { FontSize = 30 }, Gui.ALIGN_CENTER);
		gridBox.AddChild(new WidgetLabel(gui, "Item 8") { FontSize = 30 }, Gui.ALIGN_CENTER);
		hPanedTop.AddChild(gridBox, Gui.ALIGN_OVERLAP);
		// add groupbox to top hpaned
		groupBox = new WidgetGroupBox(gui, "Group Box", 30, 30);
		groupBox.Background = 1;
		groupBox.BackgroundColor = new vec4(0.0f, 1.0f, 0.0f, 0.5f);
		groupBox.AddChild(new WidgetLabel(gui, "Item 0") { FontSize = 30 });
		groupBox.AddChild(new WidgetLabel(gui, "Item 1") { FontSize = 30 });
		groupBox.AddChild(new WidgetLabel(gui, "Item 2") { FontSize = 30 });
		groupBox.AddChild(new WidgetLabel(gui, "Item 3") { FontSize = 30 });
		hPanedTop.AddChild(groupBox, Gui.ALIGN_EXPAND);
		// add tabbox to bottom hbox
		tabBox = new WidgetTabBox(gui);
		tabBox.AddTab("Tab 0");
		tabBox.AddChild(new WidgetLabel(gui, "Tab 0 Content") { FontSize = 50 });
		tabBox.AddTab("Tab 1");
		tabBox.AddChild(new WidgetLabel(gui, "Tab 1 Content") { FontSize = 50 });
		tabBox.AddTab("Tab 2");
		tabBox.AddChild(new WidgetLabel(gui, "Tab 2 Content") { FontSize = 50 });
		hBoxBottom.AddChild(tabBox, Gui.ALIGN_EXPAND);
		// add scroll box to bottom hbox
		scrollBox = new WidgetScrollBox(gui, 30, 30);
		scrollBox.BackgroundColor = new vec4(0.0f, 0.0f, 1.0f, 0.5f);
		scrollBox.Width = 250;
		scrollBox.Height = 250;
		scrollBox.AddChild(new WidgetLabel(gui, "Item 0") { FontSize = 20 });
		scrollBox.AddChild(new WidgetLabel(gui, "Item 1") { FontSize = 20 });
		scrollBox.AddChild(new WidgetLabel(gui, "Item 2") { FontSize = 20 });
		scrollBox.AddChild(new WidgetLabel(gui, "Item 3") { FontSize = 20 });
		scrollBox.AddChild(new WidgetLabel(gui, "Item 4") { FontSize = 20 });
		scrollBox.AddChild(new WidgetLabel(gui, "Item 5") { FontSize = 20 });
		scrollBox.AddChild(new WidgetLabel(gui, "Item 6") { FontSize = 20 });
		scrollBox.AddChild(new WidgetLabel(gui, "Item 7") { FontSize = 20 });
		scrollBox.AddChild(new WidgetLabel(gui, "Item 8") { FontSize = 20 });
		hBoxBottom.AddChild(scrollBox, Gui.ALIGN_EXPAND);
		vBox.SetFocus();
	}
	private void Shutdown()
	{
		Gui.GetCurrent().RemoveChild(vBox);
	}
}
```
---
## WidgetsEditline.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "9113b1e5d2ad3a009e56e55f4c837332bee3b8e9")]
public class WidgetsEditline : Component
{
	public int x = 750;
	public int y = 50;
	public int width = 150;
	public int height = 30;
	public string text = "Enter text...";
	public int fontSize = 16;
	private WidgetEditLine editLine = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create editline
		editLine = new WidgetEditLine(gui, text);
		editLine.SetPosition(x, y);
		editLine.Width = width;
		editLine.Height = height;
		editLine.FontSize = fontSize;
		editLine.FontOutline = 1;
		editLine.EventChanged.Connect(() => Unigine.Console.OnscreenMessageLine($"Editline text: {editLine.Text}"));
		// add editline to current gui
		gui.AddChild(editLine, Gui.ALIGN_OVERLAP);
		Unigine.Console.Onscreen = true;
	}
	private void Shutdown()
	{
		// remove editline form current gui
		Gui.GetCurrent().RemoveChild(editLine);
		Unigine.Console.Onscreen = false;
	}
}
```
---
## WidgetsEdittext.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "00ce6ffe7757d74551675fe0ebc211bf4402f412")]
public class WidgetsEdittext : Component
{
	public int x = 250;
	public int y = 150;
	public int width = 150;
	public int height = 100;
	public string text = "Enter text...";
	public int fontSize = 16;
	private WidgetEditText editText = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create edittext
		editText = new WidgetEditText(gui, text);
		editText.SetPosition(x, y);
		editText.Width = width;
		editText.Height = height;
		editText.FontSize = fontSize;
		editText.FontOutline = 1;
		editText.EventChanged.Connect(() => Unigine.Console.OnscreenMessageLine($"Edittext: {editText.Text}"));
		// add edittext to current gui
		gui.AddChild(editText, Gui.ALIGN_OVERLAP);
		Unigine.Console.Onscreen = true;
	}
	private void Shutdown()
	{
		// remove edittext from current gui
		Gui.GetCurrent().RemoveChild(editText);
		Unigine.Console.Onscreen = false;
	}
}
```
---
## WidgetsIcon.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "18ecb4706f48bd0f54eb8722968e5c1998cc96a4")]
public class WidgetsIcon : Component
{
	public int x = 500;
	public int y = 450;
	public int width = 32;
	public int height = 32;
	[ParameterFile]
	public string iconImage = "";
	private WidgetIcon icon = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create icon
		icon = new WidgetIcon(gui, iconImage, width, height);
		icon.Toggleable = true;
		icon.SetPosition(x, y);
		icon.EventClicked.Connect(() => Unigine.Console.OnscreenMessageLine($"Icon: {icon.Toggled}"));
		// add icon to current gui
		gui.AddChild(icon, Gui.ALIGN_OVERLAP);
		Unigine.Console.Onscreen = true;
	}
	private void Shutdown()
	{
		// remove icon from current gui
		Gui.GetCurrent().RemoveChild(icon);
		Unigine.Console.Onscreen = false;
	}
}
```
---
## WidgetsLabel.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "a09052cffaf8a1e2daa137d907103f6442480373")]
public class WidgetsLabel : Component
{
	public int x = 800;
	public int y = 150;
	public string text = "Label";
	public int fontSize = 16;
	private WidgetLabel label = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create label
		label = new WidgetLabel(gui, text);
		label.SetPosition(x, y);
		label.FontSize = fontSize;
		label.FontOutline = 1;
		// add label to current gui
		gui.AddChild(label, Gui.ALIGN_OVERLAP);
	}
	private void Shutdown()
	{
		// remove label from current gui
		Gui.GetCurrent().RemoveChild(label);
	}
}
```
---
## WidgetsListbox.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "0b957b466bfa19ede69bb268679472a7d6f64fda")]
public class WidgetsListbox : Component
{
	public int x = 300;
	public int y = 300;
	public int fontSize = 16;
	private WidgetListBox listBox = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create listbox
		listBox = new WidgetListBox(gui);
		listBox.SetPosition(x, y);
		listBox.AddItem("item 0");
		listBox.AddItem("item 1");
		listBox.AddItem("item 2");
		listBox.FontSize = fontSize;
		listBox.FontOutline = 1;
		listBox.EventChanged.Connect(() => Unigine.Console.OnscreenMessageLine($"Listbox: {listBox.GetCurrentItemText()}"));
		// add listbox to current gui
		gui.AddChild(listBox, Gui.ALIGN_OVERLAP);
		Unigine.Console.Onscreen = true;
	}
	private void Shutdown()
	{
		// remove listbox from current gui
		Gui.GetCurrent().RemoveChild(listBox);
		Unigine.Console.Onscreen = false;
	}
}
```
---
## WidgetsMenu.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "b3a398566ad5f3bdb5dea1f8fe7dfcf70fbe35ea")]
public class WidgetsMenu : Component
{
	public int x = 450;
	public int y = 300;
	public int fontSize = 16;
	[ParameterColor]
	public vec4 selectionColor = vec4.ZERO;
	private WidgetMenuBar menuBar = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create menubars and menuboxes
		menuBar = new WidgetMenuBar(gui);
		menuBar.SelectionColor = selectionColor;
		menuBar.AddItem("File");
		menuBar.AddItem("Edit");
		menuBar.AddItem("Help");
		menuBar.SetPosition(x, y);
		menuBar.FontSize = fontSize;
		menuBar.FontOutline = 1;
		// add file menubox
		WidgetMenuBox fileMenuBox = new WidgetMenuBox(gui);
		fileMenuBox.FontSize = fontSize;
		fileMenuBox.FontOutline = 1;
		fileMenuBox.AddItem("File 0");
		fileMenuBox.AddItem("File 1");
		fileMenuBox.AddItem("File 2");
		fileMenuBox.EventClicked.Connect(() => Unigine.Console.OnscreenMessageLine($"Menubar: {fileMenuBox.CurrentItemText}"));
		menuBar.SetItemMenu(0, fileMenuBox);
		// add edit menubox
		WidgetMenuBox editMenuBox = new WidgetMenuBox(gui);
		editMenuBox.FontSize = fontSize;
		editMenuBox.FontOutline = 1;
		editMenuBox.AddItem("Edit 0");
		editMenuBox.AddItem("Edit 1");
		editMenuBox.AddItem("Edit 2");
		editMenuBox.EventClicked.Connect(() => Unigine.Console.OnscreenMessageLine($"Menubar: {editMenuBox.CurrentItemText}"));
		menuBar.SetItemMenu(1, editMenuBox);
		// add help menubox
		WidgetMenuBox helpMenuBox = new WidgetMenuBox(gui);
		helpMenuBox.FontSize = fontSize;
		helpMenuBox.FontOutline = 1;
		helpMenuBox.AddItem("Help 0");
		helpMenuBox.AddItem("Help 1");
		helpMenuBox.AddItem("Help 2");
		helpMenuBox.EventClicked.Connect(() => Unigine.Console.OnscreenMessageLine($"Menubar: {helpMenuBox.CurrentItemText}"));
		menuBar.SetItemMenu(2, helpMenuBox);
		// add menu to current gui
		gui.AddChild(menuBar, Gui.ALIGN_OVERLAP);
		Unigine.Console.Onscreen = true;
	}
	private void Shutdown()
	{
		// remove menu from current gui
		Gui.GetCurrent().RemoveChild(menuBar);
		Unigine.Console.Onscreen = false;
	}
}
```
---
## WidgetsRadioButtons.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "e4bb213d04b6d5fb523bd6459d00da4eb01a4354")]
public class WidgetsRadioButtons : Component
{
	[ShowInEditor]
	vec2 widgetPosition = new vec2(600, 450);
	[ShowInEditor]
	int fontSize = 16;
	[ShowInEditor]
	int horizontaLayoutSpace = 4;
	[ShowInEditor]
	int verticalLayoutSpace = 4;
	[ShowInEditor]
	string firstButtontext = "Check Me";
	[ShowInEditor]
	string secondButtontext = "Or Me";
	WidgetVBox _verticalLayout = null;
	WidgetCheckBox _firstCheckBox = null;
	WidgetCheckBox _secondCheckBox = null;
	bool _consoleOnScreenState = false;
	void Init()
	{
		var gui = Gui.GetCurrent();
		_verticalLayout = new WidgetVBox(horizontaLayoutSpace, verticalLayoutSpace)
		{
			PositionX = (int)widgetPosition.x,
			PositionY = (int)widgetPosition.y,
			Background = 1 // 1 = true
		};
		gui.AddChild(_verticalLayout, Gui.ALIGN_OVERLAP);
		_firstCheckBox = new WidgetCheckBox(firstButtontext)
		{
			Checked = true, // Set the first checkbox as selected by default
			FontSize = fontSize
		};
		_verticalLayout.AddChild(_firstCheckBox, Gui.ALIGN_LEFT);
		_firstCheckBox.EventClicked.Connect(() =>
		{
			if (_firstCheckBox.Checked)
				Console.OnscreenMessageLine("Radio buttons: first option");
		});
		;
		_secondCheckBox = new WidgetCheckBox(secondButtontext)
		{
			FontSize = fontSize
		};
		_verticalLayout.AddChild(_secondCheckBox, Gui.ALIGN_LEFT);
		_firstCheckBox.AddAttach(_secondCheckBox); // Attach the second checkbox to the first to group them as radio buttons
		_secondCheckBox.EventClicked.Connect(() =>
		{
			if (_secondCheckBox.Checked)
				Console.OnscreenMessageLine("Radio buttons: second option");
		});
		_consoleOnScreenState = Console.Onscreen;
		Console.Onscreen = true; // Enable to see messages in the viewport
	}
	void Shutdown()
	{
		Console.Onscreen = _consoleOnScreenState; // Restore to default state to avoid breaking logic in other worlds
		_verticalLayout.DeleteLater();
	}
}
```
---
## WidgetsScroll.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "392801192f57f7a4df95deb30d4194768970815d")]
public class WidgetsScroll : Component
{
	public int x = 500;
	public int y = 150;
	private WidgetScroll scroll = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create scroll
		scroll = new WidgetScroll(gui);
		scroll.SetPosition(x, y);
		scroll.Orientation = 0;
		scroll.EventChanged.Connect(() => Unigine.Console.OnscreenMessageLine($"Scroll: {scroll.Value}"));
		// add scroll to current gui
		gui.AddChild(scroll, Gui.ALIGN_OVERLAP);
		Unigine.Console.Onscreen = true;
	}
	private void Shutdown()
	{
		// remove scroll from current gui
		Gui.GetCurrent().RemoveChild(scroll);
		Unigine.Console.Onscreen = false;
	}
}
```
---
## WidgetsSlider.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "9b7d1ea8fa6871e7b0adc61acaba5eb108e9485c")]
public class WidgetsSlider : Component
{
	public int x = 600;
	public int y = 150;
	public int width = 100;
	public int height = 50;
	public int buttonWidth = 30;
	private WidgetSlider slider = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create slider
		slider = new WidgetSlider(gui);
		slider.Width = width;
		slider.Height = height;
		slider.ButtonWidth = buttonWidth;
		slider.SetPosition(x, y);
		slider.EventChanged.Connect(() => Unigine.Console.OnscreenMessageLine($"Slider: {slider.Value}"));
		// add slider to current gui
		gui.AddChild(slider, Gui.ALIGN_OVERLAP);
		Unigine.Console.Onscreen = true;
	}
	private void Shutdown()
	{
		// remove slider from current gui
		Gui.GetCurrent().RemoveChild(slider);
		Unigine.Console.Onscreen = false;
	}
}
```
---
## WidgetsSpinbox.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "8cfeb36765e8199a63759c31666c67dcc5cde18b")]
public class WidgetsSpinbox : Component
{
	public int x = 625;
	public int y = 300;
	private WidgetSpinBox spinBox = null;
	private WidgetEditLine spinBoxLine = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create spinbox line
		spinBoxLine = new WidgetEditLine(gui, "0");
		spinBoxLine.SetPosition(x, y);
		spinBoxLine.FontOutline = 1;
		// add spinbox line to current gui
		gui.AddChild(spinBoxLine, Gui.ALIGN_OVERLAP);
		// create spinbox
		spinBox = new WidgetSpinBox(gui);
		spinBox.Order = spinBoxLine.Order + 1;
		spinBoxLine.AddAttach(spinBox);
		spinBox.EventChanged.Connect(() => Unigine.Console.OnscreenMessageLine($"Spinbox: {spinBox.Value}"));
		// add spinbox to current gui
		gui.AddChild(spinBox, Gui.ALIGN_OVERLAP);
		Unigine.Console.Onscreen = true;
	}
	private void Shutdown()
	{
		// remove spinbox line and spinbox from current gui
		Gui.GetCurrent().RemoveChild(spinBox);
		Gui.GetCurrent().RemoveChild(spinBoxLine);
		Unigine.Console.Onscreen = false;
	}
}
```
---
## WidgetsSprite.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "32f19f98ae229e174f6d248a5302ca6153aee04c")]
public class WidgetsSprite : Component
{
	public int x = 275;
	public int y = 450;
	public int width = 100;
	public int height = 50;
	[ParameterFile]
	public string spriteImage = "";
	private WidgetSprite sprite = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create sprite
		sprite = new WidgetSprite(gui, spriteImage);
		sprite.Width = width;
		sprite.Height = height;
		sprite.SetPosition(x, y);
		// add sprite to current gui
		gui.AddChild(sprite, Gui.ALIGN_OVERLAP);
	}
	private void Shutdown()
	{
		// remove sprite form current gui
		Gui.GetCurrent().RemoveChild(sprite);
	}
}
```
---
## WidgetsTargetMarker.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "af5207a29175e299b8a56b24fc09bc334ae4d3f6")]
public class WidgetsTargetMarker : Component
{
	[ShowInEditor] public AssetLink arrowSprite;
	[ShowInEditor] public AssetLink pointSprite;
	[ShowInEditor] public Node target;
	[ShowInEditor] private vec2 pointSpritePivot = new vec2(0.5f, 0.5f);
	private WidgetSprite arrow;
	private WidgetSprite point;
	private Player camera;
	private int pointWidth;
	private int pointHeight;
	private int arrowWidth;
	private int arrowHalfWidth;
	private int arrowHeight;
	private int arrowHalfHeight;
	private void Init()
	{
		if (!arrowSprite.IsFileExist)
		{
			Log.ErrorLine("WidgetsTargetMarker.Init(): Source file for the pointer sprite image is not found.");
			return;
		}
		arrow = new WidgetSprite(arrowSprite.Path);
		WindowManager.MainWindow.AddChild(arrow, Gui.ALIGN_OVERLAP);
		if (!pointSprite.IsFileExist)
		{
			Log.ErrorLine("WidgetsTargetMarker.Init(): Source file for the marker sprite image is not found.");
			return;
		}
		point = new WidgetSprite(pointSprite.Path);
		WindowManager.MainWindow.AddChild(point, Gui.ALIGN_OVERLAP);
		if (!target)
		{
			Log.ErrorLine("WidgetsTargetMarker.Init(): No target object specified.");
			return;
		}
		camera = node as Player;
		if (!camera)
		{
			Log.ErrorLine("WidgetsTargetMarker.Init(): Camera is not valid.");
			return;
		}
	}
	private void Update()
	{
		if (!arrow || !point || !camera || !target)
			return;
		arrowWidth = arrow.GetLayerWidth(0);
		arrowHalfWidth = arrowWidth / 2;
		arrowHeight = arrow.GetLayerHeight(0);
		arrowHalfHeight = arrowHeight / 2;
		pointWidth = point.GetLayerWidth(0);
		pointHeight = point.GetLayerHeight(0);
		mat4 translation = MathLib.Translate(new vec3(-pointWidth * pointSpritePivot.x, -pointHeight * pointSpritePivot.y, 0.0f) * WindowManager.MainWindow.DpiScale);
		point.Transform = translation;
		arrow.Hidden = true;
		point.Hidden = true;
		int width = WindowManager.MainWindow.ClientSize.x;
		int height = WindowManager.MainWindow.ClientSize.y;
		int halfWidth = width / 2;
		int halfHeight = height / 2;
		int x = 0;
		int y = 0;
		vec3 targetDirection = new vec3(target.WorldBoundBox.Center - camera.WorldPosition);
		bool behind = MathLib.Dot(camera.GetWorldDirection(), targetDirection) < 0;
		if(!behind)
		{
			camera.GetScreenPosition(out x, out y, target.WorldBoundBox.Center, width, height);
			x -= halfWidth;
			y -= halfHeight;
			y *= -1;
		}
		else
		{
			vec3 inverseScreenPlaneNormal = new vec3(camera.ViewDirection * -1);
			vec3 relativeToCameraTargetPosition = (vec3)(target.WorldBoundBox.Center - camera.WorldPosition);
			// ortho projection of vector relativeToCameraTargetPosition to vector inverseScreenPlaneNormal
			vec3 orthoProjectionTarget = inverseScreenPlaneNormal * MathLib.Dot(relativeToCameraTargetPosition, inverseScreenPlaneNormal);
			vec3 reflectedTargetPosition = (vec3)((relativeToCameraTargetPosition - orthoProjectionTarget * 2) + camera.WorldPosition);
			camera.GetScreenPosition(out x, out y, reflectedTargetPosition, width, height);
			x -= halfWidth;
			y -= halfHeight;
			if (y > 0)
				y *= -1;
		}
		if (!behind && x >= -halfWidth && x <= halfWidth && y >= -halfHeight && y <= halfHeight)
		{
			point.Hidden = false;
			point.SetPosition(x + halfWidth, -y + halfHeight);
		}
		else
		{
			int point_x = 0;
			int point_y = 0;
			GetIntersectionWithScreenRect(out point_x, out point_y, x, y, halfWidth, halfHeight);
			float angle = 0.0f;
			if (halfHeight - MathLib.Abs(point_y) <= arrowHalfHeight && halfWidth - MathLib.Abs(point_x) <= arrowHalfWidth)
			{
				float dx, dy;
				if (point_y > 0)
					dy = point_y - (halfHeight - arrowHalfWidth);
				else
					dy = point_y + (halfHeight - arrowHalfWidth);
				if (point_x > 0)
					dx = point_x - (halfWidth - arrowHalfWidth);
				else
					dx = point_x + (halfWidth - arrowHalfWidth);
				angle = -MathLib.Atan2(dy, dx) * MathLib.RAD2DEG;
				if (point_x > 0)
					point_x = halfWidth - arrowWidth;
				else
					point_x = -halfWidth;
				if (point_y > 0)
					point_y = halfHeight;
				else
					point_y = -halfHeight + arrowHeight;
			}
			else if (point_y == halfHeight)
			{
				point_x -= arrowHalfWidth;
				angle = -90;
			}
			else if (point_y == -halfHeight)
			{
				point_y += arrowHeight;
				point_x -= arrowHalfWidth;
				angle = 90;
			}
			else if (point_x == -halfWidth)
			{
				point_y += arrowHalfHeight;
				angle = 180;
			}
			else if (point_x == halfWidth)
			{
				point_x -= arrowWidth;
				point_y += arrowHalfHeight;
				angle = 0;
			}
			arrow.Hidden = false;
			arrow.SetPosition(point_x + halfWidth, -point_y + halfHeight);
			mat4 rotation = new mat4(
				MathLib.Translate(new vec3(arrowHalfWidth, arrowHalfHeight, 0.0f) * WindowManager.MainWindow.DpiScale) *
				MathLib.Rotate(new quat(vec3.UP, angle)) *
				MathLib.Translate(new vec3(-arrowHalfWidth, -arrowHalfHeight, 0.0f) * WindowManager.MainWindow.DpiScale)
			);
			arrow.Transform = rotation;
		}
	}
	private void Shutdown()
	{
		arrow.DeleteLater();
		point.DeleteLater();
	}
	private void GetIntersectionWithScreenRect(out int x, out int y, int vec_x, int vec_y, int halfWidth, int halfHeight)
	{
		if (vec_y >= 0)
		{
			if (vec_y == 0)
			{
				if (vec_x > 0)
				{
					x = halfWidth;
					y = 0;
				}
				else
				{
					x = -halfWidth;
					y = 0;
				}
				return;
			}
			x = (int)(halfHeight * ((float)vec_x / (float)vec_y));
			y = halfHeight;
			if (x >= -halfWidth && x <= halfWidth)
				return;
			if (vec_x >= 0)
			{
				if (vec_x == 0)
				{
					x = 0;
					y = halfHeight;
					return;
				}
				x = halfWidth;
				y = (int)(halfWidth * ((float)vec_y / (float)vec_x));
				return;
			}
			else
			{
				x = -halfWidth;
				y = (int)(-halfWidth * ((float)vec_y / (float)vec_x));
				return;
			}
		}
		else
		{
			x = (int)(-halfHeight * ((float)vec_x / (float)vec_y));
			y = -halfHeight;
			if (x >= -halfWidth && x <= halfWidth)
				return;
			if (vec_x >= 0)
			{
				if (vec_x == 0)
				{
					x = 0;
					y = -halfHeight;
					return;
				}
				x = halfWidth;
				y = (int)(halfWidth * ((float)vec_y / (float)vec_x));
				return;
			}
			else
			{
				x = -halfWidth;
				y = (int)(-halfWidth * ((float)vec_y / (float)vec_x));
				return;
			}
		}
	}
}
```
---
## WidgetsTreebox.cs
```csharp
﻿using Unigine;
[Component(PropertyGuid = "16c02db76d472faacda41b01b39fb86c019258db")]
public class WidgetsTreebox : Component
{
	public int x = 775;
	public int y = 300;
	public int fontSize = 16;
	private WidgetTreeBox treeBox = null;
	private void Init()
	{
		Gui gui = Gui.GetCurrent();
		// create treebox
		treeBox = new WidgetTreeBox(gui);
		treeBox.SetPosition(x, y);
		treeBox.FontSize = fontSize;
		treeBox.FontOutline = 1;
		// add first parent and children
		treeBox.AddItem("parent 0");
		treeBox.AddItem("child 0");
		treeBox.AddItem("child 1");
		treeBox.AddItem("child 2");
		treeBox.AddItemChild(0, 1);
		treeBox.AddItemChild(0, 2);
		treeBox.AddItemChild(0, 3);
		// add second parent and children
		treeBox.AddItem("parent 1");
		treeBox.AddItem("child 0");
		treeBox.AddItem("child 1");
		treeBox.AddItem("child 2");
		treeBox.AddItemChild(4, 5);
		treeBox.AddItemChild(4, 6);
		treeBox.AddItemChild(4, 7);
		treeBox.EventChanged.Connect(() => Unigine.Console.OnscreenMessageLine($"Treebox: {treeBox.CurrentItemText}"));
		// add treebox to current gui
		gui.AddChild(treeBox, Gui.ALIGN_OVERLAP);
		Unigine.Console.Onscreen = true;
	}
	private void Shutdown()
	{
		// remove tree box from current gui
		Gui.GetCurrent().RemoveChild(treeBox);
		Unigine.Console.Onscreen = false;
	}
}
```
---
## Window.cs
```csharp
﻿using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "5dbaec014a558a2f578248cdc24add62a2707e73")]
public class Window : Component
{
	private WidgetWindow window;
	void Init()
	{
		Gui gui = Gui.GetCurrent();
		window = new WidgetWindow(gui, "Hello from C#", 4, 4);
		window.Flags = Gui.ALIGN_OVERLAP | Gui.ALIGN_CENTER;
		window.Width = 320;
		window.Sizeable = true;
		var editline = new WidgetEditLine(gui, "Edit me");
		editline.Flags = Gui.ALIGN_EXPAND;
		window.AddChild(editline);
		editline.EventChanged.Connect(widget => {
			WidgetEditLine el = widget as WidgetEditLine;
			Log.Message("EditLine changed: {0}\n", el.Text);
		});
		editline.FontSize = 16;
		var button = new WidgetButton(gui, "Press me");
		button.Flags = Gui.ALIGN_EXPAND;
		window.AddChild(button);
		button.EventClicked.Connect(() => Log.Message("Button pressed\n"));
		button.FontSize = 18;
		window.Arrange();
		Gui.GetCurrent().AddChild(window);
		Console.Onscreen = true;
	}
	void Shutdown()
	{
		Console.Onscreen = false;
		window.DeleteLater();
	}
}
```
---
## XmlSample.cs
```csharp
﻿using System;
using System.Collections;
using System.Collections.Generic;
using Unigine;
[Component(PropertyGuid = "44c94e2c465fd46226bdd1babce978afdced818c")]
public class XmlSample : Component
{
	float onscreenTime;
	void Init()
	{
		Unigine.Console.Onscreen = true;
		onscreenTime = Unigine.Console.OnscreenTime;
		Unigine.Console.OnscreenTime = 100.0f;
		Log.Message("\n");
		// create the XML tree
		Xml xml = xml_create();
		// print xml tree
		xml_print(xml, 0);
	}
	void Shutdown()
	{
		Unigine.Console.Onscreen = false;
		Unigine.Console.OnscreenTime = onscreenTime;
	}
	private Xml xml_create()
	{
		Xml xml = new Xml("node");
		Xml xml_0 = xml.AddChild("child", "arg=\"0\"");
		Xml xml_1 = xml_0.AddChild("child", "arg=\"1\"");
		Xml xml_2 = xml_1.AddChild("child", "arg=\"2\"");
		xml_2.Data = "data";
		xml.APIInterfaceOwner = false;
		return xml;
	}
	private static void xml_print(Xml xml, int offset)
	{
		for (int i = 0; i < offset; i++)
		{
			Log.Message(" ");
		}
		Log.Message("{0}: ", xml.Name);
		for (int i = 0; i < xml.NumArgs; i++)
		{
			Log.Message("{0}={1} ", xml.GetArgName(i), xml.GetArgValue(i));
		}
		Log.Message(": {0}\n", xml.Data);
		for (int i = 0; i < xml.NumChildren; i++)
		{
			xml_print(xml.GetChild(i), offset + 1);
		}
	}
}
```
---
## ZoomController.cs
```csharp
﻿using Unigine;
#region Math Variables
#if UNIGINE_DOUBLE
using Vec3 = Unigine.dvec3;
using Mat4 = Unigine.dmat4;
#else
using Vec3 = Unigine.vec3;
using Mat4 = Unigine.mat4;
#endif
#endregion
[Component(PropertyGuid = "d303a5eb1cf092c71065f658f6fe6e9724e6cd81")]
public class ZoomController : Component
{
	private float defaultFOV = 60.0f;
	private float defaultDistanceScale = 1.0f;
	private float defaultSensivity = 1.0f;
	private float defaultPlayerTurning = 90.0f;
	private Player player;
	private void Init()
	{
		player = node as Player;
		if (!player)
		{
			Log.Error("ZoomSample::init cannot cast node to player!\n");
		}
		defaultFOV = player.Fov;
		defaultDistanceScale = Render.DistanceScale;
		defaultSensivity = ControlsApp.MouseSensitivity;
		if (node.Type == Node.TYPE.PLAYER_SPECTATOR)
		{
			PlayerSpectator playerSpectator = player as PlayerSpectator;
			defaultPlayerTurning = playerSpectator.Turning;
		}
		if (node.Type == Node.TYPE.PLAYER_ACTOR)
		{
			PlayerActor playerActor = player as PlayerActor;
			defaultPlayerTurning = playerActor.Turning;
		}
	}
	private void Shutdown()
	{
		//so settings won't be affected between sessions
		Render.DistanceScale = defaultDistanceScale;
		ControlsApp.MouseSensitivity = defaultSensivity;
	}
	public void FocusOnTarget(Node target)
	{
		Vec3 dir = target.WorldPosition - node.WorldPosition;
		dir.Normalize();
		player.ViewDirection = (vec3)dir;
	}
	public void UpdateZoomFactor(float zoomFactor)
	{
		player.Fov = defaultFOV / zoomFactor;
		Render.DistanceScale = defaultDistanceScale * zoomFactor;
		ControlsApp.MouseSensitivity = defaultSensivity / zoomFactor;
		if (node.Type == Node.TYPE.PLAYER_SPECTATOR || node.Type == Node.TYPE.PLAYER_ACTOR)
		{
			UpdateTurning(zoomFactor);
		}
	}
	private void UpdateTurning(float zoomFactor)
	{
		// Turning determines speed at which the Player is rotated. It should be lowered and heightened depending on zoom factor
		// ZoomController has been made for the base Player class so it could work with any Player derived class But not every Player has a Turning property
		// Because of that we should regulate Turning depeping on player node type.
		// There is no work around this situation and since changing behavior depending on class type is bad practice this functionality has been hidden from public interface
		if (node.Type == Node.TYPE.PLAYER_SPECTATOR)
		{
			PlayerSpectator playerSpectator = player as PlayerSpectator;
			playerSpectator.Turning = defaultPlayerTurning / zoomFactor;
		}
		if (node.Type == Node.TYPE.PLAYER_ACTOR)
		{
			PlayerActor player_actor = player as PlayerActor;
			player_actor.Turning = defaultPlayerTurning / zoomFactor;
		}
	}
	public void Reset()
	{
		UpdateZoomFactor(1);
	}
}
```
---
## ZoomSample.cs
```csharp
﻿using System.Globalization;
using Unigine;
using static Unigine.Input;
[Component(PropertyGuid = "63bf2eeb11e75a752c591032808e2fa7197514e0")]
public class ZoomSample : Component
{
	[ShowInEditor]
	[Parameter(Title = "Target one node")]
	private Node targetOneNode = null;
	[ShowInEditor]
	[Parameter(Title = "Target two node")]
	private Node targetTwoNode = null;
	[ShowInEditor]
	[Parameter(Title = "Target three node")]
	private Node targetThreeNode = null;
	[ShowInEditor]
	[Parameter(Title = "Zoom controller")]
	private ZoomController zoom = null;
	private SampleDescriptionWindow window = null;
	private WidgetLabel fov_label = null;
	private WidgetLabel mouse_sensivity_label = null;
	private WidgetLabel render_scale_label = null;
	private WidgetSlider sliderZoom = null;
	[ShowInEditor]
	[Parameter(Title = "Player Camera")]
	private Player player = null;
	private Input.MOUSE_HANDLE mouse_handle = Input.MOUSE_HANDLE.USER;
	private void Init()
	{
		InitComponents();
		mouse_handle = Input.MouseHandle;
		Input.MouseHandle = Input.MOUSE_HANDLE.SOFT;
		window = new SampleDescriptionWindow();
		window.createWindow();
		Widget parameters = window.getParameterGroupBox();
		Widget grid = parameters.GetChild(0);
		sliderZoom = window.addFloatParameter("Zoom:", "Zoom", 1, 1, 100.0f, (float val) =>
		{
			zoom.UpdateZoomFactor(val);
			fov_label.Text = "FOV:" + player.Fov.ToString("0.00", CultureInfo.InvariantCulture) + " deg";
			mouse_sensivity_label.Text = "Mouse sensivity:" + ControlsApp.MouseSensitivity.ToString("0.000", CultureInfo.InvariantCulture) + " deg";
			render_scale_label.Text = "Render distance scale:" + Render.DistanceScale.ToString("0.00", CultureInfo.InvariantCulture) + " deg";
		});
		fov_label = new WidgetLabel("FOV:" + player.Fov.ToString("0.00", CultureInfo.InvariantCulture) + " deg");
		fov_label.Width = 100;
		parameters.AddChild(fov_label, Gui.ALIGN_LEFT);
		mouse_sensivity_label = new WidgetLabel("Mouse sensivity:" + ControlsApp.MouseSensitivity.ToString("0.000", CultureInfo.InvariantCulture) + " deg");
		mouse_sensivity_label.Width = 100;
		parameters.AddChild(mouse_sensivity_label, Gui.ALIGN_LEFT);
		render_scale_label = new WidgetLabel("Render distance scale:" + Render.DistanceScale.ToString("0.00", CultureInfo.InvariantCulture) + " deg");
		render_scale_label.Width = 100;
		parameters.AddChild(render_scale_label, Gui.ALIGN_LEFT);
		var resetButton = new WidgetButton("Reset");
		parameters.AddChild(resetButton, Gui.ALIGN_LEFT);
		resetButton.EventClicked.Connect(() =>
		{
			sliderZoom.Value = 1 * 100;
		});
		WidgetHBox hbox = new WidgetHBox();
		hbox.SetSpace(2, 1);
		parameters.AddChild(hbox, Gui.ALIGN_LEFT);
		var focusLabel = new WidgetLabel("Focus on:");
		hbox.AddChild(focusLabel);
		var targetOne = new WidgetButton("Target 1");
		hbox.AddChild(targetOne);
		targetOne.EventClicked.Connect(() =>
		{
			zoom.FocusOnTarget(targetOneNode);
		});
		var targetTwo = new WidgetButton("Target 2");
		hbox.AddChild(targetTwo);
		targetTwo.EventClicked.Connect(() =>
		{
			zoom.FocusOnTarget(targetTwoNode);
		});
		var targetThree = new WidgetButton("Target 3");
		hbox.AddChild(targetThree);
		targetThree.EventClicked.Connect(() =>
		{
			zoom.FocusOnTarget(targetThreeNode);
		});
	}
	private void Shutdown()
	{
		Input.MouseHandle = mouse_handle;
		window.shutdown();
	}
	void InitComponents()
	{
		if (!zoom)
		{
			Log.Error("ZoomSample.Init.InitComponents zoom controller is not assigned !\n");
		}
		if (!player)
		{
			Log.Error("ZoomSample.Init.InitComponents player is not assigned!\n");
		}
		if (!targetOneNode || !targetTwoNode || !targetThreeNode)
		{
			Log.Error("ZoomSample.Init.InitComponents targets are not assigned!\n");
		}
	}
}
```
---