# Unreal Engine Behaviour Tree Tutorial

## Author: Minsi Chen

## Introduction

Behaviour Trees in Unreal Engine provide developers an asset-based approach to designing and implementing intelligence into non-player characters in video games.
The screenshot below shows a behaviour tree representing the actions that can be executed by a non-player enemy character.

![An example behaviour tree.](/figures/behavior-tree-quick-start-step-3-12.png "An example behaviour tree.")
*An example behaviour tree representing an enemy NPC's actions. Source: <https://dev.epicgames.com/documentation/en-us/unreal-engine/behavior-tree-in-unreal-engine---quick-start-guide>*

In this tutorial session, you will be given a vehicle project as a starting point.
Our aim is to introduce a simply chase behaviour to the computer controlled vehicles. YEA BUDDEEEEEE

The programming part of this tutorial is C++ based.
However, you do not need any prior coding experience in C++.
You will be given the code snippet and the tasks set in this document will guide you through the code base of this project.


## Learning outcomes:

Working through this activity, you should be able to:
- Understand the basic components of a behaviour tree
- Configure a working behaviour tree with simple tasks and services
- Extend the start-up project in your own time and at your own pace

## Activity 1: Getting acquainted with Unreal Engine Editor and Visual Studio

<details>

<summary> Task 1: Open the VehicleStarter project in Unreal Engine Editor</summary>

1. Open the Unreal Engine Editor by double clickingt the "Unreal Engine 5.4.3" shortcut on the desktop

2. When presented with the following dialog box, click Browse to open the VehicleStarter project. The project should be in the C:\Work folder.

![](/figures/UE5_open_existing_project_UI.png)

3. Unreal Engine may need to convert the project. If you see the following dialog box, select More Options then click "Convert in-place".

![](/figures/UE5_project_conversion_UI.png)

4. Unreal Engine Editor should now convert and open the project. This may take a few minutes. Once the project is loaded, you should see a window similar to the screenshot below without the additional cars. You can play the level by clicking the green triangle play button.
![](/figures/UE5_editor.png)

</details>

<details>

<summary> Task 2: Adding an NPC vehicle to the map </summary>

1. Open the Content Browser by clicking the "Content Browser" button located at the bottom left corner of the editor. This will bring up the asset for this project as shown in the following screenshot

![](/figures/UE5_content_browser.png)

2. To add an NPC vehicle to the map, drag the "AIStarterWheeledVehiclePawn" from the Content Browser into the level. You can place it anywhere you like. This process spawns an instance of AI vehicle but the vehicle will not do anything if you hit play.

3. The NPC vehicle does not have a torque and AI controller setup by default. To add a torque curve, select the vehicle you just added and use the detail panel on the right to assign a torque curve as shown in the image below.

![](/figures/UE5_BT_adjust_torque.png)

4. Finally, we need setup the AI vehicle to use our own custom AIController. To do this, we will use the detail panel to change the AI Controller Class to AIWheeledVehicleController, see the screenshot below.

![](/figures/UE5_changing_aicontroller.png)

5. If you hit play now, you should see the AI vehicle making a left hand turn at full throttle. This is not very useful but it verifies that our controller and engine torque are setup.
</details>

<details>

<summary> Task 3: Explore the VehicleStarter project source code in Visual Studio </summary>

At this point, you should have the project loaded for Activity 2.
The purpose of this task is to get you familiarise with Visual Studio which will be used later for adding additional functionality into the project.
Additionally, it also shows you how these source files are related to the game objects and asset you have seen so far.

To open the Visual Studio project, click Tools on the menu bar located at the top of the Unreal Engine Editor, then select Open Visual Studio. You should see Visual Studio similar to the screenshot below.

![](/figures/UE5_VS_UI.png)

The panel on the left is known as the Solution Explore which shows all the files related to this project.
The files we are interested in are all located under Games->VehicleStart->Source->VehicleStarter, e.g. those files with .cpp and .h extension.

This may look overwhelming at first glance, but we can ignore all the implementation details in this tutorial.

There is also a close relationship between the source code and the vehicle you have seen in the editor.
For example, the player controlled vehicle is actually a class called `StarterWheeledVehiclePawn` defined in the source file `StarterWheeledVehiclePawn.h` and `StarterWheeledVehiclePawn.cpp`.

Similarly, the AI controlled vehicle is represented by the class `AIStarterWheeledVehiclePawn`.
**Can you locate the source files for this class in the Solution Explorer?** 
</details>

## Activity 2: Implementing simple steering behaviour using behaviour trees and blackboard

