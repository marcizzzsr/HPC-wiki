# Tmux
**Tmux** (Terminal Multiplexer) is a program that enables users to create and manage multiple terminal sessions from a single window. It's a powerful tool for developers, system administrators, and anyone who frequently uses the command line.

## Key Uses

*   **Session Management:** Tmux allows you to detach from a session and reattach to it later, even after you've closed your terminal or lost your network connection. This is incredibly useful for long-running processes, as it prevents them from being terminated if your connection drops.
    
*   **Pane Splitting:** You can split the terminal window into multiple panes, each running a separate shell. This lets you monitor logs, edit code, and run commands side-by-side without needing to switch between different terminal windows or tabs.
    
*   **Window Management:** Within a single tmux session, you can create multiple windows, each with its own set of panes. This helps organize different tasks or projects within the same terminal environment.

## Usage
You can find more about this tool and a helpful how-to guide [here](https://github.com/tmux/tmux/wiki/Getting-Started#about-this-document)

* * *

# Byobu
**Byobu** is an open-source text-based window manager and terminal multiplexer. It's not a standalone multiplexer itself, but rather a set of enhancements built on top of either **GNU Screen** or **Tmux**. Its primary goal is to provide a more user-friendly and feature-rich experience out of the box, with a focus on simplifying the often complex configuration and keybindings of its back-end tools.

## Key Uses and Features

*   **Simplifies Session Management:** Byobu provides a more intuitive interface for managing sessions, windows, and panes. It uses simple function key shortcuts (like F2 for a new window, F6 to detach) that are often easier to remember than the multi-key combinations of Tmux or Screen.
    
*   **On-screen Status Notifications:** A key feature of Byobu is its informative status bar at the bottom of the screen. This bar can display a wide range of real-time system information, such as CPU usage, memory consumption, IP address, system load, available updates, and more.
    
*   **Improved User Interface:** Byobu provides a clean, elegant visual layout that makes it easy to see which window you are in and to navigate between them. It was originally designed to enhance the experience of connecting to remote servers, providing a consistent look and feel across different systems.
    
*   **Flexible Back-end:** Byobu is unique in that it can use either GNU Screen or Tmux as its underlying engine. While Tmux is now the default back-end in newer versions, users can switch between them, allowing for a consistent experience regardless of which is available on a given system.

## Usage
Byobu guide exists as a man page, so you can access it by running `man byobu` directly in your terminal. If you prefer an online version, you can find it [here](https://www.byobu.org/documentation).
