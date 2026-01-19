# Resume CV

RenderCV-based resume using YAML input.

## Quick Start

```bash
# Install uv if not already installed
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create virtual environment and install RenderCV
uv venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
uv pip install rendercv

# Generate PDF
rendercv render Raunak_Burrows_CV.yaml
```

Output will be in `rendercv_output/` directory.
