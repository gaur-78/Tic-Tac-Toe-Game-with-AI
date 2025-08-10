# Tic-Tac-Toe-Game-with-AI
#It is basically a game which is known as Tic Tac Toe and is played with  your device etc Laptop, Mobile.


import tkinter as tk
from tkinter import messagebox, simpledialog
import copy

# ----- Game logic -----

WIN_COMBINATIONS = [
    (0,1,2), (3,4,5), (6,7,8),  # rows
    (0,3,6), (1,4,7), (2,5,8),  # cols
    (0,4,8), (2,4,6)            # diags
]

class TicTacToeGame:
    def __init__(self, mode=1):
        # board as list of 9 cells: 'X', 'O', or ''
        self.board = [''] * 9
        self.current = 'X'  # X always starts
        self.mode = mode    # 1: HvH, 2: HvC
        self.game_over = False

    def reset(self):
        self.board = [''] * 9
        self.current = 'X'
        self.game_over = False

    def available_moves(self):
        return [i for i, c in enumerate(self.board) if c == '']

    def make_move(self, idx):
        if self.board[idx] == '' and not self.game_over:
            self.board[idx] = self.current
            return True
        return False

    def switch_player(self):
        self.current = 'O' if self.current == 'X' else 'X'

    def winner(self):
        for (a,b,c) in WIN_COMBINATIONS:
            if self.board[a] != '' and self.board[a] == self.board[b] == self.board[c]:
                return self.board[a], (a,b,c)
        if all(cell != '' for cell in self.board):
            return 'Draw', None
        return None, None

    # ----- Minimax for unbeatable AI (plays as 'O') -----
    def minimax(self, board, player):
        # player is 'O' or 'X' (the one to play)
        winner, _ = self._check_winner_board(board)
        if winner == 'O':
            return 1
        elif winner == 'X':
            return -1
        elif winner == 'Draw':
            return 0

        scores = []
        for move in [i for i,c in enumerate(board) if c == '']:
            board_copy = board[:]
            board_copy[move] = player
            score = self.minimax(board_copy, 'X' if player == 'O' else 'O')
            scores.append(score)

        return max(scores) if player == 'O' else min(scores)

    def best_move(self):
        best_score = -999
        best_move = None
        for move in self.available_moves():
            board_copy = self.board[:]
            board_copy[move] = 'O'
            score = self.minimax(board_copy, 'X')
            # choose highest score for O
            if score > best_score:
                best_score = score
                best_move = move
        return best_move

    def _check_winner_board(self, board):
        for (a,b,c) in WIN_COMBINATIONS:
            if board[a] != '' and board[a] == board[b] == board[c]:
                return board[a], (a,b,c)
        if all(cell != '' for cell in board):
            return 'Draw', None
        return None, None

# ----- GUI -----

class TicTacToeUI(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title('Tic‑Tac‑Toe (3x3)')
        self.resizable(False, False)
        self.configure(padx=12, pady=12)

        # Choose mode first
        mode = self.ask_mode()
        self.game = TicTacToeGame(mode=mode)

        # Visual elements
        self.status_var = tk.StringVar()
        self.status_var.set("Turn: X")

        self.header = tk.Label(self, text='Tic‑Tac‑Toe', font=('Segoe UI', 18, 'bold'))
        self.header.grid(row=0, column=0, columnspan=3, pady=(0,8))

        self.status = tk.Label(self, textvariable=self.status_var, font=('Segoe UI', 12))
        self.status.grid(row=1, column=0, columnspan=3)

        # Board buttons
        self.buttons = []
        btn_font = ('Segoe UI', 36, 'bold')
        for i in range(9):
            r = 2 + i // 3
            c = i % 3
            b = tk.Button(self, text='', width=4, height=2, font=btn_font,
                          command=lambda i=i: self.on_cell_click(i))
            b.grid(row=r, column=c, padx=6, pady=6, ipadx=6, ipady=6)
            self.buttons.append(b)

        # Bottom controls
        self.reset_btn = tk.Button(self, text='Reset', command=self.reset_game)
        self.reset_btn.grid(row=5, column=0, pady=(10,0))

        self.newgame_btn = tk.Button(self, text='New Game (choose mode)', command=self.new_game_dialog)
        self.newgame_btn.grid(row=5, column=1, pady=(10,0))

        self.quit_btn = tk.Button(self, text='Quit', command=self.quit)
        self.quit_btn.grid(row=5, column=2, pady=(10,0))

        # If computer starts (rare because X always starts), we could handle it — currently X starts.
        self.update_ui()

    def ask_mode(self):
        # simple dialog to choose mode
        answer = None
        while answer not in ('1','2'):
            answer = simpledialog.askstring('Choose mode', 'Choose mode:\n1) Human vs Human\n2) Human vs Computer (unbeatable)\n\nEnter 1 or 2:', parent=self)
            if answer is None:
                # user cancelled; default to HvC
                return 2
        return int(answer)

    def on_cell_click(self, idx):
        if self.game.game_over:
            return

        if not self.game.make_move(idx):
            return  # invalid move

        self.after(50, self.post_move_actions)  # slight delay for UI responsiveness

    def post_move_actions(self):
        winner, combo = self.game.winner()
        if winner:
            self.game.game_over = True
            self.highlight_win(combo)
            if winner == 'Draw':
                self.status_var.set("It's a draw!")
                messagebox.showinfo('Draw', "The game is a draw.")
            else:
                self.status_var.set(f"{winner} wins!")
                messagebox.showinfo('Winner', f"{winner} wins!")
            return

        # swap player
        self.game.switch_player()
        self.update_ui()

        # If mode is HvC and it's O's turn, let AI move
        if self.game.mode == 2 and self.game.current == 'O' and not self.game.game_over:
            self.after(250, self.computer_turn)

    def computer_turn(self):
        move = self.game.best_move()
        if move is None:
            # fallback random move
            moves = self.game.available_moves()
            if not moves:
                return
            import random
            move = random.choice(moves)
        self.game.make_move(move)
        self.post_move_actions()

    def update_ui(self):
        # refresh button texts and status
        for i, b in enumerate(self.buttons):
            b['text'] = self.game.board[i]
            b['state'] = tk.NORMAL if self.game.board[i] == '' and not self.game.game_over else tk.DISABLED
            b['bg'] = 'SystemButtonFace'

        if not self.game.game_over:
            self.status_var.set(f"Turn: {self.game.current}")
        else:
            self.status_var.set('Game over')

    def highlight_win(self, combo):
        if combo is None:
            return
        for i in combo:
            self.buttons[i]['bg'] = '#90ee90'  # light green highlight
        self.update_ui()

    def reset_game(self):
        self.game.reset()
        self.update_ui()

    def new_game_dialog(self):
        mode = self.ask_mode()
        self.game = TicTacToeGame(mode=mode)
        self.update_ui()

if __name__ == '__main__':
    app = TicTacToeUI()
    app.mainloop()
