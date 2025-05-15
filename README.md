
````markdown
# 🎮 Raylib 장애물 피하기 게임

## 📌 프로젝트 개요
- **팀명**: 9팀
- **팀원**: 전재민(팀장), 김준용, 김무진
- **개발 기간**: 2025년 05월 14일 ~ 2025년 05월 18일 (총 5일)
- **주요 기술**: C++, raylib 라이브러리, 게임 루프, 충돌 감지, 객체 관리 등

## 👥 팀원 역할 분담
| 이름 | 역할 | 주요 담당 업무 |
|------|------|----------------|
| 전재민 | 프로젝트 관리 / 기획 및 설계 | 전체 게임 구조 설계, 핵심 기능 정의, 개발 방향 제시, Git 관리 |
| 김무진 | 개발 | 게임 핵심 기능 구현 (주인공, 장애물, 충돌, 점수), 속도/크기 변화 로직 코딩, raylib 기능 활용 |
| 김준용 | 문서 / 테스트 | 프로젝트 보고서 작성, README 문서 관리, 테스트 계획 수립 및 실행, 버그 리포팅 |

## 🏗️ 시스템 구조

본 게임은 Raylib의 메인 게임 루프를 기반으로 객체 지향 프로그래밍 개념을 활용하여 구현되었습니다. 주요 구성 요소는 다음과 같습니다.

* **Player**: 플레이어 객체의 위치, 크기, 이동속도, 현재 생명 수, 최대 생명 수 의 속성을 가집니다. 이동 및 상태(예: 무적 시간) 관리를 담당합니다.
* **Obstacle**: 장애물 객체의 위치, 크기, 낙하 속도 의 속성을 가집니다. 화면 상단에서 생성되어 하단으로 이동하는 로직을 처리합니다.
* **LifeItem**: 생명 회복 아이템 객체의 위치, 크기, 낙하 속도 의 속성 을 가집니다. 장애물과 유사하게 동작하지만, 충돌 시 생명을 회복시키는 역할을 합니다.

```mermaid

    struct Player {
        +Vector2 position
        +Vector2 size
        +float speed
        +int lives
        +int maxLives
    }

    struct Obstacle {
        +Vector2 position
        +Vector2 size
        +float speed
        +Update(float)
        +IsOffScreen(int) bool
    }

     struct LifeItem {
        +Vector2 position
        +Vector2 size
        +Color color
        +float speed
        +Update(float)
        +IsOffScreen(int) bool
    }
````

## 🛠️ 핵심 기능

### 1\. 게임 루프 및 상태 관리

Raylib의 `InitWindow`, `SetTargetFPS`, `WindowShouldClose`, `BeginDrawing`, `EndDrawing` 함수를 사용하여 게임의 기본 루프를 구성하고, `CheckCollisions` 함수를 통해 `player.lives` 상태를 전환하며 각 상태에 맞는 로직을 처리합니다.

```cpp
// 예시: main 함수 및 주요 함수 호출 구조
int main() {
    InitWindow(screenWidth, screenHeight, "Raylib Game");
    SetTargetFPS(60);

    // 지면 정의
    Rectangle ground = { 0, screenHeight - 20, (float)screenWidth, 20 };

    InitGame();

    while (!WindowShouldClose()) {
        float deltaTime = GetFrameTime();

        // ---------------------
        // 입력 처리
        // ---------------------

        // ---------------------
        // 게임 로직 업데이트
        // ---------------------

        // ---------------------
        // 그리기 시작
        // ---------------------

         EndDrawing();
    }

    CloseWindow();
    return 0;
}
```
### 2\. 플레이어 조작 및 경계 처리

`Player`는 `position`과 `speed`를 가지고 있으며, `IsKeyDown` 함수에서 좌우 방향키 입력을 감지하여 `position.x`를 업데이트합니다. 화면 좌우 경계를 벗어나지 않도록 위치를 제한하는 로직을 포함합니다.

```cpp
 if (IsKeyPressed(KEY_ESCAPE)) break;

        if (gameState == GAME_START) {
            if (IsKeyPressed(KEY_SPACE)) {
                gameState = GAME_PLAYING;
            }
        }
        else if (gameState == GAME_PLAYING) {
            if (IsKeyDown(KEY_LEFT)) player.position.x -= player.speed * deltaTime;
            if (IsKeyDown(KEY_RIGHT)) player.position.x += player.speed * deltaTime;

            // 플레이어가 화면 밖으로 나가지 않도록 제한
            if (player.position.x < 0) player.position.x = 0;
            if (player.position.x > screenWidth - player.size.x) player.position.x = screenWidth - player.size.x;
        }
