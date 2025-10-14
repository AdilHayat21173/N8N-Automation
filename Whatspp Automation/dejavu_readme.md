# DejaVu Tours - Luna WhatsApp Assistant 🌴

An intelligent multilingual WhatsApp sales assistant built with n8n that automates customer qualification, package recommendations, and lead management for DejaVu Tours in Bali.

## 📋 What Does Luna Do?

Luna is a WhatsApp bot that handles the entire customer journey:

- **Greets customers** in their language and qualifies them with simple questions
- **Recommends personalized Bali packages** (Family or Couple)
- **Answers package questions** using AI-powered search
- **Schedules sales calls** based on business hours
- **Prevents spam** with intelligent rate limiting
- **Sends weekly reports** to track leads

## 🏗️ Technical Architecture

### Core Technologies

- **n8n** - Workflow automation platform
- **Airtable** - Customer data and package catalog
- **Pinecone** - Vector database for package information retrieval
- **OpenAI** - GPT-4.1 for conversations and qualification
- **MongoDB** - Chat history and conversation memory

### How It Works

```
WhatsApp Message → Spam Check → User Lookup → AI Agent → Response
                                      ↓
                          Update Airtable & MongoDB
```

## 🔄 Customer Journey States

Luna guides users through four states:

1. **NEW** - First-time visitor, gets a friendly greeting
2. **QUALIFYING** - Collects: name, country, dates, package type, group size
3. **SELECTING** - Generates package link, answers questions (max 10)
4. **RETURNING** - Recognizes past customers, offers to continue or explore new packages

## 📊 Database Structure

### Airtable Base: `DejaVu_tours`

#### Table 1: Qualification_table
Stores customer information and conversation state.

| Field | Type | Description |
|-------|------|-------------|
| `wa_id` | String | WhatsApp ID (unique) |
| `Full Name` | String | Customer's name |
| `country` | String | Country of origin |
| `group size` | Number | Number of travelers |
| `travel dates` | String | Format: YYYY-MM-DD to YYYY-MM-DD |
| `Package Type` | Options | Family or Couple |
| `Selected Package Name` | String | Generated package URL |
| `no of questions` | Number | Tracks questions asked (max 10) |
| `status` | Options | NEW/QUALIFYING/SELECTING/RETURNING |

#### Table 2: rate_limiting
Tracks message frequency to prevent spam.

| Field | Type |
|-------|------|
| `wa_id` | String |
| `message` | String |
| `Created time` | Auto-generated |

#### Table 3: List (Blocklist)
Manages blocked users.

| Field | Type |
|-------|------|
| `wa_id` | String |
| `Name` | String |
| `Status` | Normal or Blocked |

## 🎯 Key Features

### 1. Intelligent Package Recommendation

Luna generates custom package links based on trip duration:

- **Family Packages**: `https://dejavubali.com/FM-[DAYS]D[NIGHTS]N`
- **Couple Packages**: `https://dejavubali.com/HM-[DAYS]D[NIGHTS]N`

Example: 8-day trip → `FM-8D7N` or `HM-8D7N` (8 Days, 7 Nights)

### 2. Smart Question Quota

- Each user can ask **up to 10 detailed questions** about packages
- Uses Pinecone vector search to find relevant answers
- After 10 questions, suggests scheduling a call with a specialist

### 3. Group Size Handling

- Groups **≤10 people**: Self-service flow continues
- Groups **>10 people**: Automatically routes to a specialist for custom planning

### 4. Spam Protection

**Rate Limiting**: Blocks users sending ≥10 messages in 5 minutes

**Keyword Detection**: Flags messages containing:
- "click here", "free money", "winner"
- "urgent action", "limited time", "act now"
- "bitcoin", "crypto investment"

**URL Spam**: Blocks messages with more than 3 URLs

### 5. Business Hours Management

**Operating Hours**: 09:00 - 18:00 (server timezone)

- **During hours**: Notifies sales team for immediate follow-up
- **After hours**: Sends booking link `https://schedule.dejavubali.com/`

### 6. Multilingual Support

Luna detects and responds in the customer's language using OpenAI's language detection capabilities.

## 🤖 AI Models Used

| Purpose | Model | Temperature |
|---------|-------|-------------|
| Greeting & Qualification | GPT-4.1-mini | 0.2-0.3 |
| Package Selection | GPT-4.1 | Default |
| Embeddings (Pinecone) | OpenAI text-embedding | 512 dimensions |

## 💾 Conversation Memory

All conversations are stored in MongoDB:
- **Database**: `dejaVu`
- **Collection**: `dejaVu`
- **Key**: `wa_id` (WhatsApp ID)

This allows Luna to remember context across conversations and recognize returning customers.

## 📈 Weekly Reporting

An automated scheduler generates weekly reports showing:
- New leads from the past 7 days
- Conversion status
- Customer demographics

## 🚀 Setup Requirements

1. **n8n instance** (self-hosted or cloud)
2. **Airtable account** with base ID: `appE9PPe6PtxpWYhP`
3. **Pinecone account** with index: `dejavu` (512 dimensions)
4. **OpenAI API key**
5. **MongoDB instance**
6. **WhatsApp Business API** access

## 📝 Configuration Checklist

- [ ] Set up Airtable base with all three tables
- [ ] Configure Pinecone index with package information
- [ ] Add OpenAI API credentials to n8n
- [ ] Connect MongoDB for chat history
- [ ] Set up WhatsApp webhook trigger
- [ ] Configure business hours timezone
- [ ] Enable weekly report scheduler
- [ ] Test spam detection rules

## 🛠️ Workflow Logic

### New Customer Flow
```
1. Receive WhatsApp message
2. Check blocklist → If blocked, stop
3. Rate limit check → If spam, block
4. Look up user in Airtable
5. If new → Create record (status: NEW)
6. Send greeting → Set status: QUALIFYING
7. Ask qualification questions one by one
8. Generate package link → Set status: SELECTING
9. Answer questions (up to 10) using Pinecone
10. Offer call scheduling
```

### Returning Customer Flow
```
1. Recognize user by wa_id
2. Greet by name + mention previous package
3. Offer to continue or explore alternatives
4. Apply same quota and scheduling logic
```

## 🎨 Sample Interactions

**First Message:**
> Luna: "Welcome to DejaVu Tours! 🌴 I'm Luna, your Bali travel assistant. May I have your name?"

**After Qualification:**
> Luna: "Perfect! Based on your 8-day family trip, I recommend this package: [FM-8D7N link]. Would you like more details or schedule a call with our team?"

**Quota Reached:**
> Luna: "You've reached the limit for detailed package questions. Let me schedule a call with our specialist to answer all your questions!"

## 📞 Support

For technical issues or workflow modifications, contact the development team or refer to the n8n workflow documentation.

---

**Built with ❤️ for DejaVu Tours Bali**