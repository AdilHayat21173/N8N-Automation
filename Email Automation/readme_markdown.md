# README

## Project Overview
This repository contains two n8n automation workflows designed to generate and manage personalized cold outreach emails for Luca Barberis's consulting business.

---

## Project 1: Lucas - Lead Processing Workflow

### Purpose
Automatically generates personalized cold emails for leads from Google Sheets based on different campaign types (Apollo Hiring, Fast Growth, CB Non US).

### Key Features
- **Automated Scheduling**: Runs every 2 minutes
- **Multi-Campaign Support**: Processes three distinct campaign types simultaneously
- **AI-Powered Email Generation**: Uses Google Gemini to create personalized emails
- **Google Sheets Integration**: Reads leads and updates email content back to sheets

### Workflow Components

#### 1. **Trigger**
- Schedule Trigger: Executes every 2 minutes

#### 2. **Data Sources** (3 parallel branches)
- **Apollo Hiring**: Filters for campaign "APOLLO HIRING" with empty emails
- **FAST GROWTH**: Filters for campaign "FAST GROWTH" with empty emails  
- **CB NON US**: Filters for campaign "CB NON US" with empty emails

#### 3. **AI Agents** (3 parallel agents)
Each agent:
- Analyzes prospect data (name, company, industry, location, etc.)
- Researches company website and LinkedIn profiles
- Generates personalized 50-70 word paragraphs
- Selects appropriate email template (B2B or B2C)
- Inserts personalization into complete email

#### 4. **Sheet Updates** (3 parallel updates)
- Updates corresponding rows in Google Sheets with generated emails
- Matches records by email address

### Email Templates
- **B2C Template**: For consumer-facing businesses
- **B2B Template**: For business-to-business companies

### Technologies Used
- n8n workflow automation
- Google Sheets API
- Google Gemini Chat Model (AI)
- Google OAuth2 authentication

---

## Project 2: Luca 2 - Crunchbase Lead Enrichment

### Purpose
Enriches Crunchbase leads by analyzing company websites, determining business models, assigning appropriate email templates, and generating personalized outreach emails.

### Key Features
- **Automated Scheduling**: Runs every minute
- **Website Analysis**: Extracts company overview, business model, industry, and distribution channels
- **Template Matching**: 18 specialized email templates based on business type
- **Error Handling**: Fallback to generic template if analysis fails

### Workflow Components

#### 1. **Trigger**
- Schedule Trigger: Executes every minute

#### 2. **Data Retrieval**
- Fetches rows from Google Sheets where Email column is empty
- Source: "Crunchbase 1-129243" spreadsheet

#### 3. **Website Scraping**
- HTTP Request to fetch company website content
- Error handling with retry logic (max 2 attempts)

#### 4. **Conditional Logic**
- **True Branch**: If website fetch fails → assigns generic template
- **False Branch**: If successful → proceeds to AI analysis

#### 5. **AI Analysis Chain** (3 sequential agents)

**Agent 1 - Company Insight Extractor:**
- Analyzes website content
- Extracts: company overview, business model, industry, distribution channels, location

**Agent 2 - Template Selector:**
- Matches company to most relevant email template
- 18 available templates: Marketplace, SaaS, AI, eCommerce, DTC, Social Commerce, Ad Tech, Web3, Gig Economy, Fashion, Beauty, Sport, F&B, Retail, Travel, Generic B2C, Generic B2B, Generic

**Agent 3 - Email Generator:**
- Retrieves complete email copy for selected template
- Personalizes with founder name and LinkedIn profile

#### 6. **Sheet Updates**
- Updates Google Sheets with campaign type and generated email
- Matches records by website URL (success path) or contact_email (error path)

### Email Template Categories

**Industry-Specific:**
- Fashion, Beauty, Sport, F&B, Retail, Travel

**Business Model:**
- Marketplace, SaaS, AI, eCommerce, DTC, Social Commerce, Ad Tech, Web3, Gig Economy

**Generic Fallbacks:**
- Generic B2C, Generic B2B, Generic

### Technologies Used
- n8n workflow automation
- Google Sheets API
- Google Gemini Chat Model (AI)
- HTTP Request for web scraping
- Google OAuth2 authentication

---

## Setup Instructions

### Prerequisites
1. n8n instance (self-hosted or cloud)
2. Google Account with Sheets API access
3. Google Gemini API credentials

### Configuration Steps

1. **Import Workflows**
   - Import JSON files into n8n

2. **Configure Google Sheets Credentials**
   - Set up Google OAuth2 connection
   - Grant access to relevant spreadsheets

3. **Configure Google Gemini API**
   - Add API key for Google Gemini Chat Model

4. **Update Sheet References**
   - Modify document IDs and sheet names to match your Google Sheets

5. **Adjust Schedules** (Optional)
   - Modify trigger intervals based on your needs

---

## Data Requirements

### Project 1 - Input Fields
- first_name, last_name, email, name, title
- COMPANY NAME, website_url, linkedin_url, organization_linkedin_url
- country, industry, keywords, estimated_num_employees
- CAMPAIGN, GROUP CAMPAIGN, Email (output field)

### Project 2 - Input Fields
- title, founders, linkedin, website, contact_email
- phone_number, company_type, funding details
- Category fields (1-25)
- Compaign, Email (output fields)

---

## Error Handling
- Retry logic on HTTP requests (2 attempts)
- Conditional fallbacks for failed website analysis
- Generic email templates as last resort

---

## Best Practices
- Monitor execution logs for errors
- Regularly review generated emails for quality
- Adjust AI prompts based on output quality
- Keep Google Sheets credentials secure
- Set reasonable execution intervals to avoid rate limits

---

## Support & Maintenance
- Review workflow execution history in n8n
- Update AI prompts as email performance improves
- Maintain Google Sheets data hygiene
- Monitor API usage and costs

---

## License
Internal use only - Luca Barberis Consulting