```

### 3\. 장애물 및 아이템 생성/관리

전역 변수`obstacleSpawnRate`와 `float lifeItemSpawnRate`를 사용하여 일정 간격마다 화면 상단에 `Obstacle` 또는 `LifeItem` 객체를 생성합니다.. 각 객체의 `Update` 함수를 호출하여 하단으로 이동시키고, `IsOffScreen` 함수를 통해 화면 밖으로 나간 객체는 목록에서 제거합니다.
```cpp
// -------------------------
// 장애물 구조체 정의
// -------------------------
struct Obstacle {
    Vector2 position;
    Vector2 size;
    float speed;

    void Update(float deltaTime) {
        position.y += speed * deltaTime;  // 아래로 이동
    }

    bool IsOffScreen(int screenHeight) const {
        return position.y > screenHeight; // 화면 아래로 벗어났는지 검사
    }
};

// -------------------------
// 생명 아이템 구조체 정의
// -------------------------
struct LifeItem {
    Vector2 position;
    Vector2 size;
    float speed;

    void Update(float deltaTime) {
        position.y += speed * deltaTime;  // 아래로 이동
    }

    bool IsOffScreen(int screenHeight) const {
        return position.y > screenHeight; // 화면 아래로 벗어났는지 검사
    }
};
```

```cpp
// 장애물 생성 간격 계산 (스테이지가 올라갈수록 빨라짐)
            float currentObstacleRate = obstacleSpawnRate - (currentStage - 1) * 0.1f;
            if (currentObstacleRate < 0.5f) currentObstacleRate = 0.5f;

            // 장애물 생성
            obstacleSpawnTimer += deltaTime;
            if (obstacleSpawnTimer >= currentObstacleRate) {
                Obstacle obs;
                //다양한 크기의 장애물 생성
                float minSize = 30.0f;
                float maxSize = 40.0f + currentStage * 5.0f;
                if (maxSize > 100.0f) maxSize = 100.0f;

                float sizeX = GetRandomValue((int)minSize, (int)maxSize);
                float sizeY = GetRandomValue((int)minSize, (int)maxSize);
                obs.size = { sizeX, sizeY };

                obs.position = { (float)GetRandomValue(0, screenWidth - (int)obs.size.x), -obs.size.y };

                // 스테이지에 따라 다양한 속도로 생성 (느린/빠른 혼합)
                float minSpeed = 80.0f + currentStage * 10.0f;
                float maxSpeed = 150.0f + currentStage * 20.0f;
                if (maxSpeed > 500.0f) maxSpeed = 500.0f;

                obs.speed = GetRandomValue((int)minSpeed, (int)maxSpeed);

                obstacles.push_back(obs);
                obstacleSpawnTimer = 0.0f;
            }

            // 생명 아이템 생성
            lifeItemSpawnTimer += deltaTime;
            if (lifeItemSpawnTimer >= lifeItemSpawnRate) {
                LifeItem item;
                item.size = { 30, 30 };
                item.position = { (float)GetRandomValue(0, screenWidth - (int)item.size.x), -item.size.y };
                item.speed = 120;
                lifeItems.push_back(item);
                lifeItemSpawnTimer = 0.0f;
            }

            // 모든 오브젝트 이동 업데이트
            for (auto& obs : obstacles) obs.Update(deltaTime);
            for (auto& item : lifeItems) item.Update(deltaTime);

            // 충돌 처리
            CheckCollisions();
        }
