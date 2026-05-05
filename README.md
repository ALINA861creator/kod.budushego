# kod.budushego
import tkinter as tk
from tkinter import ttk, messagebox
import json
import os

MOVIES_FILE = 'movies.json'

def load_movies():
    if os.path.exists(MOVIES_FILE):
        try:
            with open(MOVIES_FILE, 'r', encoding='utf-8') as f:
                return json.load(f)
        except (json.JSONDecodeError, IOError):
            messagebox.showwarning("Предупреждение", "Файл данных повреждён или пуст. Создаётся новый.")
            return []
    return []

def save_movies(data):
    try:
        with open(MOVIES_FILE, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False, indent=2)
    except IOError as e:
        messagebox.showerror("Ошибка", f"Не удалось сохранить данные: {e}")

def validate_input():
    title = entry_title.get().strip()
    genre = entry_genre.get().strip()
    year = entry_year.get().strip()
    rating = entry_rating.get().strip()

    if not title:
        messagebox.showerror("Ошибка", "Название фильма не может быть пустым.")
        return False
    if not genre:
        messagebox.showerror("Ошибка", "Жанр не может быть пустым.")
        return False

    if not year.isdigit():
        messagebox.showerror("Ошибка", "Год должен быть целым числом.")
        return False
    year_int = int(year)
    if year_int < 1800 or year_int > 2100:
        messagebox.showerror("Ошибка", "Год должен быть в диапазоне от 1800 до 2100.")
        return False

    try:
        rating_float = float(rating)
        if rating_float < 0 or rating_float > 10:
            messagebox.showerror("Ошибка", "Рейтинг должен быть числом от 0 до 10.")
            return False
    except ValueError:
        messagebox.showerror("Ошибка", "Рейтинг должен быть числом (например, 7.5).")
        return False

    return True

def add_movie():
    if validate_input():
        movie = {
            "title": entry_title.get(),
            "genre": entry_genre.get(),
            "year": int(entry_year.get()),
            "rating": float(entry_rating.get())
        }
        movies.append(movie)
        save_movies(movies)
        refresh_table()
        clear_fields()
        messagebox.showinfo("Успех", "Фильм успешно добавлен!")

def refresh_table(filter_genre=None, filter_year=None):
    for item in tree.get_children():
        tree.delete(item)
    for movie in movies:
        if filter_genre and filter_genre.lower() not in movie["genre"].lower():
            continue
        if filter_year and str(movie["year"]) != filter_year:
            continue
        tree.insert("", "end", values=(movie["title"], movie["genre"], movie["year"], f"{movie['rating']:.1f}"))

def apply_filters():
    genre_filter = entry_filter_genre.get().strip()
    year_filter = entry_filter_year.get().strip()
    refresh_table(genre_filter, year_filter)

def clear_fields():
    entry_title.delete(0, tk.END)
    entry_genre.delete(0, tk.END)
    entry_year.delete(0, tk.END)
    entry_rating.delete(0, tk.END)

# Загрузка данных
movies = load_movies()

# Создание GUI
root = tk.Tk()
root.title("Movie Library — Личная кинотека")
root.geometry("850x600")

tab_control = ttk.Notebook(root)
tab_main = ttk.Frame(tab_control)
tab_filter = ttk.Frame(tab_control)

tab_control.add(tab_main, text="Добавить фильм")
tab_control.add(tab_filter, text="Фильтр")
tab_control.pack(expand=1, fill="both")

# Вкладка "Добавить фильм"
tk.Label(tab_main, text="Название:").grid(row=0, column=0, padx=10, pady=10, sticky="w")
entry_title = tk.Entry(tab_main, width=40)
entry_title.grid(row=0, column=1, padx=10, pady=10)

tk.Label(tab_main, text="Жанр:").grid(row=1, column=0, padx=10, pady=10, sticky="w")
entry_genre = tk.Entry(tab_main, width=40)
entry_genre.grid(row=1, column=1, padx=10, pady=10)

tk.Label(tab_main, text="Год выпуска:").grid(row=2, column=0, padx=10, pady=10, sticky="w")
entry_year = tk.Entry(tab_main, width=40)
entry_year.grid(row=2, column=1, padx=10, pady=10)

tk.Label(tab_main, text="Рейтинг (0–10):").grid(row=3, column=0, padx=10, pady=10, sticky="w")
entry_rating = tk.Entry(tab_main, width=40)
entry_rating.grid(row=3, column=1, padx=10, pady=10)

btn_add = ttk.Button(tab_main, text="Добавить фильм", command=add_movie)
btn_add.grid(row=4, column=0, columnspan=2, pady=20)

tree = ttk.Treeview(tab_main, columns=("Название", "Жанр", "Год", "Рейтинг"), show="headings", height=12)
tree.heading("Название", text="Название")
tree.heading("Жанр", text="Жанр")
tree.heading("Год", text="Год")
tree.heading("Рейтинг", text="Рейтинг")
tree.column("Название", width=250)
tree.column("Жанр", width=150)
tree.column("Год", width=80)
tree.column("Рейтинг", width=80)
tree.grid(row=5, column=0, columnspan=2, padx=10, pady=10, sticky="nsew")

scrollbar = ttk.Scrollbar(tab_main, orient="vertical", command=tree.yview)
scrollbar.grid(row=5, column=2, sticky="ns")
tree.configure(yscrollcommand=scrollbar.set)

# Вкладка "Фильтр"
tk.Label(tab_filter, text="Фильтровать по жанру (можно часть слова):").grid(row=0, column=0, padx=10, pady=10, sticky="w")
entry_filter_genre = tk.Entry(tab_filter, width=40)
entry_filter_genre.grid(row=0, column=1, padx=10, pady=10)

tk.Label(tab_filter, text="Фильтровать по году:").grid(row=1, column=0, padx=10, pady=10, sticky="w")
entry_filter_year = tk.Entry(tab_filter, width=40)
entry_filter_year.grid(row=1, column=1, padx=10, pady=10)

btn_apply = ttk.Button(tab_filter, text="Применить фильтр", command=apply_filters)
btn_apply.grid(row=2, column=0, columnspan=2, pady=20)

btn_clear_filter = ttk.Button(tab_filter, text="Сбросить фильтр", command=lambda: refresh_table())
btn_clear_filter.grid(row=3, column=0, columnspan=2, pady=5)

refresh_table()
root.mainloop()
