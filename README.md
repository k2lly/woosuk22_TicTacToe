### Python TicTacToe 프로그램
------------------------------------------

### 프로젝트 정보
* 해당 프로젝트 진행 인원 : 2명
* 프로젝트 의도 : 학기 말 프로그램 전시회 제출

### 팀 소개
* 김상호 : 우석대학교 컴퓨터공학과
* 손지승 : 우석대학교 컴퓨터공학과

### 프로젝트 소개
틱택토(tic-tac-toe)는 두 명이 번갈아가며 O와 X를 3×3 판에 써서 같은 글자를 가로, 세로, 혹은 대각선 상에 놓이도록 하는 놀이입니다.
m,n,k-게임으로, (3,3,3)-게임입니다.
쉽게 말해 3X3 화면에서 자기의 도형(O or X)을 번갈아 놓게 되며, 같은 모양의 도형이 3개가 연속해서 놓이게 되면 이기는 게임입니다.
이런 게임을 pygame 라이브러리를 이용하여 제작하였습니다.
틱택토 게임에서 컴퓨터의 최적의 수를 찾기 위해 미니맥스 알고리즘을 사용하여 구현하였습니다다.

### 코드 설명
*tttAI
변수 선언:
computer = 'X'
user = 'O'
empty = '-'

infinity = 1000

클래스 AI
초기화 메서드 (__init__):
class AI(object):
    def __init__(self, board):
        self.board = board[:]
        self.at_score = []
        self.is_finish()
        self.get_coords()
        self.cnt = 0

    def is_full(self, board):
        for y in range(3):
            for x in range(3):
                if board[y][x] == empty:
                    return False
        return True

보드가 가득 찼는지 확인합니다. 모든 칸이 채워졌다면 True를 반환합니다.
    def is_win(self, player, board):
        # 가로와 세로  체크
        for y in range(3):
            row, column = 0, 0
            for x in range(3):
                if board[y][x] == player:
                    row += 1
                if board[x][y] == player:
                    column += 1
            if row == 3 or column == 3:
                return True
            
        # 대각선 체크
        x, row, column = 2, 0, 0
        for y in range(3):
            if board[y][y] == player:
                row += 1
            if board[y][x] == player:
                column += 1
            x -= 1
        if row == 3 or column == 3:
            return True

        return False


주어진 플레이어(computer 또는 user)가 승리했는지 확인합니다. 가로, 세로, 대각선에서 같은 기호가 3개 연속으로 있는지 체크합니다.
    def is_finish(self):
        return (self.is_full(self.board) or
                self.is_win(computer, self.board) or
                self.is_win(user, self.board))

게임이 종료되었는지(보드가 가득 찼거나 누군가 승리했는지) 확인합니다.
    def get_coords(self):
        coords = []
        for y in range(3):
            for x in range(3):
                if self.board[y][x] == empty:
                    coords.append((x, y))
        return coords

현재 보드에서 빈 칸의 좌표를 리스트로 반환합니다.
    def evaluate(self):
        if self.is_win(computer, self.board):
            return 1
        elif self.is_win(user, self.board):
            return -1
        else:
            return 0

컴퓨터가 승리하면 1, 사용자가 승리하면 -1, 비기면 0을 반환합니다.
    def get_best_coord(self):
        score = -100
        best_coord = None
        for best in self.at_score:
            if best[0] > score:
                score = best[0]
                best_coord = best[1]
        self.at_score.clear()
        return best_coord

미니맥스 알고리즘 실행 후, 최적의 수를 반환합니다.
    def fill_board(self, coord, player):
        x, y = coord
        self.board[y][x] = player

주어진 좌표에 해당 플레이어의 기호를 채웁니다.
  def minimax(self, depth, alpha, beta, player):
        self.cnt += 1
        if alpha >= beta:
            if player == computer:
                return infinity
            else:
                return -infinity
        if self.is_finish():
            return self.evaluate()

        coords = self.get_coords()
        max_score = -infinity
        min_score = infinity
        for coord in coords:
            if player == computer:
                self.fill_board(coord, computer)
                score = self.minimax(depth + 1, alpha, beta, user)
                max_score = max(score, max_score)
                alpha = max(max_score, alpha)
                
                if depth == 0:
                    self.at_score.append((max_score, coord))
            else:
                self.fill_board(coord, user)
                score = self.minimax(depth + 1, alpha, beta, computer)
                min_score = min(score, min_score)
                beta = min(min_score, beta)
                
            self.board[coord[1]][coord[0]] = empty
            if score == infinity or score == -infinity:
                break
            
        if player == computer:
            return max_score
        else:
            return min_score

