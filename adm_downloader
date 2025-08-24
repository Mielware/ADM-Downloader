# file: get_adm.py
import os
import io
import re
import zipfile
import requests
from bs4 import BeautifulSoup

BASE = "https://pubfs-rma.fpac.usda.gov/pub/References/actuarial_data_master/"
OUT = os.path.join("data", "ADM")
os.makedirs(OUT, exist_ok=True)

def list_dir(url: str):
    r = requests.get(url, timeout=60)
    r.raise_for_status()
    soup = BeautifulSoup(r.text, "html.parser")
    # capture both file and directory hrefs
    hrefs = [a.get("href") for a in soup.select("a[href]")]
    return hrefs

def years():
    hrefs = list_dir(BASE)
    # directories like "2018/" etc.
    return sorted(y for y in hrefs if re.fullmatch(r"\d{4}/", y))

def download_and_extract(zip_url: str, to_dir: str):
    os.makedirs(to_dir, exist_ok=True)
    print(f"→ Downloading {zip_url}")
    r = requests.get(zip_url, timeout=600)
    r.raise_for_status()
    with zipfile.ZipFile(io.BytesIO(r.content)) as z:
        z.extractall(to_dir)
    print(f"   Extracted into {to_dir}")

def main():
    for y in years():
        year = y.strip("/")
        year_url = BASE + y
        print(f"\n=== Year {year} ===")
        hrefs = list_dir(year_url)
        zips = [h for h in hrefs if h.lower().endswith(".zip")]
        if not zips:
            print("   (no ZIPs found)")
            continue
        for z in zips:
            download_and_extract(year_url + z, os.path.join(OUT, year))
    print("\nAll ADM files downloaded and extracted.")

if __name__ == "__main__":
    main()
