import certifi
import pandas as pd
import requests
from bs4 import BeautifulSoup

URL = "https://www.bcu.gub.uy/Estadisticas-e-Indicadores/Paginas/Cotizaciones.aspx"


def fetch_cotizaciones(url: str = URL) -> str:
    """Return HTML page content from BCU using certifi certificates."""
    response = requests.get(url, verify=certifi.where())
    response.raise_for_status()
    return response.text


def parse_cotizaciones(html: str) -> pd.DataFrame:
    """Parse cotizaciones for USD and Euro from HTML."""
    soup = BeautifulSoup(html, "html.parser")
    tables = soup.find_all("table")
    cotizaciones = []
    for table in tables:
        if "DLS. USA BILLETE" in table.text or "EURO" in table.text:
            rows = table.find_all("tr")
            for row in rows:
                cols = row.find_all("td")
                if len(cols) >= 4:
                    moneda = cols[0].text.strip()
                    fecha = cols[1].text.strip()
                    compra = cols[2].text.strip()
                    venta = cols[3].text.strip()
                    if moneda in ["DLS. USA BILLETE", "EURO"]:
                        cotizaciones.append(
                            {
                                "Moneda": moneda,
                                "Fecha": fecha,
                                "Compra": compra,
                                "Venta": venta,
                            }
                        )
    return pd.DataFrame(cotizaciones)


def main() -> None:
    html = fetch_cotizaciones()
    df = parse_cotizaciones(html)
    df.to_excel("cotizaciones_desde_html.xlsx", index=False)
    print("✅ Cotizaciones extraídas y guardadas como cotizaciones_desde_html.xlsx")
    print(df.head())


if __name__ == "__main__":
    main()