```
### 4\. 충돌 감지 및 생명 시스템

`CheckCollisions` 함수에서 Raylib의 `CheckCollisionRecs`를 사용하여 플레이어와 장애물 또는 아이템의 충돌을 감지합니다. 장애물과 충돌 시 플레이어의 생명을 감소시키고 (필요시 무적 시간 고려), 아이템과 충돌 시 생명을 증가시킵니다 (최대 생명 제한). 충돌한 객체는 목록에서 제거합니다. 플레이어 생명이 0이 되면 게임 오버 상태로 전환합니다.

```cpp
// -------------------------
// 충돌 검사 함수
// -------------------------
void CheckCollisions() {
    Rectangle playerRect = { player.position.x, player.position.y, player.size.x, player.size.y };

    // 장애물 충돌 검사 및 삭제
    obstacles.erase(std::remove_if(obstacles.begin(), obstacles.end(), [&](const Obstacle& obs) {
        Rectangle rect = { obs.position.x, obs.position.y, obs.size.x, obs.size.y };
        if (CheckCollisionRecs(playerRect, rect)) {
            player.lives--;
            if (player.lives < 0) { //생명이 0일 때 장애물을 한 번 더 맞으면 게임오버
                gameState = GAME_OVER;
                finalScore = score;
            }
            return true; // 충돌 시 제거
        }
        return obs.IsOffScreen(screenHeight); // 화면 밖이면 제거
        }), obstacles.end());

    // 생명 아이템 충돌 검사 및 삭제
    lifeItems.erase(std::remove_if(lifeItems.begin(), lifeItems.end(), [&](const LifeItem& item) {
        Rectangle rect = { item.position.x, item.position.y, item.size.x, item.size.y };
        if (CheckCollisionRecs(playerRect, rect)) {
            player.lives++;
            if (player.lives > player.maxLives) player.lives = player.maxLives;
            return true; // 획득 시 제거
        }
        return item.IsOffScreen(screenHeight); // 화면 밖이면 제거
        }), lifeItems.end());
}
```

### 5\. 점수, 시간, 스테이지 시스템 및 UI 표시

`CheckCollisions`에서 `player.lives`를 업데이트하고, `main` 의 게임 로직 업데이트 부분에서 `elapsedTime` 을 업데이트 합니다.`CheckStageIncrease` 함수를 통해 점수에 따라 `currentStage`를 증가시킵니다. `main` 에서 `DrawText`를 사용하여 현재 점수, 남은 생명, 경과 시간, 현재 스테이지 정보를 화면에 표시합니다.

```cpp
// -------------------------
// 전역 변수
// -------------------------
const int screenWidth = 800;
const int screenHeight = 450;

GameState gameState;
Player player;
std::vector<Obstacle> obstacles;
std::vector<LifeItem> lifeItems;

float obstacleSpawnTimer = 0.0f;
float obstacleSpawnRate = 1.2f;  // 장애물 생성 간격 (초)

float lifeItemSpawnTimer = 0.0f;
float lifeItemSpawnRate = 10.0f; // 생명 아이템 생성 간격 (초)

float elapsedTime = 0.0f;
float lastScoreTime = 0.0f;

int score = 0;
int finalScore = 0;

int currentStage = 1;
const int scorePerStage = 5; // n점마다 스테이지 증가

// UI 정보 표시
            DrawText(TextFormat("Score: %i", score), 10, 10, 20, WHITE);
            DrawText(TextFormat("Lives: %i", player.lives), 10, 30, 20, WHITE);
            DrawText(TextFormat("Time: %.0f", elapsedTime), 10, 50, 20, WHITE);
            DrawText(TextFormat("Stage: %i", currentStage), screenWidth - 120, 10, 20, RAYWHITE);
            DrawText("Press ESC to Exit", 10, 430, 20, RAYWHITE);
        }
```

### 6\. 게임 오버 및 재시작/종료

`CheckCollisions` 함수 에서 생명이 0이 되면 `gameState`를 `GAME_OVER`로 설정합니다. `main`에서 게임 오버 상태일 때 최종 점수와 재시작/종료 안내 메시지를 중앙에 표시합니다. 게임 오버 상태 중 Space 키 입력이 있으면 게임을 초기 상태로 되돌립니다. Esc 키 입력은 게임 루프를 종료시킵니다.

```cpp

else if (gameState == GAME_OVER) {
            // 게임 오버 화면
            const char* gameOverText = "Game Over!";
            DrawText(gameOverText, screenWidth / 2 - MeasureText(gameOverText, 40) / 2, screenHeight / 2 - 60, 40, RED);
            DrawText(TextFormat("Final Score: %i", finalScore), screenWidth / 2 - 100, screenHeight / 2, 30, RAYWHITE);
            DrawText(TextFormat("Total Time: %.0fs", elapsedTime), screenWidth / 2 - 100, screenHeight / 2 + 40, 30, RAYWHITE);
            DrawText("Press SPACE to Restart", screenWidth / 2 - 100, screenHeight / 2 + 100, 20, DARKGRAY);
            DrawText("Press ESC to Exit", screenWidth / 2 - 100, screenHeight / 2 + 120, 20, DARKGRAY);
        }

        EndDrawing();
    }
