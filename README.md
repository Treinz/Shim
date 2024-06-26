# Shim
Напиши приложение на python используя tkinter в качестве графического интерфейса в одном файле по ТЗ:
Возможность добавления заявок в базу данных с указанием следующих параметров:
Номер заявки;
Дата добавления;
Вид авто;
Модель авто;
Описание проблемы;
ФИО клиента;
Номер телефона;
Статус заявки (новая заявка, в процессе ремонта, завершена).
Возможность редактирования заявок:
Изменение этапа выполнения (готова к выдаче, в процессе ремонта, ожидание автозапчастей);
Изменение описания проблемы;
Изменение ответственного за выполнение работ.
Возможность отслеживания статуса заявки:
Отображение списка заявок;
Получение уведомлений о смене статуса заявки;
Поиск заявки по номеру или по параметрам.
Возможность назначения ответственных за выполнение работ:
Добавление автомеханика к заявке;
Отслеживание состояния работы и получение уведомлений о ее завершении;
Автомеханик может добавлять комментарии на форме заявки и фиксировать информацию о заказанных автозапчастях и материалах.
Расчет статистики работы отдела обслуживания:
Количество выполненных заявок;
Среднее время выполнения заявки;
Статистика по типам неисправностей.

import tkinter as tk
from tkinter import messagebox, ttk
import sqlite3
from datetime import datetime

# Создание/подключение к базе данных
conn = sqlite3.connect('service_requests.db')
c = conn.cursor()

# Создание таблицы заявок
c.execute('''CREATE TABLE IF NOT EXISTS requests (
             id INTEGER PRIMARY KEY AUTOINCREMENT,
             request_number TEXT,
             date_added TEXT,
             car_type TEXT,
             car_model TEXT,
             problem_description TEXT,
             client_name TEXT,
             phone_number TEXT,
             status TEXT,
             mechanic TEXT,
             comments TEXT
             )''')
conn.commit()

