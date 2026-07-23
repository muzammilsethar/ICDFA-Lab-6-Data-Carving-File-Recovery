# 📁 ICDFA Lab 6: Data Carving & File Recovery Walkthrough

## 📝 Executive Overview
This laboratory report documents the process of digital forensics analysis, file system structure examination, deleted file recovery, and automated data carving in a Kali Linux environment. The analysis utilizes **The Sleuth Kit (TSK)**, **Scalpel**, **xxd**, and native CLI utilities to process raw disk images (`.dd`, `.001`), reconstruct deleted database records, and extract embedded XMP/EXIF metadata.

---

## 🛠️ Environment & Forensic Toolset

* **Operating System:** Kali Linux (`kalimomi@kali`)
* **Working Directory:** `~/CIP-B102_Lab6_Data_Carving`
* **Forensic Software & Packages:**
  * **Sleuth Kit (TSK):** `img_stat`, `mmls`, `fsstat`, `fls`, `icat`, `blkcat`
  * **Data Carving:** `scalpel`, `7z` (p7zip-full)
  * **Binary & Utilities:** `xxd`, `file`, `strings`, `md5sum`, `sha256sum`, `imagemagick`

---

## 🚀 Step-by-Step Execution & Technical Findings

### Step 1: Environment Setup & Artifact Verification

#### Executed Commands:
bash
# Create working directory
mkdir -p ~/CIP-B102_Lab6_Data_Carving
cd ~/CIP-B102_Lab6_Data_Carving

# Download lab evidence artifacts
wget [https://www.dropbox.com/s/nw23q14vzs3ykup/Ch01InChap01.dd](https://www.dropbox.com/s/nw23q14vzs3ykup/Ch01InChap01.dd) -O Ch01InChap01.dd
wget [https://raw.githubusercontent.com/frankwxu/digital-forensics-lab/main/Basic_Computer_Skills_for_Forensics/file_carving/usb_file/J_ub_law.jpg](https://raw.githubusercontent.com/frankwxu/digital-forensics-lab/main/Basic_Computer_Skills_for_Forensics/file_carving/usb_file/J_ub_law.jpg) -O J_ub_law.jpg
wget [https://raw.githubusercontent.com/frankwxu/digital-forensics-lab/main/Basic_Computer_Skills_for_Forensics/file_carving/usb_image/120M.7z](https://raw.githubusercontent.com/frankwxu/digital-forensics-lab/main/Basic_Computer_Skills_for_Forensics/file_carving/usb_image/120M.7z) -O 120M.7z

# Verify evidence integrity
md5sum Ch01InChap01.dd J_ub_law.jpg 120M.7z
sha256sum Ch01InChap01.dd J_ub_law.jpg 120M.7z

Technical Data Output & Findings:
Ch01InChap01.dd Cryptographic Hashes:

MD5: a11773bcf1fc088ec0ab8e0a349ffbcb

SHA-256: 3ce8053e4f3d9c8ab98b3aadb2480685efb4e4980d34297b83bd5a09b1a7b122

Step 2: File System Analysis & Inode Extraction (Ch01InChap01.dd)

Executed Commands:
# Analyze image file system format
img_stat Ch01InChap01.dd
fsstat Ch01InChap01.dd

# Recursively list allocated & deleted directory entries
fls -r -o 0 Ch01InChap01.dd

# Extract allocated Microsoft Access database using Inode 5
icat -o 0 Ch01InChap01.dd 5 > extracted_client_info.mdb
file extracted_client_info.mdb

# Inspect raw sector cluster (Sector 128)
blkcat Ch01InChap01.dd 128 1 > recovered_sector.bin
xxd recovered_sector.bin | head -n 10

Technical Data Output & Findings:
Image Type & File System: Raw (dd), FAT12 (Cluster Size: 512 bytes).

Recovered Database: Inode 5 ➔ Client Info.mdb (Microsoft Access Database File).

Identified Deleted Artifacts (Marked with *):

Inode 8: Billing Letter.doc

Inode 11: confirmation.txt

Inode 15: letter1.txt

Inode 17: Regrets.doc

Raw Cluster Data: Extracted database schema tables (MSysRelationships, MSysQueries, mmaryInfo).

Step 3: Hex Stream Reconstruction & Metadata Extraction (J_ub_law.jpg)

Executed Commands:
# Convert binary stream to plain hex dump
xxd -p J_ub_law.jpg > J_ub_law_hexdump.txt

# Reconstruct image binary from plain hex dump
xxd -r -p J_ub_law_hexdump.txt J_ub_law_reconstructed.jpg

# Compare MD5 hashes to prove zero-data loss reconstruction
md5sum J_ub_law.jpg J_ub_law_reconstructed.jpg

# Extract embedded EXIF and XMP metadata strings
strings J_ub_law.jpg | grep -Ei "nikon|photoshop|adobe|date|copyright|author"

Technical Data Output & Findings:
MD5 Verification: Both original and reconstructed image hashes matched (83a360ac7f7e0ca318e5bfe39f95f137).

Camera Model: NIKON D4

Editing Software: Adobe Photoshop CS6 (Macintosh) & Camera Raw 8.3

Timestamps: Creation Date: 2013-10-08, Modify Date: 2020-08-20

Author / Institution: Merrick School of Business / HOWARD KRUP

Step 4: Automated Data Carving (120M.7z & usb_fat_carving.001)

Executed Commands:
# Decompress archive
7z e 120M.7z

# Analyze partition structure
mmls usb_fat_carving.001

# Configure Scalpel carver rules in /etc/scalpel/scalpel.conf
# (Uncommented JPG and DOC header/footer signatures)

# Execute automated carving run
scalpel usb_fat_carving.001 -o output_usb_carving

# Review carving audit log
cat output_usb_carving/audit.txt | head -n 30

Technical Data Output & Findings:
Partition Layout: Win95 FAT16 filesystem starting at Sector Offset 128.

Total Carved Artifacts: 23 files

Images (.jpg): 17 files

Documents (.doc): 6 files

Carved Directories: Organized into doc-2-0/, doc-3-0/, jpg-0-0/, and jpg-1-0/.


📊 Complete Forensic Summary Matrix
Target Artifact	         Forensic Technique	      Tool Used	          Result / Recovered Data
Ch01InChap01.dd	         Direct Inode Recovery	   icat	              Recovered intact Client Info.mdb database file.
Ch01InChap01.dd          Directory Traversal       fls                 Located 4 deleted document references (.doc, .txt).
J_ub_law.jpg             Binary Hex Manipulation   xxd                 Reconstructed image binary with 100% hash parity.
J_ub_law.jpg             Metadata Extraction       strings             Extracted NIKON D4 & Howard Krup EXIF data.
usb_fat_carving.001      Signature Carving         scalpel             Recovered 23 lost files (17 JPG, 6 DOC).

Maintained for ICDFA Training & Portfolio Submission.
