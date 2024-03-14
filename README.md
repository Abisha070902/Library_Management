# Library_Management
import sqlite3
import requests
import json
from flask import Flask, request, jsonify

def create_tables():
    conn = sqlite3.connect('library.db')
    c = conn.cursor()

    # Create table for books
    c.execute('''CREATE TABLE IF NOT EXISTS books (
                    id INTEGER PRIMARY KEY,
                    title TEXT NOT NULL,
                    authors TEXT,
                    isbn TEXT,
                    publisher TEXT,
                    quantity INTEGER DEFAULT 0
                )''')

    c.execute('''CREATE TABLE IF NOT EXISTS members (
                    id INTEGER PRIMARY KEY,
                    name TEXT NOT NULL,
                    email TEXT,
                    phone TEXT
                )''')

    c.execute('''CREATE TABLE IF NOT EXISTS transactions (
                    id INTEGER PRIMARY KEY,
                    book_id INTEGER,
                    member_id INTEGER,
                    issue_date DATE,
                    return_date DATE,
                    FOREIGN KEY (book_id) REFERENCES books (id),
                    FOREIGN KEY (member_id) REFERENCES members (id)
                )''')

    conn.commit()
    conn.close()

def fetch_books(page, title=None, authors=None, isbn=None, publisher=None, page_size=20):
    FRAPPE_API_URL = "https://frappe.io/api/method/frappe-library"
    params = {"page": page, "title": title, "authors": authors, "isbn": isbn, "publisher": publisher}
    response = requests.get(FRAPPE_API_URL, params=params)
    if response.status_code == 200:
        return json.loads(response.text)
    else:
        print("Error fetching books:", response.text)
        return None

app = Flask(_name_)

@app.route('/import_books', methods=['POST'])
def import_books():
    data = request.get_json()
    title = data.get('title')
    page_size = data.get('page_size', 20)
    import_books_from_api(title, page_size)
    return jsonify({"message": "Books imported successfully"})

def import_books_from_api(title, page_size=20):
    page = 1
    while True:
        books_data = fetch_books(page, title=title, page_size=page_size)
        if not books_data:
            break
        for book in books_data["message"]:
            process_book(book)
        page += 1

def process_book(book):
    # Process and insert book records into the database
    pass

def search_books(keyword):
    conn = sqlite3.connect('library.db')
    c = conn.cursor()
    c.execute("SELECT * FROM books WHERE title LIKE ? OR authors LIKE ?", ('%' + keyword + '%', '%' + keyword + '%'))
    books = c.fetchall()
    conn.close()
    return books

if _name_ == "_main_":
    create_tables()
    app.run(debug=True)
