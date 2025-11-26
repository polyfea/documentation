# Documentation Site

This repository contains a documentation site built with [MkDocs](https://www.mkdocs.org/) and the [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) theme.

## Features

- Modern Material Design theme
- Light/Dark mode toggle
- Full-text search
- Responsive design
- Automated deployment to GitHub Pages
- Markdown-based content

## Local Development

### Prerequisites

- Python 3.x
- pip

### Setup

1. Clone this repository:
   ```bash
   git clone <your-repo-url>
   cd documentation
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Start the development server:
   ```bash
   mkdocs serve
   ```

4. Open your browser and navigate to `http://127.0.0.1:8000/`

The site will automatically reload when you make changes to the documentation files.

## Building the Site

To build the static site locally:

```bash
mkdocs build
```

This creates a `site/` directory with all the static HTML files.

## GitHub Pages Deployment

This repository is configured for automatic deployment to GitHub Pages using GitHub Actions.

### Initial Setup

1. **Enable GitHub Pages** in your repository settings:
   - Go to Settings → Pages
   - Under "Source", select "GitHub Actions"

2. **Push to main branch**:
   ```bash
   git add .
   git commit -m "Initial documentation setup"
   git push origin main
   ```

3. **Automatic Deployment**:
   - The GitHub Actions workflow will automatically build and deploy your site
   - Check the "Actions" tab to monitor the deployment
   - Once complete, your site will be available at: `https://<username>.github.io/<repository-name>/`

### Manual Deployment

You can also manually trigger the deployment:
- Go to the "Actions" tab in your repository
- Select the "Deploy Documentation to GitHub Pages" workflow
- Click "Run workflow"

## Project Structure

```
documentation/
├── .github/
│   └── workflows/
│       └── deploy.yml       # GitHub Actions workflow
├── docs/
│   ├── index.md            # Home page
│   ├── getting-started.md  # Getting started guide
│   └── about.md            # About page
├── mkdocs.yml              # MkDocs configuration
├── requirements.txt        # Python dependencies
└── README.md              # This file
```

## Customization

### Update Site Information

Edit `mkdocs.yml` to customize:
- `site_name`: Your site name
- `site_description`: Site description
- `site_author`: Your name
- `site_url`: Your GitHub Pages URL

### Add New Pages

1. Create a new `.md` file in the `docs/` folder
2. Add it to the navigation in `mkdocs.yml`:
   ```yaml
   nav:
     - Home: index.md
     - Your New Page: your-page.md
   ```

### Change Theme Colors

Modify the `theme.palette` section in `mkdocs.yml` to change colors:
```yaml
theme:
  palette:
    primary: indigo  # Change to: red, pink, purple, etc.
    accent: indigo
```

## Writing Documentation

Documentation is written in Markdown with additional features:

### Code Blocks

\`\`\`python
def hello_world():
    print("Hello, World!")
\`\`\`

### Admonitions

```markdown
!!! note
    This is a note admonition.

!!! warning
    This is a warning admonition.

!!! tip
    This is a tip admonition.
```

### Tables

```markdown
| Column 1 | Column 2 |
|----------|----------|
| Value 1  | Value 2  |
```

## Troubleshooting

### Site Not Deploying

1. Check that GitHub Pages is enabled in repository settings
2. Verify the workflow ran successfully in the Actions tab
3. Ensure the branch name in `deploy.yml` matches your default branch

### Local Development Issues

- Make sure Python and pip are installed correctly
- Try creating a virtual environment:
  ```bash
  python -m venv venv
  source venv/bin/activate  # On Windows: venv\Scripts\activate
  pip install -r requirements.txt
  ```

## Resources

- [MkDocs Documentation](https://www.mkdocs.org/)
- [Material for MkDocs Documentation](https://squidfunk.github.io/mkdocs-material/)
- [Markdown Guide](https://www.markdownguide.org/)

## License

[Add your license here]
