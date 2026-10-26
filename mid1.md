```c
#include "raylib.h"
#include <stdbool.h>
#include <stdlib.h>
#include <time.h>

#define SCREEN_WIDTH 800
#define SCREEN_HEIGHT 600
#define ROWS 7
#define COLS 10
#define BLOCK_WIDTH 75
#define BLOCK_HEIGHT 25
#define BLOCK_PADDING 5
#define OFFSET_X 35
#define OFFSET_Y 50
#define MAX_BALLS 5

// 이벤트 종류 정의
typedef enum {
    EVENT_NONE = 0,
    EVENT_SCORE_BONUS,  // 점수 추가
    EVENT_WIDEN_PADDLE, // 패들 확장
    EVENT_SPEED_UP,     // 공 속도 증가
    EVENT_MULTI_BALL    // 멀티볼 생성
} EventType;

typedef struct {
    Rectangle rect;
    Color color;
    bool isActive;
    EventType eventType;
} Block;

typedef struct {
    Vector2 position;
    Vector2 velocity;
    float radius;
    bool isActive;
    Color color;
} Ball;

typedef struct {
    Rectangle rect;
    float speed;
    Color color;
    float originalWidth;
} Paddle;

typedef enum {
    GAME_STATE_READY,
    GAME_STATE_PLAYING,
    GAME_STATE_GAMEOVER,
    GAME_STATE_CLEAR
} GameState;

// 무지개 색상 반환 함수
Color GetRainbowColor(int row) {
    Color rainbow[ROWS] = {
        RED,
        ORANGE,
        YELLOW,
        GREEN,
        BLUE,
        (Color) {
 75, 0, 130, 255
}, // Indigo
PURPLE
    };
    return rainbow[row % ROWS];
}

// 이벤트 빈도를 낮추어 무작위 혹은 특정 조건으로 할당하는 함수
EventType DetermineBlockEvent(int row, int col) {
    // 전체 블록 중 약 20% 확률로만 이벤트 부여, 혹은 특정 행/열에 드물게 배치
    // 여기서는 무작위 난수를 활용하되 확률을 낮춤 (예: 15~20% 확률)
    int randomValue = rand() % 100; // 0 ~ 99

    if (randomValue < 15) { // 15퍼센트 확률로 이벤트 당첨
        int eventTypeChoice = rand() % 4;
        switch (eventTypeChoice) {
        case 0: return EVENT_SCORE_BONUS;
        case 1: return EVENT_WIDEN_PADDLE;
        case 2: return EVENT_SPEED_UP;
        case 3: return EVENT_MULTI_BALL;
        }
    }
    return EVENT_NONE;
}

// 게임 초기화
void InitializeGame(Ball balls[MAX_BALLS], int* ballCount, Paddle* paddle, Block blocks[ROWS][COLS], GameState* state, int* score) {
    *score = 0;
    *ballCount = 1;

    for (int i = 0; i < MAX_BALLS; i++) {
        balls[i].position = (Vector2){ SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT - 60.0f };
        balls[i].velocity = (Vector2){ 5.0f, -5.0f };
        balls[i].radius = 8.0f;
        balls[i].isActive = (i == 0);
        balls[i].color = WHITE;
    }

    paddle->originalWidth = 100.0f;
    paddle->rect = (Rectangle){ SCREEN_WIDTH / 2.0f - paddle->originalWidth / 2.0f, SCREEN_HEIGHT - 40.0f, paddle->originalWidth, 15.0f };
    paddle->speed = 8.0f;
    paddle->color = LIGHTGRAY;

    for (int r = 0; r < ROWS; r++) {
        for (int c = 0; c < COLS; c++) {
            blocks[r][c].rect = (Rectangle){
                OFFSET_X + c * (BLOCK_WIDTH + BLOCK_PADDING),
                OFFSET_Y + r * (BLOCK_HEIGHT + BLOCK_PADDING),
                BLOCK_WIDTH,
                BLOCK_HEIGHT
            };
            blocks[r][c].color = GetRainbowColor(r);
            blocks[r][c].isActive = true;
            blocks[r][c].eventType = DetermineBlockEvent(r, c);
        }
    }
    *state = GAME_STATE_READY;
}

// 패들 이동 조건 캡슐화
bool IsPaddleOutOfBounds(Rectangle paddleRect, float nextX) {
    return (nextX < 0 || nextX + paddleRect.width > SCREEN_WIDTH);
}

// 이벤트 발동 처리 함수
void TriggerBlockEvent(EventType eventType, Paddle* paddle, Ball balls[MAX_BALLS], int* ballCount, int* score) {
    switch (eventType) {
    case EVENT_SCORE_BONUS:
        *score += 300;
        break;
    case EVENT_WIDEN_PADDLE:
        paddle->rect.width = 140.0f;
        break;
    case EVENT_SPEED_UP:
        for (int i = 0; i < MAX_BALLS; i++) {
            if (balls[i].isActive) {
                balls[i].velocity.x *= 1.05f;
                balls[i].velocity.y *= 1.05f;
            }
        }
        break;
    case EVENT_MULTI_BALL:
        for (int i = 0; i < MAX_BALLS; i++) {
            if (!balls[i].isActive) {
                balls[i].position = (Vector2){ SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT / 2.0f };
                balls[i].velocity = (Vector2){ (rand() % 2 == 0 ? 4.0f : -4.0f), -5.0f };
                balls[i].isActive = true;
                (*ballCount)++;
                break;
            }
        }
        break;
    default:
        break;
    }
}

// 게임 상태 업데이트 로직
void UpdateGame(Ball balls[MAX_BALLS], int* ballCount, Paddle* paddle, Block blocks[ROWS][COLS], GameState* state, int* score) {
    if (*state == GAME_STATE_READY) {
        if (IsKeyPressed(KEY_SPACE) || IsMouseButtonPressed(MOUSE_BUTTON_LEFT)) {
            *state = GAME_STATE_PLAYING;
        }
        return;
    }

    if (*state == GAME_STATE_GAMEOVER || *state == GAME_STATE_CLEAR) {
        if (IsKeyPressed(KEY_ENTER)) {
            InitializeGame(balls, ballCount, paddle, blocks, state, score);
        }
        return;
    }

    // 패들 이동 처리
    float paddleMovement = 0.0f;
    if (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A)) paddleMovement -= paddle->speed;
    if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D)) paddleMovement += paddle->speed;

    float nextPaddleX = paddle->rect.x + paddleMovement;
    if (!IsPaddleOutOfBounds(paddle->rect, nextPaddleX)) {
        paddle->rect.x = nextPaddleX;
    }

    // 공 업데이트
    int activeBalls = 0;
    for (int i = 0; i < MAX_BALLS; i++) {
        if (!balls[i].isActive) continue;
        activeBalls++;

        balls[i].position.x += balls[i].velocity.x;
        balls[i].position.y += balls[i].velocity.y;

        // 벽 충돌 (좌우)
        if (balls[i].position.x - balls[i].radius <= 0 || balls[i].position.x + balls[i].radius >= SCREEN_WIDTH) {
            balls[i].velocity.x *= -1.0f;
        }

        // 벽 충돌 (상단)
        if (balls[i].position.y - balls[i].radius <= 0) {
            balls[i].velocity.y *= -1.0f;
        }

        // 바닥 충돌
        if (balls[i].position.y - balls[i].radius >= SCREEN_HEIGHT) {
            balls[i].isActive = false;
            (*ballCount)--;
            continue;
        }

        // 패들 충돌
        if (CheckCollisionCircleRec(balls[i].position, balls[i].radius, paddle->rect)) {
            balls[i].velocity.y *= -1.0f;
            float hitFactor = (balls[i].position.x - (paddle->rect.x + paddle->rect.width / 2.0f)) / (paddle->rect.width / 2.0f);
            balls[i].velocity.x = hitFactor * 6.0f;
        }

        // 블록 충돌 체크
        for (int r = 0; r < ROWS; r++) {
            for (int c = 0; c < COLS; c++) {
                if (blocks[r][c].isActive) {
                    if (CheckCollisionCircleRec(balls[i].position, balls[i].radius, blocks[r][c].rect)) {
                        blocks[r][c].isActive = false;
                        balls[i].velocity.y *= -1.0f;
                        *score += 100;

                        // 특수 이벤트 발동 (확률적으로 등장하는 블록 파괴 시)
                        TriggerBlockEvent(blocks[r][c].eventType, paddle, balls, ballCount, score);
                    }
                }
            }
        }
    }

    if (activeBalls <= 0) {
        *state = GAME_STATE_GAMEOVER;
    }

    // 블록 클리어 판정
    bool remainingBlocks = false;
    for (int r = 0; r < ROWS; r++) {
        for (int c = 0; c < COLS; c++) {
            if (blocks[r][c].isActive) {
                remainingBlocks = true;
                break;
            }
        }
        if (remainingBlocks) break;
    }

    if (!remainingBlocks) {
        *state = GAME_STATE_CLEAR;
    }
}

// 화면 렌더링 로직
void DrawGame(Ball balls[MAX_BALLS], Paddle paddle, Block blocks[ROWS][COLS], GameState state, int score) {
    BeginDrawing();
    ClearBackground((Color) { 20, 20, 30, 255 });

    DrawText(TextFormat("SCORE: %05d", score), 35, 15, 20, RAYWHITE);

    for (int r = 0; r < ROWS; r++) {
        for (int c = 0; c < COLS; c++) {
            if (blocks[r][c].isActive) {
                DrawRectangleRec(blocks[r][c].rect, blocks[r][c].color);
                // 이벤트 블록만 은은한 하얀색 테두리로 표시
                if (blocks[r][c].eventType != EVENT_NONE) {
                    DrawRectangleLinesEx(blocks[r][c].rect, 2, WHITE);
                }
                else {
                    DrawRectangleLinesEx(blocks[r][c].rect, 1, BLACK);
                }
            }
        }
    }

    DrawRectangleRec(paddle.rect, paddle.color);

    for (int i = 0; i < MAX_BALLS; i++) {
        if (balls[i].isActive) {
            DrawCircleV(balls[i].position, balls[i].radius, balls[i].color);
        }
    }

    if (state == GAME_STATE_READY) {
        DrawText("PRESS SPACE OR CLICK TO START", SCREEN_WIDTH / 2 - 180, SCREEN_HEIGHT / 2 + 50, 20, YELLOW);
    }
    else if (state == GAME_STATE_GAMEOVER) {
        DrawText("GAME OVER", SCREEN_WIDTH / 2 - 90, SCREEN_HEIGHT / 2 - 20, 30, RED);
        DrawText("PRESS [ENTER] TO RESTART", SCREEN_WIDTH / 2 - 120, SCREEN_HEIGHT / 2 + 30, 18, LIGHTGRAY);
    }
    else if (state == GAME_STATE_CLEAR) {
        DrawText("STAGE CLEAR!", SCREEN_WIDTH / 2 - 100, SCREEN_HEIGHT / 2 - 20, 30, GREEN);
        DrawText("PRESS [ENTER] TO RESTART", SCREEN_WIDTH / 2 - 120, SCREEN_HEIGHT / 2 + 30, 18, LIGHTGRAY);
    }

    EndDrawing();
}

int main(void) {
    srand((unsigned int)time(NULL));
    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT, "Rainbow Brick Breaker - Balanced Events");
    SetTargetFPS(60);

    Ball balls[MAX_BALLS];
    int ballCount = 1;
    Paddle paddle;
    Block blocks[ROWS][COLS];
    GameState state;
    int score = 0;

    InitializeGame(balls, &ballCount, &paddle, blocks, &state, &score);

    while (!WindowShouldClose()) {
        UpdateGame(balls, &ballCount, &paddle, blocks, &state, &score);
        DrawGame(balls, paddle, blocks, state, score);
    }

    CloseWindow();
    return 0;
}
```
<img width="1174" height="308" alt="image" src="https://github.com/user-attachments/assets/177cbfba-30b1-4574-9cc1-f2acf00404ab" />

