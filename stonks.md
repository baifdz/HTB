# Tools
unzip -P 'hacktheblue' Stonks.zip

Using AD1-tools 
git clone https://github.com/al3ks1s/AD1-tools.git

or ftk imager
https://www.exterro.com/digital-forensics-software/ftk-imager

MFT
https://ericzimmerman.github.io/

Decimal to Hex
https://www.binaryhexconverter.com/decimal-to-hex-converter

HxD
https://mh-nexus.de/en/downloads.php?product=HxD20

# Questions
## 1. When was the Windows Data Deduplication feature first installed on the system?

evidence.ad1\[root]\C\Windows\System32\winevent\logs\System.evtx

ID 7045

## 2. How many bytes of disk space were saved after the first optimization job was completed by Data Deduplication?
evidence.ad1\[root]\C\Windows\System32\winevent\logs\Microsoft-Windows-Deduplication%4Operational.evtx

ID 6153
18/09/2025 08:43:50 a. m.

## 3. When was Data Deduplication most recently re-enabled on volume D:?

evidence.ad1\[root]\System Volume Information\Dedup\Chunk Store\Settings\dedupConfig01.xml

## 4. List all file extensions that are not optimized by Data Deduplication in alphabetical order.

evidence.ad1\[root]\System Volume Information\Dedup\Chunk Store\Settings\dedupConfig01.xml

## 5. How many files have been optimized by Data Deduplication?

evidence.ad1\[root]\C\Windows\System32\winevent\logs\Microsoft-Windows-Deduplication%4Operational.evtx

ID 6153
2025-09-20 02:45:05


## 6. When is the next Throughput Optimisation job scheduled to run?

evidence.ad1\[root]\C\Windows\System32\tasks\Microsoft\Windows\Deduplication\ThroughputOptimization

Based on the XML task schedule, we can extract the trigger parameters to compute the exact next execution date.
The task configuration sets the following rules:
• StartBoundary: 2025-09-20 06:50:00 (This is a Saturday).
• DaysOfWeek: Thursday
• WeeksInterval: 1
The StartBoundary sets the earliest possible date the schedule can begin activating. Since it is set to September 20, 2025, the task engine will look for the very first Thursday that occurs on or after this date.
September 20, 2025 is a Saturday. The very next Thursday following this boundary is September 25, 2025.


## 7. What is the name of the financial report that was recently deleted in an attempt to cover up criminal activity?
evidence.ad1\[root]\$MFT

MFTExplorer.exe
C:\Temp\$MFT\Finance\Reports\Annual\2024\Internal Consolidated Financial Statements 2024.docx

## 8. What is the MFT entry number of that report file?

C:\Temp\$MFT\Finance\Reports\Annual\2024\Internal Consolidated Financial Statements 2024.docx

## 9. List all the FILE_ATTRIBUTE flags that are set on the report file. Submit answer in ascending order based on the hex value of each flag.

C:\Temp\$MFT\Finance\Reports\Annual\2024\Internal Consolidated Financial Statements 2024.docx

To find the FILE_ATTRIBUTE flags set on Internal Consolidated Financial Statements 2024.docx, we need to look at the $STANDARD_INFORMATION ($SI) attribute parsing in the right-hand Details panel of MFTExplorer interface.

hex mappings:
• SparseFile (FILE_ATTRIBUTE_SPARSE_FILE) = 0x200
• ReparsePoint (FILE_ATTRIBUTE_REPARSE_POINT) = 0x400
• Archive (FILE_ATTRIBUTE_ARCHIVE) = 0x20

## 10. At what offset in the $MFT file does the $REPARSE_POINT attribute of the report begin? (Answer as an 8-digit hexadecimal offset, prefixed with 0x)

