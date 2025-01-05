import os

# Define the wiki pages with Windows-safe names
pages = [
    "Home.md",
    "Project-Structure.md",
    "Development-Setup.md",
    "Setup-VS-Code-Miniconda.md",
    "Setup-Ollama-Model.md",
    "Tools-Overview.md",
    "Guide-VS-Code.md",
    "Guide-Conda.md",
    "Guide-Ollama.md",
    "Guide-CrewAI.md",
    "Guide-Selenium.md",
    "Guide-BeautifulSoup.md",
    "Guide-AnythingLLM.md",
    "Guide-LangChain.md",
    "Guide-Docker.md",
    "Guide-WSL.md"
]

# Directory where the old files are located
old_files_directory = "path/to/old/files"

# Create placeholder content for each file
placeholder_content = """# {title}

{content}

## Navigation
- [Home](Home)
- [Project Structure](Project-Structure)
- [Development Setup](Development-Setup)
- [Tools Overview](Tools-Overview)
"""

# Create the files
for page in pages:
    # Convert filename to title (remove .md and replace hyphens with spaces)
    title = os.path.splitext(page)[0].replace('-', ' ')
    
    # Determine the path to the old file
    old_file_path = os.path.join(old_files_directory, page)
    
    # Read content from old file if it exists
    if os.path.exists(old_file_path):
        with open(old_file_path, 'r', encoding='utf-8') as f:
            content = f.read()
    else:
        content = "[This is a placeholder. Original content to be migrated.]"
    
    # Create new file with content
    with open(page, 'w', encoding='utf-8') as f:
        f.write(placeholder_content.format(title=title, content=content))
    
    print(f"Created {page}")

print("\nNext steps:")
print("1. Initialize git repository:")
print("   git init")
print("2. Add all files:")
print("   git add .")
print('3. Make initial commit:')
print('   git commit -m "Initial wiki structure with valid filenames"')
print("4. Add GitHub wiki remote:")
print("   git remote add origin https://github.com/markvshaney/LLM_Project.wiki.git")
print("5. Push to GitHub (you may need to force push):")
print("   git push -f origin master")