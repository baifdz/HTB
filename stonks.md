# Tools
unzip -P 'hacktheblue' Stonks.zip

Using AD1-tools 
git clone https://github.com/al3ks1s/AD1-tools.git

or ftk imager
https://www.exterro.com/digital-forensics-software/ftk-imager

# Questions
1. When was the Windows Data Deduplication feature first installed on the system?

evidence.ad1\[root]\C\Windows\System32\winevent\logs\System.evtx

ID 7045

2. How many bytes of disk space were saved after the first optimization job was completed by Data Deduplication?
evidence.ad1\[root]\C\Windows\System32\winevent\logs\Microsoft-Windows-Deduplication%4Operational.evtx

ID 6153
18/09/2025 08:43:50 a. m.

3. When was Data Deduplication most recently re-enabled on volume D:?

evidence.ad1\[root]\System Volume Information\Dedup\Chunk Store\Settings\dedupConfig01.xml

4. List all file extensions that are not optimized by Data Deduplication in alphabetical order.

evidence.ad1\[root]\System Volume Information\Dedup\Chunk Store\Settings\dedupConfig01.xml

5. How many files have been optimized by Data Deduplication?

evidence.ad1\[root]\C\Windows\System32\winevent\logs\Microsoft-Windows-Deduplication%4Operational.evtx

ID 6153
2025-09-20 02:45:05


6. When is the next Throughput Optimisation job scheduled to run?

evidence.ad1\[root]\C\Windows\System32\tasks\Microsoft\Windows\Deduplication\ThroughputOptimization

Based on the XML task schedule, we can extract the trigger parameters to compute the exact next execution date.
The task configuration sets the following rules:
• StartBoundary: 2025-09-20 06:50:00 (This is a Saturday).
• DaysOfWeek: Thursday
• WeeksInterval: 1
The StartBoundary sets the earliest possible date the schedule can begin activating. Since it is set to September 20, 2025, the task engine will look for the very first Thursday that occurs on or after this date.
September 20, 2025 is a Saturday. The very next Thursday following this boundary is September 25, 2025.


7. What is the name of the financial report that was recently deleted in an attempt to cover up criminal activity?











