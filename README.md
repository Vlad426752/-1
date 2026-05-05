import tkinter as tk
from tkinter import ttk, messagebox
import json
import os

class MovieLibraryApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Movie Library")
        self.file_path = "movies.json"
        self.movies = self.load_data()

        # Поля ввода
        frame = tk.Frame(root, padx=10, pady=10)
        frame.pack(side=tk.TOP, fill=tk.X)

        tk.Label(frame, text="Название:").grid(row=0, column=0)
        self.title_entry = tk.Entry(frame)
        self.title_entry.grid(row=0, column=1)

        tk.Label(frame, text="Жанр:").grid(row=0, column=2)
        self.genre_entry = tk.Entry(frame)
        self.genre_entry.grid(row=0, column=3)

        tk.Label(frame, text="Год:").grid(row=1, column=0)
        self.year_entry = tk.Entry(frame)
        self.year_entry.grid(row=1, column=1)

        tk.Label(frame, text="Рейтинг (0-10):").grid(row=1, column=2)
        self.rating_entry = tk.Entry(frame)
        self.rating_entry.grid(row=1, column=3)

        tk.Button(frame, text="Добавить фильм", command=self.add_movie).grid(row=2, columnspan=4, pady=10)

        # Фильтрация
        filter_frame = tk.Frame(root, padx=10)
        filter_frame.pack(side=tk.TOP, fill=tk.X)
        
        tk.Label(filter_frame, text="Фильтр Жанр:").pack(side=tk.LEFT)
        self.filter_genre = tk.Entry(filter_frame, width=10)
        self.filter_genre.pack(side=tk.LEFT, padx=5)
        
        tk.Button(filter_frame, text="Применить", command=self.update_table).pack(side=tk.LEFT)
        tk.Button(filter_frame, text="Сброс", command=self.reset_filter).pack(side=tk.LEFT, padx=5)

        # Таблица
        self.tree = ttk.Treeview(root, columns=("Title", "Genre", "Year", "Rating"), show='headings')
        self.tree.heading("Title", text="Название")
        self.tree.heading("Genre", text="Жанр")
        self.tree.heading("Year", text="Год")
        self.tree.heading("Rating", text="Рейтинг")
        self.tree.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

        self.update_table()

    def add_movie(self):
        title = self.title_entry.get()
        genre = self.genre_entry.get()
        year = self.year_entry.get()
        rating = self.rating_entry.get()

        # Валидация
        if not (title and genre and year and rating):
            messagebox.showerror("Ошибка", "Заполните все поля!")
            return
        
        try:
            year = int(year)
            rating = float(rating)
            if not (0 <= rating <= 10):
                raise ValueError
        except ValueError:
            messagebox.showerror("Ошибка", "Год — число, Рейтинг — от 0 до 10!")
            return

        self.movies.append({"title": title, "genre": genre, "year": year, "rating": rating})
        self.save_data()
        self.update_table()
        
    def update_table(self):
        self.tree.delete(*self.tree.get_children())
        genre_filter = self.filter_genre.get().lower()
        
        for m in self.movies:
            if genre_filter in m['genre'].lower():
                self.tree.insert("", tk.END, values=(m['title'], m['genre'], m['year'], m['rating']))

    def reset_filter(self):
        self.filter_genre.delete(0, tk.END)
        self.update_table()

    def save_data(self):
        with open(self.file_path, "w", encoding="utf-8") as f:
            json.dump(self.movies, f, ensure_ascii=False, indent=4)

    def load_data(self):
        if os.path.exists(self.file_path):
            with open(self.file_path, "r", encoding="utf-8") as f:
                return json.load(f)
        return []

if __name__ == "__main__":
    root = tk.Tk()
    app = MovieLibraryApp(root)
    root.mainloop()
