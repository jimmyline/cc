#include <stdio.h>
#include <stdlib.h>
#include <conio.h>
#include <time.h>
#include <windows.h>

#define ROWS 6
#define COLS 6

// Warna ANSI untuk terminal
void setColor(int num) {
    switch (num) {
        case 1: printf("\033[0;31m"); break;  // Red
        case 2: printf("\033[0;32m"); break;  // Green
        case 3: printf("\033[0;34m"); break;  // Blue
        case 4: printf("\033[0;33m"); break;  // Yellow
    }
}

void resetColor() {
    printf("\033[0m");
}

// Fungsi untuk menampilkan grid
void displayGrid(int grid[ROWS][COLS], int selectedRow, int selectedCol) {
    system("cls");
    for (int i = 0; i < ROWS; i++) {
        for (int j = 0; j < COLS; j++) {
            if (i == selectedRow && j == selectedCol) {
                printf(":");
                setColor(grid[i][j]);
                printf("[%d]", grid[i][j]);
                resetColor();
                printf(":");
            } else {
                setColor(grid[i][j]);
                printf(" [%d] ", grid[i][j]);
                resetColor();
            }
        }
        printf("\n");
    }
}

// Fungsi untuk menukar dua angka
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

// Fungsi untuk menggeser angka ke bawah dan memberi efek animasi
void dropNumbersWithAnimation(int grid[ROWS][COLS]) {
    int hasDropped;
    do {
        hasDropped = 0;
        for (int j = 0; j < COLS; j++) {
            for (int i = ROWS - 1; i > 0; i--) {
                if (grid[i][j] == 0 && grid[i - 1][j] != 0) {  // Jika ada angka yang hilang
                    grid[i][j] = grid[i - 1][j];  // Geser angka ke bawah
                    grid[i - 1][j] = 0;
                    hasDropped = 1;
                    displayGrid(grid, -1, -1);  // Tampilkan perubahan
                    Sleep(200);  // Jeda animasi
                }
            }
        }
    } while (hasDropped);

    // Setelah semua angka bergeser, tambahkan angka baru di atas
    for (int j = 0; j < COLS; j++) {
        for (int i = 0; i < ROWS; i++) {
            if (grid[i][j] == 0) {
                grid[i][j] = rand() % 4 + 1;  // Masukkan angka baru
                displayGrid(grid, -1, -1);
                Sleep(200);
            }
        }
    }
}

// Fungsi untuk mengecek apakah ada 3 angka yang sama dalam baris atau kolom
int checkMatches(int grid[ROWS][COLS], int *score) {
    int found = 0;

    // Cek baris untuk tiga angka yang sama
    for (int i = 0; i < ROWS; i++) {
        for (int j = 0; j < COLS - 2; j++) {
            if (grid[i][j] == grid[i][j + 1] && grid[i][j] == grid[i][j + 2] && grid[i][j] != 0) {
                *score += 3;  // Tambah poin
                grid[i][j] = grid[i][j + 1] = grid[i][j + 2] = 0;  // Hilangkan angka
                found = 1;
            }
        }
    }

    // Cek kolom untuk tiga angka yang sama
    for (int j = 0; j < COLS; j++) {
        for (int i = 0; i < ROWS - 2; i++) {
            if (grid[i][j] == grid[i + 1][j] && grid[i][j] == grid[i + 2][j] && grid[i][j] != 0) {
                *score += 3;  // Tambah poin
                grid[i][j] = grid[i + 1][j] = grid[i + 2][j] = 0;  // Hilangkan angka
                found = 1;
            }
        }
    }

    return found;
}

int main() {
    srand(time(0));  // Untuk randomisasi angka

    int grid[ROWS][COLS] = {
        {2, 1, 2, 1, 2, 4},
        {3, 4, 4, 2, 4, 3},
        {4, 4, 2, 4, 4, 1},
        {2, 2, 4, 4, 1, 4},
        {1, 2, 4, 3, 1, 2},
        {3, 3, 1, 4, 3, 1}
    };

    int selectedRow = 0, selectedCol = 0;  // Posisi awal
    int targetRow = 0, targetCol = 0;
    char input;
    int score = 0, moves = 15;  // Jumlah moves tersisa

    // Tampilkan grid dan navigasi menggunakan WASD
    while (moves > 0) {
        displayGrid(grid, selectedRow, selectedCol);
        printf("Current Score : %d\n", score);
        printf("Moves Left    : %d\n", moves);

        input = getch();
        if (input == 'w' && selectedRow > 0) {
            selectedRow--;
        } else if (input == 's' && selectedRow < ROWS - 1) {
            selectedRow++;
        } else if (input == 'a' && selectedCol > 0) {
            selectedCol--;
        } else if (input == 'd' && selectedCol < COLS - 1) {
            selectedCol++;
        } else if (input == '\r') {  // Enter key untuk memilih angka yang akan diganti
            targetRow = selectedRow;
            targetCol = selectedCol;
            printf("Now, choose where to swap the number!\n");

            while (1) {
                input = getch();
                if (input == 'w' && targetRow > 0) {
                    targetRow--;
                } else if (input == 's' && targetRow < ROWS - 1) {
                    targetRow++;
                } else if (input == 'a' && targetCol > 0) {
                    targetCol--;
                } else if (input == 'd' && targetCol < COLS - 1) {
                    targetCol++;
                } else if (input == '\r') {  // Enter lagi untuk konfirmasi pertukaran
                    if ((targetRow == selectedRow && (targetCol == selectedCol - 1 || targetCol == selectedCol + 1)) ||
                        (targetCol == selectedCol && (targetRow == selectedRow - 1 || targetRow == selectedRow + 1))) {
                        swap(&grid[selectedRow][selectedCol], &grid[targetRow][targetCol]);
                        moves--;  // Kurangi jumlah move
                    }
                    break;
                }
                displayGrid(grid, targetRow, targetCol);  // Tampilkan target
            }
        }

        // Cek apakah ada angka yang sama dan hilang
        if (checkMatches(grid, &score)) {
            dropNumbersWithAnimation(grid);  // Geser angka ke bawah dengan animasi dan masukkan angka baru
        }
    }

    printf("Game Over! Final Score: %d\n", score);
    return 0;
}

