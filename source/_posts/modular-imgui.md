---
title: Modular ImGUI
author: Calin Gavriliuc
tags:
- ImGUI
- GameDev
date: 2021-01-24 01:01:01
---

# A More Modular ImGUI Method

## Introduction

[ImGUI](https://github.com/ocornut/imgui) is one of the most popular graphical user interface libraries available for content creation, debugging, and visualization tools. Since this library can be used for many applications, it often becomes highly entangled and messy to work with.

As a game developer, I was tasked with implementing the main framework and a large amount of ImGUI content for a game called [Arc Apellago](https://store.steampowered.com/app/1454430/Arc_Apellago/). It was overall a simple game, but it also contained a surprising amount of ImGUI content.

![Arc Apellago ImGUI](/images/ModularImGUI/ArcApellagoImGUI.png)

## Designing the Interface

Given the design of ImGUI, content separated across windows with some content on them, how can we clean the code up?

<!-- <img src="/images/ModularImGUI/ImGuiWindow.png" alt="ImGUI Window" width="300px"/> -->
![Arc Apellago ImGUI Window](/images/ModularImGUI/ImGUIWindow.png)

For one, we can contain all the functionality and related information for our given window in its own object. So far, a window is just a context and some updated content. First, let us make a class that holds a `title`, a `showWindow_` class variable, and an `Update` to update this window's content.

A parent class that follows these requirements could look as follows:

```c++
class EditorWindow
{
public:
  explicit EditorWindow(std::string title);

  virtual ~EditorWindow();

  virtual void Update(float dt);

private:
    std::string title_;
    bool showWindow_;
};
```

When the previous image is reduced and grouped, you get the following:

![Arc Apellago ImGUI Small Window](/images/ModularImGUI/ImGUIWindowSmall.png)

This reduced-content window contains three major pieces of content:

- FPS / Display Statistics
- Debug Settings
- Collision Settings

As these are only related as configuration/stats content, we can break these up as well. To do so, let us create another class, `EditorBlock`, which can be attached to any `EditorWindow`.

```c++
class EditorBlock
{
public:
  EditorBlock() = default;

  virtual ~EditorBlock() = default;

  virtual void Update(float dt) = 0;
};
```

These blocks of content are quite simple - just an update for the content they should display.
They both allow for separation of code and allows multiple teammates' content on the same window if you are working on a team.

Next, we must adapt our current `EditorWindow` to store these `EditorBlock`s.

```c++
class EditorWindow
{
public:
  explicit EditorWindow(std::string title);

  virtual ~EditorWindow();

  virtual void Update(float dt);

  void AddBlock(EditorBlock* editorBlock);

private:
    std::string title_;
    std::vector<EditorBlock*> blocks_;
    bool showWindow_;
};
```

It should be noted that the `Update` function will now just call the `EditorBlock`s' `Update` functions.

## Creating a Block

As an example of using the blocks, let's implement the `StatsEditorBlock` which displays the following:

![Stats Editor Block](/images/ModularImGUI/ImGUIStatsBlock.png)

**Note:** The `ImGUI FPS` stat has been omitted for simplicity.

`StatsEditorBlock.h`

```c++
#include "EditorBlock.h"

class StatsEditorBlock : public EditorBlock
{
public:
    void Update(float dt) override;
};
```

Note that creating a class for each block allows it to implement helper methods and have internal state variables - which are often useful.

`StatsEditorBlock.cpp`

```c++
#include "imgui.h"
#include "StatsEditorBlock.h"

void StatsEditorBlock::Update(float dt)
{
    // Get the needed information from ImGUI
    auto io = ImGui::GetIO();
    // Display the FPS
    ImGui::Text("FPS: %f", io.Framerate);
    // Display the display size
    ImGui::Text("Display X:%d", static_cast<int>(io.DisplaySize.x));
    ImGui::Text("Display Y:%d", static_cast<int>(io.DisplaySize.y));
}
```

## Using the Interface

Now that all the functionality is abstracted out and self-contained in other classes, the interface is as simple as choosing which blocks go where.

![Arc Apellago ImGUI Window Sections](/images/ModularImGUI/ImGUIWindowBlocks.png)

```c++
// Create a window and give it a title
auto statsConfigWindow = new EditorWindow("Stats and Config");
// Add a stats block, debug draw config block, and collision config block to the window
statsConfigWindow->AddBlock(new StatsEditorBlock());
statsConfigWindow->AddBlock(new DebugDrawConfigBlock());
statsConfigWindow->AddBlock(new CollisionConfigBlock());
// Add the window to the main ImGUI update list
windows_.emplace_back(statsConfigWindow);
```

## Conclusion

This layout separates unrelated code into manageable blocks to allow for easier layout planning and code decoupling. With ImGUI's initialization and update code out of the way, it will be much easier for your teammates to create ImGUI blocks or windows as they require - this played a large role in my team's final ImGUI implementation.

---

**Some final notes and tips:**

- Use the `ifdef` paradigm to compile-out ImGUI from your project if you don't want to ship with it and _test that this define works regularly_.
- Check out the [docking](https://github.com/ocornut/imgui/tree/docking) branch of ImGUI to help organize your ImGUI windows further.

---
