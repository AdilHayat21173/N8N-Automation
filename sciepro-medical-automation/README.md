# SciePro Medical Automation Workflows

AI-powered automation workflows for medical content generation and translation using n8n and OpenAI.

## 📋 Overview

This repository contains two n8n workflows designed to automate medical content creation for SciePro's stock catalog:

1. **Description Generator** - Creates professional medical illustration descriptions
2. **Translation System** - Translates anatomical terms into 8 languages

## 🚀 Workflows

### 1. Medical Illustration Description Generator

Automatically generates 250-350 word professional descriptions for medical illustrations using AI vision and expert medical writing standards.

**Features:**
- ✅ AI vision analysis of medical images
- ✅ Expert-level anatomical descriptions
- ✅ Clinical relevance and use cases
- ✅ Batch processing (up to 500 items)
- ✅ Google Drive & Sheets integration

**Process Flow:**
```
Form → Fetch Data → Loop → Analyze Image → Generate Description → Update Sheet
```

[📖 Full Documentation](./description-making/)

---

### 2. Medical Terminology Translation System

Translates English medical and anatomical terms into 8 languages with accurate medical terminology.

**Features:**
- ✅ 8 language support (Japanese, Spanish, Portuguese, Russian, Chinese, Arabic, French, German)
- ✅ Medical terminology accuracy
- ✅ Structured JSON output
- ✅ Batch processing with filtering
- ✅ Google Sheets integration

**Process Flow:**
```
Form → Fetch Terms → Filter → Loop → AI Translation → Update Sheet
```

[📖 Full Documentation](./translation/)

## 🛠️ Technology Stack

| Component | Technology |
|-----------|-----------|
| Automation Platform | n8n |
| AI Models | OpenAI GPT-4, GPT-4 Vision, GPT-5.2 |
| Storage | Google Drive |
| Database | Google Sheets |
| Language | JavaScript (n8n nodes) |

## 📦 Installation

### Prerequisites

- n8n instance (self-hosted or cloud)
- Google Drive API credentials
- Google Sheets API credentials
- OpenAI API key

### Setup Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/sciepro-medical-automation.git
   cd sciepro-medical-automation
   ```

2. **Import workflows to n8n**
   - Open n8n interface
   - Go to Workflows → Import from File
   - Import `Description-making.json` and `Translation.json`

3. **Configure credentials**
   - Add Google Drive OAuth2 credentials
   - Add Google Sheets OAuth2 credentials
   - Add OpenAI API credentials

4. **Update configuration**
   - Update Google Drive folder IDs
   - Update Google Sheets spreadsheet IDs
   - Adjust batch processing ranges if needed

5. **Activate workflows**
   - Test each workflow with sample data
   - Activate for production use

## 📁 Repository Structure

```
sciepro-medical-automation/
├── description-making/
│   ├── Description-making.json       # n8n workflow file
│   └── README.md                     # Detailed documentation
├── translation/
│   ├── Translation.json              # n8n workflow file
│   └── README.md                     # Detailed documentation
└── README.md                         # This file
```

## 🔧 Configuration

### Description Generator
- **Source Sheet**: SciePro - Stock Meta (Null tab)
- **Batch Size**: Up to 500 records
- **AI Model**: GPT-4 with Vision
- **Output**: 250-350 word descriptions

### Translation System
- **Source Sheet**: SciePro 3d Anatomy - List II
- **Processing Range**: Rows 10,000-13,000 (configurable)
- **AI Model**: GPT-5.2
- **Output**: JSON with 8 language translations

## 📊 Supported Languages (Translation)

| Language | Code | Example |
|----------|------|---------|
| Japanese | ja | 正中 |
| Spanish | es | mediano |
| Portuguese (Brazil) | pt-BR | mediano |
| Russian | ru | срединный |
| Chinese | zh | 正中 |
| Arabic | ar | أوسط |
| French | fr | médian |
| German | de | median |

## 🎯 Use Cases

**Description Generator:**
- Medical stock photography catalogs
- Educational resource libraries
- Healthcare marketing materials
- Clinical documentation systems

**Translation System:**
- Multilingual medical databases
- International medical education platforms
- Global healthcare applications
- Medical terminology standardization

## 📝 Requirements

### API Access
- OpenAI API (GPT-4, GPT-5.2 access)
- Google Cloud Project with enabled APIs:
  - Google Drive API
  - Google Sheets API

### n8n Version
- Minimum: n8n v1.0.0
- Recommended: Latest stable version

## 🤝 Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## 📄 License

[Add your license here]

## 👥 Authors

SciePro Medical Content Team

## 📞 Support

For questions or issues, please open an issue in this repository.

## 🔒 Security

- Never commit API keys or credentials
- Use environment variables for sensitive data
- Keep workflow files sanitized before sharing

## ⚠️ Important Notes

- **Description Generator**: Processes null/empty description fields only
- **Translation System**: Currently inactive - activate when needed
- Both workflows use sequential processing to avoid rate limits
- Ensure proper API quotas before running large batches

## 🚦 Status

| Workflow | Status |
|----------|--------|
| Description Generator | ✅ Active |
| Translation System | ⏸️ Inactive |

---

**Last Updated**: February 2026
