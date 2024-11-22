import pygame
import random
import sys

# Initialize Pygame
pygame.init()

# Constants
WINDOW_WIDTH, WINDOW_HEIGHT = 800, 600
BALL_RADIUS = 20
COLORS = {
    "basic": (255, 255, 255),  # 기본: 흰색
    "red": (255, 0, 0),        # 빨간색: 2점
    "pink": (255, 20, 147),    # 핑크색: 3점
    "blue": (0, 191, 255),     # 하늘색(빛남): 5점 (1회 등장)
    "bomb": (0, 0, 0),         # 폭탄: -1점
}
FPS = 60
GAME_TIME = 30  # 게임 제한 시간 (초)

# Initialize screen
screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("Click Game")
font = pygame.font.Font(None, 36)
clock = pygame.time.Clock()

# Game variables
difficulty = None  # 선택된 난이도
score = 0
balls = []

# Functions
def create_ball():
    """Create a new ball with random position, speed, and color."""
    x = random.randint(BALL_RADIUS, WINDOW_WIDTH - BALL_RADIUS)
    y = random.randint(BALL_RADIUS, WINDOW_HEIGHT - BALL_RADIUS)
    speed_x = random.choice([-3, 3]) if difficulty == "Easy" else random.choice([-5, 5])
    speed_y = random.choice([-3, 3]) if difficulty == "Easy" else random.choice([-5, 5])
    color = random.choices(
        ["basic", "red", "pink", "blue", "bomb"],
        weights=[60, 20, 10, 5, 5],  # 등장 확률
        k=1
    )[0]
    return {"x": x, "y": y, "speed_x": speed_x, "speed_y": speed_y, "color": color}


def reset_game():
    """Reset game variables."""
    global score, balls
    score = 0
    balls = []


def select_difficulty():
    """Display difficulty selection screen."""
    global difficulty

    while True:
        screen.fill((0, 0, 0))  # Clear screen
        title_text = font.render("Select Difficulty", True, (255, 255, 255))
        easy_text = font.render("1. Easy", True, (0, 255, 0))
        medium_text = font.render("2. Medium", True, (255, 255, 0))
        hard_text = font.render("3. Hard", True, (255, 0, 0))

        screen.blit(title_text, (WINDOW_WIDTH // 2 - title_text.get_width() // 2, 100))
        screen.blit(easy_text, (WINDOW_WIDTH // 2 - easy_text.get_width() // 2, 200))
        screen.blit(medium_text, (WINDOW_WIDTH // 2 - medium_text.get_width() // 2, 300))
        screen.blit(hard_text, (WINDOW_WIDTH // 2 - hard_text.get_width() // 2, 400))

        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_1:
                    difficulty = "Easy"
                    return
                elif event.key == pygame.K_2:
                    difficulty = "Medium"
                    return
                elif event.key == pygame.K_3:
                    difficulty = "Hard"
                    return


def end_game():
    """End the game and show the final score."""
    global game_active
    screen.fill((0, 0, 0))
    end_text = font.render(f"Game Over! Your score: {score}", True, (255, 255, 255))
    screen.blit(end_text, (WINDOW_WIDTH // 2 - end_text.get_width() // 2, WINDOW_HEIGHT // 2))
    pygame.display.flip()
    pygame.time.delay(3000)  # Wait 3 seconds before closing
    pygame.quit()
    sys.exit()


# Main game loop
def game_loop():
    global score, balls

    start_ticks = pygame.time.get_ticks()  # Start time

    while True:
        screen.fill((0, 0, 0))  # Clear screen

        # Timer
        elapsed_time = (pygame.time.get_ticks() - start_ticks) / 1000
        if elapsed_time >= GAME_TIME:
            end_game()

        # Display timer and score
        timer_text = font.render(f"Time Left: {GAME_TIME - int(elapsed_time)}s", True, (255, 255, 255))
        score_text = font.render(f"Score: {score}", True, (255, 255, 255))
        screen.blit(timer_text, (10, 10))
        screen.blit(score_text, (10, 50))

        # Event handling
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
                mouse_x, mouse_y = pygame.mouse.get_pos()
                for ball in balls:
                    dx = mouse_x - ball["x"]
                    dy = mouse_y - ball["y"]
                    distance = (dx**2 + dy**2)**0.5
                    if distance <= BALL_RADIUS:
                        if ball["color"] == "bomb":
                            score -= 1
                        elif ball["color"] == "basic":
                            score += 1
                        elif ball["color"] == "red":
                            score += 2
                        elif ball["color"] == "pink":
                            score += 3
                        elif ball["color"] == "blue":
                            score += 5
                        balls.remove(ball)
                        break

        # Update ball positions
        for ball in balls:
            ball["x"] += ball["speed_x"]
            ball["y"] += ball["speed_y"]

            # Bounce off walls
            if ball["x"] - BALL_RADIUS <= 0 or ball["x"] + BALL_RADIUS >= WINDOW_WIDTH:
                ball["speed_x"] *= -1
            if ball["y"] - BALL_RADIUS <= 0 or ball["y"] + BALL_RADIUS >= WINDOW_HEIGHT:
                ball["speed_y"] *= -1

            # Draw balls
            pygame.draw.circle(screen, COLORS[ball["color"]], (ball["x"], ball["y"]), BALL_RADIUS)

        # Add new balls
        if len(balls) < 5:
            balls.append(create_ball())

        pygame.display.flip()
        clock.tick(FPS)


# Start the game
select_difficulty()  # 난이도 선택
reset_game()         # 게임 변수 초기화
game_loop()          # 게임 시작
