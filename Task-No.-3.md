<code>from tkinter import *
import tkinter as tk
from tkinter import ttk
import pymysql
from PIL import ImageTk

def execute_query(query, fetch=False):
    connection = pymysql.connect(
        host='localhost',
        user='root',
        password='',
        database='trading_organization_llc_torg'
    )
    cursor = connection.cursor()
    cursor.execute(query)
    if fetch:
        return cursor.fetchall()
    connection.commit()
    connection.close()

def update_client_dropdown():
    query = "SELECT FIO FROM form"
    clients = [row[0] for row in execute_query(query, fetch=True)]
    client_var.set("")
    client_dropdown['values'] = clients

def update_table():
    selected_client = client_var.get()
    sort_field = sort_field_var.get()
    sort_order = sort_order_var.get()

    query = f"SELECT * FROM form"
    
    if selected_client:
        query += f" WHERE FIO = '{selected_client}'"
    
    query += f" ORDER BY {sort_field} {sort_order}"
    
    data = execute_query(query, fetch=True)

    for i in tree.get_children():
        tree.delete(i)

    for record in data:
        tree.insert("", "end", values=record)

def clear_filters():
    client_var.set("")
    sort_field_var.set("FIO")
    sort_order_var.set("ASC")
    update_table()

def search():
    search_text = search_entry.get().lower()
    for child in tree.get_children():
        for value in tree.item(child)['values']:
            if search_text in str(value).lower():
                tree.item(child, tags=('found',))
                tree.tag_configure('found', background='yellow')
                break
        else:
            tree.item(child, tags=())    

root = tk.Tk()
root.title("Торговая организация ООО «Торг»")
icon = PhotoImage(file = "favicon1.png")
root.iconphoto(False, icon)

filter_frame = tk.Frame(root)
filter_frame.pack(pady=10)

client_label = tk.Label(filter_frame, text="Выберите клиента:")
client_label.grid(row=0, column=0)
client_var = tk.StringVar()
client_dropdown = ttk.Combobox(filter_frame, textvariable=client_var)
client_dropdown.grid(row=0, column=1)
update_client_dropdown()

filter_button = tk.Button(filter_frame, text="Применить фильтры", command=update_table)
filter_button.grid(row=0, column=2, padx=10)
clear_button = tk.Button(filter_frame, text="Сбросить фильтры", command=clear_filters)
clear_button.grid(row=0, column=3)

search_label = tk.Label(filter_frame, text="Поиск:")
search_label.grid(row=1, column=0)
search_entry = tk.Entry(filter_frame)
search_entry.grid(row=1, column=1)
search_button = tk.Button(filter_frame, text="Найти", command=search)
search_button.grid(row=1, column=2)

sort_frame = tk.Frame(root)
sort_frame.pack(pady=10)

sort_field_label = tk.Label(sort_frame, text="Выберите поле для сортировки:")
sort_field_label.grid(row=0, column=0)
sort_field_var = tk.StringVar()
sort_field_dropdown = ttk.Combobox(sort_frame, textvariable=sort_field_var, values=["FIO", "Date_of_sale", "Status"])
sort_field_dropdown.grid(row=0, column=1)
sort_field_dropdown.set("FIO")

sort_order_label = tk.Label(sort_frame, text="Выберите порядок сортировки:")
sort_order_label.grid(row=0, column=2)
sort_order_var = tk.StringVar()
sort_order_var.set("ASC")
sort_order_radiobutton_asc = ttk.Radiobutton(sort_frame, text="По возрастанию", variable=sort_order_var, value="ASC")
sort_order_radiobutton_asc.grid(row=0, column=3)
sort_order_radiobutton_desc = ttk.Radiobutton(sort_frame, text="По убыванию", variable=sort_order_var, value="DESC")
sort_order_radiobutton_desc.grid(row=0, column=4)

sort_button = tk.Button(sort_frame, text="Сортировать", command=update_table)
sort_button.grid(row=0, column=5, padx=10)


tree = ttk.Treeview(root, columns=("ID","FIO", "Phone", "E_mail", "Date_of_sale", "Status"), show="headings")
tree.heading("ID", text="ID")
tree.heading("FIO", text="ФИО")
tree.heading("Phone", text="Телефон")
tree.heading("E_mail", text="Эл.почта")
tree.heading("Date_of_sale", text="Дата заказа")
tree.heading("Status", text="Статус")
tree.pack(padx=10, pady=10)

update_table()

root.mainloop()</code>
