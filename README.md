# Data Insights Platform

This repository hosts a robust and automated data processing solution, designed to extract, transform, and analyze data from Excel spreadsheets, providing actionable insights. The project incorporates best practices for code quality, data handling, and continuous integration/delivery using GitHub Actions.

## Project Structure

The key files in this repository are:

*   `execute.py`: The Python script responsible for data processing.
*   `data.xlsx`: The raw source data in Excel format.
*   `data.csv`: The converted CSV version of `data.xlsx`, used by `execute.py`. (Assumed to be committed alongside `data.xlsx` after initial conversion).
*   `index.html`: A responsive web page using Tailwind CSS, serving as a dashboard or entry point to view results.
*   `LICENSE`: The MIT License for this project.
*   `.github/workflows/ci.yml`: The GitHub Actions workflow definition for continuous integration and deployment.
*   `result.json`: (Generated in CI, *not* committed) The output of `execute.py`, containing processed data.

## `execute.py`: Data Processing Script

This Python script, `execute.py`, is the core of the data processing pipeline. It reads raw data, performs necessary cleaning, and aggregates insights.

### Original Problem & Solution

The script was initially prone to errors when encountering inconsistent or non-numeric data in the 'Value' column, or if required columns ('Category', 'Value') were missing. For instance, direct summation on a column containing 'N/A' or text strings would cause a `TypeError` or `ValueError`.

The following non-trivial error fixes have been implemented:

1.  **Robust File Handling:** Added `try-except` blocks to gracefully handle `FileNotFoundError` if `data.csv` is missing.
2.  **Column Validation:** Implemented checks to ensure that essential columns (`Category`, `Value`) exist in the DataFrame, preventing `KeyError` exceptions. If columns are missing, a descriptive `ValueError` is raised.
3.  **Numeric Coercion & NaN Handling:** The `Value` column is now safely converted to a numeric type using `pd.to_numeric(errors='coerce')`. This converts any non-numeric entries into `NaN` (Not a Number) without crashing the script. Subsequently, rows with `NaN` in the 'Value' column are dropped using `dropna(subset=['Value'])`, ensuring only valid numeric data is aggregated.
4.  **Empty DataFrame Handling:** The script now checks if the DataFrame is empty after cleaning. If no valid data remains, it outputs an empty JSON array, maintaining a consistent output structure.
5.  **Comprehensive Error Output:** In case of any error (file not found, value error, or unexpected exception), `result.json` will contain a structured JSON object detailing the error message, aiding in debugging.

The script expects `data.csv` to be present in the same directory, which is generated from `data.xlsx` and committed to the repository. It groups data by `Category` and calculates the sum of `Value` for each category.

## `data.xlsx` & `data.csv`

`data.xlsx` serves as the primary source of raw data. For performance and consistency within the pipeline, `data.xlsx` is converted to `data.csv` and then `data.csv` is committed to the repository. The `execute.py` script then operates on `data.csv`. This ensures that the data format consumed by the script is stable and version-controlled.

## `index.html`: Responsive Dashboard

`index.html` provides a simple, responsive web interface built with Tailwind CSS. It serves as a user-friendly entry point to the project, giving an overview and linking directly to the `result.json` file published via GitHub Pages.

## Continuous Integration and Deployment with GitHub Actions

A GitHub Actions workflow (`.github/workflows/ci.yml`) automates several critical steps on every `push` to the `main` or `master` branch:

1.  **Code Checkout:** Fetches the latest code from the repository.
2.  **Python Setup:** Configures Python 3.11 for the environment.
3.  **Dependency Installation:** Installs `pandas`, `openpyxl` (a common dependency for projects dealing with Excel files, though `execute.py` uses `data.csv` in this version), and `ruff` for linting.
4.  **Ruff Linting:** Runs `ruff check .` to ensure code quality and adherence to style guidelines. Results are visible in the CI logs.
5.  **Script Execution:** Executes `python execute.py > result.json`. This generates the `result.json` file with the processed data.
6.  **GitHub Pages Deployment:** The generated `result.json` is then published to GitHub Pages, making the latest data insights publicly accessible via a static URL.

### Accessing Results

After a successful CI/CD run, `result.json` will be available at your GitHub Pages URL (e.g., `https://<your-username>.github.io/<your-repository>/result.json`). The `index.html` page will also be accessible at `https://<your-username>.github.io/<your-repository>/` and contains a link to the `result.json`.

## Local Setup (Optional)

To run the script locally:

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/your-org/your-repo.git
    cd your-repo
    ```
2.  **Create a virtual environment (recommended):**
    ```bash
    python3.11 -m venv .venv
    source .venv/bin/activate
    ```
3.  **Install dependencies:**
    ```bash
    pip install pandas openpyxl ruff
    ```
4.  **Ensure `data.csv` is present:** (If `data.csv` is not yet converted and committed, you would convert `data.xlsx` first: `python -c "import pandas as pd; pd.read_excel('data.xlsx').to_csv('data.csv', index=False)"`)
5.  **Run the script:**
    ```bash
    python execute.py
    ```
    This will generate `result.json` in your current directory.
6.  **Run Ruff:**
    ```bash
    ruff check .
    ```
