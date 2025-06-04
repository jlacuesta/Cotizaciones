# Cotizaciones
Script para extraer cotizaciones de BCU Uruguay
from bs4 import BeautifulSoup
import pandas as pd

# Ruta al archivo HTML descargado
archivo_html = "cotizaciones_bcu.html"

# Abrir y parsear el HTML
with open(archivo_html, 'r', encoding='utf-8') as f:
    soup = BeautifulSoup(f, 'html.parser')

# Buscar todas las tablas
tablas = soup.find_all('table')

# Extraer cotizaciones
cotizaciones = []

for tabla in tablas:
    if 'DLS. USA BILLETE' in tabla.text or 'EURO' in tabla.text:
        filas = tabla.find_all('tr')
        for fila in filas:
            columnas = fila.find_all('td')
            if len(columnas) >= 4:
                moneda = columnas[0].text.strip()
                fecha = columnas[1].text.strip()
                compra = columnas[2].text.strip()
                venta = columnas[3].text.strip()

                if moneda in ['DLS. USA BILLETE', 'EURO']:
                    cotizaciones.append({
                        'Moneda': moneda,
                        'Fecha': fecha,
                        'Compra': compra,
                        'Venta': venta
                    })

# Convertir a DataFrame
df = pd.DataFrame(cotizaciones)

# Guardar en Excel
df.to_excel("cotizaciones_desde_html.xlsx", index=False)

print("✅ Cotizaciones extraídas y guardadas como cotizaciones_desde_html.xlsx")
print(df.head())

