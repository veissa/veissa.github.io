---
title: Snake game in C
date: 2023-12-01 09:20:11 +/-TTTT
categories: [Programming, game-dev]
tags: [c,game-dev]     # TAG names should always be lowercase
description: basic snake game using libncurses-dev and ncurses-doc libs.
author : anaddam med
---
<br>
# Using ncurses

I went through the man page for initscr, and the other functions in the `ncurses` lib. This game includes a simple menu and dynamic score display (I'll add more features later on).

## Prerequisites

Before we start, ensure you have the `ncurses` library installed. If you're using a Linux distribution, you can install it using:

```sh
sudo apt install libncurses-dev ncurses-doc
```
going through the man pages for these is recommended fr better understanding of the craft, here are the key ideas functions :<br>

- `initscr()`: Initializes the ncurses library.
- `noecho()`: Disables echoing of typed characters (We need this fr the menu stuff).
- `cbreak()`: Disables line buffering.
- `mvprintw()`: Prints text at a specified position.
- `scanw()`: Reads formatted input.
- `newwin()`: Creates a new window.
- `keypad()`: Enables the keypad for the window :
      it takes 2 parameters, the window and a boolean `keypad(win, true)` This function enables the keypad for the specified window (win). When enabled, wgetch() can capture special keys like arrow keys and function keys
- `nodelay()`: Makes `wgetch()` non-blocking.
- `wgetch()`: Gets a character from the window.
- `werase()`: Erases the window.
- `usleep()`: Suspends execution for microsecond intervals.
- `mvwaddch()`: Adds a character to the window at a specified position.
- `wrefresh()`: Refreshes the window to update the screen.
<br>

importing the libs, the `unistd.h` stands for UNIX Standard, meaning the POSIX (Portable Operating System Interface) API, I needed this to use the `usleep()` syscall, it suspends the execution of the calling thread for a specified number of microseconds, in simple words, I needed it to make the snake move at a controlled speed.

```c
#include <ncurses.h>
#include <stdlib.h>
#include <unistd.h>

void start_game();
```
<br> Creating our menu part  :

```c
int main() {
    initscr();
    noecho();
    cbreak();

    int choice;
    while (1) {
        clear();
        mvprintw(0, 0, "Choose an option:\n");
        mvprintw(1, 0, "1 - Start\n");
        mvprintw(2, 0, "2 - About\n");
        mvprintw(3, 0, "3 - Hall of Fame\n");
        mvprintw(4, 0, "4 - Rules\n");
        mvprintw(5, 0, "5 - Exit\n");
        mvprintw(6, 0, "Please enter your choice: ");
        refresh();
        scanw("%d", &choice);

        if (choice == 1) {
            start_game();
        } else if (choice == 5) {
            break;
        } else {
            mvprintw(8, 0, "Option not implemented yet.");
            refresh();
            sleep(2);
        }
    }

    endwin();
    return 0;
}
```
<br>
Implementing the game function :

```c

void start_game() {
    WINDOW* win = newwin(20, 40, 1, 1);
    keypad(win, true);
    nodelay(win, true);
    int posX = 0, posY = 0;
    int foodX = rand() % 20, foodY = rand() % 20; // this is not actually random, but again this is basic fr nw
    int dirX = 1, dirY = 0;
    int score = 0;

    while (true) {
        int pressed = wgetch(win); //getting the arrow keys and crafting the logic of movements
        if (pressed == KEY_LEFT) {
            dirX = -1;
            dirY = 0;
        }
        if (pressed == KEY_RIGHT) {
            dirX = 1;
            dirY = 0;
        }
        if (pressed == KEY_UP) {
            dirX = 0;
            dirY = -1;
        }
        if (pressed == KEY_DOWN) {
            dirX = 0;
            dirY = 1;
        }

        posX += dirX;
        posY += dirY;
        //staying in the same boundary 
        if (posX < 0) posX = 19;
        if (posX > 19) posX = 0;
        if (posY < 0) posY = 19;
        if (posY > 19) posY = 0;

        werase(win);
        usleep(100000);
        mvwaddch(win, posY, posX, '*'); // the sanke fr now
        mvwaddch(win, foodY, foodX, 'X'); // the food 
        mvwprintw(win, 0, 0, "Score: %d", score); // the score 

        if (foodX == posX && foodY == posY) { //the logic fr the food management etc ,,
            score++;
            foodX = rand() % 20;
            foodY = rand() % 20;
        }

        wrefresh(win);
    }
}
```
<br>

## Happy debugging

Main Function
The main function initializes the ncurses library and sets up the main menu loop. The user can choose from several options, but only the "Start" and "Exit" options are implemented fr nw .

start_game Function
The start_game function contains the game logic. It creates a new window for the game, handles user input for controlling the snake, and updates the game state.

Movement: The snake's direction is controlled using the arrow keys.

Food: The snake eats food represented by 'X', and the score increases.

Score: The score is displayed at the top of the game window.

## enough fr nw

This basic snake game demonstrates how to use the ncurses library for creating text-based games in C. 
ofcrs I'll add new fetures etc later on, having fun with ncurses ~ and reading through man pages takes a while ...