The objective of this activity is to implement a simple steering behaviour for the NPC vehicle.
This simple behaviour should enable our computer controlled vehicle to track and pursue the player vehicle.

<details>

<summary> Task 1: Calculating the steering angle and driving direction </summary>

This task does not require any programming work.
Instead, you will use your maths knowledge to construct a solution to the steering and tracking problem.

See the figure below.
![](/figures/vehicle.png)

Assuming the position of the player and NPC vehicle are known, and you are also given the forward direction and right direction of the NPC vehicle.
Our objective is to use the most appropriate vector product to determine the followings:
- is the player vehicle to the left or the right side of the NPC vehicle. This will later determines the steering direction.
- is the player vehicle in front or behind the NPC vehicle. This will later determine if forward or reverse throttle is needed.
- the Euclidean distance between the two vehicles

Hint: you will need to use vector arithmetic and dot product.
</details>

<details>

<summary> Task 2: Implementing the steering and throttling as behaviour tree tasks and services </summary>
<blockquote>

Assuming you have derived a set of formulae for determining steering direction and throttle.
This task will show you its implementation in C++.

The behaviour tree we are building consists of one service responsible for making decision on steering and throttle.
The steering and throttling actions are implemented as two behaviour tasks.

Your task is to copy the following three code snippets into the body of their designated class and method.

  <details>

  <summary> Click here to see the Steering Service code snippet </summary>
  <blockquote>
  In Visual Studio, open the <mark>BTSteeringService.cpp</mark> file from the Solution Explorer.
  Copy and paste the following code snippet into the body of <mark>void UBTSteeringService::TickNode(UBehaviorTreeComponent & OwnerComp, uint8 * NodeMemory, float DeltaSeconds)</mark> method.


  ```c++
  UWorld* world = OwnerComp.GetWorld();
  TActorIterator<AStarterWheeledVehiclePawn> PlayerPawnIter(world);
  PlayerPawn = *PlayerPawnIter;

  if (PlayerPawn)
  {
    APawn* OwnerPawn;// = OwnerComp.GetOwner();
    AAIWheeledVehicleController* AIVehicle = Cast<AAIWheeledVehicleController>(OwnerComp.GetAIOwner());
    OwnerPawn = AIVehicle->GetPawn();
    FVector ForwardVector = OwnerPawn->GetActorForwardVector();
    FVector RightVector = OwnerPawn->GetActorRightVector();
    FVector PlayerLocation = PlayerPawn->GetActorLocation();
    FVector TargetDirection = PlayerLocation - OwnerPawn->GetActorLocation();
    
    float Distance = FVector::DistSquared(PlayerLocation, OwnerPawn->GetActorLocation());
    float Throttle = FMath::Clamp<float>(0.0f, 1.0f, Distance / 5000.0f);
    TargetDirection.Normalize(1.05f);
    float AngleValue = FVector::DotProduct(ForwardVector, TargetDirection);
    float RightValue = FVector::DotProduct(RightVector, TargetDirection);

    float SteerValue = 0.0f; //-RightValue;// < 0.0f ? 1.0f - AngleValue : -1.0f + AngleValue;
    if (RightValue < 0.0f)
    {
      SteerValue = AngleValue < 0.0f ? -1.0f : RightValue;
    }
    else
    {
      SteerValue = AngleValue < 0.0f ? 1.0f : RightValue;
    }

    AIVehicle->BlackboardComp->SetValueAsVector("PlayerLocation", OwnerPawn->GetActorLocation());
    AIVehicle->BlackboardComp->SetValueAsFloat("SteeringValue", SteerValue);
    AIVehicle->BlackboardComp->SetValueAsFloat("ThrottleValue", AngleValue < 0 ? -.8f : Throttle);
  }
  ```
  </blockquote>
  </details>

  <details>

  <summary> Click here to see the Steering Task code snippet </summary>
  <blockquote>
  In Visual Studio, open the <mark>BTTaskSteerVehicle.cpp</mark> file from the Solution Explorer.
  Copy and paste the following code snippet into the body of <mark>EBTNodeResult::Type UBTTaskSteerVehicle::ExecuteTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory)</mark> method.

  ```c++
  AAIWheeledVehicleController* AIController = Cast<AAIWheeledVehicleController>(OwnerComp.GetAIOwner());

  if (AIController)
  {
    UBlackboardComponent* Blackboard = OwnerComp.GetBlackboardComponent();
    float Steering;
    
    if (Blackboard->HasValidAsset())
    {
      Steering = Blackboard->GetValueAsFloat("SteeringValue");
    }
    else
    {
      Steering = FMath::RandRange(-1.0f, 1.0f);
    }
    
    AIController->VehicleMovementComp->SetSteeringInput(Steering);

    return EBTNodeResult::Succeeded;
  }
  ```
  </blockquote>
  </details>
    <details>

  <summary> Click here to see the Vehicle Throttle Task code snippet </summary>
  <blockquote>
  In Visual Studio, open the <mark>BTTaskThrottle.cpp</mark> file from the Solution Explorer.
  Copy and paste the following code snippet into the body of <mark>EBTNodeResult::Type UBTTaskThrottle::ExecuteTask(UBehaviorTreeComponent & OwnerComp, uint8 * NodeMemory)</mark> method.
  
  ```c++
  AAIWheeledVehicleController* AIController = Cast<AAIWheeledVehicleController>(OwnerComp.GetAIOwner());

  if (AIController)
  {
    UBlackboardComponent* Blackboard = OwnerComp.GetBlackboardComponent();
    float Throttle;
    
    if (Blackboard->HasValidAsset())
    {
      Throttle = Blackboard->GetValueAsFloat("ThrottleValue");
    }
    else 
    {
      Throttle = FMath::RandRange(-1.0f, 1.0f);
    }
    
    if (Throttle > 0.0f)
    {
      AIController->VehicleMovementComp->SetThrottleInput(Throttle);
      AIController->VehicleMovementComp->SetBrakeInput(0.0f);
    }
    else 
    {
      AIController->VehicleMovementComp->SetThrottleInput(0);
      AIController->VehicleMovementComp->SetBrakeInput(-Throttle);
    }
    return EBTNodeResult::Succeeded;
  }
  ```
  </blockquote>
  </details>

