import certifi
import pandas as pd
import requests
from bs4 import BeautifulSoup
from pathlib import Path

URL = "https://www.bcu.gub.uy/Estadisticas-e-Indicadores/Paginas/Cotizaciones.aspx"

def fetch_cotizaciones(from_date: str, to_date: str, url: str = URL) -> str:
    """Fetch page content with a date range.

    Tries with certifi bundle first; on SSL failure, retries without
    certificate verification as a fallback.
    """
    params = {"FecDesde": from_date, "FecHasta": to_date}
    try:
        response = requests.get(url, params=params, verify=certifi.where(), timeout=10)
        response.raise_for_status()
        return response.text
    except requests.exceptions.SSLError:
        print("Advertencia: error SSL, intentando sin verificación de certificado...")
        response = requests.get(url, params=params, verify=False, timeout=10)
        response.raise_for_status()
        return response.text

def parse_cotizaciones(html: str) -> pd.DataFrame:
    """Parse cotizaciones for USD and Euro from HTML."""
    soup = BeautifulSoup(html, "html.parser")
    tables = soup.find_all("table")
    cotizaciones = []
    for table in tables:
        if "DLS. USA BILLETE" in table.text or "EURO" in table.text:
            for row in table.find_all("tr"):
                cols = row.find_all("td")
                if len(cols) >= 4:
                    moneda = cols[0].text.strip()
                    fecha = cols[1].text.strip()
                    compra = cols[2].text.strip()
                    venta = cols[3].text.strip()
                    if moneda in ["DLS. USA BILLETE", "EURO"]:
                        cotizaciones.append({
                            "Moneda": moneda,
                            "Fecha": fecha,
                            "Compra": compra,
                            "Venta": venta,
                        })
    return pd.DataFrame(cotizaciones)

def save_excel(df: pd.DataFrame, path: str) -> None:
    """Save DataFrame to Excel handling PermissionError."""
    try:
        df.to_excel(path, index=False)
    except PermissionError:
        print(f"No se pudo escribir en {path}. ¿Está abierto? Guardando como temporal...")
        tmp = Path(path).with_suffix(".tmp.xlsx")
        df.to_excel(tmp, index=False)
        print(f"Archivo guardado como {tmp}")

def main() -> None:
    html = fetch_cotizaciones("01/11/2024", "31/03/2025")
    df = parse_cotizaciones(html)
    save_excel(df, "cotizaciones_desde_html.xlsx")
    print("✅ Cotizaciones extraídas")
    print(df.head())

if __name__ == "__main__":
    main()
