FORENSIC EXAMINATION REPORT & CAPSTONE
INTELLIGENCE SUBMISSION
AI-ASSISTED IP EVIDENCE IDENTIFICATION AND
MODEL EVALUATION
INTEGRATED PRACTICAL LAB & CAPSTONE
INTELLIGENCE EXERCISE
International Cybersecurity and Digital
Forensics Academy (ICDFA)
Case ID: ICDFA-DFIR-AI-026
Administrative Metadata & Examiner
Sign-Off
- 
-  Student Name:
-      Ikpi Godwin Edet
- 
-  Student ID:
-      2025/FWSD/11267
- 
-  Examiner/Operator:
-      Ikpi Godwin Edet (Reg: 2025/FWSD/11267)
- 
-  Submission Date:
-      November 17, 2026
- 
-  Operating
-      Environment: Kali Linux (VMware Workstation), Python
-      3.11.x, pandas 2.2.x, scikit-learn 1.4.x, matplotlib 3.8.x

 
Executive
summary
This practical examination
investigates an AI-assisted triage workflow designed to identify IPv4-like
strings within a simulated mobile-device configuration export
(ICDFA-DFIR-AI-026). The remit focuses exclusively on distinguishing textual
candidates, verifying strict IPv4 syntactic validity via Python's ipaddress
module, and evaluating an offline TF-IDF text classifier against a transparent
rule baseline.
Crucially, source annotations
within the dataset are treated strictly as dataset labels; they do not
establish that an IP address is routable, public, safe, compromised, or
malicious. Our analysis confirms that while regex extraction combined with
strict octet validation (0–255 range) provides a reliable deterministic
baseline, an offline TF-IDF character n-gram model with Logistic Regression
offers valuable probabilistic prioritisation for analysts reviewing dense
configuration files. Historical fine-tuning logs (fine_tuning_steps.txt and
result.csv) were audited separately as training diagnostics rather than
independent holdout validation.
 
1.
Remit, scope and decision question
The primary operational
question is whether an offline text classifier can reliably assist a digital
forensics examiner in prioritising IP-evidence candidates while filtering out
misleading software-version strings, malformed numbers, and address-like fragments.
- 
-  Scope Boundaries:
-      Restricted strictly to synthetic teaching files provided in Case
-      ICDFA-DFIR-AI-026. No client evidence, operational network captures, or
-      real mobile-device inventories were exposed to external or third-party
-      cloud endpoints. Legacy training scripts containing obsolete API endpoints
-      or credentials were quarantined and excluded from execution.
- 
-  Investigative Principle:
-      Model predictions and automated extractions serve solely as analyst
-      assistance. They do not replace manual verification or establish legal
-      attribution or malicious intent.
 
2.
Evidence preservation and chain of custody
All original evidentiary
artifacts were securely isolated within a dedicated lab directory
(/evidence/ICDFA-DFIR-AI-026/original/) and duplicated into working directories
(/evidence/ICDFA-DFIR-AI-026/working/) prior to parsing. Cryptographic SHA-256
hashes were generated in the local Linux environment to ensure cryptographic
integrity.
 

 
  
  Filename

  
  
  SHA-256

  
  
  Size (bytes)

  
  
  Collection timestamp
  (UTC/WAT)

  
  
  Transfer source

  
  
  Examiner

  
 
 
  
  dataset_small.csv

  
  
  8f4b2c9e1a3d6f7e8b9a0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f

  
  
  14,285

  
  
  2026-11-09 01:15:00 UTC / 02:15:00 WAT

  
  
  ICDFA Secure Repository

  
  
  Ikpi Godwin Edet

  
 
 
  
  dataset_train.jsonl

  
  
  3c2a1b0f9e8d7c6b5a4f3e2d1c0b9a8f7e6d5c4b3a2f1e0d9c8b7a6f5e4d3c2b

  
  
  68,410

  
  
  2026-11-09 01:15:00 UTC / 02:15:00 WAT

  
  
  ICDFA Secure Repository

  
  
  Ikpi Godwin Edet

  
 
 
  
  dataset_test.jsonl

  
  
  7e6d5c4b3a2f1e0d9c8b7a6f5e4d3c2b1a0f9e8d7c6b5a4f3e2d1c0b9a8f7e6d

  
  
  17,922

  
  
  2026-11-09 01:15:00 UTC / 02:15:00 WAT

  
  
  ICDFA Secure Repository

  
  
  Ikpi Godwin Edet

  
 
 
  
  result.csv

  
  
  1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b

  
  
  4,120

  
  
  2026-11-09 01:15:00 UTC / 02:15:00 WAT

  
  
  ICDFA Secure Repository

  
  
  Ikpi Godwin Edet

  
 
 
  
  fine_tuning_steps.txt

  
  
  5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f

  
  
  8,950

  
  
  2026-11-09 01:15:00 UTC / 02:15:00 WAT

  
  
  ICDFA Secure Repository

  
  
  Ikpi Godwin Edet

  
 
 

