# Code Visualization Tool with Single LLM Prompt

This repository provides a Python-based tool to generate architecture diagrams from a codebase folder using Mermaid.js and company-provided LLM APIs. The tool uses a single, expertly crafted system prompt to analyze the input folder’s file tree and README, producing a detailed diagram with interactive click events.

## Repository Structure

```
code_visualizer/
├── src/
│   ├── main.py
│   ├── analyzer/
│   │   ├── __init__.py
│   │   ├── file_tree.py
│   │   └── readme_reader.py
│   ├── llm/
│   │   ├── __init__.py
│   │   ├── vertex.py
│   │   ├── open.py
│   │   ├── anthropic.py
│   │   └── llm_client.py
│   ├── diagram/
│   │   ├── __init__.py
│   │   ├── prompts.py
│   │   └── generator.py
│   └── utils/
│       ├── __init__.py
│       ├── config.py
│       └── logger.py
├── tests/
├── docs/
├── requirements.txt
├── .env
├── .gitignore
└── README.md
```

## File Descriptions and Code Snippets

### `src/main.py`

The entry point that parses command-line arguments, retrieves the file tree and README, generates the diagram, and saves the output.

```python
import argparse
from analyzer.file_tree import get_file_tree
from analyzer.readme_reader import get_readme
from diagram.generator import generate_diagram
from llm.llm_client import LLMClient
from utils.config import get_config
from utils.logger import setup_logger

def main():
    parser = argparse.ArgumentParser(description="Generate architecture diagram from code folder")
    parser.add_argument("input_folder", help="Path to the input folder")
    parser.add_argument("--output", "-o", help="Output file for the Mermaid code", default="diagram.mmd")
    args = parser.parse_args()

    logger = setup_logger()
    config = get_config()
    llm_client = LLMClient(provider=config['llm_provider'])

    file_tree = get_file_tree(args.input_folder)
    readme = get_readme(args.input_folder)

    try:
        mermaid_code = generate_diagram(file_tree, readme, llm_client)
        with open(args.output, 'w') as f:
            f.write(mermaid_code)
        logger.info(f"Mermaid code saved to {args.output}")
    except Exception as e:
        logger.error(f"Error generating diagram: {e}")

if __name__ == "__main__":
    main()
```

### `src/analyzer/file_tree.py`

Generates a string representation of the input folder’s file tree.

```python
import os

def get_file_tree(folder_path):
    def walk_directory(directory, prefix=""):
        tree = []
        for item in sorted(os.listdir(directory)):
            path = os.path.join(directory, item)
            if os.path.isdir(path):
                tree.append(f"{prefix}{item}/")
                tree.extend(walk_directory(path, prefix + "  "))
            else:
                tree.append(f"{prefix}{item}")
        return tree

    return "\n".join(walk_directory(folder_path))
```

### `src/analyzer/readme_reader.py`

Reads the README file from the input folder.

```python
import os

def get_readme(folder_path):
    for filename in ["README.md", "readme.md", "README", "readme"]:
        path = os.path.join(folder_path, filename)
        if os.path.exists(path):
            with open(path, 'r', encoding='utf-8') as f:
                return f.read()
    return ""
```

### `src/llm/vertex.py`

Placeholder for the Vertex AI API (replace with your company’s API implementation).

```python
import os
import requests

def call_vertex(prompt):
    api_key = os.getenv("VERTEX_API_KEY")
    # Replace with actual API endpoint and request format
    response = requests.post(
        "https://vertex-api.example.com/v1/generate",
        json={"prompt": prompt},
        headers={"Authorization": f"Bearer {api_key}"}
    )
    return response.json().get('text', '')
```

### `src/llm/open.py`

Placeholder for the OpenAI-compatible API (replace with your company’s API implementation).

```python
import os
import requests

def call_openai(prompt):
    api_key = os.getenv("OPENAI_API_KEY")
    # Replace with actual API endpoint and request format
    response = requests.post(
        "https://openai-api.example.com/v1/completions",
        json={"prompt": prompt},
        headers={"Authorization": f"Bearer {api_key}"}
    )
    return response.json().get('text', '')
```

### `src/llm/anthropic.py`

Placeholder for the Anthropic API (replace with your company’s API implementation).

```python
import os
import requests

def call_anthropic(prompt):
    api_key = os.getenv("ANTHROPIC_API_KEY")
    # Replace with actual API endpoint and request format
    response = requests.post(
        "https://anthropic-api.example.com/v1/generate",
        json={"prompt": prompt},
        headers={"Authorization": f"Bearer {api_key}"}
    )
    return response.json().get('text', '')
```

### `src/llm/llm_client.py`

Provides a unified interface to call the selected LLM provider.

```python
class LLMClient:
    def __init__(self, provider):
        if provider == 'vertex':
            from .vertex import call_vertex
            self.call = call_vertex
        elif provider == 'openai':
            from .open import call_openai
            self.call = call_openai
        elif provider == 'anthropic':
            from .anthropic import call_anthropic
            self.call = call_anthropic
        else:
            raise ValueError(f"Unknown LLM provider: {provider}")

    def generate(self, prompt):
        return self.call(prompt)
```

