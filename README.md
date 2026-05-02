<h1>OLD CODE</h1>
name: pytest CI 

 

**Trigger: run on push or pull request to main branch**

on: 

  push: 

    branches: [ main ] 

  pull_request: 

    branches: [ main ] 

 

jobs: 

  test: 

    # Use the latest Ubuntu runner 

    runs-on: ubuntu-latest 

 

    # Test against multiple Python versions 

    strategy: 

      matrix: 

        python-version: ['3.10', '3.11', '3.12'] 

 

    steps: 

      # 1. Check out the repository code 

      - name: Checkout code 

        uses: actions/checkout@v4 

 

      # 2. Set up the Python version from the matrix 

      - name: Set up Python ${{ matrix.python-version }} 

        uses: actions/setup-python@v5 

        with: 

          python-version: ${{ matrix.python-version }} 

 

      # 3. Install all dependencies from requirements.txt 

      - name: Install dependencies 

        run: | 

          python -m pip install --upgrade pip 

          pip install -r requirements.txt 

 

      # 4. Run all tests (unit + BDD) with coverage 

      - name: Run pytest with coverage 

        run: | 

          pytest -v \ 

            --cov=calculator \ 

            --cov-report=xml \ 

            --cov-fail-under=80 \ 

            --html=reports/report.html \ 

            --self-contained-html 

 

      # 5. Upload the HTML report as a downloadable artifact 

      - name: Upload test report 

        if: always()   # upload even if tests fail 

        uses: actions/upload-artifact@v4 

        with: 

          name: test-report-py${{ matrix.python-version }} 

          path: reports/ 

-----------------------------------------------------------------------------------------

**🔧 WHAT I FIXED (IMPORTANT)**

🔴 1. Your backslash (\) formatting was broken
❌ Your version:
pytest -v \ 

👉 There is a space after \
👉 In Linux, this breaks line continuation

Result:

Command becomes invalid
Pytest may interpret wrong input → “file not found”

✅ Fixed:
- name: Run tests with pytest
  run: |
    mkdir -p reports
    pytest -v tests --cov=calculator --cov-report=xml --cov-report=term --cov-fail-under=80 --html=reports/report.html --self-contained-html\

👉 No space after \ → correct multiline command

🔴 Problem with \ (backslash)

When you write commands like this in YAML:

run: pytest -v tests \
            --cov=calculator \
            --cov-report=xml
\ means line continuation (join next line)
But YAML + shell + GitHub Actions can be sensitive to spaces and indentation
One small mistake = ❌ pipeline fails with weird errors

👉 Common issues:

Extra space after \
Missing \ on a line
Wrong indentation
Hidden formatting issues
🟢 Why your | method is safer
- name: Run tests with pytest
  run: |
    mkdir -p reports
    pytest -v tests --cov=calculator --cov-report=xml --cov-report=term --cov-fail-under=80 --html=reports/report.html --self-contained-html

This uses:

👉 | (multi-line block in YAML)

Meaning:

Everything below runs as a shell script
No need for \
Each line runs naturally like terminal commands

-------------------------------------------------------------------

🔴 2. Reports folder may not exist
❌ Problem:
--html=reports/report.html

👉 If reports/ folder doesn’t exist → failure or silent issue

✅ Fixed:
mkdir -p reports

👉 Always creates folder before running pytest


🔴 The actual problem
--html=reports/report.html

Pytest (with pytest-html plugin) does NOT create parent directories.

👉 So if reports/ doesn’t exist:

❌ It may fail
❌ Or sometimes skip report generation silently
❌ Or throw a file path error depending on environment
🟢 Your fix is the correct industry practice
mkdir -p reports

✔ Creates the folder if it doesn’t exist
✔ Does nothing if it already exists
✔ Prevents pipeline crashes

💡 Why this matters in CI/CD (very important)

In your local machine:

Folder might already exist → works fine

In CI (GitHub Actions / Jenkins):

Fresh environment every time 🧼
No folders exist by default

👉 That’s why pipelines fail even if code works locally

🔥 Production-grade version (what companies actually do)
- name: Run tests with pytest
  run: |
    set -e
    mkdir -p reports
    pytest -v tests \
      --cov=calculator \
      --cov-report=xml \
      --cov-report=term \
      --cov-fail-under=80 \
      --html=reports/report.html \
      --self-contained-html
⚠️ One more hidden improvement (most beginners miss this)

If report generation is critical, you can enforce it:

test -f reports/report.html

👉 This checks:

If file exists → OK
If not → ❌ pipeline fails
🧠 Key DevOps mindset takeaway

Always assume:

“Nothing exists unless I create it explicitly.”

That includes:

folders
dependencies
environment variables
tools


---------------------------------------------------------



🔴 ROOT CAUSE

Your YAML is now correct ✅
But your project structure is wrong or incomplete ❌

GitHub Actions is looking here:

/home/runner/work/pwskills_pytest_cicd/pwskills_pytest_cicd/tests

👉 And it doesn’t find that folder

✅ FIX OPTION 1 (BEST PRACTICE)
👉 Create a proper tests/ folder

Your project should look like this:

pwskills_pytest_cicd/
│
├── calculator/
│   └── calc.py
│
├── tests/                ✅ MUST EXIST
│   └── test_calc.py      ✅ MUST START WITH test_
│
├── requirements.txt
└── .github/workflows/ci.yml
✅ Example test file
# tests/test_calc.py

def test_example():
    assert 2 + 2 == 4
✅ FIX OPTION 2 (if your tests are in root)

If your test file is like:

pwskills_pytest_cicd/
  test_calc.py

👉 Then change YAML:

pytest -v .

instead of:

pytest -v tests
✅ FIX OPTION 3 (auto-discovery)

Simplest fix:

pytest -v

👉 Pytest will automatically search for:

test_*.py
*_test.py
🔍 DEBUG (VERY IMPORTANT)

Add this step to confirm what's inside your repo in CI:

- name: Debug files
  run: |
    pwd
    ls -R

👉 This will show:

Whether tests/ exists
Where your files actually are
🧠 WHY THIS HAPPENED

Earlier:

Pytest ran but found nothing

Now:

You explicitly told pytest:

“Go to tests folder”

But:

That folder doesn’t exist → crash ❌


-------------------------------------------------------------------------------------------------
<H1>NEW CODE</H1>


name: Python CI with Pytest

# Trigger workflow on push & pull request
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: ['3.10', '3.11', '3.12']

    steps:
      # 1. Checkout your code from GitHub
      - name: Checkout repository
        uses: actions/checkout@v4

      # 2. Set up Python (based on matrix)
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      # 3. Install dependencies
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      # 4. Debug (optional but VERY useful)
      - name: Show project structure
        run: ls -R

      # 5. Run pytest with coverage
      - name: Run tests with pytest
        run: |
          mkdir -p reports
          pytest -v . --cov=calculator --cov-report=xml --cov-report=term --cov-fail-under=80 --html=reports/report.html --self-contained-html
     
      # 6. Upload test report (even if tests fail)
      - name: Upload test report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-report-py${{ matrix.python-version }}
          path: reports/
