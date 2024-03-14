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
(or)
import sqlite3
from datetime import date
def connect_to_database():
    return sqlite3.connect('library.db')
def add_book(title, authors, isbn, publisher, quantity):
    conn = connect_to_database()
    c = conn.cursor()
    c.execute("INSERT INTO books (title, authors, isbn, publisher, quantity) VALUES (?, ?, ?, ?, ?)",
              (title, authors, isbn, publisher, quantity))
    conn.commit()
    conn.close()
def add_member(name, email, phone):
    conn = connect_to_database()
    c = conn.cursor()
    c.execute("INSERT INTO members (name, email, phone) VALUES (?, ?, ?)", (name, email, phone))
    conn.commit()
    conn.close()
def issue_book(book_id, member_id):
    conn = connect_to_database()
    c = conn.cursor()
    issue_date = date.today()
    c.execute("INSERT INTO transactions (book_id, member_id, issue_date) VALUES (?, ?, ?)",
              (book_id, member_id, issue_date))
    conn.commit()
    conn.close()
def return_book(transaction_id):
    conn = connect_to_database()
    c = conn.cursor()
    return_date = date.today()
    c.execute("UPDATE transactions SET return_date = ? WHERE id = ?", (return_date, transaction_id))
    conn.commit()
    conn.close()
def search_books(keyword):
    conn = connect_to_database()
    c = conn.cursor()
    c.execute("SELECT * FROM books WHERE title LIKE ? OR authors LIKE ?", ('%' + keyword + '%', '%' + keyword + '%'))
    books = c.fetchall()
    conn.close()
    return books
def member_transactions(member_id):
    conn = connect_to_database()
    c = conn.cursor()
    c.execute("SELECT * FROM transactions WHERE member_id = ?", (member_id,))
    transactions = c.fetchall()
    conn.close()
    return transactions
def main():
    while True:
        print("\nLibrary Management System\n")
        print("1. Add Book")
        print("2. Add Member")
        print("3. Issue Book")
        print("4. Return Book")
        print("5. Search Books")
        print("6. View Member Transactions")
        print("0. Exit")
        
        choice = input("\nEnter your choice: ")
        
        if choice == "1":
            title = input("Enter book title: ")
            authors = input("Enter author(s): ")
            isbn = input("Enter ISBN: ")
            publisher = input("Enter publisher: ")
            quantity = int(input("Enter quantity: "))
            add_book(title, authors, isbn, publisher, quantity)
            print("Book added successfully!")
        elif choice == "2":
            name = input("Enter member name: ")
            email = input("Enter member email: ")
            phone = input("Enter member phone: ")
            add_member(name, email, phone)
            print("Member added successfully!")
        elif choice == "3":
            book_id = int(input("Enter book ID: "))
            member_id = int(input("Enter member ID: "))
            issue_book(book_id, member_id)
            print("Book issued successfully!")
        elif choice == "4":
            transaction_id = int(input("Enter transaction ID: "))
            return_book(transaction_id)
            print("Book returned successfully!")
        elif choice == "5":
            keyword = input("Enter book title or author: ")
            books = search_books(keyword)
            if books:
                print("\nSearch results:")
                for book in books:
                    print(book)
            else:
                print("No matching books found.")
        elif choice == "6":
            member_id = int(input("Enter member ID: "))
            transactions = member_transactions(member_id)
            if transactions:
                print("\nMember transactions:")
                for transaction in transactions:
                    print(transaction)
            else:
                print("No transactions found for this member.")
        elif choice == "0":
            print("Exiting program...")
            break
        else:
            print("Invalid choice. Please try again.")

if _name_ == "_main_":
    main()
