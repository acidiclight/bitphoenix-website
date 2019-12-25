---
title: 'The Peacenet Complete Refactor: Day 1'
date: 2019-12-25T02:25:16.946Z
description: >-
  Did you guys really think I was going to give up The Peacenet that easily?
  Sure, I've talked about it in the Discord that I've thought of it, but that
  doesn't mean I'm not willing to give it one last go.  On this very merry
  Christmas Eve, I'm setting out on a goal to COMPLETELY refactor the game's
  Unreal codebase to be a lot less like spaghetti and more like lasagna.  And
  here's the results of Day 1.
---
## The pasta analogy

You may have heard the term "Spaghetticode" before.  I'd like to extend that term into *"the pasta analogy,"* referring to two main types of codebases.

 - **Spaghetti code:** Describes code that's heavily intertwined and tangled together.  Making a change anywhere could potentially break the whole damn thing.  It's very hard to add new features and remove old/unnecessary code.  This is what Peacenet's code is like.

 - **Lasagna code:** The code is built in layers.  Adding, removing, or changing something in a layer should only affect the layer above it if anything.  As such, it is much easier to maintain.  This is what I'd like Peacenet's code to be like.

## So what's the plan?

Overall, the plan is to:

1. Remove any unnecessary "top-layer" code - GUI programs, terminal commands, exploits, payloads, computer services, protocol implementations, missions, anything that's just going to get in our way as we completely rework the core.  We can add them back later.  They're not necessary for the game to function.
2. Start using Unreal's **Game Framework** classes to build the framework for Peacenet's core.
3. Introduce concepts like computing devices, in place of system contexts, to the game that use the new core framework.
4. Remove unneccessary bottom-layer code - Peacenet World State Actor, stuff like that. By now, this functionality is built into the new core framework.
5. Add all top-layer content back!

Seems like a simple plan on paper, right?  Yeah... Screw you.  Let me tell you why it's not.  But let me also tell you how I'm doing it anyway.

## Removing unnecessary content from the game

This is easy.  I basically just batch-deleted all exploits, payloads, services, implementations, missions, and any other asset related to hacking within the Unreal editor.  Then, I started deleting any non-essential programs.

Deleting all the non-essential programs from the game left me with **Terminal,** **Icewolf,** **Upgrades,** **Editor,** and **Network View**.  These are the only programs I need to be able to control the game and know things are working.

But that's not all.  After batch-deleting all the hacking assets from the game, I decided to give things a little bit of a speed boost in the compiler department by removing their UE4 data asset classes as well.  This lead to a huge headache of constant recompiling as I had to remove every function in the game that used them, and any function in the game that uses any function I remove.  Yeah... spaghetti code isn't fun to work with.  Thankfully, I did all of this a week ago.

## Using the Game Framework

Now that a lot of the unneccessary code has been removed from the game, I started to look into using the Game Framework classes in Unreal to build a new Peacenet core framework.

Ideally, this core framework will be entirely written in C++ and virtually unaware of exactly what the Peacegate layer, and subsequently, the GUI, is doing.  This is our first step to a lasagna codebase!

The Game Framework helps us achieve this by providing us with various base classes we can use to build our games inside.  These include:

 - `GameMode`: The rules of the game.
 - `GameState`: What's currently happening in the game.
 - `HUD`: The player's screen.
 - `PlayerController`: The player.
 - `AIController`: An artificial intelligence.
 - `Pawn`: Something that either the player or an AI can control.
 - `PlayerState`: What the player's doing.

Seems pretty straightforward right? These classes work together to build the framework of your game.  Hence why they're called the Game Framework!  So how do we use them?

### Game Mode

We'll start with the game mode.  This is the first game framework class that's ever spawned in by Unreal Engine and handles spawning in everything else for us.  I created my own `GameMode` and defined some game rules for Peacenet.  It's C++ header looks like this:

```
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/GameMode.h"
#include "Blueprint/UserWidget.h"
#include "PeacenetGameState.h"
#include "DesktopWidget.h"
#include "MainMenuWidget.h"
#include "PeacenetGameMode.generated.h"

UCLASS(BlueprintType)
class PROJECTOGLOWIA_API APeacenetGameMode : public AGameModeBase {
    GENERATED_BODY()

public:
    APeacenetGameMode(const FObjectInitializer& ObjectInitializer);

public:
    UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "User interface")
    TSubclassOf<UMainMenuWidget> MainMenuWidget;

    UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "User interface")
    TSubclassOf<UDesktopWidget> DesktopEnvironment;

    UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "User interface")
    TSubclassOf<UWindow> WindowDecorationWidget;

public:
    UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Gameplay")
    UCommandInfo* ShellCommand = nullptr;

    UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Gameplay")
    bool EnableMissionSystem = false;

    UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Gameplay")
    bool EnableTutorials = false;

    UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Gameplay")
    bool EnableSystemUpgrades = false;

    UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Gameplay")
    int MaximumNonPlayerCharacters = 10;

    UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Gameplay")
    int MaximumCorporateNetworks = 2;

protected:
    virtual void InitGame(const FString& MapName, const FString& Options, FString& ErrorMessage) override;
};
```