### `src/diagram/prompts.py`

Defines the single system prompt for generating the architecture diagram.

```python
SYSTEM_PROMPT = """
You are the world's best system design architect, renowned for your ability to understand and visualize complex software architectures. You are tasked with generating an architecture diagram for a software project based on its file tree and README.

You will be provided with:

- The complete file tree of the project, enclosed in <file_tree> tags.

- The README file of the project, enclosed in <readme> tags.

Your task is to:

1. Thoroughly analyze the file tree and README to understand the project's structure, purpose, and architectural design.

2. Identify all key components of the system, including but not limited to frontend, backend, databases, APIs, services, and any other significant modules.

3. Determine the relationships, interactions, and data flows between these components.

4. Map each key component to its corresponding file or directory path in the project, where possible.

5. Generate a detailed and accurate Mermaid.js diagram that represents the architecture, including all identified components and their relationships.

6. Include click events in the diagram that link to the corresponding file or directory paths for interactivity.

First, provide the mapping of components to paths in the following format:

<component_mapping>

Component1: path/to/component1

Component2: path/to/component2

...

</component_mapping>

Then, provide the Mermaid.js code for the diagram, ensuring that for each component that has a mapping, you include a click event like:

click Component1 "path/to/component1"

Make sure that the component names in the diagram match those in the mapping exactly.

Also, ensure that the diagram is oriented vertically to avoid long horizontal sections, and use appropriate shapes and colors to distinguish different types of components.

Your response should contain only the <component_mapping> section followed by the Mermaid.js code.

Important notes on Mermaid.js syntax:

- Use quotes for node labels that contain special characters, e.g., `Node["Label with spaces"]`.

- Apply class styles to nodes, not directly to subgraphs.

- Ensure relationship labels are enclosed in quotes if they contain spaces, e.g., `A -->|"calls API"| B`.

- Do not assign aliases to subgraphs; use `subgraph "Subgraph Name"`.

Strive for excellence in your analysis and diagram generation, as if you were presenting to a team of senior engineers.
"""
```

### `src/diagram/generator.py`

Handles the diagram generation by sending the prompt to the LLM and extracting the Mermaid.js code.

```python
import re
from .prompts import SYSTEM_PROMPT
from ..llm.llm_client import LLMClient

def generate_diagram(file_tree, readme, llm_client: LLMClient):
    prompt = f"{SYSTEM_PROMPT}\n<file_tree>\n{file_tree}\n</file_tree>\n<readme>\n{readme}\n</readme>"
    response = llm_client.generate(prompt)
    
    # Extract Mermaid code after </component_mapping>
    match = re.search(r'</component_mapping>\s*(.*)', response, re.DOTALL)
    if not match:
        raise ValueError("Failed to find Mermaid code in response")
    mermaid_code = match.group(1).strip()
    return mermaid_code
```

### `src/utils/config.py`

Loads configuration settings from environment variables.

```python
import os
from dotenv import load_dotenv

def get_config():
    load_dotenv()
    return {
        'llm_provider': os.getenv('LLM_PROVIDER', 'vertex'),
    }
```

### `src/utils/logger.py`

Sets up logging for the application.

```python
import logging

def setup_logger():
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s - %(levelname)s - %(message)s'
    )
    return logging.getLogger(__name__)
```

### `requirements.txt`

Lists required Python packages.

```
requests
python-dotenv
```

### `.env`

Stores environment variables (not committed to version control).

```
VERTEX_API_KEY=your_vertex_api_key
OPENAI_API_KEY=your_openai_api_key
ANTHROPIC_API_KEY=your_anthropic_api_key
LLM_PROVIDER=vertex
```

### `.gitignore`

Ignores unnecessary files.

```
__pycache__/
*.pyc
.env
*.mmd
```

### `README.md`

Provides project overview and usage instructions.

```markdown
# Code Visualization Tool

This tool generates architecture diagrams from a codebase folder using Mermaid.js and company-provided LLM APIs.

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/your-org/code-visualizer.git
   cd code_visualizer
```

2. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

3. Set up environment variables in `.env`:

   ```plaintext
   VERTEX_API_KEY=your_vertex_api_key
   OPENAI_API_KEY=your_openai_api_key
   ANTHROPIC_API_KEY=your_anthropic_api_key
   LLM_PROVIDER=vertex
   ```

## Usage

Run the tool with the input folder path:

```bash
python src/main.py /path/to/your/codebase --output diagram.mmd
```

The generated Mermaid.js code will be saved to `diagram.mmd`. Render it using tools like Mermaid Live Editor.

## How It Works

1. The tool analyzes the input folder to extract the file tree and README content.
2. It sends a single, optimized prompt to the selected LLM API to:
   - Identify key components and their relationships.
   - Map components to files/directories.
   - Generate Mermaid.js code with interactive click events.
3. The output is saved as a `.mmd` file.

## Contributing

Submit issues or pull requests to the GitHub repository.

## License

MIT License

