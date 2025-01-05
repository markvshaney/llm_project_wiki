my_project/
├── my_project_env/        # Virtual environment directory
├── models/               # Local model storage
│   └── ollama/          # Ollama model files
├── memory/
│   ├── __init__.py
│   ├── vector_store.py  # Chroma implementation
│   └── database.py      # SQLite implementation
├── agents/              # CrewAI agents
│   ├── __init__.py
│   ├── researcher.py
│   ├── writer.py
│   └── custom_agents.py
├── scraping/            # Web scraping components
│   ├── __init__.py
│   ├── selenium_scraper.py
│   ├── bs4_scraper.py
│   └── utils.py
├── chains/              # LangChain components
│   ├── __init__.py
│   ├── prompts/
│   │   └── templates.py
│   ├── processors/
│   │   └── text_processor.py
│   └── memory_chain.py
├── llm/                 # LLM configurations
│   ├── __init__.py
│   ├── ollama_config.py
│   └── anything_config.py
├── storage/
│   ├── chroma_db/      # Chroma vector storage
│   └── memory.db       # SQLite database
├── config/
│   ├── __init__.py
│   └── settings.py     # Global settings
├── requirements.txt
└── app.py