And the source code:

```
#include "PeacenetGameMode.h"
#include "PeacenetHUD.h"
#include "PeacenetPlayerController.h"
#include "CommonUtils.h"

APeacenetGameMode::APeacenetGameMode(const FObjectInitializer& ObjectInitializer) {
    this->bPauseable = false;
    this->bStartPlayersAsSpectators = false;
    this->bUseSeamlessTravel = false;

    this->HUDClass = APeacenetHUD::StaticClass();
    this->PlayerControllerClass = APeacenetPlayerController::StaticClass();
    this->GameStateClass = APeacenetGameState::StaticClass();
}

void APeacenetGameMode::InitGame(const FString& MapName, const FString& Options, FString& ErrorMessage) {
    // Check that all the UI bullshit is valid.
    if(!this->MainMenuWidget->IsValidLowLevel()) {
        ErrorMessage = "Missing main menu widget.";
    }

    if(!this->WindowDecorationWidget->IsValidLowLevel()) {
        ErrorMessage = "Missing window decorator widget.";
    }

    if(!this->DesktopEnvironment->IsValidLowLevel()) {
        ErrorMessage = "Missing desktop environment.";
    }

    Super::InitGame(MapName, Options, ErrorMessage);
}
```

Not too bad! All this does is define some rules for the game that can be set in the Unreal editor - things like max NPC counts, whether different gameplay elements are enabled, and which widgets should be used for things like the desktop UI.  The source code simply makes sure that things that need to be set to valid values, are.  We also tell Unreal which classes we'd like to use for the rest of the Game Framework.

### HUD

I was over-simplifying when I said that the HUD was the player's screen.  In Peacenet, it's used like that.  But in other Unreal games...not really.  But for our use case, and for all intents in purposes, this is the player's screen.  Each player has their own HUD.  Even though Peacenet's a single player game, it makes sense that we do all of our UI stuff in here so each player has their own UI.

And, this is the header for Peacenet's HUD class:

```
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/HUD.h"
#include "Blueprint/UserWidget.h"
#include "MainMenuWidget.h"
#include "PeacenetHUD.generated.h"

UCLASS()
class PROJECTOGLOWIA_API APeacenetHUD : public AHUD {
    GENERATED_BODY()

private:
    UPROPERTY()
    UMainMenuWidget* MainMenu = nullptr;

protected:
    virtual void BeginPlay() override;
};
```

And the code:

```
#include "PeacenetHUD.h"
#include "CommonUtils.h"
#include "PeacenetGameMode.h"

void APeacenetHUD::BeginPlay() {
    // Get the game mode so that we can see what the main menu widget is.
    APeacenetGameMode* gm = Cast<APeacenetGameMode>(this->GetWorld()->GetAuthGameMode());

    if(gm) {
        // Get the main menu widget.
        TSubclassOf<UMainMenuWidget> menuWidget = gm->MainMenuWidget;

        // Create the widget.
        this->MainMenu = CreateWidget<UMainMenuWidget, APlayerController>(this->GetOwningPlayerController(), menuWidget);

        if(this->MainMenu) {
            this->MainMenu->AddToViewport();
        } else {
            UCommonUtils::PrintError("HUD: Couldn't create a valid main menu widget.  Fuck.");
        }
    } else {
        UCommonUtils::PrintError("HUD: Must be used with a PeacenetGameMode.");
    }

    Super::BeginPlay();
}
```

Literally all this does is ask the Game Mode "Hey, what widget should I use for the main menu?" and spawns it in.  If there are any errors, they're printed to the debug log and the game ceases to run.  Of course, later on, we'll also use this HUD to handle the game's desktop and windows for any open program.

### Player Controller

For now, we don't need any AI or a Player State, so I only worried about implementing a custom Player Controller.  This is the class that defines the things that the player can do.  Most games would use this to take input from an actual controller or the keyboard/mouse and do things either with the HUD or the possessed Pawn.

For now, I just need this to set the engine's input mode and mouse cursor mode to suit a UI-heavy game. Here's the header:

```
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/PlayerController.h"
#include "PeacenetPlayerController.generated.h"

UCLASS()
class PROJECTOGLOWIA_API APeacenetPlayerController : public APlayerController {
    GENERATED_BODY()

protected:
    virtual void BeginPlay() override;
};
```

And the code:

```
#include "PeacenetPlayerController.h"

void APeacenetPlayerController::BeginPlay() {
    // Input mode "Game and UI" allows UE4 to handle things like F11 and Tilde for PIE fullscreen toggle and
    // dev console access respectively.
    this->SetInputMode(FInputModeGameAndUI());

    // And this lets us see the god damn mouse.
    this->bShowMouseCursor = true;

    Super::BeginPlay();
}
```

