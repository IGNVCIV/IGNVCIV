import os
import re
from pathlib import Path

import requests
from deep_translator import GoogleTranslator


README_PATH = Path("README.md")

FACT_URL = "https://uselessfacts.jsph.pl/api/v2/facts/random"

START_MARKER = "<!-- START_SECTION:fact -->"
END_MARKER = "<!-- END_SECTION:fact -->"

SUPPORTED_LANGUAGES = ("en", "es", "pt")

LANGUAGE_CONFIG = {
    "en": {
        "label": "Curious fact",
        "flag": "🇬🇧",
        "fallback": "A new curious fact will appear here soon.",
    },
    "es": {
        "label": "Dato curioso",
        "flag": "🇪🇸",
        "fallback": "Pronto aparecerá un nuevo dato curioso.",
    },
    "pt": {
        "label": "Curiosidade",
        "flag": "🇧🇷",
        "fallback": "Em breve aparecerá uma nova curiosidade.",
    },
}


def obtener_idiomas() -> list[str]:
    """
    Lee los idiomas desde README_LANGUAGES.

    Valores válidos:
        all
        en
        es
        pt
        en,es,pt
    """

    valor = os.getenv("README_LANGUAGES", "all").strip().lower()

    if valor == "all":
        return list(SUPPORTED_LANGUAGES)

    idiomas = [
        idioma.strip()
        for idioma in valor.split(",")
        if idioma.strip()
    ]

    idiomas = list(dict.fromkeys(idiomas))

    invalidos = [
        idioma
        for idioma in idiomas
        if idioma not in SUPPORTED_LANGUAGES
    ]

    if invalidos:
        raise ValueError(
            "Idiomas no compatibles: "
            + ", ".join(invalidos)
            + ". Usa en, es, pt o all."
        )

    if not idiomas:
        raise ValueError("Debes seleccionar al menos un idioma.")

    return idiomas


def obtener_dato_curioso() -> str:
    """Obtiene un dato curioso general en inglés."""

    respuesta = requests.get(
        FACT_URL,
        params={"language": "en"},
        headers={
            "Accept": "application/json",
            "User-Agent": "github-readme-updater/3.0",
        },
        timeout=15,
    )

    respuesta.raise_for_status()

    contenido = respuesta.json()
    dato = contenido.get("text", "").strip()

    # Elimina saltos de línea y espacios repetidos.
    dato = " ".join(dato.split())

    if not dato:
        raise ValueError("La API devolvió un dato vacío.")

    return dato


def traducir_dato(texto: str, idioma: str) -> str:
    """Traduce el dato sin utilizar una clave de DeepL."""

    if idioma == "en":
        return texto

    traduccion = GoogleTranslator(
        source="en",
        target=idioma,
    ).translate(texto)

    traduccion = " ".join(traduccion.split())

    if not traduccion:
        raise ValueError(
            f"La traducción al idioma '{idioma}' está vacía."
        )

    return traduccion


def preparar_traducciones(
    dato_original: str,
    idiomas: list[str],
) -> dict[str, str]:
    """Genera una traducción para cada idioma solicitado."""

    traducciones = {}

    for idioma in idiomas:
        try:
            traducciones[idioma] = traducir_dato(
                dato_original,
                idioma,
            )
        except Exception as error:
            print(
                f"Advertencia: no se pudo traducir a "
                f"'{idioma}': {error}"
            )

            traducciones[idioma] = (
                LANGUAGE_CONFIG[idioma]["fallback"]
            )

    return traducciones


def construir_bloque(
    traducciones: dict[str, str],
    idiomas: list[str],
) -> str:
    """Construye la sección Markdown del README."""

    lineas = [START_MARKER]

    for indice, idioma in enumerate(idiomas):
        configuracion = LANGUAGE_CONFIG[idioma]

        lineas.append(
            f"> {configuracion['flag']} "
            f"**{configuracion['label']}:** "
            f"{traducciones[idioma]} 🐾"
        )

        if indice < len(idiomas) - 1:
            lineas.append(">")

    lineas.append(END_MARKER)

    return "\n".join(lineas)


def actualizar_readme() -> bool:
    """Reemplaza la sección dinámica del README."""

    if not README_PATH.exists():
        raise FileNotFoundError(
            f"No se encontró {README_PATH.resolve()}."
        )

    contenido_actual = README_PATH.read_text(
        encoding="utf-8",
    )

    patron = re.compile(
        rf"{re.escape(START_MARKER)}"
        rf".*?"
        rf"{re.escape(END_MARKER)}",
        flags=re.DOTALL,
    )

    if not patron.search(contenido_actual):
        raise ValueError(
            "No se encontraron los marcadores "
            "de la sección en README.md."
        )

    idiomas = obtener_idiomas()

    try:
        dato_original = obtener_dato_curioso()

        traducciones = preparar_traducciones(
            dato_original,
            idiomas,
        )

    except (requests.RequestException, ValueError) as error:
        print(
            "Advertencia: no se pudo obtener "
            f"el dato curioso: {error}"
        )

        traducciones = {
            idioma: LANGUAGE_CONFIG[idioma]["fallback"]
            for idioma in idiomas
        }

    nuevo_bloque = construir_bloque(
        traducciones,
        idiomas,
    )

    contenido_actualizado = patron.sub(
        lambda _: nuevo_bloque,
        contenido_actual,
        count=1,
    )

    if contenido_actualizado == contenido_actual:
        print("🐾 El README ya estaba actualizado.")
        return False

    README_PATH.write_text(
        contenido_actualizado,
        encoding="utf-8",
    )

    return True


def main() -> None:
    try:
        actualizado = actualizar_readme()

        if actualizado:
            print(
                "🐾 README actualizado correctamente "
                "en inglés, español y portugués."
            )

    except (FileNotFoundError, ValueError) as error:
        print(f"Error: {error}")
        raise SystemExit(1) from error


if __name__ == "__main__":
    main()