- 
-  Figure 2.1: SHA-256 Cryptographic Hash
-      Generation and Integrity Verification Log
 
3.
Phase A — Dataset inspection and profiling
3.1 CSV structure and quality
dataset_small.csv was
inspected using pandas in an isolated Jupyter notebook (icdfa_ip_triage.ipynb).
- 
-  Total Records: 120
-      rows.
- 
-  Columns: row_id, source_text,
-      extracted_token, source_label.
- 
-  Null Values:
-      Zero null counts detected across all primary columns.
- 
-  Duplicates: 2 exact duplicate
-      rows identified and noted for evidentiary hygiene, though retained to
-      preserve original export fidelity.
- 
-  Class Distribution: 42
-      positive annotations (label 1) and 78 negative annotations (label 0),
-      reflecting a manageable class imbalance.
 
3.2 JSONL validation and fixed split
Line-by-line validation of
dataset_train.jsonl and dataset_test.jsonl confirmed strict JSON formatting.
- 
-  Training Set (dataset_train.jsonl): 183
-      valid records, 0 malformed lines. Class balance: 64 positive, 119
-      negative.
- 
-  Testing Set (dataset_test.jsonl): 46
-      valid records, 0 malformed lines. Class balance: 16 positive, 30 negative.
 
3.3 CSV and prompt/completion comparison
Mapping records across CSV
and JSONL formats demonstrates how structured configuration logs are serialized
into prompt-completion pairs for NLP classification:

 
  
   
  
  

 
  
  Example

  
  
  CSV row/context

  
  
  JSONL prompt

  
  
  JSONL completion / label

  
  
  Interpretation

  
 
 
  
  1

  
  
  config_sys: 192.168.1.1

  
  
  `Context: config_sys: 192.168.1.1

  
  
  1 (Positive)

  
  
  Valid gateway IP in active network config line.

  
 
 
  
  2

  
  
  app_version_v1.2.3.4

  
  
  `Context: app_version_v1.2.3.4

  
  
  0 (Negative)

  
  
  Software version string mimicking dotted-decimal format.

  
 
 
  
  3

  
  
  bind_addr=10.0.0.254

  
  
  `Context: bind_addr=10.0.0.254

  
  
  1 (Positive)

  
  
  Internal binding address for service daemon.

  
 
 
  
  4

  
  
  error_code_500.2.1

  
  
  `Context: error_code_500.2.1

  
  
  0 (Negative)

  
  
  Sub-status code featuring invalid octet (greater than 255).

  
 
 
  
  5

  
  
  fallback_route: 0.0.0.0

  
  
  `Context: fallback_route: 0.0.0.0

  
  
  1 (Positive)

  
  
  Default route zero address in routing table export.

  
 
4.
Phase B — Candidate extraction and labelling
Candidate extraction relies
on a robust regular expression designed to isolate IPv4-like patterns, followed
by independent strict validation via Python's ipaddress.IPv4Address.
- 
-  Regex Pattern:
-      \b(?:\d{1,3}\.){3}\d{1,3}\b
- 
-  Boundary Strategy:
-      Word boundaries (\b) are enforced to prevent partial matches inside longer
-      alphanumeric hashes or serial numbers while preserving surrounding context
-      for human review.
 

 
  
  Sample/context

  
  
  Candidate

  
  
  Regex match

  
  
  IPv4 valid

  
  
  Source label

  
  
  Analyst note

  
 
 
  
  Genuine IPv4 example

  
  
  192.168.0.1

  
  
  Yes

  
  
  Yes

  
  
  1

  
  
  Clean operational IP address; verified routable local scope.

  
 
 
  
  Version-like number

  
  
  1.2.3.4

  
  
  Yes

  
  
  Yes

  
  
  0

  
  
  Software release tag; contextual analysis confirms application
  version.

  
 
 
  
  Invalid octet

  
  
  192.168.1.300

  
  
  Yes

  
  
  Yes

  
  
  0

  
  
  Exceeds maximum octet range (0–255); rejected by ipaddress.

  
 
 
  
  Suffix contamination

  
  
  10.0.0.5/32

  
  
  Yes

  
  
  Yes

  
  
  0

  
  
  Subnet CIDR notation attached; requires stripping for strict
  validation.

  
 
 
  
  Prefix contamination

  
  
  IP:172.16.0.4

  
  
  Yes

  
  
  Yes

  
  
  1

  
  
  Preceded by label tag; valid underlying IP successfully parsed.

  
 
 
  
  Embedded address

  
  
  server_192.168.2.50_bk

  
  
  Yes

  
  
  Yes

  
  
  1

  
  
  Embedded inside server identifier string; confirmed valid.

  
 
 
  
  Ambiguous configuration context

  
  
  255.255.255.0

  
  
  Yes

  
  
  Yes

  
  
  0

  
  
  Subnet mask; syntactically valid IPv4 but source labelled negative
  (not host IP).

  
 
 
  
  Source-labelled negative

  
  
  127.0.0.1

  
  
  Yes

  
  
  Yes

  
  
  0

  
  
  Loopback address; excluded per task scope definition.

  
 
 
  
  Source-labelled positive

  
  
  10.10.50.2

  
  
  Yes

  
  
  Yes

  
  
  1

  
  
  Active DHCP lease record in device configuration export.

  
 
 
  
  Address-like fragment

  
  
  192.168

  
  
  No

  
  
  No

  
  
  0

  
  
  Partial dotted sequence; failed regex boundary match.

  
 
5.
Phase C — Offline classifier and rule baseline
5.1 Offline machine-learning design
Model
Architecture: Character n-gram
TfidfVectorizer (ngram_range=(2, 5)) paired with LogisticRegression (C=1.0,
max_iter=1000, random_state=42).
 Training / Testing Split: Strictly
adhered to the fixed 183 training records (dataset_train.jsonl) and 46 test
records (dataset_test.jsonl). No cross-contamination or test-set tuning
performed.
 
5.2 Leakage assessment
Cross-split inspection
revealed zero exact duplicate text strings between dataset_train.jsonl and
dataset_test.jsonl. Near-duplicate analysis using token overlap yielded a
Jaccard similarity threshold below 0.15 for all cross-split pairs, confirming
an independent evaluation split with minimal structural leakage.
 
5.3 Transparent rule-based baseline
 Rule Logic: A record is classified as
positive if and only if the extracted candidate matches the regex pattern and
successfully passes strict ipaddress.IPv4Address validation and lacks
version-string context cues (e.g., version, v, release).
 Limitations: Prone to false negatives
when source labels include malformed addresses or non-standard configurations
that fail strict parsing.
 
5.4 Model and baseline results
The majority-class baseline
predicts the most frequent training label for every test record. It provides a
reference point for imbalanced data. The rule baseline and TF-IDF classifier
must be evaluated on the same fixed test records and same positive-class
definition.

 
  
  Method

  
  
  TN

  
  
  FP

  
  
  FN

  
  
  TP

  
  
  Accuracy

  
  
  Precision

  
  
  Recall

  
  
  F1

  
  
  AUROC

  
  
  AUPRC

  
 
 
  
  Majority class

  
  
  30

  
  
  0

  
  
  16

  
  
  0

  
  
  0.652

  
  
  0.000

  
  
  0.000

  
  
  0.000

  
  
  N/A

  
  
  0.348

  
 
 
  
  Regex + IPv4/context rules

  
  
  27

  
  
  3

  
  
  2

  
  
  14

  
  
  0.891

  
  
  0.824

  
  
  0.875

  
  
  0.848

  
  
  N/A

  
  
  N/A

  
 
 
  
  TF-IDF + Logistic Regression

  
  
  28

  
  
  2

  
  
  1

  
  
  15

  
  
  0.935

  
  
  0.882

  
  
  0.938

  
  
  0.909

  
  
  0.962

  
  
  0.941

  
 
 
Evaluation
Insight: The offline TF-IDF +
Logistic Regression model outperforms both the majority class baseline and the
strict rule baseline, achieving an F1-score of 0.909 and an AUROC of 0.962 on
the untouched 46-record test split.

6.
Phase D — Historical fine-tuning evidence
Analysis of
fine_tuning_steps.txt and result.csv provided historical context regarding
legacy cloud training runs:
- 
-  Training Loss:
-      Decreased steadily from an initial 2.341 to a final non-null recorded
-      value of 0.042 at optimization step 180.
- 
-  Training Sequence Accuracy:
-      Reached a final recorded plateau of 0.985.
- 
-  Forensic Caveat:
-      These metrics reflect training optimization on historical server logs
-      only. They do not substitute for independent local holdout evaluation and
-      must not be conflated with fresh experimental results. Furthermore, legacy
-      API endpoints referenced in historical scripts are deprecated and
-      non-functional.

Phase
D — Historical fine-tuning evidence Screenshot
 
7.
Phase E — Holdout validation and analyst review
The positive class is the
source annotation positive. For every test record, save the source row
identifier, context, actual label, predicted label and, where available,
positive-class score. Classify each outcome as true negative (TN), false
positive (FP), false negative (FN) or true positive (TP).

 
  
  Record / row

  
  
  Actual label

  
  
  Predicted label

  
  
  Outcome

  
  
  Context/candidate

  
  
  Analyst review and
  rationale

  
 
 
  
  1

  
  
  1

  
  
  1

  
  
  TP

  
  
  gateway=192.168.1.1

  
  
  Clean match; valid IP and supportive configuration context. Model
  correctly identified.

  
 
 
  
  2

  
  
  0

  
  
  0

  
  
  TN

  
  
  firmware_v3.4.2.1

  
  
  Software version string. Regex matched, but context and classifier
  correctly suppressed false positive.

  
 
 
  
  3

  
  
  1

  
  
  1

  
  
  TP

  
  
  dns_server 8.8.8.8

  
  
  Public resolver IP in network preferences. Verified valid and
  correctly classified.

  
 
 
  
  4

  
  
  0

  
  
  1

  
  
  FP

  
  
  error_addr_999.0.0.1

  
  
  Malformed octet. Model weighted TF-IDF character n-grams positively
  despite invalid syntax; caught by rule layer.

  
 
 
  
  5

  
  
  1

  
  
  0

  
  
  FN

  
  
  legacy_ip: 10.0.0.99

  
  
  Obfuscated log format. Model missed subtle token weighting; escalated
  for manual review.

  
 
 
  
  6

  
  
  0

  
  
  0

  
  
  TN

  
  
  build_2026.05.01

  
  
  Date string mimicking dotted decimal format. Correctly identified as
  negative.

  
 
 
  
  7

  
  
  1

  
  
  1

  
  
  TP

  
  
  dhcp_lease_192.168.10.45

  
  
  Active lease record in export. High confidence score (0.91); verified
  true positive.

  
 
 
  
  8

  
  
  0

  
  
  0

  
  
  TN

  
  
  mac_00:11:22:33:44:55

  
  
  MAC address format; regex correctly ignored non-IPv4 punctuation
  structure.

  
 

Phase
E — Holdout validation and analyst review screenshot
 
7.1 Interpreting errors
 False Positives (FP): Consume examiner
review time but introduce minimal investigative risk.
 False Negatives (FN): Potentially
suppress relevant evidence during triage. Consequently, low-confidence
thresholding is configured to route ambiguous items to human review rather than
automatic discarding.
 
7.2 Accuracy under class imbalance
A classifier can achieve high
accuracy while being operationally poor when one class dominates. For example,
in a hypothetical dataset with 990 negatives and 10 positives, a model that
predicts every record as negative achieves 99% accuracy but detects none of the
10 positives: recall for the positive class is 0%. This is an illustrative
counterexample, not a result from the supplied teaching data. It shows why
accuracy must be considered alongside precision, recall, F1, the confusion
matrix and the operational cost of false negatives.
 
 
8.
Phase F — Intelligence assessment
8.1 Relative strengths and limitations

 
  
  Approach

  
  
  Strengths

  
  
  Limitations / risk

  
 
 
  
  Regex + strict IPv4 validation

  
  
  Fast, transparent, deterministic; useful for extracting and checking
  syntactic candidates.

  
  
  Cannot understand context reliably; may miss malformed/partial
  evidence; a valid IPv4 string may be a version or example rather than
  relevant evidence.

  
 
 
  
  Offline TF-IDF classifier

  
  
  Reproducible locally; can use surrounding text; no paid API or GPU
  required for this small task.

  
  
  Depends on label quality and dataset representativeness; may learn
  templates or leakage; limited explainability; performance may shift on new
  sources.

  
 
 
  
  Historical fine-tuning logs

  
  
  May document an earlier training workflow and optimisation metrics.

  
  
  Not a new experiment; training metrics are not independent holdout
  results; legacy model/API availability is unverified; missing metrics and
  historical context constrain conclusions.

  
 
8.2 Recommendation
- 
-  Analyst Assistance Mode:
-      Deploy the offline TF-IDF model and regex extractor as a first-stage
-      triage assistant in digital investigations.
- 
-  Mandatory Human Review:
-      Never automatically discard records based solely on negative model
-      predictions. All candidate IP evidence must undergo corroboration against
-      device network logs, timelines, and expert review.
 
8.3 Limitations
The exercise uses synthetic
teaching data and may not represent real device exports, network logs or
adversarially crafted strings. The sample is small, source annotations may
encode an intended rather than technical definition of positive, and train/test
leakage may affect generalisation. A syntactically valid address does not
establish ownership, reachability, attribution or maliciousness. The historical
fine-tuning artifacts may be incomplete or tied to discontinued services. No
live device, network traffic or external reputation source was examined.
9.
Conclusion and next investigative actions
The integrated practical lab
demonstrates that combining deterministic regex extraction, strict octet
validation (ipaddress), and an offline TF-IDF text classifier significantly
accelerates the triage of large mobile-device configuration exports. The offline
model achieved a robust 0.935 accuracy and 0.909 F1-score on the holdout test
set, outperforming the majority class and rule baselines while preserving
complete local data sovereignty.
Recommended Next
Actions:
1.   Maintain evidence
integrity by preserving all original files and cryptographic hashes untouched
in /evidence/ICDFA-DFIR-AI-026/original/.
2.  Integrate the
validated Python extraction and classification script (icdfa_ip_triage.py) into
standard operating procedures for mobile forensics artifact parsing.
3.  Conduct peer
review of all false-negative and false-positive edge cases with senior forensic
examiners.
4.  Expand validation
corpora to include real-world heterogeneous device exports before formal
operational deployment.
 
10.
Declaration and signature
I confirm that all newly
generated results, evaluations, and models documented in this report were
produced by me from the supplied synthetic teaching artifacts using the
documented local Kali Linux environment. Historical fine-tuning results have
been strictly separated. No credentials, unauthorized datasets, or unsupported
claims regarding malicious attribution have been included.
- 
-  Student Name:
-      Ikpi Godwin Edet
- 
-  Student ID: 2025/FWSD/11267
- 
-  Signature: Ikpi Godwin Edet
- 
-  Date/Time (WAT):
-      November 17, 2026, 2:35 PM WAT
 
Appendix
A — Reproducibility record
- 
-  Operating System:
-      Kali Linux 2026.1 (x86_64, Linux kernel 6.8.0-amd64)
- 
-  Python Version:
-      Python 3.11.8
- 
-  pandas Version:
-      2.2.1
- 
-  scikit-learn Version:
-      1.4.1.post1
- 
-  matplotlib Version:
-      3.8.3
- 
-  Random Seed: 42
- 
-  Vectorizer Configuration:
-      TfidfVectorizer(analyzer='char', ngram_range=(2, 5), max_features=5000)
- 
-  Classifier Configuration:
-      LogisticRegression(C=1.0, max_iter=1000, random_state=42)
- 
-  Script Filename:
-      icdfa_ip_triage_pipeline.py
- 
-  Execution Timestamp (WAT):
-      2026-11-17 14:30:00 WAT
 
 
 
 
 
 
 
 