```

## Implementation Details

### System Prompt Design
The single system prompt is designed to emulate the expertise of a world-class system design architect. It instructs the LLM to:
- Analyze the file tree and README thoroughly to understand the project’s structure and purpose.
- Identify key components (e.g., frontend, backend, databases) and their interactions.
- Map components to specific files or directories for interactivity.
- Generate syntactically correct Mermaid.js code with vertical orientation, appropriate shapes, colors, and click events.
- Adhere to Mermaid.js syntax rules to ensure renderable output.

The prompt emphasizes detail and accuracy, suitable for presenting to senior engineers, and is flexible enough to handle various project types, from web applications to libraries.

### Diagram Generation Process
The `generate_diagram` function sends the prompt with the file tree and README to the LLM, then extracts the Mermaid.js code from the response. The LLM’s output includes a `<component_mapping>` section (used for click events) followed by the diagram code. The function uses regex to isolate the Mermaid code, ensuring robust parsing.

### LLM Integration
The `llm_client.py` module provides a unified interface to your company’s LLM APIs, selected via the `LLM_PROVIDER` environment variable. The placeholder API implementations (`vertex.py`, `open.py`, `anthropic.py`) must be updated with your actual API endpoints and authentication details.

### Error Handling
Basic error handling is included in `main.py` and `generator.py` to catch issues like missing Mermaid code in the LLM response. For production use, consider adding:
- Validation of Mermaid.js syntax.
- Fallback mechanisms for LLM failures.
- Logging of intermediate outputs for debugging.

### Extensibility
The modular structure supports extensions, such as:
- Adding support for other diagram formats (e.g., PlantUML).
- Including code file analysis for deeper insights.
- Generating HTML output for direct rendering.

## Comparison with Existing Tools
Several tools generate architecture diagrams, but this solution is tailored for your company’s APIs and requirements. Here’s a comparison:

| Tool            | Diagram Type       | LLM Integration         | Open Source | Custom API Support |
|-----------------|--------------------|-------------------------|-------------|--------------------|
| Structurizr     | C4 Model           | No                      | Yes         | No                 |
| DiagramGPT      | Multiple Types     | Yes (GPT-4)             | No          | Limited            |
| This Tool       | Mermaid.js         | Yes (Custom APIs)       | Custom      | Yes                |

- **Structurizr**: Uses a DSL for diagrams, lacking LLM-driven analysis ([Structurizr](https://structurizr.com/)).
- **DiagramGPT**: Leverages GPT-4 but is not optimized for custom APIs ([Eraser DiagramGPT](https://www.eraser.io/diagramgpt)).
- **This Tool**: Integrates with your APIs, generates interactive Mermaid.js diagrams, and is customizable.

## Best Practices
- **Security**: Store API keys in `.env` and ensure `.gitignore` includes it.
- **Logging**: Use the logger to track execution and diagnose issues.
- **Testing**: Add unit tests in `tests/` for file analysis, LLM response parsing, and diagram generation.
- **Documentation**: Maintain detailed guides in `docs/` for onboarding and maintenance.

## Future Improvements
- **Interactive Rendering**: Generate an HTML file to render the Mermaid diagram in browsers.
- **Caching**: Cache LLM responses to reduce API costs for repeated runs.
- **Validation**: Check Mermaid code syntax before saving.
- **Scalability**: Optimize for large codebases by filtering irrelevant files (e.g., excluding `node_modules`).

## Example Workflow
For a project with the following structure:
```

my_app/ ├── frontend/ │ ├── src/ │ │ └── app.js │ └── package.json ├── backend/ │ ├── src/ │ │ └── server.js │ └── config/ │ └── db.js ├── README.md

```

1. Run `python src/main.py my_app --output diagram.mmd`.
2. The tool generates a file tree and reads `README.md`.
3. The LLM processes the prompt, producing a mapping (e.g., `Frontend: frontend/`, `Backend: backend/`) and Mermaid.js code.
4. The output `diagram.mmd` might look like:
```

flowchart TD subgraph "Frontend" FE\["Frontend (React)"\]:::frontend end subgraph "Backend" BE\["Backend (Node.js)"\]:::backend end FE --&gt;|"API calls"| BE BE --&gt;|"queries"| DB\[(Database)\] click FE "frontend/" click BE "backend/" classDef frontend fill:#f9f,stroke:#333 classDef backend fill:#bbf,stroke:#333

```

5. Render the diagram in [Mermaid Live Editor](https://mermaid.live/) or integrate it into documentation.

## Limitations
- **Complex Projects**: The single-prompt approach may struggle with very large or poorly documented codebases, potentially requiring a multi-step process for better accuracy.
- **LLM Dependence**: The quality of the diagram depends on the LLM’s ability to interpret the prompt and codebase correctly.
- **API Reliability**: Ensure your company’s APIs are robust, as the tool relies on consistent LLM responses.

## Key Citations
- [Structurizr: C4 Model Diagram Tool](https://structurizr.com/)
- [Eraser DiagramGPT: AI-Powered Diagram Generation](https://www.eraser.io/diagramgpt)
- [Mermaid Live Editor: Render Mermaid.js Diagrams](https://mermaid.live/)
```