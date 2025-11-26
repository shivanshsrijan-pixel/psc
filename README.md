#include <stdio.h>
#include <stdlib.h>

#define WIDTH 40          // Width of the game screen
#define HEIGHT 20         // Height of the game screen
#define MAX_INVADERS 12   // Number of invaders in the game

//---------------------------------------------------------------
// Function: drawScreen
// Purpose : Redraws the entire game screen ONCE per user action.
//---------------------------------------------------------------
void drawScreen(
    int ship_x, int ship_y,         // Player ship position
    int bullet_x, int bullet_y,     // Bullet position
    int inv_x[], int inv_y[],       // Invader positions
    int alive[]                     // Whether each invader is alive
) {
    system("clear");                // Clears terminal (use "cls" on Windows)

    // Loop over every row of the screen
    for (int y = 0; y < HEIGHT; y++) {

        // Loop over every column of the screen
        for (int x = 0; x < WIDTH; x++) {

            int printed = 0;        // Tracks whether we printed something here

            // ---- INVADER DRAWING ----
            for (int i = 0; i < MAX_INVADERS; i++) {
                // If this invader is alive AND positioned at (x,y) → draw it
                if (alive[i] && inv_x[i] == x && inv_y[i] == y) {
                    putchar('W');   // 'W' represents an invader
                    printed = 1;
                    break;          // Stop checking other invaders
                }
            }

            // ---- BULLET DRAWING ----
            if (!printed && bullet_x == x && bullet_y == y) {
                putchar('|');       // '|' represents the bullet
                printed = 1;
            }

            // ---- SHIP DRAWING ----
            if (!printed && ship_x == x && ship_y == y) {
                putchar('^');       // '^' represents the player's ship
                printed = 1;
            }

            // ---- EMPTY SPACE ----
            if (!printed)
                putchar(' ');
        }
        putchar('\n');              // Move to next row
    }
}

//---------------------------------------------------------------
// MAIN GAME FUNCTION
//---------------------------------------------------------------
int main() {

    // ---- SHIP START POSITION ----
    int ship_x = WIDTH / 2;         // Middle of the screen horizontally
    int ship_y = HEIGHT - 2;        // Near the bottom

    // ---- BULLET INITIAL STATE ----
    int bullet_x = -1;              // -1 means "no bullet currently active"
    int bullet_y = -1;

    // ---- INVADER ARRAYS ----
    int inv_x[MAX_INVADERS];        // Invader X positions
    int inv_y[MAX_INVADERS];        // Invader Y positions
    int alive[MAX_INVADERS];        // 1 = alive, 0 = dead

    // ---- INITIALIZE INVADERS ----
    for (int i = 0; i < MAX_INVADERS; i++) {
        inv_x[i] = 2 + (i % 6) * 6;     // Spread them horizontally
        inv_y[i] = 2 + (i / 6) * 2;     // Two rows
        alive[i] = 1;                   // All start alive
    }

    int turn = 0;                       // Counts turns (used for timed movement)
    char move;                          // Stores player input

    // ---- DRAW THE INITIAL SCREEN ----
    drawScreen(ship_x, ship_y, bullet_x, bullet_y, inv_x, inv_y, alive);

    //===============================================================
    // GAME LOOP — executes ONCE per player input (turn-based)
    //===============================================================
    while (1) {

        // Ask user for a movement command
        printf("\nMove (A=left  D=right  W=shoot  Q=quit): ");
        scanf(" %c", &move);            // Read one character (+ space avoids leftover newline)

        // ---- QUIT GAME ----
        if (move == 'q' || move == 'Q')
            break;

        // ---- SHIP MOVEMENT ----
        if (move == 'a' && ship_x > 0)
            ship_x--;                   // Move left
        if (move == 'd' && ship_x < WIDTH - 1)
            ship_x++;                   // Move right

        // ---- SHOOTING ----
        // Only allow a shot if no bullet is currently active
        if (move == 'w' && bullet_y < 0) {
            bullet_x = ship_x;          // Start bullet at ship position
            bullet_y = ship_y - 1;      // One row above the ship
        }

        // ---- BULLET MOVEMENT ----
        if (bullet_y >= 0)
            bullet_y--;                 // Move bullet upward

        // If bullet goes off-screen, reset it (-1 means "inactive")
        if (bullet_y < 0)
            bullet_x = -1;

        // ---- INVADER MOVEMENT ----
        // Move invaders downward every 3 turns
        if (turn % 3 == 0) {
            for (int i = 0; i < MAX_INVADERS; i++) {

                // Only move if alive
                if (alive[i])
                    inv_y[i]++;

                // If ANY invader reaches the ship's row → game over
                if (alive[i] && inv_y[i] >= ship_y) {
                    system("clear");
                    printf("GAME OVER — Invaders reached your position!\n");
                    return 0;
                }
            }
        }

        // ---- BULLET COLLISION DETECTION ----
        for (int i = 0; i < MAX_INVADERS; i++) {
            if (alive[i] &&
                bullet_x == inv_x[i] &&
                bullet_y == inv_y[i]) {

                alive[i] = 0;           // Kill invader
                bullet_y = -1;          // Remove bullet
                bullet_x = -1;
            }
        }

        // ---- CHECK FOR WIN ----
        int remaining = 0;
        for (int i = 0; i < MAX_INVADERS; i++)
            if (alive[i]) remaining++;

        if (remaining == 0) {
            system("clear");
            printf("YOU WIN — All invaders destroyed!\n");
            return 0;
        }

        turn++;                         // Increase turn counter

        // ---- REDRAW SCREEN ONE TIME ----
        drawScreen(ship_x, ship_y, bullet_x, bullet_y, inv_x, inv_y, alive);
    }

    return 0;
}
