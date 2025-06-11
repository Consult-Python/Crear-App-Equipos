# Crear-App-Equipos

Este repo tiene una app web hecha en Flask que sirve para armar grupos al azar a partir de una lista de nombres. Ponés los nombres y cuántos grupos querés, y la app los reparte parejo. <br> 
Todo queda guardado en una base de datos, y después podés ver el historial de sorteos. #PhytonVenezuela

from flask import Flask, render_template, request, flash
import sqlite3
from datetime import datetime
import random

app = Flask(__name__)
app.secret_key = 'clave_secreta'

@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        nombres_str = request.form["nombres"]
        num_grupos = int(request.form["grupos"])
        lista_nombres = [n.strip() for n in nombres_str.split(",") if n.strip()]

        if num_grupos > len(lista_nombres):
            flash("El número de grupos no puede ser mayor que el número de personas.")
            return render_template("index.html", nombres=nombres_str, grupos=num_grupos)

        # Aleatorizar y crear grupos
        random.shuffle(lista_nombres)
        grupos = [[] for _ in range(num_grupos)]
        for i, nombre in enumerate(lista_nombres):
            grupos[i % num_grupos].append(nombre)

        # Insertar en la base de datos
        fecha = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        with sqlite3.connect("grupos.db") as conn:
            for idx, grupo in enumerate(grupos, 1):
                for nombre in grupo:
                    conn.execute("INSERT INTO sorteos (nombre, grupo, fecha) VALUES (?, ?, ?)",
                                 (nombre, idx, fecha))

        return render_template("grupos.html", grupos=grupos)

    return render_template("index.html")

@app.route("/resultados")
def resultados():
    with sqlite3.connect("grupos.db") as conn:
        cursor = conn.cursor()
        cursor.execute("SELECT nombre, grupo, fecha FROM sorteos ORDER BY fecha DESC")
        datos = cursor.fetchall()
    return render_template("resultados.html", datos=datos)

if __name__ == "__main__":
    app.run(debug=True)

<br>
- BBDD
import sqlite3

def crear_tabla():
    with sqlite3.connect("grupos.db") as conn:
        conn.execute('''
            CREATE TABLE IF NOT EXISTS sorteos (
                nombre TEXT,
                grupo INTEGER,
                fecha TEXT
            )
        ''')
        print("Base de datos inicializada correctamente.")

if __name__ == "__main__":
    crear_tabla()

<br>

  

