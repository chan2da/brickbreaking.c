#include <SDL.h>
#include <iostream>
#include <vector>
#include <cstdlib>
#include <ctime>

const int SCREEN_WIDTH = 1650;
const int SCREEN_HEIGHT = 900;
const int PADDLE_WIDTH = 100;
const int PADDLE_HEIGHT = 20;
const int BALL_SIZE = 15;
const int BRICK_WIDTH = 80;
const int BRICK_HEIGHT = 20;

SDL_Window* gWindow = nullptr;
SDL_Renderer* gRenderer = nullptr;

SDL_Rect gPaddle = { SCREEN_WIDTH / 2 - PADDLE_WIDTH / 2, SCREEN_HEIGHT - PADDLE_HEIGHT - 20, PADDLE_WIDTH, PADDLE_HEIGHT };
SDL_Rect gBall = { SCREEN_WIDTH / 2 - BALL_SIZE / 2, SCREEN_HEIGHT / 1.5 - BALL_SIZE / 2, BALL_SIZE, BALL_SIZE };
std::vector<SDL_Rect> gBricks;

int ballVelX = 5;
int ballVelY = -5;

bool init() {
    if (SDL_Init(SDL_INIT_VIDEO) < 0) {
        std::cerr << "SDL 초기화 실패: " << SDL_GetError() << std::endl;
        return false;
    }

    gWindow = SDL_CreateWindow("벽돌 깨기 게임", SDL_WINDOWPOS_CENTERED, SDL_WINDOWPOS_CENTERED, SCREEN_WIDTH, SCREEN_HEIGHT, SDL_WINDOW_SHOWN);
    if (!gWindow) {
        std::cerr << "윈도우 생성 실패: " << SDL_GetError() << std::endl;
        return false;
    }

    gRenderer = SDL_CreateRenderer(gWindow, -1, SDL_RENDERER_ACCELERATED);
    if (!gRenderer) {
        std::cerr << "렌더러 생성 실패: " << SDL_GetError() << std::endl;
        return false;
    }

    // 랜덤 시드 설정
    std::srand(std::time(nullptr));

    return true;
}

void close() {
    SDL_DestroyRenderer(gRenderer);
    SDL_DestroyWindow(gWindow);
    SDL_Quit();
}

void handleInput() {
    SDL_Event e;
    while (SDL_PollEvent(&e)) {
        if (e.type == SDL_QUIT) {
            close();
            exit(0);
        }
    }

    const Uint8* currentKeyStates = SDL_GetKeyboardState(nullptr);
    int speed = 10;
    if (currentKeyStates[SDL_SCANCODE_LSHIFT]) {
        speed = 5;
    }
    if (currentKeyStates[SDL_SCANCODE_LEFT]) {
        if (gPaddle.x - speed >= 0) { // Check if paddle goes out of left boundary
            gPaddle.x -= speed;
        }
        else {
            gPaddle.x = 0; // Adjust paddle position to left boundary
        }
    }
    if (currentKeyStates[SDL_SCANCODE_RIGHT]) {
        if (gPaddle.x + speed <= SCREEN_WIDTH - gPaddle.w) { // Check if paddle goes out of right boundary
            gPaddle.x += speed;
        }
        else {
            gPaddle.x = SCREEN_WIDTH - gPaddle.w; // Adjust paddle position to right boundary
        }
    }
}


void update() {
    // Ball movement
    gBall.x += ballVelX;
    gBall.y += ballVelY;

    // Collision detection with screen boundaries
    if (gBall.x <= 0 || gBall.x >= SCREEN_WIDTH - BALL_SIZE) {
        ballVelX = -ballVelX;
    }
    if (gBall.y <= 0 || gBall.y >= SCREEN_HEIGHT - BALL_SIZE) {
        ballVelY = -ballVelY;
    }

    // Collision detection with paddle
    if (SDL_HasIntersection(&gBall, &gPaddle)) {
        ballVelY = -ballVelY;
    }

    // Collision detection with bricks
    for (size_t i = 0; i < gBricks.size(); ++i) {
        if (SDL_HasIntersection(&gBall, &gBricks[i])) {
            gBricks.erase(gBricks.begin() + i);
            ballVelY = -ballVelY;
            break;
        }
    }
}

void render() {
    SDL_SetRenderDrawColor(gRenderer, 0, 0, 0, 0);
    SDL_RenderClear(gRenderer);

    // Draw bricks with random colors
    for (const auto& brick : gBricks) {
        // Generate random color
        Uint8 r = std::rand() % 256;
        Uint8 g = std::rand() % 256;
        Uint8 b = std::rand() % 256;

        SDL_SetRenderDrawColor(gRenderer, r, g, b, 0xFF);
        SDL_RenderFillRect(gRenderer, &brick);
    }

    // Draw paddle
    SDL_SetRenderDrawColor(gRenderer, 0x00, 0xFF, 0x00, 0xFF);
    SDL_RenderFillRect(gRenderer, &gPaddle);

    // Draw ball
    SDL_SetRenderDrawColor(gRenderer, 0xFF, 0xFF, 0xFF, 0xFF);
    SDL_RenderFillRect(gRenderer, &gBall);

    SDL_RenderPresent(gRenderer);
}

int main(int argc, char* args[]) {
    if (!init()) {
        std::cerr << "초기화 실패" << std::endl;
        return -1;
    }

    // Create bricks
    for (int i = 0; i < 18; ++i) {
        for (int j = 0; j < 15; ++j) {
            gBricks.push_back({ i * (BRICK_WIDTH + 10), j * (BRICK_HEIGHT + 10), BRICK_WIDTH, BRICK_HEIGHT });
        }
    }

    while (true) {
        handleInput();
        update();   
        render();
        SDL_Delay(10);
    }

    close();
    return 0;
}