```

## 📅 개발 일정

| 일차 | 날짜 | 주요 작업 | 참여 |
|------|------|----------|------|
| 1일차 | 05.14 | 프로젝트 주제 선정, 요구사항 분석, 일정 산정, 역할 분배, 계획 및 설계 | 전재민, 김무진, 김준용 |
| 2일차 | 05.15 | raylib 환경 구성, 기본창, 플레이어, 장애물, 생명 아이템, 점수 및 스테이지 구현 | 전재민, 김무진 |
| 3일차 | 05.16 | 시작화면, 게임시작 및 종료 키 이벤트, 지면 추가, 장애물 속도 및 크기 난이도 조정 | 김준용, 김무진 |
| 4일차 | 05.17 | 플레이어 생명 표시 및 점수, 시간 HUD 개선, 전체 코드 구조 주석화 및 유지보수 편의성 개선 | 김무진 |
| 5일차 | 05.18 | 

## 💡 성과 및 개선점

### ✅ 성공 요인

  - Raylib 라이브러리 학습 및 활용 능력 배양
      * 게임 개발에 필요한 기본적인 그래픽, 입력, 게임 루프 관리를 경험
  - 객체 지향 설계를 통한 코드 모듈화
      * Player, Obstacle, LifeItem 등 역할을 분담하여 코드의 가독성 및 관리 용이성 향상
  - 기본적인 게임 메커니즘 구현 경험
      * 이동, 충돌, 점수, 생명, 난이도 조절 등 핵심 게임 기능을 직접 구현
  - Git을 활용한 팀 협업
      * 변경 이력 관리 및 팀원 간 효율적인 코드 공유/병합 진행

### 🔧 어려웠던 점

  - 게임 루프 내 시간 관리 및 객체 상태 동기화
      * `GetFrameTime()` 활용 및 deltaTime 기반 업데이트 로직 설계의 필요성 인지
  - 충돌 감지 및 객체 제거 로직 구현
      * 벡터를 순회하며 객체를 안전하게 제거하는 방법 (`std::remove_if` + `erase`) 학습
  - 게임 난이도 조절 파라미터 튜닝
      * 스테이지별 장애물 속도 및 생성 빈도 조절이 게임 플레이에 미치는 영향 테스트 및 조정
  - 게임 상태에 따른 UI 및 로직 분기 처리

### 📌 보완할 점 및 개선 방향

1.  **무적 시간 구현**: 플레이어가 장애물과 충돌 후 짧은 시간 동안 무적 상태가 되어 연속 피해를 입지 않도록 기능 추가
2.  **다양한 장애물/아이템 타입**: 속도, 크기, 효과가 다른 다양한 종류의 장애물 또는 아이템 추가를 통한 게임성 강화
3.  **배경 및 효과음**: 게임 배경 이미지 또는 간단한 효과음을 추가하여 사용자 경험 개선
4.  **코드 최적화**: 객체 관리 및 충돌 체크 로직에서 잠재적인 성능 병목 지점 개선 검토

## 📄실행화면

📸 기능별 실행 화면
게임 초기 화면 (게임 시작화면 없이 바로 게임구현)
![8d27d426d127a634](https://github.com/user-attachments/assets/7912de87-c4fb-4c60-b226-eb04a956eda1)

주인공 이동 및 점수, 생명 시스템
![23b28e50ebd8bfd5](https://github.com/user-attachments/assets/38768c64-c734-4ae9-80ac-6f8b46102a14)

장애물 및 생명 블록 생성 및 이동(장애물,생명 블록 화면), 속도/크기 변화(점수 변화에 따른 장애물 크기 변화)
![3a9bf9d9f6c2bb36](https://github.com/user-attachments/assets/de7d29dc-7f56-46ef-9347-7ad89fc4c8f5)

속도/크기 변화(점수 변화에 따른 장애물 크기 변화)

게임 오버 UI(장애물에 닿았을시 게임종료)
![9c3a9a9deb621d77](https://github.com/user-attachments/assets/18e3a99e-24e8-4d54-a2da-96036af42fcb)

## 프로젝트 소감

### 🔹전재민(팀장)

> **Raylib을 처음 사용해보면서 게임 개발의 기본적인 파이프라인을 경험할 수 있었습니다. 객체 관리와 게임 상태 관리를 GameManager 클래스로 집중시킨 설계 방식이 팀원들과의 협업 및 코드 통합에 도움이 되었습니다.**

### 🔹김무진

> **C++과 Raylib을 활용하여 실제 게임의 움직임과 상호작용을 구현하는 과정이 흥미로웠습니다. 특히 장애물의 속도와 생성 빈도를 조절하며 난이도를 변화시키는 로직을 구현하면서 게임 밸런스에 대한 이해도를 높일 수 있었습니다.**

### 🔹김준용

> **게임 기능을 테스트하고 발생한 버그를 추적하는 역할을 맡으면서 코드의 작은 변경이 전체 게임 동작에 미치는 영향을 파악하는 중요성을 알게 되었습니다. README 문서를 작성하며 프로젝트의 기능과 구조를 명확하게 정리하는 연습을 할 수 있었습니다.**

-----

```
```
