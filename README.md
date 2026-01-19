# Resume CV

RenderCV-based resume using YAML input.

## Quick Start

```bash
# Activate the existing virtual environment
source rendercv_env/bin/activate  # On Windows: rendercv_env\Scripts\activate

# Generate PDF (use any of the YAML files)
rendercv render Raunak_Burrows_CV.yaml
rendercv render Raunak_Burrows_CV_Azure_AI_Engineer.yaml
rendercv render Raunak_Burrows_CV_FullStack_PriView.yaml
```

Output will be in `rendercv_output/` directory.

## Available CV Versions

- **`Raunak_Burrows_CV.yaml`** - General software engineer CV
- **`Raunak_Burrows_CV_Azure_AI_Engineer.yaml`** - Tailored for Azure + AI Engineering roles (Robert Walters JD)
- **`Raunak_Burrows_CV_FullStack_PriView.yaml`** - Tailored for Full Stack Engineer roles (PriView JD)

## Initial Setup (if rendercv_env doesn't exist)

```bash
# Install uv if not already installed
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create virtual environment and install RenderCV
python3 -m venv rendercv_env
source rendercv_env/bin/activate  # On Windows: rendercv_env\Scripts\activate
pip install rendercv
```
