from pathlib import Path
import xml.etree.ElementTree as ET

import requests


RSS_ORIGINAL = "https://www.prim.es/category/noticias/feed/"
ARCHIVO_SALIDA = Path("prim.xml")

CABECERAS = {
    "User-Agent": (
        "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
        "AppleWebKit/537.36 (KHTML, like Gecko) "
        "Chrome/131.0.0.0 Safari/537.36"
    ),
    "Accept": (
        "application/rss+xml, application/xml, text/xml, "
        "application/xhtml+xml, text/html;q=0.9,*/*;q=0.8"
    ),
    "Accept-Language": "es-ES,es;q=0.9,en;q=0.8",
    "Cache-Control": "no-cache",
}


def descargar_rss() -> bytes:
    respuesta = requests.get(
        RSS_ORIGINAL,
        headers=CABECERAS,
        timeout=60,
        allow_redirects=True,
    )
    respuesta.raise_for_status()

    contenido = respuesta.content

    if not contenido.strip():
        raise RuntimeError("PRIM devolvió una respuesta vacía")

    return contenido


def validar_rss(contenido: bytes) -> int:
    try:
        raiz = ET.fromstring(contenido)
    except ET.ParseError as error:
        inicio = contenido[:300].decode("utf-8", errors="replace")
        raise RuntimeError(
            f"PRIM no devolvió un XML válido. Inicio: {inicio}"
        ) from error

    nombre_raiz = raiz.tag.lower()

    if not (
        nombre_raiz.endswith("rss")
        or nombre_raiz.endswith("feed")
        or nombre_raiz.endswith("rdf")
    ):
        raise RuntimeError(
            f"El documento recibido no parece una RSS: {raiz.tag}"
        )

    noticias = raiz.findall(".//item")

    if not noticias:
        noticias = [
            elemento
            for elemento in raiz.iter()
            if elemento.tag.lower().endswith("entry")
        ]

    if not noticias:
        raise RuntimeError(
            "La RSS de PRIM no contiene ninguna noticia"
        )

    return len(noticias)


def guardar_rss(contenido: bytes) -> None:
    archivo_temporal = ARCHIVO_SALIDA.with_suffix(".xml.tmp")
    archivo_temporal.write_bytes(contenido)
    archivo_temporal.replace(ARCHIVO_SALIDA)


def main() -> None:
    contenido = descargar_rss()
    numero_noticias = validar_rss(contenido)
    guardar_rss(contenido)

    print(
        f"RSS de PRIM actualizada correctamente: "
        f"{numero_noticias} noticias"
    )


if __name__ == "__main__":
    main()
