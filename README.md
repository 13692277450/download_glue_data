# download_glue_data
download_glue_data.py  
///////////////////////////////////////////////////////////

#!/usr/bin/env python3
"""
GLUE Data Download Script - Downloads all 11 GLUE benchmark datasets
Official website: https://gluebenchmark.com/tasks/
Usage: python download_glue_data.py --data_dir glue_data
"""

import os
import sys
import argparse
import shutil
import urllib.request
import urllib.error
import zipfile
from pathlib import Path

# GLUE dataset download URLs from official Facebook Research repository
GLUE_DATASETS = {
    "CoLA": {
        "url": "https://dl.fbaipublicfiles.com/glue/data/CoLA.zip",
        "description": "Corpus of Linguistic Acceptability - Grammaticality judgments"
    },
    "SST-2": {
        "url": "https://dl.fbaipublicfiles.com/glue/data/SST-2.zip", 
        "description": "Stanford Sentiment Treebank - Sentiment classification"
    },
    "MRPC": {
        "url": "https://dl.fbaipublicfiles.com/glue/data/MRPC.zip",
        "description": "Microsoft Research Paraphrase Corpus - Paraphrase detection"
    },
    "STS-B": {
        "url": "https://dl.fbaipublicfiles.com/glue/data/STS-B.zip",
        "description": "Semantic Textual Similarity Benchmark - Similarity scoring"
    },
    "QQP": {
        "url": "https://dl.fbaipublicfiles.com/glue/data/QQP.zip",
        "description": "Quora Question Pairs - Duplicate question detection"
    },
    "MNLI": {
        "url": "https://dl.fbaipublicfiles.com/glue/data/MNLI.zip",
        "description": "Multi-Genre Natural Language Inference - Text entailment"
    },
    "QNLI": {
        "url": "https://dl.fbaipublicfiles.com/glue/data/QNLI.zip",
        "description": "Question Natural Language Inference - QA NLI task"
    },
    "RTE": {
        "url": "https://dl.fbaipublicfiles.com/glue/data/RTE.zip",
        "description": "Recognizing Textual Entailment - Text entailment"
    },
    "WNLI": {
        "url": "https://dl.fbaipublicfiles.com/glue/data/WNLI.zip",
        "description": "Winograd Natural Language Inference - Coreference NLI"
    },
    "AX": {
        "url": "https://dl.fbaipublicfiles.com/glue/data/AX.zip",
        "description": "GLUE Diagnostic set - Model analysis and diagnostics"
    },
    "diagnostic": {
        "url": "https://dl.fbaipublicfiles.com/glue/data/AX.zip",
        "description": "Same as AX, alternative name"
    }
}

def download_file(url, output_path, dataset_name):
    """
    Download a file from URL to output_path
    """
    try:
        print(f"Downloading {dataset_name}...")
        urllib.request.urlretrieve(url, output_path)
        print(f"✓ Downloaded {dataset_name} to {output_path}")
        return True
    except urllib.error.URLError as e:
        print(f"✗ Failed to download {dataset_name}: {e}")
        return False
    except Exception as e:
        print(f"✗ Error downloading {dataset_name}: {e}")
        return False

def extract_to_dataset_folder(zip_path, data_dir, dataset_name):
    """
    Extract ZIP file to dataset-specific folder
    """
    dataset_dir = os.path.join(data_dir, dataset_name)
    
    # Create dataset directory
    os.makedirs(dataset_dir, exist_ok=True)
    
    print(f"Extracting {dataset_name} to {dataset_dir}...")
    
    try:
        with zipfile.ZipFile(zip_path, 'r') as zip_ref:
            # Get list of files in ZIP
            file_list = zip_ref.namelist()
            
            # Extract all files
            for file in file_list:
                zip_ref.extract(file, data_dir)
            
            # Move extracted files to dataset folder
            for file in file_list:
                src_path = os.path.join(data_dir, file)
                
                # Only process if it's a file (not directory)
                if os.path.exists(src_path) and os.path.isfile(src_path):
                    dest_path = os.path.join(dataset_dir, os.path.basename(file))
                    shutil.move(src_path, dest_path)
            
            # Clean up any empty directories created by extraction
            for file in file_list:
                src_path = os.path.join(data_dir, file)
                if os.path.exists(src_path) and os.path.isdir(src_path):
                    try:
                        os.rmdir(src_path)
                    except:
                        pass  # Directory not empty, skip
        
        print(f"✓ Extracted {dataset_name} successfully")
        return True
        
    except zipfile.BadZipFile:
        print(f"✗ {dataset_name}: Invalid ZIP file")
        return False
    except Exception as e:
        print(f"✗ Error extracting {dataset_name}: {e}")
        return False

def cleanup_zip_file(zip_path):
    """
    Remove the downloaded ZIP file after extraction
    """
    try:
        if os.path.exists(zip_path):
            os.remove(zip_path)
            print(f"  Cleaned up ZIP file: {os.path.basename(zip_path)}")
    except Exception as e:
        print(f"  Warning: Could not remove ZIP file {zip_path}: {e}")