# Основное окно приложения
class ServiceApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Service Requests Management")

        # Вкладки
        self.notebook = ttk.Notebook(root)
        self.notebook.pack(fill='both', expand=True)

        # Вкладка добавления заявок
        self.add_tab = ttk.Frame(self.notebook)
        self.notebook.add(self.add_tab, text="Добавить заявку")
        self.create_add_tab()

        # Вкладка редактирования заявок
        self.edit_tab = ttk.Frame(self.notebook)
        self.notebook.add(self.edit_tab, text="Редактировать заявки")
        self.create_edit_tab()

        # Вкладка просмотра заявок
        self.view_tab = ttk.Frame(self.notebook)
        self.notebook.add(self.view_tab, text="Просмотр заявок")
        self.create_view_tab()

        # Вкладка статистики
        self.stats_tab = ttk.Frame(self.notebook)
        self.notebook.add(self.stats_tab, text="Статистика")
        self.create_stats_tab()

    def create_add_tab(self):
        # Поля ввода данных
        labels = ["Номер заявки", "Дата добавления", "Вид авто", "Модель авто", "Описание проблемы", "ФИО клиента", "Номер телефона", "Статус"]
        self.entries = {}

        for i, label in enumerate(labels):
            lbl = tk.Label(self.add_tab, text=label)
            lbl.grid(row=i, column=0, padx=10, pady=5, sticky=tk.W)
            entry = tk.Entry(self.add_tab, width=40)
            entry.grid(row=i, column=1, padx=10, pady=5)
            self.entries[label] = entry

        # Кнопка добавления заявки
        self.add_button = tk.Button(self.add_tab, text="Добавить заявку", command=self.add_request)
        self.add_button.grid(row=len(labels), column=1, pady=10)

    def add_request(self):
        # Сохранение заявки в базу данных
        data = [self.entries[label].get() for label in self.entries]
        c.execute("INSERT INTO requests (request_number, date_added, car_type, car_model, problem_description, client_name, phone_number, status) VALUES (?, ?, ?, ?, ?, ?, ?, ?)",
                  data)
        conn.commit()
        messagebox.showinfo("Успех", "Заявка добавлена!")
        for entry in self.entries.values():
            entry.delete(0, tk.END)

    def create_edit_tab(self):
        # Поле для поиска заявки по номеру
        self.search_entry = tk.Entry(self.edit_tab, width=40)
        self.search_entry.grid(row=0, column=0, padx=10, pady=10)
        self.search_button = tk.Button(self.edit_tab, text="Поиск", command=self.search_request)
        self.search_button.grid(row=0, column=1, padx=10, pady=10)

        # Поля для редактирования заявки
        self.edit_entries = {}
        labels = ["Номер заявки", "Дата добавления", "Вид авто", "Модель авто", "Описание проблемы", "ФИО клиента", "Номер телефона", "Статус", "Механик", "Комментарии"]
        for i, label in enumerate(labels):
            lbl = tk.Label(self.edit_tab, text=label)
            lbl.grid(row=i+1, column=0, padx=10, pady=5, sticky=tk.W)
            entry = tk.Entry(self.edit_tab, width=40)
            entry.grid(row=i+1, column=1, padx=10, pady=5)
            self.edit_entries[label] = entry

        # Кнопка сохранения изменений
        self.save_button = tk.Button(self.edit_tab, text="Сохранить изменения", command=self.save_changes)
        self.save_button.grid(row=len(labels)+1, column=1, pady=10)

    def search_request(self):
        request_number = self.search_entry.get()
        c.execute("SELECT * FROM requests WHERE request_number=?", (request_number,))
        request = c.fetchone()
        if request:
            for i, label in enumerate(self.edit_entries):
                self.edit_entries[label].delete(0, tk.END)
                self.edit_entries[label].insert(0, request[i+1])
        else:
            messagebox.showerror("Ошибка", "Заявка не найдена")

    def save_changes(self):
        request_number = self.edit_entries["Номер заявки"].get()
        data = [self.edit_entries[label].get() for label in self.edit_entries]
        data.append(request_number)
        c.execute("UPDATE requests SET request_number=?, date_added=?, car_type=?, car_model=?, problem_description=?, client_name=?, phone_number=?, status=?, mechanic=?, comments=? WHERE request_number=?", data)
        conn.commit()
        messagebox.showinfo("Успех", "Изменения сохранены")

    def create_view_tab(self):
        self.requests_tree = ttk.Treeview(self.view_tab, columns=("number", "date", "type", "model", "description", "client", "phone", "status", "mechanic", "comments"), show="headings")
        for col in self.requests_tree["columns"]:
            self.requests_tree.heading(col, text=col)
            self.requests_tree.column(col, width=100)
        self.requests_tree.pack(fill="both", expand=True)
        self.load_requests()

    def load_requests(self):
        for row in self.requests_tree.get_children():
            self.requests_tree.delete(row)
        c.execute("SELECT * FROM requests")
        for row in c.fetchall():
            self.requests_tree.insert("", "end", values=row[1:])

    def create_stats_tab(self):
        self.stats_text = tk.Text(self.stats_tab, width=100, height=20)
        self.stats_text.pack(fill="both", expand=True)
        self.load_stats()

    def load_stats(self):
        c.execute("SELECT COUNT(*) FROM requests WHERE status='завершена'")
        completed_requests = c.fetchone()[0]

        c.execute("SELECT AVG(julianday('now') - julianday(date_added)) FROM requests WHERE status='завершена'")
        avg_time = c.fetchone()[0]

        stats_text = f"Количество выполненных заявок: {completed_requests}\n"
        stats_text += f"Среднее время выполнения заявки: {avg_time:.2f} дней\n"

        self.stats_text.insert("1.0", stats_text)

# Запуск приложения
root = tk.Tk()
app = ServiceApp(root)
root.mainloop()
