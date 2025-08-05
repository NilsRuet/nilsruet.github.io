---
title: "Ydis - Live in-game statistics tracker"
summary: "An app that tracks player stats in the game Geometry Dash"
tags: ["C#", "WPF", "Reverse engineering"]
draft: false
---

## What is Ydis?
**Ydis** [(link)](https://github.com/exyl-exe/ydis) is a (today archived) Windows application that automatically attaches itself to the game _Geometry Dash_, a rhythm/platforming game, and **collects various statistics about the player's performance**. Features include tracking individual level statistics such as **total play time, total attempts, and detailed performance per section of the level**, with the aim of providing **insight into areas of improvement for the player**.
This application was released publicly and has been used by at least a few hundred users, although it is now obsolete due to a lack of maintenance and the availibility of better alternative/tools.


{{< side-by-side src1="/images/ydis/ui-1.png" src2="/images/ydis/ui-2.png" caption="Live data visualization">}}

## Technical summary
- **Language**: C#
- **UI Framework**: WPF
- **Architecture**: Model-View-ViewModel (MVVM)
- **Data collection**: Windows API, non-intrusive reading (reverse engineering)
- **Data format**: JSON serialization

## Details and design choices

The application has two main critical features, data collection and presentation, as well as stability constraints that influenced the technical design.

_Geometry Dash_ doesn't provide any API or method to interact with the game, meaning that any data from the game has to be collected through memory reading or modding. Being my first publicly released project, I chose to make the application incapable of crashing the game or hindering the player experience by design. As such, all of the collected data is gathered through memory reading only, ensuring the game is unmodified and essentially unable to malfunction because of the app. This requires reverse-engineering the locations of several variables of interest in memory to detect what is happening in the game.

The core of the data collection feature is done through periodically accessing key variables in the game's memory, and inferring the various events that occur during gameplay (e.g., the player started a level, started an attempt, exited the level, etc.). A class called `GameWatcher` is responsible for invoking the relevant C# events, while another class called `Recorder` builds the data structure based on the invoked events. The collected data is then serialized and can be used by the visualization feature.

The entire application is made in C#, with WPF used for the UI. The main motivation for these technologies was their compatibility with the already Windows-only nature of the project, as well as my objective to be proficient with them.
The application has two main critical features, data collection and presentation, as well as stability constraints that motivate the technical design.

## Main challenges and roadblocks
- Locating in-game variables using static analysis (Ghidra) and runtime debugging (Cheat Engine)
- Inferring in-game events based on changes in memory values over time
- Designing a UI that remains responsive with large datasets
- Managing the full lifecycle of a user-facing application: design, release, communication and feedback handling.

## Takeaways
- Gained hands-on experience with reverse engineering, C# and WPF
- Learned the full process of designing, building, publishing and maintaining a complete application
- In hindsight, a few design choices would be reconsidered:
    - Low-level operations like memory reading should be abstracted into a framework or library to avoid depending on a specific platform or game version.
    - UI should account for large-scale data from the start, to avoid performance and accessibility issues later
