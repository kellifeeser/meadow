# meadow

<u>**M**</u>**icroarray <u>E</u>xpression <u>A</u>nalysis & <u>D</u>ata <u>O</u>rganization <u>W</u>orkflow**

------------------------------------------------------------------------

### Phase 1: Environment & Project Scaffolding

-   **Initialize project**

    -   Create an RStudio project file (`meadow.Rproj`) at the repo root.
    -   Activate **`renv`** to snapshot and manage all package versions (R 4.x, Bioconductor 3.x).

-   **Version control & CI**

    -   Initialize a Git repository, add a `.gitignore`, and make an initial scaffold commit.
    -   Configure a GitHub Actions workflow to run on each push or pull request (or other?):
        -   Restore the environment (`renv::restore()`)
        -   Run each stub script in `scripts/` to ensure they load without errors
    -   Move code/scripts to target destination or stay in Github for now (?)

-   **Directory hierarchy**

    `meadow/ ├── docs/                       │   ├── project_overview.md     │   ├── acronyms_list.md        │   └── requirements.md         │ ├── config/                     │   ├── geo_ids.yaml            │   ├── normalization.yaml      │   └── gene_panel.yaml         │ ├── scripts/                    │   ├── 01_acquire_metadata.R   │   ├── 02_process_microarray.R │   ├── 03_integration_eval.R   │   └── 04_export_ai_ready.R    │ ├── workflows/                  │   └── pipeline_diagram.svg    │ ├── metadata/                   │   ├── study_metadata_template.csv │   └── sample_metadata_template.csv │ ├── reports/                    │   ├── QC_plan.md              │   └── integration_strategy.md │ ├── output/                     │   └── ai_ready_matrix.csv     │ └── README.md`

-   **Draft stub docs & configs**

    -   In `docs/`, draft `project_overview.md`, list acronym candidates in `acronyms_list.md`, capture open questions in `requirements.md`
    -   In `config/`, create commented YAML templates for \`geo_ids\`, \`normalization\`, and \`gene_panel\`

-   **Establish conventions**

------------------------------------------------------------------------

### Phase 2: Data Acquisition & Metadata Extraction

-   Fetch study summaries via `rentrez::entrez_summary()` and download raw & processed microarray data with `GEOquery::getGEO()` into `ExpressionSet` objects.
-   Extract study-level metadata (title, platform, organism, publication date) and sample-level metadata (treatment, timepoint, cell line) into `metadata/study_metadata.csv` and `metadata/sample_metadata.csv`.
-   Flag missing fields and annotate via Llama (using `reticulate`)

------------------------------------------------------------------------

### Phase 3: Microarray Preprocessing, QC & Normalization

-   Detect platform per study (Affymetrix vs. Illumina vs. Agilent) from metadata.
-   Preprocess raw data using the matching Bioconductor package:
    -   **Affymetrix**: `affy` or `oligo`
    -   **Illumina**: `lumi` (or `beadarray` for bead-level)
    -   **Agilent**: `limma::read.maimages()`
-   Qualityassessment with `affyPLM` (probe-level models, NUSE/RLE) and `arrayQualityMetrics` (automated QC report); **filter** out poor-quality arrays.
-   Normalize cleaned data via `preprocessCore::normalize.quantiles()` (quantile) or `vsn::justVSN()` (variance-stabilizing); save as `ExpressionSet` or `SummarizedExperiment`.

------------------------------------------------------------------------

### Phase 4: Cross-Study Integration Evaluation

-   Batch-correct across studies using `sva::ComBat` or `limma::removeBatchEffect`; optionally prototype the **GEDI** workflow.
-   Evaluate integration quality via silhouette scores, PCA clustering, etc., and capture results in `reports/integration_strategy.md`.

------------------------------------------------------------------------

### Phase 5: AI-Ready Data Transformation & Export

-   Target(?) curated gene/protein panel in `config/gene_panel.yaml`.

-   Extract a filtered sample-by-gene matrix from each normalized study(?) and transform to target standards and formatting

-   Flag control samples (mock-infected, cell lines) in metadata.

-   Export the final AI-ready dataset as CSV/HDF5/JSON/other in `output/ai_ready_matrix.{csv,h5}`.
