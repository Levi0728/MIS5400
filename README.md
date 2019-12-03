# MIS5400

# Access SQL and post to API
from flask import Flask
import pyodbc
from flask import Flask, g, render_template, abort, request
import json

app = Flask(__name__)

app.config.from_object(__name__)

@app.before_request
def before_request():
    try:
        g.sql_conn = pyodbc.connect(r'DRIVER={ODBC Driver 13 for SQL server};'      
                                    r'SERVER=LAPTOP-30499G3V;'  
                                    r'DATABASE=PythonConnect;'    
                                    r'Trusted_Connection=yes', autocommit=True)
    except Exception:
        abort(500, "No database connection could be established.")


@app.teardown_request
def teardown_request(exception):
    try:
        g.sql_conn.close()
    except AttributeError:
        pass

@app.route('/')
@app.route('/home', methods=['GET'])
def home():
    curs = g.sql_conn.cursor()
    query = 'select * from PythonConnect.dbo.Table_1 '
    curs.execute(query)

    columns = [column[0] for column in curs.description]
    data = []

    for row in curs.fetchall():
        data.append(dict(zip(columns, row)))
    return json.dumps(data, indent=4, sort_keys=True, default=str)


if __name__ == '__main__':
    app.run()
    
    
# Pull API and post on Juypter
import requests as r
import pandas as pd
data = r.get('http://127.0.0.1:5000/')
df = pd.read_json('http://127.0.0.1:5000/')
