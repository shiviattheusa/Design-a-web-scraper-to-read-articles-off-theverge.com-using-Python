# Design-a-web-scraper-to-read-articles-off-theverge.com-using-Python
Read the headline, get the link of the article, the author, and the date of each of the articles found on "theverge.com" - Store these in a CSV file titled `ddmmyyy_verge.csv`, with the following header `id, URL, headline, author, date`. - Create an SQLite database to store the same data, and make sure that the id is the primary key

import requests
from bs4 import BeautifulSoup
import csv
import sqlite3
from datetime import datetime

# Scrape data from theverge.com
url = "https://www.theverge.com/"
response = requests.get(url)
soup = BeautifulSoup(response.text, "html.parser")
articles = soup.find_all("div", class_="c-entry-box--compact__body")

# Get the current date in ddmmyyyy format
date = datetime.now().strftime("%d%m%Y")

# Save the data in CSV file and SQLite database
csv_file = f"{date}_verge.csv"
db_file = f"{date}_verge.db"

conn = sqlite3.connect(db_file)
c = conn.cursor()
c.execute('''CREATE TABLE articles
             (id INTEGER PRIMARY KEY, url TEXT, headline TEXT, author TEXT, date TEXT)''')

with open(csv_file, "w", newline="") as file:
    writer = csv.writer(file)
    writer.writerow(["id", "URL", "headline", "author", "date"])

    for i, article in enumerate(articles):
        url = article.find("a")["href"]
        headline = article.find("h2").text.strip()
        author = article.find("span", class_="c-byline__item").text.strip()
        date = article.find("time")["datetime"]

        # Write to CSV file
        writer.writerow([i+1, url, headline, author, date])

        # Write to SQLite database
        c.execute(f"INSERT INTO articles VALUES ({i+1}, '{url}', '{headline}', '{author}', '{date}')")

conn.commit()
conn.close()
