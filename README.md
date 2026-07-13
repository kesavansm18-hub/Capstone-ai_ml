# Capstone-ai_ml
Capstone Project

ai-ml-capstone/
├── .env.txt                       # API key for LLM (gitignored)
├── .gitignore                     # Git ignore rules
├── README.MD                      # This global README
│
├── part1_eda/                     # EDA & Data Cleaning
│   ├── eda_cleaning.py
│   ├── cleaned_data.csv
│   ├── README.md
│   └── data/                      # Raw dataset source
│
├── part2_supervised_ml/           # Supervised Machine Learning
│   ├── supervised_ml.py
│   └── README.md
│
├── part3_Advanced_modeling/       # Advanced Ensemble Modeling
│   ├── advanced_modeling.py
│   ├── best_model.pkl             # Serialized best model
│   ├── README.md
│   └── plt.savefig/               # Saved visualizations
│
├── part4_llm_featured/            # LLM-Powered Feature (Track C)
│   ├── part4_llm_featured.py
│   └── README.md
│
└── plt.savefig/                   # Shared visualizations

# Environment Variables

This project uses environment variables to securely manage the OpenRouter API key for the LLM-powered explanation feature (Part 4). Secrets are never hardcoded in any source file.

# Required Variables

Variable	       |                Description	           |      Example Value
OPENROUTER_API_KEY |	Your OpenRouter API key (required) |	your_api_key_here

# Liabaries:

 Pandas
 Numpy
 os
 Sckit-learn

 # Notebook env

 Jupyter Notebook / JupyterLab

 # Schema 

 JSON Schema

 # Config & env:  
 
 python-dotenv, 
 
 pydantic-settings



 