def download_and_extract_dataset(dataset_name, dataset_info, data_dir, force_redownload=False):
    """
    Download and extract a single GLUE dataset
    """
    print(f"\n{'='*60}")
    print(f"Processing: {dataset_name}")
    print(f"Description: {dataset_info['description']}")
    print(f"{'='*60}")
    
    # Create dataset directory
    dataset_dir = os.path.join(data_dir, dataset_name)
    
    # Check if dataset already exists
    if os.path.exists(dataset_dir) and not force_redownload:
        # Check if directory has any .tsv files
        tsv_files = [f for f in os.listdir(dataset_dir) if f.endswith('.tsv')]
        if tsv_files:
            print(f"✓ {dataset_name} already exists with {len(tsv_files)} data files")
            return True
    
    # Download the dataset
    url = dataset_info['url']
    zip_filename = url.split('/')[-1]
    zip_path = os.path.join(data_dir, zip_filename)
    
    # Download the file
    if not download_file(url, zip_path, dataset_name):
        return False
    
    # Extract the file
    if not extract_to_dataset_folder(zip_path, data_dir, dataset_name):
        return False
    
    # Clean up ZIP file
    cleanup_zip_file(zip_path)
    
    # Verify extraction
    if os.path.exists(dataset_dir):
        files = os.listdir(dataset_dir)
        tsv_files = [f for f in files if f.endswith('.tsv')]
        txt_files = [f for f in files if f.endswith('.txt')]
        
        print(f"✓ {dataset_name} extraction complete")
        print(f"  Files in {dataset_name}/: {len(files)} total")
        if tsv_files:
            print(f"  TSV files: {', '.join(tsv_files[:3])}{'...' if len(tsv_files) > 3 else ''}")
        if txt_files:
            print(f"  TXT files: {', '.join(txt_files[:3])}{'...' if len(txt_files) > 3 else ''}")
        
        return True
    
    return False

def check_internet_connection():
    """
    Check if internet connection is available
    """
    try:
        urllib.request.urlopen('https://www.bing.com', timeout=10)
        return True
    except:
        return False

def main():
    parser = argparse.ArgumentParser(
        description='Download all GLUE benchmark datasets',
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Examples:
  python download_glue_data.py                     # Download all to ./glue_data
  python download_glue_data.py --data_dir my_data  # Download to custom directory
  python download_glue_data.py --force             # Force redownload
  python download_glue_data.py --datasets CoLA,SST-2,MRPC  # Download specific datasets
        """
    )
    
    parser.add_argument('--data_dir', type=str, default='./glue_data',
                       help='Directory to store downloaded data (default: ./glue_data)')
    parser.add_argument('--datasets', type=str, default='all',
                       help='Comma-separated list of datasets to download (default: all)')
    parser.add_argument('--force', action='store_true',
                       help='Force redownload even if dataset exists')
    parser.add_argument('--list', action='store_true',
                       help='List all available datasets')
    
    args = parser.parse_args()
    
    # List available datasets
    if args.list:
        print("Available GLUE datasets:")
        print("-" * 50)
        for i, (name, info) in enumerate(GLUE_DATASETS.items(), 1):
            print(f"{i:2d}. {name:12s} - {info['description']}")
        print("-" * 50)
        print(f"Total: {len(GLUE_DATASETS)} datasets")
        return
    
    # Check internet connection
    print("Checking internet connection...")
    if not check_internet_connection():
        print("✗ No internet connection. Please check your network settings.")
        sys.exit(1)
    print("✓ Internet connection available")
    
    # Create data directory
    data_dir = Path(args.data_dir)
    data_dir.mkdir(parents=True, exist_ok=True)
    print(f"\nData directory: {data_dir.absolute()}")
    
    # Determine which datasets to download
    if args.datasets.lower() == 'all':
        # Exclude 'diagnostic' as it's the same as AX
        datasets_to_download = {k: v for k, v in GLUE_DATASETS.items() if k != 'diagnostic'}
    else:
        datasets_to_download = {}
        requested_datasets = [d.strip() for d in args.datasets.split(',')]
        
        for dataset_name in requested_datasets:
            if dataset_name in GLUE_DATASETS:
                datasets_to_download[dataset_name] = GLUE_DATASETS[dataset_name]
            else:
                print(f"✗ Unknown dataset: {dataset_name}")
                print(f"Available datasets: {', '.join(GLUE_DATASETS.keys())}")
                sys.exit(1)
    
    print(f"Downloading {len(datasets_to_download)} dataset(s)")
    print("-" * 60)
    
    # Download each dataset
    success_count = 0
    total_count = len(datasets_to_download)
    
    for i, (dataset_name, dataset_info) in enumerate(datasets_to_download.items(), 1):
        print(f"\nDataset {i}/{total_count}:")
        if download_and_extract_dataset(dataset_name, dataset_info, data_dir, args.force):
            success_count += 1
    
    # Print summary
    print(f"\n{'='*60}")
    print("DOWNLOAD SUMMARY")
    print(f"{'='*60}")
    print(f"Successfully processed: {success_count}/{total_count} datasets")
    print(f"Data location: {data_dir.absolute()}")
    
    if success_count == total_count:
        print("\n✓ All datasets downloaded and extracted successfully!")
        
        # Show directory structure
        print(f"\nDirectory structure:")
        print(f"{data_dir.name}/")
        for dataset in sorted(datasets_to_download.keys()):
            dataset_path = data_dir / dataset
            if dataset_path.exists():
                files = list(dataset_path.glob("*"))
                tsv_count = len([f for f in files if f.suffix == '.tsv'])
                print(f"  ├── {dataset}/ ({len(files)} files, {tsv_count} .tsv)")
        
        print(f"\nTo load the data in Python:")
        print(f"  import pandas as pd")
        print(f"  data = pd.read_csv('{data_dir}/SST-2/train.tsv', sep='\\t')")
    else:
        print(f"\n⚠ Some datasets failed to download.")
        print(f"  Successful: {success_count}")
        print(f"  Failed: {total_count - success_count}")
    
    print(f"\nDone!")

if __name__ == "__main__":
    main()
////////////////////////////////////////////////////////////
