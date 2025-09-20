"""
Interactive SQL Puzzle Game
Single-file program demonstrating SQL puzzles with hints and validation.
"""

import psycopg
import random

DB_URL = "postgresql://postgres:postgres@localhost:5432/sql_puzzle_db"

def init_db():
    with psycopg.connect(DB_URL, autocommit=True) as conn:
        with conn.cursor() as cur:
            # Create tables
            cur.execute("""
            CREATE TABLE IF NOT EXISTS employees (
                id SERIAL PRIMARY KEY,
                name TEXT,
                department TEXT,
                salary INT
            );
            """)
            # Seed data
            cur.execute("""
            INSERT INTO employees (name, department, salary)
            SELECT 'Employee'||i, 'Dept'||((i%3)+1), 3000 + (i*100)
            FROM generate_series(1,15) AS s(i)
            ON CONFLICT DO NOTHING;
            """)

PUZZLES = [
    {
        "description": "Select all employees with salary > 3500",
        "solution": "SELECT * FROM employees WHERE salary > 3500;",
        "hint": "Use a WHERE clause comparing salary"
    },
    {
        "description": "Count number of employees in each department",
        "solution": "SELECT department, COUNT(*) FROM employees GROUP BY department;",
        "hint": "Use GROUP BY"
    },
    {
        "description": "Find the average salary in 'Dept2'",
        "solution": "SELECT AVG(salary) FROM employees WHERE department='Dept2';",
        "hint": "Use AVG() and WHERE"
    }
]

def check_solution(user_query, expected_query):
    # Simplistic check: compare results
    with psycopg.connect(DB_URL) as conn:
        with conn.cursor() as cur:
            try:
                cur.execute(user_query)
                user_result = cur.fetchall()
                cur.execute(expected_query)
                expected_result = cur.fetchall()
                return user_result == expected_result
            except Exception as e:
                print(f"Error executing query: {e}")
                return False

def play_game():
    print("Welcome to SQL Puzzle Game!\n")
    random.shuffle(PUZZLES)
    for puzzle in PUZZLES:
        print(f"Puzzle: {puzzle['description']}")
        while True:
            user_query = input("Enter SQL query (or 'hint'): ")
            if user_query.lower() == "hint":
                print(f"Hint: {puzzle['hint']}")
                continue
            if check_solution(user_query, puzzle['solution']):
                print("Correct! ✅\n")
                break
            else:
                print("Incorrect, try again.\n")
    print("🎉 All puzzles solved!")

if __name__ == "__main__":
    init_db()
    play_game()
