# fast_app

from fastapi import FastAPI
from pydantic import BaseModel
import sqlite3
from typing import List

app = FastAPI()

# دالة للبحث في قاعدة البيانات
def search_database(query: str):
    conn = sqlite3.connect("database.db")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM pages WHERE title LIKE ? OR content LIKE ?", (f'%{query}%', f'%{query}%'))
    results = cursor.fetchall()
    conn.close()
    return results

# نموذج البيانات (Response)
class SearchResult(BaseModel):
    title: str
    content: str
    url: str

# الصفحة الرئيسية
@app.get("/")
def home():
    return {"message": "مرحبًا بك في محرك البحث API"}

# نقطة النهاية للبحث
@app.get("/search", response_model=List[SearchResult])
def search(query: str):
    if not query:
        return {"error": "يجب إدخال مصطلح للبحث"}
    results = search_database(query)
    return [{"title": r[1], "content": r[2], "url": r[3]} for r in results]