<details>
<summary> Compile the newly added source code </summary>

<blockquote>
Once the code snippets are in the right place, you can compile the source in the Unreal Engine Editor.
The compile button is located near the bottom right corner of the Unreal Engine editor see the image below.

![](/figures/UE5_compile_project.png)

Note, we do not normally use compile the source inside Visual Studio as Unreal Engine has its own build and live-coding facility.

If there is any compiler error, you will see a pop-up dialog showing all the errors.
If compilation is successful, proceed to the next task.
</blockquote>
</details>
</blockquote>
</details>

<details>

<summary> Task 3: Examine the behaviour tree asset in Unreal Engine Editor </summary>
The behaviour tree used in this tutorial has already been setup.
You can find it in the Content Browser under Content->VehicleTemplate as shown in the screenshot below.

![](/figures/UE5_content_BT.png)

Double clicking the BTAIImproved behaviour tree in the content browser will show you its content similar to the screenshot below.

![](/figures/UE5_BT_sample.png)

This behaviour tree consists of two leaf nodes at the bottom which execute the steering and throttling tasks implemented in the previous task.
The node below the ROOT node contains our Steering Service which is used to track the player and determine steering angles and throttle application.
These values are written onto a Blackboard (see the image below) which are then communicated to the two tasks 

![](/figures/UE5_blackboard.png)

Once you have checked through these assets, we can move onto the final task.
</details>

<details>

<summary> Task 4: Test the behaviour tree on the AI controlled vehicle </summary>
Before testing the behaviour tree, we need to ascertain the AI controlled vehicle is using the correct behaviour tree and controller.
Select the AI vehicle you added in the previous activity and check its setting in the Detail panel.
Change the behaviour tree and AI controller settings so that they are identical to the circled area in the following screenshot.

![](/figures/UE5_content_BT_annotated.png)

If they look correct, you can hit the play button and test the game.
The AI vehicle should now try to drive towards you and keeps pursuing the player vehicle.

Note, you can add more AI vehicle by simply duplicating the vehicle that are already added to the map.

</details>

## Summary

This tutorial shows you the basic workflow of creating autonomous behaviours using Unreal Engine's behaviour tree system.
Whilst the resultant behaviour is limited, this should provide you with a foundation for further exploration.
There are a few areas you can improve in your own time:
- Make the AI vehicle to priortise driving forward rather than simply backing into the player
- Take into account of the vehicle dimension when determining the relative position between the vehicles
- Make the AI vehicle to drive away from the player rather than pursuing
- Enable the AI vehicle to avoid collision with the player

You can also further develop this into a check point racing or vehicle combat game.

Finally, if you wish to learn more about Behaviour Tree, please check out the starter guide <https://dev.epicgames.com/documentation/en-us/unreal-engine/behavior-tree-in-unreal-engine---quick-start-guide>.