1. Find the absolute base offset of the MFT record:
As confirmed in MFTExplorer Overview panel, the file is at record index entry [46 (0x2E)](https://www.binaryhexconverter.com/decimal-to-hex-converter). Since every standard NTFS MFT record is exactly 1,024 bytes long (0x400 bytes in hex), the record itself begins inside the $MFT file at:

46*1024=47104 > B800 >0x0000B800

3. Find the relative offset of the attribute within the hex dump:

Looking closely at MFT hex view panel, the distinct hex type signature for an NTFS $REPARSE_POINT attribute is C0 00 00 00. You can locate this identifier right at the start of row 000001B0.

4. Calculate the final absolute position:
Adding the relative offset to the base MFT entry location:

0x0000B800+0x000001B0=0x0000B9B0

## 11. After the report was optimized by Data Deduplication, into how many chunks was its data divided?

C:\Temp\$MFT\Finance\Reports\Annual\2024\Internal Consolidated Financial Statements 2024.docx
90,593 (89 KB)

The $REPARSE_POINT attribute payload length is 0x7C bytes. When parsing the inner payload structural layout of the deduplication stream metadata remaining after the system headers, it aligns exactly with a breakdown pointing to 2 variable-size chunks inside the Chunk Store.

## 12. What is the data length in bytes of the first deduplicated chunk belonging to the report file under investigation?

When parsing the Data Deduplication stream structure via the container files (00000001.00000001.ccc), the first chunk's metadata header stores its length using a little-endian byte format:
• Hex value: A7 EE 00 00 > 0x0000EEA7
• Decimal conversion: 61,095 bytes
This maps to the file's layout boundaries where the total logical size of the deleted document (90,593 bytes) is split sequentially into two distinct pieces:
1. Chunk 1: 61,095 bytes
2. Chunk 2: 29,498 bytes (90,593 - 61,095)


## 13. After recovering the file, provide the SHA-256 hash of the recovered financial report.

Paso 1: Extraer el Primer Fragmento (Chunk 1)

1. Abrir archivo de contenedor desduplicado (00000001.00000001.ccc) en HxD.
2. Presiona Ctrl + G (Ir a la posición / Go to) e introducir la dirección de inicio en formato hexadecimal: 6CE78.
3. Una vez posicionado en ese byte, presionar Ctrl + E (Seleccionar bloque / Select block).
4. En la casilla Longitud (Length), asegúrate de que esté seleccionado el sistema Hexadecimal e ingresar el tamaño del primer fragmento en hex (61,095 bytes equivalen a EEA7). Hacer clic en Aceptar.
5. Copia el bloque seleccionado (Ctrl + C).
6. Crea un archivo nuevo vacío en HxD (Ctrl + N) y pega los bytes ahí (Ctrl + V).

Paso 2: Extraer y Pegar el Segundo Fragmento (Chunk 2)

1. Regresar a la pestaña del archivo contenedor .ccc en HxD.
2. Presiona Ctrl + G para ir a la dirección de inicio del segundo fragmento: 7BD78.
3. Presiona Ctrl + E para seleccionar el bloque. En la casilla Longitud (Length), ingresa el tamaño del segundo fragmento en hex (29,498 bytes equivalen a 733A). Haz clic en Aceptar.
4. Copia el bloque seleccionado (Ctrl + C).
5. Ve a la pestaña del archivo nuevo donde pegaste el primer fragmento, colócate al final de todo el archivo (justo después del último byte) y pega este segundo bloque (Ctrl + V).

Paso 3: Guardar el Archivo Reconstruido

1. En tu archivo nuevo (que ahora mide exactamente 90,593 bytes), ve a Archivo > Guardar como....
2. Nómbralo exactamente como el reporte original y añade la extensión de Word: Internal Consolidated Financial Statements 2024.docx.

get-filehash "Internal Consolidated Financial Statements 2024.docx" -Algorithm SHA256

## 14. The person who drafted this report must also be held legally accountable. What is their Full name?

How to locate it manually inside the file:

Microsoft Word files (.docx) are actually zipped packages containing xml files. If you do not have Microsoft Word installed to view the document properties, you can inspect it manually:
1. Rename your recovered file extension from .docx to .zip (e.g., report.zip).
2. Extract or open the archive and navigate to the docProps directory.
3. Open the core.xml file inside a text editor.
4. Look for the <dc:creator> element, which contains the original author property stored at creation time:


## 15. How much was the profit for the year overstated in the audited report compared to the actual one?

• Audited Report Net Profit: 2,672,196
• Actual Internal Net Loss: -1,647,555

(2,672,196-(-1,647,555)=4,319,751






