# ADM-Downloader
Downloader for ADM Files
import os
import requests
from bs4 import BeautifulSoup
import zipfile
import io
import pandas as pd

# Base FTP directory for ADM files
BASE_URL = "https://pubfs-rma.fpac.usda.gov/pub/References/actuarial_data_master/"

# Local folder to save ADM files
OUTPUT_DIR = "ADM_files"
os.makedirs(OUTPUT_DIR, exist_ok=True)

def list_files_from_directory(url):
    """Scrape a USDA FTP directory listing to find all links."""
    r = requests.get(url)
    if r.status_code != 200:
        raise Exception(f"Failed to access {url}")
    soup = BeautifulSoup(r.text, "html.parser")
    files = [a["href"] for a in soup.find_all("a", href=True)]
    return files

def download_and_extract_zip(zip_url, output_dir):
    """Download a zip file from USDA and extract its contents."""
    print(f"Downloading {zip_url}...")
    r = requests.get(zip_url)
    if r.status_code != 200:
        print(f"Failed to download {zip_url}")
        return
    with zipfile.ZipFile(io.BytesIO(r.content)) as z:
        z.extractall(output_dir)
        print(f"Extracted to {output_dir}")

def main():
    # Step 1: List all year directories
    year_dirs = list_files_from_directory(BASE_URL)
    year_dirs = [d for d in year_dirs if d.strip("/").isdigit()]  # keep only year folders
    
    for year in year_dirs:
        year_url = BASE_URL + year
        print(f"Processing year: {year.strip('/')}")
        files = list_files_from_directory(year_url)
        
        # Step 2: Download all ZIP files in the year folder
        for f in files:
            if f.endswith(".zip"):
                zip_url = year_url + f
                year_out_dir = os.path.join(OUTPUT_DIR, year.strip("/"))
                os.makedirs(year_out_dir, exist_ok=True)
                download_and_extract_zip(zip_url, year_out_dir)

    print("All ADM files downloaded and extracted!")

    # Optional: Example of loading into pandas
    all_txt = []
    for root, _, files in os.walk(OUTPUT_DIR):
        for f in files:
            if f.endswith(".txt") or f.endswith(".csv"):
                filepath = os.path.join(root, f)
                try:
                    df = pd.read_csv(filepath, delimiter="|")  # ADM files are often pipe-delimited
                    all_txt.append((f, df.head()))
                except Exception as e:
                    print(f"Could not read {filepath}: {e}")
    
    print("Example files loaded into pandas:")
    for name, preview in all_txt[:5]:
        print(f"\n{name}\n", preview)

if __name__ == "__main__":
    main()
