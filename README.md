# CSV-Domain-Counter
This Node.js program reads a large CSV file (~1M rows) containing user emails, counts how many users belong to each email domain, and writes the results to a JSON file without loading the entire CSV into memory. It uses streams, readline, and backpressure for memory efficiency.

1️⃣ Prerequisites
Node.js v14+

ES modules support ("type": "module" in package.json)
{
  "type": "module",
  "scripts": {
    "start": "node code8.js"
  }
}
2️⃣ Project Structure
project/
│
├─ code8.js           # Main CSV processing script
├─ data/
│   └─ users.csv      # CSV file with user emails
├─ out/               # Output folder (will be created automatically)
└─ package.json
3️⃣ code8.js Explained
import fs from "fs";
import readline from "readline";
import path from "path";
Import built-in Node.js modules:

fs → file system operations
readline → read file line by line
path → handle file paths
const inputFile = "data/users.csv";
const outputFile = "out/domains.json";
Define input CSV path and output JSON path.

async function main() {
 fs.mkdirSync(path.dirname(outputFile), { recursive: true });
Create the out folder if it doesn't exist, using recursive: true to create nested directories.
 if (!fs.existsSync(inputFile)) {
  throw new Error(`❌ Input file not found: ${inputFile}`);
 }
Check if the input CSV exists; throw an error if not.
 const domainCounts = {};
 let lineCount = 0;
 const startTime = Date.now();
domainCounts → stores email domain counts.

lineCount → tracks how many lines processed.
startTime → to calculate processing duration.
 console.log("📊 Processing started...");
Log start message.
 const rl = readline.createInterface({
  input: fs.createReadStream(inputFile),
  crlfDelay: Infinity,
 });
Use readline with a stream to read the CSV line by line efficiently.
crlfDelay: Infinity ensures proper handling of \r\n (Windows line endings).
 for await (const line of rl) {
  if (lineCount === 0 && line.startsWith("id")) {
   lineCount++;
   continue;
  }
Skip the header line (id,email).

for await allows asynchronous iteration over each line in the file.
  const [, email] = line.split(",");
  if (!email || !email.includes("@")) continue;
Split the line by comma to extract the email column.

Skip invalid or empty emails.
  const domain = email.split("@")[1].trim();
  domainCounts[domain] = (domainCounts[domain] || 0) + 1;
Extract the domain from the email (after @).

Increment its count in domainCounts.
  lineCount++;
  if (lineCount % 1000 === 0) {
   process.stdout.write(`\r🧮 Processed: ${lineCount.toLocaleString()} lines`);
  }
 }
Increment line count.

Log progress every 1000 lines in-place using \r to overwrite the previous log.
 console.log(`\r✅ Finished processing ${lineCount.toLocaleString()} lines.`);
 console.log("💾 Writing results to", outputFile, "...");
Log completion of processing.

 fs.writeFileSync(outputFile, JSON.stringify(domainCounts, null, 2));
 const duration = ((Date.now() - startTime) / 1000).toFixed(2);
 console.log(`🎉 Done! (${duration}s total) Results in ${outputFile}`);
Write domainCounts to JSON file in a human-readable format (2-space indentation).

Log total duration and location of results.

main().catch((err) => console.error(err.message));
Run main() and catch any errors to display in the console.

4️⃣ Example Input (users.csv)
id,email
1,asmi@gmail.com
2,riya@yahoo.com
3,aman@gmail.com
4,meena@outlook.com
5,riya@gmail.com
5️⃣ Example Output (domains.json)
{
  "gmail.com": 3,
  "yahoo.com": 1,
  "outlook.com": 1
}
6️⃣ How to Run
npm run start
Progress will be shown in console.

JSON file is saved in out/domains.json.