미니맥스 알고리즘을 이용해 최적의 수를 찾습니다.
재귀적으로 각 가능한 수를 시뮬레이션하고, 컴퓨터와 사용자의 최선의 결과를 계산하여 반환합니다.
alpha-beta pruning을 통해 비효율적인 경로를 조기에 차단하여 계산을 최적화합니다.

---------------------------------------------------------------------------------

*TicTacToeAI
*기본 설정
import pygame, sys, random
from pygame.locals import *
from tttAI import *

width = 300
height = 400
box_size = 80
text_size = 50
white = (255, 255, 255)
black = (0, 0, 0)
blue = (0, 0, 255)

fps = 30
fps_clock = pygame.time.Clock()
화면의 크기, 색상, 글자 크기 등을 설정합니다.
tttAI 모듈에서 AI 클래스를 가져옵니다.

*메인 루프
def main():
    pygame.init()
    surface = pygame.display.set_mode((width, height))
    pygame.display.set_caption('Tic Tac Toe')
    surface.fill(white)
    menu = Menu(surface)
    ttt = TTT(surface, menu)
    while True:
        run_game(surface, menu, ttt)
        menu.is_continue()
파이게임을 초기화하고, 화면을 설정합니다.
게임의 메인 루프를 실행하여 틱택토 게임을 관리합니다.

*게임 실행 및 종료
def run_game(surface, menu, ttt):
    reset_game(surface, menu, ttt)
    while True:
        is_user = True
        if ttt.turn == computer:
            is_user = False
            ttt.play_computer()

        for event in pygame.event.get():
            if event.type == QUIT:
                terminate()
            elif event.type == MOUSEBUTTONUP and is_user:
                if not ttt.check_board(event.pos):
                    if menu.check_rect(event.pos):
                        reset_game(surface, menu, ttt)

        if ttt.check_gameover():
            return
        pygame.display.update()
        fps_clock.tick(fps)
        
def terminate():
    pygame.quit()
    sys.exit()
run_game: 게임을 실행하고, 사용자의 입력을 받아 처리합니다.
terminate: 게임을 종료하는 함수입니다.

*틱택토 클래스
class TTT(object):
    def __init__(self, surface, menu):
        self.board = [['-' for i in range(3)] for j in range(3)]
        self.surface = surface
        self.menu = menu

    def init_game(self):
        self.ai = AI(self.board)
        self.draw_line(self.surface)
        self.finish = False
        self.init_board()
        self.set_first_player()

    def play_computer(self):
        self.ai.minimax(0, -infinity, infinity, computer)
        x, y = self.ai.get_best_coord()
        self.draw_shape(x, y)
틱택토 게임 보드의 상태와 AI를 관리하는 클래스입니다.
init_game: 게임을 초기화합니다.
play_computer: AI가 최적의 수를 선택하여 플레이합니다.

* 메뉴 클래스
class Menu(object):
    def __init__(self, surface):
        self.font = pygame.font.Font('freesansbold.ttf', 20)
        self.surface = surface
        self.draw_menu()

    def show_msg(self, msg_id):
        msg = {'X': 'LOSE!', 'O': 'WIN!', 'tie': 'DRAW!'}
        self.make_text(msg[msg_id], blue, white, center_x, 30)

    def make_text(self, text, color, bgcolor, cx, cy):
        surf = self.font.render(text, True, color, bgcolor)
        rect = surf.get_rect()
        rect.center = (cx, cy)
        self.surface.blit(surf, rect)
        return rect
게임의 메뉴와 메시지 표시를 담당하는 클래스입니다.
show_msg: 게임의 결과 메시지를 화면에 표시합니다.

*게임 상태 초기화
def reset_game(surface, menu, ttt):
    surface.fill(white)
    menu.draw_menu()
    ttt.init_game()
게임을 초기 상태로 재설정하는 함수입니다.