### Game State

And this is where the magic goes.  This game state should work with the game mode to enforce game rules, and is where a lot of the gameplay logic goes.  I'm currently using it as a place to implement the save system, since ultimately the save file holds the game's state information.  So, in essence, the Game State is an abstraction between the save file and the rest of the Game Framework.  Simple, right?

I'm not showing code for this, you can find it on GitHub.  It's a lot more code than what's shown above, simply because it needs to handle the multi-save system of the old API...which it does a mighty fine job of doing already. :)

## Getting the Main Menu to work

I needed something to verify that my current Game Framework was working right, so I created a Main Menu base class that interfaces with the Game State.  It lets me load, create, delete, and list save files.  Here's its header and code:

```
#pragma once

#include "CoreMinimal.h"
#include "Blueprint/UserWidget.h"
#include "Profile.h"
#include "MainMenuWidget.generated.h"

UCLASS(Blueprintable, Abstract)
class PROJECTOGLOWIA_API UMainMenuWidget : public UUserWidget {
    GENERATED_BODY()

protected:
    UFUNCTION(BlueprintCallable, BlueprintPure)
	TArray<FProfileData> GetProfiles();

    UFUNCTION(BlueprintCallable, BlueprintPure)
    bool ProfileExists(FString InProfileName);

    UFUNCTION(BlueprintCallable)
    bool LoadProfile(FString InProfileName);

    UFUNCTION(BlueprintCallable)
    bool CreateNewProfile(FString InProfile);

    UFUNCTION(Blueprintcallable)
    bool DeleteProfile(FString InProfileName);
};
```

```
#include "MainMenuWidget.h"
#include "PeacenetGameState.h"

TArray<FProfileData> UMainMenuWidget::GetProfiles() {
    APeacenetGameState* gs = Cast<APeacenetGameState>(this->GetWorld()->GetGameState());
    return (gs != nullptr) ? gs->GetProfiles() : TArray<FProfileData>();
}

bool UMainMenuWidget::ProfileExists(FString InProfileName) {
    APeacenetGameState* gs = Cast<APeacenetGameState>(this->GetWorld()->GetGameState());
    return gs && gs->ProfileExists(InProfileName);
}

bool UMainMenuWidget::LoadProfile(FString InProfileName) {
    APeacenetGameState* gs = Cast<APeacenetGameState>(this->GetWorld()->GetGameState());
    return gs && gs->LoadProfile(InProfileName);
}

bool UMainMenuWidget::CreateNewProfile(FString InProfileName) {
    APeacenetGameState* gs = Cast<APeacenetGameState>(this->GetWorld()->GetGameState());
    return gs && gs->CreateNewProfile(InProfileName);
}

bool UMainMenuWidget::DeleteProfile(FString InProfileName) {
    APeacenetGameState* gs = Cast<APeacenetGameState>(this->GetWorld()->GetGameState());
    return gs && gs->DeleteProfile(InProfileName);
}
```

Then it was just a matter of having the actual main menu UI base itself off this class and use the functions built into this class to talk to the Game Framework.  This offloads all of the actual save system behaviour into C++ land so that the GUI never needs to know what actually happens, and the bottom layer, the Game State, never needs to know that a GUI's telling it to do things.  Awesome.

## Setting the Story Mode game rules

For now, Story Mode is the only game mode in the game - but we still need to define these rules in the Unreal editor.

This is as easy as right-clicking somewhere in the Content Browser and choosing to create a new **Blueprint Class**.  Base it off **Peacenet Game Mode**.  Then you get to set all the game rules in a friendly UI.  No code required!

I set the rules as such for story mode - these can be changed later if needed.

 - **Enable system upgrades:** yes
 - **Enable mission system:** yes
 - **Enable tutorials:** yes
 - **Command shell:** Bash
 - **Max corporate networks:** Default
 - **Max non-player characters:** Default
 - **Desktop environment:** Desktop
 - **Main menu widget:** Main Menu
 - **Window decorator:** Window Border

Then, you can set the game mode to this new Story Mode by going to **Edit** -> **Project Settings** -> **Maps and Modes** in the Unreal editor and setting the relevant setting.  Hit "Play" at the top, and bam, I was thrown right into the Peacenet main menu as if I'd not changed a thing.  But believe me, I did.

## End of day 1

At the end of day 1, we have:

 - The basics of the new Game Framework
 - Fully implemented multi-save system
 - Removed Peacenet Game Instance class
 - Functioning Main Menu

Not a bad start, and not a bad place to leave the game off for Christmas Eve. :)

Next is to introduce a concept of Computing Devices in place of System Contexts, and start getting the Peacegate Layer to sit ontop of our new Game Framework.  As this is being done, most of the functionality of the old API should be re-implemented.  I'll keep you guys updated on this as we go. :)
