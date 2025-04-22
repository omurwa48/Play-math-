import random
import time
import json
import os
import matplotlib.pyplot as plt

SCORE_FILE = "play_math_scores.json"
TIME_LIMIT = 15  # seconds per question

# Load past scores for the graph
def load_scores():
    if os.path.exists(SCORE_FILE):
        with open(SCORE_FILE, "r") as file:
            return json.load(file)
    return []

def save_score(name, score, percentage):
    scores = load_scores()
    scores.append({
        "name": name,
        "score": score,
        "percentage": percentage,
        "time": time.strftime("%Y-%m-%d %H:%M:%S")
    })
    with open(SCORE_FILE, "w") as file:
        json.dump(scores, file, indent=4)

def show_progress_graph():
    scores = load_scores()
    if not scores:
        print("No past scores to show.")
        return

    names = [entry["name"] for entry in scores]
    percentages = [entry["percentage"] for entry in scores]

    plt.plot(range(len(percentages)), percentages, marker='o')
    plt.title("Player Progress Over Time")
    plt.xlabel("Game Session")
    plt.ylabel("Score Percentage")
    plt.grid(True)
    plt.show()

def generate_question(level):
    if level == "easy":
        a, b = random.randint(1, 10), random.randint(1, 10)
        operation = random.choice(['+'])
    elif level == "medium":
        a, b = random.randint(5, 20), random.randint(5, 20)
        operation = random.choice(['+', '-'])
    elif level == "hard":
        a, b = random.randint(10, 50), random.randint(1, 10)
        operation = random.choice(['+', '-', '*'])
    else:  # combined
        a, b = random.randint(1, 50), random.randint(1, 10)
        operation = random.choice(['+', '-', '*', '/'])

    # Avoid division by zero or float divisions
    if operation == '/':
        a = a * b  # Make it divisible
        question = f"What is {a} / {b}? "
        correct_answer = a // b
    elif operation == '+':
        question = f"What is {a} + {b}? "
        correct_answer = a + b
    elif operation == '-':
        question = f"What is {a} - {b}? "
        correct_answer = a - b
    elif operation == '*':
        question = f"What is {a} * {b}? "
        correct_answer = a * b

    return question, correct_answer

def play_game():
    score = 0
    total_questions = 5
    print("Welcome to Play Math! Let’s go!\n")

    difficulty = input("Choose difficulty (easy, medium, hard, combined): ").lower()
    if difficulty not in ['easy', 'medium', 'hard', 'combined']:
        print("Invalid choice. Defaulting to 'medium'.")
        difficulty = 'medium'

    for i in range(total_questions):
        print(f"\nQuestion {i+1}:")
        question, correct_answer = generate_question(difficulty)

        print(f"You have {TIME_LIMIT} seconds to answer.")
        start_time = time.time()

        try:
            user_input = input(question)
            if time.time() - start_time > TIME_LIMIT:
                print("Too slow! Time's up.")
                continue
            user_answer = int(user_input)

            if user_answer == correct_answer:
                print("Correct!")
                score += 1
            else:
                print(f"Wrong. The correct answer was {correct_answer}")
        except ValueError:
            print("Invalid input! Please enter a number.")

    percentage = (score / total_questions) * 100
    print(f"\nGame Over. You scored {score}/{total_questions} ({percentage:.2f}%)")

    name = input("Enter your name to save your score: ")
    save_score(name, score, percentage)

    show_progress = input("Would you like to see your progress graph? (yes/no): ").lower()
    if show_progress == "yes":
        show_progress_graph()

# Start the game
play_game()
